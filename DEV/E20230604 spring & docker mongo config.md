## spring & docker mongo config

### Docker mongo 

run mongo
```
docker pull mongo
docker run -d --name mongodb-container \
      -v ~/data:/data/db \
      -e MONGO_INITDB_ROOT_USERNAME=admin \
      -e MONGO_INITDB_ROOT_PASSWORD=password \
      -p 27017:27017 \
      mongo
```

execute mongosh
```
docker exec -it mongodb-container bash
mongosh -u admin -p password
``` 

sample usage
```
use mymaket                           // create db
db.createCollection("order-audit")    // create collection
```

### Add gradle dependency (build.gradle)
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
}
```

### Configure Spring.data.mongodb
```
spring.data.mongodb.uri=mongodb://admin:password@localhost:27017/${dbName}?authSource=admin
```

### Sample entity

``` java
@Document(collection = "user")
public class User {

    private Long userId;
    private String userName;

    public User() {
    }

    public User(Long userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }

    public Long getUserId() {
        return userId;
    }

    public String getUserName() {
        return userName;
    }
}
```

### Sample mongoRepository 
```
public interface UserRepository extends MongoRepository<User, Long> {
}
```
