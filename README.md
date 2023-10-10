# CI-CD-project
This is a project for creating a CI/CD pipeline with Github and Docker to deploy a node js app
# Prerequisites
Before you begin, make sure you have the following:
• an AWS free tier account
• a github account
• fork repository from https://github.com/LondheShubham153/node-todo-cicd.git

# Getting started
# Step 1: Create ec2 instance
Make an ec2 instance in free tier with a key pair and configure security group to allow all traffic

# Step 2: Login to ec2 instance
Login to ec2 instance using the key pair remotely or from aws console

# Step 3: Install Java
update apt and install java
   sudo apt update
   sudo apt install openjdk-11-jre
   java -version 

# Step 4: Install jenkins
  curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
   /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \  
   https://pkg.jenkins.io/debian binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins 
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   sudo systemctl status jenkins

# Step 5: Configure security group
configure ec2 intance security group to allow inbound connection to jenkins
type = Custom TCP , port = 8080 , source = anywhere ipv4

# Step 6: open jenkins
copy IPv4 address of the ec2 intance and paste into browser as <ip>:8080
enter password from /var/lib/jenkins/secrets/initialAdminPassword and make account

# Step 7: Make pipeline
Select 'new item' in jenkins dashboard and make freestyle project named todo-node-app
to give public access to your github through an ssh connection make new ssh key pair in ec2 instance by command
   ssh-keygen 
go to github and open settings, go to ssh and gpg keys 
add new key name it jenkins-project and paste the public key from ec2 intance

# Step 8: Configure pipeline
configuration of new freestyle pipeline named todo-node-app:
   in General
   Select 'GitHub project' and paste node-todo-cicd repo link
   in Source Code Management
   Select 'Git' and paste repo url
   Add 'Credentials' clicking on jenkins dropdown and in it
   Kind = SSH username with private key
   username = ubuntu
   Select 'enter private key directly'
   copy the private key from ec2 and paste it in here
   select 'add'
   after completing dropdown 'credentials' and select ubuntu
   Select 'Save'
Click on 'Build now' to build pipeline will build in workspace /var/lib/jenkins/workspace/todo-node-app
go to /var/lib/jenkins/workspace/todo-node-app directory on ec2
the github repo content is now on ec2 
to start node.js app,Run these commands:
   sudo apt install nodejs
   sudo apt install npm
   sudo npm install
   node app.js
or use dockerfile

# Step 9: Install docker and build image
on ec2 configure security group inbound rule :
   type = Custom TCP , port = 8000 anywhere ipv4
install docker by
   sudo apt install docker.io
give docker permission to run by 
   sudo usermod -a -G docker $USER
build the image using dockerfile by
   sudo docker build . -t node-app
run the image by
   sudo docker run -d --name node-todo-app -p 8000:8000 node-app
Continuous Integration is now complete.
open node app by <ipv4>:8000 into browser

# Step 10: Run app through jenkins
now kill the container so that we can run node app by Jenkins pipeline
   sudo docker kill <container id>
give jenkins permission to run docker by
   sudo usermod -a -G docker jenkins
   sudo systemctl restart jenkins
go to todo-node-app and configure pipeline
go to 'build steps' and in dropdown select 'Execute Shell' and write the commands into it
   docker build . -t node-app-todo
   docker run -d --name node-app-container -p 8000:8000 node-app-todo
save and build now
node app will run successfully

# Step 11: Create webhooks 
next we can create webhooks so that any changes in the github code will trigger the jenkins pipeline
for this go to jenkins dashboard and manage plugins and open plugins
open available plugins and search 'github integration'
install plugin and restart jenkins
next goto github node js app repository
go to repository settings and select webhooks and add webhook
payload url = jenkins url + github webhook (http://<ipv4>:8080/github-webhook/)
select 'Add webhook'
next go to jenkins and configure node-todo-app job
   in 'Build Triggers'
   select 'Github hook trigger for GITScm polling' then Save
Now any changes in the source code will trigger the pipeline to build
make sure to change the container name in todo-node-app job or kill the previous containers
