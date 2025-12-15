##### Ansible Automated Configuration \& Patch Management Project

##### 

###### 1\. What is the Project?

Instead of manually configuring each Linux server individually for this project, I utilized Ansible to automatically configure and manage a collection of servers.  Three Ubuntu servers hosted on AWS EC2 free-tier instances and one Ansible control node operating on my local Ubuntu system via WSL make up the environment.  The project's objective was to develop a completely automated setup that could apply default system configurations, install necessary services and packages, install SSH keys, enhance server security through baseline hardening, and carry out routine operating system updates.  The system also takes care of rebooting if updates call for it.  This project's use of Infrastructure-as-Code guarantees that every server is set up uniformly and can be readily changed or maintained from a single control point.



###### 2\. What Problem Does It Solve?

Manually maintaining several servers takes a lot of effort, is tedious, and frequently results in setup problems.  Inconsistent environments and security threats from out-of-date packages might arise from tasks like manually copying SSH keys, configuring servers one at a time, and neglecting to apply updates.  As the number of systems increases, manually configuring a server does not scale well and can take hours.  By centrally managing SSH keys, applying the same baseline configuration to each server, and setting up automatic operating system updates, this project employs Ansible automation to address these problems.  Because of this, servers can be swiftly, reliably, and securely deployed and maintained without requiring frequent manual logins.


###### 3\. How Is It Being Solved? (Architecture \& Automation)

###### 



&nbsp;  The solution is built using Ansible inventories, roles, and scheduled automation.





&nbsp;  A. Inventory Structure (Static Inventory)



&nbsp;  The project uses a static hosts.ini inventory with 3 AWS EC2 instances:



&nbsp;  inventories/

&nbsp;  └── prod/

&nbsp;       └── hosts.ini





&nbsp;   Example:



&nbsp;   \[allnodes]

&nbsp;   host1 ansible\_host=44.200.103.209 ansible\_user=ubuntu ansible\_ssh\_private\_key\_file=/home/maver/.ssh/ansible-key.pem

&nbsp;   host2 ansible\_host=34.204.189.251 ansible\_user=ubuntu ansible\_ssh\_private\_key\_file=/home/maver/.ssh/ansible-key.pem

&nbsp;   host3 ansible\_host=35.170.74.153 ansible\_user=ubuntu ansible\_ssh\_private\_key\_file=/home/maver/.ssh/ansible-key.pem





&nbsp;   This tells Ansible how to connect to each EC2 host.







&nbsp;  B. Roles Implemented in This Project



&nbsp;  Three Ansible roles are used to organize the automation:



&nbsp;  roles/

&nbsp;  ├── ssh\_keys/

&nbsp;  ├── baseline/

&nbsp;  └── patching/



&nbsp;   1. ssh\_keys Role



&nbsp;   Purpose:

&nbsp;   Public SSH keys are automatically deployed into each managed node from the keys/folder.



&nbsp;   Function:

By reading public keys from the keys/*.pub directory and verifying that the necessary users are present on each host, this project component automates SSH key management.  Next, it prevents duplicate keys from being added by updating each user's ~/.ssh/authorized_keys file in a controlled and idempotent manner.  This method makes it simple to add or delete access across all servers from a single location and eliminates the need for manual key copying by centralizing SSH key management.



&nbsp;   2. baseline Role



&nbsp;   Purpose:

&nbsp;   All servers should have the same configuration.



&nbsp;    Functions include:



&nbsp;    Establish baseline users



&nbsp;    Set up sudo access.



&nbsp;    Install the necessary packages (git, curl, vim, etc.).



&nbsp;    Set up NTP/Chrony



&nbsp;    Turn on the UFW firewall



&nbsp;    Use packages for log rotation.



&nbsp;    Maintain a stable system state



&nbsp;    This establishes a secure and consistent environment.





&nbsp;    3. patching Role



&nbsp;    Purpose:

&nbsp;    Update OS packages automatically and only restart when necessary.



&nbsp;     Features:



&nbsp;     Update the lists of apt packages.



&nbsp;     Install security updates.



&nbsp;     Determine whether a reboot is necessary.



&nbsp;     Restart the host securely



&nbsp;     Make sure hosts are constantly current.



&nbsp;     This eliminates the need for manual server maintenance while maintaining server security.



C. Playbooks Used

playbooks/

├── site.yml        -> Full baseline configuration

├── update.yml      -> Run patch cycle only

└── keys.yml        -> Deploy SSH keys only





Depending on the use case, these enable either partial or complete automation.





D. Automation With Cron (Scheduled Updates)



&nbsp;  A cron job was added on the control node to run patch updates weekly:



&nbsp;  "0 2 \* \* 0 ansible-playbook /home/maver/ansible-project/playbooks/update.yml -i /home/maver/ansible-project/inventories/prod/hosts.ini"





&nbsp;  This means:



&nbsp;  All EC2 hosts get updated with zero manual effort on Every Sunday at 2 AM



E. Validation



&nbsp;  To confirm everything works:



&nbsp;  1. Connectivity Test

&nbsp;     ansible -m ping all



&nbsp;  2. Full Setup Deployment

&nbsp;     ansible-playbook site.yml



&nbsp;  3. Only Keys Deployment

&nbsp;     ansible-playbook keys.yml



&nbsp;  4. Patch Cycle Test

&nbsp;     ansible-playbook update.yml





Outputs showed:



&nbsp;  Keys deployed



&nbsp;  Baseline configured



&nbsp;  Updates applied across all hosts



###### 4\. Repository Structure



ansible-project/

│

├── inventories/

│   └── prod/

│       └── hosts.ini

│

├── group\_vars/

│

├── roles/

│   ├── ssh\_keys/

│   ├── baseline/

│   └── patching/

│

├── playbooks/

│   ├── site.yml

│   ├── keys.yml

│   └── update.yml

│

├── keys/

&nbsp;    └── \*.pub





