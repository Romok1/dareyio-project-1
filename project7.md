## Devops Tooling Website Solution

### **`The biggest takeaways for me:`**

>Understanding of how to setup a 3-tier Web Application architecture with Webservers, Database and NFS server.

> I became even more familiar with setting up Logical Volumes as I got to practicalize it again in this project.

> I learnt how to setup NFS server for file share - end to end.

`Tools, components and Infrastructure used for this project:`

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: GitHub

<br/>

## Step 1 — Prepare NFS Server

- I spinned up a new EC2 instance with RHEL Linux 8 OS.
- I carried out Logical Volume Management as per steps below. I launched three (3) volumes (storage) on AWS each was 10GB in size.

> *[ec2-user@ip-172-31-21-72 ~]$ lsblk  
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT  
xvda    202:0    0  10G  0 disk  
├─xvda1 202:1    0   1M  0 part  
└─xvda2 202:2    0  10G  0 part /  
xvdf    202:80   0  10G  0 disk  
xvdg    202:96   0  10G  0 disk  
xvdh    202:112  0  10G  0 disk*

> *[root@ip-172-31-21-72 ec2-user]# gdisk /dev/xvdf  
[root@ip-172-31-21-72 ec2-user]# gdisk /dev/xvdg  
[root@ip-172-31-21-72 ec2-user]# gdisk /dev/xvdh*

> *[root@ip-172-31-21-72 ec2-user]# lsblk  
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT  
xvda    202:0    0  10G  0 disk  
├─xvda1 202:1    0   1M  0 part  
└─xvda2 202:2    0  10G  0 part /  
xvdf    202:80   0  10G  0 disk  
└─xvdf1 202:81   0  10G  0 part  
xvdg    202:96   0  10G  0 disk  
└─xvdg1 202:97   0  10G  0 part  
xvdh    202:112  0  10G  0 disk  
└─xvdh1 202:113  0  10G  0 part* 

> *[root@ip-172-31-21-72 ec2-user]# yum install lvm2*

> *[root@ip-172-31-21-72 ec2-user]# pvcreate /dev/xvdf1  
  Physical volume "/dev/xvdf1" successfully created. <br/> <br/> 
[root@ip-172-31-21-72 ec2-user]# pvcreate /dev/xvdg1  
  Physical volume "/dev/xvdg1" successfully created. <br/><br/> 
[root@ip-172-31-21-72 ec2-user]# pvcreate /dev/xvdh1  
  Physical volume "/dev/xvdh1" successfully created.*

> *[root@ip-172-31-21-72 ec2-user]# pvs  
  PV         VG Fmt  Attr PSize   PFree  
  /dev/xvdf1    lvm2 ---  <10.00g <10.00g  
  /dev/xvdg1    lvm2 ---  <10.00g <10.00g  
  /dev/xvdh1    lvm2 ---  <10.00g <10.00g*

> *[root@ip-172-31-21-72 ec2-user]# lvmdiskscan**
```
  /dev/xvda2 [     <10.00 GiB]
  /dev/xvdf1 [     <10.00 GiB] LVM physical volume
  /dev/xvdg1 [     <10.00 GiB] LVM physical volume
  /dev/xvdh1 [     <10.00 GiB] LVM physical volume
  0 disks
  1 partition
  0 LVM physical volume whole disks
  3 LVM physical volumes
```
> *[root@ip-172-31-21-72 ec2-user]# vgcreate websol-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1  
  Volume group "websol-vg" successfully created*

> *[root@ip-172-31-21-72 ec2-user]# vgs*
```
  VG        #PV #LV #SN Attr   VSize   VFree
  websol-vg   3   0   0 wz--n- <29.99g <29.99g
```
> *[root@ip-172-31-21-72 ec2-user]# lvcreate -n lv-opt -L 9G websol-vg
  Logical volume "lv-opt" created.*

