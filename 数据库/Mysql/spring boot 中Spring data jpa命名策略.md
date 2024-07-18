数据库，表字段命名是驼峰命名法（UserID）,Spring data jpa 自动更新之后是 user_id, 表字段不对照

Spring data jpa基于Hibernate5.0



application.yml 写法

1、无修改命名 

```yml
spring:
  jpa:
    hibernate:
      naming:                     
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

2、遇到大写字母 加”_”的命名

```yml
spring:
  jpa:
    hibernate:
      naming:                     
        physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
```

![image.png](https://s2.loli.net/2024/07/18/5FYvop8qEKLrOQ9.png)