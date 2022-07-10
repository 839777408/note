---
title: Daily Support
categories:
  - 总结
tags:
  - 总结
top_img: 'https://npm.elemecdn.com/nan-picture/img/wp9.jpg'
cover: 'https://npm.elemecdn.com/nan-picture/img/wp9.jpg'
abbrlink: 24756
top: 99
date: 2022-05-09 19:31:40
updated: 2022-05-09 19:31:44
---

# SQL

## 操作前的判断

```sql
#创建表前的判断
USE [db_crn]
GO

IF  EXISTS (SELECT 1 FROM dbo.sysobjects WHERE type = 'U' AND name = 'tatp_dus_extr_result_rtn_log')
	BEGIN
		DROP TABLE [dbo].[tatp_dus_extr_result_rtn_log]
	END

#创建存储过程前的判断
USE [db_crn]
GO

IF EXISTS (SELECT 1 FROM dbo.sysobjects WHERE id = OBJECT_ID(N'[dbo].[po_client_sel_in_man_log]') AND OBJECTPROPERTY(id, N'IsProcedure') = 1)
	BEGIN
		DROP PROCEDURE [dbo].[po_client_sel_in_man_log]
	END
GO

#添加表字段前的判断
USE [db_crn]
GO

IF NOT EXISTS (SELECT 1 FROM sys.columns WHERE object_id = OBJECT_ID(N'[dbo].[cm_mandate_log]')
AND name = 'acct_type')
    BEGIN
    	ALTER TABLE dbo.cm_mandate_log ADD acct_type char(2) NULL
    END
GO

#创建触发器前的判断
USE [db_crn]
GO

IF OBJECT_ID ('dbo.trg_tatp_dus_extr_result_rtn', 'TR') IS NOT NULL
	BEGIN
		DROP TRIGGER  [dbo].[trg_tatp_dus_extr_result_rtn]
	END
GO

#创建索引前的判断
USE [db_crn]
GO

IF NOT EXISTS(SELECT 1 FROM sysindexes WHERE id = OBJECT_ID('customer_usci_info_ext_hist_staging') AND name = 'icustomer_usci_info_ext_hist_staging')
	BEGIN
		DROP INDEX customer_usci_info_ext_hist_staging.icustomer_usci_info_ext_hist_staging
	END
GO
```

## 索引的操作

```sql
# 删除主键
alter table customer_usci_info_ext_hist_staging drop constraint pk_customer_usci_info_ext_hist_staging 

# 删除普通索引
drop index customer_usci_info_ext_hist_staging.icustomer_usci_info_ext_hist_staging

# 新建唯一索引
CREATE UNIQUE INDEX uk_customer_activity_log_pos ON db_crn.dbo.customer_activity_log_pos (seq)

# 新建主键
alter table customer_usci_info_ext_hist_staging add constraint pk_customer_usci_info_ext_hist_staging PRIMARY key(policy_no,client_type,cycle_date,info_ind)
```

## 获取下一个sequence

```sql
#sql server
declare @tm_new_detail_id numeric(20, 0)
SELECT @tm_new_detail_id = CONVERT(numeric(20, 0), NEXT VALUE FOR db_xgfe_cm.customer_details_seq)

#SYBASE
declare @new_cust_detail_id unsigned bigint
select @new_cust_detail_id = convert(unsigned bigint, reserve_identity('customer_details_seq',1)) 
```

## 查看表锁级别

`select lockscheme('db_crn..customer_policy')`

## 查包含某字符串的存储过程

`select distinct object_name(id) from syscomments where id in(select id from sysobjects where type ='P') and text like '%po_client_insert_man_log%'`

## 查看不包含数字

`select * from customer a where PATINDEX('%[0-9]%', a.name) = 0 `

## 查询SQL执行时CPU的占用情况

