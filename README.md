# Encrypted ZFS on Intel NUC

This example contains build instructions for a container with encrypted zfs file system for user data. These instructions are valid for Intel NUC balenaOS image.


1. Git clone this repository
2. Edit zfs/Dockerfile.template and set OSVERSION and KERNEL variables
3. balena push "appname"
5. Once zfs service is running, ssh to the main OS and run these commands
```
	# mount -o remount,rw /
	# find /mnt/data/docker/ | grep zfs.ko | grep extra
	/mnt/data/docker/aufs/diff/25b034ac33a7d0118496ed19177fd4e0115ec1ea7f3a97b3889874c3079be925/lib/modules/5.2.10-yocto-standard/extra/zfs/zfs.ko
	# cp -rp /mnt/data/docker/aufs/diff/25b034ac33a7d0118496ed19177fd4e0115ec1ea7f3a97b3889874c3079be925/lib/modules/5.2.10-yocto-standard/extra/* /lib/modules/5.2.10-yocto-standard/extra/
	# echo zfs >> /etc/modules-load.d/modules.conf
	# depmod
	# reboot
```
5. Once the system is back up, create new zpool on the SD card from zfs container:
```
    # zpool create mypool /dev/mmcblk0
```
6. Add encrypted file system:
```
    # zfs create -o encryption=aes-256-gcm -o keyformat=passphrase mypool/encryptedfs
    Enter passphrase:
    Re-enter passphrase:
```
When balenaOS reboots, the encryption key will have to be reloaded: 
```
    zfs reload-key mypool/encryptedfs
```

The filesystem can then be mounted
```
    # zfs mount -a
    # zfs mount
    mypool                          /mypool
    mypool/encryptedfs              /mypool/encryptedfs
```