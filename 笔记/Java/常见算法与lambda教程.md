# Java 常见算法与 lambda 教程

> 用途：学习笔记 · 定位：只要"技"（怎么写、怎么用、复杂度直觉、怎么避坑），不讲"道"（严格证明、算法分析课）
> 覆盖：常见算法-01 ~ 06 + lambda 表达式
> 所有示例和输出都是在本机 JDK 27 上用 jshell 实测跑出来的

---

## 0. 先建立一张地图

| 主题     | 解决什么问题           | 一句话记忆    |
| ------ | ---------------- | -------- |
| 基本查找   | 在数组里找一个数         | 挨个看      |
| 二分查找   | 在**有序**数组里快速找    | 对半砍      |
| 插值查找   | 在**均匀分布**的有序数组里找 | 按比例猜     |
| 斐波那契查找 | 二分的一种变体          | 按菲波拉契切   |
| 分块查找   | 数据分块 + 索引表       | 先找块再找数   |
| 哈希查找   | 一次定位             | 拿值当坐标    |
| 冒泡排序   | 把小的顶到前面          | 相邻两两比    |
| 选择排序   | 每轮找最小值放前面        | 找最小的     |
| 插入排序   | 像整理扑克牌           | 往已排好里插   |
| 快速排序   | 分而治之             | 挖坑填数     |
| 递归     | 自己调用自己           | 先想出口     |
| Arrays | 数组工具类            | 排序/查找/拷贝 |
| lambda | 把"方法"当参数传        | 简写匿名内部类  |

**复杂度速览（记住这个就够了）：**

| 算法 | 时间复杂度 | 用不用 |
|---|---|---|
| 基本查找 | O(n) | 小数据随便用 |
| 二分查找 | O(log n) | **前提必须有序** |
| 插值查找 | O(log log n) 理想 / O(n) 最差 | 数据均匀时好用 |
| 分块查找 | O(√n) 左右 | 数据多、分块管理 |
| 哈希查找 | O(1) 平均 | **最快，实际最爱用** |
| 冒泡 | O(n²) | 学习用 |
| 选择 | O(n²) | 学习用 |
| 插入 | O(n²) | 近乎有序时快 |
| 快速排序 | O(n log n) 平均 | **实际最快** |

**实战建议**：真要排序直接用 `Arrays.sort()`，真要查找直接用 `HashMap` 或 `Arrays.binarySearch()`。手写算法是为了**面试和看懂别人的代码**。

---

# 第一部分：查找

## 1. 基本查找（线性查找）

挨个看，找到就返回下标。

```java
int[] a = {7, 23, 79, 81, 103};
int target = 79;
int idx = -1;
for (int i = 0; i < a.length; i++) {
    if (a[i] == target) {
        idx = i;
        break;          // 找到了就停
    }
}
// idx = 2
```

**适用**：数组无序、数据量小。
**不适合**：数据量大（一千万个要查一千万次）。

### 1.1 小技巧：返回 -1 表示没找到

约定俗成：找到了返回下标（≥0），没找到返回 **-1**。不要返回 0（0 是合法下标）。

---

## 2. 二分查找（重点）

### 2.1 前提：数组必须有序

```java
int[] a = {7, 23, 79, 81, 103, 127, 131, 147};   // 已升序

int lo = 0, hi = a.length - 1;
int find = 131, pos = -1;

while (lo <= hi) {
    int mid = (lo + hi) >>> 1;          // 取中间
    if (a[mid] == find) { pos = mid; break; }
    if (a[mid] < find) lo = mid + 1;    // 目标在右半
    else hi = mid - 1;                  // 目标在左半
}
// pos = 6
```

### 2.2 三个循环条件，一个都不能错

| 写错 | 后果 |
|---|---|
| `while (lo < hi)` | 会漏掉最后一个元素 |
| `hi = mid` / `lo = mid` | 死循环（区间没缩小） |
| `mid = (lo + hi) / 2` | 数组极大时溢出 |

### 2.3 坑 1：无序数组二分会**静默查不到**

