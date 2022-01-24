---
layout: post
title:  "实习日记107~112"
date:   2022-01-24(工具类学习)
categories: jekyll update
typora-root-url: ./
---

## Day107~112

工具类学习

### ObjectUtils

继承自org.apache.commons.lang3.ObjectUtils，用于对Object对象执行相关操作

#### isNull()

为继承后新添加的方法，用于判断Object对象是否为空，如果该Object对象为一个集合或者数组类型，则只要其中任一元素为空，则返回true。

使用案例：

```java
Integer a = new Integer(1);
Object[] array = {1, null, "hello world"};
System.out.println(ObjectUtils.isNull(a));
System.out.println(ObjectUtils.isNull(array));
```

输出结果：

```
false
true
```

isNotNull()方法效果相反，不多赘述

#### isEmpty()

继承自org.apache.commons.lang3.ObjectUtils的isEmpty()方法，用于判断Object对象是否包含元素

- 如果Object对象为CharSequence类型，则返回长度是否为0
- 如果Object对象为Array类型，则返回长度是否为0
- 如果Object对象为Collection，则返回Collection.isEmpty()结果

使用案例：

```java
Integer a = new Integer(1);
Object[] array = {1, null, "hello world"};
ArrayList<Integer> arrayList = new ArrayList<>();
System.out.println(ObjectUtils.isEmpty(a));
System.out.println(ObjectUtils.isEmpty(array));
System.out.println(ObjectUtils.isEmpty(arrayList));
```

输出结果：

```
false
false
true
```

isNotEmpty()方法效果相反，不多赘述

#### hasNull()&hasNotNull()

为继承后新方法，用于判断Object对象含有几个空值/非空值。

使用案例：

```java
Object[] array = {1, null, "hello world"};
System.out.println(ObjectUtils.hasNull(array));
System.out.println(ObjectUtils.hasNotNull(array));
```

输出结果：

```
1
2
```

#### allsNull()&allsNotNull()

为继承后新方法，用于判断Object是否全为空值/全不为空值。

```java
Object[] array = {1, null, "hello world"};
Object[] array_1 = {1, "hello world"};
Object[] array_2 = {null};
System.out.println(ObjectUtils.allIsNull(array));
System.out.println(ObjectUtils.allIsNotNull(array));
System.out.println(ObjectUtils.allIsNull(array_1));
System.out.println(ObjectUtils.allIsNotNull(array_1));
System.out.println(ObjectUtils.allIsNull(array_2));
System.out.println(ObjectUtils.allIsNotNull(array_2));
```

输出结果：

```
false
false
false
true
true
false
```

#### nullToDefault()

用于将空值转化为设定的默认值，若原本对象不为空则不会改变原对象。

使用案例：

```java
Integer a = 1;
Integer b = 2;
System.out.println(object);
System.out.println(a);
System.out.println(ObjectUtils.nullToDefault(object, b));
System.out.println(ObjectUtils.nullToDefault(a, b));
```

输出结果：

```
null
1
2
1
```

#### cloneObject()

继承自org.apache.commons.lang3.ObjectUtils的clone()方法，克隆一个新对象

#### equalsToValue()&notEqualsToValue()

判断source对象是否和base对象值相同/不同（使用org.apache.commons.lang3.ObjectUtils的equals()方法，只判断值是否相同），是则返回result对象，反之则返回source对象。

使用案例：

```java
Integer a = 1;
Integer b = 1;
Integer c = 2;
Integer result = 0;
System.out.println(ObjectUtils.equalsToValue(a, b, result));
System.out.println(ObjectUtils.equalsToValue(a, c, result));
```

输出结果：

```
0
1
```

#### softCast()

将source对象转换成target类型对象，如果出现无法转换或者转换失败的情况，依然会返回target对象。

使用案例：

```java
object = "str";
Integer b= 2;
System.out.println(object);
System.out.println(ObjectUtils.softCast(object, b));
```

输出结果：

```
str
2
```

**注意：如果source对象和target对象类型一致，则无法转换**

#### nullSafeEquals()

判断两个对象是否相同，包含对java所有基础数据类型的判断

#### foreachObejct()

用于遍历map类型object，生成类似{`string`=`object`}的形式，有三种重载方法：

