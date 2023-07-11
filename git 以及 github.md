##### git的配置级别 level
1. local 本地配置

2. golbal 全局配置(）

3. system 系统配置(登录当前系统的所有用户)

   ```
   git config --global user.name 'ekko'
   ```

##### git仓库的划分

	1. 工作区  就是.git文件夹所在目录下的区域
	1. 暂存区  工作区的文件通过 git add 添加到暂存区
	1. 本地仓库  暂存区的文件通过commit提交到本地仓库
	1. 远程仓库  本地仓库通过push 提交到远程仓库	

##### git命令

1. git status

2. git add   文件名/all   (将文件加入到暂存区)

3. git commit -m (暂存区提交到本地仓库)

4. git diff 文件名（比较本地仓库和和暂存区文件的差异）

​			git diff 文件名  比较的是工作区和暂存区的区别

​			git diff -cached 文件名  或者git diff --staged  文件名  比较的是暂存区和commit仓库的区别

​			git diff HEAD  比较的是工作区和本地仓库的区别

5. git log

6. git reomte  add gitee 具体的仓库地址

   可以同时把本地仓库推送到两个远程仓库上

   git push gitee  main   (gitee 是远程仓库 main是本地分支的名称)

### github 使用 

1. ​	连接方式

   1. 连接方式分为https和ssh两种,https是每次pull或者push时验证账号密码。
   2. ssh是在github上验证自己本地的私钥

2. ssh方式的使用

   1. 本地生成ssh私钥

      ```
      ssh-keygen -t rsa -C ‘自己的邮箱’
      -t 指定密钥类型，默认是 rsa ，可以省略。
      -C 设置注释文字，比如邮箱。
      -f 指定密钥文件存储文件名。
      ```

      要是不知道自己git配置的邮箱,可以通过命令查看

      ```
       git config user.name
       git config user.email
      ```

      在用户目录下的.ssh文件夹中会生成id_ras(私有密钥)和id_ras.pub(共有蜜)

   2. 现在本地

​		