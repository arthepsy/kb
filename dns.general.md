# DNS

## unbound
[Unbound](https://www.unbound.net/) is a validating, recursive, and caching DNS resolver.

#### OpenBSD
Available in base system since OpenBSD 5.6.  

* Download `root.hints`:  
  ```
  doas curl -o /var/unbound/db/root.hints https://www.internic.net/domain/named.cache
  ```

* Download `root.key`:  
  ```
  doas unbound-anchor -a /var/unbound/db/root.key
  doas chown _unbound:_unbound /var/unbound/db/root.key
  ```
  
* Enable service:  
  ```
  doas rcctl enable unbound
  ```

* Add root hints and trust anchor in `/var/unbound/etc/unbound.conf`:  
  ```
  server:
        ...
        auto-trust-anchor-file: "/var/unbound/db/root.key"
        root-hints: "/var/unbound/db/root.hints"
  
  ```
  
* Start service:   
  ```
  doas rcctl start unbound
  ```

* Edit `/etc/resolv.conf`
  ```
  nameserver 127.0.0.1
  ```

#### FreeBSD
Available in base system since FreeBSD 10.0.  

* Download `root.hints`:  
  ```
  sudo curl -o /var/unbound/root.hints https://www.internic.net/domain/named.cache
  ```
  
* Enable service:  
  ```
  sudo sysrc local_unbound_enable=YES
  ```

* Start service:   
  ```
  sudo service local_unbound start
  ```

* Add root hints, trust anchor and remove forwarders in `/var/unbound/unbound.conf`:  
  ```
  server:
        ...
        auto-trust-anchor-file: /var/unbound/root.key
        root-hints: /var/unbound/root.hints
  
  #include: /var/unbound/forward.conf
  ```

* Reload service:   
  ```
  sudo service local_unbound reload
  ```

#### general

* Harden `unbound` settings:  
  ```
  server:
        ...
        hide-identity: yes
        hide-version: yes         
  ```

* Test DNSSEC verification:  
  ```
  drill -S FreeBSD.org
  dig +dnssec FreeBSD.org
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