- SetHashMap<String, Object> foreachObject(Object obj)，默认string为null
- void foreachObject(Object obj, SetHashMap<String, Object> relations)
- void foreachObject(String parentObj, Object obj, SetHashMap<String, Object> relations)

使用案例：

```java
Map<String, Integer> a = new HashMap<>();
a.put("1", 1);
System.out.println(ObjectUtils.foreachObject(a));
```

输出结果：

```
{1=1}
```

#### isSingleType()

判断Object对象是否属于以下类型：

- String
- Date
- java.sql.Date
- 原始类型（boolean、char、byte、short、int、long、float、double）
- 任一元素为原始类型

如满足上述任一条件，则返回true，反之返回false

#### isLoopType()

判断Object对象是否属于以下类型实例：

- List
- Collection
- Set
- Iterable

如满足上述任一条件，则返回true，反之返回false

#### printObjectDefField()

输出Object私有属性值

使用案例：

```java
Student student = new Student();
ObjectUtils.printObjectDefField(student);
```

输出结果：

```
private int id = 0;
private int record_id = 0;
private int gender = 0;
```

#### allPropertyIsNull()

判断Object对象是否全为空，按照如下条件判断：

- 如果Object本身为空，返回true
- 如果Object属于Map示例并且不含任何键值对，则返回true
- 如果Object属于的Class本身和父类的所有Field为空，则返回true

使用案例

```java
Map<String, Integer> map = new HashMap<>();
System.out.println(ObjectUtils.allPropertyIsNull(map));
map.put("1", 1);
System.out.println(ObjectUtils.allPropertyIsNull(map));
```

输出结果：

```
true
false
```

### CollectionUtils

#### addAll()

向Collection对象中一次性添加多个对象，使用了org.apache.commons.collections.CollectionUtils包的addAll()方法，但是如果需要添加的Object为空，则不会执行，优化了效率

使用案例：

```java
List<Integer> a = new ArrayList<>();
a.add(1);
Integer[] b = {2, 3};
CollectionUtils.addAll(a, b);
System.out.println(a);
```

输出结果：

```
[1,2,3]
```

#### uniqAdd()

向Collection对象中添加原先其中不存在的对象，如果添加的对象在Collection中存在（使用Collection的contains方法判断），则不添加。

使用案例：

```java
List<Integer> a = new ArrayList<>();
a.add(1);
Integer b = 1;
Integer c = 2;
CollectionUtils.uniqAdd(a, b);
System.out.println(a);
CollectionUtils.uniqAdd(a, c);
System.out.println(a);
```

输出结果：

```
[1]
[1, 2]
```

#### firstResult()

返回一个Collection的第一个元素，如果Collection实例为空，则返回def

存在两种重载方法：

- firstResult(Collection<T> coll)，此时def为null
- firstResult(Collection<T> coll, T def)

使用案例：

```java
List<Integer> a = new ArrayList<>();
System.out.println(CollectionUtils.firstResult(a, 2));
a.add(1);
System.out.println(CollectionUtils.firstResult(a, 2));
```

输出结果：

```
2
1
```

#### firstNotEmptyResult()

返回Collection的第一个非空元素，如果Collection为空或只含有null，则返回null

使用案例：

```java
List<Object> a = new ArrayList<>();
System.out.println(CollectionUtils.firstNonEmptyResult(a));
a.add(null);
System.out.println(CollectionUtils.firstNonEmptyResult(a));
a.add(1);
System.out.println(CollectionUtils.firstNonEmptyResult(a));
```

输出结果：

```
null
null
1
```

#### length()

返回Collection实例的元素数。

使用案例：

```java
List<Integer> a = new ArrayList<>();
a.add(1);
System.out.println(CollectionUtils.length(a));
```

输出结果：

```
1
```

#### excavate()

查找Collection中指定某个字段的值

使用案例：

```
List<Map<String, Integer>> list = new ArrayList<>();
Map<String, Integer> map = new HashMap<>();
map.put("id", 1);
Map<String, Integer> map_1 = new HashMap<>();
map_1.put("id", 2);
map_1.put("id_1", 3);
list.add(map);
list.add(map_1);
System.out.println(CollectionUtils.excavate(list, "id"));
```

输出结果：

```
[1,2]
```

#### excavateMap()

查找Collection中指定多个字段的值，并输出全部需要查找的字段，如果一个元素含有多个需要查找的字段，则会重复显示，如果不包含需要查找的字段，则对应字段返回null

