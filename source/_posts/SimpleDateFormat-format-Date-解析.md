---
title: SimpleDateFormat.format(Date)解析
date: 2019-04-17 15:48:54
tags: Java
---

# 格式化原理

用过`SimpleDateFormat`的朋友都知道，`format`方法会根据构造的`pattern`解析日期，比较常见的：

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); // 24小时制
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"); // 12小时制
```

翻看了源码，发现jdk的格式化思路是将`Date`转换成`Calendar`，将诸如`yyyy-MM-dd hh:mm:ss`的格式化`pattern`转化成一个数组，对于数组中每个元素，找到对应的`Calendar`中的`Field`，最后用`get(int field)`方法拿到对应的数值。

```java
// Map index into pattern character string to Calendar field number
// 这里写的很清楚，将pattern字符映射到Calendar的field
private static final int[] PATTERN_INDEX_TO_CALENDAR_FIELD =
{
    Calendar.ERA, Calendar.YEAR, Calendar.MONTH, Calendar.DATE,
    Calendar.HOUR_OF_DAY, Calendar.HOUR_OF_DAY, Calendar.MINUTE,
    Calendar.SECOND, Calendar.MILLISECOND, Calendar.DAY_OF_WEEK,
    Calendar.DAY_OF_YEAR, Calendar.DAY_OF_WEEK_IN_MONTH,
    Calendar.WEEK_OF_YEAR, Calendar.WEEK_OF_MONTH,
    Calendar.AM_PM, Calendar.HOUR, Calendar.HOUR, Calendar.ZONE_OFFSET,
    Calendar.ZONE_OFFSET,
    // Pseudo Calendar fields
    CalendarBuilder.WEEK_YEAR,
    CalendarBuilder.ISO_DAY_OF_WEEK,
    Calendar.ZONE_OFFSET
};
```

```java
/**
 * Private member function that does the real date/time formatting.
 * 实际完成格式化工作的方法
 */
private void subFormat(int patternCharIndex, int count,
                       FieldDelegate delegate, StringBuffer buffer,
                       boolean useDateFormatSymbols)
{
    int     maxIntCount = Integer.MAX_VALUE;
    String  current = null;
    int     beginOffset = buffer.length();

    // 从映射中拿到Calendar的field索引值
    int field = PATTERN_INDEX_TO_CALENDAR_FIELD[patternCharIndex];
    int value;
    if (field == CalendarBuilder.WEEK_YEAR) {
        if (calendar.isWeekDateSupported()) {
            value = calendar.getWeekYear();
        } else {
            // use calendar year 'y' instead
            patternCharIndex = PATTERN_YEAR;
            field = PATTERN_INDEX_TO_CALENDAR_FIELD[patternCharIndex];
            value = calendar.get(field);
        }
    } else if (field == CalendarBuilder.ISO_DAY_OF_WEEK) {
        value = CalendarBuilder.toISODayOfWeek(calendar.get(Calendar.DAY_OF_WEEK));
    } else {
        // 获取真实值
        value = calendar.get(field);
    }
```

当调用`SimpleDateFormat.format(Date)`方法时，实际是如下的调用：

```flow
st=>operation: SimpleDateFormat.format(Date)
op1=>operation: SimpleDateFormat.format(Date,StringBuffer,FieldPosition)
op2=>operation: SimpleDateFormat.format(Date,StringBuffer,FieldDelegate)
op3=>operation: SimpleDateFormat.subFormat(int, int,FieldDelegate,StringBuffer,boolean)
st->op1->op2->op3
```

`pattern`字符串的转换过程是在构造`SimpleDateFormat`时就完成了，具体可以查看类中的`compile(String pattern)`方法。

# 测试

```java
Calendar c = Calendar.getInstance();
c.setTime(date);
System.out.println(c.get(10)); // 4。  这里的10对应 Calendar.HOUR
System.out.println(c.get(11)); // 16。 11对应 Calendar.HOUR_OF_DAY
```

