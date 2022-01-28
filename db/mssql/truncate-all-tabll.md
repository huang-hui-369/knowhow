# truncate one table

```sql
/***************************************
call sample 

DECLARE @RC int
DECLARE @SCHEMA_NAME varchar(64)
DECLARE @TBL_NAME varchar(64),
 @VERBOSE				INT
 
-- 方法: パラメーター値をここに指定します。
SET @SCHEMA_NAME = 'SAI16'
SET @TBL_NAME = 'ANKEN_GYOUSYA'
SET @VERBOSE=1

EXECUTE @RC = [SAI16].[TRUNCATE_ONE_TABLE] 
   @SCHEMA_NAME
  ,@TBL_NAME
  ,@VERBOSE
****************************************/

DROP PROCEDURE IF EXISTS [SAI16].[TRUNCATE_ONE_TABLE] 
GO
CREATE PROCEDURE [SAI16].[TRUNCATE_ONE_TABLE]
-- ALTER PROCEDURE [SAI16].[TRUNCATE_ONE_TABLE]

 @SCHEMA_NAME         VARCHAR(64),
 @TBL_NAME            VARCHAR(64),
 @VERBOSE				INT -- 1の場合、print exec sql statment, 0の場合、no print
 
AS 
BEGIN
	
/***************************************
1, DELETE ALL TABLE FK
2. TRUNCATE ALL TABLE
3. CREATE ALL TABLE FK
****************************************/

	BEGIN TRY

	-- BEGIN TRAN

		-- Drop Temporary tables
		IF OBJECT_ID('tempdb..#FK_TBLS') IS NOT NULL
			DROP TABLE #FK_TBLS

		-- 外鍵カラム情報を一時テーブルに保存する
		SELECT constraint_column_id,                           -- 外鍵カラム順番
			   OBJECT_NAME(constraint_object_id) as FK,        -- 外鍵名
			   OBJECT_NAME(parent_object_id) as FK_TBL,             -- 外鍵のテーブル
			   CLM1.name AS FK_COLUMN,                          -- 外鍵のカラム
			   OBJECT_NAME(referenced_object_id) as REF_TBL,   -- 参照のテーブル
			   CLM2.name AS REF_COLUMN                          -- 参照のカラム

		INTO #FK_TBLS
       
		FROM sys.foreign_key_columns FKC                            -- 外鍵カラム情報テーブル

			JOIN sys.columns CLM1                                   -- カラム情報テーブル
				ON FKC.parent_object_id = CLM1.object_id
				AND FKC.parent_column_id = CLM1.column_id
			JOIN sys.columns CLM2                                   -- カラム情報テーブル
				ON FKC.referenced_object_id = CLM2.object_id
				AND FKC.referenced_column_id = CLM2.column_id

		WHERE OBJECT_NAME(referenced_object_id) = @TBL_NAME 
		ORDER BY FKC.constraint_column_id,FK, FK_TBL

		declare @FK_NAME VARCHAR(128),
				@FK_TBL_NAME VARCHAR(128),
				@REF_TBL_NAME VARCHAR(128),
				@FK_COLUMN_NAME VARCHAR(128),
				@REF_COLUMN_NAME VARCHAR(128),
				@DROP_FK_SQL  NVARCHAR(MAX)
				

		-- 1, DROP ONE TABLE ALL FK
		
		declare CURS_ALL_FK CURSOR FOR 
				SELECT DISTINCT FK, FK_TBL, REF_TBL
				FROM #FK_TBLS

		OPEN CURS_ALL_FK
		-- loop all FK
		WHILE 1 = 1
		BEGIN
			FETCH CURS_ALL_FK into @FK_NAME, @FK_TBL_NAME, @REF_TBL_NAME

			IF @@FETCH_STATUS <> 0
				BREAK

			SET @DROP_FK_SQL = 'ALTER TABLE [' + @SCHEMA_NAME + '].[' + @FK_TBL_NAME  + ']  DROP CONSTRAINT [' + @FK_NAME + '] '
			IF @VERBOSE = 1 
				PRINT @DROP_FK_SQL

			EXECUTE sp_executesql @DROP_FK_SQL
		
		END
		CLOSE CURS_ALL_FK

		-- 2. TRUNCATE ALL TABLE

		declare @TRUNCATE_TBL_SQL  NVARCHAR(MAX)

		SET @TRUNCATE_TBL_SQL = 'TRUNCATE TABLE [' + @SCHEMA_NAME + '].[' + @TBL_NAME + ']' 

		IF @VERBOSE = 1 
			PRINT @TRUNCATE_TBL_SQL

		EXECUTE sp_executesql @TRUNCATE_TBL_SQL
		

		-- 3. CREATE ONE TABLE ALL FK
		declare @CREATE_FK_SQL  NVARCHAR(MAX)

		OPEN CURS_ALL_FK
		-- loop all FK
		WHILE 1 = 1
		BEGIN
			FETCH CURS_ALL_FK into @FK_NAME, @FK_TBL_NAME, @REF_TBL_NAME

			IF @@FETCH_STATUS <> 0
				BREAK

			SET @CREATE_FK_SQL = 'ALTER TABLE [' + @SCHEMA_NAME + '].[' + @FK_TBL_NAME  + ']  WITH NOCHECK ADD  CONSTRAINT [' + @FK_NAME + '] FOREIGN KEY('
			
			declare CURS_ONE_FK CURSOR FOR 
				SELECT FK_COLUMN, REF_COLUMN
				FROM #FK_TBLS
				WHERE FK = @FK_NAME
				ORDER BY constraint_column_id  -- 外鍵カラム順番(この順番ではない場合、外鍵の作成ができない) 

			OPEN CURS_ONE_FK

			-- loop all FK COLUMNS
			WHILE 1 = 1
			BEGIN
				FETCH CURS_ONE_FK into @FK_COLUMN_NAME, @REF_COLUMN_NAME

				IF @@FETCH_STATUS <> 0
					BREAK

				SET @CREATE_FK_SQL += '['+ @FK_COLUMN_NAME + '],'
				
			END
			CLOSE CURS_ONE_FK

			SET @CREATE_FK_SQL = LEFT(@CREATE_FK_SQL, LEN(@CREATE_FK_SQL) -1 ) + ') REFERENCES [' + + @SCHEMA_NAME + '].[' + @REF_TBL_NAME + '] ('; 

			OPEN CURS_ONE_FK
			-- loop all FK COLUMNS
			WHILE 1 = 1
			BEGIN
				FETCH CURS_ONE_FK into @FK_COLUMN_NAME, @REF_COLUMN_NAME

				IF @@FETCH_STATUS <> 0
					BREAK

				SET @CREATE_FK_SQL += '['+ @REF_COLUMN_NAME + '],'
				
			END
			CLOSE CURS_ONE_FK
			DEALLOCATE CURS_ONE_FK
			
			SET @CREATE_FK_SQL = LEFT(@CREATE_FK_SQL, LEN(@CREATE_FK_SQL) -1 ) + ')'

			IF @VERBOSE = 1 
				PRINT @CREATE_FK_SQL

			EXECUTE sp_executesql @CREATE_FK_SQL
			
			-- CHECK CONSTRAINT
			SET @CREATE_FK_SQL = 'ALTER TABLE [' + @SCHEMA_NAME + '].[' + @FK_TBL_NAME  + ']  CHECK CONSTRAINT [' + @FK_NAME + ']' 

			IF @VERBOSE = 1 
				PRINT @CREATE_FK_SQL

			EXECUTE sp_executesql @CREATE_FK_SQL


		END

		CLOSE CURS_ALL_FK
		DEALLOCATE CURS_ALL_FK

		-- Drop Temporary tables
		IF OBJECT_ID('tempdb..#FK_TBLS') IS NOT NULL
			DROP TABLE #FK_TBLS



	-- IF @@TRANCOUNT > 0 AND @ISCOMMIT = 1 
	--	COMMIT TRAN

	END TRY

	BEGIN CATCH
		DECLARE
            @errornumber int

         SET @errornumber = ERROR_NUMBER()

         DECLARE
            @errormessage nvarchar(4000)

         SET @errormessage = 'ERROR LINE: ' + CONVERT(VARCHAR, ERROR_LINE()) + ' ' + ERROR_MESSAGE()

		-- IF @@TRANCOUNT > 0
		--	ROLLBACK TRAN

		RAISERROR(59999, 16, 1, @errormessage)
	END CATCH

END
```



