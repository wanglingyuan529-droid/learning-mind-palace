# JDK 时间 API 与正则分组教程

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"
> 覆盖：常用API-14 ~ 19（捕获分组、Date、SimpleDateFormat、Calendar、JDK8 时间类）
> 所有示例和输出都是在本机 JDK 27 上用 jshell 实测跑出来的

---

## 0. 一句话认识

| 内容 | 解决什么问题 | 现状 |
|---|---|---|
| **捕获分组** `(...)` | 从匹配结果里把想要的部分单独取出来 | 正则必备 |
| **非捕获分组** `(?:...)` | 只要分组效果，不占编号、不占内存 | 正则必备 |
| **Date** | 表示一个"时刻"（毫秒时间戳） | **JDK7 老 API，能用但坑多** |
| **SimpleDateFormat** | Date 和字符串互相转换 | **JDK7，线程不安全** |
| **Calendar** | 日历操作（加年份、取星期） | **JDK7，可变、坑多** |
| **JDK8 时间类** | 上面三个的现代替代品 | ✅ **新项目一律用它** |

**一句话结论：JDK7 的三个类你只要能看懂老代码就行，自己写代码全部用 JDK8 的。**

---

# 第一部分：捕获分组和非捕获分组

## 1. 捕获分组 `( )`

括号里的内容会被"捕获"，可以单独取出来：

```java
import java.util.regex.*;

Matcher m = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})").matcher("2024-10-19");
m.find();

m.groupCount();   // 3      ← 一共 3 个捕获分组
m.group(0);       // 2024-10-19   整段
m.group(1);       // 2024         第 1 个括号
m.group(2);       // 10           第 2 个括号
m.group(3);       // 19           第 3 个括号
```

## 2. 非捕获分组 `(?: )`

在括号里加 `?:`，表示"**只当整体用，别捕获**"：

```java
// 捕获分组：3 个组
Matcher m1 = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})").matcher("2024-10-19");
m1.find();
m1.groupCount();   // 3
m1.group(1);       // 2024

// 非捕获分组：年份不捕获
Matcher m2 = Pattern.compile("(?:\\d{4})-(\\d{2})-(\\d{2})").matcher("2024-10-19");
m2.find();
m2.groupCount();   // 2      ← 少了一个
m2.group(1);       // 10     ← 编号前移了！现在是月，不是年
```

**关键理解**：`(?:...)` 不占编号，所以**后面的分组编号会往前挪**。这是最容易搞错的地方。

## 3. 为什么要用非捕获分组

### 原因 1：只需要"或"的效果，不需要取内容

```java
// 只判断是不是 2024-10-19 或 2024/10/19 这种格式，不需要取年月日
Pattern.compile("\\d{4}[-/]\\d{2}[-/]\\d{2}");           // 不用括号也行
Pattern.compile("(?:\\d{4})[-/](?:\\d{2})[-/]\\d{2}");   // 只要分组效果 → 用 (?:)
```

### 原因 2：性能，少捕获一个少存一份

```java
"(?:\\d{4})-(\\d{2})-(\\d{2})"   // 比你真需要年份时少存一份，图个干净
```

### 原因 3（最重要）：或运算时，不让别的分支"抢编号"

这是个真坑，看实测：

```java
// 要求：既能匹配 2024-10-19，也能匹配 19/10/2024，并且都能取年月日
String both = "(?:(\\d{4})-(\\d{2})-(\\d{2}))|(?:(\\d{2})/(\\d{2})/(\\d{4}))";

Matcher m = Pattern.compile(both).matcher("19/10/2024");
m.find();

m.groupCount();   // 6
m.group(3);       // null   ← 第一个分支的"月"没参与匹配，返回 null
m.group(4);       // 19     ← 第二个分支的"日"
m.group(6);       // 2024   ← 第二个分支的"年"
```

**记住**：`|` 两边的分组编号是**共享**的，匹配时才决定哪个分支有值，**没匹配到的分支全是 null**。取值前一定要判空。

## 4. 命名分组 `(?<名字> )`（推荐）

数字分组多了容易数错，直接起名字：

