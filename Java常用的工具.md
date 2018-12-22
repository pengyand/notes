#### 时间工具类-LocalDateTime

> java8新出的时间工具类：
>
> 链接：https://my.oschina.net/githubhty/blog/1610049
>
> ### 1、创建
>
> 根据年、月、日、时、分、秒、纳秒等创建LocalDateTime
>
> ```java
> eg：
> LocalTime zero = LocalTime.of(0, 0, 0); // 00:00:00
> LocalTime mid = LocalTime.parse("12:00:00"); // 12:00:00
> LocalTime now = LocalTime.now(); // 23:11:08.006
> all method
> LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute)
> LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second)
> LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond)
> LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute)
> LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second)
> LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond)
> LocalDateTime of(LocalDate date, LocalTime time)
> ```
>
> ### 2、LocalDatetime 的所有方法：
>
> ```java
> eg：
> // 取当前日期：
> LocalDate today = LocalDate.now(); // -> 2014-12-24
> // 根据年月日取日期：
> LocalDate crischristmas = LocalDate.of(2014, 12, 25); // -> 2014-12-25
> // 根据字符串取：
> LocalDate endOfFeb = LocalDate.parse("2014-02-28"); // 严格按照ISO yyyy-MM-dd验证，02写成2都不行，当然也有一个重载方法允许自己定义格式
> LocalDate.parse("2014-02-29"); // 无效日期无法通过：DateTimeParseException: Invalid date
> // 取本月第1天：
> LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth()); // 2017-03-01
> // 取本月第2天：
> LocalDate secondDayOfThisMonth = today.withDayOfMonth(2); // 2017-03-02
> // 取本月最后一天，再也不用计算是28，29，30还是31：
> LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth()); // 2017-12-31
> // 取下一天：
> LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1); // 变成了2018-01-01
> // 取2017年1月第一个周一，用Calendar要死掉很多脑细胞：
> LocalDate firstMondayOf2015 = LocalDate.parse("2017-01-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2017-01-02
> ```

> ~~~properties
> 所有的方法:
> 1.	adjustInto	调整指定的Temporal和当前LocalDateTime对
> 2.	atOffset	结合LocalDateTime和ZoneOffset创建一个
> 3.	atZone	结合LocalDateTime和指定时区创建一个ZonedD
> 4.	compareTo	比较两个LocalDateTime
> 5.	format	格式化LocalDateTime生成一个字符串
> 6.	from	转换TemporalAccessor为LocalDateTi
> 7.	get	得到LocalDateTime的指定字段的值
> 8.	getDayOfMonth	得到LocalDateTime是月的第几天
> 9.	getDayOfWeek	得到LocalDateTime是星期几
> 10.	getDayOfYear	得到LocalDateTime是年的第几天
> 11.	getHour	得到LocalDateTime的小时
> 12.	getLong	得到LocalDateTime指定字段的值
> 13.	getMinute	得到LocalDateTime的分钟
> 14.	getMonth	得到LocalDateTime的月份
> 15.	getMonthValue	得到LocalDateTime的月份，从1到12
> 16.	getNano	得到LocalDateTime的纳秒数
> 17.	getSecond	得到LocalDateTime的秒数
> 18.	getYear	得到LocalDateTime的年份
> 19.	isAfter	判断LocalDateTime是否在指定LocalDateT
> 20.	isBefore	判断LocalDateTime是否在指定LocalDateT
> 21.	isEqual	判断两个LocalDateTime是否相等
> 22.	isSupported	判断LocalDateTime是否支持指定时间字段或单元
> 23.	minus	返回LocalDateTime减去指定数量的时间得到的值
> 24.	minusDays	返回LocalDateTime减去指定天数得到的值
> 25.	minusHours	返回LocalDateTime减去指定小时数得到的值
> 26.	minusMinutes	返回LocalDateTime减去指定分钟数得到的值
> 27.	minusMonths	返回LocalDateTime减去指定月数得到的值
> 28.	minusNanos	返回LocalDateTime减去指定纳秒数得到的值
> 29.	minusSeconds	返回LocalDateTime减去指定秒数得到的值
> 30.	minusWeeks	返回LocalDateTime减去指定星期数得到的值
> 31.	minusYears	返回LocalDateTime减去指定年数得到的值
> 32.	now	返回指定时钟的当前LocalDateTime
> 33.	of	根据年、月、日、时、分、秒、纳秒等创建LocalDateTi
> 34.	ofEpochSecond	根据秒数(从1970-01-0100:00:00开始)创建L
> 35.	ofInstant	根据Instant和ZoneId创建LocalDateTim
> 36.	parse	解析字符串得到LocalDateTime
> 37.	plus	返回LocalDateTime加上指定数量的时间得到的值
> 38.	plusDays	返回LocalDateTime加上指定天数得到的值
> 39.	plusHours	返回LocalDateTime加上指定小时数得到的值
> 40.	plusMinutes	返回LocalDateTime加上指定分钟数得到的值
> 41.	plusMonths	返回LocalDateTime加上指定月数得到的值
> 42.	plusNanos	返回LocalDateTime加上指定纳秒数得到的值
> 43.	plusSeconds	返回LocalDateTime加上指定秒数得到的值
> 44.	plusWeeks	返回LocalDateTime加上指定星期数得到的值
> 45.	plusYears	返回LocalDateTime加上指定年数得到的值
> 46.	query	查询LocalDateTime
> 47.	range	返回指定时间字段的范围
> 48.	toLocalDate	返回LocalDateTime的LocalDate部分
> 49.	toLocalTime	返回LocalDateTime的LocalTime部分
> 50.	toString	返回LocalDateTime的字符串表示
> 51.	truncatedTo	返回LocalDateTime截取到指定时间单位的拷贝
> 52.	until	计算LocalDateTime和另一个LocalDateTi
> 53.	with	返回LocalDateTime指定字段更改为新值后的拷贝
> 54.	withDayOfMonth	返回LocalDateTime月的第几天更改为新值后的拷贝
> 55.	withDayOfYear	返回LocalDateTime年的第几天更改为新值后的拷贝
> 56.	withHour	返回LocalDateTime的小时数更改为新值后的拷贝
> 57.	withMinute	返回LocalDateTime的分钟数更改为新值后的拷贝
> 58.	withMonth	返回LocalDateTime的月份更改为新值后的拷贝
> 59.	withNano	返回LocalDateTime的纳秒数更改为新值后的拷贝
> 60.	withSecond	返回LocalDateTime的秒数更改为新值后的拷贝
> 61.	withYear	返回LocalDateTime年份更改为新值后的拷贝
> ~~~
>
> ### 3、对应的SQL的类型
>
> ```java
> SQL -> Java
> 
> date -> LocalDate
> time -> LocalTime
> timestamp -> LocalDateTime
> ```

### ObjectMapper

> https://blog.csdn.net/zxc_user/article/details/79713586
>
> 主要是用来把对象转换成为一个json字符串返回到前端
>
> ~~~java
> 		ObjectMapper objectMapper = new ObjectMapper();
> 		
> 		//序列化的时候序列对象的所有属性
> 		objectMapper.setSerializationInclusion(Include.ALWAYS);
> 		
> 		//反序列化的时候如果多了其他属性,不抛出异常
> 		objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
> 		
> 		//如果是空对象的时候,不抛异常
> 		objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
> 		
> 		//取消时间的转化格式,默认是时间戳,可以取消,同时需要设置要表现的时间格式
> 		objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
> 		objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"))
> 
> ~~~
>
>

#### Preconditions 

> 优雅的参数检查
>
> http://www.cnblogs.com/peida/p/Guava_Preconditions.html

