# Mybatis中的#{}和${}格式的占位符

在使用Mybatis时，在SQL语句中的参数，可以使用`#{}`或`${}`格式的占位符。

当配置的SQL语句如下时：

```SQL
SELECT
    <include refid="StandardQueryFields" />
FROM
    ams_admin
WHERE
    id=#{id}
```

以上SQL语句中的参数，无论使用`#{}`还是`${}`，执行效果完全相同。

当配置的SQL语句如下时：

```SQL
SELECT
    <include refid="LoginQueryFields"/>
FROM
    ams_admin
WHERE
    username=#{username}
```

以上SQL语句中的参数，使用`#{}`格式的占位符时可以正常执行，使用`${}`格式的占位符将执行出错，错误信息例如：

```
java.sql.SQLSyntaxErrorException: Unknown column 'fanchuanqi' in 'where clause'
; bad SQL grammar []; nested exception is java.sql.SQLSyntaxErrorException: Unknown column 'fanchuanqi' in 'where clause'
```

其实，在SQL语句中，除了关键字、数值、特定位置的字符或字符串以外，只要没有使用特殊符号框住，SQL语句中的其它内容都会被视为“字段名”，例如：

```mysql
SELECT
    id, username, password, enable
FROM
    `ams_admin`
WHERE
    username=fanchuanqi
```

> 提示：使用一对单撇符号框住的就是自定义名称
>
> 提示：使用一对单引号框住的都是字符串

在使用Mybatis时，如果SQL语句中的参数使用`#{}`格式的占位符，会进行预编译的处理，如果使用的是`${}`格式的占位符，则不会预编译。

> 提示：计算机能够直接识别并执行的只有机器语言，即二进制语言，所有其它编程语言编写的源代码都需要经过编译、解释，转换成机制语言才可以被识别并执行。在执行编译之前，通常都还有词法分析、语义分析等过程。由于语义分析是在编译之前执行的，所以，一旦执行到了编译，则SQL语句的“意思”不会再发生变化！

以以下SQL语句为例：

```sql
SELECT
    <include refid="LoginQueryFields"/>
FROM
    ams_admin
WHERE
    username=#{username}
```

由于使用的是`#{}`格式的占位符，则`#{username}`会被识别成参数，经过预编译（先编译，再传值，再执行）处理后，无论在此处传入什么值，都会被认为是参数，语义不会发生变化！

如果使用的是`${}`格式的占位符，则会先将参数值代入到SQL语句中，然后再执行编译！

所以，使用`#{}`格式的占位符，不必关心参数值的数据类型问题，并且，**没有SQL注入的风险**，因为在编译之前就已经确定了此SQL语句的语义，无论传的值是什么样的，语义都不会改变！但是，这种格式的占位符只能表示某个值，不能表示SQL语句中的其它部分！

使用`${}`格式的占位符，需要关心参数值的数据类型问题，如果参数的值是字符串类型的，必须在值的两侧添加单引号，这种做法**存在SQL注入的风险**，因为传入的参数值可能会改变语义！需要注意，这种格式的占位符可以表示SQL语句中的任何片段！

另外，对于SQL注入，应该有正确的认识，不需要盲目拒绝！

```mysql
SELECT * FROM user WHERE username='?' AND password='?'

username: root
password: secret

SELECT * FROM user WHERE username='root' AND password='secret'

username: root
password: a' OR 'a'='a

SELECT * FROM user WHERE username='root' AND password='a' OR 'x'='x' OR 'a'='a'
```

