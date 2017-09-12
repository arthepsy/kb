# Linux init systems

### systemd
* Show status:  
  ```
  systemctl status
  ```

* List running:
  ```
  systemctl
  systemctl list-units
  ```

* List available:
  ```
  systemctl list-unit-files
  ```

* Change state:
  ```
  systemctl status|enable|disable|start|stop|restart|stop unit
  ```

* Remove service:
  ```
  systemctl stop servicename
  systemctl disable servicename
  rm /usr/lib/systemd/system/servicename.service
  ```

* Display logs by service:
  ```
  journalctl -ru servicename.service
  ```
  
* Display logs by uid:
  ```
  journalctl -r _UID=`id -u user`
  ```
  
