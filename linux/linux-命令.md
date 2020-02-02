##### > 和 >> 的区别

大于号：将一条命令执行结果（标准输出，或者错误输出，本来要打印到屏幕上面的）重定向其它输出设备（文件，打开文件操作符，或打印机等等）

小于号：命令默认从键盘获得的输入，改成从文件，或者其它打开文件以及设备输入

**>>** 是追加内容

**>** 是覆盖原有内容

示例：

~~~javascript
[root@izwz9emoxkbmipgs7k6gcwz ~]# echo "abc" > test.txt
[root@izwz9emoxkbmipgs7k6gcwz ~]# more test.txt 
abc
[root@izwz9emoxkbmipgs7k6gcwz ~]# echo "123" >> test.txt
[root@izwz9emoxkbmipgs7k6gcwz ~]# more test.txt
abc
123
[root@izwz9emoxkbmipgs7k6gcwz ~]# echo "123" > test.txt
[root@izwz9emoxkbmipgs7k6gcwz ~]# more test.txt
123
~~~

**<** 小于号

导入数据示例：

~~~javascript
mysql -u root -p -h test < test.sql
~~~

