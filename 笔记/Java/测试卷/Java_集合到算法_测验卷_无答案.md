# Java 阶段性测验卷

> 覆盖范围：包装类 / BigInteger / BigDecimal · 集合框架（List / Set / Map） · 泛型与通配符 · 不可变集合 · Stream 流 · 方法引用 · 正则表达式 · 常见查找与排序算法。
> **不含异常处理、File / IO 流、Docker、英语与金融内容；卷内不含答案与答题卡。**

## 考生须知

1. 本卷按内容分 4 个模块，每个模块按「单选 → 多选 → 判断 → 代码分析 → 填空/简答」排列。
2. **多选题错选、少选、多选均不得分。**
3. 判断题：正确画 √，错误画 ×。
4. 代码分析题写出**实际运行结果**；有异常就写异常类名（如 `ArithmeticException`）。
5. 填空题按「①…②…」顺序作答；简答题写出关键点即可。
6. 建议分次完成，每次 45 分钟做一个模块。

---

# 模块一　包装类 / BigInteger / BigDecimal

## 一、单选题（6 小题，每题 2 分，共 12 分）

**1.** Java 的 8 个包装类中，与基本类型 `char` 对应的是（　　）

- A. `Char`
- B. `Character`
- C. `Characterer`
- D. `String`

**2.** `Integer a = 127, b = 127; System.out.println(a == b);` 输出 `true`，原因是（　　）

- A. `Integer` 重写了 `==`
- B. 127 在 `Integer` 的对象缓存范围（-128~127）内，`Integer.valueOf(127)` 返回同一个缓存对象
- C. 编译器把两个 `Integer` 优化成了同一个变量
- D. `Integer` 是不可变类，所以任何值比较都是 `true`

**3.** 下列代码会抛出哪种异常？（　　）

```java
Integer n = null;
System.out.println(n == 1);
```

- A. `NumberFormatException`
- B. `NullPointerException`
- C. `IllegalArgumentException`
- D. 不会抛异常，输出 `false`

**4.** 比较两个包装类对象的**值**，正确且可靠的做法是（　　）

- A. 用 `==`，因为包装类会自动拆箱
- B. 用 `equals()` 或 `compareTo()`
- C. 用 `hashCode()` 比较
- D. 用 `toString()` 比较

**5.** 下列创建 `BigDecimal` 的写法中，正确的是（　　）

- A. `new BigDecimal(0.1)`
- B. `new BigDecimal("0.1")`
- C. `BigDecimal.valueOf(0.1)`
- D. B 和 C 都可以；A 得到的是 `0.1000000000000000055511151231257827...`

**6.** `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` 的结果是（　　）

- A. `true`，因为数值相等
- B. `false`，因为 `equals` 同时比较数值和 scale（精度）
- C. 抛 `ArithmeticException`
- D. 编译错误

## 二、多选题（3 小题，每题 3 分，共 9 分）

**7.** 下列关于包装类缓存的说法，正确的有（　　）

- A. `Byte` 全部值都缓存
- B. `Integer` 缓存 -128~127
- C. `Double` 和 `Float` 没有缓存
- D. `Boolean` 缓存 `true` 和 `false` 两个对象
- E. `Integer` 缓存 -128~128

**8.** 关于 `BigDecimal` 的说法，正确的有（　　）

- A. 所有运算（`add`/`subtract`/`multiply`/`divide`）都返回新对象，不改原对象
- B. 除法必须指定精度和舍入模式，否则遇到无限小数会抛 `ArithmeticException`
- C. `compareTo()` 返回 0 表示数值相等，`equals()` 还要看 scale
- D. `setScale(2, RoundingMode.HALF_UP)` 是四舍五入保留两位
- E. `BigDecimal` 能精确表示 `0.1`，`double` 不能，这也是它存在的理由之一

**9.** `Arrays.asList(...)` 和 `List.of(...)` 的区别，说法正确的有（　　）

