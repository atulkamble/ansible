```
// ec2 - ansible

1. ansible.pem
2. default-vpc
3. default-SG
4. launch 3 instances (t3.medium) - amazon linux 
>> 1 - server - ansible-server
>> 2 - hosts 
host1
host2
5. Auto-Assign-Public-IP ON
6. SSD - 50GB

sudo yum update -y
sudo yum install python python3-pip tree -y
sudo pip install ansible
OR
sudo yum search ansible
sudo yum install ansible.noarch -y
ansible --version
cd /etc/ansible/

7. /etc/ansible
tree

├── ansible.cfg
├── aws
│   └── ansible.pem
├── inventory
│   └── hosts
├── playbooks
│   └── ping.yml
└── roles

sudo touch ansible.cfg

sudo nano ansible.cfg 

[defaults]
inventory = ./inventory
remote_user = ec2-user
private_key_file= aws/ansible.pem
host_key_checking=False
retry_files_enabled=False

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false


sudo mkdir playbooks
sudo touch playbooks/ping.yml

sudo mkdir inventory
sudo touch inventory/hosts
sudo nano inventory/hosts 

[all]
18.207.165.57
34.229.47.230

sudo mkdir aws
sudo nano aws/ansible.pem

sudo chmod 400 aws/ansible.pem 

sudo ansible -m ping all

sudo cp /etc/ansible/aws/* /home/ec2-user/aws/

cd /home/ec2-user/aws
touch ansible.pem
```
