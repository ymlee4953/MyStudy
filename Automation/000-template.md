2. init 
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
          