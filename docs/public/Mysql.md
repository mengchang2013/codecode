# Mysql

## 查询某字段是否包含值
```sql
select * from 表 where locate('value',字段);
```

## 获取当前时间
```sql
 select now(),sysdate(),sleep(3),now(),sysdate();
```
NOW()、SYSDATE()  
区别：
NOW()取的是语句开始执行的时间，
SYSDATE() 取的是语句执行过程中动态的实时时间。

## 关联更新、关联删除
```sql
#关联更新语法
UPDATE A a INNER JOIN B  b ON a.id= b.uid SET a.name = b.name WHERE a.name != ''
#关联删除语法
delete a from A a inner join B b on a.PId=b.FId where 条件
#删除重复数据
 ##1、自联接
delete p1 from t_user p1, t_user p2 where p1.name = p2.name and p1.id > p2.id;
```

## sql四大排名函数
```sql
row_number()：依次递增排名，无重复排名
rank()：相同分数有重复排名，但是重复后下一个人按照实际排名
dense_rank()：分数一致排名一致，分数不一致排名+1
NTILE(4)：分组排名，里面的数字是几，最多排名就是几，里面的数字是4，最多的排名就是4
```

## 时间比较
```sql
DATEDIFF()
含义：函数返回两个日期之间的天数。
语法：（date1 - date2）
DATEDIFF(date1,date2)
```

## Mysql更新字段为另一张表的字段值
```sql
update a inner join b on a.bid=b.id set a.x=b.x,a.y=b.y;
```

## utf8mb4_unicode_ci和utf8mb4_general_ci的对比
①准确性  
utf8mb4_unicode_ci是基于标准的Unicode来排序和比较，能够在各种语言之间精确排序;  
utf8mb4_general_ci没有实现Unicode排序规则，在遇到某些特殊语言或者字符集，排序结果可能不一致。  
②性能  
utf8mb4_general_ci在比较和排序的时候更快  
utf8mb4_unicode_ci在特殊情况下，Unicode排序规则为了能够处理特殊字符的情况，实现了略微复杂的排序算法。

##  utf8和utfmb4区别
1、utf8mb4是utf8的超集,Mysql在5.5.3版本增加编码字符集utf8mb4;  
2、utf8只支持最长3个字节，utf8mb4支持最长4个字节,任何不在基本多文本平面的 Unicode字符，都无法使用 Mysql 的 utf8 字符集存储。包括 Emoji 表情和很多不常用的汉字，以及任何新增的 Unicode 字符等;
## mysql查询语句中的 limit 与 offset 的区别
limit n  等价于limit 0,n ;表示: 查询结果返回前n条数据  
limit m, n 分句表示: 跳过 m 条数据，读取 n 条数据  
&emsp; &emsp; &emsp; 第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目，  
&emsp; &emsp; &emsp; 为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1  
limit n offset m 分句表示: 跳过 m 条数据，读取 n 条数据  


## window安装mysql

1. 下载zip格式的[mysql](https://dev.mysql.com/downloads/mysql/)文件，将其解压到某一盘符下(如：*D:\java\MySQL* 目录下)，根目录创建data文件夹；

2. 配置环境变量
   ```
   ①新建变量
   MYSQL_HOME
   D:\java\MySQL\MySQL Server 5.7  
   ②修改Path
   %MYSQL_HOME%\bin
   ```

3. 创建 my.ini 配置文件(my.ini文件放在MySQL解压后的根目录下)
```
    [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8
   [mysqld]
   #设置3306端口
   port = 3306
   # 设置mysql的安装目录
   basedir=E:\java\mysql
   # 设置mysql数据库的数据的存放目录
   datadir=E:\java\mysql\data
   # 允许最大连接数
   max_connections=200
   # 服务端使用的字符集默认为8比特编码的latin1字符集
   character-set-server=utf8
   #创建新表时将使用的默认存储引擎
   explicit_defaults_for_timestamp=true
   default-storage-engine=INNODB
```

4. 初始化数据库,自动生成无密码的root用户
```shell
mysqld --initialize-insecure --user=mysql
```
5. 安装服务
以管理员身份打开cmd命令窗口，进入到mysql解压文件下的bin目录输入命令：
```shell
mysqld install
```
6. 命令启动服务、终止服务
```shell
net start mysql   #启动服务
net stop mysql   #终止服务
```
7. 修改密码(默认为空)  
cmd进入bin目录，  
登录命令： mysql -h 地址 -P端口 -u用户名 -p密码  
（提示输入密码时，直接回车即可以root身份进入管理MySQL了）  
修改密码命令：  
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root'

`移除mysql服务:cmd进入bin目录，输入下面命令： sc delete mysql`



## employees测试库
1. **[地址](https://launchpad.net/test-db/employees-db-1/1.0.6)** 下载文件 employees_db-full-1.0.6.tar.bz2
2. 解压缩下载的文件
> tar jxvf employees_db-full-1.0.6.tar.bz2
>cd employees_db-full-1.0.6

3. 修改文件 employees.sql
```sql
# 38行 set storage_engine = InnoDB; 替换
set default_storage_engine = InnoDB;
# 44行 select CONCAT('storage engine: ', @@storage_engine) as INFO; 替换
select CONCAT('storage engine: ', @@default_storage_engine) as INFO;
```
4. 导入数据库
> mysql -uroot -p -t < employees.sql