- A. `Arrays.asList` 返回固定长度列表，`add`/`remove` 抛 `UnsupportedOperationException`，但 `set` 可以改
- B. `Arrays.asList` 与原数组共享同一份数据
- C. `List.of` 返回不可变列表，`set` 也抛 `UnsupportedOperationException`
- D. `List.of(null)` 抛 `NullPointerException`
- E. `List.of("a","b").add("c")` 会抛 `UnsupportedOperationException`


## 三、判断题（6 小题，每题 1 分，共 6 分）

**10.** `Integer` 和 `int` 可以互相赋值，是因为自动装箱/拆箱。（　　）

**11.** `Integer.valueOf(200) == Integer.valueOf(200)` 的结果是 `true`。（　　）

**12.** `Double d1 = 1.0; Double d2 = 1.0; d1 == d2` 为 `false`，因为 `Double` 没有缓存。（　　）

**13.** `Integer n = null; int x = n;` 会抛 `NullPointerException`。（　　）

**14.** `BigDecimal` 的 `divide` 可以不写精度，编译器会自动补默认精度。（　　）

**15.** `new BigDecimal("1E-7").toPlainString()` 输出 `"0.0000001"`，而 `toString()` 输出 `"1E-7"`。（　　）

## 四、代码分析题（1 小题，每题 4 分，共 4 分）

**16.** 写出输出结果。

```java
Integer a = 100, b = 100;
Integer c = 300, d = 300;
System.out.println(a == b);
System.out.println(c == d);
System.out.println(c.equals(d));
System.out.println(a + 1);
```

## 五、填空题 / 简答题（4 小题，共 8 分）

**17.** 包装类里，`int` 对应 `________`，`char` 对应 `________`，`boolean` 对应 `________`。

**18.** 已知 `Integer m = 128, n = 128;`，`m == n` 的结果是 `________`，`m.equals(n)` 的结果是 `________`。

**19.** `BigDecimal` 做除法时，`________` 必须给出舍入模式；比较数值大小应使用 `________` 方法而不是 `equals`。

**20.** 简答：为什么比较包装类**永远不该用 `==`**？请结合「缓存」和「自动拆箱」两点说明。

---

# 模块二　集合框架

## 一、单选题（21 小题，每题 2 分，共 42 分）

**21.** 下列**不是** `Collection` 的子接口的是（　　）

- A. `List`
- B. `Set`
- C. `Map`
- D. `Queue`

**22.** `ArrayList` 的底层结构和性能特点是（　　）

- A. 双向链表，随机访问 O(1)
- B. 数组，随机访问 O(1)，中间插入/删除要搬元素
- C. 数组，随机访问 O(n)
- D. 哈希表，查找 O(1)

**23.** `LinkedList` 在头部/中间频繁插入删除优于 `ArrayList` 的原因是（　　）

- A. 链表节点本身更省内存
- B. 只需改动指针，不需要像数组那样整体搬移元素
- C. 链表自带排序
- D. 链表支持下标访问

**24.** 有 `List<Integer> list = new ArrayList<>(List.of(10, 20, 30));`，执行 `list.remove(1)` 后 `list` 是（　　）

- A. `[10, 30]`，删的是索引 1 的 20
- B. `[10, 20]`，删的是值 1
- C. `[10, 30]`，删的是值 1
- D. 抛异常

**25.** `List<String> sub = list.subList(0, 2);` 之后再改 `sub`，原 `list` 会（　　）

- A. 不受影响，`subList` 是独立拷贝
- B. 一起变化，`subList` 是视图
- C. 编译报错
- D. 抛 `UnsupportedOperationException`

**26.** 遍历集合时想安全删除元素，推荐的做法是（　　）

- A. 在增强 for 里直接 `list.remove(x)`
- B. 用 `Iterator` 并在 `next()` 之后调 `it.remove()`
- C. 用 `list.clear()` 再重新加
- D. 在普通 for 里正着删

**27.** 增强 for 循环本质上是（　　）

