# Java 综合测试卷（阶段二 · 从集合到 IO）

> 出卷依据：`日记/2026/2026-10-29 ~ 2026-11-23` 共 20 篇日记 + `笔记/` 下全部 Java / Docker / 英语教程
> 本卷只考你日记和教程里**明确出现过**的内容；标 `【拓展】` 的题来自教程的补充小节，不计入核心掌握度。
> **共 7 个模块、195 题、满分 318 分（必做 298 分 + 选做英语 20 分）；卷内不含答案。**

---

> ## 📊 本次批改结果（王老师批注版）
>
> | 模块 | 得分 | 满分 | 掌握度 |
> |:---|:---:|:---:|:---:|
> | 一、包装类 / BigInteger / BigDecimal | 16 | 40 | 40% |
> | 二、集合框架 | 46 | 98 | 47% |
> | 三、泛型 / 不可变集合 / Stream / 方法引用 | 29 | 54 | 54% |
> | 四、异常 / File / IO / Properties | 43 | 94 | 46% |
> | 五、正则 / 时间 API / 算法 | 19 | 44 | 43% |
> | 六、Docker | 18 | 30 | 60% |
> | 七、英语（选做） | 8 | 30 | 27% |
> | **七模块合计** | **179** | **390** | **45.9%** |
> | **必做（一~六）** | **171** | **360** | **47.5%** |
>
> 全错 69 道，部分给分 3 道（Q94、Q96、Q136）。每道错题旁边都写了 `❌ 错因`，按序号对应看即可。

## 考生须知

1. 本卷分七大模块，每个模块按「单选 → 多选 → 判断 → 代码分析 → 填空/简答」排列。
2. **多选题错选、少选、多选均不得分。**
3. 判断题：正确画 √，错误画 ×。
4. 代码分析题写出**实际运行结果**；有异常就写异常类名（如 `ArithmeticException`）。
5. 填空题按「①…②…」顺序作答；简答题写出关键点即可。
6. 建议每次做 45 分钟、一次做一个模块，不用一口气做完。
7. 做完把答案按题号发我（或填卷末答题卡），我逐题批改并给出掌握度评估。

---

# 模块一　包装类 / BigInteger / BigDecimal（39 分）

> 出自：`2026-11-08` 之前的 `包装类教程.md`、`BigInteger与BigDecimal教程.md`

## 一、单选题（6 小题，每题 2 分，共 12 分）

**1.** Java 的 8 个包装类中，与基本类型 `char` 对应的是（　B　）

- A. `Char`
- B. `Character`
- C. `Characterer`
- D. `String`

**2.** `Integer a = 127, b = 127; System.out.println(a == b);` 输出 `true`，原因是（　C　）

- A. `Integer` 重写了 `==`
- B. 127 在 `Integer` 的对象缓存范围（-128~127）内，`Integer.valueOf(127)` 返回同一个缓存对象
- C. 编译器把两个 `Integer` 优化成了同一个变量
- D. `Integer` 是不可变类，所以任何值比较都是 `true`

> ❌ 错因：你选了「编译器把两个变量合并」。实际是缓存：`Integer a=127` 编译后是 `Integer.valueOf(127)`，-128~127 命中 Integer 缓存返回同一个对象，所以 `==` 为 true。

**3.** 下列代码会抛出哪种异常？（　B　）

```java
Integer n = null;
System.out.println(n == 1);
```

- A. `NumberFormatException`
- B. `NullPointerException`
- C. `IllegalArgumentException`
- D. 不会抛异常，输出 `false`

**4.** 比较两个包装类对象的**值**，正确且可靠的做法是（　B　）

- A. 用 `==`，因为包装类会自动拆箱
- B. 用 `equals()` 或 `compareTo()`
- C. 用 `hashCode()` 比较
- D. 用 `toString()` 比较

**5.** 下列创建 `BigDecimal` 的写法中，正确的是（　C　）

- A. `new BigDecimal(0.1)`
- B. `new BigDecimal("0.1")`
- C. `BigDecimal.valueOf(0.1)`
- D. B 和 C 都可以；A 得到的是 `0.1000000000000000055511151231257827...`

> ❌ 错因：B 和 C 都能正确构造，所以「只有 C 对」的选项不成立，答案是 D。`new BigDecimal(0.1)` 得到的是 double 0.1 的二进制近似值那一长串，必须用字符串构造或 `BigDecimal.valueOf()`。

**6.** `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` 的结果是（　B　）

- A. `true`，因为数值相等
- B. `false`，因为 `equals` 同时比较数值和 scale（精度）
- C. 抛 `ArithmeticException`
- D. 编译错误

## 二、多选题（3 小题，每题 3 分，共 9 分）

**7.** 下列关于包装类缓存的说法，正确的有（　C\D　）

- A. `Byte` 全部值都缓存
- B. `Integer` 缓存 -128~127
- C. `Double` 和 `Float` 没有缓存
- D. `Boolean` 缓存 `true` 和 `false` 两个对象
- E. `Integer` 缓存 -128~128

> ❌ 错因：漏选 A、B：`Byte` 全部值都缓存、`Integer` 缓存 -128~127 都成立。E 错在 128 超出缓存范围。你选的 C、D 是对的。正确：A B C D。

**8.** 关于 `BigDecimal` 的说法，正确的有（　B\C\E　）

- A. 所有运算（`add`/`subtract`/`multiply`/`divide`）都返回新对象，不改原对象
- B. 除法必须指定精度和舍入模式，否则遇到无限小数会抛 `ArithmeticException`
- C. `compareTo()` 返回 0 表示数值相等，`equals()` 还要看 scale
- D. `setScale(2, RoundingMode.HALF_UP)` 是四舍五入保留两位
- E. `BigDecimal` 能精确表示 `0.1`，`double` 不能，这也是它存在的理由之一

> ❌ 错因：正确是 **A B C D E**。你选的 B、C、E 对，但漏了 A（运算都返回新对象、不改原对象）和 D（setScale 保留两位）。这题五个选项全对。

**9.** `Arrays.asList(...)` 和 `List.of(...)` 的区别，说法正确的有（　A\D　）

- A. `Arrays.asList` 返回固定长度列表，`add`/`remove` 抛 `UnsupportedOperationException`，但 `set` 可以改
- B. `Arrays.asList` 与原数组共享同一份数据
- C. `List.of` 返回不可变列表，`set` 也抛 `UnsupportedOperationException`
- D. `List.of(null)` 抛 `NullPointerException`
- E. `List.of("a","b").add("c")` 会抛 `UnsupportedOperationException`

> 说明：第 9 题对应 11-08 日记里「三样东西别混」那一节。

> ❌ 错因：漏选 B、C、E：`Arrays.asList` 与原数组共享数据；`List.of` 的 `set` 与 `add` 一样抛 `UnsupportedOperationException`；`List.of(null)` 抛 NPE。正确：A B C D E。

## 三、判断题（6 小题，每题 1 分，共 6 分）

**10.** `Integer` 和 `int` 可以互相赋值，是因为自动装箱/拆箱。（　T　）

**11.** `Integer.valueOf(200) == Integer.valueOf(200)` 的结果是 `true`。（　T　）

> ❌ 错因：`Integer.valueOf(200)` 超出 -128~127，两次调用各造一个新对象，`==` 比地址得 false。只有缓存范围内才复用同一对象。正确：×。

**12.** `Double d1 = 1.0; Double d2 = 1.0; d1 == d2` 为 `false`，因为 `Double` 没有缓存。（　T　）

**13.** `Integer n = null; int x = n;` 会抛 `NullPointerException`。（　F　）

> ❌ 错因：`Integer n = null; int x = n;` 是把 null 拆箱，会抛 `NullPointerException`，所以这句话本身是**正确**的（应打 √）。

**14.** `BigDecimal` 的 `divide` 可以不写精度，编译器会自动补默认精度。（　F　）

**15.** `new BigDecimal("1E-7").toPlainString()` 输出 `"0.0000001"`，而 `toString()` 输出 `"1E-7"`。（　T　）

## 四、代码分析题（1 小题，4 分）

**16.** 写出输出结果。

```java
Integer a = 100, b = 100;
Integer c = 300, d = 300;
System.out.println(a == b);
System.out.println(c == d);
System.out.println(c.equals(d));
System.out.println(a + 1);
```
False
False
True
101

