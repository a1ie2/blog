title: MySql大批量插入数据的方法
date:  2017-05-12
tags: MySql
categories: 
   - 后端
   - MySql   
------

之前在做项目的时候用到了Mysql,因为要处理千万级别的数据，从文件中读取，然后再插入到Mysql中。从开始到研究之后发现了如下几种插入的方法

### 事务插入 ###

``` bash
insert into table values (a,b,c),(a1,b1,c1),...
```

事务插入就是拼接sql语句，然后放入事务中批量执行。因为Mysql支持Values(a,b,c)(a1,b1,c1)…所以可以一条sql语句可以拼接多点值

网上资料显示，每个事务中执行500或者1000左右需要提交一次事务，这样性能不会有所下降，也防止因为事务中的sql语句过多有错误回滚，没错的也插入不了,代码参考如下(c#)

``` bash
public static void MySqlExcuteBatch(List<string> sqlList, string conStr)
{
    using (MySqlConnection conn = new MySqlConnection(conStr))
    {
        using (MySqlCommand cmd = new MySqlCommand())
        {
            cmd.Connection = conn;
            cmd.CommandType = CommandType.Text;
            if (conn.State != ConnectionState.Open)
                conn.Open();
            MySqlTransaction tx = conn.BeginTransaction();
            cmd.Transaction = tx;
            try
            {
                for (int n = 0; n < sqlList.Count; n++)
                {
                    string strsql = sqlList[n].ToString();
                    if (strsql.Trim().Length > 1)
                    {
                        cmd.CommandText = strsql;
                        cmd.ExecuteNonQuery();
                    }
                    if (n > 0 && (n % 800 == 0 || n == sqlList.Count - 1))
                    {
                        tx.Commit();
                        tx = conn.BeginTransaction();
                    }
                }
            }
            catch (Exception e)
            {
                tx.Rollback();
                throw new Exception(e.Message);
            }
        }
    }
}
```

### 数据通过生成文件导入数据库 ###

mysql支持把csv文件倒入到数据库，生成csv文件的时候，不需要特别生成一行头文件，每行生成的数据直接用 ‘,’ 分隔就行，一行代表数据库的一条记录，所以生成csv文件时候，记得数据的顺序

``` bash
public static void MySqlExcuteBatch(string conStr, string filePath, string tableName)
{
    using (MySqlConnection conn = new MySqlConnection(conStr))
    {
        conn.Open();
        MySqlBulkLoader bulk = new MySqlBulkLoader(conn)
        {
            FieldTerminator = ",",//这个地方字段间的间隔方式，为逗号
            FieldQuotationCharacter = '"',
            EscapeCharacter = '"',
            LineTerminator = "\r\n",//每行
            FileName = filePath,//文件地址
            NumberOfLinesToSkip = 0,
            TableName = tableName,
        };
        bulk.Load();
    }
}
```

### 总结 ###

当数据量不大的时候可以优先选择第一种方法，只不过拼接sql语句有点麻烦

当数据量大的时候优先选择第二种方法，速度比第一种方法快n倍

> Tip:创建表的时候，索引要选择好，本人亲测990万条数据插入一张表，表有一个datetime类型的字段索引，第二种方法比第一种方法快4分钟左右。把索引换成string类型的字段，第二种方法耗时是原先的5倍左右。还有网上说的字段能用string类型就不要用其他类型，我测试一下，datetime类型换成string类型，反而插入速度很慢。

mysql的配置文件也可以优化一下，会大大提高插入速度