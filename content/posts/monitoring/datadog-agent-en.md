# Preface 

Datadog Application Performance Monitoring (APM) Gain insight into your applications with out-of-the-box performance dashboards for web services, queues, and databases to monitor requests, errors, and latencies. Distributed tracing seamlessly correlates browser sessions, logs, configuration files, synthetic inspections, network, process and infrastructure metrics across hosts, containers, agents and serverless functions. Navigate directly from investigating slow traces to identifying specific lines of code causing code hotspot performance bottlenecks.

The deployment of this article mainly describes the monitoring of k8s services using datadog APM. Generate corresponding monitoring alarm information. By default, the k8s container platform and datadog platform have been installed and deployed.
Select bookinfo-demo for deployment application: https://github.com/solo-cat/bookinfo-demo


2.1 Install Datadog-Agent

To monitor a Kubernetes cluster with Datadog, we first need to install the Datadog agent in the cluster. There are many ways to install datadog-agent, including helm, deamonset, and operator. This article describes how to deploy using helm. For other deployment methods, please refer to the following link: Install the Datadog Agent on Kubernetes (datadoghq.com)

Datadog Agent Deployment and Configuration by helm:

```
helm repo add datadog https://helm.datadoghq.com
helm repo update
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
helm upgrade --install datadog-agent -n datadog --create-namespace -f datadog-values.yaml datadog/datadog
```

Datadog Web Console : https://app.datadoghq.eu/account/login/


This is because the APM agent running in the container communicates with the Datadog cluster agent through the TCP port. 
* datadog.apm.portEnabledtrue If the operating system is not set in , add it to this command. helm upgrade -f values.yaml <RELEASE NAME> datadog/datadogvalues.yaml --set targetSystem=linux --set targetSystem=windows
Warning: This parameter will open a port on the host. Make sure your firewall only allows access from applications or trusted sources. If your network plugin does not support , add a proxy pod specification. This shares the host's network namespace with the Datadog agent. This also means that all ports opened on the container are also opened on the host. If ports are used in both the host and the container, they will conflict (since they share the same network namespace) and the Pod will not start.

# Testing

Use scripts to trigger corresponding microservices to generate corresponding traffic

1. Start the linux microservice

kubectl apply -f apline-cli.yaml

2. Enter the container to trigger the corresponding script

```
kubectl exec -t -i alpine -- sh
while true; do curl http://details:9080; done
while true; do curl http://ratings:9080; done
while true; do curl http://reviews:9080; done
while true; do curl http://productpage:9080; done
```