```java
Matcher m = Pattern.compile("(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})")
                   .matcher("2024-10-19");
m.find();

m.group("year");    // 2024
m.group("month");   // 10
m.group("day");     // 19
```

命名分组**照样占编号**，`group(1)` 也还是能取到年份。名字只是多了个好记的访问方式。

---

# 第二部分：JDK7 时间类

## 5. Date（表示一个时刻）

### 5.1 创建和取值

```java
import java.util.Date;

Date now = new Date();          // 当前时间
Date epoch = new Date(0);       // 从 1970-01-01 00:00:00 UTC 开始算的毫秒数

epoch.getTime();     // 0        ← 毫秒时间戳
now.getTime();       // 比如 1729...（long）
```

### 5.2 toString() 的样子（注意时区）

```java
System.out.println(new Date(0));
// 输出：Thu Jan 01 08:00:00 CST 1970
```

**为什么是 8 点不是 0 点？** 因为 `Date(0)` 是 UTC 零点，本机在东八区，显示成本地时间就是 8 点。**Date 是个"时刻"，显示成什么格式取决于时区。**

### 5.3 比较

```java
Date a = new Date(0);
Date b = new Date(1000);

a.before(b);        // true
a.after(b);         // false
a.compareTo(b);     // -1（a < b）
a.compareTo(a);     // 0
```

### 5.4 Date 的坑

| 坑 | 说明 |
|---|---|
| **可变** | `date.setTime(...)` 能改，传给别人可能被改掉 |
| **toString 格式固定** | 无法直接输出 `2024-10-19`，必须配合 SimpleDateFormat |
| **月份从 0 开始？** | 不，Date 没有月份方法，是 Calendar 的锅（见第 7 节） |

---

## 6. SimpleDateFormat（Date ↔ 字符串）

### 6.1 格式化和解析

```java
import java.text.SimpleDateFormat;

SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

// 格式化：Date -> String
String s = sdf.format(new Date(0));
// s = "1970-01-01 08:00:00"

// 解析：String -> Date
Date d = sdf.parse("2024-10-19 12:30:45");
// d = Sat Oct 19 12:30:45 CST 2024
```

### 6.2 模式字母表（最常用的）

| 字母 | 含义 | 例子 |
|---|---|---|
| `yyyy` | 年（4 位） | 2024 |
| `MM` | 月（2 位） | 10 |
| `dd` | 日（2 位） | 19 |
| `HH` | 时（24 小时制） | 14 |
| `hh` | 时（12 小时制，要配 `a`） | 02 |
| `mm` | 分 | 30 |
| `ss` | 秒 | 45 |
| `SSS` | 毫秒 | 123 |
| `a` | 上午/下午 | PM |
| `E` | 星期 | SAT |

实测：

```java
new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(0));  // 1970-01-01 08:00:00
new SimpleDateFormat("yyyy-MM-dd hh:mm:ss a").format(new Date(0)); // 1970-01-01 08:00:00 AM
```

### 6.3 坑 1：解析格式必须严格匹配

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
sdf.parse("2024-10-19");
// 抛 ParseException: Unparseable date: "2024-10-19"
```

**格式串里有 `HH:mm:ss`，输入里就必须有**。想解析"只有日期"的字符串，得换一个 `yyyy-MM-dd` 的格式对象。

### 6.4 坑 2：`yy` 两位年份有歧义

```java
DateTimeFormatter yy = DateTimeFormatter.ofPattern("yy-MM-dd");
LocalDate.parse("24-10-19", yy);   // 2024-10-19
LocalDate.parse("00-01-01", yy);   // 2000-01-01
LocalDate.parse("99-01-01", yy);   // 2099-01-01
```

实测两位年份落在 **2000~2099**（`00`→2000，`99`→2099）。

**建议永远用 `yyyy`**，省得猜。

### 6.5 坑 3：线程不安全（最严重，实测复现）

SimpleDateFormat **不是线程安全的**。多个线程共享同一个对象时，会输出错乱数据。

实测（16 个线程共享 1 个 SimpleDateFormat，各跑 20000 次 format + parse）：

```
16 线程共享 1 个 SimpleDateFormat，20000 次出错: 19091 次   ← 几乎全错！
```

换成 DateTimeFormatter 做同样的事：

```
16 线程共享 1 个 DateTimeFormatter，20000 次出错: 0 次
```

**怎么避免：**
- 不要在 `static` 字段里放 SimpleDateFormat
- 每次用之前 new 一个（最简单，但频繁创建有开销）
- 用 `ThreadLocal<SimpleDateFormat>` 包一层（老项目的标准做法）
- **新代码直接用 `DateTimeFormatter`**（不可变 + 线程安全）

---

## 7. Calendar（日历操作）

### 7.1 创建和基本用法

```java
import java.util.Calendar;

