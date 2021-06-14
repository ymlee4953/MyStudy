# **001 - Ansible Setup**
          
          ## **OS update**
          - update CentOS to latest
            d
          
          ## **Environment**
          - d
            - d
          
          ## **Steps** 
          
          ### **Step 1: OS update**
          -  update CentOS to latest
            - d
              - f
          
          1. ddd
          
              1.1. ddd 
          
              - Check initial version
          
                    cat /etc/centos-release
          
              - Update to the latest
          
                    sudo yum update
          
              - Check updated version
          
                    cat /etc/centos-release
          
              - Restart
          
                    sudo reboot
          
              - Done
          
          
          
          ### **Step 2: Install Ansible**
          -  Install Ansible stabel and latest
            - d
              - f
          
          1. ddd
          
              1.1. ddd 
          
              - Configure EPEL repository and check 
          
                    sudo yum install -y epel-release
          
                    yum repolist
          
              - Install Ansible & check the version
          
                    sudo yum install -y ansible
          
                    ansible --version
          
              - Done
          
          
          
          ### **Step 3: Create SSH key**
          -  create SSH-key to login without password
            - d
              - f
          
          1. ddd
          
              1.1. ddd 
          
              - Create SSH_key 
          
                    ssh-keygen
          
                    cat /root/.ssh/id_rsa.pub
          
              - Done
          
          
          
          
          
          ---
          # **End**
          
          
