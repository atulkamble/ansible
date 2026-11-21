# 21-11-2026 
```
// Ansible - Configuration Management 

repetitions - unique 
automation 
remote connect

RHEL 

app >> OS 

vpc 

ansible server - host1, host2... 50


50vms >> apache



// Ansible setup 

2 hosts
1 server

Amazon Linux - ansible.pem | 60GB SSD | t3.medium / c7.iflex.large

server >>

sudo yum update -y 
sudo yum install python python3-pip tree -y 
sudo yum search ansible 
sudo yum install ansible.noarch -y 
ansible --version 

cd /etc/ansible 

sudo mkdir inventory
cd inventory 
sudo touch hosts 
sudo nano hosts 

// 
[all]
host1-public-ip
host2-public-ip

cd ..
sudo mkdir playbooks
cd playbooks
sudo touch ping.yml
cd ..

sudo touch ansible.cfg 
sudo nano ansible.cfg 

[defaults]
inventory = ./inventory/hosts
remote_user = ec2-user
private_key_file = ./aws/ansible.pem
host_key_checking = False
retry_files_enabled = False
interpreter_python = auto_silent

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

sudo mkdir aws 
cd aws
sudo touch ansible.pem 
sudo nano ansible.pem

// paste ansible.pem code 

tree

├── ansible.cfg
├── aws
│   └── ansible.pem
├── inventory
│   └── hosts
├── playbooks
│   └── ping.yml
└── roles

sudo chmod 400 aws/ansible.pem

sudo ansible -m ping all 
```

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
