# ANSIBLE-REFACTORING-AND-STATIC-ASSIGNMENTS(IMPORTS AND ROLES)

In this project, we will contiue working with the [ansible-config-mgt](https://github.com/JohnUmeh/ansible-config-mgt) and improve on it. We will:

1. Refactor the codes

2. Create Assignments

3. Learn about the import function - which allows us to effectively re-use previously created playbooks in a new playbook

**Code Refactoring**
  Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of     the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity,     add proper comments without affecting the logic.
  In this project, we will move codes around but the infrastructure of the code will remain the same.

**Step 1 – Jenkins job enhancement**
  Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not   very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change.   Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.

  Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each   build.

  `sudo mkdir /home/ubuntu/ansible-config-artifact`

![newdir](https://user-images.githubusercontent.com/77943759/230754559-d869db71-39af-4fbe-8350-b3d4608711e3.png)

Change permissions to this directory, so Jenkins could save files there – 

`chmod -R 0777 /home/ubuntu/ansible-config-artifact`

Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![copy_artifact_install](https://user-images.githubusercontent.com/77943759/230754660-dbbccf04-bdec-49b7-9624-90f42ad28956.png)

Create a new Freestyle project (this have been done in [Project 9](https://github.com/JohnUmeh/Tooling-Website-Deployment-Automation-with-continous-Integration)) and name it save_artifacts

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

![buildsettings](https://user-images.githubusercontent.com/77943759/230754957-08be623b-2376-43d5-ba0e-6ae330a3e950.png)

Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

![ansible,](https://user-images.githubusercontent.com/77943759/230754986-a05acc20-ecc6-4ccc-9b6b-e6abe071135b.png)

The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![artifacttocopy](https://user-images.githubusercontent.com/77943759/230755043-4cbad02d-c7e2-4636-859a-4152d77c7aae.png)

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch)

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.


## **REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML**

**Step 2 – Refactor Ansible code by importing other playbooks into site.yml**

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch

create a new branch, name it refactor

In [Project 11](https://github.com/JohnUmeh/Tooling-Website-Deployment-Automation-with-continous-Integration) I wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine we have many more tasks and  need to apply this playbook to other servers with different requirements. In this case, we will have to read through the whole playbook to check if all tasks written there are applicable and if is there anything that we need to add for certain server/OS families. Very fast it will become a tedious exercise and our playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

1. Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire        infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to        all other playbooks that will be developed. Including common.yml that we created previously.

2. Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children      playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can        choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

3. Move common.yml file into the newly created static-assignments folder.

4. Inside site.yml file, import common.yml playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
Our folder structure should look like this;

```
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```

5. Run ansible-playbook command against the dev environment

Since we need to apply some tasks to our dev servers and wireshark is already installed – You can go ahead and create another playbook under static-assignments repository and name it common-del.yml. In this playbook, configure deletion of wireshark utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```


update site.yml with:

`- import_playbook: ../static-assignments/common-del.yml`

Instead of common.yml

![commondel](https://user-images.githubusercontent.com/77943759/230971359-ac5ecbf9-1a3b-4b15-a4f1-8b04bad8a2b4.png)


Run it against dev servers

```
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```
Confirm that wireshark is deleted on all the servers by running:

`wireshark --version`

## **CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’**

**Step 3 – Configure UAT Webservers with a role ‘Webserver’**

We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-    UAT.
2. Create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory. Do that in any of these two ways:

-Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)

```
mkdir roles
cd roles
ansible-galaxy init webserver
```
-Create the directory/files structure manually

The entire folder structure should look like:

```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

After removing unnecessary directories and files, the roles structure should look like this

```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```
3. Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers

NOTE: Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance use this as guide;

For Windows users – ssh-agent [on windows](https://www.youtube.com/watch?v=OplGrY74qog&feature=youtu.be)
For Linux users – ssh-agent on [linux](https://www.youtube.com/watch?v=OplGrY74qog&feature=youtu.be)

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```
![uatupdate](https://user-images.githubusercontent.com/77943759/230972206-7db51083-52fe-488b-ab5b-37a02486024d.png)


4.  In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory , so Ansible could know where to find configured roles

`roles_path    =  /home/ubuntu/ansible-config-mgt/roles`

![roles](https://user-images.githubusercontent.com/77943759/230972278-e3681837-282e-4927-be7a-a7a3aba53c67.png)

It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started
  
Paste this in your main.yml file:
  
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
![webservertask](https://user-images.githubusercontent.com/77943759/230972699-0715f950-a610-454d-90df-f4af3c4fb57d.png)
  
## **REFERENCE WEBSERVER ROLE**

**Step 4 – Reference ‘Webserver’ role**
  
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role
  
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
![uatserversyml](https://user-images.githubusercontent.com/77943759/230972859-2d404264-cd79-40ce-a7c5-b5a4bfd51b14.png)

  
We need to refer your uat-webservers.yml role inside site.yml since the entry point to our ansible configuration is site.yml.

Paste this in site.yml:
  
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```
 ![uatserversyml](https://user-images.githubusercontent.com/77943759/230973116-310ebbda-ad22-4201-b8da-5cf2f386637b.png)
  
**Step 5 – Commit & Test**
  
Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.
  
Now run the playbook against your uat inventory and see what happens:
  
`sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml`
  
You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

`http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`

or

`http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`
  
![changedfiles](https://user-images.githubusercontent.com/77943759/230973306-7dd83ac8-af57-4112-a183-5023394d9a65.png)

![jenkinsfinal](https://user-images.githubusercontent.com/77943759/230973430-2c497330-f472-4a8b-988d-a04dc106969e.png)

  
This is the new architechture for the project

![project12_architecture](https://user-images.githubusercontent.com/77943759/230969274-455024d1-13fe-4aec-afa3-715729f652e1.png)


