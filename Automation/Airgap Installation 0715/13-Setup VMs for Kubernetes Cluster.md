# **130. Setup VMs for Kubernetes Cluster**

1. Setup Configuration


    1.4. Install Linux Netcat to each K8s master server

    - at ansible Server

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/initialize/yum_install_nc.yml

        -

          echo nc -v ${LB_1} 6443

          ssh root@${MASTER_1}
        -
        
          # nc -v ${LB_1} 6443

          exit


    1.2. Copy Configuration file to each server

    - at ansible Server

    - Copy Configuration file to each server

          ansible-playbook -i k8s-cluster-hosts $HOME/ansible-playbooks/initialize/set_yum_repo.yml


2. Install Container Runtime (containerd)

    2.2. Copy CRI configuration to each server

    - Copy CRI configuration to each server

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/base_for_containerd.yml

    2.3. Install containerd
    - Install containerd

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/install_containerd.yml


    2.4 Modity Containerd Configuration File
    - with ansible plyabooks

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/modify_containerd_configuration.yml

    2.5 Start Containerd
    - with ansible plyabooks
          
          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/start_containerd.yml

         
3. Install Kubernetes package

    3.1 Setup base configuration for Kubernetes
    - with ansible plyabooks

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/base_for_kubernetes.yml

    3.2 Install Kubernetes
    - with ansible plyabooks    


          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/containerd/install_kubernetes.yml


    3.3. Systemd Configuration
    - with ansible plyabooks    

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/modify_systemd_configuration.yml

4. Restart kubelet

    4.1  Restart kubelet
    - with ansible plyabooks 

          ansible-playbook -i k8s-cluster-hosts ~/ansible-playbooks/kubernetes/restart_kubelet.yml

---
