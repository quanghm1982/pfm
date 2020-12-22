# AWS STEPs by STEPs
```a1-put-aws.creq
PUT
/a5c66a8e7c6/hashes.html

content-type:application/x-www-form-urlencoded; charset=utf-8
host:a5c66a8e7c6.s3.amazonaws.com
x-amz-content-sha256:238a6633e2b4d8de18598ef172414e46d2d956880dcc9af2337adf817c6e32c5
x-amz-date:20201222T104600Z

content-type;host;x-amz-content-sha256;x-amz-date
238a6633e2b4d8de18598ef172414e46d2d956880dcc9af2337adf817c6e32c5
```

```a2-put-aws.sts
AWS4-HMAC-SHA256
20201222T104600Z
20201222/us-east-2/s3/aws4_request
0ff9d0da4ca9262570b88f48439d7744ccd1320a46f144ecb631e862dff8e6ec
```

```a-signing-key.sh
#!/bin/bash
function hmac_sha256 {
  key="$1"
  data="$2"
  echo -n "$data" | openssl dgst -sha256 -mac HMAC -macopt "$key" | sed 's/^.* //'
}

secret="$1"
date="$2"
region="$3"
service="$4"

# Four-step signing key calculation
dateKey=$(hmac_sha256 key:"AWS4$secret" $date)
dateRegionKey=$(hmac_sha256 hexkey:$dateKey $region)
dateRegionServiceKey=$(hmac_sha256 hexkey:$dateRegionKey $service)
signingKey=$(hmac_sha256 hexkey:$dateRegionServiceKey "aws4_request")

echo $signingKey
```

```bash shell
#openssl dgst -sha256 a1-put-aws.creq
#./a-signing-key.sh bsbQ0fM6R2HGYVMDsW6e9tZuwjkst7UMJga2Jb08 20201222 us-east-2 s3
# openssl dgst -sha256 \
-mac HMAC \
-macopt hexkey:25922f37739877f19cb060079619f49ec2523243230864a1b686488bd9ea3493 \
a2-put-aws.sts
```
