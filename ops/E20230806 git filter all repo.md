## Change git author all commits on repo

```
git filter-branch -f --env-filter "
    GIT_AUTHOR_NAME='ecsimsw'
    GIT_AUTHOR_EMAIL='ecsimsw@gmail.com'
    GIT_COMMITTER_NAME='ecsimsw'
    GIT_COMMITTER_EMAIL='ecsimsw@gmail.com'
  " HEAD
```
