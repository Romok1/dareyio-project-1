## Ansible Dynamic Assignments (Include) and Community Roles

### **`Lessons:`**
> Learnt about dynamic assigments by using `include module`. In previous project, I majorly worked with static assignments by using the `import module`/functionality.

**In summary:**
```
import = Static
include = Dynamic

import: All statements are pre-processed at the time playbooks are parsed. Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered.
```
```
include: All statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.
```
<br/>

## Step 1 — Introducing Dynamic Assignment Into Our structure

> In my https://github.com/xxxxx/ansible-config-mgt GitHub repository, I started a new branch and called it `dynamic-assignments`.

> I created a new folder under this branch, named it `dynamic-assignments`. 

> Then inside this folder, I created a new file and named it `env-vars.yml`. (Note: We will instruct `site.yml` to include this playbook later).

**My Github now has the below structure:**

![dynamic assignment new branch and folder env-vars yaml](https://user-images.githubusercontent.com/70076627/121094762-09c1ca00-c7e7-11eb-9206-4a4cf928c8e9.PNG)

> We will require a way to set values to variables per specific environment. This is because, in configuring multiple environments, we have to consider that each of these environments will have certain unique attributes such as servername, ip-address etc.

- In this setup, I created a folder to keep each environment's variables file.

> I created a new folder called `env-vars`, which will have files that will be used to set variables for each environment.

> I pasted the below script inside my `env-vars.yml` under `dynamic-assignments` folder.
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
   - name: looping through list of available files
     include_vars: "{{ item }}"
     with_first_found:
       - files:  
          - uat.yml       
          - prod.yml        
          - dev.yml       
         paths:      
          - "{{ playbook_dir }}/../env-vars"      
     tags: 
       - always
```

![env var folder and env var file declare script](https://user-images.githubusercontent.com/70076627/121097895-ac307c00-c7ec-11eb-8d65-1f41f6f1c529.PNG)

I also used another format for the env-vars.yaml as shown in the script (env-vars2.yaml) below:
 ```
---
- name: collate variables from env specific file, if it exists
  include_vars: "{{ item }}"
  with_first_found:
    - "{{ playbook_dir }}/../env_vars/{{ inventory_file }}.yml"
    - "{{ playbook_dir }}/../env_vars/default.yml"
  tags:
    - always
```
 ![env double files](https://user-images.githubusercontent.com/70076627/121932510-dd550300-cd3c-11eb-9474-cf1ab702014d.PNG)


**Quick notes pertaining to the env-vars yaml script above:**
- It can be seen that include_vars syntax was used instead of include which as become deprecated. Variants of include_* like include_role, include_tasks, include_vars must be used.

`Hence, variants of include_* must be used. They include:`

- Similarly, for the same version of Ansible, variants of import such as import_role, import_tasks were also introduced.

- Special variables used in the script:
`{{ playbook_dir }}` will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem.

- `{{ inventory_file }}` will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

- The loop used `with_first_found` implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

- I found some other special variables on this site --> https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html 

<br/>

## Step 2 — Update site.yml with dynamic assignments

In this step, I will update `site.yml` file to make use of the dynamic assignment. Script used:
```
---
- name: Include dynamic variables 
  hosts: all

- name: import common file
  import_playbook: ../static-assignments/common.yml 
  tags:
    - always

- name: include env-vars file
  include_playbook: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- name: Webserver assignment
  hosts: webservers
    - import_playbook: ../static-assignments/webservers.yml
```

![update sit yaml with env var](https://user-images.githubusercontent.com/70076627/121103381-bf951480-c7f7-11eb-8778-f6ed229fa496.PNG)

**Observation/Issue encountered:** So I ran the playbook at this point and I observed that include_playbook `include_playbook: ../dynamic-assignments/env-vars.yml` was not working, it was throwing errors. Meanwhile, `import_playbook` worked.

```
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/dynamic-assignments$ ansible-playbook siteenvvar.yml
ERROR! 'include_playbook' is not a valid attribute for a Play

The error appears to be in '/home/ubuntu/ansible-config-mgt/dynamic-assignments/siteenvvar.yml': line 3, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

- name: include env-vars file
- include_playbook: ../dynamic-assignments/env-vars.yml
  ^ here
```

> For experiment sake, I converted the env-var yaml file from a `playbook` to a `task` and updated my `site.yml` with `include_task` definition (`include_tasks: ../dynamic-assignments/env-vars.yml`), and it worked.

```
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/dynamic-assignments$ vi siteenvvar.yml
---
- name: include env-vars file
  include_task: ../dynamic-assignments/env-vars.yml
  tags:
    - always
"siteenvvar.yml" 5L, 110C written                                                                                      

ubuntu@ip-172-31-24-124:~/ansible-config-mgt/dynamic-assignments$ ansible-playbook siteenvvar.yml

PLAY [collate variables from env specific file, if it exists] *************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.16.78]
ok: [172.31.21.102]
ok: [172.31.24.124]

