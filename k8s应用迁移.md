kubectl get namespaces
kubectl config set-context --current --namespace=licos-platform-prod

导出
kubectl get deployment --namespace=licos-platform-prod -o yaml  > deployment.yaml
kubectl get services --namespace=licos-platform-prod -o yaml > services.yaml
kubectl get configmaps --namespace=licos-platform-prod -o yaml  > configmaps.yaml
kubectl get secrets --namespace=licos-platform-prod -o yaml  > secrets.yaml
kubectl get pv --namespace=licos-platform-prod -o yaml  > pv.yaml
kubectl get pvc --namespace=licos-platform-prod -o yaml  > pvc.yaml
kubectl get statefulset --namespace=licos-platform-prod -o yaml  > statefulset.yaml
kubectl get daemonset --namespace=licos-platform-prod -o yaml  > daemonset.yaml

删除资源
kubectl delete configmap --all -n licos-test

删除命名空间lee-dev
kubectl delete ns licos-test

docker save -o 保存的PathName 镜像名:标签
docker load -i 镜像备份文件
docker tag