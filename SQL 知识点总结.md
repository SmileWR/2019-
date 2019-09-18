#SQL 知识点总结
###group by
Group by中select 字段的限制
在select指定的字段要么就要包含在Group By语句的后面，作为分组的依据；要么就要被包含在聚合函数中。

###Having与Where的区别
where 子句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，where条件中不能包含聚组函数，使用where条件过滤出特定的行。
having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数，使用having 条件过滤出特定的组，也可以使用多个分组标准进行分组。
**示例**
```
select 类别, sum(数量) as 数量之和 from A
group by 类别
having sum(数量) > 18
```
**示例：**Having和Where的联合使用方法
```
select 类别, SUM(数量)from A
where 数量 > 8
group by 类别
having SUM(数量) > 10
```

### Limit 

**SELECT * FROM table  LIMIT [offset] rows | rows** 
- 当省略offset的时候，offset作为0处理，表示提取查询到的前rows条数据；
- 当offset>=0时候，表示提取查询到的从offset开始的rows条数据；此时如果rows<0表示提取查询到的从offset开始的所有数据
- 当offset<0的时候，表示提取查询到的除出后rows条数据的所有数据，即剔除last row-rows到last rows之间的-rows条数据
- 另外，如果rows大于实际查询的数据条数，则取rows为实际查询的数据条数。

**简单使用**
```
#检索记录行6-15
select * from table limit 5,10;

#检索记录行96-last
select * from table limit 96,-1;

```

**跳过前三行，取接下来的5行**

```
#使用mysql可以使用limit offset子句较短的形式
select employee_id,first_name, last_name 
from employees
order by first_name
limit 3,5;
```

**获取具有最高或最低值的前N行**

```
#获得薪资最高的前五名员工
select employee_id,first_name, last_name 
from employees
order by salary desc
limit 5;
```

**获取具有第N个最高值的行**

```
#获取公司薪水第二高的员工
SELECT 
    employee_id, first_name, last_name, salary
FROM
    employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

- ORDER BY子句按工资降序对员工进行排序。 LIMIT 1 OFFSET 1子句从结果集中获取第二行。
- 此查询的假设是每个员工都有不同的薪水。 如果有两名员工拥有相同的最高薪水，那么它将会失败。 此外，如果有两个或更多具有相同第二高薪的员工，则查询只返回第一个。
- 用如下**子查询**的方式

```
SELECT 
    employee_id, first_name, last_name, salary
FROM
    employees
WHERE
    salary = (SELECT DISTINCT
            salary
        FROM
            employees
        ORDER BY salary DESC
        LIMIT 1 , 1);
```

### isnull()、nvl()、ifnull()、coalesce()、nullif()、iif()

**isnull()**——微软sql server
- ISNULL(Expression1,Expression2):给定两个参数Expression1和Expression2，如果Expression1是NULL，那么返回Expression2，否则返回Expression1。
- 对空值处理不起作用
- Select ISNULL(NULL,1)返回1，Select ISNULL(1,2)返回1
**nvl()**——oracle
- 同isnull()
- oracle中也可以使用 case when....then....else......end

**ifnull()**——mysql
- 同isnull()

**coalesce()**——mysql
- 同ifnull()
- 对空值处理和null值都起作用
- coalesce相比nvl优点是，coalesce中**参数**可以有**多个**，而**nvl()**中参数就只有**两个**
- coalesce函数表示可以返回参数中的第一个非空表达式，当你有N个参数时选取第一个非空值（从左到右）
```
 select coalesce（null,"carrot","apple"）
 #返回carrot