Calendar c = Calendar.getInstance();     // 拿当前时间
c.set(2024, Calendar.OCTOBER, 19);       // 设置年月日
c.get(Calendar.YEAR);                    // 2024
c.get(Calendar.MONTH);                   // 9  ← 注意！
c.get(Calendar.DAY_OF_MONTH);            // 19
c.getTime();                             // 转回 Date
```

### 7.2 坑 1：MONTH 从 0 开始（最坑）

```java
Calendar.OCTOBER;          // 9   ← 10 月是 9！
Calendar.JANUARY;          // 0
Calendar.DECEMBER;         // 11

c.set(2024, 10, 19);
// 实际得到的是 2024年11月19日，不是 10 月！
```

**记法：`MONTH` 永远要 `+1` 才是人看的月份，人给的月份要 `-1` 才能 set。** 用 `Calendar.OCTOBER` 常量最稳。

### 7.3 坑 2：DAY_OF_WEEK 周日是 1

```java
c.set(2024, Calendar.OCTOBER, 19);     // 2024-10-19 是周六
c.get(Calendar.DAY_OF_WEEK);           // 7

Calendar.SUNDAY;    // 1   ← 周日是 1
Calendar.MONDAY;    // 2
Calendar.SATURDAY;  // 7
```

**和 JDK8 的 `DayOfWeek` 完全相反**：`DayOfWeek.MONDAY.getValue()` 是 **1**，`SUNDAY` 是 **7**。

### 7.4 加时间：add

```java
c.set(2024, Calendar.OCTOBER, 31);      // 10 月 31 日
c.add(Calendar.DAY_OF_MONTH, 1);
// 得到 11月1日  ← 自动跨月，不用自己判断

c.add(Calendar.MONTH, -1);              // 减一个月
```

### 7.5 坑 3：set 不设毫秒，会保留当前毫秒

```java
Calendar c = Calendar.getInstance();
c.set(2024, Calendar.OCTOBER, 19, 10, 30, 0);   // 只设到秒
c.get(Calendar.MILLISECOND);                     // 971  ← 不是 0！
c.getTimeInMillis();                             // 1729305000971  ← 末尾带着毫秒
```

**想精确到秒，必须补一句 `c.set(Calendar.MILLISECOND, 0)`。**

### 7.6 Calendar 的坑总结

| 坑 | 后果 |
|---|---|
| `MONTH` 从 0 开始 | 10 月写成 10 → 变成 11 月 |
| `DAY_OF_WEEK` 周日=1 | 和 `DayOfWeek` 相反，容易搞混 |
| 不设毫秒保留原值 | 时间对不上，比较时出错 |
| **可变对象** | 传参给别人可能被改掉 |

**结论：新代码别用 Calendar，直接用 JDK8 的 `LocalDate` 等。**

---

# 第三部分：JDK8 时间类（重点，新代码用这个）

## 8. 为什么 JDK8 时间类更好

| 对比项 | JDK7（Date/Calendar） | JDK8（LocalDate 等） |
|---|---|---|
| 可变性 | 可变（能改） | **不可变**（改就返回新对象） |
| 线程安全 | 不安全 | **线程安全** |
| 月份 | 从 0 开始 | **从 1 开始（正常了！）** |
| API 命名 | 混乱（get/set/add） | 清晰（plus/minus/with） |
| 时区 | 藏在 Date 里 | 显式的 `ZoneId` |

## 9. LocalDate / LocalTime / LocalDateTime

### 9.1 三个类分工

| 类 | 表示 | 例子 |
|---|---|---|
| `LocalDate` | 只有日期 | 2024-10-19 |
| `LocalTime` | 只有时间 | 10:30:45 |
| `LocalDateTime` | 日期 + 时间 | 2024-10-19T10:30:45 |

### 9.2 创建

```java
import java.time.*;

