**1.Maven依赖中的scope详解**

​		Maven默认的依赖配置项中，scope的默认值是compile。

​		scope的分类：

compile

​		默认就是compile，什么都不配置也就是意味着compile。compile表示依赖项目需要参与当前项目的编译，当然后续的的测试，运行期间也参与其中，是一个比较强的依赖。打包的时候也需要包含进去。

test

​		scope为test表示依赖项目仅仅需要参与测试相关的工作，包括测试代码的编译，执行。比较典型的如junit。

runntime

​		runntime表示依赖项目无需参与项目的编译，不过后期的测试和运行期间都需要参与其中。与compile相比，跳过编译而已。

provided

​		provided意味着打包的时候可以不用打包进去，别的容器会提供，比如Web Container。事实上该依赖参与编译，测试，运行。只是打包的时候没有包含进去。

system

​		从参与度来说，也provided相同，不过被依赖项不会从maven仓库抓，而是从本地文件系统拿，一定需要配合systemPath属性使用。



1、创建web项目
mvn archetype:generate -DgroupId=org.alex -DartifactId=testcase_jQueryMobile -DarchetypeArtifactId=maven-archetype-webapp -e

上面会卡死在

解决
mvn archetype:generate -DgroupId=com.net.train.upload -DartifactId=trainUpload -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false -DarchetypeCatalog=local

2、创建java项目
mvn archetype:generate -DgroupId=org.mq -DartifactId=mqapp

mvn archetype:generate -DgroupId=org.alex.springboot -DartifactId=chapter1 -DinteractiveMode=false -DarchetypeCatalog=local


3、将本地jar加到类库
mvn install:install-file -Dfile=e:\3des-1.0.jar -DgroupId=com.asiainfo.uap.util.des -DartifactId=3des -Dversion=1.0 -Dpackaging=jar -DgeneratePom=true