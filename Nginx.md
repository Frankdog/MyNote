# nginx 
    sudo apt-get instal nginx
 
## 配置文件
路径
    /etc/nginx/nginx.conf
配置文件包含
     /etc/nginx/site-enabled/default

配置文件结构
 
> directivity
 1. simple directivity (keyword value;)
 2. block keyword { }
 3. contex block with directivity inside

 eg:
```
 http {
     server{
         location / {
             root /home/frankdog/workspace;
         }
     }

 }
```

# 用途
* 网页服务器
* 反向代理服务器(FastCGI)  
>客户端访问nginx地址，比如/hostname/index.php,
nginx 并不支持php，而是将php文件发给某php服务器处理，获取处理结果返回给客户端
此处 nginx 需要在配置文件组中指定 php服务器的地址及监听端口，当用户访问的url中匹配
location 事将报文转发给对应的服务器

CDN

#动态DNS
##解决什么问题
用户公网IP不固定，如何通过固定的URL进行访问

##原理
1.用户通过用户名密码在动态DNS运营商处注册账号
2.动态DNS服务商提供给用户一个固定的URL
3.用户在路由器上动态DNS处使用注册的用户名密码登陆
4.路由器向服务商主动上报自己的公网IP
5.这样用户通过服务商提供的固定URL访问时，就可以动态获取到其公网地址
6.用户可以进一步在路由器上配置端口转发，将某些报文转发给私网中的某台固定设备
    