LocalDate d = LocalDate.of(2024, 10, 19);          // 注意：10 就是 10 月！
LocalDate today = LocalDate.now();                 // 当前日期
LocalDate parsed = LocalDate.parse("2024-10-19");  // 默认格式就是 yyyy-MM-dd

LocalTime t = LocalTime.of(10, 30, 45);            // 10:30:45
LocalDateTime dt = LocalDateTime.of(2024, 10, 19, 10, 30, 45);
// dt = 2024-10-19T10:30:45
```

### 9.3 取值

```java
LocalDate d = LocalDate.of(2024, 10, 19);

d.getYear();          // 2024
d.getMonthValue();    // 10     ← 正常了，从 1 开始
d.getMonth();         // OCTOBER（枚举）
d.getDayOfMonth();    // 19
d.getDayOfWeek();     // SATURDAY（枚举）
d.isLeapYear();       // true   （2024 是闰年）
d.lengthOfMonth();    // 31     （10 月有 31 天）
```

### 9.4 加减（不可变！）

```java
LocalDate d = LocalDate.of(2024, 10, 19);

d.plusDays(1);        // 2024-10-20
d.plusMonths(3);      // 2025-01-19（自动跨年）
d.minusDays(1);       // 2024-10-18

// 重要：不会改原对象！
d;                    // 还是 2024-10-19
```

**必须接住返回值**，`d.plusDays(1);` 单独一行等于白写。

### 9.5 with：改某一个字段

```java
LocalDate d = LocalDate.of(2024, 10, 19);

d.withYear(2025);       // 2025-10-19
d.withMonth(12);        // 2024-12-19
d.withDayOfMonth(1);    // 2024-10-01
```

### 9.6 比较

```java
LocalDate a = LocalDate.of(2024, 10, 19);
LocalDate b = LocalDate.of(2024, 12, 25);

a.isBefore(b);     // true
a.isAfter(b);      // false
a.isEqual(a);      // true
a.compareTo(b);    // -1
```

---

## 10. Instant（时间戳）

`Instant` 表示"从 1970-01-01 00:00:00 UTC 到现在的秒数/毫秒数"，是**纯时间戳，不带时区**。

```java
Instant i = Instant.ofEpochMilli(0);
// i = 1970-01-01T00:00:00Z

i.getEpochSecond();    // 0      ← 秒
i.toEpochMilli();      // 0      ← 毫秒

Instant now = Instant.now();   // 当前时间戳
```

**`Instant` 和 `Date` 的关系**：`Date` 本质上就是个毫秒时间戳，`Instant` 是它的现代等价物。

```java
Date d = Date.from(Instant.ofEpochMilli(0));    // Instant -> Date
Instant i = d.toInstant();                       // Date -> Instant
```

---

## 11. 时区：ZoneId 和 ZonedDateTime

### 11.1 同一时刻，不同时区显示不同时间

```java
Instant fixed = Instant.ofEpochMilli(1729000000000L);   // 固定一个时刻

fixed.atZone(ZoneId.of("Asia/Shanghai"));    // 2024-10-15T21:46:40+08:00[Asia/Shanghai]
fixed.atZone(ZoneId.of("Asia/Tokyo"));       // 2024-10-15T22:46:40+09:00[Asia/Tokyo]
fixed.atZone(ZoneId.of("America/New_York")); // 2024-10-15T09:46:40-04:00[America/New_York]

// 这三个是"同一个时刻"，只是显示不同
fixed.atZone(ZoneId.of("Asia/Shanghai")).toInstant()
     .equals(fixed.atZone(ZoneId.of("America/New_York")).toInstant());   // true
