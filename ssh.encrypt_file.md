# SSH - encrypt/decrypt file
## with RSA keys
#### encryption

##### 1. get public key
For encryption, public key should be in PEM format. Either convert existing public key ...
```
ssh-keygen -f ~/.ssh/id_rsa.pub -e -m PKCS8 > ~/.ssh/id_rsa.pub.pem
```
... or extract from private key (_will ask for passphrase_):
```
openssl rsa -in ~/.ssh/id_rsa -pubout > ~/.ssh/id_rsa.pub.pem
```

##### 2. generate key
Check the maxium size for symmetric key (_in bytes_):
```
ssh-keygen -l -f .ssh/id_rsa | awk '{ printf("%d", ($1 - 96) / 8) }'
```
Generate random key (_assuming 2048-bit RSA key_):
```
openssl rand 244 > key.bin
```

##### 3. encrypt key
Encrypt symmetric key:
```
openssl rsautl -encrypt -pubin -inkey ~/.ssh/id_rsa.pub.pem -in key.bin -out key.bin.enc
```
Note: now, public key in PEM format can be removed.

##### 4. encrypt file
Encrypt file:
```
openssl enc -aes-256-cbc -salt -pass file:key.bin -in sample.txt -out sample.txt.enc
```
Note: now, key file and original file can be removed.

#### decryption
##### 1. decrypt key
Decrypt symmetric key (_will ask for passphrase_):
```
openssl rsautl -decrypt -inkey ~/.ssh/id_rsa -in key.bin.enc -out key.bin
```

##### 2. decrypt file
Decrypt file:
```
openssl enc -d -aes-256-cbc -pass file:key.bin -in sample.txt.enc -out sample.txt
```
