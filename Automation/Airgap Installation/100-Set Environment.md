

1. Declare Cluster Information

    1.1. export variable with value

    - cluster information and server IP

          export K8S_CLUSTER_NAME=auto_cluster
          export K8S_VERSION=v1.20.5
          export POD_SUBNET=192.168.0.0/16
          export SERVICE_SUBNET=20.96.0.0/16          

          export ETCD_1=10.168.180.175
          export ETCD_2=10.168.180.209
          export ETCD_3=10.168.180.130

          export LB_1=10.168.180.155

          export MASTER_1=10.168.180.246
          export MASTER_2=10.168.180.196
          export MASTER_3=10.168.180.249

          export LB_2=10.168.180.247

          export INGRESS_1=10.168.180.142
          export INGRESS_2=10.168.180.200
          export INGRESS_3=10.168.180.143

          export INFRA_1=10.168.180.149

          export WORKER_1=10.168.180.225
          export WORKER_2=10.168.180.190
          export WORKER_3=10.168.180.164

          export BASTION_0=10.168.180.165
          export NEXUS_0=10.168.180.184

2. Create Directoy for new Cluster

    2.1. Create Directory

    - cluster information and server IP

          mkdir -p $HOME/ansible-playbooks
          mkdir -p $HOME/k8s-clusters/${K8S_CLUSTER_NAME}/configurations



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
