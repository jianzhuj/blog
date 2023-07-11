查看某个端口是不是有进程占用
netstat -lnp | grep 8031
netstat -ntlp |grep 9200  




usr/lib/systemd/system/nginx.service
systemctl daemon-reload
systemctl start  nginx.service
systemctl stop  nginx.service
systemctl  reload nginx.servic
systemctl  enable nginx



jps
nohup ./mqnamesrv &

ssh -p22 root@47.102.205.21


ss -lntpd | grep :22

jps -l
ll /proc/32237  查看cwd

select  concat("\t",channo) as channo ,label,px,py from video_device where tree_type ='CHANNEL'  导出



/etc/tinyproxy/tinyproxy.conf
systemctl restart tinyproxy.service 







权限相关

su root

chown  -R 递归文件夹  eshore(用户名)  file(文件或文件夹)
chown  -R   eshore:eshore  dir

chmod -R 754 文件夹

chmox   -R +x  文件夹



cat  *****   | grep  "***"  -A -B -C
-A 是目标行后面多少行
-B 是目标行前面多少行
-C 是上下多少行


上传文件
rz -bey
b 二进制传输
e 控制字符进行转义
y 覆盖同名文件   