- A. 下标循环的语法糖
- B. 迭代器的语法糖
- C. 递归实现
- D. Stream 的语法糖

**28.** 关于 `ConcurrentModificationException`，说法正确的是（　　）

- A. 只要在遍历时用 `list.remove()`，就**一定**会抛
- B. 它不一定每次都抛，所以不能靠「我试过没报错」来判断安全
- C. 它是 `Error` 的子类
- D. 它只能通过 `catch` 修复，没有别的办法避免

**29.** `HashSet` 去重依赖的两道关卡是（　　）

- A. `==` 和 `equals`
- B. `hashCode` 和 `equals`
- C. `compareTo` 和 `equals`
- D. `toString` 和 `hashCode`

**30.** 既要保证**插入顺序**又要去重的集合，应选（　　）

- A. `HashSet`
- B. `LinkedHashSet`
- C. `TreeSet`
- D. `ArrayList`

**31.** `TreeSet` 判断「元素重复」的依据是（　　）

- A. `equals()` 返回 `true`
- B. `==` 为 `true`
- C. `compareTo` / `compare` 返回 `0`
- D. `hashCode` 相同

**32.** 一个没有实现 `Comparable` 的类，往 `TreeSet` 里添加元素时（　　）

- A. 加第一个就抛 `ClassCastException`
- B. 加第二个才抛 `ClassCastException`
- C. 永远不报错，只是无序
- D. 编译报错

**33.** 关于 `hashCode`，说法正确的是（　　）

- A. `hashCode` 返回的就是对象的内存地址
- B. `String` 的 `hashCode` 是按内容计算的
- C. `hashCode` 不同，`equals` 也可能为 `true`
- D. 重写 `equals` 不需要重写 `hashCode`

**34.** `HashMap` 的底层结构是（　　）

- A. 纯数组
- B. 数组 + 链表（链表过长且数组 ≥ 64 时转红黑树）
- C. 纯红黑树
- D. 双向链表

**35.** 关于 `HashMap` 的 `null`，说法正确的是（　　）

- A. 不允许 `null` 键，也不允许 `null` 值
- B. 允许一个 `null` 键，允许多个 `null` 值
- C. 允许任意多个 `null` 键
- D. 允许 `null` 键但只允许一个 `null` 值

**36.** `HashMap` 默认容量 16、负载因子 0.75，放第几个元素时触发扩容？（　　）

- A. 第 12 个
- B. 第 13 个
- C. 第 16 个
- D. 第 17 个

**37.** `LinkedHashMap` 比 `HashMap` 多的能力是（　　）

- A. 按键排序
- B. 保持插入顺序
- C. 线程安全
- D. 支持泛型

**38.** 关于 `TreeMap`，说法正确的是（　　）

- A. 允许 `null` 键
- B. 按值排序
- C. 按 key 排序，不允许 `null` 键
- D. 保持插入顺序

**39.** `map.put(k, v)` 的返回值是（　　）

- A. 新值 `v`
- B. `null` 永远
- C. 这个 key 原来的旧值；没有旧值时返回 `null`
- D. 布尔值，表示是否添加成功

**40.** 自定义对象作为 `HashMap` 的 key，只重写 `equals` 不重写 `hashCode`，`get` 可能返回（　　）

- A. 一定能查到
- B. `null`，因为两个「内容相同」的对象哈希不同、落到不同桶
- C. 抛 `ClassCastException`
- D. 抛 `NullPointerException`

**41.** 要同时遍历 Map 的 key 和 value，且数据量大时，效率更高的方式是（　　）

- A. `keySet()` 再逐个 `get()`
- B. `entrySet()`
- C. `values()` 再手动配 key
- D. 转成数组遍历

## 二、多选题（5 小题，每题 3 分，共 15 分）

**42.** 关于 `ArrayList` vs `LinkedList`，说法正确的有（　　）

