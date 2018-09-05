---
layout: post
title: 编译安装OpenLDAP, phpldapadmin
---

一  软件版本
    openldap-2.4.46
    berkeley-db-5.3.21


二  安装BDB
    tar -zxf db-5.3.21.tar.gz 
    cd db-5.3.21/build_unix
    ../dist/configure --prefix=/usr/local/BDB5.3.21
    make && make install

三  安装OpenLDAP
    tar xvzf openldap-2.4.46.tgz
    env CPPFLAGS="-I/usr/local/BDB5.3.21/include" LDFLAGS="-L/usr/local/BDB5.3.21/lib" ./configure --prefix=/usr/local/openldap --with-tls=openssl
    make && make depend && make install

    注意：需要先安装openssl-devel， 不然会出错
          可以使用ldd slapd查看是否关联了openssl

四  修改配置文件slapd.conf

    添加生成好的证书，密钥
    TLSCACertificateFile    /usr/local/openldap/etc/cacerts/cacert.pem
    TLSCertificateFile      /usr/local/openldap/etc/cacerts/servercrt.cert
    TLSCertificateKeyFile   /usr/local/openldap/etc/cacerts/serverkey.key

五  启动服务
    ./slapd -f ../etc/openldap/slapd.conf -F ../etc/slapd.d/ -h "ldap://0.0.0.0:389 ldaps://0.0.0.0:636" 
    这是同时启动非加密， 加密两个端口， 便于测试， 生产环境只开启加密服务


六  安装phpldapadmin
    yum install phpldapadmin

    配置nginx, phpldapadmin文件在/usr/share目录
    location /htdocs {
            alias /usr/share/phpldapadmin/htdocs;
            index index.php;
            location ~ \.php$ {
                    alias /usr/share/phpldapadmin;
                    fastcgi_pass  127.0.0.1:9000;
                    fastcgi_index index.php;
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                    include fastcgi_params;
            }

    }

    修改配置config.php
    $servers->setValue('server','host','xxx.xxx.xxx.xxx');
    $servers->setValue('server','base',array('dc=xxx,dc=com'));
    $servers->setValue('login','bind_id','cn=admin,dc=xxx,dc=com');     //openldap里配置的管理员
    $servers->setValue('login','attr','dn');        // uid改为dn
    $servers->setValue('login','allowed_dns',array('cn=admin,dc=xxx,dc=com'));  //只允许管理员登录


    然后用浏览器打开x.x.x.x/htdocs/登录即可
    注意：用户名是完整的dn


***

下面是TLS配置

1  CA中心操作
    1.1  生成CA根密钥
    openssl genrsa -out cakey.pem 2048  //可以加上-des3使用Triple DES算法，但之后调用时要求输入密码

    查看生成的密钥
    openssl rsa --noout -text -in cakey.pem

    基于安全性， 密钥权限建议为600或400

    1.2  生成CA根证书
    openssl req -new -x509 -days 365 -key cakey.pem -out ca.crt
    之后要求输入一些信息， Common Name填主机名或者ip

    查看证书
    openssl x509 --noout -text -in ca.crt

2  openldap服务器端操作
    2.1  生成openldap服务器私钥
    openssl genrsa -out ldapkey.pem

    2.2  生成openldap服务器证书签署请求文件
    //请求文件需要发给CA中心签署生成证书，相当于公钥
    openssl req -new -key ldapkey.pem -out ldapserver.csr

3  CA中心签署证书
    3.1  准备工作
    cp /etc/pki/tls/openssl.cnf ./
    mkdir -p newcerts
    touch index.txt
    echo "00" > serial

    修改openssl.cnf， dir部分， 指向当前配置文件工作目录

    3.2  签署证书
    //将openldap生成的csr文件发给ca
    //如果重新签署证书， 需要将index.txt的内容还原为index.txt.old的内容
    openssl ca -days 365 -cert ca.crt -keyfile cakey.pem -in ldapserver.csr -out ldapsever.crt -config openssl.cnf

    3.3  把相关文件放到四的位置














