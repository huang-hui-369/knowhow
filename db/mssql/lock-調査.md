# find lock sql

## 前提条件

https://raw.githubusercontent.com/MasayukiOzawa/SQLServer-Util/master/Lock/%E3%83%96%E3%83%AD%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%83%81%E3%82%A7%E3%83%BC%E3%83%B3%E3%81%AE%E5%8F%96%E5%BE%97.sql

需要賦予权限
```sql
use master
go
GRANT VIEW SERVER STATE TO user1;
```

## sql
```sql
DECLARE @headblocker bit = 0	-- Head Blocker のみを取得するか (0 : ブロッキングをすべて取得 / 1 : Head Blocker のみを取得)
DECLARE @level int = 100		-- 何レベルまで取得するか

;WITH 
-- プロセス一覧
sp AS(
	SELECT 
		spid, 
		blocked, 
		cmd, 
		lastwaittype,
		waitresource,
		status, 
		open_tran,
		text 
	FROM sys.sysprocesses 
	CROSS APPLY sys.dm_exec_sql_text(sql_handle)
	WHERE spid > 50
),
-- ブロッキングリスト
BlockList AS(

--  Blocker
SELECT
	spid, 
	CAST(blocked AS varchar(100)) AS blocked,
	1 AS level, 
	1 AS is_HeadBlocker,
	CAST(RIGHT(REPLICATE('0', 8) + CAST(spid AS varchar(10)), 8) AS varchar(100)) AS blocked_chain,
	CAST('' AS varchar(100)) AS blocked_path, 
	RTRIM(cmd) AS cmd,
	RTRIM(lastwaittype) AS lastwaittype,
	RTRIM(waitresource) AS waitresource,
	RTRIM(status) AS status,
	open_tran,
	text
FROM
	sp
WHERE
	blocked = 0
	AND
	spid in (SELECT blocked FROM sp WHERE blocked <> 0)
UNION ALL

--  Blocked
SELECT
	r.spid, 
	CAST(r.blocked AS varchar(100)),
	BlockList.level + 1 AS level,
	0 AS is_HeadBlocker,
	CAST(BlockList.blocked_chain + CAST(r.spid AS varchar(10)) AS varchar(100)) AS blocked_chain,
	CAST(
		CASE BlockList.blocked_path 
		WHEN '' THEN CAST(BlockList.spid AS varchar(10))
		ELSE BlockList.blocked_path + '->' + CAST(r.blocked AS varchar(10)) 
		END
		AS varchar(100)
	) , 
	RTRIM(r.cmd) AS cmd,
	RTRIM(r.lastwaittype) AS lastwaittype,
	RTRIM(r.waitresource) AS waitresource,
	RTRIM(r.status) AS status,
	r.open_tran,
	r.text
FROM
	sp r
	INNER JOIN
	BlockList
	ON
	r.blocked = BlockList.spid
)
-- ブロッキング情報の取得
SELECT 
	BlockList.level, 
	BlockList.spid, 
	is_HeadBlocker,
	CASE 
		WHEN BlockList.blocked_path = '' THEN ''
		ELSE BlockList.blocked_path + '->' + CAST(BlockList.spid AS varchar(10))
	END	AS blocked_path,
	er.start_time,
	at.transaction_begin_time,
	datediff(SECOND, er.start_time,GETDATE()) AS elapsed_time_sec,
	at.name AS transaction_name,
	CASE at.transaction_type -- 読み取り専用トランザクションのトランザクション経過時間はトランザクション解析のノイズになる可能性があるため、Elapsed で判断
		WHEN 2 THEN NULL
		ELSE datediff(SECOND, at.transaction_begin_time, GETDATE())
	END AS transaction_elapsed_time_sec,
	er.wait_time,
	er.status AS er_status,
	BlockList.cmd, 
	er.wait_type,
	BlockList.lastwaittype, 
	er.wait_resource AS er_wait_resource,
	BlockList.waitresource AS BlockList_wait_resource, 
	BlockList.status AS blocklist_status, 
	es.host_name,
	es.program_name,
	es.login_name,
	er.open_transaction_count,
	BlockList.open_tran AS sysprocess_open_tran_count,
	CASE at.transaction_type
		WHEN 1 THEN N'読み取り/書き込み'
		WHEN 2 THEN N'読み取り専用'
		WHEN 3 THEN N'システム'
		WHEN 4 THEN N'分散トランザクション'
		ELSE CAST(at.transaction_type AS nvarchar(50))
	END AS transaction_type,
	CASE at.transaction_state
		WHEN 1 THEN N'初期化待ち'
		WHEN 1 THEN N'開始待ち'
		WHEN 2 THEN N'アクティブ'
		WHEN 3 THEN N'終了'
		WHEN 4 THEN N'分散トランザクションコミット開始'
		WHEN 5 THEN N'解決待ち'
		WHEN 6 THEN N'コミット完了'
		WHEN 7 THEN N'ロールバック中'
		WHEN 8 THEN N'ロールバック完了'
		ELSE CAST(at.transaction_type AS nvarchar(50))
	END AS transaction_state,		
	at.transaction_id,
	at.transaction_uow,
	BlockList.text,
		SUBSTRING(st.text, (er.statement_start_offset/2)+1,   
	((CASE er.statement_end_offset  
		WHEN -1 THEN DATALENGTH(st.text)  
		ELSE er.statement_end_offset  
		END - er.statement_start_offset)/2) + 1) AS statement_text ,
	st.text AS exec_request_text,
	qp.query_plan

FROM 
	BlockList
	LEFT JOIN sys.dm_exec_requests AS er ON er.session_id = BlockList.spid
	LEFT JOIN sys.dm_exec_sessions as es ON es.session_id = BlockList.spid
	
	LEFT JOIN sys.dm_tran_session_transactions AS tst WITH(NOLOCK) ON tst.session_id = BlockList.spid
	LEFT JOIN sys.dm_tran_active_transactions AS at ON at.transaction_id = tst.transaction_id

	OUTER APPLY sys.dm_exec_query_plan(er.plan_handle) AS qp
	OUTER APPLY sys.dm_exec_sql_text(er.sql_handle) AS st
WHERE
	is_HeadBlocker >= @headblocker
	AND
	level BETWEEN 1 AND @level
ORDER BY 
	level ASC,
	blocked_chain ASC,
	spid 
	ASC
OPTION (MAXRECURSION 100, RECOMPILE)
```
## 执行结果
```
level	spid	is_HeadBlocker	blocked_path	start_time	transaction_begin_time	elapsed_time_sec	transaction_name	transaction_elapsed_time_sec	wait_time	er_status	cmd	wait_type	lastwaittype	er_wait_resource	BlockList_wait_resource	blocklist_status	host_name	program_name	login_name	open_transaction_count	sysprocess_open_tran_count	transaction_type	transaction_state	transaction_id	transaction_uow	text	statement_text	exec_request_text	query_plan
1	59	1		2021-09-14 18:13:35.830	2021-09-14 18:13:28.633	60543	implicit_transaction	60550	60542919	running	EXECUTE	MSQL_XP	PREEMPTIVE_OS_GETPROCADDRESS			runnable	GUT-D2019-06	Microsoft JDBC Driver for SQL Server	sai1	1	1	読み取り/書き込み	アクティブ	1056873	NULL	xp_ora2ms_exec2_ex	xp_ora2ms_exec2_ex	xp_ora2ms_exec2_ex	NULL
2	102	0	59->102	2021-09-14 18:13:35.850	NULL	60543	NULL	NULL	60543965	suspended	SELECT	LCK_M_U	LCK_M_U	KEY: 9:72057594236895232 (56aacad743b4)	KEY: 9:72057594236895232 (56aacad743b4)	suspended	GUT-D2019-06	Microsoft SQL Server (SSMA Extensions)	sai1	0	0	NULL	NULL	NULL	NULL	CREATE PROCEDURE [SAI16].[GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_SIKKOU$IMPL]       @TARGET_ID numeric(9, 0),         @return_value_argument float(53)  OUTPUT  AS      BEGIN          DECLARE           @TMP_DATA_ZYOUTAI float(53)          BEGIN TRY             EXECUTE ssma_oracle.db_fn_check_init_package 'SAI16', 'GET_DATA_ZYOUTAI_FOR_UPDATE'             /* 連携データテーブルにレコードがあるかチェック。*/           SELECT @TMP_DATA_ZYOUTAI = SIKKOU.DATA_ZYOUTAI           FROM SAI16.SIKKOU               WITH ( UPDLOCK )           WHERE SIKKOU.SIKKOU_ID = @TARGET_ID             EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT             IF @TMP_DATA_ZYOUTAI IN (  SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP3() )              BEGIN                   SET @return_value_argument = SAI16.GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_ANKEN(SAI16.GET_BODY_ID$FUNC_SIKKOU(@TARGET_ID))                   RETURN                 END           ELSE               BEGIN                   SET @return_value_argument = @TMP_DATA_ZYOUTAI                   RETURN                 END          END TRY          BEGIN CATCH             DECLARE              @errornumber int             SET @errornumber = ERROR_NUMBER()             DECLARE              @errormessage nvarchar(4000)             SET @errormessage = ERROR_MESSAGE()             DECLARE              @exceptionidentifier nvarchar(4000)             SELECT @exceptionidentifier = ssma_oracle.db_error_get_oracle_exception_id(@errormessage, @errornumber)             IF (@exceptionidentifier LIKE N'ORA-00100%')              /* 連携データCSVテーブルにレコードがあるかチェック。あるならSTEP0完了。*/              BEGIN                   BEGIN TRY                      SELECT @TMP_DATA_ZYOUTAI = SIKKOU_CSV.SIKKOU_ID                    FROM SAI16.SIKKOU_CSV                        WITH ( UPDLOCK )                    WHERE SIKKOU_CSV.SIKKOU_ID = @TARGET_ID                      EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT                      SET @return_value_argument = SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP0()                      RETURN                    END TRY                   BEGIN CATCH                      DECLARE                       @errornumber$2 int                      SET @errornumber$2 = ERROR_NUMBER()                      DECLARE                       @errormessage$2 nvarchar(4000)                      SET @errormessage$2 = ERROR_MESSAGE()                      DECLARE                       @exceptionidentifier$2 nvarchar(4000)                      SELECT @exceptionidentifier$2 = ssma_oracle.db_error_get_oracle_exception_id(@errormessage$2, @errornumber$2)                      IF (@exceptionidentifier$2 LIKE N'ORA-00100%')                       BEGIN                            SET @return_value_argument = NULL                            RETURN                          END                    ELSE                        BEGIN                          BEGIN                             IF (SAI16.IS_NOT_NULL(@exceptionidentifier$2) = 1)                                BEGIN                                   IF @errornumber$2 = 59998                                      BEGIN                                         EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier$2                                         RAISERROR(59998, 16, 1, @exceptionidentifier$2)                                      END                                     ELSE                                       BEGIN                                         EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier$2                                         RAISERROR(59999, 16, 1, @exceptionidentifier$2)                                      END                                  END                             ELSE                                 BEGIN                                   EXECUTE SAI16.rethrowerror                                END                          END                       END                   END CATCH                END           ELSE               BEGIN                 BEGIN                    IF (SAI16.IS_NOT_NULL(@exceptionidentifier) = 1)                       BEGIN                          IF @errornumber = 59998                             BEGIN                                EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier                                RAISERROR(59998, 16, 1, @exceptionidentifier)                             END                            ELSE                              BEGIN                                EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier                                RAISERROR(59999, 16, 1, @exceptionidentifier)                             END                         END                    ELSE                        BEGIN                          EXECUTE SAI16.rethrowerror                       END                 END              END          END CATCH       END  	SELECT @TMP_DATA_ZYOUTAI = SIKKOU.DATA_ZYOUTAI           FROM SAI16.SIKKOU               WITH ( UPDLOCK )           WHERE SIKKOU.SIKKOU_ID = @TARGET_ID	CREATE PROCEDURE [SAI16].[GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_SIKKOU$IMPL]       @TARGET_ID numeric(9, 0),         @return_value_argument float(53)  OUTPUT  AS      BEGIN          DECLARE           @TMP_DATA_ZYOUTAI float(53)          BEGIN TRY             EXECUTE ssma_oracle.db_fn_check_init_package 'SAI16', 'GET_DATA_ZYOUTAI_FOR_UPDATE'             /* 連携データテーブルにレコードがあるかチェック。*/           SELECT @TMP_DATA_ZYOUTAI = SIKKOU.DATA_ZYOUTAI           FROM SAI16.SIKKOU               WITH ( UPDLOCK )           WHERE SIKKOU.SIKKOU_ID = @TARGET_ID             EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT             IF @TMP_DATA_ZYOUTAI IN (  SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP3() )              BEGIN                   SET @return_value_argument = SAI16.GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_ANKEN(SAI16.GET_BODY_ID$FUNC_SIKKOU(@TARGET_ID))                   RETURN                 END           ELSE               BEGIN                   SET @return_value_argument = @TMP_DATA_ZYOUTAI                   RETURN                 END          END TRY          BEGIN CATCH             DECLARE              @errornumber int             SET @errornumber = ERROR_NUMBER()             DECLARE              @errormessage nvarchar(4000)             SET @errormessage = ERROR_MESSAGE()             DECLARE              @exceptionidentifier nvarchar(4000)             SELECT @exceptionidentifier = ssma_oracle.db_error_get_oracle_exception_id(@errormessage, @errornumber)             IF (@exceptionidentifier LIKE N'ORA-00100%')              /* 連携データCSVテーブルにレコードがあるかチェック。あるならSTEP0完了。*/              BEGIN                   BEGIN TRY                      SELECT @TMP_DATA_ZYOUTAI = SIKKOU_CSV.SIKKOU_ID                    FROM SAI16.SIKKOU_CSV                        WITH ( UPDLOCK )                    WHERE SIKKOU_CSV.SIKKOU_ID = @TARGET_ID                      EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT                      SET @return_value_argument = SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP0()                      RETURN                    END TRY                   BEGIN CATCH                      DECLARE                       @errornumber$2 int                      SET @errornumber$2 = ERROR_NUMBER()                      DECLARE                       @errormessage$2 nvarchar(4000)                      SET @errormessage$2 = ERROR_MESSAGE()                      DECLARE                       @exceptionidentifier$2 nvarchar(4000)                      SELECT @exceptionidentifier$2 = ssma_oracle.db_error_get_oracle_exception_id(@errormessage$2, @errornumber$2)                      IF (@exceptionidentifier$2 LIKE N'ORA-00100%')                       BEGIN                            SET @return_value_argument = NULL                            RETURN                          END                    ELSE                        BEGIN                          BEGIN                             IF (SAI16.IS_NOT_NULL(@exceptionidentifier$2) = 1)                                BEGIN                                   IF @errornumber$2 = 59998                                      BEGIN                                         EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier$2                                         RAISERROR(59998, 16, 1, @exceptionidentifier$2)                                      END                                     ELSE                                       BEGIN                                         EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier$2                                         RAISERROR(59999, 16, 1, @exceptionidentifier$2)                                      END                                  END                             ELSE                                 BEGIN                                   EXECUTE SAI16.rethrowerror                                END                          END                       END                   END CATCH                END           ELSE               BEGIN                 BEGIN                    IF (SAI16.IS_NOT_NULL(@exceptionidentifier) = 1)                       BEGIN                          IF @errornumber = 59998                             BEGIN                                EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier                                RAISERROR(59998, 16, 1, @exceptionidentifier)                             END                            ELSE                              BEGIN                                EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier                                RAISERROR(59999, 16, 1, @exceptionidentifier)                             END                         END                    ELSE                        BEGIN                          EXECUTE SAI16.rethrowerror                       END                 END              END          END CATCH       END  	<ShowPlanXML xmlns="http://schemas.microsoft.com/sqlserver/2004/07/showplan" Version="1.6" Build="14.0.2037.2"><BatchSequence><Batch><Statements><StmtSimple StatementText="CREATE PROCEDURE [SAI16].[GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_SIKKOU$IMPL]  &#xD;&#xA;   @TARGET_ID numeric(9, 0),&#xD;&#xA;&#xD;&#xA;&#xD;&#xA;   @return_value_argument float(53)  OUTPUT&#xD;&#xA;AS &#xD;&#xA;   BEGIN&#xD;&#xA;&#xD;&#xA;      DECLARE&#xD;&#xA;         @TMP_DATA_ZYOUTAI float(53)&#xD;&#xA;&#xD;&#xA;      BEGIN TRY&#xD;&#xA;&#xD;&#xA;         " StatementId="1" StatementCompId="2" StatementType="BEGIN TRY" RetrievedFromCache="true" /><StmtSimple StatementText="EXECUTE ssma_oracle.db_fn_check_init_package 'SAI16', 'GET_DATA_ZYOUTAI_FOR_UPDATE'" StatementId="2" StatementCompId="3" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;         /* 連携データテーブルにレコードがあるかチェック。*/&#xD;&#xA;         SELECT @TMP_DATA_ZYOUTAI = SIKKOU.DATA_ZYOUTAI&#xD;&#xA;         FROM SAI16.SIKKOU &#xD;&#xA;            WITH ( UPDLOCK )&#xD;&#xA;         WHERE SIKKOU.SIKKOU_ID = @TARGET_ID" StatementId="3" StatementCompId="4" StatementType="SELECT" RetrievedFromCache="true" StatementSubTreeCost="0.0032832" StatementEstRows="1" SecurityPolicyApplied="false" StatementOptmLevel="TRIVIAL" QueryHash="0xD3AC67998FE50254" QueryPlanHash="0xED07D4FAB6AF8EC4" CardinalityEstimationModelVersion="140"><StatementSetOptions QUOTED_IDENTIFIER="true" ARITHABORT="false" CONCAT_NULL_YIELDS_NULL="true" ANSI_NULLS="true" ANSI_PADDING="true" ANSI_WARNINGS="true" NUMERIC_ROUNDABORT="false" /><QueryPlan CachedPlanSize="16" CompileTime="0" CompileCPU="0" CompileMemory="464"><MemoryGrantInfo SerialRequiredMemory="0" SerialDesiredMemory="0" /><OptimizerHardwareDependentProperties EstimatedAvailableMemoryGrant="415290" EstimatedPagesCached="103822" EstimatedAvailableDegreeOfParallelism="2" MaxCompileMemory="2993296" /><RelOp NodeId="0" PhysicalOp="Compute Scalar" LogicalOp="Compute Scalar" EstimateRows="1" EstimateIO="0" EstimateCPU="1e-007" AvgRowSize="15" EstimatedTotalSubtreeCost="0.0032832" Parallel="0" EstimateRebinds="0" EstimateRewinds="0" EstimatedExecutionMode="Row"><OutputList><ColumnReference Column="Expr1002" /></OutputList><ComputeScalar><DefinedValues><DefinedValue><ColumnReference Column="Expr1002" /><ScalarOperator ScalarString="CONVERT_IMPLICIT(float(53),[SAITAMA].[SAI16].[SIKKOU].[DATA_ZYOUTAI],0)"><Convert DataType="float" Precision="53" Style="0" Implicit="1"><ScalarOperator><Identifier><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU]" Column="DATA_ZYOUTAI" /></Identifier></ScalarOperator></Convert></ScalarOperator></DefinedValue></DefinedValues><RelOp NodeId="1" PhysicalOp="Clustered Index Seek" LogicalOp="Clustered Index Seek" EstimateRows="1" EstimatedRowsRead="1" EstimateIO="0.003125" EstimateCPU="0.0001581" AvgRowSize="12" EstimatedTotalSubtreeCost="0.0032831" TableCardinality="98487" Parallel="0" EstimateRebinds="0" EstimateRewinds="0" EstimatedExecutionMode="Row"><OutputList><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU]" Column="DATA_ZYOUTAI" /></OutputList><IndexScan Ordered="1" ScanDirection="FORWARD" ForcedIndex="0" ForceSeek="0" ForceScan="0" NoExpandHint="0" Storage="RowStore"><DefinedValues><DefinedValue><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU]" Column="DATA_ZYOUTAI" /></DefinedValue></DefinedValues><Object Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU]" Index="[PK_SIKKOU]" IndexKind="Clustered" Storage="RowStore" /><SeekPredicates><SeekPredicateNew><SeekKeys><Prefix ScanType="EQ"><RangeColumns><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU]" Column="SIKKOU_ID" /></RangeColumns><RangeExpressions><ScalarOperator ScalarString="[@TARGET_ID]"><Identifier><ColumnReference Column="@TARGET_ID" /></Identifier></ScalarOperator></RangeExpressions></Prefix></SeekKeys></SeekPredicateNew></SeekPredicates></IndexScan></RelOp></ComputeScalar></RelOp><ParameterList><ColumnReference Column="@TARGET_ID" ParameterDataType="numeric(9,0)" ParameterCompiledValue="(270800.)" /></ParameterList></QueryPlan></StmtSimple><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;         EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT" StatementId="4" StatementCompId="5" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtCond StatementText="&#xD;&#xA;&#xD;&#xA;         IF @TMP_DATA_ZYOUTAI IN (  SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP3() )" StatementId="5" StatementCompId="6" StatementType="COND WITH QUERY" RetrievedFromCache="true"><Condition /><Then><Statements><StmtSimple StatementText="&#xD;&#xA;            BEGIN&#xD;&#xA;&#xD;&#xA;               SET @return_value_argument = SAI16.GET_DATA_ZYOUTAI_FOR_UPDATE$FUNC_ANKEN(SAI16.GET_BODY_ID$FUNC_SIKKOU(@TARGET_ID))" StatementId="6" StatementCompId="7" StatementType="ASSIGN WITH QUERY" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;               RETURN" StatementId="7" StatementCompId="8" StatementType="RETURN NONE" RetrievedFromCache="true" /></Statements></Then><Else><Statements><StmtSimple StatementText=" &#xD;&#xA;&#xD;&#xA;            END&#xD;&#xA;         ELSE &#xD;&#xA;            BEGIN&#xD;&#xA;&#xD;&#xA;               SET @return_value_argument = @TMP_DATA_ZYOUTAI&#xD;&#xA;&#xD;&#xA;               " StatementId="8" StatementCompId="11" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="RETURN" StatementId="9" StatementCompId="12" StatementType="RETURN NONE" RetrievedFromCache="true" /></Statements></Else></StmtCond><StmtSimple StatementText=" &#xD;&#xA;&#xD;&#xA;            END&#xD;&#xA;&#xD;&#xA;      END TRY&#xD;&#xA;&#xD;&#xA;      " StatementId="10" StatementCompId="14" StatementType="END TRY" RetrievedFromCache="true" /><StmtSimple StatementText="BEGIN CATCH&#xD;&#xA;&#xD;&#xA;         " StatementId="11" StatementCompId="15" StatementType="BEGIN CATCH" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;            @errornumber int&#xD;&#xA;&#xD;&#xA;         SET @errornumber = ERROR_NUMBER()&#xD;&#xA;&#xD;&#xA;         " StatementId="12" StatementCompId="16" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;            @errormessage nvarchar(4000)&#xD;&#xA;&#xD;&#xA;         SET @errormessage = ERROR_MESSAGE()&#xD;&#xA;&#xD;&#xA;         " StatementId="13" StatementCompId="17" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;            @exceptionidentifier nvarchar(4000)&#xD;&#xA;&#xD;&#xA;         SELECT @exceptionidentifier = ssma_oracle.db_error_get_oracle_exception_id(@errormessage, @errornumber)" StatementId="14" StatementCompId="18" StatementType="ASSIGN WITH QUERY" RetrievedFromCache="true" /><StmtCond StatementText="&#xD;&#xA;&#xD;&#xA;         IF (@exceptionidentifier LIKE N'ORA-00100%')" StatementId="15" StatementCompId="19" StatementType="COND" RetrievedFromCache="true"><Condition /><Then><Statements><StmtSimple StatementText="&#xD;&#xA;            /* 連携データCSVテーブルにレコードがあるかチェック。あるならSTEP0完了。*/&#xD;&#xA;            BEGIN&#xD;&#xA;&#xD;&#xA;               BEGIN TRY&#xD;&#xA;&#xD;&#xA;                  " StatementId="16" StatementCompId="20" StatementType="BEGIN TRY" RetrievedFromCache="true" /><StmtSimple StatementText="SELECT @TMP_DATA_ZYOUTAI = SIKKOU_CSV.SIKKOU_ID&#xD;&#xA;                  FROM SAI16.SIKKOU_CSV &#xD;&#xA;                     WITH ( UPDLOCK )&#xD;&#xA;                  WHERE SIKKOU_CSV.SIKKOU_ID = @TARGET_ID" StatementId="17" StatementCompId="21" StatementType="SELECT" RetrievedFromCache="true" StatementSubTreeCost="0.0032832" StatementEstRows="1" SecurityPolicyApplied="false" StatementOptmLevel="TRIVIAL" QueryHash="0x5BDAA5FE70CA4566" QueryPlanHash="0xA493F582DC263F9C" CardinalityEstimationModelVersion="140"><StatementSetOptions QUOTED_IDENTIFIER="true" ARITHABORT="false" CONCAT_NULL_YIELDS_NULL="true" ANSI_NULLS="true" ANSI_PADDING="true" ANSI_WARNINGS="true" NUMERIC_ROUNDABORT="false" /><QueryPlan CachedPlanSize="16" CompileTime="0" CompileCPU="0" CompileMemory="512"><MemoryGrantInfo SerialRequiredMemory="0" SerialDesiredMemory="0" /><OptimizerHardwareDependentProperties EstimatedAvailableMemoryGrant="415290" EstimatedPagesCached="103822" EstimatedAvailableDegreeOfParallelism="2" MaxCompileMemory="2993296" /><RelOp NodeId="0" PhysicalOp="Compute Scalar" LogicalOp="Compute Scalar" EstimateRows="1" EstimateIO="0" EstimateCPU="1e-007" AvgRowSize="15" EstimatedTotalSubtreeCost="0.0032832" Parallel="0" EstimateRebinds="0" EstimateRewinds="0" EstimatedExecutionMode="Row"><OutputList><ColumnReference Column="Expr1002" /></OutputList><ComputeScalar><DefinedValues><DefinedValue><ColumnReference Column="Expr1002" /><ScalarOperator ScalarString="CONVERT_IMPLICIT(float(53),[SAITAMA].[SAI16].[SIKKOU_CSV].[SIKKOU_ID],0)"><Convert DataType="float" Precision="53" Style="0" Implicit="1"><ScalarOperator><Identifier><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU_CSV]" Column="SIKKOU_ID" /></Identifier></ScalarOperator></Convert></ScalarOperator></DefinedValue></DefinedValues><RelOp NodeId="1" PhysicalOp="Clustered Index Seek" LogicalOp="Clustered Index Seek" EstimateRows="1" EstimatedRowsRead="1" EstimateIO="0.003125" EstimateCPU="0.0001581" AvgRowSize="12" EstimatedTotalSubtreeCost="0.0032831" TableCardinality="5589" Parallel="0" EstimateRebinds="0" EstimateRewinds="0" EstimatedExecutionMode="Row"><OutputList><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU_CSV]" Column="SIKKOU_ID" /></OutputList><IndexScan Ordered="1" ScanDirection="FORWARD" ForcedIndex="0" ForceSeek="0" ForceScan="0" NoExpandHint="0" Storage="RowStore"><DefinedValues><DefinedValue><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU_CSV]" Column="SIKKOU_ID" /></DefinedValue></DefinedValues><Object Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU_CSV]" Index="[PK_SIKKOU_CSV]" IndexKind="Clustered" Storage="RowStore" /><SeekPredicates><SeekPredicateNew><SeekKeys><Prefix ScanType="EQ"><RangeColumns><ColumnReference Database="[SAITAMA]" Schema="[SAI16]" Table="[SIKKOU_CSV]" Column="SIKKOU_ID" /></RangeColumns><RangeExpressions><ScalarOperator ScalarString="[@TARGET_ID]"><Identifier><ColumnReference Column="@TARGET_ID" /></Identifier></ScalarOperator></RangeExpressions></Prefix></SeekKeys></SeekPredicateNew></SeekPredicates></IndexScan></RelOp></ComputeScalar></RelOp><ParameterList><ColumnReference Column="@TARGET_ID" ParameterDataType="numeric(9,0)" ParameterCompiledValue="(270800.)" /></ParameterList></QueryPlan></StmtSimple><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;                  EXECUTE ssma_oracle.db_error_exact_one_row_check @@ROWCOUNT" StatementId="18" StatementCompId="22" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;                  SET @return_value_argument = SAI16.COMMON_LIBRARY$C$DATA_ZYOUTAI_STEP0()" StatementId="19" StatementCompId="23" StatementType="ASSIGN WITH QUERY" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;&#xD;&#xA;                  RETURN" StatementId="20" StatementCompId="24" StatementType="RETURN NONE" RetrievedFromCache="true" /><StmtSimple StatementText=" &#xD;&#xA;&#xD;&#xA;               END TRY&#xD;&#xA;&#xD;&#xA;               " StatementId="21" StatementCompId="25" StatementType="END TRY" RetrievedFromCache="true" /><StmtSimple StatementText="BEGIN CATCH&#xD;&#xA;&#xD;&#xA;                  " StatementId="22" StatementCompId="26" StatementType="BEGIN CATCH" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;                     @errornumber$2 int&#xD;&#xA;&#xD;&#xA;                  SET @errornumber$2 = ERROR_NUMBER()&#xD;&#xA;&#xD;&#xA;                  " StatementId="23" StatementCompId="27" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;                     @errormessage$2 nvarchar(4000)&#xD;&#xA;&#xD;&#xA;                  SET @errormessage$2 = ERROR_MESSAGE()&#xD;&#xA;&#xD;&#xA;                  " StatementId="24" StatementCompId="28" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="DECLARE&#xD;&#xA;                     @exceptionidentifier$2 nvarchar(4000)&#xD;&#xA;&#xD;&#xA;                  SELECT @exceptionidentifier$2 = ssma_oracle.db_error_get_oracle_exception_id(@errormessage$2, @errornumber$2)" StatementId="25" StatementCompId="29" StatementType="ASSIGN WITH QUERY" RetrievedFromCache="true" /><StmtCond StatementText="&#xD;&#xA;&#xD;&#xA;                  IF (@exceptionidentifier$2 LIKE N'ORA-00100%')" StatementId="26" StatementCompId="30" StatementType="COND" RetrievedFromCache="true"><Condition /><Then><Statements><StmtSimple StatementText="&#xD;&#xA;                     BEGIN&#xD;&#xA;&#xD;&#xA;                        SET @return_value_argument = NULL&#xD;&#xA;&#xD;&#xA;                        " StatementId="27" StatementCompId="31" StatementType="ASSIGN" RetrievedFromCache="true" /><StmtSimple StatementText="RETURN" StatementId="28" StatementCompId="32" StatementType="RETURN NONE" RetrievedFromCache="true" /></Statements></Then><Else><Statements><StmtCond StatementText=" &#xD;&#xA;&#xD;&#xA;                     END&#xD;&#xA;                  ELSE &#xD;&#xA;                     BEGIN&#xD;&#xA;                        BEGIN&#xD;&#xA;                           IF (SAI16.IS_NOT_NULL(@exceptionidentifier$2) = 1)" StatementId="29" StatementCompId="35" StatementType="COND WITH QUERY" RetrievedFromCache="true"><Condition /><Then><Statements><StmtCond StatementText="&#xD;&#xA;                              BEGIN&#xD;&#xA;                                 IF @errornumber$2 = 59998" StatementId="30" StatementCompId="36" StatementType="COND" RetrievedFromCache="true"><Condition /><Then><Statements><StmtSimple StatementText="&#xD;&#xA;                                    BEGIN&#xD;&#xA;                                       EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier$2" StatementId="31" StatementCompId="37" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;                                       RAISERROR(59998, 16, 1, @exceptionidentifier$2)&#xD;&#xA;                                    " StatementId="32" StatementCompId="38" StatementType="RAISERROR" RetrievedFromCache="true" /></Statements></Then><Else><Statements><StmtSimple StatementText="END&#xD;&#xA;&#xD;&#xA;                                 ELSE &#xD;&#xA;                                    BEGIN&#xD;&#xA;                                       EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier$2" StatementId="33" StatementCompId="41" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;                                       RAISERROR(59999, 16, 1, @exceptionidentifier$2)&#xD;&#xA;                                    " StatementId="34" StatementCompId="42" StatementType="RAISERROR" RetrievedFromCache="true" /></Statements></Else></StmtCond></Statements></Then><Else><Statements><StmtSimple StatementText="END&#xD;&#xA;&#xD;&#xA;                              END&#xD;&#xA;                           ELSE &#xD;&#xA;                              BEGIN&#xD;&#xA;                                 EXECUTE SAI16.rethrowerror" StatementId="35" StatementCompId="46" StatementType="EXECUTE PROC" RetrievedFromCache="true" /></Statements></Else></StmtCond></Statements></Else></StmtCond><StmtSimple StatementText="&#xD;&#xA;                              END&#xD;&#xA;                        END&#xD;&#xA;                     END&#xD;&#xA;&#xD;&#xA;               END CATCH" StatementId="36" StatementCompId="49" StatementType="END CATCH" RetrievedFromCache="true" /></Statements></Then><Else><Statements><StmtCond StatementText="&#xD;&#xA;&#xD;&#xA;            END&#xD;&#xA;         ELSE &#xD;&#xA;            BEGIN&#xD;&#xA;               BEGIN&#xD;&#xA;                  IF (SAI16.IS_NOT_NULL(@exceptionidentifier) = 1)" StatementId="37" StatementCompId="53" StatementType="COND WITH QUERY" RetrievedFromCache="true"><Condition /><Then><Statements><StmtCond StatementText="&#xD;&#xA;                     BEGIN&#xD;&#xA;                        IF @errornumber = 59998" StatementId="38" StatementCompId="54" StatementType="COND" RetrievedFromCache="true"><Condition /><Then><Statements><StmtSimple StatementText="&#xD;&#xA;                           BEGIN&#xD;&#xA;                              EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59998 , 16 , 1 , @exceptionidentifier" StatementId="39" StatementCompId="55" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;                              RAISERROR(59998, 16, 1, @exceptionidentifier)&#xD;&#xA;                           " StatementId="40" StatementCompId="56" StatementType="RAISERROR" RetrievedFromCache="true" /></Statements></Then><Else><Statements><StmtSimple StatementText="END&#xD;&#xA;&#xD;&#xA;                        ELSE &#xD;&#xA;                           BEGIN&#xD;&#xA;                              EXECUTE SAI16.SSMA_EXCEPTION_PUSH 59999 , 16 , 1 , @exceptionidentifier" StatementId="41" StatementCompId="59" StatementType="EXECUTE PROC" RetrievedFromCache="true" /><StmtSimple StatementText="&#xD;&#xA;                              RAISERROR(59999, 16, 1, @exceptionidentifier)&#xD;&#xA;                           " StatementId="42" StatementCompId="60" StatementType="RAISERROR" RetrievedFromCache="true" /></Statements></Else></StmtCond></Statements></Then><Else><Statements><StmtSimple StatementText="END&#xD;&#xA;&#xD;&#xA;                     END&#xD;&#xA;                  ELSE &#xD;&#xA;                     BEGIN&#xD;&#xA;                        EXECUTE SAI16.rethrowerror" StatementId="43" StatementCompId="64" StatementType="EXECUTE PROC" RetrievedFromCache="true" /></Statements></Else></StmtCond></Statements></Else></StmtCond><StmtSimple StatementText="&#xD;&#xA;                     END&#xD;&#xA;               END&#xD;&#xA;            END&#xD;&#xA;&#xD;&#xA;      END CATCH" StatementId="44" StatementCompId="67" StatementType="END CATCH" RetrievedFromCache="true" /></Statements></Batch></BatchSequence></ShowPlanXML>
```


# kill sid
需要 sysadmin权限
```
KILL 53;  
GO 
```