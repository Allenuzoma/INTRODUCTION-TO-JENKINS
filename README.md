
# INTRODUCTION-TO-JENKINS


**Leveraging Jenkins for Automating a 3-Tier Web Application**

Recently, I completed a project on deploying and configuring a load balancer for a 3-tier web application, and now I'm diving deeper into automation with Jenkins!

Jenkins is a powerful automation server that streamlines the building, testing, and deployment process. For a 3-tier application, Jenkins plays a key role by automating Continuous Integration and Continuous Deployment (CI/CD) pipelines across the web, application, and database tiers. This ensures smooth integration, testing, and rollout of changes to each layer.

In my previous project, I manually configured a load balancer, but with Jenkins, the entire deployment process becomes more efficient. It automates updates, reduces human error, and minimizes downtime, making the whole application more robust and scalable. 🚀



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



          # Update package lists
          sudo apt update
          
          # Install Java 17 JRE
          sudo apt install openjdk-17-jre
          
          # Set the JAVA_HOME environment variable
          export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
          
          # Add the Java bin directory to the PATH
          export PATH="$JAVA_HOME/bin:$PATH"
          
          # Verify the Java installation
          java -version
          
          # Add the following lines to your .bashrc file:
          export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
          export PATH="$JAVA_HOME/bin:$PATH"
          
          # Source the file to apply the changes immediately:
          source ~/.bashrc


We can now proceed to install Jenkins, install step by step:

     

          sudo apt update
          
          sudo apt install openjdk-17-jre
          
          java -version
           
          curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
            /usr/share/keyrings/jenkins-keyring.asc > /dev/null
          echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
            https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
            /etc/apt/sources.list.d/jenkins.list > /dev/null
          
          sudo apt-get update
          
          sudo apt-get install jenkins
          
          sudo systemctl start jenkins.service
          
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
    
From your browser access 


      http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080


      
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


We will go to Github.com --> Settings --> Webhooks 

Add a new webhook, using the Jenkins URL as the Payload URL 

