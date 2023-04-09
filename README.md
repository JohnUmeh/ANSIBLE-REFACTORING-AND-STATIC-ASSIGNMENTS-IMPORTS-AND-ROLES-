# ANSIBLE-REFACTORING-AND-STATIC-ASSIGNMENTS(IMPORTS AND ROLES)

In this project, we will contiue working with the [ansible-config-mgt](https://github.com/JohnUmeh/ansible-config-mgt) and improve on it. We will:

1. Refactor the codes

2. Create Assignments

3. Learn about the import function - which allows us to effectively re-use previously created playbooks in a new playbook

**Code Refactoring**
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.
In this project, we will move codes around but the infrastructure of the code will remain the same.

**Step 1 – Jenkins job enhancement**
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.

Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.

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




















