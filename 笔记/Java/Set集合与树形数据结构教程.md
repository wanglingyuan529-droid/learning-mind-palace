# Java Set 集合与树形数据结构教程（小白版）

> 用途：学习笔记 · 定位：讲得细、面向刚入门；只讲"技"
> 覆盖：集合进阶-10 ~ 15
> 数据结构部分（二叉树/平衡树/红黑树）：**讲清楚"是什么、为什么需要、怎么工作"，不抠旋转的每一步和红黑树的完整代码**
> 所有示例和输出都是在本机 JDK 27 上用 jshell 实测跑出来的

---

## 0. 先看一眼 Set 家族

### 0.1 三个 Set 的共同点

**Set 的核心就一句话：不能装重复的东西。**

```java
Set<String> set = new HashSet<>();
set.add("a");
set.add("a");     // 加第二次，加不进去（返回 false）
System.out.println(set);   // [a]
```

### 0.2 三个 Set 的区别（先背这张表）

| 实现类 | 有序吗 | 底层结构 | 查找速度 | 什么时候用 |
|---|---|---|---|---|
| **HashSet** | ❌ 不保证 | 哈希表 | **最快 O(1)** | 只需要去重，不关心顺序 |
| **LinkedHashSet** | ✅ 按**添加顺序** | 哈希表 + 链表 | 快 | 要**去重又要保序** |
| **TreeSet** | ✅ **自动排序** | 红黑树 | 较慢 O(log n) | 要**去重又要排序** |

实测：

```java
HashSet:       [a, b, c]      ← 顺序不可控
LinkedHashSet: [c, a, b]      ← 严格按添加顺序（先加 c 就是 c 在前）
TreeSet:       [a, b, c]      ← 自动按大小排
```

**一句话选择：**

```
只去重 → HashSet
去重 + 保持添加顺序 → LinkedHashSet
去重 + 自动排序 → TreeSet
```

---

# 第一部分：哈希值（hashCode）

## 1. 什么是哈希值

**哈希值就是一个整数**，由对象的"内容"算出来，用来表示"这个对象大概长什么样"。

```java
"abc".hashCode();          // 96354
Integer.valueOf(123).hashCode();   // 123
Boolean.valueOf(true).hashCode();  // 1231
```

**最重要的特点：同一个值，算出来永远是同一个数。**

```java
"abc".hashCode();   // 96354
"abc".hashCode();   // 96354   每次都一样
```

### 1.1 哈希值的作用（先建立直觉）

**哈希值 = 对象的"身份证号"，用来快速找到它放在哪。**

想象一个图书馆：
- 你有 100 万本书
- 如果一本本找（线性查找），要翻 100 万次
- 但如果每本书有个编号，你按编号直接走到那个书架（哈希查找），一次就到

**哈希值就是那个"编号"。**

### 1.2 String 的 hashCode 是怎么算的

```java
String s = "abc";
int h = 0;
for (int i = 0; i < s.length(); i++) {
    h = 31 * h + s.charAt(i);
}
// h = 96354，和 JDK 算出来的一样
```

**记住结论就行**：JDK 用 `31 * 之前的结果 + 当前字符的编码`，逐位算出来。

**为什么是 31？** 31 是质数，能让哈希值分布更均匀，减少冲突。这是经验值，不用深究。

---

# 第二部分：HashSet（集合进阶-13，重点）

## 2. HashSet 长什么样

```java
Set<String> set = new HashSet<>();
set.add("a");
set.add("b");
set.add("a");      // 重复，加不进去

set.size();        // 2
set.contains("a"); // true
set.remove("a");
```

**它就是个"去重版"的集合**，别的没什么特别的。

## 3. 去重原理（这节是核心）

### 3.1 一句话：先比 hashCode，再比 equals

`HashSet.add()` 内部做了这些事：

```
1. 算要添加元素的 hashCode
2. 用 hashCode 算出它在数组的哪个位置（哪个"桶"）
3. 那个桶是空的？ → 直接放进去
4. 桶里已经有东西？ → 用 equals 和桶里每个元素比
      - 有一个 equals 为 true → 重复了，不加
      - 全都 equals 为 false → 挂到链表后面
```

