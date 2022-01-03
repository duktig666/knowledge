

## 数据库优化

[MySQL 对于千万级的大表要怎么优化？](https://www.zhihu.com/question/19719997/answer/81930332)

### mysql 单表多次查询和多表联合查询，哪个效率高?

![image-20210726210904393](https://gitee.com/koala010/typora/raw/master/img/20210726210911.png)



单表查询有利于后期数据量大了分库分表，如果联合查询的话，一旦分库，原来的sql都需要改动

## 慢查询优化

首先开启慢查询日志，记录慢查询语句，然后分析SQL语句，看看是不是查询了多余的行，然后分析语句执行计划查看索引使用情况，之后修改语句或者修改索引，使语句尽可能命中索引，如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行分表。

通过explain查看sql语句的执行计划，通过执行计划来分析索引使用情况，只需要将explain添加在sql语句之前即可。

表中的索引：

![表中的索引](https://gitee.com/koala010/typora/raw/master/img/20210626214501.png)

通过explain查看sql是否用到索引：

![explain查看sql的结果](https://gitee.com/koala010/typora/raw/master/img/20210626214556.png)

- **type** 的信息很明显的体现是否用到索引，它提供了判断查询是否高效的重要依据依据，如const(主键索引或者唯一二级索引进行等值匹配的情况下)，ref(普通的⼆级索引列与常量进⾏等值匹配)，index(扫描全表索引的覆盖索引) 。性能如下：`ALL < index < range ~ index_merge < ref < eq_ref < const < system`。 `ALL` 类型因为是全表扫描, 因此在相同的查询条件下, 它是速度最慢的. 而 `index` 类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比 ALL 类型的稍快。

  一般情况下，得保证查询至少达到range级别，最好能达到ref

  ```sql
  --all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
  explain select * from emp;
  
  --index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
  explain  select empno from emp;
  
  --range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
  explain select * from emp where empno between 7000 and 7500;
  
  --index_subquery：利用索引来关联子查询，不再扫描全表
  explain select * from emp where emp.job in (select job from t_job);
  
  --unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
   explain select * from emp e where e.deptno in (select distinct deptno from dept);
   
  --index_merge：在查询过程中需要多个索引组合使用，没有模拟出来
  explain select * from rental where rental_date like '2005-05-26 07:12:2%' and inventory_id=3926 and customer_id=321\G
  
  --ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
  explain select * from emp e where  e.mgr is null or e.mgr=7369;
  
  --ref：使用了非唯一性索引进行数据的查找
   create index idx_3 on emp(deptno);
   explain select * from emp e,dept d where e.deptno =d.deptno;
  
  --eq_ref ：使用唯一性索引进行数据查找
  explain select * from emp,emp2 where emp.empno = emp2.empno;
  
  --const：这个表至多有一个匹配行，
  explain select * from emp where empno = 7369;
   
  --system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
  ```

- **select_type：**主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

  ```sql
  --sample:简单的查询，不包含子查询和union
  explain select * from emp;
  
  --primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
  explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;
  
  --union:若第二个select出现在union之后，则被标记为union
  explain select * from emp where deptno = 10 union select * from emp where sal >2000;
  
  --dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
  explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)
  
  --union result:从union表获取结果的select
  explain select * from emp where deptno = 10 union select * from emp where sal >2000;
  
  --subquery:在select或者where列表中包含子查询
  explain select * from emp where sal > (select avg(sal) from emp) ;
  
  --dependent subquery:subquery的子查询要受到外部表查询的影响
  explain select * from emp e where e.deptno in (select distinct deptno from dept);
  
  --DERIVED: from子句中出现的子查询，也叫做派生类，
  explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;
  
  --UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
   explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
   
  --uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
  ```

- table：每个查询对应的表名 。

  1、对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集 1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

  2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

  3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

- **possible_key：**查询中可能用到的索引，一个或多个，查询涉及到的字段上若存在索引

- **key：**此字段是 MySQL 在当前查询时所真正使用到的索引。

- key_len：表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

- filtered：查询器预测满足下一次查询条件的百分比 。

- **rows:** 显示MySQL认为它执行查询时必须检查的行数。这个值非常直观显示 SQL 的效率好坏, 原则上 rows 越少越好。

- ref：显示索引的哪一列被使用了，如果可能的话，是一个常数

- extra：表示额外信息。



