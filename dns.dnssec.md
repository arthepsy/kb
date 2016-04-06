# DNSSEC
### NSEC3PARAM
Reference: [RFC5155](https://tools.ietf.org/html/rfc5155)  

Format: `NSEC3PARAM algorithm flags iterations salt`  

DNS record example: `example.com. IN 3600 NSEC3PARAM 1 0 350 a1b2c3d4e5f6`  

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
  
