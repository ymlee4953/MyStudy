

1. Declare Cluster Information

    1.1. export variable with value

    - cluster information and server IP

          export K8S_CLUSTER_NAME=auto_cluster_ab
          export CLUSTER_DOMAIN=ab.k8s.demo.ymlee
          export K8S_VERSION=v1.20.5
          export POD_SUBNET=192.168.0.0/16
          export SERVICE_SUBNET=20.96.0.0/16          

          export ETCD_1=10.168.180.135
          export ETCD_2=10.168.180.220
          export ETCD_3=10.168.180.186

          export LB_1=10.168.180.173

          export MASTER_1=10.168.180.162
          export MASTER_2=10.168.180.170
          export MASTER_3=10.168.180.137

          export LB_2=10.168.180.195

          export INGRESS_1=10.168.180.169
          export INGRESS_2=10.168.180.254
          export INGRESS_3=10.168.180.158

          export INFRA_1=10.168.180.235

          export WORKER_1=10.168.180.148
          export WORKER_2=10.168.180.160
          export WORKER_3=10.168.180.212

          export BASTION_0=10.168.180.214
          export NEXUS_0=10.168.180.184

2. Create Directoy for new Cluster

    2.1. Create Directory

    - cluster information and server IP

          mkdir -p $HOME/ansible-playbooks
          mkdir -p $HOME/configurations
          mkdir -p $HOME/files


3. Create Ansible Inventory file

    - Ansible Inventory file

          cat <<EOF> k8s-cluster-hosts
          # k8s-cluster-hosts
                             
          [etcd]
          ETCD-1 ansible_host=${ETCD_1}
          ETCD-2 ansible_host=${ETCD_2}
          ETCD-3 ansible_host=${ETCD_3}

          [lbControlPlane]
          LB-1 ansible_host=${LB_1}

          [master]
          MASTER-1 ansible_host=${MASTER_1}
          MASTER-2 ansible_host=${MASTER_2}
          MASTER-3 ansible_host=${MASTER_3}

          [lbApplication]
          LB-2 ansible_host=${LB_2}

          [ingress]
          INGRESS-1 ansible_host=${INGRESS_1}
          INGRESS-2 ansible_host=${INGRESS_2}
          INGRESS-3 ansible_host=${INGRESS_3}

          [infra]
          INFRA-1 ansible_host=${INFRA_1}

          [worker]
          WORKER-1 ansible_host=${WORKER_1}
          WORKER-2 ansible_host=${WORKER_2}
          WORKER-3 ansible_host=${WORKER_3}

          [shared]
          BASTION-0 ansible_host=${BASTION_0}
          NEXUS-0 ansible_host=${NEXUS_0}

          [local]
          control ansible_connection=local

          EOF

          cat k8s-cluster-hosts
          cp k8s-cluster-hosts k8s-cluster-hosts-$(date '+%Y%m%d'-'%H%M%S')

4. Generate SSH Key and set login without password

    4.1 Generate SSH-Key 
    
    - Generate SSH-Key 

          ssh-keygen

          cat /root/.ssh/id_rsa.pub


    4.2 Copy SSH-Key to Server
    - Copy

          cat <<EOF> ssh-key-copy.txt
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${LB_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${LB_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${INGRESS_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${INGRESS_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${INGRESS_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${INFRA_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${BASTION_0}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${NEXUS_0}
          EOF

          cat ssh-key-copy.txt
          rm -rf ssh-key-copy.txt


### **Step 4: Create Password Yaml**
-  I
  - d
    - f

1. ddd

    1.1. ddd 

    - Configure EPEL repository and check 

          yum install python-passlib


          mkdir -p ~/ansible-playbooks/initialize

          cat <<EOF> ~/ansible-playbooks/initialize/change-password.yml
          ---
          - hosts: etcd:master:worker:ingress:infra:lb*
            become: yes
            tasks:
              - name: Change user password
                user:
                  name: root
                  update_password: always
                  password: "{{ newpassword|password_hash('sha512') }}"
          EOF

          cat ~/ansible-playbooks/initialize/change-password.yml

    - change password

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/initialize/change-password.yml --extra-vars newpassword=12345678
         
    - Done
