## submodule fetch erorr

Submodule path (commit version)을 잃었을 때

#### Error 
```
Fetched in submodule path 'src/framework', but it did not contain cc8c38e9d853491c672452d8dbced4666fc73ec8. Direct fetching of that commit failed.
```

#### Sol 
```
git submodule update --force --recursive --init --remote
```
