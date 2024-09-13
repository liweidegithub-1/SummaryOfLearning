# Hive函数总结

## 1、常用日期函数([【超详细】HIVE 日期函数（当前日期、时间戳转换、前一天日期等）_hive當前時間-CSDN博客](https://blog.csdn.net/ymzhu385/article/details/136222437))

### 	1.1、常量：当前日期、时间戳

| 输入类型 | 返回类型  | 名称              | 样例                       | 描述                                     |
| -------- | --------- | ----------------- | -------------------------- | ---------------------------------------- |
| void     | date      | current_date      | select current_date()      | 返回当前日期 '2024-09-09'                |
| void     | timestamp | current_timestamp | select current_timestamp() | 返回当前时间戳 '2024-09-09 01:32:52.529' |

### 	1.2、前一天日期、后一天日期

| 输入类型     | 返回类型 | 名称     | 样例                              | 描述                   |
| ------------ | -------- | -------- | --------------------------------- | ---------------------- |
| (string,int) | date     | date_add | select date_add('2024-01-01', 1); | 返回后一天'2024-01-02' |
| (string,int) | date     | date_sub | select date_sub('2024-01-02', 1); | 返回前一天'2024-01-01' |

### 	1.3、获取日期中的年、月、日、时、分、秒

| 输入类型              | 返回类型 | 名称     | 样例                                                         | 描述                                               |
| --------------------- | -------- | -------- | ------------------------------------------------------------ | -------------------------------------------------- |
| string/date/timestamp | int      | year     | select year('2024-09-09');                                   | 返回当前日期中的年份'2024'                         |
| string/date/timestamp | int      | month    | SELECT month('2024-09-09');                                  | 返回当前日期中的月份'9'                            |
| string/date/timestamp | int      | day      | select day('2024-09-09');                                    | 返回当前日期中的日'9'                              |
| string/timestamp      | int      | hour     | select hour('2024-01-01 20:08:13');                          | 返回当前日期中的时'20'                             |
| string/timestamp      | int      | minute   | select minute('2024-01-01 20:08:13');                        | 返回当前日期中的分'8'                              |
| string/timestamp      | int      | second   | select second('2024-01-01 20:08:13');                        | 返回当前日期中的秒'13'                             |
| string/date/timestamp | string   | last_day | select last_day('2024-09-09');                               | 返回本月最后一天'2024-09-30'                       |
| (string,string)       | string   | trunc    | select trunc('2024-09-09','MM');/select trunc('2024-09-09','YY'); | 返回月初日期'2024-09-01';/返回年初日期'2024-01-01' |

### 	1.4、格式转换

| 输入类型              | 返回类型 | 名称           | 样例                                                         | 描述                                                      |
| --------------------- | -------- | -------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| string/date/timestamp | string   | to_date        | select to_date('2023-09-01 12:13:14');                       | 返回十位日期格式'2023-09-01'                              |
| (string,[string])     | bigint   | unix_timestamp | select unix_timestamp('2024-09-09','yyyy-MM-dd');/select unix_timestamp('2023-09-01 12:13:14'); | 返回unix时间戳'1725840000'/返回当前unix时间戳'1693570394' |
| bigint                | string   | from_unixtime  | select from_unixtime(1725878350,'yyyy-MM-dd');               | 返回当前unix时间戳的指定格式日期'2024-09-09'              |
| string/date/timestamp | string   | date_format    | select date_format('2024-09-09','yyyyMMdd');                 | 返回指定格式的日期'20240909'                              |

### 	1.5、日期之差

| 输入类型              | 返回类型 | 名称           | 样例                                              | 描述                       |
| --------------------- | -------- | -------------- | ------------------------------------------------- | -------------------------- |
| string/date/timestamp | double   | months_between | select months_between('2024-09-09','2024-08-01'); | 返回日期月份之差1.25806452 |
| string                | int      | datediff       | select datediff('2024-01-02','2023-02-02');       | 返回日期天数差值334        |

## 2、解析json([Hive解析Json数组超全讲解_hive解析son工具-CSDN博客](https://blog.csdn.net/helloHbulie/article/details/116699133))

### 	2.1、json函数

| 输入格式                                     | 名称            | 样例                                                         | 描述                            |
| -------------------------------------------- | --------------- | ------------------------------------------------------------ | ------------------------------- |
| (json字符串,'$.json字段名')                  | get_json_object | select get_json_object('{"name":"Bob","age":20}','$.name');  | 获取json字符串中的name字段'Bob' |
| (json字符串,'json字段名1','json字段名2',...) | json_tuple      | select t2.name, t2.age from dual t1 lateral view json_tuple('{"name":"Bob","age":20}','name', 'age') t2 as name,age; | 获取json字符串中的多个字段      |