```

### 11.2 ZonedDateTime

```java
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
zdt.getZone();          // Asia/Shanghai
zdt.toInstant();        // 转成时间戳
zdt.toLocalDateTime();  // 转成不带时区的本地时间
```

### 11.3 常用时区 ID

| 时区 ID | 说明 |
|---|---|
| `Asia/Shanghai` | 中国标准时间（UTC+8） |
| `Asia/Tokyo` | 日本（UTC+9） |
| `UTC` | 世界标准时间 |
| `America/New_York` | 美国东部 |

**不要用 `GMT+8` 这种写法**（虽然能用），`Asia/Shanghai` 更准确（会处理历史夏令时）。

---

## 12. DateTimeFormatter（格式化，线程安全）

### 12.1 格式化和解析

```java
import java.time.format.DateTimeFormatter;

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

// 格式化：时间 -> 字符串
String s = fmt.format(LocalDateTime.of(2024, 10, 19, 10, 30, 45));
// s = "2024-10-19 10:30:45"

// 解析：字符串 -> 时间
LocalDateTime dt = LocalDateTime.parse("2024-10-19 10:30:45", fmt);
// dt = 2024-10-19T10:30:45
```

### 12.2 常用格式串实测

```java
LocalDateTime dt = LocalDateTime.of(2024, 10, 19, 14, 5, 9);

DateTimeFormatter.ofPattern("yyyy-MM-dd").format(dt);        // 2024-10-19
DateTimeFormatter.ofPattern("yyyy年MM月dd日").format(dt);     // 2024年10月19日
DateTimeFormatter.ofPattern("HH:mm:ss").format(dt);          // 14:05:09
DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm").format(dt);  // 2024/10/19 14:05
```

### 12.3 坑：解析格式必须能对上

```java
LocalDateTime.parse("2024-10-19");
// 抛 DateTimeParseException: Text '2024-10-19' could not be parsed at index 10
// 因为 LocalDateTime 默认要求 "2024-10-19T10:30" 这种完整格式

// 正确做法：给格式，或者用对应的类
LocalDate.parse("2024-10-19");                     // ✅ 用 LocalDate
LocalDateTime.parse("2024-10-19T10:30");           // ✅ 带上 T
LocalDateTime.parse("2024-10-19 10:30", fmt);      // ✅ 自定义格式
```

**经验**：只有日期就用 `LocalDate.parse`，只有时间用 `LocalTime.parse`，别拿 `LocalDateTime` 硬套。

### 12.4 和 SimpleDateFormat 的对比（记住结论）

| | SimpleDateFormat | DateTimeFormatter |
|---|---|---|
| 线程安全 | ❌ 不安全（实测 20000 次错 19091 次） | ✅ 安全（20000 次 0 错） |
| 可变 | 可变 | 不可变 |
| 打包 | `java.text` | `java.time.format` |
| 结论 | 老代码兼容用 | **新代码一律用这个** |

---

# 第四部分：JDK8 日历和工具类

## 13. Period（日期间隔，按年月日分开算）

```java
LocalDate start = LocalDate.of(2024, 1, 1);
LocalDate end = LocalDate.of(2024, 3, 15);

Period p = Period.between(start, end);
// p = P2M14D  （2 个月 14 天）

p.getMonths();   // 2
p.getDays();     // 14

p.getYears();    // 0
p.toTotalMonths(); // 2   ← 注意是 to 不是 get
```

### 13.1 坑：getDays() 不是总天数

```java
Period.between(LocalDate.of(2024,1,1), LocalDate.of(2024,3,15)).getDays();   // 14，不是 74！
```

**`Period` 是"2 个月零 14 天"这种拆开算的表示**，`getDays()` 只返回"零头天数"。想要总天数，用 `ChronoUnit`（下一节）。

## 14. ChronoUnit（算总差值）

```java
import java.time.temporal.ChronoUnit;

LocalDate start = LocalDate.of(2024, 1, 1);
LocalDate end = LocalDate.of(2024, 3, 15);

