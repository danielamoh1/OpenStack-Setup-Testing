# OpenStack-Setup-and-Testing
In this project ill be completing 3 tasks:

- Deploying a Basic OpenStack Setup (All-in-One Node)✅
- Setting up a Multi-node OpenStack Deployment (ENV)
- Deploying a highly available Openstack ENV and using Automation with Heat Orchestartion

```mermaid
flowchart TD;

    %% Beginner Project: Single-Node OpenStack Setup
    subgraph Beginner_Project
        direction TB
        A1[Prepare CentOS] --> B1[Install Packstack]
        A1 --> C1[Run Packstack with All-in-One]
        B1 --> D1[Configure Neutron Networking]
        C1 --> E1[Set Up External Network]
        D1 --> F1[Create Internal Network and Subnet]
        E1 --> G1[Upload Image to Glance]
        F1 --> H1[Create VM Flavor]
        G1 --> I1[Launch Instance]
        H1 --> J1[Associate Floating IP]
    end

    %% Intermediate Project: Multi-Node OpenStack Setup
    subgraph Intermediate_Project
        direction TB
        A2[Prepare Multiple CentOS Nodes] --> B2[Install OpenStack on Controller Node]
        A2 --> C2[Install OpenStack on Compute Nodes]
        B2 --> D2[Configure Neutron with VLAN/VXLAN]
        C2 --> E2[Run Packstack with Multi-Node Answer File]
        D2 --> F2[Create Networks, Flavors, and Images]
        E2 --> G2[Launch Instances on Compute Nodes]
        F2 --> H2[Associate Floating IPs]
    end

    %% Advanced Project: High Availability & Orchestration
    subgraph Advanced_Project
        direction TB
        A3[Prepare Controller and Compute Nodes for HA] --> B3[Configure HAProxy for Load Balancing]
        A3 --> C3[Set Up Galera Cluster for DB HA]
        B3 --> D3[Set Up RabbitMQ Cluster for Message Queueing HA]
        C3 --> E3[Configure OpenStack Services in HA]
        D3 --> F3[Deploy Heat for Orchestration]
        E3 --> G3[Create Heat Template for Auto-Scaling]
        F3 --> H3[Set Up Centralized Logging & Monitoring]
    end

    %% Connecting Projects Together
    Beginner_Project --> Intermediate_Project
    Intermediate_Project --> Advanced_Project
```

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

- I stopped and disabled the `NetworkManger` service
- I ran the `less /var/tmp/packstack/20241002-194116-lj_a_ufq/manifests/10.0.0.151_controller.pp.log` command to get more context and information on what exactly was causing the issue
- 2 things...that I read from the errors that could be potential fixes: First, Openstack-glance was failing to install. Second, python3-pyxattr was struggling to be installed during the automation sequence
- Action to fix: First, I ran the command `dnf config-manager --set-enabled crb` to enable the crb repo. Next, I ran the command `dnf clean all && dnf makecache` to rebuild DNF Cache and update repos. Next, I did a manual installation of pyxattr by using the command `dnf install -y python3-pyxattr`. And then I ran the command `dnf install -y openstack-glance` to install the openstack-glance package now that pyxattr was available and successfully installed. Finally I reiniated the build by running the `packstack --answer-file /root/packstack-answers-20241002-194118.txt` command...and tested for success after the build was finished

Now there's Issues Authenticating:

![image](https://github.com/user-attachments/assets/e20bb93a-377d-4194-ac3d-7e8e4d1d7c8b)

Could Stem from this error:

![2024_10_02_12b_Kleki](https://github.com/user-attachments/assets/06cb96c0-956d-4cd3-86a1-a4075dc60fda)

Will TroubleShoot

SUCCESS!:

![2024_10_03_0j4_Kleki](https://github.com/user-attachments/assets/597b9651-e1f7-4ab4-8a50-769019099cd2)


Lets quickly walk through how I Fixed this:

- I stopped the Apache service with the `systemctl stop httpd` command
- I resetted the passwd with the command: `keystone-manage bootstrap --bootstrap-password 'ENTER_NEW_PASSWD_HERE' \
    --bootstrap-admin-url http://10.0.0.151:5000/v3/ \
    --bootstrap-internal-url http://10.0.0.151:5000/v3/ \
    --bootstrap-public-url http://10.0.0.151:5000/v3/ \
    --bootstrap-region-id RegionOne`
- I updated the ~/keystonerc_admin dir with the new passwd then sourced the dir
- I then started the apache service with `systemctl start httpd`
- Then I retested the login with the new passwd

I expect to encounter More issues as I go along but thats part of the work so I expect it and have full confidence ill always figure it out

======== NOW ON TO THE NEXT TASK IN THIS PROJECT =========

##### == STEP BY STEP COMPLETION OF Setting up a Multi-node OpenStack Deployment (ENV) == ########

Since I already have the controller node built ill go ahead and configure the other two nodes within the packstack answers.txt file

![2024_10_03_13y_Kleki](https://github.com/user-attachments/assets/12bd8e83-3f4a-49a6-b032-303d051aa543)

now ill run the `packstack --answer-file=packstack-answers-20241002-194118.txt` command to deploy the multi-node env

NOTE: Make sure you have a passwordless SSH connection established to each node before deploying the env

DEPLOYING MULTI-NODE ENV:

![image](https://github.com/user-attachments/assets/51c0711b-f1ae-49f0-bfd7-6ba7089e5997)

WILL TAKE A WHILE (15 - 30 mins)