```
```
select coalesce(1,"carrot","apple")
#返回1
```

**nullif()**
- NULLIF(Expression1,Expression2):给定两个参数Expression1和Expression2，如果**两个参数相等**，则返回**NULL**；否则就返回第一个参数。
- 除0操作：a/nullif(b,0)

**iif()**
- IIF ( boolean_expression, true_value, false_value )
- select iif(30>45,'对','错') as 结果——错
- select iif(null=null,'对','错') as 结果——错
- select iif(null is null,'对','错') as 结果 ——对

### 存储过程
- 是一组为了完成特定功能的SQL语句，类似一门程序设计语言，包括了数据类型、流程控制、输入和输出和它自己的函数库。

==**优点**==
- **提高性能**：存储过程只在创造时进行编译，以后每次执行存储过程都不需要重新编译。而一般SQL语句每执行一次就编译一次，故效率比T-SQL语句高。
	- **T-SQL**: SQL server对SQL语言的扩展
	- **PL-SQL**：ORACLE 对SQL语言的扩展
- **重复使用**：对数据库进行**复杂操作**时，可将此复杂操作用存储过程封装起来与数据库提供的事务处理结合一起使用。
- **减少网络流量**：一个存储过程在程序在网络中交互时可以替代大堆的T-SQL语句，提高通信速率。
- 一个存储过程在程序在网络中交互时可以替代大堆的T-SQL语句，所以也能降低网络的通信量，提高通信速率。
- **安全性高**：可设定只有某些用户才具有对指定存储过程的使用权

-------------------------------------------------------------------

==**基本语法**==

```sql
# 创建存储过程

CREATE PROC [ EDURE ] procedure_name [ ; number ]
    [ { @parameter data_type }
        [ VARYING ] [ = default ] [ OUTPUT ]
    ] [ ,...n ]
[ WITH
    { RECOMPILE | ENCRYPTION | RECOMPILE , ENCRYPTION } ]
[ FOR REPLICATION ]
AS sql_statement [ ...n ]

```

```sql
# 调用存储过程
# 存储过程如果有参数，后面加参数格式为：@参数名=value，也可直接为参数值value

EXECUTE Procedure_name '' 
```

```sql
# 删除存储过程
# 在存储过程中能调用另外一个存储过程，而不能删除另外一个存储过程

drop procedure procedure_name    
```
- procedure_name ：存储过程的名称，在前面加#为局部临时存储过程，加##为全局临时存储过程。
- number：是可选的整数，用来对同名的过程分组，以便用一条 DROP PROCEDURE 语句即可将同组的过程一起除去。例如，名为 orders 的应用程序使用的过程可以命名为 orderproc;1、orderproc;2 等。DROP PROCEDURE orderproc 语句将除去整个组。如果名称中包含定界标识符，则数字不应包含在标识符中，只应在 procedure_name 前后使用适当的定界符。
- @parameter：存储过程的参数。可以有一个或多个。用户必须在执行过程时提供每个所声明参数的值（除非定义了该参数的默认值）。存储过程最多可以有 ==2100== 个参数。
- data_type：参数的数据类型。（text、ntext、image）
- VARTING：制定作为输出参数支持的结果集（由于存储过程动态构造，内容可以变化）
- default：参数的默认值。如果定义了默认值，不必制定该参数的值即可执行过程。
- OUOTPUT：表明参数是返回参数。使用OUTPUT参数可将信息返回给调用过程。
- RECOMPILE：表明sql不会缓存该过程的计划，该过程将在运行时重新编译。
- ENCRYPTION：表示sql加密syscomments表中包含create procedure语句文本的条目。
- FOR REPLICATION: 指定不能在订阅服务器上执行为复制创建的存储过程。
- AS: 指定过程要执行的操作。
- sql_statement: 过程中要包含的任意数目和类型的 Transact-SQL 语句。但有一些限制。

**创建简单的存储过程**

```sql
CREATE PROCEDURE UserLogin
-- 定义全局变量
name varchar(20),
password varchar(20)

AS

-- 定义一个临时用来保存密码的变量
--DECLARE @strPwd NVARCHAR(20) 这里先不介绍变量。稍后的文章会详细讲到
BEGIN
select * from userinfo where userName=@name and userPass=@password
END
GO
```

**执行存储过程**

```sql
exec UserLogin admin,admin
--或者
EXEC UserLogin @name='admin',@password='admin'
```
