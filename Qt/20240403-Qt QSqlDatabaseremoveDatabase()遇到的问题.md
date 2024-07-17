在Qt中想要关闭数据库，并删除连接时发现出错，代码如下：

```
1 if（db.isOpen())
2 {
3     QString connection;
4     connection = db.connectionName();
5     db.close();
6     db = QSqlDatabase();
7     db.removeDatabase(connection);
8 }
```



运行时会出现如下警告：

```
QSqlDatabasePrivate::removeDatabase: connection ‘in_mem_db’ is still in use, all queries will cease to work.
```



按照Qt官方教程的说法，内容如下：

```
Warning: There should be no open queries on the database connection when this function is called, otherwise a resource leak will occur.

Example:

 // WRONG
 QSqlDatabase db = QSqlDatabase::database("sales");
 QSqlQuery query("SELECT NAME, DOB FROM EMPLOYEES", db);
 QSqlDatabase::removeDatabase("sales"); // will output a warning
 // "db" is now a dangling invalid database connection,
 // "query" contains an invalid result set
The correct way to do it:
 {
     QSqlDatabase db = QSqlDatabase::database("sales");
     QSqlQuery query("SELECT NAME, DOB FROM EMPLOYEES", db);
 }
 // Both "db" and "query" are destroyed because they are out of scope
 QSqlDatabase::removeDatabase("sales"); // correct
```



故将代码修改成如下：

```
1 if(db.isOpen()){db.close();}
2 QString connection;
3 {
4    connection = QSqlDatabase::database().connectionName();    
5 }
6 QSqlDatabase::removeDatabase(connection);
7 db = QSqlDatabase();
```

