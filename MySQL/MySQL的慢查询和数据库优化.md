

## 数据库优化

[MySQL 对于千万级的大表要怎么优化？](https://www.zhihu.com/question/19719997/answer/81930332)

### mysql 单表多次查询和多表联合查询，哪个效率高?

![image-20210726210904393](https://gitee.com/koala010/typora/raw/master/img/20210726210911.png)



单表查询有利于后期数据量大了分库分表，如果联合查询的话，一旦分库，原来的sql都需要改动

