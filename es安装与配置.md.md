1. 下载

   ```
    curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-linux-x86_64.tar.gz
   ```

2. 解压

   ```
   tar -zxvf elasticsearch-7.12.0-linux-x86_64.tar.gz
   ```

3. 创建非root用户

   ```
   useradd es
   passwd es
   ```

4. 修改linux配置来启动es

   1. 修改允许用户启动的线程数和创建文件数

      ```
      vi /etc/security/limits.conf
      添加以下配置
      es soft nofile 65536
      es hard nofile 65536
      es soft nproc 4096
      es hard nproc 4096
      
      ```

   2. 可以修改es目录下config/elasticsearch.yml中的日志和数据存放位置

      ```
      path.data:
      path.logs
      ```

   3. 修改jvm的堆内存大小 config/jvm.options  默认4g，如果测试机器较小，可以改成1g

      ```
      -Xms4g
      -Xmx4g
      ```

5. 设置es外网访问  config/elasticsearch.yml中的network.host   es只能本机访问

   ```
   改成 0.0.0.0 允许外网访问
   network.host: 0.0.0.0
   ```

6. 设置es密码

   1. 修改onfig/elasticsearch.yml

      ```
      xpack.security.enabled: true
      xpack.license.self_generated.type: basic
      xpack.security.transport.ssl.enabled: true
      ```

   2. 启动：脚本设置密码

      ```
      bin/elasticsearch-setup-passwords interactive
      ```

      

7. 启动es  启动脚本在es目录下的bin文件夹下

   ```
   	./elasticsearch  -d
   ```

8. 安装启动kibana

   1. 安装和解压

      ```
      wget https://artifacts.elastic.co/downloads/kibana/kibana-8.8.2-linux-x86_64.tar.gz 
      tar -zxvf kibana-8.8.2-linux-x86_64.tar.gz 
      ```

   	2. 修改kibana配置,指定连接es的账号密码以及允许外网访问

       ```
       // 修改kibana.yml   password就是在6.2中允许脚本设置的kibana的密码。
       server.host: "0.0.0.0"
       elasticsearch.username: "kibana_system"
       elasticsearch.password: "*****"
       
       
       ```

   	3. ​	运行kibana

       ```
       nohuo ./kibana &
       ```

       

   