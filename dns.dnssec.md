# DNSSEC
- [NSEC3PARAM](#nsec3param)  
- [DNSKEY](#dnskey)  

### NSEC3PARAM
Reference: [RFC5155](https://tools.ietf.org/html/rfc5155)  

Format: `NSEC3PARAM algorithm flags iterations salt`  

DNS record example: `example.com. IN NSEC3PARAM 1 0 350 a1b2c3d4e5f6`  

* **algorithm** (_0-255_):  

  | value  | algorithm |
  | -------|-----------| 
  | 0      | reserved  | 
  | 1      | sha1      |
  | 2-255  | available |

* **flags** (_0-255_):  

  | value                 | flag    |
  | ----------------------|---------| 
  | 0                     | none    | 
  | least significant bit | Opt-Out |

* **iterations** (_1-2500_)  

  | key-size  | max iterations |
  | ----------|----------------| 
  | 1024      | 150            | 
  | 2048      | 500            |
  | 4096      | 2500           |

* **salt**  
  * use `-` as empty salt  
  * use hexadecimal digits  
  * max salt length is 255
  

### DNSKEY

Reference: [RFC4034](https://tools.ietf.org/rfc/rfc4034)  

Format: `DNSKEY flags protocol algorithm public_key`  

DNS record example: `example.com. IN DNSKEY 256 3 8 AwEAAalqhw...`  

* **flags** (_0-65535_):  

  | value  | flag                                                 |
  | -------|------------------------------------------------------| 
  | 256    | key used to sign zone                                |
  | 257    | key used to sign other keys (e.g., zone signing key) |

* **protocol** (_0-255_):  

  | value  | protocol |
  | -------|----------| 
  | 3      | default  |

* **algorithm** (_0-255_):  

  | value  | algorithm                      |
  | -------|--------------------------------|
  | 0      | reserved                       |
  | 1      | RSA/MD5 (deprecated, see 5)    |
  | 2      | Diffie-Hellman                 |
  | 3      | DSA/SHA1                       |
  | 4      | reserved                       |
  | 5      | RSA/SHA-1                      |
  | 6      | DSA-NSEC3-SHA1                 |
  | 7      | RSASHA1-NSEC3-SHA1             |
  | 8      | RSA/SHA-256                    |
  | 9      | reserved                       |
  | 10     | RSA/SHA-512                    |
  | 12     | GOST R 34.10-2001              |
  | 13     | ECDSA Curve P-256 with SHA-256 |
  | 14     | ECDSA Curve P-384 with SHA-384 |

  More: [DNSSEC algorithm numbers](http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml)

* **public_key**  
  * base64 encoded public key

