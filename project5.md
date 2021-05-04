## Client-Server Architecture with MySQL

### **`I learnt the following:`**

>Good understanding of the client-server architecture.

>I setup two Linux servers running Mysql-client/server, connected the two servers for proper communication.

<br/>

## TASK - Implement a Client Server Architecture using MySQL Database Management System (DBMS).

`Created and configured two Linux-based virtual servers (EC2 instances in AWS).`

**On mysql client Linux Server I installed MySQL Client software.**

> *ubuntu@ip-172-31-2-55:~$ sudo su*  
> *root@ip-172-31-2-55:/home/ubuntu# apt update*

> *root@ip-172-31-2-55:/home/ubuntu# apt-get install mysql-client  
Reading package lists... Done  
Building dependency tree         
Reading state information... Done  
The following additional packages will be installed:  
  mysql-client-8.0 mysql-client-core-8.0 mysql-common* <br/>

  <br/>

**On mysql server Linux Server I installed MySQL Server software.**

> *root@ip-172-31-0-149:/home/ubuntu# sudo apt update  
root@ip-172-31-0-149:/home/ubuntu# sudo apt-get install mysql-server  
root@ip-172-31-0-149:/home/ubuntu# mysql_secure_installation*   
```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
Please set the password for root here.

New password: 

Re-enter new password: 
Sorry, passwords do not match.

New password: 

Re-enter new password: 

Estimated strength of the password: 50 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```
> *root@ip-172-31-0-149:/home/ubuntu# mysql  
mysql>* 


`Because MySQL server uses TCP port 3306 by default, so I opened it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups (EC2 instance running on AWS). `

**I configured MySQL server to allow connections from remote hosts.**
> sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf   
Changed ‘127.0.0.1’ to ‘0.0.0.0’ 

> sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf   
#localhost which is more compatible and is not less secure.  
bind-address            = 127.0.0.1   (new: 0.0.0.0)  
mysqlx-bind-address     = 127.0.0.1  


**On my MySql Server:**  
`I created a user and configured password for it, this will be used to access the server from the client side:`
> *mysql> CREATE USER 'temilord'@'%' IDENTIFIED WITH mysql_native_password   BY 'password3';  
Query OK, 0 rows affected (0.00 sec)*  

> *mysql> GRANT ALL PRIVILEGES ON newdatabase.* TO 'temilord'@'%';  
Query OK, 0 rows affected (0.00 sec)


**I also created test database:**  
> mysql> CREATE DATABASE testdb;  
Query OK, 1 row affected (0.01 sec)  
<br/>
mysql> CREATE DATABASE newdatabase;  
Query OK, 1 row affected (0.01 sec)

> mysql> SHOW DATABASES;  
+--------------------+  
| Database    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;      |  
+--------------------+  
| information_schema &nbsp;|  
| mysql  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;          |   
| newdatabase &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;|  
| performance_schema |  
| sys   &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|  
| testdb&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;         |  
+--------------------+  
6 rows in set (0.00 sec)

<br/>

`From mysql client Linux Server I connected remotely to mysql server Database Engine with `**mysql utility**` to perform the action.`

> *root@ip-172-31-2-55:/home/ubuntu# mysql -u temilord -h 172.31.0.149 -p*
```
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.23-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
mysql>   
mysql> SHOW DATABASES;  
+--------------------+  
| Database &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |  
+--------------------+  
| information_schema |  
| newdatabase &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       |  
+--------------------+  
2 rows in set (0.00 sec)


`I have successfully connected to a remote MySQL server and can perform SQL queries:
Show databases;`

