# Kubernetes包管理器——Helm

## helm简介

### 为什么需要helm

在没使用helm之前，向`kubernetes`部署应用，我们要依次部署`deployment`,`service`,`configMap`等,步骤较繁琐。况且随着很多项目微服务化，复杂的应用在容器中部署以及管理显得较为复杂.

`helm`通过打包的方式，支持发布的版本管理和控制,很大程度上简化了`Kubernetes`应用的部署和管理

### helm中几个概念

`Helm`可以理解为`Kubernetes`的包管理工具，可以方便地发现、共享和使用为`Kubernetes`构建的应用，它包含几个基本概念

- **Chart**: 一个Helm包，其中包含了运行一个应用所需要的镜像、依赖和资源定义等，还可能包含Kubernetes集群中的服务定义

> 可以理解为docker的image

- **Release**: 在`Kubernetes`集群上运行的 `Chart`的一个实例。在同一个集群上，一个 `Chart`可以安装很多次。每次安装都会创建一个新的`release`

> 可以理解为docker的container实例

- **Repository**: 用于发布和存储 Chart 的仓库

**Helm包含两个组件：Helm客户端和Tiller服务器** 。如下图所示：

![k8vx8x0o9x](Kubernetes包管理器——Helm.assets/k8vx8x0o9x.png)

> Helm客户端负责chart和release的创建和管理以及和Tiller的交互。Tiller服务器运行在k8s集群中，它会处理Helm客户端的请求，与k8s API Server进行交互。

### helm 用途

做为`Kubernetes`的一个包管理工具，Helm具有如下功能：

- 创建新的`chart`

- `chart`打包成`tgz`格式

- 上传`chart`到`chart`仓库或从仓库中下载 `chart`

  - 官方`chart`仓库是: [https://hub.helm.sh](https://hub.helm.sh/)
- 在`Kubernetes`集群中安装或卸载`chart`

- 用`Helm`管理安装的`chart`的发布周期

## helm 部署

- 注意：这里安装的是`helm v3.2.4`，如需下载更新的版本，可以至github官方repo选择

> 官方下载地址：https://github.com/helm/helm/releases

![image-20201202164820477](Kubernetes包管理器——Helm.assets/image-20201202164820477.png)

```shell
# 如无需更换版本，直接执行下载
wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz

# 解压
tar -zxvf helm-v3.2.4-linux-amd64.tar.gz

# 进入到解压后的目录
cd linux-amd64/

# 移动
cp helm /usr/local/bin/

# 查看版本
helm version
```

## helm 详解

可以直接使用官方的chart仓库或者其他仓库来安装一些`chart`

> [https://hub.helm.sh](https://hub.helm.sh/)
>
> `chart仓库：`https://artifacthub.io/

### helm 使用

**以官方仓库的一个redis为例**

1. 访问 `Helm` 仓库：https://artifacthub.io/
2. 搜索需要的包
3. 按照 `install` 步骤操作即可

![å¨è¿éæå¥å¾çæè¿°](Kubernetes包管理器——Helm.assets/20201119102952506.png)

![img](Kubernetes包管理器——Helm.assets/20201119103129715.png)

### helm 自定义模板

#### 文件目录结构

```sh
.
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   └── service.yaml
└── values.yaml
```

#### 自定义chart的示例

**第一步: 准备自定义chart相关文件**

```shell
# 1. 新建myapp文件夹存放chart
[root@k8s-master01 ~]# mkdir myapp
[root@k8s-master01 ~]# cd myapp/
```

```shell
# 2. 新建Chart.yaml
cat << EOF > Chart.yaml
name: hello-world
version: 1.0.0
EOF
```

```shell
# 3. 新建./templates/deployment.yaml
[root@k8s-master01 myapp]# mkdir templates
cat <<'EOF' > ./templates/deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: hello-world
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          ports:
            - containerPort: 80
              protocol: TCP
EOF
```

```shell
# 4. 新建./templates/service.yaml
cat <<'EOF' > ./templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: hello-world
EOF
```

```shell
# 5. 新建values.yaml
cat <<'EOF' > ./values.yaml
image:
  repository: qianzai/k8s-myapp
  tag: 'v1'
EOF
```

**第二步: 使用上面的自定义chart**

```shell
# 将chart实例化成release
# 格式：helm install [RELEASE-NAME] [CHART-PATH]
helm install testname .

# 查看release
helm ls

# 安装成功！！
```

#### helm 命令详解

###### helm install

`helm install`——chart 安装

```shell
helm install [NAME] [CHART] [flags]
```

- `-f, --values`：指定 `values.yaml` 文件
- `--set`：在命令行中直接设置 `values` 的值
- `--dry-run`：模拟执行，测试能不能创建，但不创建
- `--debug`：允许冗长的输出（输出多余信息）

###### helm upgrade

helm upgrade——chart 升级为一个新版本

```shell
helm upgrade [RELEASE] [CHART] [flags]
```

> 发现helm v2，v3 版本不一样，命令差别挺大，详细请查看官方文档：https://helm.sh/zh/docs/helm/

## Helm 部署 dashboard

```shell
# 1.添加仓库
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard

# 2.下载 Chart
helm pull kubernetes-dashboard/kubernetes-dashboard
tar -zxvf kubernetes-dashboard

# 3. 创建 kubernetes-dashboard.yaml
```

`kubernetes-dashboard.yaml`

```yaml
image:
  repository: kubernetesui/dashboard
  tag: v2.0.4
ingress:
  enabled: true
  hosts:
    - k8s.frognew.com
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  tls:
    - secretName: frognew-com-tls-secret
      hosts:
        - k8s.frognew.com
rbac:
  clusterAdminRole: true
```

```shell
# 4.安装
helm install kubernetes-dashboard . \
--namespace kube-system \
-f kubernetes-dashboard.yaml

# 5. 更改 svc
$ kubectl edit svc kubernetes-dashboard -n kube-system
将 spec.type 的值修改为 NodePort

# 查看端口（端口为：30354）
$ kubectl get svc -n kube-system
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
kubernetes-dashboard   NodePort    10.102.46.217   <none>        443:32671/TCP            4m46s
```

最后访问地址：https://192.168.200.61:32671

![image-20201210173720061](Kubernetes包管理器——Helm.assets/image-20201210173720061.png)

> 可以发现有两种方式验证，下面查看token方式

```shell
# 查看 kubernetes-dashboard-token
 kubectl get secret -n kube-system | grep kubernetes-dashboard-token
 
 kubectl describe secret -n kube-system kubernetes-dashboard-token-bqtxz
```

![image-20201210174143616](Kubernetes包管理器——Helm.assets/image-20201210174143616.png)

> 将token复制进去，即可登录成功

![image-20201210174609176](Kubernetes包管理器——Helm.assets/image-20201210174609176.png)

> 登录成功后，进去界面，发现什么资源都显示不了，是因为 `dashboard` 默认的 `serviceaccount` 并没有权限，所以我们需要给予它授权。

创建 `dashboard-admin.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

```shell
[root@k8s-master01 ~]# kubectl apply -f dashboard-admin.yaml 
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
```

> 授权成功，就可以查看到资源了

![image-20201210175221863](Kubernetes包管理器——Helm.assets/image-20201210175221863.png)



## 其他

[玩K8S不得不会的HELM](https://zhuanlan.zhihu.com/p/79046244)

