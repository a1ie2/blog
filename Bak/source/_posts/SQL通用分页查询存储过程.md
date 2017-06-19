title: SQL通用分页查询存储过程
date:  2017-05-12
tags: sql
categories: 
   - 后端
   - sql   
------

### SQL通用分页查询存储过程 ###

#### 存储过程 ####

``` bash
CREATE PROCEDURE [dbo].[QueryByPage]
@TableName NVARCHAR(50),--表名
@PageSize INT =10,--每次多少条
@PageIndex INT =1,--第几页
@Filter NVARCHAR(MAX) =NULL,--筛选条件
@SortColumn NVARCHAR(MAX) =NULL,--按哪个字段排序，这个必须赋值
@IsAsc BIT=1,--按ASC还是DESC排序,
@ShowColumn NVARCHAR(MAX) ='*'--显示的列名字

AS
--用到的变量
DECLARE @SQLStr NVARCHAR(MAX)   --最终执行的sql语句
DECLARE @CountSqlStr NVARCHAR(MAX)  --获取总共条数的sql语句
DECLARE @AscStr  NVARCHAR(100) --排序
DECLARE @FilterStr NVARCHAR(MAX) --筛选
DECLARE @Count INT --总共的条数

BEGIN

---必传参数判断
IF @TableName IS NULL OR  @TableName='' OR @SortColumn IS NULL OR @SortColumn=''
BEGIN
    RETURN
END

--如果没有填写显示的列，默认显示全部
IF @ShowColumn IS NULL OR @ShowColumn=''
BEGIN 
SET @ShowColumn='*'
END 
--1:ASC;0:DESC
IF @IsAsc=1
BEGIN
SET @AscStr =@SortColumn+' ASC ';
END
ELSE 
BEGIN
SET @AscStr =@SortColumn+' DESC ';
END

--组合筛选的SQL
IF @Filter IS  NULL OR @Filter =''
BEGIN
SET @FilterStr='';
END
ELSE
BEGIN
SET @FilterStr=' WHERE '+@Filter;
END

---拼接分页查询的SQL
SET  @SQLStr='SELECT '+@ShowColumn +' FROM ( SELECT ROW_NUMBER() OVER ( ORDER BY  '
+@AscStr+' ) AS NUM,'+@ShowColumn + ' FROM '+@TableName +@FilterStr +' )  AS T WHERE T.NUM BETWEEN ('
+CONVERT(NVARCHAR,((@PageIndex-1)*@PageSize+1))+') AND ' +CONVERT(NVARCHAR,(@PageSize*@PageIndex));

---拼接查询总条数的SQL
SET @CountSqlStr= ' SELECT @TEST=COUNT(*) FROM '+@TableName +@FilterStr  
EXEC   sp_executesql   @CountSqlStr ,N'@TEST INT OUTPUT',@Count  OUTPUT  
SELECT @Count AS Total;

---执行
EXEC(@SQLStr)
END

GO
```

#### C#调用 ####

``` bash
var sqlParas = new SqlParameter[7];
sqlParas[0] = new SqlParameter("@ShowColumn", DbType.String)
{
    Value = "Column1,Column2,Column3,Column4",
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[1] = new SqlParameter("@TableName", DbType.String)
{
    Value = "TableName",
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[2] = new SqlParameter("@PageSize", DbType.Int32)
{
    Value = pageSize,
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[3] = new SqlParameter("@PageIndex", DbType.Int32)
{
    Value = pageIndex,
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[4] = new SqlParameter("@Filter", DbType.String)
{
    //类似 :" USERID >= '100' AND TIME >= GETDATE()"
    Value = filterStr,
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[5] = new SqlParameter("@SortColumn", DbType.String)
{
    //按哪个字段排序，这个必须赋值
    Value = "SortColumn",
    Direction = System.Data.ParameterDirection.Input
};
sqlParas[6] = new SqlParameter("@IsAsc", DbType.Binary)
{
    Value = 0,
    Direction = System.Data.ParameterDirection.Input
};
#endregion
DBHelper dbhelper = new DBHelper();
var ds = dbhelper.ExcuteDataByProcedure("QueryByPage", sqlParas);
//总条数
total = int.Parse(ds.Tables[0].Rows[0][0].ToString());
//pageSize 和pageIndex 查出的值
var dt = ds.Tables[1];
```