# Linux package management

### yum
* List repositories:  
  ```
  yum repolist
  yum repolist enabled
  ```

* List (installed) packages:  
  ```
  yum list
  yum list installed
  ```

* List available packages in specific repository:  
  ```
  yum --disablerepo="*" --enablerepo="mysql57-community" list available
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
