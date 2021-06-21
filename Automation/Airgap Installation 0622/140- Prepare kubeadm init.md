# **140. Prepare kubeadm init**

1. Copy Etcd cert to master-1

    1.1. Fetch etcd cert to Bastion
    - with ansible playbook

          cat <<EOF> ~/ansible-playbooks/kubernetes/fetch_etcd_cert_file.yml
          # fetch_etcd_cert_file.yml
          - hosts: ETCD-1
            become: true
            tasks:
              - name: Fetch the etcd cert files from the ETCD_1
                fetch: 
                  src: /etc/etcd/pki/{{ item }}
                  dest: ~/files/
                with_items:
                - apiserver-etcd-client.crt
                - apiserver-etcd-client.key
          EOF
          
          cat ~/ansible-playbooks/kubernetes/fetch_etcd_cert_file.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/fetch_etcd_cert_file.yml

    1.2. Copy etcd cert to master-1
    - with ansible playbook

          cat <<EOF> ~/ansible-playbooks/kubernetes/copy_etcd_cert_file.yml
          # copy_etcd_cert_file.yml
          - hosts: MASTER-1
            become: true
            tasks: 
              - name: make a directory for etcd certs
                file:
                  path: /etc/kubernetes/pki/etcd/
                  state: directory
              - name: Copy the ca file to the etcd directory
                copy: 
                  src: ~/files/ETCD-1/etc/etcd/pki/{{ item }}
                  dest: /etc/kubernetes/pki/etcd
                with_items:
                - ca.crt
                - ca.key
              - name: Copy the file to the pki directory
                copy: 
                  src: ~/files/ETCD-1/etc/etcd/pki/{{ item }}
                  dest: /etc/kubernetes/pki
                with_items:
                - apiserver-etcd-client.crt
                - apiserver-etcd-client.key
          EOF

          cat ~/ansible-playbooks/kubernetes/copy_etcd_cert_file.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/copy_etcd_cert_file.yml

2. Create the kubeadm-config.yaml

    2.1. Generate the Configuration file for Kubernetes cluster
    - at the ansible server

          mkdir -p ~/configurations/master-1/

          cat <<EOF> ~/configurations/master-1/kubeadm-config.yaml
          apiVersion: kubeadm.k8s.io/v1beta2
          kind: InitConfiguration
          nodeRegistration:
            criSocket: "/run/containerd/containerd.sock"
          ---
          apiVersion: kubeadm.k8s.io/v1beta2
          kind: ClusterConfiguration
          clusterName: ${K8S_CLUSTER_NAME}
          kubernetesVersion: "${K8S_VERSION}"
          networking:
            podSubnet: ${POD_SUBNET}
            serviceSubnet: ${SERVICE_SUBNET}
          apiServer:
            certSANs:
            - "${LB_1}"
          controlPlaneEndpoint: "${LB_1}:6443"
          etcd:
            external:
              endpoints:
              - https://${ETCD_1}:2379
              - https://${ETCD_2}:2379
              - https://${ETCD_3}:2379
              caFile: /etc/kubernetes/pki/etcd/ca.crt
              certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
              keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
          ---
          apiVersion: kubelet.config.k8s.io/v1beta1
          kind: KubeletConfiguration
          cgroupDriver: "systemd"
          EOF

          cat ~/configurations/master-1/kubeadm-config.yaml

    2.2. Copy the Configuration file
    - From the ansible server to the master-1

          cat <<EOF> ~/ansible-playbooks/kubernetes/copy_kubeadm_config_file.yml
          # copy_kubeadm_config_file.yml
          - hosts: MASTER-1
            become: true
            tasks: 
              - name: Copy the ca file to the etcd directory
                copy: 
                  src: ~/configurations/master-1/kubeadm-config.yaml
                  dest: ~/
          EOF

          cat ~/ansible-playbooks/kubernetes/copy_kubeadm_config_file.yml

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/copy_kubeadm_config_file.yml