ChronoUnit.DAYS.between(start, end);     // 74
ChronoUnit.MONTHS.between(start, end);   // 2
ChronoUnit.YEARS.between(LocalDate.of(2020, 1, 1), LocalDate.of(2024, 10, 19));  // 4

// 也可以对时间用
ChronoUnit.HOURS.between(LocalTime.of(9, 0), LocalTime.of(17, 30));   // 8
ChronoUnit.MINUTES.between(LocalTime.of(9, 0), LocalTime.of(17, 30)); // 510
```

**记忆：要"总共多少天/月/年"用 `ChronoUnit`；要"几年零几个月零几天"用 `Period`。**

## 15. Duration（时间间隔）

专门算"两个时刻差多少秒/毫秒"：

```java
Duration dur = Duration.between(LocalTime.of(9, 0), LocalTime.of(17, 30));
// dur = PT8H30M

dur.toHours();     // 8     （小时数，取整）
dur.toMinutes();   // 510   （总分钟）
dur.getSeconds();  // 30600 （总秒）
dur.toMillis();    // 30600000
```

**`Duration` 是绝对时间差**（秒/纳秒），**`Period` 是日历差**（年月日）。别混用。

## 16. TemporalAdjusters（找特定日期）

```java
import java.time.temporal.TemporalAdjusters;

LocalDate d = LocalDate.of(2024, 10, 19);

d.with(TemporalAdjusters.firstDayOfMonth());   // 2024-10-01   本月第一天
d.with(TemporalAdjusters.lastDayOfMonth());    // 2024-10-31   本月最后一天
d.with(TemporalAdjusters.next(DayOfWeek.MONDAY));  // 2024-10-21  下一个周一
d.with(TemporalAdjusters.previous(DayOfWeek.MONDAY)); // 上一个周一
d.with(TemporalAdjusters.dayOfWeekInMonth(3, DayOfWeek.MONDAY)); // 本月第 3 个周一
d.with(TemporalAdjusters.firstDayOfNextMonth());   // 下个月第一天
```

**用途**：算"本月第一天/最后一天""下一个周一""本月第 N 个周几"这类固定规则，不用自己写循环。

## 17. DayOfWeek / Month 枚举

```java
DayOfWeek.MONDAY.getValue();    // 1   ← 周一 = 1
DayOfWeek.SUNDAY.getValue();    // 7   ← 周日 = 7
DayOfWeek.SATURDAY.getValue();  // 6

