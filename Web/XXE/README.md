# WebGoat 搭建  
1. 
安装docker,使用以下命令  
```bash
docker run -it -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 webgoat/webgoat
```  


2. 
https://github.com/WebGoat/WebGoat/releases/download/v2025.3/webgoat-2025.3.jar  


3. 使用源码搭建  

- 下载源码  
```
git clone https://github.com/WebGoat/WebGoat.git
``` 
进入，在当前目录打开pom.xml文件，找到java.version标签，把java版本修改为自己的java版本。 使用管理员权限，执行  
```bash
./mvnw.cmd clean install
```  
~/.m2/repository/org/eclipse/jgit/org.eclipse.jgit
cd ~/.m2/repository/org/apache/pdfbox/pdfbox/2.0.24  

# php和httpd  
httpd下载连接  
```
https://www.apachelounge.com/download/ 
```  
解压缩之后有一个叫Apache24的文件夹，如果移动到其他地方，需要自己修改配置，如果直接放在C盘，就不需要再修改配置。  

再Apache24/bin目录运行ApacheMoniter.exe  
![alt text](image.png)  
![alt text](image-1.png)  
如果没有打开，左键单击图标可以开启  
![alt text](image-2.png)

启动成功后访问localhost，页面应该和Apache24/htdocs/index.html一样  
![alt text](image-3.png)  

php下载连接 
```
https://www.php.net/downloads.php  
```
选择一个版本，点击Windows downloads，
![alt text](image-4.png)  

有很多，比如这里有两个，上面是Non Thread Safe,下面是Thread Safe,下载Thread Safe的，解压之后放好  

打开Apache24/conf/httpd.conf文件，在合适的地方添加下面内容，文件内有很多标签，不要加在标签内部就可以了。文件路径该为自己php的路径。  
```
LoadModule php_module 'C:/server/php8.4.5/php8apache2_4.dll'
PHPIniDir 'C:/server/php8.4.5'
AddType application/x-httpd-php .php
```

在Apache24/htdocs/下添加一个hello.php文件，填写
```php
<?php

echo "hello";
```  
然后重启httpd
![alt text](image-2.png)  
访问localhost/hello.php  
下面是成功的情况  
![alt text](image-5.png)  
如果没有配置好，可能是  
![alt text](image-6.png)  

