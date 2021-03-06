# 《MySQL必知必会》读书笔记
部分补充内容参考自网络

## MySQL数据类型
### 串数据类型
CHAR 1~255个字符的定长串，长度必须在创建时指定，否则MySQL假定为CHAR(1)
ENUM 接收最多64K个串组成的一个预定义集合的某个串
LONGTEXT 与TEXT相同，但最大长度为4GB
MEDIUMTEXT 与TEXT相同，但最大长度为16K
SET 接受最多64个串组成的一个预定义集合的零个或多个串
TEXT 最大长度为64K的变长文本
TINYTEXT 与TEXT相同，但最大长度为255字节
VARCHAR 长度可变，最多不超过255字节。如果在创建时指定为VARCHAR(n)，则可存储0~n个字符的变长串
数值数据类型
BIT 					位字段，1~64位
BIGINT 				整数值，8个字节，-2^63到2^63-1的整型数据
BOOLEAN (BOOL) 	布尔值，为0或1
DECIMAL (DEC) 		精度可变的浮点数
DOUBLE 			双精度浮点数
FLOAT 				单精读浮点数
INT (INTEGER) 		整数值，4个字节，-2^31到2^31–1的整型数据
MEDIUMINT 		整数值，3个字节
REAL 				4字节的浮点值
SMALLINT 			整数值，2个字节
TINYINT 			整数值，1个字节
日期和时间数据类型
DATE 				表示1000-01-01~9999-12-31的日期，格式为YYYY-MM-DD
DATETIME 			DATE和TIME的组合
TIMESTAMP 		功能和DATETIME相同，但范围较小
TIME 				格式为HH:MM:SS
YEAR 				1970~2069年
二进制数据类型
BLOB 				最大长度为64K
MEDIUMBLOB 		最大长度为16M
LONGBLOB 		最大长度为4GB
TINYBLOB 			最大长度为255字节

### MySQL的语句执行顺序
MySQL的语句一共分为11步，如下图所标注的那样，最先执行的总是FROM操作，最后执行的是LIMIT操作。其中每一个操作都会产生一张虚拟的表，这个虚拟的表作为一个处理的输入，只是这些虚拟的表对用户来说是透明的，但是只有最后一个虚拟的表才会被作为结果返回。如果没有在语句中指定某一个子句，那么将会跳过相应的步骤。
1.	FORM: 对FROM的左边的表和右边的表计算笛卡尔积。产生虚表VT1
2.	ON: 对虚表VT1进行ON筛选，只有那些符合<join-condition>的行才会被记录在虚表VT2中。
3.	JOIN： 如果指定了OUTER JOIN（比如left join、 right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。
4.	WHERE： 对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。
5.	GROUP BY: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.
6.	CUBE | ROLLUP: 对表VT5进行cube或者rollup操作，产生表VT6.
7.	HAVING： 对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。
8.	SELECT： 执行select操作，选择指定的列，插入到虚拟表VT8中。
9.	DISTINCT： 对VT8中的记录进行去重。产生虚拟表VT9.
10.	ORDER BY: 将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10.
11.	LIMIT：取出指定行的记录，产生虚拟表VT11, 并将结果返回。

### MySQL语句总结
USE crashcourse;							# 选择数据库

SHOW DATABASES;						# 返回可用数据库的列表

SHOW TABLES;							# 获得一个数据库内的表的列表

SHOW COLUMNS FROM customers;			# 显示表的列

DESCRIBE customers;						# 显示表的列

SHOW STATUS;							# 显示服务器状态信息

SHOW CREATE DATABASE crashcourse;		# 显示创建特定数据库的MySQL语句

SHOW CREATE TABLE orders;				# 显示创建特定表的MySQL语句

SHOW GRANTS;							# 显示授权用户的安全权限

SHOW ERRORS;							# 显示服务器错误信息

SHOW WARNINGS;						# 显示服务器警告信息

