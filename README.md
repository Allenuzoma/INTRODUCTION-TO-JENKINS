# INTRODUCTION-TO-JENKINS


Step 1 - Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and
  name it "Jenkins"

![instance creation](https://github.com/user-attachments/assets/cc8c1d40-4a62-44a8-adb5-b6a58dd20879)


3. Install JDK (since Jenkins is a Java-based application)


      sudo apt update
      sudo apt install default-jdk-headless

![sudo apt install jdk headless](https://github.com/user-attachments/assets/6dac7c2c-bf7a-484a-8cae-899d0e94f8e8)


   
   
5. Install Jenkins
Due to the fact that Jenkins requires JDK to run will will have to install it first


sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
source ~/.bashrc 



We can now proceed to install Jenkins

      wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key |sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg
      sudo sh -c 'echo deb [signed-by=/usr/share/keyrings/jenkins.gpg] http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
      sudo apt update
      sudo apt install jenkins
      sudo systemctl start jenkins.service
      #Since systemctl doesn’t display status output, we’ll use the status command to verify that Jenkins started successfully:
      sudo systemctl status jenkins

   

   
7. By default Jenkins server uses TCP port 8080 - open it by creating a
new Inbound Rule in your EC2 Security Group

![inbound rules](https://github.com/user-attachments/assets/b3424e03-1c03-4fbe-9f1c-a9c9a516ae34)

By default, Jenkins runs on port 8080. Open that port using ufw:


    sudo ufw allow 8080

Note: If the firewall is inactive, the following commands will allow OpenSSH and enable the firewall:

    
    sudo ufw allow OpenSSH
    sudo ufw enable
    

Check ufw’s status to confirm the new rules:

    sudo ufw status

  ![ufw status](https://github.com/user-attachments/assets/e57e0f14-d5c0-4db9-b017-c3c6818758e1)


9. Perform initial Jenkins setup.
    
From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS￾Name>:8080
You will be prompted to provide a default admin password


![unlock jenkins](https://github.com/user-attachments/assets/9ee7d1d6-860c-426b-a831-f2ea6f3e45b0)

Retrieve it from your server:


    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

    
  ![getting the jenkins admi password](https://github.com/user-attachments/assets/bebf45e8-1059-49ec-a398-b419a141b144)

![after signing in](https://github.com/user-attachments/assets/1382759b-0914-427b-8b0f-b45692244e50)


Then you will be asked which plugings to install - choose suggested plugins.

![after signing in](https://github.com/user-attachments/assets/c15e2bd3-c73b-44ab-a830-5b517b9e38ac)

Once plugins installation is done - create an admin user and you will get
your Jenkins server address.

![create first admin user](https://github.com/user-attachments/assets/70d048e5-bfc6-4ebf-8122-7a71ac340179)


![instance config](https://github.com/user-attachments/assets/a7cb92be-2f7b-4d26-a620-236a41209efe)

The installation is completed


Step 2 - Configure Jenkins to retrieve source codes from
GitHub using Webhooks

In this part, you will learn how to configure a simple Jenkins job/project
(these two terms can be used interchangeably). 

This job will will be triggered by GitHub webhooks and will execute a 'build' task to retrieve
codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings
0:00 / 0:17

3. Go to Jenkins web console, click "New Item" and create a "Freestyle
project"

To connect your GitHub repository, you will need to provide its URL, you
can copy from the repository itself

In configuration of your Jenkins freestyle project choose Git repository,
provide there the link to your Tooling GitHub repository and credentials
(user/password) so Jenkins could access files in the repository.

Save the configuration and let us try to run the build. 
For now we can only
do it manually. Click "Build Now" button, if you have configured everything
correctly, the build will be successfull and you will see it under #1

You can open the build and check in "Console Output" if it has run
successfully.

If so - congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it
manually. Let us fix it.

5. Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:
Configure "Post-build Actions" to archive all the files - files resulted from a
build are called "artifacts".
step1

Step2

Now, go ahead and make some change in any file in your GitHub repository
(e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by
webhook) and you can see its results - artifacts, saved on Jenkins server.

You have now configured an automated Jenkins job that receives files from
GitHub by webhook trigger (this method is considered as 'push' because the
changes are being 'pushed' and files transfer is initiated by GitHub). 
There
are also other methods: trigger one job (downstreadm) from another
(upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/


Step 3 - Confi gure Jenkins to copy fi les to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is
to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins
available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins"
menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it

3. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure
System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure
it to be able to connect to your NFS server:

1. Provide a private key (content of .pem file that you use to connect to
NFS server via SSH/Putty)
2. Arbitrary name
3. Hostname - can be private IP address of your NFS server
4. Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
5. Remote directory - /mnt/apps since our Web Servers use it as a mointing
point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success.
Remember, that TCP port 22 on NFS server must be open to receive SSH
connections.

Save the configuration, open your Jenkins job/project configuration page
and add another one "Post-build Action"

Configure it to send all files probuced by the build into our previouslys
define remote directory. In our case we want to copy all files and
directories - so we use **. If you want to apply some particular pattern to
define which files to send - use this syntax.

Save this configuration and go ahead, change something in README.MD file in
your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you
will find something like this:
SSH: Transferred 25 file(s)
Finished: SUCCESS

To make sure that the files in /mnt/apps have been udated - connect via
SSH/Putty to your NFS server and check README.MD file
cat /mnt/apps/README.md

If you see the changes you had previously made in your GitHub - the job
works as expected.

Congratulations!
You have just implemented your first Continous Integration solution using
Jenkins CI. Watch out for advanced CI configurations in upcoming projects
