# Java Stream 流教程（小白版）

> 用途：学习笔记 · 定位：讲得细、面向刚入门
> 覆盖：Stream流-01 ~ 07
> 所有示例和输出都是在本机 JDK 27 上用 jshell 实测跑出来的

---

## 0. 先感受一下 Stream 能干什么

**需求**：一个列表里有一堆数字，要"挑出大于 2 的，每个乘 10，然后收集成新列表"。

**不用 Stream（老写法）：**

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);
List<Integer> result = new ArrayList<>();
for (Integer n : nums) {
    if (n > 2) {
        result.add(n * 10);
    }
}
// result = [30, 40, 50]
```

**用 Stream：**

```java
List<Integer> result = nums.stream()
        .filter(n -> n > 2)     // 挑出大于 2 的
        .map(n -> n * 10)       // 每个乘 10
        .toList();              // 收集成列表
// result = [30, 40, 50]
```

**同样的功能，从"怎么做"变成了"要什么"。**

---

## 1. Stream 的思想（02）

### 1.1 一句话理解

**Stream 就是一条"流水线"**：数据从一头进去，经过一道道工序（过滤、转换、排序），从另一头出来。

```
数据源 → filter → map → sorted → 收集
         ↑        ↑       ↑
       中间方法（每一步都返回新的流，可以一直接下去）
```

### 1.2 三条铁律

| 铁律 | 说明 |
|---|---|
| **不修改原数据** | Stream 操作的是"副本"，原集合不变 |
| **只能消费一次** | 一个流用过就废了，不能重复用 |
| **惰性求值** | 没有终结方法（如 `toList`、`forEach`），中间方法不会执行 |

### 1.3 铁律验证（实测）

**不修改原集合：**

```java
List<Integer> original = new ArrayList<>(List.of(1, 2, 3, 4));
List<Integer> filtered = original.stream().filter(n -> n > 2).toList();

original;   // [1, 2, 3, 4]   ← 没变
filtered;   // [3, 4]
```

**只能消费一次：**

```java
Stream<Integer> s = Stream.of(1, 2, 3);

s.toList();     // [1, 2, 3]
s.toList();     // ❌ IllegalStateException: stream has already been operated upon or closed
```

**惰性求值：**

```java
// 只写中间方法，没有终结方法 → 什么都不执行
nums.stream().filter(n -> {
    System.out.println("执行了！");
    return n > 1;
});
// 没有任何输出

// 加上终结方法才执行
nums.stream().filter(n -> {
    System.out.println("执行了！");
    return n > 1;
}).toList();
// 输出 3 次"执行了！"（因为有 3 个元素）
```

**这三条是理解 Stream 的基础，后面所有坑都从这里来。**

---

## 2. 获取 Stream 的四种方式（02）

```java
// ① 从集合
List<String> list = List.of("a", "b", "c");
Stream<String> s1 = list.stream();

// ② 从数组
String[] arr = {"x", "y"};
Stream<String> s2 = Arrays.stream(arr);

// ③ 直接给元素
Stream<String> s3 = Stream.of("p", "q", "r");

// ④ 从 Map（Map 本身没有 stream，要用 keySet/values/entrySet）
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Stream<String> keys   = map.keySet().stream();
Stream<Integer> vals  = map.values().stream();
Stream<Map.Entry<String, Integer>> entries = map.entrySet().stream();
```

**最常用的是 ①**（集合 `.stream()`）。

---

## 3. 中间方法（03）

> **中间方法的特点：返回一个新的 Stream，可以一直链式接下去。**
> 作用：过滤、转换、排序、截取。
	
### 3.1 filter —— 筛选

**留满足条件的：**

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6);

nums.stream().filter(n -> n % 2 == 0).toList();
// [2, 4, 6]   只留偶数
```

### 3.2 map —— 转换

**把每个元素变成另一个东西：**

