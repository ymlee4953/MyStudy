2. init 
  -  
    - 
      -  

    - containerd

          mkdir -p ~/configurations/etc/modules-load.d/

          cat <<EOF | tee ~/configurations/etc/modules-load.d/containerd.conf
          overlay
          br_netfilter
          EOF          
          cat ~/configurations/etc/modules-load.d/containerd.conf

          cat <<EOF | tee ~/configurations/etc/modules-load.d/k8s.conf
          br_netfilter
          EOF
          cat ~/configurations/etc/modules-load.d/k8s.conf

          mkdir -p ~/configurations//etc/sysctl.d/

          cat <<EOF | tee ~/configurations//etc/sysctl.d/99-kubernetes-cri.conf
          net.bridge.bridge-nf-call-iptables  = 1
          net.ipv4.ip_forward                 = 1
          net.bridge.bridge-nf-call-ip6tables = 1
          EOF          
          cat ~/configurations//etc/sysctl.d/99-kubernetes-cri.conf

          cat <<EOF | tee ~/configurations/etc/sysctl.d/k8s.conf
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1
          EOF
          cat ~/configurations/etc/sysctl.d/k8s.conf

    - copy

          mkdir -p ~/ansible-playbooks/containerd/

          cat <<EOF> ~/ansible-playbooks/containerd/base_for_containerd.yml
          # base_for_containerd.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: copy configurations files containerd.conf
                copy:
                  src: ~/configurations/etc/modules-load.d/containerd.conf
                  dest: /etc/modules-load.d/
              - name: modprobe overlay
                command: modprobe overlay
              - name: modprobe br_netfilter
                command: modprobe br_netfilter
              - name: copy configurations files containerd.conf
                copy:
                  src: ~/configurations/etc/sysctl.d/99-kubernetes-cri.conf
                  dest: /etc/sysctl.d/
              - name: sysctl system
                command: sysctl --system
          EOF

          cat ~/ansible-playbooks/containerd/base_for_containerd.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/base_for_containerd.yml

    - install containerd

          cat <<EOF> ~/ansible-playbooks/containerd/install_containerd.yml
          # install_containerd.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: install containerd
                yum: 
                  name:
                    - containerd.io
          EOF

          cat ~/ansible-playbooks/containerd/install_containerd.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/install_containerd.yml



    - Modity Containerd Configuration File

          cat <<EOF> ~/ansible-playbooks/containerd/modify_containerd_configuration.yml
          # modify_containerd_configuration.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: make containerd directory
                file:
                  path: /etc/containerd
                  state: directory
              - name:  make containerd/config.toml
                ansible.builtin.shell: containerd config default | tee /etc/containerd/config.toml
              - name: insertafter SystemdCgroup String in file
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'containerd.runtimes.runc.options'
                  line: "            SystemdCgroup = true"
              - name: delete Nexus String in file
                lineinfile:
                  path: /etc/containerd/config.toml
                  regexp: 'https://registry-1.docker.io'
                  state: absent
              - name: insertafter Nexus String in file 6
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."docker.io"'
                  line: '        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."${NEXUS_0}:5001"]'
              - name: insertafter Nexus String in file 7                  
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."${NEXUS_0}:5001"'
                  line: '          endpoint = ["http://${NEXUS_0}:5001"]      # for ${NEXUS_0}:5001'                  
              - name: insertafter Nexus String in file 4
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."docker.io"'
                  line: '        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]'
              - name: insertafter Nexus String in file 5
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."quay.io"'
                  line: '          endpoint = ["http://${NEXUS_0}:5001"]      # for qauy.io'                  
              - name: insertafter Nexus String in file 2
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."docker.io"'
                  line: '        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]'
              - name: insertafter Nexus String in file 3
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."k8s.gcr.io"'
                  line: '          endpoint = ["http://${NEXUS_0}:5001"]      # for k8s.gcr.io'
              - name: insertafter Nexus String in file 1
                lineinfile:
                  path: /etc/containerd/config.toml
                  insertafter: 'registry.mirrors."docker.io"'
                  line: '          endpoint = ["http://${NEXUS_0}:5001"]      # for docker.io'         
          EOF

          cat ~/ansible-playbooks/containerd/modify_containerd_configuration.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/modify_containerd_configuration.yml


          cat <<EOF> ~/ansible-playbooks/containerd/start_containerd.yml
          # install_containerd.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: start containerd
                command: 
                  systemctl restart containerd
              - name: enable containerd
                command: 
                  systemctl enable containerd
          EOF

          cat ~/ansible-playbooks/containerd/start_containerd.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/start_containerd.yml

         


    - d




          mkdir -p ~/ansible-playbooks/kubernets/

          cat <<EOF> ~/ansible-playbooks/kubernets/base_for_kubernets.yml
          # base_for_kubernets.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: copy configurations files k8s.conf to module-load.d
                copy:
                  src: ~/configurations/etc/modules-load.d/k8s.conf
                  dest: /etc/modules-load.d/
              - name: copy configurations files k8s.conf to sysctl.d
                copy:
                  src: ~/configurations/etc/sysctl.d/k8s.conf
                  dest: /etc/sysctl.d/
              - name: sysctl system
                command: sysctl --system
              - name: setenforce 0
                command: setenforce 0
              - name: sed /etc/selinux/config
                command: sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
              - name: swapoff 
                command: swapoff -a
              - name: sed /etc/fstab
                command: sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
          EOF

          cat ~/ansible-playbooks/kubernets/base_for_kubernets.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernets/base_for_kubernets.yml

    - install containerd

          cat <<EOF> ~/ansible-playbooks/containerd/install_kubernetes.yml
          # install_kubernetes.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: install kubernetes
                yum: 
                  name:
                    - kubelet-1.20.5
                    - kubeadm-1.20.5
                    - kubectl-1.20.5
              - name: enable kubelet service
                command: systemctl enable kubelet.service
              - name: download 
                command: kubeadm config images pull --kubernetes-version v1.20.5
          EOF

          cat ~/ansible-playbooks/containerd/install_kubernetes.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/install_kubernetes.yml


    - d

          mkdir -p ~/ansible-playbooks/kubernetes/

          cat <<EOF> ~/ansible-playbooks/kubernetes/modify_systemd_configuration.yml
          # modify_systemd_configuration.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: insertafter
                lineinfile:
                  path: /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
                  insertafter: 'Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"'
                  line: 'Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"'
              - name: Add CGROUP ARGS line
                lineinfile:
                  path: /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
                  backrefs: true
                  regexp: "^(.*/usr/bin/kubelet.*)$"
                  line: '\1 \$KUBELET_CGROUP_ARGS' 
          EOF
          
          cat ~/ansible-playbooks/kubernetes/modify_systemd_configuration.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/modify_systemd_configuration.yml



---
- d

    d
    - d


          mkdir -p ~/ansible-playbooks/correction/
          
          cat <<EOF> ~/ansible-playbooks/correction/correct_systemd_configuration.yml
          # correct_systemd_configuration.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: insert original
                lineinfile:
                  path: /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
                  backrefs: true
                  regexp: "^(.*/usr/bin/kubelet.*)$"
                  line: '\1 \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS \$KUBELET_EXTRA_ARGS' 
          EOF
          
          cat ~/ansible-playbooks/correction/correct_systemd_configuration.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/correction/correct_systemd_configuration.yml
          
    - done
