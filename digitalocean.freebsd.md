# DigitalOcean - FreeBSD
- [Initial setup](#initial-setup) - various steps after booting a fresh FreeBSD instance
- [Custom kernel](#custom-kernel) - peaceful living with `freebsd-update` and custom kernel  

## initial setup
##### 1. accounts
Add new user and copy ssh keys:
```
set _user=newuser
sudo adduser
sudo mkdir /home/${_user}/.ssh
sudo cp /home/freebsd/.ssh/authorized_keys /home/${_user}/.ssh/
sudo chown -R ${_user}:${_user} /home/${_user}/.ssh
unset _user
```
Add sudo access, i.e., run `visudo` and add line:
```
newuser ALL=(ALL) ALL
```
Disconnect and reconnect with new user.  

Remove `freebsd` user:
```
pw groupmod wheel -d freebsd
rmuser freebsd
```
Change `root` password...
```
head -c 1024 /dev/urandom | sha256
passwd root
```
...or lock `root` account:
```
pw lock root
```
##### 2. sshd
Edit `/etc/ssh/sshd_config` configuration (e.g., `sudo vi /etc/ssh/sshd_config`):  

change these lines...
```
#VersionAddendum FreeBSD-20140420
PermitRootLogin without-password
#ChallengeResponseAuthentication yes
```
...to these lines
```
VersionAddendum none
PermitRootLogin no
ChallengeResponseAuthentication no
```
and remove these:
```
Match User root
        ForceCommand echo "Please use the freebsd@ user to access this droplet."
```
Restart sshd:
```
service sshd restart
```

##### 3. time
Set time zone:
```
sudo tzsetup
```
Enable `ntpd`:
```
sudo sh -c 'echo ntpd_enable=\"YES\" >> /etc/rc.conf'
sudo sh -c 'echo ntpd_sync_on_start=\"YES\" >> /etc/rc.conf'
sudo service ntpd start
```

##### 4. swap
Add 2GB swap as file:
```
sudo truncate -s 2G /swap
sudo chmod 0600 /swap
sudo sh -c 'echo "md99 none swap sw,file=/swap,late 0 0" >> /etc/fstab'
sudo swapon -aqL
```

##### 5. droplet.conf
Merge configuration:
```
cat /etc/rc.digitalocean.d/droplet.conf > tmp.conf
cat /etc/rc.conf >> tmp.conf
sudo mv tmp.conf /etc/rc.conf
```
Remove remains of DigitalOcean's initial configuration:
```
sudo rm -rf /etc/rc.digitalocean.d
sudo rm /etc/rc.d/digitalocean
```
Edit `/etc/rc.subr` (e.g., `sudo vi /etc/rc.subr`) and remove these lines:
```
        if [ -L /etc/rc.digitalocean.d/droplet.conf -a -f /etc/rc.digitalocean.d/droplet.conf ]
        then
                . /etc/rc.digitalocean.d/droplet.conf
        fi
```

##### 6. unused packages
Stop and remove `avahi`, used for DigitalOcean's initial configuration:
```
pkill avahi-autoipd
pkg delete avahi-app
rmuser avahi
```
Remove other unused packages:
```
pkg autoremove
```

##### 7. update packages
Update ports tree:
```
sudo portsnap fetch update
```
Upgrade `pkg` utility and install `portmaster`:
```
cd /usr/ports/ports-mgmt/pkg
sudo make deinstall reinstall clean
cd /usr/ports/ports-mgmt/portmaster
sudo make install clean
rehash
```
Check and upgrade packages:
```
pkg version -vIL=
sudo portmaster -a
```

##### 8. update system
Disable `/usr/src` updating by editing `/etc/freebsd-update.conf`:
```
-Components src world kernel
+Components world kernel
```
Fetch and install updates:
```
sudo freebsd-update fetch
sudo freebsd-update install
```
Reboot:
```
sudo reboot
```

##### 9. sources
Install `svnup` utility:
```
cd /usr/ports/net/svnup
sudo make install clean
```
Edit `/usr/local/etc/svnup.conf` (e.g., `sudo vi /usr/local/etc/svnup.conf`)  

set appropriate `host`...
```
host=svn0.eu.freebsd.org
```
...and appropriate `branch` under `release` category:
```
branch=base/releng/10.2
```
Update sources:
```
sudo svnup release
```

## custom kernel
Custom kernel and `freebsd-update` utility doesn't mix too well, therefore one must keep custom kernel in different location. If default kernel is updated by this utility, one should rebuild the custom kernel.

##### sources
Update sources:
```
sudo svnup release
```

##### initial configuration
Copy kernel configuration, based on GENERIC:
```
cd /usr/src/sys/`uname -m`/conf
sudo cp GENERIC /root/MYKERNEL
sudo ln -s /root/MYKERNEL .
```
Edit kernel configuration:
```
sudo vi MYKERNEL
```
Edit `/etc/make.conf` (e.g., `sudo vi /etc/make.conf`) and specify kernel file and location for build tools:  
(`KERNCONF` - kernel configuration file name, `INSTKERNNAME` - kernel install directory under `/boot`)
```
KERNCONF=MYKERNEL
INSTKERNNAME=my-kernel
```
Edit `/boot/loader.conf.local` (e.g., `sudo vi /boot/loader.conf.local`) and specify to boot custom kernel:
```
kernel="my-kernel"
```

##### building and installing
Go to source directory:
```
cd /usr/src
```
Build tools required for building kernel:
```
sudo make -j `sysctl -n hw.ncpu` kernel-toolchain
```
Build kernel:
```
sudo make -j `sysctl -n hw.ncpu` buildkernel
```
Install kernel:
```
sudo make -j `sysctl -n hw.ncpu` installkernel
```
Reboot:
```
sudo reboot
```