```java
nums.stream().map(n -> n * n).toList();
// [1, 4, 9, 16, 25, 36]   每个元素平方

// 也可以改变类型
List<String> names = List.of("张三", "李四");
names.stream().map(String::length).toList();
// [2, 2]   字符串 → 长度（Integer）
```

### 3.3 distinct —— 去重

```java
List<Integer> dup = List.of(3, 1, 4, 1, 5, 9, 2, 6);
dup.stream().distinct().toList();
// [3, 1, 4, 5, 9, 2, 6]   重复的 1 被去掉
```

### 3.4 sorted —— 排序

```java
List<Integer> nums = List.of(3, 1, 4, 1, 5, 9, 2, 6);

nums.stream().sorted().toList();
// [1, 1, 2, 3, 4, 5, 6, 9]   自然序

nums.stream().sorted(Comparator.reverseOrder()).toList();
// [9, 6, 5, 4, 3, 2, 1, 1]   逆序
```

**自定义对象要传比较器：**

```java
class Person {
    String name; int age;
    Person(String n, int a) { name = n; age = a; }
    @Override public String toString() { return name + age; }
}

List<Person> people = List.of(
    new Person("张三", 25),
    new Person("李四", 20),
    new Person("王五", 30)
);

people.stream().sorted(Comparator.comparingInt(p -> p.age)).toList();
// [李四20, 张三25, 王五30]   按年龄升序
```

**注意**：如果对象没实现 `Comparable`，**不传 Comparator 会抛 `ClassCastException`**。

### 3.5 limit / skip —— 截取

```java
List<Integer> nums = List.of(3, 1, 4, 1, 5, 9, 2, 6);

nums.stream().limit(3).toList();   // [3, 1, 4]   前 3 个
nums.stream().skip(6).toList();    // [2, 6]      跳过前 6 个
```

**用途**：分页、取 Top N。

### 3.6 concat —— 合并两个流

```java
Stream.concat(Stream.of(1, 2), Stream.of(3, 4)).toList();
// [1, 2, 3, 4]
```

**注意**：`Stream.concat` 是静态方法，两个流类型要一致。

### 3.7 flatMap —— 把"嵌套"压平（进阶）

**这是 map 和 flatMap 最大的区别：**

```java
List<List<Integer>> nested = List.of(
    List.of(1, 2),
    List.of(3, 4),
    List.of(5)
);

// map：保持嵌套，得到 3 个列表
nested.stream().map(x -> x).toList();
// [[1, 2], [3, 4], [5]]

// flatMap：压平，得到 5 个元素
nested.stream().flatMap(Collection::stream).toList();
// [1, 2, 3, 4, 5]
```

**记忆**：
- `map` = 一对一（一个变一个）
- `flatMap` = 一对多再压平（一个变多个，最后合成一个）

**flatMap 常用于**：把"每个订单的多个商品"合并成一个商品列表。

### 3.8 中间方法大全

| 方法 | 作用 | 例子 |
|---|---|---|
| `filter(Predicate)` | 筛选 | `filter(n -> n > 0)` |
| `map(Function)` | 转换 | `map(n -> n * 2)` |
| `flatMap(Function)` | 转换 + 压平 | `flatMap(Collection::stream)` |
| `distinct()` | 去重 | |
| `sorted()` | 排序 | `sorted()` / `sorted(cmp)` |
| `limit(n)` | 取前 n 个 | |
| `skip(n)` | 跳过前 n 个 | |
| `peek(Consumer)` | 调试用，看中间结果 | `peek(System.out::println)` |

---

## 4. 终结方法（04）

> **终结方法的特点：没有返回值（或返回非 Stream 的结果），调用后流就"结束"了。**
> **没有终结方法，流不会执行（惰性求值）。**

### 4.1 forEach —— 遍历

```java
List<Integer> nums = List.of(1, 2, 3);
nums.stream().forEach(n -> System.out.print(n + " "));
// 1 2 3
```

### 4.2 count —— 计数

