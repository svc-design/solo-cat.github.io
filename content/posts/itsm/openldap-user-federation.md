# AWS IAM Identity Center 与keycloak OIDC Provider 关联实现SSO

## 部署openldap

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

