## Why NoSql is better at scaling out

So I've been trying to figure out the real bottom-line when it comes to NoSQL vs RDBMS myself, and always end up with a response that doesn't quite cut it. In my search there are really 2 primary differences between NoSQL and SQL, with only 1 being a true advantage.

1. <b>ACID vs BASE</b> - NoSQL typically leaves out some of the ACID features of SQL, sort of 'cheating' it's way to higher performance by leaving this layer of abstraction to the programmer. This has already been covered by previous posters.
2. <b>Horizontal Scaling</b> - The real advantage of NoSQL is horizontal scaling, aka sharding. Considering NoSQL 'documents' are sort of a 'self-contained' object, objects can be on different servers without worrying about joining rows from multiple servers, as is the case with the relational model.

Let's say we want to return an object like this:

```
post {
    id: 1
    title: 'My post'
    content: 'The content'
    comments: {
      comment: {
        id: 1
      }
      comment: {
        id: 2
      }
      ...

    views: {
      view: {
        user: 1
      }
      view: {
        user: 2
      }
      ...
    }
}
```

In NoSQL, that object would basically be stored as is, and therefore can reside on a single server as a sort of self-contained object, without any need to join with data from other tables that could reside on other DB servers.

However, with Relational DBs, the post would need to join with comments from the comments table, as well as views from the views table. This wouldn't be a problem in SQL ~UNTIL~ the DB is broken into shards, in which case 'comment 1' could be on one DB server, while 'comment 2' yet on another DB server. This makes it much more difficult to create the very same object in a RDBMS that has been scaled horizontally than in a NoSQL DB.


## 내가 이해한 예시

사용자가 치킨을 먹은 사실을 저장해야 한다. 사용자에 대한 정보, 치킨에 대한 정보, 사용자가 먹은 음식에 대한 정보를 기록해야 할 것이다.     

#### RDBMS

테이블이 서로 다른 서버에 있을 수 있다. 사용자와 음식 테이블이 다른 DB 서버에 있다면 결국 사용자 정보 / 먹은 음식 정보를 읽기 위해선 두 DB 서버를 모두 엑세스 해야한다.   

한 테이블 안에서 수평 분리하기도 불리하다. A 서버에 사용자 정보와 치킨 정보가 함께 있었는데, 사용자 테이블을 수평 분리하게 되어 사용자 정보가 A, B로 나뉘게 되어도 B 서버의 사용자 정보/ 먹은 음식 정보를 읽어야 하는 경우 결국 A, B 서버를 모두 엑세스해야 한다.

서버 분리에 비효율적이다.

#### NoSQL

NoSQL에선 그럴 일이 없다. 어차피 사용자 안에서 먹은 치킨 정보를 저장하기 때문이다. 사용자 도큐먼트를 읽고, 그 내부 필드인 먹은 음식에 대한 정보를 읽으면 되는 것이다.     

그렇기에 수평 분리에도 문제가 없다. 사용자 정보가 A, B 서버로 나뉘어도 사용자 도쿠먼트 안에 먹은 음식 정보가 있기 때문에, 엑세스 범위에 따라 해당 서버 한 서버로 사용자, 음식 정보를 엑세스 할 수 있다는게 보장된다.    

단, 사용자 A가 먹은 치킨과 사용자 B가 먹은 치킨이 동일해도 중복해서 정보를 저장해야 한다. RDBMS에선 치킨 정보를 한번 저장하고 JOIN하면 될 문제였다.   