```java
int[] unsorted = {7, 23, 79, 81, 103, 127, 131, 147, 5};   // 5 在最后
// 查 5 的结果 = -1（明明在索引 8，但查不到）
```

**二分不检查数组是否有序，它只是"假设有序"。** 传进去无序数组不会报错，就是查不到。这是最难查的 bug 之一。

**解决**：用 `Arrays.sort()` 先排序，或者用 `Arrays.binarySearch()`（它也不检查有序，同样会静默失败）。

### 2.4 坑 2：`(lo + hi) / 2` 会溢出

```java
int lo = 2_000_000_000, hi = 2_100_000_000;

(lo + hi) / 2        // -97483648   ← 溢出成负数了！
lo + (hi - lo) / 2   // 2050000000  ✅ 正确
(lo + hi) >>> 1      // 2050000000  ✅ 也可以
```

**三种安全写法，任选一个：**
```java
int mid = lo + (hi - lo) / 2;   // 最推荐，含义清楚
int mid = (lo + hi) >>> 1;      // JDK 源码写法，快一点
```

> 普通数组长度到不了 21 亿，但面试和 LeetCode 会考这个点。

### 2.5 用现成的 `Arrays.binarySearch()`

```java
int[] a = {1, 2, 5, 9};   // 必须已排序

Arrays.binarySearch(a, 5);    // 2   找到，返回下标
Arrays.binarySearch(a, 7);    // -4  没找到，返回负数
```

**返回负数不是"没找到返回 -1"**，而是 `-(插入点) - 1`。上例里 7 应该插在索引 3，所以返回 `-(3)-1 = -4`。

**判断是否找到：看返回值是否 `>= 0`。**

---

## 3. 插值查找

### 3.1 思路：不看中间，按"比例"猜位置

二分是"永远从中间切"，插值查找是"**估计目标大概在哪**"。

比如在 `{1,3,5,...,19}` 里找 15，它明显靠右，不该从中间找。

```java
int[] arr = {1, 3, 5, 7, 9, 11, 13, 15, 17, 19};
int find = 15;

int lo = 0, hi = arr.length - 1, idx = -1;
while (lo <= hi && find >= arr[lo] && find <= arr[hi]) {
    if (lo == hi) { if (arr[lo] == find) idx = lo; break; }

    // 按比例估算 mid（核心公式）
    int mid = lo + (hi - lo) * (find - arr[lo]) / (arr[hi] - arr[lo]);

    if (arr[mid] == find) { idx = mid; break; }
    else if (arr[mid] < find) lo = mid + 1;
    else hi = mid - 1;
}
// idx = 7，第一次就猜中了
```

### 3.2 关键公式

```java
mid = lo + (hi - lo) * (find - arr[lo]) / (arr[hi] - arr[lo]);
```

**含义**：目标是这段区间里的百分之几，就从区间起点往后走百分之几。

### 3.3 适用条件（很苛刻）

| 条件 | 说明 |
|---|---|
| 数据**有序** | 和二分一样 |
| 数据**分布均匀** | 得分点！数组是 `{1,2,3,4,5}` 好用，`{1,2,4,8,16,1024}` 就退化成 O(n) |

**不均匀时比二分还慢**，实际项目里很少用，主要考概念。

---

## 4. 斐波那契查找

### 4.1 一句话：用斐波那契数当分割点

二分每次切一半，斐波那契查找每次按黄金比例切（约 0.618）。

**了解结论就行**：
- 平均性能略优于二分（减少比较次数）
- 但实现复杂、必须构造斐波那契数列、还要给数组补长度
- **实际项目从不手写，考试一般只考概念**

### 4.2 核心思路（不要求手写）

1. 找一个斐波那契数 `F(k)`，使 `F(k)-1 >= 数组长度`
2. 把数组补长到 `F(k)-1`（补最后一个元素）
3. 分割点取 `lo + F(k-1) - 1`
4. 目标在左半 → `k = k-1`；在右半 → `k = k-2`

**如果你要背，记住这句：二分用"对半"，斐波那契用"黄金分割"。**

---

