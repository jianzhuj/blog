redis安装

```
1.安装redis数据库

yum install redis

2.下载fedora的epel仓库

yum install epel-release

3.启动redis服务

systemctl start redis

4.查看redis状态

systemctl status redis

systemctl stop redis 停止服务

systemctl restart redis 重启服务

5.查看redis进程

ps -ef | grep redis

6.设置开机自启动

systemctl enable redis

7.开放端口号

firewall-cmd --zone=public --add-port=80/tcp --permanent

firewall-cmd --zone=public --add-port=6379/tcp --permanent

8.查看端口

netstat -lnp|grep 6379

9.设置redis 远程连接和密码

输入命令vi /etc/redis.conf进入编辑模式

将这部注释掉，否则只有本机才能访问

\#bind 127.0.0.1

protected-mode no

port 6379

requirepass 密码

10.重启redis

systemctl restart redis

11.进入redis

命令redis-cli -h 127.0.0.1 -p 6379

然后输入info,提示必须验证

auth 密码
```