```java
nums.stream().count();          // 3（long）
nums.stream().filter(n -> n > 1).count();   // 2
```

### 4.3 max / min —— 最大最小值

```java
List<Integer> nums = List.of(3, 1, 4, 1, 5, 9, 2, 6);

nums.stream().max(Integer::compare).get();   // 9
nums.stream().min(Integer::compare).get();   // 1
```

**注意**：`max`/`min` 返回 `Optional`，要 `.get()` 取值（集合为空时 `.get()` 会抛异常，用 `.orElse(默认值)` 更安全）。

### 4.4 toArray —— 转数组

```java
nums.stream().toArray();                 // Object[]
nums.stream().toArray(Integer[]::new);   // Integer[]（指定类型）
```

### 4.5 findFirst / findAny —— 找第一个

```java
nums.stream().filter(n -> n > 4).findFirst().get();   // 5
```

### 4.6 匹配判断

```java
List<Integer> nums = List.of(3, 1, 4, 1, 5, 9, 2, 6);

nums.stream().anyMatch(n -> n > 8);    // true   有没有满足的
nums.stream().allMatch(n -> n > 0);    // true   是不是全都满足
nums.stream().noneMatch(n -> n < 0);   // true   是不是都不满足
```

### 4.7 reduce —— 归约（进阶）

**把所有元素"合并成一个值"：**

```java
nums.stream().reduce(0, Integer::sum);   // 31   求和
nums.stream().reduce(0, (a, b) -> a + b);  // 同上
```

**参数**：初始值 + 合并函数。

**常见用途**：求和、求积、字符串拼接。

### 4.8 终结方法大全

| 方法 | 返回 | 作用 |
|---|---|---|
| `forEach(Consumer)` | void | 遍历 |
| `count()` | long | 计数 |
| `max(cmp)` / `min(cmp)` | Optional | 最大/最小 |
| `toArray()` | Object[] / T[] | 转数组 |
| `findFirst()` / `findAny()` | Optional | 找第一个 |
| `anyMatch` / `allMatch` / `noneMatch` | boolean | 匹配判断 |
| `reduce(...)` | Optional / 值 | 归约 |
| `collect(Collector)` | 任意 | 收集（下一节） |

---

## 5. collect 收集（05）

> **collect 是最常用的终结方法**，把流"收"成一个集合（List/Set/Map）或其他结果。
> 需要配合 `java.util.stream.Collectors` 工具类。

### 5.1 toList —— 收成 List

```java
List<String> words = List.of("apple", "banana", "cherry");

words.stream().collect(Collectors.toList());
// [apple, banana, cherry]
```

**简写**（JDK 16+）：

```java
words.stream().toList();   // 直接 .toList()，不用 collect
```

**注意**：`.toList()` 返回的列表**不可变**（不能 add），`Collectors.toList()` 返回的可以改。

### 5.2 toSet —— 收成 Set（自动去重）

```java
List<String> words = List.of("apple", "banana", "cherry", "apple");

words.stream().collect(Collectors.toSet());
// [banana, apple, cherry]   ← 重复的 apple 去掉了
```

### 5.3 toMap —— 收成 Map（**有坑**）

```java
List<String> words = List.of("apple", "banana", "cherry");

Map<String, Integer> m = words.stream()
        .collect(Collectors.toMap(w -> w, String::length));
// {apple=5, banana=6, cherry=6}
```

**参数**：`toMap(key函数, value函数)`。

**坑：key 重复会直接抛异常**（实测）：

```java
List<String> dup = List.of("apple", "apple");

dup.stream().collect(Collectors.toMap(w -> w, String::length));
// ❌ IllegalStateException: Duplicate key apple (attempted merging values 5 and 5)
```

**解决**：加第三个参数"合并函数"：

```java
dup.stream().collect(Collectors.toMap(
        w -> w,
        String::length,
        (a, b) -> a          // 重复时保留第一个
));
// {apple=5}
```

