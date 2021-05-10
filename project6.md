## Web Solution With WordPress

### **`I learnt the following:`**

>I configured storage subsystem for Web and Database servers based on Linux OS.

>I learnt how to deploy web solutions by installing WordPress and connecting it to a remote MySQL database server.

> I became more familiar with disk and storage management as I praticalized it.

<br/>

`The plan for this web solution project is to set up a 3-Tier architecture. Requirements below:` 
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install Wordpress)
- An EC2 Linux server as a database (DB) server

<br/>

## Step 1 — Prepare a Web Server

`I Launched an EC2 instance that will serve as “Web Server”. 
I Created 3 volumes in the same AZ as my Web Server EC2, each of 10 GiB. Then, I attached the volumes to my Instance`

![Create volume proj 6](https://user-images.githubusercontent.com/70076627/117556634-da426500-b062-11eb-88b1-6fc8c55f9e55.PNG)


**`From my terminal:`**

- I ran the lsblk (inspect what block devices are attached to the server), df -h (to check mounts and free spaces)
> *[root@ip-172-31-25-98 ec2-user]# **lsblk**  
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT  
xvda    202:0    0  10G  0 disk   
├─xvda1 202:1    0   1M  0 part   
└─xvda2 202:2    0  10G  0 part /  
xvdf    202:80   0  10G  0 disk /newStorage  
xvdg    202:96   0  10G  0 disk   
xvdh    202:112  0  10G  0 disk*  

> *[root@ip-172-31-25-98 ec2-user]# **df -h**  
Filesystem      Size  Used Avail Use% Mounted on  
devtmpfs        378M     0  378M   0% /dev  
tmpfs           403M     0  403M   0% /dev/shm  
tmpfs           403M   11M  393M   3% /run  
tmpfs           403M     0  403M   0% /sys/fs/cgroup  
/dev/xvda2       10G  1.2G  8.9G  12% /  
tmpfs            81M     0   81M   0% /run/user/1000*  

- Then I used the `gdisk` utility to create a single partition.


> *[root@ip-172-31-25-98 ec2-user]# sudo gdisk /dev/xvdf*
```
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): F3C3E2BA-0EC4-4032-ACC3-CD019CE12477
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): F3C3E2BA-0EC4-4032-ACC3-CD019CE12477
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
```

- I did same for the rest;
> *[root@ip-172-31-25-98 ec2-user]# sudo gdisk /dev/xvdg  
[root@ip-172-31-25-98 ec2-user]# sudo gdisk /dev/xvdh*

- I ran the lsblk command to check the blocked devices again and the partitions.
> *[root@ip-172-31-25-98 ec2-user]# **lsblk***
```
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk 
├─xvda1 202:1    0   1M  0 part 
└─xvda2 202:2    0  10G  0 part /
xvdf    202:80   0  10G  0 disk 
└─xvdf1 202:81   0  10G  0 part 
xvdg    202:96   0  10G  0 disk 
└─xvdg1 202:97   0  10G  0 part 
xvdh    202:112  0  10G  0 disk 
└─xvdh1 202:113  0  10G  0 part 
```

- I installed `lvm2 package` (package manager) and used the `lvmdiskscan` command to check for available partitions. 

> *[root@ip-172-31-25-98 ec2-user]# yum install lvm2*
```
Last metadata expiration check: 0:48:09 ago on Thu 06 May 2021 08:07:07 PM UTC.
Dependencies resolved.
...
 to mark each of 3 disks as physical volumes (PVs) to be used by LVM


 [root@ip-172-31-25-98 ec2-user]# lvmdiskscan
  /dev/xvda2 [     <10.00 GiB] 
  /dev/xvdf1 [     <10.00 GiB] 
  /dev/xvdg1 [     <10.00 GiB] 
  /dev/xvdh1 [     <10.00 GiB] 
  0 disks
  4 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes
```


  - I used `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.

  > *[root@ip-172-31-25-98 ec2-user]# pvcreate /dev/xvdf1  
  Physical volume "/dev/xvdf1" successfully created.  
[root@ip-172-31-25-98 ec2-user]# pvcreate /dev/xvdg1  
  Physical volume "/dev/xvdg1" successfully created.  
[root@ip-172-31-25-98 ec2-user]# pvcreate /dev/xvdh1  
  Physical volume "/dev/xvdh1" successfully created.*  

- To check that the physical volume has been created successfully:
> *[root@ip-172-31-25-98 ec2-user]# **pvs**  
  PV         VG Fmt  Attr PSize   PFree    
  /dev/xvdf1    lvm2 ---  <10.00g <10.00g  
  /dev/xvdg1    lvm2 ---  <10.00g <10.00g  
  /dev/xvdh1    lvm2 ---  <10.00g <10.00g*


- I used `vgcreate` utility to add all 3 PVs to a volume group:
> *[root@ip-172-31-25-98 ec2-user]# vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1  
  Volume group "webdata-vg" successfully created.*

  - To check that it was successfully created.:
  > *[root@ip-172-31-25-98 ec2-user]# **vgs**  
  VG         #PV #LV #SN Attr   VSize   VFree    
  webdata-vg   3   0   0 wz--n- <29.99g <29.99g*

  - I used `lvcreate` utility to create 2 logical volumes.

  > *[root@ip-172-31-25-98 ec2-user]# **lvcreate -n apps-lv -L 14G webdata-vg**  
  Logical volume "apps-lv" created.*

> *[root@ip-172-31-25-98 ec2-user]# lvcreate -n logs-lv -L 14G webdata-vg  
  Logical volume "logs-lv" created.*

  - To verify:
  > *[root@ip-172-31-25-98 ec2-user]# **lvs**  
  LV      VG         Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  apps-lv webdata-vg -wi-a----- 14.00g                                                      
  logs-lv webdata-vg -wi-a----- 14.00g*  

  - To verify the entire setup, I ran the `vgdisplay` -v command:

  > *[root@ip-172-31-25-98 ec2-user]# **vgdisplay -v***

```
  --- Volume group ---
  VG Name               webdata-vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <29.99 GiB
  PE Size               4.00 MiB
  Total PE              7677
  Alloc PE / Size       7168 / 28.00 GiB
  Free  PE / Size       509 / <1.99 GiB
  VG UUID               UTrWOh-xLrI-JZ3Q-xWaY-NbW1-Z2yI-aJGWo8
   
  --- Logical volume ---
  LV Path                /dev/webdata-vg/apps-lv
  LV Name                apps-lv
  VG Name                webdata-vg
  LV UUID                pB5wVe-9bgL-Opl2-LxJR-51Sd-m9c8-EVXX2k
  LV Write Access        read/write
  LV Creation host, time ip-172-31-25-98.us-west-2.compute.internal, 2021-05-06 21:16:02 +0000
  ...
  ...
  ...
```

  - I formatted the logical volumes with ext4 filesystem using `mkfs.ext4`.

 > *[root@ip-172-31-25-98 ec2-user]# **mkfs -t ext4 /dev/webdata-vg/apps-lv**  
```
mke2fs 1.45.6 (20-Mar-2020)  
Creating filesystem with 3670016 4k blocks and 917504 inodes  
Filesystem UUID: d3009cbc-9266-4497-897a-5ebde285bd40  
Superblock backups stored on blocks:   
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208*  

Allocating group tables: done                              
Writing inode tables: done                              
Creating journal (16384 blocks): done  
Writing superblocks and filesystem accounting information: done     
```
> *[root@ip-172-31-25-98 ec2-user]# **mkfs -t ext4 /dev/webdata-vg/logs-lv***
```
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 3670016 4k blocks and 917504 inodes
Filesystem UUID: f63b78c0-b479-4a29-8539-1824e26b46e4
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done   
```

I created directories to store website files and log data.

> *[root@ip-172-31-25-98 ec2-user]# mkdir -p /var/www/html*

> *[root@ip-172-31-25-98 ec2-user]# mkdir -p /home/recovery/logs*


I mounted the new directories on the logical volumes created.
> *[root@ip-172-31-25-98 ec2-user]# mount /dev/webdata-vg/apps-lv /var/www/html*

Before mounting the logical volume for the log directory, backup is required:

> *[root@ip-172-31-25-98 ec2-user]# sudo rsync -av /var/log/. /home/recovery/logs/*
```
sending incremental file list
./
boot.log
btmp
choose_repo.log
cloud-init-output.log
cloud-init.log
cron
dnf.librepo.log
dnf.log
dnf.rpm.log
...
...
```

> *[root@ip-172-31-25-98 ec2-user]# **mount /dev/webdata-vg/logs-lv /var/log***

> *[root@ip-172-31-25-98 logs]# **rsync -av /home/recovery/logs/. /var/log**  
sending incremental file list  
./*


`The /etc/fstab had to be updated to persist the configuration after restart of the server:`

- To get the UUID information:
> *[root@ip-172-31-25-98 logs]# **blkid**  
/dev/xvda2: UUID="949779ce-46aa-434e-8eb0-852514a5d69e" BLOCK_SIZE="512"   TYPE="xfs" PARTUUID="f3dc379a-e15b-46c0-82ff-b9c089f7f27f"  
/dev/xvda1: PARTUUID="1227fbb3-a2f6-4933-a7ac-98432f8f15af"  
/dev/xvdh1: UUID="aLwJvl-OfJD-yCAW-C93D-Ggi9-PH1f-v3pSzx"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="536f3399-f7b1-473a-95c8-7a7f8d172d0f"  
/dev/xvdg1: UUID="yfUmbV-HdJL-lNpt-zvYC-EC2x-q9xW-TYIzUA"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="ba8333be-fb2d-4567-a330-004e85318b7d"  
/dev/xvdf1: UUID="G7vUzF-f2YP-dnxP-cElc-sQqT-zNpV-SIsU3S"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="ebd5ea68-de2d-4dfe-8b50-017b9eca3ccd"  
/dev/mapper/webdata--vg-apps--lv:   UUID="d3009cbc-9266-4497-897a-5ebde285bd40" BLOCK_SIZE="4096" TYPE="ext4"  
/dev/mapper/webdata--vg-logs--lv:   UUID="f63b78c0-b479-4a29-8539-1824e26b46e4" BLOCK_SIZE="4096" TYPE="ext4"*

<br/>

> *[root@ip-172-31-25-98 logs]# vi /etc/fstab*

> *[root@ip-172-31-25-98 logs]# cat /etc/fstab* 
```
#
# /etc/fstab
# Created by anaconda on Sat Oct 31 05:00:52 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=949779ce-46aa-434e-8eb0-852514a5d69e /                       xfs     defaults        0 0
#/dev/xvdf   /newStorage   ext4    defaults,nofail,    0        0
#mounts for wordpress webserver
UUID=d3009cbc-9266-4497-897a-5ebde285bd40    /var/www/html       ext4    defaults      0  0
UUID=f63b78c0-b479-4a29-8539-1824e26b46e4   /var/log             ext4    defaults      0  0
```


- To test the configuration:
> *[root@ip-172-31-25-98 logs]# **mount -a***


- Reloaded the daemon:
> *[root@ip-172-31-25-98 logs]# **systemctl daemon-reload***

- Verified my setup:
> *[root@ip-172-31-25-98 logs]# **df -h**  
Filesystem                        Size  Used Avail Use% Mounted on  
devtmpfs                          378M     0  378M   0% /dev  
tmpfs                             403M     0  403M   0% /dev/shm  
tmpfs                             403M   16M  388M   4% /run  
tmpfs                             403M     0  403M   0% /sys/fs/cgroup  
/dev/xvda2                         10G  1.2G  8.8G  12% /  
tmpfs                              81M     0   81M   0% /run/user/1000  
/dev/mapper/webdata--vg-apps--lv   14G   41M   13G   1% /var/www/html  
/dev/mapper/webdata--vg-logs--lv   14G   54M   13G   1% /var/log*

<br/>

## Step 2 — Prepare the Database Server
`I launched another RedHat EC2 Instance which is to serve as Database Server. I repeated all the procedures in STEP 1 (Storage and logical volume management configuration).`
There is however a slight difference as shown below:

**Instead of app(/var/www/html) directory created for the Applicaton Web server, db/ directory was created here.**

> *[root@ip-172-31-24-248 ec2-user]# **mkdir -p /db***

> *[root@ip-172-31-24-248 ec2-user]# **mount /dev/webdata-vg/db-lv /db***

> *[root@ip-172-31-24-248 ec2-user]# **blkid**  
/dev/xvda2: UUID="949779ce-46aa-434e-8eb0-852514a5d69e" BLOCK_SIZE="512"   TYPE="xfs" PARTUUID="f3dc379a-e15b-46c0-82ff-b9c089f7f27f"  
/dev/xvda1: PARTUUID="1227fbb3-a2f6-4933-a7ac-98432f8f15af"  
/dev/xvdi1: UUID="hdIkTw-dQqk-DWHt-yXZx-gGlk-VFAp-Op8glq"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="22b4f507-a319-4921-ba46-d0604456d0ae"  
/dev/xvdj1: UUID="0Qb15d-VYeh-oCTw-9BYA-0idp-9arh-699x1j"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="cb13d955-4e88-4023-9649-d0550eeacd0a"  
/dev/xvdk1: UUID="5IEUfW-b1aB-TL8K-20oQ-uZ9J-X3lK-4iFQuB"   TYPE="LVM2_member" PARTLABEL="Linux filesystem"   PARTUUID="98a4415c-eb06-4dfd-9efa-be8e991c91af"  
`/dev/mapper/webdata--vg-db--lv: UUID="8d0c860c-1e35-456d-bc76-933554eb6e60" BLOCK_SIZE="4096" TYPE="ext4"`  
`/dev/mapper/webdata--vg-logs--lv:   UUID="b8b71b87-f866-4b79-9c2d-a8abefc2f91a" BLOCK_SIZE="4096" TYPE="ext4"*`

> *[root@ip-172-31-24-248 ec2-user]# cat /etc/fstab* 
```
#
# /etc/fstab
# Created by anaconda on Sat Oct 31 05:00:52 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=949779ce-46aa-434e-8eb0-852514a5d69e /                       xfs     defaults        0 0
UUID=8d0c860c-1e35-456d-bc76-933554eb6e60     /db        ext4    defaults    0    0  
UUID=b8b71b87-f866-4b79-9c2d-a8abefc2f91a     /var/log   ext4    defaults    0    0
```

> *[root@ip-172-31-24-248 ec2-user]# **df -h**  
Filesystem                        Size  Used Avail Use% Mounted on  
devtmpfs                          378M     0  378M   0% /dev  
tmpfs                             403M     0  403M   0% /dev/shm  
tmpfs                             403M   11M  393M   3% /run  
tmpfs                             403M     0  403M   0% /sys/fs/cgroup  
/dev/xvda2                         10G  1.2G  8.8G  12% /  
tmpfs                              81M     0   81M   0% /run/user/1000  
/dev/mapper/webdata--vg-db--lv     14G   41M   13G   1% /db  
/dev/mapper/webdata--vg-logs--lv   14G   53M   13G   1% /var/log*


<br/>

## Step 3 — Install Wordpress on your Web Server EC2
- Repo update:

> *[root@ip-172-31-25-98 ec2-user]# **sudo yum -y update**  
Red Hat Update Infrastructure 3 Client Configuration Server  8                                                                                                            9.6 kB/s | 2.1 kB     00:00    
Red Hat Enterprise Linux 8 for x86_64 - AppStream from RHUI (RPMs)* 


- Installed wget, Apache and it's dependencies:

> *[root@ip-172-31-25-98 ec2-user]# **yum -y install wget httpd php php-mysqlnd php-fpm php-json**  
Last metadata expiration check: 0:05:24 ago on Fri 07 May 2021 06:02:26 PM UTC.  
Dependencies resolved.*

> *[root@ip-172-31-25-98 ec2-user]# systemctl enable httpd  
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.  
[root@ip-172-31-25-98 ec2-user]#   
[root@ip-172-31-25-98 ec2-user]# systemctl start httpd*


> *[root@ip-172-31-25-98 ec2-user]# systemctl status httpd*
```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Fri 2021-05-07 18:10:01 UTC; 11s ago
     Docs: man:httpd.service(8)
 Main PID: 51937 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 4836)
   Memory: 25.1M
   CGroup: /system.slice/httpd.service
           ├─51937 /usr/sbin/httpd -DFOREGROUND
           ├─51943 /usr/sbin/httpd -DFOREGROUND
           ├─51944 /usr/sbin/httpd -DFOREGROUND
           ├─51945 /usr/sbin/httpd -DFOREGROUND
           └─51946 /usr/sbin/httpd -DFOREGROUND
