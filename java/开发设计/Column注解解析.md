功能：@Column注解用来标识实体类中属性与数据表中字段的对应关系  
语法：

```java
@Column(columnDefinition = "bigint(18)  comment '公司ID'")
   private Long ouId;
//columnDefinition表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用。（也就是说，如果DB中表已经建好了，该属性就没有必要使用了）
//comment(注释的意思-我这个文盲硬生生以为是@Column里面带的，类似于name属性的字段)
public @interface Column {
    String name() default "";
//name属性定义了，被标注字段在数据库表中所对应字段的名称
    boolean unique() default false;
//unique属性表示该字段是否为唯一标识，默认为false。如果表中有一个字段需要唯一标识，则既可以使用该标记，也可以使用@Table标记中的@UniqueConstraint
    boolean nullable() default true;
//nullable属性表示该字段是否可以为null值，默认为true
    boolean insertable() default true;
//insertable属性表示在使用“INSERT”脚本插入数据时，是否需要插入该字段的值
    boolean updatable() default true;
//updatable属性表示在使用“UPDATA”脚本插入数据时，是否需要更新该字段的值。
    //ps:insertable和updatable属性一般多用于只读属性，例如主键和外键等。这些字段的值通常是自动生成的。
    String columnDefinition() default "";
//上面的代码注释已经写了
    String table() default "";
//table属性定义了包含当前字段的表名
    int length() default 255;
//length属性表示字段的长度，当字段的类型为varchar时，该属性才有效，默认为255个字符
    int precision() default 0;
//precision和scale属性表示精度，当字段类型为double时，precision表示数值的总长度，scale表示小数点所占的位数。
    int scale() default 0;
}

```