```sql
WITH AggregatedCPU AS (SELECT q.query_hash, SUM(count_executions * avg_cpu_time / 1000.0) AS total_cpu_millisec, SUM(count_executions * avg_cpu_time / 1000.0)/ SUM(count_executions) AS avg_cpu_millisec, MAX(rs.max_cpu_time / 1000.00) AS max_cpu_millisec, MIN(rs.min_cpu_time / 1000.00) AS min_cpu_millisec,MAX(max_logical_io_reads) max_logical_reads, COUNT(DISTINCT p.plan_id) AS number_of_distinct_plans, COUNT(DISTINCT p.query_id) AS number_of_distinct_query_ids, SUM(CASE WHEN rs.execution_type_desc='Aborted' THEN count_executions ELSE 0 END) AS Aborted_Execution_Count, SUM(CASE WHEN rs.execution_type_desc='Regular' THEN count_executions ELSE 0 END) AS Regular_Execution_Count, SUM(CASE WHEN rs.execution_type_desc='Exception' THEN count_executions ELSE 0 END) AS Exception_Execution_Count, SUM(count_executions) AS total_executions, MIN(qt.query_sql_text) AS sampled_query_text
FROM sys.query_store_query_text AS qt
JOIN sys.query_store_query AS q ON qt.query_text_id=q.query_text_id
JOIN sys.query_store_plan AS p ON q.query_id=p.query_id
JOIN sys.query_store_runtime_stats AS rs ON rs.plan_id=p.plan_id
JOIN sys.query_store_runtime_stats_interval AS rsi ON rsi.runtime_stats_interval_id=rs.runtime_stats_interval_id
WHERE rs.execution_type_desc IN ('Regular', 'Aborted', 'Exception')AND rsi.start_time>='2022-05-09 06:00:00' --AND rsi.start_time<='2021-10-28 05:00:00'
GROUP BY q.query_hash), OrderedCPU AS (SELECT query_hash, total_cpu_millisec, avg_cpu_millisec, max_cpu_millisec, min_cpu_millisec,max_logical_reads, number_of_distinct_plans, number_of_distinct_query_ids, total_executions, Aborted_Execution_Count, Regular_Execution_Count, Exception_Execution_Count, sampled_query_text, ROW_NUMBER() OVER (ORDER BY total_cpu_millisec DESC, query_hash ASC) AS RN
FROM AggregatedCPU)
SELECT OD.query_hash, OD.total_cpu_millisec, OD.avg_cpu_millisec, OD.max_cpu_millisec, OD.min_cpu_millisec,OD.max_logical_reads, OD.number_of_distinct_plans, OD.number_of_distinct_query_ids, OD.total_executions, OD.Aborted_Execution_Count, OD.Regular_Execution_Count, OD.Exception_Execution_Count, OD.sampled_query_text, OD.RN
FROM OrderedCPU AS OD
WHERE OD.RN<=100 -- and sampled_query_text like '%VS9251B%'
ORDER BY total_cpu_millisec DESC;
```

## 触发器记录数据的IUD操作