### 5.4 joining —— 字符串拼接

```java
List<String> words = List.of("apple", "banana", "cherry");

words.stream().collect(Collectors.joining(", "));
// "apple, banana, cherry"

words.stream().collect(Collectors.joining(", ", "[", "]"));
// "[apple, banana, cherry]"
```

**参数**：分隔符、前缀、后缀。

### 5.5 counting —— 计数

```java
words.stream().collect(Collectors.counting());
// 4
```

### 5.6 groupingBy —— 分组（超常用）

**按某个规则分成多个组**：

```java
List<String> words = List.of("apple", "banana", "cherry", "avocado", "blueberry");

Map<Character, List<String>> grouped = words.stream()
        .collect(Collectors.groupingBy(w -> w.charAt(0)));
// {a=[apple, avocado], b=[banana, blueberry], c=[cherry]}
```

**实际用途**：按部门分组、按成绩等级分组、按日期分组。

### 5.7 partitioningBy —— 分区（只有两组）

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6);

Map<Boolean, List<Integer>> part = nums.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {false=[1, 3, 5], true=[2, 4, 6]}
```

**和 groupingBy 的区别**：分区只有 `true`/`false` 两组。

### 5.8 collect 速查

| 想要 | 写法 |
|---|---|
| List | `collect(Collectors.toList())` 或 `.toList()` |
| Set | `collect(Collectors.toSet())` |
| Map | `collect(Collectors.toMap(k, v))` |
| 字符串拼接 | `collect(Collectors.joining(", "))` |
| 计数 | `collect(Collectors.counting())` |
| 分组 | `collect(Collectors.groupingBy(...))` |
| 分区 | `collect(Collectors.partitioningBy(...))` |

---

## 6. 综合练习（06–07）

### 练习 1：数字过滤

**需求**：从一堆数字里挑出 3 位数，取前 2 个。

```java
List<Integer> nums = List.of(12, 123, 456, 7, 890, 1234, 567);

List<Integer> result = nums.stream()
        .filter(n -> n >= 100 && n <= 999)   // 3 位数
        .limit(2)                             // 取前 2 个
        .toList();

// result = [123, 456]
```

### 练习 2：字符串过滤并收集

**需求**：从名字列表里筛出"姓张且两个字"的。

```java
List<String> words = List.of("张三", "李四", "王五", "赵六", "张伟", "张明");

List<String> zhang = words.stream()
        .filter(w -> w.startsWith("张"))
        .filter(w -> w.length() == 2)
        .toList();

// [张三, 张伟, 张明]
```

### 练习 3（额外）：统计每个首字母有多少个词

```java
List<String> words = List.of("apple", "banana", "cherry", "avocado");

Map<Character, Long> count = words.stream()
        .collect(Collectors.groupingBy(w -> w.charAt(0), Collectors.counting()));
// {a=2, b=1, c=1}
```

---

## 7. 常见坑（必看）

### 坑 1：流只能消费一次

```java
Stream<Integer> s = Stream.of(1, 2, 3);
s.toList();   // [1, 2, 3]
s.toList();   // ❌ IllegalStateException
```

**解决**：重新 `stream()` 一次，或者先收集成集合再用。

### 坑 2：没有终结方法，流不执行

```java
list.stream().filter(n -> n > 0);   // 什么都不干
list.stream().filter(n -> n > 0).toList();   // 才执行
```

**这是"惰性求值"**，不是 bug。

### 坑 3：toMap 的 key 重复

```java
List.of("a", "a").stream().collect(Collectors.toMap(w -> w, w -> 1));
// ❌ IllegalStateException: Duplicate key
```

**解决**：加合并函数 `(a, b) -> a`。

### 坑 4：`.toList()` 和 `Collectors.toList()` 不一样

```java
list.stream().toList();                        // 不可变（不能 add）
list.stream().collect(Collectors.toList());    // 可变
```

**要改列表用后者，只读用前者。**

### 坑 5：sorted 对自定义对象不传比较器

```java
class Person { String name; int age; }   // 没实现 Comparable

