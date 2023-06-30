## Semantic Versioning regex

ref, https://semver.org


### My regex sample

```
^(v?)([0-9]+)\.([0-9]+)\.([0-9]+)(?:-([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?(?:\+[0-9A-Za-z-]+)?$
```

ex) v1.0.0, v1.0.0-release, 1.0.0, 1.0.0-beta