> *[root@ip-172-31-21-72 ec2-user]# lvcreate -n lv-apps -L 9G websol-vg*
  Logical volume "lv-apps" created.*

> *[root@ip-172-31-21-72 ec2-user]# lvcreate -n lv-logs -L 9G websol-vg
  Logical volume "lv-logs" created.*

> *[root@ip-172-31-21-72 ec2-user]# vgdisplay -v*

- Installed NFS Server and confirmed status

> *[root@ip-172-31-21-72 ec2-user]# yum -y update  
[root@ip-172-31-21-72 ec2-user]# yum install nfs-utils -y  
[root@ip-172-31-21-72 ec2-user]# systemctl start nfs-server.service  
[root@ip-172-31-21-72 ec2-user]# systemctl status nfs-server.service*

> *[root@ip-172-31-21-72 ec2-user]# mkfs.xfs /dev/websol-vg/lv-opt  
[root@ip-172-31-21-72 ec2-user]# mkfs.xfs /dev/websol-vg/lv-apps  
[root@ip-172-31-21-72 ec2-user]# mkfs.xfs /dev/websol-vg/lv-logs*

> *[root@ip-172-31-21-72 ec2-user]# mkdir -p /mnt/apps  
[root@ip-172-31-21-72 ec2-user]# mkdir -p /mnt/logs  
[root@ip-172-31-21-72 ec2-user]# mkdir -p /mnt/opts*

> *[root@ip-172-31-21-72 ec2-user]# mount /dev/websol-vg/lv-opt /mnt/opts  
[root@ip-172-31-21-72 ec2-user]# mount /dev/websol-vg/lv-apps /mnt/apps  
[root@ip-172-31-21-72 ec2-user]# mount /dev/websol-vg/lv-logs /mnt/logs*

*For the log mount, I skipped backup and resync, will ensure to this next time.

> *[root@ip-172-31-21-72 ec2-user]# blkid*
```
/dev/xvda1: PARTUUID="1227fbb3-a2f6-4933-a7ac-98432f8f15af"
/dev/xvda2: UUID="949779ce-46aa-434e-8eb0-852514a5d69e" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="f3dc379a-e15b-46c0-82ff-b9c089f7f27f"
/dev/xvdf1: UUID="3pmOF7-G8PE-40h4-zJvG-QPB0-APtS-8fkz8x" TYPE="LVM2_member" PARTLABEL="Linux filesystem" PARTUUID="56891c63-6d4c-48e5-80d2-80317bf2c773"
/dev/xvdg1: UUID="BC2HO7-Du5x-oQIb-BlW9-S5cH-4Llb-JecX9E" TYPE="LVM2_member" PARTLABEL="Linux filesystem" PARTUUID="73d8bf99-0776-4d79-8050-73449c86c6b0"
/dev/xvdh1: UUID="TOmc7E-ku6X-wawO-oDEM-NH4j-FMTO-GUzV0p" TYPE="LVM2_member" PARTLABEL="Linux filesystem" PARTUUID="af9d68e1-6a40-40c9-b995-f99a0a91a8d7"
/dev/mapper/websol--vg-lv--opt: UUID="f1adb649-ea98-414b-ab52-1cb31d0d7d13" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/websol--vg-lv--apps: UUID="49a1bf16-8252-43ac-bff5-18ed9d15325a" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/websol--vg-lv--logs: UUID="2112ac5a-f0ba-4d43-850c-4727388fa0e1" BLOCK_SIZE="512" TYPE="xfs"
```

> *[root@ip-172-31-21-72 ec2-user]# df -h*
```
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         378M     0  378M   0% /dev
tmpfs                            403M     0  403M   0% /dev/shm
tmpfs                            403M   11M  393M   3% /run
tmpfs                            403M     0  403M   0% /sys/fs/cgroup
/dev/xvda2                        10G  1.3G  8.8G  13% /
tmpfs                             81M     0   81M   0% /run/user/1000
/dev/mapper/websol--vg-lv--opt   9.0G   97M  8.9G   2% /mnt/opts
/dev/mapper/websol--vg-lv--apps  9.0G   97M  8.9G   2% /mnt/apps
/dev/mapper/websol--vg-lv--logs  9.0G   97M  8.9G   2% /mnt/logs
```