## 5. 分块查找

### 5.1 思路：数据分块 + 建索引表

```
原数据: 16 5 9 12 | 21 18 32 23 37 | 26 45 34 50 48 61 | 52 73 66
块内:   块1 无序   | 块2 无序        | 块3 无序           | 块4 无序
特点:   块内可以无序，但块与块之间必须整体有序
索引表: 每块记录「最大值」和「起始下标」
```

**查找两步走**：先在索引表里找块（可以二分），再在块内顺序找。

```java
int[] block = {16, 5, 9, 12, 21, 18, 32, 23, 37, 26, 45, 34, 50, 48, 61, 52, 73, 66};
int blockSize = 6;
int blocks = (block.length + blockSize - 1) / blockSize;

// 建索引：每块的最大值
int[] maxPerBlock = new int[blocks];
for (int b = 0; b < blocks; b++) {
    int m = Integer.MIN_VALUE;
    for (int i = b * blockSize; i < Math.min((b + 1) * blockSize, block.length); i++) {
        m = Math.max(m, block[i]);
    }
    maxPerBlock[b] = m;
}
// maxPerBlock = [21, 45, 73]
```

### 5.2 适用场景

| 情况 | 适不适合 |
|---|---|
| 数据总量大、要频繁查 | ✅ 适合 |
| 数据经常增删（会打乱分块） | ❌ 不适合 |
| 写数据库索引、文件分块 | ✅ 典型用途 |
| 单纯的数组查找 | ❌ 直接哈希更好 |

---

## 6. 哈希查找

### 6.1 一句话：拿"值"当"位置"

```java
HashMap<Integer, Integer> hash = new HashMap<>();
int[] data = {16, 5, 9, 12, 21, 18, 32, 23, 37};

for (int i = 0; i < data.length; i++) {
    hash.put(data[i], i);        // 值 -> 下标
}

hash.get(23);    // 7     一次就查到
hash.get(99);    // null  不存在
```

### 6.2 为什么最快

平均 **O(1)**：算出哈希值 → 直接跳到那个位置 → 看一眼对不对。

**代价**：占额外内存（要存一张表），而且数据变动要维护表。

### 6.3 实战结论

> **有序数组查找：二分。无序数据频繁查：先建 HashMap。**
> 真项目里 90% 的"查找"都是用哈希，手写算法的场合只有面试。

---

# 第二部分：排序与递归

## 7. 冒泡排序

### 7.1 思路：相邻两两比较，大的往后冒

```java
int[] b = {5, 2, 9, 1, 7};

for (int i = 0; i < b.length - 1; i++) {
    boolean swapped = false;
    for (int j = 0; j < b.length - 1 - i; j++) {
        if (b[j] > b[j + 1]) {
            int t = b[j]; b[j] = b[j + 1]; b[j + 1] = t;
            swapped = true;
        }
    }
    if (!swapped) break;      // 本轮一次没交换，说明已有序，提前停
}
```

### 7.2 每次跑完的样子（实测）

```
初始:      [5, 2, 9, 1, 7]
第 1 轮后: [2, 5, 1, 7, 9]
第 2 轮后: [2, 1, 5, 7, 9]
第 3 轮后: [1, 2, 5, 7, 9]
第 4 轮后: [1, 2, 5, 7, 9]
本轮无交换，提前结束
```

**看到没**：第 3 轮就已经有序了，`swapped` 的优化让第 4 轮直接跳出。

### 7.3 两个容易漏的细节

| 细节 | 说明 |
|---|---|
| 内层循环是 `length - 1 - i` | 每轮末尾已经排好 i 个，不用再比 |
| 加 `swapped` 标志 | 一次交换都没发生 → 已经有序 → 提前结束 |

---

## 8. 选择排序

### 8.1 思路：每轮找最小值，放到前面

```java
int[] s = {5, 2, 9, 1, 7};

for (int i = 0; i < s.length - 1; i++) {
    int minIdx = i;
    for (int j = i + 1; j < s.length; j++) {
        if (s[j] < s[minIdx]) minIdx = j;
    }
    int t = s[i]; s[i] = s[minIdx]; s[minIdx] = t;
}
```

