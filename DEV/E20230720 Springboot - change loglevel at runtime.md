## Springboot - change loglevel at runtime

ref, `https://rogerwelin.github.io/java/springboot/logback/2017/03/11/change-loglevel-at-runtime-springboot.html`

``` java
@RequestMapping(value = "/loglevel/{loglevel}", method = RequestMethod.POST)
public String loglevel(@PathVariable("loglevel") String logLevel, @RequestParam(value="package") String packageName) throws Exception {
    log.info("Log level: " + logLevel);
    log.info("Package name: " + packageName);
    String retVal = setLogLevel(logLevel, packageName);
    return retVal;
}

public String setLogLevel(String logLevel, String packageName) {
    String retVal;
    LoggerContext loggerContext = (LoggerContext)LoggerFactory.getILoggerFactory();
    if (logLevel.equalsIgnoreCase("DEBUG")) {
        loggerContext.getLogger(packageName).setLevel(Level.DEBUG);
        retVal = "ok";
    } else if (logLevel.equalsIgnoreCase("INFO")) {
        loggerContext.getLogger(packageName).setLevel(Level.INFO);
        retVal = "ok";
    } else if (logLevel.equalsIgnoreCase("TRACE")) {
        loggerContext.getLogger(packageName).setLevel(Level.TRACE);
        retVal = "ok";
    } else {
        log.error("Not a known loglevel: " + logLevel);
        retVal = "Error, not a known loglevel: " + logLevel;
    }
    return retVal;
}
```
