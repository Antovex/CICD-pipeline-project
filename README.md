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
   
2. Connect to all the servers in different tabs and rename using the below command as Jenkins, Ansible, Web-server and Dev-server.
   ```sudo su ```

   ```hostnamectl set-hostname <name>```

   ```exit```

   ```sudo su```

   Your will now see the respective names of the server.
   
3. [Install and configure Jenkins](https://www.jenkins.io/doc/book/installing/) on the Jenkins server.

4. [Install and configure Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) on the Ansible server.

5. [Install httpd](https://docs.oracle.com/en/operating-systems/oracle-linux/6/admin/install-apache.html) on the Web server.

6. [Install git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) in the Jenkins and Dev-server.

7. In the ansible and the webserver we need to set passwordless Authentication so that Jenkins can connect to ansible server and ansible server can connect to webserver.
   
   Follow for the Jenkins, Ansible and Web-server.
   
   ```passwd root (set a new password)```
   
   ```vi /etc/ssh/sshd_config```
   
   and uncomment “PermitRootLogin” and set it to “yes” and scroll down to “PasswordAuthentication” and set it to “yes also”.
   Then,
   
   ```systemctl restart sshd```
   
8. Now generate a new ssh key in jenkins server.

   ```ssh-keygen```

9. Then copy the ssh key to ansible server as jenkins will communicate with ansible.

    ```ssh-copy-id -i /root/.ssh/id_rsa.pub root@<Private IP of ansible server>```

10. Do the above step for Ansible to Web-server.

11. Add the PRIVATE IP of webserver in the Ansible Host/Inventory file.

12. Create a new Github repo (Public) and add a new index.html file with simple text.

13. Open your Jenkins website using the public IP address of the Jenkins server and appending it with ":8080"\
    
    ```http://<Public IP of Jenkins Server>:8080```

    Get Your temporary password using

    ```cat /var/jenkins_home/secrets/initialAdminPassword```
    
    Install all the Recommended Plugins and enter all details.
    
15. Go to the jenkins website and create a new Freestyle project and select “**Source Code Management**” as “git” and add your newly made public repo url. Finally Save it.

16. <p>Go to Dashboard > Manage Jenkins > Configuration<br>
    and scroll to the end and click “Add SSH server”<br>
    Add 2 servers, 1 is Jenkins with Private IP and Ansible with private IP.<br>
    NOTE: Add Username as “root” and click on “Advance” and check “**Use password authentication, or use a different key**” and set the password that you used while creating new password.</p>


## Project Demo

To see the project in action, you can watch this video:

[Project Demo Video]