```sql
#创建日志表
USE [db_bcf]
GO

SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF  exists (select * from sysobjects where type = 'U' and name = 'tatp_dus_extr_result_rtn_log')
	BEGIN
		DROP TABLE [dbo].[tatp_dus_extr_result_rtn_log]
	END


CREATE TABLE [dbo].[tatp_dus_extr_result_rtn_log]
(
    [file_content] [char](150) NOT NULL,
    [spid] int NULL,
    [prog_name] [varchar](100) NULL,
    [user_name] [varchar](100) NULL,
    [hostname] varchar(100) NULL,
    [ip_address] varchar(100) NULL,
    [action_type] [varchar](40)  NULL,
    [update_time] [datetime]  NULL
)
GO

#创建触发器
USE [db_bcf]
GO

IF OBJECT_ID ('dbo.trg_tatp_dus_extr_result_rtn', 'TR') IS NOT NULL
	BEGIN
		DROP TRIGGER  [dbo].[trg_tatp_dus_extr_result_rtn]
	END
GO

SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TRIGGER [dbo].[trg_tatp_dus_extr_result_rtn]
ON [dbo].[tatp_dus_extr_result_rtn] FOR UPDATE,INSERT,DELETE
AS
BEGIN

declare @op varchar(10)
select @op=case when exists(select 1 from inserted) and exists(select 1 from deleted)
                    then 'Update'
                when exists(select 1 from inserted) and not exists(select 1 from deleted)
                    then 'Insert'
                when not exists(select 1 from inserted) and exists(select 1 from deleted)
                    then 'Delete' end

if @op = 'Update'
    begin
        insert into [dbo].[tatp_dus_extr_result_rtn_log]
        (file_content,spid,prog_name,user_name,hostname,ip_address,action_type,update_time)
        select file_content,
               @@spid,
               (select program_name as prog_name from sys.dm_exec_sessions where session_id=@@spid),
               (select login_name as user_name from sys.dm_exec_sessions where session_id=@@spid),
               (select hostname as hostname from sys.sysprocesses where spid=@@spid),
               (select client_net_address as ip_address from sys.dm_exec_connections where session_id=@@spid),
               'Before Update',
               getdate()
        from deleted

        insert into [dbo].[tatp_dus_extr_result_rtn_log]
        (file_content,spid,prog_name,user_name,hostname,ip_address,action_type,update_time)
        select file_content,
               @@spid,
               (select program_name as prog_name from sys.dm_exec_sessions where session_id=@@spid),
               (select login_name as user_name from sys.dm_exec_sessions where session_id=@@spid),
               (select hostname as hostname from sys.sysprocesses where spid=@@spid),
               (select client_net_address as ip_address from sys.dm_exec_connections where session_id=@@spid),
               'After Update',
               getdate()
        from inserted
    end
else if @op = 'Delete'
    begin
        insert into [dbo].[tatp_dus_extr_result_rtn_log]
        (file_content,spid,prog_name,user_name,hostname,ip_address,action_type,update_time)
        select file_content,
               @@spid,
               (select program_name as prog_name from sys.dm_exec_sessions where session_id=@@spid),
               (select login_name as user_name from sys.dm_exec_sessions where session_id=@@spid),
               (select hostname as hostname from sys.sysprocesses where spid=@@spid),
               (select client_net_address as ip_address from sys.dm_exec_connections where session_id=@@spid),
               @op,
               getdate()
        from deleted
    end
else
    begin
        insert into [dbo].[tatp_dus_extr_result_rtn_log]
        (file_content,spid,prog_name,user_name,hostname,ip_address,action_type,update_time)
        select file_content,
               @@spid,
               (select program_name as prog_name from sys.dm_exec_sessions where session_id=@@spid),
               (select login_name as user_name from sys.dm_exec_sessions where session_id=@@spid),
               (select hostname as hostname from sys.sysprocesses where spid=@@spid),
               (select client_net_address as ip_address from sys.dm_exec_connections where session_id=@@spid),
               @op,
               getdate()
        from inserted
    end
END
GO
```

## ISNULL与NULLIF函数区别（SQL Server）
`ISNULL(check_expression, replacement_value)`
- 如果 check_expression 为 NULL，则返回 replacement_value
- 如果 check_expression 不为 NULL，则返回 check_expression

`NULLIF(expression, expression)`
- 如果两个 expression 相等，则返回 NULL，该 NULL 为第一个 expression 的数据类型
- 如果两个 expression 不相等，则返回第一个 expression
---

# Excel

**生成SQL：**

1. 这一列的文本类型选 `general `
2.  公式文本框输入`="'"&A1&"',"`

**快速填充整列：**

1. 左上角输入 D2:D414716，按住ctrl+enter，光标移到公式后，继续按住ctrl+enter

**判断两列内容是否相同：**

1. 这一列的文本类型选 `general `
2. `=IF((A1=B1),TRUE,FALSE)`

**判断C列中的内容有没有在B列中出现过：**

1. 这一列的文本类型选 `general `
2. `=countif(B:B,C2)`
3. 结果为0的，就是B列中没有出现过的，而结果不为0，则是在B列中出现过

**快速生成字母开头的连续序号：**
1. 这一列的文本类型选 `general `
2. `="T"&TEXT(0+ROW(A1),"0000")`
3. 第一行就是T0001,下一行就是T0002

---

# Note pad++

**行前行尾添加字符：**

1. 按Ctrl + H 打开替换窗口
2. 查找模式选择正则表达式
3. 输入 ^  行前添加字符串
4. 输入 $  行尾添加字符串

**删除包含某字符串的行：**

1. 按Ctrl + H 打开替换窗口
2. 查找模式选择正则表达式
3. 查找目标输入`.*DROP.*`，DROP是想替换的关键词
4. 全部替换

**删除空白行：**

1. 按Ctrl + H 打开替换窗口
2. 查找模式选择正则表达式
3. 查找目标输入`^\s` 
4. 全部替换

---

# 单元测试

[JMockit中文网](http://jmockit.cn/index.htm)
