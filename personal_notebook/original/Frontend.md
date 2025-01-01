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

