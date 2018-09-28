title: HTTPS证书及Nginx配置相关
author: Peng Fang
tags:
  - HTTPS
categories:
  - Nginx
date: 2018-06-08 18:11:00
---
本文档包括：  
Nginx支持HTTPS，自签名证书生成，配置Nginx支持HTTPS，客户端证书导入。   
作者：方鹏
## Nginx安装SSL模块
1. 安装依赖包
``` shell
yum -y install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel
```
2. 执行编译
解压安装包，进入目录，执行命令：
``` shell
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module
make install
```
## 自签名证书生成  
>     [root@vultr ~]# openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout bcp.key -out bcp.crt
>     Generating a 2048 bit RSA private key
>     ...................................................>     ................................+++
>     ....................+++
>     writing new private key to 'bcp.key'
>     ------
>     You are about to be asked to enter information that will be incorporated into your certificate request.
>     What you are about to enter is what is called a Distinguished Name or a DN.
>     There are quite a few fields but you can leave some blank For some fields there will be a default value, If you enter '.', the field will be left blank.
>     Country Name (2 letter code) [XX]:CN
>     State or Province Name (full name) []:SICHUAN
>     Locality Name (eg, city) [Default City]:CHENGDU
>     Organization Name (eg, company) [Default Company Ltd]:MIGU
>     Organizational Unit Name (eg, section) []:TSG 
>     Common Name (eg, your name or your server's hostname) []:10.146.50.22 #注意：这里填服务端的IP地址
>     Email Address []:fangpeng@address.cn


## 配置Nginx支持HTTPS
``` shell
# HTTPS server
    server {
        listen       443 ssl;
        server_name  localhost;
        ssl on;
        ssl_certificate      /usr/local/nginx/key/bcp.crt; #证书文件存放路径
        ssl_certificate_key  /usr/local/nginx/key/bcp.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_set_header X-NginX-Proxy true;
            proxy_pass http://127.0.0.1:8080/; #后端服务地址
            proxy_redirect off;
        }
    }
```
## 客户端证书导入
如果在调用服务的客户端没有导入自签名证书，那么可能在接口调用时会出现如下错误：
```
sun.security.validator.ValidatorException: PKIX path building 
failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```
导入证书：
```
keytool -import -alias ${alias} -keystore ${JAVA_HOME}/jre/lib/security/cacerts -file ${path-to-certificate-file}
```
```
cd ${JAVA_HOME}/jre/lib/security/
sudo keytool -import -alias 5022-cert -keystore cacerts -file ~/server.crt
```
server.crt：服务端自签名的证书文件  
keystore密码：默认changit   
如果导入过证书，alias相同，会出现错误提示：  
`Certificate not imported, alias <xxx> already exists.`  
从keystone删除证书：
```
sudo keytool -delete -keystore cacerts -alias 'bcp-cert'
```
导入后调用依然出现异常：
```
javax.net.ssl.SSLException: Certificate for XXX doesn't match common name of the certificate subject: XXX
```
原因：需要连接服务器的域名或IP和证书中的common name不一致。
重新生成证书，Common Name 填服务器IP。

> 证书文件也可以通过命令请求远端站点获取

1. 获取远程网站（服务器）的根证书和中间证书
```
openssl s_client -showcerts -connect 10.146.50.22:443
```
2. 保存证书文件  
保存证书hash（上图中包含--BEGIN CERTIFICATE--到--END CERTIFICATE--部分）
为文件，例如server.crt
3. 证书导入keystore  
使用keytool import 命令导入根证书和中间证书到JAVA信任的根证书中（通常叫cacerts)
```
sudo keytool -importcert -keystore ${JAVA_HOME}/jre/lib/security/cacerts -storepass changeit -file ~/server.crt -alias "5022-cert"
```
证书添加到keystore   
[参考文章](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html)