> *[root@ip-172-31-21-72 ec2-user]# vi /etc/fstab*

> *[root@ip-172-31-21-72 ec2-user]# mount -a*

> *[root@ip-172-31-21-72 ec2-user]# cat /etc/fstab*
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
UUID=f1adb649-ea98-414b-ab52-1cb31d0d7d13  /mnt/opts    xfs   defaults    0    0
UUID=49a1bf16-8252-43ac-bff5-18ed9d15325a  /mnt/apps    xfs   defaults    0    0
UUID=2112ac5a-f0ba-4d43-850c-4727388fa0e1  /mnt/logs     xfs   defaults   0   0
```
> *[root@ip-172-31-21-72 ec2-user]# systemctl daemon-reload*

- I set up permission that will allow our Web servers to read, write and execute files on NFS:
> *[root@ip-172-31-21-72 ec2-user]# chown -R nobody: /mnt/apps  
[root@ip-172-31-21-72 ec2-user]# chown -R nobody: /mnt/logs  
[root@ip-172-31-21-72 ec2-user]# chown -R nobody: /mnt/opts*

> *[root@ip-172-31-21-72 ec2-user]# chmod -R 777 /mnt/apps  
[root@ip-172-31-21-72 ec2-user]# chmod -R 777 /mnt/logs  
[root@ip-172-31-21-72 ec2-user]# chmod -R 777 /mnt/opts*

> *[root@ip-172-31-21-72 ec2-user]# systemctl restart nfs-server.service*

`So here, I spinned up 3 EC2 Instances on RHEL Linux 8 OS to serve as our webservers. I put them on the same subnet and used it in my NFS configuration.`

- I configured access to NFS for clients within the same subnet and exported the mounts for our webservers.

> *[root@ip-172-31-21-72 ec2-user]# vi /etc/exports*

> *[root@ip-172-31-21-72 ec2-user]# cat /etc/exports*
```
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opts 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
```

> *[root@ip-172-31-21-72 ec2-user]# exportfs -arv*

- I checked which port(s) is used by NFS and opened it using Security Groups. The Inbound rule for the instances were modified to open these ports used by NFS server to allow clients  access it.

> *[root@ip-172-31-21-72 ec2-user]# rpcinfo -p | grep nfs*
```
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
```

I opened up the following ports: TCP 111, UDP 111, UDP 2049.

<br/>

## Step 2 — Configure the database server
- I spinned up a new EC2 instance with Ubuntu 20.04 OS.
- I installed MySQL server, created database, database user, granted permission for access, and opened MySQL port on AWS.   
`Steps are below:`

> *ubuntu@ip-172-31-17-4:~$ sudo su  
root@ip-172-31-17-4:/home/ubuntu# apt update  
root@ip-172-31-17-4:/home/ubuntu# apt install mysql-server  
root@ip-172-31-17-4:/home/ubuntu# mysql_secure_installation*

> *root@ip-172-31-17-4:/home/ubuntu# systemctl status mysql*
```
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset:>
     Active: active (running) since Mon 2021-05-17 07:49:49 UTC; 6min ago
   Main PID: 2777 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 1160)
     Memory: 327.9M
     CGroup: /system.slice/mysql.service
             └─2777 /usr/sbin/mysqld
