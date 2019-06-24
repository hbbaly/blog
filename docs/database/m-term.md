# 概念解析
## 概念解析
|SQL术语/概念|MongoDB术语/概念|解释/说明|
|---|:--:|:--:|
|database|database|数据库|
|table|collection|数据库表/集合|
|row|document|数据记录行/文档|
|column|field|数据字段/域|
|index|index|索引|
|table joins|表连接,MongoDB不支持|
|primary key|primary key|主键,MongoDB自动将_id字段设置为主键|


一个`mongodb`中可以建立多个数据库。

`MongoDB`的默认数据库为"`db`"，该数据库存储在`data`目录中。

`MongoDB`的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限，不同的数据库也放置在不同的文件中。

"`show dbs`" 命令可以显示所有数据的列表

```sql
show dbs
```
![dbs](../.vuepress/public/img/mongo-2.png  '随意试试')

执行 "`db`" 命令可以显示当前数据库对象或集合

```sql
>db
test
> 
```
运行"`use`"命令，可以连接到一个指定的数据库

```sql
> use local
switched to db local
>db
local
```

有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库
- **admin**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
- **local**: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
- **config**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

## 文档(Document)

```sql
{"site":"www.runoob.com", "name":"菜鸟教程"}
```

文档键命名规范：

- 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
- .和$有特别的意义，只有在特定环境下才能使用。
- 以下划线"_"开头的键是保留的(不是严格要求的)。

注意：

1. 文档中的键/值对是有序的。
2. 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
3. `MongoDB`区分类型和大小写。
4. `MongoDB`的文档不能有重复的键。
5. 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

## 集合

集合就是 `MongoDB` 文档组,集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。
```sql
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.runoob.com","name":"菜鸟教程","num":5}
```

### 合法的集合名
- 集合名不能是空字符串""。
- 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。
- 集合名不能以"`system.`"开头，这是为系统集合保留的前缀。
- 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。

## 元数据

数据库的信息是存储在集合中。它们使用了系统的命名空间：

```js
dbname.system.*
```

在`MongoDB`数据库中名字空间 `<dbname>.system.*` 是包含多种系统信息的特殊集合(Collection)

| 集合命名空间        | 描述           |
| ------------- |:-------------|
| dbname.system.namespaces    | 列出所有名字空间。 |
| dbname.system.indexes     | 列出所有索引。      |
| dbname.system.profile | 包含数据库概要(profile)信息。     |
| dbname.system.users | 列出所有可访问数据库的用户。     |
| dbname.local.sources | 包含复制对端（slave）的服务器信息和状态。    |

## MongoDB 数据类型

| 数据类型        | Are           |
| ------------- |:-------------|
| String      | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer      | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。|
| Boolean     | 布尔值。用于存储布尔值（真/假）。 |
| Double	      | 双精度浮点值。用于存储浮点值。
 |
| Min/Max keys      | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。 |
| Array     | 用于将数组或列表或多个值存储为一个键。 |
| Timestamp      | 时间戳。记录文档修改或添加的具体时间。|
| Object      | 用于内嵌文档。 |
| Null      | 用于创建空值。 |
| Symbol| 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date     | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID    | 对象 ID。用于创建文档的 ID。 |
| Binary Data | 二进制数据。用于存储二进制数据。|
| Code| 代码类型。用于在文档中存储 JavaScript 代码。 |
| Regular expression| 正则表达式类型。用于存储正则表达式。 |