### 查询单个列
SELECT prod_name 
FROM products;					

### 查询多个列
SELECT prod_id, prod_name, prod_price
FROM products;

### 查询所有列
SELECT *
FROM products;

### 只返回不同的值
SELECT DISTINCT vend_id
FROM products;

DISTINCT关键字应用于所有列（即所有列的值都不相同），而不仅仅是前置它的列。如：
SELECT DISTINCT vend_id, prod_price FROM products;
将返回：
	

### 返回不多于5行（小于等于）
SELECT prod_name
FROM products
LIMIT 5;

### 返回从第6行开始的5行（行号从0开始）
SELECT prod_name
FROM products
LIMIT 5,5;

### 返回从第6行开始的5行（LIMIT的一种替代语法）
SELECT prod_name
FROM products
LIMIT 5 OFFSET 5;

### 使用完全限定的列名
SELECT products.prod_name
FROM products;

### 使用完全限定的表名
SELECT products.prod_name
FROM crashcourse.products;

### 指定按列排序
SELECT prod_name
FROM products
ORDER BY prod_name;
用非检索的列排序数据也是可以的。排序不区分大小写，比如A和a是相同的。

### 按多个列排序
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price, prod_name;

### 指定排序方向
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC;
默认升序（ASC），可以指定降序（DESC）。DESC只应用到直接位于其前面的列名，如果想对多个列进行降序排序，必须对每个列指定DESC关键字。

### 找出一个列中最高或最低的值
SELECT prod_price
FROM products
ORDER BY prod_price DESC
LIMIT 1;

### 使用过滤条件
SELECT prod_name, prod_price
FROM products
WHERE prod_price = 2.50;

WHERE子句操作符
=	Equality
<>	Nonequality
!=	Nonequality
<	Less than
<=	Less than or equal to
>	Greater than
>=	Greater than or equal to
BETWEEN	Between two specified values
 
### 检查单个值
SELECT prod_name, prod_price
FROM products
WHERE prod_name = 'fuses';
在执行匹配时不区分大小写

### 使用BETWEEN操作符
SELECT prod_name, prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;
BETWEEN...AND相当于>=并且<=，而不是>并且<。

### 空值检查
SELECT prod_name
FROM products
WHERE prod_price IS NULL;
无法通过过滤条件“选择出不具有特定值的行”来返回具有NULL值的行，因为“未知”具有特殊的含义，数据库不知道它们是否匹配，所以在匹配过滤或不匹配过滤时，不返回它们。即NULL值既非等于，也非不等于。

### 使用多个过滤条件（AND）
SELECT prod_id, prod_price, prod_name
FROM products
WHERE vend_id = 1003 AND prod_price <= 10;

### 使用多个过滤条件（OR）
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003;

### 组合使用AND和OR
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;
SQL语句在处理OR操作符前优先处理AND操作符。

### 使用括号来组合过滤条件
SELECT prod_name, prod_price
FROM products
WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;

### 指定条件范围
SELECT prod_name, prod_price
FROM products
WHERE vend_id IN (1002,1003)
ORDER BY prod_name;
IN的最大优点是可以包含其他SELECT语句。

### 使用取反条件
SELECT prod_name, prod_price
FROM products
WHERE vend_id NOT IN (1002,1003)
ORDER BY prod_name;

### 搜索以指定字符串开头的记录
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%';

### 搜索包含指定字符串的记录
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE '%anvil%';

### 搜索以指定字符串开始、以指定字符串结束的记录
SELECT prod_name
FROM products
WHERE prod_name LIKE 's%e';
%表示任何字符出现任意次数（0、1或多个），是否区分大小写取决于MySQL的配置。
尾空格可能会干扰通配符匹配，例如“anvil ”（最后有一个空格），则LIKE “%anvil”将不会匹配，因为在最后有一个多余的字符。
通配符不能匹配NULL值，即使使用‘%’也不行。

