# spark sql总结

## 一、内置函数

### 1.1、聚合函数

```sql
（1）any(expr)：如果'expr'的至少一个值为true，则返回true
eg：
SELECT any(col) FROM VALUES (true), (false), (false) AS tab(col);
+--------+
|any(col)|
+--------+
|    true|
+--------+

SELECT any(col) FROM VALUES (NULL), (true), (false) AS tab(col);
+--------+
|any(col)|
+--------+
|    true|
+--------+

SELECT any(col) FROM VALUES (false), (false), (NULL) AS tab(col);
+--------+
|any(col)|
+--------+
|   false|
+--------+
```

```sql
（2）approx_count_distinct(expr[, relativeSD])：近似统计expr去重后的数据量，有一定的误差（忽略null值）
eg:
SELECT approx_count_distinct(col1) FROM VALUES (1), (1), (2), (2), (3) tab(col1);
+---------------------------+
|approx_count_distinct(col1)|
+---------------------------+
|                          3|
+---------------------------+
```

```sql
（3）approx_percentile(col, percentage [, accuracy])：使用accuracy和percentage计算得到一个分布百分数，将col列从小到大排序，然后从col列中抽取出这个分布位置的值。
percentage必须介于0到1直接，accuracy默认值为10000
eg:
SELECT approx_percentile(col, array(0.5, 0.4, 0.1), 100) FROM VALUES (0), (1), (2), (10) AS tab(col);
+-------------------------------------------------+
|approx_percentile(col, array(0.5, 0.4, 0.1), 100)|
+-------------------------------------------------+
|                                        [1, 1, 0]|
+-------------------------------------------------+

SELECT approx_percentile(col, 0.5, 100) FROM VALUES (0), (6), (7), (9), (10) AS tab(col);
+------------------------------------------------+
|approx_percentile(col, CAST(0.5 AS DOUBLE), 100)|
+------------------------------------------------+
|                                               7|
+------------------------------------------------+
```

```sql
（4）avg(expr)：计算expr的平均值
eg:
SELECT avg(col) FROM VALUES (1), (2), (3) AS tab(col);
+--------+
|avg(col)|
+--------+
|     2.0|
+--------+

SELECT avg(col) FROM VALUES (1), (2), (NULL) AS tab(col);
+--------+
|avg(col)|
+--------+
|     1.5|
+--------+
```

```   sql
（5）bit_and(expr)：将所有非空值按位AND，如果没有返回null
eg:
SELECT bit_and(col) FROM VALUES (3), (5) AS tab(col);
+------------+
|bit_and(col)|
+------------+
|           1|
+------------+

① 先转成二进制数
3:011
5:101
② 按位AND（都为1则1，否则为0）
011
101
-----
001
③ 将二进制数转为十进制
001:1
```

```sql
（6）bit_or(col)：将所有非空值按位取OR，如果没有返回null
eg:
SELECT bit_or(col) FROM VALUES (3), (5) AS tab(col);
+-----------+
|bit_or(col)|
+-----------+
|          7|
+-----------+

① 十进制数转成二进制
3:011
5:101
② 按位OR（有1为1，无1为0）
011
101
-----
111
③ 将二进制数转为十进制
111:7
```

```
（7）bit_xor(col)：将所有非空值按位取XOR，如果没有返回null
eg:
SELECT bit_xor(col) FROM VALUES (3), (5) AS tab(col);
+------------+
|bit_xor(col)|
+------------+
|           6|
+------------+

① 十进制数转成二进制
3:011
5:101
② 按位XOR（相同为0，否则为1）
011
101
-----
110
③ 将二进制数转为十进制
110:6
```

```sql
（8）bool_and(expr)：如果expr都为true则为true，否则false（忽略null）
eg:
SELECT bool_and(col) FROM VALUES (true), (true), (true) AS tab(col);
+-------------+
|bool_and(col)|
+-------------+
|         true|
+-------------+

SELECT bool_and(col) FROM VALUES (NULL), (true), (true) AS tab(col);
+-------------+
|bool_and(col)|
+-------------+
|         true|
+-------------+

SELECT bool_and(col) FROM VALUES (true), (false), (true) AS tab(col);
+-------------+
|bool_and(col)|
+-------------+
|        false|
+-------------+
```

```sql
（9）bool_or(expr)：如果expr至少一个值为true，则返回true
eg:
SELECT bool_or(col) FROM VALUES (true), (false), (false) AS tab(col);
+------------+
|bool_or(col)|
+------------+
|        true|
+------------+

SELECT bool_or(col) FROM VALUES (NULL), (true), (false) AS tab(col);
+------------+
|bool_or(col)|
+------------+
|        true|
+------------+

SELECT bool_or(col) FROM VALUES (false), (false), (NULL) AS tab(col);
+------------+
|bool_or(col)|
+------------+
|       false|
+------------+
```

```sql
（10）collect_list(expr)：收集返回非唯一元素列表
eg:
SELECT collect_list(col) FROM VALUES (1), (2), (1) AS tab(col);
+-----------------+
|collect_list(col)|
+-----------------+
|        [1, 2, 1]|
+-----------------+
```

```sql
（11）collect_set(expr)：收集返回一组唯一元素
eg:
SELECT collect_set(col) FROM VALUES (1), (2), (1) AS tab(col);
+----------------+
|collect_set(col)|
+----------------+
|          [1, 2]|
+----------------+
```

```sql
（12）count(*)：返回检索到的行总数，包含null
eg:
SELECT count(*) FROM VALUES (NULL), (5), (5), (20) AS tab(col);
+--------+
|count(1)|
+--------+
|       4|
+--------+
```

```sql
（13）count(expr[, expr...])：返回表达式全部非空的行数
eg:
SELECT count(col) FROM VALUES (NULL), (5), (5), (20) AS tab(col);
+----------+
|count(col)|
+----------+
|         3|
+----------+
```

```sql
（14）count(DISTINCT expr[, expr...])：返回表达式唯一且非空的行数
eg:
SELECT count(DISTINCT col) FROM VALUES (NULL), (5), (5), (10) AS tab(col);
+-------------------+
|count(DISTINCT col)|
+-------------------+
|                  2|
+-------------------+
```

```sql
（15）count_if(expr)：返回表达式为true的值的数量
eg:
SELECT count_if(col % 2 = 0) FROM VALUES (NULL), (0), (1), (2), (3) AS tab(col);
+-------------------------+
|count_if(((col % 2) = 0))|
+-------------------------+
|                        2|
+-------------------------+

SELECT count_if(col IS NULL) FROM VALUES (NULL), (0), (1), (2), (3) AS tab(col);
+-----------------------+
|count_if((col IS NULL))|
+-----------------------+
|                      1|
+-----------------------+
```

```sql
（16）every(expr)：如果expr所有值都为true，则返回true（忽略null）
eg:
SELECT every(col) FROM VALUES (true), (true), (true) AS tab(col);
+----------+
|every(col)|
+----------+
|      true|
+----------+

SELECT every(col) FROM VALUES (NULL), (true), (true) AS tab(col);
+----------+
|every(col)|
+----------+
|      true|
+----------+

SELECT every(col) FROM VALUES (true), (false), (true) AS tab(col);
+----------+
|every(col)|
+----------+
|     false|
+----------+
```

```sql
（17）first(expr[, isIgnoreNull])/first_value(expr[, isIgnoreNull])：返回一组行的expr的第一个值，如果isIgnoreFull为true，则只返回非空值
eg:
SELECT first(col) FROM VALUES (10), (5), (20) AS tab(col);
+----------+
|first(col)|
+----------+
|        10|
+----------+

SELECT first(col) FROM VALUES (NULL), (5), (20) AS tab(col);
+----------+
|first(col)|
+----------+
|      null|
+----------+

SELECT first(col, true) FROM VALUES (NULL), (5), (20) AS tab(col);
+----------+
|first(col)|
+----------+
|         5|
+----------+

SELECT first_value(col) FROM VALUES (10), (5), (20) AS tab(col);
+----------------+
|first_value(col)|
+----------------+
|              10|
+----------------+

SELECT first_value(col) FROM VALUES (NULL), (5), (20) AS tab(col);
+----------------+
|first_value(col)|
+----------------+
|            null|
+----------------+

SELECT first_value(col, true) FROM VALUES (NULL), (5), (20) AS tab(col);
+----------------+
|first_value(col)|
+----------------+
|               5|
+----------------+
```

```sql
（18）last(expr[, isIgnoreNull])/last_value(expr[, isIgnoreNull])：返回一组expr的最后一个值，如果isIgnoreFull为true，则只返回非空值
eg:
SELECT last(col) FROM VALUES (10), (5), (20) AS tab(col);
+---------+
|last(col)|
+---------+
|       20|
+---------+

SELECT last(col) FROM VALUES (10), (5), (NULL) AS tab(col);
+---------+
|last(col)|
+---------+
|     null|
+---------+

SELECT last(col, true) FROM VALUES (10), (5), (NULL) AS tab(col);
+---------+
|last(col)|
+---------+
|        5|
+---------+

SELECT last_value(col) FROM VALUES (10), (5), (20) AS tab(col);
+---------------+
|last_value(col)|
+---------------+
|             20|
+---------------+

SELECT last_value(col) FROM VALUES (10), (5), (NULL) AS tab(col);
+---------------+
|last_value(col)|
+---------------+
|           null|
+---------------+

SELECT last_value(col, true) FROM VALUES (10), (5), (NULL) AS tab(col);
+---------------+
|last_value(col)|
+---------------+
|              5|
+---------------+
```

```sql
（19）max(expr)：返回expr的最大值
eg:
SELECT max(col) FROM VALUES (10), (50), (20) AS tab(col);
+--------+
|max(col)|
+--------+
|      50|
+--------+
```

```sql
（20）max_by(x, y)：返回与最大值y关联的x值
eg:
SELECT max_by(x, y) FROM VALUES (('a', 10)), (('b', 50)), (('c', 20)) AS tab(x, y);
+------------+
|max_by(x, y)|
+------------+
|           b|
+------------+
```

```sql
（21）mean(expr)：根据组值计算的平均值
eg:
SELECT mean(col) FROM VALUES (1), (2), (3) AS tab(col);
+---------+
|mean(col)|
+---------+
|      2.0|
+---------+

SELECT mean(col) FROM VALUES (1), (2), (NULL) AS tab(col);
+---------+
|mean(col)|
+---------+
|      1.5|
+---------+
```

```sql
（22）min(expr)：返回expr的最小值
eg:
SELECT min(col) FROM VALUES (10), (-1), (20) AS tab(col);
+--------+
|min(col)|
+--------+
|      -1|
+--------+
```

```sql
（23）min_by(x, y)：返回与最小值y关联的x值
eg:
SELECT min_by(x, y) FROM VALUES (('a', 10)), (('b', 50)), (('c', 20)) AS tab(x, y);
+------------+
|min_by(x, y)|
+------------+
|           a|
+------------+
```

```sql
（24）some(expr)：如果expr至少一个值为true则返回true
eg：
SELECT some(col) FROM VALUES (true), (false), (false) AS tab(col);
+---------+
|some(col)|
+---------+
|     true|
+---------+

SELECT some(col) FROM VALUES (NULL), (true), (false) AS tab(col);
+---------+
|some(col)|
+---------+
|     true|
+---------+

SELECT some(col) FROM VALUES (false), (false), (NULL) AS tab(col);
+---------+
|some(col)|
+---------+
|    false|
+---------+
```

```sql
（25）sum(expr)：根据组的值计算的总和
eg:
SELECT sum(col) FROM VALUES (5), (10), (15) AS tab(col);
+--------+
|sum(col)|
+--------+
|      30|
+--------+

SELECT sum(col) FROM VALUES (NULL), (10), (15) AS tab(col);
+--------+
|sum(col)|
+--------+
|      25|
+--------+

SELECT sum(col) FROM VALUES (NULL), (NULL) AS tab(col);
+--------+
|sum(col)|
+--------+
|    null|
+--------+
```

### 1.2、窗口函数

```sql
（1）cume_dist()：计算一个值相对于分区中所有值的位置
eg:
SELECT a, b, cume_dist() OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+--------------------------------------------------------------------------------------------------------------+
|  a|  b|cume_dist() OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+--------------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                            0.6666666666666666|
| A1|  1|                                                                                            0.6666666666666666|
| A1|  2|                                                                                                           1.0|
| A2|  3|                                                                                                           1.0|
+---+---+--------------------------------------------------------------------------------------------------------------+
```

```sql
（2）dense_rank()：计算一组值中某个值的排名，有重复值，但不会产生间隔（1,1,2,3,3,4）
eg:
SELECT a, b, dense_rank(b) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+--------------------------------------------------------------------------------------------------------------+
|  a|  b|DENSE_RANK() OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+--------------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                             1|
| A1|  1|                                                                                                             1|
| A1|  2|                                                                                                             2|
| A2|  3|                                                                                                             1|
+---+---+--------------------------------------------------------------------------------------------------------------+
```

```sql
（3）rank()：计算一组值中某个值的排名，有重复值，会产生间隔（1,1,3,4,4,6）
eg:
SELECT a, b, rank(b) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+--------------------------------------------------------------------------------------------------------+
|  a|  b|RANK() OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+--------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                       1|
| A1|  1|                                                                                                       1|
| A1|  2|                                                                                                       3|
| A2|  3|                                                                                                       1|
+---+---+--------------------------------------------------------------------------------------------------------+
```

```sql
（4）row_number()：计算一组值中某个值的排名，无重复值，不产生间隔（1,2,3,4,5,6）
eg:
SELECT a, b, row_number() OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+--------------------------------------------------------------------------------------------------------------+
|  a|  b|row_number() OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+--------------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                             1|
| A1|  1|                                                                                                             2|
| A1|  2|                                                                                                             3|
| A2|  3|                                                                                                             1|
+---+---+--------------------------------------------------------------------------------------------------------------+
```