### 8.2 每轮结果（实测）

```
第 1 轮后: [1, 2, 9, 5, 7]   ← 找到了最小的 1
第 2 轮后: [1, 2, 9, 5, 7]   ← 2 已经在正确位置，看起来没变
第 3 轮后: [1, 2, 5, 9, 7]
第 4 轮后: [1, 2, 5, 7, 9]
```

**注意第 2 轮**：结果看起来没变，是正常的——最小值本来就在 `i` 位置，交换前后一样。

### 8.3 和冒泡的区别

| | 冒泡 | 选择 |
|---|---|---|
| 每轮做的事 | 相邻交换多次 | 找最小值**只交换一次** |
| 交换次数 | 多 | **少** |
| 比较次数 | 一样 | 一样 |

**想交换少 → 选择排序。** 这是它比冒泡唯一强的地方。

---

## 9. 插入排序

### 9.1 思路：像整理手里的扑克牌

```java
int[] ins = {5, 2, 9, 1, 7};

for (int i = 1; i < ins.length; i++) {
    int cur = ins[i];          // 当前要插入的牌
    int j = i - 1;
    while (j >= 0 && ins[j] > cur) {
        ins[j + 1] = ins[j];   // 大的往后挪
        j--;
    }
    ins[j + 1] = cur;          // 插到空出来的位置
}
```

### 9.2 每轮结果（实测）

```
插入 2 后: [2, 5, 9, 1, 7]
插入 9 后: [2, 5, 9, 1, 7]
插入 1 后: [1, 2, 5, 9, 7]
插入 7 后: [1, 2, 5, 7, 9]
```

### 9.3 为什么它"近乎有序时特别快"

如果数组本来就基本有序，内层 `while` 几乎立刻退出，接近 **O(n)**。
所以实际工程里，对"基本有序"的小数组，插入排序比快排还快——很多快排实现**在小数组时会切换成插入排序**。

---

## 10. 快速排序（重点）

### 10.1 思路：挖坑填数 + 分治

1. 选一个基准值 `key`（一般取最左边）
2. 从右往左找比 `key` 小的，从左往右找比 `key` 大的，交换
3. 最后把 `key` 放到中间，左边都比它小、右边都比它大
4. 对左右两半**递归**重复

```java
void quickSort(int[] a, int left, int right) {
    if (left >= right) return;              // 递归出口
    int p = partition(a, left, right);      // 返回基准最终位置
    quickSort(a, left, p - 1);              // 排左半
    quickSort(a, p + 1, right);             // 排右半
}

int partition(int[] a, int left, int right) {
    int key = a[left];                      // 基准值
    int i = left, j = right;

    while (i < j) {
        while (i < j && a[j] >= key) j--;   // 从右找小的
        while (i < j && a[i] <= key) i++;   // 从左找大的
        if (i < j) {                        // 交换
            int t = a[i]; a[i] = a[j]; a[j] = t;
        }
    }
    a[left] = a[i];                         // 基准归位
    a[i] = key;
    return i;
}
```

实测：

```
排序前: [5, 2, 9, 1, 7, 3]
快排结果: [1, 2, 3, 5, 7, 9]
```

### 10.2 三个最容易写错的地方

| 地方 | 易错点 |
|---|---|
| `while (i < j && a[j] >= key)` | 顺序错：**必须先移动 j（右），再移动 i（左）** |
| `a[j] >= key` | 少了 `=` 会死循环（等于 key 的元素卡住不动） |
| `if (left >= right) return;` | 少了出口 → `StackOverflowError` |

### 10.3 为什么必须先从右往左扫

因为基准值取的是 `a[left]`。最后 `a[left] = a[i]` 要求 `a[i]` 是个**小于等于基准**的值。从右往左扫能保证最后停在的位置满足这个条件；反过来会出问题。

**这个点面试常问，记住："基准在左，就先从右边扫"。**

### 10.4 快排的坑

