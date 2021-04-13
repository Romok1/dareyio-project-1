### WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

### **`I learnt the following:`**

>Different technology stacks: LAMP, LEMP, MERN, MEAN.

>How to login to AWS EC2 instance using putty and PEM file conversion to PPK.

>Installing/setting up Apache HTTP Server and accessing it via the created Instance running on Ubuntu over Public Ip Address or the hostname on port 80.

>Installing/setting up MySQL server and the security script that is recommended to be run.

>I learnt how to install PHP and its modules and why the dependencies are required.

>I was clear on how to set up virtual hosts for websites using Apache. This allows for one to be able to setup multiple websites on a single machine.

>Learnt how to create index.hmtl and index.php inside a default or custom web root folder. 

>Enabling PHP by modifying the content of the default DirectoryIndex. This is achieved by changing the order.  

<br/>

### **`Below are some of the logs and screenshots of the results generated:`** 

<br/>

## 1. Installing Apache and Updating the Firewall 

<br/>

>*After spinning up an EC2 Instance on AWS, I logged in through Secure crt (I prefer it to Putty because of more flexibility and better aesthetics even though it requires license after 30 days).*

>**ubuntu@ip-172-31-1-200:~$ sudo apt update**

>***ubuntu@ip-172-31-1-200:~$ sudo apt install apache2
Reading package lists... Done***

>**ubuntu@ip-172-31-1-200:~$ sudo systemctl status apache2** * apache2.service - The Apache HTTP Server*
     *Loaded: loaded (/lib/systemd/system/apache2.service; *enabled; vendor preset: enabled)*
    * Active: active (running) since Mon 2021-04-12 11:13:42* *UTC; 2min 3s ago*

> **ubuntu@ip-172-31-1-200:~$ curl http://localhost:80**   
*!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">* 

> **ubuntu@ip-172-31-1-200:~$ curl http://127.0.0.1:80**  
`<p><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 </p>`Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">* </p>

![Apache2 Ubuntu Default page snapshot](https://user-images.githubusercontent.com/70076627/114529325-c06b5900-9c41-11eb-9b79-64fc4841b620.PNG)

<br/>

## 2. Installing MySQL
<br/>

>*ubuntu@ip-172-31-1-200:~$ sudo apt install mysql-server*

>*ubuntu@ip-172-31-1-200:~$ sudo mysql_secure_installation*  
*Securing the MySQL server deployment.*
*Connecting to MySQL using a blank password.*<br/>
...<br/>
...<br/>
...<br/>
*Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y<br/>
Success.*<br/><br/>
*All done!*


>*ubuntu@ip-172-31-1-200:~$ sudo mysql*  
*Welcome to the MySQL monitor.<br/>Commands end with ; or \g.*<br/>
*Your MySQL connection id is 10*<br/>
*Server version: 8.0.23-0ubuntu0.20.04.1 (Ubuntu)* <br/>. .

>*mysql>*  
*mysql> quit*  
*Bye* 

<br/>

## 3. Installing PHP 

<br/>

>*ubuntu@ip-172-31-1-200:~$ sudo apt install php*<br/>    *libapache2-mod-php php-mysql*<br/>
*Reading package lists... Done*<br/>
*Building dependency tree*     

>*ubuntu@ip-172-31-1-200:~$ php -v*<br/>
*PHP 7.4.3 (cli) (built: Oct  6 2020 15:47:56) ( NTS )*<br/>
*Copyright (c) The PHP Group*<br/>
*Zend Engine v3.4.0, Copyright (c) Zend Technologies*
    *with Zend OPcache v7.4.3, Copyright (c), by Zend Technologies*
<br/>
<br/>

<br/>

## 4. Creating a Virtual Host for your Website using Apache 
<br/>

> Created a different directory for the new domain instead of using default /var/www/html.  
*ubuntu@ip-172-31-1-200:~$ sudo mkdir /var/www/projectlamp*  
*ubuntu@ip-172-31-1-200:~$ sudo chown -R $USER:$USER /var/www/projectlamp*

> *ubuntu@ip-172-31-1-200:~$ sudo vi /etc/apache2/sites-available/projectlamp.conf*

> `I pasted the below in it and saved:`  
    ServerName projectlamp  
    ServerAlias www.projectlamp   
    ServerAdmin webmaster@localhost  
    DocumentRoot /var/www/projectlamp  
    ErrorLog ${APACHE_LOG_DIR}/error.log   
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

> *ubuntu@ip-172-31-1-200:~$ sudo ls /etc/apache2/sites-available*  
*000-default.conf  default-ssl.conf  projectlamp.conf*

> `To enable new virtual host:`  
*ubuntu@ip-172-31-1-200:~$ sudo a2ensite projectlamp  
Enabling site projectlamp.  
To activate the new configuration, you need to run:  
  systemctl reload apache2*

>*`To disable Apache’s default website (when one is not using a custom domain):`  
ubuntu@ip-172-31-1-200:~$ sudo a2dissite 000-default  
Site 000-default disabled.  
To activate the new configuration, you need to run:  
  systemctl reload apache2*

>*`To check for syntax error:`
ubuntu@ip-172-31-1-200:~$ sudo apache2ctl configtest
Syntax OK
ubuntu@ip-172-31-1-200:~$ sudo systemctl reload apache2*

> *`Creating index.html and testing the virtual host:`  
ubuntu@ip-172-31-1-200:~$ sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html*

> *ubuntu@ip-172-31-1-200:~$ cat /var/www/projectlamp/index.html
Hello LAMP from hostname ec2-34-215-73-24.us-west-2.compute.amazonaws.com with public IP 34.215.73.24*    
</br>


**My Apache server worked as expected, screenshot below:**

![Apache2 my lamp server - PublicIP](https://user-images.githubusercontent.com/70076627/114529582-00324080-9c42-11eb-95c3-bddef2535826.PNG)<br/>
<br/>
<br/>
![Apache2 my lamp server - dnsserver name](https://user-images.githubusercontent.com/70076627/114530036-7b93f200-9c42-11eb-8c13-21dec43e767e.PNG)<br/>
<br/>

## 5. Enable PHP on the website
<br/>

>*ubuntu@ip-172-31-1-200:~$ sudo vim /etc/apache2/mods-enabled/dir.conf*

>*I modified as below and saved, next time will "cat" it for the sake of logging.* 
<br/>     
*DirectoryIndex `index.php` index.html index.cgi index.pl index.xhtml index.htm*

>*ubuntu@ip-172-31-1-200:~$ sudo systemctl reload apache2*
 
>*ubuntu@ip-172-31-1-200:~$ sudo vi /var/www/projectlamp/index.php*

> <br/>I pasted the below into it and saved.   
<br/>
*<?php*  
*phpinfo();*

> <br/>I refreshed the http site and got the page below: 
<br/>                
![Apache2 my lamp server - PHP](https://user-images.githubusercontent.com/70076627/114531587-f1e52400-9c43-11eb-8422-79cbf5053c14.PNG)

> <br/>I brought down the page by removing the php file as it contained sensitive information.
<br/>  
*ubuntu@ip-172-31-1-200:~$ sudo rm /var/www/projectlamp/index.php*
*ubuntu@ip-172-31-1-200:~$*

#### `I must say, I gained a lot from this Project_1, on to the next!`
~~Just playing around with the markdown language, fun!~~