# 查询相关

## 子查询

参考：[MySQL里面的子查询](https://www.cnblogs.com/zhuiluoyu/p/5822481.html)

### 子查询分类

#### 标量子查询

> 是指子查询返回的是单一值的标量，如一个数字或一个字符串，也是子查询中最简单的返回形式。 可以使用 = > < >= <= <> 这些操作符对子查询的标量结果进行比较，通常子查询的位置在比较式的右侧。
>
> 返回单一值的标量，最简单的形式。

例子：

```sql
SELECT * FROM article WHERE uid = (SELECT uid FROM user WHERE status=1 ORDER BY uid DESC LIMIT 1)
SELECT * FROM t1 WHERE column1 = (SELECT MAX(column2) FROM t2)
SELECT * FROM article AS t WHERE 2 = (SELECT COUNT(*) FROM article WHERE article.uid = t.uid)
```

#### 列子查询

> 指子查询返回的结果集是 N 行一列，该结果通常来自对表的某个字段查询返回。
>
> 可以使用 IN、ANY、SOME 和 ALL 操作符，不能直接使用 = > < >= <= <> 这些比较标量结果的操作符。

**例子**:

```sql
SELECT * FROM article WHERE uid IN(SELECT uid FROM user WHERE status=1)
SELECT s1 FROM table1 WHERE s1 > ANY (SELECT s2 FROM table2)
SELECT s1 FROM table1 WHERE s1 > ALL (SELECT s2 FROM table2) 
```

NOT IN 是 <> ALL 的别名，二者相同。

**特殊情况**：

如果 table2 为空表，则 ALL 后的结果为 TRUE；

如果子查询返回如 (0,NULL,1) 这种尽管 s1 比返回结果都大，但有空行的结果，则 ALL 后的结果为 UNKNOWN 。

注意：对于 table2 空表的情况，下面的语句均返回 NULL：

```sql
SELECT s1 FROM table1 WHERE s1 > (SELECT s2 FROM table2)
SELECT s1 FROM table1 WHERE s1 > ALL (SELECT MAX(s1) FROM table2)
```

#### 行子查询

> 指子查询返回的结果集是一行 N 列，该子查询的结果通常是对表的某行数据进行查询而返回的结果集。

**例子**：

```sql
# 注：(1,2) 等同于 row(1,2)
SELECT * FROM table1 WHERE (1,2) = (SELECT column1, column2 FROM table2)
SELECT * FROM article WHERE (title,content,uid) = (SELECT title,content,uid FROM blog WHERE bid=2)
```

#### 表子查询

> 指子查询返回的结果集是 N 行 N 列的一个表数据。

**例子**：

```sql
SELECT * FROM article WHERE (title,content,uid) IN (SELECT title,content,uid FROM blog)
```

### 子查询举例

#### ANY进行子查询

> any关键词的意思是“对于子查询返回的列中的任何一个数值，如果比较结果为TRUE，就返回TRUE”。

好比“10 >any(11, 20, 2, 30)”，由于10>2，所以，该该判断会返回TRUE；只要10与集合中的任意一个进行比较，得到TRUE时，就会返回TRUE。　　

#### 使用IN进行子查询

使用in进行子查询，这个我们在日常写sql的时候是经常遇到的。in的意思就是指定的一个值是否在这个集合中，如何在就返回TRUE；否则就返回FALSE了。

- in是“=any”的别名，在使用“=any”的地方，我们都可以使用“in”来进行替换。

- 有了in，肯定就有了not in；not in并不是和<>any是同样的意思，not in和<>all是一个意思。

#### 使用ALL进行子查询

all必须与比较操作符一起使用。all的意思是“对于子查询返回的列中的所有值，如果比较结果为TRUE，则返回TRUE”。

- 好比“10 >all(2, 4, 5, 1)”，由于10大于集合中的所有值，所以这条判断就返回TRUE；而如果为“10 >all(20, 3, 2, 1, 4)”，这样的话，由于10小于20，所以该判断就会返回FALSE。
- <>all的同义词是not in，表示不等于集合中的所有值，这个很容易和<>any搞混，平时多留点心就好了。

#### 标量子查询

根据子查询返回值的数量，将子查询可以分为标量子查询和多值子查询。在使用比较符进行子查询时，就要求必须是标量子查询；如果是多值子查询时，使用比较符，就会抛出异常。

#### 多值子查询

与标量子查询对应的就是多值子查询了，多值子查询会返回一列、一行或者一个表，它们组成一个集合。我们一般使用的any、in、all和some等词，将外部查询与子查询的结果进行判断。如果将any、in、all和some等词与标量子查询，就会得到空的结果。

#### 独立子查询

> 独立子查询是不依赖外部查询而运行的子查询。什么叫依赖外部查询？先看下面两个sql语句。

sql语句1：获得所有hangzhou顾客的订单号。　

```sql
select order_id 
from table2 
where customer_id in
          (select customer_id 
          from table1 
          where city='hangzhou');
```

sql语句2：获得城市为hangzhou，并且存在订单的用户。

```sql
select * 
from table1 
where city='hangzhou' and exists
                (select * 
                from table2 
                where table1.customer_id=table2.customer_id);
```

上面的两条sql语句，虽然例子举的有点不是很恰当，但是足以说明这里的问题了。

　　　　对于sql语句1，我们将子查询单独复制出来，也是可以单独执行的，就是子查询与外部查询没有任何关系。

　　　　对于sql语句2，我们将子查询单独复制出来，就无法单独执行了，由于sql语句2的子查询依赖外部查询的某些字段，这就导致子查询就依赖外部查询，就产生了相关性。

　　对于子查询，很多时候都会考虑到效率的问题。当我们执行一个select语句时，可以加上explain关键字，用来查看查询类型，查询时使用的索引以及其它等等信息。比如这么用：

```sql
explain select order_id 
  from table2 
  where customer_id in
            (select customer_id 
            from table1 
            where city='hangzhou');
```

 　使用独立子查询，如果子查询部分对集合的最大遍历次数为n，外部查询的最大遍历次数为m时，我们可以记为：O(m+n)。而如果使用相关子查询，它的遍历 次数可能会达到O(m+m*n)。可以看到，效率就会成倍的下降；所以，大伙在使用子查询时，一定要考虑到子查询的相关性。

#### 相关子查询

　　相关子查询是指引用了外部查询列的子查询，即子查询会对外部查询的每行进行一次计算。但是在MySQL的内部，会进行动态优化，会随着情况的不同会 有所不同。使用相关子查询是最容易出现性能的地方。而关于sql语句的优化，这又是一个非常大的话题了，只能通过实际的经验积累，才能更好的去理解如何进 行优化。

#### EXISTS谓词

EXISTS是一个非常牛叉的谓词，它允许数据库高效地检查指定查询是否产生某些行。根据子查询是否返回行，该谓词返回TRUE或FALSE。与其 它谓词和逻辑表达式不同的是，无论输入子查询是否返回行，EXISTS都不会返回UNKNOWN，对于EXISTS来说，UNKNOWN就是FALSE。 还是上面的语句，获得城市为hangzhou，并且存在订单的用户。

```sql
select * 
from table1 
where city='hangzhou' and exists
                (select * 
                from table2 
                where table1.customer_id=table2.customer_id); 　
```

　　关于IN和EXISTS的主要区别在于三值逻辑的判断上。EXISTS总是返回TRUE或FALSE，而对于IN，除了TRUE、FALSE值外， 还有可能对NULL值返回UNKNOWN。但是在过滤器中，UNKNOWN的处理方式与FALSE相同，因此使用IN与使用EXISTS一样，SQL优化 器会选择相同的执行计划。

　　说到了IN和EXISTS几乎是一样的，但是，就不得不说到NOT IN和NOT EXISTS，对于输入列表中包含NULL值时，NOT EXISTS和NOT IN之间的差异就表现的非常大了。输入列表包含NULL值时，IN总是返回TRUE和UNKNOWN，因此NOT IN就会得到NOT TRUE和NOT UNKNOWN，即FALSE和UNKNOWN。

#### 派生表

上面也说到了，在子查询返回的值中，也可能返回一个表，如果将子查询返回的虚拟表再次作为FROM子句的输入时，这就子查询的虚拟表就成为了一个派生表。语法结构如下：

```sql
FROM (subquery expression) AS derived_table_alias
```

 　由于派生表是完全的虚拟表，并没有也不可能被物理地具体化。

例子：

```sql
SELECT s.name FROM (SELECT MIN(l.score) FROM student s LEFT JOIN lesson l ON s.id = l.id GROUP BY s.student_id) t WHERE t.min_score > 80 ;
```















