# Day70
Nginx和uwsgi部署Flask项目  

  
### 在虚拟环境下使用Nginx和uwsgi部署Flask项目  

#### 环境准备  

1、安装virtualenv虚拟环境  
2、安装Nginx（不需要再虚拟环境下）  
```
yum install nginx
```
3、安装python库：uwsgi和你项目需要的包  


#### 使用uWSGI部署Flask项目  
使⽤ uWSGI 部署 Flask 项⽬只需要换⼀种命令来启动服务即可：  
```
uwsgi --socket 0.0.0.0:5000 --protocol=http -p 3 -w application:app

注：这里的application指的是你启动项目的模块名，app指的是Flask实例对象
```

#### 使⽤ Nginx + uWSGI 部署 Flask 项⽬  

1、添加uwsgi的配置文件uwsgi.ini  

uwsgi.ini配置文件  
```
[uwsgi]
# 允许主线程存在（true）
master = true
# 指定项目的目录
chdir = /home/lqz/flask-news_env/flask-news2 
# 项目启动模块
module = manager
# 真正的 py 脚本文件
wsgi-file = /home/lqz/flask-news_env/flask-news2/manager.py
# python 程序内用于启动的 application 变量名
callable = app
# 处理器数
processes = 4
# 线程数
threads = 2

# protocol=http

# uwsgi 启动时所使用的地址和端口
#socket = 192.168.244.131:5000
http = 192.168.244.131:5000

```

2、后台启动uwsgi（通过配置文件）  

```
uwsgi --ini 配置文件路径 &

（这里是uwsgi.ini的路径，比如：我的是/home/lqz/flask-news_env/uwsgi.ini）
那么我的命令就是：uwsgi --ini /home/lqz/flask-news_env/uwsgi.ini &
```

&emsp；此时uwsgi已经启动，但是还是无法直接使用http，需要经过nginx反向代理。  

3、打开nginx配置文件   /etc/nginx/nginx.conf  

```
http {
     server {
     listen 80;
     location / {
     include uwsgi_params;
     uwsgi_pass IP:5000;
     uwsgi_param UWSGI_CHDIR /home/moocba/flask_project/;  # 自己项目所在的目录
     uwsgi_param UWSGI_SCRIPT application:app;   # 项目模块名：Flask实例对象
      }
   } 
 }
```

4、启动nginx  


此时使用Nginx和uWSGI部署Flask项目的步骤已经结束了，但是在过程中总会有很多坑，在此，我总结了我在部署过程中所遇到的坑  

#### 踩过的坑
1、启动uwsgi的时候总是报错，找不到项目文件中的某个模块，比如flask等等，我这里的原因是因为uwsgi.ini配置文件中的chdir项没有写完整，这里改成启动的manager.py所在的目录位置即可  
2、启动uwsgi的时候会报错invalid request block size，此时把uwsgi.ini配置文件中的“socket = 192.168.244.131:5000”修改为“http = 192.168.244.131:5000”  
3、都正常启动之后，访问http，还是无法访问，查看错误日志，原因是权限不够，被拒绝了。 我这里把nginx的配置文件里的“User Nginx”修改为“User root”，临时关闭了selinux，然后可以正常访问了  


以上是我遇到的些坑和自己的解决方法（不一定适用所有人，可能情况不一样，希望对大家有帮助）
