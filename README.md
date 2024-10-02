# OpenStack-Setup-and-Testing
In this project ill be completing 3 tasks:

- Deploying a Basic OpenStack Setup (All-in-One Node)
- Setting up a Multi-node OpenStack Deployment (ENV)
- Deploying a highly available Openstack ENV and using Automation with Heat Orchestartion

##### == STEP BY STEP COMPLETION OF Deploying a Basic OpenStack Setup (All-in-One Node) == ########
[] PREQ:
+ CentOS Stream 9 OS
+ At least 4 GB RAM, 20 GB Disk Space, 2 CPU cores

### Preperation and Setup ===

Step 1: .
Update system package::
```
sudo yum update -y
```
Step 2: .
Temporarily Disable SELinux and Firewalld (NOTE: DO NOT DO THIS IN PRODUCTION!!)
![OS #1](https://github.com/user-attachments/assets/12aad4d5-11eb-47eb-a10a-06f5f955c69a)

Step 3: .
When configuring the network settings, make sure the IP is static

### Installing Packstack and OpenStack ===

Step 1: .
Install the OpenStack Repo zed::
![OS #2](https://github.com/user-attachments/assets/65f312b5-bec2-4b2e-8e59-ca353de21b1b)

Step 2: .
Install Packstack::
![2024_10_02_0wp_Kleki](https://github.com/user-attachments/assets/34cd10d6-a95c-4918-afe7-7b973ed99495)

Step 3: .
Now run the Pack installer. This will iniate an automated deployment of all core OpenStack Components. (Command is below):

```
sudo packstack --allinone
```
This will take 15-30minutes to complete..

![2024_10_02_0ww_Kleki](https://github.com/user-attachments/assets/786d2154-5986-4a7c-8ffe-3d32980bb34a)

