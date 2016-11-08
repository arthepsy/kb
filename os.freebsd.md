# FreeBSD
- [maintenance](#maintenance) - update/upgrade system with `freebsd-update` and custom kernel  
  - [system update](#system-update), [packages update](#packages-update)  

## maintenance
#### system update

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
    ... and respective `/usr/local/etc/svnup.conf` as (for 11.0-RELEASE):
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