TASK [looping through list of available files] ****************************************************************************************************
ok: [172.31.16.78] => (item=/home/ubuntu/ansible-config-mgt/env-vars/prod.yml)
ok: [172.31.21.102] => (item=/home/ubuntu/ansible-config-mgt/env-vars/prod.yml)
ok: [172.31.24.124] => (item=/home/ubuntu/ansible-config-mgt/env-vars/prod.yml)

PLAY RECAP ****************************************************************************************************************************************
172.31.16.78               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0      
172.31.21.102              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.24.124              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**In light of the above, I had to make use of `import_playbook` in place of include_plabook in my `site.yml` for subsequent playbook run.**

<br/>

## Community Roles
> Here, I created a role for MySQL database.
> This will install MySQL package, create a database and configure users.
> This will be achieved with the help of `Ansible Galaxy` (simply download a ready to use ansible role).

<br/>

## Step 3 — Download Mysql Ansible Role
> The plan is to use MySQL role developed by geerlingguy.

> On my Jenkins-Ansible server, where I already have my ‘ansible-config-mgt’ repo cloned, I went into the ansible-config-mgt directory and ran the below commands:

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git branch roles-feature*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git switch roles-feature  
Switched to branch 'roles-feature'*

> Inside `roles` directory, I created my new MySQL role with `ansible-galaxy install geerlingguy.mysql` and renamed the folder to `mysql`.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ansible-galaxy install geerlingguy.mysql*
```
- downloading role 'mysql', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-mysql/archive/3.3.1.tar.gz
- extracting geerlingguy.mysql to /home/ubuntu/ansible-config-mgt/roles/geerlingguy.mysql
- geerlingguy.mysql (3.3.1) was installed successfully
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ git switch roles-feature    
Switched to branch 'roles-feature'    
ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ls    
README.md  inventory  playbooks  roles  static-assignments*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ cd roles/      
ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ls      
geerlingguy.mysql  webserver*


> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ mv geerlingguy.mysql/ mysql*


> I read the `README.md` file, and edited `roles` configuration to use correct credentials for MySQL required for the tooling website.

![mysql role defaults main yml](https://user-images.githubusercontent.com/70076627/121948983-db486f80-cd4f-11eb-93fd-46f1ad94e6fc.PNG)

> I then uploaded the changes into my GitHub:

> I went through the codes again, after that I created a Pull Request and merged it to master branch on GitHub.

<br/>

## Load Balancer roles
> We want to be able to choose which Load Balancer (Nginx or Apache) to use.

> I found some available roles from the community and downloaded them.

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ansible-galaxy install nginxinc.nginx*
```
- downloading role 'nginx', owned by nginxinc
- downloading role from https://github.com/nginxinc/ansible-role-nginx/archive/0.20.0.tar.gz
- extracting nginxinc.nginx to /home/ubuntu/ansible-config-mgt/roles/nginxinc.nginx
- nginxinc.nginx (0.20.0) was installed successfully
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt$ ansible-galaxy install geerlingguy.apache*
```
- downloading role 'apache', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-apache/archive/3.1.4.tar.gz
- extracting geerlingguy.apache to /home/ubuntu/ansible-config-mgt/roles/geerlingguy.apache
- geerlingguy.apache (3.1.4) was installed successfully
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ls      
geerlingguy.apache  mysql  nginxinc.nginx  webserver*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ mv geerlingguy.apache/ apache*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ mv nginxinc.nginx nginx*



> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ ls    
apache  mysql  nginx  webserver*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ git status*
```
On branch dynamic-assignments
Your branch is up to date with 'origin/dynamic-assignments'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        apache/
        nginx/