```sql
（5）lag(input[, offset[, default]])：返回窗口中当前行向前偏移offset行的input值，当向前不足offset行时，填充default值
eg:
SELECT a, b, lag(b) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+-----------------------------------------------------------------------------------------------------------+
|  a|  b|lag(b, 1, NULL) OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN -1 FOLLOWING AND -1 FOLLOWING)|
+---+---+-----------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                       NULL|
| A1|  1|                                                                                                          1|
| A1|  2|                                                                                                          1|
| A2|  3|                                                                                                       NULL|
+---+---+-----------------------------------------------------------------------------------------------------------+
```

```sql
（6）lead(input[, offset[, default]])：返回窗口中当前行向后偏移offset行的input值，当向后不足offset行时，填充default值
eg:
SELECT a, b, lead(b) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+----------------------------------------------------------------------------------------------------------+
|  a|  b|lead(b, 1, NULL) OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING)|
+---+---+----------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                         1|
| A1|  1|                                                                                                         2|
| A1|  2|                                                                                                      NULL|
| A2|  3|                                                                                                      NULL|
+---+---+----------------------------------------------------------------------------------------------------------+
```

```sql
（7）nth_value(input[, offset])：返回窗口中offset行的input值
eg:
SELECT a, b, nth_value(b, 2) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+------------------------------------------------------------------------------------------------------------------+
|  a|  b|nth_value(b, 2) OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+------------------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                                 1|
| A1|  1|                                                                                                                 1|
| A1|  2|                                                                                                                 1|
| A2|  3|                                                                                                              NULL|
+---+---+------------------------------------------------------------------------------------------------------------------+
```

```sql
（8）ntile(n)：将窗口中的数据按照顺序切分为n片，返回当前切片值
eg:
SELECT a, b, ntile(2) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+----------------------------------------------------------------------------------------------------------+
|  a|  b|ntile(2) OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+----------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                         1|
| A1|  1|                                                                                                         1|
| A1|  2|                                                                                                         2|
| A2|  3|                                                                                                         1|
+---+---+----------------------------------------------------------------------------------------------------------+
```

```sql
（9）percent_rank()：计算窗口中当前值的百分比排名
eg：
SELECT a, b, percent_rank(b) OVER (PARTITION BY a ORDER BY b) FROM VALUES ('A1', 2), ('A1', 1), ('A2', 3), ('A1', 1) tab(a, b);
+---+---+----------------------------------------------------------------------------------------------------------------+
|  a|  b|PERCENT_RANK() OVER (PARTITION BY a ORDER BY b ASC NULLS FIRST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)|
+---+---+----------------------------------------------------------------------------------------------------------------+
| A1|  1|                                                                                                             0.0|
| A1|  1|                                                                                                             0.0|
| A1|  2|                                                                                                             1.0|
| A2|  3|                                                                                                             0.0|
+---+---+----------------------------------------------------------------------------------------------------------------+
```

### 1.3、数组函数

```sql
（1）array_contains(array, value)：如果数组array中包含value值，返回true
eg：
SELECT array_contains(array(1, 2, 3), 2);
+---------------------------------+
|array_contains(array(1, 2, 3), 2)|
+---------------------------------+
|                             true|
+---------------------------------+
```

```
（2）array_distinct(array)：删除数组中的重复值
eg:
SELECT array_distinct(array(1, 2, 3, null, 3));
+----------------------------------------------------+
|array_distinct(array(1, 2, 3, CAST(NULL AS INT), 3))|
+----------------------------------------------------+
|                                     [1, 2, 3, null]|
+----------------------------------------------------+
```

```sql
（3）array_except(array1, array2)：返回数组array1中不包含数组array2中值的值，且不包含重复值
eg:
SELECT array_except(array(1, 2, 3), array(1, 3, 5));
+--------------------------------------------+
|array_except(array(1, 2, 3), array(1, 3, 5))|
+--------------------------------------------+
|                                         [2]|
+--------------------------------------------+
```

```sql
（4）array_intersect(array1, array2)：返回数组array1和数组array2的交集，不包含重复值
eg:
SELECT array_intersect(array(1, 2, 3), array(1, 3, 5));
+-----------------------------------------------+
|array_intersect(array(1, 2, 3), array(1, 3, 5))|
+-----------------------------------------------+
|                                         [1, 3]|
+-----------------------------------------------+
```

```sql
（5）array_join(array, delimiter[, nullReplacement])：使用分隔符delimiter将数组array中的元素连接起来，数组中的null值使用nullReplacement替换，如果没有指定nullReplacement，则忽略数组中的null值
eg:
SELECT array_join(array('hello', 'world'), ' ');
+----------------------------------+
|array_join(array(hello, world),  )|
+----------------------------------+
|                       hello world|
+----------------------------------+

SELECT array_join(array('hello', null ,'world'), ' ');
+--------------------------------------------------------+
|array_join(array(hello, CAST(NULL AS STRING), world),  )|
+--------------------------------------------------------+
|                                             hello world|
+--------------------------------------------------------+

SELECT array_join(array('hello', null ,'world'), ' ', ',');
+-----------------------------------------------------------+
|array_join(array(hello, CAST(NULL AS STRING), world),  , ,)|
+-----------------------------------------------------------+
|                                              hello , world|
+-----------------------------------------------------------+
```

```
（6）array_max(array)：返回数组中的最大值（忽略null）
eg:
SELECT array_max(array(1, 20, null, 3));
+---------------------------------------------+
|array_max(array(1, 20, CAST(NULL AS INT), 3))|
+---------------------------------------------+
|                                           20|
+---------------------------------------------+
```

```sql
（7）array_min(array)：返回数组中的最小值（忽略null）
eg:
SELECT array_min(array(1, 20, null, 3));
+---------------------------------------------+
|array_min(array(1, 20, CAST(NULL AS INT), 3))|
+---------------------------------------------+
|                                            1|
+---------------------------------------------+
```

```sql
（8）array_position(array, element)：返回元素element在数组array中第一次出现的位置（位置从1开始计数）
eg:
SELECT array_position(array(3, 2, 1), 1);
+---------------------------------+
|array_position(array(3, 2, 1), 1)|
+---------------------------------+
|                                3|
+---------------------------------+
```

```sql
（9）array_remove(array, element)：从数组中删除所有值等于element的元素
eg:
SELECT array_remove(array(1, 2, 3, null, 3), 3);
+-----------------------------------------------------+
|array_remove(array(1, 2, 3, CAST(NULL AS INT), 3), 3)|
+-----------------------------------------------------+
|                                         [1, 2, null]|
+-----------------------------------------------------+
```

```sql
（10）array_repeat(element, count)：返回包含count个element元素的数组
eg:
SELECT array_repeat('123', 2);
+--------------------+
|array_repeat(123, 2)|
+--------------------+
|          [123, 123]|
+--------------------+
```

```sql
（11）array_union(array1, array2)：返回数组array1和数组array2的并集，不包含重复项
eg:
SELECT array_union(array(1, 2, 3), array(1, 3, 5));
+-------------------------------------------+
|array_union(array(1, 2, 3), array(1, 3, 5))|
+-------------------------------------------+
|                               [1, 2, 3, 5]|
+-------------------------------------------+
```

```sql
（12）arrays_overlap(a1, a2)：如果数组a1中至少包含一个a2中也存在的非空元素，则返回true
eg:
SELECT arrays_overlap(array(1, 2, 3), array(3, 4, 5));
+----------------------------------------------+
|arrays_overlap(array(1, 2, 3), array(3, 4, 5))|
+----------------------------------------------+
|                                          true|
+----------------------------------------------+
```

```sql
（13）arrays_zip(a1, a2, ...)：返回一个合并的结构数组，其中第N个结果包含输入数组的所有第N个值
eg：
SELECT arrays_zip(array(1, 2, 3), array(2, 3, 4));
+------------------------------------------+
|arrays_zip(array(1, 2, 3), array(2, 3, 4))|
+------------------------------------------+
|                      [{1, 2}, {2, 3}, ...|
+------------------------------------------+
                        
SELECT arrays_zip(array(1, 2), array(2, 3), array(3, 4));
+-------------------------------------------------+
|arrays_zip(array(1, 2), array(2, 3), array(3, 4))|
+-------------------------------------------------+
|                             [{1, 2, 3}, {2, 3...|
+-------------------------------------------------+
```

```sql
（14）concat(col1, col2, ..., colN)：返回元素的拼接的结果
eg:
SELECT concat('Spark', 'SQL');
+------------------+
|concat(Spark, SQL)|
+------------------+
|          SparkSQL|
+------------------+

SELECT concat(array(1, 2, 3), array(4, 5), array(6));
+---------------------------------------------+
|concat(array(1, 2, 3), array(4, 5), array(6))|
+---------------------------------------------+
|                           [1, 2, 3, 4, 5, 6]|
+---------------------------------------------+
```

```sql
（15）flatten(arrayOfArrays)：将多维数组转换为单个数组
eg：
SELECT flatten(array(array(1, 2), array(3, 4)));
+----------------------------------------+
|flatten(array(array(1, 2), array(3, 4)))|
+----------------------------------------+
|                            [1, 2, 3, 4]|
+----------------------------------------+
```

```sql
（16）reverse(array)：反转数组
eg：
SELECT reverse('Spark SQL');
+------------------+
|reverse(Spark SQL)|
+------------------+
|         LQS krapS|
+------------------+

SELECT reverse(array(2, 1, 4, 3));
+--------------------------+
|reverse(array(2, 1, 4, 3))|
+--------------------------+
|              [3, 4, 1, 2]|
+--------------------------+
```

```sql
（17）sequence(start, stop, step)：生成从start到stop，步长为step的数组（步长默认为1）
eg：
SELECT sequence(1, 5);
+---------------+
| sequence(1, 5)|
+---------------+
|[1, 2, 3, 4, 5]|
+---------------+

SELECT sequence(5, 1);
+---------------+
| sequence(5, 1)|
+---------------+
|[5, 4, 3, 2, 1]|
+---------------+

SELECT sequence(to_date('2018-01-01'), to_date('2018-03-01'), interval 1 month);
+-----------------------------------------------------------------------+
|sequence(to_date(2018-01-01), to_date(2018-03-01), INTERVAL '1 months')|
+-----------------------------------------------------------------------+
|                                                   [2018-01-01, 2018...|
+-----------------------------------------------------------------------+
```

```sql
（18）shuffle(array)：返回给定数组的随机排序
eg：
SELECT shuffle(array(1, 20, 3, 5));
+---------------------------+
|shuffle(array(1, 20, 3, 5))|
+---------------------------+
|              [1, 5, 3, 20]|
+---------------------------+

SELECT shuffle(array(1, 20, null, 3));
+-------------------------------------------+
|shuffle(array(1, 20, CAST(NULL AS INT), 3))|
+-------------------------------------------+
|                           [1, 20, null, 3]|
+-------------------------------------------+
```

```sql
（19）slice(x, start, length)：从数组x的start位置开始，截取个数为length的数组元素形成新的数组。数组索引从1开始，若start为负数，则从末尾开始
eg：
SELECT slice(array(1, 2, 3, 4), 2, 2);
+------------------------------+
|slice(array(1, 2, 3, 4), 2, 2)|
+------------------------------+
|                        [2, 3]|
+------------------------------+

SELECT slice(array(1, 2, 3, 4), -2, 2);
+-------------------------------+
|slice(array(1, 2, 3, 4), -2, 2)|
+-------------------------------+
|                         [3, 4]|
+-------------------------------+
```

```sql
（20）sort_array(array[, ascendingOrder])：根据数组元素的自然排序，按升序或降序对输入数组进行排序。null元素将按升序放置在数组开头，或按降序放置在数组末尾
eg:
SELECT sort_array(array('b', 'd', null, 'c', 'a'), true);
+---------------------------------------------------------+
|sort_array(array(b, d, CAST(NULL AS STRING), c, a), true)|
+---------------------------------------------------------+
|                                       [null, a, b, c, d]|
+---------------------------------------------------------+
```

```sql
（21）element_at(array,index)：返回array中下标为index的元素（下标从1开始）；
如果索引大于数组长度或者spark.sql.ansi.enabled 设置成 false，则函数返回 NULL。
如果spark.sql.ansi.enabled设置成 true，当遇到无效索引的时候会抛出异常：ArrayIndexOutOfBoundsException
eg：
SELECT element_at(array(1, 2, 3), 2);
+-----------------------------+
|element_at(array(1, 2, 3), 2)|
+-----------------------------+
|                            2|
+-----------------------------+

SELECT element_at(array(1, 2, 3), 4);
+-----------------------------+
|element_at(array(1, 2, 3), 4)|
+-----------------------------+
|                         NULL|
+-----------------------------+
```

### 1.4、map函数

```sql
（1）map_concat(map, ...)：返回所有map的并集
eg：
SELECT map_concat(map(1, 'a', 2, 'b'), map(3, 'c'));
+--------------------------------------+
|map_concat(map(1, a, 2, b), map(3, c))|
+--------------------------------------+
|                  {1 -> a, 2 -> b, ...|
+--------------------------------------+
```

```sql
（2）map_entries(map)：以无序数组的形式返回map中的所有entry
eg：
SELECT map_entries(map(1, 'a', 2, 'b'));
+----------------------------+
|map_entries(map(1, a, 2, b))|
+----------------------------+
|            [{1, a}, {2, b}]|
+----------------------------+
```

