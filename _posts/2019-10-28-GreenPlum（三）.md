---
layout:     post
title:      GreenPlum 学习（三）
subtitle:   
date:       2019-10-28
author:     LT
header-img: img/home-bg-hacker.jpg
catalog: true
tags:
    - GreenPlum
    - data warehouse
    - postgresql

---

>@disabled 由于gp阉割了trigger功能，此章意义不大，详见POSTGRESQL RULE作为代替

# PostgreSQL 触发器

>说来惭愧，这种比较基础的关系型数据库应该是要熟练掌握的知识，学校的时候学的还不错，但是工作以来一直用得很少以致渐忘

### 什么是触发器
>触发器是一种由事件自动触发执行的特殊存储过程，这些事件可以是对一个表进行 INSERT、UPDATE、DELETE 等操作。

>触发器经常用于加强数据的完整性约束和业务规则上的约束等。

### 查看触发器
	select * from information_schema.triggers

![image.png](https://upload-images.jianshu.io/upload_images/7232713-0b71714554a1fa2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----

### 创建触发器
	# 语法如下
	CREATE [ CONSTRAINT ] TRIGGER name 
	{ BEFORE | AFTER | INSTEAD OF } { event [ OR ... ]}
	ON table_name
	[ FROM referenced_table_name ]
	{ NOT DEFERRABLE | [ DEFEREABLE ] { IINITIALLY IMMEDIATE | INITIALLY DEFERED} }
	FOR [ EACH ] { ROW | STATEMENT }
	[ WHEN { condition }]
	EXECUTE PROCEDURE function_name ( arguments )

创建触发器的步骤：

先为触发器建一个执行函数，此函数的返回类型为触发器类型 trigger；然后即可创建相应的触发器。

例如当删除学生表(student)中的一条记录时，把这个学生在成绩表 (score) 中的成绩记录也删除掉，这时就可以使用触发器。

先建触发器的执行函数：
	
	CREATE OR REPLEASE FUNCTION student_delete_trigger_fun()
	returns trigger as $$
	begin
	    delete from score where student_no = old.student_no;
	    return old;
	end;
	$$
	language plpgsql;

	# 再创建这个触发器
	CREATE TRIGGER delete_student_trigger
	after delete on student
	for each row execute procedure student_delete_trigger_fun();

-----
### 语句级触发器与行级触发器

语句级的触发器是指执行每个 SQL 时，只执行一次，行级触发器则指每行都会执行一次。

一个修改零行的操作会导致合适的语句级触发器被执行，但不会触发行级触发器。

批量插入时，语句级别的触发器只触发一次，不管 affected row 是否为 0，但是行级触发器的触发次数为 affected row。

----
### BEFORE 触发器与 AFTER 触发器
语句级别的 BEFORE 触发器是在语句开始做任何事情之前就被触发了，而语句级别的 AFTER 触发器是在语句结束时才触发的。

行级别的 BEFORE 触发器在对特定行进行操作之前触发，而行级别的 AFTER 触发器是在语句结束时才触发的，但是它会在任何语句级别的 AFTER 触发器被触发之前触发。

----

### 触发器的行为
触发器函数有返回值，语句级触发器应该总是返回 NULL，即必须显式地在触发器函数中写上 “RETURN NULL”，如果没有写，将导致出错。报错信息如下所示：

	ERROR:control reached end of trigger procedure without RETURN
	CONTEXT: PL/pgSQL function log_student_trigger()

对于 BEFORE 和 INSTEAD OF 这类行级触发器来说，如果返回的是 NULL，则表示忽略当前行的操作。如果返回非 NULL 的行，对于 INSERT 和 UPDATE 操作来说，返回的行将成为被插入的行或将要更新的行。

对于 AFTER 这类行级触发器来说，其返回值会被忽略。

---
### 触发器函数中的特殊变量

当把一个 PL/pgSQL 函数当作触发器函数调用的时候，系统会在顶层的声明段里自动创建几个特殊变量，比如在之前例子中的 “NEW”、“OLD”、“TG_OP” 变量等。可以使用的变量有如下这些。

NEW：该变量为 INSERT/UPDATE 操作触发的行级触发器中存储的新的数据行，数据类型是 RECORD。在语句级别的触发器里此变量没有分配，DELETE 操作触发的行级触发器中此变量也没有分配。

OLD：该变量为 UPDATE/DELETE 操作触发的行级触发器中存储的旧数据行，数据类型是 RECORD。在语句级别的触发器里此变量没有分配， INSERT 操作触发的行级触发器中此变量也没有分配。

TG_NAME：数据类型是 name，该变量包含实际触发的触发器名。

TG_WHEN: 内容为 BEFORE 或 AFTER 的字符串，用于指定是 BEFORE 触发器还是 AFTER 触发器。

TG_LEVEL: 内容为 ROW 或 STATEMENT 的字符串用于指定是语句级触发器还是行级触发器。

TG_OP: 内容为 INSERT、UPDATE、DELETE、TRUNCATE 之一的字符串，用于指定 DML 语句的类型。

TG_RELID: 触发器所在表的 OID。

TG_TABLE_NAME: 触发器所在表的名称。

TG_TABLE_SCHEMA: 触发器所在表的模式。

TG_NARGS: 在 CREATE TRIGGER 语句里面赋予触发器过程的参数个数。

TG_ARGV[]:  为 text 类型的一个数组；是 CREATE TRIGGER 语句里的参数。

----
### 删除触发器
语法如下：

	DROP TRIGGER [ IF EXISTS ] NAME ON TABLE [ CASCADE | RESTRICT ];
 

删除触发器时，触发器的函数不会被删除。不过，当表删除时，表上的触发器也会被删除，你可以使用 DROP FUNCTION fun_name 直接删除触发器函数。

----

### 事件触发器
创建事件触发器的语法如下：
	
	CREATE EVENT TRIGGER 
	ON event
	[ WHEN filter_variable IN (filter_value [,...]) [ and ...]]
	EXECUTE PROCEDURE function_name()

>在创建事件触发器之前，必须先创建触发器函数，事件触发器函数的返回类型为 event_trigger，注意，其与普通触发器函数的返回类型 （trigger）是不一样的。

在官方手册中，有一个禁止所有 DDL 语句的例子，如下：


	CREATE OR REPLACE FUNCTION abort_any_command()
	returns event_trigger
	language plpgsql
	AS $$
	BEGIN
	    RAISE EXCEPTION 'command % is disabled', tg_tag;
	END;
	$$;
	
	
	CREATE EVENT TRIGGER abort_DDL ON DDL_command_start
	EXECUTE PROCEDURE abort_any_command();

现在执行 DDL 语句将会报错

	postgres=# drop table test01;
	ERROR:command DROP TABLE is disabled
	STATEMENT: drop table test01;
	ERROR:  COMMAND DROP TABLE is disabled


如果想再允许 DDL 操作，可以禁止事件触发器，

	ALTER EVENT TRIGGER abort_ddl DISABLE;

TG_EVENT:为 "ddl\_command\_start"、"ddl\_command\_end"、"sql\_drop" 之一。

TG_TAG: 只具体的哪种 DDL 操作，如 "CREATE TABLE"、"DROP TABLE" 等。