nothing added to commit but untracked files present (use "git add" to track)
```

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ git add .*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ git commit -m "addedd role apache and nginx"  
[dynamic-assignments 014e071] addedd role apache and nginx*

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/roles$ git push*

> I updated both static-assignment and site.yml files to refer the roles.
 - Since both Nginx and Apache load balancer can not be used at the same time, we need to add a condition to enable either one - `Variables` will be used for this.

> Under static-assignment folder, I created lb.yml file and added to it the script below:
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

> While I updated the site.yml file with the below script:
```
 - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/lb.yml
        when: load_balancer_is_required 
```

> Next, I entered the below script in the `env-vars/uat.yml` file that I created earlier, the script is the environment setting that is used to activate the load balancer to use  (Nginx or Apache).
```
enable_nginx_lb: true
load_balancer_is_required: true
```
> When I need to use `Apache`, I can switch one to true and the other to false.

![env var folder - uat yaml file](https://user-images.githubusercontent.com/70076627/121950797-003de200-cd52-11eb-9540-b47d40ccd624.PNG)

**My complete site.yml script looks like this:**

![finall site yaml](https://user-images.githubusercontent.com/70076627/121950977-3c714280-cd52-11eb-817a-28f2c75dd748.PNG)

> I also updated my inventory to have all the servers details required.

<br/>

> I saved and pushed all my changes to github.

> I ran the ansible-playbook command against my environment.

- `For this first run, I encountered some failures. This is because I came up with two env-vars files which have different scripts (I was trying to test various scripts). One of the them caused a failure and the play ceased, ansible-playbook aborted the play after the failure (It did not continue processing other playbooks configured in the site.yml).

- So, the cause of the failure was due to `"No file was found when using first_found"`. To make the playbook continue, one needs to add Use `errors='ignore'` to the playbook script. This will allow the task to be skipped if no files are found, then move on to process others. 

```
TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.19.10]
ok: [172.31.24.217]
ok: [172.31.21.72]
ok: [172.31.17.4]
ok: [172.31.28.70]
ok: [172.31.20.209]
ok: [172.31.16.78]

TASK [collate variables from env specific file, if it exists] *************************************************************************************
fatal: [172.31.21.72]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.24.217]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.19.10]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.17.4]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.28.70]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.16.78]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
fatal: [172.31.20.209]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}

PLAY [webserver] **********************************************************************************************************************************

PLAY [dbmysql] ************************************************************************************************************************************

PLAY [lb] *****************************************************************************************************************************************

PLAY RECAP ****************************************************************************************************************************************
172.31.16.78               : ok=4    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.17.4                : ok=6    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.19.10               : ok=6    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.20.209              : ok=4    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.21.102              : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.21.72               : ok=6    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.24.217              : ok=6    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=4    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored
```

**For the second run, my db and loadbalancer playbooks ran successfully.**

> *ubuntu@ip-172-31-24-124:~/ansible-config-mgt/playbooks$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev /home/ubuntu/ansible-config-mgt/playbooks/site.yml*