### 匹配单个字符
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE '_ ton anvil';
注意：把通配符置于搜索模式的开始处，搜索起来是最慢的。

### 使用正则表达式进行基本的字符串匹配
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;

LIKE与REGEXP有一个重要区别：LIKE匹配整个列，而REGEXP则在列值内进行匹配。
比如有一个记录的prod_name为“JetPack 1000”，则使用REGEXP ‘1000’，将能匹配到该记录，而使用LIKE ‘1000’将匹配不到。
MySQL中的正则表达式不区分大小写，即大小写都会匹配。为了区分大小写，可使用BINARY关键字，如：WHERE prod_name REGEXP BINGARY ‘JetPack .000’。

### 使用正则表达式匹配单个字符（.）
SELECT prod_name
FROM products
WHERE prod_name REGEXP '.000'
ORDER BY prod_name;

### 使用正则表达式进行OR匹配
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000|2000'
ORDER BY prod_name;

### 匹配几个字符之一
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;
[123]匹配字符1、2或3，而[^123]则匹配除了这几个字符以外的任何东西。

### 匹配范围
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[1-5] Ton'
ORDER BY prod_name;

### 匹配特殊字符（\\）
SELECT vend_name
FROM vendors
WHERE vend_name REGEXP '\\.'
ORDER BY vend_name;
这里要求两个反斜杠，MySQL自己解释一个，正则表达式库解释另一个。

可以使用预定义的字符类：
[:alnum:]		Any letter or digit, (same as [a-zA-Z0-9])
[:alpha:]		Any letter (same as [a-zA-Z])
[:blank:]		Space or tab (same as [\\t ])
[:cntrl:]		ASCII control characters (ASCII 0 tHRough 31 and 127)
[:digit:]		Any digit (same as [0-9])
[:graph:]		Same as [:print:] but excludes space
[:lower:]		Any lowercase letter (same as [a-z])
[:print:]		Any printable character
[:punct:]		Any character that is neither in [:alnum:] nor [:cntrl:]
[:space:]		Any whitespace character including the space (same as [\\f\\n\\r\\t\\v ])
[:upper:]		Any uppercase letter (same as [A-Z])
[:xdigit:]		Any hexadecimal digit (same as [a-fA-F0-9])
 
正则表达式重复元字符：
*		0 or more matches
+		1 or more matches (equivalent to {1,})
?		0 or 1 match (equivalent to {0,1})
{n}		Specific number of matches
{n,}		No less than a specified number of matches
{n,m}	Range of matches (m not to exceed 255)
 
SELECT prod_name
FROM products
WHERE prod_name REGEXP '\\([0-9] sticks?\\)'
ORDER BY prod_name;
将返回：

?匹配它前面的任何字符的0次或1次出现。

### 匹配连在一起的4位数字
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[[:digit:]]{4}'
ORDER BY prod_name;


定位元字符：
^		Start of text
$		End of text
[[:<:]]	Start of word
[[:>:]]	End of word

SELECT prod_name
FROM products
WHERE prod_name REGEXP '^[0-9\\.]'
ORDER BY prod_name;

通过用^开始每个表达式，用$结束每个表达式，可以使REGEXP的作用与LIKE一样。

### 拼接字段（将值连接到一起构成单个值）
SELECT Concat(vend_name, ' (', vend_country, ')')
FROM vendors
ORDER BY vend_name;


### 去除空格
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
FROM vendors
ORDER BY vend_name;
此外还有可用函数LTrim()和Trim()

### 使用别名
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS
vend_title
FROM vendors
ORDER BY vend_name;

### 进行算术计算
SELECT prod_id,
       quantity,
       item_price,
       quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
可用的算术运算符：+、-、*、/

