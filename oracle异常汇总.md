---
title: ORACLE异常汇总 
tags: ORACLE,异常
---

**目录**

* [Oracle_ERRMG](#oracle_errmg)
	* [ORA-00000: 正常的成功的完成(操作)](#ora-00000-正常的成功的完成操作)
		* [原因1: 正常执行完成。【部分验证】](#原因1-正常执行完成-部分验证)
		* [原因2: hosts文件配置错误。【未验证，网络汇总】](#原因2-hosts文件配置错误-未验证网络汇总)
	* [ORA-00001: 违反唯一约束条件 (string.string=>[拥有者].[约束名])](#ora-00001-违反唯一约束条件-stringstring拥有者约束名)
		* [原因1: UPDATE或INSERT语句试图插入重复的键。对于在DBMS MAC模式下配置的Trusted Oracle，如果在不同级别存在重复条目，您可能会看到此信息。【已验证】](#原因1-update或insert语句试图插入重复的键-对于在dbms-mac模式下配置的trusted-oracle如果在不同级别存在重复条目您可能会看到此信息-已验证)
	* [ORA-00017: 会话被要求设置跟踪事件【ora12_ERRMG】](#ora-00017-会话被要求设置跟踪事件ora12_errmg)
		* [原因1: 当前会话被要求通过另一个会话设置一个跟踪事件](#原因1-当前会话被要求通过另一个会话设置一个跟踪事件)
	* [ORA-待补充](#ora-待补充)
		* [原因1: 待补充](#原因1-待补充)

# Oracle_ERRMG

## ORA-00000: 正常的成功的完成(操作)
> ORA-00000: normal, successful completion

### 原因1: 正常执行完成。【部分验证】
> Normal exit.

* 分析: 此异常多数为程序没有执行SQL语句或者说成功执行完SQL语句，但人为或因逻辑有误，非要使用相关方法程序去获取Oracle的错误信息，得到此异常，实质是Oracle告知没有异常产生，猜测是异常信息的默认值为这个。目前发现以下两种情况:
	1. 存储过程、PL/SQL块等，使用sqlerrm获取异常，如下例所示。【已验证】
	```sql
	declare
	  v_sqlcode number;
	  v_sqlerrm varchar2(4000);
	begin
	  /*
		 ……
		 相关执行代码
		 ……
	  */
	  
	  v_sqlcode := sqlcode;
	  v_sqlerrm := sqlerrm;
	  dbms_output.put_line('本次的异常code:' || v_sqlcode || chr(10) || '本次的异常信息:' || v_sqlerrm);
	  
	exception
	  when others then
		rollback;
		v_sqlcode := sqlcode;
		v_sqlerrm := sqlerrm;
		dbms_output.put_line('本次的异常code:' || v_sqlcode || chr(10) || '本次的异常信息:' || v_sqlerrm);
	end;
	/
	```
	2. 使用OCI的C程序中，用erhms()函数（OCIErrorGet()）获得Oracle错误信息。【未验证，网络汇总】
* 措施: 无。【如果是人为需要获取该异常，则不用做任何操作；如果是逻辑有误，那么需要调整不去此异常或者在遇到此异常时将其屏蔽去掉。】
	> None

### 原因2: hosts文件配置错误。【未验证，网络汇总】
* 分析: 这种错误通常由于数据库是复制过来的，hosts文件中的ip对应的host name和当前的主机名不一致导致甚至hosts文件丢失，都会导致数据库startup时报此错。
* 措施: 校验hosts文件是否有错或缺失，进行修改或补充。
	- hosts文件在不同系统中所处的目录:
		```sql
		Windows XP/2000/Vista/7/8/8.1/10 ==> C:\windows\system32\drivers\etc\
		Linux及其他类Unix操作系统 ==> /etc/
		```

## ORA-00001: 违反唯一约束条件 (string.string=>[拥有者].[约束名]) 
> ORA-00001: unique constraint (string.string) violated

### 原因1: UPDATE或INSERT语句试图插入重复的键。对于在DBMS MAC模式下配置的Trusted Oracle，如果在不同级别存在重复条目，您可能会看到此信息。【已验证】
> An UPDATE or INSERT statement attempted to insert a duplicate key. For Trusted Oracle configured in DBMS MAC mode, you may see this message if a duplicate entry exists at a different level.

* 分析: 如下例所示，此异常一般为违反作用于表上的唯一约束或者主键约束导致，它们限制了表的一列或多列值的唯一性，不能插入重复数据。
	```sql
	-- 创建测试表
	create table ora_00001_1(
		a char(24) /*primary key*/,  -- 亦可加注释内信息实现添加主键约束
		b number /*unique*/, -- 亦可加注释内信息实现添加唯一约束
		-- 增加主键约束
		constraint ora_00001_1_a primary key (a)
		-- 亦可加注释内信息实现添加唯一约束
		/*, constraint ora_00001_1_b unique (b)*/
	 );
	 
	 -- 亦可加注释内信息实现添加主键约束
	/*alter table ora_00001_1 add constraint ora_00001_1_a primary key (a);*/
	 -- 增加唯一约束
	alter table ora_00001_1 add constraint ora_00001_1_b unique (b);

	-- 插入测试数据
	insert into ora_00001_1(a, b) values ('1',1);
	insert into ora_00001_1(a, b) values ('2',2);
	commit;

	-- ORA-00001: 违反唯一约束条件 (C##LY.ORA_00001_1_A);
	insert into ora_00001_1(a, b) values ('1',3);
	-- ORA-00001: 违反唯一约束条件 (C##LY.ORA_00001_1_B)
	insert into ora_00001_1(a, b) values ('3',2);
	-- ORA-00001: 违反唯一约束条件 (C##LY.ORA_00001_1_A)
	update ora_00001_1 set a = '1' where a = '2';
	-- ORA-00001: 违反唯一约束条件 (C##LY.ORA_00001_1_B)
	update ora_00001_1 set b = '2' where b = '1';
	```
* 措施: 删除唯一约束限制或不插入重复值。
	> Either remove the unique restriction or do not insert the key.
	
	- 如果分析确定此处唯一约束或主键约束不需要，那么则可使用下面语句删除约束
		```sql
		-- 查询约束与索引信息
		select a.owner 约束所有者,
			   a.constraint_name 约束名,
			   case a.constraint_type
				 when 'P' then
				  'Primary key'
				 when 'U' then
				  'Unique key'
				 when 'C' then
				  ' Check constraint on a table'
				 when 'R' then
				  'Referential integrity'
				 when 'V' then
				  'With check option, on a view'
				 when 'O' then
				  'With read only, on a view'
				 when 'H' then
				  'Hash expression'
				 when 'F' then
				  'Constraint that involves a REF column'
				 when 'S' then
				  'Supplemental logging'
				 else
				  'unkown'
			   end 约束类型,
			   b.table_name 表名,
			   b.column_name 列名,
			   c.index_name 索引名,
			   c.uniqueness 是否唯一索引/*,
			   d.table_name 表名,
			   d.column_name 列名*/
		  from user_constraints a, user_cons_columns b, user_indexes c/*, user_ind_columns d*/
		 where a.constraint_name = b.constraint_name
		   and a.index_name = c.index_name
		   /*and c.index_name = d.index_name*/
		   and a.owner = b.owner
		   and a.owner = c.table_owner
		   and a.owner = &"[拥有者]"
		   and a.constraint_name = &"[约束名]";
		
		-- 由于如果约束对应的唯一索引若是事先手工创建的，那么在删除约束时索引不会被删除，Oracle之后自动删除自己隐式创建的索引。
		-- 因此加上drop index，可确保一定将索引删除。
		alter table [表名] drop constraint [约束名] drop index;
		-- 如果是主键约束，有可能遇到有用作外键的情况，那么在删除时仍会报=>ORA-02273: 此唯一/主键已被某些外键引用
		-- 报错后了解是否有问题，是否需去除此主键和外键，然后可考虑用下面语句删除主键约束，会同时删除外键约束
		alter table [表名] drop constraint [约束名] cascade drop index;
		alter table [表名] drop primary key cascade drop index;
		```
	- 如果分析确定是值重复，那么需排查表数据与预执行的SQL语句的重复值冲突、同一个事务内执行的SQL语句之间的重复值冲突，去掉重复值的插入或更新。
* 备注:
	1. 唯一约束与主键约束的同:
		- 都通过唯一索引来限制约束列的唯一性，确保任何使表中约束的列在行与行之间存在重复值的操作失败
		- 若无事先创建好唯一索引，都会在创建唯一约束或主键约束时隐式创建同名的唯一约束
		- 有约束必定有索引（无法在保持约束存在的情况下删除索引=>ORA-02429: 无法删除用于强制唯一/主键的索引）
		- 有索引不一定有约束（只删除约束但不删除索引则仍然会限制索引列的值的唯一性）
	2. 唯一约束与主键约束的异:
		- 唯一约束允许在该列或多列上存在NULL值，但主键约束不能存在NULL值
		- 一个表只能创建一个主键约束，但可创建多个唯一约束
		- 主键可扩展作为外键，唯一约束不可

## ORA-00017: 会话被要求设置跟踪事件【ora12_ERRMG】
> ORA-00017: session requested to set trace event

### 原因1: 当前会话被要求通过另一个会话设置一个跟踪事件
> The current session was requested to set a trace event by another session.

* 措施: 内部使用；无需操作。
	> This is used internally; no action is required.

## ORA-待补充
> ORA-待补充

### 原因1: 待补充
* 分析: 待补充
* 措施: 待补充
* 备注: 待补充
