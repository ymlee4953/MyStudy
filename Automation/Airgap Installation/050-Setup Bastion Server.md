# **050. Setup Bastion Server**
  
---  
## **For etcdadm**

1. Base Package Installation to Bastion Server

    1.1 install Git and make
    - **At the Bastion Server**

          yum install -y make git

    1.2 Install Go
    - **At the Bastion Server**

    - Down Load File

          wget https://dl.google.com/go/go1.16.2.linux-amd64.tar.gz
          tar -C /usr/local -xzf ./go1.16.2.linux-amd64.tar.gz

    - add Go Path

          vi ~/.bash_profile

          ...
          PATH=$PATH:$HOME/bin:/usr/local/go/bin
          ...
    
    - apply profile 

          source ~/.bash_profile

2. Download etcdadm source file and make
 
    - **At the Bastion Server**
    - Air-gap 환경 설치임으로 Bastion 을 통해 etcdadm 소스 및 etcd 설치 파일 다운로드 후 etcd 서버로 ftp 전송

          git clone https://github.com/kubernetes-sigs/etcdadm.git

          cd etcdadm
          make etcdadm 

3. Download etcd source file

    3.1 Check the default etcd version of etcdadm
    - **At the Bastion Server**

          cat constants/constants.go | grep DefaultVersion
 
          ==> DefaultVersion = "3.4.13"

    3.2 Download etcd installation file
    - **At the Bastion Server**

          wget https://github.com/coreos/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
          
          
---
## **For haproxy**

4. Download and Copy the Installation File

    4.1 Download Installation File to Bastion Server
    - **At the Bastion Server**
    - HA Proxy 서버는 Air-gap 환경임으로 Bastion 을 통해 설치 파일을 다운로드
      
          # go to root directory and download the installation files
          
          cd 
          wget https://www.haproxy.org/download/1.7/src/haproxy-1.7.8.tar.gz

          mkdir -p ~/haproxy  
          mv ./haproxy-1.7.8.tar.gz ~/haproxy


---
## **For Calico**

5. Install CNI : Calico
    - CNI : Calico
    - Typha 적용

    5.1 Download and copy the yaml file for the Calico Typha
    - Download the yaml File
    - **At the Bastion server**
     
          mkdir -p ~/calico

          curl https://docs.projectcalico.org/manifests/calico.yaml -o calico.yaml
          curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico-typha.yaml

          mv ./calico*.yaml  ~/calico


---
## **For Kubesphere**

6. Kubesphere
    - d

    5.1 Download and copy the yaml file for the kubesphere
    - Download the yaml File
    - **At the Bastion server**
     
          mkdir -p ~/kubesphere

          cd ~/kubesphere

          curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/cluster-configuration.yaml
          curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/kubesphere-installer.yaml

          cd ~/
