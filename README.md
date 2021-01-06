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
- Click on Send

**Step 15.Goto Postman Tool and put the following value**
- http://{{url}}/insertData
- Click on Send

**Step 16.Now Goto AWS Console>DynamoDB>Tables>Items>info**
- New Item added successfully

**Step 17.Goto Postman Tool and put the following value**
- http://{{url}}/updateData
- Click on Send

**Step 18.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data updated

**Step 19.Goto Postman Tool and put the following value**
- http://{{url}}/deleteData
- Click on Send

**Step 20.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data deleted

**Step 21.Goto Postman Tool and put the following value**
- http://{{url}}/deleteTable
- Click on Send

**Step 22.Now Goto AWS Console>DynamoDB>Tables**
- Refresh and see that Table is deleted

# End of Lab





# 3-crud-deployment-on-asg-lab
**Step1-AWS Console>All Services>EC2>Auto Scaling>Launch Configuration>lc-cd**

**Step 2. Select launch configuration “lc-cd”**
- Click Actions>create auto scaling group

**Step 3. Give auto scaling group name - asg-cd**

See launch-configuration and click on Next

**Step 4. Select default vpc and subnets all three subnets**

Click Next

**Step 5. Select No Load Balancer and Health checks with ELB**
- health check grace period 120 seconds
Click Next

**Step 6. Keep Group size as following**
- Desired - 1
- Minimum - 1
- Maximum - 1

**Step 7. Select Scaling policies-None Click on Next**

**Step 8. In Add Notifications Click Next**

**Step 9. In Add tags Click Next**

**Step 10. See Review and Click on Create Auto Scaling Group**

**Step 11. AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>Create Deployment Group**

**Step 12.Provide details:**
- Deployment group name - crud-app-cd-asg-dg
- Service role - Same
- Deployment Type - In-Place
- Environment configuration - Select Auto Scaling group asg-cd
- Deployment settings - CodeDeployDefault:AllAtOnce
- Load Balancer - No Load Balancer

Click Create deployment group

