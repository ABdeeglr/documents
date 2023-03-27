# MySQL SELECT 详解

## 01 基本使用方法

在确定使用的数据库后，使用

```sql
SELECT * FROM <table_name>;
```

获取数据库中某张表的详细数据。这段语句的返回值被称为“结果集”，目前不清楚这个概念的具体含义，但似乎是数据库专用的一种数据形式。

在进行更精细的数据筛选前，通常需要了解该数据表的各个字段，使用以下命令即可：

```sql
DESC <table_name>;
```

了解了这两条命令的格式，我们可以根据表的字段来精确选取我们想要的列的数据。比如说一张表有五个字段，分别是 customer_name, customer_id, birthday, city, address, 现在的需求是只展示 customer_name 和 birthday 列，此时只需要执行以下命令即可

```sql
SELECT customer_name, birthday
FROM <table_name>;
```

此外，关于 SELECT 的基础使用，还需要记住一个小技巧，那就是关键字 DISTINCT. 它的作用可以使 SELECT 返回的结果总是不同的。比如说

```sql
SELECT DISTINCT customer_name, birthday
FROM <table_name>;
```

可以避免返回具有相同生日的信息。需要注意的是，DISTINCT 应用于所有的列，当且仅当展示的所有字段上的值都相同时才会出现省略。

接下来，使用子句可以对输出内容进行更精细的控制，然而，这些字句的书写必须遵循特定顺序。下面将对各个字句的作用和语法分别介绍，最后简短地介绍字句的书写顺序。

## 02 条件筛选 - WHERE

条件筛选可以过滤出只符合特定条件的数据。

假设有这样的需求：一张表中存放着职员和工资等若干数据，现在要分别做四件事，分别为：

1. 查看名为 "Jack" 的职员的数据
2. 查看所有工资超过 2500 的数据
3. 查看所有工资在 2000 到 3000 以内的数据
4. 查看所有工资恰好为 2200, 2400 和 2800 的数据

满足四个需求的语句为：

```sql
SELECT ... FROM ... WHERE name = "Jack";
SELECT ... FROM ... WHERE salary > 2500;
SELECT ... FROM ... WHERE salary BETWEEN 2000 AND 3000;
SELECT ... FROM ... WHERE salary IN (2200, 2400, 2800);
```

这是四种基本的条件筛选方式。

如果你想设计更多的与数值有关的筛选，使用第二条语句作为模板，将 `>` 改成以下符号均可：

1. `=` 等于
2. `<>` 不等于
3. `!=` 不等于
4. `<` 小于
5. `<=` 小于等于
6. `>=` 大于等于

### 小 Tips

最后来说一些特殊需求。

**寻找空缺数据**

有时候一张表中会有空缺的数据，这时候使用缺省值 NULL 来筛选。比如说以下语句

```sql
SELECT * 
FROM customers 
WHERE phone IS NULL;
```

意味着从 customers 表中选出所有没有记录电话的行。

**混合条件**

如果既想要作空缺检查，还想要在另一个字段中筛选出符合某个范围的值怎么办？

或者想要选择姓为 "John", 且积分超过 300 的客户怎么办？

只用 AND / OR / NOT 这三个逻辑逻辑运算符就可以了。

实际上，条件筛选可以被分成两个部分：关键词 WHERE 和 条件表达式 （CONDITIONAL EXPRESSION）。如果某一行的数值代入到表达式中为真，那么就将其显示，否则就跳过。使用 AND OR NOT 逻辑运算符连接表达式，就能够进行混合条件筛选。

现在来回答上面提到的第二个问题，可以大概写成：

```sql
SELECT ...
FROM ...
WHERE name = "John" AND points > 300;
```

而如果要找住在洛杉矶，或者积分小于 500 分的客户，可以写成：

```sql
SELECT ...
FROM ...
WHERE living_city = "LA" OR points < 500;
```

## 03 排序 - ORDER BY

使用 ORDER BY 子句可以对结果集进行排序，比如一份数据本应按照主键进行排序输出，但现在使用 ORDER BY 字句根据积分进行排序：

```sql
SELECT customer_id, points
FROM customers
ORDER BY points;
```

其结果不再按照主键排序：

