### Mysql的varchar(20)到底可以存储多少个中文字符

这个问题容易产生一些错误的判断，最好是实际测试一下。

#### 新建表

```sql
CREATE TABLE varchar_test (

`id` int(11) NOT NULL ,

`string` varchar(20)

) ENGINE=InnoDB

DEFAULT CHARACTER SET=utf8COLLATE=utf8_general_ci
```

#### 插入表

```sql
INSERT INTO varchar_test (id, string)

VALUES (1, '一二三四五六七八九十');

 

INSERT INTO varchar_test (id, string)

VALUES (2, '一二三四五六七八九十一二三四五六七八九十');

 

INSERT INTO varchar_test (id, string)

VALUES (3, '12345678901234567890');
```

#### 测试结果
|  id   | string  |
|  ----  | ----  |
| 1  | 一二三四五六七八九十 |
| 2  | 一二三四五六七八九十一二三四五六七八九十 |
| 3  | 12345678901234567890 |

如果插入字符超过21个，则报错

```sql
INSERT INTO varchar_test (id, string)

VALUES (3, '123456789012345678901');

[Err] 1406 - Data too long for column'string' at row 1
```

 **可见MySQL的varchar(n)可以存储的中文字符数和英文字符数是一致的，都是n个字符。**