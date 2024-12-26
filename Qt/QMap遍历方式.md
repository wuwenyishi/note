
QMap 是 Qt 中的一个关联容器，它提供了一种键值对的映射关系。可以通过键快速查找对应的值。遍历 QMap 可以使用以下几种方式：

```
   QMap&lt;QString, int&gt; map;
   map.insert("a", 1);
   map.insert("b", 2);
   map.insert("c", 3);
```

#### 第一种：迭代器遍历

###### 1\. 可以修改容器中的元素值，是可读可写的遍历方式。

```
QMap&lt;QString, int&gt;::iterator itor;
for (itor = map.begin(); itor != map.end(); ++itor)
{
qDebug() &lt;&lt; itor.key() &lt;&lt; ":" &lt;&lt; itor.value();
}
```

###### 2\. 使用了 const\_iterator，不能修改容器中的元素值，是只读的遍历方式。其中 constBegin() 返回指向 QMap 开头的常量迭代器，constEnd() 返回指向 QMap 结尾的常量迭代器，itor.key() 返回当前迭代器指向的键，itor.value() 返回当前迭代器指向的值。

```
for (QMap&lt;QString, int&gt;::const_iterator itor = map.constBegin(); itor != map.constEnd(); ++itor)
{
qDebug() &lt;&lt; itor.key() &lt;&lt; ":" &lt;&lt; itor.value();
}
```

###### 3\. 使用了 QMapIterator，也可以修改容器中的元素值，是可读可写的遍历方式。与第一种遍历方式相比，QMapIterator 提供了更多的功能，例如可以使用 remove() 函数删除当前迭代器指向的元素。

```
QMapIterator&lt;QString, int&gt; itor(map);
while (itor.hasNext())
{
itor.next(); //移动到下一个元素  
    qDebug() &lt;&lt; itor.key() &lt;&lt; ":" &lt;&lt; itor.value();
}
```

#### 第二种：C++11 的 for 循环遍历

###### 1\. 使用了 QMap 的 toStdMap() 函数将 QMap 转换为 std::map。

```
for (auto &amp;pair : map.toStdMap())
{
qDebug() &lt;&lt; pair.first &lt;&lt; ":" &lt;&lt; pair.second;
}
```

###### 2\. 使用 QMap 的 keys() 函数获取所有键的列表

```
for (const auto &amp;key : map.keys())
{
qDebug() &lt;&lt; key &lt;&lt; ":" &lt;&lt; map.value(key);
}
```

#### 第三种：Qt 中的 Q\_FOREACH 循环遍历

Q\_FOREACH 在 Qt 2 中被引入，在 Qt 4 中进行了重构和改进，成为了一个更加强大和灵活的语言结构。

```
foreach(const QString &amp;key, map.keys())
{
   qDebug() &lt;&lt; key &lt;&lt; ":" &lt;&lt; map.value(key);
}
```

#### 第四种：std::for\_each() 算法遍历

std::for\_each() 接受一个迭代器范围和一个函数对象，函数对象可以是 lambda 表达式，用于处理每个遍历到的元素。

```
std::for_each(map.constBegin(), map.constEnd(), [](const auto &amp;item) {
qDebug() &lt;&lt; item.key() &lt;&lt; ":" &lt;&lt; item.value();
});
```

#### 总结

使用 C++11 的 for 循环遍历和 std::for\_each 遍历 QMap 的方式最为常用和简洁，效率也比较高，占用资源较少。但是，不同的遍历方式适用于不同的场景，需要根据具体情况选择合适的遍历方式。如果需要修改 QMap 中的元素，应该使用迭代器进行遍历。如果只需要读取 QMap 中的元素，可以使用 const 迭代器或者 C++11 的 for 循环遍历。
