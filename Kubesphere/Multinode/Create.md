# **Multi-node Installation**

source : https://kubesphere.io/docs/installing-on-linux/introduction/multioverview/


1. Install and configure prerequisites:

    - at All Linux Host

          cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
          overlay
          br_netfilter
          EOF

          sudo modprobe overlay
          sudo modprobe br_netfilter

          # Setup required sysctl params, these persist across reboots.
          
          cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
          net.bridge.bridge-nf-call-iptables  = 1
          net.ipv4.ip_forward                 = 1
          net.bridge.bridge-nf-call-ip6tables = 1
          EOF

          # Apply sysctl params without reboot
          
          sudo sysctl --system

2. Install containerd: (for CentOS/RHEL 7.4+)

    - at All Linux Host

          # (Install containerd)
          ## Set up the repository
          ### Install required packages
          
          sudo yum install -y yum-utils device-mapper-persistent-data lvm2

          ## Add docker repository
          
          sudo yum-config-manager \
              --add-repo \
              https://download.docker.com/linux/centos/docker-ce.repo

          ## Install containerd
          
          sudo yum update -y && sudo yum install -y containerd.io

          ## Configure containerd
          
          sudo mkdir -p /etc/containerd
          containerd config default | sudo tee /etc/containerd/config.toml

          # Restart containerd
          
          sudo systemctl restart containerd
          sudo systemctl enable containerd

3. systemd

    - To use the systemd cgroup driver in **/etc/containerd/config.toml** with **runc**, set

          vi /etc/containerd/config.toml
          :97

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
            ...
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
              SystemdCgroup = true


    - Restart containerd
       
          sudo systemctl restart containerd

    - When using kubeadm, manually configure the cgroup driver for kubelet.  


4. Install Dependency
  
    4.1 socat
    - at the master

          yum install socat-1.7.3.2-2.el7.x86_64 -y


    4.2 conntrack
    - at the master

          yum install conntrack-tools-1.4.4-7.el7.x86_64 -y

    4.3 ebtables
    - at the master

          yum install ebtables-2.0.10-16.el7.x86_64 -y


5. Download KubeKey

    5.1 Download kubekey
    - at the master

          curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.0 sh -

    5.2 Make kk executable:
    - at the master

          chmod +x kk

6. Create configuration file

    6.1 Create an example configuration file
    - v1.20.4

          ./kk create config --with-kubernetes v1.20.4 --with-kubesphere v3.1.0

    6.2 edit the configuration file
    - d

          vi config-sample.yaml
          ------------------------------------------------------------------------------------------------------------

          apiVersion: kubekey.kubesphere.io/v1alpha1
          kind: Cluster
          metadata:
            name: sample
          spec:
            hosts:
            - {name: master, address: 169.45.97.140, internalAddress: 10.168.180.195, user: root, password: K8sadmin@test}
            - {name: node1, address: 169.45.97.132, internalAddress: 10.168.180.170, user: root, password: K8sadmin@test}
            - {name: node2, address: 169.45.97.135, internalAddress: 10.168.180.162, user: root, password: K8sadmin@test}
            roleGroups:
              etcd:
              - master
              master:
              - master
              worker:
              - node1
              - node2
            kubernetes:
              version: v1.20.4
              imageRepo: kubesphere
              clusterName: cluster-s.local
            network:
              plugin: calico
              kubePodsCIDR: 10.233.64.0/18
              kubeServiceCIDR: 10.233.0.0/18
            registry:
              registryMirrors: []
              insecureRegistries: []
            addons: []


7. Create Cluster

    7.1 Create
    - d

          ./kk create cluster -f config-sample.yaml

