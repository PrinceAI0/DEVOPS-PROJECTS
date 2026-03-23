## **TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS**

- In previous **Project 8**, we introduced the **horizontal scalability** concept, which allows us to add new Web Servers to our Tooling Website. You have successfully deployed a set-up with 2 Web Servers and a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that you would need to repeat the same task repeatedly adding dozens or even hundreds of servers.
DevOps is about Agility and the speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is the **Automation** of routine tasks.

- In this project, we are going to start automating part of our routine tasks with a free and open-source automation server – [Jenkins](https://en.wikipedia.org/wiki/Jenkins_(software)). It is one of the most popular [CI/CD](https://en.wikipedia.org/wiki/CI/CD) tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

  According to Circle CI, **Continuous integration (CI)** is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.
In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub ``https://github.com/<yourname>/tooling`` will be automatically be updated to the Tooling Website.

- ### Task
  Enhance the architecture prepared in Project 8 by adding a Jenkins server, and configuring a job to automatically deploy source code changes from Git to the NFS server.
  Here is what your updated architecture will look like upon completion of this project:

![alt text](Images/1.png)

## **INSTALL AND CONFIGURE JENKINS SERVER**
**Step 1 – Install the Jenkins server**

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins" and  open TCP port 8080 in the security group.
- Install JDK (since Jenkins is a Java-based application) and verify java worked

  ``sudo apt install fontconfig openjdk-17-jre -y``

  ``java -version``

![alt text](Images/5.png)

- We’ll install Jenkins manually:

```
wget https://pkg.jenkins.io/debian-stable/binary/jenkins_2.541.3_all.deb

sudo dpkg -i jenkins_2.541.3_all.deb

sudo apt --fix-broken install -y
```

![alt text](Images/6.png)

-  Start Jenkins and make sure Jenkins is up and running

```
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
![alt text](Images/7.png)

- From your browser access ``http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080``

![alt text](Images/2.png)

- You will be prompted to provide a default admin password: ``sudo cat /var/lib/jenkins/secrets/initialAdminPassword`` copy and paste the password.

![alt text](Images/3.png)

- Once plugin installation is done:  Create an admin user and you will get your Jenkins server address.
The installation is completed!

![alt text](Images/4.png)


### **Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks**

- In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will be triggered by GitHub [webhooks](https://en.wikipedia.org/wiki/Webhook) and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.
- Enable webhooks in your GitHub repository settings

![alt text](Images/8.png)

![alt text](Images/9.png)

![alt text](Images/10.png)

- Go to Jenkins web console, click **"New Item"** and create a **"Freestyle project"**

![alt text](Images/12.1.png)

![alt text](Images/12.2.png)

- To connect your GitHub repository, you will need to provide its URL, you can copy it from the repository itself

![alt text](Images/11.png)

- In the configuration of your Jenkins freestyle project choose Git repository, and provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![alt text](Images/13.png)

- Save the configuration and let us try to run the build. For now, we can only do it manually.
Click the **"Build Now"** button, if you have configured everything correctly, the build will be successful and you will see it under ``#1``

- You can open the build and check in "Console Output" if it has run successfully.
If so – congratulations! You have just made your very first Jenkins build!
But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

![alt text](Images/14.png)

- Click **"Configure"** your job/project and add these two configurations

1. Configure triggering the job from the GitHub webhook:

![alt text](Images/15.png)

- Configure **"Post-build Actions"** to archive all the files – files resulting from a build are called **"artifacts"**.

![alt text](Images/16.png)

- Now, go ahead and make some changes in any file in your GitHub repository (e.g. ``README.MD`` file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on the Jenkins server.

![alt text](Images/17.png)

- You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and file transfer is initiated by GitHub). There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.
By default, the artifacts are stored on the Jenkins server locally

  ``ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/``

![alt text](Images/18.png)

### **Step 3 – Configure Jenkins to copy files to NFS server via SSH**

- Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to ``/mnt/apps`` directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called [Publish Over SSH](https://plugins.jenkins.io/publish-over-ssh/).
1. Create an EC2 RHEL instance and add 3 EBS volume(10GB each).
2. Created 3 web server 

- Install the "Publish Over SSH" plugin.

  On the main dashboard select "Manage Jenkins" and choose the "Manage Plugins" menu item.
On the "Available" tab search for the **"Publish Over SSH"** plugin and install it

![alt text](Images/19.png)

- Configure the job/project to copy artifacts over to the NFS server.

- On the main dashboard select "Manage Jenkins" and choose the **"Configure System"** menu item.

- Scroll down to **Publish over the SSH plugin** configuration section and configure it to be able to connect to your NFS server:

1. Provide a private key (the content of .pem file that you use to connect to the NFS server via SSH/Putty)
2. Arbitrary name
3. Hostname – can be **private IP** address of your NFS server
4. Username – **ec2-user** (since the NFS server is based on EC2 with RHEL 8)
5. Remote directory – **/mnt/apps** since our Web Servers use it as a mounting point to retrieve files from the NFS server

- Test the configuration and make sure the connection returns **Success**. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![alt text](Images/20.png)

![alt text](Images/21.png)

- Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

![alt text](Images/22.png)

![alt text](Images/23.png)

- Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this syntax.

- Save this configuration and go ahead, and change something in ``README.MD`` file in your GitHub Tooling repository and git push.
Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
```
Finish: SUCCESS
```
![alt text](Images/24.png)

- But mine in the picture above says ``unstable`` because root user created the directory ( ``sudo mkdir /mnt/apps`` ) and Jekins connect as ``ec2-user`` not ``root``, that means ``ec2-user`` cannot write to ``/mnt/apps`` so jenkins failed and showed ``unstable``..

- So i change from ``root user`` to ``ec2-user`` by using the command below: 

  ``sudo chown -R ec2-user:ec2-user /mnt/apps``  

   ``sudo chmod -R 755 /mnt/apps``

   `` ls -ld /mnt/apps`` To confirm if it worked. 

![alt text](Images/25.png)

- After that make a little change in your GitHub Tooling repository (``README.md``) and Push

![alt text](Images/26.png)

- To make sure that the files in ``/mnt/apps`` have been updated – connect via SSH/Putty to your NFS server and check README.MD file

  ``cat /mnt/apps/README.md``

![alt text](Images/27.png)



## **Congratulations!**

You have just implemented your first **Continous Integration solution using Jenkins CI**. Watch out for advanced CI configurations in upcoming projects.