使用案例：

```java
List<Map<String, Integer>> list = new ArrayList<>();
Map<String, Integer> map = new HashMap<>();
map.put("id", 1);
map.put("id_2", 5);
Map<String, Integer> map_1 = new HashMap<>();
map_1.put("id", 2);
map_1.put("id_1", 3);
map_1.put("id_2", 5);
list.add(map);
list.add(map_1);
System.out.println(CollectionUtils.excavateMap(list, "id", "id_1"));
```

输出结果：

```
[{id=1, id_1=null}, {id=1, id_1=null}, {id=2, id_1=3}, {id=2, id_1=3}]
```

#### unique()

去除Collection对象中的重复元素

使用案例：

```java
List<Integer> a = new ArrayList<>();
a.add(1);
a.add(1);
a.add(2);
System.out.println(CollectionUtils.unique(a));
```

输出结果：

```
[1, 2]
```

#### foreach()

遍历Collection对象，每遍历一项便执行自定义的execute()方法

使用案例：

```java
List<Integer> a = new ArrayList<>();
Closure<Integer> b = new Closure<Integer>() {
	@Override
		public void execute(Integer integer) {
			System.out.println(integer);
		}
	};
a.add(1);
a.add(1);
a.add(2);
CollectionUtils.foreach(a, b);
```

输出结果：

```
1
1
2
```

#### derivate()

通过source的Collection对象以及自定义派生方法，填充指定的target Collection对象

使用案例：

```java
List<Object> a = new ArrayList<>();
a.add(1);
a.add("hello");
List<String> b = new ArrayList<>();
Derivative derivative = new Derivative() {
	@Override
		public Object execute(Object o) {
			return o.getClass().getName();
		}
	};
CollectionUtils.derivative(a, b, derivative);
System.out.println(b);
```

输出结果：

```
[java.lang.Integer, java.lang.String]
```

### StringUtils

#### randomString()

生成指定长度的随机字符段

使用案例：

```
System.out.println(StringUtils.randomString(10));
```

输出结果：

```
iwbdhNNv6G
```

#### templateReplace()

用于替换string中#{}格式的子字符串，可能用于mybatis相关

有两种重载方法：

- templateReplace(String template, Map<String, String>dataMap, String defaultValue)，若template为Empty则返回template
- templateReplace(String template, Map<String, String>dataMap)，此时defaultValue默认为""

使用案例：

```java
String template = "#{e}";
String defaultValue = "peek";
Map<String, String> map = new HashMap<>();
map.put("e", "yee");
System.out.println(StringUtils.templateReplace(template, map, defaultValue));
```

输出结果：

```
yee
```

#### bond()

将多个字符串按顺序拼接在一起

使用案例：

```java
String str1 = "Hello";
String str2 = " ";
String str3 = "World";
System.out.println(StringUtils.bond(str1, str2, str3));
```

输出结果：

```
Hello World
```

#### join()

将多个字符串按顺序以指定字符合并

使用案例：

```java
String str1 = "Hello";
String separator = " ";
String str2 = "World";
System.out.println(StringUtils.join(separator, str1, str2));
```

输出结果：

```
Hello World
```

#### isTrimedEmpty()

判断字符串是否只包含空白符，若是或字符串为空则返回True

使用案例：

```java
String str = " ";
System.out.println(StringUtils.isTrimedEmpty(str));
```

输出结果：

```
true
```

#### splitToEmpty()

将字符串按逗号（,）分割，返回分割后的字符串数组（不包含分割后的空字符串）

使用案例：

```java
String str = "dsajda  ,,afsdasd";
System.out.println(StringUtils.splitToEmpty(str).length);
for(String string : StringUtils.splitToEmpty(str)){
	System.out.println(string);
}
```

输出结果：

```
2
dsajda  
afsdasd
```

同时有重载方法splitToEmpty(String str, String separatorChar)来指定分割字符串

#### splitToEmptyList()

返回splitToEmpty()方法产生的String数组的List形式

#### isAllBlank()

判断输入的字符串是否全为空，若是则返回true

使用案例：

```java
String str1 = "";
String str2 = "yes";
System.out.println(StringUtils.isAllBlank(str1, str2));
```

输出结果：

```
false
```

#### isNotAllBlank()

判断输入的字符串是否全为非空，若是则返回true

使用案例：