**关键顺序：先 hashCode 定位，再用 equals 确认。** 两个步骤缺一不可。

### 3.2 坑 1：自定义对象不重写，去重直接失效

**这是个实测出来的坑，很多人第一次写到这里都会踩：**

```java
class StudentA {
    String name;
    int age;
    StudentA(String n, int a) { name = n; age = a; }
}

Set<StudentA> set = new HashSet<>();
set.add(new StudentA("张三", 23));
set.add(new StudentA("张三", 23));   // 同样的内容

set.size();   // 2  ← 没去掉！
```

**为什么？** 因为你没重写 `hashCode` 和 `equals`，Java 用的是 `Object` 的默认实现：
- 默认 `hashCode()`：根据**内存地址**算，两个不同对象一定不同
- 默认 `equals()`：比较**是不是同一个对象**（`==`）

所以两个"内容相同"的新对象，在 HashSet 看来是完完全全不同的两个东西。

### 3.3 坑 2：**只重写 equals 还不够**（最坑）

```java
class StudentB {
    String name;
    int age;

    // 只重写了 equals，没重写 hashCode
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof StudentB b)) return false;
        return b.age == age && Objects.equals(b.name, name);
    }
}

Set<StudentB> set = new HashSet<>();
set.add(new StudentB("张三", 23));
set.add(new StudentB("张三", 23));

set.size();   // 2  ← 还是没去掉！
```

**明明 `equals` 返回 true 了啊？** 实测给你看：

```java
b1.equals(b2);      // true     ← equals 确实认为相等
b1.hashCode();      // 1775282465
b2.hashCode();      // 1147985808   ← 但 hashCode 不同！
```

**因为 hashCode 不同，这两个对象被分到了不同的桶里，HashSet 根本不会拿它们去比 equals。**

**记住 Java 的死规矩：**

> **重写 equals 必须同时重写 hashCode。**
> 两个对象 equals 相等 → hashCode 必须相等。
> （反过来不要求：hashCode 相等的对象，equals 可以不等）

### 3.4 正确写法：两个都重写

```java
class StudentC {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof StudentC c)) return false;
        return age == c.age && Objects.equals(name, c.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);    // 用 Objects.hash 最省事
    }
}

Set<StudentC> set = new HashSet<>();
set.add(new StudentC("张三", 23));
set.add(new StudentC("张三", 23));

set.size();   // 1  ✅ 终于去重了
```

**三种情况实测对照：**

| 做法 | 结果 |
|---|---|
| 什么都不重写 | size = 2（没去重） |
| 只重写 `equals` | size = 2（**还是没去重**） |
| `equals` + `hashCode` 都重写 | size = 1（✅ 去重成功） |

### 3.5 在 IDEA 里怎么快速生成

在类里按 `Alt + Insert`（或右键 → Generate）→ 选 `equals() and hashCode()` → 全选字段 → 确定。

**别手写**，容易漏字段。

## 4. HashSet 的底层结构（简略版）

> 知道这几句话就够，不用看源码。

```
数组 + 链表 + 红黑树
```

**工作流程：**

```
① 算 hashCode → 找到数组的下标
② 桶是空的 → 直接放
③ 桶里有东西 → equals 挨个比
      - 有相等的 → 不加（重复）
      - 都不等 → 挂到链表末尾
④ 链表太长（≥ 8）且数组够大（≥ 64）→ 链表转成红黑树
```

**为什么要转红黑树？** 因为链表太长时，查找要一个个比（O(n)），转成红黑树后变成 O(log n)。

**默认容量 16，负载因子 0.75**（存到 12 个就扩容到 32）。这些是源码细节，记住数字即可。

## 5. HashSet 的其他特点

### 5.1 可以放 null

```java
Set<String> set = new HashSet<>();
set.add(null);      // ✅ 可以，而且只能放一个
```

### 5.2 无序，但"看起来像有序"

```java
Set<String> set = new HashSet<>();
set.add("c"); set.add("a"); set.add("b");
System.out.println(set);    // [a, b, c]   ← 看着像有序，但别依赖
```

**为什么看着有序？** 因为 `String` 的 hashCode 算出来的位置刚好是 a、b、c 的顺序。换个数据就乱了。