文本处理函数：
Left()		Returns characters from left of string
Length()		Returns the length of a string
Locate()		Finds a substring within a string
Lower()		Converts string to lowercase
LTrim()		Trims white space from left of string
Right()		Returns characters from right of string
RTrim()		Trims white space from right of string
Soundex()	Returns a string's SOUNDEX value
SubString()	Returns characters from within a string
Upper()		Converts string to uppercase
 
### 转换查询结果为大写
SELECT vend_name, UPPER(vend_name) AS vend_name_upcase
FROM vendors
ORDER BY vend_name;


### 进行发音类似的匹配
SELECT cust_name, cust_contact
FROM customers
WHERE Soundex(cust_contact) = Soundex('Y Lie');


日期时间处理函数
AddDate()		Add to a date (days, weeks, and so on)
AddTime()		Add to a time (hours, minutes, and so on)
CurDate()		Returns the current date
CurTime()		Returns the current time
Date()			Returns the date portion of a date time
DateDiff()		Calculates the difference between two dates
Date_Add()		Highly flexible date arithmetic function
Date_Format()		Returns a formatted date or time string
Day()			Returns the day portion of a date
DayOfWeek()		Returns the day of week for a date
Hour()			Returns the hour portion of a time
Minute()			Returns the minute portion of a time
Month()			Returns the month portion of a date
Now()			Returns the current date and time
Second()			Returns the second portion of a time
Time()			Returns the time portion of a date time
Year()			Returns the year portion of a date
 
### 只使用Date部分进行比较（即忽略Time部门）
SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) = '2005-09-01';

### 执行指定年、月份的查询-使用BETWEEN
SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';

### 执行指定年、月份的查询-使用函数
SELECT cust_id, order_num
FROM orders
WHERE Year(order_date) = 2005 AND Month(order_date) = 9;

聚集函数
AVG()		Returns a column's average value
COUNT()	Returns the number of rows in a column
MAX()		Returns a column's highest value
MIN()		Returns a column's lowest value
SUM()		Returns the sum of a column's values
 
### 求平均值
SELECT AVG(prod_price) AS avg_price
FROM products;
AVG函数忽略列值为NULL的行。

### 统计行的数目，不管表列中包含的是NULL还是非空值
SELECT COUNT(*) AS num_cust
FROM customers;

### 对特定列中具有值的行进行计数，忽略NULL值的行
SELECT COUNT(cust_email) AS num_cust
FROM customers;

### 求指定列中的最大值
SELECT MAX(prod_price) AS max_price
FROM products;

### 求指定列中的最小值
SELECT MIN(prod_price) AS min_price
FROM products;

### 返回指定列值的和
SELECT SUM(quantity) AS items_ordered
FROM orderitems
WHERE order_num = 20005;

### 合计计算值的和
SELECT SUM(item_price*quantity) AS total_price
FROM orderitems
WHERE order_num = 20005;

所有聚集函数都可以指定ALL参数或DISTINCT参数，ALL是默认行为。
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;
DISTINCT必须使用列名，不能用于计算或者表达式。

### 分组：把数据分为多个逻辑组，以便能对每个组进行聚集计算。
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;
GROUP BY子句可以包含任意数目的列，因此能对分组进行嵌套，数据将在最后规定的分组上进行汇总。
GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数），如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式，不能使用别名。
除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
如果分组列中具有NULL值，则NULL将作为一个分组返回，如果列中有多行NULL值，它们将分为一组。

### 得到每个分组及每个分组汇总级别的值
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id WITH ROLLUP;


### 过滤分组
SELECT cust_id, COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;

### 子查询
SELECT cust_id
FROM orders
WHERE order_num IN (SELECT order_num
                    FROM orderitems
                    WHERE prod_id = 'TNT2');
子查询总是从内向外处理，对于能嵌套的子查询的数目没有限制。

