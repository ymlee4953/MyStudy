        


3. make directory
  -  
    - 
      -  

          cat <<EOF> storage_class_local.yaml
          apiVersion: storage.k8s.io/v1
          kind: StorageClass
          metadata:
            name: local-storage
          provisioner: kubernetes.io/no-provisioner
          volumeBindingMode: WaitForFirstConsumer
          EOF
          cat storage_class_local.yaml


          kubectl apply -f storage_class_local.yaml
          kubectl get sc
          kubectl get storageclass

          kubectl patch storageclass local-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
          
          kubectl get storageclass


3. Download install file
  -  
    - 
      -  

          mkdir -p ~/kubesphere/install_on_k8s_cluster

          cd ~/kubesphere/install_on_k8s_cluster

          curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/kubesphere-installer.yaml

          curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/cluster-configuration.yaml
