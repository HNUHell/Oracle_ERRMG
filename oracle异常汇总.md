---
title: ORACLE异常汇总 
tags: ORACLE,异常
---

**目录**

* [Oracle_ERRMG](#oracle_errmg)
	* [ORA-00000: 正常的成功的完成(操作)](#ora-00000-正常的成功的完成操作)
		* [原因1: 正常执行完成。【部分验证】](#原因1-正常执行完成部分验证)
		* [原因2: hosts文件配置错误。【未验证，网络汇总】](#原因2-hosts文件配置错误未验证网络汇总)
	* [ORA-00001: 违反唯一约束条件 (string.string=>[拥有者].[约束名])](#ora-00001-违反唯一约束条件-stringstring拥有者约束名)
		* [原因1: UPDATE或INSERT语句试图插入重复的键。对于在DBMS MAC模式下配置的Trusted Oracle，如果在不同级别存在重复条目，您可能会看到此信息。【已验证】](#原因1-update或insert语句试图插入重复的键对于在dbms-mac模式下配置的trusted-oracle如果在不同级别存在重复条目您可能会看到此信息已验证)
	* [ORA-00017: 会话被要求设置跟踪事件](#ora-00017-会话被要求设置跟踪事件)
		* [原因1: 当前会话被要求通过另一个会话设置一个跟踪事件](#原因1-当前会话被要求通过另一个会话设置一个跟踪事件)
	* [ORA-00018: 超出最大会话数](#ora-00018-超出最大会话数)
		* [原因1: 所有会话状态对象都在使用中。【部分验证】](#原因1-所有会话状态对象都在使用中部分验证)
	* [ORA-00019: 超出最大许可会话数](#ora-00019-超出最大许可会话数)
		* [原因1: 所有许可会话都在使用中。【部分验证】](#原因1-所有许可会话都在使用中部分验证)
	* [ORA-00020: 超出最大进程数(string=>[最大进程数])](#ora-00020-超出最大进程数string最大进程数)
		* [原因1: 所有进程状态对象都在使用中。【部分验证】](#原因1-所有进程状态对象都在使用中部分验证)
	* [ORA-01747: user.table.column, table.column 或列说明无效](#ora-01747-usertablecolumn-tablecolumn-或列说明无效)
		* [原因1: 列名为关键字。【已验证】](#原因1-列名为关键字已验证)
		* [原因2: DML语句缺失列或多逗号。【已验证】](#原因2-dml语句缺失列或多逗号已验证)
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
		```
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

## ORA-00017: 会话被要求设置跟踪事件

> ORA-00017: session requested to set trace event

### 原因1: 当前会话被要求通过另一个会话设置一个跟踪事件【ora12_ERRMG】

> The current session was requested to set a trace event by another session.

* 措施: 内部使用；无需操作。

	> This is used internally; no action is required.

## ORA-00018: 超出最大会话数

> ORA-00018: maximum number of sessions exceeded

### 原因1: 所有会话状态对象都在使用中。【部分验证】

> All session state objects are in use.

* 分析: 很明显，系统中所有会话数目已经达到设置的**SESSIONS**值，因此准备要创建的会话无法成功创建，而这个会话包括有用户建立连接至数据库是产生的会话、后台进程产生的会话以及各类涉及到硬解析处理数据字典基表的DML、DDL语句产生的递归会话。
* 措施: 增加**SESSIONS**初始化参数值。【是否需要增加**SESSIONS**值还需进行判断，是否是由于此值过小而现实场景需要更大的值？】

	> Increase the value of the SESSIONS initialization parameter.
	
	- 如果判断确定是由于**SESSIONS**值过小导致，则需修改增大此参数值: 
	
		```sql
		 /*
			alter system set 参数名=值  [scope=应用范围];
			scope需知：
				scope=both，表示修改会立即生效且会修改spfile文件以确保数据库在重启后也会生效如果（以spfile启动此项为缺省值）；
				scope=memory，表示修改会立即生效但不会修改spfile文件，因此重启后失效（以pfile启动此项为缺省值，且只可设置这个值）；
				scope=spfile，表示只修改spfile文件，在重启数据库后才生效（对应静态参数则只可设置此项值，设置其它值会报错：
												 ORA-02095: specified initialization parameter cannot be modified）。
		 */
	 
		-- 查看"是否可用ALTER SYSTEM修改"列值，根据结果进行修改
		select name 参数名,
			   case type
				 when 1 then
				  'Boolean'
				 when 2 then
				  'String'
				 when 3 then
				  'Integer'
				 when 4 then
				  'Parameter file'
				 when 5 then
				  'Reserved'
				 when 6 then
				  'Big integer'
				 else
				  'unknown'
			   end 参数类型,
			   value "会话级(若可修改)或实例级参数值",
			   display_value 展示值,
			   isses_modifiable "是否可用ALTER SESSION修改",
			   case issys_modifiable
				 when 'IMMEDIATE' then
				  '无论pfile还是spfile启动，都可用"alter system set ' || name || '=&' ||
				  '新的参数值;"更改参数并立即生效。'
				 when 'DEFERRED' then
				  '无论pfile还是spfile启动，都可用"alter system set ' || name || '=&' ||
				  '新的参数值;"更改参数并将在之后的会话中生效。'
				 when 'FALSE' then
				  case a.is_spfile
					when 0 then
					 '使用pfile启动，需手动修改pfile文件中对应参数值再重启。'
					else
					 '使用spfile启动，可用"alter system set ' || name || '=&' || name ||
					 ' scope=spfile;"更改参数。更改将在后续的实例中生效（当前数据库需重启）。'
				  end
				 else
				  '?'
			   end "是否可用ALTER SYSTEM修改",
			   isinstance_modifiable "是否不同实例间值可不同"
		  from v$parameter,
			   (select count(1) is_spfile from v$parameter t where t.name = 'spfile') a
		 where name = 'sessions';
		```
	
	- 如果判断确定**SESSIONS**值合理，则需分析确定产生大量会话的原因，是否相关程序代码建立了连接未释放？或者其它原因等。【待完善】

* 备注: 
    参数[SESSIONS](http://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams234.htm#REFRN10197):

	| 属性 | 描述 |
	| ---- | ---- |
	| 参数类型 | Integer |
	| 默认值 | 派生公式: (1.1 \* **PROCESSES**) \+ 5） ``[11gR1,11gR2]`` (1.5 \* **PROCESSES**) \+ 22 |
	| 可修改(不用重启及时生效) | 否 ``[11gR2,12cR1]`` 可用**ALTER SYSTEM**修改 |
	| 取值范围 | 1\~2^31 ``[11gR1,11gR2]`` 1\~2^16(1\~65536) |
	| 基础参数 | 是 |
	
	**SESSIONS**指定可以在系统中创建的最大会话数。因为每次登录都需要一个会话，所以这个参数有效地确定了系统中最大并发用户数。您应该始终将此参数显式设置等于最大并发用户数的估计值+后台进程数+递归会话数（大约占总数的10%）。
	
	Oracle使用此参数的**默认值**作为其**最小值**。 将**SESSIONS**值设置成\[1~默认值)不会触发错误，因为Oracle会忽略此值直接使用默认值。
	
	**ENQUEUE_RESOURCES**和**TRANSACTIONS**参数的默认值派生自**SESSIONS**。因此，如果增加**SESSIONS**的值，则应考虑是否也调整**ENQUEUE_RESOURCES**和**TRANSACTIONS**的值。 （请注意，从Oracle Database 10g release 2（10.2）起，**ENQUEUE_RESOURCES**已被废弃。）
	
	在共享服务器环境中，**PROCESSES**的值可能相当小。因此，Oracle建议您将**SESSIONS**的值调整为大约1.1 \*总连接数。

## ORA-00019: 超出最大许可会话数

> ORA-00019: maximum number of session licenses exceeded

### 原因1: 所有许可会话都在使用中。【部分验证】

> All licenses are in use.

* 分析: 很明显，系统中并发用户会话已经达到设置的**LICENSE_MAX_SESSIONS**值，因此准备要创建的用户会话无法创建。
* 措施: 增大**LICENSE_MAX_SESSIONS**初始化参数的值。【是否需要增加**LICENSE_MAX_SESSIONS**值还需进行判断，是否是由于此值过小而现实场景需要更大的值？】


	> Increase the value of the LICENSE MAX SESSIONS initialization parameter.
	
	- 如果判断确定是由于**LICENSE_MAX_SESSIONS**值过小导致，则需修改增大此参数值，【详情参见[ORA-00018=>原因1=>措施](#原因1-所有会话状态对象都在使用中部分验证)，将SQL语句中``name = 'sessions'``修改为``name = 'license_max_sessions'``即可】 
	- 如果判断确定**LICENSE_MAX_SESSIONS**值合理，则需分析确定产生大量会话的原因，是否相关程序代码建立了连接未释放？或者其它原因等。【待完善】
	
* 备注: 
	参数[LICENSE_MAX_SESSIONS](http://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams116.htm#REFRN10079):

	| 属性 | 描述 |
	| ---- | ---- |
	| 参数类型 | Integer |
	| 默认值 | 0 |
	| 可修改(不用重启及时生效) | 可用**ALTER SYSTEM**修改 |
	| 取值范围 | 0\~许可会话数 |
	| 基础参数 | 否 |
	| Oracle实时应用集群 | 多个实例可以具有不同的值，但是安装数据库的所有实例的总和应小于或等于该数据库许可的会话总数。 |
	
	**LICENSE_MAX_SESSIONS**指定允许的并发用户会话的最大数量。达到此限制后，只有具有RESTRICTED SESSION权限的用户才能连接到数据库。无法连接的用户收到表示系统达到最大容量的警告消息。
	
	零值表示不强制执行并发使用（会话）许可。如果将此参数设置为非零数字，则可能还需要设置**LICENSE_SESSIONS_WARNING**（请参阅“[LICENSE_SESSIONS_WARNING](http://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams117.htm#REFRN10080)”）。
	
	不要同时启用并发使用许可和用户许可，即**LICENSE_MAX_SESSIONS**与**LICENSE_MAX_USERS**两参数值至少一个要设置为零。

## ORA-00020: 超出最大进程数(string=>[最大进程数])

> ORA-00020: maximum number of processes (string) exceeded

### 原因1: 所有进程状态对象都在使用中。【部分验证】

> All process state objects are in use.

* 分析: 很明显，系统中进程数已经达到设置的**PROCESSES**值，因此准备要创建的用户会话无法创建。
* 措施: 增加**PROCESSES**初始化参数的值。【是否需要增加**PROCESSES**值还需进行判断，是否是由于此值过小而现实场景需要更大的值？】

	> Increase the value of the PROCESSES initialization parameter.
	
	- 如果判断确定是由于**PROCESSES**值过小导致，则需修改增大此参数值，【详情参见[ORA-00018=>原因1=>措施](#原因1-所有会话状态对象都在使用中部分验证)，将SQL语句中``name = 'sessions'``修改为``name = 'processes'``即可】 
	- 如果判断确定**PROCESSES**值合理，则需分析确定产生大量进程的原因，是否相关程序代码建立了连接未释放？或者其它原因等。【待完善】

* 备注: 
	参数[PROCESSES](http://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams202.htm#REFRN10175):

	| 属性 | 描述 |
	| ---- | ---- |
	| 参数类型 | Integer |
	| 默认值 | 40\~操作系统依赖数 ``[10gR2,11gR1]`` 100 ``[11gR2,12cR1]`` 该值是派生的，它通常取决于警报日志中报告的核心数。 |
	| 可修改(不用重启及时生效) | 否 |
	| 取值范围 | 6\~操作系统依赖数 |
	| 基础参数 | 是 |
	| Oracle实时应用集群 | 多个实例可以具有不同的值。 |
	
	**PROCESSES**指定可以同时连接到Oracle的最大操作系统用户进程数。它的值应允许所有后台进程运行，如锁，作业队列进程和并行执行进程。
	
	该参数派生了**SESSIONS**和**TRANSACTIONS**参数的默认值。因此，如果更改**PROCESSES**的值，则应评估是否要调整这些派生参数的值。

## ORA-01747: user.table.column, table.column 或列说明无效

> ORA-01747: invalid user.table.column, table.column, or column specification

### 原因1: 列名为关键字。【已验证】

* 分析: 一般为在SQL语句或存储过程、函数等中使用到的此字段为oracle的保留关键字，且保留方式标识了此关键字在某些情况下，例如在DML中是否不允许作为标识符的。如下列情况:

	```sql
	-- 查询能做属性但不能作为标识符或某些场景（如DML操作）下不能作为标识符的关键字
	select t.*
	  from v$reserved_words t
	 where (t.res_semi = 'Y' or t.reserved = 'Y')
	   and t.res_attr = 'N';
	-- 根据上面关键字建表，为测试需要，实际使用时请避免将Oracle保留关键字作为表的字段！
	create table ora_01747_1 (
	   "TRIGGER" number, "WHERE" number, "REVOKE" number, "INCREMENT" number, "THEN" number, 
	   "FILE" number, "PRIOR" number, "CONNECT" number, "COMMENT" number, "SYSDATE" number, 
	   "ONLINE" number, "DECIMAL" number, "SESSION" number, "MODIFY" number, "IN" number, 
	   "@" number, "," number, "GRANT" number, "INTO" number, "VALIDATE" number, "." number, 
	   "ADD" number, "ORDER" number, "HAVING" number, "TO" number, "NULL" number, "RENAME" number, 
	   "LEVEL" number, "USER" number, "ANY" number, /*"ROWID" number, --不可作建表属性*/ 
	   "SHARE" number, "MODE" number, "UNION" number, "/" number, "SET" number, "INDEX" number, 
	   "MAXEXTENTS" number, "VALUES" number, "|" number, "VIEW" number, "[" number, "WITH" number, 
	   "EXCLUSIVE" number, "ALTER" number, "FROM" number, "SELECT" number, "BY" number, "-" number, 
	   "MLSLABEL" number, "AND" number, "+" number, "ROWS" number, "CHECK" number, ":" number, 
	   "VARCHAR2" number, "IMMEDIATE" number, "CURRENT" number, "AS" number, "*" number, "TABLE" number, 
	   "LONG" number, "SYNONYM" number, "ASC" number, "UNIQUE" number, "LIKE" number, "DESC" number, 
	   "VARCHAR" number, "INITIAL" number, "CHAR" number, "=" number, "DROP" number, "AUDIT" number, 
	   "ROWNUM" number, "FLOAT" number, "COMPRESS" number, "OFFLINE" number, "NOT" number, "DELETE" number, 
	   "^" number, "BETWEEN" number, "EXISTS" number, "IDENTIFIED" number, "WHENEVER" number, "INTEGER" number, 
	   "SIZE" number, "NOWAIT" number, ")" number, "]" number, "NOCOMPRESS" number, "COLUMN" number, "ELSE" number, 
	   "FOR" number, "INTERSECT" number, "!" number, "PRIVILEGES" number, "SUCCESSFUL" number, "PCTFREE" number, 
	   "UPDATE" number, "ACCESS" number, "RESOURCE" number, "UID" number, "DATE" number, "NOAUDIT" number, 
	   "RAW" number, /*"&" number,--不可作建表属性 */"OPTION" number, "ROW" number, "SMALLINT" number, 
	   "MINUS" number, "OF" number, "ON" number, ">" number, "INSERT" number, "DEFAULT" number, "ALL" number, 
	   "START" number, "IS" number, "CREATE" number, "DISTINCT" number, "LOCK" number, "CLUSTER" number, 
	   "GROUP" number, "PUBLIC" number, "OR" number, "<" number, "NUMBER" number, "(" number/* ,"" number --不可作建表属性*/
	);

	-- 异常测试（不是所有属性错误使用都会产生ORA-01747的异常，还可能产生ORA-00936、ORA-01788、ORA-01745等异常）
	-- ORA-01747: user.table.column, table.column 或列说明无效
	select SET from ora_01747_1;
	update ora_01747_1 set NUMBER = 1;
	update ora_01747_1 set ,"NUMBER" = 1;
	insert into ora_01747_1("TRIGGER", ,WHERE) values(1, ,1);
	```
	
* 措施: 在使用此字段时，添加英文双引号（""）包裹。
	- 通过下面语句可查询此类关键字的使用情况: 
	
		```sql
		select a.table_name 表,
			   a.column_name 原字段,
			   '"' || a.column_name || '"' 使用需加双引号,
			   'select "' || a.column_name || '" from ' || a.table_name ||
			   ' where rownum = 1;' 简单查询语句,
			   b.reserved "是否不能作标识符",
			   b.res_type "是否不能作类型名称",
			   b.res_attr "是否不能作属性名称",
			   b.res_semi "是否某些环境(DML)不能作标识符",
			   b.duplicate
		  from user_tab_columns a, v$reserved_words b
		 where a.column_name = b.keyword
		   and b.res_semi = 'Y'
		--  and a.table_name = '表名'
		--  and a.column_name = '字段名'
		;
		```
		
* 备注: **实际使用时请避免将Oracle保留关键字作为表的字段！**

### 原因2: DML语句缺失列或多逗号。【已验证】

* 分析: 一般为使用各种方式动态拼接SQL时，拼接有误，使得insert语句或update语句中逗号分隔的左边或右边出现空，示例可参见上面原因1的分析代码。
* 措施: 检查拼接逻辑，加上缺失字段或去除多余的逗号

## ORA-待补充

> ORA-待补充

### 原因1: 待补充

> 待补充

* 分析: 待补充
* 措施: 待补充

	> 待补充
	
* 备注: 待补充

