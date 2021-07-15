# **12. Setup LB-1**

1. Install haproxy
  
    1.1 install Base package for haproxy

    - for LB-1, LB-2 both

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/base_package_for_haproxy.yml

    1.2 Install haproxy

    - for LB-1, LB-2 both

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/install_haproxy.yml

    1.4. prepare haproxy.cfg at the ansible server

    - for LB-1

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

          frontend ksconsole
              bind *:30880
              mode tcp
              option tcplog
              default_backend ksconsole-backend

          backend ksconsole-backend
              mode tcp
              option tcp-check
              balance roundrobin
              server master-1 ${MASTER_1}:30880 check fall 3 rise 2
              server master-2 ${MASTER_2}:30880 check fall 3 rise 2
              server master-3 ${MASTER_3}:30880 check fall 3 rise 2              
          EOF

          cat ~/configurations/LB-1/etc/haproxy/haproxy.cfg

    1.5. Start haproxy

    - for LB-1

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/start_haproxy.yml
