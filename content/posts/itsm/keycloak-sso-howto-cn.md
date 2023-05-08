# AWS IAM Identity Center 与keycloak OIDC Provider 关联实现SSO

# 登录 keycloak.onwalk.net

基础设置

1. 登录keycloak
2. 点击左侧菜单栏中的Realm Settings
3. 点击Themes，设置所有属性为keycloak
4. Internationalization Enabled选择ON
5. 将Default Locale设置为zh-CN
6. 重新登录时，在登录页面会显示语言切换项


Authentication ->  required-actions :
* configure-opt -> Set as default action : ON
* authentication/policies ->  Reusable token : ON

master/users -> 必需的用户操作
* configure-opt

https://keycloak.apollo-ev.com/admin/master/console/#/master/realm-settings   ## 新建 realm 

## Setup User federation -> Add LDAP provider

General options
	UI display name:  ldap
         Vendor                  : other

Connection and authentication settings
	Connection URL  *     : ldap://ldap.onwalk.net
         Enable StartTLS        : Off
         Use Truststore SPI    : Never
         Connection pooling   : Off
         Bind type *                   : simple
        Bind DN *                      : cn=admin,dc=onwalk,dc=net
        Bind credentials *        : <ldap_admin_password>
  
LDAP searching and updating
	Edit mode *                            : READ ONLY
        Users DN *                             : ou=users,dc=onwalk,dc=net
        Username LDAP attribute *  : cn
	RDN LDAP attribute *            : cn
	UUID LDAP attribute *           : entryUUID
	User obiect classes *            : inetOrgPerson, organizationalPerson
        Search scope                         : Subtree
	
其他选项默认，保存即可 最后执行 Action -> Sync all users

# Keycloak 服务配置 	
## Create Realm

1. https://keycloak.onwalk.net/admin/master/console/#/master/add-realm
	Realm name *                       : SSO
	Enabled                                   : On
2. 点击 Create 完成 Realm创建， 
3. 选择SSO , 左侧菜单栏 -> realm-settings -> General ：
    1. 点击 Endpoints，选择OpenID Endpoint Configuration   记录返回的信息中” issuer":"https://keycloak.onwalk.net/realms/SSO"
    2. 设置 identity_providers需要使用这个地址。例如（在AWS IAM 添加身份供应商） 
4. realm-settings 参考参考

## Realm roles
1. 选择SSO , 左侧菜单栏 -> Realm roles -> Create role
    1. Role name * :  sso-role-admin

##  Keycloak  Server 配置
1. 导出 https://keycloak.apollo-ev.com/realms/cloud-sso/protocol/saml/descriptor 保存为 keycloak-sso-saml.xml 
3. 创建  AWS CN SAML Client  导入从 AWS CN 下载https://signin.amazonaws.cn/static/saml-metadata.xml
4. 导入客户端配置 
    1.  IDP-Initiated SSO URL: aws-cn
    2.  Home URL:  https://keycloak.apollo-ev.com/realms/cloud-sso/protocol/saml/clients/aws-cn
## AWS CN Console 配置
1. IAM -> identity_providers: 
    1. 添加身份提供商 -> 配置提供商
    2. 提供商类型: SAML
    3. 提供商名称: cloud-sso
    4. 导入 Keycloak  Server 配置 .1  步骤导出的 keycloak-sso-saml.xml 文件
    5. 绑定一个IAM角色，并分配权限
        1. 选择受信任实体的类型： SAML 2.0 身份联合
        2. SAML 提供商： cloud-sso
        3. 允许编程访问和 亚马逊云科技 管理控制台访问
        4. Attach 权限策略： 选择一个需要的权限策略
        5. 角色名称：aws-saml-admin
        6. 创建角色, 记录ARN

## Keycloak  Server 配置

    1. 
    2.  Realm roles -> Create role 输入上一步记录的ARN 
        1. <aws-cn-sso-role-arn>, <aws-cn-sso-aml-provider-arn>
        2. arn:aws-cn:iam::155742804536:role/aws-saml-admin, arn:aws-cn:iam::155742804536:saml-provider/cloud-sso
    3.  Create a group -> 
        1. 导入用户
        2. Assign roles 上一步骤的角色
    4. 
        1. Clien-> urn:amazon:webservices:cn-north-1 -> 允许的全范围: Off


# Create Users
1. 选择SSO , 左侧菜单栏 -> users -> Add user
    1. Username *：sso-test
    2. Email           :   sso-test@domain.com
2. 选择SSO , 左侧菜单栏 -> users -> 
    1. 选项卡credentials 设置密码
    2. 


## 创建  OIDC AWS Client  
1. 选择SSO , 左侧菜单栏 -> clients -> Create client
2. Create client 
    1. Client type : OpenID Connect  
    2. Client ID *   :  aws-console
    3. Name          :  aws-console
3. Client authentication 
    1. Client authentication : On
    2. Authorization                : On
    3. Authentication flow    : 全部选择 
4. Loggin Settings: 
    1. Valid redirect URIs     : https://signin.aws.amazon.com/oauth2/callback
    2. 其他参数保持默认

点击 Save 完成 OIDC AWS Client 创建

