# Linux package management

### yum
* List installed packages:  
  ```
  yum list installed
  ```

* List package contents (_requires `yum-utils` package_):
  ```
  repoquery -l package
  repoquery -l --installed package
  ```
