Using minio as a custom backend

#### Powershell

```powershell
$env:AWS_ACCESS_KEY_ID="<minio_user>"
$env:AWS_REGION="<region-name>"
$env:AWS_SECRET_ACCESS_KEY="<minio_user_secret>"

pulumi login "s3://<bucket_name>?endpoint=minio.example.com&s3ForcePathStyle=true"
```

#### Bash

```bash
```powershell
export AWS_ACCESS_KEY_ID="<minio_user>"
export AWS_REGION="<region-name>"
export AWS_SECRET_ACCESS_KEY="<minio_user_secret>"

pulumi login "s3://<bucket_name>?endpoint=minio.example.com&s3ForcePathStyle=true"
```
```