# AWS SSO 服务配置 
1. AWS IAM 添加身份供应商， IAM -> 访问管理 -> 身份供应商 -> 添加提供商
    1. 提供商类型，选择  OpenID Connect
    2. 提供商 URL，添加  https://keycloak.onwalk.net/realms/SSO 点击获取指纹
    3. 受众： aws-console 
添加提供商，完成添加身份供应商配置

2. AWS IAM 新增角色， 创建角色
    1. 选择可信实体 -> 可信实体类型 : Web 身份
    2. 身份提供商 -> 选择上一步骤添加的身份提供商（keycloak.onwalk.net/realms/SSO ） 
    3. Audience ->  选择 aws-console
3. 添加权限，选择一个需要的策略，填入角色名称：sso-role-admin
4. IAM-ROLE Json参考

## Gitlab/keycloak SSO
 https://git.xcvtc.edu.cn/help/administration/auth/oidc.md

部署 keycloak ingress CM

kubectl  edit cm -n ingress nginx-nginx-ingress -o yaml

data:
  client-body-buffer-size: 64k
  client-header-buffer-size: 64k
  client-max-body-size: 10m
  external-status-address: 10.1.10.110
  proxy-body-size: 1024m
  proxy-buffer-size: 32k
  proxy-buffers: 8 32k
  proxy-connect-timeout: 10s
  proxy-read-timeout: 10s  解决502问题
  
  appConfig:
    omniauth:
      enabled: true
      autoLinkLdapUser: false
      autoLinkSamlUser: false
      autoSignInWithProvider: null
      blockAutoCreatedUsers: false
      autoLinkUser:
        - 'openid_connect'
      allowSingleSignOn:
        - 'openid_connect'
      providers:
      - secret: gitlab-sso-secret
        key: provider

cat > provider.yaml << EOF
name: 'openid_connect'
label: 'keycloak-sso'
args:
  name: 'openid_connect'
  scope:
    - 'openid'
    - 'profile'
    - 'email'
  pkce: true
  discovery: true
  response_type: 'code'
  client_auth_method: 'query'
  send_scope_to_token_endpoint: true
  issuer: 'https://keycloak.apollo-ev.com/realms/cloud-sso'
  client_options:
    identifier: 'gitlab-oidc'
    secret: 'dkGUOmOEMTK4261LtcjHYE6MaOOJ2h9t'
    redirect_uri: 'https://gitlab.apollo-ev.com/users/auth/openid_connect/callback'
EOF

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm repo add gitlab https://charts.gitlab.io/
helm repo up
kubectl delete secret gitlab-sso-secret -n gitlab || echo true
kubectl create secret generic gitlab-sso-secret --from-file="provider=provider.yaml" -n gitlab
helm upgrade --install gitlab gitlab/gitlab --namespace gitlab -f gitlab-values.yaml


## Keycloak/ Harbor

\c registry
select * from harbor_user;
delete  from harbor_user where username='haha#4'; 


要将AWS IAM Identity Center与Keycloak OIDC Provider关联以实现单点登录(SSO)，您需要执行以下步骤：
* 在AWS IAM中创建身份提供者： 
    * a. 登录AWS控制台，选择IAM服务，然后选择“身份提供者”选项卡。 
    * b. 点击“创建提供者”，选择“OpenID Connect提供者”。 
    * c. 输入提供者的名称和描述，然后输入Keycloak的OIDC配置信息（包括授权和令牌端点URL、客户端ID和密钥等）。
    * d. 点击“下一步”，然后查看提供者的详细信息，确保它们正确无误，最后单击“创建提供者”。
* 在Keycloak中创建客户端： 
    * a. 登录Keycloak控制台，选择您的Realm
    * b. 点击“客户端”选项卡，然后单击“创建” 
    * c. 输入客户端的名称和描述，然后选择“OpenID Connect”作为客户端协议。
    * d. 在“有效的重定向URI”字段中，输入AWS IAM身份提供者的重定向URI（例如：https://signin.aws.amazon.com/oauth2/callback)
    * e. 点击“保存”，然后查看客户端的详细信息，确保它们正确无误。
* 在AWS IAM中创建角色： 
    * a. 登录AWS控制台，选择IAM服务，然后选择“角色”选项卡。 
    * b. 点击“创建角色”，选择“受信任实体类型”为“AWS服务”，“选择您的身份提供者”，然后选择“允许假定角色”。
    * c. 在“选择允许假定角色的实体”字段中，选择Keycloak OIDC提供者。 
    * d. 在“选择您要为其授予此角色的AWS服务”字段中，选择您想要授予访问权限的AWS服务。
    * e. 点击“下一步”，然后为角色命名并添加策略，最后单击“创建角色”。
* 在Keycloak中创建用户： 
    * a. 登录Keycloak控制台，选择您的Realm
    * b. 点击“用户”选项卡，然后单击“添加用户”。
    * c. 输入用户的详细信息，然后单击“保存”。
    * d. 在“凭据”选项卡中，为用户创建密码或选择其他身份验证方法。
    * e. 在“角色映射”选项卡中，将Keycloak客户端的角色映射到AWS IAM角色。  
    * 现在，您应该能够通过Keycloak OIDC Provider登录AWS IAM Identity Center并访问您授予的AWS服务。
