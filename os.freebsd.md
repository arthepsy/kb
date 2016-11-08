# FreeBSD
- [hardening](#hardening) - hardening the system  
  - [sysctl](#sysctl), [tmp](#tmp), [ntpd](#ntpd), [ssl](#ssl), [syslogd](#syslogd), [sendmail](#sendmail), [sshd](#sshd)  
- [maintenance](#maintenance) - system and package maintenance  
  - [system update](#system-update), [packages update](#packages-update)  

## hardening
_NOTE: This section describes system hardening for FreeBSD 11.x._  

#### sysctl
* Edit `/etc/sysctl.conf` to harden system settings:  

  ```
  security.bsd.see_other_uids=0
  security.bsd.see_other_gids=0
  security.bsd.unprivileged_read_msgbuf=0
  security.bsd.unprivileged_proc_debug=0
  security.bsd.stack_guard_page=1
  ```
  
  These settings correspond to:
  * hide processes running as other users
  * hide processes running as other groups
  * disable reading kernel message buffer for unprivileged users
  * disable process debugging facilities for unprivileged users
  * insert stack guard page ahead of the growable segments

#### tmp
* Clean the `/tmp` filesystem on system startup:  

  ```
  sudo sysrc clear_tmp_enable="YES"
  ```

#### ntpd
Replace system `ntpd` with `OpenNTPD`.  

* Stop and disable system `ntpd`:  
  ```
  sudo service ntpd stop
  sudo sysrc ntpd_enable=NO
  sudo sysrc -X ntpd_sync_on_start
  ```
  
* Install `OpenNTPD`:
  ```
  cd /usr/ports/net/openntpd
  sudo make install clean
  ```
  
* Edit `/usr/local/etc/ntpd.conf` and configure `OpenNTPD` (_enabled settings shown_):
  ```
  servers pool.ntp.org
  constraints from "https://www.google.com/"
  ```

* Enable and start `OpenNTPD`:
  ```
  sudo sysrc openntpd_enable=YES
  sudo service openntpd start
  ```

#### ssl
Use `LibreSSL` instead of `OpenSSL`.

* Edit `/etc/make.conf` and configure ports to use `LibreSSL`:
  ```
  DEFAULT_VERSIONS+=  ssl=libressl
  ```
  
  _Note: Ports depending on OpenSSL will have to be recompiled._

#### syslogd
* Change settings (_choose either variant_):

  a) operate in secure mode and bind socket to loopback address:  
    ```
    sudo sysrc syslogd_flags="-s -b 127.0.0.1"
    ```
  b) operate in secure mode and don't open any network sockets:
    ```
    sudo sysrc syslogd_flags="-ss"
    ```

* Restart `syslogd`:  
  ```
  sudo service restart syslogd
  ```

#### sendmail
* Stop and disable `sendmail`:  

  ```
  sudo service sendmail stop
  sudo sysrc sendmail_enable="NO"
  ```

#### sshd
Replace system `OpenSSH` with ports `OpenSSH`.  

* Stop and disable system `sshd`:  
  ```
  sudo service sshd stop
  sudo sysrc sshd_enable=NO
  ```

* Install `OpenSSH` from ports:  
  ```
  cd /usr/ports/security/openssh-portable
  sudo make install clean
  ```
  
* Edit `/usr/local/etc/ssh/sshd_config` and configure `OpenSSH`:  
  ```
  Port 22
  Protocol 2
  
  HostKeyAlgorithms ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,ssh-rsa,ssh-rsa-cert-v01@openssh.com
  KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group18-sha512,diffie-hellman-group16-sha512,diffie-hellman-group14-sha256
  Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
  MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com
  
  PermitRootLogin no
  PasswordAuthentication no
  ChallengeResponseAuthentication no
  X11Forwarding no
  VersionAddendum none
  ```

  _NOTE: This example configuration is adjusted for strict and secure setup of OpenSSH 7.3. Test before making permanent._

* Enable and start `OpenSSH`:  
  ```
  sudo sysrc openssh_enable=YES
  sudo service openssh start
  ```


## maintenance
#### system update
_NOTE: This section describes binary system updates via `freebsd-update` and custom kernel compilation from sources._

* Ensure that `src` should not be updated, i.e., remove `src` from `Components` in default `/etc/freebsd-update.conf`:
  ```
  Components world kernel
  ```

* Fetch system updates:
  ```
  sudo freebsd-update fetch
  ```

  If output contains message about updating kernel, e.g. ...
  ```
  The following files will be updated as part of updating to ...
  (..)
  /boot/kernel/kernel
  ```
  ... and custom kernel is being used, then it needs to be updated aswell.
  
* (_if required_) Update custom kernel:  
  
  * Verify configuration:  
  
    This section relies that custom kernel is configured in `/etc/make.conf` as...  
    ```
    KERNCONF=MY-KERNEL
    INSTKERNNAME=my-kernel
    ```
    
    ... and respective `/boot/loader.conf.local` as...
    
    ```
    kernel="my-kernel"
    kernels="my-kernel kernel"
    ```
    ... and package `net/svnup` is installed with respective `/usr/local/etc/svnup.conf` as (for 11.0-RELEASE):
    ```
    [release]
    branch=base/releng/11.0
    target=/usr/src
    ```
  
  * Update sources:  
    ```
    sudo svnup release
    ```
  
  * Compile and install kernel:  
    ```
    cd /usr/src
    sudo make -j `sysctl -n hw.ncpu` kernel-toolchain
    sudo make -j `sysctl -n hw.ncpu` buildkernel
    sudo make installkernel
    ```

* Install updates
  ```
  sudo freebsd-update install
  ```

* Reboot
  ```
  sudo reboot
  ```

#### packages update
_NOTE: This section describes package updates by compilation them from sources (ports tree) by `portmaster`._

* Update ports tree:
  ```
  sudo portsnap fetch update
  ```

* Check out-of-date packages:
  ```
  pkg version -vIL=
  ```

* Read important changes and follow instructions:
  ```
  less /usr/ports/UPDATING
  ```

* Update rest of packages (requires `ports-mgmt/portmaster` package):
  ```
  portmaster -a
  ```