**结论：`HashSet` 的打印顺序是巧合，绝不要依赖它。**

### 5.3 没有索引

```java
set.get(0);    // ❌ 没有这个方法
```

因为底层是哈希表，元素不在连续的位置上，根本没有"第几个"的概念。

**要按位置取，用 `List`，不能用 `Set`。**

---

# 第三部分：LinkedHashSet（集合进阶-13）

## 6. LinkedHashSet 就加了一个功能：保序

```java
Set<String> hash = new HashSet<>();
hash.add("c"); hash.add("a"); hash.add("b");
System.out.println(hash);        // [a, b, c]   顺序不可控

Set<String> linked = new LinkedHashSet<>();
linked.add("c"); linked.add("a"); linked.add("b");
System.out.println(linked);      // [c, a, b]   ✅ 严格按添加顺序
```

**除此之外，和 HashSet 完全一样**：去重规则、null、底层哈希表，全都一样。

### 6.1 什么时候用它

| 场景 | 用哪个 |
|---|---|
| 只要去重 | HashSet（快一点） |
| 去重 + 想知道"谁先谁后" | **LinkedHashSet** |
| 去重 + 要按大小排 | TreeSet |

**典型用途**：记录用户第一次访问的页面顺序、去重日志但保持时间顺序。

---

# 第四部分：树形数据结构（集合进阶-10 ~ 12，简略版）

> 这一部分**知道"是什么、为什么需要、大概怎么工作"就行**。
> 因为 `TreeSet` 和 `HashSet` 的底层都用到了树，不懂树就没法理解它们为什么快。

## 7. 二叉树

### 7.1 是什么

**每个节点最多有两个"孩子"的树。**

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

**术语（扫一眼就行）：**

| 术语 | 意思 |
|---|---|
| 根节点 | 最上面那个（A） |
| 父节点 / 子节点 | B 是 A 的子，A 是 B 的父 |
| 叶子节点 | 没孩子的（D、E、F） |
| 左子树 / 右子树 | 左边那一坨、右边那一坨 |
| 高度 | 有几层 |

**二叉树本身没什么用**，真正有用的是它的"排序版"——二叉查找树。

## 8. 二叉查找树（BST）

### 8.1 规则（就一条）

> **每个节点：左边所有都比它小，右边所有都比它大。**

```
        8
       / \
      3   10
     / \    \
    1   6    14
```

校验一下：
- 8 的左边（3、1、6）全都 < 8 ✅
- 8 的右边（10、14）全都 > 8 ✅
- 3 的左边 1 < 3，右边 6 > 3 ✅

### 8.2 查找过程：每次砍掉一半

```
找 6：
  8 → 6 < 8，往左走
  3 → 6 > 3，往右走
  6 → 找到了
```

**每次比较都能排除一半，所以查找是 O(log n)，和二分查找一个道理。**

### 8.3 添加过程

```
加 5：
  8 → 5 < 8，往左
  3 → 5 > 3，往右
  6 → 5 < 6，往左
  6 没有左孩子 → 5 放这里
```

### 8.4 致命缺陷：会"退化"成链表

**如果按顺序往二叉查找树里放数据**：

```
依次放入：1, 2, 3, 4, 5

结果：
1
 \
  2
   \
    3
     \
      4
       \
        5
```

**变成了一条斜线！** 这时候查找 5 要走 5 步，和链表一样慢（O(n)）。

**所以需要"平衡二叉树"——让树尽量保持"矮胖"，别长成"高瘦"。**

## 9. 平衡二叉树（AVL 树）

### 9.1 规则

> **每个节点的左右子树高度差不超过 1。**

```
     ✅ 平衡               ❌ 不平衡
       4                      1
      / \                      \
     2   6                      2
    / \ / \                      \
   1  3 5  7                      3
                                    \
                                     4
```

### 9.2 怎么保持平衡：旋转

当插入数据导致"一边特别长"时，**通过旋转把树掰正**。

**两种基本旋转：**

**左旋**（右边太重时）：

```
     A                    B
      \                  / \
       B      →         A   C
        \
         C
```

**右旋**（左边太重时）：

