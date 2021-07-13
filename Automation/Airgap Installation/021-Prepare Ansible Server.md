# **021-Setup Ansible Server**

## 

### **Step 1: OS update**

1. update CentOS to latest

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

1. Install Ansible stabel and latest

    - Configure EPEL repository and check 

          sudo yum install -y epel-release

          yum repolist

    - Install Ansible & check the version

          sudo yum install -y ansible

          ansible --version

    - Done


---