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


gitlab-ci.yml
```yaml
stages:
  - package
  - build
  - deploy
variables:
  MAVEN_CLI_OPTS: "-s settings.xml --batch-mode" # 配置Maven使用的settings.xml文件
  API_URL: "http://gitlabhwy.kmlckj.com/api/v4"
package:
  stage: package
  image: maven:3.6.3-openjdk-17-slim
  script:
    - echo "$MVN_SETTING" > settings.xml
    - mvn $MAVEN_CLI_OPTS clean package -Dmaven.test.skip=true
  # 定义流水线输出文件，将生成的JAR文件作为artifact保存
  artifacts:
    paths:
      - target/fileSystem-2.6.0-RELEASE.jar
docker_build:
  needs: [package]
  stage: build
  image: docker:latest
  before_script:
    - apk update
    - apk add curl &&apk add jq
    - 'VERSION=$(curl --header "PRIVATE-TOKEN: $CI_TOKEN" "$API_URL/projects/$CI_PROJECT_ID/variables/VERSION" | jq -r ".value")'
    - VERSION=$((VERSION + 1))
    - 'curl --request PUT --header "PRIVATE-TOKEN:$CI_TOKEN" --header "Content-Type: application/json" --data "{\"value\":\"$VERSION\"}" "$API_URL/projects/$CI_PROJECT_ID/variables/VERSION"'

  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - DOCKER_IMAGE_NAME=$CI_REGISTRY/kmlc/licos-file:$CI_COMMIT_REF_NAME-$VERSION
    - echo "准备推送镜像:$DOCKER_IMAGE_NAME"
    - docker build -f devops/docker/Dockerfile -t $DOCKER_IMAGE_NAME .
    - docker push  $DOCKER_IMAGE_NAME
    - 'sed -i "s|image:.*|image: $DOCKER_IMAGE_NAME|" devops/deploy/deployment.yaml'
  artifacts:
    paths:
      - devops/deploy/deployment.yaml
kubernetes_deploy:
  needs: [docker_build]
  stage: deploy
  image: kubesphere/kubectl:v1.17.0  # 使用kubectl作为执行环境
  script:
    - kubectl apply -f devops/deploy/deployment.yaml

```