// 和 Calendar 完全相反！
// Calendar: SUNDAY = 1, SATURDAY = 7
```

---

# 第五部分：对照与速查

## 18. 新旧 API 对照表（用哪个）

| 想干什么 | JDK7（老） | JDK8（新，推荐） |
|---|---|---|
| 当前时刻 | `new Date()` | `Instant.now()` / `LocalDateTime.now()` |
| 当前日期 | `new Date()` | `LocalDate.now()` |
| 只要时间戳 | `date.getTime()` | `Instant.now().toEpochMilli()` |
| 格式化 | `SimpleDateFormat` | `DateTimeFormatter` |
| 解析字符串 | `SimpleDateFormat.parse` | `LocalDate.parse` / `LocalTime.parse` |
| 加减天 | `Calendar.add` | `LocalDate.plusDays` |
| 取星期 | `Calendar.get(DAY_OF_WEEK)`（周日=1） | `LocalDate.getDayOfWeek()`（周一=1） |
| 取月份 | `Calendar.get(MONTH)`（0-11） | `LocalDate.getMonthValue()`（1-12） |
| 两个日期差几天 | 自己算毫秒 | `ChronoUnit.DAYS.between` |
| 跨时区 | `TimeZone` + `Calendar` | `ZoneId` + `ZonedDateTime` |

## 19. 常见错误速查（照着躲）

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| `new SimpleDateFormat(...)` 放 static 字段 | 每次 new / `ThreadLocal` / 用 `DateTimeFormatter` | 多线程输出错乱（实测 19091/20000 错） |
| `calendar.set(2024, 10, 19)` 以为是 10 月 | `calendar.set(2024, Calendar.OCTOBER, 19)` | 变成 11 月 |
| `calendar.get(Calendar.MONTH)` 直接当月份用 | `+1` | 比实际月份少 1 |
| `calendar.set(...)` 不设毫秒 | 加 `set(Calendar.MILLISECOND, 0)` | 时间对不上 |
| `LocalDate.plusDays(1);` 不接返回值 | `d = d.plusDays(1);` | 没生效（不可变） |
| `LocalDateTime.parse("2024-10-19")` | `LocalDate.parse("2024-10-19")` | 抛 DateTimeParseException |
| `Period.between(...).getDays()` 当总天数 | `ChronoUnit.DAYS.between(...)` | 得到零头，不是总数 |
| 用 `yy` 不用 `yyyy` | 一律 `yyyy` | 两位年份有歧义 |
| `SimpleDateFormat.parse` 不处理异常 | try-catch `ParseException` | 编译不过 |

## 20. 一页速查表

### JDK8 创建

| 想要 | 写法 |
|---|---|
| 当前日期 | `LocalDate.now()` |
| 指定日期 | `LocalDate.of(2024, 10, 19)` |
| 当前时间 | `LocalTime.now()` |
| 当前日期时间 | `LocalDateTime.now()` |
| 时间戳 | `Instant.now()` |
| 从字符串 | `LocalDate.parse("2024-10-19")` |
| 格式化 | `DateTimeFormatter.ofPattern("yyyy-MM-dd").format(dt)` |
| 当天 0 点 | `LocalDate.now().atStartOfDay()` |

### JDK8 操作

| 想要 | 写法 |
|---|---|
| 加天/月/年 | `plusDays/plusMonths/plusYears` |
| 减 | `minusDays` 等 |
| 改字段 | `withYear/withMonth/withDayOfMonth` |
| 比较 | `isBefore/isAfter/isEqual/compareTo` |
| 间隔天数 | `ChronoUnit.DAYS.between(a, b)` |
| 拆分间隔 | `Period.between(a, b)` |
| 时刻差 | `Duration.between(t1, t2)` |
| 本月第一天 | `d.with(TemporalAdjusters.firstDayOfMonth())` |

### 新旧互转

| 方向 | 写法 |
|---|---|
| Date → Instant | `date.toInstant()` |
| Instant → Date | `Date.from(instant)` |
| Date → LocalDateTime | `LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault())` |
| LocalDateTime → Date | `Date.from(ldt.atZone(ZoneId.systemDefault()).toInstant())` |
| 毫秒 → Instant | `Instant.ofEpochMilli(ms)` |

---

## 21. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.time.*;
import java.time.format.*;
import java.time.temporal.*;

// ① 算自己活了多少天
LocalDate birth = LocalDate.of(2000, 1, 1);
System.out.println(ChronoUnit.DAYS.between(birth, LocalDate.now()) + " 天");

// ② 格式化当前时间
System.out.println(LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss")));

// ③ 本月第一天和最后一天
LocalDate today = LocalDate.now();
System.out.println(today.with(TemporalAdjusters.firstDayOfMonth()));
System.out.println(today.with(TemporalAdjusters.lastDayOfMonth()));

// ④ 下个周一
System.out.println(today.with(TemporalAdjusters.next(DayOfWeek.MONDAY)));

// ⑤ 解析一个日期字符串
System.out.println(LocalDate.parse("2024-10-19").getDayOfWeek());

// ⑥ 故意踩坑：Calendar 的月份
java.util.Calendar c = java.util.Calendar.getInstance();
c.set(2024, 10, 19);
System.out.println("set(2024,10,19) 实际是: " + (c.get(java.util.Calendar.MONTH) + 1) + " 月");
```

---

## 记忆口诀

> **新代码用 JDK8，Date/Calendar 只用来怀旧；**
> **LocalDate 不可变，plus 之后要接住；**
> **Calendar 月份从 0 起，星期从周日算 1；**
> **SimpleDateFormat 不线程安全，格式化用 DateTimeFormatter；**
> **要总天数用 ChronoUnit，拆开用 Period。**
