#大纲

	一、为什么要学习数据库
	二、数据库的相关概念      
		DBMS、DB、SQL
	三、数据库存储数据的特点
	四、初始MySQL
		MySQL产品的介绍        
		MySQL产品的安装          ★        
		MySQL服务的启动和停止     ★
		MySQL服务的登录和退出     ★      
		MySQL的常见命令和语法规范      
	五、DQL语言的学习   ★              
		基础查询        ★             
		条件查询  	   ★			
		排序查询  	   ★				
		常见函数        ★               
		分组函数        ★              
		分组查询		   ★			
		连接查询	 	★			
		子查询       √                  
		分页查询       ★              
		union联合查询	√			
		
	六、DML语言的学习    ★             
		插入语句						
		修改语句						
		删除语句						
	七、DDL语言的学习  
		库和表的管理	 √				
		常见数据类型介绍  √          
		常见约束  	  √			
	八、TCL语言的学习
		事务和事务处理                 
	九、视图的讲解           √
	十、变量                      
	十一、存储过程和函数   
	十二、流程控制结构       



#数据库相关

## 数据库的好处

1.持久化数据到本地
2.可以实现结构化查询，方便管理



## 数据库相关概念

保存数据的容器：数组、集合、文件等

1、DB：数据库，存储数据的“仓库”，保存一组有组织的数据的容器
2、DBMS：数据库管理系统，又称为数据库软件（产品），用于管理DB中的数据
3、SQL：结构化查询语言，用于和数据库通信的语言，不是某个数据库软件特有的，而是几乎所有的主流数据库软件通用的语言



## 数据库存储数据的特点

1、将数据放到表中，表再放到库中
2、一个数据库中可以有多个表，每张表具有唯一的表名用来标识自己
3、表具有一些特性，这些特性定义了数据在表中如何存储，类似java中 “类”的设计。
4、表中有一个或多个列，列又称为“字段”，相当于java中“属性”
5、表中的每一行数据，相当于java中“对象”



## SQL的优点

1. 不是某个特定数据库供应商专有的语言，几乎所有 DBMS都支持SQL 
2. 简单易学 
3. 虽然简单，但实际上是一种强有力的语言，灵活使 用其语言元素，可以进行非常复杂和高级的数据库操作。



## SQL语言分类

1. DML（Data Manipulation Language):数据操纵语句，用于添 加、删除、修改、查询数据库记录，并检查数据完整性 
2. DDL（Data Definition Language):数据定义语句，用于库和 表的创建、修改、删除。 
3. DCL（Data Control Language):数据控制语句，用于定义用 户的访问权限和安全级别。

### DML

DML用于查询与修改数据记录，包括如下SQL语句： 

- INSERT：添加数据到数据库中 
- UPDATE：修改数据库中的数据 
- DELETE：删除数据库中的数据 
- SELECT：选择（查询）数据 
  -  SELECT是SQL语言的基础，最为重要

### DDL

DDL用于定义数据库的结构，比如创建、修改或删除 数据库对象，包括如下SQL语句： 

- CREATE TABLE：创建数据库表 
- ALTER TABLE：更改表结构、添加、删除、修改列长度 
- DROP TABLE：删除表 
- CREATE INDEX：在表上建立索引 
- DROP INDEX：删除索引

### DCL

DCL用来控制数据库的访问，包括如下SQL语句： 

- GRANT：授予访问权限 
- REVOKE：撤销访问权限 
- COMMIT：提交事务处理 
- ROLLBACK：事务处理回退 
- SAVEPOINT：设置保存点 
- LOCK：对数据库的特定部分进行锁



# MySQL概述

MySQL数据库隶属于MySQL AB公司，总部位于瑞典，08年被sun公司收购，09年sun被oracle收购

优点： 

1、开源、免费、成本低
2、性能高、移植性也好
3、体积小，便于安装



## MySQL程序结构