```java
// 已排序数组 + 每次取最左当基准 → 退化成 O(n²)
int[] sorted = {1, 2, 3, 4, 5, 6, 7, 8, 9};
```

**已有序的数组是最坏情况**，递归深度 = n。数据量大会 `StackOverflowError`。
实际 JDK 的 `Arrays.sort()` 用的是 **双轴快排 + 三路划分 + 插入排序兜底**，已经解决了这个问题。

---

## 11. 递归

### 11.1 两个必备条件

1. **必须有一个"出口"**（终止条件），否则无限递归
2. **每次递归都要向出口靠近**（参数变小）

### 11.2 三个经典例子（实测结果）

```java
// 阶乘
int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);   // 出口：n <= 1
}
factorial(5);    // 120

// 斐波那契
int fib(int n) {
    return n <= 2 ? 1 : fib(n-1) + fib(n-2);    // 出口：n <= 2
}
fib(10);         // 55

// 1 加到 n
int sumN(int n) {
    return n == 1 ? 1 : n + sumN(n - 1);        // 出口：n == 1
}
sumN(100);       // 5050
```

### 11.3 坑：忘记出口 = StackOverflowError

```java
int boom(int n) {
    return boom(n + 1);     // 没有出口
}
boom(1);    // StackOverflowError（栈溢出）
```

**注意报错类型**：`StackOverflowError` 是 `Error` 不是 `Exception`，`catch (Exception e)` **抓不到它**。要抓得写 `catch (StackOverflowError e)`。

### 11.4 递归的代价：斐波那契的 O(2ⁿ)

```java
fib(40);    // 要等很久，因为重复计算了大量子问题
```

**怎么优化：**
- **记忆化**：用一个数组/Map 缓存算过的结果
- **改循环**：斐波那契完全可以写成两个变量滚动（O(n) 空间 O(1)）
- **动态规划**：本质就是"递归 + 记忆化"的正规版本

### 11.5 什么时候用递归

| 场景 | 建议 |
|---|---|
| 树、图的遍历 | ✅ 递归最自然 |
| 分治（快排、归并） | ✅ 适合 |
| 简单循环能做的事 | ❌ 别递归，可读性差还慢 |
| 递归深度可能上万 | ⚠️ 会栈溢出，改循环或显式栈 |

---

# 第三部分：Arrays 与 lambda

## 12. Arrays 工具类

> **Java 有数组，但没有"数组的方法"。数组的所有工具都在 `Arrays` 类里。**

### 12.1 最常用的几个（全部实测）

```java
import java.util.Arrays;

int[] a = {5, 2, 9, 1};

Arrays.toString(a);              // "[5, 2, 9, 1]"   打印数组（不写就是 hashcode）
Arrays.sort(a);                  // 原地升序 → [1, 2, 5, 9]
Arrays.binarySearch(a, 5);       // 2                必须先排序！
Arrays.binarySearch(a, 7);       // -4               负数 = 没找到
Arrays.equals(new int[]{1,2}, new int[]{1,2});   // true
Arrays.copyOf(a, 2);             // "[1, 2]"         截取前 2 个
Arrays.copyOf(a, 6);             // "[1, 2, 5, 9, 0, 0]"   补 0
Arrays.copyOfRange(a, 1, 3);     // "[2, 5]"         [1,3) 左闭右开
Arrays.fill(f, 7);               // "[7, 7, 7, 7]"   全部填成 7
```

### 12.2 逆序排序（基本类型数组的坑）

```java
int[] a = {3, 1, 2};
Arrays.sort(a);                              // ✅ 升序可以
// Arrays.sort(a, Comparator.reverseOrder()); // ❌ 编译错误！
```

**原因**：`Comparator` 只接受**对象**，`int[]` 是基本类型数组，没有比较器版本。

**解决**：用包装类数组。

```java
Integer[] boxed = {3, 1, 2};
Arrays.sort(boxed, Comparator.reverseOrder());   // [3, 2, 1]  ✅
```

### 12.3 二维数组打印

