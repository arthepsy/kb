# FreeBSD install

- [on linux via iPXE](#via-ipxe) - install FreeBSD on existing Linux via iPXE  
- [on linux via local ISO](#via-local-iso) - install FreeBSD on existing Linux via local ISO

## on linux

### via iPXE

* download iPXE linux kernel
  ```
  cd /boot
  curl -O https://boot.netboot.xyz/ipxe/generic-ipxe.lkrn
  ```

* create [netboot.xyz](https://netboot.xyz) initrd file in `/boot/netboot.xyz-initrd`
  ```
  #!ipxe
  #/boot/netboot.xyz-initrd
  imgfree
  set net0/ip <instance public ip>
  set net0/netmask <instance public netmask>
  set net0/gateway <instance public gateway>
  set dns <instance dns address>
  ifopen net0
  chain --autofree https://boot.netboot.xyz
  ```

* add custom entry to grub in `/etc/grub.d/40_custom`
  ```
  menuentry 'netboot.xyz' {
    set root='hd0,msdos1'
    linux16 /boot/generic-ipxe.lkrn
    initrd16 /boot/netboot.xyz-initrd
  }
  ```

* regenerate grub config
  ```
  grub2-mkconfig -o /boot/grub2/grub.cfg
  ```

* reboot, select `netboot.xyz` item and install FreeBSD as usual

### via local ISO

* download [mfsbsd](http://mfsbsd.vx.sk) iso image
  ```
  cd /boot
  curl -O http://mfsbsd.vx.sk/files/iso/11/amd64/mfsbsd-se-11.1-RELEASE-amd64.iso
  ```

* add custom entry to grub in `/etc/grub.d/40_custom`
  ```
  menuentry "mfsbsd" {
    set isofile=/boot/mfsbsd-se-11.1-RELEASE-amd64.iso
    loopback loop (hd0,1)$isofile
    kfreebsd (loop)/boot/kernel/kernel.gz -v
    kfreebsd_module (loop)/boot/kernel/ahci.ko
    kfreebsd_module (loop)/mfsroot.gz type=mfs_root
    set kFreeBSD.vfs.root.mountfrom="ufs:/dev/md0"
  }
  ```

* set grub wait time longer in `/etc/default/grub`
  ```
  GRUB_TIMEOUT=10
  ```
  
* regenerate grub config
  ```
  grub2-mkconfig -o /boot/grub2/grub.cfg
  ```

* reboot, select `mfsbsd` item and boot into mfsbsd (_root password: mfsroot_)

* configure network (`ifconfig`, `route`, `/etc/resolv.conf`)
  ```
  ifconfig em0 192.168.0.1 netmask 255.255.255.0
  route add default 192.168.0.254
  echo "nameserver 8.8.8.8" > /etc/resolv.conf
  ```

* download `MANIFEST` file from mirror into `/usr/freebsd-dist/MANIFEST`
  ```
  mkdir -p /usr/freebsd-dist
  ftp -o /usr/freebsd-dist/MANIFEST http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/11.1-RELEASE/MANIFEST
  ```

* run installation as usual
  ```
  bsdinstall
  ```

