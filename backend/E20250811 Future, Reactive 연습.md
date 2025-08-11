``` java
public void asIs() {
    var deviceIds = new ArrayList<String>();
    var windowSize = 5;
    var hasNextCount = windowSize;
    var page = 0;
    while(hasNextCount == windowSize) {
        var futures = new ArrayList<CompletableFuture<PlatformUserDevicesResponse>>();
        for (var i = 1; i <= windowSize; i++) {
            var pageNumber = windowSize * page + i;
            futures.add(getDeviceListWithPage(pageNumber));
        }
        var deviceResponses = futures.stream()
            .map(CompletableFuture::join)
            .toList();
        hasNextCount = (int) deviceResponses.stream()
            .filter(res -> res.hasNext)
            .count();
        deviceResponses.stream()
            .filter(res -> !res.isEmpty())
            .map(res -> res.deviceId)
            .forEach(deviceIds::add);
        page++;
    }
    System.out.println(deviceIds);
}

private final AtomicInteger index = new AtomicInteger(1);

@SneakyThrows
private CompletableFuture<PlatformUserDevicesResponse> getDeviceListWithPage(int page) {
    log.info("Api call page : {} {}", page, System.currentTimeMillis());
    var i = index.getAndIncrement();
    if(i > 14) {
        return CompletableFuture.supplyAsync(() -> {
            sleep(1000);
            return new PlatformUserDevicesResponse(false, null);
        });
    }
    return CompletableFuture.supplyAsync(() -> {
        sleep(1000);
        return new PlatformUserDevicesResponse(true, i+ "");
    });
}
```
