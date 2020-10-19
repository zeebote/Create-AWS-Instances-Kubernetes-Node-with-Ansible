# Create-AWS-Instances-Kubernetes-Node-with-Ansible
This Ansible script use to create AWS instances then join them into existing Kubernetes cluster. <br>
It is also group new the instances to dynamic groups using tags with aws dynamic inventory plugin .<br>
**Requirements:**
1. amazon.aws.ec2 plugin - Please follow this [link](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_module.html#synopsis) for information and requirements
1. AWS SDK for Python (boto and boto3) - Please follow this [link](https://aws.amazon.com/sdk-for-python/) for installation and information about AWS SDK for Python
1. AWS account with access key to your EC2 - You can use an existing account or create a new one with AWS IAM console then go to manage access key and generate 
a new key. This account must have IAM role which have minimum policy to create instances in EC2. Please follow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) for information how to create IAM role. You also need a key-pair which is used for ssh to instances. Please follwow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair) for how to create a key pair on ec2 console.<br>

**How to use**
1. Setup virtual env and install requirement packages
   ```
   python3 -m venv ansible-env
   cd ansible-env
   source bin/activate
   pip3 install AWSCLI boto boto3 ansible
   ```
   
1. Get files from this repo
   ```
   git clone https://github.com/zeebote/Create-AWS-Instances-Kubernetes-Node-with-Ansible
   cd Create-AWS-Instances-Kubernetes-Node-with-Ansible/
   ```
1. Configure AWSCLI credential (this is used by EC2 dynamic inventory)
   ```
   aws configure
   AWS Access Key ID [None]:Your AWS Access Key
   AWS Secret Access Key [None]: Your AWS Secret Access Key 
   Default region name [None]: us-east-1
   Default output format [None]: json
   ```
1. Update vault.yml (This is used by playbook)
   ```vi ./staging/group_vars/all/vault.yml```
   Update with proper info
   ```
   ---
   # staging/group_vars_all/vault.yml
   vault_join_command: "This is out put of 'kubeadm token create --print-join-command' on your cluster master
   vault_ec2_access_key: "This is your IAM access key"
   vault_ec2_secret_key: "This is above secret"
   ```
1. Encrypt the vault file
   ``` 
   ansible-vault encrypt statging/group_vars/all/vault.yml
   New Vault password:
   Confirm New Vault password:
   Encryption successful
   ```
1. Download the private key of the key pair create on step 3 and update "ec2_keypair" variable in staging/group_vars/all/vars.yml with proper info. <br>
   Folder structure:
   ```
   .
   ├── ec2-instances-keypair.pem # This is private key of IAM key-pair
   ├── ansible.cfg               # Ansible configure file
   ├── ec2_provision.yml         # Provisioning instances playbook
   ├── log                       # Log
   ├── README.md                 # Readme.md from this repo
   ├── roles
   │   ├── docker                # Docker role folder
   │   │   ├── handlers
   │   │   │   └── main.yml      # handlers job
   │   │   └── tasks
   │   │       └── main.yml      # docker role main tasks
   │   └── kubernetes            # Kubernetes node role folder
   │       ├── handlers
   │       │   └── main.yml
   │       ├── meta
   │       │   └── main.yml      # Kubernetes dependentcy
   │       └── tasks
   │           └── main.yml      # Kubernetes role main tasks
   └── staging
       ├── aws_ec2.yml           # EC2 dynamic inventory setting
       └── group_vars
           └── all              
               ├── vars.yml      # Variable for all groups
               └── vault.yml     # Vault file - encrypted
   ```
1. Inventory before deploy the provision playbook
   ```
   ansible-inventory --graph
   @all:
   |--@Department_Engineering:
   |  |--13.56.179.70
   |  |--13.57.189.207
   |--@Kubernetes_Master:
   |  |--13.56.179.70      # Master node
   |--@Kubernetes_Node:
   |  |--13.57.189.207     # Agent node
   |--@aws_ec2:
   |  |--13.56.179.70
   |  |--13.57.189.207
   |--@ungrouped:
   ```
   notice that we have 2 instances on this staging, one in Kubernetes_Master and one in Kubernetes_Node group
   
1. Run the playbook to create instances and join them to cluster
   ```
   ansible-playbook ec2_provision.yml --ask-vault-pass
   ```
1. Inventory after deploy ec2_provision.yml
   ```
   @all:
   |--@Department_Engineering:
   |  |--13.56.179.70
   |  |--13.57.189.207
   |  |--18.144.39.142     
   |  |--54.193.76.87
   |  |--54.215.252.174
   |--@Kubernetes_Master:
   |  |--13.56.179.70
   |--@Kubernetes_Node:
   |  |--13.57.189.207
   |  |--18.144.39.142     # new instance added
   |  |--54.193.76.87      # new instance added
   |  |--54.215.252.174    # new instance added
   |--@aws_ec2:
   |  |--13.56.179.70
   |  |--13.57.189.207
   |  |--18.144.39.142
   |  |--54.193.76.87
   |  |--54.215.252.174
   |--@ungrouped:
   ```
1. Test connection
   ```
   ansible Kubernetes_Node -m ping --ask-vault-pass
   Vault password: 
   54.215.252.174 | SUCCESS => {
      "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
   }
   54.193.76.87 | SUCCESS => {
      "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
   }
   13.57.189.207 | SUCCESS => {
      "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
   }
   18.144.39.142 | SUCCESS => {
      "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
   }
1. Check if new nodes are online and ready in Kubernetes cluster
   ```
   ubuntu@ip-172-31-31-19:~$ kubectl get node
   NAME               STATUS   ROLES    AGE     VERSION
   ip-172-31-16-12    Ready    <none>   7m8s    v1.18.3
   ip-172-31-18-155   Ready    <none>   21h     v1.18.3
   ip-172-31-24-207   Ready    <none>   7m7s    v1.18.3
   ip-172-31-26-216   Ready    <none>   6m55s   v1.18.3
   ip-172-31-31-19    Ready    master   23h     v1.18.3
   ubuntu@ip-172-31-31-19:~$ 
   ```
   