```shell
+-------------+--------+
| customer_id | points |
+-------------+--------+
|          11 |      0 |
|           8 |    205 |
|           4 |    457 |
|          10 |    796 |
|           2 |    947 |
|           3 |   2967 |
|           5 |   3675 |
+-------------+--------+
```

使用 DESC 关键字可以使排序的方向反转，因此

```sql
SELECT customer_id, points
FROM customers
ORDER BY points DESC;
```

的结果为

```shell
+-------------+--------+
| customer_id | points |
+-------------+--------+
|           1 |   2273 |
|           7 |   1672 |
|           9 |   1486 |
|           2 |    947 |
|          10 |    796 |
|          13 |    205 |
|          11 |      0 |
+-------------+--------+
```

当根据多个列进行排序时，DESC 关键字只应用于其前面的列，整体排序顺序仍然根据字段的书写顺序排序，比如

```sql
SELECT * FROM <table_name> ORDER BY f1, f2 DESC, f3;
```

表示，首先按照 f1 排序，然后对 f1 相同的内容根据 f2 排序，并且 f2 的排序为逆序，最后再按照 f3 排序。



## 04 抽样筛选 - LIMIT

使用 SELECT 语句筛选出的内容太多，只想要前五个，或者第 15 到 25 个怎么办？

LIMIT 字句解决了这个问题。

比如在使用

```sql
SELECT customer_id, name 
FROM customers;
```

时，总共有 12 个结果：

```shell
+-------------+-------------------+
| customer_id | name              |
+-------------+-------------------+
|           1 | Babara MacCaffrey |
|           2 | Ines Brushfield   |
|           3 | Freddi Boagey     |
|           4 | Ambur Roseburgh   |
|           5 | Clemmie Betchley  |
|           6 | Elka Twiddell     |
|           7 | Ilene Dowson      |
|           8 | Thacher Naseby    |
|           9 | Romola Rumgay     |
|          10 | Levy Mynett       |
|          11 | Jonathan Jhoster  |
|          12 | Milana Stein      |
+-------------+-------------------+
```

如果只需要前 5 行，则使用

```sql
SELECT customer_id, name 
FROM customers
LIMIT 5;
```

即可，如果还有更多需求，比如只需要 4-6 行的内容，则需要使用

```sql
SELECT customer_id, name 
FROM customers
LIMIT 3, 3;
```

其结果为

```shell
+-------------+------------------+
|           4 | Ambur Roseburgh  |
|           5 | Clemmie Betchley |
|           6 | Elka Twiddell    |
+-------------+------------------+
```

LIMIT 后面的两个数字的意思分别是，放弃前三行，再选择三行。假设有一个 100 行的数据表，要选择第 49-88 行的数据，那么就是说要放弃前 48 行，再选择 40 (88 -48) 行，因此可以写成

```sql
SELECT ... 
FROM ... 
LIMIT 48, 40;
```

## 05 复杂条件筛选 - WHERE 的子句 LIKE/REGEXP 等

前面介绍的条件筛选已经能够做很多事情了，不过在实际情况中，针对字符串进行的条件筛选更加多样，比如说要着找名字中包含 "BA" 的客户，或者是以 "Ja" 或 "Je" 作为名字开头的用户，这些针对字符串进行判断的需求不是简单的条件筛选能完成的，还需要下面两个概念：

1. LIKE 关键字和通配符
2. REGEXP 关键字和正则表达式

### 5.1 LIKE 和通配符







### 5.2 REGEXP 和正则表达式



## 06 数据分组

使用 GROUP BY 关键字和 HAVING 关键字。

示例：

```sql
SELECT COUNT(*) AS num_prods
FROM products
WHERE vend_id = 1003;

SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;

SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id WITH ROLLUP;

SELECT cust_id, COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;

SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;


```





## A - SELECT 子句顺序

SELECT 的字句必须遵循固定顺序，否则会报错。

```sql
SELECT ... -- 要返回的表达式或列
FROM ... -- 选择数据的范围
WHERE ... -- 行级过滤
GROUP BY ... -- 分组说明（在计算聚集时使用）
ORDER BY ... -- 输出排序顺序
LIMIT ... -- 要检索的行数
```