```java
int[][] mat = {{1,2}, {3,4}};

Arrays.toString(mat);        // "[[I@1b6d3586, [I@4554617c]"   ← 一堆地址
Arrays.deepToString(mat);    // "[[1, 2], [3, 4]]"              ✅
```

**二维（或多维）数组必须用 `deepToString`。**

### 12.4 坑：`Arrays.asList` 返回的列表不能改

```java
List<Integer> list = Arrays.asList(1, 2, 3);
list.add(4);    // ❌ UnsupportedOperationException
```

**`Arrays.asList` 返回的是"固定大小"的视图**，底层还是那个数组，所以只能改值不能增删。需要可变列表要再包一层：

```java
new ArrayList<>(Arrays.asList(1, 2, 3));   // ✅ 可增删
```

### 12.5 Arrays 的常用方法总表

| 方法 | 作用 | 注意 |
|---|---|---|
| `toString(a)` | 打印一维数组 | |
| `deepToString(a)` | 打印多维数组 | |
| `sort(a)` | 升序排序 | 原地修改 |
| `sort(a, cmp)` | 按比较器排序 | 只支持对象数组 |
| `binarySearch(a, k)` | 二分查找 | **先排序**，返回负数=没找到 |
| `equals(a, b)` | 比较内容 | 不是 `==` |
| `copyOf(a, n)` | 复制/截断/补零 | |
| `copyOfRange(a, f, t)` | 复制区间 | 左闭右开 |
| `fill(a, v)` | 全部填值 | |
| `asList(...)` | 转成 List | 不可增删 |
| `stream(a)` | 转成流 | 配合 lambda |

---

## 13. lambda 表达式

### 13.1 一句话：lambda 就是"简写的匿名内部类"

```java
// 老写法：匿名内部类
Comparator<Integer> c1 = new Comparator<Integer>() {
    @Override
    public int compare(Integer a, Integer b) { return a - b; }
};

// lambda 写法
Comparator<Integer> c2 = (a, b) -> a - b;
```

**两个例子效果完全一样**（实测 `compare(3, 5)` 都返回 `-2`）。

### 13.2 语法拆解

```
(a, b)  ->  a - b
 ↑        ↑    ↑
参数    箭头   方法体
```

| 情形 | 写法 |
|---|---|
| 无参数 | `() -> 语句` |
| 一个参数 | `x -> 语句`（括号可省） |
| 多个参数 | `(a, b) -> 语句` |
| 单行表达式 | `(a, b) -> a - b`（自动 return） |
| 多行代码 | `(a, b) -> { 语句1; return 结果; }` |
| 参数类型 | 可以省略，编译器自己推断 |

### 13.3 为什么能这么简写：函数式接口

lambda 不能随便写，它必须**对应一个"只有一个抽象方法"的接口**（函数式接口）。

### 13.4 四种最常用的函数式接口（必背）

| 接口 | 方法 | 含义 | 例子 |
|---|---|---|---|
| `Function<T,R>` | `R apply(T)` | 输入一个，输出一个 | `s -> s.length()` |
| `Consumer<T>` | `void accept(T)` | 消费一个，不返回 | `s -> System.out.println(s)` |
| `Supplier<T>` | `T get()` | 不输入，产出一个 | `() -> Math.random()` |
| `Predicate<T>` | `boolean test(T)` | 判断，返回 true/false | `s -> s.isEmpty()` |

加上 `BiFunction<T,U,R>`（两个输入一个输出）：

```java
Function<String, Integer> len = s -> s.length();
len.apply("hello");        // 5

Consumer<String> print = s -> System.out.println(s);
print.accept("你好");       // 打印"你好"

Supplier<Double> sup = () -> Math.random();
sup.get();                 // 一个随机数

Predicate<String> isEmpty = s -> s.isEmpty();
isEmpty.test("");          // true

BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
add.apply(3, 4);           // 7
```

**记忆技巧**：`Function`=函数（有进有出）、`Consumer`=消费（只进不出）、`Supplier`=供给（不出不进，只产出）、`Predicate`=断言（返回布尔）。

### 13.5 方法引用 `::`（lambda 的进一步简写）

