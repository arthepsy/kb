# SSL
## view information
* private key  

  ```
  openssl rsa -in test.key -text -noout
  ```

* certificate

  ```
  openssl x509 -in test.crt -text -noout
  ```

* signing request
  ```
  openssl req -in test.csr -text -noout
  ```
