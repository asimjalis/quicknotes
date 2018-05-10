# S3 Demo

## Test API

```python
import boto3

key='1.txt'
file1_path='1.txt'
file2_path='2.txt'

with open(file1_path, "w") as f: f.write("Hello world!!")

s3 = boto3.client('s3')

# Create bucket
bucket_name = 'asimj-demo'
s3.create_bucket(Bucket=bucket_name)
print s3.list_buckets()

# Upload file.
s3.upload_file(file1_path, bucket, key)

# Download file.
s3.download_file(bucket, key, file2_path)

# Check content.
with open(file2_path, "r") as f: print f.read()

# Generate presigned URL.
presigned_url = s3.generate_presigned_url('put_object', {'Bucket':bucket,'Key':key})

print 'Presigned URL: ' + presigned_url
```

## Test Presigned URL

```bash
curl --location --upload-file FILE PRESIGNED_URL
aws s3 cp s3://asimj-demo/1.txt
```