### 将子查询作为计算字段
SELECT cust_name,
       cust_state,
       (SELECT COUNT(*)
        FROM orders
        WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;

### 返回笛卡尔积结果（即没有任何联结条件的联结）
SELECT vend_name, prod_name, prod_price
FROM vendors, products
ORDER BY vend_name, prod_name;

### 指定联结条件
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name;
等价于：
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
 ON vendors.vend_id = products.vend_id;

### 联结多个表
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
  AND orderitems.prod_id = products.prod_id
  AND order_num = 20005;

### 在联结中使用表别名
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
  AND oi.order_num = o.order_num
  AND prod_id = 'TNT2';
表别名只在查询执行中使用，与列别名不一样，表别名不返回到客户机。

### 自联结（同一张表使用不同的别名，自己联结自己）
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
  AND p2.prod_id = 'DTNTR';

### 自然联结
SELECT c.*, o.order_num, o.order_date,
       oi.prod_id, oi.quantity, oi.item_price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
  AND oi.order_num = o.order_num
  AND prod_id = 'FB';

自然联结是一种特殊的内联结：在结果中把重复的属性列去掉。其实现需要自己来保证，一般是通过对表使用通配符，对其他表的列使用明确的子集来完成。
上例中，customer表和orders表都分别有一个“cust_id”字段，orderitems表和orders表都分别有一个order_num字段，但是查询结果中，所有这些字段都只会出现一次。
自然联结只能是同名属性的等值联结，因此可以省去联结条件。内联结（等值联结）可以指定不同列名的联结。

### 外部联结（包含没有关联行的那些行）
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
 ON customers.cust_id = orders.cust_id;
左外联接：包括左边表的所有行。


### 组合查询（执行多个查询，并将结果作为单个查询结果返回）
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);

等价于：
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
  OR vend_id IN (1001,1002);

任何具有多个WHERE子句的SELECT语句都可以作为一个组合查询给出。
UNION中的每个查询必须包含相同的列、表达式或聚集函数。
UNION查询可以应用于不同的表，列数据类型必须兼容。
UNION从查询结果集中自动去除重复的行，如果想返回所有匹配行，可以使用UNION ALL。

###　组合排序
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002)
ORDER BY vend_id, prod_price;
在用UNION组合查询时，只能使用一条ORDER BY子句，且必须出现在最后一条SELECT语句之后。

MyISAM支持全文搜索，而InnoDB不支持。
为了进行全文本搜索，必须索引被搜索的列。当对表列进行适当的设计后，MySQL会自动进行所有的索引和重新索引。

### 启用全文本搜索支持
CREATE TABLE productnotes
(
  note_id    int           NOT NULL AUTO_INCREMENT,
  prod_id    char(10)      NOT NULL,
  note_date datetime       NOT NULL,
  note_text  text          NULL ,
  PRIMARY KEY(note_id),
  FULLTEXT(note_text)
) ENGINE=MyISAM;
MySQL将自动维护索引。

### 使用全文本搜索
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');

### 查看全文本搜索等级值
SELECT note_text,
       Match(note_text) Against('rabbit') AS rank
FROM productnotes;

### 使用查询扩展（能找出可能相关的结果，即使它们并不精确包含所查找的词）
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION);

第2行与anvils无关，但因为它包含第1行中的两个词customer和recommend，所以也被检索出来。

### 布尔文本搜索
SELECT note_text
FROM productnotes
WHERE MATCH(note_text) AGAINST('heavy' IN BOOLEAN MODE);
布尔方式即使没有定义FULLTEXT索引也可以使用。

### 匹配包含heavy但不包含任意以rope开始的词的行
 SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);

全文本搜索布尔操作符
+		Include, word must be present.
-		Exclude, word must not be present.
>		Include, and increase ranking value.
<		Include, and decrease ranking value.
()		Group words into subexpressions (allowing them to be included, excluded, ranked, and so forth as a group).
~		Negate a word's ranking value.
*		Wildcard at end of word.
""		Defines a phrase (as opposed to a list of individual words, the entire phrase is matched for inclusion or exclusion).
 
### 包含词rabbit和bait的行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+rabbit +bait"' IN BOOLEAN MODE);