May 17 07:49:48 ip-172-31-17-4 systemd[1]: Starting MySQL Community Server...
May 17 07:49:49 ip-172-31-17-4 systemd[1]: Started MySQL Community Server.
```
> *root@ip-172-31-17-4:/home/ubuntu# mysql  
mysql>*

> *mysql> CREATE DATABASE tooling;  
Query OK, 1 row affected (0.01 sec)*

> *mysql> CREATE USER `webaccess`@`%` IDENTIFIED BY 'admin123';  
Query OK, 0 rows affected (0.00 sec)*

> *mysql> GRANT ALL PRIVILEGES ON tooling.* TO `webaccess`@`%` WITH GRANT OPTION;  
Query OK, 0 rows affected (0.05 sec)*


> *mysql> select user, host from mysql.user;*
```
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| webaccess        | %         |
| debian-sys-maint | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)
```

> *mysql> SHOW DATABASES;*
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| tooling            |
+--------------------+
5 rows in set (0.00 sec)
```
> *root@ip-172-31-17-4:/home/ubuntu# vi /etc/mysql/mysql.conf.d/mysqld.cnf  
Changed ‘127.0.0.1’ to ‘0.0.0.0’*

> *root@ip-172-31-17-4:/home/ubuntu# systemctl restart mysql*


<br/>

## Step 3 — Prepare the Web Servers
- Three (3) EC2 Instances on RHEL Linux 8 OS were launched to serve as web server.
- It is required that the 3 web servers can serve the same content from shared storage solution (NFS Server) and MySQL DB.
- The folder `(/var/www)` where Apache is stored will be mounted to the previously created NFS logical volume `(lv-apps/ (/mnt/apps))`.

`Setup requirement:`
- Configure NFS client (this step must be done on all three servers).
- Deploy a Tooling application to our Web Servers into a shared NFS folder.
- Configure the Web Servers to work with a single MySQL database.

<br/>

`The following steps were carried out on all the 3 Webservers:`
- Install NFS client

> *[ec2-user@ip-172-31-19-10 ~]$ sudo su  
[root@ip-172-31-19-10 ec2-user]# yum install nfs-utils nfs4-acl-tools -y*

- Mount /var/www and target the NFS server's export for apps  
>*[root@ip-172-31-19-10 ec2-user]# mkdir /var/www  
[root@ip-172-31-19-10 ec2-user]# mount -t nfs -o rw,nosuid 172.31.21.72:/mnt/apps /var/www*

*172.31.21.72 being our NFS server IP. This IP must be reachable from the webservers.

- To verify that NFS was mounted successfully.
> *[root@ip-172-31-19-10 ec2-user]# df -h*
```
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                378M     0  378M   0% /dev
tmpfs                   403M     0  403M   0% /dev/shm
tmpfs                   403M   11M  393M   3% /run
tmpfs                   403M     0  403M   0% /sys/fs/cgroup
/dev/xvda2               10G  1.3G  8.8G  13% /
tmpfs                    81M     0   81M   0% /run/user/1000
172.31.21.72:/mnt/apps  9.0G   97M  8.9G   2% /var/www
```
- To persist the above configuration:
> *[root@ip-172-31-19-10 ec2-user]# vi /etc/fstab*

> *[root@ip-172-31-19-10 ec2-user]# cat /etc/fstab*
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
172.31.21.72:/mnt/apps   /var/www     nfs      defaults  0   0
```

- I also mounted the log folder
> *[root@ip-172-31-19-10 httpd]# mount -t nfs -o rw,nosuid 172.31.21.72:/mnt/logs /var/log/httpd/*

> *[root@ip-172-31-19-10 httpd]# df -h*
```
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                378M     0  378M   0% /dev
tmpfs                   403M     0  403M   0% /dev/shm
tmpfs                   403M   11M  393M   3% /run
tmpfs                   403M     0  403M   0% /sys/fs/cgroup
/dev/xvda2               10G  1.3G  8.8G  13% /
tmpfs                    81M     0   81M   0% /run/user/1000
172.31.21.72:/mnt/apps  9.0G   97M  8.9G   2% /var/www
172.31.21.72:/mnt/logs  9.0G   97M  8.9G   2% /var/log/httpd
```
> *[root@ip-172-31-19-10 httpd]# cat /etc/fstab*
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
172.31.21.72:/mnt/apps   /var/www     nfs      defaults  0   0
172.31.21.72:/mnt/logs   /var/log/httpd/   nfs  defaults   0   0
```