### 	2.3、json数组

#### 		2.3.1、将json数组内部的分割符号由逗号替换为分号，便于区分数组内每个json元素和json内部的字段

```mysql
select regexp_replace(regexp_replace(json_str,'\\[|\\]',''),'\\},\\{','\\};\\{')
```

![原始数据](D:\markdown_picture\picture_store\屏幕截图 2024-09-09 151805.png)

![替换分隔符后数据](D:\markdown_picture\picture_store\屏幕截图 2024-09-09 152112.png)

#### 		2.3.2、将json数组炸裂为一个个的json字符串

```mysql
select
t2.j
from json_test t1
lateral view explode(split(regexp_replace(regexp_replace(json_str,'\\[|\\]',''),'\\},\\{','\\};\\{'),';')) t2 as j;
```

![炸裂后的数据](D:\markdown_picture\picture_store\屏幕截图 2024-09-09 151701.png)

#### 		2.3.4、使用json_tuple解析json字符串

```
select
json_tuple(t3.j, 'website','name')
from (
select
t2.j
from json_test t1
lateral view explode(split(regexp_replace(regexp_replace(json_str,'\\[|\\]',''),'\\},\\{','\\};\\{'),';')) t2 as j) t3
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-09 155643.png)

## 3、开窗函数

### 	3.1、开窗排序

```mysql
3.1.1、row_number() over(partition by 分区字段 order by 排序字段)
	在指定分区内，按照指定字段进行排序，分区内排序不重复（1,2,3,4...）
eg：
SELECT
identify, 
age,
row_number() over(PARTITION BY identify ORDER BY age) rn
FROM person;
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 143018.png)

```mysql
3.1.2、rank() over(partition by 分区字段 order by 排序字段)
	在指定分区内，按照指定字段进行排序，分区内排序会重复，且会有跳行(1,2,2,4,5,5,5,8...)
eg:
SELECT
identify, 
age,
rank() over(PARTITION BY identify ORDER BY age) rn
FROM person;
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 143621.png)

```mysql
3.1.3、dense_rank() over(partition by 分区字段 order by 排序字段)
	在指定分区内，按照指定字段进行排序，分区内排序会重复，且不会有跳行(1,2,2,3,4，5,5,5,6...)
eg:
SELECT
identify, 
age,
dense_rank() over(PARTITION BY identify ORDER BY age) rn
FROM person;
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 143817.png)

### 	3.2、窗口内向前、向后取值

```mysql
3.2.1、rows between...and...
	unbounded：无界限
	preceding：向前
	following：向后
	current row：当前行
	unbounded preceding：分区内首行
	unbounded following：分区内尾行
eg:
over(partition by 分区字段 order by 排序字段 rows between unbounded preceding and current row)
解释：从分区内首行到当前行
eg:
over(partition by 分区字段 order by 排序字段 rows between 1 preceding and current row)
解释：在分区内，从当前行的前一行到当前行
eg:
over(partition by 分区字段 order by 排序字段 rows between current row and 2 following)
解释：在分区内，从当前行到当前行的后2行
```

```mysql
3.2.2、lag(取值字段,向前取第几行，默认填充值)：向前取值
eg：
SELECT
identify, 
age,
lag(age,1,0) over(PARTITION BY identify ORDER BY age ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) pre_age,
DENSE_RANK() over(PARTITION BY identify ORDER BY age ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) rn
FROM person;
解释：按照身份进行分区，年龄进行排序，在分区内的第一行至当前行的范围内，取前一行的age值，如果前一行没有值，则使用默认值0进行填充
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 152514.png)

```
3.2.3、lead(取值字段,向后取第几行，默认填充值)：向后取值
eg：
SELECT
identify, 
age,
lead(age,1,0) over(PARTITION BY identify ORDER BY age ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING) pre_age,
DENSE_RANK() over(PARTITION BY identify ORDER BY age ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING) rn
FROM person;
解释：按照身份进行分区，年龄进行排序，在分区内的当前行至后三行的范围内，取后一行的age值，如果后一行没有值，则使用默认值0进行填充
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 155657.png)

```mysql
3.2.4、max()取值
eg：
SELECT
identify, 
age,
max(age) over(PARTITION BY identify ORDER BY age ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING) max_age,
DENSE_RANK() over(PARTITION BY identify ORDER BY age ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING) rn
FROM person;
```

![](D:\markdown_picture\picture_store\屏幕截图 2024-09-10 160857.png)