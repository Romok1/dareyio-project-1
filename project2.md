
### WEB STACK IMPLEMENTATION (LEMP STACK)

### **`I learnt the following:`**

>Took the SQL tutorial, I learnt a lot about SQL syntax and commands.

>I connected to my AWS EC2 Instance for the first time using Git Bash. I always used putty or secure crt.

>I learnt how to install Nginx and PHP modules that are required for it.

>I was clear on how to set up Nginx and testing PHP with it.

>I became familiar with configuring SQL Database and SQL User. I also learnt how to configure entries for the database and information retrieval from DB.

> I practised integration of PHP with MySQL.

<br/>

### **`Below are some of the logs and screenshots of the results generated:`** 

<br/>

## 1. Installing the Nginx Web Server

<br/>

> I made use of the existing EC2 instance from project 1, this time around I logged into it using git bash, the process is actually simpler and more straightforward than the other options.

>**ubuntu@ip-172-31-1-200:~$ sudo apt install nginx**

>***ubuntu@ip-172-31-1-200:~$ sudo systemctl status nginx***  
● nginx.service - A high performance web server and a reverse proxy server  
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset:>  
     Active: inactive (dead)  
       Docs: man:nginx(8)

> **ubuntu@ip-172-31-1-200:~$ sudo systemctl start nginx**  
Job for nginx.service failed because the control process exited with error code.  
See "systemctl status nginx.service" and "journalctl -xe" for details.

<br/>

> So I was alarmed when I discovered that nginx would not start. I then checked the output of "journalctl -xe". <br/>      
`Error below was showing:`
-- A start job for unit nginx.service has begun execution.  
--  
-- The job identifier is 11641.  
*Apr 13 19:03:41 ip-172-31-1-200 nginx[47551]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)*  
*Apr 13 19:03:41 ip-172-31-1-200 nginx[47551]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)*

<br/>

> Then I went online to check how to troubleshoot it even though the error message seemed straight forward. What came to mind first was removing the mapping of port 80 for the service using it. But i also needed to check the service in question. (Meanwhile, I forgot at the time that I had used same instance for my Apache setup).

 `Remedy:`  
>I got the below command online and ran it and right there I remembered that Apache which I installed in Project 1 was using the port 80.

> ubuntu@ip-172-31-1-200:~$ sudo netstat -plant | grep 80
sudo: netstat: command not found

> ubuntu@ip-172-31-1-200:~$ sudo apt install net-tools  
Reading package lists... Done

> ubuntu@ip-172-31-1-200:~$ sudo netstat -plant | grep 80
tcp        0      0 172.31.1.200:37924      54.218.137.160:80       TIME_WAIT   -  <br/>      
tcp6       0      0   :::   80                     :::*                   LISTEN          534/apache2

>ubuntu@ip-172-31-1-200:~$ sudo systemctl stop apache2
(I ran this twice and checked for the result again)

> ubuntu@ip-172-31-1-200:~$ sudo netstat -plant | grep 80  
ubuntu@ip-172-31-1-200:~$

> ubuntu@ip-172-31-1-200:~$ sudo systemctl start nginx


> ubuntu@ip-172-31-1-200:~$ sudo systemctl status nginx  
● nginx.service - A high performance web server and a reverse proxy server  
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)  
     `Active: active (running)` since Wed 2021-04-14 13:52:35 UTC; 19s ago  
       Docs: man:nginx(8)


> ubuntu@ip-172-31-1-200:~$ curl http://localhost:80
DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
...  
...  
title>Apache2 Ubuntu Default Page: It works
   
> `Another Problem was observed`: My Apache site was still showing when I ran the curl command.

> I checked online again as to what to do, I read that I needed to remove all the files, settings and properties of Apache2.

`So I did the below steps:`

> ubuntu@ip-172-31-1-200:~$ sudo ls /etc/apache2/sites-available
000-default.conf  default-ssl.conf  projectlamp.conf

> ubuntu@ip-172-31-1-200:/etc/apache2/sites-available$ whereis apache2  
apache2: /usr/sbin/apache2 /usr/lib/apache2 /etc/apache2 

> ubuntu@ip-172-31-1-200:/etc/apache2/sites-available$ sudo rm projectlamp.conf

> ubuntu@ip-172-31-1-200:/var/www/projectlamp$ cd ..  
ubuntu@ip-172-31-1-200:/var/www$ sudo rm -r projectlamp/

`After removing the files, it still was not working (As at this time of report, I think it could be that I still needed to disable it from sites-enabled).`