```sql
（3）map_from_entries(arrayOfEntries)：通过给定的entry创建map
eg：
SELECT map_from_entries(array(struct(1, 'a'), struct(2, 'b')));
+---------------------------------------------------+
|map_from_entries(array(struct(1, a), struct(2, b)))|
+---------------------------------------------------+
|                                   {1 -> a, 2 -> b}|
+---------------------------------------------------+
```

```sql
（4）map_from_arrays(keys, values)：通过给定的键值对数组创建map（keys 中所有的元素都不能为 null）
eg：
SELECT map_from_arrays(array(1.0, 3.0), array('2', '4'));
+---------------------------------------------+
|map_from_arrays(array(1.0, 3.0), array(2, 4))|
+---------------------------------------------+
|                  			 {1.0:"2",3.0:"4"}|
+---------------------------------------------+
```

```sql
（5）map_keys(map)：返回一个包含key的无序数组
eg：
SELECT map_keys(map(1, 'a', 2, 'b'));
+-------------------------+
|map_keys(map(1, a, 2, b))|
+-------------------------+
|                   [1, 2]|
+-------------------------+
```

```sql
（6）map_values(map)：返回一个包含values的无序数组
eg：
SELECT map_values(map(1, 'a', 2, 'b'));
+---------------------------+
|map_values(map(1, a, 2, b))|
+---------------------------+
|                     [a, b]|
+---------------------------+
```

```sql
（7）element_at(map, key)：返回map中key对应的value。如果map中不包含当前key：如果spark.sql.ansi.enabled设置成false，则返回null；如果spark.sql.ansi.enabled设置为true，则抛出异常NoSuchElementException
eg：
SELECT element_at(map(1, 'a', 2, 'b'), 2);
+------------------------------+
|element_at(map(1, a, 2, b), 2)|
+------------------------------+
|                             b|
+------------------------------+
```

```sql
（8）map(key0, value0, key1, value1, …)：创建一个包含给定key和value的键值对map
eg:
SELECT map(1.0, '2', 3.0, '4');
+--------------------+
| map(1.0, 2, 3.0, 4)|
+--------------------+
|  {1.0:"2",3.0:"4"} |
+--------------------+
```

```sql
（9）str_to_map(text[, pairDelim[, keyValueDelim]])：使用分割符将字符串切分为键值对，使用该键值对创建一个map
默认的键值对间的分割符pairDelim为',',键值对的分割符keyValueDelim为':'
eg：
SELECT str_to_map('a:1,b:2,c:3', ',', ':');
+-----------------------------+
|str_to_map(a:1,b:2,c:3, ,, :)|
+-----------------------------+
|    {"a":"1","b":"2","c":"3"}|
+-----------------------------+

SELECT str_to_map('a');
+-------------------+
|str_to_map(a, ,, :)|
+-------------------+
|        {"a":null}	|
+-------------------+
```

### 1.5、日期和时间戳函数

```sql
（1）add_months(start_date, num_months)：返回在start_date的基础上加num_months个月后的日期
eg：
SELECT add_months('2016-08-31', 1);
+---------------------------------------+
|add_months(CAST(2016-08-31 AS DATE), 1)|
+---------------------------------------+
|                             2016-09-30|
+---------------------------------------+
```

```sql
（2）current_date()：返回当前日期（10位日期）
eg：
SELECT current_date();
+--------------+
|current_date()|
+--------------+
|    2021-02-22|
+--------------+

SELECT current_date;
+--------------+
|current_date()|
+--------------+
|    2021-02-22|
+--------------+
```

```sql
（3）current_timestamp()：返回当前时间戳（含毫秒）
eg：
SELECT current_timestamp();
+--------------------+
| current_timestamp()|
+--------------------+
|2021-02-22 03:03:...|
+--------------------+

SELECT current_timestamp;
+--------------------+
| current_timestamp()|
+--------------------+
|2021-02-22 03:03:...|
+--------------------+
```

```sql
（4）current_timezone()：返回当前时区
eg：
SELECT current_timezone();
+------------------+
|current_timezone()|
+------------------+
|           Etc/UTC|
+------------------+
```

```sql
（5）date_add(start_date, num_days)：返回在start_date的基础上加num_days天后的日期
eg:
SELECT date_add('2016-07-30', 1);
+-------------------------------------+
|date_add(CAST(2016-07-30 AS DATE), 1)|
+-------------------------------------+
|                           2016-07-31|
+-------------------------------------+
```

```sql
（6）date_format(timestamp, fmt)：将timestamp转换为fmt格式的字符串值
eg：
SELECT date_format('2016-04-08', 'y');
+---------------------------------------------+
|date_format(CAST(2016-04-08 AS TIMESTAMP), y)|
+---------------------------------------------+
|                                         2016|
+---------------------------------------------+
```

```sql
（7）date_from_unix_date(days)：根据自1970-01-01以来的天数创建日期
eg：
SELECT date_from_unix_date(1);
+----------------------+
|date_from_unix_date(1)|
+----------------------+
|            1970-01-02|
+----------------------+
```

```sql
（8）date_part(field, source)：提取日期、时间戳或间隔的一部分
eg：
SELECT date_part('YEAR', TIMESTAMP '2019-08-12 01:00:00.123456');
+-------------------------------------------------------+
|date_part(YEAR, TIMESTAMP '2019-08-12 01:00:00.123456')|
+-------------------------------------------------------+
|                                                   2019|
+-------------------------------------------------------+

SELECT date_part('week', timestamp'2019-08-12 01:00:00.123456');
+-------------------------------------------------------+
|date_part(week, TIMESTAMP '2019-08-12 01:00:00.123456')|
+-------------------------------------------------------+
|                                                     33|
+-------------------------------------------------------+

SELECT date_part('doy', DATE'2019-08-12');
+---------------------------------+
|date_part(doy, DATE '2019-08-12')|
+---------------------------------+
|                              224|
+---------------------------------+

SELECT date_part('SECONDS', timestamp'2019-10-01 00:00:01.000001');
+----------------------------------------------------------+
|date_part(SECONDS, TIMESTAMP '2019-10-01 00:00:01.000001')|
+----------------------------------------------------------+
|                                                  1.000001|
+----------------------------------------------------------+

SELECT date_part('days', interval 1 year 10 months 5 days);
+----------------------------------------------------+
|date_part(days, INTERVAL '1 years 10 months 5 days')|
+----------------------------------------------------+
|                                                   5|
+----------------------------------------------------+

SELECT date_part('seconds', interval 5 hours 30 seconds 1 milliseconds 1 microseconds);
+--------------------------------------------------------+
|date_part(seconds, INTERVAL '5 hours 30.001001 seconds')|
+--------------------------------------------------------+
|                                               30.001001|
+--------------------------------------------------------+
```

```sql
（9）date_sub(start_date, num_days)：返回start_date减去num_days的日期
eg：
SELECT date_sub('2016-07-30', 1);
+-------------------------------------+
|date_sub(CAST(2016-07-30 AS DATE), 1)|
+-------------------------------------+
|                           2016-07-29|
+-------------------------------------+
```

```sql
（10）date_trunc(fmt, ts)：返回截断格式为fmt的时间戳
eg：
SELECT date_trunc('YEAR', '2015-03-05T09:32:05.359');
+------------------------------------------------------------+
|date_trunc(YEAR, CAST(2015-03-05T09:32:05.359 AS TIMESTAMP))|
+------------------------------------------------------------+
|                                         2015-01-01 00:00:00|
+------------------------------------------------------------+

SELECT date_trunc('MM', '2015-03-05T09:32:05.359');
+----------------------------------------------------------+
|date_trunc(MM, CAST(2015-03-05T09:32:05.359 AS TIMESTAMP))|
+----------------------------------------------------------+
|                                       2015-03-01 00:00:00|
+----------------------------------------------------------+

SELECT date_trunc('DD', '2015-03-05T09:32:05.359');
+----------------------------------------------------------+
|date_trunc(DD, CAST(2015-03-05T09:32:05.359 AS TIMESTAMP))|
+----------------------------------------------------------+
|                                       2015-03-05 00:00:00|
+----------------------------------------------------------+

SELECT date_trunc('HOUR', '2015-03-05T09:32:05.359');
+------------------------------------------------------------+
|date_trunc(HOUR, CAST(2015-03-05T09:32:05.359 AS TIMESTAMP))|
+------------------------------------------------------------+
|                                         2015-03-05 09:00:00|
+------------------------------------------------------------+

SELECT date_trunc('MILLISECOND', '2015-03-05T09:32:05.123456');
+----------------------------------------------------------------------+
|date_trunc(MILLISECOND, CAST(2015-03-05T09:32:05.123456 AS TIMESTAMP))|
+----------------------------------------------------------------------+
|                                                  2015-03-05 09:32:...|
+----------------------------------------------------------------------+
```

```sql
（11）datediff(endDate, startDate)：返回endDate减去startDate的天数
eg：
SELECT datediff('2009-07-31', '2009-07-30');
+------------------------------------------------------------+
|datediff(CAST(2009-07-31 AS DATE), CAST(2009-07-30 AS DATE))|
+------------------------------------------------------------+
|                                                           1|
+------------------------------------------------------------+

SELECT datediff('2009-07-30', '2009-07-31');
+------------------------------------------------------------+
|datediff(CAST(2009-07-30 AS DATE), CAST(2009-07-31 AS DATE))|
+------------------------------------------------------------+
|                                                          -1|
+------------------------------------------------------------+
```

```sql
（12）dayofweek(date)：返回日期date是一周中的第几天
eg：
SELECT dayofweek('2009-07-30');
+-----------------------------------+
|dayofweek(CAST(2009-07-30 AS DATE))|
+-----------------------------------+
|                                  5|
+-----------------------------------+
```

```sql
（13）dayofyear(date)：返回日期date是一年中的第几天
eg：
SELECT dayofyear('2016-04-09');
+-----------------------------------+
|dayofyear(CAST(2016-04-09 AS DATE))|
+-----------------------------------+
|                                100|
+-----------------------------------+
```

```sql
（14）from_unixtime(unix_time[, fmt])：将unix时间戳转换为fmt格式的日期
eg：
SELECT from_unixtime(0, 'yyyy-MM-dd HH:mm:ss');
+-----------------------------------------------------+
|from_unixtime(CAST(0 AS BIGINT), yyyy-MM-dd HH:mm:ss)|
+-----------------------------------------------------+
|                                  1970-01-01 00:00:00|
+-----------------------------------------------------+

SELECT from_unixtime(0);
+-----------------------------------------------------+
|from_unixtime(CAST(0 AS BIGINT), yyyy-MM-dd HH:mm:ss)|
+-----------------------------------------------------+
|                                  1970-01-01 00:00:00|
+-----------------------------------------------------+
```

```sql
（15）from_utc_timestamp(timestamp, timezone)：将日期转换为utc时间戳
eg：
SELECT to_utc_timestamp('2016-08-31', 'Asia/Seoul');
+-----------------------------------------------------------+
|to_utc_timestamp(CAST(2016-08-31 AS TIMESTAMP), Asia/Seoul)|
+-----------------------------------------------------------+
|                                        2016-08-30 15:00:00|
+-----------------------------------------------------------+
```

```sql
（16）hour(timestamp)：返回日期中的小时
eg:
SELECT hour('2009-07-30 12:58:59');
+--------------------------------------------+
|hour(CAST(2009-07-30 12:58:59 AS TIMESTAMP))|
+--------------------------------------------+
|                                          12|
+--------------------------------------------+
```

```sql
（17）last_day(date)：返回日期所属月份的最后一天
eg：
SELECT last_day('2009-01-12');
+----------------------------------+
|last_day(CAST(2009-01-12 AS DATE))|
+----------------------------------+
|                        2009-01-31|
+----------------------------------+
```

```sql
（18）make_date(year, month, day)：使用给定的年、月、日创建日期
eg：
SELECT make_date(2013, 7, 15);
+----------------------+
|make_date(2013, 7, 15)|
+----------------------+
|            2013-07-15|
+----------------------+

SELECT make_date(2019, 13, 1);
+----------------------+
|make_date(2019, 13, 1)|
+----------------------+
|                  null|
+----------------------+

SELECT make_date(2019, 7, NULL);
+-------------------------------------+
|make_date(2019, 7, CAST(NULL AS INT))|
+-------------------------------------+
|                                 null|
+-------------------------------------+

SELECT make_date(2019, 2, 30);
+----------------------+
|make_date(2019, 2, 30)|
+----------------------+
|                  null|
+----------------------+
```

```sql
（19）make_timestamp(year, month, day, hour, min, sec[, timezone])：从年、月、日、时、分、秒和时区创建时间戳
eg：
SELECT make_timestamp(2014, 12, 28, 6, 30, 45.887);
+-----------------------------------------------------------------+
|make_timestamp(2014, 12, 28, 6, 30, CAST(45.887 AS DECIMAL(8,6)))|
+-----------------------------------------------------------------+
|                                             2014-12-28 06:30:...|
+-----------------------------------------------------------------+

SELECT make_timestamp(2014, 12, 28, 6, 30, 45.887, 'CET');
+----------------------------------------------------------------------+
|make_timestamp(2014, 12, 28, 6, 30, CAST(45.887 AS DECIMAL(8,6)), CET)|
+----------------------------------------------------------------------+
|                                                  2014-12-28 05:30:...|
+----------------------------------------------------------------------+

SELECT make_timestamp(2019, 6, 30, 23, 59, 60);
+-------------------------------------------------------------+
|make_timestamp(2019, 6, 30, 23, 59, CAST(60 AS DECIMAL(8,6)))|
+-------------------------------------------------------------+
|                                          2019-07-01 00:00:00|
+-------------------------------------------------------------+

SELECT make_timestamp(2019, 13, 1, 10, 11, 12, 'PST');
+------------------------------------------------------------------+
|make_timestamp(2019, 13, 1, 10, 11, CAST(12 AS DECIMAL(8,6)), PST)|
+------------------------------------------------------------------+
|                                                              null|
+------------------------------------------------------------------+

SELECT make_timestamp(null, 7, 22, 15, 30, 0);
+-------------------------------------------------------------------------+
|make_timestamp(CAST(NULL AS INT), 7, 22, 15, 30, CAST(0 AS DECIMAL(8,6)))|
+-------------------------------------------------------------------------+
|                                                                     null|
+--
```