Set Content type to application/json.

  ![webhooks configured at the repo settings](https://github.com/user-attachments/assets/35df42e1-5904-4b6a-a60f-61830a29d3c4)

![webhooks configured at the repo settings png contents](https://github.com/user-attachments/assets/29dc10ad-b675-4329-93a0-fa11dd33c47e)



3. Go to Jenkins web console, click "New Item" and create a "Freestyle
project"



To connect your GitHub repository, you will need to provide its URL, you
can copy from the repository itself

In configuration of your Jenkins freestyle project choose Git repository,
provide there the link to your Tooling GitHub repository and credentials
(user/password) so Jenkins could access files in the repository.

![console output jenkins](https://github.com/user-attachments/assets/4df88a04-f55f-4d10-ae46-2e93709b58c1)




![build status](https://github.com/user-attachments/assets/9e6ef259-181b-4947-8e88-91e1006c92df)





Save the configuration and let us try to run the build. 
For now we can only
do it manually. Click "Build Now" button, if you have configured everything
correctly, the build will be successfull and you will see it under #1

You can open the build and check in "Console Output" if it has run
successfully.

If so - congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it
manually. Let us fix it.

5. Go to source code management and add the github repository link to ensure that changes trigger builds

   
   ![image](https://github.com/user-attachments/assets/fb5fb362-fea3-47d1-980e-6c07112a67c3)


Also  adjust the branch specifier to be */main


  ![image](https://github.com/user-attachments/assets/2e9cf6d8-d7b1-4954-bafb-db04e9c38207)


    
6. Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:
Configure "Post-build Actions" to archive all the files - files resulted from a
build are called "artifacts".
step1


![build triggers](https://github.com/user-attachments/assets/de6758c9-b49a-417a-b8d5-6e9f965b2dd4)

![post build actions](https://github.com/user-attachments/assets/5bf8bf38-587f-4ded-bd5f-70235d9ff976)

![archive artifacts settings](https://github.com/user-attachments/assets/748b60a6-c69c-4396-abb7-e545c109b1d5)



![archive artifacts settings png 2](https://github.com/user-attachments/assets/32aa5948-f897-448b-915f-c0d5e2fb76db)






Step2

Now, go ahead and make some change in any file in your GitHub repository
(e.g. README.MD file) and push the changes to the main branch.

You will see that a new build has been launched automatically (by
webhook) and you can see its results - artifacts, saved on Jenkins server.


![build 2 status](https://github.com/user-attachments/assets/2f8fe6e5-679f-4c6e-afe0-3d670ac232f5)

You have now configured an automated Jenkins job that receives files from
GitHub by webhook trigger (this method is considered as 'push' because the
changes are being 'pushed' and files transfer is initiated by GitHub). 
There
are also other methods: trigger one job (downstreadm) from another
(upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally



    ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
    

  ![jenkins build archive](https://github.com/user-attachments/assets/01292d92-a4a3-4618-a3a7-b799e30b79c4)

Step 3 - Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is
to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins
available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins"
menu item.

2. On "Available" tab search for "Publish Over SSH" plugin and install it

3. Configure the job/project to copy artifacts over to NFS server.

  First of all we have to generate our public and private key which will be crucial for the connection between Jenkins server and NFS server.
  Extract the keys by entering the command on the Jenkins server CLI:

        ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

   This command generates two rsa keys with 4096 bit size for stronger encryption and are stored in the .ssh directory

   Private Key: id_rsa (This will be used in the Publish over SSH section)
   
   Public Key: id_rsa.pub (This will be copied into the authorized key file located in the NFS server's .ssh directory)

  The private key can be accessed using the command:

      cat ~/.ssh/id_rsa

  The public key can be accessed using the command:

      cat ~/.ssh/id_rsa.pub

  Next we will copy this public key to the remote server's (NFS server) ./ssh/authorized_keys file using the command:


      ssh-copy-id -i ~/.ssh/id_rsa.pub <remote_user>@<remote_server_ip>


  Replace the <remote_user> by "ec2-user" for RHEL linux and <remote_server_ip> by the IP address of the NFS server.

  
  Alternatively, manually copy the contents of the above command to the NFS server by entering this command on the NFS server CLI:
      
      sudo nano ~/.ssh/authorized_keys
      
  We will have to secure the private key and render it unaccessible to unauthorized users by modifying permissions on the jenkins server 
      
      chmod 600 ~/.ssh/id_rsa
  
On main dashboard select "Manage Jenkins" and choose "Configure
  System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure
it to be able to connect to your NFS server:

1. Provide a private key (content of .pem file that you use to connect to
NFS server via SSH/Putty). The private key is 
2. Arbitrary name
3. Hostname - can be private IP address of your NFS server
4. Username - ec2-user (since NFS server is based on EC2 with RHEL 8, would have been "ubuntu" if we were using an ubuntu ec2 server) 
5. Remote directory - /mnt/apps since our Web Servers use it as a mounting
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

NOTE: On the NFS server, you have to create a new user for jenkins and assign appropriate ownership and permission for it to be able to access the /mnt/apps directory on NFS server.
     
      
      #check user
      id jenkins
      #add user 
      sudo useradd -u 1001 --system --shell /bin/bash --home /var/lib/jenkins jenkins
      #grant ownership
      sudo chown -R jenkins:jenkins /mnt/apps
      #grant permission
      sudo chmod -R 755 /mnt/apps
      #verify 
      ls -ld /mnt/apps





Webhook will trigger a new job and in the "Console Output" of the job you
will find something like this:
SSH: Transferred 25 file(s)
Finished: SUCCESS



![successful over ssh](https://github.com/user-attachments/assets/3e37c10f-decc-4f3b-b8e4-6118391ce032)


To make sure that the files in /mnt/apps have been udated - connect via
SSH/Putty to your NFS server and check README.MD file
      cat /mnt/apps/README.md

or 

      sudo nano /mnt/apps/README.md


![content of readme md](https://github.com/user-attachments/assets/f925676a-fd8a-44d0-956f-652e22605c5c)

If you see the changes you had previously made in your GitHub - the job
works as expected.


We have just implemented our first Continous Integration solution using
Jenkins CI. 
