# passwords
### generate random password
* alpha-numeric  
  ```
  cat /dev/urandom | env LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 16 | head -n 5
  ```

* alpha-numeric and special characters  
  ```
  cat /dev/urandom | env LC_ALL=C tr -dc 'a-zA-Z0-9!\@#\$%^&*-+_=' | fold -w 16 | head -n 5
  ```

* hexadecimal  
  ```
  cat /dev/urandom | env LC_ALL=C tr -dc '0-9a-f' | fold -w 16 | head -n 5
  ```

