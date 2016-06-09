# DNSSEC
- [NSEC3PARAM](#nsec3param)  
- [DNSKEY](#dnskey)  
- [DS](#ds)  

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

Reference: [RFC4034](https://tools.ietf.org/html/rfc4034)  

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


### DS

Reference: [RFC4034](https://tools.ietf.org/html/rfc4034), [RFC4509](https://tools.ietf.org/html/rfc4509)  

Format: `DS key_tag algorithm digest_type digest`  

DNS record example: `example.com. IN DS 29659 8 1 eeb10dfa246bb46fec9cab6f3641e71c45003ebd`  

* **key_tag** (_0-65535_):  

  * [Key Tag calculation](https://tools.ietf.org/html/rfc4034#appendix-B)

* **algorithm** (_0-255_):  

  Same as for DNSSEC record. More: [DNSSEC algorithm numbers](http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml)

* **digest_type** (_0-255_):  

  | value  | digest type |
  | -------|-------------| 
  | 0      | reserved    |
  | 1      | SHA-1       |
  | 2      | SHA-256     |

* **digest**  

  `digest = digest_algorithm(DNSKEY owner name | DNSKEY RDATA)`
  
  `DNSKEY owner name`
  * zone name (ending with `.`) encoded in wire format
  * wire format:
    1. split in parts between `.`
    2. encode as  `length | part`
    3. concatenate
  * _example_: 
    1. `example.com.` -> `example`, `com`,  
    2. `0x076578616d706c65`, `0x03636f6d`, `0x00`  
    3. 0x076578616d706c6503636f6d00  
  
  `DNSKEY RDATA = Flags | Protocol | Algorithm | Public Key`
  * converted to binary

### DNSKEY -> DS

Python 2/3 script to create DS record from DNSKEY record:
```
import base64, struct, hashlib

domain, flags, protocol, algorithm = b'example.com.', 256, 3, 8
pk = b'AwEAAalqhwPdx5XFOb1lZBKuZPXlO5+AMDDHpEn3cA1iy6NQN3JM4Gtw3TYwzasZWaldwu0AmocNfIDFjD87/nOb9hp30r6IaxTCps5dZjx9ubSfibDkKBG/QXVrNTuCSldwTV0ARz/pRpVsxHhVEpvUFRQz8unTeAqHD8CDWmgRO5r1O3vT'

o = b''.join([struct.pack('B%ds'%len(x), len(x), x) for x in domain.split(b'.')])
r = struct.pack('!HBB', flags, protocol, algorithm) + base64.b64decode(pk)

c = sum((lambda x: x if i % 2 else x << 8)(struct.unpack('B', r[i:i+1])[0]) for i in range(len(r)))
ktag = ((c & 0xffff) + (c >> 16)) & 0xffff

h1 = hashlib.sha1(o+r).hexdigest()
h2 = hashlib.sha256(o+r).hexdigest()

print('IN DS {} {} 1 {}'.format(ktag, algorithm, h1))
print('IN DS {} {} 2 {}'.format(ktag, algorithm, h2))
```

Example run:
```
IN DS 29659 8 1 eeb10dfa246bb46fec9cab6f3641e71c45003ebd
IN DS 29659 8 2 976eca0ec608d4935e4232e9f3a356a4bb19134599962c3c57219b147d1e33ec
```
