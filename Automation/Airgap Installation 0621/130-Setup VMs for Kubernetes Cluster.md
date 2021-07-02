# **130. Setup VMs for Kubernetes Cluster**

1. Setup Nexus Configuration

    1.1. Create Configuration file

    - at ansible server

          mkdir -p ~/configurations/etc/yum.repos.d

          cat <<EOF> ~/configurations/etc/yum.repos.d/nexus.repo

          [nexus]
          name=Nexus Proxy
          baseurl=http://${NEXUS_0}:8081/repository/yum-group/
          enabled=1
          gpgcheck=0

          EOF

          cat ~/configurations/etc/yum.repos.d/nexus.repo

    1.2. Copy Configuration file to each server

    - at ansible Server

          cat <<EOF> ~/ansible-playbooks/initialize/set_yum_repo.yml
          # set_yum_repo.yml
          ---
          - hosts: master:worker
            become: true
            tasks:
              - name: copy airgap YUM Repo File
                copy:
                  src: ~/configurations/etc/yum.repos.d/nexus.repo
                  dest: /etc/yum.repos.d/nexus.repo
          EOF

          cat ~/ansible-playbooks/initialize/set_yum_repo.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/initialize/set_yum_repo.yml

    1.3. Install NFS Client to each K8s server

    - at ansible Server
    - get host ip of file storage

          host ${FILE_STORAGE_HOST}

          export

    - mount file storage

    - reference: https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-mount-centos&locale=ko

          cat <<EOF> ~/ansible-playbooks/initialize/yum_install_NFS_utils.yml
          # yum_install_NFS_utils.yml
          ---
          - hosts: master:worker
            become: true
            tasks:
              - name: yum install NFS utils
                yum:
                  name:
                    - nfs-utils
          EOF

          cat ~/ansible-playbooks/initialize/yum_install_NFS_utils.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/initialize/yum_install_NFS_utils.yml

        



    1.4. Install Linux Netcat to each K8s master server

    - at ansible Server

          cat <<EOF> ~/ansible-playbooks/initialize/yum_install_nc.yml
          # yum_install_nc.yml
          ---
          - hosts: master
            become: true
            tasks:
              - name: yum install nc
                yum:
                  name:
                    - nc
          EOF

          cat ~/ansible-playbooks/initialize/yum_install_nc.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/initialize/yum_install_nc.yml



2. Install Container Runtime (containerd)

    2.1. Create configuration files for containerd
    - prepare at ansible server

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

          mkdir -p ~/configurations/etc/sysctl.d/

          cat <<EOF | tee ~/configurations/etc/sysctl.d/99-kubernetes-cri.conf
          net.bridge.bridge-nf-call-iptables  = 1
          net.ipv4.ip_forward                 = 1
          net.bridge.bridge-nf-call-ip6tables = 1
          EOF

          cat ~/configurations/etc/sysctl.d/99-kubernetes-cri.conf

          cat <<EOF | tee ~/configurations/etc/sysctl.d/k8s.conf
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1
          EOF

          cat ~/configurations/etc/sysctl.d/k8s.conf

    2.2. Copy CRI configuration to each server
    - with ansible plyabooks

          mkdir -p ~/ansible-playbooks/containerd/

          cat <<EOF> ~/ansible-playbooks/containerd/base_for_containerd.yml          
          # base_for_containerd.yml
          ---
          - hosts: master:worker
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

    2.3. Install containerd
    - with ansible plyabooks

          cat <<EOF> ~/ansible-playbooks/containerd/install_containerd.yml
          # install_containerd.yml
          ---
          - hosts: master:worker
            become: true
            tasks:
              - name: install containerd
                yum: 
                  name:
                    - containerd.io
          EOF

          cat ~/ansible-playbooks/containerd/install_containerd.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/install_containerd.yml


    2.4 Modity Containerd Configuration File
    - with ansible plyabooks

          cat <<EOF> ~/ansible-playbooks/containerd/modify_containerd_configuration.yml
          # modify_containerd_configuration.yml
          ---
          - hosts: master:worker
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

    2.5 Start Containerd
    - with ansible plyabooks
          
          cat <<EOF> ~/ansible-playbooks/containerd/start_containerd.yml
          # start_containerd.yml
          ---
          - hosts: master:worker
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

         
3. Install Kubernetes package

    3.1 Setup base configuration for Kubernetes
    - with ansible plyabooks

          mkdir -p ~/ansible-playbooks/kubernetes/

          cat <<EOF> ~/ansible-playbooks/kubernetes/base_for_kubernetes.yml
          # base_for_kubernetes.yml
          ---
          - hosts: master:worker
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

          cat ~/ansible-playbooks/kubernetes/base_for_kubernetes.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/base_for_kubernetes.yml

    3.2 Install Kubernetes
    - with ansible plyabooks    

          cat <<EOF> ~/ansible-playbooks/containerd/install_kubernetes.yml
          # install_kubernetes.yml
          ---
          - hosts: master:worker
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


    3.3. Systemd Configuration
    - with ansible plyabooks    

          mkdir -p ~/ansible-playbooks/kubernetes/

          cat <<EOF> ~/ansible-playbooks/kubernetes/modify_systemd_configuration.yml
          # modify_systemd_configuration.yml
          ---
          - hosts: master:worker
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

4. Restart kubelet

    4.1  Restart kubelet
    - with ansible plyabooks 
    
          cat <<EOF> ~/ansible-playbooks/kubernetes/restart_kubelet.yml
          # restart_kubelet.yml
          ---
          - hosts: master:worker:ingress:infra
            become: true
            tasks:
              - name: daemon-reload
                command: systemctl daemon-reload
              - name: start kubelet
                command: systemctl start kubelet
              - name: enable kubelet
                command: systemctl enable kubelet            
          EOF

          cat ~/ansible-playbooks/kubernetes/restart_kubelet.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/restart_kubelet.yml

---
