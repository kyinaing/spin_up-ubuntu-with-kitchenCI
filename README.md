# spin_up-ubuntu-with-kitchenCI
Here's a step-by-step guide to setting up a lab environment for spinning up an Ubuntu machine using Chef KitchenCI:

---

# What is KitchenCI
Chef KitchenCI, commonly referred to as just "KitchenCI" or "Test Kitchen," is a powerful open-source framework and tool for automated testing and validation of infrastructure code. It is an essential component of the Chef ecosystem and is widely used by DevOps teams and infrastructure engineers.

KitchenCI provides a platform-agnostic testing framework, allowing you to test infrastructure code across different platforms, such as Linux, Windows, and macOS. It integrates seamlessly with popular testing frameworks like InSpec and Serverspec, enabling comprehensive and automated testing of infrastructure configurations.

The primary purpose of Chef KitchenCI is to automate the process of testing and validating infrastructure code, including Chef cookbooks and associated configurations. It ensures that the desired state of the infrastructure is accurately reflected by the code and that changes can be confidently deployed to production environments.

---

# How to download KitchenCI

[Click Here](https://community.chef.io/downloads/tools/workstation) to download and install Chef Workstation on your laptop.
Verify the installation, run:
```
chef -v
```
```
Chef Workstation version: 23.5.1040
Chef InSpec version: 5.21.29
Chef CLI version: 5.6.11
Chef Habitat version: 1.6.652
Test Kitchen version: 3.5.0
Cookstyle version: 7.32.2
Chef Infra Client version: 18.2.7
```

Chef Workstation installs everything you need to get started using Chef products on Windows, Mac and Linux. It includes:
- Chef Workstation App
- Chef Infra Client
- Chef InSpec
- Chef Command Line Tool
- Test Kitchen
- Cookstyle
- Various Test Kitchen and Knife plugins for clouds

---

# Spin up ubuntu machine using kitchen

[Click Here](https://kitchen.ci/) to read what is KitchenCI

#Demo
- Create a folder chef-aws under your $HOME
`cd $HOME
 mkdir chef-aws && cd chef-aws
 mkdir cookbooks
 chef generate cookbook cookbooks/node-ubuntu2204-aws-sg`
 
 ![image](https://github.com/kyinaing/spin_up-ubuntu-with-kitchenCI/assets/12751896/ac780e24-9807-4748-a38f-9a763e5c11ba)

---
## Step for AWS

- Before edit the kitchen.yml, you should create and get the information from AWS Console for following resources,
    - AWS Subnet ID
    - AWS Security Group ID
    - AWS Keypair Name
    - AWS Keypair file location

## Update the Kitchen.yml file

- Update kitchen.yml as below

```
---
driver:
  name: ec2
  aws_ssh_key_id: chefkitchen-demo #your_aws_ssh_key_here
  region: ap-southeast-1 
  subnet_id: subnet-039021dd036a889df # your_subnet
  security_group_ids: sg-063bf9b689a9c8fbe # your_security_group
  associate_public_ip: true # public subnet (not isolated, not private)
  instance_type: t2.micro
  tags:
    Name: "node-ubuntu2204-aws-sg" #node-ubuntu1804-aws-sg
    User: Administrator
    X-Contact: "Administrator" #your_name
    X-Application: "TESTAPP" #your_app_name
    X-Dept: "CONSULTING" #your_dept
    X-Customer: "Test"
    X-Project: "AWS-Chef-Demo"
    X-Termination-Date: "2020-06-28T12:00:30Z"
    X-TTL: 2

provisioner:
  name: chef_zero
  product_name: chef
  chef_license: accept
  
verifier:
  name: inspec
  format: documentation

platforms:
  - name: ubuntu-22.04
    driver:
      block_device_mappings:
        - device_name: /dev/sda1
          ebs:
            volume_type: gp2
            # virtual_name: test
            volume_size: 8
            delete_on_termination: true
    transport:
      username: ubuntu
      ssh_key: C:\tools\Keypair/chefkitchen-demo.pem #your_pem

suites:
  - name: default
    run_list:
      - recipe[node-ubuntu2204-aws-sg::default]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
```
---
## Run Kitchen Command from $HOME/chef-aws/cookbooks/node-ubuntu2204-aws-sg

```
PS C:\Cloud-Learning\chef-demo\cookbooks\node-ubuntu2204-aws-sg> kitchen list
Instance             Driver  Provisioner  Verifier  Transport  Last Action    Last Error
default-ubuntu-2204  Ec2     ChefInfra    Inspec    Ssh        <Not Created>  <None>
PS C:\Cloud-Learning\chef-demo\cookbooks\node-ubuntu2204-aws-sg> kitchen create
-----> Starting Test Kitchen (v3.5.0)
-----> Creating <default-ubuntu-2204>...
       Detected platform: ubuntu version 22.04 on x86_64. Instance Type: t2.micro. Default username: ubuntu (default).
       If you are not using an account that qualifies under the AWS
free-tier, you may be charged to run these suites. The charge
should be minimal, but neither Test Kitchen nor its maintainers
are responsible for your incurred costs.

       Instance <i-0fd964716649b71f8> requested.
       Polling AWS for existence, attempt 0...
       EC2 instance <i-0fd964716649b71f8> created.
       Waited 0/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 5/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 10/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 15/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 20/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 25/300s for instance <i-0fd964716649b71f8> to become ready.
       Waited 30/300s for instance <i-0fd964716649b71f8> to become ready.
       EC2 instance <i-0fd964716649b71f8> ready (hostname: ec2-18-141-219-223.ap-southeast-1.compute.amazonaws.com).
       Waiting for SSH service on ec2-18-141-219-223.ap-southeast-1.compute.amazonaws.com:22, retrying in 3 seconds
       [SSH] Established
       Finished creating <default-ubuntu-2204> (0m55.84s).
-----> Test Kitchen is finished. (1m0.73s)
```
  
## Try to Kitchen Login
  
```
  PS C:\Cloud-Learning\chef-demo\cookbooks\node-ubuntu2204-aws-sg> kitchen login
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'C:/tools/Keypair/chefkitchen-demo.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "C:/tools/Keypair/chefkitchen-demo.pem": bad permissions
ubuntu@ec2-18-141-219-223.ap-southeast-1.compute.amazonaws.com: Permission denied (publickey).
```

## If bad permissions issue occurred, pls change permission as below
  

  - For Linux Machine
```
  Open an SSH client.

Locate your private key file. The key used to launch this instance is chefkitchen-demo.pem

Run this command, if necessary, to ensure your key is not publicly viewable.
 chmod 400 chefkitchen-demo.pem

Connect to your instance using its Public DNS:
 ec2-18-141-219-223.ap-southeast-1.compute.amazonaws.com
```

- For Windows Machine

    - select .pem file -> right click -> properties
    - Security > Advanced > Disable inheritance
    - Remove all Users
    - Add > Select a principal
    - In "Enter the object name to select" type your Windows username > ok
    - Give all permissions > ok > apply

  ![image](https://github.com/kyinaing/spin_up-ubuntu-with-kitchenCI/assets/12751896/43bfb410-d413-42e4-80a6-641792012df1)

 - And login again with Kitchen login
  
  ```
  PS C:\Cloud-Learning\chef-demo\cookbooks\node-ubuntu2204-aws-sg> kitchen login
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.19.0-1026-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jun 10 17:45:41 UTC 2023

  System load:  0.0048828125      Processes:             96
  Usage of /:   20.9% of 7.57GB   Users logged in:       0
  Memory usage: 26%               IPv4 address for eth0: 172.31.64.111
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sat Jun 10 17:40:40 2023 from 49.228.233.87
ubuntu@ip-172-31-64-111:~$
```
---

# Delete Kitchen Resource
The bottom line when comes to cloud is shut it down if we don't need it.
**kitchen destroy**
```
PS C:\Cloud-Learning\chef-demo\cookbooks\node-ubuntu2204-aws-sg> kitchen destroy
-----> Starting Test Kitchen (v3.5.0)
-----> Destroying <default-ubuntu-2204>...
       EC2 instance <i-08e9b9182fcf799a5> destroyed.
       Finished destroying <default-ubuntu-2204> (0m0.86s).
-----> Test Kitchen is finished. (0m6.14s)
```
