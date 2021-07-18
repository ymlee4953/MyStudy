# **12. Setup LB-1**

1. Install haproxy
  
    1.1 install Base package for haproxy

    - for LB-1, LB-2 both

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/base_package_for_haproxy.yml

    1.2 Install haproxy

    - for LB-1, LB-2 both

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/install_haproxy.yml

    1.3. prepare haproxy.cfg at the ansible server

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
          
          cp ~/configurations/LB-1/etc/haproxy/haproxy.cfg configurations/LB-1/etc/haproxy/haproxy-${K8S_CLUSTER_SHORT}.cfg


    1.4. prepare haproxy.cfg at the ansible server

    - for LB-2

          rm -rf ~/configurations/LB-2/etc/haproxy/haproxy.cfg
          cp ~/configurations/LB/etc/haproxy/haproxy.cfg ~/configurations/LB-2/etc/haproxy/haproxy.cfg

          cat <<EOF>> ~/configurations/LB-2/etc/haproxy/haproxy.cfg

          #---------------------------------------------------------------------
          # For a High Available kubenetes router node cluster 
          #---------------------------------------------------------------------

          frontend http_front
            bind *:80
            option tcplog
            mode tcp
            default_backend http_back

          backend http_back
            mode tcp
            balance roundrobin
            server router-1 ${ROUTER_1}:32142
            server router-2 ${ROUTER_2}:32142
            server router-3 ${ROUTER_3}:32142

          frontend https_front
            bind *:443
            option tcplog
            mode tcp
            default_backend https_back

          backend https_back
            mode tcp
            balance roundrobin
            option ssl-hello-chk
            server router-1 ${ROUTER_1}:30266 check
            server router-2 ${ROUTER_2}:30266 check
            server router-3 ${ROUTER_3}:30266 check

          EOF

          cat ~/configurations/LB-2/etc/haproxy/haproxy.cfg
          
          cp ~/configurations/LB-2/etc/haproxy/haproxy.cfg configurations/LB-2/etc/haproxy/haproxy-${K8S_CLUSTER_SHORT}.cfg


    1.5. Start haproxy

    - for LB-1

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/start_haproxy_1.yml

    - for LB-2

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/LB/start_haproxy_2.yml
