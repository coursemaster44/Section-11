# Section-11
# Deploying-Sample-App-With-CRUD-Functionality-on AWS-With-(CI-CD)

# 0-crud-cb-lab
**Step 1.Open the Visual Studio Code**
- Run the following commands
```sh
$ git status
$ git add .
$ cp ../buildspec.yml .
$ git status
$ git add .
$ git commit -m "commiting changes made to env.prod,Procfile,package and buildspec"
$ git push
```
**Step 2.Goto Developer Tools>CodeCommit>Repositories>crud-app**
- See All the things have been copied 

**Step 3.Goto CodeBuild>Build Projects>Create build project**
- Project name - crud-cb
- In Source Tab
  - Source provider - AWS CodeCommit
  - Repository - crud-app
  - Reference Type select “Branch” and Branch as “Master”
    - Source version - "commiting changes made to env.prod,Procfile,package and buildspec"

- In Environment Tab
  - Environment image - Managed image
  - Operating system - Amazon Linux 2
  - Runtime(s) - Standard
  - Image - aws/codebuild/amazonlinux2-x86_64-standard:3.0
  - Image version - Always use the latest image for this version
  - Service role - Existing service role
  
- In Buildspec section:
  - Build specifications - select “Use a buildspec file”

- Artifacts Section:
  - Type - Amazon S3
  - Bucket name - Sample-Node_App-amit
  - Name - devbuild-crud
  - Path - devbuilds/
  - Artifacts packaging - Select Zip
  - Select “Disable artifacts encryption”

- Logs section
  - CloudWatch - Select Cloudwatch logs
  - Group name - “cb-project-s3”
  - Stream name - "cb-project-s3"

Click on Create Build Project

**Step 4.Click on Start Build**
- In Build configuration
  - Timeout - 0 Hours 5 Minutes
  
Click on Start Build

**Step 4. See Build Status>Phase details**

**Step 5. Now Goto S3>sample-node-app-amit>devbuild-crud**
- See crud-cb >zip file

# End of Lab












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

**Step 1.Open the Visual Studio Code**
- Run the following commands
```sh
$ git status
# modified: deploy_scripts/run.sh
$ git add .
$ git commit -m "modified run.sh script"
$ git push
```
**Step 2.Goto Ec2>Security Groups>sg-xxxx>Edit inbound rules**
- Protocol:Port = Tcp:8080 from Source(0.0.0.0/0) 
- Click on Save

**Step 3.Goto Developers Tools>CodeBuild>Build Projects>crud-cb**
- Click on Start Build
- See the Source version is "modified run.sh script"
- Click on Start Build

**Step 4.See Phase details**
- Wait for all phases to complete successfully

**Step 5.Goto S3>Buckets>sample-node-app>devbuild-crud/>crud-cb**
- Hit Refresh to see the changes
- To make crud-cb public - Object actions>Make Public>exit
- Copy S3 URI 

**Step 6.Developers Tools>CodeDeploy>Applications>crud-app-cd-**
- Click on Deployment group "crud-app-cd-dg"

**Step 7.Click on Create Deployment**
- See deployment settings 
- Revision type - My application is stored in S3 
- Revision location - Paste S3 URI
- Revision file type - .Zip

Click on Create Deployment

**Step 8.Click on View Events**
```sh
ApplicationStop------Success
DownloadBundle------Success
BeforeInstall------Success
Install-----------Success
AfterInstall------Success
ApplicationStart------Success
Validateservice  -------Success
```
**Step 9.Click Instance and copy & paste the Public IP address in browser to see it running**
- e.g.- 13.222.125.52:8080

**Step 10.Open Postman Tool>Environment>Manage Environments**
- Select ec2 environment and change the url Value as follows:
  - 13.222.125.52:8080

Click on Update and exit

**Step 11.Now select ec2 as Environment**

**Step 12.Now Open Postman Tool and select {{url}}/create table and Click on Send**
- Table created successfully

**Step 13.Check the Table created in DynamoDB**
- Goto AWS Console>DynamoDB>Tables
  - Table is created

**Step 14.Goto Postman Tool and put the following value**
- http://{{url}}/readData

**Step 15.Goto Postman Tool and put the following value**
- http://{{url}}/insertData

**Step 16.Now Goto AWS Console>DynamoDB>Tables>Items>info**
- New Item added successfully

**Step 17.Goto Postman Tool and put the following value**
- http://{{url}}/updateData

**Step 18.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data updated

**Step 19.Goto Postman Tool and put the following value**
- http://{{url}}/deleteData

**Step 20.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data deleted

**Step 21.Goto Postman Tool and put the following value**
- http://{{url}}/deleteTable

**Step 22.Now Goto AWS Console>DynamoDB>Tables**
- Refresh and see that Table is deleted

# End of Lab