May 07 18:10:01 ip-172-31-25-98.us-west-2.compute.internal systemd[1]: Starting The Apache HTTP Server...
May 07 18:10:01 ip-172-31-25-98.us-west-2.compute.internal systemd[1]: Started The Apache HTTP Server.
May 07 18:10:01 ip-172-31-25-98.us-west-2.compute.internal httpd[51937]: Server configured, listening on: port 80
```

- Installation of PHP and its dependencies:

> *[root@ip-172-31-25-98 ec2-user]# **yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm**  
Last metadata expiration check: 0:11:37 ago on Fri 07 May 2021 06:02:26 PM UTC.  
epel-release-latest-8.noarch.rpm*                                   

> *[root@ip-172-31-25-98 ec2-user]# **yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm***

> *[root@ip-172-31-25-98 ec2-user]#**yum module list php**  
[root@ip-172-31-25-98 ec2-user]# **yum module reset php**  
Last metadata expiration check: 0:00:29 ago on Fri 07 May 2021 06:15:05 PM UTC.  
Dependencies resolved.*

> *[root@ip-172-31-25-98 ec2-user]# **yum module enable php:remi-7.4**  
Last metadata expiration check: 0:01:32 ago on Fri 07 May 2021 06:15:05 PM UTC.    
Dependencies resolved.*

> *[root@ip-172-31-25-98 ec2-user]# **yum install php php-opcache php-gd php-curl php-mysqlnd**  
Last metadata expiration check: 0:02:04 ago on Fri 07 May 2021 06:15:05 PM UTC.  
Package php-7.2.24-1.module+el8.2.0+4601+7c76a223.x86_64 is already installed.  
Package php-common-7.2.24-1.module+el8.2.0+4601+7c76a223.x86_64 is already installed.  
Package php-mysqlnd-7.2.24-1.module+el8.2.0+4601+7c76a223.x86_64 is already installed.  
Dependencies resolved.*

> *[root@ip-172-31-25-98 ec2-user]# **systemctl start php-fpm***

> *[root@ip-172-31-25-98 ec2-user]# systemctl **enable php-fpm**  
Created symlink /etc/systemd/system/multi-user.target.wants/php-fpm.service → /usr/lib/systemd/system/php-fpm.service.*

> *[root@ip-172-31-25-98 ec2-user]# **setsebool -P httpd_execmem 1***


- Restarted Apache after the installations:
> *[root@ip-172-31-25-98 ec2-user]# systemctl restart httpd*


<br/>

- Downloaded Wordpress and copied to /var/www/html.

> *[root@ip-172-31-25-98 ec2-user]# mkdir wordpress  
[root@ip-172-31-25-98 ec2-user]# cd wordpress/  
[root@ip-172-31-25-98 wordpress]# wget http://wordpress.org/latest.tar.gz*


> *[root@ip-172-31-25-98 wordpress]# tar xzvf latest.tar.gz  
wordpress/  
wordpress/xmlrpc.php  
wordpress/wp-blog-header.php  
wordpress/readme.html  
...  
...  
...*


> *[root@ip-172-31-25-98 wordpress]# rm -rf latest.tar.gz  
[root@ip-172-31-25-98 wordpress]# cp wordpress/wp-config-sample.php   wordpress/wp-config.php*


> *[root@ip-172-31-25-98 wordpress]# cp -R wordpress /var/www/html/  
[root@ip-172-31-25-98 wordpress]# ls -l  
total 4  
drwxr-xr-x. 5 nobody nobody 4096 May  7 18:25 wordpress*

> *[root@ip-172-31-25-98 wordpress]# cd wordpress/  
[root@ip-172-31-25-98 wordpress]# ls -l   
total 208  
-rw-r--r--.  1 nobody nobody   405 Feb  6  2020 index.php  
-rw-r--r--.  1 nobody nobody 19915 Jan  1 00:19 license.txt  
-rw-r--r--.  1 nobody nobody  7345 Dec 29 20:14 readme.html  
-rw-r--r--.  1 nobody nobody  7165 Jan 21 01:37 wp-activate.php  
drwxr-xr-x.  9 nobody nobody  4096 Apr 15 02:08 wp-admin  
-rw-r--r--.  1 nobody nobody   351 Feb  6  2020 wp-blog-header.php  
-rw-r--r--.  1 nobody nobody  2328 Feb 17 13:08 wp-comments-post.php  
-rw-r--r--.  1 root   root    2913 May  7 18:25 wp-config.php  
-rw-r--r--.  1 nobody nobody  2913 Feb  6  2020 wp-config-sample.php  
drwxr-xr-x.  4 nobody nobody    52 Apr 15 02:08 wp-content  
-rw-r--r--.  1 nobody nobody  3939 Jul 30  2020 wp-cron.php  
drwxr-xr-x. 25 nobody nobody  8192 Apr 15 02:08 wp-includes  
-rw-r--r--.  1 nobody nobody  2496 Feb  6  2020 wp-links-opml.php  
-rw-r--r--.  1 nobody nobody  3313 Jan 10 19:28 wp-load.php  
-rw-r--r--.  1 nobody nobody 44994 Apr  4 18:34 wp-login.php  
-rw-r--r--.  1 nobody nobody  8509 Apr 14  2020 wp-mail.php  
-rw-r--r--.  1 nobody nobody 21125 Feb  2 00:10 wp-settings.php  
-rw-r--r--.  1 nobody nobody 31328 Jan 27 21:03 wp-signup.php  
-rw-r--r--.  1 nobody nobody  4747 Oct  8  2020 wp-trackback.php  
-rw-r--r--.  1 nobody nobody  3236 Jun  8  2020 xmlrpc.php*


`Configured SELinux Policies:`

> *[root@ip-172-31-25-98 ec2-user]# chown -R apache:apache /var/www/html/wordpress <br/><br/>
[root@ip-172-31-25-98 ec2-user]# chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R <br/>
[root@ip-172-31-25-98 ec2-user]# setsebool -P httpd_can_network_connect=1*

<br/>

## Step 4 — Install MySQL on your DB Server EC2
On my DB server, I installed mysql-server.
> *[root@ip-172-31-24-248 ec2-user]# yum -y update*

> *[root@ip-172-31-24-248 ec2-user]# yum install mysql-server  
Last metadata expiration check: 1:20:05 ago on Sat 08 May 2021 12:44:20 PM UTC.
Dependencies resolved.*

> *[root@ip-172-31-24-248 ec2-user]# systemctl start mysqld*  
> *[root@ip-172-31-24-248 ec2-user]# systemctl enable mysqld*  
Created symlink /etc/systemd/system/multi-user.target.wants/mysqld.service → /usr/lib/systemd/system/mysqld.service.

> *[root@ip-172-31-24-248 ec2-user]# systemctl status mysqld*
```
● mysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2021-05-08 14:15:43 UTC; 2min 35s ago
  Process: 67316 ExecStopPost=/usr/libexec/mysql-wait-stop (code=exited, status=0/SUCCESS)
  Process: 67450 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 67368 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mysqld.service (code=exited, status=0/SUCCESS)
  Process: 67344 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 67405 (mysqld)
   Status: "Server is operational"
    Tasks: 38 (limit: 4836)
   Memory: 373.1M
   CGroup: /system.slice/mysqld.service
           └─67405 /usr/libexec/mysqld --basedir=/usr