```sql
（20）minute(timestamp)：返回日期中的分钟
eg：
SELECT minute('2009-07-30 12:58:59');
+----------------------------------------------+
|minute(CAST(2009-07-30 12:58:59 AS TIMESTAMP))|
+----------------------------------------------+
|                                            58|
+----------------------------------------------+
```

```sql
（21）month(date)：返回日期中的月份
eg：
+-------------------------------+
|month(CAST(2016-07-30 AS DATE))|
+-------------------------------+
|                              7|
+-------------------------------+
```

```sql
（22）months_between(timestamp1, timestamp2[, roundOff])：

eg：
SELECT months_between('1997-02-28 10:30:00', '1996-10-30');
+-------------------------------------------------------------------------------------------+
|months_between(CAST(1997-02-28 10:30:00 AS TIMESTAMP), CAST(1996-10-30 AS TIMESTAMP), true)|
+-------------------------------------------------------------------------------------------+
|                                                                                 3.94959677|
+-------------------------------------------------------------------------------------------+

SELECT months_between('1997-02-28 10:30:00', '1996-10-30', false);
+--------------------------------------------------------------------------------------------+
|months_between(CAST(1997-02-28 10:30:00 AS TIMESTAMP), CAST(1996-10-30 AS TIMESTAMP), false)|
+--------------------------------------------------------------------------------------------+
|                                                                          3.9495967741935485|
+--------------------------------------------------------------------------------------------+
```

```sql
（23）next_day(start_date, day_of_week)：

eg：
SELECT next_day('2015-01-14', 'TU');
+--------------------------------------+
|next_day(CAST(2015-01-14 AS DATE), TU)|
+--------------------------------------+
|                            2015-01-20|
+--------------------------------------+
```

```sql
（24）now()：返回当前时间戳
eg：
+--------------------+
|               now()|
+--------------------+
|2021-02-22 03:03:...|
+--------------------+
```

```sql
（25）quarter(date)：返回日期的季度，范围在1-4之间
eg：
SELECT quarter('2016-08-31');
+---------------------------------+
|quarter(CAST(2016-08-31 AS DATE))|
+---------------------------------+
|                                3|
+---------------------------------+
```

```
（26）second(timestamp)：返回日期中的秒
eg：
SELECT second('2009-07-30 12:58:59');
+----------------------------------------------+
|second(CAST(2009-07-30 12:58:59 AS TIMESTAMP))|
+----------------------------------------------+
|                                            59|
+----------------------------------------------+
```

```sql
（27）to_date(date_str[, fmt])：将字符串date_str和fmt解析为日期
eg：
SELECT to_date('2009-07-30 04:17:52');
+----------------------------+
|to_date(2009-07-30 04:17:52)|
+----------------------------+
|                  2009-07-30|
+----------------------------+

SELECT to_date('2016-12-31', 'yyyy-MM-dd');
+-------------------------------+
|to_date(2016-12-31, yyyy-MM-dd)|
+-------------------------------+
|                     2016-12-31|
+-------------------------------+
```

```sql
（28）to_timestamp(timestamp_str[, fmt])：将字符串timestamp_str和fmt解析为时间戳
eg：
SELECT to_timestamp('2016-12-31 00:12:00');
+---------------------------------+
|to_timestamp(2016-12-31 00:12:00)|
+---------------------------------+
|              2016-12-31 00:12:00|
+---------------------------------+

SELECT to_timestamp('2016-12-31', 'yyyy-MM-dd');
+------------------------------------+
|to_timestamp(2016-12-31, yyyy-MM-dd)|
+------------------------------------+
|                 2016-12-31 00:00:00|
+------------------------------------+
```

```sql
（29）to_unix_timestamp(timeExp[, fmt])：将指定时间转换为unix时间戳
eg：
SELECT to_unix_timestamp('2016-04-08', 'yyyy-MM-dd');
+-----------------------------------------+
|to_unix_timestamp(2016-04-08, yyyy-MM-dd)|
+-----------------------------------------+
|                               1460073600|
+-----------------------------------------+
```

```sql
（30）to_utc_timestamp(timestamp, timezone)：将指定时间转换为utc时间戳
eg：
SELECT to_utc_timestamp('2016-08-31', 'Asia/Seoul');
+-----------------------------------------------------------+
|to_utc_timestamp(CAST(2016-08-31 AS TIMESTAMP), Asia/Seoul)|
+-----------------------------------------------------------+
|                                        2016-08-30 15:00:00|
+-----------------------------------------------------------+
```

```sql
（31）trunc(date, fmt)：返回date截断为格式fmt的日期
eg：
SELECT trunc('2019-08-04', 'week');
+-------------------------------------+
|trunc(CAST(2019-08-04 AS DATE), week)|
+-------------------------------------+
|                           2019-07-29|
+-------------------------------------+

SELECT trunc('2019-08-04', 'quarter');
+----------------------------------------+
|trunc(CAST(2019-08-04 AS DATE), quarter)|
+----------------------------------------+
|                              2019-07-01|
+----------------------------------------+

SELECT trunc('2009-02-12', 'MM');
+-----------------------------------+
|trunc(CAST(2009-02-12 AS DATE), MM)|
+-----------------------------------+
|                         2009-02-01|
+-----------------------------------+

SELECT trunc('2015-10-27', 'YEAR');
+-------------------------------------+
|trunc(CAST(2015-10-27 AS DATE), YEAR)|
+-------------------------------------+
|                           2015-01-01|
+-------------------------------------+
```

```sql
（32）unix_date(date)：返回自1970-01-01以来过了几天
eg：
SELECT unix_date(DATE("1970-01-02"));
+-----------------------------------+
|unix_date(CAST(1970-01-02 AS DATE))|
+-----------------------------------+
|                                  1|
+-----------------------------------+
```

```sql
（33）unix_seconds(timestamp)：返回自1970-01-01 00:00:00 utc以来的秒数
eg：
SELECT unix_seconds(TIMESTAMP('1970-01-01 00:00:01Z'));
+-----------------------------------------------------+
|unix_seconds(CAST(1970-01-01 00:00:01Z AS TIMESTAMP))|
+-----------------------------------------------------+
|                                                    1|
+-----------------------------------------------------+
```

```sql
（34）unix_timestamp([timeExp[, fmt]])：返回当前指定时间的unix时间戳
eg：
SELECT unix_timestamp();
+--------------------------------------------------------+
|unix_timestamp(current_timestamp(), yyyy-MM-dd HH:mm:ss)|
+--------------------------------------------------------+
|                                              1613962997|
+--------------------------------------------------------+

SELECT unix_timestamp('2016-04-08', 'yyyy-MM-dd');
+--------------------------------------+
|unix_timestamp(2016-04-08, yyyy-MM-dd)|
+--------------------------------------+
|                            1460073600|
+--------------------------------------+
```

```sql
（35）weekday(date)：返回日期所在月份第几周
eg：
SELECT weekday('2009-07-30');
+---------------------------------+
|weekday(CAST(2009-07-30 AS DATE))|
+---------------------------------+
|                                3|
+---------------------------------+
```

```sql
（36）weekofyear(date)：返回日期是所在年份的第几周
eg:
SELECT weekofyear('2008-02-20');
+------------------------------------+
|weekofyear(CAST(2008-02-20 AS DATE))|
+------------------------------------+
|                                   8|
+------------------------------------+
```

```sql
（37）year(date)：返回日期所属年份
eg：
SELECT year('2016-07-30');
+------------------------------+
|year(CAST(2016-07-30 AS DATE))|
+------------------------------+
|                          2016|
+------------------------------+
```

### 1.5、json函数

```sql
（1）from_json(jsonStr, schema[, options])：根据给定的JSONstr创建JSON
eg：
SELECT from_json('{"a":1, "b":0.8}', 'a INT, b DOUBLE');
+---------------------------+
|from_json({"a":1, "b":0.8})|
+---------------------------+
|                   {1, 0.8}|
+---------------------------+

SELECT from_json('{"time":"26/08/2015"}', 'time Timestamp', map('timestampFormat', 'dd/MM/yyyy'));
+--------------------------------+
|from_json({"time":"26/08/2015"})|
+--------------------------------+
|            {2015-08-26 00:00...|
+--------------------------------+
```

```sql
（2）get_json_object(json_txt, path)：从path中提取json_txt对象（path中使用$表示json变量标识，用 . 或 [] 读取对象或数组）
eg：
SELECT get_json_object('{"a":"b"}', '$.a');
+-------------------------------+
|get_json_object({"a":"b"}, $.a)|
+-------------------------------+
|                              b|
+-------------------------------+

SELECT  get_json_object('{
 "store":
        {
         "fruit":[{"weight":8,"type":"apple"}, {"weight":9,"type":"pear"}],  
         "bicycle":{"price":19.95,"color":"red"}
         }, 
 "email":"amy@only_for_json_udf_test.net", 
 "owner":"amy" 
}', '$.store.bicycle.price');
19.95

SELECT  get_json_object('{
 "store":
        {
         "fruit":[{"weight":8,"type":"apple"}, {"weight":9,"type":"pear"}],  
         "bicycle":{"price":19.95,"color":"red"}
         }, 
 "email":"amy@only_for_json_udf_test.net", 
 "owner":"amy" 
}', '$.store.fruit[0]');
{"weight":8,"type":"apple"}
```

```sql
（3）json_array_length(jsonArray)：返回最外层JSON数组的元素数
eg：
SELECT json_array_length('[1,2,3,4]');
+----------------------------+
|json_array_length([1,2,3,4])|
+----------------------------+
|                           4|
+----------------------------+

SELECT json_array_length('[1,2,3,{"f1":1,"f2":[5,6]},4]');
+------------------------------------------------+
|json_array_length([1,2,3,{"f1":1,"f2":[5,6]},4])|
+------------------------------------------------+
|                                               5|
+------------------------------------------------+

SELECT json_array_length('[1,2');
+-----------------------+
|json_array_length([1,2)|
+-----------------------+
|                   null|
+-----------------------+
```

```sql
（4）json_object_keys(json_object)：以数组形式返回最外层JSON对象的所有键
eg：
SELECT json_object_keys('{}');
+--------------------+
|json_object_keys({})|
+--------------------+
|                  []|
+--------------------+

SELECT json_object_keys('{"key": "value"}');
+----------------------------------+
|json_object_keys({"key": "value"})|
+----------------------------------+
|                             [key]|
+----------------------------------+

SELECT json_object_keys('{"f1":"abc","f2":{"f3":"a", "f4":"b"}}');
+--------------------------------------------------------+
|json_object_keys({"f1":"abc","f2":{"f3":"a", "f4":"b"}})|
+--------------------------------------------------------+
|                                                [f1, f2]|
+--------------------------------------------------------+
```

```sql
（5）json_tuple(jsonStr, p1, p2, ..., pn)：返回JSON中多个对象
eg：
SELECT json_tuple('{"a":1, "b":2}', 'a', 'b');
+---+---+
| c0| c1|
+---+---+
|  1|  2|
+---+---+
```

```sql
（6）schema_of_json(json[, options])：以JSON字符串的形式返回架构
eg：
SELECT schema_of_json('[{"col":0}]');
+---------------------------+
|schema_of_json([{"col":0}])|
+---------------------------+
|  array<struct<col:bigint>>|
+---------------------------+

SELECT schema_of_json('[{"col":01}]', map('allowNumericLeadingZeros', 'true'));
+----------------------------+
|schema_of_json([{"col":01}])|
+----------------------------+
|   array<struct<col:bigint>>|
+----------------------------+
```

```sql
（7）to_json(expr[, options])：返回具有给定结构的JSON字符串
eg：
SELECT to_json(named_struct('a', 1, 'b', 2));
+---------------------------------+
|to_json(named_struct(a, 1, b, 2))|
+---------------------------------+
|                    {"a":1,"b":2}|
+---------------------------------+

SELECT to_json(named_struct('time', to_timestamp('2015-08-26', 'yyyy-MM-dd')), map('timestampFormat', 'dd/MM/yyyy'));
+-----------------------------------------------------------------+
|to_json(named_struct(time, to_timestamp(2015-08-26, yyyy-MM-dd)))|
+-----------------------------------------------------------------+
|                                             {"time":"26/08/20...|
+-----------------------------------------------------------------+

SELECT to_json(array(named_struct('a', 1, 'b', 2)));
+----------------------------------------+
|to_json(array(named_struct(a, 1, b, 2)))|
+----------------------------------------+
|                         [{"a":1,"b":2}]|
+----------------------------------------+

SELECT to_json(map('a', named_struct('b', 1)));
+-----------------------------------+
|to_json(map(a, named_struct(b, 1)))|
+-----------------------------------+
|                      {"a":{"b":1}}|
+-----------------------------------+

SELECT to_json(map(named_struct('a', 1),named_struct('b', 2)));
+----------------------------------------------------+
|to_json(map(named_struct(a, 1), named_struct(b, 2)))|
+----------------------------------------------------+
|                                     {"[1]":{"b":2}}|
+----------------------------------------------------+

SELECT to_json(map('a', 1));
+------------------+
|to_json(map(a, 1))|
+------------------+
|           {"a":1}|
+------------------+

SELECT to_json(array((map('a', 1))));
+-------------------------+
|to_json(array(map(a, 1)))|
+-------------------------+
|                [{"a":1}]|
+-------------------------+
```

