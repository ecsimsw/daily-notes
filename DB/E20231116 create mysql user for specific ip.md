## Create mysql user for specific ip range

IP를 구체화한 User 생성을 습관화하자.

<img width="1431" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/d36e3c40-d611-4599-892e-8dfc38dbfd18">

```
create user {username}@{ip} identified by '{password}';
grant all privileges on *.* to {username}@{ip};
```
