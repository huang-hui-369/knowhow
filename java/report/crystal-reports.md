# 准备工作

1. Crystal Reports  download

   可以试用30天

   https://www.crystalreports.com/download/

2. JSON ODBC Driver download

   可以试用30天

   https://www.cdata.com/jp/drivers/json/odbc/?utm_source=google&utm_medium=cpc&utm_campaign=CData_JP_Search&utm_content=ODBC_JSON&utm_term=b|odbc%20driver%20json&gclid=CjwKCAiA4KaRBhBdEiwAZi1zzmUjE4sNRxekf6fg6JtxdCOSgh2cKdWsbgfucbUVAL3QZgrKFcPo0hoCGrAQAvD_BwE

   

# 配置JSON ODBC Driver

1. 打开 odbc datasource

   ![image-20220310184748448](D:\github\knowhow\java\report\Untitled.assets\image-20220310184748448.png)

2. 最简单的设置json文件为数据源

   设置到文件，设置到目录不好使

   ![image-20220310184931033](D:\github\knowhow\java\report\Untitled.assets\image-20220310184931033.png)

3. Test connection

4. 确认Tables

   ![image-20220310185114730](D:\github\knowhow\java\report\Untitled.assets\image-20220310185114730.png)

5. json 数据

   ```json
   [
   	{
   	    "name":"test name1"
   	    ,"age":"18"
   	    ,"AcceptHeadName":"test header1"
   
   	}
   	,{
   	    "name":"test name2"
   	    ,"age":"19"
   	    ,"AcceptHeadName":"test header2"
   
   	}
   ]
   ```

   

# 报表设计

1. 新建空白报表

   ![image-20220310182002062](D:\github\knowhow\java\report\Untitled.assets\image-20220310182002062.png)

2. 选择数据源

   选择json odbc driver 数据源

   ![image-20220310183931836](D:\github\knowhow\java\report\Untitled.assets\image-20220310183931836.png)

3. 根据选择的数据源，会显示出表的字段，拖拽表的字段来设计报表

   ![image-20220310185532514](D:\github\knowhow\java\report\Untitled.assets\image-20220310185532514.png)

4. 预览

   ![image-20220310190610760](D:\github\knowhow\java\report\Untitled.assets\image-20220310190610760.png)

   ![image-20220310190641872](D:\github\knowhow\java\report\Untitled.assets\image-20220310190641872.png)



# Crystal Reports for Visual Studio访问数据库

Crystal Reports for Visual Studio访问数据库的策略有两种方式，拉模式和推模式。

- 拉模式：

  是报表从数据库把数据拉到报表上，这种模式，是报表本身查询数据，不需要开发人员编写代码。所以在不需编码时使用拉模式。拉模式相对来说比较死板。

- 推模式：

  推模式需要开发人员编写代码以连接到数据库，查询与报表中字段匹配的数据，并把数据给报表。所以推模式在设计报表的时候，报表设计人员需要一个表格的定义格式，就是表的字段格式，来设计报表的框架。

# 推模式   source

```c#
private void Window_Loaded(object sender, RoutedEventArgs e) {
ReportDocument report = new ReportDocument();
report.Load("../../CrystalReport1.rpt");
var connectionString = ConfigurationManager.ConnectionStrings["MyAppConfigConnectionStringName"].ConnectionString;
using (JSONConnection connection = new JSONConnection(connectionString)) {
JSONDataAdapter dataAdapter = new JSONDataAdapter(
"SELECT [people].[personal.age] AS age, [people].[personal.gender] AS gender, [people].[personal.name.first] AS first_name, [people].[personal.name.last] AS last_name, [vehicles].[model], FROM [people] JOIN [vehicles] ON [people].[_id] = [vehicles].[people_id]", connection);
DataSet set = new DataSet("_set");
DataTable table = set.Tables.Add("_table");
dataAdapter.Fill(table);
report.SetDataSource(table);
}
reportViewer.ViewerCore.ReportSource = report;
// reportViewer.ViewerCore..DataBind();
}
```