### 1.6、字符串函数

```sql
（1）btrim(str)/trim(str)：删除'str'中开头和结尾的空字符串
eg:
SELECT btrim('    SparkSQL   ');
+----------------------+
|btrim(    SparkSQL   )|
+----------------------+
|              SparkSQL|
+----------------------+

SELECT btrim(encode('    SparkSQL   ', 'utf-8'));
+-------------------------------------+
|btrim(encode(    SparkSQL   , utf-8))|
+-------------------------------------+
|                             SparkSQL|
+-------------------------------------+

（2）btrim(str,trimStr)：删除'str'中开头和结尾的'trimStr'字符
eg:
SELECT btrim('SSparkSQLS', 'SL');
+---------------------+
|btrim(SSparkSQLS, SL)|
+---------------------+
|               parkSQ|
+---------------------+

SELECT btrim(encode('SSparkSQLS', 'utf-8'), encode('SL', 'utf-8'));
+---------------------------------------------------+
|btrim(encode(SSparkSQLS, utf-8), encode(SL, utf-8))|
+---------------------------------------------------+
|                                             parkSQ|
+---------------------------------------------------+

（3）concat_ws(sep[,str|array(str)]+)：返回由'sep'分隔的字符串的串联，跳过空值
eg:
SELECT concat_ws(' ', 'Spark', 'SQL');
+------------------------+
|concat_ws( , Spark, SQL)|
+------------------------+
|               Spark SQL|
+------------------------+

SELECT concat_ws('s');
+------------+
|concat_ws(s)|
+------------+
|            |
+------------+

SELECT concat_ws('/', 'foo', null, 'bar');
+----------------------------+
|concat_ws(/, foo, NULL, bar)|
+----------------------------+
|                     foo/bar|
+----------------------------+

SELECT concat_ws(null, 'Spark', 'SQL');
+---------------------------+
|concat_ws(NULL, Spark, SQL)|
+---------------------------+
|                       NULL|
+---------------------------+

（4）contains(left,right)：返回一个布尔值。如果在left中找到right，则返回true；如果任意输入表达式为NULL，则返回NULL；否则，返回false。left和right都必须是String或binary类型。（区分大小写）
eg:
SELECT contains('Spark SQL', 'Spark');
+--------------------------+
|contains(Spark SQL, Spark)|
+--------------------------+
|                      true|
+--------------------------+

SELECT contains('Spark SQL', 'SPARK');
+--------------------------+
|contains(Spark SQL, SPARK)|
+--------------------------+
|                     false|
+--------------------------+

SELECT contains('Spark SQL', null);
+-------------------------+
|contains(Spark SQL, NULL)|
+-------------------------+
|                     NULL|
+-------------------------+

SELECT contains(x'537061726b2053514c', x'537061726b');
+----------------------------------------------+
|contains(X'537061726B2053514C', X'537061726B')|
+----------------------------------------------+
|                                          true|
+----------------------------------------------+

（5）decode(expr,search,result[,search,result]...[,default])：按顺序将expr与每个search进行比较，如果expr等于某个search，则decode返回相应的result。如果没有则返回default。如果省略default，则返回null。
eg：
SELECT decode(2, 1, 'Southlake', 2, 'San Francisco', 3, 'New Jersey', 4, 'Seattle', 'Non domestic');
+----------------------------------------------------------------------------------+
|decode(2, 1, Southlake, 2, San Francisco, 3, New Jersey, 4, Seattle, Non domestic)|
+----------------------------------------------------------------------------------+
|                                                                     San Francisco|
+----------------------------------------------------------------------------------+

SELECT decode(6, 1, 'Southlake', 2, 'San Francisco', 3, 'New Jersey', 4, 'Seattle', 'Non domestic');
+----------------------------------------------------------------------------------+
|decode(6, 1, Southlake, 2, San Francisco, 3, New Jersey, 4, Seattle, Non domestic)|
+----------------------------------------------------------------------------------+
|                                                                      Non domestic|
+----------------------------------------------------------------------------------+

SELECT decode(6, 1, 'Southlake', 2, 'San Francisco', 3, 'New Jersey', 4, 'Seattle');
+--------------------------------------------------------------------+
|decode(6, 1, Southlake, 2, San Francisco, 3, New Jersey, 4, Seattle)|
+--------------------------------------------------------------------+
|                                                                NULL|
+--------------------------------------------------------------------+

SELECT decode(null, 6, 'Spark', NULL, 'SQL', 4, 'rocks');
+-------------------------------------------+
|decode(NULL, 6, Spark, NULL, SQL, 4, rocks)|
+-------------------------------------------+
|                                        SQL|
+-------------------------------------------+

（6）elt(n,input1,input2,...)：返回第n个输入，当n为2时返回input2。如果索引超过数组的长度并且'spark.sql.ansi.enabled'设置为false，则该函数返回NULL。如果'spark.sql.ansi.enabled'设置为true，则对于无效索引，它会抛出异常。
eg:
SELECT elt(1, 'scala', 'java');
+-------------------+
|elt(1, scala, java)|
+-------------------+
|              scala|
+-------------------+

SELECT elt(2, 'a', 1);
+------------+
|elt(2, a, 1)|
+------------+
|           1|
+------------+

（7）endswith(left,right)：返回一个布尔值，如果left以right结尾，则值为true；如果任一输入表达式为NULL，则返回NULL；否则返回false。left和right都必须为String或binary类型。startswith(left,right)同理
eg:
SELECT endswith('Spark SQL', 'SQL');
+------------------------+
|endswith(Spark SQL, SQL)|
+------------------------+
|                    true|
+------------------------+

SELECT endswith('Spark SQL', 'Spark');
+--------------------------+
|endswith(Spark SQL, Spark)|
+--------------------------+
|                     false|
+--------------------------+

SELECT endswith('Spark SQL', null);
+-------------------------+
|endswith(Spark SQL, NULL)|
+-------------------------+
|                     NULL|
+-------------------------+

SELECT endswith(x'537061726b2053514c', x'537061726b');
+----------------------------------------------+
|endswith(X'537061726B2053514C', X'537061726B')|
+----------------------------------------------+
|                                         false|
+----------------------------------------------+

SELECT endswith(x'537061726b2053514c', x'53514c');
+------------------------------------------+
|endswith(X'537061726B2053514C', X'53514C')|
+------------------------------------------+
|                                      true|
+------------------------------------------+

（8）find_in_set(str,str_array)：返回逗号分隔符列表str_array中给定的字符串str的索引（索引从1开始）。如果未找到则返回0
eg:
SELECT find_in_set('ab','abc,b,ab,c,def');
+-------------------------------+
|find_in_set(ab, abc,b,ab,c,def)|
+-------------------------------+
|                              3|
+-------------------------------+

（9）format_number(expr1,expr2)：将数字expr1格式化为'#,###,###.##'，四舍五入到expr2位小数。如果expr2为0，则结果没有小数部分。
eg:
SELECT format_number(12332.123456, 4);
+------------------------------+
|format_number(12332.123456, 4)|
+------------------------------+
|                   12,332.1235|
+------------------------------+

SELECT format_number(12332.123456, '##################.###');
+---------------------------------------------------+
|format_number(12332.123456, ##################.###)|
+---------------------------------------------------+
|                                          12332.123|
+---------------------------------------------------+

（10）format_string(strfmt,obj,...)/printf(strfmt,obj,...)：从printf风格的格式字符串返回格式化的字符串。
eg:
SELECT format_string("Hello World %d %s", 100, "days");
+-------------------------------------------+
|format_string(Hello World %d %s, 100, days)|
+-------------------------------------------+
|                       Hello World 100 days|
+-------------------------------------------+

（11）initcap(str)：str的每个单词首字母大写，空格分隔。
eg:
SELECT initcap('sPark sql');
+------------------+
|initcap(sPark sql)|
+------------------+
|         Spark Sql|
+------------------+

（12）instr(str,substr)：返回substr在str中第一次出现的索引（索引从1开始）
eg:
SELECT instr('SparkSQL', 'SQL');
+--------------------+
|instr(SparkSQL, SQL)|
+--------------------+
|                   6|
+--------------------+

（13）lcase(str)/lower(str)：str转小写
eg:
SELECT lcase('SparkSql');
+---------------+
|lcase(SparkSql)|
+---------------+
|       sparksql|
+---------------+

（14）left(str,len)：返回字符串str中最左边的len个字符，如果len小于或等于0，则结果为空字符串。
eg:
SELECT left('Spark SQL', 3);
+------------------+
|left(Spark SQL, 3)|
+------------------+
|               Spa|
+------------------+

SELECT left(encode('Spark SQL', 'utf-8'), 3);
+---------------------------------+
|left(encode(Spark SQL, utf-8), 3)|
+---------------------------------+
|                       [53 70 61]|
+---------------------------------+

（15）len(expr)/length(expr)：返回字符串expr的字符长度或二进制数据的字节数。
eg:
SELECT len('Spark SQL ');
+---------------+
|len(Spark SQL )|
+---------------+
|             10|
+---------------+

SELECT len(x'537061726b2053514c');
+--------------------------+
|len(X'537061726B2053514C')|
+--------------------------+
|                         9|
+--------------------------+

（16）locate(substr,str[,pos])：返回substr在str中位置pos之后第一次出现的位置。（索引从1开始）。
eg:
SELECT locate('bar', 'foobarbar');
+-------------------------+
|locate(bar, foobarbar, 1)|
+-------------------------+
|                        4|
+-------------------------+

SELECT locate('bar', 'foobarbar', 5);
+-------------------------+
|locate(bar, foobarbar, 5)|
+-------------------------+
|                        7|
+-------------------------+

（17）lpad(str,len[,pad])：使用pad填充str的左侧，使str的长度达到len。如果未指定pad，并且str是字符串，则使用空格填充，如果str是字节序列，则使用0填充。rpad同理
eg:
SELECT lpad('hi', 5, '??');
+---------------+
|lpad(hi, 5, ??)|
+---------------+
|          ???hi|
+---------------+

SELECT lpad('hi', 1, '??');
+---------------+
|lpad(hi, 1, ??)|
+---------------+
|              h|
+---------------+

SELECT lpad('hi', 5);
+--------------+
|lpad(hi, 5,  )|
+--------------+
|            hi|
+--------------+

（18）ltrim(str)：删除str左侧空格字符，rtrim同理
eg:
SELECT ltrim('    SparkSQL   ');
+----------------------+
|ltrim(    SparkSQL   )|
+----------------------+
|           SparkSQL   |
+----------------------+

（19）mask(input[,upperChar,lowerChar,digitChar,otherChar])：屏蔽给定的字符串值。该函数将字符串替换为'X'或'x'，将数字替换为'n'。
eg:
SELECT mask('abcd-EFGH-8765-4321');
+----------------------------------------+
|mask(abcd-EFGH-8765-4321, X, x, n, NULL)|
+----------------------------------------+
|                     xxxx-XXXX-nnnn-nnnn|
+----------------------------------------+

SELECT mask('abcd-EFGH-8765-4321', 'Q');
+----------------------------------------+
|mask(abcd-EFGH-8765-4321, Q, x, n, NULL)|
+----------------------------------------+
|                     xxxx-QQQQ-nnnn-nnnn|
+----------------------------------------+

SELECT mask('AbCD123-@$#', 'Q', 'q');
+--------------------------------+
|mask(AbCD123-@$#, Q, q, n, NULL)|
+--------------------------------+
|                     QqQQnnn-@$#|
+--------------------------------+

SELECT mask('AbCD123-@$#');
+--------------------------------+
|mask(AbCD123-@$#, X, x, n, NULL)|
+--------------------------------+
|                     XxXXnnn-@$#|
+--------------------------------+

SELECT mask('AbCD123-@$#', 'Q');
+--------------------------------+
|mask(AbCD123-@$#, Q, x, n, NULL)|
+--------------------------------+
|                     QxQQnnn-@$#|
+--------------------------------+

SELECT mask('AbCD123-@$#', 'Q', 'q');
+--------------------------------+
|mask(AbCD123-@$#, Q, q, n, NULL)|
+--------------------------------+
|                     QqQQnnn-@$#|
+--------------------------------+

SELECT mask('AbCD123-@$#', 'Q', 'q', 'd');
+--------------------------------+
|mask(AbCD123-@$#, Q, q, d, NULL)|
+--------------------------------+
|                     QqQQddd-@$#|
+--------------------------------+

SELECT mask('AbCD123-@$#', 'Q', 'q', 'd', 'o');
+-----------------------------+
|mask(AbCD123-@$#, Q, q, d, o)|
+-----------------------------+
|                  QqQQdddoooo|
+-----------------------------+

SELECT mask('AbCD123-@$#', NULL, 'q', 'd', 'o');
+--------------------------------+
|mask(AbCD123-@$#, NULL, q, d, o)|
+--------------------------------+
|                     AqCDdddoooo|
+--------------------------------+

SELECT mask('AbCD123-@$#', NULL, NULL, 'd', 'o');
+-----------------------------------+
|mask(AbCD123-@$#, NULL, NULL, d, o)|
+-----------------------------------+
|                        AbCDdddoooo|
+-----------------------------------+

SELECT mask('AbCD123-@$#', NULL, NULL, NULL, 'o');
+--------------------------------------+
|mask(AbCD123-@$#, NULL, NULL, NULL, o)|
+--------------------------------------+
|                           AbCD123oooo|
+--------------------------------------+

SELECT mask(NULL, NULL, NULL, NULL, 'o');
+-------------------------------+
|mask(NULL, NULL, NULL, NULL, o)|
+-------------------------------+
|                           NULL|
+-------------------------------+

SELECT mask(NULL);
+-------------------------+
|mask(NULL, X, x, n, NULL)|
+-------------------------+
|                     NULL|
+-------------------------+

SELECT mask('AbCD123-@$#', NULL, NULL, NULL, NULL);
+-----------------------------------------+
|mask(AbCD123-@$#, NULL, NULL, NULL, NULL)|
+-----------------------------------------+
|                              AbCD123-@$#|
+-----------------------------------------+

（20）overlay(input,replace,pos[,len])：将input中从pos开始且长度为len的部分替换为replace。
eg:
SELECT overlay('Spark SQL' PLACING '_' FROM 6);
+----------------------------+
|overlay(Spark SQL, _, 6, -1)|
+----------------------------+
|                   Spark_SQL|
+----------------------------+

SELECT overlay('Spark SQL' PLACING 'CORE' FROM 7);
+-------------------------------+
|overlay(Spark SQL, CORE, 7, -1)|
+-------------------------------+
|                     Spark CORE|
+-------------------------------+

SELECT overlay('Spark SQL' PLACING 'ANSI ' FROM 7 FOR 0);
+-------------------------------+
|overlay(Spark SQL, ANSI , 7, 0)|
+-------------------------------+
|                 Spark ANSI SQL|
+-------------------------------+

SELECT overlay('Spark SQL' PLACING 'tructured' FROM 2 FOR 4);
+-----------------------------------+
|overlay(Spark SQL, tructured, 2, 4)|
+-----------------------------------+
|                     Structured SQL|
+-----------------------------------+

SELECT overlay(encode('Spark SQL', 'utf-8') PLACING encode('_', 'utf-8') FROM 6);
+----------------------------------------------------------+
|overlay(encode(Spark SQL, utf-8), encode(_, utf-8), 6, -1)|
+----------------------------------------------------------+
|                                      [53 70 61 72 6B 5...|
+----------------------------------------------------------+

SELECT overlay(encode('Spark SQL', 'utf-8') PLACING encode('CORE', 'utf-8') FROM 7);
+-------------------------------------------------------------+
|overlay(encode(Spark SQL, utf-8), encode(CORE, utf-8), 7, -1)|
+-------------------------------------------------------------+
|                                         [53 70 61 72 6B 2...|
+-------------------------------------------------------------+

SELECT overlay(encode('Spark SQL', 'utf-8') PLACING encode('ANSI ', 'utf-8') FROM 7 FOR 0);
+-------------------------------------------------------------+
|overlay(encode(Spark SQL, utf-8), encode(ANSI , utf-8), 7, 0)|
+-------------------------------------------------------------+
|                                         [53 70 61 72 6B 2...|
+-------------------------------------------------------------+

SELECT overlay(encode('Spark SQL', 'utf-8') PLACING encode('tructured', 'utf-8') FROM 2 FOR 4);
+-----------------------------------------------------------------+
|overlay(encode(Spark SQL, utf-8), encode(tructured, utf-8), 2, 4)|
+-----------------------------------------------------------------+
|                                             [53 74 72 75 63 7...|
+-----------------------------------------------------------------+

（21）position(substr,str[,pos])：返回substr在str中位置pos之后第一次出现的位置（索引从1开始）。
eg:
SELECT position('bar', 'foobarbar');
+---------------------------+
|position(bar, foobarbar, 1)|
+---------------------------+
|                          4|
+---------------------------+

SELECT position('bar', 'foobarbar', 5);
+---------------------------+
|position(bar, foobarbar, 5)|
+---------------------------+
|                          7|
+---------------------------+

SELECT POSITION('bar' IN 'foobarbar');
+-------------------------+
|locate(bar, foobarbar, 1)|
+-------------------------+
|                        4|
+-------------------------+

（22）regexp_count(str,regexp)：返回正则表达式regexp在字符串str中匹配的次数。
eg:
SELECT regexp_count('Steven Jones and Stephen Smith are the best players', 'Ste(v|ph)en');
+------------------------------------------------------------------------------+
|regexp_count(Steven Jones and Stephen Smith are the best players, Ste(v|ph)en)|
+------------------------------------------------------------------------------+
|                                                                             2|
+------------------------------------------------------------------------------+

SELECT regexp_count('abcdefghijklmnopqrstuvwxyz', '[a-z]{3}');
+--------------------------------------------------+
|regexp_count(abcdefghijklmnopqrstuvwxyz, [a-z]{3})|
+--------------------------------------------------+
|                                                 8|
+--------------------------------------------------+

（23）regexp_extract(str,regexp[,idx])：提取str中与regexp匹配的第一个字符串，并对应于正则表达式组索引。
eg:
SELECT regexp_extract('100-200', '(\\d+)-(\\d+)', 1);
+---------------------------------------+
|regexp_extract(100-200, (\d+)-(\d+), 1)|
+---------------------------------------+
|                                    100|
+---------------------------------------+

（24）regexp_extract_all(str,regexp[,idx])：提取str中与regexp匹配的所有字符串，并对应于正则表达式组索引。
eg:
SELECT regexp_extract_all('100-200, 300-400', '(\\d+)-(\\d+)', 1);
+----------------------------------------------------+
|regexp_extract_all(100-200, 300-400, (\d+)-(\d+), 1)|
+----------------------------------------------------+
|                                          [100, 300]|
+----------------------------------------------------+

（25）regexp_instr(str,regexp)：在字符串中搜索正则表达式，并返回一个整数，该整数指示匹配字符串的开始位置（索引从1开始），没有匹配到则返回0。
eg:
SELECT regexp_instr('user@spark.apache.org', '@[^.]*');
+----------------------------------------------+
|regexp_instr(user@spark.apache.org, @[^.]*, 0)|
+----------------------------------------------+
|                                             5|
+----------------------------------------------+

（26）regexp_replace(str,regexp,rep[,position])：将str中与regexp匹配的所有字符串替换为rep。
eg:
SELECT regexp_replace('100-200', '(\\d+)', 'num');
+--------------------------------------+
|regexp_replace(100-200, (\d+), num, 1)|
+--------------------------------------+
|                               num-num|
+--------------------------------------+
                                               
（27）regexp_substr(str,regexp)：返回str中正则表达式regexp匹配到的子字符串。如果没有找到则返回null。
eg:
SELECT regexp_substr('Steven Jones and Stephen Smith are the best players', 'Ste(v|ph)en');
+-------------------------------------------------------------------------------+
|regexp_substr(Steven Jones and Stephen Smith are the best players, Ste(v|ph)en)|
+-------------------------------------------------------------------------------+
|                                                                         Steven|
+-------------------------------------------------------------------------------+

SELECT regexp_substr('Steven Jones and Stephen Smith are the best players', 'Jeck');
+------------------------------------------------------------------------+
|regexp_substr(Steven Jones and Stephen Smith are the best players, Jeck)|
+------------------------------------------------------------------------+
|                                                                    NULL|
+------------------------------------------------------------------------+
                                               
（28）repeat(str,n)：返回str重复n次的结果。
eg:
SELECT repeat('123', 2);
+--------------+
|repeat(123, 2)|
+--------------+
|        123123|
+--------------+
                                               
（29）replace(str,search[,replace])：将所有出现的search替换为replace。
eg:
SELECT replace('ABCabc', 'abc', 'DEF');
+-------------------------+
|replace(ABCabc, abc, DEF)|
+-------------------------+
|                   ABCDEF|
+-------------------------+
                                               
（30）right(str,len)：返回str最右边的len个字符，如果len小于等于0则返回空字符串。
eg:
SELECT right('Spark SQL', 3);
+-------------------+
|right(Spark SQL, 3)|
+-------------------+
|                SQL|
+-------------------+
                                               
（31）split(str,regex[,limit])：使用regexp分割符将str分割为数组，并返回长度最多为limit的数组。
eg:
SELECT split('oneAtwoBthreeC', '[ABC]');
+--------------------------------+
|split(oneAtwoBthreeC, [ABC], -1)|
+--------------------------------+
|             [one, two, three, ]|
+--------------------------------+

SELECT split('oneAtwoBthreeC', '[ABC]', -1);
+--------------------------------+
|split(oneAtwoBthreeC, [ABC], -1)|
+--------------------------------+
|             [one, two, three, ]|
+--------------------------------+

SELECT split('oneAtwoBthreeC', '[ABC]', 2);
+-------------------------------+
|split(oneAtwoBthreeC, [ABC], 2)|
+-------------------------------+
|              [one, twoBthreeC]|
+-------------------------------+
                                               
（32）split_part(str,delimiter,partNum)：按分隔符拆分str并返回拆分后的请求部分（从1开始）。如果任何输入为null，则返回null。如果partNum超出拆分范围，则返回空字符串；如果partNum为0，则抛出错误；如果partNum为负数，则从字符串末尾开始倒数。如果delimiter是空字符串，则不拆分str。
eg：
SELECT split_part('11.12.13', '.', 3);
+--------------------------+
|split_part(11.12.13, ., 3)|
+--------------------------+
|                        13|
+--------------------------+
                                               
（33）substr(str,pos[,len])/substring(str,pos[,len])：返回str从pos开始且长度为len的子字符串。
eg:
SELECT substr('Spark SQL', 5);
+--------------------------------+
|substr(Spark SQL, 5, 2147483647)|
+--------------------------------+
|                           k SQL|
+--------------------------------+

SELECT substr('Spark SQL', -3);
+---------------------------------+
|substr(Spark SQL, -3, 2147483647)|
+---------------------------------+
|                              SQL|
+---------------------------------+

SELECT substr('Spark SQL', 5, 1);
+-----------------------+
|substr(Spark SQL, 5, 1)|
+-----------------------+
|                      k|
+-----------------------+

SELECT substr('Spark SQL' FROM 5);
+-----------------------------------+
|substring(Spark SQL, 5, 2147483647)|
+-----------------------------------+
|                              k SQL|
+-----------------------------------+

SELECT substr('Spark SQL' FROM -3);
+------------------------------------+
|substring(Spark SQL, -3, 2147483647)|
+------------------------------------+
|                                 SQL|
+------------------------------------+

SELECT substr('Spark SQL' FROM 5 FOR 1);
+--------------------------+
|substring(Spark SQL, 5, 1)|
+--------------------------+
|                         k|
+--------------------------+

SELECT substr(encode('Spark SQL', 'utf-8'), 5);
+-----------------------------------------------+
|substr(encode(Spark SQL, utf-8), 5, 2147483647)|
+-----------------------------------------------+
|                               [6B 20 53 51 4C]|
+-----------------------------------------------+
                                               
（34）substring_index(str,delim,count)：使用delim分隔符分割str，如果count为正数，则返回从左侧开始的前count个字符，如果count为负数，则返回从右侧开始的前count个字符。
eg:
SELECT substring_index('www.apache.org', '.', 2);
+-------------------------------------+
|substring_index(www.apache.org, ., 2)|
+-------------------------------------+
|                           www.apache|
+-------------------------------------+
                                               
（35）to_char(numberExpr,formatExpr)/to_varchar(numberExpr,formatExpr)：根据formatExpr将numberExpr转换为字符串。
eg:
SELECT to_char(454, '999');
+-----------------+
|to_char(454, 999)|
+-----------------+
|              454|
+-----------------+

SELECT to_char(454.00, '000D00');
+-----------------------+
|to_char(454.00, 000D00)|
+-----------------------+
|                 454.00|
+-----------------------+

SELECT to_char(12454, '99G999');
+----------------------+
|to_char(12454, 99G999)|
+----------------------+
|                12,454|
+----------------------+

SELECT to_char(78.12, '$99.99');
+----------------------+
|to_char(78.12, $99.99)|
+----------------------+
|                $78.12|
+----------------------+

SELECT to_char(-12454.8, '99G999D9S');
+----------------------------+
|to_char(-12454.8, 99G999D9S)|
+----------------------------+
|                   12,454.8-|
+----------------------------+
                                               
（36）to_number(expr,fmt)：根据fmt将expr转换为数字。
eg:
SELECT to_number('454', '999');
+-------------------+
|to_number(454, 999)|
+-------------------+
|                454|
+-------------------+

SELECT to_number('454.00', '000.00');
+-------------------------+
|to_number(454.00, 000.00)|
+-------------------------+
|                   454.00|
+-------------------------+

SELECT to_number('12,454', '99,999');
+-------------------------+
|to_number(12,454, 99,999)|
+-------------------------+
|                    12454|
+-------------------------+

SELECT to_number('$78.12', '$99.99');
+-------------------------+
|to_number($78.12, $99.99)|
+-------------------------+
|                    78.12|
+-------------------------+

SELECT to_number('12,454.8-', '99,999.9S');
+-------------------------------+
|to_number(12,454.8-, 99,999.9S)|
+-------------------------------+
|                       -12454.8|
+-------------------------------+

                                               
（37）translate(input,from,to)：将input中的from替换为to。
eg:
SELECT translate('AaBbCc', 'abc', '123');
+---------------------------+
|translate(AaBbCc, abc, 123)|
+---------------------------+
|                     A1B2C3|
+---------------------------+
                                               
（38）ucase(str)/upper(str)：将str全部转为大写。
eg：
SELECT ucase('SparkSql');
+---------------+
|ucase(SparkSql)|
+---------------+
|       SPARKSQL|
+---------------+
```

