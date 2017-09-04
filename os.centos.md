# CentOS

### OpenNTPD
This section describes how to replace `chrony` with `openntpd`.

* Remove `chrony`:
  ```
  sudo systemctl stop chronyd
  sudo yum remove chrony
  ```

* Download and extract OpenNTPD
   ```
   curl -O https://ftp.eu.openbsd.org/pub/OpenBSD/OpenNTPD/openntpd-6.2p1.tar.gz
   tar -xvzf openntpd-6.2p1.tar.gz
   ```

* Install compiler and OpenNTPD
  ```
  sudo yum install gcc
  cd openntpd-6.2p1
  ./configure --with-privsep-path=/var/empty/openntpd
  make
  sudo make install
  ```

* Create necessary user/group and configure file-system
  ```
  sudo groupadd _ntp
  sudo useradd -g _ntp -s /sbin/nologin -d /var/empty/openntpd -c 'OpenNTP daemon' _ntp
  sudo mkdir -p /var/empty/openntpd
  sudo chown 0 /var/empty/openntpd
  sudo chgrp 0 /var/empty/openntpd
  sudo chmod 0755 /var/empty/openntpd
  ```

* Create `systemd` service, e.g., `sudoedit /usr/lib/systemd/system/openntpd.service`:
  ```
  [Unit]
  Description=OpenNTP Daemon
  After=network.target
  Conflicts=systemd-timesyncd.service
  
  [Service]
  Type=forking
  ExecStart=/usr/local/sbin/ntpd -s
  
  [Install]
  WantedBy=multi-user.target
  ```

* Configure OpenNTPD:
  ```
  sudoedit /usr/local/etc/ntpd.conf
  ```

* Enable service and start it:
  ```
  systemctl enable openntpd.service
  systemctl start openntpd.service
  ```

### iptables
* Stop and disable `firewalld`:  
  ```
  sudo systemctl stop firewalld
  sudo systemctl disable firewalld
  ```

* Install and enable `iptables`:
  ```
  sudo yum install iptables
  sudo systemctl enable iptables
  ```

* Edit `/etc/sysconfig/iptables`, verify and reload rules:
  ```
  iptables-restore -t /etc/sysconfig/iptables
  systemctl restart iptables
  ```
