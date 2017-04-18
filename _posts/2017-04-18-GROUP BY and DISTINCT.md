---
layout: post
title: GROUP BY and DISTINCT
categories: sql
tags: sql
---


mysql version 5.7.15

初始数据
```
CREATE TEMPORARY TABLE tmp_group (
	val INT(10),
	group_1 VARCHAR (10),
	group_2 VARCHAR (10),
	group_3 VARCHAR (10)
);
INSERT INTO tmp_group VALUES (1, '1', '1', '1');
INSERT INTO tmp_group VALUES (10, '1', '1', '2');
INSERT INTO tmp_group VALUES (100, '1', '2', '1');
INSERT INTO tmp_group VALUES (1000, '1', '2', '2');
INSERT INTO tmp_group VALUES (10000, '2', '1', '1');
INSERT INTO tmp_group VALUES (100000, '2', '1', '2');
INSERT INTO tmp_group VALUES (1000000, '2', '2', '1');
INSERT INTO tmp_group VALUES (10000000, '2', '1', '2');
INSERT INTO tmp_group VALUES (100000000, '2', '2', '2');
INSERT INTO tmp_group VALUES (1, '1', '1', '1');
INSERT INTO tmp_group VALUES (10, '1', '1', '1');
INSERT INTO tmp_group VALUES (100, '1', '2', '1');
INSERT INTO tmp_group VALUES (1000, '2', '1', '1');
INSERT INTO tmp_group VALUES (10000, '1', '1', '1');
```


实际需求是这个sql分页查询（通过条件分组,并将结果数据分页查询）
```
SELECT SUM(val),group_1,group_2,group_3 FROM tmp_group GROUP BY group_1,group_2,group_3;
```
下面有两种方法求总的条数：
```
SELECT COUNT(1) FROM (SELECT SUM(val),group_1,group_2,group_3 FROM tmp_group GROUP BY group_1,group_2,group_3) a;
```
```
SELECT COUNT(DISTINCT group_1,group_2,group_3) FROM tmp_group;
```