> `Anyway, I had to uninstall Apache completely from the server:`  
ubuntu@ip-172-31-1-200:/var/www$ sudo apt-get purge apache2 apache2-utils  apache2-bin apache2.2-common  
Reading package lists... Done

> ubuntu@ip-172-31-1-200:/var/www$ sudo systemctl restart nginx
ubuntu@ip-172-31-1-200:/var/www$

> I refreshed the site and was able to see that my Nginx was up and running. 

![Nginx default webpage](https://user-images.githubusercontent.com/70076627/114897682-60271380-9e09-11eb-849c-4e4b0c3fb28d.PNG)



<br/>

## 2. Installing MySQL
>Even though I had installed mysql in Project 1, I still ran the command.

> *ubuntu@ip-172-31-1-200:/var/www$ sudo apt install mysql-server  
Reading package lists... Done  
Building dependency tree  
Reading state information... Done  
mysql-server is already the newest version (8.0.23-0ubuntu0.20.04.1).  
The following packages were automatically installed and are no longer required:  
  apache2-data libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libjansson4 liblua5.2-0*

<br/>

> I thought of leaving the "mysql secure_installation" step out but I still ran it to see if I will get same prompts as before, I discovered that there were slight differences. One of the differences was that, it skipped asking me to choose from the 3 levels of password validation policy.

> *ubuntu@ip-172-31-1-200:/var/www$ sudo mysql_secure_installation
New password:  
Re-enter new password:  
Estimated strength of the password: 25
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y  
...  
...  
...*  

> ubuntu@ip-172-31-1-200:~$ sudo mysql  
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.23-0ubuntu0.20.04.1 (Ubuntu)  
Copyright (c) 2000, 2021, Oracle and/or its affiliates.      
Oracle is a registered trademark of Oracle Corporation and/or its affiliates. Other names may be trademarks of their respective owners.  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

> mysql>  
mysql> exit  
Bye


<br/>

## 3. Installing PHP
<br/>

> I installed php and additional component related to Nginx and MySQL.

>*ubuntu@ip-172-31-1-200:~$ sudo apt install php-fpm php-mysql  
Reading package lists... Done  
Building dependency tree  
...*

<br/>

## 4. Configuring Nginx to Use PHP Processor

<br/>

This is similar to creating virtual host in Apache setup.
>*ubuntu@ip-172-31-1-200:~$ sudo mkdir /var/www/projectLEMP
ubuntu@ip-172-31-1-200:~$  
ubuntu@ip-172-31-1-200:~$ sudo chown -R $USER:$USER /var/www/projectLEMP*

>I created the projectLEMP folder under the sites-available directory.<br/>  
ubuntu@ip-172-31-1-200:~$ cat /etc/nginx/sites-available/projectLEMP    

```
#/etc/nginx/sites-available/projectLEMP   
server {   
    listen 80;  
    server_name projectLEMP www.projectLEMP;  
    root /var/www/projectLEMP;  
index index.html index.htm index.php;  
location / {  
        try_files $uri $uri/ =404;  
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }
}
```
`I read through the explanation of what each directive above means, I will have to go over them a couple of times again for them to stick.`

> I then activated my configuration as below:<br/><br/>
ubuntu@ip-172-31-1-200:~$ sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

> I checked my configuration:  
*ubuntu@ip-172-31-1-200:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
nginx: configuration file /etc/nginx/nginx.conf test is successful*

> I disabled the default link:   
*ubuntu@ip-172-31-1-200:~$ sudo unlink /etc/nginx/sites-enabled/default*

> ubuntu@ip-172-31-1-200:~$ sudo systemctl reload nginx

> I created a new index.html file and pasted info I want displayed on the site inside it.<br/>  
*ubuntu@ip-172-31-1-200:~$ sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html*

> I accessed the page and it displayed correctly:

![LEMP site_1](https://user-images.githubusercontent.com/70076627/114904985-59e86580-9e10-11eb-8a85-464320f336b8.PNG)

<br/>

## 5. Testing PHP with Nginx

> I created another file - php file, inside the projectLEMP directory.  
ubuntu@ip-172-31-1-200:~$ sudo nano /var/www/projectLEMP/info.php

> ubuntu@ip-172-31-1-200:/var/www/projectLEMP$ ls -l
total 8  
-rw-rw-r-- 1 ubuntu ubuntu 106 Apr 14 17:18 index.html  
-rw-r--r-- 1 root   root    17 Apr 14 21:00 info.php

> I renamed the index.html file as it takes precedence over php file by default.  
ubuntu@ip-172-31-1-200:/var/www/projectLEMP$ mv index.html indextemp.html

> I accessed the page and it displayed correctly:

![LEMP Php site](https://user-images.githubusercontent.com/70076627/114907062-9917b600-9e12-11eb-955f-0993dc47c2e1.PNG)

<br/>


## 6. Retrieving data from MySQL database with PHP 

`I created a test database and user:`

> *ubuntu@ip-172-31-1-200:~$ sudo mysql*

> *mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password1';  
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements*

> I had to strenghten the password before it accepted it.

> *mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password1@F';
Query OK, 0 rows affected (0.01 sec)*

> mysql>  
*mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';  
Query OK, 0 rows affected (0.01 sec)*

> *mysql> exit*  
Bye

<br/>

> *ubuntu@ip-172-31-1-200:~$ mysql -u example_user -p  
Enter password:  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 13  
Server version: 8.0.23-0ubuntu0.20.04.1 (Ubuntu)*

> I displayed the content of the database:

> mysql> SHOW DATABASES;  
+--------------------+  
| Database   &ensp; &ensp;    &ensp; &ensp;      |  
+--------------------+  
| example_database   |  
| information_schema |  
+--------------------+  
2 rows in set (0.03 sec)

<br/>

> I created a new database with additional table.<br/>  
*mysql> CREATE TABLE example_database.todo_list (  
    -> item_id INT AUTO_INCREMENT,  
    -> content VARCHAR(255),  
    -> PRIMARY KEY(item_id)  
    -> );  
Query OK, 0 rows affected (0.11 sec)*

> *mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");  
Query OK, 1 row affected (0.02 sec)*

> To confirm that the data was saved successfully, I ran the below to display info stored in the DB:

> *mysql> SELECT * FROM example_database.todo_list;*    
+---------+----------------------------+  
| item_id&ensp; &ensp; | content &ensp; &ensp;&ensp; &ensp;&ensp; &ensp;&ensp; &ensp; &ensp; &ensp;             |  
+---------+-----------------------------+  
|&ensp; &ensp;   1 &ensp; &ensp;&ensp; &ensp; | My first important item |  
+---------+-----------------------------+  
 
1 row in set (0.00 sec)*

`I added more entries:`

> mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");  
Query OK, 1 row affected (0.00 sec)<br/>  
mysql> INSERT INTO example_database.todo_list (content) VALUES ("what to do next");  
Query OK, 1 row affected (0.00 sec)<br/>   
mysql> INSERT INTO example_database.todo_list (content) VALUES ("next lesson to take");  
Query OK, 1 row affected (0.00 sec)

<br/>

*mysql>  SELECT * FROM example_database.todo_list;  
+---------+-------------------------+  
| item_id &ensp; | content &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp;  |  
+---------+-------------------------+  
|       1 &ensp; &ensp; &ensp; &ensp; &ensp;| My first important item |  
|       2 &ensp; &ensp; &ensp; &ensp; &ensp;| My first important item |  
|       3 &ensp; &ensp; &ensp; &ensp; &ensp;| what to do next  &ensp; &ensp; &ensp; &ensp;       |  
|       4 &ensp; &ensp; &ensp; &ensp; &ensp;| next lesson to take  &ensp; &ensp;    |  
+---------+-------------------------+  
4 rows in set (0.00 sec)*

<br/>

To connect my PHP script to my database for queries:

> I created a new php file under the projectLEMP directory as below and pasted into it the below output:

> *ubuntu@ip-172-31-1-200:~$ cat /var/www/projectLEMP/todo_list.php*
```
<?php
$user = "example_user";
$password = "password1@F";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

> *ubuntu@ip-172-31-1-200:/var/www/projectLEMP$ ls -l  
total 12  
-rw-rw-r-- 1 ubuntu ubuntu 106 Apr 14 17:18 indextemp.html  
-rw-r--r-- 1 root   root    17 Apr 14 21:00 info.php  
-rw-r--r-- 1 root   root   441 Apr 14 22:15 todo_list.php*  

> *ubuntu@ip-172-31-1-200:~$ curl -I http://54.214.124.145/todo_list.php  
HTTP/1.1 200 OK  
Server: nginx/1.18.0 (Ubuntu)  
Date: Thu, 15 Apr 2021 17:55:27 GMT  
Content-Type: text/html; charset=UTF-8  
Connection: keep-alive*

I refreshed the page and got the below result:

![php mysql result](https://user-images.githubusercontent.com/70076627/114918865-7d66dc80-9e1f-11eb-9f1c-068286b266c2.PNG)

<br/>


#### `I am getting more familiar with the procedures involved in setting up LAMP and LEMP stacks!`  

