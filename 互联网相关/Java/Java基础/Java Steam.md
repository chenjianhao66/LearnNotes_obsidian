# 概述
Java 8 是一个非常成功的版本，这个版本新增的`Stream`，配合同版本出现的 `Lambda` ，给我们操作集合（Collection）提供了极大的便利。

那么什么是`Stream`？

>  `Stream`将要处理的元素集合看作一种流，在流的过程中，借助`Stream API`对流中的元素进行操作，比如：筛选、排序、聚合等。

Stream可以由数组或集合创建，对流的操作分为两种：

**中间操作**，每次返回一个新的流，可以有多个。
**终端操作**，每个流只能进行一次终端操作，终端操作结束后流无法再次使用。终端操作会产生一个新的集合或值。

另外，Stream有几个特性：

stream不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果。
stream不会改变数据源，通常情况下会产生一个新的集合或一个值。
stream具有延迟执行特性，只有调用终端操作时，中间操作才会执行。

# Stream的创建
`Stream` 流可以通过集合和数组来创建；

1、通过 `java.util.Collection.stream()` 方法用集合创建流

```java
List<String> list = Arrays.asList("a", "b", "c");
// 创建一个顺序流
Stream<String> stream = list.stream();
// 创建一个并行流
Stream<String> parallelStream = list.parallelStream();
```

2、使用 `Array`类的`stream`静态方法来创建流
```java
int[] array={1,3,5,6,8};
IntStream stream = Arrays.stream(array);
```

3、使用`Stream`的静态方法：`of()、iterate()、generate()`
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);

Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 3).limit(4);
stream2.forEach(System.out::println);

Stream<Double> stream3 = Stream.generate(Math::random).limit(3);
stream3.forEach(System.out::println);
```

**`stream`和`parallelStream`的简单区分：** `stream`是顺序流，由主线程按顺序对流执行操作，而`parallelStream`是并行流，内部以多线程并行执行的方式对流进行操作，但前提是流中的数据处理没有顺序要求。例如筛选集合中的奇数，两者的处理不同之处：

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202203072254094.png)

# Stream的使用
在使用stream之前，先理解一个概念：`Optional` 。

>`Optional`类是一个可以为`null`的容器对象。如果值存在则`isPresent()`方法会返回`true`，调用`get()`方法会返回该对象。  
更详细说明请见：[菜鸟教程Java 8 Optional类](https://www.runoob.com/java/java8-optional-class.html)

**测试数据**

```
// import已省略，请自行添加，后面代码亦是

List<Person> personList = new ArrayList<Person>();
personList.add(new Person("Tom", 8900, "male", "New York"));
personList.add(new Person("Jack", 7000, "male", "Washington"));
personList.add(new Person("Lily", 7800, "female", "Washington"));
personList.add(new Person("Anni", 8200, "female", "New York"));
personList.add(new Person("Owen", 9500, "male", "New York"));
personList.add(new Person("Alisa", 7900, "female", "New York"));

@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class Person {  
	 /**  
	 * 姓名  
	 */  
	 private String name;  

	 /**  
	 * 薪资  
	 */  
	 private int salary;  

	 /**  
	 * 年龄  
	 */  
	 private int age;  

	 /**  
	 * 性别  
	 */  
	 private String sex;  

	 /**  
	 * 地区  
	 */  
	 private String area;  
}

```


## 遍历/匹配（foreach/find/match）

`Stream`也是支持类似集合的遍历和匹配元素的，只是`Stream`中的元素是以`Optional`类型存在的。`Stream`的遍历、匹配非常简单。

```java
/**  
 * 流的遍历  
 */  
