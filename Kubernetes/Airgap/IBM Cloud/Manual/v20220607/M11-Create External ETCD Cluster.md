


3. Install utilities
          
    3.1. Install wget


    - Task 1131
    - to download with wget 
          
          yum install -y wget


    - Task 1132
    3.2. Install bind-utils
                   
    - to get host ip 
          
          yum install -y bind-utils
---
## **For External etcd**


4. Download Software Package for etcdadm
    

    - Task 
    4.1 install Git and make
    - yum install and make

          yum install -y make git

    4.2 Install Go
    - Download File

          wget https://dl.google.com/go/go1.16.2.linux-amd64.tar.gz
          tar -C /usr/local -xzf ./go1.16.2.linux-amd64.tar.gz

    - add Go Path

          vi ~/.bash_profile

    - edit Go Path
    
          ...
          PATH=$PATH:$HOME/bin:/usr/local/go/bin
          ...
    
    - apply profile 

          source ~/.bash_profile

    4.3. Download etcdadm source file and make
    - Air-gap 환경 설치임으로 Bastion 을 통해 etcdadm 소스 및 etcd 설치 파일 다운로드 후 etcd 서버로 전송

          git clone https://github.com/kubernetes-sigs/etcdadm.git

          cd etcdadm
          make etcdadm 

    4.4 Download etcd source file

    - Check the default etcd version of etcdadm
    
          cat constants/constants.go | grep DefaultVersion

    - the result : the default etcd version of etcdadm is "3.5.1"

          ==> DefaultVersion = "3.5.1"

    - Download etcd installation file

          wget https://github.com/coreos/etcd/releases/download/v3.5.1/etcd-v3.5.1-linux-amd64.tar.gz
          
 
    4.5 Copy the etcd source files for Ansible

    - Copy files
    
          mkdir -p ~/files/BASTION-0/etcdadm/

          cp ~/etcdadm/etcdadm ~/files/BASTION-0/etcdadm/
          cp ~/etcdadm/etcd-v3.5.1-linux-amd64.tar.gz ~/files/BASTION-0/etcdadm/


at BASTION-0



    ssh-copy-id -i ~/.ssh/id_rsa.pub root@${ETCD_1}

    ssh $ETCD_1

    passwd
    K8sadmin@dell






