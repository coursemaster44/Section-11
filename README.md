# Section-11
# Deploying-Sample-App-With-CRUD-Functionality-on AWS-With-(CI-CD)
# 1-cd-single-ec2-lab-1

**Step 1.Open the Visual Studio Code**
- Run the following commands
```sh
$ cp ../appsec.yml .
$ cp -r ../deploy_scripts .
$ git status
$ git add .
$ git commit -m"commiting appsec and deploy_scripts"
$ git push
```
**Step 2.AWS Console>All services>EC2>Launch Instance**

**Step 3.Select Amazon Linux2 AMI**

**Step 4. Choose Instance type t2 micro**

Click Next:Configure Instance
**Step 5.Leave everything default beside IAM Role and User Data Section**
- IAM Role - Ec2S3FullAccess

- User Data>As Text:
```sh
#!/bin/bash
$ yum update -y
$ yum install ruby -y
$ yum install wget -y
$ yum install nmap-ncat -y
$ cd /home/ec2-user
$ wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
$ chmod +x ./install
$ ./install auto
$ service codedeploy-agent status
```
Click on Next:Add Storage

**Step 6.Keep Default**

Click on Next:Add Tags

**Step 7.Give key as Name and value as crud-server**

Click on Next:Configure Security Group

**Step 8.Configure Security Group:**

- Select Security Group with:
  - HTTP:80 Source-0.0.0.0/0
  - TCP:22 Source-0.0.0.0/0
  - Custom TCP Rule:3000 Source-0.0.0.0/0

Now Click on Review and Launch

**Step 9.Click on Launch>Select existing Key pair**

Now Click on Launch Instances

**Step 10.Goto AWS Console>Developers Tools>CodeDeploy>Applications>Create Application**
- Give Application name:- curd-app-cd
- Compute platform - Ec2/On-premises

Click on Create application

**Step 11.Deployment groups>Create deployment group**
- Give the name “ crud-app-cd-dg” in the deployment group name section.

**Step 12.Service Role-Create service role as following**
- Keep Default

**Step 13.In Deployment type choose - In-Place**

**Step 14.Select Amazon Ec2 instances in Environment configuration**
- In tags select key - Name and value - crud-server

**Step 15. Agent configuration with AWS Systems Manager**
- Never

**Step 16. Deployment settings - CodeDeployDefault:AllAtOnce**

**Step 17.Load Balancer - deselect it**

Click on Create deployment group

**Step 18.Now Click on Create Deployment**
- See deployment settings 

- Goto S3>Buckets>sample-node-app>devbuild-crud/>crud-cb
   - To make crud-cb public>Object actions>Make Public
   - Copy S3 URI 
- Revision type - My application is stored in S3 
- Revision location - Paste S3 URI
- Revision file type - .Zip

**Step 19.Click on Create deployment**
- Failed due to no AppSec file

**Step 20.Goto Developers Tools>CodeBuild>Build Projects>crud-cb**
- Click on Start Build
- See the Source version is commiting appsec and deploy_scripts
- Click on Start Build

**Step 21.See Phase details**
- Wait for all phases to complete successfully

**Step 22.Goto S3>Buckets>sample-node-app>devbuild-crud/>crud-cb**
- Hit Refresh to see the changes
- To make crud-cb public - Object actions>Make Public>exit
- Copy S3 URI 

**Step 23.Developers Tools>CodeDeploy>Applications>crud-app-cd-**
- Click on Deployment group "crud-app-cd-dg"

**Step 24.Click on Create Deployment**
- See deployment settings 
- Revision type - My application is stored in S3 
- Revision location - Paste S3 URI
- Revision file type - .Zip

Click on Create Deployment

**Step 25.Click on View Events**
```sh
ApplicationStop------Success
DownloadBundle------Success
BeforeInstall------Success
Install-----------Success
AfterInstall------Success
ApplicationStart------Success
Validateservice  - -----Failed---ScriptFailed
```
Click on ScriptFailed to see the details

**Step 26.Open Visual Studio Code**
- Goto deploy_scripts>validate.sh
  - Change port 3000 to 8080
- Security Group should also be opened for 8080

**Step 27.Now Run the following commands**
```sh
$ git add .
$ git commit -m "modified validate.sh to ping on port 8080"
$ git push
```
# End of lab
# 2-cd-single-ec2-lab-2
