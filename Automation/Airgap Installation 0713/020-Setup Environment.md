# **020.Setup Environment**

- at the ansible Server

---


1. Declare Cluster Information

    1.1. export variable with value

    - cluster information and server IP
    - At the ansible Server

          export K8S_CLUSTER_TYPE=k8s_base
          export K8S_CLUSTER_SHORT=d0
          export K8S_CLUSTER_NAME=k8s_cluster_d0
          export CLUSTER_DOMAIN=d0.k8s.demo.ymlee
          export K8S_VERSION=v1.20.5
          export POD_SUBNET=192.168.0.0/16
          export SERVICE_SUBNET=20.96.0.0/16

          export ETCD_1=10.168.40.61
          export ETCD_2=10.168.40.49
          export ETCD_3=10.168.40.8

          export LB_1=10.168.40.2

          export MASTER_1=10.168.40.14
          export MASTER_2=10.168.40.37
          export MASTER_3=10.168.40.42

          export WORKER_1=10.168.40.39
          export WORKER_2=10.168.40.19
          export WORKER_3=10.168.40.58
          
          export GLUSTERFS_1=10.168.40.32
          export GLUSTERFS_2=10.168.40.51
          export GLUSTERFS_3=10.168.40.10

          export HEKETI_0=10.168.40.43

          export BASTION_0=10.168.40.9
          export NEXUS_0=10.168.40.11

2. Create Directoy for new Cluster

    2.1. Create Directory

    - At the ansible Server

          mkdir -p $HOME/ansible-playbooks
          mkdir -p $HOME/configurations
          mkdir -p $HOME/files

3. Create Ansible Inventory file

    3.1 Ansible Inventory file
    
    - K8s CLuster Type : base

          cat <<EOF> $HOME/k8s-cluster-hosts
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

          [worker]
          WORKER-1 ansible_host=${WORKER_1}
          WORKER-2 ansible_host=${WORKER_2}
          WORKER-3 ansible_host=${WORKER_3}

          [glusterfs]  
          HEKETI-0 ansible_host=${HEKETI_0} 
          GLUSTERFS-1 ansible_host=${GLUSTERFS_1}
          GLUSTERFS-2 ansible_host=${GLUSTERFS_2}
          GLUSTERFS-3 ansible_host=${GLUSTERFS_3}

          [shared]
          BASTION-0 ansible_host=${BASTION_0}
          NEXUS-0 ansible_host=${NEXUS_0}

          [local]
          control ansible_connection=local

          EOF

          cat $HOME/k8s-cluster-hosts

          cp $HOME/k8s-cluster-hosts $HOME/k8s-cluster-hosts-${K8S_CLUSTER_SHORT}-$(date '+%Y%m%d'-'%H%M%S')  

4. Set login without password

    4.1 Generate SSH-Key (If already generated, skip)
    
    - Generate SSH-Key
    - At the ansible Server
    
          ssh-keygen

          cat /root/.ssh/id_rsa.pub


    4.2 Copy SSH-Key to Server

    - make script for each server
    - At the ansible Server

          cat <<EOF> ssh-key-copy.txt
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${LB_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${MASTER_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_1}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_2}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${WORKER_3}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${BASTION_0}
          ssh-copy-id -i ~/.ssh/id_rsa.pub root@${NEXUS_0}
          EOF

          cat ssh-key-copy.txt

          rm -rf ssh-key-copy.txt


5. Change Password for manual login to server

    1.1. Create ansible playbook to change server password

    - prepare python-passlib
    - At the ansible Server

          yum install -y python-passlib

    - K8s CLuster Type : base

          mkdir -p $HOME/ansible-playbooks/initialize

          cat <<EOF> $HOME/ansible-playbooks/initialize/change-password-k8s_base.yml
          # change-password-k8s_base.yml
          ---
          - hosts: etcd:master:worker:lb*:shared:glusterfs
            become: yes
            tasks:
              - name: Change user password
                user:
                  name: root
                  update_password: always
                  password: "{{ newpassword|password_hash('sha512') }}"
          EOF

          cat $HOME/ansible-playbooks/initialize/change-password-k8s_base.yml

    - change password

          ansible-playbook -i k8s-cluster-hosts $HOME/ansible-playbooks/initialize/change-password-${K8S_CLUSTER_TYPE}.yml --extra-vars newpassword=K8sadmin@ds
         
    - Done
