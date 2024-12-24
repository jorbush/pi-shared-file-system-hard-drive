# Set up a shared file system on a Raspberry Pi with external hard drives

## Format drive

See disks:
```sh
lsblk -f
```
You will see something like this:
```
raspberry@raspberrypi:~ $ lsblk -f
NAME        FSTYPE   FSVER LABEL                     UUID                                 FSAVAIL FSUSE% MOUNTPOINT
loop0       squashfs 4.0                                                                        0   100% /snap/btop/784
loop1       squashfs 4.0                                                                        0   100% /snap/btop/794
loop2       squashfs 4.0                                                                        0   100% /snap/core/17202
loop3       squashfs 4.0                                                                        0   100% /snap/core/16932
loop4       squashfs 4.0                                                                        0   100% /snap/core22/1721
loop5       squashfs 4.0                                                                        0   100% /snap/core22/1437
sda
├─sda1      ntfs           Reservado para el sistema 04361FBB361FACA4                       73.7M    26% /media/raspberry/Reservado para el sistema
└─sda2      ntfs                                     4CF02256F0224714                      223.5G    25% /media/raspberry/4CF02256F0224714
sdb
└─sdb1      vfat     FAT32                           CD41-F882                                29G     0% /media/raspberry/CD41-F882
sdc         ext4     1.0                             120134c2-fea4-4ae4-9820-7a5698bdb955  555.9G     0% /mnt/drive1
sdd
├─sdd1      vfat     FAT32 PQSERVICE                 A20A-9608
├─sdd2      ntfs           ACER                      A22AA8492AA81BF3                       67.1G    40% /media/raspberry/ACER
└─sdd3      ntfs           DATA                      FE923C7D923C3D09                      111.5G     0% /media/raspberry/DATA
mmcblk0
├─mmcblk0p1 vfat     FAT32 boot                      17B6-FC00                             200.8M    20% /boot
└─mmcblk0p2 ext4     1.0   rootfs                    b101bb80-3338-4b94-a775-b3844f8f2aa8   49.4G    11% /
```
Format the drive the you want:
```sh
sudo mkfs.ext4 /dev/sdd
```
> [!NOTE]
> Replace `sdd` for the drive that you want

## Mount the drive

Create your mount directory:
```sh
sudo mkdir /mnt/drive2
```
Mount:
```sh
sudo mount /dev/sdd /mnt/drive2
```
Add it to `/etc/fstab` (you can use `nano` too):
```sh
sudo vi /etc/fstab
```
And add the following line with the uuid of your drive (you can see it using `lsblk -f` that we have used at the beginning)
```
/dev/disk/by-uuid/be89d5f1-815c-4f88-b691-48662ecf1884 /mnt/drive2 auto nosuid,nodev,nofail,x-gvfs-show 0 0
```
Update permissions:
```sh
sudo chown -R raspberry:raspberry /mnt
```

## Enable shared file system

Install samba:
```sh
sudo apt-get install samba
```
Write a new entry in the config:
```
sudo vi /etc/samba/smb.conf
```
With this content at the end:
```
[drive2]
  comment = My shared folder 2
  path = /mnt/drive2
  browseable = yes
  read only = no
  guest ok = no
  create mask = 0755
  directory mask = 0755
  valid users = raspberry
```
Restart samba:
```sh
sudo systemctl restart smbd
```
Add user for samba:
```sh
sudo smbpasswd -a raspberry
```
In other computer you can add the device in `Network`:
```sh
smb://<IP-Raspberry>/drive1
```