public static void streamFind(){  
	 List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);  

	 /*  
	 筛选集合中大于6的元素并输出  
	 */ list.stream().filter(x -> x > 6).forEach(System.out::print);  
	 System.out.println();  

	 /*  
	 筛选集合中大于6的第一个元素赋值给first  
	 */ Optional<Integer> first = list.stream().filter(x -> x > 6).findFirst();  

	 /*  
	 是否包含符合 大于6 的元素  
	 */ boolean anyMatch = list.stream().anyMatch(x -> x > 6);  

	 System.out.println("匹配第一个值：" + first.get());  
	 System.out.println("是否存在大于6的值：" + anyMatch);  
}
```
> `forEach`方法中是对命中的元素都进行的函数操作，里面使用的是lambad表达式，`::`左边的代表类，而右边代表的是方法名，**lambad表达式就是对方法的调用**
## 筛选
筛选出集合中大于7的元素，并打印出来
```java
public static void streamFilter(){  
	 List<Integer> list = Arrays.asList(6, 7, 3, 8, 1, 2, 9);  
	 Stream<Integer> stream = list.stream();  
	 stream.filter(x -> x > 7).forEach(System.out::println);  
}
```
预期结果
> 8 9


## 聚合（max/min/count）
`max`、`min`、`count`这些字眼你一定不陌生，没错，在mysql中我们常用它们进行数据统计。Java stream中也引入了这些概念和用法，极大地方便了我们对集合、数组的数据统计工作。
```java
public static void streamMaxMinCount(){  
	 List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");  
	 // 筛选出字符串长度最长的字符串  
	 // 这里传进一个实现了Comparator接口的函数，这里调用String类的length  
	 Optional<String> max = list.stream().max(Comparator.comparing(String::length));  
	 System.out.println(max);  

	 // 筛选出长度最小的字符串  
	 Optional<String> min = list.stream().min(Comparator.comparing(String::length));  
	 System.out.println(min);  

	 // 筛选出字符串长度大于5的字符串个数  
	 long count = list.stream().filter(x -> x.length() > 5).count();  
	 System.out.println(count);  
}
```
预期结果
>Optional[weoujgsd]
Optional[pot]
2


## 映射（map/flatMap)
映射，可以将一个流的元素按照一定的映射规则映射到另一个流中。分为`map`和`flatMap`：

-   `map`：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
-   `flatMap`：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

```java
public static void streamMap(){  
	 String[] strArr = { "abcd", "bcdd", "defde", "fTr" };  
	 // 将流中的每一个函数都使用String类的toUpperCase方法，该方法是将字符串转换成大写  
	 List<String> strings = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());  

	 List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);  
	 // 将流中的每一个元素自增3  
	 List<Integer> newList = intList.stream().map(x -> x + 3).collect(Collectors.toList());  
	 System.out.println("每个元素大写：" + strings);  
	 System.out.println("每个元素+3：" + newList);  
}
```
预期结果
>每个元素大写：[ABCD, BCDD, DEFDE, FTR]
每个元素+3：[4, 6, 8, 10, 12, 14]

## 排序（sorted）
sorted，中间操作。有两种排序：

-   sorted()：自然排序，流中元素需实现Comparable接口
-   sorted(Comparator com)：Comparator排序器自定义排序

```java
public static void streamSorted() {
		List<Person> personList = new ArrayList<Person>();

		personList.add(new Person("Sherry", 9000, 24, "female", "New York"));
		personList.add(new Person("Tom", 8900, 22, "male", "Washington"));
		personList.add(new Person("Jack", 9000, 25, "male", "Washington"));
		personList.add(new Person("Lily", 8800, 26, "male", "New York"));
		personList.add(new Person("Alisa", 9000, 26, "female", "New York"));

		// 按工资升序排序（自然排序）
		List<String> newList = personList.stream().sorted(Comparator.comparing(Person::getSalary)).map(Person::getName)
				.collect(Collectors.toList());
		// 按工资倒序排序
		List<String> newList2 = personList.stream().sorted(Comparator.comparing(Person::getSalary).reversed())
				.map(Person::getName).collect(Collectors.toList());
		// 先按工资再按年龄升序排序
		List<String> newList3 = personList.stream()
				.sorted(Comparator.comparing(Person::getSalary).thenComparing(Person::getAge)).map(Person::getName)
				.collect(Collectors.toList());
		// 先按工资再按年龄自定义排序（降序）
		List<String> newList4 = personList.stream().sorted((p1, p2) -> {
			if (p1.getSalary() == p2.getSalary()) {
				return p2.getAge() - p1.getAge();
			} else {
				return p2.getSalary() - p1.getSalary();
			}
		}).map(Person::getName).collect(Collectors.toList());

		System.out.println("按工资升序排序：" + newList);
		System.out.println("按工资降序排序：" + newList2);
		System.out.println("先按工资再按年龄升序排序：" + newList3);
		System.out.println("先按工资再按年龄自定义降序排序：" + newList4);
}
```

## 收集
  
因为流不存储数据，那么在流中的数据完成处理后，需要将流中的数据重新归集到新的集合里。`toList`、`toSet`和`toMap`比较常用，另外还有`toCollection`、`toConcurrentMap`等复杂一些的用法。

### 统计
Collectors提供了一系列用于数据统计的静态方法：

计数：count
平均值：averagingInt、averagingLong、averagingDouble
最值：maxBy、minBy
求和：summingInt、summingLong、summingDouble
统计以上所有：summarizingInt、summarizingLong、summarizingDouble

```java
public static void streamAveraging(){  
	List<Person> personList = new ArrayList<>();  
	personList.add(new Person("Tom", 8900, 23, "male", "New York"));  
	personList.add(new Person("Jack", 7000, 25, "male", "Washington"));  
	personList.add(new Person("Lily", 7800, 21, "female", "Washington"));  

	// 求总数  
	Long count = personList.stream().collect(Collectors.counting());  
	//求平均工资  
	Double averaging = personList.stream().collect(Collectors.averagingDouble(Person::getSalary));  
	// 求工资总额  
	Integer sum = personList.stream().collect(Collectors.summingInt(Person::getSalary));  
	// 一次性统计所有  
	DoubleSummaryStatistics all = personList.stream().collect(Collectors.summarizingDouble(Person::getSalary));  

	System.out.println("员工总数：" + count);  
	System.out.println("员工平均工资：" + averaging);  
	System.out.println("员工工资总和：" + sum);  
	System.out.println("员工工资所有统计：" + all);  
}
```
预期结果
>员工总数：3
员工平均工资：7900.0
员工工资总和：23700
员工工资所有统计：DoubleSummaryStatistics{count=3, sum=23700.000000, min=7000.000000, average=7900.000000, max=8900.000000}

### 分组
-   分区：将`stream`按条件分为两个`Map`，比如员工按薪资是否高于8000分为两部分。
-   分组：将集合分为多个Map，比如员工按性别分组。有单级分组和多级分组。

```java
public static void streamGroup(){  
	List<Person> personList = new ArrayList<Person>();  
	personList.add(new Person("Tom", 8900, 15,"male", "New York"));  
	personList.add(new Person("Jack", 7000, 20,"male", "Washington"));  
	personList.add(new Person("Lily", 7800, 55,"female", "Washington"));  
	personList.add(new Person("Anni", 8200, 25,"female", "New York"));  
	personList.add(new Person("Owen", 9500, 42,"male", "New York"));  
	personList.add(new Person("Alisa", 7900, 21,"female", "New York"));  

	// 将员工按薪资是否高于8000分组  
	Map<Boolean, List<Person>> part = personList.stream().collect(Collectors.partitioningBy(elem -> elem.getSalary() > 8000));  
	// 将员工按性别分组  
	Map<String, List<Person>> group = personList.stream().collect(Collectors.groupingBy(Person::getSex));  

	System.out.println("员工按薪资是否大于8000分组情况：" + part);  
	System.out.println("员工按性别分组情况：" + group);  
}
```
预期结果
>员工按薪资是否大于8000分组情况：{false=[Person(name=Jack, salary=7000, age=20, sex=male, area=Washington), Person(name=Lily, salary=7800, age=55, sex=female, area=Washington), Person(name=Alisa, salary=7900, age=21, sex=female, area=New York)], true=[Person(name=Tom, salary=8900, age=15, sex=male, area=New York), Person(name=Anni, salary=8200, age=25, sex=female, area=New York), Person(name=Owen, salary=9500, age=42, sex=male, area=New York)]}
员工按性别分组情况：{female=[Person(name=Lily, salary=7800, age=55, sex=female, area=Washington), Person(name=Anni, salary=8200, age=25, sex=female, area=New York), Person(name=Alisa, salary=7900, age=21, sex=female, area=New York)], male=[Person(name=Tom, salary=8900, age=15, sex=male, area=New York), Person(name=Jack, salary=7000, age=20, sex=male, area=Washington), Person(name=Owen, salary=9500, age=42, sex=male, area=New York)]}
