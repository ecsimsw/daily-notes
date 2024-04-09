## E20240410 CloudFront signed url.md


### Create RSA PEM key
```
openssl genrsa -out private_key.pem 2048

openssl rsa -pubout -in private_key.pem -out public_key.pem
```

### Create public key
![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/5cd04abf-0f07-4e0c-9c5d-5b153492517a)

### Create key group
![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/edc3f3fa-6187-4976-8a4f-c7498745e80f)

![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/77950b69-f691-4e52-a96f-290629bf402f)

### Restrict viewer access

![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/ebc06893-6096-42ae-85fe-f8aae76ab2b7)

![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/7c0666a1-c64f-4119-b356-1d1426e30df4)

### Test request

![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/7d0f0fd5-2e65-45a6-b137-7c3ed1786386)


### Java 

``` java
implementation 'software.amazon.awssdk:aws-sdk-java:2.25.27'
```

``` java 
public static CannedSignerRequest createRequestForCannedPolicy(
    String distributionDomainName,
    String fileNameToUpload,
    String publicKeyId
) throws Exception {
    String protocol = "https";
    String resourcePath = "/" + fileNameToUpload;
    String cloudFrontUrl = new URL(protocol, distributionDomainName, resourcePath).toString();
    Instant expirationDate = Instant.now().plus(7, ChronoUnit.DAYS);
    return CannedSignerRequest.builder()
        .privateKey(Path.of(${PRIVATE_PEM_KEY_PATH})) 
        .resourceUrl(cloudFrontUrl)
        .keyPairId(publicKeyId)
        .expirationDate(expirationDate)
        .build();
}
```

``` java
CannedSignerRequest cannedSignerRequest = createRequestForCannedPolicy(X,X,X);
CloudFrontUtilities cloudFrontUtilities = CloudFrontUtilities.create();
SignedUrl signedUrl = cloudFrontUtilities.getSignedUrlWithCannedPolicy(cannedSignerRequest);
```

/// custom 으로 하면 ip range 도 제한 가능

``` java
return CustomSignerRequest.builder()
    .resourceUrl(cloudFrontUrl)
    .privateKey(Path.of("/Users/kimjinhwan/temp/private_key.pem"))
    .keyPairId(publicKeyId)
    .expirationDate(expirationDate)
//            .activeDate(activeDate) // Optional.
     .ipRange("121.185.9.18/32") // Optional.
    .build();
```
