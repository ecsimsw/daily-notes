### Signed url 생성 - AWS  SDK Java V2
- Java 에서 Signed URL 을 만들기 위해선 aws-sdk-java-v2 가 필요하다. 
- 이때 공식 문서에서 소개하는 "'software.amazon.awssdk:aws-sdk-java:2.X.X'"가 아니라 아래처럼 필요한 모듈만 가져오길 추천

``` java 
dependencies {
    implementation 'software.amazon.awssdk:s3:2.25.27'
    implementation 'software.amazon.awssdk:cloudfront:2.25.27'
}
```
- CloudFront에 Public key를 등록하고, public 키와 함께 생성된 private key를 로컬 스토리지에 저장해둔다.
``` java
var cloudFrontKeyPairId = $KEY_PAIR_ID;
var cloudFrontDomainName = "${CLOUD_FRONT_DOMAIN_NAME}";
var privateKeyPath = "${PRIVATE_KEY_PATH}";
var resourcePath = "${RESOURCE_PATH}";

var sign = CannedSignerRequest.builder()
    .privateKey(Path.of(privateKeyPath))
    .resourceUrl(new URL("https", cloudFrontDomainName, resourcePath).toString())
    .keyPairId(cloudFrontKeyPairId)
    .expirationDate(Instant.now().plus(7, ChronoUnit.DAYS))
    .build();
var signedUrl = cloudFrontUtilities.getSignedUrlWithCannedPolicy(sign);
return signedUrl.url();
```
- CustomSignerRequest 를 사용하면 접근 가능한 자원에 와일드카드를 사용하거나, IP 범위, 유효 시작 시간을 지정할 수 있다.

``` java
var sign = CustomSignerRequest.builder()
    .privateKey(Path.of(PRIVATE_KEY_PATH))
    .ipRange("0.0.0.0")
    .resourceUrl(new URL(CDN_PROTOCOL, CLOUD_FRONT_DOMAIN_NAME, "/users/1/*").toString())
    .keyPairId(publicKeyId)
    .activeDate(Instant.now())
    .expirationDate(Instant.now().plus(EXPIRATION_AFTER_DAYS, ChronoUnit.DAYS))
    .build();
var signedUrl = cloudFrontUtilities.getSignedUrlWithCustomPolicy(sign);
return signedUrl.url();
```
