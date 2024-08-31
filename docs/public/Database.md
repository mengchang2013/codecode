# 数据库 <!-- {docsify-ignore-all} -->
<!-- MarkdownTOC -->
- [In和exists ](#In和exists )


<!-- /MarkdownTOC -->
## In和exists 
```
in和exists对比：
select * from T1 where T1.a in (select T2.a from T2) ;
    T1数据量非常大而T2数据量小时，T1>>T2 时，2) 的查询效率高。

select * from T1 where exists(select 1 from T2 where T1.a=T2.a) ;
    T1数据量小而T2数据量非常大时，T1<<T2 时，1) 的查询效率高。
    
in：优先查询子表，把外表和内表作hash 连接，然后按照条件进行筛选。所以相对内表比较小的时候，in的速度较快。exists：优先查询外表，对外表作loop循环，每次loop循环再对内表进行查询。

简而言之，一般式：外表大，用IN；内表大，用EXISTS。最优化匹配原则，拿最小记录匹配大记录

not in 和not exists
not in ：内外表都进行全表扫描，没有用到索引；
not exists 的子查询依然能用到表上的索引。  (尽量使用)
not in：如果子查询中返回的任意一条记录含有空值，则查询将不返回任何记录。如果子查询字段有非空限制，这时可以使用in
```

## select执行顺序
```
SQL查询处理的步骤序号：

(8)SELECT  (9) DISTINCT (11) <TOP_specification> <select_list>
     (1) FROM <left_table> (3) <join_type> JOIN <right_table> 
            (2) ON <join_condition>  (4) WHERE <where_condition>  
                    (5) GROUP BY <group_by_list>  (6) WITH {CUBE | ROLLUP} 
                         (7) HAVING <having_condition>  (10) ORDER BY <order_by_list>

select语句完整的执行顺序： 
    1、from子句组装来自不同数据源的数据；
    2、where子句基于指定的条件对记录行进行筛选；
    3、group by子句将数据划分为多个分组；
    4、使用聚集函数进行计算；
    5、使用having子句筛选分组；
    6、计算所有的表达式；
    7、select 的字段；
    8、使用order by对结果集进行排序。
```

## 分页查询
**mysql**
```mysql
select * from 表名 where 条件 limit 从第几条取， 取几条;
```
**oracle**
```oracle
SELECT
	t2.* 
FROM
	( SELECT t1.*, rownum rn FROM ( SELECT * FROM emp ) t1 WHERE rownum <= 'currPage*pageSize' ) t2 
WHERE
	rn >=(
	currPage - 1 
	)* pageSize + 1;
```
`currPage:当前页数;pageSize:每页显示的条数`