May 08 14:15:42 ip-172-31-24-248.us-west-2.compute.internal systemd[1]: Starting MySQL 8.0 database server...
May 08 14:15:43 ip-172-31-24-248.us-west-2.compute.internal systemd[1]: Started MySQL 8.0 database server.
```
<br/>

## Step 5 — Configure DB to work with WordPress

> *[root@ip-172-31-24-248 ec2-user]# **mysql***
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.21 Source distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

> *mysql>  CREATE DATABASE wordpress;   
Query OK, 1 row affected (0.00 sec)*

> *mysql> CREATE USER `admin`@`172.31.25.98` IDENTIFIED BY 'password123';  
Query OK, 0 rows affected (0.00 sec)*

> *mysql> GRANT ALL ON wordpress.* TO `admin`@`172.31.25.98`;  
Query OK, 0 rows affected (0.01 sec)*

> *mysql> FLUSH PRIVILEGES;*  
Query OK, 0 rows affected (0.00 sec)*

> *mysql> SHOW DATABASES;*   
+--------------------+  
| Database           |  
+--------------------+  
| information_schema |  
| mysql              |  
| performance_schema |  
| sys                |  
| wordpress          |  
+--------------------+  
5 rows in set (0.01 sec)  

> *mysql> exit  
Bye*

<br/>

## Step 6 — Configure WordPress to connect to remote database.

Here, I installed mysql-client and logged into the DB server successfully.
> *[root@ip-172-31-24-248 ec2-user]# **yum -y update***

> *[root@ip-172-31-25-98 ec2-user]# **yum install mysql**  
Last metadata expiration check: 1:36:30 ago on Sat 08 May 2021 12:04:24 PM UTC.
Dependencies resolved.*

> *[root@ip-172-31-25-98 ec2-user]# **mysql -u admin -p -h 172.31.24.248***   
Enter password:   
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 11  
Server version: 8.0.21 Source distribution  
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.  <br/><br/>
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.  <br/><br/>
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

`mysql> SHOW DATABASES;`  
+--------------------+  
| Database           |  
+--------------------+  
| information_schema |  
| wordpress          |  
+--------------------+  
2 rows in set (0.00 sec)

`**On my DB Server I opened up MySQL port 3306 to allow only my Web Server's IP address.`  <br/>

