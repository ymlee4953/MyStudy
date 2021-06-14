# **120 - Setup LB-1**

1. Install haproxy
  
    1.1 install Base package for haproxy

    - for LB-1, LB-2 both

          mkdir -p ~/ansible-playbooks/LB

          cat <<EOF> ~/ansible-playbooks/LB/base_package_for_haproxy.yml
          # base_package_for_haproxy.yml
          ---
          - hosts: lb*
            become: true
            tasks:
              - name: install base package 
                yum:
                  name: 
                    - gcc
                    - pcre-static
                    - pcre-devel
                    - wget
                    - make
                    - openssl-devel
          EOF
          cat ~/ansible-playbooks/LB/base_package_for_haproxy.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/base_package_for_haproxy.yml

    1.2 copy haproxy file from bastion server

    - for LB-1, LB-2 both

          cat <<EOF> ~/ansible-playbooks/LB/copy_haproxy.tar.gz_to_LB_servers.yml
          # copy_haproxy.tar.gz_to_LB_servers.yml
          - hosts: BASTION-0
            become: true
            tasks:
              - name: Fetch the haproxy-1.7.8.tar.gz file from the Bastion_0
                fetch: 
                  src: ~/haproxy/{{ item }}
                  dest: ~/files/
                with_items:
                - haproxy-1.7.8.tar.gz
          EOF
          
          cat ~/ansible-playbooks/LB/copy_haproxy.tar.gz_to_LB_servers.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/copy_haproxy.tar.gz_to_LB_servers.yml
      
    1.3 Install haproxy

    - for LB-1, LB-2 both

          cat <<EOF> ~/ansible-playbooks/LB/install_haproxy.yml
          # install_haproxy.yml
          - hosts: lb*
            become: true
            tasks: 
              - name: Copy the file to the haproxy
                copy: 
                  src: ~/files/BASTION-0/root/haproxy/haproxy-1.7.8.tar.gz
                  dest: /opt/
              - name: unarchive haproxy-1.7.8.tar.gz
                unarchive:
                  src: ~/files/BASTION-0/root/haproxy/haproxy-1.7.8.tar.gz
                  dest: /opt/
              - name: make directory at loadbalancer servers
                file:                  
                  path: "{{ item }}"
                  state: directory
                loop:            
                  - /opt/haproxy-1.7.8/
                  - /etc/haproxy/
                  - /var/lib/haproxy/
              - name: make haproxy files
                make:
                  chdir: /opt/haproxy-1.7.8
                  params:
                    TARGET=linux2628
                    USE_OPENSSL=1
              - name: run haproxy install files
                make:
                  chdir: /opt/haproxy-1.7.8
                  target: install
              - name: touch files
                file:
                  path: /var/lib/haproxy/stats
                  state: touch
              - name: Create symbolic link 
                file:
                  src: "/usr/local/sbin/haproxy"
                  dest: "/usr/sbin/haproxy"
                  state: link
              - name: cp haproxy.init   
                command: 
                  cp /opt/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
              - name: chmod haproxy.init 755  
                file: 
                  path: /etc/init.d/haproxy
                  mode: 0755          
              - name: create system user
                ansible.builtin.user:
                  name: haproxy
                  system: yes
                  state: present                  
          EOF

          cat ~/ansible-playbooks/LB/install_haproxy.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/install_haproxy.yml

    1.4. prepare haproxy.cfg at the ansible server

    - for LB-1

          mkdir -p ~/configurations/LB/etc/haproxy

          cat <<EOF > ~/configurations/LB/etc/haproxy/haproxy.cfg 
          global
                log /dev/log local0
                log /dev/log local1 notice
                chroot /var/lib/haproxy
                stats timeout 30s
                user haproxy
                group haproxy
                daemon
            
          defaults
                maxconn 20000
                mode    tcp
                option  dontlognull
                timeout http-request 10s
                timeout queue        1m
                timeout connect      10s
                timeout client       86400s
                timeout server       86400s
                timeout tunnel       86400s
          EOF

          cat ~/configurations/LB/etc/haproxy/haproxy.cfg 

          mkdir -p ~/configurations/LB-1/etc/haproxy

          rm -rf ~/configurations/LB-1/etc/haproxy/haproxy.cfg

          cp ~/configurations/LB/etc/haproxy/haproxy.cfg ~/configurations/LB-1/etc/haproxy/haproxy.cfg

          cat <<EOF>> ~/configurations/LB-1/etc/haproxy/haproxy.cfg

          #---------------------------------------------------------------------
          # For a High Available kubenetes master node cluster 
          #---------------------------------------------------------------------

          frontend kubernetes
              bind ${LB_1}:6443
              mode tcp
              option tcplog
              default_backend kubernetes-backend

          backend kubernetes-backend
              mode tcp
              option tcp-check
              balance roundrobin
              server master-1 ${MASTER_1}:6443 check fall 3 rise 2
              server master-2 ${MASTER_2}:6443 check fall 3 rise 2
              server master-3 ${MASTER_3}:6443 check fall 3 rise 2
          EOF

          cat ~/configurations/LB-1/etc/haproxy/haproxy.cfg

    1.5. Start haproxy

    - for LB-1

          cat <<EOF> ~/ansible-playbooks/LB/start_haproxy.yml
          # start_haproxy.yml
          ---
          - hosts: LB-1
            become: true
            tasks:
              - name: copy haproxy config file
                copy:
                  src: ~/configurations/LB-1/etc/haproxy/haproxy.cfg
                  dest: /etc/haproxy/haproxy.cfg
              - name: daemon-reload
                command: systemctl daemon-reload
              - name: start haproxy
                command: systemctl start haproxy
              - name: enable haproxy
                command: systemctl enable haproxy
              - name: status haproxy
                command: systemctl status haproxy
          EOF

          cat ~/ansible-playbooks/LB/start_haproxy.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/start_haproxy.yml
