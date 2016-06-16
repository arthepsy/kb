# DNS

## unbound
[Unbound](https://www.unbound.net/) is a validating, recursive, and caching DNS resolver.

#### OpenBSD
Available in base system since OpenBSD 5.6.  

* Enable service:  
  ```
  rcctl enable unbound
  ```

* Start service:   
  ```
  rcctl start unbound
  ```

* Edit `/etc/resolv.conf`
  ```
  nameserver 127.0.0.1
  ```

## DHCP and resolv.conf
#### OpenBSD
To use custom DNS settings, 

* Edit `/etc/dhclient.conf`:  
  ```
  supersede domain-name-servers 127.0.0.1, 8.8.8.8;
  supersede domain-name "example.com";
  ```

* Edit `/etc/resolv.conf.tail`:  
  ```
  lookup file bind
  ```

* Restart `dhclient`:
  ```
  kill -HUP `pgrep dhclient`
  ```