- To Install Apache:  
> *[root@ip-172-31-19-10 ec2-user]# yum install httpd -y*

I tested if the NFS was mounted correctly by creating a new file on the /mnt/apps directory on the NFS server. Same file was seen in the /var/www folder on the webservers which is what we want.

- `For the below requirement:`  
**Fork the tooling source code from Darey.io Github Account to your Github account.** ----
I decided to do a git clone on the NFS server, and transfer the html folder to the /mnt/apps directory so that it can be served to the /var/www of all the webservers automatically.

<br/>

**`ON THE NFS SERVER`**

After this transfer and conf file modification, the html code from repo was successfully deployed to /var/www/html on the web servers.

> *[ec2-user@ip-172-31-21-72 ~]$ sudo su  
[root@ip-172-31-21-72 ec2-user]# yum install git  
[root@ip-172-31-21-72 ec2-user]# cd /tmp 
[root@ip-172-31-21-72 tmp]# git clone https://github.com/darey-io/tooling.git  
Cloning into 'tooling'...*

> *[ec2-user@ip-172-31-21-72 tmp]$ cd tooling/  
[ec2-user@ip-172-31-21-72 tooling]$ ls -l*
```
total 28
-rw-r--r--. 1 root root  332 May 17 23:52 apache-config.conf
-rw-r--r--. 1 root root  313 May 17 23:52 Dockerfile
drwxr-xr-x. 3 root root  205 May 17 23:52 html
-rw-r--r--. 1 root root 4202 May 17 23:52 Jenkinsfile
-rw-r--r--. 1 root root 2331 May 17 23:52 README.md
-rwxr-xr-x. 1 root root  163 May 17 23:52 start-apache
-rw-r--r--. 1 root root 1674 May 17 23:52 tooling-db.sql
```
> *[root@ip-172-31-21-72 tooling]# cp -R html/. /mnt/apps/html/  
[root@ip-172-31-21-72 tooling]# cd /mnt/apps/html/  
[root@ip-172-31-21-72 html]# ls -l*
```
total 40
-rw-r--r--. 1 root root 2909 May 18 00:38 admin_tooling.php
-rw-r--r--. 1 root root 1531 May 18 00:38 create_user.php
-rw-r--r--. 1 root root 4385 May 18 00:38 functions.php
drwxr-xr-x. 2 root root  183 May 18 00:38 img
-rw-r--r--. 1 root root 3162 May 18 00:38 index.php
-rw-r--r--. 1 root root  780 May 18 00:38 login.php
-rw-r--r--. 1 root root   19 May 18 00:38 README.md
-rw-r--r--. 1 root root 1097 May 18 00:38 register.php
-rw-r--r--. 1 root root 1704 May 18 00:38 style.css
-rw-r--r--. 1 root root 1027 May 18 00:38 tooling_stylesheets.css
```

- I modified the function.php on the NFS server:
> *[root@ip-172-31-21-72 html]# vi functions.php*

> *[root@ip-172-31-19-10 html]# cat functions.php*
```
// connect to database
$db = mysqli_connect('172.31.17.4', 'webaccess', 'admin123', 'tooling');
#$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');
```
<br/>

**` BACK ON THE WEBSERVERS`**  
Initially, I created an empty directory and then started httpd. 

> *[root@ip-172-31-19-10 ~]# mkdir /var/www/html  
[root@ip-172-31-19-10 ~]# systemctl start httpd  
[root@ip-172-31-19-10 ~]# systemctl status httpd*
```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-05-16 15:02:18 UTC; 15s ago
     Docs: man:httpd.service(8)
 Main PID: 7661 (httpd)
   Status: "Running, listening on: port 80"
```
I browsed to the Public IP and I got the default Apache page which I had to disable so that the new site can be loaded in it's place.