```
       C                B
      /                / \
     B       →        A   C
    /
   A
```

**实际是四种情况：**
- LL（左左）→ 右旋
- RR（右右）→ 左旋
- LR（左右）→ 先左旋再右旋
- RL（右左）→ 先右旋再左旋

> **这四种情况的名字和步骤不用记**，知道"不平衡就旋转，旋转是为了让树变矮"就够了。JDK 里没直接用 AVL 树（用的是红黑树）。

## 10. 红黑树（TreeSet / HashSet 底层）

### 10.1 为什么不用 AVL 树

AVL 树太严格（高度差 ≤ 1），**每次插入删除都可能要旋转好几次**，维护成本高。

**红黑树放松了要求**：不追求"完全平衡"，只保证"大致平衡"，查找依然是 O(log n)，但插入删除时旋转次数少多了。

### 10.2 红黑规则（5 条，看一眼）

1. 每个节点不是红就是黑
2. **根节点必须是黑**
3. **叶子节点（NIL）都是黑**
4. **红节点的子节点必须是黑**（不能有连续两个红）
5. **从任一节点到它所有叶子，经过的黑节点数量相同**

**记住核心思想就行**：

> **靠"红黑两种颜色"约束树的形状，让最长路径不超过最短路径的 2 倍，从而保证 O(log n)。**

### 10.3 添加节点后怎么调整

新加的节点**默认是红色**，然后看情况调整：

```
情况 1：父节点是黑的 → 直接插入，不用调
情况 2：父节点是红的 → 违反"红红不能相邻"，要调整
         ├─ 叔叔节点是红的 → 变色（父、叔变黑，祖父变红）
         └─ 叔叔节点是黑的 → 旋转（左旋/右旋）+ 变色
```

**知道"插进去可能要变色和旋转"就够**，具体的 4 种旋转情况不用手推。

### 10.4 一句话总结红黑树

> **红黑树 = 一种"自动保持大致平衡"的二叉查找树，查找 O(log n)，插入删除代价比 AVL 小，所以 JDK 选它做 TreeSet/HashMap（链表过长的部分）的底层。**

---

# 第五部分：TreeSet 自然排序（集合进阶-14，重点）

## 11. TreeSet 是什么

**TreeSet = 会自动排序 + 自动去重的 Set。**

```java
TreeSet<Integer> ts = new TreeSet<>();
ts.add(5); ts.add(2); ts.add(9); ts.add(1); ts.add(5);

System.out.println(ts);   // [1, 2, 5, 9]   自动排序 + 去重
```

## 12. 为什么它能排序：Comparable 接口

`TreeSet` 放入元素时，会调用元素的 `compareTo()` 方法决定放左边还是右边。

所以元素必须**实现 `Comparable` 接口**。

**好消息：Java 自带的类基本都实现了。**

```java
Integer、String、Double、Character、Date  // 都自带排序规则
```

| 类型 | 排序规则 |
|---|---|
| 数字 | 从小到大 |
| String | 按字典序（ASCII 码逐个比） |
| Character | 按字符编码 |

## 13. 坑 1：放自己写的类，直接抛异常

```java
class User {
    String name;
    User(String n) { name = n; }
}

TreeSet<User> ts = new TreeSet<>();
ts.add(new User("a"));
ts.add(new User("b"));
```

**实测结果：**

```
ClassCastException: User cannot be cast to java.lang.Comparable
```

**原因**：TreeSet 不知道"两个 User 谁大谁小"，没法排序，直接报错。

## 14. 解决：让类实现 Comparable

```java
class Person implements Comparable<Person> {
    String name;
    int age;

    Person(String n, int a) { name = n; age = a; }

    @Override
    public int compareTo(Person o) {
        return this.age - o.age;      // 按年龄升序
    }

    @Override
    public String toString() { return name + "(" + age + ")"; }
}

TreeSet<Person> tp = new TreeSet<>();
tp.add(new Person("张三", 25));
tp.add(new Person("李四", 20));
tp.add(new Person("王五", 30));

System.out.println(tp);
// [李四(20), 张三(25), 王五(30)]   按年龄排好了
```

### 14.1 compareTo 的返回值规则