- A. `ArrayList` 随机访问快
- B. 两者在中间插入元素都主要是 O(n)（数组要搬移、链表要遍历找位置）
- C. `LinkedList` 每个节点额外存前后指针，内存开销更大
- D. `ArrayList` 支持下标访问，`LinkedList` 用 `get(i)` 需要从头/尾遍历
- E. `Arrays.asList()` 返回的是可增删的标准 `ArrayList`

**43.** 遍历集合又需要删除元素，安全的写法有（　　）

- A. `removeIf(...)`
- B. `Iterator` 的 `it.remove()`
- C. 对 `List` 倒着用普通 for 按下标删
- D. 正着用普通 for 按下标删
- E. 在增强 for 里直接 `list.remove(x)`

**44.** 往 `HashSet` 里加自定义对象去重，必须重写（　　）

- A. `hashCode()`
- B. `equals()`
- C. `toString()`
- D. `compareTo()`
- E. `getClass()`

**45.** 关于 `TreeSet` / `TreeMap`，说法正确的有（　　）

- A. `compareTo` 返回 0 时，`TreeSet` 丢弃新元素
- B. `compareTo` 返回 0 时，`TreeMap` 用新 value 覆盖旧 value
- C. `TreeMap` 的 key 必须实现 `Comparable` 或传入 `Comparator`
- D. `TreeSet` 的排序规则只能写死一套，不能用 `Comparator`
- E. 红黑树的中序遍历能得到有序序列

**46.** 关于 `HashMap` 的遍历方式，说法正确的有（　　）

- A. `keySet` + `get` 每次都要二次查找，数据量大时慢
- B. `entrySet` 遍历时键和值从同一个节点取，不用二次查找
- C. `forEach((k, v) -> ...)` 本质上是 `entrySet` 遍历的语法糖
- D. 遍历 Map 想删元素，用 `removeIf` 或迭代器
- E. `keySet` 能连 value 一起直接拿到

## 三、判断题（10 小题，每题 1 分，共 10 分）

**47.** `ArrayList` 实现了 `List` 接口，`HashSet` 实现了 `Set` 接口。（　　）

**48.** `HashMap` 是 `Collection` 接口的实现类。（　　）

**49.** `LinkedHashSet` 的「保序」保的是插入顺序。（　　）

**50.** `TreeSet` 允许存放 `null`。（　　）

**51.** `List.of(...)` 返回的集合可以调用 `add`。（　　）

**52.** `List.of(...)` 返回的集合调用 `set` 也会抛 `UnsupportedOperationException`。（　　）

**53.** `HashSet` 的迭代顺序是插入顺序。（　　）

**54.** 通过 `Iterator` 的 `remove()` 删除元素，是推荐的做法。（　　）

**55.** `Map` 的 key 可以重复，value 不能重复。（　　）

**56.** 自定义对象作 key 时，`hashCode` 和 `equals` 必须成套重写。（　　）

## 四、代码分析题（4 小题，每题 4 分，共 16 分）

**57.** 写出输出结果。

```java
List<Integer> list = new ArrayList<>(List.of(10, 20, 30));
list.remove(1);
System.out.println(list);
List<String> s = new ArrayList<>(List.of("a", "b", "c"));
s.remove("b");
System.out.println(s);
```

**58.** 写出输出结果。

```java
class Person {
    int age;
    Person(int age) { this.age = age; }
}
Set<Person> set = new HashSet<>();
set.add(new Person(18));
set.add(new Person(18));
System.out.println(set.size());
```

**59.** 写出输出结果。

```java
Map<String, Integer> map = new HashMap<>();
System.out.println(map.put("a", 1));
System.out.println(map.put("a", 2));
System.out.println(map.get("a"));
System.out.println(map.size());
```

**60.** 写出输出结果。

```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
map.put("a", 3);
System.out.println(map.size());
System.out.println(map.get("a"));
Map<String, Integer> nm = new HashMap<>();
nm.put(null, 1);
nm.put(null, 2);
System.out.println(nm.size());
System.out.println(nm.get(null));
```

## 五、填空题 / 简答题（8 小题，共 8 分）

