## E20240624 Mysql backup.md

#### dump
```
mysqldump -u $USER_NAME -p $DATABASE > $BACKUP_FILE_NAME.sql
```

#### load
```
mysql -u $USER_NAME -p $DATABASE < $BACKUP_FILE_NAME.sql
```

#### docker copy
```
docker cp $CONTAINER:$FILE_PATH $FILE_PATH_TO_SAVE
```

#### scp
```
scp -P $SSH_PORT $USER@$HOST:$FILE_PATH $FILE_PATH_TO_SAVE
```
