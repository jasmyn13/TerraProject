# Terra Gitlab Project

# Description

Create a CI/CD Pipeline through Docker and a self hosted Gitlab server to create an output that says "Hello, Jasmyn."

# What you need
 1. Docker
 2. Terminal
 3. Vim/Nano
 4. Gitlab (that you will provision yourself)

# Creating your Gitlab Host

Use this command in your terminal to create the Gitlab Host:
```
docker run --detach \
  --publish 1443:443 --publish 8080:80 --publish 1001:22 \
  --name gitlab \
  --restart always \
  --volume gitlab_config:/etc/gitlab \
  --volume gitlab_logs:/var/log/gitlab \
  --volume gitlab_data:/var/opt/gitlab \
  --shm-size 4gb \
  gitlab/gitlab-ce:latest
  ```
  Give it about 5 to 10 minutes, then type in your browser: http://localhost:8080 where you will be able to log into your Gitlab Server.
  Use "root" as the username and you can find your password using this command: ``` docker exec -it gitlab grep "Password:" /etc/gitlab/initial_root_password ``` Save your password!
  
  # Create a New Project
   1. Log into Gitlab
   2. Click "New Project"
   3. Click "Create Blank Project"
   4. Create a unique project name, initalize with a README, keep it Public, and press "Create Project."
  
  # Creating your Gitlab Runner
  
  Use this command in your terminal once your Gitlab Host is up:
  
