# 一 引言

JAVA1.8得益于Lambda所带来的函数式编程，引入了一个全新的Stream流概念Stream流式思想类似于工厂车间的“生产流水线”，Stream流不是一种数据结构，不保存数据，而是对数据进行加工处理。Stream可以看作是流水线上的一个工序。在流水线上，通过多个工序让一个原材料加工成一个商品。

# 二 常用方法介绍

### 2.1 获取Stream流

所有的 Collection 集合都可以通过 stream 默认方法获取流；

java.util.Collection 接口中加入了default方法 stream 用来获取流，所以其所有实现类均可获取流。

```java
ArrayList<XyBug> xyBugList = new ArrayList();
Stream<XyBug> stream = xyBugList.stream();
```

Stream 接口的静态方法 of 可以获取数组对应的流。

```java
//String
Stream<String> stream = Stream.of("aa", "bb", "cc");
//数组
String[] arr = {"aa", "bb", "cc"};
Stream<String> stream7 = Stream.of(arr);
Integer[] arr2 = {11, 22, 33};
Stream<Integer> stream8 = Stream.of(arr2);
//对象
XyBug xyBug1 = new XyBug();
XyBug xyBug2 = new XyBug();
XyBug xyBug3 = new XyBug();
Stream<XyBug> bugStream = Stream.of(xyBug1, xyBug2, xyBug3);
```

### 2.2 Stream 数据处理常用方法

#### forEach方法

该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理

```java
List<String> list = new ArrayList<>();
Collections.addAll(list, "str1", "str2", "str3", "str4", "str5", "str6");
list.stream().forEach((String s) -> {
  System.out.println(s);
  });
//简写
list.stream().forEach(s -> System.out.println(s));
```

s代表list中的每一个元素，流式处理依次遍历每个元素

->后的代码为每个元素处理逻辑

#### count方法

count 方法来统计其中的元素个数，返回值为long类型

```java
long count = list.stream().count();
```

#### distinct方法

对流中的数据进行去重操作，普通类型可直接去重

```java
//将22、33重复数据去除
Stream.of(22, 33, 22, 11, 33).distinct().collect(Collectors.toList());
```

自定义类型是根据对象的hashCode和equals来去除重复元素的

XyBug实体类中加@Data注解，hashCode和equals会别重写，在使用distinct方法时判断去重

```java
ArrayList bugList = JSON.parseObject(bugs, ArrayList.class);
ArrayList<XyBug> xyBugList = new ArrayList();
List collect = (List) bugList.stream().distinct().collect(Collectors.toList());
```

通过distinct()方法去重，去重后的数据通过collect(Collectors.toList())组成新的list

#### limit方法

方法可以对流进行截取，只取用前n个，参数是一个long型，如果集合当前长度大于参数则进行截取。否则不进行操作

```java
List<String> list = new ArrayList<>();
Collections.addAll(list, "1", "2", "3", "4", "5", "6");
List<String> collect = list.stream().limit(3).collect(Collectors.toList());
```

将前3个String对象截取，组成新的list

#### skip方法

如果希望跳过前几个元素，可以使用 skip 方法获取一个截取之后的新流，如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流

```java
List<String> list = new ArrayList<>();
Collections.addAll(list, "1", "2", "3", "4", "5", "6");
List<String> collect = list.stream().skip(3).collect(Collectors.toList());
```

跳过前3个String对象，后三个组成新的list

#### filter方法

filter用于过滤数据，返回符合过滤条件的数据，可以通过 filter 方法将一个流转换成另一个子集流，该接口接收一个 Predicate 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件

```java
List<String> list = new ArrayList<>();
Collections.addAll(list, "1", "22", "3", "4", "55", "6");
//filter方法中写入筛选条件，将过滤后的数据组成新的list
list.stream().filter(s -> s.length() == 2).collect(Collectors.toList());
```

通过该条语句s -> s.length() == 2，筛选出22、55

#### map方法

将流中的元素映射到另一个流中，可以将当前流中的T类型数据转换为另一种R类型的流

```java
List<PersonCrDto> laputaCrDtos = queryListLaputaByBeginEndTime(begin, end);
//将list中的PersonCrDto对象的userName属性取到，收集成set集合
laputaCrDtos.stream().map(PersonCrDto::getUserName).collect(Collectors.toSet())
```

将list中的每个对象的userName数据拿到，组成Set集合

#### stream分组

```java
List<XyBug> list = new ArrayList<>();
Map<String, List<XyBug>> collect = list.stream().collect(Collectors.groupingBy(XyBug::getBugType));
```

根据bug类型进行分组，分组后会组成map，key是组名，value是组下的数据

#### stream排序

sort()，默认正序排列，加入reversed()方法后倒叙排列

```java
List<XyBug> list = new ArrayList<>();
//根据createTime正序排列
List<XyBug> collect = list.stream().sorted(Comparator.comparing(XyBug::getCreateTime)).collect(Collectors.toList());
//根据createTime倒叙排列
List<XyBug> collect = list.stream().sorted(Comparator.comparing(XyBug::getCreateTime).reversed()).collect(Collectors.toList());
```

#### collect方法

将处理后数据收集为list，collect（Collectors.toList()）

将处理后数据收集为set，collect（Collectors.toSet()）

根据某个字段值将数据分组map，collect（Collectors.groupingBy(o -> o.value()))）

# 三 实践举例

需求：将bug数据通过orgTierName分组，存储到map中

未使用Stream，需要使用for循环并且进行各种判断，代码行数较多

```java
HashMap<String, List<XyBug>> map = new HashMap<>();
for (XyBug one : bugList){
    if(one.getOrgTierName() != null){
        if(map.get(one.getOrgTierName()) == null){
            List<XyBug> list = new ArrayList();
            list.add(one);
            map.put(one.getOrgTierName(),list);
        }else {
            map.get(one.getOrgTierName()).add(one);
        }
    }
}
```

使用Stream，**一行代码**搞定，直观并高效

```java
collectDeptBugMap = bugList.stream().filter(o -> o.getOrgTierName() != null).collect(Collectors.groupingBy(o -> o.getOrgTierName()));
```

