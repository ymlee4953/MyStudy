# **040. Setup to use Nexus Server**

1. Setup Nexus Configuration

    1.1. Create Configuration file

    - at ansible server

          mkdir -p $HOME/configurations/etc/yum.repos.d

          cat <<EOF> $HOME/configurations/etc/yum.repos.d/nexus.repo

          [nexus]
          name=Nexus Proxy
          baseurl=http://${NEXUS_0}:8081/repository/yum-group/
          enabled=1
          gpgcheck=0

          EOF

          cat $HOME/configurations/etc/yum.repos.d/nexus.repo

    1.2. Copy Configuration file to each server

    - at ansible Server

    - K8s CLuster Type : base

          cat <<EOF> $HOME/ansible-playbooks/initialize/set_yum_repo-k8s_base.yml
          # set_yum_repo-k8s_base.yml
          ---
          - hosts: etcd:master:worker:glusterfs:lb*
            become: true
            tasks:
              - name: copy airgap YUM Repo File
                copy:
                  src: $HOME/configurations/etc/yum.repos.d/nexus.repo
                  dest: /etc/yum.repos.d/nexus.repo
          EOF

          cat $HOME/ansible-playbooks/initialize/set_yum_repo-k8s_base.yml

    - Copy Configuration file to each server

          ansible-playbook -i k8s-cluster-hosts $HOME/ansible-playbooks/initialize/set_yum_repo-${K8S_CLUSTER_TYPE}.yml
  
  
  
