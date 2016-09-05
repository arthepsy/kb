# Linux package management

### yum
* List (installed) packages:  
  ```
  yum list
  yum list installed
  ```

* List package contents (_requires `yum-utils` package_):
  ```
  repoquery -l package
  repoquery -l --installed package
  ```

* Information about package:
  ```
  yum info package
  ```

* Modify package:
  ```
  yum install|reinstall|update|remove package
  ```


### rpm
* Verify package (_owner, group, mode, size, checksum, etc._):  
  ```
  rpm -V package
  ```
