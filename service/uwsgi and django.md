### python-env


1, mysql 
```
grant all on demo.* to demo@'127.0.0.1' identified by 'wd1023';
flush privileges;
create database demo character set = utf8;
```
2,nginx 和初始坏境脚本
```
# onestack的脚本
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ~/oneinstack/install.sh --nginx_option 1     

# python编译安装和虚拟环境												
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz && tar xvf Python-3.6.5.tar.xz && cd Python-3.6.5/		
./configure && make && make install				
mkdir -p /data/ && cd /data							
python3.6 -m venv py36							
source /data/py36/bin/activate

# 项目安装
unzip demo.zip 
mv demo wwwroot/project
cd /data/wwwroot/project/
apt install libmysqlclient-dev -y
pip install -r requirements.txt
```


3, Django 项目配置
```
vi demo/settings.py								
ALLOWED_HOSTS = ['*']								#允许任意主机访问
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'demo',
        'USER':'demo',
        'PASSWORD':'wd1023',
        'HOST':'192.168.1.200'
        'PORT':'3306',
        'OPTIONS': MYSQL_DATABASE_OPTIONS,
    }
}															
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')		#设置静态目录

python manage.py collectstatic      				#采集静态文件到指定的STATIC_ROOT

python manage.py makemigrations
python manage.py migrate							#数据库对象生成
python manage.py createsuperuser  					#超级管理员
python manage.py runserver 0.0.0.0:8000				#测试运行
```


4, uwsgi相关配置文件和目录
```
mkdir /data/uwsgi-script -p							#创建uwsgi-script存放log等文件(原来是想存放自动脚本)
chown www.www -R /data/wwwroot/project				#修改项目文件属主和uwsgi中一致
mkdir /etc/uwsgi/sites/ -p							#建立uwsgi统一的站点配置目录(类似于nginx的conf/vhost目录)
cat /etc/uwsgi/sites/demo.ini						#生成项目的uwsgi配置demo.ini
[uwsgi]
#socket = :8000									
uid = www
socket = /run/uwsgi/demo.sock						#用unix-socket方式
chmod-socket = 660									#socket权限
chown-socket = www:www								#socket属主
	
#http = 192.168.1.210:8000
static-map=/static=/data/wwwroot/project/static/	#静态文件路径和setting里的STATIC_ROOT一致
chdir = /data/wwwroot/project/						#项目主目录
module = demo.wsgi:application						#项目的app中的wsgi路径(因为uwsgi其实是封装wsgi)
home = /data/py36									#虚拟环境主目录
pidfile = /data/uwsgi-script/demo/uwsgi.pid			#pid文件
daemonize = /data/uwsgi-script/demo/uwsgi.log		#日志文件
master = true										
processes = 3										#进程和线程数
threads = 2
vacuum = true										#这项必须开启,才能使用emperor模式
max-requests = 2000
```

> PS: 不创建会报错   
mkdir /data/uwsgi-script/demo/   
touch /data/uwsgi-script/demo/uwsgi.log   



5, nginx相关配置
```
mkdir /usr/local/nginx/conf/vhost
vi /usr/local/nginx/conf/vhost/demo.conf
server {
    listen 80;
    server_name _;
    access_log      /data/wwwlogs/demo_nginx_access.log;
    client_max_body_size 75M;
    charset utf-8;
    gzip_types text/plain application/x-javascript text/css text/javascript application/x-httpd-php application/json text/json image/jpeg image/gif image/png application/octet-stream;
    location /static {									#nginx跳过uwsgi直接处理静态请求
        alias /data/wwwroot/project/static/;
    }

    location / {
        include uwsgi_params;
#        uwsgi_pass 127.0.0.1:8000;
        uwsgi_pass unix:/run/uwsgi/demo.sock;			#nginx代理uwsgi的unix-socket
    }
}

vi /usr/local/nginx/conf/nginx.conf   #备注主配置中的default server 
:61,92 s/^/#/g

nginx -t
nginx -s reload
```


6, systemcd脚本
```
cat /lib/systemd/system/demo.service
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown www.www /run/uwsgi'
ExecStart=/data/py36/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target

systemctl enable demo
```



7,uwsgi 测试命令

启动：uwsgi --ini your_path/demo.ini   
停止：uwsgi --stop your_path/uwsgi.pid   
重启：uwsgi --reload your_path/uwsgi.pid   
