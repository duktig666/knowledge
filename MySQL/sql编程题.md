## 一、学生成绩查询（自如三面）

```sql
CREATE TABLE `classes`  (
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `classes` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `score` int(10) NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of classes
-- ----------------------------
INSERT INTO `classes` VALUES ('张三', '语文', 75);
INSERT INTO `classes` VALUES ('张三', '数学', 82);
INSERT INTO `classes` VALUES ('李四', '语文', 89);
INSERT INTO `classes` VALUES ('李四', '数学', 92);
INSERT INTO `classes` VALUES ('王五', '英语', 77);
```

1、 查询所有科目都大于80分的学生

```sql
SELECT DISTINCT name FROM grade WHERE name NOT IN(SELECT DISTINCT name FROM grade WHERE score <=80);

SELECT name FROM grade GROUP BY name HAVING MIN(score) > 80;
```

2、 查询平均分大于90分的学生

```sql
SELECT name FROM (SELECT COUNT(*) AS t,SUM(score) AS num,name FROM `grade` GROUP BY name) AS a WHERE a.num > 80*t;

select name, avg(score) as sc from grade g1 group by name having avg(score)>80 ;
```

## 二、Java全栈知识体系  SQL编程题

### sql文件

```sql
/*
 Navicat MySQL Data Transfer

 Source Server         : 本地mysql
 Source Server Type    : MySQL
 Source Server Version : 80012
 Source Host           : localhost:3306
 Source Schema         : practice

 Target Server Type    : MySQL
 Target Server Version : 80012
 File Encoding         : 65001

 Date: 10/01/2022 20:24:48
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for course
-- ----------------------------
DROP TABLE IF EXISTS `course`;
CREATE TABLE `course`  (
  `CNO` varchar(5) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `CNAME` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `TNO` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of course
-- ----------------------------
INSERT INTO `course` VALUES ('3-105', '计算机导论', '825');
INSERT INTO `course` VALUES ('3-245', '操作系统', '804');
INSERT INTO `course` VALUES ('6-166', '数据电路', '856');
INSERT INTO `course` VALUES ('9-888', '高等数学', '100');

-- ----------------------------
-- Table structure for score
-- ----------------------------
DROP TABLE IF EXISTS `score`;
CREATE TABLE `score`  (
  `SNO` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `CNO` varchar(5) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `DEGREE` decimal(10, 1) NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of score
-- ----------------------------
INSERT INTO `score` VALUES ('103', '3-245', 86.0);
INSERT INTO `score` VALUES ('105', '3-245', 75.0);
INSERT INTO `score` VALUES ('109', '3-245', 68.0);
INSERT INTO `score` VALUES ('103', '3-105', 92.0);
INSERT INTO `score` VALUES ('105', '3-105', 88.0);
INSERT INTO `score` VALUES ('109', '3-105', 76.0);
INSERT INTO `score` VALUES ('101', '3-105', 64.0);
INSERT INTO `score` VALUES ('107', '3-105', 91.0);
INSERT INTO `score` VALUES ('101', '6-166', 85.0);
INSERT INTO `score` VALUES ('107', '6-106', 79.0);
INSERT INTO `score` VALUES ('108', '3-105', 78.0);
INSERT INTO `score` VALUES ('108', '6-166', 81.0);

-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`  (
  `SNO` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `SNAME` varchar(4) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `SSEX` varchar(2) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `SBIRTHDAY` datetime(0) NULL DEFAULT NULL,
  `CLASS` varchar(5) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('108', '曾华', '男', '1977-09-01 00:00:00', '95033');
INSERT INTO `student` VALUES ('105', '匡明', '男', '1975-10-02 00:00:00', '95031');
INSERT INTO `student` VALUES ('107', '王丽', '女', '1976-01-23 00:00:00', '95033');
INSERT INTO `student` VALUES ('101', '李军', '男', '1976-02-20 00:00:00', '95033');
INSERT INTO `student` VALUES ('109', '王芳', '女', '1975-02-10 00:00:00', '95031');
INSERT INTO `student` VALUES ('103', '陆君', '男', '1974-06-03 00:00:00', '95031');

-- ----------------------------
-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher`  (
  `TNO` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `TNAME` varchar(4) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `TSEX` varchar(2) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `TBIRTHDAY` datetime(0) NOT NULL,
  `PROF` varchar(6) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `DEPART` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of teacher
-- ----------------------------
INSERT INTO `teacher` VALUES ('804', '李诚', '男', '1958-12-02 00:00:00', '副教授', '计算机系');
INSERT INTO `teacher` VALUES ('856', '张旭', '男', '1969-03-12 00:00:00', '讲师', '电子工程系');
INSERT INTO `teacher` VALUES ('825', '王萍', '女', '1972-05-05 00:00:00', '助教', '计算机系');
INSERT INTO `teacher` VALUES ('831', '刘冰', '女', '1977-08-14 00:00:00', '助教', '电子工程系');

SET FOREIGN_KEY_CHECKS = 1;
```

![image-20220110203957838](https://cos.duktig.cn/typora/202201102040477.png)



**1、查询Student表中的所有记录的Sname、Ssex和Class列。**

```sql
SELECT sname,ssex,class FROM student;
```

**2、 查询教师所有的单位即不重复的Depart列。**

```sql
SELECT DISTINCT depart FROM teacher;
```

**3、 查询Student表的所有记录。**

```sql
SELECT * FROM student;
```

**4、 查询Score表中成绩在60到80之间的所有记录。**

```sql
SELECT * FROM score WHERE degree BETWEEN 60 AND 80;
```

**5、 查询Score表中成绩为85，86或88的记录。**

```sql
SELECT * FROM score WHERE degree IN (85,86,88);
```

**6、 查询Student表中“95031”班或性别为“女”的同学记录。**

```sql
SELECT * FROM student WHERE class="95031" OR ssex = "女";
```

**7、 以Class降序查询Student表的所有记录。**

```sql
SELECT * FROM student ORDER BY class DESC;
```

**8、 以Cno升序、Degree降序查询Score表的所有记录。**

```sql
SELECT * FROM score ORDER BY cno ASC,degree DESC;
```

**9、 查询“95031”班的学生人数。**

```sql
SELECT COUNT(1) FROM student WHERE class="95031";
```

**10、查询Score表中的最高分的学生学号和课程号。**

```sql
SELECT sno,cno FROM score GROUP BY sno,cno HAVING MAX(degree); ### 错

select sno,CNO from SCORE where DEGREE = (select max(DEGREE) from SCORE);
```

**11、查询‘3-105’号课程的平均分。**

```sql
SELECT AVG(degree) FROM score WHERE cno="3-105";
```

**12、查询Score表中至少有5名学生选修的并以3开头的课程的平均分数。**

```sql
SELECT AVG(degree) FROM score WHERE cno LIKE "3%" GROUP BY cno HAVING COUNT(sno) >= 5;
```

**13、查询最低分大于70，最高分小于90的Sno列。**

```sql
SELECT sno FROM score GROUP BY sno HAVING MIN(degree) > 70 AND MAX(degree) < 90;
```

**14、查询所有学生的Sname、Cno和Degree列。**

```sql
SELECT sname,cno,degree FROM student st LEFT JOIN score sc ON st.sno = sc.sno;
```

**15、查询所有学生的Sno、Cname和Degree列。**SELECT AVG(degree) FROM score WHERE sno IN (SELECT sno FROM student WHERE class = "95033");

```sql
SELECT st.sno,cname,degree FROM student st,score sc,course c WHERE st.sno = sc.sno AND sc.CNO = c.CNO;
```

**16、查询所有学生的Sname、Cname和Degree列。**

```sql
SELECT sname,cname,degree FROM student st,score sc,course c WHERE st.sno = sc.sno AND sc.CNO = c.CNO;

SELECT
  A.SNAME,
  B.CNAME,
  C.DEGREE
FROM STUDENT A
  JOIN (COURSE B, SCORE C)
    ON A.SNO = C.SNO AND B.CNO = C.CNO;
```

**17、查询“95033”班所选课程的平均分。**

```sql
SELECT AVG(degree) FROM score WHERE sno IN (SELECT sno FROM student WHERE class = "95033");
```

**18、假设使用如下命令建立了一个grade表:**

```sql
CREATE TABLE `grade`  (
  `low` decimal(3, 0) NULL DEFAULT NULL,
  `upp` decimal(3, 0) NULL DEFAULT NULL,
  `rank` char(1) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

insert into grade values (90, 100, 'A');
insert into grade values (80, 89, 'B');
insert into grade values (70, 79, 'C');
insert into grade values (60, 69, 'D');
insert into grade values (0, 59, 'E');
```

**现查询所有同学的Sno、Cno和rank列。**

```sql
SELECT s.sno,s.cno,g.rank FROM score s, grade g WHERE s.degree BETWEEN g.low AND g.upp; 
```

**19、查询选修“3-105”课程的成绩高于“109”号同学成绩的所有同学的记录。**

```sql
SELECT * FROM score WHERE cno = "3-105" AND degree > (SELECT degree FROM score WHERE sno = "109");
```

**20、查询score中选学一门以上课程的同学中分数为非最高分成绩的学生记录**

```sql
SELECT * FROM student WHERE sno
IN (SELECT sno FROM score 
			WHERE degree < (SELECT MAX(degree) FROM score) 
			GROUP BY sno HAVING COUNT(cno) > 1);
```

**21、查询成绩高于学号为“109”、课程号为“3-105”的成绩的所有记录。**

```sql
SELECT * FROM score WHERE cno = "3-105" AND degree > (SELECT degree FROM score WHERE sno = "109" AND cno = "3-105");
```

**22、查询和学号为108的同学同年出生的所有学生的Sno、Sname和Sbirthday列。**

```sql
SELECT sno,sname,sbirthday FROM student WHERE YEAR(sbirthday) = (SELECT YEAR(sbirthday) FROM student WHERE sno ="108");
```

**23、查询“张旭“教师任课的学生成绩。**

```sql
SELECT s.degree FROM teacher t,score s,course c WHERE t.tno = c.tno AND c.cno = s.cno AND t.tname = "张旭"; 

SELECT * FROM score s WHERE cno = (SELECT cno FROM course c INNER JOIN teacher t ON t.tno = c.tno WHERE t.tname = "张旭"); 
```

**24、查询选修某课程的同学人数多于5人的教师姓名。**

```sql
SELECT tname FROM teacher t 
INNER JOIN course c 
ON c.tno = t.tno 
WHERE cno = (SELECT cno FROM score GROUP BY cno HAVING COUNT(sno) >= 5);
```

**25、查询95033班和95031班全体学生的记录。**

```sql
SELECT * FROM student WHERE class = "95033" OR class = "95031";

SELECT * FROM student WHERE class IN ("95033","95031");
```

**26、查询存在有85分以上成绩的课程Cno.**

```sql
SELECT cno FROM score GROUP BY cno HAVING MAX(DEGREE )> 85;
```

**27、查询出“计算机系“教师所教课程的成绩表。**

```sql
SELECT * FROM score WHERE cno IN 
	(SELECT cno FROM teacher t LEFT JOIN course c ON t.tno = c.tno WHERE t.depart = '计算机系')；
```

**28、查询“计算机系”与“电子工程系“不同职称的教师的Tname和Prof**

```sql
SELECT 
  tname,prof
FROM teacher 
WHERE depart = '计算机系' AND prof NOT IN (
  SELECT prof
  FROM teacher 
  WHERE depart = '电子工程系'
);
```

**29、查询选修编号为“3-105“课程且成绩至少高于选修编号为“3-245”的同学的Cno、Sno和Degree,并按Degree从高到低次序排序。**

```sql
SELECT sno,cno,degree FROM score 
WHERE cno = '3-105' AND  
	degree > ANY(SELECT degree FROM score WHERE cno = '3-245') 
	ORDER BY degree DESC;
```

**30、查询选修编号为“3-105”且成绩高于选修编号为“3-245”课程的同学的Cno、Sno和Degree.**

```sql

SELECT sno,cno,degree FROM score 
WHERE cno = '3-105' AND  
	degree > ALL(SELECT degree FROM score WHERE cno = '3-245');
```

**31、查询所有教师和同学的name、sex和birthday.**

```sql
SELECT 
  tname     name,
  tsex     sex,
  tbirthday birthday
FROM teacher 
UNION
SELECT 
  sname     name,
  ssex      sex,
  sbirthday birthday
FROM student;
```

**32、查询所有“女”教师和“女”同学的name、sex和birthday.**

```sql
SELECT 
  tname     name,
  tsex     sex,
  tbirthday birthday
FROM teacher 
WHERE tsex = '女'
UNION
SELECT 
  sname     name,
  ssex      sex,
  sbirthday birthday
FROM student
WHERE ssex = '女';
```

**33、查询成绩比该课程平均成绩低的同学的成绩表。**

```sql
SELECT s1.* FROM score s1 
WHERE degree < (
    SELECT AVG(degree ) FROM score s2 
    WHERE s1.cno = s2.cno);
```

**34、查询所有任课教师的Tname和Depart.**

```sql
SELECT 
  tname,depart
FROM teacher t
WHERE EXISTS(SELECT tno
             FROM course c
             WHERE t.TNO = c.TNO);
```

```sql
SELECT 
  tname,depart
FROM teacher t
WHERE tno IN(SELECT tno
                  FROM course);
```

**35、查询所有未讲课的教师的Tname和Depart.**

```sql
SELECT 
  tname,depart
FROM teacher t
WHERE NOT EXISTS(SELECT tno
             FROM course c
             WHERE t.TNO = c.TNO);
```

```sql
SELECT 
  tname,depart
FROM teacher t
WHERE tno NOT IN(SELECT tno
                  FROM course);
```

**36、查询至少有2名男生的班号。**

```sql
SELECT class FROM student WHERE ssex = '男' GROUP BY class HAVING COUNT(class) >= 2;	
```

**37、查询Student表中不姓“王”的同学记录。**

```sql
SELECT * FROM student WHERE sname NOT LIKE '王%';
```

**38、查询Student表中每个学生的姓名和年龄。**

```sql
SELECT sname,YEAR(NOW())-YEAR(sbirthday) FROM student;
```

**39、查询Student表中最大和最小的Sbirthday日期值。**

```sql
SELECT MAX(sbirthday) max_sbirthday,MIN(sbirthday) min_sbirthday FROM student;
```

**40、以班号和年龄从大到小的顺序查询Student表中的全部记录。**

```sql
SELECT * FROM student ORDER BY class DESC,YEAR(NOW())-YEAR(sbirthday) DESC;
```

**41、查询“男”教师及其所上的课程。**

```sql
SELECT c.* FROM teacher t LEFT JOIN course c ON t.tno = c.tno WHERE t.tsex = '男';
```

**42、查询最高分同学的Sno、Cno和Degree列。**

```sql
SELECT
  sno,
  cno,
  degree
FROM score
WHERE DEGREE = (SELECT MAX(degree)
                FROM score);
```

**43、查询和“李军”同性别的所有同学的Sname.**

```sql
SELECT sname FROM student WHERE ssex = (SELECT ssex FROM student WHREE sname = '李军');
```

**44、查询和“李军”同性别并同班的同学Sname.**

```sql
SELECT sname FROM student WHERE (ssex, class) = (SELECT ssex,class FROM student WHREE sname = '李军');
```

**45、查询所有选修“计算机导论”课程的“男”同学的成绩表**

```sql
SELECT *
FROM score, student
WHERE score.SNO = student.SNO and ssex = '男' and cno = (
  SELECT cno
  FROM course
  WHERE cname = '计算机导论');
```