> ❌ 错因：四个输出要小写：`true false true 101`。你写成了 False/False/True，严格按运行结果判会扣分。`a==b` 是缓存（100 在范围内）、`c==d` 是超出缓存（300）、`c.equals(d)` 比的是值、`a+1` 拆箱得 101。
## 五、填空题 / 简答题（4 小题，共 8 分）

**17.**（2 分）包装类里，`int` 对应 `____Integer____`，`char` 对应 `____Character____`，`boolean` 对应 `____Boolean____`。

**18.**（2 分）已知 `Integer m = 128, n = 128;`，`m == n` 的结果是 `___False_____`，`m.equals(n)` 的结果是 `____True____`。

**19.**（2 分）`BigDecimal` 做除法时，`divide(除数, 保留位数, ____精度____)` 必须给出舍入模式；比较数值大小应使用 `___compareTo()_____` 方法而不是 `equals`。

**20.**（2 分）简答：为什么比较包装类**永远不该用 `==`**？请结合「缓存」和「自动拆箱」两点说明。
不知道

> ❌ 错因：两点都没答到。①缓存：-128~127 的 Integer 是同一对象，`==` 碰巧为 true；超出范围（如 128）就 false，结果随机。②自动拆箱：`==` 一边包装类一边基本类型时会拆箱，包装类若是 null 直接抛 `NullPointerException`。所以值比较只能用 equals/compareTo。

---

# 模块二　集合框架（73 分）

> 出自：`2026-10-29`、`2026-10-31`、`2026-11-01`、`2026-11-02`、`2026-11-03`、`2026-11-08` 日记
> + `集合进阶教程`、`Set集合与树形数据结构教程`、`Map集合教程`、`不可变集合教程`

## 一、单选题（12 小题，每题 2 分，共 24 分）

**21.** 下列**不是** `Collection` 的子接口的是（　D　）

- A. `List`
- B. `Set`
- C. `Map`
- D. `Queue`

> ❌ 错因：不是 Collection 子接口的是 **C. Map**（双列集合，独立体系）。Queue 是 Collection 的子接口。

**22.** `ArrayList` 的底层结构和性能特点是（　B　）

- A. 双向链表，随机访问 O(1)
- B. 数组，随机访问 O(1)，中间插入/删除要搬元素
- C. 数组，随机访问 O(n)
- D. 哈希表，查找 O(1)

**23.** `LinkedList` 在头部/中间频繁插入删除优于 `ArrayList` 的原因是（　B　）

- A. 链表节点本身更省内存
- B. 只需改动指针，不需要像数组那样整体搬移元素
- C. 链表自带排序
- D. 链表支持下标访问

**24.** 有 `List<Integer> list = new ArrayList<>(List.of(10, 20, 30));`，执行 `list.remove(1)` 后 `list` 是（　A　）

- A. `[10, 30]`，删的是索引 1 的 20
- B. `[10, 20]`，删的是值 1
- C. `[10, 30]`，删的是值 1
- D. 抛异常

**25.** `List<String> sub = list.subList(0, 2);` 之后再改 `sub`，原 `list` 会（　A　）

- A. 不受影响，`subList` 是独立拷贝
- B. 一起变化，`subList` 是视图
- C. 编译报错
- D. 抛 `UnsupportedOperationException`

> ❌ 错因：`subList` 是**视图**不是拷贝：改子列表原 list 跟着变，改原 list 子列表也变。要独立副本得 `new ArrayList<>(list.subList(0,2))`。正确：B。

**26.** 遍历集合时想安全删除元素，推荐的做法是（　B　）

- A. 在增强 for 里直接 `list.remove(x)`
- B. 用 `Iterator` 并在 `next()` 之后调 `it.remove()`
- C. 用 `list.clear()` 再重新加
- D. 在普通 for 里正着删

**27.** 增强 for 循环本质上是（　B　）

- A. 下标循环的语法糖
- B. 迭代器的语法糖
- C. 递归实现
- D. Stream 的语法糖

**28.** 关于 `ConcurrentModificationException`，说法正确的是（　　）

- A. 只要在遍历时用 `list.remove()`，就**一定**会抛
- B. 它不一定每次都抛，所以不能靠「我试过没报错」来判断安全
- C. 它是 `Error` 的子类
- D. 它只能通过 `catch` 修复，没有别的办法避免

> ❌ 错因：CE 不是 Error，它是 `RuntimeException` 的子类。而且它不一定每次抛：删「倒数第二个」元素时 hasNext 已返回 false，版本检查没机会执行。正确：B。

**29.** `HashSet` 去重依赖的两道关卡是（　B　）

- A. `==` 和 `equals`
- B. `hashCode` 和 `equals`
- C. `compareTo` 和 `equals`
- D. `toString` 和 `hashCode`

**30.** 既要保证**插入顺序**又要去重的集合，应选（　C　）

- A. `HashSet`
- B. `LinkedHashSet`
- C. `TreeSet`
- D. `ArrayList`

> ❌ 错因：既要插入顺序又要去重应选 **B. LinkedHashSet**；TreeSet 是按比较规则排序、不是插入顺序。

**31.** `TreeSet` 判断「元素重复」的依据是（　A　）

- A. `equals()` 返回 `true`
- B. `==` 为 `true`
- C. `compareTo` / `compare` 返回 `0`
- D. `hashCode` 相同

> ❌ 错因：TreeSet 判重复看 **`compareTo`/`compare` 返回 0**，根本不看 equals。比较器只比姓名时，同名不同人会被静默丢掉。正确：C。

**32.** 一个没有实现 `Comparable` 的类，往 `TreeSet` 里添加元素时（　B　）

- A. 加第一个就抛 `ClassCastException`
- B. 加第二个才抛 `ClassCastException`
- C. 永远不报错，只是无序
- D. 编译报错

**33.** 关于 `hashCode`，说法正确的是（　A　）

- A. `hashCode` 返回的就是对象的内存地址
- B. `String` 的 `hashCode` 是按内容计算的
- C. `hashCode` 不同，`equals` 也可能为 `true`
- D. 重写 `equals` 不需要重写 `hashCode`

> ❌ 错因：hashCode 不是内存地址。String 按内容算，Object 默认与对象身份相关；hashCode 不同只能推出 equals 不相等。正确：B。

**34.** `HashMap` 的底层结构是（　B　）

- A. 纯数组
- B. 数组 + 链表（链表过长且数组 ≥ 64 时转红黑树）
- C. 纯红黑树
- D. 双向链表

**35.** 关于 `HashMap` 的 `null`，说法正确的是（　C　）

- A. 不允许 `null` 键，也不允许 `null` 值
- B. 允许一个 `null` 键，允许多个 `null` 值
- C. 允许任意多个 `null` 键
- D. 允许 `null` 键但只允许一个 `null` 值

> ❌ 错因：HashMap 允许**一个** null 键（hash 记为 0，固定落 0 号桶），值可以有任意多个 null。你选 C「任意多个 null 键」方向反了，正确：B。

**36.** `HashMap` 默认容量 16、负载因子 0.75，放第几个元素时触发扩容？（　A　）

- A. 第 12 个
- B. 第 13 个
- C. 第 16 个
- D. 第 17 个

> ❌ 错因：阈值 = 16×0.75 = **12**，size**超过** 12（放入第 13 个）才扩容。不是第 12 个，也不是装满 16 个。

**37.** `LinkedHashMap` 比 `HashMap` 多的能力是（　B　）

- A. 按键排序
- B. 保持插入顺序
- C. 线程安全
- D. 支持泛型

**38.** 关于 `TreeMap`，说法正确的是（　C　）

- A. 允许 `null` 键
- B. 按值排序
- C. 按 key 排序，不允许 `null` 键
- D. 保持插入顺序

**39.** `map.put(k, v)` 的返回值是（　C　）

- A. 新值 `v`
- B. `null` 永远
- C. 这个 key 原来的旧值；没有旧值时返回 `null`
- D. 布尔值，表示是否添加成功

**40.** 自定义对象作为 `HashMap` 的 key，只重写 `equals` 不重写 `hashCode`，`get` 可能返回（　B　）

- A. 一定能查到
- B. `null`，因为两个「内容相同」的对象哈希不同、落到不同桶
- C. 抛 `ClassCastException`
- D. 抛 `NullPointerException`

