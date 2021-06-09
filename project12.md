## Ansible Refactoring & Static Assignments (Imports and Roles)

### **`Lessons:`**
> This project has solidified my knowledge of Ansible, I have further learnt about the need to refactor Ansible code, create assignments, and how to use the imports functionality (Imports allow to effectively re-use previously created playbooks in a new playbook - it allows you to organize your tasks and reuse them when needed).

<br/>

## Step 1 — Jenkins job enhancement
> Here we want to enhance the way codes are deployed to a separate directory (before now we have to change the directory we use when the ansible command is run after every change. This method of modifying directory each time a change is pushed is inconvenient and space consuming.

> One of the solutions to this, is to introduce a new Jenkins Job that will require `Copy Artifacts` plugin.

1. On Jenkins-Ansible server I created a new directory called ansible-config-artifact - `we will store there all artifacts after each build`.

> *root@ip-172-31-24-124:/home/ubuntu# mkdir /home/ubuntu/ansible-config-artifact  
root@ip-172-31-24-124:/home/ubuntu# chmod -R 0777 /home/ubuntu/ansible-config-artifact*


2. From Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab, searched for `Copy Artifact` and installed the plugin without restarting Jenkins.


![copy artifact plugin](https://user-images.githubusercontent.com/70076627/120923990-6038e680-c6c9-11eb-9b70-c49bce03230d.PNG)

3. I created a new Freestyle project and named it `save_artifacts`.
4. I configured the number of builds (3) to keep in order to save space on the server, for example, one might want to keep only last 2 or 5 build results (This change can also be made to the ansible job).
This project will be triggered by completion of the existing ansible project.

5. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, I created a `Build step` and chose `Copy artifacts from other project`, specified `ansible` (previous Job/Project) as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory.

![copy artifacts setup_1](https://user-images.githubusercontent.com/70076627/120923998-69c24e80-c6c9-11eb-9a26-2a92de8d34d8.PNG)

![copy artifacts setup_2](https://user-images.githubusercontent.com/70076627/120924009-73e44d00-c6c9-11eb-8b9a-1acd36d8f393.PNG)

![copy artifacts setup_3](https://user-images.githubusercontent.com/70076627/120924013-7cd51e80-c6c9-11eb-859d-bae0fc9e258b.PNG)

6. I tested my set up by making some changes in README.MD file inside my ansible-config-mgt repository (right inside master branch).
One of the Jenkins jobs (previous ansible project) was completed but my `save_artifacts` kept on failing. 

**`ERROR ENCOUNTERED`**:
![ERROR- Access denied to ansible_config_artifact (to be used)](https://user-images.githubusercontent.com/70076627/120925316-fd971900-c6cf-11eb-8f5c-a06e6eb1f3fa.PNG)

> I searched for a solution to the above error for a while, after trying a couple of things without any of them working, I saw a suggestion to use another directory other than the /home/ubuntu. Reason was that Jenkins may not be able to make changes to this default home directory.
I tried this by changing the directory to `tmp/` and ran the build again, it worked, so I continued using this /tmp folder for my '`copy artifacts`'.

> *ubuntu@ip-172-31-24-124:~$ cd /tmp  
ubuntu@ip-172-31-24-124:/tmp$ mkdir ansible-config-artifact_1  
ubuntu@ip-172-31-24-124:/tmp$ chmod -R 777 /tmp/ansible-config-artifact_1*

> After the above change and update on Jenkins, both Jenkins Jobs were successful.  
I could also see my files inside `/tmp/ansible-config-artifact_1 directory` and it got updated with every commit to my master branch.

> *ubuntu@ip-172-31-24-124:/tmp/ansible-config-artifact_1$ ls   
README.md  inventory  playbooks*

![copy artifacts setup_Successful copy](https://user-images.githubusercontent.com/70076627/120925401-5e265600-c6d0-11eb-876f-848e0ab5ec24.PNG)

<br/>

## Step 2 — Refactor Ansible code by importing other playbooks into site.yml
> I first ensured to pull the latest code from my master branch to a new branch before I started with the code refactoring.
```
Refactoring is essential in organizing your codes and playbooks in an environment where you have several servers with different requirements, OS , code changes, troubleshooting. Refactoring is simply a technique used to reduce complexity, enhance code readability, increase maintainability and extensibility and add proper comments without affecting the logic.
 It also means making changes to the source code without changing expected behaviour of the software.
```
> In previous project, I wrote all tasks (simple) in a single playbook common.yml (except for other tests scripts I had to use to run practice playbook). Now, in a situation where we need to handle complex tasks, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them. 

**Code re-use case (Importing other playbooks):**

1. Within playbooks folder, I created a new file and named it `site.yml` - This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. 

2. Created a new folder in root of the repository and named it `static-assignments`. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. 

3. I then moved `common.yml` file into the newly created `static-assignments` folder.

4. Inside site.yml file, I imported common.yml playbook.
`The screenshot of the above implementation is shown below:`

![site yaml setup script](https://user-images.githubusercontent.com/70076627/120933893-131e3a00-c6f4-11eb-9c97-ac33454a8e1e.PNG)

<br/>

- Run ansible-playbook command against the dev environment
Here, I will apply some tasks to my dev servers, as wireshark is already installed on them, another playbook called `common-del.yml` will be created under `static-assignments` to delete the wireshark utility on these servers.

> I updated the site.yml with - `import_playbook: ../static-assignments/common-del.yml`.

![common-del yaml del wireshark - define in site yaml](https://user-images.githubusercontent.com/70076627/120933957-5678a880-c6f4-11eb-86d2-c1cc641f3798.PNG)

![common-del yaml del wireshark](https://user-images.githubusercontent.com/70076627/120933962-5aa4c600-c6f4-11eb-9a7d-3a160e14cc37.PNG)

> I commited and pushed to github, it was successful as seen below:

![Successful trigger - Jenkins-github for common-del wireshark 19](https://user-images.githubusercontent.com/70076627/120934010-83c55680-c6f4-11eb-9671-3fc9f234ca74.PNG)

**Also successfully copied to our new artifacts diectory:**  
> *root@ip-172-31-24-124:/tmp/ansible-config-artifact_1# ls -l*
```
total 16
-rw-r--r-- 1 jenkins jenkins  180 Jun  5 17:45 README.md
drwxr-xr-x 2 jenkins jenkins 4096 Jun  5 16:38 inventory
drwxr-xr-x 2 jenkins jenkins 4096 Jun  5 18:30 playbooks
drwxr-xr-x 2 jenkins jenkins 4096 Jun  5 18:30 static-assignments
```

**I then ran it against dev servers:**
> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev.yml /tmp/ansible-config-artifact_1/playbooks/site.yml*
```
PLAY [all] ****************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.24.217]
ok: [172.31.19.10]
ok: [172.31.21.72]
ok: [172.31.17.4]
ok: [172.31.28.70]

PLAY [update web, nfs] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.19.10]
ok: [172.31.24.217]
ok: [172.31.21.72]
ok: [172.31.17.4]

TASK [delete wireshark] ***************************************************************************************************************************
fatal: [172.31.17.4]: FAILED! => {"ansible_facts": {"pkg_mgr": "apt"}, "changed": false, "msg": "value of state must be one of: absent, build-dep, fixed, latest, present, got: removed"}
changed: [172.31.24.217]
changed: [172.31.21.72]
changed: [172.31.19.10]

PLAY [update LB server and db servers] ************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.28.70]

TASK [delete wireshark] ***************************************************************************************************************************
changed: [172.31.28.70]

PLAY RECAP ****************************************************************************************************************************************
172.31.17.4                : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.19.10               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.72               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.24.217              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> *After troubleshooting the above failed task, I realized that there was an error for that defined host in the /static-assignment/common.yml file, it was placed in a wrong line, it was successful after the configuration was done correctly.*
```
My DB is an ubuntu server and this was previously done ==> 
hosts: webservers, nfs, db
  remote_user: ec2-user
----------------------

I modified as below:
name: update web and nfs
  hosts: webservers, nfs
  remote_user: ec2-user

 name: update LB server and db servers
  hosts: lb, db
remote_user: ubuntu
```

**SUCCESSFUL RUN:**
```
ubuntu@ip-172-31-24-124:~$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev.yml /tmp/ansible-config-artifact_1/playbooks/site.yml

PLAY [all] ****************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.24.217]
ok: [172.31.21.72]
ok: [172.31.19.10]
ok: [172.31.17.4]
ok: [172.31.28.70]

PLAY [update web, nfs] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.19.10]
ok: [172.31.24.217]
ok: [172.31.21.72]

TASK [delete wireshark] ***************************************************************************************************************************
ok: [172.31.21.72]
ok: [172.31.19.10]
ok: [172.31.24.217]

PLAY [update LB server and db servers] ************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.28.70]
ok: [172.31.17.4]

TASK [delete wireshark] ***************************************************************************************************************************
ok: [172.31.28.70]
changed: [172.31.17.4]

PLAY RECAP ****************************************************************************************************************************************
172.31.17.4                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.19.10               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.72               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.24.217              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> Below shows output from some of the servers with wireshark uninstalled:

`nfs server`
> [ec2-user@ip-172-31-21-72 ~]$ wireshark --version   
-bash: wireshark: command not found   
[ec2-user@ip-172-31-21-72 ~]$ which wireshark   
/usr/bin/which: no wireshark in (/home/ec2-user/.local/bin:/home/ec2-user/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)

`DB Server`
> ubuntu@ip-172-31-17-4:~$ which wireshark  
ubuntu@ip-172-31-17-4:~$ wireshark --version   
-bash: /usr/bin/wireshark: No such file or directory


`Such a great learning on how to use import_playbooks module.` 

<br/>

## Step 3 — Configure UAT Webservers with a role ‘Webserver’

> Configured two (2) new Web servers (RHEL 8 servers) as `uat`.

> I updated my inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of my 2 new UAT Web servers.

![uat yaml for new webservers](https://user-images.githubusercontent.com/70076627/120936407-73ff3f80-c6ff-11eb-98b7-d4532dcbc0e0.PNG)

> I created a directory called roles/ relative to the playbook file. 

> I created the file structure using Ansible utility called `ansible-galaxy` as shown below.

**Note:** Unnecessary directories and files like tests, files, and vars can be removed.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git pull*
```
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 4 (delta 3), reused 4 (delta 3), pack-reused 0
Unpacking objects: 100% (4/4), 340 bytes | 340.00 KiB/s, done.
From https://github.com/Romok1/ansible-config-mgt
   75794c9..d6887dd  master     -> origin/master
Updating 75794c9..d6887dd
Fast-forward
 inventory/uat.yml | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

`I already created the role directory from VS code.`

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ cd roles/  
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ls*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ansible-galaxy init webserver*
- Role webserver was created successfully

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ls -l  
total 4    
drwxrwxr-x 10 ubuntu ubuntu 4096 Jun  6 00:00 webserver*



- In `/etc/ansible/ansible.cfg` file, I uncommented roles_path string and provided a full path to my roles directory (roles_path = /home/ubuntu/ansible-config-mgt/roles), so Ansible could know where to find configured roles.

> *ubuntu@ip-172-31-24-124:~$ sudo vi /etc/ansible/ansible.cfg*
```
# additional paths to search for roles in, colon separated
#roles_path    = /etc/ansible/role
roles_path    = /home/ubuntu/ansible-config-mgt/roles
```
> Now we start adding some logic to the `webserver role`. Inside `tasks` directory, and within the `main.yml` file, I added some configuration tasks as shown below in the screenshot.

`The tasks will implement the following:`
- Install and configure Apache (httpd service).
- Clone Tooling website from GitHub https://github.com/your-name>/tooling.git.
- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
- Make sure httpd service is started

![roles_webserver_tasks_main yaml](https://user-images.githubusercontent.com/70076627/120936632-b8d7a600-c700-11eb-8707-648e5b6548af.PNG)

<br/>

## Step 4 — Reference ‘Webserver’ role

> Within the static-assignments folder, I created a new assignment for uat-webservers `uat-webservers.yml`. This is where I will reference the role (role webserver).

**Summary:**
> So the tasks have been created under role/webserver ---> tasks ---> main.yml.  
> Then under static-assignments directory, a new file called uat-webservers.yml (like the common.yml created before) is added, and it points to tasks/main.yml.
> Lastly, our entry point to our ansible configuration which is `site.yml` file then points to or refers the uat-webservers.yml role.

![step 4 - ref webserver roles_tasks_configs screenshot](https://user-images.githubusercontent.com/70076627/120936850-115b7300-c702-11eb-889c-7735018cb93f.PNG)

## Step 5 — Commit & Test

> I committed my changes, created a Pull Request and merged them to master branch. 

>Webhook triggered two consequent Jenkins jobs and they ran successfully and copied all the files to my Jenkins-Ansible server into /tmp/ansible-config-artifact_1/ directory.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ cd /tmp/ansible-config-artifact_1/    
ubuntu@ip-172-31-24-124:/tmp/ansible-config-artifact_1$ ls -l*
```
total 20
-rw-r--r-- 1 jenkins jenkins  161 Jun  6 00:15 README.md
drwxr-xr-x 2 jenkins jenkins 4096 Jun  5 16:38 inventory
drwxr-xr-x 2 jenkins jenkins 4096 Jun  5 18:30 playbooks
drwxr-xr-x 3 jenkins jenkins 4096 Jun  6 00:15 roles
drwxr-xr-x 2 jenkins jenkins 4096 Jun  6 01:19 static-assignments
```
> *ubuntu@ip-172-31-24-124:/tmp/ansible-config-artifact_1$ cd playbooks/  
ubuntu@ip-172-31-24-124:/tmp/ansible-config-artifact_1/playbooks$ ls -l*
```
total 16
-rw-r--r-- 1 jenkins jenkins  450 Jun  1 22:47 common.yml
-rw-r--r-- 1 jenkins jenkins  228 Jun  2 23:41 lbdb.yml
-rw-r--r-- 1 jenkins jenkins  157 Jun  6 01:19 site.yml
-rw-r--r-- 1 jenkins jenkins 1012 Jun  3 01:06 siteupdate.yml
```

> *ubuntu@ip-172-31-24-124:/tmp/ansible-config-artifact_1/playbooks$ cat site.yml* 
```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

**Test my connection:**
> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible webserver -m ping*
```
172.31.20.209 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.31.21.102 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

**I ran the playbook against my uat inventory, below is the result:**

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /tmp/ansible-config-artifact_1/playbooks/site.yml*
```
PLAY [all] ****************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.20.209]
ok: [172.31.21.102]
[WARNING]: Could not match supplied host pattern, ignoring: webservers
[WARNING]: Could not match supplied host pattern, ignoring: nfs

PLAY [update web, nfs] ****************************************************************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: lb
[WARNING]: Could not match supplied host pattern, ignoring: db

PLAY [update LB server and db servers] ************************************************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: uat-webservers

PLAY [uat-webservers] *****************************************************************************************************************************
skipping: no hosts matched

PLAY [uat-webservers] *****************************************************************************************************************************
skipping: no hosts matched

PLAY RECAP ****************************************************************************************************************************************
172.31.20.209              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.102              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**To fix the above errors:**

> 1. I modified my `site.yml` to only include assignment for the `uat-webservers`.
```
site.yml
---
- hosts: webserver
- import_playbook: ../static-assignments/uat-webservers.yml
```

> 2. I also modified the **host part** of the `site.yml` and the `uat-webserver.yml` to read `webserver` as this is what was used in our **inventory/uat.yml**.

`After the above modification, everything worked perfectly:`
> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/inventory$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /tmp/ansible-config-artifact_1/playbooks/site.yml*
```
PLAY [webserver] **********************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.21.102]
ok: [172.31.20.209]

PLAY [webserver] **********************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.21.102]
ok: [172.31.20.209]

TASK [webserver : install apache] *****************************************************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

TASK [webserver : install git] ********************************************************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

TASK [webserver : clone a repo] *******************************************************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

TASK [webserver : copy html content to one level up] **********************************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

TASK [webserver : Start service httpd, if not started] ********************************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

TASK [webserver : recursively remove /var/www/html/html/ directory] *******************************************************************************
changed: [172.31.20.209]
changed: [172.31.21.102]

PLAY RECAP ****************************************************************************************************************************************
172.31.20.209              : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.102              : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**My UAT Web servers are now configured and I can reach them from my browser:**

http://Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php  
http://34.220.110.177/index.php

![Webserver1 index homepage](https://user-images.githubusercontent.com/70076627/120937128-998e4800-c703-11eb-9a2c-d6c9207ccb0e.PNG)

http://Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php  
http://35.161.215.43/index.php

![Webserver2 index homepage](https://user-images.githubusercontent.com/70076627/120937138-a743cd80-c703-11eb-9165-5835d1d8e928.PNG)

`Glad to have learnt how to deploy and configure UAT Web Servers using Ansible imports and roles!. So much knowledge packed into one project. The outcome was like magic, great work.`

