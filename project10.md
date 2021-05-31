## Load Balancer Solution With Nginx and SSL/TLS

### **`Lessons:`**

>Understanding of how to setup an Nginx loadbalncer, configure A-record, setup a domain and have an Elastic IP mapped to it. I also became familaiar with configuring SSL/TLS - uses digital certificates (which is required if I want to secure my web solutions through encryption and connection over HTTP using HTTPS protocol)

<br/>

## Part 1 — Configure Nginx As A Load Balancer

> I uninstalled Apache from the existing Load Balancer server (EC2 VM based on Ubuntu Server 20.04 LTS) and in addition to TCP port 80 that was opened before for HTTP connections, I also opened TCP port 443 (- this port is used for secured HTTPS connections).  
The /etc/hosts file for local DNS with Web Servers’ names ( Web1 and Web2) and their local IP addresses remain the same because of re-use.

> I installed and configured Nginx as a load balancer as per steps below to point traffic to the resolvable DNS names of the webservers.

<br/>

`Steps:`

- Remove Apache:

> root@ip-172-31-28-70:/home/ubuntu# systemctl stop apache2  
root@ip-172-31-28-70:/home/ubuntu# systemctl status apache2
```
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: inactive (dead) since Fri 2021-05-28 20:14:44 UTC; 2min 4s ago
```
> *root@ip-172-31-28-70:/home/ubuntu# apt remove apache2  
root@ip-172-31-28-70:/home/ubuntu# apt purge apache2*

- Install Nginx:

> *root@ip-172-31-28-70:/home/ubuntu# apt update  
root@ip-172-31-28-70:/home/ubuntu# apt install nginx*


- Configure Nginx LB using Web Servers’ names defined in /etc/hosts


> root@ip-172-31-28-70:/home/ubuntu# cat /etc/nginx/nginx.conf
```
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        upstream myproject {
            server Web1 weight=5;
            server Web2 weight=5;
         }

       server {
           listen 80;
           server_name www.domain.com;
           location / {
             proxy_pass http://myproject;
           }
         }

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        #include /etc/nginx/sites-enabled/*;

```
> *root@ip-172-31-28-70:/home/ubuntu#systemctl restart nginx  
root@ip-172-31-28-70:/home/ubuntu#systemctl status nginx*

<br/>

## Part 2 — Register a new domain name and configure secured connection using SSL/TLS certificates

 *To make connections to our Tooling Web Solution secure, I registered a new domain name and obtained a valid SSL certificate as required.*

 `Steps:`
1. Registered a new domain name with Godaddy.
2. Assigned an Elastic IP to my Nginx LB server and associated my domain name with this Elastic IP (Elastic IP enables us to have a static IP address that does not change after reboot).

