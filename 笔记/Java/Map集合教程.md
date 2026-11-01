# Java Map 集合教程（小白版）

> 用途：学习笔记 · 定位：讲得细、面向刚入门
> 覆盖：集合进阶-01 ~ 20
> **源码部分（13–18）：只讲结论，不抠代码**
> 所有示例和输出都是在本机 JDK 27 上用 jshell 实测跑出来的

---

## 0. 先弄明白 Map 是什么

### 0.1 单列 vs 双列

前面学的 `List`、`Set`，都是**单列集合**——一次存一个东西：

```java
List<String> list = new ArrayList<>();
list.add("张三");
```

**Map 是双列集合**——一次存**一对**东西：**键（key）→ 值（value）**。

```java
Map<String, Integer> map = new HashMap<>();
map.put("张三", 23);      // 键是"张三"，值是 23
```

**生活类比**：
- `List` 像一排座位，按位置坐
- `Set` 像一袋弹珠，只保证不重复
- **`Map` 像一本字典**——**查词（key），得到解释（value）**

### 0.2 一张表记住区别

| | List | Set | **Map** |
|---|---|---|---|
| 存什么 | 一个元素 | 一个元素 | **键值对** |
| 能不能重复 | 元素可重复 | 元素不可重复 | **键不可重复，值可重复** |
| 有没有索引 | 有 | 没有 | **没有索引，用键取** |
| 取元素方式 | `get(下标)` | 遍历 | **`get(键)`** |

**最重要的一句话**：

> **Map 的"键"是唯一的，同一个键重复 put 会覆盖旧值。**

---

# 第一部分：双列集合的特点（01）

## 1. Map 的三个特点

### 特点 1：键唯一，值可重复

```java
Map<String, Integer> map = new HashMap<>();
map.put("张三", 23);
map.put("李四", 23);      // ✅ 值重复没问题
map.put("张三", 30);      // ⚠️ 键重复 → 覆盖，不是新增

map.size();    // 2，不是 3
```

### 特点 2：一个键对应一个值

不能"一个键对应两个值"（那叫多值，得用 `Map<String, List<Integer>>`）。

### 特点 3：键和值都是对象

`Map<int, String>` 编译不过，得写 `Map<Integer, String>`（包装类，老知识点了）。

## 2. Map 的家族

```
             Map（双列集合）
           /      |        \
      HashMap  TreeMap  LinkedHashMap
           |
       (Hashtable、Properties 是老古董，了解即可)
```

| 实现类 | 特点 | 什么时候用 |
|---|---|---|
| **HashMap** | 无序、查找最快 O(1) | **默认选它** |
| **LinkedHashMap** | 按**插入顺序** | 要保序 |
| **TreeMap** | 按**键排序** | 要按 key 排序 |

---

# 第二部分：Map 常用 API（02）

## 3. 最基础的六个方法

```java
Map<String, Integer> map = new HashMap<>();

// 增 / 改：put 是"有就改，没有就加"
map.put("张三", 23);
map.put("李四", 24);

// 查
map.get("张三");            // 23
map.get("赵六");            // null（键不存在）
map.containsKey("李四");    // true
map.containsValue(24);      // true
map.size();                 // 2
map.isEmpty();              // false

// 删
map.remove("李四");         // 返回被删的值 24
map.clear();                // 全清空
```

## 4. 重点：`put` 的返回值（第一个坑）

**`put` 会返回"这个键之前的旧值"，没有旧值返回 `null`：**

```java
Map<String, Integer> map = new HashMap<>();

map.put("张三", 23);        // 返回 null  ← 第一次放，没旧值
map.put("张三", 30);        // 返回 23    ← 覆盖了，返回旧值
```

**这个特性有两个用处：**

### 用法 1：判断是不是"第一次放"

```java
if (map.put(key, value) == null) {
    System.out.println("这是新增");
} else {
    System.out.println("这是覆盖");
}
```

### 用法 2：统计词频（最经典）

```java
Map<String, Integer> count = new HashMap<>();
String text = "apple banana apple cherry banana apple";

for (String word : text.split(" ")) {
    count.put(word, count.getOrDefault(word, 0) + 1);
}
// {apple=3, banana=2, cherry=1}
```

