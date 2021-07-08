# **110. External etcd Installation**


1. Copy etcd files from Bastion to etcd Servers

    1.1. Copy etcdadm and etcd installation files from Bastion
    
    - create ansible playbook

          mkdir -p ~/ansible-playbooks/etcd/

          cat <<EOF> ~/ansible-playbooks/etcd/copy_etcdadm.yml
          # copy_etcdadm.yml
          ---
          - hosts: BASTION-0
            become: true
            tasks:
              - name: Fetch the etcdadm file from the Bastion_0
                fetch: 
                  src: /root/etcdadm/{{ item }}
                  dest: ~/files/
                with_items:
                - etcdadm
                - etcd-v3.5.0-linux-amd64.tar.gz  
          - hosts: etcd
            become: true
            tasks:           
              - name: Copy the file to etcd
                copy:
                  src:  ~/files/BASTION-0/root/etcdadm/etcdadm
                  dest: ~/
          - hosts: etcd
            become: true
            tasks:           
              - name: chmod of etcdadm
                command: chmod 755 ~/etcdadm
          - hosts: etcd
            become: true
            tasks:           
              - name: Copy the tar file to etcd v3.5.0
                copy:
                  src:  ~/files/BASTION-0/root/etcdadm/etcd-v3.5.0-linux-amd64.tar.gz
                  dest: /var/cache/etcdadm/etcd/v3.5.0/
          EOF

          cat ~/ansible-playbooks/etcd/copy_etcdadm.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/etcd/copy_etcdadm.yml

2. Initialize etcd cluster 
    
    2.1. "Run etcdadm init"

    - with ansible playbook
    
          cat <<EOF> ~/ansible-playbooks/etcd/etcdadm_init.yml
          # etcdadm_init.yml
          ---
          - hosts: ETCD-1
            become: true
            tasks:
              - name: initialize etcd cluster
                command: ~/etcdadm init
          EOF

          cat ~/ansible-playbooks/etcd/etcdadm_init.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/etcd/etcdadm_init.yml


3. Copy cert files

    3.1.from ETCD-1 to ETCD-2,3

    - with ansible playbook

          mkdir -p ~/files

          cat <<EOF> ~/ansible-playbooks/etcd/copy_etcd_cert.yml
          # copy_etcd_cert_.yml
          ---
          - hosts: ETCD-1
            become: true
            tasks:
              - name: Fetch the etcd-cert files from the ETCD_1
                fetch: 
                  src: /etc/etcd/pki/{{ item }}
                  dest: ~/files/
                with_items:
                - ca.crt
                - ca.key  
          - hosts: ETCD-2:ETCD-3
            become: true
            tasks:           
              - name: Copy the file to etcd2 etcd3
                copy:
                  src:  ~/files/ETCD-1/etc/etcd/pki/{{ item }}
                  dest: /etc/etcd/pki/
                with_items:
                - ca.crt
                - ca.key  
          EOF

          cat ~/ansible-playbooks/etcd/copy_etcd_cert.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/etcd/copy_etcd_cert.yml


4. Build ETCD cluster

    4.1 Join to the ECDD cluster

    - with ansible playbook

          cat <<EOF> ~/ansible-playbooks/etcd/etcdadm_join.yml
          # etcdadm_join.yml
          ---
          - hosts: ETCD-2:ETCD-3
            become: true
            tasks:
              - name: join to etcd cluster
                command:  ~/etcdadm join https://${ETCD_1}:2379
          EOF

          cat ~/ansible-playbooks/etcd/etcdadm_join.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/etcd/etcdadm_join.yml


5. Check the status (it can be skipped)

    5.1. Check the status of creation

    - check at the etcd server
         
          /opt/bin/etcdctl.sh member list -w table

---
