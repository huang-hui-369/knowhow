# jdbc driver download

## 1. mavenからjdbc driver download

以下の三つバージョンが有ります、適当なものを選んでください。

* jre15
* jre11
* jre8


```
<!-- https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc -->
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>9.3.1.jre8-preview</version>
</dependency>

```

## 2. microsoftからdownload

sqljdbc_9.2.1.0_jpn.zipをダウンロードする。

簡単なサンプルがあります。

![](img\2021-07-05-16-58-33.png)

# a simple jdbc demo

以下の機能を実現しました。

* db connction 作成して取得する
* pool からdb connction の取得する
* select query sql を実行して結果

```java
package gut.saitama.prototype;

import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;
import java.sql.Types;

import com.beust.jcommander.JCommander;
import com.beust.jcommander.Parameter;
import com.microsoft.sqlserver.jdbc.SQLServerDataSource;
import com.microsoft.sqlserver.jdbc.SQLServerException;
import com.microsoft.sqlserver.jdbc.SQLServerXADataSource;

import jp.saitama.pref.gyousya.logic.common.util.PropertiesUtil;
import jp.saitama.pref.gyousya.logic.db.exception.ConSysException;

// 時刻の SQL Server データ型を持つ java.sql.Time を使用する場合は、sendTimeAsDatetime 接続プロパティを false に設定します。

public class MssqlJdbcDemo {
	
	@Parameter(names = {"--procedure", "-p"}, description = "call procedure COMMON_LIBRARY$FUNC_TO_NUMBER")
	private boolean isCallProcedure = false;
	@Parameter(names = {"--query", "-q"}, description = "query for sql")
	private boolean isQuery = true;
	@Parameter(names = {"--sql", "-s"}, description = "sql text")
	private String querySql = null;
	
	@Parameter(names = {"--help", "-h"}, help = true)
	private boolean isHelp;
	
	/**
	 * 
	 * execute sql query print result set
	 * 
	 * @param sql
	 */
	public static void execQuery(String sql) {
	
		try (Connection con = getCon();
				PreparedStatement pstmt = con.prepareStatement(sql);
		) {
			// Execute a query sql.
			ResultSet rs = pstmt.executeQuery();

			// Iterate through the data in the result set and display it.
			System.out.format("query for \"%s\"\n", sql);
			
			// MetaDataを取得する
			ResultSetMetaData md = rs.getMetaData();//获取键名
			int columnCount = md.getColumnCount();
			
			// print　all column name
			for (int i = 1; i <= columnCount; i++) {
				System.out.format("%s,\t\t\t ", md.getColumnName(i));
			}
			System.out.println();
			
			// print all column value
			while (rs.next()) {
				for (int i = 1; i <= columnCount; i++) {
					System.out.format("%s,\t\t\t ", rs.getObject(i));
				}
				System.out.println();
			}
		}
		// Handle any errors that may have occurred.
		catch (SQLException e) {
			e.printStackTrace();
		}
	}
	
	public static void selectAllTable() {
		// SELECT GYOUSYU_GYOUMU_L1_CODE,KETUGOU_CODE,SEISIKI_MEISYOU,MEISYOU FROM GYOUSYU_GYOUMU_L1_MASTER
		// SELECT * FROM INFORMATION_SCHEMA.TABLES;
		execQuery("SELECT * FROM INFORMATION_SCHEMA.TABLES");
	}
	
	/**
	 *  a simple call procedure demo
	 */
	public static void callProcedure() {
		
		String proc2number = "{call COMMON_LIBRARY$FUNC_TO_NUMBER$IMPL(?, ?)}";
		try (Connection con = getCon();
	    		CallableStatement cstmt = con.prepareCall(proc2number);
		){
			cstmt.setString(1, "50");
			cstmt.registerOutParameter(2, Types.INTEGER);
			cstmt.execute();
			System.out.format("call procedure: \"%s\"\n", proc2number);
			System.out.format("\tparameter1: \"%s\"\n", "50");
			System.out.format("result: %d", cstmt.getInt(2));
			
		}
		catch (SQLException e) {
			e.printStackTrace();
		} 
	}
	
	/**
	 * 
	 * create a simple db connection 
	 * @return a db connection
	 * @throws SQLServerException
	 */
	private static Connection getCon() throws SQLServerException {
		SQLServerDataSource ds = new SQLServerDataSource();
		ds.setUser("saitama");
		ds.setPassword("Grandunit123");
		ds.setServerName("localhost");
		ds.setPortNumber(Integer.parseInt("1433"));
		ds.setDatabaseName("saitama");
		return ds.getConnection();
	}
	
	
	/**
	 * @return  a db connection from pool 
	 * @throws SQLException
	 */
	private static Connection getConFromPool() throws SQLException {
		 //DataSource生成
		SQLServerXADataSource ds = new SQLServerXADataSource();
        // ユーザ名設定
		ds.setUser("saitama");
        // パスワード設定
		ds.setPassword("Grandunit123");
        // ホスト名設定
		ds.setPortNumber(Integer.parseInt("1433"));
		ds.setPortNumber(Integer.parseInt("1433"));
        // データベース名
		ds.setDatabaseName("saitama");
        
        Connection conn = ds.getConnection();
        //        DatabaseMetaData meta = conn.getMetaData();
        return conn;
	}
	
	public void exec(JCommander commander) {
		if(isHelp) {
			commander.usage();
			return;
		}
		
		if(isCallProcedure) {
			callProcedure();
		} else if(isQuery){
			if(querySql!=null) {
				execQuery(querySql);
			} else {
				selectAllTable();
			}
		} 
		
	}
	
	
	public static void main(String[] args) {
		
		MssqlJdbcDemo demo = new MssqlJdbcDemo();
		
		JCommander commander = JCommander.newBuilder().addObject(demo).build();
		commander.parse(args);
		commander.setProgramName("mssqlJdbcDemo");
		demo.exec(commander);
		
	}

}

```



# jdbc自动commit

```
jdbc:sqlserver://localhost:1433;SELECTMETHOD=cursor;asteria_autocommit=TRUE
```