```
TASK [webserver : install apache] *****************************************************************************************************************
ok: [172.31.20.209]

TASK [webserver : install git] ********************************************************************************************************************
ok: [172.31.20.209]

TASK [webserver : clone a repo] *******************************************************************************************************************
changed: [172.31.20.209]

TASK [webserver : copy html content to one level up] **********************************************************************************************
changed: [172.31.20.209]

TASK [webserver : Start service httpd, if not started] ********************************************************************************************
changed: [172.31.20.209]

TASK [webserver : recursively remove /var/www/html/html/ directory] *******************************************************************************
changed: [172.31.20.209]

PLAY [dbmysql] ************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/variables.yml for 172.31.16.78

TASK [mysql : Include OS-specific variables.] *****************************************************************************************************
ok: [172.31.16.78] => (item=/home/ubuntu/ansible-config-mgt/roles/mysql/vars/Debian.yml)

TASK [mysql : Define mysql_packages.] *************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_daemon.] ***************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_slow_query_log_file.] **************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_log_error.] ************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_syslog_tag.] ***********************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_pid_file.] *************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_config_file.] **********************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_config_include_dir.] ***************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_socket.] ***************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Define mysql_supports_innodb_large_prefix.] *****************************************************************************************
ok: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/setup-Debian.yml for 172.31.16.78

TASK [mysql : Check if MySQL is already installed.] ***********************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Update apt cache if MySQL is not yet installed.] ************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Ensure MySQL Python libraries are installed.] ***************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Ensure MySQL packages are installed.] ***********************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Ensure MySQL is stopped after initial install.] *************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Delete innodb log files created by apt package after initial install.] **************************************************************
skipping: [172.31.16.78] => (item=ib_logfile0) 
skipping: [172.31.16.78] => (item=ib_logfile1) 

TASK [mysql : include_tasks] **********************************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Check if MySQL packages were installed.] ********************************************************************************************
ok: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/configure.yml for 172.31.16.78

TASK [mysql : Get MySQL version.] *****************************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Copy my.cnf global MySQL configuration.] ********************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Verify mysql include directory exists.] *********************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Copy my.cnf override files into include directory.] *********************************************************************************

TASK [mysql : Create slow query log file (if configured).] ****************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Create datadir if it does not exist] ************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Set ownership on slow query log file (if configured).] ******************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Create error log file (if configured).] *********************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Set ownership on error log file (if configured).] ***********************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Ensure MySQL is started and enabled on boot.] ***************************************************************************************
ok: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/secure-installation.yml for 172.31.16.78

TASK [mysql : Ensure default user is present.] ****************************************************************************************************
[WARNING]: Module did not set no_log for update_********
changed: [172.31.16.78]

TASK [mysql : Copy user-my.cnf file with password credentials.] ***********************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Disallow root login remotely] *******************************************************************************************************
ok: [172.31.16.78] => (item=DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1'))

TASK [mysql : Get list of hosts for the root user.] ***********************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Update MySQL root password for localhost root account (5.7.x).] *********************************************************************

TASK [mysql : Update MySQL root password for localhost root account (< 5.7.x).] *******************************************************************

TASK [mysql : Copy .my.cnf file with root password credentials.] **********************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Get list of hosts for the anonymous user.] ******************************************************************************************
ok: [172.31.16.78]

TASK [mysql : Remove anonymous MySQL users.] ******************************************************************************************************

TASK [mysql : Remove MySQL test database.] ********************************************************************************************************
ok: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/databases.yml for 172.31.16.78

TASK [mysql : Ensure MySQL databases are present.] ************************************************************************************************

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/users.yml for 172.31.16.78

TASK [mysql : Ensure MySQL users are present.] ****************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : include_tasks] **********************************************************************************************************************
included: /home/ubuntu/ansible-config-mgt/roles/mysql/tasks/replication.yml for 172.31.16.78

TASK [mysql : Ensure replication user exists on master.] ******************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Check slave replication status.] ****************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Check master replication status.] ***************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Configure replication on the slave.] ************************************************************************************************
skipping: [172.31.16.78]

TASK [mysql : Start replication.] *****************************************************************************************************************
skipping: [172.31.16.78]

PLAY [lb] *****************************************************************************************************************************************

PLAY RECAP ****************************************************************************************************************************************
172.31.16.78               : ok=34   changed=1    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
172.31.17.4                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.19.10               : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.20.209              : ok=10   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.21.102              : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
172.31.21.72               : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.24.217              : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.28.70               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> Some parts for mysql setup were skipped because they were not needed.

`I struggled a bit with the dynamic configuration part of Ansible, but I gained a lot on this project and will definitely find more courses I can take to solidify my understanding of it, I will also practice with all kinds of scenarios.`