```java
String str1 = "";
String str2 = "yes";
System.out.println(StringUtils.isAllNotBlank(str1, str2));
```

输出结果：

```
false
```

#### stringMaker()

根据字符串中{i}这样的格式进行字符串填充，i为从0开始的正整数

使用案例：

```java
String temp = "{0} {1}";
String str1 = "hello";
String str2 = "world";
System.out.println(StringUtils.stringMaker(temp, str1, str2));
```

输出结果：

```
hello world
```

#### exchangeType()

尝试将字符串转化为其他类型的对象，有以下三种重载方法：

- exchange(String source, T defaultValue)，clazz默认为defaultValue的Class
  - exchange(String source, Class<T> clazz)，defaultValue默认为null
- exchange(String source,  Class<? extends Object> clazz, T defaultValue)

如果无法转化，则返回defaultValue

使用案例：

```
String str = "1";
Integer defaultValue = 1;
System.out.println(StringUtils.exchangeType(str, Integer.class, defaultValue).getClass());
```

输出结果：

```
class java.lang.Integer
```

#### hasLength()

判断字符串是否长度大于0，若是则返回true，可以使用String和CharSequence作为参数

使用案例：

```java
CharSequence charSequence = "yes";
System.out.println(StringUtils.hasLength(charSequence));
```

输出结果：

```
true
```

hasText()

判断字符串是否含有内容，若是则返回true，可以使用String和CharSequence作为参数

使用案例：

```java
CharSequence charSequence = "   ";
System.out.println(StringUtils.hasText(charSequence));
```

输出结果：

```
false
```

#### beanTrimToNull()

对于输入参数Bean，输入的Bean的所有属性(为字符串类型实例)，为null时返回null，否则去除掉字符串两边的空格或者制表符

#### splitToList()

读取输入字符串，转化成JSON格式字符串后返回对应类对象实例的List的格式。

#### splitToArray()

同splitToList()方法，但返回数组类型数据

#### replace()

把原字符串中的指定字符串替换成指定的新字符串

使用案例：

```java
String inString = "Hello World";
String oldPattern = "Hello";
String newPattern = "Goodbye";
System.out.println(StringUtils.replace(inString, oldPattern, newPattern));
```

输出结果：

```
GoodBye World
```

#### delimitedListToStringArray()

按照指定字符串分割并删除指定字符串，返回String数组类型

有两种重载方法：

- delimitedListToStringArray(String str, String delimiter, String charsToDelete)
- delimitedListToStringArray(String str, String delimiter)，此时无指定删除字符串

使用案例：

```java
String str = "1, sad, 34jsd";
for(String string : StringUtils.delimitedListToStringArray(str, ",", "s")){
	System.out.println(string);
}
```

输出结果：

```
1
 ad
 34jd
```

#### toStringArray()

将指定Collection对象（其中元素必须为String类型）转化为String数组类型

使用案例：

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("1");
a.add("2");
System.out.println(StringUtils.toStringArray(a).getClass());
```

输出结果：

```
class [Ljava.lang.String;
```

#### collectionToDelimitedString()

将collection转化成字符串格式并添加指定分隔符和前后缀

有两种重载方法：

- collectionToDelimitedString(Collection<?> coll, String delim, String prefix, String suffix) 
- collectionToDelimitedString(Collection<?> coll, String delim)，此时默认无前后缀

使用案例：

```
List<String> a = new ArrayList<>();
a.add("1");
a.add("1");
a.add("2");
System.out.println(StringUtils.collectionToDelimitedString(a, ",", "{", "}"));
```

 输出结果：

```
{1},{1},{2}
```

#### deleteAny()

删除字符串中所有指定字符串

使用案例：

```
System.out.println(StringUtils.deleteAny("Hello world", "o"));
```

输出结果：

```
Hell wrld
```

#### toString()

将输入参数转化为String类型，支持三种类型输入参数：

- Collection
- Object[]
- Map

使用案例：

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("1");
a.add("2");
System.out.println(StringUtils.toString(a));
```

输出结果：

```
1,1,2
```

#### nullToEmpty()

如果输入对象为null，则返回""，反之返回输入Object的String类型

#### contains()

判断输入字符串是否包含全部的指定字符串，若是则返回true，反之则返回false

使用案例：

```
System.out.println(StringUtils.contains("Hello World", "o", "World", "hello"));
```

输出结果：

```
false
```

