## Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

### **`Lessons:`**

>Understanding of how to use Jenkins CI to automate the testing, building and deployment of all the changes (continuous commited code in small increments) made to our codes (for this Tooling Website deployment solution for example).

> Continuous Integration helps to automate testing and deployment processes, this leads to speedy release of software and web solutions and ensuring the quality of code that teams deploy.

`Solution:`
A Jenkins server will be added to the architecture prepared in Project 8.

<br/>

## Step 1: Install Jenkins server 
`Infra:` 
- I created an AWS EC2 server based on Ubuntu Server 20.04 LTS.
- Installed JDK (since Jenkins is a Java-based application).

> *root@ip-172-31-24-124:/home/ubuntu# apt update  
root@ip-172-31-24-124:/home/ubuntu# apt install default-jdk-headless*

- Install Jenkins

> *root@ip-172-31-24-124:/home/ubuntu# wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -*  
OK

> *root@ip-172-31-24-124:/home/ubuntu# sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'*

> *root@ip-172-31-24-124:/home/ubuntu# apt update  
root@ip-172-31-24-124:/home/ubuntu# apt-get install jenkins*

- Check if Jenkins is up and running: 

> *root@ip-172-31-24-124:/home/ubuntu#systemctl status jenkins 

`In my EC2 Security Group, I configured new Inbound rule with Type as Custom TCP and port as 8080` 

- Initial Jenkins setup  

> From my browser I accessed the Jenkins URL:

http://Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

> I entered the default adminpassword (retrieved from /var/lib/jenkins/secrets/initialAdminPassword on the Jenkins server).

> Next, I installed Plugins (I selected the option: `Install suggested plugins`)

> Lastly, I created an admin user, which I later used to login to the server.

