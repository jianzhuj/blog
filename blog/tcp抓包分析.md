tcpdump dst host 192.168.1.0 -w package.cap

抓取当前机器上目标地址为192.168.1.0的所有数据包

tcpdump host 192.168.1.0 -s 0 -vv -w /tmp/dump.log

可以抓取和192.168.1.0 交互的所有包

然后抓包的文件可以放入wireshark分析