当 lambda 体**只是调用一个已有的方法**时，可以再简化：

```java
// 完整 lambda
s -> System.out.println(s)
// 方法引用
System.out::println          // 实例方法引用

// 完整 lambda
s -> s.toUpperCase()
// 方法引用
String::toUpperCase          // 实例方法引用

// 完整 lambda
s -> s.length()
// 方法引用
String::length
```

**四种形式：**

| 形式 | 写法 | 例子 |
|---|---|---|
| 静态方法引用 | `类名::静态方法` | `Integer::parseInt` |
| 实例方法引用 | `对象::方法` | `System.out::println` |
| 特定类型的任意对象方法 | `类名::方法` | `String::toUpperCase` |
| 构造器引用 | `类名::new` | `ArrayList::new` |

### 13.6 集合里 lambda 的实战用法

```java
List<Integer> nums = new ArrayList<>(Arrays.asList(3, 8, 1, 9, 2, 7));

// 1. 遍历
nums.forEach(n -> System.out.print(n + " "));

// 2. 条件删除（比 for 循环里删安全得多）
nums.removeIf(n -> n % 2 == 0);        // 删除所有偶数

// 3. 排序
names.sort(Comparator.comparingInt(String::length));

// 4. 链式比较器：先按长度，再按字母
words.sort(Comparator.comparingInt(String::length)
                     .thenComparing(Comparator.naturalOrder()));
// ["bb","a","ccc","dd"] → [a, bb, dd, ccc]
```

**`removeIf` 是重点**：以前在 `for (Integer n : list)` 里删元素会 `ConcurrentModificationException`，用 `removeIf` 完全没这个问题。

### 13.7 Stream 常用（lambda 的典型搭档）

```java
List<Integer> data = Arrays.asList(1, 2, 3, 4, 5, 6);

data.stream().filter(n -> n % 2 == 0).toList();           // [2, 4, 6]
data.stream().map(n -> n * n).toList();                    // [1, 4, 9, 16, 25, 36]
data.stream().mapToInt(Integer::intValue).sum();           // 21
data.stream().anyMatch(n -> n > 5);                        // true
data.stream().filter(n -> n > 3).count();                  // 3
```

**Stream 三件套**：
- `filter` —— 筛（保留满足条件的）
- `map` —— 变（把每个元素转成另一个）
- `toList` / `sum` / `count` / `anyMatch` —— 收（把结果收出来）

### 13.8 坑：lambda 捕获的变量必须"有效 final"

```java
int base = 10;
Function<Integer, Integer> fn = x -> x + base;   // ✅ 可以读
base = 20;                                        // ❌ 编译错误！
```

**lambda 里能读外部变量，但那个变量之后不能再赋值**（effectively final）。想改就用一个 `final` 的容器（数组、`AtomicInteger`）或改成返回值。

---

# 第四部分：对照与速查

## 14. 算法速查表

### 查找

| 算法 | 前提 | 时间复杂度 | 用不用 |
|---|---|---|---|
| 线性查找 | 无 | O(n) | 小数据 |
| 二分查找 | **有序** | O(log n) | ✅ 常用 |
| 插值查找 | 有序 + **均匀** | 理想 O(log log n) | 特定场景 |
| 斐波那契查找 | 有序 | O(log n) | 考概念 |
| 分块查找 | 块间有序 | ~O(√n) | 数据分块管理 |
| 哈希查找 | 无 | **O(1)** | ✅ 最实用 |

### 排序

| 算法 | 平均 | 最坏 | 稳定 | 记忆点 |
|---|---|---|---|---|
| 冒泡 | O(n²) | O(n²) | ✅ 稳定 | 加 swapped 提前停 |
| 选择 | O(n²) | O(n²) | ❌ | 交换次数少 |
| 插入 | O(n²) | O(n²) | ✅ 稳定 | 近乎有序时最快 |
| 快速 | O(n log n) | O(n²) | ❌ | 基准在左先从右扫 |

