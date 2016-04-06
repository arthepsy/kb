# passwords
### generate random password
* alpha-numeric  
  ```
  cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 16 | head -n 5
  ```
  
* alpha-numeric and special characters  
  ```
  cat /dev/urandom | tr -dc 'a-zA-Z0-9!\@#\$%^&*-+_=' | fold -w 16 | head -n 5
  ```
