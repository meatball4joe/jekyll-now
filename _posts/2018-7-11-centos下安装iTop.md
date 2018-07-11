---
layout: post
title: centos下安装iTop 
---

下载了iTop，按照官方文档说明安装过程如下：


#### 配置nginx
添加如下location配置

```
location / {
        try_files $uri $uri/ =404;
}

location ~ ^(.+.\.php)(/|$) {
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_split_path_info ^(.+\.php)(/.*)$;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

#### 安装php 5
...


#### 安装iTop
按照说明，解压代码包， 把web目录copy到nginx/html目录下，
访问urlhttp://x.x.x.x/web/index.php，运行安装向导。


按照提示填写相应配置，最后提交时报错，提示无法找到conf/production/config-itop.php文件，源码包中
根本没有这个目录，网上搜索无果，解决办法基本不可用。

查看网页后发现，其引用的js程序路径不对, 都是类似下面的样子

```
<script type="text/javascript" src="http://_/web/js/jquery-migrate-1.4.1.min.js?t=2.5.0-beta"></script>
<script type="text/javascript" src="http://_/web/js/jquery-ui-1.11.4.custom.min.js?t=2.5.0-beta"></script>
```


解决办法如下：
修改文件application/utils.inc.php， 找到GetAbsoluteUrlAppRoot函数， 修改其中内容，指定url为你自己的机器ip和路径

```
/*$sUrl = self::GetConfig()->Get('app_root_url');*/
$sUrl = 'http://x.x.x.x/web/';
```

然后按照指引一步步来就好， 注意Application URL需要填写成你需要的url。






