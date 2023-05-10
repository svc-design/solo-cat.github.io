# IT基础服务部署

# 前提

1. 熟悉ssh,ansible
2. 熟悉linux系统基本使用
3. 具备基础编程知识: python3, shell, jinjia2模版
4. 需要拥有一个域名以及对应域名https证书 

# 准备

1. 准备一台linux主机, 配置ssh key 免密码登录 
2. 本地电脑安装软件: git ssh ansible
2. 本地电脑执行命令克隆仓库: `git clone git@gitlab.apollo-ev.com:ops/playbook.git`
 
# 部署

## 初始化 K3S

ansible-playbook -i hosts/inventory jobs/init_k3s_cluster -D

## 部署 Harbor 服务

ansible-playbook -i hosts/inventory jobs/init_harbor -D

## 部署 Keycloak 服务

ansible-playbook -i hosts/inventory jobs/init_keycloak -D

## 部署 openldap 服务

ansible-playbook -i hosts/inventory jobs/init_keycloak -D


# 配置参考

## 配置Grafana
## 配置指标采集
## 配置日志采集
## 配置告警
