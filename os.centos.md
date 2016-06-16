# CentOS

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
