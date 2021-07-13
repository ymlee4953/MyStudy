cat <<EOF> storage_class_local.yaml
  119  apiVersion: storage.k8s.io/v1
  120  kind: StorageClass
  121  metadata:
  122      name: local-storage
  123  provisioner: kubernetes.io/no-provisioner
  124  volumeBindingMode: WaitForFirstConsumer
  125  EOF
  126  cat storage_class_local.yaml
  127  cat <<EOF> storage_class_local.yaml
  128  apiVersion: storage.k8s.io/v1
  129  kind: StorageClass
  130  metadata:
  131    name: local-storage
  132  provisioner: kubernetes.io/no-provisioner
  133  volumeBindingMode: WaitForFirstConsumer
  134  EOF
  135  cat storage_class_local.yaml
  136  kubectl apply -f storage_class_local.yaml
  137  kubectl get sc
  138  kubectl get storageclass
 