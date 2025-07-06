``` java
public static void main(String[] args) {
    var app = new SpringApplication(DeviceApplication.class);
    var ctx = app.run(args);
    printActiveProperties(ctx.getEnvironment());
}

private static void printActiveProperties(ConfigurableEnvironment env) {
    System.out.println("************************* ACTIVE APP PROPERTIES ******************************");
    env.getPropertySources().stream().filter(ps -> ps instanceof OriginTrackedMapPropertySource)
        .map(ps -> ((OriginTrackedMapPropertySource) ps).getSource().entrySet())
        .flatMap(it -> it.stream())
        .forEach(property -> System.out.println(property.getKey() + "=" + property.getValue()));
}
```