| 返回值 | 含义 | TreeSet 怎么处理 |
|---|---|---|
| 正数 | 当前对象 **大于** 参数 | 排后面 |
| 0 | **相等** | **当作重复，不加** |
| 负数 | 当前对象 **小于** 参数 | 排前面 |

**写法模板：**

```java
// 升序：this - o
return this.age - o.age;

// 降序：o - this
return o.age - this.age;

// 更安全的写法（防止溢出）
return Integer.compare(this.age, o.age);
```

**注意**：`this.age - o.age` 在数值极端时可能溢出，**推荐用 `Integer.compare`**。

## 15. 坑 2：compareTo 返回 0 = 被当成重复丢掉（超坑）

**这是 TreeSet 最反直觉的地方，实测：**

```java
class Dog implements Comparable<Dog> {
    String name;
    int age;

    Dog(String n, int a) { name = n; age = a; }

    @Override
    public int compareTo(Dog o) {
        return this.age - o.age;      // ⚠️ 只比较年龄，没比名字
    }

    @Override
    public String toString() { return name + "(" + age + ")"; }
}

TreeSet<Dog> dogs = new TreeSet<>();
dogs.add(new Dog("大黄", 3));
dogs.add(new Dog("小白", 3));      // 年龄也是 3
dogs.add(new Dog("小黑", 5));

System.out.println(dogs);
// [大黄(3), 小黑(5)]   ← 小白不见了！！
```

**为什么会这样？**

`TreeSet` 判断"重复"用的是 **`compareTo()` 返回 0**，而不是 `equals()`。
`小白` 和 `大黄` 年龄都是 3，`compareTo` 返回 0，就被当成了"同一个元素"，**直接丢弃**。

**这是 TreeSet 和 HashSet 去重规则的根本区别：**

| | 判断重复依据 |
|---|---|
| HashSet | `hashCode` + `equals` |
| **TreeSet** | **`compareTo` / `compare` 返回 0** |

### 15.1 怎么避免

**`compareTo` 里要把"所有能区分元素的字段"都比一遍。**

```java
@Override
public int compareTo(Dog o) {
    int r = this.age - o.age;
    if (r != 0) return r;
    return this.name.compareTo(o.name);    // 年龄相同再比名字
}
```

改完以后三只狗都在了。

**记忆**：**写 `compareTo` 时把"主排序字段"之外的字段也兜上**，否则数据会莫名其妙地"消失"。

---

# 第六部分：TreeSet 比较器排序（集合进阶-15）

## 16. 为什么还需要比较器

`Comparable` 的问题是：**一个类只能有一种排序规则**（写在类里，改不了）。

但现实中经常需要多种排序：
- 学生既能按分数排，也能按年龄排
- 商品既能按价格排，也能按销量排

**比较器（Comparator）让你在创建 TreeSet 时才决定排序规则，不改类。**

## 17. Comparator 的写法

### 17.1 三种写法（从老到新）

```java
// 写法 1：匿名内部类（最啰嗦）
TreeSet<String> t1 = new TreeSet<>(new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
});

// 写法 2：lambda（推荐）
TreeSet<String> t2 = new TreeSet<>((a, b) -> a.length() - b.length());

// 写法 3：Comparator 工具方法（最推荐，不容易写错）
TreeSet<String> t3 = new TreeSet<>(Comparator.comparingInt(String::length));
```

**三种效果完全一样。**

### 17.2 实战：按字符串长度排序

```java
TreeSet<String> byLen = new TreeSet<>(Comparator.comparingInt(String::length));
byLen.add("banana");
byLen.add("apple");
byLen.add("kiwi");

System.out.println(byLen);
// [kiwi, apple, banana]   按长度从短到长
```

### 17.3 多条件排序：先长度，再字母

```java
TreeSet<String> multi = new TreeSet<>(
    Comparator.comparingInt(String::length)
              .thenComparing(Comparator.naturalOrder())
);

multi.add("bb"); multi.add("a"); multi.add("ccc"); multi.add("dd");

System.out.println(multi);
// [a, bb, dd, ccc]
// 长度：1, 2, 2, 3
// 长度相同的按字母：bb 在 dd 前面
```

### 17.4 降序排序

