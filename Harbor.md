# Harbor
レジストリとしてHarborを立てる

## サーバ
dockerが入ってるサーバ  
主にk8sのレジストリとして使う

| ホスト名          | IPアドレス       |
|:-----------       |:------------     |
|zspt-k8scf01.local |192.168.2.170/24  |

## 構築
参考  
https://github.com/vmware/harbor  
https://github.com/vmware/harbor/blob/master/docs/installation_guide.md  

### インストーラをDL
~~~
https://github.com/vmware/harbor/releases
-harbor-online-installer-v1.4.0.tgz

# tar xvf harbor-online-installer-v1.4.0.tgz

~~~

### オレオレSSL証明書作成
https://github.com/vmware/harbor/blob/master/docs/customize_token_service.md  
http://d.hatena.ne.jp/ozuma/20130511/1368284304
~~~
# mkdir -p  /data/cert/
# cd /data/cert/

# openssl genrsa -out server.key 4096 

# openssl req -new -x509 -key server.key -out server.crt -days 36500

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:jp
State or Province Name (full name) []:keita mochizuki
Locality Name (eg, city) [Default City]:tokyo
Organization Name (eg, company) [Default Company Ltd]:local
Organizational Unit Name (eg, section) []:section
Common Name (eg, your name or your server's hostname) []:reg.harbor.local
Email Address []:mochiduki@example.com

~~~
### SSL証明書設置
~~~
mkdir cert
cp server.key cert/
cp server.crt cert/
~~~



### 設定

harbor/harbor.cfg
~~~
## Configuration file ofuth_mode Harbor

#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname = reg.harbor.local ###★変更

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = https ###★変更

#Maximum number of job workers in job service
max_job_workers = 3

#Determine whether or not to generate certificate for the registry's token.
#If the value is on, the prepare script creates new root cert and private key
#for generating token to access the registry. If the value is off the default key/cert will be used.
#This flag also controls the creation of the notary signer's cert.
customize_crt = on ###★デフォルト

#The path of cert and key files for nginx, they are applied only the protocol is set to https
ssl_cert = /data/cert/server.crt  ###★変えるとバグる(Bad Gateway 502)
ssl_cert_key = /data/cert/server.key  ###★変えるとバグる(Bad Gateway 502)

#The path of secretkey storage
secretkey_path = /data  ###★変えるとバグる(Bad Gateway 502)

#Admiral's url, comment this attribute, or set its value to NA when Harbor is standalone
admiral_url = NA

#Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
log_rotate_count = 50
#Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
#If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
#are all valid.
log_rotate_size = 200M

#NOTES: The properties between BEGIN INITIAL PROPERTIES and END INITIAL PROPERTIES
#only take effect in the first boot, the subsequent changes of these properties
#should be performed on web ui

#************************BEGIN INITIAL PROPERTIES************************

#Email account settings for sending out password resetting emails.

#Email server uses the given username and password to authenticate on TLS connections to host and act as identity.
#Identity left blank to act as username.
email_identity =

email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

##The initial password of Harbor admin, only works for the first time when Harbor starts.
#It has no effect after the first launch of Harbor.
#Change the admin password from UI after launching Harbor.
harbor_admin_password = 09480948  ###★変更

##By default the auth mode is db_auth, i.e. the credentials are stored in a local database.
#Set it to ldap_auth if you want to verify a user's credentials against an LDAP server.
auth_mode = db_auth  ###★デフォルト

#The url for an ldap endpoint.
ldap_url = ldaps://ldap.mydomain.com

#A user's DN who has the permission to search the LDAP/AD server.
#If your LDAP/AD server does not support anonymous search, you should configure this DN and ldap_search_pwd.
#ldap_searchdn = uid=searchuser,ou=people,dc=mydomain,dc=com

#the password of the ldap_searchdn
#ldap_search_pwd = password

#The base DN from which to look up a user in LDAP/AD
ldap_basedn = ou=people,dc=mydomain,dc=com

#Search filter for LDAP/AD, make sure the syntax of the filter is correct.
#ldap_filter = (objectClass=person)

# The attribute used in a search to match a user, it could be uid, cn, email, sAMAccountName or other attributes depending on your LDAP/AD
ldap_uid = uid

#the scope to search for users, 0-LDAP_SCOPE_BASE, 1-LDAP_SCOPE_ONELEVEL, 2-LDAP_SCOPE_SUBTREE
ldap_scope = 2

#Timeout (in seconds)  when connecting to an LDAP Server. The default value (and most reasonable) is 5 seconds.
ldap_timeout = 5

#Verify certificate from LDAP server
ldap_verify_cert = true

#Turn on or off the self-registration feature
self_registration = on

#The expiration time (in minute) of token created by token service, default is 30 minutes
token_expiration = 30

#The flag to control what users have permission to create projects
#The default value "everyone" allows everyone to creates a project.
#Set to "adminonly" so that only admin user can create project.
project_creation_restriction = everyone

#************************END INITIAL PROPERTIES************************

#######Harbor DB configuration section#######

#The address of the Harbor database. Only need to change when using external db.
db_host = mysql

#The password for the root user of Harbor DB. Change this before any production use.
db_password = 09480948  ###★変更

#The port of Harbor database host
db_port = 3306

#The user name of Harbor database
db_user = root

##### End of Harbor DB configuration#######

#The redis server address. Only needed in HA installation.
redis_url =

##########Clair DB configuration############

#Clair DB host address. Only change it when using an exteral DB.
clair_db_host = postgres

#The password of the Clair's postgres database. Only effective when Harbor is deployed with Clair.
#Please update it before deployment. Subsequent update will cause Clair's API server and Harbor unable to access Clair's database.
clair_db_password = password

#Clair DB connect port
clair_db_port = 5432

#Clair DB username
clair_db_username = postgres

#Clair default database
clair_db = postgres

##########End of Clair DB configuration############

#The following attributes only need to be set when auth mode is uaa_auth
uaa_endpoint = uaa.mydomain.org
uaa_clientid = id
uaa_clientsecret = secret
uaa_verify_cert = true
uaa_ca_cert = /path/to/ca.pem


### Docker Registry setting ###
#registry_storage_provider can be: filesystem, s3, gcs, azure, etc.
registry_storage_provider_name = filesystem
#registry_storage_provider_config is a comma separated "key: value" pairs, e.g. "key1: value, key2: value2".
#Refer to https://docs.docker.com/registry/configuration/#storage for all available configuration.
registry_storage_provider_config =
~~~
### インストール実行
~~~
./install.sh --with-notary --with-clair
~~~

### Harborにアクセス
~~~
https://reg.harbor.local.
~~~

### insecure-registriesの設定
Harborを利用するdockerクライアントに対して以下の設定を入れる  
https://qiita.com/nohhoso/items/28f76604a637791c9deb  
/etc/docker/daemon.json
~~~
{
  "insecure-registries" : ["reg.harbor.local"]
}
~~~
~~~
systemctl daemon-reload
systemctl restart docker
docker info
~~~

~~~
docker login https://reg.harbor.local


~~~

### サーバ再起動時(暫定対処)
ブラウザでアクセスするとBad Gatewayが出るので再度以下を実施する

~~~
./install.sh --with-notary --with-clair
~~~

### 補足
インストール中にディスク容量が不足したためLVM拡張  
https://qiita.com/egnr-in-6matroom/items/d284dd306753b2f0591a