``` 
  docker volume create gitlab-runner-config
  docker run -d --name gitlab-runner --restart always --network host \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitlab-runner-config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
 ```
  
  Next you will need to CD into your Gitlab-Runner container using this command: 
  ``` docker ps ```
   ``` docker exec -ti <containerid> bash ```
 
 # Registering your Runner
 
 Use this command INSIDE the Gitlab-Runner container: ``` gitlab-runner register ```
 
 You will need to add the following:
  1. URL (found on the Settings/CICD/Runners of your Gitlab Server)
  2. Registration Token (You'll find that in the same place as the URL)
 <img width="1444" alt="Screen Shot 2022-09-23 at 10 21 01 AM" src="https://user-images.githubusercontent.com/100259466/191982809-364ec2aa-8c70-4862-a1ce-c325f4cda514.png">
 <img width="1441" alt="Screen Shot 2022-09-23 at 10 23 56 AM" src="https://user-images.githubusercontent.com/100259466/191983277-549f9db7-ee25-49d4-8fad-afea242ab8d9.png">
  3. Leave everything else blank, and when it asks about your preference of an Executor, use "shell"
 <img width="568" alt="Screen Shot 2022-09-23 at 10 25 41 AM" src="https://user-images.githubusercontent.com/100259466/191983788-6bafe435-9896-4d39-92ad-db242d3bcb7b.png">
 
 # Editing the Config File
 
 1. Make sure you are in your Gitlab Runner container. ``` docker exec -it <gitlab-runner container ID> bash ```
 2. Use the command ``` vim /etc/gitlab-runner/config ``` to edit the config file.
 3. Change the URL to  ``` clone_url = "http://localhost:8080" ``` as shown in the image attached:
  <img width="575" alt="Screen Shot 2022-09-23 at 10 39 29 AM" src="https://user-images.githubusercontent.com/100259466/191986580-689d34f4-77e2-4ff7-8494-5b37f571633c.png">
  
  # Create a .gitlab-ci.yml file
   1. Go into your Projects on your Gitlab Server
   2. Click the + sign, then click "add file."
   3. Create a .gitlab-ci.yml file which allows the pipelines to be triggered.
   4. Place the following code into the text box, and click "commit changes."
    ``` build-job:
  stage: build
  script:
    - echo "Hello <replace with your name>" ```
    
 It should look like this when you complete it: 
<img width="1171" alt="Screen Shot 2022-09-23 at 10 44 02 AM" src="https://user-images.githubusercontent.com/100259466/191987622-9e07c751-cdee-40fb-8739-c99e73abf444.png">

Once you commit changes, the CI/CD Pipeline will automatically trigger, if successful, your finished project should look like this:

<img width="1378" alt="Screen Shot 2022-09-23 at 10 45 53 AM" src="https://user-images.githubusercontent.com/100259466/191988092-042bf2c1-cbcf-4c9d-b2c2-2d43e5a13e88.png">

# Installing Terraform on Docker's Gitlab-Runner
 1. Use the command to go into your Gitlab-Runner container: ``` docker exec -it <container ID> bash ```
 2. I did it the unzip/zip method - you can also follow the Hashicorp method of installing: https://learn.hashicorp.com/tutorials/terraform/install-cli
 
  ``` apt-get install zip ```
  
  Confirm latest version of Terraform, which is 1.3.0
  
  ``` unzip terraform_1.3.0_linux_amd64.zip ```
  
  ``` mv terraform /usr/local/bin/ ```
  
  ``` terraform --version ```
  
  If that is succesful, it should look like this:
  
  <img width="567" alt="Screen Shot 2022-09-23 at 2 12 03 PM" src="https://user-images.githubusercontent.com/100259466/192030997-17cb20cf-08ac-4e19-b483-414c913d2fbb.png">
  
 # Using Terraform Code to Provison an EC2 Instance through the Docker Container:
  
 1. SSH into your Gitlab-Runner Docker container, and create a VI document  
     
     ``` vi <insert directory name> ```
 2. Once that opens up, you want to copy and paste this code:
 
 ``` 
 provider "aws" {
 region     = "us-east-1"

}

resource "aws_instance" "instance1" {
   ami = "ami-05fa00d4c63e32376"
   instance_type = "t1.micro"
   key_name= "aws_key"
   count = 2
   vpc_security_group_ids = ["sg-070aeb8eb6df2f931"]
   ```
  <img width="566" alt="Screen Shot 2022-09-30 at 11 00 04 AM" src="https://user-images.githubusercontent.com/100259466/193299109-d2897172-4692-4649-9fe6-2724fccc9e65.png">
 
   
   Keep in mind to replace the AMI with your AMI, and to change the "count" to 2, if you want to run two EC2 instances.
   Also, change the VPC_Security_group_ids to your default security group listed on your AWS console.
   Make sure the Region listed is YOUR default region.

 # Adding Variables to GitLab so you don't hardwire your Secret Keys:
    
 1. Go into Gitlab, Settings, CI/CD, and then Variables.
    
<img width="1549" alt="Screen Shot 2022-09-27 at 2 03 13 PM" src="https://user-images.githubusercontent.com/100259466/192602767-cdda67bf-20dc-4f47-be84-32952ec24fd8.png">
  
 2. Click "Add Variable."
 3. Put in the Key for your AWS Credentials.
     
 ``` AWS_ACCESS_KEY_ID ```
     
 ``` AWS_SECRET_ACCESS_KEY ```
     
 4. In the "value" box, put in the corresponding AWS Keys.
 5. Add another variable, the Key should be "region." In the "value" box, place the default region of your AWS Console in there. You can find that in your AWS Console at the top right of your screen. Example picture:
     
<img width="1640" alt="Screen Shot 2022-09-27 at 2 09 26 PM" src="https://user-images.githubusercontent.com/100259466/192603870-f96b694c-5ec8-40bb-8dbe-72e9989edb62.png">
     
 Once you've completed those steps, rerun your Pipeline and it should look like this if it was successful:
    
<img width="1567" alt="Screen Shot 2022-09-27 at 2 11 29 PM" src="https://user-images.githubusercontent.com/100259466/192604138-92d8947b-ba40-4555-a5c3-7fad8cdaa74f.png">

# Using Logic to create a Merge_Request Pipeline

1. Open up the .gitlab-ci.yml and use this code:

```
job:
  script: echo "Hello, Jasmyn!"
  rules:
    - if: $CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "merge_request_event" 
      when: manual
      allow_failure: false
```
The code is allowing the pipeline to autoamtically start when there is a successful merge request. 
The "when" and "allow_failure" modules are not necessary in your code for this to run successfully. Allow-failure can be listed as "true" when running scans or tests, but should never run "true" when actually deploying. The "when" module allows you to specify whether you want it done manually, and when you want it done manually.
