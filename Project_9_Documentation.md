# **Project_9_Continuous-Integration-Pipeline-for-Tooling-Website**
___
____
### **Step 1 - Install & Configure Jenkins server**
___
>#### Launch 1 EC2 instance with Ubuntu 20.04 Operating System and name it Jenkins  Open TCP port 8080 

![ec2](./Project_9_Images/ec2.PNG)
>#### Install JDK (since Jenkins is a Java-based application) uisng the following commands;
* *`sudo apt update`*
* *`sudo apt install default-jdk-headless`*

![apt](./Project_9_Images/apt-install-jdk.PNG)
>#### Install Jenkins:

* *`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`*
* *`sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \ /etc/apt/sources.list.d/jenkins.list'`*
* *`sudo apt update`*
* *`sudo apt-get install jenkins`*
* *`sudo systemctl status jenkins`*

![jenkins](./Project_9_Images/apt-install-wget.PNG)
![jenkins](./Project_9_Images/apt-install-jenkins.PNG)
![jenkins](./Project_9_Images/apt-sysctl.PNG)
![8080](./Project_9_Images/8080.PNG)

###  **Configure Jenkins server**

>#### Accessing the Jenkins Server 
The Server is access from Web Browser with url *`http://Jenkins-Server-Public-IP-Address-or-Public-DNS-Name:8080`* which is *`http://18.134.10.151:8080`*

![url](./Project_9_Images/unlock%20jenkins.PNG)
![url](./Project_9_Images/jenkins%20getting%20stsrted.PNG)
![url](./Project_9_Images/Jenkins%20ready.PNG)
___
### **Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks**
___
Here I will configure a simple Jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.
>#### Enable webhooks 
in your GitHub repository settings enable the webhooks settings.
![webhooks](./Project_9_Images/webhook.PNG)
>#### Create Jenkins Job
* Go to Jenkins web console, click New Item and create a Freestyle project, Press OK at the bottom.

![free](./Project_9_Images/jenkinf%20frestyle.PNG)
 * In the Source Code management Tab I select Git and put the URL of tooling repository https://github.com/OlusegunMichael/tooling.git Click Save

![source](./Project_9_Images/sourcecode.PNG)

* "Build Now" button is clicked, after all configuration is done correctly, the build will success will be displayed under #1

![build](./Project_9_Images/jenkins%20build.PNG)
![console](./Project_9_Images/console%20output.PNG)
![console](./Project_9_Images/console%20output1.PNG)

Configuring  the job to trigger whenever there is a change in the sourcecode in Github's Repository

* On In Build Triggers Tab  GitHub hook trigger for GITScm polling box is checked
* On Post-build Actions Tab  archive all the artifacts with **

![Triggers](./Project_9_Images/Build%20trigger.PNG)
![Triggers](./Project_9_Images/archive%20artifats.PNG)

To confirm the Build Trigger on the job whenever there is a change in the sourcecode in Github's Repository I  make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

![test](./Project_9_Images/Testing%20Jenking.PNG)
![test](./Project_9_Images/Testing%20Jenking1.PNG)
![test](./Project_9_Images/Testing%20Jenking2.PNG)

By default, the artifacts are stored on Jenkins server locally *`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`*
![local](./Project_9_Images/Local%20dir%20for%20artifats.PNG)

___
### **Step 3 - Configure Jenkins to copy files to NFS server via SSH**
___
Currently, the artifacts is stored in Jenkins but in order to implement a Jenkins CI it needs to be in the NFS Server to serve data to the Web Servers - in doing so, any changes made in the (GitHub) would automatically update the Web Servers via the NFS Server.

* To copy the artifacts is stored locally in Jenkins to the NFS server *`/mnt/apps`* directory first is to; Install a plugin called *`Publish Over SSH`*
* On the main dashboard click *`Manage Jenkins`* and select *`Manage Plugins`*

![plugins](./Project_9_Images/Manage%20plugins.PNG)
* Click on Available and search for *Publish Over SSH* select and install without restart

![plugins](./Project_9_Images/publish%20over%20ssh.PNG)
![publish](./Project_9_Images/Plugin%20isntalled.PNG)
![publish](./Project_9_Images/Plugin%20isntalled1.PNG)

Configure the job to copy artifacts over to NFS server

* On main dashboard Manage Jenkins tab is selected and choose Configure System menu item.
* Scrolling down to Publish over SSH plugin configuration section and it is configured to be able to connect to your NFS server by;
* Providing a private key (content of .pem file that you use to connect to NFS server via SSH).
* Click on SSH Servers to expand and file the required details, Test configuration and save.
* Hostname – Set to be private IP address of the NFS server
* Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
* Remote directory – /mnt/apps
* Test the configuration to ensure the connection returns Success and save.

![configure](./Project_9_Images/Configure%20System.PNG)
![configure](./Project_9_Images/jenkins_NFS%20_Success.PNG)
Configure Post Build Action

* On Jenkins job click on configure and select  Post-build Action tab.
* Add post-build action is clicked and Send build artifacts over SSH Configure is selected, to send all ( denoted by ** ) files probuced by the build into our previouslys define remote directory and configuration is saved.

![PostBuild](./Project_9_Images/post%20build.PNG)
![postbuild](./Project_9_Images/post%20buil%20failed.PNG)

The build failed due to no access the the specified directory so the NFS server instance was Logged onto to change the permission to allow Jenkins to save files in the /mnt directory.

![permission](./Project_9_Images/permission.PNG)
The README File on GitHib was Edited again to Trigger a another Build.
![Build](./Project_9_Images/Artifats%20to%20NSF%20Success.PNG)

## Project End