# 开源的容器可观测平台维护手册

# 前提

1. 熟悉linux系统基本使用
2. 熟悉ssh,ansible
3. 具备基础编程知识: python3, shell, jinjia2模版
4. 需要拥有一个域名以及对应域名https证书 

# 准备

1. 准备一台linux主机，安装软件: git ssh-client ansible
2. 执行命令克隆仓库: `git clone git@gitlab.apollo-ev.com:ops/playbook.git`
 
# 部署

## 部署 ObservabilityServer 服务端

ansible-playbook -i hosts/inventory jobs/init_k3s_cluster -D
ansible-playbook -i hosts/inventory jobs/init_harbor -D
ansible-playbook -i hosts/inventory jobs/init_keycloak -D
ansible-playbook -i hosts/inventory jobs/init_observability-server -D

## 部署 ObservabilityAgent 采集端

## Delpoy Test
```
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring -D -C
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_sit -D -C
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_uat -D -C
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_common -D -C
```

## Deploy

```
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring -D
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_sit -D
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_uat -D
ansible-playbook -i hosts/aws-hosts jobs/init_ec2_monitoring_common -D
```

## Post Setup

```
kubectl edit  deployment.apps/observability-server-prometheus-server -n  observability
- args:
  - --enable-feature=remote-write-receiver
```

## Troubleshooting

ansible -i  hosts/aws-hosts sit -m shell -a 'sudo pkill -9 prometheus'

# 配置参考

## 配置Grafana
## 配置指标采集
## 配置日志采集
## 配置告警