## 15. 常见错误速查（照着躲）

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| 二分查无序数组 | 先 `Arrays.sort()` | 静默查不到，返回 -1 |
| `mid = (lo + hi) / 2` | `lo + (hi-lo)/2` 或 `(lo+hi)>>>1` | 超大数组溢出 |
| `while (lo < hi)` 二分 | `while (lo <= hi)` | 漏掉最后一个元素 |
| 二分 `hi = mid` | `hi = mid - 1` | 死循环 |
| 快排先从左扫 | 基准在左 → **先从右扫** | 结果错误 |
| 快排 `a[j] > key` | `a[j] >= key` | 死循环 |
| 递归忘写出口 | 加终止条件 | `StackOverflowError` |
| `catch (Exception)` 抓递归溢出 | `catch (StackOverflowError)` | 抓不到 |
| `Arrays.sort(int[], 逆序Comparator)` | 用 `Integer[]` | 编译错误 |
| `Arrays.toString(二维数组)` | `Arrays.deepToString` | 输出地址 |
| `Arrays.asList(...).add()` | `new ArrayList<>(Arrays.asList(...))` | `UnsupportedOperationException` |
| for 循环里 `list.remove()` | `list.removeIf(...)` | `ConcurrentModificationException` |
| lambda 里改外部变量 | 改用 final 容器或返回值 | 编译错误 |

## 16. 一页速查

### 查找

| 想要 | 写法 |
|---|---|
| 找某个值 | `Arrays.binarySearch(a, k)`（先 sort），或 `HashMap` |
| 二分模板骨架 | `while (lo <= hi)` + `mid` + 缩区间 |
| 判断找到 | 返回值 `>= 0` |

### 排序

| 想要 | 写法 |
|---|---|
| 升序 | `Arrays.sort(a)` |
| 逆序（对象数组） | `Arrays.sort(arr, Comparator.reverseOrder())` |
| 自定义 | `Arrays.sort(arr, (a, b) -> ...)` |
| 多条件 | `Comparator.comparingInt(...).thenComparing(...)` |
| List 排序 | `list.sort(Comparator...)` |

### lambda

| 想要 | 写法 |
|---|---|
| 遍历 | `list.forEach(x -> ...)` |
| 条件删除 | `list.removeIf(x -> ...)` |
| 判断存在 | `stream.anyMatch(x -> ...)` |
| 转换集合 | `stream.map(x -> ...).toList()` |
| 求和 | `stream.mapToInt(...).sum()` |
| 方法引用 | `Xxx::方法名` |

## 17. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.util.*;

// ① 手动二分
int[] a = {1, 3, 5, 7, 9, 11};
int find = 7, lo = 0, hi = a.length - 1, pos = -1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (a[mid] == find) { pos = mid; break; }
    if (a[mid] < find) lo = mid + 1; else hi = mid - 1;
}
System.out.println("位置 = " + pos);

// ② 冒泡排序
int[] b = {5, 2, 9, 1};
for (int i = 0; i < b.length - 1; i++)
    for (int j = 0; j < b.length - 1 - i; j++)
        if (b[j] > b[j+1]) { int t = b[j]; b[j] = b[j+1]; b[j+1] = t; }
System.out.println(Arrays.toString(b));

// ③ 用 Arrays 一行搞定
int[] c = {9, 3, 7, 1};
Arrays.sort(c);
System.out.println(Arrays.toString(c));

// ④ lambda 排序 + 过滤
List<Integer> nums = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));
nums.removeIf(n -> n % 2 == 0);
nums.sort((x, y) -> y - x);
System.out.println(nums);

// ⑤ Stream 一把梭
System.out.println(nums.stream().mapToInt(Integer::intValue).sum());
```

---

## 记忆口诀

> **找：有序用二分，无序建哈希，手写只为面试；**
> **排：冒泡带标志，选择少交换，插入怕乱不怕顺，快排基准在左先从右扫；**
> **递归：必须有出口，栈溢出抓 Error 不抓 Exception；**
> **数组：打印 toString，二维用 deepToString，查找先排序；**
> **lambda：函数式接口才能接，四兄弟 Function/Consumer/Supplier/Predicate。**
