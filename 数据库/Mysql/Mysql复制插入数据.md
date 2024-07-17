## 新建表  
```sql
CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `age` int DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;

SET FOREIGN_KEY_CHECKS = 1;
```

> 主键设为自增
## 创建两条数据  

```sql
INSERT INTO t VALUES (null,30, 'c'),(null,22, 'd'); 
```

## 复制插入 
```sql
INSERT INTO t 
SELECT null,age,name
FROM t
WHERE
	age > 1;
```

 ![pIBzxW](https://xuemingde.com/pages/image/others/pIBzxW.png)

## ID为UUID时。
可用 REPLACE 函数将字符串中的某个字符替换即可。

```sql
INSERT INTO t 
SELECT REPLACE(id,'a','b'),age,name
FROM t
WHERE
	age > 1;  
```

将ID中的a替换成b  

## 返回值时间加1个月 
```sql
	DATE_ADD( create_time, INTERVAL 1 MONTH ) 
```

create_time 为返回值，在原来的时间上加一个月。