# truncate all table

```sql
/***************************************
call sample 

DECLARE @RC int
DECLARE @SCHEMA_NAME varchar(64)
 
-- 方法: パラメーター値をここに指定します。
SET @SCHEMA_NAME = 'SAI16'

EXECUTE @RC = [SAI16].[TRUNCATE_ALL_TABLE] 
   @SCHEMA_NAME
****************************************/

DROP PROCEDURE IF EXISTS [SAI16].[TRUNCATE_ALL_TABLE] 
GO
CREATE PROCEDURE [SAI16].[TRUNCATE_ALL_TABLE]

 @SCHEMA_NAME            VARCHAR(64)

AS 
BEGIN

        BEGIN TRY

        BEGIN TRAN

                declare @TBL_NAME nvarchar(max)
        
                -- get all table
                declare CURS_GET_ALL_TBL CURSOR FOR 
                                select TABLE_NAME 
                                FROM INFORMATION_SCHEMA.TABLES
                                where table_type = 'BASE TABLE' 
                                AND TABLE_SCHEMA = @SCHEMA_NAME


                OPEN CURS_GET_ALL_TBL
                -- loop all table
                WHILE 1 = 1
                BEGIN
        
                        FETCH CURS_GET_ALL_TBL into @TBL_NAME

                        IF @@FETCH_STATUS <> 0
                                BREAK
        
						-- TRUNCATE ONE table	
						DECLARE @RC int,
							@VERBOSE				INT
	
						-- 方法: パラメーター値をここに指定します。
						SET @VERBOSE=1
				

						EXECUTE @RC = [SAI16].[TRUNCATE_ONE_TABLE] 
							@SCHEMA_NAME
							,@TBL_NAME
							,@VERBOSE
	                        

                END

                CLOSE CURS_GET_ALL_TBL
                DEALLOCATE CURS_GET_ALL_TBL



        IF @@TRANCOUNT > 0
                COMMIT TRAN

        END TRY

        BEGIN CATCH
                DECLARE
            @errornumber int

			 SET @errornumber = ERROR_NUMBER()

			 DECLARE
				@errormessage nvarchar(4000)

			 SET @errormessage = ERROR_MESSAGE()

            IF @@TRANCOUNT > 0
                    ROLLBACK TRAN
            RAISERROR(59999, 16, 1, @errormessage)

        END CATCH

END
```