**41.** 要同时遍历 Map 的 key 和 value，且数据量大时，效率更高的方式是（D）

- A. `keySet()` 再逐个 `get()`
- B. `entrySet()`
- C. `values()` 再手动配 key
- D. 转成数组遍历

> ❌ 错因：数据量大时用 **B. entrySet()**：一次拿到键值对，不用对每个 key 再算哈希、再查表。keySet+get 是二次查找，转数组更慢。

**42.** 通过**方法引用**调用一个不需要对象就能执行的函数，应使用（　B　）

- A. `对象名::静态方法`
- B. `类名::静态方法`
- C. `类名::实例方法`（第一个参数当调用者）
- D. `this::静态方法`

## 二、多选题（5 小题，每题 3 分，共 15 分）

**43.** 关于 `ArrayList` vs `LinkedList`，说法正确的有（　A\C\D\E ）

- A. `ArrayList` 随机访问快
- B. 两者在中间插入元素都主要是 O(n)（数组要搬移、链表要遍历找位置）
- C. `LinkedList` 每个节点额外存前后指针，内存开销更大
- D. `ArrayList` 支持下标访问，`LinkedList` 用 `get(i)` 需要从头/尾遍历
- E. `Arrays.asList()` 返回的是可增删的标准 `ArrayList`

> ❌ 错因：正确是 **A B C D**。你漏了 B（中间插入两者都主要是 O(n)：数组搬移、链表遍历找位置）；多选了 E，`Arrays.asList` 是固定长度列表，不是标准 ArrayList。

**44.** 遍历集合又需要删除元素，安全的写法有（　A\E　）

- A. `removeIf(...)`
- B. `Iterator` 的 `it.remove()`
- C. 对 `List` 倒着用普通 for 按下标删
- D. 正着用普通 for 按下标删
- E. 在增强 for 里直接 `list.remove(x)`

> ❌ 错因：正确是 **A B C**。你多选了 E：增强 for 里直接删正是触发 `ConcurrentModificationException` 的错误写法。

**45.** 往 `HashSet` 里加自定义对象去重，必须重写（　A\B　）

- A. `hashCode()`
- B. `equals()`
- C. `toString()`
- D. `compareTo()`
- E. `getClass()`

**46.** 关于 `TreeSet` / `TreeMap`，说法正确的有（　A\B\C　）

- A. `compareTo` 返回 0 时，`TreeSet` 丢弃新元素
- B. `compareTo` 返回 0 时，`TreeMap` 用新 value 覆盖旧 value
- C. `TreeMap` 的 key 必须实现 `Comparable` 或传入 `Comparator`
- D. `TreeSet` 的排序规则只能写死一套，不能用 `Comparator`
- E. 红黑树的中序遍历能得到有序序列

> ❌ 错因：漏选 E：红黑树中序遍历（左→根→右）天然有序，这也是 TreeMap 遍历有序的原因。正确：A B C E。

**47.** 关于 `HashMap` 的遍历方式，说法正确的有（　A\B\C\D　）

- A. `keySet` + `get` 每次都要二次查找，数据量大时慢
- B. `entrySet` 遍历时键和值从同一个节点取，不用二次查找
- C. `forEach((k, v) -> ...)` 本质上是 `entrySet` 遍历的语法糖
- D. 遍历 Map 想删元素，用 `removeIf` 或迭代器
- E. `keySet` 能连 value 一起直接拿到

> ❌ 错因：正确是 **A B C D**。你多选了 E：keySet 只能拿到 key，拿不到 value。D 项（用 removeIf 或迭代器删除）是对的。

## 三、判断题（10 小题，每题 1 分，共 10 分）

**48.** `ArrayList` 实现了 `List` 接口，`HashSet` 实现了 `Set` 接口。（　T　）

**49.** `HashMap` 是 `Collection` 接口的实现类。（　T　）

> ❌ 错因：`HashMap` **不是** Collection 的实现类，它是 Map 体系的（Collection 只有 List/Set/Queue 三支）。正确：×。

**50.** `LinkedHashSet` 的「保序」保的是插入顺序。（　T　）

**51.** `TreeSet` 允许存放 `null`。（　F　）

**52.** `List.of(...)` 返回的集合可以调用 `add`。（　T　）

> ❌ 错因：`List.of` 返回不可变列表，任何写操作都抛 `UnsupportedOperationException`，所以这句**错误**（应打 ×）。

**53.** `List.of(...)` 返回的集合调用 `set` 也会抛 `UnsupportedOperationException`。（　F　）

> ❌ 错因：不可变列表连 `set` 也不行，所以这句**正确**（应打 √）。你 52、53 两题的答案正好相反。

**54.** `HashSet` 的迭代顺序是插入顺序。（　F　）

**55.** 通过 `Iterator` 的 `remove()` 删除元素，是推荐的做法。（　F　）

> ❌ 错因：迭代器 `remove()` 删的是 `next()` 刚返回的元素、会同步版本号，正是推荐做法（应打 √）。

**56.** `Map` 的 key 可以重复，value 不能重复。（　F　）

**57.** 自定义对象作 key 时，`hashCode` 和 `equals` 必须成套重写。（　T　）

## 四、代码分析题（4 小题，每题 4 分，共 16 分）

**58.** 写出输出结果。

```java
List<Integer> list = new ArrayList<>(List.of(10, 20, 30));
list.remove(1);
System.out.println(list);
List<String> s = new ArrayList<>(List.of("a", "b", "c"));
s.remove("b");
System.out.println(s);
```
[10, 30]
["a","c"]
**59.** 写出输出结果。

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
18,18

> ❌ 错因：问的是 `set.size()`。两个 Person 没重写 equals/hashCode，去重按内存地址判断 → 结果 **1**，不是把对象打印成 18,18。
**60.** 写出输出结果。

```java
Map<String, Integer> map = new HashMap<>();
System.out.println(map.put("a", 1));
System.out.println(map.put("a", 2));
System.out.println(map.get("a"));
System.out.println(map.size());
```
a:1
a:1
2
1
**61.** 写出输出结果。

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
2
3
错误
错误

> ❌ 错因：后两行应是 **1** 和 **2**：HashMap 允许一个 null 键，第二次 `put(null,2)` 是覆盖 value，size 仍为 1，`get(null)` 得 2。不是「错误」。
## 五、填空题 / 简答题（8 小题，共 8 分）

**62.**（1 分）`HashMap` 默认容量是 `_____16___`，默认负载因子是 `____0.75____`，扩容倍数是 `________`。

**63.**（1 分）链表转红黑树的要求是链表长度 ≥ `_____6___` **且** 数组长度 ≥ `___64_____`；红黑树节点数降到 ≤ `___6_____` 时退回链表。

> ❌ 错因：两个阈值记反：链表长度 ≥ **8**（不是 6）且数组 ≥ 64 才转红黑树；节点数降到 ≤ **6** 才退回链表。

**64.**（1 分）集合契约：`equals` 返回 `true` 时，`hashCode` 必须 `_____true___`；反过来不要求。

**65.**（1 分）`HashSet` 的底层其实是包了一个 `____ArrayList____`（填具体类名）。

> ❌ 错因：HashSet 底层包的其实是 **HashMap**（value 统一放一个占位对象），不是 ArrayList。

**66.**（1 分）遍历 `List` 时安全删除，除了迭代器的 `remove()`，还可以用 `____forEach____` 方法配合 lambda。

> ❌ 错因：安全删除的另一个方法是 **`removeIf`**，不是 forEach。forEach 只遍历，删元素还是得靠迭代器或 removeIf。

**67.**（1 分）`TreeSet` 的元素排序规则写在 `____红黑树____` 里；`TreeMap` 的 key 必须实现 `____Comparable____` 或传入 `___Comparator_____`。

> ❌ 错因：排序规则写在 **compareTo**（或传入 Comparator）里，**不是红黑树**——红黑树只是底层存储结构，不负责定规则。后两空 Comparable/Comparator 是对的。

**68.**（1 分）`Collections.shuffle(list)` 的作用是 `________`，它只接受 `________` 类型的集合。

**69.**（2 分）简答：为什么「往 HashMap 里放自定义对象当 key，`get` 却拿到 `null`」？请从 hashCode、equals、桶 三个词讲清楚。
不知道

