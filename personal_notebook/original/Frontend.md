# VUE 部署到 IIS, 刷新页面报404

## 安装

下载url重写工具：https://www.iis.net/downloads/microsoft/url-rewrite

## 根目录新建web.config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
 <system.webServer>
 <staticContent>
  <remove fileExtension=".woff" />
  <mimeMap fileExtension=".woff" mimeType="font/x-woff" />
  <remove fileExtension=".woff2" />
  <mimeMap fileExtension=".woff2" mimeType="font/x-woff2" />
  <remove fileExtension=".ttf" />
  <mimeMap fileExtension=".ttf" mimeType="font/x-ttf" />
  <remove fileExtension=".json" />
  <mimeMap fileExtension=".json" mimeType="text/json" />
 </staticContent>
 <rewrite>
  <rules>
  <rule name="vue" stopProcessing="true">
   <match url=".*" />
   <conditions logicalGrouping="MatchAll">
   <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
   <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
   </conditions>
   <action type="Rewrite" url="/" />
  </rule>
  </rules>
 </rewrite>
 </system.webServer>
</configuration>
```







# [Vue项目上线后刷新报错404问题（apache，nginx，tomcat）](https://www.cnblogs.com/LindaBlog/p/14485033.html)

## apache

1.修改httpd.conf文件，开启rewrite_module功能。

​    `LoadModule rewrite_module libexec/apache2/mod_rewrite.so`，去掉前面的#

2.找到`AllowOverride None`的那行，把它改成`AllowOverride All`，来使`.htaccess`文件生效。

3.在你的项目目录下，也就是我的根目录添加一个.htaccess文件：

```xml
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

4.重启apache httpd





















