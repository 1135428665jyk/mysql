# 高性能Mysql

## 1. Mysql架构及历史











## 2. Mysql基准测试









## 3. 服务性能剖析











## 4. Schema和数据库优化

### varchar和char类型

```bash
varchar类型是可变长度，比定长节省空间，使用1个或者2个额外字节记录字符串的长度，如果列的长度小于255，则采用1个字节，如果大于255使用2个字节，varchar节省的存储空间，如果一行占用的空间比原来长，导致做额外的工作，MyIsam拆分不同的片段进行存储，Innodb则需要分裂页来放进页内。如果Innodb过长，会把varchar存储为BLOB.
Char类型是定长的，当存储Char值时，Mysql会删除所有的末尾空格，适合存储Md5
Varchar(5)和varchar(200)存储'hello'开销是一样的，短的有更大的优势，最好策略是分配真正需要的空间



```

### Blob和text类型

```bash
Blob和Text都是存储最大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储，
Blob存储二进制，没有排序规则或字符集。
Text有字符集和排序规则。
Memory引擎不支持Blob和Text类型，如果使用了Blob和Text存储引擎使用MyIsam,临时表的大小超过max_heap_table_size或者tmp_table_size，则内存临时表会转换成MyIsam磁盘临时表
使用Eplain执行计划Extra中包括了'Using tempory'说明使用了临时表。
```

### 使用枚举Enum代替字符串类型

```bash
枚举类型非常紧凑，会根据列表值的数量压缩到一个或两个字节中，保存为整数，也可以使用Field()函数显式指定排序顺序，导致mysql无法利用索引，消除排序。
create table enum_test(e Enum('fish','dog','apple') Not Null);
```

### 日期和时间类型

```bash
Datetime 范围从1001-9999，与时区无关
TimeStamp 范围从1979-2038,只使用4个字节的存储空间
```

### 位类型操作

```bash
CREATE TABLE bittest(a bit(8))
INSERT INTO bittest VALUES(b'011101');
select a,a+0 from bittest;
返回结果011101 29
最好避免使用bit类型
Set类型
```

### 选择标识符

```bash
随机字符串需要多加注意，插入值会随机写道索引的不同位置，所以Insert语句比较慢，会导致页分裂，磁盘随机访问，聚簇索引产生聚簇索引碎片，
select查询会变慢，因为相邻的行分布在磁盘和内存不同的位置。
随机值导致对所有类型的数据的查询效果都很差，会使得缓存工作的局部性原理失效。
```

### 特殊类型

```bash
INET_ATON Ip类型转换
INET_NTOA
```

### 范式和反范式

```bash
范式：数据不冗余；
反范式：数据冗余，可以存储在多个地方；
范式优点：更新操作快，很少有重复的数据，只需要更改更少的数据，范式表更小，操作快
范式缺点：需要做关联，索引策略可能无效
反范式优点：避免关联查询，单独的表可以使用更有效的策略。
```

| Employee | Department  | Head  |
| -------- | ----------- | ----- |
| Jones    | Accounting  | Jones |
| Smith    | Engineering | Smith |
| Brown    | Accounting  | Jones |
| Green    | Engineering | Smith |

范式设计如表：

表1

| Employee | Department  |
| -------- | ----------- |
| Jones    | Accounting  |
| Smith    | Engineering |
| Brown    | Accounting  |
| Green    | Engineering |

表2

| Department  | Head  |
| ----------- | ----- |
| Accounting  | Jones |
| Engineering | Smith |

### 混合使用范式和反范式

```bash
使用部分范式，缓存表或者其他技巧
有时提升性能最好的方法时在同一张表中保存衍生的冗余数据。有时需要创建一张完全独立的汇总表或缓存表
```

### 物化视图

```bash
物化视图是预先计算并且存储在磁盘上的表，可以通过各样的策略刷新和更新
最常见的反范式的
```

