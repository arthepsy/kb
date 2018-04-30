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

## chain
* create certificate chain of trust 
  ```
  cat domain.crt intermediate.crt root.crt > domain.pem
  ```

* create full chain of trust 
  ```
  cat domain.key domain.crt intermediate.crt root.crt > domain.pem
  ```

## passphrase

* remove passphrase
  ```
  openssl rsa -in test.key -out nopass_test.key
  ```
