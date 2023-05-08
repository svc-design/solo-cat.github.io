第1章前言

Datadog应用程序性能监控（APM）通过针对web服务、队列和数据库的开箱即用性能仪表板来监控请求、错误和延迟，从而深入了解您的应用程序。分布式跟踪与主机、容器、代理和无服务器功能之间的浏览器会话、日志、配置文件、合成检查、网络、流程和基础设施指标无缝关联。直接从调查慢速跟踪导航到识别导致代码热点性能瓶颈的特定代码行。
本文部署主要讲述将k8s服务使用datadog APM进行监控。生成相应的监控告警信息。默认已经安装部署好k8s容器平台和datadog平台。
部署应用选择bookinfo-demo：https://github.com/solo-cat/bookinfo-demo

第2章部署APM

2.1安装Datadog-Agent
要使用 Datadog 监控 Kubernetes 集群，我们首先需要在集群中安装 Datadog 代理。安装datadog-agent的方式很多，有helm、deamonset以及operator等，本文讲述使用helm部署的方式部署。其他几种部署方式可以参考以下链接：Install the Datadog Agent on Kubernetes (datadoghq.com)

2.1.1使用helm方式安装部署datadog-agent
安装datadog-agent的方式很多，有helm、deamonset以及operator等，本文讲述使用helm部署的方式部署。默认已经安装好helm。
1、导入helm对应镜像库
helm repo add datadog https://helm.datadoghq.com
helm repo update

2、导入datadog-agent对应的yaml文件
cat > datadog-values.yaml << EOF
targetSystem: "linux"    #选择datadog-agent部署环境                                      
registry: artifact.onwalk.net/public/datadog
clusterAgent:
  enabled: true
  admissionController:
    enabled: true
    mutateUnlabelled: true
datadog:
  site: 'datadoghq.eu'
  apiKeyExistingSecret: datadog-agent
  logs:
    enabled: true
    containerCollectAll: true
  apm:
    portEnabled: true   #代理配置为通过 TCP 接收跟踪。
  networkMonitoring:
    enabled: true
  env:
    - name: DD_APM_FEATURES
      value: 'enale_cid_stats'
EOF
这是因为容器中运行的 APM 代理通过 TCP 端口与 Datadog 集群代理进行通信。datadog.apm.portEnabledtrue
如果未在 中设置操作系统，请将 添加到此命令中。helm upgrade -f values.yaml <RELEASE NAME> datadog/datadogvalues.yaml--set targetSystem=linux--set targetSystem=windows
警告：该参数将在主机上打开一个端口。确保您的防火墙仅允许来自应用程序或受信任来源的访问。如果您的网络插件不支持 ，请添加代理 Pod 规范。这会与 Datadog 代理共享主机的网络命名空间。这也意味着在容器上打开的所有端口都在主机上打开。如果在主机和容器中同时使用端口，则它们会冲突（因为它们共享相同的网络命名空间），并且 Pod 不会启动。
3、创建datadog-agent
创建名为dadadog的namespace，在对应的namespace中部署想要的datadog-agent微服务。
kubectl create namespace datadog 
kubectl delete secret datadog-agent --namespace=datadog 
kubectl create secret generic datadog-agent --from-literal api-key=‘xxxxxxxxxxxxxxxxxxx --namespace=datadog
helm upgrade --install datadog-agent -n datadog --create-namespace -f datadog-values.yaml datadog/datadog
datadog-agent对应的key需要到datadog页面进行生成，然后导入。


2.2安装DEMO
安装的bookinfo文件，针对不通环境写入不同代码，下图是bookinfo微服务的架构图，通过ingress访问前端product page前端微服务，前端会调用后端java架构和ruby架构的微服务。

Bookinfo微服务主要分为details、productpage、ratings以及reviews。分别使用了不同的代码。根据不同的编程语言使用的方式是不一样的，根据本文该案例，我们使用了不同的代码实现。
2.2.1Ruby代码
需要在对应的yaml文件中添加以下

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: <env>
    tags.datadoghq.com/service: <service>
    tags.datadoghq.com/version: <version>
spec:
  template:
    metadata:
      labels:
        tags.datadoghq.com/env: <env>
        tags.datadoghq.com/service: <service>
        tags.datadoghq.com/version: <version>
        admission.datadoghq.com/enabled: "true"
      annotations:
        admission.datadoghq.com/ruby-lib.version: v1.11.1
    spec:
      containers:
        - name: <CONTAINER_NAME>
          image: <CONTAINER_IMAGE>/<TAG>
          env:
            - name: DD_AGENT_HOST
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
kubectl apply -f bookinfo-details.yaml

2.2.2Python
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: <env>
    tags.datadoghq.com/service: <service>
    tags.datadoghq.com/version: <version>
spec:
  template:
    metadata:
      labels:
        tags.datadoghq.com/env: <env>
        tags.datadoghq.com/service: <service>
        tags.datadoghq.com/version: <version>
        admission.datadoghq.com/enabled: "true"
      annotations:
        admission.datadoghq.com/python-lib.version: v1.12.4
    spec:
      containers:
        - name: <CONTAINER_NAME>
          image: <CONTAINER_IMAGE>/<TAG>
          env:
            - name: DD_LOGS_INJECTION
              value: "true"
kubectl apply -f bookinfo-productpage-python.yaml

2.2.3Node

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: <env>
    tags.datadoghq.com/service: <service>
    tags.datadoghq.com/version: <version>
spec:
  template:
    metadata:
      labels:
        tags.datadoghq.com/env: <env>
        tags.datadoghq.com/service: <service>
        tags.datadoghq.com/version: <version>
        admission.datadoghq.com/enabled: "true"
      annotations:
        admission.datadoghq.com/js-lib.version: ""
    spec:
      containers:
        - name: <CONTAINER_NAME>
          image: <CONTAINER_IMAGE>/<TAG>
          env:
            - name: DD_AGENT_HOST
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: DD_LOGS_INJECTION
              value: "true"
kubectl apply -f bookinfo-ratings.yaml

2.2.4Java
在部署它之前，我们需要指示 Datadog 准入控制器改变我们的 Pod。为此，请将以下标签添加到 Pod 定义中。
labels:
  admission.datadoghq.com/enabled: "true"
为了指示准入控制器将 Java APM 库注入容器，我们必须将以下注释添加到 Pod 中。
annotations:
  admission.datadoghq.com/java-lib.version: "latest"

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: <env>
    tags.datadoghq.com/service: <service>
    tags.datadoghq.com/version: <version>
spec:
  template:
    metadata:
      labels:
        tags.datadoghq.com/env: <env>
        tags.datadoghq.com/service: <service>
        tags.datadoghq.com/version: <version>
        admission.datadoghq.com/enabled: "true" 
      annotations:
        admission.datadoghq.com/java-lib.version: v1.13.0
    spec:
      containers:
        - name: <CONTAINER_NAME>
          image: <CONTAINER_IMAGE>/<TAG>
          env:
            - name: DD_LOGS_INJECTION
              value: "true"

kubectl apply -f bookinfo-reviews.yaml

第3章测试
使用脚本触发相应微服务产生相应的流量
1、启动linux微服务
kubectl apply -f apline-cli.yaml
2、进入容器触发相应脚本
kubectl  exec -t -i alpine -- sh
脚本内容
while true; do curl http://details:9080; done
while true; do curl http://ratings:9080; done
while true; do curl http://reviews:9080; done
while true; do curl http://productpage:9080; done