### 1.7、条件函数

```sql
（1）coalesce(expr1,expr2,...)：从左往右，返回第一个不为空的表达式，否则返回null。
eg:
SELECT coalesce(NULL, 1, NULL);
+-----------------------+
|coalesce(NULL, 1, NULL)|
+-----------------------+
|                      1|
+-----------------------+
```

```sql
（2）if(expr1,expr2,expr3)：如果expr1为true，则返回expr2，否则返回expr3。
eg：
SELECT if(1 < 2, 'a', 'b');
+-------------------+
|(IF((1 < 2), a, b))|
+-------------------+
|                  a|
+-------------------+
```

```sql
（3）ifnull(expr1，expr2)/nvl(expr1,expr2)：如果expr1为null，则返回expr2，否则返回expr1。
eg:
SELECT ifnull(NULL, array('2'));
+----------------------+
|ifnull(NULL, array(2))|
+----------------------+
|                   [2]|
+----------------------+
```

```sql
（4）nanvl(expr1,expr2)：如果expr1为NAN，则返回expr1，否则返回expr2。
eg:
SELECT nanvl(cast('NaN' as double), 123);
+-------------------------------+
|nanvl(CAST(NaN AS DOUBLE), 123)|
+-------------------------------+
|                          123.0|
+-------------------------------+
```