**61.** `HashMap` 默认容量是 `________`，默认负载因子是 `________`，扩容倍数是 `________`。

**62.** 链表转红黑树的要求是链表长度 ≥ `________` **且** 数组长度 ≥ `________`；红黑树节点数降到 ≤ `________` 时退回链表。

**63.** 集合契约：`equals` 返回 `true` 时，`hashCode` 必须 `________`；反过来不要求。

**64.** `HashSet` 的底层其实是包了一个 `________`（填具体类名）。

**65.** 遍历 `List` 时安全删除，除了迭代器的 `remove()`，还可以用 `________` 方法配合 lambda。

**66.** `TreeSet` 的元素排序规则写在 `________` 里；`TreeMap` 的 key 必须实现 `________` 或传入 `________`。

**67.** `Collections.shuffle(list)` 的作用是 `________`，它只接受 `________` 类型的集合。

**68.** 简答：为什么「往 HashMap 里放自定义对象当 key，`get` 却拿到 `null`」？请从 hashCode、equals、桶 三个词讲清楚。

---

# 模块三　泛型 / 不可变集合 / Stream / 方法引用

## 一、单选题（11 小题，每题 2 分，共 22 分）

**69.** 泛型最大的价值是（　　）

- A. 让程序跑得更快
- B. 把运行期才暴露的类型错误提前到编译期
- C. 让集合能自动排序
- D. 减少内存占用

**70.** 代码 `List<int> list = new ArrayList<>();` 的问题是（　　）

- A. `int` 不能作为泛型参数，应该用 `Integer`
- B. `ArrayList` 不支持泛型
- C. 缺少 `import`
- D. 没问题

**71.** 已知 `List<Integer> ints`，执行 `List<Number> nums = ints;` 的结果是（　　）

- A. 编译通过，`Integer` 是 `Number` 的子类
- B. 编译错误，因为泛型不变（`List<Integer>` 不是 `List<Number>` 的子类）
- C. 运行时抛 `ClassCastException`
- D. 编译通过但运行期丢失泛型

**72.** `List<? extends Number>` 能做什么？（　　）

- A. 能读，读出可当 `Number` 用
- B. 能写，可以 `add` 任何 `Number` 的子类
- C. 能读也能写
- D. 不能读也不能写

**73.** 下面哪个写法**无法通过编译**？（　　）

- A. `list instanceof List<String>`
- B. `list instanceof List`
- C. `new ArrayList<String>()`
- D. `new ArrayList<>(List.of(1, 2, 3))`

**74.** Stream 的三条铁律**不包括**（　　）

- A. 不修改原数据
- B. 只能消费一次
- C. 惰性求值
- D. 自动并行执行

**75.** 想把 `List<List<Integer>>` 拍平成 `List<Integer>`，应该用（　　）

- A. `map`
- B. `flatMap`
- C. `filter`
- D. `distinct`

**76.** 下列全部属于**中间方法**的一组是（　　）

- A. `filter`、`map`、`sorted`
- B. `filter`、`count`、`collect`
- C. `forEach`、`map`、`limit`
- D. `count`、`collect`、`reduce`

**77.** `stream.toList()` 与 `stream.collect(Collectors.toList())` 的关键区别是（　　）

- A. 前者不可变，后者可变
- B. 前者可变，后者不可变
- C. 两者完全一样
- D. 前者只能用于字符串流

**78.** 下列能写成方法引用的是（　　）

- A. `s -> s.trim()`
- B. `s -> s.trim().length()`
- C. `s -> s.length() > 5`
- D. `(a, b) -> a + b`

**79.** `String::length` 等价于（　　）

- A. `() -> "abc".length()`
- B. `(s) -> s.length()`
- C. `(a, b) -> a.length(b)`
- D. `(s) -> String.length(s)`

## 二、多选题（4 小题，每题 3 分，共 12 分）

**80.** 关于泛型通配符，说法正确的有（　　）

