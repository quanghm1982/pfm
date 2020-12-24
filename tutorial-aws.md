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
#./a-signing-key.sh bsbQ0fM6R2HGYVMDsW + 'yyyy' + Zuwjkst7UMJga2Jb08 20201222 us-east-2 s3
# openssl dgst -sha256 \
-mac HMAC \
-macopt hexkey:25922f37739877f19cb060079619f49ec2523243230864a1b686488bd9ea3493 \
a2-put-aws.sts
```

```python
import sys, os, base64, datetime, hashlib, hmac 

method = 'GET'
service = 's3'
region = 'us-east-2'
a0_hash = 'mybucket'
a1_hash = 'mykey'
host = a0_hash + '.' + service + '.' + service + '.amazonaws.com'
endpoint = 'https://' + a0_hash + '.' + service + '.' + service + '.amazonaws.com/' + a1_hash
request_parameters = ''

def sign(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).digest()

def getSignatureKey(key, dateStamp, regionName, serviceName):
    kDate = sign(('AWS4' + key).encode('utf-8'), dateStamp)
    kRegion = sign(kDate, regionName)
    kService = sign(kRegion, serviceName)
    kSigning = sign(kService, 'aws4_request')
    return kSigning

access_key = 'AKIAWOBX' + mykey1 + 'F3H26CGM'.encode("UTF-8") #os.environ.get('AWS_ACCESS_KEY_ID')
secret_key = '/z7i0pjMs0MPQnhfaO' + mykey2 + '4G2SyF2IyeFUy30c9x'.encode("UTF-8") #os.environ.get('AWS_SECRET_ACCESS_KEY')
if access_key is None or secret_key is None:
    print('No access key is available.')
    sys.exit()

t = datetime.datetime.utcnow()
amzdate = t.strftime('%Y%m%dT%H%M%SZ')
datestamp = t.strftime('%Y%m%d') # Date w/o time, used in credential scope

canonical_uri = '/' + a1_hash

canonical_querystring = request_parameters

canonical_headers = 'content-type:text/plain\nhost:' + host + '\n' + 'x-amz-content-sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855\nx-amz-date:' + amzdate + '\n'

signed_headers = 'content-type;host;x-amz-content-sha256;x-amz-date'

payload_hash = hashlib.sha256(('').encode('utf-8')).hexdigest()

canonical_request = method + '\n' + canonical_uri + '\n' + canonical_querystring + '\n' + canonical_headers + '\n' + signed_headers + '\n' + payload_hash

algorithm = 'AWS4-HMAC-SHA256'
credential_scope = datestamp + '/' + region + '/' + service + '/' + 'aws4_request'
string_to_sign = algorithm + '\n' +  amzdate + '\n' +  credential_scope + '\n' +  hashlib.sha256(canonical_request.encode('utf-8')).hexdigest()

signing_key = getSignatureKey(secret_key, datestamp, region, service)

signature = hmac.new(signing_key, (string_to_sign).encode('utf-8'), hashlib.sha256).hexdigest()

authorization_header = algorithm + ' ' + 'Credential=' + access_key + '/' + credential_scope + ', ' +  'SignedHeaders=' + signed_headers + ', ' + 'Signature=' + signature

headers = {'x-amz-date':amzdate, 'x-amz-content-sha256':payload_hash, 'content-type':'text/plain' 'Authorization':authorization_header}

print("{},{}".format(headers,string_to_sign))
```
