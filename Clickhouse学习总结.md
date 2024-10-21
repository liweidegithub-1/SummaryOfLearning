# Clickhouse学习总结

## 一、数据类型

```
1、整型：固定长度类型，包括有符号整型和无符号整型
有符号整型范围（-2^(n-1)~2^(n-1)-1）:
Int8:[-128-127](byte)
Int16:[-32768-32767](short)
Int32:[-2147483648-147483647](int)
Int64:[-9223372036854775808-9223372036854775807](long)
无符号整型范围（0~2^(n-1)）
UInt8:[0-255]
UInt16:[0-65535]
UInt32:[0-4294967295]
UInt64:[0-18446744073709551615]

2、浮点型
Float32(float)
Float64(double)

3、布尔型
没有单独的类型来存储布尔型值，可以使用UInt8来表示，限制值为0或1

4、decimal型（有符号浮点数，可在加、减和乘法运算过程中保持精度。对于除法、最低有效数字会被丢弃）
Decimal32(s)，相当于Decimal(9-s,s),有效位数为1~9，s标识小数位长度
Decimal64(s)，相当于Decimal(18-s,s),有效位数为1~18
Decimal128(s)，相当于Decimal(38-s,s),有效位数为1~38
使用场景：一般金额字段、汇率、利率等字段为了保证小数点精度，都使用Decimal进行存储

5、字符串
String：字符串可以任意长度，可以包含任意字符集，包含空字节
FixedString(N)：固定长度N的字符串，N必须是严格的正自然数。当服务端读取长度小于N的字符串的时候，通过在字符串结尾添加空字节来达到N字节长度，当服务端读取长度大于N的字符串的时候，将返回错误消息。
使用场景：名称、文字描述、字符型编码。固定长度的可以保存一些定长的内容，比如一些编码，性别等

6、枚举类型
Enum8：用'String'=Int8对描述
Enum16：用'String'=Int16对描述
eg:
创建一个带有一个枚举Enum8('hello'=1,'world'=2)类型的列
create table t_enum(
	x Enum8('hello'=1, 'world'=2)
)ENGINE=TinyLog;
这个x列只能存储类型定义中列出的值：'hello'或'world'，插入其他值会抛出异常
insert into t_enum8 values('hello')('world');
insert into t_enum8 values('a'); 报错，插入的值没有定义
如果要查看对应行的数值，则必须将Enum值转换为整数类型
select cast(x, 'Int8') from t_enum8;

7、时间类型
Date32接受年-月-日的字符串，如'2019-12-16'
Datetime接受年-月-日 时:分:秒的字符串，如'2019-12-16 10:10:10'
Datetime64接受年-月-日 时:分:秒.亚秒的字符串，如'2019-10-10 10:10:10.10'
日期类型，用两个字节存储，表示从1970-01-01到当前的日期值

8、数组
Array(T)：由T类型元素组成的数组
T可以是任意类型，包含数组类型，但不推荐使用多维数组，在MergeTree表中存储多维数组
创建数组方式1，使用array函数
select array(1,3) as x, toTypeName(x);
创建数组方式2，使用方括号
select [1,2] as x, toTypeName(x);

9、Map
Map(key type,value type)可以存储key:value键值对类型的数据
eg：
CREATE TABLE t_map(
	m Map(String, Int32)
)ENGINE=Memory;
INSERT INTO t_map values({'key1': 1, 'key2': 2})({'key3': 3});
查看key为'key1'的所有值，如key没有匹配则返回0
SELECT m['key1'] FROM t_map; 
查看所有的key
SELECT m.keys FROM t_map;
查看所有的value
SELECT m.values FROM t_map;

10、元组
Tuple(t1,t2,...)
不能在表中进行存储，可以用于临时列分组
eg：
SELECT tuple(1,3,'a') AS x, toTypeName(x);
将tuple类型转换为map类型
SELECT CAST((['a','b','c'], [1,2,3]),'Map(String, Int8)') AS x, toTypeName(x);
```