**`getOrDefault(key, 默认值)`**：键存在就返回它的值，不存在返回默认值。**这是统计场景的神器**，比 `if (map.containsKey(...))` 简洁得多。

## 5. 其他常用方法

```java
map.keySet();          // 所有键 → Set
map.values();          // 所有值 → Collection
map.entrySet();        // 所有键值对 → Set<Map.Entry>
map.getOrDefault(k, 0); // 没有就返回默认值
```

实测：

```java
keySet   = [张三, 王五]
values   = [30, 25]
entrySet = [张三=30, 王五=25]
```

---

# 第三部分：三种遍历方式（03–05）

> **这是 Map 最常用的操作，三种都要会。**
> Map 不能像 List 那样用 `for (int i...)`，因为它没有索引。

## 6. 方式一：keySet 遍历（先拿所有键）

**思路：拿到所有键 → 用键去 `get` 值**

```java
Map<String, Integer> map = new HashMap<>();
map.put("张三", 23);
map.put("李四", 24);
map.put("王五", 25);

for (String key : map.keySet()) {
    System.out.println(key + " = " + map.get(key));
}
```

**问题**：每次 `get` 都要**再查一次哈希表**。数据量小无所谓，数据量大时比方式二慢一点。

## 7. 方式二：entrySet 遍历（直接拿键值对）

**思路：一次性拿到所有"键值对对象"**

```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

**`Map.Entry` 是什么？**

可以理解成"**一张对折的卡片**"，打开是两半：`getKey()` 拿键，`getValue()` 拿值。

**这是三种方式里效率最高的**，因为一次遍历就把键和值都拿到了，不用再查一次。

## 8. 方式三：forEach + lambda（最简洁）

```java
map.forEach((key, value) -> {
    System.out.println(key + " = " + value);
});
```

**一行搞定**，推荐日常使用。底层其实就是 `entrySet` 遍历。

## 9. 三种方式怎么选

| 方式 | 写法 | 适合 |
|---|---|---|
| keySet | `for (k : keySet)` + `get(k)` | **只需要键**时 |
| entrySet | `for (entry : entrySet)` | **要键也要值，追求性能** |
| forEach | `map.forEach((k,v) ->...)` | **日常首选，简洁** |

## 10. 坑：遍历时删元素

**和 List 一样**，遍历 `keySet` 时直接 `remove` 会抛异常：

```java
Map<String, Integer> m = new HashMap<>();
for (String k : m.keySet()) {
    if (k.equals("李四")) m.remove(k);    // ❌ ConcurrentModificationException
}
```

**正确的三种删法：**

```java
// 方式 1：entrySet().removeIf（推荐）
map.entrySet().removeIf(e -> e.getKey().equals("李四"));

// 方式 2：先收集要删的键，遍历完再删
List<String> toRemove = new ArrayList<>();
for (String k : map.keySet()) {
    if (k.equals("李四")) toRemove.add(k);
}
map.keySet().removeAll(toRemove);

// 方式 3：迭代器
Iterator<Map.Entry<String,Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    if (it.next().getKey().equals("李四")) it.remove();
}
```

---

# 第四部分：HashMap（06–08）

## 11. 基本使用

```java
Map<String, Integer> map = new HashMap<>();

map.put("张三", 23);
map.put("李四", 24);
map.put("王五", 25);

System.out.println(map);    // {李四=24, 张三=23, 王五=25}   注意：顺序是乱的！
```

**HashMap 的特点：**
- **无序**（顺序不可控，不同 JDK 版本可能不同）
- **查找最快**（O(1)）
- 允许 **null 键** 和 **null 值**

```java
map.put(null, "空键");      // ✅ 可以
map.put("键", null);        // ✅ 可以
```

## 12. 大坑：用自定义对象当 key

**HashMap 找键靠的是 `hashCode` + `equals`**（和 HashSet 完全一样的原理）。

**如果你用自己写的类当 key，却不重写这两个方法，就"取不出来"：**

```java
class Student {
    String name;
    int age;
    Student(String n, int a) { name = n; age = a; }
}

Map<Student, String> map = new HashMap<>();
map.put(new Student("张三", 23), "北京");

