---
title: 《MySQL必知必会》第4-5章 SELECT检索与排序
date: 2019-12-19 22:54:41
categories: MySQL必知必会
toc: true
tags:
- MySQL
- MySQL必知必会
---

# 1. SELECT 语句
## 1.1 检索单个列
关键字SELECT后写需要查询的列名

<!-- more -->

```sql
mysql> select prod_name from products;
+----------------+
| prod_name      |
+----------------+
| Fuses          |
| JetPack 1000   |
| TNT (5 sticks) |
+----------------+
14 rows in set (0.00 sec)
```

> 1. 多条SQL语句必须使用分号(;)分隔,推荐做法.
> 2. SQL语句不区分大小写

## 1.2 检索多个列
和检索单列类似,关键字SELECT后写需要查询的列名,逗号分隔

```sql
mysql> select prod_id,prod_name,prod_price from products;
+---------+----------------+------------+
| prod_id | prod_name      | prod_price |
+---------+----------------+------------+
| ANV01   | .5 ton anvil   |       5.99 |
| ANV02   | 1 ton anvil    |       9.99 |
| ANV03   | 2 ton anvil    |      14.99 |
+---------+----------------+------------+
14 rows in set (0.00 sec)
```

## 1.3 检索所有列
使用通配符(*)即可
```sql
select * from products;
```

> 最好别使用*通配符,虽然此种写法方便,但是索引不需要的列通常会降低检索和应用程序的性能

## 1.4 检索不同行并去重
使用关键字DISTINCT,指示MySQL只返回不同值,从而达到去重目的.

```sql
mysql> select vend_id from products;
+---------+
| vend_id |
+---------+
|    1001 |
|    1002 |
|    1003 |
|    1003 |
|    1005 |
+---------+
14 rows in set (0.00 sec)

mysql> select distinct vend_id from products;
+---------+
| vend_id |
+---------+
|    1001 |
|    1002 |
|    1003 |
|    1005 |
+---------+
4 rows in set (0.01 sec)
```

> 1. DISTINCT关键字必须直接放在列名的前面
> 2. 不能不分使用DISTINCT,即DISTINCT关键字应用于所有列而不仅是前置它的列

## 1.5 限制结果 Limit使用
只返回指定个数的结果,在语句后使用关键字 limit,关键字后加行数即可
```sql
mysql> select prod_name from products limit 2;
+--------------+
| prod_name    |
+--------------+
| .5 ton anvil |
| 1 ton anvil  |
+--------------+
2 rows in set (0.00 sec)
```

指定检索的开始行和行数 `limit m,n`
```sql
mysql> select prod_name from products limit 0,2;
+--------------+
| prod_name    |
+--------------+
| .5 ton anvil |
| 1 ton anvil  |
+--------------+
2 rows in set (0.00 sec)
```

    第一个数为开始位置,第二个数为要检索出来的行数.
    上面语句的意思 : 从第0行开始,检索出2行的数据.

> 在LIMIT中指定要检索的行数为检索的最大行数,如果没有足够的行,MySQL将只返回它能返回的那么多行.

## 1.6 使用完全限定的表名
完全限定的表名可以默认不添加,但是在某些场景下(后续介绍)完全限定的表名还是非常有必须要的.
```sql
mysql> select products.prod_name from products;
+----------------+
| prod_name      |
+----------------+
| .5 ton anvil   |
| 1 ton anvil    |
| 2 ton anvil    |
+----------------+
14 rows in set (0.00 sec)
```

# 2. 排序检索数据
## 2.1 按单个列排序
数据默认检索出来的顺序是按照数据最初在表中被添加的顺序,我们可以使用`ORDER BY`子句来显示的指定排序字段.
```sql
mysql> select prod_name from products order by prod_name;
+----------------+
| prod_name      |
+----------------+
| .5 ton anvil   |
| 1 ton anvil    |
| 2 ton anvil    |
| Bird seed      |
| Carrots        |
| Detonator      |
| Fuses          |
| JetPack 1000   |
| JetPack 2000   |
| Oil can        |
| Safe           |
| Sling          |
| TNT (1 stick)  |
| TNT (5 sticks) |
+----------------+
14 rows in set (0.00 sec)
```

> 此处是按照字典书序排序

## 2.2 按多个列排序
多个列排序,只要指定列名,列名之间用逗号分隔即可.
检索三个列,首先按价格排序,然后再按名称排序:
```sql
mysql> select prod_id,prod_price,prod_name from products order by prod_price,prod_name;
+---------+------------+----------------+
| prod_id | prod_price | prod_name      |
+---------+------------+----------------+
| FC      |       2.50 | Carrots        |
| TNT1    |       2.50 | TNT (1 stick)  |
| FU1     |       3.42 | Fuses          |
| SLING   |       4.49 | Sling          |
| ANV01   |       5.99 | .5 ton anvil   |
| OL1     |       8.99 | Oil can        |
| ANV02   |       9.99 | 1 ton anvil    |
| FB      |      10.00 | Bird seed      |
| TNT2    |      10.00 | TNT (5 sticks) |
| DTNTR   |      13.00 | Detonator      |
| ANV03   |      14.99 | 2 ton anvil    |
| JP1000  |      35.00 | JetPack 1000   |
| SAFE    |      50.00 | Safe           |
| JP2000  |      55.00 | JetPack 2000   |
+---------+------------+----------------+
14 rows in set (0.00 sec)
```

> 此例中,仅在多个行具有相同的prod_price值时才对产品按prod_name进行排序.
> 如果prod_price列中所有的值都是唯一的,则不会按prod_name排序.

## 2.3 指定排序方向
数据排序分为两种 : 升序(ASC)和降序(DESC). 默认的排序方式是升序.

DESC关键字只应用到直接位于其前面的列名,如果想在多个列上进行降序排序,就必须对每个列指定DESC关键字

```sql
mysql> select prod_id,prod_price,prod_name from products order by prod_price desc,prod_name desc;
+---------+------------+----------------+
| prod_id | prod_price | prod_name      |
+---------+------------+----------------+
| JP2000  |      55.00 | JetPack 2000   |
| SAFE    |      50.00 | Safe           |
| JP1000  |      35.00 | JetPack 1000   |
| ANV03   |      14.99 | 2 ton anvil    |
| DTNTR   |      13.00 | Detonator      |
| TNT2    |      10.00 | TNT (5 sticks) |
| FB      |      10.00 | Bird seed      |
| ANV02   |       9.99 | 1 ton anvil    |
| OL1     |       8.99 | Oil can        |
| ANV01   |       5.99 | .5 ton anvil   |
| SLING   |       4.49 | Sling          |
| FU1     |       3.42 | Fuses          |
| TNT1    |       2.50 | TNT (1 stick)  |
| FC      |       2.50 | Carrots        |
+---------+------------+----------------+
14 rows in set (0.00 sec)
```

> 在字典排序中,A被视为与a相同,这是MySQL等大多数数据库管理系统默认行为,
> 如果想要更改需要DBA的帮助

## 2.4 ORDER BY与LIMIT组合
ORDER BY与LIMIT组合,可以找出一列中最高或最低的值

```sql
mysql> select prod_price from products order by prod_price desc limit 1;
+------------+
| prod_price |
+------------+
|      55.00 |
+------------+
1 row in set (0.00 sec)
```

> ORDER BY子句必须在FROM子句之后,LIMIT子句必须在ORDER BY子句之后
> 但是这两个关键字联合使用会有问题,后面会单独讲解.


## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