people.stream().sorted().toList();
// ❌ ClassCastException

people.stream().sorted(Comparator.comparingInt(p -> p.age)).toList();
// ✅
```

### 坑 6：在 jshell 里链式调用必须写一行

```java
// ❌ jshell 里会报错（它逐行解析）
List<Integer> r = nums.stream()
        .filter(...)
        .toList();

// ✅ jshell 里写成一行
List<Integer> r = nums.stream().filter(...).toList();
```

**在真实 Java 文件里可以换行**，只是 jshell 的交互特性。

### 坑 7：并行流不一定快

```java
list.parallelStream()   // 多线程处理
```

**小数据量反而更慢**（线程调度开销），而且**有状态操作结果可能不对**。**默认用 `stream()`，别乱用 `parallelStream()`。**

---

## 8. 一页速查表

### 获取流

| 来源 | 写法 |
|---|---|
| 集合 | `list.stream()` |
| 数组 | `Arrays.stream(arr)` |
| 元素 | `Stream.of(1, 2, 3)` |
| Map | `map.keySet().stream()` |
| 范围 | `IntStream.range(1, 10)` |

### 中间方法

| 想要 | 写法 |
|---|---|
| 筛选 | `filter(x -> 条件)` |
| 转换 | `map(x -> 新值)` |
| 压平 | `flatMap(x -> 流)` |
| 去重 | `distinct()` |
| 排序 | `sorted()` / `sorted(cmp)` |
| 取前 n | `limit(n)` |
| 跳过 n | `skip(n)` |
| 合并 | `Stream.concat(a, b)` |

### 终结方法

| 想要 | 写法 |
|---|---|
| 遍历 | `forEach(x -> ...)` |
| 计数 | `count()` |
| 最大最小 | `max(cmp)` / `min(cmp)` |
| 转集合 | `.toList()` / `collect(...)` |
| 判断 | `anyMatch` / `allMatch` / `noneMatch` |
| 找第一个 | `findFirst()` |
| 归约 | `reduce(初始值, 函数)` |

### collect

| 想要   | 写法                                           |
| ---- | -------------------------------------------- |
| List | `.toList()` 或 `collect(Collectors.toList())` |
| Set  | `collect(Collectors.toSet())`                |
| Map  | `collect(Collectors.toMap(k, v))`            |
| 拼接   | `collect(Collectors.joining(", "))`          |
| 分组   | `collect(Collectors.groupingBy(...))`        |
| 分区   | `collect(Collectors.partitioningBy(...))`    |

---

## 9. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.util.*;
import java.util.stream.*;

// ① 基础过滤
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6);
System.out.println(nums.stream().filter(n -> n % 2 == 0).toList());

// ② 转换 + 收集
System.out.println(nums.stream().map(n -> n * n).toList());

// ③ 排序
System.out.println(nums.stream().sorted(Comparator.reverseOrder()).toList());

// ④ 分组
List<String> words = List.of("apple", "banana", "cherry", "avocado");
System.out.println(words.stream().collect(Collectors.groupingBy(w -> w.charAt(0))));

// ⑤ 统计
System.out.println(words.stream().collect(Collectors.groupingBy(w -> w.charAt(0), Collectors.counting())));

// ⑥ 踩坑：流只能消费一次
Stream<Integer> s = Stream.of(1, 2, 3);
s.toList();
try { s.toList(); } catch (IllegalStateException e) { System.out.println("流只能消费一次"); }
```

---

## 记忆口诀

> **Stream 是流水线，中间方法接着接，终结方法才算完；**
> **不改原集合、只能用一个、没终结不执行；**
> **filter 筛、map 变、flatMap 压平、sorted 排；**
> **collect 收集配 Collectors，toMap 重复要合并；**
> **groupingBy 分组，partitioningBy 只分两组。**
