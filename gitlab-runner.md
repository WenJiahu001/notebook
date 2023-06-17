如果是在k8s中部署记得添加docker.sock的挂载,让runner启动的runner可以连接宿主机的docker

```
[runners.kubernetes]
...
[runners.kubernetes.volumes]
[[runners.kubernetes.volumes.host_path]]
name = "docker-sock"
mount_path = "/var/run/docker.sock"
host_path = "/var/run/docker.sock"
```

k8s创建管理员role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```
k8s创建管理员rolebinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: user-cluster-admin-binding
subjects:
- kind: ServiceAccount
  name: default
  namespace: wjh
roleRef:
  kind: ClusterRole
  name: cluster-admin-role
  apiGroup: rbac.authorization.k8s.io
```