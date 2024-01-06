## IMDS (Instance Metadata Service)
This service allows you to fetch metadata through HTTP requests. This means that every EC2 instance can query the metadata Service to know information about itself, such as its own region and zone.

IMDS also exposes endpoints for these instances to fetch temporary credentials, which will allow them to perform actions based on their instance profiles.

If you want to fetch credentials manually, you need to first request the IMDS token, through the following request:

> curl -X PUT http://169.254.169.254/latest/api/token -H “X-aws-ec2-metadata-token-ttl-seconds: 21600”

After that, you are able to retrieve the temporary credentials providing this token as a header on another API call:

> curl -H “X-aws-ec2-metadata-token: $TOKEN” -v http://169.254.169.254/latest/meta-data/iam/security-credentials/<<ROLE_NAME>>

You can also omit the role name from the query above to see which roles are available. If the role name is provided, the credentials are returned in the following format:

```
{
  "Code" : "Success",
  "LastUpdated" : "2022–08–26T04:55:06Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "<.....>",
  "SecretAccessKey" : "<.....>",
  "Token" : "<.....>",
  "Expiration" : "2022–08–26T11:20:23Z"
}
```

Calling this same endpoint multiple times will return the same credentials unless we are getting close to its “Expiration”. At least 5 minutes before the expiration, new credentials (with a more recent expiry) will be returned, which allows your application to automate the process of refreshing credentials.

## Resources
1. https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html