```sql
（5）nullif(expr1,expr2)：如果expr1等于expr2，则返回null，否则返回expr1。
eg:
SELECT nullif(2, 2);
+------------+
|nullif(2, 2)|
+------------+
|        NULL|
+------------+
```

```sql
（6）nvl2(expr1,expr2,expr3)：如果expr1不为null，则返回expr2，否则返回expr3。
eg:
SELECT nvl2(NULL, 2, 1);
+----------------+
|nvl2(NULL, 2, 1)|
+----------------+
|               1|
+----------------+
```

```sql
（7）case when expr1 then expr2 [when expr3 then expr4]* [else expr5] end：当expr1为true时，返回expr2；否则，当expr3为true时，返回expr4；否则返回默认值expr5。
eg:
SELECT CASE WHEN 1 > 0 THEN 1 WHEN 2 > 0 THEN 2.0 ELSE 1.2 END;
+-----------------------------------------------------------+
|CASE WHEN (1 > 0) THEN 1 WHEN (2 > 0) THEN 2.0 ELSE 1.2 END|
+-----------------------------------------------------------+
|                                                        1.0|
+-----------------------------------------------------------+

SELECT CASE WHEN 1 < 0 THEN 1 WHEN 2 > 0 THEN 2.0 ELSE 1.2 END;
+-----------------------------------------------------------+
|CASE WHEN (1 < 0) THEN 1 WHEN (2 > 0) THEN 2.0 ELSE 1.2 END|
+-----------------------------------------------------------+
|                                                        2.0|
+-----------------------------------------------------------+

SELECT CASE WHEN 1 < 0 THEN 1 WHEN 2 < 0 THEN 2.0 END;
+--------------------------------------------------+
|CASE WHEN (1 < 0) THEN 1 WHEN (2 < 0) THEN 2.0 END|
+--------------------------------------------------+
|                                              NULL|
+--------------------------------------------------+
```

### 1.8、CSV函数

```sql
（1）from_csv(csvStr,schema[,options])：返回给定的csvStr和schema的结构值。
eg:
SELECT from_csv('1, 0.8', 'a INT, b DOUBLE');
+----------------+
|from_csv(1, 0.8)|
+----------------+
|        {1, 0.8}|
+----------------+

SELECT from_csv('26/08/2015', 'time Timestamp', map('timestampFormat', 'dd/MM/yyyy'));
+--------------------+
|from_csv(26/08/2015)|
+--------------------+
|{2015-08-26 00:00...|
+--------------------+
```

```sql
（2）schema_of_csv(csv[,options])：以csv字符串的DDL格式返回。
eg:
SELECT schema_of_csv('1,abc');
+--------------------+
|schema_of_csv(1,abc)|
+--------------------+
|STRUCT<_c0: INT, ...|
+--------------------+
```

```sql
（3）to_csv(expr[,options])：返回给定结构值的CSV字符串。
eg:
SELECT to_csv(named_struct('a', 1, 'b', 2));
+--------------------------------+
|to_csv(named_struct(a, 1, b, 2))|
+--------------------------------+
|                             1,2|
+--------------------------------+

SELECT to_csv(named_struct('time', to_timestamp('2015-08-26', 'yyyy-MM-dd')), map('timestampFormat', 'dd/MM/yyyy'));
+----------------------------------------------------------------+
|to_csv(named_struct(time, to_timestamp(2015-08-26, yyyy-MM-dd)))|
+----------------------------------------------------------------+
|                                                      26/08/2015|
+----------------------------------------------------------------+
```



## 二、sparksql语法

### 2.1、DDL

#### 2.1.1、创建数据库

```sql
CREATE { DATABASE | SCHEMA } [ IF NOT EXISTS ] database_name
    [ COMMENT database_comment ]
    [ LOCATION database_directory ]
    [ WITH DBPROPERTIES ( property_name = property_value [ , ... ] ) ]
    
eg：
CREATE DATABASE IF NOT EXISTS db_name
COMMENT 'This is my first db'
LOCATION '/spark_db'
WITH DBPROPERTIES(create_time='2024-10-28');
```

#### 2.1.2、创建表

```sql
① 创建数据源表
CREATE TABLE [ IF NOT EXISTS ] table_identifier
    [ ( col_name1 col_type1 [ COMMENT col_comment1 ], ... ) ]
    USING data_source
    [ OPTIONS ( key1=val1, key2=val2, ... ) ]
    [ PARTITIONED BY ( col_name1, col_name2, ... ) ]
    [ CLUSTERED BY ( col_name3, col_name4, ... ) 
        [ SORTED BY ( col_name [ ASC | DESC ], ... ) ] 
        INTO num_buckets BUCKETS ]
    [ LOCATION path ]
    [ COMMENT table_comment ]
    [ TBLPROPERTIES ( key1=val1, key2=val2, ... ) ]
    [ AS select_statement ]
  
eg:
CREATE TABLE IF NOT EXISTS tbl_name(
	id INT,
    name STRING,
    class STRING
)
USING org.apache.spark.sql.jdbc
OPTIONS(
	url="jdbc:mysql://localhost:3306/test",
    dbtable="test.student",
    driver="com.mysql.cj.jdbc.Driver",
    user="root",
    password="123456"
)
PARTITIONED BY (class)
CLUSTERED BY (id)
SORTED BY (id)
INTO 6 BUCKETS
LOCATION "/spark_db/"
COMMENT "my first jdbc table"
TBLPROPERTIES (create_time="2024-10-28");

② 创建hive格式表
CREATE [ EXTERNAL ] TABLE [ IF NOT EXISTS ] table_identifier
    [ ( col_name1[:] col_type1 [ COMMENT col_comment1 ], ... ) ]
    [ COMMENT table_comment ]
    [ PARTITIONED BY ( col_name2[:] col_type2 [ COMMENT col_comment2 ], ... ) 
        | ( col_name1, col_name2, ... ) ]
    [ CLUSTERED BY ( col_name1, col_name2, ...) 
        [ SORTED BY ( col_name1 [ ASC | DESC ], col_name2 [ ASC | DESC ], ... ) ] 
        INTO num_buckets BUCKETS ]
    [ ROW FORMAT row_format ]
    [ STORED AS file_format ]
    [ LOCATION path ]
    [ TBLPROPERTIES ( key1=val1, key2=val2, ... ) ]
    [ AS select_statement ]

row_format:    
    : SERDE serde_class [ WITH SERDEPROPERTIES (k1=v1, k2=v2, ... ) ]
    | DELIMITED [ FIELDS TERMINATED BY fields_terminated_char [ ESCAPED BY escaped_char ] ] 
        [ COLLECTION ITEMS TERMINATED BY collection_items_terminated_char ] 
        [ MAP KEYS TERMINATED BY map_key_terminated_char ]
        [ LINES TERMINATED BY row_terminated_char ]
        [ NULL DEFINED AS null_char ]

eg:(ESCAPED BY '\n'的意思是\n应该被转义)
CREATE EXTERNAL TABLE family(
	name STRING,
    friends ARRAY<STRING>,
    children MAP<STRING, INT>,
    address STRUCT<street:STRING, city:STRING>
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ESCAPED BY '\n'
    COLLECTION ITEMS TERMINATED BY '_'
    MAP KEYS TERMINATED BY ':'
    LINES TERMINATED BY '\n'
    NULL DEFINED AS 'foonull'
    STORED AS TEXTFILE
    LOCATION '/tmp/family/';
    
③ 使用like复制已存在的表结构
CREATE TABLE [IF NOT EXISTS] table_identifier LIKE source_table_identifier
    USING data_source
    [ ROW FORMAT row_format ]
    [ STORED AS file_format ]
    [ TBLPROPERTIES ( key1=val1, key2=val2, ... ) ]
    [ LOCATION path ]

eg:
CREATE TABLE Student_Dupli like Student USING CSV;

CREATE TABLE Student_Dupli like Student
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE
    TBLPROPERTIES ('owner'='xxxx');
```

#### 2.1.3、创建视图

```sql
CREATE [ OR REPLACE ] [ [ GLOBAL ] TEMPORARY ] VIEW [ IF NOT EXISTS ] view_identifier
    create_view_clauses AS query
    
eg:
CREATE OR REPLACE VIEW experienced_employee
    (ID COMMENT 'Unique identification number', Name) 
    COMMENT 'View for experienced employees'
    AS SELECT id, name FROM all_employee
        WHERE working_years > 5;
        
        
eg:
CREATE GLOBAL TEMPORARY VIEW IF NOT EXISTS subscribed_movies 
    AS SELECT mo.member_id, mb.full_name, mo.movie_title
        FROM movies AS mo INNER JOIN members AS mb 
        ON mo.member_id = mb.id;
```

#### 2.1.4、修改表

```sql
1、修改表名
ALTER TABLE tbl_name RENAME TO new_tbl_name;

eg:
ALTER TABLE Student RENAME TO StudentInfo;
```

```sql
2、修改表分区名
ALTER TABLE tbl_name part_name RENAME TO new_part_name;

eg:
ALTER TABLE default.StudentInfo PARTITION (age='10') RENAME TO PARTITION (age='15');
```

```sql
3、新增列
ALTER TABLE tbl_name ADD COLUMNS (col_name col_type[,...]);

eg:
ALTER TABLE StudentInfo ADD columns (LastName string, DOB timestamp);
```

```sql
4、删除列
ALTER TABLE tbl_name DROP COLUMNS (col_name[,...]);

eg:
ALTER TABLE StudentInfo DROP columns (LastName, DOB);
```

```sql
5、修改列名
ALTER TBALE tbl_name RENAME COLUMN col_name TO new_col_name;

eg:
ALTER TABLE StudentInfo RENAME COLUMN name TO FirstName;
```

```sql
6、修改列定义
ALTER TABLE tbl_name [ALTER|CHANGE] COLUMN col_name new_col_type;

eg:
ALTER TABLE StudentInfo ALTER COLUMN FirstName COMMENT "new comment";
```

```sql
7、重新定义列
ALTER TABLE tbl_name REPLACE COLUMNS (col_name col_type[,...]);

eg:
ALTER TABLE StudentInfo REPLACE COLUMNS (name string, ID int COMMENT 'new comment');
```

```sql
8、添加分区
ALTER TABLE tbl_name ADD [IF NOT EXISTS] PARTITION (part_name1) PARTITION (part_name2);

eg:
ALTER TABLE StudentInfo ADD IF NOT EXISTS PARTITION (age=18);

ALTER TABLE StudentInfo ADD IF NOT EXISTS PARTITION (age=18) PARTITION (age=20);
```

```sql
9、删除分区
ALTER TABLE tbl_name DROP [IF EXISTS] PARTITION (part_name[,...]);

eg:
ALTER TABLE StudentInfo DROP IF EXISTS PARTITION (age=18);
```

```sql
10、设置表属性
ALTER TABLE tbl_name SET TBLPROPERTIES(key1=val1[,...]);

eg:
ALTER TABLE dbx.tab1 SET TBLPROPERTIES ('winner' = 'loser');

ALTER TABLE dbx.tab1 SET TBLPROPERTIES ('comment' = 'A table comment.');
```

```sql
11、删除表属性
ALTER TABLE tbl_name UNSET TBLPROPERTIES [IF EXISTS](key1[,...]);

eg:
ALTER TABLE dbx.tab1 UNSET TBLPROPERTIES ('winner');
```

```sql
12、设置序列化格式
ALTER TABLE tbl_name [part_name] SET SERDEPROPERTIES (key1=val1, key2=val2[,...]);

ALTER TABLE tbl_name [part_name] SET SERDE serde_class_name [WITH SERDEPROPERTIES ( key1 = val1, key2 = val2, ... )];

eg:
ALTER TABLE test_tab SET SERDE 'org.apache.hadoop.hive.serde2.columnar.LazyBinaryColumnarSerDe';
```

```sql
13、设置表或分区存储路径
ALTER TABLE tbl_name [part_name] SET LOCATION 'new_location';

eg:
ALTER TABLE dbx.tab1 PARTITION (a='1', b='2') SET LOCATION '/path/to/part/ways'
```

```sql
14、设置表文件格式
ALTER TABLE tbl_name [part_name] SET FILEFORMAT file_format;

eg:
ALTER TABLE loc_orc SET fileformat orc;

ALTER TABLE p1 partition (month=2, day=2) SET fileformat parquet;
```

```sql
15、修复表
ALTER TABLE tbl_name RECOVER PARTITIONS;
或 MSCK REPAIR TABLE tbl_name [{ADD|DROP|SYNC} PARTITIONS];(默认ADD，SYNC是ADD和DROP的组合)

eg:
ALTER TABLE dbx.tab1 RECOVER PARTITIONS;
```

#### 2.1.5、创建函数

```sql
CREATE [ OR REPLACE ] [ TEMPORARY ] FUNCTION [ IF NOT EXISTS ]
    function_name AS class_name [ resource_locations ];
    
eg:
CREATE FUNCTION simple_udf AS 'SimpleUdf'
    USING JAR '/tmp/SimpleUdf.jar';
```

