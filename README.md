# Create-AWS-Instances-Kubernetes-Node-with-Ansible
This Ansible script use to create AWS instances then join them into existing Kubernetes cluster
**Requirements:**
1. amazon.aws.ec2 plugin - Please follow this [link](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_module.html#synopsis) for information and requirements
1. AWS SDK for Python (boto or boto3) - Please follow this [link](https://aws.amazon.com/sdk-for-python/) for installation and information about AWS SDK for Python
1. AWS account with access key to your EC2 - You can use an existing account or create a new one with AWS IAM console then go to manage access key and generate 
a new key. This account must have IAM role which have minimum policy to create instances in EC2. Please follow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) for information how to create IAM role. You also need a key-pair which is used for ssh to instances. Please follwow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair) for how to create a key pair on ec2 console.<br>
**How to use**
1. Setup environment
   - Create virtual env and install requirement packages
      ```
      python3 -m venv ansible-env
      cd ansible-env
      source bin/activate
      pip3 install boto3 ansible
      ```
   
