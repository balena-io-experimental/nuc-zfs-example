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

Alternative way for FDE can be achieved with LUKS. This may be necessary when installing zfs versions below 0.8 or any other file system without native encryption. We are going to use /dev/sdb here. Here's what to do:
1. Create LUKS header on /dev/sdb
```
    # lsblk /dev/sdb
    NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sdb    8:16   0  1.4T  0 disk
    # cryptsetup luksFormat /dev/sdb

    WARNING!
    ========
    This will overwrite data on /dev/sdb irrevocably.

    Are you sure? (Type uppercase yes): YES
    Enter passphrase for /dev/sdb:
    Verify passphrase:
    #
```

2. Open the disk and add device mapper entry for it:
```
    # cryptsetup open /dev/sdb crypt
    Enter passphrase for /dev/sdb:
    #
```
3. Create the pool using mapped device:
```
    # zpool create lukspool /dev/mapper/crypt
    # zpool list
    NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
    lukspool  1.36T   112K  1.36T        -         -     0%     0%  1.00x    ONLINE  -
    mypool     119G   400K   119G        -         -     0%     0%  1.00x    ONLINE  -
```