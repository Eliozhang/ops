# ops
简单的运维管理后台，python2.7-tornado3

#依赖
基于python2.7    
基于tornado    
基于bootstrap的AdminX界面模板    
使用了paramiko，需要安装    
使用了celery，需要安装    
使用了redis，需要安装    
使用了mysql，需要安装，需要建库建表

# 功能说明
主要实现了
* 命令下发执行
* 文件下发

# 申明
本代码仅为运维开发测试代码，功能并不强大，算是一个demo，仅作为参考，不作为一个完善的项目产品。

#安装
###安装python2.7
```
yum install -y zlib-devel
wget --no-check-certificate https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
tar xf Python-2.7.9.tar.xz
cd Python-2.7.9
./configure --prefix=/usr/local --with-zlib
make && make altinstall
```
###安装pip
```
wget https://pypi.python.org/packages/source/s/setuptools/setuptools-18.7.1.tar.gz#md5=a0984da9cd8d7b582e1fd7de67dfdbcc  --no-check-certificate
tar xvf setuptools-18.7.1.tar.gz
cd setuptools-18.7.1
python2.7 setup.py install

rm -rf pip-7.1.0.tar.gz
wget http://demo.acache.cn/pip-7.1.0.tar.gz
tar xvf pip-7.1.0.tar.gz
cd pip-7.1.0
python2.7 setup.py install
```

###安装Paramiko模块
```
yum -y install python-devel
wget http://ftp.dlitz.net/pub/dlitz/crypto/pycrypto/pycrypto-2.6.tar.gz
tar -zxvf pycrypto-2.6.tar.gz
cd pycrypto-2.6/
python2.7 setup.py build && python2.7 setup.py install
cd ..
wget http://www.lag.net/paramiko/download/paramiko-1.7.7.1.tar.gz
tar xvzf paramiko-1.7.7.1.tar.gz
cd paramiko-1.7.7.1/
python2.7 setup.py build && python2.7 setup.py install
```

###安装redis
```
wget http://download.redis.io/releases/redis-3.0.6.tar.gz
tar zxf redis-3.0.6.tar.gz
cd redis-3.0.6
make
make PREFIX=/usr/local/redis install
mkdir -p /usr/local/redis/bin
mkdir -p /usr/local/redis/etc
mkdir -p /usr/local/redis/var
mkdir -p /data/redis-data/redis6001
cp redis.conf /usr/local/redis/etc/redis.conf.default
cd src
cd /usr/local/redis/etc/
cat >> /usr/local/redis/etc/redis6001.conf <<'EOF'
daemonize yes
pidfile /var/run/redis6001.pid
port 6001
loglevel notice
logfile /data/logs/redis6001.log
databases 16
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename redis6001.rdb
dir /data/redis-data/redis6001
slave-serve-stale-data yes
slave-read-only yes
slave-priority 100
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
requirepass kdredis1
EOF
echo "ok"
```

####启动redis
```
mkdir -p /data/logs/
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis6001.conf
```

###安装celery
```
pip2.7 install celery
pip2.7 install torndb
pip2.7 install mysql
pip2.7 install redis
```

###mysql建库建表
```
create database ops default character set utf8;
grant all privileges on ops.* to ops@'localhost' identified by'ops';
use ops

create table task(
   task_id INT NOT NULL AUTO_INCREMENT,
   shell_name VARCHAR(256) NOT NULL,
   do_user VARCHAR(20) NOT NULL,
   dst_hosts VARCHAR(2048) NOT NULL,
   shell_from VARCHAR(20) NOT NULL,
   shell_content VARCHAR(5096) NOT NULL,
   PRIMARY KEY ( task_id )
);

create table task_res(
   id INT NOT NULL AUTO_INCREMENT,
   keyname VARCHAR(512) NOT NULL,
   hostdev VARCHAR(2048) NOT NULL,
   status int(1) NOT NULL,
   res_content MEDIUMBLOB NOT NULL,
   PRIMARY KEY ( id )
);

create table host_devs(
   id INT NOT NULL AUTO_INCREMENT,
   ip VARCHAR(256) NOT NULL,
   dev_desc VARCHAR(256) NOT NULL DEFAULT 'default',
   status int(4) NOT NULL DEFAULT '2',
   dev_grp VARCHAR(256) NOT NULL DEFAULT 'default',
   PRIMARY KEY ( id )
);

```

###启动celery
在上面的准备都就绪后
在src目录下，跟tasks.py文件同级执行
```
celery -A tasks worker -c 10 --loglevel=info
```
跑到后台模式
```
nohup celery -A tasks worker -c 10 --loglevel=info &
```

##启动ops.py
python2.7 ops.py

会监听8000端口，可以自行修改。

#界面
![image1](https://github.com/cnkedao/ops/raw/master/1/1.jpg)
![image2](https://github.com/cnkedao/ops/raw/master/1/2.jpg)
![image3](https://github.com/cnkedao/ops/raw/master/1/3.jpg)
![image4](https://github.com/cnkedao/ops/raw/master/1/4.jpg)