### 2.2、DML

#### 2.2.1、使用INSERT向表中插入数据

```sql
INSERT [ INTO | OVERWRITE ] [ TABLE ] table_identifier [ partition_spec ] [ ( column_list ) ]
    { VALUES ( { value | NULL } [ , ... ] ) [ , ( ... ) ] | query };
    
eg:
INSERT INTO students VALUES
    ('Amy Smith', '123 Park Ave, San Jose', 111111);
    
INSERT INTO students VALUES
    ('Bob Brown', '456 Taylor St, Cupertino', 222222),
    ('Cathy Johnson', '789 Race Ave, Palo Alto', 333333);
```

```sql
INSERT INTO [ TABLE ] table_identifier REPLACE WHERE boolean_expression query;

eg:
INSERT INTO students PARTITION (student_id = 444444)
    SELECT name, address FROM persons WHERE name = "Dora Williams";
    
INSERT INTO students TABLE visiting_students;

INSERT INTO students
     FROM applicants SELECT name, address, student_id WHERE qualified = true;
     
INSERT INTO persons REPLACE WHERE ssn = 123456789 SELECT * FROM persons2
解释：将person2中的数据替换person中ssn=123456789的数据
```

#### 2.2.2、使用INSERT OVERWRITE DIRECTORY覆盖现有文件

```sql
INSERT OVERWRITE [ LOCAL ] DIRECTORY [ directory_path ]
    { spark_format | hive_format }
    { VALUES ( { value | NULL } [ , ... ] ) [ , ( ... ) ] | query };
```

```sql
其中spark_format定义为
USING file_format [ OPTIONS ( key = val [ , ... ] ) ]
```

```sql
hive_format 定义为
[ ROW FORMAT row_format ] [ STORED AS hive_serde ]
```

```sql
spark格式
eg:
INSERT OVERWRITE DIRECTORY '/tmp/destination'
    USING parquet
    OPTIONS (col1 1, col2 2, col3 'test')
    SELECT * FROM test_table;

INSERT OVERWRITE DIRECTORY
    USING parquet
    OPTIONS ('path' '/tmp/destination', col1 1, col2 2, col3 'test')
    SELECT * FROM test_table;
```

```sql
hive格式
eg:
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/destination'
    STORED AS orc
    SELECT * FROM test_table;

INSERT OVERWRITE LOCAL DIRECTORY '/tmp/destination'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    SELECT * FROM test_table;
```

#### 2.2.3、使用LOAD DATA将数据加载到表

```sparql
LOAD DATA [ LOCAL ] INPATH path [ OVERWRITE ] INTO TABLE table_identifier [ partition_spec ]

eg:
LOAD DATA LOCAL INPATH '/user/hive/warehouse/students' OVERWRITE INTO TABLE test_load;

LOAD DATA LOCAL INPATH '/user/hive/warehouse/test_partition/c2=2/c3=3'
    OVERWRITE INTO TABLE test_load_partition PARTITION (c2=2, c3=3);
```

### 2.3、查询

#### 2.3.1、WITH

```SPARQL
WITH common_table_expression [ , ... ]
其中 common_table_expression 为
expression_name [ ( column_name [ , ... ] ) ] [ AS ] ( query );

eg:
WITH
    t AS (SELECT 1),
    t2 AS (
        WITH t AS (SELECT 2)
        SELECT * FROM t
    );
```

#### 2.3.2、CLUSTER BY

```SPARQL
对查询数据进行分区，然后对每个分区中的数据进行排序（等同于先执行DISTRIBUTE BY，再执行SORT BY）
CLUSTER BY { expression [ , ... ] }

eg:
SELECT age, name FROM person CLUSTER BY age;
+---+-------+
|age|   name|
+---+-------+
| 18| John A|
| 18| Anil B|
| 25|Zen Hui|
| 25| Mike A|
| 16|Shone S|
| 16| Jack N|
+---+-------+
```

#### 2.3.3、DISTRIBUTE BY

```SPARQL
对查询数据重新分区
DISTRIBUTE BY { expression [ , ... ] };

eg:
SELECT age, name FROM person DISTRIBUTE BY age;
+---+-------+
|age|   name|
+---+-------+
| 25|Zen Hui|
| 25| Mike A|
| 18| John A|
| 18| Anil B|
| 16|Shone S|
| 16| Jack N|
+---+-------+
```

#### 3.3.4、GROUP BY

```SPARQL
GROUP BY group_expression [ , group_expression [ , ... ] ] [ WITH { ROLLUP | CUBE } ];

GROUP BY { group_expression | { ROLLUP | CUBE | GROUPING SETS } (grouping_set [ , ...]) } [ , ... ];

聚合函数定义为
aggregate_name ( [ DISTINCT ] expression [ , ... ] ) [ FILTER ( WHERE boolean_expression ) ];

参数解释：
GROUPING SETS：将多个分组条件合并为一个集合，相当于多个group by的结果UNION ALL。GROUP BY GROUPING SETS((name,age),(name))相当于GROUP BY name,age UNION ALL GROUP BY name。

ROLLUP：是GROUPING SETS的简写形式。例如GROUP BY warehouse,product WITH ROLLUP或GROUP BY ROLLUP(warehouse,product)等同于GROUP BY GROUPING SETS((warehouse,product),(warehouse))。ROLLUP规范的N个元素会生成N+1个GROUPING SETS。

CUBE：是GROUPING SETS的简写形式。例如GROUP BY warehouse,product WITH CUBE或GROUP BY CUBE(warehouse,product)等同于GROUP BY GROUPING SETS((warehouse,product),(warehouse),(product),())。CUBE规范的N个元素会生成2^N个GROUPING SETS。

FILTER：过滤输入行，其中where子句为true的行将传递给聚合函数，其他行将被丢弃。

eg:
SELECT id, sum(quantity) FILTER (
            WHERE car_model IN ('Honda Civic', 'Honda CRV')
        ) AS `sum(quantity)` FROM dealer
    GROUP BY id ORDER BY id;
+---+-------------+
| id|sum(quantity)|
+---+-------------+
|100|           17|
|200|           23|
|300|            5|
+---+-------------+


SELECT city, car_model, sum(quantity) AS sum FROM dealer
    GROUP BY GROUPING SETS ((city, car_model), (city), (car_model), ())
    ORDER BY city;
+---------+------------+---+
|     city|   car_model|sum|
+---------+------------+---+
|     null|        null| 78|
|     null| HondaAccord| 33|
|     null|    HondaCRV| 10|
|     null|  HondaCivic| 35|
|   Dublin|        null| 33|
|   Dublin| HondaAccord| 10|
|   Dublin|    HondaCRV|  3|
|   Dublin|  HondaCivic| 20|
|  Fremont|        null| 32|
|  Fremont| HondaAccord| 15|
|  Fremont|    HondaCRV|  7|
|  Fremont|  HondaCivic| 10|
| San Jose|        null| 13|
| San Jose| HondaAccord|  8|
| San Jose|  HondaCivic|  5|
+---------+------------+---+

SELECT city, car_model, sum(quantity) AS sum FROM dealer
    GROUP BY city, car_model WITH ROLLUP
    ORDER BY city, car_model;
+---------+------------+---+
|     city|   car_model|sum|
+---------+------------+---+
|     null|        null| 78|
|   Dublin|        null| 33|
|   Dublin| HondaAccord| 10|
|   Dublin|    HondaCRV|  3|
|   Dublin|  HondaCivic| 20|
|  Fremont|        null| 32|
|  Fremont| HondaAccord| 15|
|  Fremont|    HondaCRV|  7|
|  Fremont|  HondaCivic| 10|
| San Jose|        null| 13|
| San Jose| HondaAccord|  8|
| San Jose|  HondaCivic|  5|
+---------+------------+---+

SELECT city, car_model, sum(quantity) AS sum FROM dealer
    GROUP BY city, car_model WITH CUBE
    ORDER BY city, car_model;
+---------+------------+---+
|     city|   car_model|sum|
+---------+------------+---+
|     null|        null| 78|
|     null| HondaAccord| 33|
|     null|    HondaCRV| 10|
|     null|  HondaCivic| 35|
|   Dublin|        null| 33|
|   Dublin| HondaAccord| 10|
|   Dublin|    HondaCRV|  3|
|   Dublin|  HondaCivic| 20|
|  Fremont|        null| 32|
|  Fremont| HondaAccord| 15|
|  Fremont|    HondaCRV|  7|
|  Fremont|  HondaCivic| 10|
| San Jose|        null| 13|
| San Jose| HondaAccord|  8|
| San Jose|  HondaCivic|  5|
+---------+------------+---+
```

#### 3.3.5、提示

```SPARQL
提示为用户提供了一种方法，可以建议sparksql使用特定方法来生成执行计划
/*+ hint [ , ... ] */
```

```SPARQL
分区提示：允许用户指定spark应该遵循的分区策略
COALESCE：使用coalesce提示将分区数量减少到指定的分区数量。将分区数量作为参数
REPARTITION：使用指定的分区表达式将分区重新分区到指定的分区数量。将分区数量、列名作为参数
REPARTITION_BY_RANGE：使用指定的分区表达式将分区重新分区到指定的分区数量。将列名和分区数量作为参数
REBALANCE：重新平衡查询结果输出分区，以便每个分区的大小都合理。将列名作为参数

eg:
SELECT /*+ COALESCE(3) */ * FROM t;

SELECT /*+ REPARTITION(3) */ * FROM t;

SELECT /*+ REPARTITION(c) */ * FROM t;

SELECT /*+ REPARTITION(3, c) */ * FROM t;

SELECT /*+ REPARTITION_BY_RANGE(c) */ * FROM t;

SELECT /*+ REPARTITION_BY_RANGE(3, c) */ * FROM t;

SELECT /*+ REBALANCE */ * FROM t;

SELECT /*+ REBALANCE(3) */ * FROM t;

SELECT /*+ REBALANCE(c) */ * FROM t;

SELECT /*+ REBALANCE(3, c) */ * FROM t;
```

```SPARQL
连接提示：允许用户指定spark应该使用的连接策略
BROADCAST：使用广播连接。具有提示的连接侧将被广播，如果两侧都有广播，则大小较小的一侧将被广播。
MERGE：使用shuffle排序合并连接。
SHUFFLE_HASH：使用shuffle哈希连接、
SHUFFLE_REPLICATE_NL：使用shuffle和复制嵌套循环连接。

eg:
-- Join Hints for broadcast join
SELECT /*+ BROADCAST(t1) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;
SELECT /*+ BROADCASTJOIN (t1) */ * FROM t1 left JOIN t2 ON t1.key = t2.key;
SELECT /*+ MAPJOIN(t2) */ * FROM t1 right JOIN t2 ON t1.key = t2.key;

-- Join Hints for shuffle sort merge join
SELECT /*+ SHUFFLE_MERGE(t1) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;
SELECT /*+ MERGEJOIN(t2) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;
SELECT /*+ MERGE(t1) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;

-- Join Hints for shuffle hash join
SELECT /*+ SHUFFLE_HASH(t1) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;

-- Join Hints for shuffle-and-replicate nested loop join
SELECT /*+ SHUFFLE_REPLICATE_NL(t1) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;

-- When different join strategy hints are specified on both sides of a join, Spark
-- prioritizes the BROADCAST hint over the MERGE hint over the SHUFFLE_HASH hint
-- over the SHUFFLE_REPLICATE_NL hint.
-- Spark will issue Warning in the following example
-- org.apache.spark.sql.catalyst.analysis.HintErrorLogger: Hint (strategy=merge)
-- is overridden by another hint and will not take effect.
SELECT /*+ BROADCAST(t1), MERGE(t1, t2) */ * FROM t1 INNER JOIN t2 ON t1.key = t2.key;
```

#### 3.3.6、使用SQL直接查询指定格式文件

```SPARQL
file_format.'file_path'

eg:
-- PARQUET file
SELECT * FROM parquet.`examples/src/main/resources/users.parquet`;
+------+--------------+----------------+
|  name|favorite_color|favorite_numbers|
+------+--------------+----------------+
|Alyssa|          null|  [3, 9, 15, 20]|
|   Ben|           red|              []|
+------+--------------+----------------+

-- ORC file
SELECT * FROM orc.`examples/src/main/resources/users.orc`;
+------+--------------+----------------+
|  name|favorite_color|favorite_numbers|
+------+--------------+----------------+
|Alyssa|          null|  [3, 9, 15, 20]|
|   Ben|           red|              []|
+------+--------------+----------------+

-- JSON file
SELECT * FROM json.`examples/src/main/resources/people.json`;
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
```

#### 3.3.7、OFFSET指定跳过的行数

```SPARQL
OFFSET integer_expression;

eg:
CREATE TABLE person (name STRING, age INT);
INSERT INTO person VALUES
    ('Zen Hui', 25),
    ('Anil B', 18),
    ('Shone S', 16),
    ('Mike A', 25),
    ('John A', 18),
    ('Jack N', 16);

-- Skip the first two rows.
SELECT name, age FROM person ORDER BY name OFFSET 2;
+-------+---+
|   name|age|
+-------+---+
| John A| 18|
| Mike A| 25|
|Shone S| 16|
|Zen Hui| 25|
+-------+---+

-- Skip the first two rows and returns the next three rows.
SELECT name, age FROM person ORDER BY name LIMIT 3 OFFSET 2;
+-------+---+
|   name|age|
+-------+---+
| John A| 18|
| Mike A| 25|
|Shone S| 16|
+-------+---+

-- A function expression as an input to OFFSET.
SELECT name, age FROM person ORDER BY name OFFSET length('SPARK');
+-------+---+
|   name|age|
+-------+---+
|Zen Hui| 25|
+-------+---+
```