**Step 13. Click on Create Deployment**
- keep Deployment group as it is
- Revision type - My application is stored in Amazon S3
- Revision location - Paste the S3 URI(s3://sample-node-app-amit/devbuild-crud/crud-cb)
- Revision file type - .zip

Click on Create Deployment

**Step 14.Monitor Deployment LifeCycle Events**
```sh
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
Validateservice
```
**Step 18. Copy the Instance Ip address and paste it in browser**
- e.g. 13.221.112.30:8080

See the application running

**Step 19.Ec2>Auto Scaling groups>asg-cd>Edit Group size**
- Desired - 2
- Minimum - 2
- Maximum - 2

Click on Update

**Step 20.Ec2>Auto Scaling Group>asg-cd>Instances**
- Click on the new Instance and copy Public IP address 

**Step 21.Open Postman Tool**
- Click on Manage Environments>Add environment
  - Name - ec2-another
  - Variable - url
  - Initial value - http://13.222.125.52:8080
- Click on Add and exit

**Step 22.Click on Manage Environments>Ec2**
- Copy 2nd Instance's Public IP address
- Paste it in Initial value of ec2
- Click on Update

**Step 23.Now select ec2 as Environment**

**Step 24.Now select {{url}}/create table and Click on Send**
- Table created successfully

**Step 25.Check the Table created in DynamoDB**
- Goto AWS Console>DynamoDB>Tables
  - Table is created

**Step 26.Goto Postman Tool and select ec2-another as Environment**
- Put the value - http://{{url}}/readData
- Click on Send

**Step 27.Goto Postman Tool and select ec2 as Environment**
- Put the value - http://{{url}}/insertData
- Click on Send

**Step 28.Now Goto AWS Console>DynamoDB>Tables>Items>info**
- New Item added successfully

**Step 29.Goto Postman Tool and select ec2-another as Environment**
- Put the value - http://{{url}}/updateData
- Click on Send

**Step 30.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data updated

**Step 31.Goto Postman Tool and select ec2 as Environment**
- Put the value - http://{{url}}/deleteData
- Click on Send

**Step 32.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data deleted

**Step 33.Goto Postman Tool and select ec2-another as Environment**
- Put the value - http://{{url}}/deleteTable
- Click on Send

**Step 34..Now Goto AWS Console>DynamoDB>Tables**
- Refresh and see that Table is deleted

# End of Lab



# 4-crud-deployment-on-asg-alb-lab

**Step 1. Go to AWS Console>All Services>EC2>Load Balancing>Target Groups>Create Target Group**

**Step 2. Specify group details as following**
- In Basic Configuration
  - Select Instances in target type
  - Target group name - tg-cd
  - Protocol-HTTP,Port-8080
  - VPC-Default
  - Protocol version - HTTP1
- Health Checks
  - Health check protocol- HTTP
  - Health check path- /

Now Click on Next

**Step 3. In Register targets**
- Select 2 instances from the available instances
- Ports for the selected instances-8080
- Click on "Include as pending below"

Now Click on Create target group

**Step 4. "tg-cd" Target Group has been created**

**Step 5. Go to AWS Console>All Services>EC2>Load Balancing>Load Balancers>Create load balancer**

**Step 6. Click on Create in Application Load Balancer**

**Step 7. Configure Load Balancer as following In Basic Configuration**
- Name- alb-cd
- Scheme- internet-facing
- Ip address type- ipv4
- Listeners-Protocol-HTTP,Port-80
- Availability Zones - VPC-Default - Availability Zones-Select all 3 Zones

Click Next:Configure Security Settings

**Step 8.Configure Security Groups**
- Select security group

Click on Next:Configure Routing

**Step 9.Configure Routing**
- Target group- Select Existing target group Name - tg-cd

Click on Register Targets

**Step 10.Click on Next:Review>Click on Create>Click on Close**

**Step 11.Developers Tools>CodeDeploy>Applications>crud-app-cd>crud-app-cd-dg>Edit**
- Provide the following details:
  - Load Balancer - Enable Load Balancing with Application load Balancer
  - Choose target group - tg-cd

Click on Save changes

**Step 12.EC2>Load Balancing>Load Balancers>Target groups>tg-cd**
- See that registered instances are Healthy

**Step 13.Goto Visual Studio Code**
- Edit index.ejs file for the new color
- Run the following commands
```sh
$ git status
$ git add .
$ git commit -m "changed index color"
$ git push
```
**Step 14.Goto Developers Tools>CodeBuild>Build Projects>crud-cb**
- Click on Start Build
- change timeout - 0 hrs 5 mins
- See the Source version is "changed index color"

Click on Start Build

**Step 15 .See Phase details**
- Build has been completed successfully


**Step 16.Goto S3>Buckets>sample-node-app>devbuild-crud/>crud-cb**
- Hit Refresh to see the changes
- To make crud-cb public - Object actions>Make Public>exit
- Copy S3 URI 

**Step 17.Developers Tools>CodeDeploy>Applications>crud-app-cd>crud-app-cd-dg**
- Click on Create Deployment

**Step 18.In Create Deployment settings** 
- Revision type - My application is stored in S3 
- Revision location - Paste S3 URI
- Revision file type - .Zip

Click on Create Deployment

**Step 19.Click on View Events of one instance**
- Wait for the success of following events
```sh
BeforeBlockTraffic
BlockTraffic
AfterBlockTraffic
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
Validateservice
BeforeAllowTraffic
AllowTraffic
AfterAllowTraffic
```

**Step 20.Copy DNS name value of ALB and paste it in browser**
- Hit refresh to see change in color for second instance


**Step 21.Open Postman Tool>Environment>Manage Environments**
- Select ALB environment and change the url current Value DNS name of ALB:
  - http://alb-crud-xxxxx.ap-south-1-1.elb.amazonaws.com

Click on Update and exit

**Step 22.Now select ec2-another as Environment**

**Step 23.select {{url}}/create table and Click on Send**
- Table created successfully

**Step 24.Check the Table created in DynamoDB**
- Goto AWS Console>DynamoDB>Tables
  - Table is created

**Step 25.Goto Postman Tool and Now select ALB as Environment**
- Put the following value - http://{{url}}/insertData
- Click on Send

**Step 26.Now Goto AWS Console>DynamoDB>Tables>Items>info**
- New Item added successfully

**Step 27.Goto Postman Tool and Now select ec2 as Environment**
- Put the following value - http://{{url}}/readData
- Click on Send


**Step 28.Goto Postman Tool and Now select ALB as Environment**
- http://{{url}}/updateData
- Click on Send

**Step 29.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data updated

**Step 30.Goto Postman Tool and Now select ec2-aother as Environment**
- http://{{url}}/deleteData
- Click on Send

**Step 31.Now Goto AWS Console>DynamoDB>Tables>Items>info>actors**
- Check for the data deleted

**Step 32.Goto Postman Tool and Now select ALB as Environment**
- http://{{url}}/deleteTable
- Click on Send

**Step 33.Now Goto AWS Console>DynamoDB>Tables**
- Refresh and see that Table is deleted

**Step 34.Goto Ec2Dashboard>Select Instance>Security>Securtiy groups>sg-xxxx**
- Edit inbound rules
  - Protocol:Port | TCP:8080 from Source:"SG of ALB"
- Click on Save rules

**Step 35.Goto Postman Tool and Now select ALB as Environment**
- Put the following value - http://{{url}}/createTable
- Click on Send

**Step 36.Check the Table created in DynamoDB**
- Goto AWS Console>DynamoDB>Tables
  - Table is created

**Step 37.Goto Postman Tool and Now select ec2 as Environment**
- Put the following value - http://{{url}}/insertData
- Click on Send
- Error:It is not working 

**Step 39.Goto Postman Tool and Now select ALB as Environment**
- Put the following value - http://{{url}}/insertData
- Click on Send
- Item added successfully

**Step 40.Goto Postman Tool and Now select ALB as Environment**
- Put the following value - http://{{url}}/readData
- Click on Send
- It is working

# End of Lab