### 包含rabbit和bait中至少一个的行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);

### 包含“rabbit bait”这个字符串的行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);

### 匹配safe和combination，且降低后者的等级
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+safe +(<combination)' IN BOOLEAN
MODE);

### 插入数据（不带列名，依赖表中列定义的次序）
INSERT INTO Customers
VALUES(NULL,
   'Pep E. LaPew',
   '100 Main Street',
   'Los Angeles',
   'CA',
   '90046',
   'USA',
   NULL,
   NULL);

### 带列名插入数据（比较安全的方式）
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country,
   cust_contact,
   cust_email)
VALUES('Pep E. LaPew',
   '100 Main Street',
   'Los Angeles',
   'CA',
   '90046',
   'USA',
   NULL,
   NULL);
可以在插入数据的时候省略列，省略的列必须满足以下某个条件：
该列定义为允许NULL
在表定义中给出默认值

### 降低INSERT语句的优先级
INSERT LOW_PRIORITY INTO...

###　插入多个行
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES(
        'Pep E. LaPew',
        '100 Main Street',
        'Los Angeles',
        'CA',
        '90046',
        'USA'
     ),
      (
        'M. Martian',
        '42 Galaxy Way',
        'New York',
        'NY',
        '11213',
        'USA'
   );

### 插入查询到的数据
INSERT INTO customers(cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
SELECT cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
FROM custnew;
MySQL不关心SELECT返回的列名，只使用对应位置的值进行插入。

### 更新字段
UPDATE customers
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;

### 更新多个列
UPDATE customers
SET cust_name = 'The Fudds',
    cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;

### 更新多行时忽略错误
UPDATE IGNORE customers ...

### 删除一行
DELETE FROM customers
WHERE cust_id = 10006;

### 更快地删除表中所有的行
TRUNCATE TABLE 
TRUNCATE语句实际删除原来的表，并重建一个新表，因此速度更快

### 创建表
CREATE TABLE customers
(
  cust_id      int       NOT NULL AUTO_INCREMENT,
  cust_name    char(50)  NOT NULL ,
  cust_address char(50)  NULL ,
  cust_city    char(50)  NULL ,
  cust_state   char(5)   NULL ,
  cust_zip     char(10)  NULL ,
  cust_country char(50)  NULL ,
  cust_contact char(50)  NULL ,
  cust_email   char(255) NULL ,
  PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
每个表只允许有一个AUTO_INCREMENT，而且它必须被索引。
如果在INSERT语句中为AUTO_INCREMENT的列指定了一个值，该值将被用来替代自动生成的值，且后续的增量将从该值开始。

### 返回最后一个AUTO_INCREMENT的值
SELECT last_insert_id();

### 指定默认值
CREATE TABLE orderitems
(
  order_num  int          NOT NULL ,
  order_item int          NOT NULL ,
  prod_id    char(10)     NOT NULL ,
  quantity   int          NOT NULL  DEFAULT 1,
  item_price decimal(8,2) NOT NULL ,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;

引擎类型：
InnoDB，不支持全文本搜索；
MEMORY，数据存储在内存，速度很快，适合于临时表
MyISAM，MySQL的默认引擎，支持全文本搜索，但不支持事务处理
外键不能跨引擎。

### 给表添加一个列
ALTER TABLE vendors
ADD vend_phone CHAR(20);

### 删除一个列
ALTER TABLE Vendors
DROP COLUMN vend_phone;

### 定义一个外键
ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);

### 删除表
DROP TABLE customers2;

### 重命名表
RENAME TABLE customers2 TO customers;

### 重命名多个表
RENAME TABLE backup_customers TO customers,
             backup_vendors TO vendors,
             backup_products TO products;

### 创建视图
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
  AND orderitems.order_num = orders.order_num;

视图的规则和限制：
（1）视图名必须唯一，不能与已有的视图名或表名相同
视图可以嵌套，即利用从其他视图检索到的数据来构造一个视图
如果视图的SELECT语句中有ORDER BY，那么该视图中的ORDER BY将被覆盖
视图不能索引，也不能有关联的触发器
视图可以和表一起使用，比如联结

### 使用视图重新格式化检索出的数据
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
       AS vend_title
FROM vendors
ORDER BY vend_name;


更新一个视图将更新其基表，如果对视图增加或删除行，实际是对其基表增加或删除行。
如果不能正确地确定被更新的基数据，则不允许更新，这意味着如果视图定义中有以下操作则不能进行视图的更新：
分组，使用GROUP BY和HAVING
联结
子查询
UNION
聚集函数
DISTINCT
计算得到的列

### 执行存储过程
CALL productpricing(@pricelow,
                    @pricehigh,
                    @priceaverage);

### 显示变量值
SELECT @pricehigh, @pricelow, @priceaverage;

### 创建存储过程
CREATE PROCEDURE productpricing()
BEGIN
   SELECT Avg(prod_price) AS priceaverage
   FROM products;
END;

### 删除存储过程
DROP PROCEDURE productpricing;

### 创建带参数的存储过程
CREATE PROCEDURE productpricing(
   OUT pl DECIMAL(8,2),
   OUT ph DECIMAL(8,2),
   OUT pa DECIMAL(8,2)
)
BEGIN
   SELECT Min(prod_price)
   INTO pl
   FROM products;
   SELECT Max(prod_price)
   INTO ph
   FROM products;
   SELECT Avg(prod_price)
   INTO pa
   FROM products;
END;

### 建立智能存储过程
-- Name: ordertotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable, 1 if taxable
--             ototal = order total variable

CREATE PROCEDURE ordertotal(
   IN onumber INT,
   IN taxable BOOLEAN,
   OUT ototal DECIMAL(8,2)
) COMMENT 'Obtain order total, optionally adding tax'
BEGIN

   -- Declare variable for total
   DECLARE total DECIMAL(8,2);
   -- Declare tax percentage
   DECLARE taxrate INT DEFAULT 6;

   -- Get the order total
   SELECT Sum(item_price*quantity)
   FROM orderitems
   WHERE order_num = onumber
   INTO total;

   -- Is this taxable?
   IF taxable THEN
      -- Yes, so add taxrate to the total
      SELECT total+(total/100*taxrate) INTO total;
   END IF;

   -- And finally, save to out variable
   SELECT total INTO ototal;

END;

### 使用
CALL ordertotal(20005, 0, @total);
SELECT @total;

游标只能用于存储过程和函数。

### 创建游标
CREATE PROCEDURE processorders()
BEGIN
   DECLARE ordernumbers CURSOR
   FOR
   SELECT ordernum FROM orders;
END;

### 打开游标
OPEN ordernumbers;

### 关闭游标
CLOSE ordernumbers;

### 使用游标数据
CREATE PROCEDURE processorders()
BEGIN

   -- Declare local variables
   DECLARE o INT;

   -- Declare the cursor
   DECLARE ordernumbers CURSOR
   FOR
   SELECT order_num FROM orders;

   -- Open the cursor
   OPEN ordernumbers;

   -- Get order number
   FETCH ordernumbers INTO o;

   -- Close the cursor
   CLOSE ordernumbers;

END;

触发器名必须在每个表中唯一。
视图不支持触发器。
每个表、每个事件，每次只允许执行一个触发器。因此每个表最多只支持6个触发器。
如果BEFORE触发器失败，将不执行请求的操作。如果BEFORE触发器失败，或者语句本身失败，将不执行AFTER触发器。

### 创建触发器
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added';

### 删除触发器
DROP TRIGGER newproduct;

### INSERT触发器
定义
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num;

使用
INSERT INTO orders(order_date, cust_id)
VALUES(Now(), 10001);
在INSERT触发器代码中可以引用一个名为NEW的虚拟表来访问被插入的行。NEW中的值也可以被更新，对于AUTO_INCREMENT列，NEW在INSERT执行之前包含0，在INSERT执行之后包含新的自动生成值。

### DELETE触发器
CREATE TRIGGER deleteorder BEFORE DELETE ON orders
FOR EACH ROW
BEGIN
   INSERT INTO archive_orders(order_num, order_date, cust_id)
   VALUES(OLD.order_num, OLD.order_date, OLD.cust_id);
END;
在DELETE触发器代码内，可以引用一个名为OLD的虚拟表访问被删除的行。OLD中的值全是只读的，不能被更新。

### UPDATE触发器
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = Upper(NEW.vend_state);
在UPDATE触发器代码中可以引用一个名为OLD的虚拟表访问更新前的值，引用一个名为NEW的表访问新更新的值。在BEFORE UPDATE触发器中，NEW中的值也可能被更新。OLD中的值全是只读的，不能更新。

MySQL不支持从触发器内调用存储过程。

### 开始事务
START TRANSACTION

### 回滚事务
SELECT * FROM ordertotals;
START TRANSACTION;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertotals;
不能回滚SELECT、DROP或CREATE操作。

### 提交事务
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
一般的MySQL语句都是隐含提交的，即提交操作是自动进行的。

### 使用保留点（用于部分提交或回退）
SAVEPOINT delete1;

### 回退到保留点
ROLLBACK TO delete1;

### 更改默认的提交行为：设置MySQL不自动提交更改
SET autocommit=0;
autocommit标志针对每个链接，而不是整个服务器。

字符集：字母和符号的集合
编码：某个字符集成员的内部表示
校对：规定字符如何比较的指令（比如是否区分大小写）
字符集很少是服务器范围的设置，不同的表甚至不同的列都可能需要不同的字符集，而且两者都可以在创建表时指定。

### 查看所支持的字符集
SHOW CHARACTER SET;

### 查看所支持的校对
SHOW COLLATION;

### 确定当前所用的字符集和校对
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';

### 创建表时指定字符集
CREATE TABLE mytable
(
   columnn1   INT,
   columnn2   VARCHAR(10)
) DEFAULT CHARACTER SET hebrew COLLATE hebrew_general_ci;

### 指定列的字符集
CREATE TABLE mytable
(
   columnn1   INT,
   columnn2   VARCHAR(10),
   column3    VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;

### 在SELECT语句中指定校对，以使用与建表时不同的校对，比如临时支持区分大小写
SELECT * FROM customers
ORDER BY lastname, firstname COLLATE latin1_general_cs;
如果确实有必要，可以使用Cast()或Convert()函数来对串进行字符集转换。

### 查看MySQL用户账号信息
USE mysql;
SELECT user FROM user;
user表包含所有用户账号

### 创建用户账户
CREATE USER ben IDENTIFIED BY 'p@$$w0rd';

###　重命名用户账户
RENAME USER ben TO bforta;

### 删除用户账户
DROP USER bforta;

### 查看赋予用户账户的权限
SHOW GRANTS FOR bforta;

### 授予权限
GRANT SELECT ON crashcourse.* TO beforta;

### 撤销权限
REVOKE SELECT ON crashcourse.* FROM beforta;
GRANT和REVOKE可以在几个层次上控制访问权限：
整个服务器，GRANT ALL和REVOKE ALL
整个数据库，ON database.*
特定的表，ON database.table
特定的列
特定的存储过程

### 同时授予多个权限
GRANT SELECT, INSERT ON crashcourse.* TO beforta;

### 更改密码
SET PASSWORD FOR bforta = Password('n3w p@$$w0rd');

### 更改自己的密码
SET PASSWORD = Password('n3w p@$$w0rd');

### 检查表键是否正确
ANALYZE TABLE orders;

### 检查表
CHECK TABLE orders, orderitems