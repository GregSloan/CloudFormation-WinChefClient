# CloudFormation-WinChefClient
CF Template and supporting files to launch a Windows 2012 R2 instance, install chef client, and register with chef server

## Pre-Requisites/Assumptions
* Existing Chef Server
* private validation key file for Chef Server Organization
* WebHost accessible from the VPC/subnet that the AWS instance will be deployed into with no authenticaiton required to download files
* This implementation assumes existing VPC/Subnet/Security Groups (tested with Chef Server in the same VPC/subnet)
* Cookbooks and Roles with runlists existing on Chef Server
* Template currently only works in us-west-2 (Oregon) and with Windows 2012 R2.  See Setup for other regions

## Setup
* If not already available, generate Chef Server Organization validation key
    * SSH into Chef Server
    * execute: sudo knife reregister [ORGANIZATION-NAME]-validator > validation.pem
* Edit "first-boot.json"
    * Replace "role[WebServer]" with the name of the desired role(s) existing on your Chef Server for the deployed instance
* Edit "client.rb"
    * Replace IP address with correct IP of your Chef Server
    * Replace "automate" with the name of your organization
* Upload these three files (validation.pem, client.rb, first-boot.json) to WebHost and note their URL locations
* Download WinChefClient.template to your local machine
* NOTE: If working in region other than us-west-2 (Oregon):
    * Edit WinChefClient.template - under "Mappings" replace the ami-id for your selected region with a different (valid) ami-id for the OS
* NOTE: If Windows 2008 is desired:
    * Under "Resources":"WindowsNode":"Properties", change "Windows2012r2" to "Windows2008r2"

## Operation
* Log in to AWS Console
* Open CloudFormation
* click "Create Stack"
* Select "Upload a template to Amazon S3" and click "Choose File"
    * Browse to the WinChefClient.template file downloaded previously
* Fill in the required parameters
    * ClientFileLocation = web address of client.rb (e.g. http://WebHost/chef-files/client.rb)
    * FirstBootLocation = web address of first-boot.json (e.g. http://WebHost/chef-files/first-boot.json)
    * ValidatorKeyLocation = web address of validation.pem (e.g. http://WebHost/chef-files/validation.pem)
* Click "Next" and accept defaults for following screens
* After ~5 minutes, the instance should be registered as a new node and Role cookbooks should be executed

## What Does It Do?
* Deploys base AMI
* downloads chef-client-14.4.56-1-x64.msi to C:\Users\Administrator\Downloads\chef-client.msi
* downloads validation.pem, first-boot.json, and client.rb to c:\chef
* Silent installs chef-client.msi (msiexec /qn /i C:\users\administrator\downloads\chef-client.msi ADDLOCAL="ChefClientFeature,ChefSchTaskFeature,ChefPSModuleFeature")
* Executes chef-client, passing first-boot.json (c:\opscode\chef\bin\chef-client.bat -j c:\chef\first-boot.json)