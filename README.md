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
Update system packages::
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

Still going, Ill waiting for it to finish building::

![2024_10_02_0x2_Kleki](https://github.com/user-attachments/assets/9e488648-4b51-45ba-82bd-3788d2f54285)

Build completed with some errors, but lets test to see if the UI is still accessible with out any issues:

![2024_10_02_0z9_Kleki](https://github.com/user-attachments/assets/59add165-fbc4-47c7-8a01-40f2791df822)

CHECKING..:

![image](https://github.com/user-attachments/assets/b90b4124-5f50-4efe-bff0-23f186e95188)

UI Looks to be Inaccessible::

![image](https://github.com/user-attachments/assets/5f0b1973-9ec5-4777-b973-28dd3b66010d)

I'll TroubleShoot the Issue.

RESULT::

![2024_10_02_11f_Kleki](https://github.com/user-attachments/assets/ed366bbd-07d7-4e76-bfc6-4c66a3170783)

UI is now accessible..

Lets quickly walk through what I did in the Backend to fix this:

- I stopped and disable the `NetworkManger` service
- I ran the `less /var/tmp/packstack/20241002-194116-lj_a_ufq/manifests/10.0.0.151_controller.pp.log` command to get more context and information on what exactly was causing the issue
- 2 things...that I read from the errors that could be potential fixes: First, Openstack-glance was failing to install. Second, python3-pyxattr was struggling to be installed during the automation sequence
- Action to fix: First, I ran the command `dnf config-manager --set-enabled crb` to enable the crb repo. Next, I ran the command `dnf clean all && dnf makecache` to rebuild DNF Cache and update repos. Next, I did a manual installation of pyxattr by using the command `dnf install -y python3-pyxattr`. And then I ran the command `dnf install -y openstack-glance` to install the openstack-glance package now that pyxattr was available and successfully installed. Finally I reiniated the build by running the `packstack --answer-file /root/packstack-answers-20241002-194118.txt` command...and tested for success after the build was finished

Now there's Issues Authenticating:

![image](https://github.com/user-attachments/assets/e20bb93a-377d-4194-ac3d-7e8e4d1d7c8b)

Could Stem from this error:

![2024_10_02_12b_Kleki](https://github.com/user-attachments/assets/2b5b7681-484a-41f0-b550-1f7607db97fd)

Will TroubleShoot

SUCCESS!:

![2024_10_03_0j4_Kleki](https://github.com/user-attachments/assets/eef9b639-c3e5-4145-bcbd-70c73a69e8f0)

Lets quickly walk through how I Fixed this:

- I stopped the Apache service with the `systemctl stop httpd` command
- I resetted the passwd with the command: `keystone-manage bootstrap --bootstrap-password 'ENTER_NEW_PASSWD_HERE' \
    --bootstrap-admin-url http://10.0.0.151:5000/v3/ \
    --bootstrap-internal-url http://10.0.0.151:5000/v3/ \
    --bootstrap-public-url http://10.0.0.151:5000/v3/ \
    --bootstrap-region-id RegionOne`
- I updated the ~/keystonerc_admin dir with the new passwd then sourced the dir
- Then I retested the loging with the new passwd

I expect to encounter More issues as I go along but thats part of the work so I expect it and have full confidence ill always figure it out

======== NOW ON TO THE NEXT TASK IN THIS PROJECT =========

##### == STEP BY STEP COMPLETION OF Setting up a Multi-node OpenStack Deployment (ENV) == ########