![image-20210308175459703](https://gitee.com/koala010/typora/raw/master/img/image-20210308175459703.png)



## 常见命令

### 启动和停止

	方式一：计算机——右击管理——服务
	方式二：通过管理员身份运行
	net start 服务名（启动服务）
	net stop 服务名（停止服务）

### MySQL服务的登录和退出   

```mysql
方式一：通过mysql自带的客户端
只限于root用户

方式二：通过windows自带的客户端
登录：
mysql 【-h主机名 -P端口号 】-u用户名 -p密码

退出：
exit或ctrl+C
```

###常见命令 

	1.查看当前所有的数据库
	show databases;
	2.打开指定的库
	use 库名
	3.查看当前库的所有表
	show tables;
	4.查看其它库的所有表
	show tables from 库名;
	5.创建表
	create table 表名(
	
		列名 列类型,
		列名 列类型，
		。。。
	);
	6.查看表结构
	desc 表名;


```sql
7.查看服务器的版本
方式一：登录到mysql服务端
select version();
方式二：没有登录到mysql服务端
mysql --version
或
mysql --V

show databases； 查看所有的数据库
use 库名； 打开指定 的库
show tables ; 显示库中的所有表
show tables from 库名;显示指定库中的所有表
create table 表名(
	字段名 字段类型,	
	字段名 字段类型
); 创建表

desc 表名; 查看指定表的结构
select * from 表名;显示表中的所有数据
```



###MySQL的语法规范

1. 不区分大小写,但建议关键字大写，表名、列名小写
2. SQL 可以写在一行或者多行，各子句一般要分行写。
3. 关键字不能被缩写也不能分行
4. 每条命令最好用分号结尾，也可用\g结尾
5. 每条命令根据需要，可以进行缩进或换行，提高语句的可读性
6. 注释
   	单行注释：#注释文字
   	单行注释：-- 注释文字
   	多行注释：/* 注释文字  */



## MySQL的语言分类

```mysql
DQL（Data Query Language）：数据查询语言
	select 
DML(Data Manipulate Language):数据操作语言
	insert 、update、delete
DDL（Data Define Languge）：数据定义语言
	create、drop、alter
TCL（Transaction Control Language）：事务控制语言
	commit、rollback
```



# 查询

## 基础查询

### 语法

	语法：
	SELECT 要查询的东西
	【FROM 表名】;
	
	SELECT *|{[DISTINCT] column|expression [alias],...}
	FROM table;
	
	类似于Java中 :System.out.println(要打印的东西);
	特点：
	①通过select查询完的结果 ，是一个虚拟的表格，不是真实存在
	②要查询的东西 可以是常量值、表达式、字段、函数

### 别名

- 重命名一个列
- 便于计算。
- 紧跟列名，也可以在列名和别名之间加入关键字 ‘AS’，别名使用**双引号**，以便在别名中包含空格或特殊的字符并区分大小写。

### 字符串

- 字符串可以是 SELECT 列表中的一个字符,数字,日 期。 
- 日期和字符只能在单引号中出现。 
-  每当返回一行时，字符串被输出一次。

### 案例

```mysql
#基础查询
/*
语法：
select 查询列表 from 表名;

类似于：System.out.println(打印东西);

特点：

1、查询列表可以是：表中的字段、常量值、表达式、函数
2、查询的结果是一个虚拟的表格
*/

USE myemployees;

#1.查询表中的单个字段

SELECT last_name FROM employees;

#2.查询表中的多个字段
SELECT last_name,salary,email FROM employees;

#3.查询表中的所有字段

#方式一：
SELECT 
    `employee_id`,
    `first_name`,
    `last_name`,
    `phone_number`,
    `last_name`,
    `job_id`,
    `phone_number`,
    `job_id`,
    `salary`,
    `commission_pct`,
    `manager_id`,
    `department_id`,
    `hiredate` 
FROM
    employees ;
#方式二：  
 SELECT * FROM employees;
 
 #4.查询常量值
 SELECT 100;
 SELECT 'john';
 
 #5.查询表达式
 SELECT 100%98;
 
 #6.查询函数
 SELECT VERSION();
 
 #7.起别名
 /*
 ①便于理解
 ②如果要查询的字段有重名的情况，使用别名可以区分开来
 */
 #方式一：使用as
SELECT 100%98 AS 结果;
SELECT last_name AS 姓,first_name AS 名 FROM employees;

#方式二：使用空格
SELECT last_name 姓,first_name 名 FROM employees;

#案例：查询salary，显示结果为 out put
SELECT salary AS "out put" FROM employees;

#8.去重
#案例：查询员工表中涉及到的所有的部门编号
SELECT DISTINCT department_id FROM employees;


#9.+号的作用
/*
java中的+号：
①运算符，两个操作数都为数值型
②连接符，只要有一个操作数为字符串

mysql中的+号：
仅仅只有一个功能：运算符

select 100+90; 两个操作数都为数值型，则做加法运算
select '123'+90;只要其中一方为字符型，试图将字符型数值转换成数值型
			如果转换成功，则继续做加法运算
select 'john'+90;	如果转换失败，则将字符型数值转换成0

select null+10; 只要其中一方为null，则结果肯定为null
*/

#案例：查询员工名和姓连接成一个字段，并显示为 姓名
SELECT CONCAT('a','b','c') AS 结果;

SELECT 
	CONCAT(last_name,first_name) AS 姓名
FROM
	employees;
	
10、【补充】concat函数
功能：拼接字符
select concat(字符1，字符2，字符3,...);

11、【补充】ifnull函数
功能：判断某字段或表达式是否为null，如果为null 返回指定的值，否则返回原本的值
select ifnull(commission_pct,0) from employees;

12、【补充】isnull函数
功能：判断某字段或表达式是否为null，如果是，则返回1，否则返回0
	
```



## 条件查询

### 语法

```mysql
条件查询：根据条件过滤原始表的数据，查询到想要的数据
语法：
select 
	要查询的字段|表达式|常量值|函数
from 
	表
where 
	条件 ;
```

### 条件表达式

```mysql
示例：salary>10000
条件运算符：
> < >= <= = != <>

#案例1：查询工资>12000的员工信息

SELECT 
	*
FROM
	employees
WHERE
	salary>12000;
	
	
#案例2：查询部门编号不等于90号的员工名和部门编号
SELECT 
	last_name,
	department_id
FROM
	employees
WHERE
	department_id<>90;
```

### 逻辑表达式

```mysql
示例：salary>10000 && salary<20000

逻辑运算符：
	and（&&）:两个条件如果同时成立，结果为true，否则为false
	or(||)：两个条件只要有一个成立，结果为true，否则为false
	not(!)：如果条件成立，则not后为false，否则为true
	
#案例1：查询工资z在10000到20000之间的员工名、工资以及奖金
SELECT
	last_name,
	salary,
	commission_pct
FROM
	employees
WHERE
	salary>=10000 AND salary<=20000;
#案例2：查询部门编号不是在90到110之间，或者工资高于15000的员工信息
SELECT
	*
FROM
	employees
WHERE
	NOT(department_id>=90 AND  department_id<=110) OR salary>15000;
```

### 模糊查询

#### like

一般搭配通配符使用，可以判断字符型或数值型
通配符：%任意多个字符，_任意单个字符

```mysql
示例：last_name like 'a%'

#案例1：查询员工名中包含字符a的员工信息

select 
	*
from
	employees
where
	last_name like '%a%';#abc
#案例2：查询员工名中第三个字符为e，第五个字符为a的员工名和工资
select
	last_name,
	salary
FROM
	employees
WHERE
	last_name LIKE '__n_l%';



#案例3：查询员工名中第二个字符为_的员工名

SELECT
	last_name
FROM
	employees
WHERE
	last_name LIKE '_$_%' ESCAPE '$';
```

#### between and

①使用between and 可以提高语句的简洁度
②包含临界值
③两个临界值不要调换顺序

```mysql
#案例1：查询员工编号在100到120之间的员工信息

SELECT
	*
FROM
	employees
WHERE
	employee_id >= 120 AND employee_id<=100;
#----------------------
SELECT
	*
FROM
	employees
WHERE
	employee_id BETWEEN 120 AND 100;
```

#### in

含义：判断某字段的值是否属于in列表中的某一项
特点：
	①使用in提高语句简洁度
	②in列表的值类型必须一致或兼容
	③in列表中不支持通配符

```mysql
#案例：查询员工的工种编号是 IT_PROG、AD_VP、AD_PRES中的一个员工名和工种编号

SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id = 'IT_PROT' OR job_id = 'AD_VP' OR JOB_ID ='AD_PRES';


#------------------

SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id IN( 'IT_PROT' ,'AD_VP','AD_PRES');
```

#### is null /is not null

用于判断null值

```mysql
#案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NULL;


#案例1：查询有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NOT NULL;

#----------以下为×
SELECT
	last_name,
	commission_pct
FROM
	employees

WHERE 
	salary IS 12000;
```

#### 安全等于  <=>

```mysql
#案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct <=>NULL;
	
	
#案例2：查询工资为12000的员工信息
SELECT
	last_name,
	salary
FROM
	employees

WHERE 
	salary <=> 12000;

```

#### is null PK <=>

```mysql
			普通类型的数值	  null值		可读性
is null		×			    √		  √
<=>		    √			    √		  ×

```



## 排序查询

### 语法

```mysql
语法：
select 查询列表
from 表名
【where  筛选条件】
order by 排序的字段|表达式|函数|别名 【asc|desc】;
```

### 特点

1. asc代表的是升序，可以省略；desc代表的是降序

2. order by子句可以支持 单个字段、别名、表达式、函数、多个字段

3. order by子句在查询语句的最后面，除了limit子句

### 案例

```mysql
#1、按单个字段排序
SELECT * FROM employees ORDER BY salary DESC;

#2、添加筛选条件再排序

#案例：查询部门编号>=90的员工信息，并按员工编号降序

SELECT *
FROM employees
WHERE department_id>=90
ORDER BY employee_id DESC;


#3、按表达式排序
#案例：查询员工信息 按年薪降序


SELECT *,salary*12*(1+IFNULL(commission_pct,0))
FROM employees
ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC;


#4、按别名排序
#案例：查询员工信息 按年薪升序

SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 ASC;

#5、按函数排序
#案例：查询员工名，并且按名字的长度降序

SELECT LENGTH(last_name),last_name 
FROM employees
ORDER BY LENGTH(last_name) DESC;

#6、按多个字段排序

#案例：查询员工信息，要求先按工资降序，再按employee_id升序
SELECT *
FROM employees
ORDER BY salary DESC,employee_id ASC;

##实例
#1.查询员工的姓名和部门号和年薪，按年薪降序 按姓名升序

SELECT last_name,department_id,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 DESC,last_name ASC;


#2.选择工资不在8000到17000的员工的姓名和工资，按工资降序
SELECT last_name,salary
FROM employees

WHERE salary NOT BETWEEN 8000 AND 17000
ORDER BY salary DESC;

#3.查询邮箱中包含e的员工信息，并先按邮箱的字节数降序，再按部门号升序

SELECT *,LENGTH(email)
FROM employees
WHERE email LIKE '%e%'
ORDER BY LENGTH(email) DESC,department_id ASC;

```



## 常见函数

概念：类似于java的方法，将一组逻辑语句封装在方法体中，对外暴露方法名
好处：1、隐藏了实现细节  2、提高代码的重用性
调用：select 函数名(实参列表) 【from 表】;
特点：
	①叫什么（函数名）
	②干什么（函数功能）

分类：

1. 单行函数
   	如 concat、length、ifnull等

2. 分组函数
   	功能：做统计使用，又称为统计函数、聚合函数、组函数

### 单行函数

```mysql
一、单行函数
1、字符函数
	concat拼接
	substr截取子串
	upper转换成大写
	lower转换成小写
	trim去前后指定的空格和字符
	ltrim去左边空格
	rtrim去右边空格
	replace替换
	lpad左填充
	rpad右填充
	instr返回子串第一次出现的索引
	length 获取字节个数
	
2、数学函数
	round 四舍五入
	rand 随机数
	floor向下取整
	ceil向上取整
	mod取余
	truncate截断
3、日期函数
	now当前系统日期+时间
	curdate当前系统日期
	curtime当前系统时间
	str_to_date 将字符转换成日期
	date_format将日期转换成字符
4、流程控制函数
	if 处理双分支
	case语句 处理多分支
		情况1：处理等值判断
		情况2：处理条件判断
5、其他函数
	version版本
	database当前库
	user当前连接用户
```

#### 案例

```mysql
#一、字符函数

#1.length 获取参数值的字节个数
SELECT LENGTH('john');
# 不同编码下的汉字字节数不同，utf8为3个字节，gbk为2个字节
SELECT LENGTH('张三丰hahaha'); 

SHOW VARIABLES LIKE '%char%'

#2.concat 拼接字符串
SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

#3.upper、lower
SELECT UPPER('john');
SELECT LOWER('joHn');
#示例：将姓变大写，名变小写，然后拼接
SELECT CONCAT(UPPER(last_name),LOWER(first_name))  姓名 FROM employees;

#4.substr、substring
注意：索引从1开始
#截取从指定索引处后面所有字符
SELECT SUBSTR('李莫愁爱上了陆展元',7)  out_put;

#截取从指定索引处指定字符长度的字符
SELECT SUBSTR('李莫愁爱上了陆展元',1,3) out_put;

#案例：姓名中首字符大写，其他字符小写然后用_拼接，显示出来

SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2)))  out_put
FROM employees;

#5.instr 返回子串第一次出现的索引，如果找不到返回0
SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷八侠') AS out_put;

#6.trim
SELECT LENGTH(TRIM('    张翠山    ')) AS out_put;

SELECT TRIM('aa' FROM 'aaaaaaaaa张aaaaaaaaaaaa翠山aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa')  AS out_put;

#7.lpad 用指定的字符实现左填充指定长度
SELECT LPAD('殷素素',2,'*') AS out_put;

#8.rpad 用指定的字符实现右填充指定长度
SELECT RPAD('殷素素',12,'ab') AS out_put;


#9.replace 替换
SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;


#二、数学函数

#round 四舍五入
SELECT ROUND(-1.55);
SELECT ROUND(1.567,2);

#ceil 向上取整,返回>=该参数的最小整数
SELECT CEIL(-1.02);

#floor 向下取整，返回<=该参数的最大整数
SELECT FLOOR(-9.99);

#truncate 截断
SELECT TRUNCATE(1.69999,1);

#mod取余
/*
mod(a,b) ：  a-a/b*b
mod(-10,-3):-10- (-10)/(-3)*（-3）=-1
*/
SELECT MOD(10,-3);
SELECT 10%3;


#三、日期函数

#now 返回当前系统日期+时间
SELECT NOW();

#curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

#curtime 返回当前时间，不包含日期
SELECT CURTIME();

#可以获取指定的部分，年、月、日、小时、分钟、秒
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;
SELECT  YEAR(hiredate) 年 FROM employees;
SELECT MONTH(NOW()) 月;
SELECT MONTHNAME(NOW()) 月;

#str_to_date 将字符通过指定的格式转换成日期
SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

#查询入职日期为1992--4-3的员工信息
SELECT * FROM employees WHERE hiredate = '1992-4-3';
SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');

#date_format 将日期转换成字符
SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;

#查询有奖金的员工名和入职日期(xx月/xx日 xx年)
SELECT last_name,DATE_FORMAT(hiredate,'%m月/%d日 %y年') 入职日期
FROM employees
WHERE commission_pct IS NOT NULL;


#四、其他函数
SELECT VERSION();
SELECT DATABASE();
SELECT USER();


#五、流程控制函数
#1.if函数： if else 的效果
SELECT IF(10<5,'大','小');

SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金，呵呵','有奖金，嘻嘻') 备注
FROM employees;

#2.case函数的使用一： switch case 的效果
/*
java中
switch(变量或表达式){
	case 常量1：语句1;break;
	...
	default:语句n;break;
}

mysql中
case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end
*/

/*案例：查询员工的工资，要求
部门号=30，显示的工资为1.1倍
部门号=40，显示的工资为1.2倍
部门号=50，显示的工资为1.3倍
其他部门，显示的工资为原工资
*/
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

#3.case 函数的使用二：类似于 多重if
/*
java中：
if(条件1){
	语句1；
}else if(条件2){
	语句2；
}
...
else{
	语句n;
}

mysql中：
case 
when 条件1 then 要显示的值1或语句1
when 条件2 then 要显示的值2或语句2
。。。
else 要显示的值n或语句n
end
*/

#案例：查询员工的工资的情况
如果工资>20000,显示A级别
如果工资>15000,显示B级别
如果工资>10000，显示C级别
否则，显示D级别

SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;

##实例
#1.	显示系统时间(注：日期+时间)
SELECT NOW();

#2.	查询员工号，姓名，工资，以及工资提高百分之20%后的结果（new salary）
SELECT employee_id,last_name,salary,salary*1.2 "new salary"
FROM employees;

#3.	将员工的姓名按首字母排序，并写出姓名的长度（length）
SELECT LENGTH(last_name) 长度,SUBSTR(last_name,1,1) 首字符,last_name
FROM employees
ORDER BY 首字符;

#4.	做一个查询，产生下面的结果
<last_name> earns <salary> monthly but wants <salary*3>
Dream Salary
King earns 24000 monthly but wants 72000

SELECT CONCAT(last_name,' earns ',salary,' monthly but wants ',salary*3) AS "Dream Salary"
FROM employees
WHERE salary=24000;

#5.	使用case-when，按照下面的条件：
job                  grade
AD_PRES            A
ST_MAN             B
IT_PROG             C
SA_REP              D
ST_CLERK           E
产生下面的结果
Last_name	Job_id	Grade
king	AD_PRES	A

SELECT last_name,job_id AS  job,
CASE job_id
WHEN 'AD_PRES' THEN 'A' 
WHEN 'ST_MAN' THEN 'B' 
WHEN 'IT_PROG' THEN 'C' 
WHEN 'SA_PRE' THEN 'D'
WHEN 'ST_CLERK' THEN 'E'
END AS Grade
FROM employees
WHERE job_id = 'AD_PRES';
```



### 分组函数

功能：用作统计使用，又称为聚合函数或统计函数或组函数

分类：

- sum 求和
- max 最大值
- min 最小值
- avg 平均值
- count 计数

特点

1. 以上五个分组函数都忽略null值，除了`count(*）`
2. `sum`和`avg`一般用于处理数值型；`max`、`min`、`count`可以处理任何数据类型
3. 都可以搭配`distinct`使用，用于统计去重后的结果
4. `count`的参数可以支持：字段、*、常量值，一般放1,建议使用 `count( * )`
5. 和分组函数一同查询的字段，要求是`group by`后出现的字段

#### 案例

```mysql
#1、简单 的使用
SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
SELECT MIN(salary) FROM employees;
SELECT MAX(salary) FROM employees;
SELECT COUNT(salary) FROM employees;
SELECT SUM(salary) 和,AVG(salary) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;
SELECT SUM(salary) 和,ROUND(AVG(salary),2) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

#2、参数支持哪些类型
SELECT SUM(last_name) ,AVG(last_name) FROM employees;
SELECT SUM(hiredate) ,AVG(hiredate) FROM employees;
SELECT MAX(last_name),MIN(last_name) FROM employees;
SELECT MAX(hiredate),MIN(hiredate) FROM employees;

# 计算不为null的个数
SELECT COUNT(commission_pct) FROM employees;
SELECT COUNT(last_name) FROM employees;

#3、是否忽略null
SELECT SUM(commission_pct) ,AVG(commission_pct),SUM(commission_pct)/35,SUM(commission_pct)/107 FROM employees;
SELECT MAX(commission_pct) ,MIN(commission_pct) FROM employees;
SELECT COUNT(commission_pct) FROM employees;
SELECT commission_pct FROM employees;

#4、和distinct搭配
SELECT SUM(DISTINCT salary),SUM(salary) FROM employees;
SELECT COUNT(DISTINCT salary),COUNT(salary) FROM employees;

#5、count函数的详细介绍
SELECT COUNT(salary) FROM employees;
SELECT COUNT(*) FROM employees;
SELECT COUNT(1) FROM employees;

效率：
MYISAM存储引擎下  ，COUNT(*)的效率高
INNODB存储引擎下，COUNT(*)和COUNT(1)的效率差不多，比COUNT(字段)要高一些


#6、和分组函数一同查询的字段有限制
SELECT AVG(salary),employee_id  FROM employees;

## 实例
#1.查询公司员工工资的最大值，最小值，平均值，总和
SELECT MAX(salary) 最大值,MIN(salary) 最小值,AVG(salary) 平均值,SUM(salary) 和
FROM employees;

#2.查询员工表中的最大入职时间和最小入职时间的相差天数 （DIFFRENCE）
SELECT MAX(hiredate) 最大,MIN(hiredate) 最小,(MAX(hiredate)-MIN(hiredate))/1000/3600/24 DIFFRENCE
FROM employees;

SELECT DATEDIFF(MAX(hiredate),MIN(hiredate)) DIFFRENCE
FROM employees;

SELECT DATEDIFF('1995-2-7','1995-2-6');

#3.查询部门编号为90的员工个数
SELECT COUNT(*) FROM employees WHERE department_id = 90;
```



## 分组查询

