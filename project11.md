## Ansible Configuration Management - Automate Project 7 to 10

### **`Lessons:`**

>How to use Devops tools like Ansible to automate different types of manual operations and routine tasks. I became more used to writing codes using declarative language like yaml.

<br/>

The required tasks:
- Install and configure Ansible client to act as a Jump Server/Bastion Host.
- Create a simple Ansible playbook to automate servers configuration.

## Step 1 — Install and configure Ansible on EC2 Instance

> I will be reusing my Jenkins server I set up in the previous project to run playbooks.

> I created a new repository in my GitHub account and named it `ansible-config-mgt`.

![Capture create git repo](https://user-images.githubusercontent.com/70076627/120245059-f0a79f00-c263-11eb-8945-b32e5d0a60ac.PNG)


- Install Ansible:
> *root@ip-172-31-24-124:/home/ubuntu# sudo apt update  
root@ip-172-31-24-124:/home/ubuntu# apt install ansible*

- I checked the installed ansible version:
> *root@ip-172-31-24-124:/home/ubuntu# ansible --version*
```
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.5 (default, Jan 27 2021, 15:41:15) [GCC 9.3.0]
```

1. Configured Jenkins build job to save my repository content every time I change it 

- Created a new Freestyle project named `ansible` in Jenkins and pointed it to my ‘ansible-config-mgt’ repository.
- Configured Webhook in GitHub and set webhook to trigger `ansible` build.
- Configured a Post-build job to save all `(**)` files.

> I tested my setup by making some changes in README.MD file in master branch and I checked to ensure builds starts automatically and Jenkins saves the files (build artifacts) in this folder ---> /var/lib/jenkins/jobs/ansible/builds/build_number>/archive/.

> **I encountered an error for my first trigger build.**

**ERROR**: Couldn't find any revision to build. Verify the repository and branch configuration for this job.

![no revision to build error - jenkins freestyle webhok](https://user-images.githubusercontent.com/70076627/120941169-298abc80-c719-11eb-8395-e9e4127fd04f.PNG)


> **SOLUTION:** This was corrected by modifying the `branch name` on Github.  I noticed that Github has changed default branch name from `master` to `main`. So I had to create `master` branch and make it `default` as that was what I have configured on Jenkins.

> After the above modification, my build triggers worked perfectly.

![github-jenkins webhook trigger_1](https://user-images.githubusercontent.com/70076627/120941318-d82efd00-c719-11eb-85fa-15f49b82caae.PNG)

![github-jenkins webhook trigger cons log_2](https://user-images.githubusercontent.com/70076627/120941323-dd8c4780-c719-11eb-8be7-a2479c12fd29.PNG)

<br/>

## Step 2 — Prepare your development environment using Visual Studio Code

> As I already have Visual Studio Code installed on my computer, I only needed to connect to my new Github repo by cloning my repository.

> I downloaded `Remote Development extension pack` plugin and configured my .ssh configuration file on my local computer. Then via SSH targets, I connected to remote hosts (In this case my Jenkins-Ansible server).

<br/>

## Step 3 — Begin Ansible Development

> In my `ansible-config-mgt` GitHub repository, I created a new branch that will be used for development of a new feature.

> I did checkout for the newly created feature branch to my local machine and started building my code and directory structure.

> *ubuntu@ip-172-31-24-124:~$ git clone https://github.com/Romok1/ansible-config-mgt.git*

> *ubuntu@ip-172-31-24-124:~$ cd ansible-config-mgt/  
ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls    
README.md*

> I had created the `devfeature` branch on github and created the directories using VS code.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git checkout devfeature     
Branch 'devfeature' set up to track remote branch 'devfeature' from 'origin'.  
Switched to a new branch 'devfeature'*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls  
README.md  inventory  playbooks*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git status  
On branch devfeature  
Your branch is up to date with 'origin/devfeature'.*

`Ansible dev steps:`
Created a directory and named it playbooks - it will be used to store all my playbook files.
Created a directory and named it inventory - it will be used to keep my hosts organised.
Within the playbooks folder, I created my first playbook, and named it common.yml.
Within the inventory folder, I created an inventory file (.yml) for each environment (Development, Staging, Testing and Production) dev, staging, uat, and prod respectively.

![setting up ansible development environment_1 github](https://user-images.githubusercontent.com/70076627/120993198-3809c000-c77b-11eb-9205-746118a1665a.PNG)

![setting up ansible development environment_1 VS code](https://user-images.githubusercontent.com/70076627/120993175-32ac7580-c77b-11eb-9684-d8a0c8aeb077.PNG)

<br/>

## Step 4 — Set up an Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. 

- To ssh into target servers from Jenkins-Ansible host - need to copy my private (.pem) key to my server. 

> I first copied my private (.pem) key into my server  
Then changed permissions to my private key  
After, I imported my key into ssh-agent

> *ubuntu@ip-172-31-24-124:~$ vi dareproject1.pem  
ubuntu@ip-172-31-24-124:~$ chmod 400 dareproject1.pem  
ubuntu@ip-172-31-24-124:~$ eval `ssh-agent -s`*

> *ubuntu@ip-172-31-24-124:~$ ssh-add /home/ubuntu/dareproject1.pem  
Identity added: /home/ubuntu/dareproject1.pem (/home/ubuntu/dareproject1.pem)*


- In other to start configuring my development servers, I saved the below inventory structure in the inventory/dev.

> I updated my inventory/dev.yml file with the code below:
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

![inventory_dev script](https://user-images.githubusercontent.com/70076627/120995950-cda64f00-c77d-11eb-8291-15a926f17d51.PNG)

<br/>

## Step 5 — Create a Common Playbook
Here, we give Ansible the instructions on what we need to be performed on all servers listed in `inventory/dev` file.
We will be writing this script in `common.yml`.

This `playbook common.yml script` will contain the configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

**The common.yaml file was updated with the below code:**
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest
```

**This playbook is divided into two parts, each of them is intended to perform the same task:**
- install wireshark utility (or make sure it is updated to the latest version) on the RHEL 8 and Ubuntu servers. 
- It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

![playbooks common script](https://user-images.githubusercontent.com/70076627/120997849-74d7b600-c77f-11eb-88ee-37b44f2db6a6.PNG)

<br/>

> I played around with other configurations as shown below: (This will be talked about in more details at the end of this documentation - `SELF-LEARNING TASKS` section)

- Create a directory and a file inside it
- Change timezone on all servers
- Run some shell script


## Step 6 — Update GIT with the latest code
> This step involves the use of git to perform a couple of tasks for our repo (create branch, pull, push, merge, check status, add, commit...).

> I checked the status of my files.  
I issued a pull request.   
I checked out to my new feature branch and assessed the content.

> I merged the code to my master branch, by checking out of the feature branch and entered the master branch where I merged, commited and pushed the changes to github repo.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git branch*
  devfeature
  main
* master

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git checkout devfeature   
Branch 'devfeature' set up to track remote branch 'devfeature' from 'origin'.  
Switched to a new branch 'devfeature'*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls  
README.md  inventory  playbooks*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git status
```
On branch devfeature
Your branch is up to date with 'origin/devfeature'.

nothing to commit, working tree clean
```
`Note`-------I had previously connected my VSC directly to github, committed and pushed , hence nothing to commit at this point.


> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git pull  
Already up to date.*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls    
README.md  inventory  playbooks  

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git checkout master  
Branch 'master' set up to track remote branch 'master' from 'origin'.  
Switched to a new branch 'master'*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git merge devfeature  
Auto-merging README.md    
CONFLICT (content): Merge conflict in README.md    
Automatic merge failed; fix conflicts and then commit the result.*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git status*

```
On branch master
Your branch is up to date with 'origin/master'.
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:
        new file:   inventory/dev.yml
        new file:   inventory/prod.yml
        new file:   inventory/staging.yml
        new file:   inventory/uat.yml
        new file:   playbooks/common.yml

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   README.md
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git add .*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git commit -m "from devfeature branch"*  
```
[master 1b14da0] from devfeature branch
 Committer: Ubuntu <ubuntu@ip-172-31-24-124.us-west-2.compute.internal>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls      
README.md  inventory  playbooks*

**My playbook directory structure:**

![ALL ansible script from branch master github](https://user-images.githubusercontent.com/70076627/120999692-4eb31580-c781-11eb-803c-375d259ced55.PNG)

> As soon as I pushed my code change into the master branch, Jenkins was triggered to create a build and save all the files (build artifacts).

> I also accessed it from inside the server (Jenkins-Ansible server) on the `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory.

> *ubuntu@ip-172-31-24-124:/var/lib/jenkins/jobs/ansible/builds$ ls -l*
```
total 24
drwxr-xr-x 2 jenkins jenkins 4096 Jun  1 02:53 1
drwxr-xr-x 2 jenkins jenkins 4096 Jun  1 03:03 2
drwxr-xr-x 3 jenkins jenkins 4096 Jun  1 03:07 3
drwxr-xr-x 3 jenkins jenkins 4096 Jun  1 03:08 4
drwxr-xr-x 3 jenkins jenkins 4096 Jun  1 22:47 5
-rw-r--r-- 1 jenkins jenkins    0 Jun  1 02:42 legacyIds
```

![build trigger 5 -update master from devfeature](https://user-images.githubusercontent.com/70076627/121001037-c6357480-c782-11eb-8950-446f647e6651.PNG)

![build trigger 5 -update master from devfeature _console log](https://user-images.githubusercontent.com/70076627/121001053-cc2b5580-c782-11eb-831c-a5b07f383329.PNG)

I added some other scripts (including tasks required to implement `SELF-LEARNING TASKS`), saved and pushed to master.

![jenkins shot updates to script](https://user-images.githubusercontent.com/70076627/121001683-71462e00-c783-11eb-8b01-6c6039e44f26.PNG)

![jenkins shot updates to script - FINAL](https://user-images.githubusercontent.com/70076627/121001704-76a37880-c783-11eb-94bf-540928cab4d4.PNG)

## Step 7 — Run first Ansible test
In this step, we will execute our playbook using the `ansible-playbook` command and verify it.

> I made the below change in ansible.cfg to point our inventory to the path we are using.

> *root@ip-172-31-24-124:/home/ubuntu/ansible-config-mgt/inventory# vi /etc/ansible/ansible.cfg*
```
[defaults]

# some basic default values...

inventory      = /home/ubuntu/ansible-config-mgt/inventory/
#library        = /usr/share/my_modules/
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ **ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/5/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/5/archive/playbooks/common.yml***

```
PLAY [update web and nfs] *******************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
The authenticity of host '172.31.19.10 (172.31.19.10)' can't be established.
ECDSA key fingerprint is SHA256:A+OKH/GLlvA9NiA4wPOF9haKQXYmmI31ktq6FwGvJrU.
The authenticity of host '172.31.24.217 (172.31.24.217)' can't be established.
ECDSA key fingerprint is SHA256:BrcLEMR3Ncrangf6wn6q3VXama8epgXI1prQYyjOadM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? fatal: [172.31.21.72]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ec2-user@172.31.21.72: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}
```

```
root@ip-172-31-24-124:/home/ubuntu/ansible-config-mgt/inventory#  ansible webservers -a "ping"
The authenticity of host '172.31.24.217 (172.31.24.217)' can't be established.
ECDSA key fingerprint is SHA256:BrcLEMR3Ncrangf6wn6q3VXama8epgXI1prQYyjOadM.
The authenticity of host '172.31.19.10 (172.31.19.10)' can't be established.
ECDSA key fingerprint is SHA256:A+OKH/GLlvA9NiA4wPOF9haKQXYmmI31ktq6FwGvJrU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
172.31.24.217 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Warning: Permanently added '172.31.24.217' (ECDSA) to the list of known hosts.\r\nec2-user@172.31.24.217: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).",
    "unreachable": true
}


ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible webservers -m ping
The authenticity of host '172.31.19.10 (172.31.19.10)' can't be established.
ECDSA key fingerprint is SHA256:A+OKH/GLlvA9NiA4wPOF9haKQXYmmI31ktq6FwGvJrU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 172.31.24.217 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}

172.31.19.10 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Host key verification failed.",
    "unreachable": true
}
```
### **PROBLEM 1**

**ERROR:** The playbook failed to execute because of permission denied error message (**Permission denied (publickey,gssapi-keyex,gssapi-with-mic)**.", "unreachable": true).

**SOLUTIONS:**
1. I ran the ssh-agent and ssh-add command again to keep the process running on the servers.

**DRAWBACK:** The terminal I was using to access these servers must remain open, once it's closed or I exit the session/log out, the process is terminated and each host becomes unreachable again (PERMISSION DENIED). `NOT AS EFFECTIVE` 

NFS SERVER
-------------
> I did this part all over again for the target hosts:  
[ec2-user@ip-172-31-21-72 ~]$ vi dareproject1.pem  
[ec2-user@ip-172-31-21-72 ~]$ ls  
dareproject1.pem  
[ec2-user@ip-172-31-21-72 ~]$ eval `ssh-agent -s`  
Agent pid 22361  

> [ec2-user@ip-172-31-21-72 ~]$ chmod 400 dareproject1.pem  

> [ec2-user@ip-172-31-21-72 ~]$ ssh-add dareproject1.pem  
Identity added: dareproject1.pem (dareproject1.pem)    


On ANSIBLE SERVER:
-------------------
I ran the below commands again: (This keeps the process running)
eval `ssh-agent -s`
ssh-add dareproject1.pem

<br/>

2. Run the command with --check -kK 

**DRAWBACK:** I needed to always enter password for SSH. `TOO MANUAL`

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/6/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/6/archive/playbooks/lbdb.yml --check -kK*
```
SSH password: 
BECOME password[defaults to SSH password]:  [ERROR]: User interrupted execution
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/6/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/6/archive/playbooks/lbdb.yml --check -kK
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [update LB server and db servers] ******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
fatal: [172.31.28.70]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [172.31.17.4]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
172.31.17.4                : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
```
*I was required to install sshpass the first time I tried it.*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ sudo apt install sshpass    
Reading package lists... Done*

<br/>

3. The use of the extra commands —user USERNAME —private-key=PATH_TO_PEM_FILE

**DRAWBACK:** Personally, I find it inconvenient or simply just prefer another method.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/6/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/6/archive/playbooks/lbdb.yml `—user ubuntu —private-key=/home/ubuntu/dareproject1.pem`*

`**On the other hand, this can be added in the the yaml file to automate login, this way it uses the PEM file by default to login as stated in your script.**`

<br/>

4. Setting of password and rsa public key between Ansible server and other hosts.

**SPECIFICALLY FOR THIS PROJECT, NO DRAWBACK:** This worked for me as I did not have to make changes to my script to accommodate PEM key file definition. Also, it does not request for password each time you run the ansible playbook (Except in setups where high security level is required, this works generally.). It is also a one time setup.

ON ALL REMOTE HOSTS
###################
> *[ec2-user@ip-172-31-19-10 ~]$ sudo vi /etc/ssh/sshd_config  
[ec2-user@ip-172-31-19-10 ~]$ sudo cat /etc/ssh/sshd_config*
```
# Authentication:

PubkeyAuthentication yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
#PermitEmptyPasswords no
#PasswordAuthentication no
```

> *[ec2-user@ip-172-31-19-10 ~]$ sudo passwd ec2-user  
Changing password for user ec2-user.  
New password:   
Retype new password:   
passwd: all authentication tokens updated successfully.*

> *[ec2-user@ip-172-31-19-10 ~]$ sudo systemctl restart sshd*

ON ANSIBLE SERVER 
#################
> *ubuntu@ip-172-31-24-124:~/.ssh$ sudo vi /etc/ssh/sshd_config  
PasswordAuthentication yes*

> *ubuntu@ip-172-31-24-124:~$  sudo systemctl restart sshd*

> *ubuntu@ip-172-31-24-124:~$ sudo passwd ubuntu  
New password:  
Retype new password:  
passwd: password updated successfully*


> *ubuntu@ip-172-31-24-124:~$ ssh-keygen -t rsa
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
...
...
```

> *ubuntu@ip-172-31-24-124:~$ ssh-copy-id ec2-user@172.31.19.10* (This copy command was run for all the other hosts also)
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ubuntu/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ec2-user@172.31.19.10's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ec2-user@172.31.19.10'"
and check to make sure that only the key(s) you wanted were added.
```


---------------------`After the above setup, I could ping all the other hosts without needing to manually define key file or entering passwords.`
```
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible lb -m ping
172.31.28.70 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
```

<br/>

### **PROBLEM 2**

**ERROR: I also encountered another error or more like one of the plays hanging - the execution does not proceed past Play 1**
Where ansible-playbook was refusing to run the second play in the script. I tried it several times but kept getting stuck at play 2.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/5/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/5/archive/playbooks/common.yml*

```
PLAY [update web and nfs] *******************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [172.31.24.217]
ok: [172.31.19.10]
ok: [172.31.21.72]

TASK [ensure wireshark is at the latest version] ********************************************************************************************************************************
changed: [172.31.19.10]
changed: [172.31.24.217]
changed: [172.31.21.72]

PLAY [update LB server and db servers] ******************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************** (<------ IT WAS STUCK HERE FOR A LONG TIME)

^C [ERROR]: User interrupted execution
```

**SOLUTION:** I added a divider (# - - - - - - - - - - - - - - - - - - - - - - - - - - - -)  <-----> **I saw this idea online while troubleshooting**
```
---
- name: update web and nfs
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- name: update LB server and db servers
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  ...
  ...
  ...
```

**After all the solutions were applied**, I successfully executed my tasks. (Please note: I separated the plays as seen below because as at this time, I had not found solution to `PROBLEM 2 - ERROR: Where ansible-playbook was refusing to run the second play in the script`). 

<br/>

However for my `self learning` task, I implemented this solution, so my play 1 amd play 2 ran together successfully.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/5/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/5/archive/playbooks/`common.yml`*

```
PLAY [update web and nfs] *******************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [172.31.24.217]
ok: [172.31.19.10]
ok: [172.31.21.72]

TASK [ensure wireshark is at the latest version] ********************************************************************************************************************************
changed: [172.31.19.10]
changed: [172.31.24.217]
changed: [172.31.21.72]

PLAY [update LB server and db servers] ******************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************

^C [ERROR]: User interrupted execution
```

**FOR MY LB abd DB in another script:**
> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/6/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/6/archive/playbooks/`lbdb.yml`*

```
PLAY [update LB server and db servers] ******************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [172.31.17.4]
ok: [172.31.28.70]

TASK [ensure wireshark is at the latest version] ********************************************************************************************************************************
changed: [172.31.28.70]
changed: [172.31.17.4]

PLAY RECAP **********************************************************************************************************************************************************************
172.31.17.4                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> I went ahead to check that wireshark was installed succesfully.  
Some of the outputs from the hosts:

- webservers1
```
[ec2-user@ip-172-31-24-217 ~]$ which wireshark
/usr/bin/wireshark
[ec2-user@ip-172-31-24-217 ~]$ wireshark --version
Wireshark 2.6.2 (v2.6.2)
```
- db
```
ubuntu@ip-172-31-17-4:~$ which wireshark
/usr/bin/wireshark
ubuntu@ip-172-31-17-4:~$ wireshark --version
Wireshark 3.2.3 (Git v3.2.3 packaged as 3.2.3-1)
```
- lb
```
ubuntu@ip-172-31-28-70:~$ which wireshark
/usr/bin/wireshark
ubuntu@ip-172-31-28-70:~$ wireshark --version
Wireshark 3.2.3 (Git v3.2.3 packaged as 3.2.3-1)
```

### **SELF-LEARNING TASKS**
So, I mentioned earlier that I tried other tasks and they were executed successfully.
- Create a directory and a file inside it
- Change timezone on all servers
- Run some shell script

I played around with more scenarios around the tasks highlighted above. Instead of using the `common.yml` created earlier, I created another file name `siteupdate.yml` where I put the script to execute all the above tasks.

![siteupdate yml](https://user-images.githubusercontent.com/70076627/121008536-01d43c80-c78b-11eb-90ca-39e7d0d7a1bb.PNG)


After pushing my changes to master and Jenkins did the build, I ran the ansible-playbook command against my new yaml file.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/13/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/13/archive/playbooks/`siteupdate.yml`*
```
PLAY [update web and nfs] *******************************************************************************************************************************************************

TASK [Creating a new directory] *************************************************************************************************************************************************
ok: [172.31.19.10]
ok: [172.31.24.217]
ok: [172.31.21.72]

TASK [set timezone to Africa/Lagos] *********************************************************************************************************************************************
ok: [172.31.19.10]
ok: [172.31.24.217]
ok: [172.31.21.72]

TASK [Transfer the script] ******************************************************************************************************************************************************
ok: [172.31.21.72]
ok: [172.31.24.217]
ok: [172.31.19.10]

TASK [Execute the script] *******************************************************************************************************************************************************
changed: [172.31.21.72]
changed: [172.31.19.10]
changed: [172.31.24.217]

PLAY [update LB server and db servers] ******************************************************************************************************************************************

TASK [Creating a new directory] *************************************************************************************************************************************************
changed: [172.31.28.70]
changed: [172.31.17.4]

TASK [set timezone to Africa/Lagos] *********************************************************************************************************************************************
changed: [172.31.28.70]
changed: [172.31.17.4]

TASK [Transfer the script] ******************************************************************************************************************************************************
changed: [172.31.17.4]
changed: [172.31.28.70]

TASK [Execute the script] *******************************************************************************************************************************************************
changed: [172.31.28.70]
changed: [172.31.17.4]

PLAY RECAP **********************************************************************************************************************************************************************
172.31.17.4                : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.19.10               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.72               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.24.217              : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

`This project was information packed and tasking, I gained so much.`