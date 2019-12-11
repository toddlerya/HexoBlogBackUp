---
title: "LIMIT和IFNULL的使用：LeetCode No.176"
date: 2019-05-12T09:37:20+08:00
lastmod: 2019-05-12T09:37:20+08:00
tags: [leetcode, SQL]
categories:
 - 笔记

typora-root-url: ..
typora-copy-images-to: ../images
---

​		下班在家无聊打开力扣（LeetCode国内版）随便逛逛，突然发现还有SQL题目，随便点开一道简单的题目，平常一般也就用用普通的CURD、分组、排序，多表关联查询什么的，这道题我是没做出来！各位小伙伴看看这道题，大家会不会做～这道题是这样子的：

![leetcode-sql-176-desc](/images/image-20190512094504177.png)

看起来普普通通哦，第一反应，排个序嘛～

```SQL
SELECT DISTINCT(Salary) AS SecondHighestSalary FROM Employee ORDER BY Salary DESC;
```

再然后就发现卡壳了，触发了我的知识盲区！

1. 如何只取出第2行数据？
2. 如何判断是否有第二高的薪水（有可能所有人薪水都一样或空表的情况）？
3. 如何返回null？

开始补习功课！

----

*各种数据库的SQL有一定差异，我们以使用较多的MySQL数据库的SQL语法为例。*

## 一、回顾LIMIT子句知识细节

首先我们回顾下如何限制只返回查询结果的前两行？

```sql
SELECT some_col FROM some_table LIMIT 2;
```

上述代码使用SELECT语言检索单独的一列数据。LIMIT 2指示MySQL数据库返回不超过2行的数据，其实这就是我们平时最常用的LIMIT子句，但这不是完整的LIMIT子句。

知识点来了！

#### 1. LIMIT子句的完整形态是这样子的：

```sql
LIMIT return_rows_number OFFSET start_index
```

#### 2. OFFSET默认值为0：

我们上面语句的LIMIT 2其实是LIMIT 2 OFFSET 0的默认简写形式，不显式声明OFFSET值，则OFFSET默认为0。

#### 3. 简化版的LIMIT子句：

```sql
LIMIT start_index, return_rows_number
```

接下来我们实战下看看效果，加深下理解：

![limit-2-sql-demo-image-20190512095326845](/images/image-20190512095326845.png)

**对于这道题，我们要获取第二行数据，应该这样写**

```sql
SELECT DISTINCT(Salary) FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1;
```

## 二、回顾IFNULL函数的细节

MySQL的IFNULL函数其实就是个if else语句，函数语法规则如下

```sql
IFNULL(expr1, expr2)
```

伪代码逻辑如下：

```python
if expr1 == null:
   return expr2;
else:
   return expr1;
```

执行两条SQL语句实践下更有助于理解：

![ifnull-sql-demo-image-20190512095828594](/images/image-20190512095828594.png)

对于我们这道题SQL应该这样写：

```sql
SELECT IFNULL((SELECT DISTINCT(Salary) FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1), null) AS SecondHighestSalary;
```

我们来测试下：

1 - 正常的数据，有各种薪资：

![image-20190512100622340](/images/image-20190512100622340.png)

2 - 所有人薪水都相同的情况：

![image-20190512100717179](/images/image-20190512100717179.png)

3 - 空表的情况：

![image-20190512100808599](/images/image-20190512100808599.png)

所有测试通过～

-----

这道题虽然标记为简单题目，但是提交通过率却不高，仅有1/3。

![image-20190512100927779](/images/image-20190512100927779.png)

说明还是有不少同学和我一样对SQL对掌握不够呀，还是要找时间好好学习使用下。

题目链接在这里：https://leetcode-cn.com/problems/second-highest-salary/

有兴趣对小伙伴可以试试其他数据库的SQL语句怎么写，欢迎交流讨论。