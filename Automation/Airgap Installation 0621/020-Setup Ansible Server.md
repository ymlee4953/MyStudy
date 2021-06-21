# **020. Setup Ansible Server**

1. OS update
          
    1.1. Setup CentOS 7.9 
          
    - Check initial OS version
          
          cat /etc/centos-release
          
    - Update to the latest
          
          sudo yum update
          
    - Check updated version
          
          cat /etc/centos-release
          
    - Restart
          
          sudo reboot
          

2. Install Ansible 
          
    2.1. Install stable and latest version
          
    - Configure EPEL repository and check 
          
          sudo yum install -y epel-release
          
          yum repolist
          
    - Install Ansible & check the version
          
          sudo yum install -y ansible
          
          ansible --version
          
         
          
3. create SSH-key
          
    3.1. create SSH-key to login without password
                   
    - Create SSH_key 
          
          ssh-keygen
          
          cat /root/.ssh/id_rsa.pub
          
                
---
# **End**
          
                              