> ❌ 错因：要点顺序：hashCode 没重写 → 内容相同的两个对象算出不同哈希 → 落到不同桶 → equals 根本没机会比 → get 在错的地方查，返回 null。

---

# 模块三　泛型 / 不可变集合 / Stream / 方法引用（52 分）

> 出自：`2026-10-29`、`2026-11-08`、`2026-11-09`、`2026-11-10` 日记 + 对应教程

## 一、单选题（10 小题，每题 2 分，共 20 分）

**70.** 泛型最大的价值是（　B　）

- A. 让程序跑得更快
- B. 把运行期才暴露的类型错误提前到编译期
- C. 让集合能自动排序
- D. 减少内存占用

**71.** 代码 `List<int> list = new ArrayList<>();` 的问题是（　A　）

- A. `int` 不能作为泛型参数，应该用 `Integer`
- B. `ArrayList` 不支持泛型
- C. 缺少 `import`
- D. 没问题

**72.** 已知 `List<Integer> ints`，执行 `List<Number> nums = ints;` 的结果是（　B　）

- A. 编译通过，`Integer` 是 `Number` 的子类
- B. 编译错误，因为泛型不变（`List<Integer>` 不是 `List<Number>` 的子类）
- C. 运行时抛 `ClassCastException`
- D. 编译通过但运行期丢失泛型

**73.** `List<? extends Number>` 能做什么？（　B　）

- A. 能读，读出可当 `Number` 用
- B. 能写，可以 `add` 任何 `Number` 的子类
- C. 能读也能写
- D. 不能读也不能写

> ❌ 错因：这是**上界通配符**：`? extends Number` 只读不可写。编译器不知道具体是 Integer 还是 Double，不敢让你 add。B 描述的是下界通配符 `? super` 的行为。正确：A。

**74.** 下面哪个写法**无法通过编译**？（　B　）

- A. `list instanceof List<String>`
- B. `list instanceof List`
- C. `new ArrayList<String>()`
- D. `new ArrayList<>(List.of(1, 2, 3))`

> ❌ 错因：带泛型的 `instanceof` 编译报错（泛型被擦除，运行时判断不了）；`list instanceof List` 是合法的。所以「无法编译」选 A。

**75.** Stream 的三条铁律**不包括**（　A　）

- A. 不修改原数据
- B. 只能消费一次
- C. 惰性求值
- D. 自动并行执行

> ❌ 错因：题问「不**包括**」：不修改原数据、只能消费一次、惰性求值都是铁律，**自动并行执行不是**。你选的「不修改原数据」恰恰是铁律之一。

**76.** 想把 `List<List<Integer>>` 拍平成 `List<Integer>`，应该用（　D　）

- A. `map`
- B. `flatMap`
- C. `filter`
- D. `distinct`

> ❌ 错因：拍平嵌套用 **`flatMap`**（一变多）；map 是一变一，distinct 是去重。

**77.** 下列全部属于**中间方法**的一组是（　D　）

- A. `filter`、`map`、`sorted`
- B. `filter`、`count`、`collect`
- C. `forEach`、`map`、`limit`
- D. `count`、`collect`、`reduce`

> ❌ 错因：问「**全部属于中间方法**」：A 的 filter/map/sorted 都返回 Stream。B、C、D 里都混了 count/collect/forEach 这类终结方法；你选的 D 全是终结方法。

**78.** `stream.toList()` 与 `stream.collect(Collectors.toList())` 的关键区别是（　D　）

- A. 前者不可变，后者可变
- B. 前者可变，后者不可变
- C. 两者完全一样
- D. 前者只能用于字符串流

> ❌ 错因：`toList()` 返回**不可变**列表，`collect(Collectors.toList())` 返回**可变** ArrayList。名字像兄弟，行为相反，A。

**79.** 下列能写成方法引用的是（　A　）

- A. `s -> s.trim()`
- B. `s -> s.trim().length()`
- C. `s -> s.length() > 5`
- D. `(a, b) -> a + b`

**80.** `String::length` 等价于（　D　）

- A. `() -> "abc".length()`
- B. `(s) -> s.length()`
- C. `(a, b) -> a.length(b)`
- D. `(s) -> String.length(s)`

> ❌ 错因：`String::length` 等价 `(s) -> s.length()`：类名::实例方法，第一个参数当调用者。A 是无参 lambda，D 是错误写法。

## 二、多选题（4 小题，每题 3 分，共 12 分）

**81.** 关于泛型通配符，说法正确的有（　B\C\D　）

- A. `? extends Number` 可读不可写
- B. `? super Integer` 可写，但读出来只能当 `Object`
- C. `List<? extends Number>` 可以接收 `List<Integer>` 和 `List<Double>`
- D. 泛型在运行时会被擦除
- E. `List<Integer>` 是 `List<Number>` 的子类

> ❌ 错因：漏选 A：`? extends Number` 可读不可写，是 PECS 的 Producer Extends。正确：A B C D。

**82.** 关于方法引用，说法正确的有（　A\B\C\E　）

- A. `forEach(System.out::println)` 可用
- B. `forEach(x -> System.out.println(x))` 与 A 等价
- C. `map(String::trim)` 可用（`trim` 没有额外参数）
- D. `map(s -> s.trim().toUpperCase())` 能写成 `String::trim`
- E. `类名::new` 可以引用构造方法

> ❌ 错因：漏选 C：`map(String::trim)` 可用（lambda 体只有一个 trim 调用）。D 不行，trim 后还有 toUpperCase，两个动作转不了方法引用。正确：A B C E。

**83.** 关于不可变集合，说法正确的有（　A\B\C\D　）

- A. `List.of` 返回的集合任何写操作都抛 `UnsupportedOperationException`
- B. `Set.of("a","a")` 创建时就抛异常，不会静默去重
- C. `Collections.unmodifiableList` 是视图，原集合改了它也变
- D. `List.copyOf` 是真拷贝，原集合改了它不变
- E. 不可变集合连元素内部状态也锁死了

> ❌ 错因：漏选 A、D：`List.of` 任何写操作都抛异常；`copyOf` 是**真拷贝**（照片），原集合改它不变，而 `unmodifiableList` 是**视图**（镜子）。正确：A B C D。

**84.** 关于 Stream，说法正确的有（　A\B\E　）

- A. 没有终结方法，中间操作一个都不会执行
- B. 同一个 Stream 对象第二次使用抛 `IllegalStateException`
- C. `collect(Collectors.toList())` 返回 `ArrayList`，可继续改
- D. `Collectors.toMap` 遇重复 key 默认抛异常，可加合并函数解决
- E. `filter`、`sorted` 会修改原集合

> ❌ 错因：多选了 B、E：B 把异常主体说反（原文是「第二次使用」抛，但 B 表述本身成立）；E 错，filter/sorted 不修改原集合。正确：A C D。

## 三、判断题（8 小题，每题 1 分，共 8 分）

**85.** 泛型只在编译期有效，运行时会被擦除。（　T　）

**86.** 可以创建泛型数组 `new T[10]`。（　F　）

**87.** `List<Integer>` 可以直接赋值给 `List<Number>`。（　F　）

**88.** `? extends` 适合「往外读」的场景，`? super` 适合「往里写」的场景。（　T　）

**89.** Stream 的 `filter` 不调用终结方法也会立刻执行。（　F　）

**90.** 一条 Stream 消费完可以重复使用。（　F　）

> ❌ 错因：Stream 消费完就废了，再走得从源重新 `stream()` 一次；变量还在不代表流还活着。正确：×。

**91.** `anyMatch` 是终结方法，`filter` 是中间方法。（　T　）

**92.** `stream.toList()` 返回的列表是不可变的。（　F　）

> ❌ 错因：`stream.toList()` 返回的恰恰是**不可变**列表，所以这句**正确**（应打 √）。

## 四、代码分析题（3 小题，每题 4 分，共 12 分）

**93.** 写出输出结果。

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4, 5));
List<Integer> result = list.stream()
        .filter(x -> x > 2)
        .map(x -> x * 2)
        .collect(Collectors.toList());