`**I opened port 80 on my Web Application server from everywhere (0.0.0.0/0).`

<br/>

### I accessed the public IP included with the wordpress folder (http://Web-Server-Public-IP-Address>/wordpress/) but I did not get the Wordpress page, instead I saw an **"Error establishing a DB connection"** error.

![Error establishing a DB connection Issue](https://user-images.githubusercontent.com/70076627/117589590-5e0f5680-b122-11eb-970c-3f2280734583.PNG)


**I checked for what could be the issue, and I found out that I needed to modify the php config file present in the wordpress folder.**

> *[root@ip-172-31-25-98 ec2-user]# cd /var/www/html  
[root@ip-172-31-25-98 html]# ls -l  
total 20  
drwx------. 2 root   root   16384 May  6 21:29 lost+found  
drwxr-xr-x. 5 apache apache  4096 May  9 10:33 wordpress*

> *[root@ip-172-31-25-98 html]# cd wordpress/*
> *[root@ip-172-31-25-98 wordpress]# ls -l*
total 212
-rw-r--r--.  1 apache apache   405 May  7 18:25 index.php  
-rw-r--r--.  1 apache apache 19915 May  7 18:25 license.txt  
-rw-r--r--.  1 apache apache  7345 May  7 18:25 readme.html  
-rw-r--r--.  1 apache apache  7165 May  7 18:25 wp-activate.php  
drwxr-xr-x.  9 apache apache  4096 May  7 18:25 wp-admin  
-rw-r--r--.  1 apache apache   351 May  7 18:25 wp-blog-header.php  
-rw-r--r--.  1 apache apache  2328 May  7 18:25 wp-comments-post.php  
-rw-r--r--.  1 apache apache  2913 May  7 18:25 wp-config.php  
-rw-r--r--.  1 apache apache  2913 May  7 18:25 wp-config-sample.php  
drwxr-xr-x.  4 apache apache  4096 May  7 18:25 wp-content  
-rw-r--r--.  1 apache apache  3939 May  7 18:25 wp-cron.php  
drwxr-xr-x. 25 apache apache 12288 May  7 18:25 wp-includes  

<br/>

`After editing as below, I was able to access the WordPress site.`
> *[root@ip-172-31-25-98 wordpress]# vi wp-config.php* 
```
/** MySQL database username */
define( 'DB_USER', 'admin' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password123' );

/** MySQL hostname */
define( 'DB_HOST', '172.31.24.248' );
```

> *[root@ip-172-31-25-98 wordpress]# systemctl restart httpd*  

> *[root@ip-172-31-25-98 wordpress]# systemctl status httpd*
```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: d>
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Sun 2021-05-09 10:49:21 UTC; 10s ago
     Docs: man:httpd.service(8)
 Main PID: 5101 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 4753)
   Memory: 24.5M
   CGroup: /system.slice/httpd.service
           ├─5101 /usr/sbin/httpd -DFOREGROUND
           ├─5103 /usr/sbin/httpd -DFOREGROUND
```

![wp-admin wordpress page 1](https://user-images.githubusercontent.com/70076627/117589607-7d0de880-b122-11eb-8c47-746e973b14da.PNG)


![wp-admin wordpress page 2](https://user-images.githubusercontent.com/70076627/117589632-a3cc1f00-b122-11eb-87a0-85f52bb90e00.PNG)
![wp-admin wordpress page 3](https://user-images.githubusercontent.com/70076627/117589634-a6c70f80-b122-11eb-93e5-cd15456c9320.PNG)
![wp-admin wordpress page 4](https://user-images.githubusercontent.com/70076627/117589640-b0e90e00-b122-11eb-80f8-9672e55d0627.PNG)
