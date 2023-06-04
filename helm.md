#helm笔记
- 仓库
仓库列表 `helm repo list`  
添加仓库`helm repo add bitnami https://charts.bitnami.com/bitnami`  
移除 `helm repo remove`  
`helm search repo bitnami`  
仓库更新`helm repo update`  
- charts
本地搜索`helm search repo`  
公开搜索 `helm search hub wordpress`  
安装 `helm install test bitnami/wordpress`  
安装状态`helm status test`  
可配置项 `helm show values`  
示例指定配置value,不使用默认 `helm install -f values.yaml bitnami/wordpress --generate-name*`   
指定配置文件set优先级最高    
```
--set a=b,c=d
--set name={a, b, c}
--set name=[],a=null
--set servers[0].port=80
```
查看配置 `helm get values test`  
 - 升级 回滚  
 升级 `helm upgrade -f panda.yaml happy-panda bitnami/wordpress`  
 回滚 `helm rollback happy-panda 1`  
 查看版本 `helm history [RELEASE]`  
 超时 `--timeout` 一个 Go duration 类型的值， 用来表示等待 Kubernetes 命令完成的超时时间，默认值为 5m0s  
 等待安装成功 `--wait` 表示必须要等到所有的 Pods 都处于 ready 状态，PVC 都被绑定，Deployments 都至少拥有最小 ready 状态 Pods 个数（Desired减去 maxUnavailable），并且 Services 都具有 IP 地址（如果是LoadBalancer， 则为 Ingress），才会标记该 release 为成功。最长等待时间由 --timeout 值指定。如果达到超时时间，release 将被标记为 FAILED。注意：当 Deployment 的 replicas 被设置为1，但其滚动升级策略中的 maxUnavailable 没有被设置为0时，--wait 将返回就绪，因为已经满足了最小 ready Pod 数。  
 卸载 `helm uninstall  happy-panda --keep-history`
查看helm`helm list`  `helm list --uninstalled`  

 - 自定义charts
  创建 `helm create deis-workflow`  
验证是否有效 `helm lint`  
打包 `helm package deis-workflow`  
然后就可以安装了 `helm install deis-workflow ./deis-workflow-0.1.0.tgz`  
 查看完整的模板信息 `helm get manifest full-coral`  
一个简单的模板实例  ,由于DNS系统的限制，name:字段长度限制为63个字符。因此发布名称限制为53个字符。 Kubernetes 1.3及更早版本限制为24个字符 (名称长度是14个字符)。  
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```
 
只渲染模板 `helm install --debug --dry-run goodly-guppy ./mychart`   使用--dry-run会让你变得更容易测试

- 内置对象  
  - Release： Release对象描述了版本发布本身。包含了以下对象
  - Release.Name： release名称
  - Release.Namespace： 版本中包含的命名空间(如果manifest没有覆盖的话)
  - Release.IsUpgrade： 如果当前操作是升级或回滚的话，该值将被设置为true
  - Release.IsInstall： 如果当前操作是安装的话，该值将被设置为true
  - Release.Revision： 此次修订的版本号。安装时是1，每次升级或回滚都会自增
  - Release.Service： 该service用来渲染当前模板。Helm里始终Helm


- values 文件
  其内容来自于多个位置：
  - chart中的values.yaml文件 
  - 如果是子chart，就是父chart中的values.yaml文件
  - 使用-f参数(helm install -f myvals.yaml ./mychart)传递到 helm install 或 helm upgrade的values文件
  - 使用--set (比如helm install --set foo=bar ./mychart)传递的单个参数
    优先级为values.yaml最低，--set参数最高。  
  - values实例 `{{ .Values.favoriteDrink }}`  
- NOTES.txt
一个示例
```
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}
```

- .helmignore
  .helmignore 文件用来指定你不想包含在你的helm chart中的文件。

如果该文件存在，helm package 命令会在打包应用时忽略所有在.helmignore文件中匹配的文件。

这有助于避免不需要的或敏感文件及目录添加到你的helm chart中。
这里是一个.helmignore文件示例：
```
# comment

# Match any file or path named .helmignore
.helmignore

# Match any file or path named .git
.git

# Match any text file
*.txt

# Match only directories named mydir
mydir/

# Match only text files in the top-level directory
/*.txt

# Match only the file foo.txt in the top-level directory
/foo.txt

# Match any file named ab.txt, ac.txt, or ad.txt
a[b-d].txt

# Match any file under subdir matching temp*
*/temp*

*/*/temp*
temp?
```

- 模板调试 
- helm lint 是验证chart是否遵循最佳实践的首选工具。
- helm template --debug 在本地测试渲染chart模板。
- helm install --dry-run --debug：我们已经看到过这个技巧了，这是让服务器渲染模板的好方法，然后返回生成的清单文件。
- helm get manifest: 这是查看安装在服务器上的模板的好方法。


