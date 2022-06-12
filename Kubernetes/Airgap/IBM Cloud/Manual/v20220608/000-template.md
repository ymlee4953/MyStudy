1. init 
  -  
    - 
      -  

          cat <<EOF> ~/ansible-playbooks/dirdir/playbookfile.yml
          # playbookfile.yml
          ---
          - hosts: ETCD-1
            become: true
            tasks:
              - name: initialize etcd cluster
                command: ~/etcdadm init
          EOF

          cat ~/ansible-playbooks/dirdir/playbookfile.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/dirdir/playbookfile.yml
          
          mkdir -p ~/files

2. fetch & copy
  -  
    - 
      -  

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
                - etcd-v3.4.9-linux-amd64.tar.gz  
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
              - name: Copy the tar file to etcd v3.4.9
                copy:
                  src:  ~/files/BASTION-0/root/etcdadm/etcd-v3.4.9-linux-amd64.tar.gz
                  dest: /var/cache/etcdadm/etcd/v3.4.9/
          EOF

          cat ~/ansible-playbooks/etcd/copy_etcdadm.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/etcd/copy_etcdadm.yml          


3. make directory
  -  
    - 
      -  
              - name: make directory at loadbalancer servers
                file:                  
                  path: "{{ item }}"
                  state: directory
                loop:            
                  - /opt/haproxy-1.7.8/
                  - /etc/haproxy/
                  - /var/lib/haproxy/      