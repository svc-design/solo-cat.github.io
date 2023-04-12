# AWS IAM Identity Center 与keycloak OIDC Provider 关联实现SSO

# 部署SSO 服务
1. openldap
2. keycloak

##  Keycloak 服务配置 	

### Create Realm

1. https://keycloak.onwalk.net/admin/master/console/#/master/add-realm
  Realm name * : SSO
  Enabled : On
2. 点击 Create 完成 Realm创建， 
3. 选择SSO , 左侧菜单栏 -> realm-settings -> General ：
    1. 点击 Endpoints，选择OpenID Endpoint Configuration   记录返回的信息中” issuer":"https://keycloak.onwalk.net/realms/SSO"
    2. 设置 identity_providers需要使用这个地址。例如（在AWS IAM 添加身份供应商） 
4. realm-settings 参考参考

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
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Principal": {
                "Federated": "arn:aws:iam::507321523889:oidc-provider/keycloak.onwalk.net/realms/SSO"
            },
            "Condition": {
                "StringEquals": {
                    "keycloak.onwalk.net/realms/SSO:aud": [
                        "aws-console"
                    ]
                }
            }
        }
    ]
}
```


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
    * 
    * 现在，您应该能够通过Keycloak OIDC Provider登录AWS IAM Identity Center并访问您授予的AWS服务。



# 登录 keycloak.onwalk.net

## Setup User federation -> Add LDAP provider

```
* General options
  UI display name:  ldap
  Vendor                  : other

* Connection and authentication settings
  Connection URL  *     : ldap://ldap.onwalk.net
  Enable StartTLS        : Off
  Use Truststore SPI    : Never
  Connection pooling   : Off
  Bind type *                   : simple
  Bind DN *                      : cn=admin,dc=onwalk,dc=net
  Bind credentials *        : <ldap_admin_password>
  
* LDAP searching and updating
  Edit mode *                            : READ ONLY
  Users DN *                             : ou=users,dc=onwalk,dc=net
  Username LDAP attribute *  : cn
  RDN LDAP attribute *            : cn
  UUID LDAP attribute *           : entryUUID
  User obiect classes *            : inetOrgPerson, organizationalPerson
  Search scope                         : Subtree
```	
其他选项默认，保存即可 最后执行 Action -> Sync all users