System.out.println(result);
```
[6,8,10]

> ❌ 错因：`filter(x>2)` 得 [3,4,5]，再 `map(x*2)` 得 **[6, 8, 10]**。你写的 [4,6,8,10] 多算了 x=2 本身（2 不大于 2，被滤掉）。
**94.** 写出输出结果（或异常）。

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
["a","b","c","d"]

> ❌ 错因：第二行应打印 `UnsupportedOperationException`（不可变列表 add 抛这个异常类名），你只写了第一行。

**95.** 写出输出结果。

```java
System.out.println(Stream.of(1, 2, 3).map(String::valueOf).collect(Collectors.joining(",")));
System.out.println(List.of(1, 2, 3, 4).stream().map(x -> x * 2).filter(x -> x > 4).count());
```
不懂
6,8

> ❌ 错因：第一行 `1,2,3`（joining 用逗号拼字符串）；第二行 `2`（map 后 [2,4,6,8]，filter>4 剩 [6,8]，count 得 2，不是打印集合）。
## 五、简答题（1 小题，2 分）

**96.** 为什么「写了一条 `stream.filter(...).map(...)` 但一句输出都没有」？用「惰性求值」「终结方法」两个词回答。
因为没有终结方法，所以中间的不输出

> ❌ 错因：方向对（没有终结方法），但没点出「**惰性求值**」：中间操作只登记不执行，靠终结方法（count/collect/forEach）才真正跑起来。

---

# 模块四　异常 / File / IO / Properties（64 分）

> 出自：`2026-11-11`、`2026-11-16`、`2026-11-23` 日记
> + `异常教程`、`File类教程`、`IO流教程` 三部曲

## 一、单选题（10 小题，每题 2 分，共 20 分）

**97.** `Throwable` 往下分成哪两支？（　A　）

- A. `Exception` 和 `RuntimeException`
- B. `Error` 和 `Exception`
- C. `IOException` 和 `Error`
- D. `Exception` 和 `Throwable`

> ❌ 错因：Throwable 往下是 **Error 和 Exception** 两支。你选的 A 把 Exception 和它的子类 RuntimeException 当成了两支。

**98.** 下列属于**编译时异常**的是（　C　）

- A. `NullPointerException`
- B. `ArithmeticException`
- C. `IOException`
- D. `ArrayIndexOutOfBoundsException`

**99.** 方法抛出异常且没被捕获时，JVM 会（　B　）

- A. 忽略异常继续执行
- B. 打印异常信息并终止程序
- C. 自动重试一次
- D. 把异常存进日志文件

**100.** `throw` 和 `throws` 的正确区别是（　B　）

- A. `throw` 写在方法签名上，`throws` 写在方法体里
- B. `throw` 在方法体里抛出异常对象，`throws` 在方法签名上声明可能抛出的异常
- C. 两者都可以写在任意位置
- D. `throws` 会真的抛异常，`throw` 不会

**101.** 多个 `catch` 的顺序应该是（　A　）

- A. 父类异常在前，子类在后
- B. 子类异常在前，父类在后
- C. 随便排
- D. 按异常代码长度排

> ❌ 错因：多个 catch 必须**子类在前、父类在后**。父类能吃下所有子类异常，排前面后面的子类分支永远到不了。正确：B。

**102.** `finally` 块的执行情况是（　C　）

- A. 只有 try 正常结束时才执行
- B. 只有 catch 执行过才执行
- C. 无论 try 正常结束、抛异常还是 return，都会执行
- D. 只要 try 里 return 了就不执行

**103.** 自定义异常 `class BalanceNotEnoughException extends RuntimeException` 属于（　C　）

- A. 编译时异常，必须处理
- B. 运行时异常，可以不强制处理
- C. `Error` 的子类
- D. 不能自定义异常

> ❌ 错因：`class BalanceNotEnoughException extends RuntimeException` 属于**运行时异常**（可不强制处理）。选项 C 错，因为它不是 Error 的子类。正确：B。

**104.** 关于 `File` 对象，说法正确的是（　B　）

- A. `new File("x.txt")` 会在磁盘上创建文件
- B. `File` 只是一个路径的封装，存不存在要另查
- C. `File` 能直接读写文件内容
- D. `new File("不存在的")` 会抛 `IOException`

**105.** `mkdir()` 和 `mkdirs()` 的区别是（　B　）

- A. 两者完全一样
- B. `mkdir()` 只能建一级，`mkdirs()` 能连父目录一起建
- C. `mkdirs()` 只能建一级
- D. `mkdir()` 会覆盖已有目录

**106.** `File.delete()` 删除一个非空目录时会（　C　）

- A. 递归删除成功
- B. 返回 `false`，不报错
- C. 抛 `IOException`
- D. 抛 `NullPointerException`

> ❌ 错因：`delete()` 删非空目录**返回 false、不报错**（不抛 IOException）。正确：B。

**107.** `dir.listFiles()` 返回 `null` 的情况**不包括**（　B　）

- A. 路径不存在
- B. 路径是个文件而不是目录
- C. 没有读取权限
- D. 目录是空的

> ❌ 错因：空目录返回的是**空数组**，不是 null；返回 null 的只有路径不存在、路径是文件、没有权限三种。所以「不包括」选 D（空目录）。

**108.** 关于字节流和字符流，说法正确的是（　B　）

- A. 字符流通用，能处理图片
- B. 字节流不涉及编码，字符流必须指定字符集
- C. 字符流底层是独立的另一套机制，跟字节流无关
- D. 字节流一次读写 16 位

**109.** 用字节流直接读 UTF-8 中文文本文件时可能出现的问题是（　B　）

- A. 一定抛异常
- B. 一个汉字占 3 字节，按字节切可能被截断，出现乱码
- C. 中文会变成大写
- D. 没有问题

**110.** `FileInputStream.read()` 读到文件末尾时返回（　B　）

- A. `0`
- B. `-1`
- C. `null`
- D. 抛 `EOFException`

**111.** 批量读写法 `fos.write(buf, 0, len)` 中必须用 `len` 而不是 `buf.length` 的原因是（　B　）

- A. `buf.length` 会编译报错
- B. 最后一次读往往没读满，写多了会把上次残留的旧数据也写进去
- C. `len` 表示文件总长度
- D. 两者效果完全一样

**112.** `try-with-resources` 的前提是资源必须实现（　B　）

- A. `Serializable`
- B. `AutoCloseable`
- C. `Iterable`
- D. `Runnable`

**113.** 对象要能被 `ObjectOutputStream` 序列化，类必须实现（　C　）

- A. `Runnable`
- B. `Serializable`
- C. `Comparable`
- D. `Cloneable`

> ❌ 错因：要被 `ObjectOutputStream` 序列化，类必须实现 **Serializable**（Comparable 是排序、Cloneable 是克隆）。正确：B。

**114.** `transient` 关键字的作用是（  B ）

- A. 让字段临时可改
- B. 让该字段不参与序列化
- C. 让字段变成静态
- D. 让字段参与序列化但加密

**115.** `BufferedInputStream` 的构造方式是（　A　）

- A. 接收文件路径字符串
- B. 接收另一个 `InputStream`，包在它外面
- C. 接收 `File` 对象
- D. 无参构造

> ❌ 错因：`BufferedInputStream` 接收的是**另一个 InputStream**，包在它外面，不是路径字符串。正确：B。

**116.** `LocalDate.of(2024, 10, 19)` 中 10 表示（　B　）

- A. 11 月
- B. 10 月（JDK8 的月份从 1 开始）
- C. 9 月（从 0 开始）
- D. 非法参数

**117.** `LocalDate` 的 `plusDays(1)` 会（　B　）

- A. 修改原对象本身
- B. 返回一个新对象，原对象不变
- C. 抛异常
- D. 只改日期里的日，不改月份

**118.** `Period.between(LocalDate.of(2024,1,1), LocalDate.of(2024,3,15)).getDays()` 返回（　A　）

- A. 74（总天数）
- B. 14（零头天数）
- C. 2（月数）
- D. 60

> ❌ 错因：`Period.getDays()` 返回零头天数 **14**，不是总天数 74；要总天数得用 `ChronoUnit.DAYS.between`。正确：B。

**119.** `Properties.load(InputStream)` 默认按什么编码读？（　D　）

- A. UTF-8
- B. GBK
- C. ISO-8859-1（Latin-1）
- D. 系统默认编码

> ❌ 错因：`Properties.load(InputStream)` 默认按 **ISO-8859-1** 读，不是系统默认编码；要 UTF-8 得 `load(InputStreamReader)` 显式指定。正确：C。

**120.** 关于 `Properties.store` 写中文的现象，实际运行结论是（　A　）

- A. 默认流下会乱码
- B. 默认流下会把中文转成 `\uXXXX` 转义，文件里看不到中文但 `load` 回来正常
- C. 会抛异常
- D. 中文会被直接丢弃

> ❌ 错因：实测结论是 **B**：默认 store 把中文写成 `\uXXXX` 转义，文件里看不到中文，但 load 回来正常。A 说的乱码是混用编码（写 UTF-8 却用默认流读）才出现。

**121.** 下列代码的输出是（　D　）

```java
File dir = new File("no_such_dir");
System.out.println(dir.listFiles());
```

- A. 空数组 `[]`
- B. `null`
- C. 抛 `NullPointerException`
- D. 抛 `IOException`

> ❌ 错因：`File("no_such_dir").listFiles()` 返回 **null**，不抛异常；所以用 listFiles 后必须先判空。正确：B。

## 二、多选题（4 小题，每题 3 分，共 12 分）

**122.** 关于异常体系，说法正确的有（　A\B\C\D ）

- A. `Error` 通常不在业务代码里处理
- B. `RuntimeException` 及其子类属于运行时异常
- C. `IOException` 属于编译时异常，必须 try-catch 或 throws
- D. `IllegalArgumentException` 是运行时异常
- E. 加了 `try-catch`，`finally` 就不一定执行了

**123.** 关于 try-with-resources，说法正确的有（　A\C\D\E　）

- A. 多个资源用分号分隔
- B. 与 `try-catch` 完全互斥，不能同时用
- C. 退出 try 块时自动关闭资源
- D. 关闭顺序与创建顺序相反
- E. 关闭时抛的异常会作为 `suppressed` 附加到主异常上

> ❌ 错因：正确 **A C D E**。你漏了 E（关闭异常会作为 suppressed 附加），另外 B 是错的（try-with-resources 可以和 catch 同时用）。

**124.** 关于序列化，说法正确的有（　A\B\D　）

- A. 类必须实现 `Serializable`
- B. `static` 字段不参与序列化
- C. `transient` 字段不参与序列化
- D. `serialVersionUID` 用于版本校验，不一致可能抛 `InvalidClassException`
- E. 用 `ObjectOutputStream` 连续写多个对象时，可以一次 `readObject` 全部读回

> ❌ 错因：正确 **A B C D**。你漏了 C：`transient` 字段**不参与序列化**，这是它的定义。

**125.** 关于 File 的路径与操作，说法正确的有（　A\B\C\d　）

- A. 相对路径的基准是「当前工作目录」
- B. 字符串路径里写 `/`，在 Windows 上 Java 也能识别
- C. `getPath()` 原样返回构造时给的路径，`getAbsolutePath()` 会转成绝对路径
- D. 目录的 `length()` 返回的是它里面所有文件的总大小
- E. `delete()` 删除目录时如果目录非空会返回 `false`

> ❌ 错因：正确 **A B C E**。你多选了 D：目录的 `length()` 是目录项本身大小，不是里面文件的总和。

## 三、判断题（10 小题，每题 1 分，共 10 分）

**126.** `IOException` 不处理就无法通过编译。（　T　）

**127.** `IllegalArgumentException` 名字里没有 `Runtime`，所以它是编译时异常。（　T　）

> ❌ 错因：`IllegalArgumentException` 继承了 `RuntimeException`，是**运行时异常**，名字里没有 Runtime 不算数——**错误**（应打 ×）。

**128.** `finally` 块即使 `try` 里执行了 `return` 也会执行。（　T　）

**129.** 一个 `catch` 块里可以用 `|` 同时捕获多种异常类型。（　T　）

**130.** `new File("不存在的文件.txt")` 会抛异常。（　F　）

**131.** `File` 类既能管路径，也能直接读写文件内容。（　F　）

**132.** `try (FileInputStream in = ...) { }` 能自动关闭 `in`。（　T　）

**133.** 字符流的底层就是字节流加编码转换。（　T　）

**134.** `LocalDate` 是可变对象，`plusDays` 会改掉它本身。（　T　）

> ❌ 错因：`LocalDate` 是**不可变**对象，plusDays 返回新对象——**错误**（应打 ×）。

**135.** `Period.getDays()` 返回的是两个日期之间的总天数。（　T　）

> ❌ 错因：`Period.getDays()` 是零头天数，不是总天数——**错误**（应打 ×）。

## 四、代码分析题（3 小题，每题 4 分，共 12 分）

**136.** 写出输出结果。

```java
try {
    int[] arr = new int[2];
    arr[5] = 1;
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println(e.getMessage());
} finally {
    System.out.println("done");
}
```
ArrayIndexOutOfBoundsException
done

> ❌ 错因：`getMessage()` 打印的是 `Index 5 out of bounds for length 2`，你只写了异常类名；第二行 done 对。
**137.** 写出输出结果。

> ❌ 错因：未作答。第一行 `NumberFormatException`，第二行 `10/0` 抛 `ArithmeticException`（程序终止）。

```java
try {
    Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println(e.getClass().getSimpleName());
}
System.out.println(10 / 0);
```

**138.** 写出输出结果。

```java
LocalDate d = LocalDate.of(2024, 10, 19);
LocalDate d2 = d.plusDays(1);
System.out.println(d2);
Calendar cal = new GregorianCalendar(2024, Calendar.JANUARY, 1);
System.out.println((cal.get(Calendar.MONTH) + 1) + " +3 = " + (cal.get(Calendar.MONTH) + 3));
```
2024,10,20

> ❌ 错因：第一行 `2024-10-20`（LocalDate 默认是带横杠的 ISO 格式），你写成 2024,10,20；第二行 `1 +3 = 3`（Calendar 的 MONTH 从 0 开始，JANUARY=0），整行漏了。

## 五、填空题（5 小题，每题 2 分，共 10 分）

**139.** 异常体系的顶层是 `___IOException_____`；它下面分两支：`________` 和 `Exception`。

> ❌ 错因：异常体系顶层是 **Throwable**，不是 IOException（IO 异常只是 Exception 分支中的一员）。第二空位应是 Error。

**140.** `InputStream.read()` 返回 `int` 而不是 `byte`，是为了用 `___-1_____` 这个特殊值表示「读完了」。

**141.** 要能被序列化，类必须实现 `________` 接口；不想被序列化的字段加 `________` 关键字。

**142.** IO 流的四个基类是 `__InputStream______` / `__OutputStream______` / `____InputRead____` / `___OutputRead_____`。

> ❌ 错因：四个基类是 **InputStream / OutputStream / Reader / Writer**。你后两个写成 InputRead/OutputRead，正确类是 Reader（字符输入）和 Writer（字符输出）。

**143.** `LocalDate` 的月份从 `_____0___` 开始（填 0 或 1）；`Calendar` 的月份从 `____1____` 开始。

> ❌ 错因：两个月份起点正好写反：`LocalDate` **从 1 开始**（你答 0 错），`Calendar` **从 0 开始**（你答 1 错）。

---

# 模块五　正则 / 时间 API / 算法（41 分）

> 出自：`2026-10-29`、`2026-10-31`、`2026-11-03` 日记 + `正则表达式教程`、`JDK时间与分组教程`、`常见算法与lambda教程`

## 一、单选题（8 小题，每题 2 分，共 16 分）

**144.** `"a123b".matches("\\d+")` 的结果是（　A　）

- A. `true`，因为包含数字
- B. `false`，因为 `matches` 要求整串匹配
- C. 编译错误
- D. 抛 `PatternSyntaxException`

> ❌ 错因：`matches()` 要求**整串**匹配："a123b" 不是纯数字，结果是 **false**，不是 true。正确：B。

**145.** 想在 `"hello123"` 里查找数字子串 `123`，应该用（　A　）

- A. `matches("\\d+")`
- B. `Pattern.compile("\\d+").matcher(s).find()`
- C. `split("\\d+")`
- D. `replace("\\d+", "")`

> ❌ 错因：只想查找子串用 **find()**；matches 是整串、split 是切割、replace 是替换。正确：B。

**146.** 在 Java 字符串里写正则 `\d`，正确写法是（　B　）

- A. `"\d"`
- B. `"\\d"`
- C. `"/d"`
- D. `"[d]"`

**147.** `String.replace()` 与 `String.replaceAll()` 的区别是（　A　）

- A. `replace` 是正则替换，`replaceAll` 是纯文本替换
- B. `replace` 是纯文本替换，`replaceAll` 的参数是正则
- C. 两者完全一样
- D. `replaceAll` 只能替换一次

> ❌ 错因：`replace` 是**纯文本**替换，`replaceAll` 的参数才是正则。你说反了。正确：B。

**148.** `"a.b.c".split("\\.")` 的结果是（　A　）

- A. `["a", "b", "c"]`
- B. `[]`
- C. `["a.b.c"]`
- D. 抛异常

**149.** 正则 `(a)(b)` 匹配 `"ab"` 时，`group(0)` / `group(1)` 分别是（　A　）

- A. `"a"` / `"b"`
- B. `"ab"` / `"a"`
- C. `"ab"` / `"ab"`
- D. `"b"` / `"a"`

> ❌ 错因：`group(0)` 是整段 **"ab"**，`group(1)` 才是第一组 **"a"**。正确：B。

**150.** 二分查找的前提是（　B　）

- A. 数组元素都是数字
- B. 数组必须已经有序
- C. 数组长度必须是 2 的幂
- D. 数组不能有重复元素

**151.** 二分查找里 `int mid = (lo + hi) / 2;` 在超大数组下可能（　B　）

- A. 变慢
- B. 溢出成负数
- C. 抛 `IndexOutOfBoundsException`
- D. 找不到最后一个元素

**152.** `Arrays.binarySearch(有序数组, 不存在的值)` 的返回值是（　B　）

- A. 固定 `-1`
- B. 负数，值为 `-(插入点) - 1`
- C. `0`
- D. 抛 `NoSuchElementException`

**153.** 下列排序算法中，平均时间复杂度为 O(n log n) 的是（　D　）

- A. 冒泡排序
- B. 选择排序
- C. 插入排序
- D. 快速排序

> ❌ 错因：平均 O(n log n) 的是 **快速排序**（D）；冒泡/选择/插入是 O(n²)。你选了冒泡，那是 O(n²)。

## 二、多选题（2 小题，每题 3 分，共 6 分）

**154.** 关于二分查找，说法正确的有（　A\B\C\D\E  ）

- A. 对无序数组使用会「静默查不到」，不会报错
- B. `while (lo <= hi)` 写成 `while (lo < hi)` 可能漏掉最后一个元素
- C. `hi = mid` 而不是 `mid - 1` 可能导致死循环
- D. 用 `mid = lo + (hi - lo) / 2` 可避免溢出
- E. `Arrays.binarySearch` 会先自动排序再查

> ❌ 错因：正确 **A B C D**。E 错：`Arrays.binarySearch` 不会先帮你排序，它假设数组已有序，传无序数组会静默查不到。

**155.** 关于递归，说法正确的有（　A\B\E　）

- A. 必须有终止条件（出口）
- B. 没有出口会抛 `StackOverflowError`
- C. `StackOverflowError` 属于 `Error`，`catch(Exception)` 抓不住
- D. 递归一定能用循环完全替代，且性能更好
- E. 写递归时可以先写「处理一个」的逻辑，再问「如果是子集合呢」

> ❌ 错因：正确 **A B C E**。你漏了 C：StackOverflowError 属于 Error，`catch(Exception)` 抓不住。D 也错（递归能用循环替代，但不总是性能更好）。

## 三、判断题（8 小题，每题 1 分，共 8 分）

**156.** `"123".matches("\\d+")` 返回 `true`。（　T　）

**157.** `"a.b"` 用 `replaceAll(".", "-")` 的结果是 `"a-b"`。（　T　）

> ❌ 错因：`replaceAll(".","-")` 里 `.` 是正则任意字符，结果是 **"---"** 不是 "a-b"。要按点替换得 `replace(".","-")` 或 `replaceAll("\\.","-")`。这句**错误**（应打 ×）。

**158.** `find()` 可以反复调用，依次找到下一个匹配。（　T　）

**159.** 二分查找即使数组无序，也能正确查找到存在的元素。（　F　）

**160.** 二分查找的时间复杂度是 O(log n)。（　T　）

**161.** `Arrays.binarySearch` 返回负数表示没找到，绝对值与插入点相关。（　T　）

**162.** 递归必须写出口，否则可能无限调用直到栈溢出。（　T　）

**163.** 快速排序在最坏情况下会退化成 O(n²)。（　T　）

## 四、代码分析题（2 小题，每题 4 分，共 8 分）

**164.** 写出输出结果。

> ❌ 错因：未作答。第一行 `false`（matches 要整串）；find() 循环打印 `123`；最后一行 `[]`（split(".") 按任意字符切，全是空串）。

```java
System.out.println("hello123".matches("\\d+"));
Matcher m = Pattern.compile("\\d+").matcher("hello123");
while (m.find()) {
    System.out.println(m.group());
}
System.out.println(Arrays.toString("a.b.c".split(".")));
```

**165.** 写出输出结果。

```java
int[] a = {7, 23, 79, 81, 103, 127, 131, 147};
int pos = Arrays.binarySearch(a, 131);
System.out.println(pos);
int[] sorted = {1, 3, 5, 9};
System.out.println(Arrays.binarySearch(sorted, 7));
```
7,23,79,81,103,127,131,147,131
1,3,5,7,9

> ❌ 错因：第一行是 `6`（binarySearch 返回下标 6），第二行是 `-4`（`-(3)-1`，7 应插在索引 3）。答案只要两个整数，不是把数组内容打印出来。

---

# 模块六　Docker（29 分）

> 出自：`2026-11-11`、`2026-11-16` 日记 + `Docker教程`

## 一、单选题（6 小题，每题 2 分，共 12 分）

**166.** Docker 三要素是（　A　）

- A. 镜像 / 容器 / 仓库
- B. 镜像 / 容器 / 卷
- C. 仓库 / 网络 / 卷
- D. 客户端 / 守护进程 / 镜像

**167.** 敲 `docker run` 时，真正干活的是（　B　）

- A. `docker` CLI 自身
- B. 宿主机的 `dockerd` 守护进程
- C. Docker Hub 服务器
- D. 容器里的 shell

> ❌ 错因：`docker` CLI 只是客户端，真正干活的是宿主机 **dockerd 守护进程**。正确：B。

**168.** 命令 `docker run -d --name mynginx -p 8080:80 nginx` 中，`-p 8080:80` 表示（　A　）

- A. 容器 8080 → 宿主机 80
- B. 宿主机 8080 → 容器 80
- C. 容器内部监听 8080 和 80
- D. 随机端口

> ❌ 错因：`-p 宿主:容器` = 8080:80 即**宿主机 8080 → 容器 80**。你选 A 把方向写反了。正确：B。

**169.** `docker exec -it 容器名 bash` 与 `docker run -it` 的区别是（　B　）

- A. 没有区别
- B. `exec` 进已运行的容器另开终端，`run` 是新起一个容器再进去
- C. `run` 进已运行的容器
- D. `exec` 会停止容器

**170.** 数据卷的主要作用是（　B　）

- A. 让容器跑得更快
- B. 把数据持久化到宿主机，容器删了数据还在
- C. 让容器不用联网
- D. 把镜像变小

**171.** 关于 `CMD` 与 `ENTRYPOINT`，说法正确的是（　B　）

- A. `CMD` 固定不可覆盖，`ENTRYPOINT` 可以被 `docker run` 覆盖
- B. `CMD` 可被 `docker run` 后面的参数覆盖，`ENTRYPOINT` 固定为入口、后面参数是追加
- C. 两者完全一样
- D. `ENTRYPOINT` 只能写在 Dockerfile 最后一行

## 二、多选题（2 小题，每题 3 分，共 6 分）

**172.** 下列关于删除与查看命令，说法正确的有（　A\B\C\E　）

- A. `docker rm` 删容器
- B. `docker rmi` 删镜像
- C. `docker rm -f` 可以强制删除运行中的容器
- D. `docker rmi` 可以删掉正在被容器使用的镜像，不受任何限制
- E. `docker ps -aq` 可以列出所有容器 ID

**173.** 关于数据卷挂载，说法正确的有（　A\B\C\D　）

- A. `-v 宿主机路径:容器内路径` 是最常用的写法
- B. `-v 卷名:容器内路径` 用具名卷
- C. `:ro` 表示容器内只读
- D. 容器删除后，挂载在宿主机目录里的数据还在
- E. 数据卷会让镜像体积变大

## 三、判断题（4 小题，每题 1 分，共 4 分）

**174.** 容器和宿主机共享同一个内核，容器镜像里不含完整的操作系统。（　T　）

**175.** Docker 默认的 bridge 网络下，容器之间可以用容器名直接互访。（　T　）

> ❌ 错因：默认 bridge 网络下容器**不能**用名字互访，要用自定义网络——**错误**（应打 ×）。

**176.** `docker attach` 退出时可能导致容器停止，日常应该用 `exec`。（　T　）

**177.** `docker commit` 适合生产环境的镜像构建，`Dockerfile` 只适合临时实验。（　T　）

> ❌ 错因：commit 适合临时实验，Dockerfile 才适合生产构建。你写的正好反了，**错误**（应打 ×）。

## 四、命令分析题（1 小题，4 分）

**178.** 逐行写出下列命令的作用（各一句话）：

```bash
docker pull nginx:latest
docker run -d --name web -p 8080:80 nginx
docker ps
docker exec -it web /bin/bash
docker rm -f web
```
从仓库拉取nginx最新版本的镜像
运行nginx的镜像，名字为web,端口宿主机映射为80端口
查看当前镜像
开一个新终端进入这个web(nginx)中，用bash shell

> ❌ 错因：三处要点：第二行 `-p 8080:80` 是**宿主机 8080 → 容器 80**，你写成「宿主机映射为 80」；第三行 `docker ps` 看的是**运行中容器**，不是镜像（看镜像是 `docker images`）；其余（拉镜像、进容器、删容器）基本对。
## 五、填空题（2 小题，每题 1.5 分，共 3 分）

**179.** Docker 的三个核心概念：`________`（只读模板）、`________`（运行实例）、`________`（存放镜像的地方）。

> ❌ 错因：未作答。三空：**镜像 / 容器 / 仓库**。

**180.** `docker run` 的完整逻辑：先看本地有没有这个 `____镜像____` → 有就直接用 → 没有就去 `___镜像仓库_____` 拉 → 拉到了再启动 `____镜像____`。

> ❌ 错因：第二空「镜像仓库」对；第三空应是启动**容器**，你写了镜像。

---

# 模块七　英语（20 分，选做）

> 出自：`笔记/英语/单词本.md`（你日记里已经背过的词）

## 一、单选题（5 小题，每题 2 分，共 10 分）

**181.** `override` 在 Java 里对应的是（  B ）

- A. 方法重载
- B. 方法重写
- C. 方法引用
- D. 方法内联

**182.** `iterator` 的词根含义是（　B　）

- A. 索引
- B. 迭代
- C. 递归
- D. 溢出

**183.** `immutable` 的意思是（　A　）

- A. 可修改的
- B. 不可变的
- C. 可序列化的
- D. 可比较的

> ❌ 错因：immutable = **不可变的**。你选「可修改的」把前缀 im- 的意思弄反了。正确：B。

**184.** `serialize` 对应 Java 里的（　A　）

- A. 序列化
- B. 反序列化
- C. 压缩
- D. 排序

**185.** `replication` 在数据库语境里常指（　A　）

- A. 主从复制
- B. 分库分表
- C. 索引重建
- D. 事务回滚

## 二、中译英（5 小题，每题 1 分，共 5 分）

**186.** 可移植性 → `________`

> ❌ 错因：未作答。可移植性 = **portability**。

**187.** 冗余 → `________`

> ❌ 错因：未作答。冗余 = **redundancy**。

**188.** 同步 → `________`

> ❌ 错因：未作答。同步 = **synchronization**（也可写 sync）。

**189.** 增量（的） → `________`

> ❌ 错因：未作答。增量（的）= **incremental**。

**190.** 适配器 → `________`

> ❌ 错因：未作答。适配器 = **adapter**。

## 三、英译中（5 小题，每题 1 分，共 5 分）

**191.** `emulation` → `________`

> ❌ 错因：未作答。emulation = **仿真 / 模拟**。

**192.** `checkpoint` → `________`

> ❌ 错因：未作答。checkpoint = **检查点**。

**193.** `interpolation` → `________`

> ❌ 错因：未作答。interpolation = **插值**。

**194.** `readability` → `___可读性_____`

**195.** `transaction` → `________`

> ❌ 错因：未作答。transaction = **事务**。

---

## 答题卡

> 可复制到下方自填，也可直接在聊天里按题号回复我（例如「1.B 2.C 3.B 4.B 5.D 6.B …」）。

### 模块一（1–20）

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |

| 10 | 11 | 12 | 13 | 14 | 15 | 16（写结果） |
|:-:|:-:|:-:|:-:|:-:|:-:|:--|
|   |   |   |   |   |   |   |

| 17 | 18 | 19 | 20（简答） |
|:-:|:-:|:-:|:--|
|   |   |   |   |

### 模块二（21–69）

| 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 | 40 | 41 | 42 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

| 43 | 44 | 45 | 46 | 47 |
|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |

| 48 | 49 | 50 | 51 | 52 | 53 | 54 | 55 | 56 | 57 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |

| 58 | 59 | 60 | 61（各写结果） |
|:-:|:-:|:-:|:--|
|   |   |   |   |

| 62 | 63 | 64 | 65 | 66 | 67 | 68 | 69（简答） |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--|
|   |   |   |   |   |   |   |   |

### 模块三（70–96）

| 70 | 71 | 72 | 73 | 74 | 75 | 76 | 77 | 78 | 79 | 80 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |   |

| 81 | 82 | 83 | 84 |
|:-:|:-:|:-:|:-:|
|   |   |   |   |

| 85 | 86 | 87 | 88 | 89 | 90 | 91 | 92 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |

| 93 | 94 | 95（写结果） | 96（简答） |
|:-:|:-:|:--|:--|
|   |   |   |   |

### 模块四（97–143）

| 97 | 98 | 99 | 100 | 101 | 102 | 103 | 104 | 105 | 106 | 107 | 108 | 109 | 110 | 111 | 112 | 113 | 114 | 115 | 116 | 117 | 118 | 119 | 120 | 121 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |

| 122 | 123 | 124 | 125 |
|:-:|:-:|:-:|:-:|
|   |   |   |   |

| 126 | 127 | 128 | 129 | 130 | 131 | 132 | 133 | 134 | 135 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |

| 136 | 137 | 138（写结果） | 139–143（填空） |
|:-:|:-:|:--|:--|
|   |   |   |   |

### 模块五（144–165）

| 144 | 145 | 146 | 147 | 148 | 149 | 150 | 151 | 152 | 153 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |

| 154 | 155 |
|:-:|:-:|
|   |   |

| 156 | 157 | 158 | 159 | 160 | 161 | 162 | 163 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |

| 164 | 165（写结果） |
|:-:|:--|
|   |   |

### 模块六（166–180）

| 166 | 167 | 168 | 169 | 170 | 171 |
|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |

| 172 | 173 |
|:-:|:-:|
|   |   |

| 174 | 175 | 176 | 177 |
|:-:|:-:|:-:|:-:|
|   |   |   |   |

| 178（写命令作用） | 179–180（填空） |
|:--|:--|
|   |   |

### 模块七（181–195）

| 181 | 182 | 183 | 184 | 185 |
|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |

| 186 | 187 | 188 | 189 | 190 | 191 | 192 | 193 | 194 | 195 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |   |   |

---

## 得分统计（阅卷后填写）

| 模块 | 一 包装/大数 | 二 集合框架 | 三 泛型/Stream | 四 异常/File/IO | 五 正则/时间/算法 | 六 Docker | 七 英语（选做） | 总分 |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 满分 | 39 | 73 | 52 | 64 | 41 | 29 | 20 | 318 |
| 得分 |   |   |   |   |   |   |   |   |

> 必做模块一~六合计 298 分；模块七选做 20 分。做完告诉我实际做了哪几个模块，我按做了的模块折算掌握度，避免没做的部分拉低评分。
