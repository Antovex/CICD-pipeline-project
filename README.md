# CICD Pipeline Project

This project demonstrates how to create a continuous integration and continuous delivery (CICD) pipeline using four servers: a web server, a developer server, a Jenkins server, and an Ansible server.

## Project Overview

The main goal of this project is to automate the deployment of a website using a CICD pipeline. The website is hosted on a web server that runs httpd. The website code is stored in a GitHub repository that is updated by the developer. The Jenkins server monitors the GitHub repository for any changes and triggers a build job that connects to the Ansible server via passwordless authentication for SSH. The Ansible server then runs a playbook that pushes the latest code to the web server.

## Project Architecture

The following diagram shows the architecture of the project:

![Project Architecture](https://github.com/Antovex/CICD-pipeline-project/blob/main/Architecture-image/CICD-Architecture.png?raw=true)

The four servers are:

- Web Server(s): This is the server that hosts the website using httpd. This server is responsible for serving the webpage. It can be 1 or more, but I have used 1 for demonstration.
- Developer Server (Optional): This is the server that the developer uses to write and push the website code to the GitHub repository. Any other machine which has Git installed on it and have the permission to push and pull form repo will also work.
- Jenkins Server: This is the server that runs Jenkins, an open source automation server that orchestrates the CICD pipeline. This server is responsible for making SSH connection to Ansible server.
- Ansible Server: This is the server that runs Ansible, an open source automation tool that executes tasks on remote servers. This server is responsible for making SSH connection to Web server(s) and update the html file.

## Project Setup

To set up this project, you need to follow these steps:

1. Make 4 instances in AWS (Free tier t2.micro will also work) with Amazon Linux 2023. Change the inbound rules for all the 4 VMs to accept "All Traffic" from "Anywhere IPv4" for demonstration.
   
2. <p>Connect to all the servers in different tabs and rename using the below command as Jenkins, Ansible, Web-server and Dev-server.<br></p>

   ```javascript
   import copyCodeBlock from '@pickra/copy-code-block';
   // OR
   const copyCodeBlock = require('@pickra/copy-code-block');
   ```
   
   ```bash
   sudo su
   ```

   ```bash
   hostnamectl set-hostname <name>
   ```

   ```bash
   exit
   ```

   ```bash
   sudo su
   ```

   Your will now see the respective names of the server.
   
4. [Install and configure Jenkins](https://www.jenkins.io/doc/book/installing/) on the Jenkins server.

5. [Install and configure Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) on the Ansible server.

6. [Install httpd](https://docs.oracle.com/en/operating-systems/oracle-linux/6/admin/install-apache.html) on the Web server.

7. [Install git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) in the Jenkins and Dev-server.

8. In the ansible and the webserver we need to set passwordless Authentication so that Jenkins can connect to ansible server and ansible server can connect to webserver.
   
   Follow for the Jenkins, Ansible and Web-server.
   
   ```bash
   passwd root (set a new password)
   ```
   
   ```bash
   vi /etc/ssh/sshd_config
   ```
   
   and uncomment “PermitRootLogin” and set it to “yes” and scroll down to “PasswordAuthentication” and set it to “yes also”.
   Then,
   
   ```bash
   systemctl restart sshd
   ```
   
10. Now generate a new ssh key in jenkins server.

   ```bash
   ssh-keygen
   ```
   now just press "Enter" untill you got back to your terminal.

11. Then copy the ssh key to ansible server as jenkins will communicate with ansible.

    ```bash
    ssh-copy-id -i /root/.ssh/id_rsa.pub root@<Private IP of ansible server>
    ```

13. Do the above step for Ansible to Web-server.

14. Add the PRIVATE IP of webserver in the Ansible Host/Inventory file.

15. Create a new Github repo (Public) and add a new index.html file with simple text.

16. Open your Jenkins website using the public IP address of the Jenkins server and appending it with ":8080"\
    
    ```bash
    http://<Public IP of Jenkins Server>:8080
    ```

    Get Your temporary password using

    ```bash
    cat /var/jenkins_home/secrets/initialAdminPassword
    ```
    Install the Suggested Plugins and enter all details.
    
18. Go to the jenkins website, click on "Create a job", then choose "Freestyle project" and select "Source Code Management" as “git” and add your newly made public repo url. Finally Save it.

19. <p>Go to  Dashboard > Manage Jenkins > Plugins, on the left pane click on "Available Plugins" and search "Publish Over SSH" Plugin<br>
    Install the Plugin and then scroll to the bottom and select the checkbox so that the jenkins restarts after the plugin installation is completed.</p>

20. <p>After the restart, Login with your details, then<br>
    Go to Dashboard > Manage Jenkins > Configuration<br>
    and scroll to the end and click “Add SSH server”<br>
    Add 2 servers, 1 is Jenkins with Private IP and Ansible with private IP.<br>
    NOTE: Add Username as “root” and click on “Advance” and check “**Use password authentication, or use a different key**” and set the password that you used while creating new password in the servers itself.</p>
    
21. Go back to Dashboard then select your jenkins project, on the left pane select "Configure" and check that you have used the correct github repo link and also mentioned the correct branch (i.e. main or master)

22. <p>Scroll down to the "Build Steps" section, then click on "Add build step" and choose "Send files or execute commands over SSH" from the dropdown menu.<br>
    Select "Name" as Jenkins.<br>
    Scroll down to "Exec command" and put the following command<br></p>
    
   ```bash
   rsync -avh /var/lib/jenkins/workspace/<Name of your FreeStyle Project>/*.html root@<Private IP of Ansible server>:/opt/index.html
   ```

20. Under "Build Triggers", check "GitHub hook trigger for GITScm polling".

21. <p>In the "Post-build Actions" section, click on "Add post-build action" and choose "Send build artifacts over SSH" from the dropdown menu.<br>
    Select "Name" as "Ansible".<br>
    Scroll down to "Exec command" and put the following command<br></p>
    
    ```bash
    ansible-playbook <Full path of the playbook location>
    ```
    (Location at which you have the [playbook](https://github.com/Antovex/CICD-pipeline-project/blob/main/playbook/deploment.yml) in the Ansible server)

22. <p>Open another browser window, open your github repo where you have your "index.html"<br>
     Go to "Settings" (of the same repo)<br>
     On the left pane click on "Webhooks" and then "Add webhook"<br>
     Under the "Payload URL" enter your Jenkins url and append it with "/github-webhook/"<br>
     Select "Content type" as "application/json"
     For "Secret", go to Jenkins and click on the username on the top righthand corner, click on "configure", then under "API token" click "Add new Token" then generate it, then copy and paste it in the "Secret" field.<br>
     Finally click on "Add webhook".<br>
     Don't forget to "Save" the configuration done till now in the Jenkins also.</p>
     
23. To open the website just copy the "Public IP" of web-server and paste it in the browser.

24. After this, whenever the Developer make changes to the index.html file and pushes it in the GitHub, the deployment and the changes will be automatically done.

## Project Demo

To see the project in action, you can watch this video:

[Project Demo Video]