- A. `? extends Number` 可读不可写
- B. `? super Integer` 可写，但读出来只能当 `Object`
- C. `List<? extends Number>` 可以接收 `List<Integer>` 和 `List<Double>`
- D. 泛型在运行时会被擦除
- E. `List<Integer>` 是 `List<Number>` 的子类

**81.** 关于方法引用，说法正确的有（　　）

- A. `forEach(System.out::println)` 可用
- B. `forEach(x -> System.out.println(x))` 与 A 等价
- C. `map(String::trim)` 可用（`trim` 没有额外参数）
- D. `map(s -> s.trim().toUpperCase())` 能写成 `String::trim`
- E. `类名::new` 可以引用构造方法

**82.** 关于不可变集合，说法正确的有（　　）

- A. `List.of` 返回的集合任何写操作都抛 `UnsupportedOperationException`
- B. `Set.of("a","a")` 创建时就抛异常，不会静默去重
- C. `Collections.unmodifiableList` 是视图，原集合改了它也变
- D. `List.copyOf` 是真拷贝，原集合改了它不变
- E. 不可变集合连元素内部状态也锁死了

**83.** 关于 Stream，说法正确的有（　　）

- A. 没有终结方法，中间操作一个都不会执行
- B. 同一个 Stream 对象第二次使用抛 `IllegalStateException`
- C. `collect(Collectors.toList())` 返回 `ArrayList`，可继续改
- D. `Collectors.toMap` 遇重复 key 默认抛异常，可加合并函数解决
- E. `filter`、`sorted` 会修改原集合

## 三、判断题（8 小题，每题 1 分，共 8 分）

**84.** 泛型只在编译期有效，运行时会被擦除。（　　）

**85.** 可以创建泛型数组 `new T[10]`。（　　）

**86.** `List<Integer>` 可以直接赋值给 `List<Number>`。（　　）

**87.** `? extends` 适合「往外读」的场景，`? super` 适合「往里写」的场景。（　　）

**88.** Stream 的 `filter` 不调用终结方法也会立刻执行。（　　）

**89.** 一条 Stream 消费完可以重复使用。（　　）

**90.** `anyMatch` 是终结方法，`filter` 是中间方法。（　　）

**91.** `stream.toList()` 返回的列表是不可变的。（　　）

## 四、代码分析题（3 小题，每题 4 分，共 12 分）

**92.** 写出输出结果。

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4, 5));
List<Integer> result = list.stream()
        .filter(x -> x > 2)
        .map(x -> x * 2)
        .collect(Collectors.toList());
