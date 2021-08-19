# ansible-jenkins


Ansible–Steps.

Pre-requisites


1.	Install Ansible plugin in for EC2(AWS) -- collection install amazon.aws.
2.	Install boto3 for Dynamic inventory for EC2.


Structure 

 


Dynamic Inventory

plugin: aws_ec2
aws_access_key: AKIAYW213324123OMMIAFX. <<access-key>>
aws_secret_key: f-e&P#D-fu899yrz34I4321123$a2343##!esxYC84 <<secret key>>

regions:
  - us-east-2

filters:
  tag:LOB:
    - EC2-Dev


Execute the below command to list the Hosts

$ansible-inventory -i aws_ec2.yaml --list
  
    "aws_ec2": {
        "hosts": [
            "ec2-3-141-34-216.us-east-2.compute.amazonaws.com"
        ]
    }
}

Sites.yml – roles 
---
- hosts: aws_ec2
  gather_facts: false
  remote_user: ubuntu
  vars:
    ansible_ssh_private_key_file:  ./aws-tiny.pem
  
  roles:
    - jenkins
  tags: jenkins

Variables – vars – main.yml - 
---
# vars file for Java

java:
  VERSION: "openjdk-11-jdk"


---
- name: Install Java 8 .
 apt:
    name: "{{java.VERSION}}"
    state: present
    install_recommends: no
  become: true
  
- name: get Java version
  shell: java --version 2>&1
  register: java_output
    
- debug: 
    var=java_output.stdout_lines

- name: Install Python
  raw: apt -y update && apt install -y python3
  become: true

- name: get Python version
  shell: python3 --version 2>&1
  register: python_output
    
- debug: 
    var=python_output.stdout_lines

- name: ensure the jenkins apt repository key is installed
  apt_key: url=https://pkg.jenkins.io/debian-stable/jenkins.io.key state=present
  become: yes

- name: ensure the repository is configured
  apt_repository: repo='deb https://pkg.jenkins.io/debian-stable binary/' state=present
  become: yes

- name: ensure jenkins is installed
  apt: name=jenkins update_cache=yes
  become: yes

- name: ensure jenkins is running
  service: name=jenkins state=started

- name: Retrieve Jenkins unlock code
  shell: "cat /var/lib/jenkins/secrets/initialAdminPassword"
  register: jenkins_unlock
  become: yes
- debug: msg="Jenkins unlock code (install admin password) is {{ jenkins_unlock.stdout }}"


Output of Ansible - playbook

JCPSI70WK:EC2Terraform root# ansible-playbook -i aws_ec2.yaml sites.yml

PLAY [aws_ec2] ********************************************************************************************************************************************************

TASK [jenkins : Install Java 8 .] *************************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : get Java version] *************************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : debug] ************************************************************************************************************************************************
ok: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com] => {
    "java_output.stdout_lines": [
        "openjdk 11.0.11 2021-04-20",
        "OpenJDK Runtime Environment (build 11.0.11+9-Ubuntu-0ubuntu2.20.04)",
        "OpenJDK 64-Bit Server VM (build 11.0.11+9-Ubuntu-0ubuntu2.20.04, mixed mode, sharing)"
    ]
}

TASK [jenkins : Install Python] ***************************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : get Python version] ***********************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : debug] ************************************************************************************************************************************************
ok: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com] => {
    "python_output.stdout_lines": [
        "Python 3.8.10"
    ]
}

TASK [jenkins : ensure the jenkins apt repository key is installed] ***************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : ensure the repository is configured] ******************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : ensure jenkins is installed] **************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : ensure jenkins is running] ****************************************************************************************************************************
ok: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : Retrieve Jenkins unlock code] *************************************************************************************************************************
changed: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com]

TASK [jenkins : debug] ************************************************************************************************************************************************
ok: [ec2-3-141-34-216.us-east-2.compute.amazonaws.com] => {
    "msg": "Jenkins unlock code (install admin password) is 0d34c7a502d2424185a352425e0ddde9"
}

PLAY RECAP ************************************************************************************************************************************************************
ec2-3-141-34-216.us-east-2.compute.amazonaws.com : ok=12   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Jenkins – output.

 

