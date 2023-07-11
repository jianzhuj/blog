如果只想构建某个子模块和它的依赖项 可以使用命令
mvn clean install=true -pl 子模块名称 -am

这里有个坑，maven打包的时候是根据目录接口来查找模块的 而不是根据pom文件中的artifactId



如果你的目录结构是

A

​	b

​	c

​	d

​		e

如果要打包e模块  在a目录下 需要执行  mvn clean install  -pl d\e -am



pl指定模块
am表示同时打包pl指定模块的依赖项

可以在父模块的dependenciseManager中定义子模块的groutId和version
这样子模块a引入子模块b的时候就不用声明版本号







maven的两个小坑



1.   项目工程中pom文件里面定义了repository 但是maven依然会去寻找maven的setting配置文件里面的mirror配置的仓库

   原因：  pom中定义的仓库是优先于setting定义的镜像的 ，除非定义的仓库id和mirror的mirrorOf配置相同，或者mirrorOf配置的是*

   所以最好是  mirrorOf配置central 只拦截central

   pom中的repository id不要命名为central。。。。。。。。。。。

​	2. dependencyManagement 中定义的依赖 如果没有子模块pom去引用这个依赖，dependencyManagement中的依赖会爆红。。。。。。