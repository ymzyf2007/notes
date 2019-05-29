##### 一、前言

​		最近开发微信支付的时候，因ios的用户昵称是可以带表情符号的，当时记录支付记录的时候自然遇到了emoji表情符号如何被MySql DB支持的问题。当时这个困扰了一下，后面改了服务端编码解决了问题。在此整理和记录一下。

##### 二、问题描述

​		如果UTF8字符集且是Java服务器的话，当存储含有emoji表情时，会抛出如下异常：

~~~mysql
SQL Error：1366
java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row 1 
    at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1073) 
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3593) 
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3525) 
    at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1986) 
    at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2140) 
    at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2620) 
    at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1662) 
    at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1581)
~~~

​		这是字符集不支持的异常。因为UTF-8编码可能是两个、三个、四个字节（常用中文字符用utf-8编码占用3个字节，GBK、GB2312收编的汉字占2个字节），其中emoji表情是4个字节，而MySql的utf8编码最多3个字节，所以导致了数据插不进去。

##### 三、解决问题

​		如果你的项目需要存储emoji表情等，将你的DB字符集从UTF8/GBK等传统字符集升级到utf8mb4将是势在必行。你可以通过应用层面转换emoji等特殊字符，以达到原DB的兼容，我认为可行，但是你可能走了弯路。utf8mb4作为utf8的super set，完全向下兼容，所以不用担心字符的兼容性问题。切换中需要顾虑的主要影响是mysql需要重新启动（启动才可生效）。

##### 四、升级步骤

1）、utf8mb4的最低mysql支持版本为5.5.3+，若不是，请升级到较新版本。

​		mysql版本查看命令：终端[mysql -V]；mysql中[status];

2）、修改database、table和column字符集。参考以下语句：

~~~mysql
ALTER DATABASE byb CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE byb.donation_record CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
~~~

3）、修改mysql配置文件my.cnf(windows为my.ini)

​		my.cnf一般在/etc/my.cnf位置。找到以后添加如下内容

~~~mysql
#port=28088 【设置端口，mysql默认端口是3306】
#character_set_server=utf8 【之前设置的utf8编码】
#collation_server=utf8_general_ci【之前设置的utf8编码】
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
#lower_case_table_names=1【linux默认是识别大小写的，设置该项忽略大小写】
~~~

##### 五、重启MySql，检查字符集

​		停止：service mysql stop

​		启动：service mysql start

​		在mysql命令行中输入：

~~~mysql
SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';
~~~