![Jenkins after install home page](https://user-images.githubusercontent.com/70076627/119872997-f7f54280-bf1b-11eb-99ef-77f3ed716741.PNG)

<br/>

## Step 2: Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this step I configured a simple Jenkins job which gets triggered by GitHub webhooks and executes a build task to retreive codes from Github and store locally on Jenkins server.

> I enabled webhooks in my GitHub repository settings, screenshot below:

![webhook settings github](https://user-images.githubusercontent.com/70076627/119905586-78c83480-bf44-11eb-95a6-fd982d513f5c.PNG)

> Next, from Jenkins web console, I clicked on New Item and created a "Freestyle project".

![Capture tooling_github](https://user-images.githubusercontent.com/70076627/119905581-7665da80-bf44-11eb-8f47-8f4404fb92c0.PNG)

I saved the configuration and clicked on the "Build Now" button. The build was successful as shown in the console output below:

![Ist build success_manual](https://user-images.githubusercontent.com/70076627/119905781-c93f9200-bf44-11eb-9a08-dc1c59f995f4.PNG)

The above is triggered manually, next steps will show how to configure Jenkins to trigger the job from Github Webhook.

> I clicked "Configure" your job/project.

> Then configured triggering the job from Github webhook. 

![Add github hook trigger](https://user-images.githubusercontent.com/70076627/119907064-68fe1f80-bf47-11eb-8e81-5d2414c30d2e.PNG)

> Next, I configured "Post-build Actions" to archive all the files (files resulted from a build are called “artifacts”).

![archive file jenkins](https://user-images.githubusercontent.com/70076627/119907072-6d2a3d00-bf47-11eb-841b-5ad96577e07a.PNG)


 > I made some changes in the README.MD file in my GitHub repository and pushed the changes to the master branch.

> A new build was launched automatically (by webhook) and I could see its results - artifacts, saved on Jenkins server. I made changes a couple of times to trigger each build.

![2nd Jenkins webhook build auto](https://user-images.githubusercontent.com/70076627/119968292-3420b580-bfa5-11eb-9b76-b4ba2eceea80.PNG)

![3rd Jenkins webhook build auto with artifacts](https://user-images.githubusercontent.com/70076627/119968310-3a169680-bfa5-11eb-8718-969f232e8784.PNG)

 `The above shows an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub).`

 - By default, the artifacts are stored on Jenkins server locally

ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

> *root@ip-172-31-24-124:/var/lib/jenkins/jobs/tooling_github/builds/6# ls -l*
```
total 20
drwxr-xr-x 3 jenkins jenkins 4096 May 24 09:33 archive
-rw-r--r-- 1 jenkins jenkins 2191 May 24 09:33 build.xml
-rw-r--r-- 1 jenkins jenkins  421 May 24 09:33 changelog.xml
-rw-r--r-- 1 jenkins jenkins 1099 May 24 09:33 log
-rw-r--r-- 1 jenkins jenkins  682 May 24 09:33 polling.log
```


<br/>

## Step 3: Configure Jenkins to copy files to NFS server via SSH

The previous steps, my artifacts were saved locally on the Jenkins server, this step involves copying them to my NFS server (to /mnt/apps directory).

- I installed a Plugin that is called "Publish Over SSH".

I followed the below steps:
- "Manage Jenkins" ---> Manage Plugins ---> On Available tab, I searched for "Publish Over SSH" plugin---> "Install without restart"

- Configured the job/project to copy artifacts over to NFS server.

- I clicked on Manage Jenkins and chose "Configure System", then I scrolled down to "Publish over SSH plugin" configuration section and configured it with the below properties (screenshots can also be found below).

`The below properties were added:`  
-- A private key (content of .pem file that I used to connect to NFS server via SSH/Putty).  
-- Arbitrary name.   
-- Hostname - I used the private IP address of my NFS server
-- Username - ec2-user (since NFS server is based on EC2 with RHEL 8).  
-- Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server.

> I applied and saved the configuration.

![publish over ssh setup on jenkins](https://user-images.githubusercontent.com/70076627/120024326-e88efb80-bfe6-11eb-8d97-b2e717c502e4.PNG)

![publish over ssh setup on jenkins_1](https://user-images.githubusercontent.com/70076627/120024362-f6dd1780-bfe6-11eb-8446-bb58d5a413ae.PNG)

- I accessed my Jenkins Job configuration page and added a "Post-build Action". Here, I selected "Send build artificats over SSH".

![send build artifacts over ssh](https://user-images.githubusercontent.com/70076627/120024384-03fa0680-bfe7-11eb-81e1-466264e8236a.PNG)

-- For my source files, I used the pattern "**" to copy all the files and directories.

`Verification`  
I triggered a build by making changes to the README file. Then I encountered an error as shown below:

```
SSH: cd [/mnt/apps]
SSH: OK
SSH: mkdir [Images]
SSH: FAILED: Message [Permission denied]
SSH: mkdir [Images]
SSH: FAILED: Message [Permission denied]
SSH: Disconnecting configuration [ec2-user@172.31.21.72] ...
ERROR: Exception when publishing, exception message [Could not create or change to directory. Directory [Images]]
Build step 'Send build artifacts over SSH' changed build result to UNSTABLE
Finished: UNSTABLE
```

`The below step resolved it:`

- So I went back to my `NFS server` to make the below changes.

> *[ec2-user@ip-172-31-21-72 ~]$ sudo su  
[root@ip-172-31-21-72 ec2-user]# ll /mnt/apps  
total 0*  
```
drwxr-xr-x. 2 root root   6 Jun 15  2020 cgi-bin
drwxr-xr-x. 3 root root 205 May 18 01:07 html
```

> *[root@ip-172-31-21-72 ec2-user]# sudo chown -R ec2-user:ec2-user /mnt/apps  
[root@ip-172-31-21-72 ec2-user]# ll /mnt/apps  
total 0*
```
drwxr-xr-x. 2 ec2-user ec2-user   6 Jun 15  2020 cgi-bin
drwxr-xr-x. 3 ec2-user ec2-user 205 May 18 01:07 html
```

### I tested the setup again a couple of times after the above step and they were successful.

![Build 9 STATUS with nfs archiving](https://user-images.githubusercontent.com/70076627/120024556-4f141980-bfe7-11eb-8f2c-b38296f34a54.PNG)

![Build 9 CONSOLE LOG with nfs archiving](https://user-images.githubusercontent.com/70076627/120024631-65ba7080-bfe7-11eb-8e12-f54c50345a1a.PNG)

<br/>

### `ON MY NFS SERVER:`  
I checked the directory to see if it has been populated.

> *[root@ip-172-31-21-72 ec2-user]# cd /mnt/apps/  
[root@ip-172-31-21-72 apps]# ls -l  
total 28*  
```
-rw-rw-r--. 1 ec2-user ec2-user  332 May 24 22:35 apache-config.conf
drwxr-xr-x. 2 ec2-user ec2-user    6 Jun 15  2020 cgi-bin
-rw-rw-r--. 1 ec2-user ec2-user  313 May 24 22:35 Dockerfile
drwxr-xr-x. 3 ec2-user ec2-user  205 May 18 01:07 html
-rw-rw-r--. 1 ec2-user ec2-user 4202 May 24 22:35 Jenkinsfile
-rw-rw-r--. 1 ec2-user ec2-user 2381 May 24 22:35 README.md
-rw-rw-r--. 1 ec2-user ec2-user  163 May 24 22:35 start-apache
-rw-rw-r--. 1 ec2-user ec2-user 1674 May 24 22:35 tooling-db.sql
```

`I found this Continous Integration solution using Jenkins CI very interesting. I look forward to doing more complex Jenkins projects.`
