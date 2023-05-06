用imp命令向oracle中导入数据后，所有查询出的中文字段都为乱码:

1.  原则上不修改服务器端的字符集，修改服务器端的字符集会出现使用第三方工具登陆数据库出现乱码的情况（具体服务器端的字符集修改办法本文有详细介绍）。
2.  将DMP文件的字符集改成与Oracleo数据库服务器端一样之后导入可正常显示。

## 一、什么是oracle字符集

Oracle字符集是一个字节数据的解释的符号集合，有大小之分，有相互的包容关系。ORACLE支持国家语言的体系结构允许你使用本地化语言来存储，处理，检索数据。它使数据库工具，错误消息，排序次序，日期，时间，货币，数字，和日历自动适应本地化语言和平台。

影响oracle数据库字符集最重要的参数是NLS\_LANG参数。它的格式如下:

```bash
NLS_LANG = LANGUAGE_TERRITORY.CHARSET
```

它有三个组成部分(语言、地域和字符集)，每个成分控制了NLS子集的特性。其中:

*   Language 指定服务器消息的语言
*   territory 指定服务器的日期和数字格式
*   charset 指定字符集。如\:AMERICAN\_AMERICA.UTF8  从NLS\_LANG的组成我们可以看出，真正影响数据库字符集的其实是第三部分。所以两个数据库之间的字符集只要第三部分一样就可以相互导入导出数据，前面影响的只是提示信息是中文还是英文

## 二、如何查询Oracle的字符集

很多人都碰到过因为字符集不同而使数据导入失败的情况。这涉及三方面的字符集：

*   oracel server端的字符集。
*   oracle client端的字符集。
*   DMP文件的字符集。

在做数据导入的时候，需要这三个字符集都一致导入后才会不出现乱码。

1.  查询oracle server端的字符集

有很多种方法可以查出oracle server端的字符集，比较直观的查询方法是以下这种:

```sql
SQL> select userenv('language') from dual;        
USERENV('LANGUAGE') 
---------------------------------------    
AMERICAN_AMERICA.UTF8        
SQL>
```

结果类似如下: AMERICAN\_AMERICA.UTF8  

1.  如何查询DMP文件的字符集

用oracle的exp工具导出的DMP文件也包含了字符集信息，DMP文件的第2和第3个字节记录了DMP文件的字符集。如果DMP文件不大，比如只有几M或几十M，可以用UltraEdit打开(16进制方式)，看第2第3个字节的内容，如0354，然后用以下SQL查出它对应的字符集:

```sql
SQL> select nls_charset_name(to_number('0354'，'xxxx')) from dual;
ZHS16GBK
```

如果DMP文件很大，比如有2G以上(这也是最常见的情况)，用文本编辑器打开很慢或者完全打不开，可以用以下命令(在unix主机上):

```bash
cat www.yeserver.com.dmp |od -x|head -1|awk '{print $2 $3}'|cut -c 3-6 
```

然后用上述SQL也可以得到它对应的字符集。

1.  查询oracle client端的字符集

在Linux/UNIX平台下，就是环境变量NLS\_LANG。

```bash
$echo $NLS_LANG
AMERICAN_AMERICA.UTF8
```

在windows平台下，注册表里面的

    HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\HOME0\NLS_LANG

还可以在DOS窗口里面临时设置，比如:

```bash
set nls_lang=AMERICAN_AMERICA.UTF8
```

这样就只影响这个窗口里面的环境变量。 请确保server端与client端字符集相一致。

## 三、修改oracle的字符集

oracle的字符集有互相的包容关系。如US7ASCII就是ZHS16GBK的子集，从US7ASCII到ZHS16GBK不会有数据解释上的问题，不会有数据丢失。在所有的字符集中UTF8应该是最大，因为它基于UNICODE，双字节保存字符(也因此在存储空间上占用更多)。

一旦数据库创建后，数据库的字符集理论上讲是不能改变的。根据Oracle的官方说明，字符集的转换是从子集到超集，反之不行。如果两种字符集之间根本没有子集和超集的关系，那么字符集的转换是不受Oracle支持的。在修改之前一定要确认两种字符集是否存在子集和超集的关系。一般来说，除非万不得已，我们不建议修改oracle数据库server端的字符集。

特别说明，我们最常用的两种字符集ZHS16CGB231280和ZHS16GBK之间不存在子集和超集关系，因此理论上讲这两种字符集之间的相互转换不受支持。

关于字符集的对应关系可以查看[Oracle官方说明](http://download.oracle.com/docs/cd/B19306_01/server.102/b14225/applocaledata.htm)

1.  修改server端字符集(不建议使用)

在oracle 8之前，可以用直接修改数据字典表props来改变数据库的字符集。但oracle8之后，至少有三张系统表记录了数据库字符集的信息，只改props表并不完全，可能引起严重的后果。正确的修改方法如下:

```bash
$sqlplus “/as sysdba”
```

先执行SHUTDOWN IMMEDIATE命令关闭数据库服务器，然后执行以下命令:

```sql
SQL>STARTUP MOUNT;
SQL>ALTER SYSTEM ENABLE RESTRICTED SESSION;
SQL>ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
SQL>ALTER SYSTEM SET AQ_TM_PROCESSES=0;
SQL>ALTER DATABASE OPEN;
SQL>ALTER DATABASE CHARACTER SET INTERNAL_USE UTF8; //跳过超子集检测 
SQL>ALTER DATABASE national CHARACTER SET INTERNAL UTF8;
```

这一行不起作用，执行后出错

    ORA-00933: SQL 命令未正确结束。

不过执行上一行命令已经生效。

1.  修改DMP文件字符集

DMP文件的第2第3字节记录了字符集信息，因此直接修改DMP文件的第2第3字节的内容就可以骗过oracle的检查。理论上也仅是从子集到超集可以修改，但很多情况下在没有子集和超集关系的情况下也可以修改，我们常用的一些字符集，如US7ASCII，WE8ISO8859P1，ZHS16CGB231280，ZHS16GBK基本都可以改。因为改的只是DMP文件，所以影响不大。

具体的修改方法比较多，最简单的就是直接用UltraEdit修改DMP文件的第2和第3个字节。比如想将DMP文件的字符集改为UTF8，可以用以下SQL查出该种字符集对应的16进制代码:

```sql
SQL> select to_char(nls_charset_id('UTF8')， 'xxxx') from dual;
TO_CH
367
SQL>
```

然后将DMP文件的2、3字节修改为0367即可。
如果DMP文件很大，用UltraEdit无法打开，就需要用其它方法的方法了。