```java
// 方式 1：Comparator.reverseOrder()
TreeSet<Integer> desc = new TreeSet<>(Comparator.reverseOrder());

// 方式 2：自己写
TreeSet<Integer> desc2 = new TreeSet<>((a, b) -> b - a);

// 方式 3：先自然序再反转
TreeSet<Integer> desc3 = new TreeSet<>(Comparator.<Integer>naturalOrder().reversed());
```

## 18. 坑 3：比较器返回 0 也当重复

**和 `compareTo` 一样，比较器返回 0 也会被当成重复丢掉：**

```java
TreeSet<String> lenOnly = new TreeSet<>(Comparator.comparingInt(String::length));
lenOnly.add("aa");
lenOnly.add("bb");
lenOnly.add("cc");

System.out.println(lenOnly);
// [aa]   ← 三个长度都是 2，只剩第一个！
```

**这是个很容易踩的坑**：你只想按长度排，结果不同内容的数据被"去重"掉了。

**解决**：比较器里补上"第二比较条件"：

```java
TreeSet<String> fix = new TreeSet<>(
    Comparator.comparingInt(String::length)
              .thenComparing(Comparator.naturalOrder())
);
```

## 19. Comparable vs Comparator 对比

| | Comparable | Comparator |
|---|---|---|
| 写在哪 | **类内部**（`implements Comparable`） | **外部**（作为参数传进去） |
| 方法名 | `compareTo(T o)` | `compare(T a, T b)` |
| 排序规则数量 | **一个类只能有一种** | **可以有任意多种** |
| 要不要改类 | **要改**（加 implements） | 不用改 |
| 什么时候用 | 类有"天然默认顺序" | 临时/多种排序需求 |

**口诀**：**`Comparable` 是"我天生能比较"，`Comparator` 是"我给你个比较规则"。**

## 20. TreeSet 的实用方法

```java
TreeSet<Integer> t = new TreeSet<>(List.of(1,2,3,4,5,6,7,8,9));

t.first();          // 1        最小的
t.last();           // 9        最大的
t.headSet(5);       // [1,2,3,4]       小于 5 的（不含 5）
t.tailSet(5);       // [5,6,7,8,9]     大于等于 5 的
t.subSet(3, 7);     // [3,4,5,6]       [3,7) 左闭右开
t.lower(5);         // 4        严格小于 5 的最大值
t.higher(5);        // 6        严格大于 5 的最小值
t.ceiling(5);       // 5        >= 5 的最小值
t.floor(5);         // 5        <= 5 的最大值
```

**这些方法是 `List` 完全没有的**，是"有序集合"特有的能力。

## 21. TreeSet 不能放 null

```java
TreeSet<String> t = new TreeSet<>();
t.add(null);     // ❌ NullPointerException

HashSet<String> h = new HashSet<>();
h.add(null);     // ✅ 可以
```

**原因**：TreeSet 要排序，null 没法比较大小。

---

# 第七部分：综合练习

## 22. 学生按分数降序排名（多条件）

**需求**：学生按分数从高到低排，分数相同的按名字排。

```java
class Student implements Comparable<Student> {
    String name;
    int score;

    Student(String n, int s) { name = n; score = s; }

    @Override
    public int compareTo(Student o) {
        // 分数降序：o.score 在前
        int r = Integer.compare(o.score, this.score);
        if (r != 0) return r;
        // 分数相同按名字升序
        return this.name.compareTo(o.name);
    }

    @Override
    public String toString() { return name + ":" + score; }
}

TreeSet<Student> rank = new TreeSet<>();
rank.add(new Student("张三", 85));
rank.add(new Student("李四", 92));
rank.add(new Student("王五", 85));
rank.add(new Student("赵六", 78));

System.out.println(rank);
// [李四:92, 张三:85, 王五:85, 赵六:78]
//  最高分在前，同分按名字
```

**关键点**：
1. 降序 = 把 `this` 和 `o` 换个位置
2. **一定要加第二比较条件**，否则同分的会被当成一个人丢掉

## 23. 用比较器做同样的需求（不改类）

```java
TreeSet<Student> rank2 = new TreeSet<>(
    Comparator.comparingInt(Student::getScore).reversed()
              .thenComparing(Student::getName)
);
```

