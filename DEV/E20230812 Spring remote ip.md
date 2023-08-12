## Spring remote ip

``` java
private static String remoteIp(HttpServletRequest request) {
    final String ip = request.getHeader("X-FORWARDED-FOR");
    if (ip == null) {
        return request.getRemoteAddr();
    }
    return ip;
}
```

웹서버나 proxy의 IP 가 나오는 것을 방지하기 위해  X-Forwarded-For 값을 먼저 확인
