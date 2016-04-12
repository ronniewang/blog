# sql server迁移到mysql遇到的坑

## 坑一

字段类型

### sql server的money类型

sql server中money在mysql中没有，mysql需要换成decimal
而且要注意长度显示制定
sql server的默认decimal的长度为255，mysql最大只有30，如果sql server中使用默认长度，mysql会报错

## 坑二

函数和存储过程的语法不同

### 变量声明

#### mysql

variable

#### sql server

@variable

### case when

## 坑三

内置函数用法不同

### ISNULL

#### mysql

**ISNULL(check_expression)**

**IFNULL(check_expression, replacement_value)**

#### sql server

**ISNULL(check_expression, replacement_value)**

在check_expression为NULL时将返回replacement_value。replacement_value必须与check_expresssion具有相同的类型

### dateadd

#### mysql

**DATE_ADD(date,INTERVAL expr type)**

date 参数是合法的日期表达式。expr 参数是您希望添加的时间间隔。
详情：<http://www.w3school.com.cn/sql/func_date_add.asp>

#### sql server

**DATEADD(datepart,number,date)**

date 参数是合法的日期表达式。number 是您希望添加的间隔数；对于未来的时间，此数是正数，对于过去的时间，此数是负数。
详情：<http://www.w3school.com.cn/sql/func_dateadd.asp>
