# DigitalOcean - FreeBSD
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