// 用"内容相同"的新对象去 get
map.get(new Student("张三", 23));    // null  ← 取不到！
```

**为什么？** 两个 `new` 出来的对象，默认 `hashCode` 不同、默认 `equals` 比地址，HashMap 认为它们是两个不同的键。

### 12.1 解决：重写 hashCode + equals

```java
class Student {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Student s)) return false;
        return age == s.age && Objects.equals(name, s.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

map.get(new Student("张三", 23));    // "北京"  ✅ 取到了
```

**实测对照：**

| 做法 | `get` 结果 |
|---|---|
| 不重写 | `null`（取不到） |
| 重写 `hashCode` + `equals` | `"北京"`（✅ 取到） |

> 这和第 8 节（Set）的结论一模一样：**重写 equals 必须同时重写 hashCode。**

## 13. 练习一：统计单词频率

**需求**：一句话里每个单词出现几次。

```java
String text = "apple banana apple cherry banana apple";
Map<String, Integer> count = new HashMap<>();

for (String word : text.split(" ")) {
    count.put(word, count.getOrDefault(word, 0) + 1);
}

System.out.println(count);    // {apple=3, banana=2, cherry=1}
```

**执行过程拆解：**

| 遇到 | getOrDefault | 加 1 后 put |
|---|---|---|
| apple | 0 | apple=1 |
| banana | 0 | banana=1 |
| apple | 1 | apple=2 |
| cherry | 0 | cherry=1 |
| banana | 1 | banana=2 |
| apple | 2 | apple=3 |

**这是 Map 最经典的用法**，一定要会。

## 14. 练习二：找最高分

**需求**：一堆学生成绩，找出分数最高的。

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("张三", 85);
scores.put("李四", 92);
scores.put("王五", 78);

String topName = null;
int topScore = Integer.MIN_VALUE;

for (Map.Entry<String, Integer> e : scores.entrySet()) {
    if (e.getValue() > topScore) {
        topScore = e.getValue();
        topName = e.getKey();
    }
}

System.out.println("最高分: " + topName + " " + topScore);   // 最高分: 李四 92
```

**为什么 `topScore` 用 `Integer.MIN_VALUE`？**
防止所有分数都是负数时，初始值 0 比它们都大，导致一个都选不出来。

---

# 第五部分：LinkedHashMap（09）

## 15. 就多一个功能：保序

```java
Map<String, Integer> hash = new HashMap<>();
Map<String, Integer> linked = new LinkedHashMap<>();

for (String k : new String[]{"c","a","b"}) {
    hash.put(k, 1);
    linked.put(k, 1);
}

System.out.println(hash);      // {a=1, b=1, c=1}    顺序不可控
System.out.println(linked);    // {c=1, a=1, b=1}    ✅ 按插入顺序
```

**除此之外和 HashMap 完全一样**：去重规则、null、底层哈希表。

**什么时候用**：需要"记着谁先谁后"的时候，比如记录用户的访问顺序。

---

# 第六部分：TreeMap（10–12）

## 16. TreeMap 的特点：按键排序

```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(3, "三");
tm.put(1, "一");
tm.put(2, "二");

System.out.println(tm);    // {1=一, 2=二, 3=三}   ✅ 按键自动排序
```

**注意：TreeMap 只对"键"排序，不对"值"排序。**

## 17. TreeMap 的实用方法

```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(1, "一"); tm.put(2, "二"); tm.put(3, "三");

tm.firstKey();          // 1        最小键
tm.lastKey();           // 3        最大键
tm.headMap(2);          // {1=一}        小于 2 的部分
tm.tailMap(2);          // {2=二, 3=三}  大于等于 2 的部分
tm.lowerEntry(2);       // 1=一      严格小于 2 的最大键
tm.higherEntry(2);      // 3=三      严格大于 2 的最小键
```

## 18. 坑 1：key 必须能比较

```java
class NoComparable {
    String n;
    NoComparable(String n) { this.n = n; }
}

TreeMap<NoComparable, String> bad = new TreeMap<>();
bad.put(new NoComparable("a"), "x");
bad.put(new NoComparable("b"), "y");
// ❌ ClassCastException
```

**和 TreeSet 一样**：要么实现 `Comparable`，要么在创建 TreeMap 时传 `Comparator`。

## 19. 坑 2：compareTo 返回 0 会"覆盖"（和 TreeSet 不一样！）

```java
class Person implements Comparable<Person> {
    String name;
    int age;

    @Override
    public int compareTo(Person o) {
        return this.age - o.age;      // 只比年龄
    }
}

TreeMap<Person, String> tm = new TreeMap<>();
tm.put(new Person("张三", 25), "第一次");
tm.put(new Person("李四", 25), "第二次");    // 同龄

System.out.println(tm);
// {张三(25)=第二次}
// 只剩下一个键，值是"第二次"
```

**和 TreeSet 的关键区别：**

| | TreeSet | TreeMap |
|---|---|---|
| 比较返回 0 时 | **丢弃新元素** | **覆盖 value** |
| 结果 | 元素不见了 | 键还在，值变了 |

**记忆**：`TreeSet` 是"不要这个元素"，`TreeMap` 是"更新这个键的值"。

## 20. 练习：按 key 排序

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("张三", 85);
scores.put("李四", 92);
scores.put("王五", 78);

// 直接丢进 TreeMap
Map<String, Integer> sorted = new TreeMap<>(scores);

System.out.println(sorted);   // {张三=85, 李四=92, 王五=78}
```

**注意**：中文按 **Unicode 编码**排，不是按拼音。想要按拼音得自己写比较器。

## 21. 练习：按 value 排序（Map 本身做不到）

**关键理解：TreeMap 只能按 key 排，不能按 value 排。**

**要按 value 排序，得转成 List 再排：**

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("张三", 85);
scores.put("李四", 92);
scores.put("王五", 78);
scores.put("赵六", 92);

// 1. 把 entrySet 转成 List
List<Map.Entry<String, Integer>> entries = new ArrayList<>(scores.entrySet());

// 2. 按 value 降序，value 相同按 key
entries.sort(
    Map.Entry.<String, Integer>comparingByValue().reversed()
             .thenComparing(Map.Entry.comparingByKey())
);

System.out.println(entries);
// [李四=92, 赵六=92, 张三=85, 王五=78]
```

**这是 Map 排序的标准套路**，记住三步：**转 List → 写比较器 → sort**。

---

# 第七部分：HashMap 源码（13–16，简略版）

> **只讲结论，面试问到能答上来就行。**

## 22. 五个关键数字

| 项 | 值 | 说明 |
|---|---|---|
| 默认容量 | **16** | 初始化时数组长度 |
| 负载因子 | **0.75** | 存到 75% 就扩容 |
| 扩容倍数 | **2 倍** | 16 → 32 → 64 |
| 链表转红黑树 | 链表长 **≥ 8** 且数组长 **≥ 64** | 两个条件都要满足 |
| 红黑树退回链表 | 长度 **≤ 6** | |

## 23. 一次 put 发生了什么

```
① 算 key 的 hash：hashCode 再经过一次"扰动"（高位异或），让分布更均匀
② 用 hash 对数组长度取模，找到桶的位置
③ 桶是空的 → 直接放
④ 桶里有东西 → equals 逐个比
      - 相等 → 覆盖 value（这就是 put 返回旧值的原因）
      - 都不等 → 挂到链表尾部
⑤ 链表太长 → 转红黑树
⑥ size 超过 容量×0.75 → 扩容 2 倍，重新分配所有元素
```

## 24. 为什么用 2 的幂做容量

因为 `hash % 数组长度` 在数组长度是 2 的幂时，**等价于 `hash & (长度-1)`**，位运算比取模快得多。

**这就是为什么扩容总是"翻倍"——这样能保证长度一直是 2 的幂。**

## 25. 两个实用结论

1. **能预估元素数量时，用 `new HashMap<>(预计数量)`**，少扩容几次
2. **HashMap 不是线程安全的**，多线程用 `ConcurrentHashMap`

---

# 第八部分：TreeMap 源码（17–18，简略版）

## 26. 一句话结论

**TreeMap 底层是红黑树**（和第 10 节学的完全一样）。

```
- 每个节点是 key-value 对
- 左小右大（按 key 比）
- 自动保持大致平衡
- 查找/插入/删除 O(log n)
- 遍历时中序遍历 → 天然就是有序的
```

**为什么 TreeMap 遍历是有序的？** 因为红黑树的中序遍历（左 → 根 → 右）就是从小到大的顺序。

---

# 第九部分：可变参数（19）

## 27. 语法

```java
public static int sum(int... nums) {
    int total = 0;
    for (int n : nums) total += n;
    return total;
}
```

**`int...` 表示"0 个或多个 int"**，在方法内部它就是一个 `int[]`。

**调用时三种都行：**

```java
sum();                 // 0
sum(1);                // 1
sum(1, 2, 3);          // 6
sum(new int[]{1,2,3,4}); // 10   ← 直接传数组也行
```

## 28. 可变参数的规则

| 规则 | 说明 |
|---|---|
| 只能有一个 | 一个方法只能有一个可变参数 |
| 必须放最后 | `void f(String s, int... nums)` ✅<br>`void f(int... nums, String s)` ❌ |
| 本质是数组 | 方法内部 `nums` 就是 `int[]` |
| 可以是任意类型 | `String...`、`Object...` 都行 |

**示例：**

```java
// 常见用法：一次添加多个元素
Collections.addAll(list, "a", "b", "c");

// 自定义：拼接任意多个字符串
String join(String sep, String... parts) {
    return String.join(sep, parts);
}
```

## 29. 坑：可变参数和数组重载会冲突

```java
void f(int... nums) { }
void f(int[] nums) { }    // ❌ 编译错误：两者是同一个方法签名
```

因为可变参数本质上就是数组，所以不能同时存在。

---

# 第十部分：Collections 工具类（20）

## 30. 它是什么

**`Collections` 是操作集合的工具类**（注意有 **s**，和接口 `Collection` 不是一回事）。

**类比**：`Arrays` 管数组，`Collections` 管集合。

## 31. 最常用的方法（全部实测）

```java
List<Integer> list = new ArrayList<>(List.of(3,1,4,1,5,9,2,6));

Collections.sort(list);            // 升序 → [1,1,2,3,4,5,6,9]
Collections.reverse(list);         // 反转
Collections.swap(list, 0, 7);      // 交换两个位置
Collections.max(list);             // 最大值 9
Collections.min(list);             // 最小值 1
Collections.frequency(list, 1);    // 1 出现了几次 → 2
Collections.shuffle(list);         // 随机打乱（抽奖/洗牌用）
Collections.replaceAll(list, 1, 100);   // 把所有的 1 换成 100
Collections.addAll(list, "a","b","c");  // 一次加多个
Collections.binarySearch(list, 3);      // 二分查找（必须先排序）
```

**实测输出：**

```
原列表: [3, 1, 4, 1, 5, 9, 2, 6]
sort: [1, 1, 2, 3, 4, 5, 6, 9]
reverse: [9, 6, 5, 4, 3, 2, 1, 1]
swap(0,7): [1, 6, 5, 4, 3, 2, 1, 9]
max=9 min=1
frequency(1)=2
shuffle: [5, 2, 9, 4, 6, 1, 1, 3]
replaceAll(1->100): [5, 2, 9, 4, 6, 100, 100, 3]
binarySearch(3) = 1
```

## 32. Collections vs List 自带方法

**很多 Collections 的方法，List 自己也有：**

```java
list.sort(null);              // 等价于 Collections.sort(list)
list.sort(Comparator...);     // 更推荐
list.forEach(...);            // 遍历
list.removeIf(...);           // 条件删除
```

**建议**：
- **排序用 `list.sort()`**（JDK 8+ 更自然）
- **`Collections` 用来做那些 List 没有的**：`shuffle`、`frequency`、`swap`、`addAll`

## 33. 其他常用

```java
Collections.emptyList();           // 空列表（不可变）
Collections.singletonList("a");    // 只有一个元素的列表
Collections.unmodifiableList(list); // 变成不可变
Collections.nCopies(3, "x");       // ["x","x","x"]
```

---

# 第十一部分：速查与避坑

## 34. 一页速查表

### Map 常用操作

| 想要 | 写法 |
|---|---|
| 加/改 | `map.put(k, v)`（返回旧值） |
| 查 | `map.get(k)` / `getOrDefault(k, 默认)` |
| 有没有键 | `map.containsKey(k)` |
| 删 | `map.remove(k)` |
| 多少个 | `map.size()` |
| 所有键 | `map.keySet()` |
| 所有值 | `map.values()` |
| 所有键值对 | `map.entrySet()` |

### 三种遍历

| 方式 | 写法 |
|---|---|
| keySet | `for (k : map.keySet()) { map.get(k) }` |
| entrySet | `for (e : map.entrySet()) { e.getKey(); e.getValue() }` |
| forEach | `map.forEach((k,v) -> ...)` |

### 三个实现类

| 需求 | 用 |
|---|---|
| 默认（要快） | `HashMap` |
| 要保插入顺序 | `LinkedHashMap` |
| 要按 key 排序 | `TreeMap` |

### JDK 8+ 新方法

| 方法 | 作用 |
|---|---|
| `getOrDefault(k, d)` | 没有就用默认值 |
| `putIfAbsent(k, v)` | 没有才放 |
| `merge(k, v, (a,b)->a+b)` | 存在就合并 |
| `compute(k, (k,v)->...)` | 计算新值 |
| `computeIfAbsent(k, k->new ...)` | 没有就创建 |

## 35. 常见错误速查（照着躲）

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| 自定义对象当 key 不重写 `hashCode`/`equals` | 两个都重写 | `get` 永远返回 null |
| 遍历 `keySet` 时 `remove` | `entrySet().removeIf(...)` | `ConcurrentModificationException` |
| 以为 `put` 是"新增" | `put` 是"有就改，没有就加" | 数据被覆盖 |
| TreeMap 的 key 没 Comparable | 实现 Comparable 或用 Comparator | `ClassCastException` |
| TreeMap 的 `compareTo` 只比一个字段 | 补全字段 | **值被意外覆盖** |
| 用 `TreeMap` 想按 value 排序 | 转 List 再排序 | 排出来还是按 key |
| 以为 HashMap 有序 | 用 LinkedHashMap | 顺序随机 |
| `map.get(k)` 返回 null 就以为没这个键 | 用 `containsKey` 区分 | 值本身是 null 时判断错误 |
| `Map<int,String>` | `Map<Integer,String>` | 编译错误 |

## 36. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.util.*;

// ① 基础操作 + put 返回值
Map<String,Integer> m = new HashMap<>();
System.out.println(m.put("a", 1));    // null
System.out.println(m.put("a", 2));    // 1

// ② 三种遍历
m.put("b", 3);
for (String k : m.keySet()) System.out.println("keySet: " + k + "=" + m.get(k));
for (Map.Entry<String,Integer> e : m.entrySet()) System.out.println("entrySet: " + e);
m.forEach((k,v) -> System.out.println("forEach: " + k + "=" + v));

// ③ 统计词频
Map<String,Integer> count = new HashMap<>();
for (String w : "a b a c a".split(" ")) count.put(w, count.getOrDefault(w,0)+1);
System.out.println(count);

// ④ 三个实现类的顺序
Map<String,Integer> h = new HashMap<>(), l = new LinkedHashMap<>(), t = new TreeMap<>();
for (String k : new String[]{"c","a","b"}) { h.put(k,1); l.put(k,1); t.put(k,1); }
System.out.println("HashMap=" + h + " Linked=" + l + " Tree=" + t);

// ⑤ Collections 工具
List<Integer> nums = new ArrayList<>(List.of(3,1,2,2));
Collections.sort(nums);
System.out.println("sort=" + nums + " freq(2)=" + Collections.frequency(nums, 2));
```

---

## 记忆口诀

> **Map 是键值对，键唯一值可重复，键重复 put 会覆盖；**
> **遍历三招：keySet 拿键再 get、entrySet 一次拿全、forEach 最简洁；**
> **自定义对象当 key，hashCode + equals 都得重写；**
> **HashMap 快但无序，LinkedHashMap 保序，TreeMap 按 key 排；**
> **TreeMap 比较返回 0 是覆盖 value，TreeSet 是丢元素——别搞混；**
> **Collections 管集合，和 Arrays 管数组一个道理。**
