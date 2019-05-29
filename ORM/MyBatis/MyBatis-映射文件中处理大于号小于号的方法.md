​		当我们需要通过xml格式处理sql语句时，经常会用到<，<=，>，>=等符号，但是很容易引起xml格式的错误，这样会导致后台将xml字符串转换为xml文档时报错，从而导致程序错误。

​		有两种方法可以解决这种问题。

1）、用转义字符把>和<替换掉，然后就没有问题了。

~~~mysql
select * from test where 1=1 and start_date &gt;= CURRENT_DATE end &lt;=CURRENT_DATE
~~~

附：XML转义字符

| 原符号   | <    | <=    | >    | >=    | &     | '      | "      |
| -------- | ---- | ----- | ---- | ----- | ----- | ------ | ------ |
| 替换符号 | &lt; | &lt;= | &gt; | &gt;= | &amp; | &apos; | &quot; |

2）、可以使用<![CDATA[]]>符号进行说明【该区段中的文本会被解析器忽略】

~~~mysql
<![CDATA[
select * from test where 1=1 and start_date >= CURRENT_DATE end <=CURRENT_DATE
]]>
~~~

