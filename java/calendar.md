# Calendar

`Calendar.getInstance()` 返回的是 Gregorian Calendar

```java
YEAR //年
MONTH //月
WEEK_OF_YEAR //一年内第几周
DATE //一月内第几天
DAY_OF_MONTH //和 DATE 值一样
DAY_OF_YEAR //一年内第几天
DAY_OF_WEEK
DAY_OF_WEEK_IN_MONTH
WEEK_OF_MONTH
AM_PM
HOUR
HOUR_OF_DAY
MINUTE
SECOND
MILLISECOND
```

```java
Calendar instance = Calendar.getInstance();
System.out.println(instance.get(Calendar.YEAR)+"年");
System.out.println((instance.get(Calendar.MONTH)+1)+"月");
System.out.println(instance.get(Calendar.DATE)+"日");
System.out.println(instance.get(Calendar.HOUR_OF_DAY)+"时"); //24小时制

switch (instance.get(Calendar.AM_PM)) {
case Calendar.AM:
    System.out.println("上午");
    break;
case Calendar.PM:
    System.out.println("下午");
    break;
}

System.out.println(instance.get(Calendar.MINUTE)+"分");
System.out.println(instance.get(Calendar.SECOND)+"秒");
System.out.println(instance.get(Calendar.MILLISECOND)+"毫秒");

```
