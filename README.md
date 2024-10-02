# OpenStack-Setup-and-Testing
In this project ill be completing 3 tasks:

- Deploying a Basic OpenStack Setup (All-in-One Node)
- Setting up a Multi-node OpenStack Deployment (ENV)
- Deploying a highly available Openstack ENV and using Automation with Heat Orchestartion

##### == STEP BY STEP COMPLETION OF Deploying a Basic OpenStack Setup (All-in-One Node) == ########
[] PREQ:
+ CentOS 7 - 9 OS
+ At least 4 GB RAM, 20 GB Disk Space, 2 CPU cores

Step 1: 
update system package::
```
sudo yum update -y
```
Step 2: 
Temporarily Disable SELinux and Firewalld (NOTE: DO NOT DO THIS IN PRODUCTION!!)
