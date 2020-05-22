# Website Resiliency

Build a resilient static web hosting solution in AWS. Create a versioned S3 bucket and configure it as a static website.

## Steps

1. Create `S3 bucket`

```shell
aws s3api create-bucket --acl public-read --bucket 0011223322110033 --region us-east-1 
```

2. Set the bucket as `Static website hosting`

```shell
aws s3 website s3://0011223322110033/ --index-document index.html --error-document error.html
```

3. Enable `versioning`
```shell
aws s3api put-bucket-versioning --bucket 0011223322110033 --versioning-configuration Status=Enabled
```

4. Sync local files with S3

```shell
aws s3 sync static/ s3://0011223322110033
```

5. Set the proper policy to allow access
```shell
aws s3api put-bucket-policy --bucket 0011223322110033 --policy file://bucket-policy.json
```

Now we can go to http://0011223322110033.s3-website-us-east-1.amazonaws.com
(`bucket`.s3-website-`region`.amazonaws.com)

6. Make a change in the `html` file

7. Re-sync assets
```shell
aws s3 sync static/ s3://0011223322110033
```

If you go to http://0011223322110033.s3-website-us-east-1.amazonaws.com you will see your changes `live`

8. List versions of the object `index.html`
```
aws s3api list-object-versions --bucket 0011223322110033 --prefix index.html
```

Example output:
```json
{
    "Versions": [
        {
            "LastModified": "2020-05-22T16:55:30.000Z", 
            "VersionId": "YojLpL3Fi_jZeDen2U32hv_nnvZXUjzN", 
            "ETag": "\"e0f83fc1fba22e5bf80ee4f47a446d47\"", 
            "StorageClass": "STANDARD", 
            "Key": "index.html", 
            "Owner": {
                "DisplayName": "your-username", 
                "ID": "07d9e07d9bbe7e3c25cc4000b9d1c9306b8ba45c9a85056e9dad6df72d747cb0"
            }, 
            "IsLatest": true, 
            "Size": 596
        }, 
        {
            "LastModified": "2020-05-22T16:54:48.000Z", 
            "VersionId": "hy9OVM_1NV__MbmWl3B0qD7zvsHsKEHK", 
            "ETag": "\"936f1822bb1e3f3ddf5a7216338c8d1e\"", 
            "StorageClass": "STANDARD", 
            "Key": "index.html", 
            "Owner": {
                "DisplayName": "your-username", 
                "ID": "07d9e07d9bbe7e3c25cc4000b9d1c9306b8ba45c9a85056e9dad6df72d747cb0"
            }, 
            "IsLatest": false, 
            "Size": 618
        }
    ]
}
```

9. Get the object (index.html) by `version id`

```shell
aws s3api get-object --bucket 0011223322110033 --key index.html index.html --version-id hy9OVM_1NV__MbmWl3B0qD7zvsHsKEHK
```

Example output:
```json
{
    "AcceptRanges": "bytes", 
    "ContentType": "text/html", 
    "LastModified": "Fri, 22 May 2020 16:54:48 GMT", 
    "ContentLength": 618, 
    "VersionId": "hy9OVM_1NV__MbmWl3B0qD7zvsHsKEHK", 
    "ETag": "\"936f1822bb1e3f3ddf5a7216338c8d1e\"", 
    "Metadata": {}
}
```

10. Put the object in the bucket

```shell
aws s3api put-object --bucket 0011223322110033 --acl public-read  --key index.html --body index.html --content-type text/html 
```

Example output:
```shell
{
    "VersionId": "NmpoJEICmU5_DVniHL10Ghu110BAa.h_", 
    "ETag": "\"936f1822bb1e3f3ddf5a7216338c8d1e\""
}
```

Now, if you go to http://0011223322110033.s3-website-us-east-1.amazonaws.com you will see the original html.

11. Delete a file

```shell
aws s3 rm s3://0011223322110033/winter.jpg
```

12. Check the version status of the file
```
aws s3api list-object-versions --bucket 0011223322110033 --prefix winter
```

Example output:
```json
{
    "DeleteMarkers": [
        {
            "Owner": {
                "DisplayName": "your-username", 
                "ID": "07d9e07d9bbe7e3c25cc4000b9d1c9306b8ba45c9a85056e9dad6df72d747cb0"
            }, 
            "IsLatest": true, 
            "VersionId": "9WD1UlqdX8Ssmn8VDg77ugCs3S5T5Rp2", 
            "Key": "winter.jpg", 
            "LastModified": "2020-05-22T17:04:25.000Z"
        }
    ], 
    "Versions": [
        {
            "LastModified": "2020-05-22T16:54:48.000Z", 
            "VersionId": "85.RTqN241LFveJRfMSsrN137IsvZON6", 
            "ETag": "\"cd3d419a6144b99a4ad59b5dca24d0c0\"", 
            "StorageClass": "STANDARD", 
            "Key": "winter.jpg", 
            "Owner": {
                "DisplayName": "your-username", 
                "ID": "07d9e07d9bbe7e3c25cc4000b9d1c9306b8ba45c9a85056e9dad6df72d747cb0"
            }, 
            "IsLatest": false, 
            "Size": 77118
        }
    ]
}
```

You can see that now we also have an object on `DeleteMarkers`.

13. Restore object deleting the `delete marker`

```shell
aws s3api delete-object --bucket 0011223322110033 --key winter.jpg --version-id 9WD1UlqdX8Ssmn8VDg77ugCs3S5T5Rp2
```

Example output:
```json
{
    "VersionId": "9WD1UlqdX8Ssmn8VDg77ugCs3S5T5Rp2", 
    "DeleteMarker": true
}
```

You can check the marker was removed...
```shell
aws s3api list-object-versions --bucket 0011223322110033 --prefix winter
```

Example output:
```json
{
    "Versions": [
        {
            "LastModified": "2020-05-22T16:54:48.000Z", 
            "VersionId": "85.RTqN241LFveJRfMSsrN137IsvZON6", 
            "ETag": "\"cd3d419a6144b99a4ad59b5dca24d0c0\"", 
            "StorageClass": "STANDARD", 
            "Key": "winter.jpg", 
            "Owner": {
                "DisplayName": "your-username", 
                "ID": "07d9e07d9bbe7e3c25cc4000b9d1c9306b8ba45c9a85056e9dad6df72d747cb0"
            }, 
            "IsLatest": true, 
            "Size": 77118
        }
    ]
}
```

---

## Delete buckets

### With files but not versioned

```shell
aws s3 rb s3://0011223322110033 --force
```

### With files, versioned and with delete markers

We need -first- to install `boto3`

```shell
pip install boto3
```

Then...

```shell
python
```

And paste...
```python
BUCKET = '0011223322110033'

import boto3

s3 = boto3.resource('s3')
bucket = s3.Bucket(BUCKET)
bucket.object_versions.delete()

bucket.delete()
```