### **Stream List 常用方法**：

List<Restriction>数组，将value属性逗号拼接

~~~java
String str = List.stream().map(Restriction::getValue).collect(Collectors.joining(","));
~~~

List<Restriction>数组，将id属性逗号拼接后转换为String类型

~~~java
String str = List.stream().map(Restriction::getId).collect(Collectors.toList()).stream().map(w->w.toString()).collect(Collectors.joining(","));

~~~

收集起始时间到结束时间之间所有的时间并以字符串集合方式返回

~~~java
/**
* 收集起始时间到结束时间之间所有的时间并以字符串集合方式返回
* @param timeStart
* @param timeEnd
* @return
*/
public static List<String> collectLocalDates(String timeStart, String timeEnd){
	return collectLocalDates(LocalDate.parse(timeStart), LocalDate.parse(timeEnd));
}
 
/**
* 收集起始时间到结束时间之间所有的时间并以字符串集合方式返回
* @param start
* @param end
* @return
*/
public static List<String> collectLocalDates(LocalDate start, LocalDate end){
	// 用起始时间作为流的源头，按照每次加一天的方式创建一个无限流
	return Stream.iterate(start, localDate -> localDate.plusDays(1))
	     // 截断无限流，长度为起始时间和结束时间的差+1个
	     .limit(ChronoUnit.DAYS.between(start, end) + 1)
	     // 由于最后要的是字符串，所以map转换一下
	     .map(LocalDate::toString)
	     // 把流收集为List
	     .collect(Collectors.toList());
}

~~~

**Stream 学习**

[详情查看](http://www.cnblogs.com/CarpenterLee/p/6545321.html)

[详情查看2](https://www.cnblogs.com/CarpenterLee/p/6550212.html)