System.out.println(result);
```

**93.** 写出输出结果（或异常）。

```java
List<String> list = List.of("a", "b", "c");
List<String> copy = new ArrayList<>(list);
copy.add("d");
System.out.println(copy);
try {
    list.add("d");
} catch (Exception e) {
    System.out.println(e.getClass().getSimpleName());
}
```

**94.** 写出输出结果。

```java
System.out.println(Stream.of(1, 2, 3).map(String::valueOf).collect(Collectors.joining(",")));
System.out.println(List.of(1, 2, 3, 4).stream().map(x -> x * 2).filter(x -> x > 4).count());
```

## 五、填空题 / 简答题（1 小题，共 2 分）

**95.** 为什么「写了一条 `stream.filter(...).map(...)` 但一句输出都没有」？用「惰性求值」「终结方法」两个词回答。

---

# 模块四　正则表达式 / 常见算法

## 一、单选题（9 小题，每题 2 分，共 18 分）

**96.** `"a123b".matches("\\d+")` 的结果是（　　）

- A. `true`，因为包含数字
- B. `false`，因为 `matches` 要求整串匹配
- C. 编译错误
- D. 抛 `PatternSyntaxException`

**97.** 想在 `"hello123"` 里查找数字子串 `123`，应该用（　　）

- A. `matches("\\d+")`
- B. `Pattern.compile("\\d+").matcher(s).find()`
- C. `split("\\d+")`
- D. `replace("\\d+", "")`

**98.** 在 Java 字符串里写正则 `\d`，正确写法是（　　）

- A. `"\d"`
- B. `"\\d"`
- C. `"/d"`
- D. `"[d]"`

**99.** `String.replace()` 与 `String.replaceAll()` 的区别是（　　）

- A. `replace` 是正则替换，`replaceAll` 是纯文本替换
- B. `replace` 是纯文本替换，`replaceAll` 的参数是正则
- C. 两者完全一样
- D. `replaceAll` 只能替换一次

**100.** `"a.b.c".split("\\.")` 的结果是（　　）

- A. `["a", "b", "c"]`
- B. `[]`
- C. `["a.b.c"]`
- D. 抛异常

**101.** 正则 `(a)(b)` 匹配 `"ab"` 时，`group(0)` / `group(1)` 分别是（　　）

- A. `"a"` / `"b"`
- B. `"ab"` / `"a"`
- C. `"ab"` / `"ab"`
- D. `"b"` / `"a"`

**102.** 二分查找的前提是（　　）

- A. 数组元素都是数字
- B. 数组必须已经有序
- C. 数组长度必须是 2 的幂
- D. 数组不能有重复元素

**103.** 二分查找里 `int mid = (lo + hi) / 2;` 在超大数组下可能（　　）

- A. 变慢
- B. 溢出成负数
- C. 抛 `IndexOutOfBoundsException`
- D. 找不到最后一个元素

**104.** `Arrays.binarySearch(有序数组, 不存在的值)` 的返回值是（　　）

- A. 固定 `-1`
- B. 负数，值为 `-(插入点) - 1`
- C. `0`
- D. 抛 `NoSuchElementException`

## 二、多选题（2 小题，每题 3 分，共 6 分）

**105.** 关于二分查找，说法正确的有（　　）

- A. 对无序数组使用会「静默查不到」，不会报错
- B. `while (lo <= hi)` 写成 `while (lo < hi)` 可能漏掉最后一个元素
- C. `hi = mid` 而不是 `mid - 1` 可能导致死循环
- D. 用 `mid = lo + (hi - lo) / 2` 可避免溢出
- E. `Arrays.binarySearch` 会先自动排序再查

**106.** 关于递归，说法正确的有（　　）

- A. 必须有终止条件（出口）
- B. 没有出口会抛 `StackOverflowError`
- C. `StackOverflowError` 属于 `Error`，`catch(Exception)` 抓不住
- D. 递归一定能用循环完全替代，且性能更好
- E. 写递归时可以先写「处理一个」的逻辑，再问「如果是子集合呢」

## 三、判断题（2 小题，每题 1 分，共 2 分）

**107.** `"123".matches("\\d+")` 返回 `true`。（　　）

**108.** `"a.b"` 用 `replaceAll(".", "-")` 的结果是 `"a-b"`。（　　）

## 四、代码分析题（2 小题，每题 4 分，共 8 分）

**109.** 写出输出结果。

```java
System.out.println("hello123".matches("\\d+"));
Matcher m = Pattern.compile("\\d+").matcher("hello123");
while (m.find()) {
    System.out.println(m.group());
}
System.out.println(Arrays.toString("a.b.c".split(".")));
```

**110.** 写出输出结果。

```java
int[] a = {7, 23, 79, 81, 103, 127, 131, 147};
int pos = Arrays.binarySearch(a, 131);
System.out.println(pos);
int[] sorted = {1, 3, 5, 9};
System.out.println(Arrays.binarySearch(sorted, 7));
```

---


## 分值统计

| 模块 | 题量 | 分值 |
|:---|:---:|:---:|
| 包装类 / BigInteger / BigDecimal | 20 | 39 |
| 集合框架 | 48 | 91 |
| 泛型 / 不可变集合 / Stream / 方法引用 | 27 | 56 |
| 正则表达式 / 常见算法 | 15 | 34 |
| **合计** | **110** | **220** |
