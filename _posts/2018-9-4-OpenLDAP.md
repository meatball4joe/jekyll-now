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





    