![Elastic IP setup and mapping](https://user-images.githubusercontent.com/70076627/120226102-70227780-c23e-11eb-92e4-295c8dee0730.PNG)


3. I Updated my `A-record` on Godaddy site via Domain settings to point to Nginx LB using Elastic IP address.
Under  Additional Settings --> Manage DNS. I entered the below details:
```
Type A
Name @
Value 34.218.98.128
TTL custom
```
`@ by default points to my domain name`

4. I configured Nginx to recognize my new domain name.

> *root@ip-172-31-28-70:/home/ubuntu# vi /etc/nginx/nginx.conf*
```
server {
           listen 80;
           server_name www.romiproject.co.uk;
           location / {
```
I tested it by accessing the domain http://your-domain-name.com> from my browser.

<br/>

5. Install certbot and request for an SSL/TLS certificate
- Below checks whether `snapd` service is active and running:
> *root@ip-172-31-28-70:/home/ubuntu# systemctl status snapd*

```
● snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset:>
     Active: active (running) since Fri 2021-05-28 19:48:05 UTC; 1 day 21h ago
TriggeredBy: ● snapd.socket
   Main PID: 446 (snapd)
      Tasks: 9 (limit: 1160)
     Memory: 62.3M
     CGroup: /system.slice/snapd.service
             └─446 /usr/lib/snapd/snapd
```
- Installed certbot
> *root@ip-172-31-28-70:/home/ubuntu# snap install --classic certbot*    
certbot 1.15.0 from Certbot Project (certbot-eff✓) installed

- I requested for certificate, and followed the certbot instructions and inputs required to fulfil them.

> *root@ip-172-31-28-70:/home/ubuntu# sudo ln -s /snap/bin/certbot /usr/bin/certbot*

> *root@ip-172-31-28-70:/home/ubuntu# certbot --nginx*
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: www.romiproject.co.uk
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Requesting a certificate for www.romiproject.co.uk
Performing the following challenges:
http-01 challenge for www.romiproject.co.uk
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/nginx.conf
Redirecting all traffic on port 80 to ssl in /etc/nginx/nginx.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://www.romiproject.co.uk
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Subscribe to the EFF mailing list (email: elithomrom@gmail.com).

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.romiproject.co.uk/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.romiproject.co.uk/privkey.pem
   Your certificate will expire on 2021-08-28. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again with the "certonly" option. To non-interactively
   renew *all* of your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

<br/>

**`I tested secured access to my Web Solution by trying to reach 
https://www.romiproject.co.uk. I was able to access it successfully.`**

The below captures the result from different browsers - chrome and Microsoft Edge.

![Elastic_LB IP https site displayed - Microsoft edge](https://user-images.githubusercontent.com/70076627/120232106-c4cbef80-c24a-11eb-9790-1fa4c40e0eec.PNG)

![cert website issuer info_1 - valid -Microsoft Edge](https://user-images.githubusercontent.com/70076627/120232124-ceedee00-c24a-11eb-9680-e6363619ce80.PNG)

![Elastic_LB IP https site displayed](https://user-images.githubusercontent.com/70076627/120232073-b67dd380-c24a-11eb-89dd-557fba5e9aa5.PNG)

![cert website issuer info_1 - invalid -chrome](https://user-images.githubusercontent.com/70076627/120232133-d2817500-c24a-11eb-87c5-36d9e25becd3.PNG)


<br/>

> For setting up periodical renewal of my SSL/TLS certificate: this can be done by using the `certbot renew` command and it can be scheduled to periodically run with the help of `crontab`.   
Good to note that by default, LetsEncrypt certificate is valid for 90 days. Renewal can be done at least every 60 days or more frequently.

- To test renewal command in dry-run mode:

> *root@ip-172-31-28-70:/home/ubuntu# certbot renew --dry-run*
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/www.romiproject.co.uk.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Account registered.
Simulating renewal of an existing certificate for www.romiproject.co.uk
Performing the following challenges:
http-01 challenge for www.romiproject.co.uk
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/www.romiproject.co.uk/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/www.romiproject.co.uk/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
`Checking out the files generated:`  
> *root@ip-172-31-28-70:/home/ubuntu# cd /etc/letsencrypt/renewal  
root@ip-172-31-28-70:/etc/letsencrypt/renewal# ls -l*
```
total 4
-rw-r--r-- 1 root root 562 May 30 17:42 www.romiproject.co.uk.conf
```
> *root@ip-172-31-28-70:/etc/letsencrypt/renewal# cat www.romiproject.co.uk.conf*
```
# renew_before_expiry = 30 days
version = 1.15.0
archive_dir = /etc/letsencrypt/archive/www.romiproject.co.uk
cert = /etc/letsencrypt/live/www.romiproject.co.uk/cert.pem
privkey = /etc/letsencrypt/live/www.romiproject.co.uk/privkey.pem
chain = /etc/letsencrypt/live/www.romiproject.co.uk/chain.pem
fullchain = /etc/letsencrypt/live/www.romiproject.co.uk/fullchain.pem

# Options used in the renewal process
[renewalparams]
account = xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
authenticator = nginx
installer = nginx
server = https://acme-v02.api.letsencrypt.org/directory
```

> *root@ip-172-31-28-70:/etc/letsencrypt/live/www.romiproject.co.uk# ls -l*
```
total 4
-rw-r--r-- 1 root root 692 May 30 17:42 README
lrwxrwxrwx 1 root root  45 May 30 17:42 cert.pem -> ../../archive/www.romiproject.co.uk/cert1.pem
lrwxrwxrwx 1 root root  46 May 30 17:42 chain.pem -> ../../archive/www.romiproject.co.uk/chain1.pem
lrwxrwxrwx 1 root root  50 May 30 17:42 fullchain.pem -> ../../archive/www.romiproject.co.uk/fullchain1.pem
lrwxrwxrwx 1 root root  48 May 30 17:42 privkey.pem -> ../../archive/www.romiproject.co.uk/privkey1.pem
```

### As best practice is to have a scheduled job to help run renew command (certbot renew) periodically, we will configure a cronjob to run the command twice a day.

`To do so, we edit the crontab file with the following command:`

> *root@ip-172-31-28-70:~# crontab -e*
```
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed
```
I pasted the below in the file:
```
 * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

> *root@ip-172-31-28-70:/home/ubuntu# cd /var/spool/cron/crontabs  
root@ip-172-31-28-70:/var/spool/cron/crontabs# ls -l  
total 4  
-rw------- 1 root crontab 1149 May 30 20:25 root  
root@ip-172-31-28-70:/var/spool/cron/crontabs# cat root*
```
# DO NOT EDIT THIS FILE - edit the master and reinstall.
# (/tmp/crontab.4oU3BG/crontab installed on Sun May 30 20:25:07 2021)
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
```

`I will practice this project more times to help me retain all the steps required to achieve the solution.`