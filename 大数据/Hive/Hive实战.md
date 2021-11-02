> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 更多文章参看github知识库：[https://github.com/duktig666/knowledge](https://github.com/duktig666/knowledge)

# 背景

学习完Hadoop，有没有感到编写一个MapReduce程序非常复杂，想要进行一次分析和统计需要很大的开发成本。那么不如就来了解了解Hadoop生态圈的另一名成员——Hive。让我们一起来了解，如何使用类SQL语言进行快速查询和分析数据吧。

前边文章我们了解了Hive的概述、DDL语句和**DML语句（重点）**、分桶表和分区表、常用函数和压缩存储，这篇文章进行Hive的实战。

Hive系列文章如下：

- [大数据基础之Hive（一）—— Hive概述](https://blog.csdn.net/qq_42937522/article/details/121096763?spm=1001.2014.3001.5501)
- [大数据基础之Hive（二）—— DDL语句和DML语句](https://blog.csdn.net/qq_42937522/article/details/121096833?spm=1001.2014.3001.5501)
- [大数据基础之Hive（三）—— 分区表和分桶表](https://blog.csdn.net/qq_42937522/article/details/121096891?spm=1001.2014.3001.5501)
- [大数据基础之Hive（四）—— 常用函数和压缩存储](https://blog.csdn.net/qq_42937522/article/details/121096983?spm=1001.2014.3001.5501)
- [大数据基础之Hive（五）——Hive实战（统计电影排名的各种问题）](https://blog.csdn.net/qq_42937522/article/details/121097029)

# Hive实战

## 需求描述

统计硅谷影音视频网站的常规指标，各种 TopN 指标： 

- 统计视频观看数 Top10 
- 统计视频类别热度 Top10 
- 统计出视频观看数最高的 20 个视频的所属类别以及类别包含 Top20 视频的个数
- 统计视频观看数 Top50 所关联视频的所属类别排序 
- 统计每个类别中的视频热度 Top10,以Music 为例 
- 统计每个类别视频观看数 Top10 
- 统计上传视频最多的用户 Top10 以及他们上传的视频观看次数在前20 的视频

## 数据准备

### 数据字典

**视频表：**

| 字段         | 备注                           | 详细描述                  |
| ------------ | ------------------------------ | ------------------------- |
| videoId      | 视频唯一 id（`String`）        | 11 位字符串               |
| uploader     | 视频上传者（`String`）         | 上传视频的用户名          |
| age          | 视频年龄（`int`）              | 视频在平台上的整数天      |
| category     | 视频类别（`Array<String>`）    | 上传视频指定的视频分类    |
| length       | 视频长度（`int`）              | 整形数字标识的视频长度    |
| views        | 观看次数（`int`）              | 视频被浏览的次数          |
| rate         | 视频评分（`Double`）           | 满分5 分                  |
| Ratings      | 流量（`int`）                  | 视频的流量，整型数字      |
| conments     | 评论数（`int`）                | 一个视频的整数评论数      |
| relatedId    | 相关视频 id（`Array<String>`） | 相关视频的 id，最多 20 个 |
| **用户表：** |                                |                           |

| 字段     | 字段类型 | 备注         |
| -------- | -------- | ------------ |
| uploader | `string` | 上传者用户名 |
| videos   | `int `   | 上传视频数   |
| friends  | `int `   | 朋友数量     |
|          |          |              |

### 创建表

创建原始数据表：gulivideo_ori，gulivideo_user_ori， 
创建最终表（orc 存储格式带snappy 压缩的表）：gulivideo_orc，gulivideo_user_orc 

**创建原始数据表——gulivideo_ori**

```sql
create table gulivideo_ori( 
    videoId string comment '视频唯一 id',  
    uploader string comment '视频上传者(用户名)',  
    age int comment '视频年龄（视频在平台上的整数天 ）',  
    category array<string> comment '视频类别',  
    length int comment '视频长度',  
    views int comment '观看次数',  
    rate float comment '视频评分',  
    ratings int comment '流量',  
    comments int comment '评论数', 
    relatedId array<string> comment '相关视频 id（最多 20 个 ）'
) 
row format delimited fields terminated by "\t" 
collection items terminated by "&" 
stored as textfile; 
```

**创建原始数据表——gulivideo_user_ori**

```sql
create table gulivideo_user_ori( 
    uploader string comment '上传者用户名', 
    videos int comment '上传视频数 ', 
    friends int comment '朋友数量'
) 
row format delimited  
fields terminated by "\t"  
stored as textfile; 
```

**创建最终表——gulivideo_orc**

```sql
create table gulivideo_orc( 
    videoId string comment '视频唯一 id',  
    uploader string comment '视频上传者(用户名)',  
    age int comment '视频年龄（视频在平台上的整数天 ）',  
    category array<string> comment '视频类别',  
    length int comment '视频长度',  
    views int comment '观看次数',  
    rate float comment '视频评分',  
    ratings int comment '流量',  
    comments int comment '评论数', 
    relatedId array<string> comment '相关视频 id（最多 20 个 ）'
) 
row format delimited fields terminated by "\t" 
collection items terminated by "&" 
stored as orc 
tblproperties("orc.compress"="SNAPPY"); 
```

**创建最终表——gulivideo_user_orc **

```sql
create table gulivideo_user_orc ( 
    uploader string comment '上传者用户名', 
    videos int comment '上传视频数 ', 
    friends int comment '朋友数量'
) 
row format delimited  
fields terminated by "\t"  
stored as orc 
tblproperties("orc.compress"="SNAPPY"); 
```

### 插入数据

vedio部分数据：

```
LKh7zAJ4nwo	TheReceptionist	653	Entertainment	424	13021	4.34	1305	744	DjdA-5oKYFQ&NxTDlnOuybo&c-8VuICzXtU&DH56yrIO5nI&W1Uo5DQTtzc&E-3zXq_r4w0&1TCeoRPg5dE&yAr26YhuYNY&2ZgXx72XmoE&-7ClGo-YgZ0&vmdPOOd6cxI&KRHfMQqSHpk&pIMpORZthYw&1tUDzOp10pk&heqocRij5P0&_XIuvoH6rUg&LGVU5DsezE0&uO2kj6_D8B4&xiDqywcDQRM&uX81lMev6_o
7D0Mf4Kn4Xk	periurban	583	Music	201	6508	4.19	687	312	e2k0h6tPvGc&yuO6yjlvXe8&VqpnWBo-R4E&bdDskrr8jRY&y3IDp2n7B48&JngPWhfCb2M&KQaUvH5oiO4&NSzrwv5MCwc&NHB0a0xtLgU&DlRodd4s86s&EzKwOYLh-S0&eUIfRyrqwp8&AK8Wtfwe-1k&Eq4hGkIqBGw&N1lkLaLJHlc&-uIffs-DHkM&zpTorUhCd8Y&AvSK0qPw7EU&WX5KLMqY4bM&VKFqqoeMdjw
n1cEq1C8oqQ	Pipistrello	525	Comedy	125	1687	4.01	363	141	eprHhmurMHg&i30NkTJOrak&2XtLgZol5wI&3nH5Tccz8EQ&bSPVayE0NhE&sEqCkwPmQ_w&hut3VRL5XRE&bWlPSLUT-6U&dsBTo5LExr0&7PSvpPXppXA&yLup8wjbSIo&lbf4d1pZI9c&uRQYan_-CTQ&gnpvEvuiFoQ&F2_5KOnSsfI&DINu35v3eMU&9uSiyn7t_0o&YfShxdbAJS8&ssdfqTwZXY0&z5wDjq8o60c
OHkEzL4Unck	ichannel	638	Comedy	299	8043	4.4	518	371	eyUSTmEUQRg&FDIH1GNQXQE&Wtj31off8-I&mDjwzhc8dQ0&N4EYgXReBzM&NyC_0Z6zoUk&4DxyF39Myto&aiYwo5K0VWg&Ml2NaXU6gms&d0VYKbEbXQ8&LQUV_XGzHmA&8OmL_BJRLRw&qeCFW97-fOA&DVNwUKAuB3I&FMuWYExDEJk&rE7TuuXkk4E&bWicrzq2ApQ&jh6EpXnMb18&9JhU2jE02gg&nfBfC8bif1Y
-boOvAGNKUc	mrpitifulband	639	Music	287	7548	4.48	606	386	fmUwUURgsX0&bR27ACWomug&LlH7WcVptw8&saBmFpuwmKA&lhWk9SXUjWI&aVhSaa6aAOg&W-pvpxlOzZk&0vhVZQEzgcU&dDhCZVQf9po&zIkvMoezI1A&eV2SdBITv8k&cIO6nFDnNs4&Bd7nAtOEA3U&RZo5MisSTWo&geiABCqmQ84&MG1Xv99426g&7wj8-HkZ0XQ&JsdCu9T47iY&OUeN4DhCIFw&sf-Ym_pFP6U
hFFH8DaOHQg	istothehalfabee	592	Music	286	1759	4.45	539	244	hFFH8DaOHQg&ZIo-7BBDaPo&83SpuBijrBY&7TyH0ipgdtY&ZOdRUn0Q9eI&jqNs_S0n7P8&aWAzYehh0ag&vEtM1q6gm9Q&r89-fFx_tHU&h6Hw5030fKs&4qf7RSNCg40&LUXn57T8H50&ejPUALKGOn8&D6ABDEdhQLA&c8UYucsGdTU&El_Xbktje1k&6PAc6ZaK_WI&GUgJKzEmsYI&_sboDb75X2I&oDIIOV4VKlA
LzHjIj3fpR8	Xelanderthomas	686	Comedy	168	4545	4.58	273	167	udr9sLkoZ0s&3IU1GyX_zio&0E7Egr8Y1YI&qr8qZcvTLng&4WwVOWIqE80&Qeeq5OoLGJ0&YYDL1SqX-SY&vWGA5iYgAOU&8FeIj2HLN8k&bKlBTr88VTw&Y_59kWK5W3s&QlJSXVglZ3g&K3h_9O6OwW0&4ALe2z---e0&kdZk1Wk7kSw&hUa7f5XEzGE&aOihMldu_pE&PlPynB10vP0&W9DPlAZUH6Q&vta4RfQ2Z-I
SDNkMu8ZT68	w00dy911	630	People&Blogs	186	10181	3.49	494	257	rjnbgpPJUks
PkGUU_ggO3k	theresident	704	Entertainment	262	11235	3.85	247	280	PkGUU_ggO3k&EYC5bWF0ss8&EUPHdnE83GY&JO1LTIFOkTw&gVSzbvFnVRY&l9NJ04JiZj4&ay3gcr84YeQ&AfBxANiGnnU&RyWz8hwGbY4&BeJ7tGRgiW4&fbq2-jd5Dto&j8fTx5E5rik&qGkCtXLN1W0&mh_MGyx9tgc&bgn6RYut2lE&HS6Nqxh4uf4&m9Gq44o5pcA&K7unV366Qr4&shU2hfHKmU0&p0lq5-8IDqY
RX24KLBhwMI	lemonette	697	People&Blogs	512	24149	4.22	315	474	t60tW0WevkE&WZgoejVDZlo&Xa_op4MhSkg&MwynZ8qTwXA&sfG2rtAkAcg&j72VLPwzd_c&24Qfs69Al3U&EGWutOjVx4M&KVkseZR5coU&R6OaRcsfnY4&dGM3k_4cNhE&ai-cSq6APLQ&73M0y-iD9WE&3uKOSjE79YA&9BBu5N0iFBg&7f9zwx52xgA&ncEV0tSC7xM&H-J8Kbx9o68&s8xf4QX1UvA&2cKd9ERh5-8
```

user部分数据：

```
barelypolitical	151	5106
bonk65	89	144
camelcars	26	674
cubskickass34	13	126
boydism08	32	50
deckthree	6	753
fiveawesomegirls	182	3
ericielfenix	6	0
erricshade	3	49
blacktreemedia	520	3199
childfoundationcom	1	2
davedays	36	32072
fiveawesomeguys	160	2230
communitychannel	71	4280
ashantimusic	12	0
futvolg0les	9	0
all4tubekids	137	1333
ewupawly	65	143
frankjpmorgan	5	0
bethany9788	35	6
dingpolistico	30	1
cpfreak730	18	26
cmcgeh	14	0
chipmunked101	2	0
barNoNsouthport	21	0
```

插入数据：

向ori 表插入数据 

```sh
load data local inpath "./video" into table gulivideo_ori; 
load data local inpath "./user" into table gulivideo_user_ori; 
```

向orc 表插入数据 

```sql
insert into table gulivideo_orc select * from gulivideo_ori; 
insert into table gulivideo_user_orc select * from  gulivideo_user_ori; 
```

## 业务分析

### 1.统计视频观看数 Top10

思路：使用 order by 按照 views 字段做一个全局排序即可，同时我们设置只显示前 10条。 

最终代码： 

```sql
SELECT  
     videoId, 
     views 
FROM  
     gulivideo_ori 
ORDER BY  
     views DESC 
LIMIT 10; 
```

### 2.统计视频类别热度 Top10

思路：

1. 即统计每个类别有多少个视频，显示出包含视频最多的前 10 个类别。 
2. 我们需要按照类别 group by 聚合，然后 count 组内的videoId 个数即可。
3. 因为当前表结构为：一个视频对应一个或多个类别。所以如果要 group by 类别，需要先将类别进行**列转行**(展开)，然后再进行 count 即可。
4. 最后按照热度排序，显示前 10 条。 

代码实现：

写法一：

```sql
SELECT  
    t1.category_name ,  
    COUNT(t1.videoId) hot 
FROM  
( 
SELECT  
    videoId,  
    category_name  
FROM  
    gulivideo_orc  
lateral VIEW explode(category) gulivideo_orc_tmp AS category_name 
) t1 
GROUP BY  
    t1.category_name  
ORDER BY 
    hot  
DESC  
LIMIT 10;
```

写法二：

```sql
SELECT  
    t1.category_name ,  
    COUNT(t1.videoId) hot 
FROM  
( 
SELECT 
    explode(category) category_name
    FROM gulivideo_orc
)t1
GROUP BY  
    t1.category_name  
ORDER BY 
    hot  
DESC  
LIMIT 10;
```

### 3.统计出视频观看数最高的 20 个视频的所属类别以及类别包含Top20 视频的个数

思路：

1. 先找到观看数最高的 20 个视频所属条目的所有信息，降序排列 

   ```sql
   # t1
   SELECT  
       videoId,  
       views , 
       category  
   FROM  
       gulivideo_orc 
   ORDER BY  
       views  
   DESC  
   LIMIT 20;
   ```

2. 把这20 条信息中的category 分裂出来(列转行) 

   ```sql
   # t2
   SELECT 
       explode(category) category_name
   FROM t1;
   ```

3. 最后查询视频分类名称和该分类下有多少个 Top20 的视频

   ```sql
   SELECT  
   	t2.category_name, 
       COUNT(t2.videoId) video_sum 
   FROM  t2
   GROUP BY t2.category_name;
   ```

 

最终代码：

```sql
SELECT  
	t2.category_name, 
    COUNT(*) video_sum 
FROM  
    ( 
    SELECT 
        explode(category) category_name
    FROM  
        ( 
        SELECT  
            videoId,  
            views , 
            category  
        FROM  
            gulivideo_orc 
        ORDER BY views DESC  
        LIMIT 20  
        ) t1 
	) t2 
GROUP BY t2.category_name 
```

### 4.统计视频观看数 Top50 所关联视频的所属类别排序

思路：

1. 求出视频观看数Top50的视频所关联的视频(数组)

   ```sql
   # t1
   SELECT  
      relatedId,  
      views, 
   FROM  
      gulivideo_orc 
   ORDER BY views DESC  
   LIMIT 50;
   ```

2. 将关联视频 列转行

   ```sql
   # t2
   SELECT 
       explode(relatedId) related_id
   FROM t1;
   ```

3. JOIN原表,取出关联视频所属的类别（数组）

   ```sql
   # t3
   SELECT 
       g.category
   FROM t2
   JOIN gulivideo_orc g
   ON t2.related_id = g.vedioId;
   ```

4. 类别字段 列转行

   ```sql
   # t4
   SELECT 
       explode(category) category_name
   FROM t3;
   ```

5. 按照类别分组,求dount,并按照count排序

   ```sql
   SELECT  
       t4.category_name,  
       COUNT(*) hot 
   FROM t4
   GROUP BY t1.category_name  
   ORDER BY hot DESC;
   ```

最终代码：

```sql
SELECT  
    t4.category_name,  
    COUNT(*) hot 
FROM 
	(
	SELECT 
    	explode(category) category_name
	FROM
        (
		SELECT 
    		g.category
		FROM 
            (
			SELECT 
    			explode(relatedId) related_id
			FROM
                (
               	SELECT  
   					relatedId,  
   					views, 
				FROM  
   					gulivideo_orc 
				ORDER BY views DESC  
				LIMIT 50
                )t1
            )t2
		JOIN gulivideo_orc g
		ON t2.related_id = g.vedioId
        )t3
    )t4
GROUP BY t4.category_name  
ORDER BY hot DESC;
```



其他内容不一一列举。
