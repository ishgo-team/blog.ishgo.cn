title: addslashes, mysql_real_escape_string并不能防止SQL注入!!!
date: 2013-09-08 07:56:29
author: Barbery  
categories: [PHP]
tags: [ADDSLASHES, MYSQL_REAL_ESCAPE_STRING, PDO, PHP, SQL注入]
---

先看代码...

    <?php
        header('Content-Type: text/html; charset=GBK');
        $input = chr(0xbf) . chr(0x27) . ' OR username = username; /*';
        $value = addslashes($input);  
        $sql = "SELECT * FROM users WHERE username='{$value}' AND password='123123';";

        echo $value;
        echo $sql;
    ?>

以上的demo, 输出结果为:
![http://ww3.sinaimg.cn/large/6915c7dcgw1e8d0o3a0l1j20lu02uq3c.jpg](http://ww3.sinaimg.cn/large/6915c7dcgw1e8d0o3a0l1j20lu02uq3c.jpg)

看出问题了吗???

<!-- more -->
ok, 再来一个, 如果执行上图的sql,结果为: 

![http://t3.qpic.cn/mblogpic/1ec6802af45da8f8cdfa/460](http://t3.qpic.cn/mblogpic/1ec6802af45da8f8cdfa/460) 

看到木有… 瞬间就可以被爆表~~
 
你也许会疑问, 为什么addslashes没有生效~~ 对于一个单引号 ' 来说, addslashes 是有效果的, 但是这里不是一个实体的单引号字符.. 我们再看回去$value的赋值

    $input = chr(0xbf) . chr(0x27) . ' OR username = username; /*';

 
chr(0xbf) 和 chr(0x27)相连接, 构成一个双字节字符(0xbf27)… 而0xbf27只是一个人为合成的双字节, 不在GBK编码表里, 也就是说不是一个合法的GBK双字节字符… 系统会把这个双字节拆分为2个单字节解析(也就是0xbf和0x27), 而addslashes并不知道它是非法的双字节字符, 单引号的GBK编码就是chr(0x27), 单独使用使addslashes是可以识别的...
我使用mysql_real_escape_string测试结果也一样, 无法识别这个非法的GBK双字节字符, 虽然无法识别, 但是使用mysql_query sql却没有绕过mysql的内部系统, 执行结果是空行数...上网搜了一下, 网友说在mysql 5.0的某个版本被修复鸟~~ 但是mysql的更新链接已经失效, 所以也无法追寻是哪个版本后修复了这个问题… 但是手动复制sql进去phpmyadmin下执行 是可以执行的~~ 知道的朋友, 可以交流下!!
 
附上测试脚本: [https://github.com/Barbery/blog/blob/master/sql_inject_test.php](https://github.com/Barbery/blog/blob/master/sql_inject_test.php)
关于上面说的, 大家有什么不懂或需要详细了解, 推荐阅读:
[http://shiflett.org/blog/2006/jan/addslashes-versus-mysql-real-escape-string](http://shiflett.org/blog/2006/jan/addslashes-versus-mysql-real-escape-string)
[http://ilia.ws/archives/103-mysql_real_escape_string-versus-Prepared-Statements.html](http://ilia.ws/archives/103-mysql_real_escape_string-versus-Prepared-Statements.html)


###解决办法:

使用PDO进行SQL操作..

> Prepared statements are 2 round-trips to the database for a single query.

> As soon as you prepare a query, it's sent to the database with the placeholders you set. So the database engine takes that prepared statement and maps out the query and optimizes it for execution. Then when you call execute() only the values you give are sent to the database, with a reference to that query you just prepared. The database engine drops in the values and runs the query. This is totally immune to SQL injection, because the database engine already knows exactly where the values begin and end (the placeholder marker(s) you set), and therefore never need escaping.

> The reason SQL injection exists in the first place is because the entire query is interpreted upon execution, values and all. So if anything interferes with the quotes surrounding your values, the engine thinks that value has ended earlier than it really should, and thus a security hole is introduced. That problem is avoided entirely with prepared statements by letting the database engine know ahead of time exactly where to put each value you pass to it later on. There is no need for escaping and there is no need to worry.
 
 
简单点说就是, 通过PDO的prepared来预处理, PDO在执行1条SQL语句时会发送2条指令, 1是要执行的SQL指令, 但是没有进行赋值, MYSQL对将要处理的SQL进行优化, 然后PDO再发送1条指令, 这条指令就是上条指令需要用到的变量, MYSQL收到后进行赋值然后执行, 这样就可以消除SQL注入...

再简单点说就是, 例如SQL为:

    SELECT * FROM user WHERE username='barbery';

只发送1条指令的情况下, mysql要从这条SQL中提取出barbery, 所以就会存在没转义, 等的注入攻击..

而如果使用PDO的话, 第一条指令会是这样发送

    SELECT * FROM user WHERE username=?;

然后第二条指令再把变量值 barbery发送过去...MYSQL接受到第一条指令时就使用编译器进行优化(使用什么索引等等), 就把指令的执行内容和范围给确定了, 后面传过来的值是什么, 都不会影响SQL的执行内容...

这里建议阅读 [http://zhangxugg-163-com.iteye.com/blog/1835721](http://zhangxugg-163-com.iteye.com/blog/1835721) 获得更直观的了解~~