**类不用改，排序规则在创建集合时才给。**

---

# 第八部分：速查与避坑

## 24. 一页速查表

### 三个 Set 怎么选

| 需求 | 用 |
|---|---|
| 只去重 | `HashSet` |
| 去重 + 保持添加顺序 | `LinkedHashSet` |
| 去重 + 自动排序 | `TreeSet` |

### HashSet 去重

```java
// 自定义类必须重写这两个
@Override public boolean equals(Object o) { ... }
@Override public int hashCode() { return Objects.hash(字段1, 字段2); }
```

### TreeSet 排序

| 方式 | 写法 |
|---|---|
| 自然排序 | `class X implements Comparable<X>` + 实现 `compareTo` |
| 比较器排序 | `new TreeSet<>(Comparator.comparingInt(X::getField))` |
| 降序 | `Comparator.reverseOrder()` 或 `(a,b) -> b - a` |
| 多条件 | `.thenComparing(...)` |

### 比较器工具（最常用）

```java
Comparator.comparingInt(X::getAge)               // 按 int 字段
Comparator.comparing(X::getName)                 // 按对象字段（需 Comparable）
Comparator.comparingInt(...).thenComparing(...)  // 多条件
Comparator.reverseOrder()                        // 降序
```

## 25. 常见错误速查（照着躲）

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| 自定义类不重写 `hashCode`/`equals` | 两个都重写 | HashSet 去重失效 |
| 只重写 `equals` | 连 `hashCode` 一起 | 还是去重失效（最坑） |
| TreeSet 放没实现 Comparable 的类 | 实现 Comparable 或用 Comparator | `ClassCastException` |
| `compareTo` 只比一个字段 | 补上其他字段 | **数据被当重复丢掉** |
| 比较器只比一个字段 | 用 `thenComparing` 补条件 | **数据被去重** |
| 用 `HashSet` 想要顺序 | `LinkedHashSet` | 顺序随机 |
| `TreeSet` 放 null | 别放 | `NullPointerException` |
| 手写 `hashCode` 漏字段 | 用 `Objects.hash(...)` 或 IDEA 生成 | 两个"相等"的对象 hash 不同 |
| 依赖 `HashSet` 的打印顺序 | 别依赖 | 换个 JDK 版本顺序就变 |

## 26. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.util.*;

// ① 三个 Set 的顺序对比
Set<String> h = new HashSet<>();
Set<String> l = new LinkedHashSet<>();
Set<String> t = new TreeSet<>();
for (String s : new String[]{"c","a","b"}) { h.add(s); l.add(s); t.add(s); }
System.out.println("HashSet       : " + h);
System.out.println("LinkedHashSet : " + l);
System.out.println("TreeSet       : " + t);

// ② 复现"只重写 equals 不去重"
class A { String n; A(String n){this.n=n;}
    public boolean equals(Object o){ return o instanceof A a && a.n.equals(n); } }
Set<A> sa = new HashSet<>();
sa.add(new A("x")); sa.add(new A("x"));
System.out.println("只重写 equals 的 size = " + sa.size());

// ③ 复现 TreeSet 丢数据
class D implements Comparable<D> { String n; int a;
    D(String n,int a){this.n=n;this.a=a;}
    public int compareTo(D o){ return this.a - o.a; }
    public String toString(){ return n + a; } }
TreeSet<D> td = new TreeSet<>();
td.add(new D("甲",3)); td.add(new D("乙",3));
System.out.println("TreeSet 丢数据了: " + td);

// ④ 用比较器排序
TreeSet<String> byLen = new TreeSet<>(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()));
byLen.addAll(List.of("banana","apple","kiwi","fig"));
System.out.println("按长度排: " + byLen);
```

---

## 记忆口诀

> **HashSet 去重靠 hashCode 定位 + equals 确认，两个必须都重写；**
> **LinkedHashSet 多一个保序；**
> **TreeSet 排序靠 Comparable 或 Comparator，返回 0 就当重复；**
> **二叉查找树左小右大，会退化成链表，所以要平衡；**
> **红黑树靠颜色约束，大致平衡，查找 O(log n)。**