![Capture default apache page](https://user-images.githubusercontent.com/70076627/118903368-93305b80-b90f-11eb-901b-fc819fa11bd0.PNG)


`I did the below to remove the default site. I will try out other methods of disabling default Apache site when next I am running the project again for practice.`

> *[root@ip-172-31-19-10 html]# mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_bac*

> *[root@ip-172-31-19-10 html]# systemctl restart httpd*

Afer deploying the codes to /var/www/html site, I tried to access the site again.
I disabled SELinux as I was getting "Permission denied error" when I tried to access the webserver Public IPs.  
`The below was done to rectify it.`  
> *[root@ip-172-31-19-10 ec2-user]# sudo setenforce 0  
[root@ip-172-31-19-10 ec2-user]# vi /etc/sysconfig/selinux  
set to SELINUX=disabled  
[root@ip-172-31-19-10 ec2-user]# systemctl restart httpd*

<br/>

### `Connecting my Webservers to MySQL database. ` 

I installed git on one of the webservers to have direct access to the github repo from it so that I can have the `tooling` folder.

> *[ec2-user@ip-172-31-19-10 ~]$ sudo su  
[root@ip-172-31-19-10 ec2-user]# yum install git  
[root@ip-172-31-19-10 ec2-user]# git clone https://github.com/darey-io/tooling.git*

> *[root@ip-172-31-19-10 ec2-user]# cd tooling/  
[root@ip-172-31-19-10 tooling]# ls -l*
```
total 28
-rw-r--r--. 1 root root  332 May 16 14:30 apache-config.conf
-rw-r--r--. 1 root root  313 May 16 14:30 Dockerfile
drwxr-xr-x. 3 root root  205 May 16 14:30 html
-rw-r--r--. 1 root root 4202 May 16 14:30 Jenkinsfile
-rw-r--r--. 1 root root 2331 May 16 14:30 README.md
-rwxr-xr-x. 1 root root  163 May 16 14:30 start-apache
-rw-r--r--. 1 root root 1674 May 16 14:30 tooling-db.sql
```
- MySQL client installation
> *[root@ip-172-31-19-10 html]# yum install mysql -y*

- Access the Database (To test)
> *[root@ip-172-31-19-10 html]# mysql -u webaccess -p -h 172.31.17.4  
Enter password:  
mysql>*

To apply the `tooling-db.sql` script:
> *[root@ip-172-31-19-10 tooling]# mysql -u webaccess -p tooling -h 172.31.17.4 < tooling-db.sql  
Enter password:*

<br/>

### `Lest I forget, Installation of PHP on the webservers to be able to interact with our codes.`

**Before PHP was installed**, I was getting the below page.

![Capture html Webserver 1_before php install](https://user-images.githubusercontent.com/70076627/118904560-076bfe80-b912-11eb-9778-7453c63ff546.PNG)

I used another documentation as guide to get the PHP and its dependencies installed. Steps below:

> *[root@ip-172-31-19-10 ec2-user]# dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm*

> *[root@ip-172-31-19-10 ec2-user]# dnf module list php  
[root@ip-172-31-19-10 ec2-user]# dnf module reset php  
[root@ip-172-31-19-10 ec2-user]# dnf module enable php:remi-7.4  
[root@ip-172-31-19-10 ec2-user]# dnf install php php-opcache   php-gd php-curl php-mysqlnd*

> *[root@ip-172-31-19-10 ec2-user]# php -v  
[root@ip-172-31-19-10 ec2-user]# start php-fpm  
[root@ip-172-31-19-10 ec2-user]# systemctl enable php-fpm  
[root@ip-172-31-19-10 ec2-user]# systemctl status php-fpm*
```
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-05-17 18:15:45 UTC; 18s ago
 Main PID: 12952 (php-fpm)
   Status: "Processes active: 0, idle: 5, Requests: 0, slow: 0, Traffic: 0req/sec"
    Tasks: 6 (limit: 4836)
   Memory: 23.7M
   CGroup: /system.slice/php-fpm.service
           ├─12952 php-fpm: master process (/etc/php-fpm.conf)
           ├─12953 php-fpm: pool www
           ├─12954 php-fpm: pool www
           ├─12955 php-fpm: pool www
           ├─12956 php-fpm: pool www
           └─12957 php-fpm: pool www

May 17 18:15:45 ip-172-31-19-10.us-west-2.compute.internal systemd[1]: Starting The PHP FastCGI>
May 17 18:15:45 ip-172-31-19-10.us-west-2.compute.internal systemd[1]: Started The PHP FastCGI >
```

> *[root@ip-172-31-19-10 ec2-user]# setsebool -P httpd_execmem 1  
[root@ip-172-31-19-10 ec2-user]# systemctl restart httpd*

<br/>

**`ON THE DATABASE SERVER`**  
In preparation of our webserver access to the database, a new admin user with username: **myuser*** and password: **password** was created in MySQL. The below was executed on DB server.

> *mysql> use tooling*

> *mysql> INSERT INTO `users` (`id`, `username`, `password`, `user_type`, `status`) VALUES (1, myuser,‘5f4dcc3b5aa765d61d8327deb882cf99’,‘admin’,‘1’);    
ERROR 1054 (42S22): Unknown column 'myuser' in 'field list'*

The above command failed at first because the quotes that was there from copying and pasting were Incorrect, I had to remove them and used the appropriate quotes.

The below command threw an error because the value that was not quoted was already entered in the DB server, hence the duplicate error result.

> *mysql> INSERT INTO `users` (`id`, `username`, `password`, `email`, `user_type`, `status`) VALUES (1, 'myuser','5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin','1');    
ERROR 1062 (23000): Duplicate entry '1' for key 'users.PRIMARY'*

> *mysql> INSERT INTO `users` (`username`, `password`, `email`, `user_type`, `status`) VALUES ('myuser','5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin','1');    
**Query OK, 1 row affected (0.01 sec)***

<br/>

**`ON THE BROWSER`** 

I opened the website in my browser http://*<Web-Server-Public-IP-Address-or-Public-DNS-Name*/index.php and logged into the website with:  
`myuser/password and webaccess/admin123` credentials.
Accessing the site using the public address of the 3 webservers was successful. Screenshots below:

**Login Page**
![Capture html page 1_Login page](https://user-images.githubusercontent.com/70076627/118904610-21a5dc80-b912-11eb-80f7-e605d728f86f.PNG)

![Capture webserver 1 login page](https://user-images.githubusercontent.com/70076627/118904919-b6103f00-b912-11eb-8845-7e90fcd4c145.PNG)

![Capture webserver 3 login page](https://user-images.githubusercontent.com/70076627/118904920-b6103f00-b912-11eb-9354-0e7e8d5cdbcc.PNG)

**Home Page**
![Capture html page 1_Login page_home page](https://user-images.githubusercontent.com/70076627/118904944-c0cad400-b912-11eb-9b9b-3cc5b168de87.PNG)

![Capture webserver 1 home page](https://user-images.githubusercontent.com/70076627/118904970-cc1dff80-b912-11eb-9660-9cc9609384f0.PNG)

![webserver 3 direct login land in home page](https://user-images.githubusercontent.com/70076627/118904988-d3dda400-b912-11eb-8ca0-1b4fe3e1b7f4.PNG)

<br/>

`The darey.io youtube video on this project assisted me in putting together the missing pieces such as forgetting to install PHP, editing the MySQL script, function.php script and exporting the MySQL script. I was able to put all the little pieces together to achieve a successful project.`