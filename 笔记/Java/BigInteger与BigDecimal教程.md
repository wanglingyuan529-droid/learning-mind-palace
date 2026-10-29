# BigInteger 与 BigDecimal 使用教程

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"（底层设计、算法原理）
> 所有示例和输出都是在本机 JDK 27 上实测跑出来的

---

## 0. 一句话认识它们

| 类型 | 解决什么问题 | 类比 |
|---|---|---|
| `BigInteger` | 整数**太大**，`long` 装不下（超过 19 位） | 超级大的整数计算器 |
| `BigDecimal` | 小数**精度不够**，`double` 有误差（0.1 + 0.2 ≠ 0.3） | 精确到每一分钱的计算器 |

**它们都不是基础类型**，是 `java.math` 包下的**类**，所以：

```java
// 错误！不能直接用 + - * /
BigInteger a = new BigInteger("100");
BigInteger b = a + a;          // 编译错误

// 正确：所有运算都要调方法
BigInteger b = a.add(a);       // 用方法代替运算符
```

这是学它们**第一个要改掉的习惯**：`+` `-` `*` `/` 全部要换成 `.add()` `.subtract()` `.multiply()` `.divide()`。

---

## 1. 为什么需要它们（两个真实事故）

### 1.1 `long` 会悄悄溢出，还不报错

```java
System.out.println(Long.MAX_VALUE);              // 9223372036854775807
System.out.println(Long.MAX_VALUE + 1);          // -9223372036854775808  ← 变负了！
System.out.println(BigInteger.valueOf(Long.MAX_VALUE).add(BigInteger.ONE));
                                                 // 9223372036854775808   ← 正确
```

**注意：`Long.MAX_VALUE + 1` 不报错、不警告，直接给你一个负数。** 这种 bug 最难查。

### 1.2 `double` 算钱会算错

```java
System.out.println(0.1 + 0.2);        // 0.30000000000000004
System.out.println(1.0 - 0.9);        // 0.09999999999999998
```

**只要涉及钱、价格、利息、分数，一律用 BigDecimal。** 这是行业铁律。

---

## 2. BigInteger 基础

### 2.1 三种创建方式

```java
BigInteger a = new BigInteger("123456789012345678901234567890");  // ← 最常用：字符串
BigInteger b = BigInteger.valueOf(100);                            // ← 从 long 转
BigInteger c = new BigInteger("1010", 2);                          // ← 二进制字符串（了解即可）
```

**为什么用字符串？** 因为 `new BigInteger(123456789012345678901234567890)` 里的数字在传进去之前就已经超出 `long` 范围了，编译都过不了。**大数必须用字符串传。**

常用常量：

```java
BigInteger.ZERO      // 0
BigInteger.ONE       // 1
BigInteger.TWO       // 2（JDK 9+）
BigInteger.TEN       // 10
```

### 2.2 四则运算（最常用）

```java
BigInteger a = new BigInteger("123456789012345678901234567890");
BigInteger b = new BigInteger("987654321098765432109876543210");

System.out.println(a.add(b));        // 加法
System.out.println(a.subtract(b));   // 减法
System.out.println(a.multiply(b));   // 乘法
System.out.println(b.divide(a));     // 除法（整数除法，直接丢小数）
System.out.println(b.remainder(a));  // 取余
```

实测输出：

```
a + b = 1111111110111111111011111111100
a - b = -864197532086419753208641975320
a * b = 121932631137021795226185032733622923332237463801111263526900
b / a = 8
b % a = 9000000000900000000090
```

**除法注意**：`b.divide(a)` 是整数除法，`7/2` 得到 `3` 不是 `3.5`。想要小数就得转 BigDecimal（见第 6 节）。

**除零会抛异常**：`a.divide(BigInteger.ZERO)` → `ArithmeticException: / by zero`，和普通数字一样。

### 2.3 比较大小

```java
BigInteger a = BigInteger.valueOf(100);
BigInteger b = BigInteger.valueOf(200);

System.out.println(a.compareTo(b));   // -1（a < b）
System.out.println(b.compareTo(a));   //  1（b > a）
System.out.println(a.compareTo(a));   //  0（相等）

// 或者更好读的写法
System.out.println(a.max(b));         // 200
System.out.println(a.min(b));         // 100
```

**绝对不要用 `==` 比较两个 BigInteger！** `==` 比的是"是不是同一个对象"，不是数值。

```java
BigInteger x = BigInteger.valueOf(100);
BigInteger y = BigInteger.valueOf(100);
System.out.println(x == y);        // 可能 true 也可能 false，千万别依赖
System.out.println(x.equals(y));   // true ← 数值相等用这个
```

### 2.4 转换回普通类型

```java
BigInteger big = new BigInteger("123");

big.intValue();       // → 123 (int)
big.longValue();      // → 123 (long)
big.doubleValue();    // → 123.0 (double)
big.toString();       // → "123"（任何类型都有，可以直接用）
```

**危险**：如果数值超出 int 范围，`intValue()` **不会报错**，直接给你一个截断后的错数：

```java
new BigInteger("99999999999").intValue();   // → 1215752191  ← 数都变了，还不报错
```

要判断能不能安全转换，用 `bitLength()`（数字占多少二进制位）：

```java
new BigInteger("123").bitLength();          // 7  ← 小于 32，能塞进 int
new BigInteger("99999999999").bitLength();  // 37 ← 超过 32，别转 int
new BigInteger("123").bitLength() < 32;     // true
```

同理，塞进 `long` 要判断 `< 64`。

---

## 3. BigInteger 进阶（用到再看）

```java
BigInteger.TEN.pow(100)                             // 10 的 100 次方
BigInteger.valueOf(-5).abs()                         // 绝对值
BigInteger.valueOf(48).gcd(BigInteger.valueOf(18))   // 最大公约数 → 6
BigInteger.valueOf(97).isProbablePrime(100)          // 判断素数 → true
```

**阶乘**（只有大数能做）：

```java
BigInteger factorial(int n) {
    BigInteger r = BigInteger.ONE;
    for (int i = 2; i <= n; i++) {
        r = r.multiply(BigInteger.valueOf(i));
    }
    return r;
}

System.out.println(factorial(20));    // 2432902008176640000（long 也能勉强装下）
System.out.println(factorial(100));   // 一个 158 位长的数字（long 完全装不下）
```

**斐波那契第 100 项**（经典练手题）：

```java
BigInteger a = BigInteger.ZERO, b = BigInteger.ONE;
for (int i = 0; i < 100; i++) {
    BigInteger t = a.add(b);
    a = b;
    b = t;
}
System.out.println(a);   // 354224848179261915075
```

---

## 4. BigDecimal 基础

### 4.1 创建方式（**这里有一个大坑**）

```java
new BigDecimal("0.1")          // ✅ 推荐：字符串构造
BigDecimal.valueOf(0.1)        // ✅ 推荐：从 double 转（内部会先转字符串）
new BigDecimal(0.1)            // ❌ 千万别用：会得到一长串垃圾
```

实测差别：

```
new BigDecimal(0.1)              = 0.1000000000000000055511151231257827021181583404541015625
BigDecimal.valueOf(0.1)          = 0.1
new BigDecimal("0.1")            = 0.1
```

**原因一句话**：`double` 本身存不准 0.1，`new BigDecimal(double)` 会把那个"存不准的二进制值"原封不动搬过来。**记住结论：永远用字符串构造，或用 `BigDecimal.valueOf()`。**

### 4.2 四则运算

```java
BigDecimal a = new BigDecimal("10.5");
BigDecimal b = new BigDecimal("3.2");

System.out.println(a.add(b));        // 加法 → 13.7
System.out.println(a.subtract(b));   // 减法 → 7.3
System.out.println(a.multiply(b));   // 乘法 → 33.60
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));  // 除法 → 3.28
```

**注意除法必须带参数**（坑，下一篇详说）。

### 4.3 除法为什么必须指定精度

```java
new BigDecimal("1").divide(new BigDecimal("3"));   // ← 直接抛异常！
// ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
```

因为 1/3 是无限小数，**BigDecimal 算不出来结果，就抛异常了**。所以除法必须告诉它"保留几位、怎么舍入"：

```java
new BigDecimal("1").divide(new BigDecimal("3"), 10, RoundingMode.HALF_UP);
// → 0.3333333333
```

参数依次是：**被除数、除数、保留几位小数、舍入方式**。

---

## 5. BigDecimal 的六个必知细节

### 5.1 equals 和 compareTo 不一样（最坑的一个）

```java
BigDecimal x = new BigDecimal("1.0");
BigDecimal y = new BigDecimal("1.00");

System.out.println(x.equals(y));      // false！因为 1.0 和 1.00 精度不同
System.out.println(x.compareTo(y));   // 0    因为数值相等
```

**结论：比较 BigDecimal 的数值大小，永远用 `compareTo() == 0`，不要用 `equals()`。**

这条坑最常见于：存进 `HashMap`、放进 `List.contains()`、写 `if (a.equals(b))` 判断金额相等——**全部会出错**。

### 5.2 不可变：所有运算都返回新对象

```java
BigDecimal money = new BigDecimal("100");
money.add(new BigDecimal("50"));      // 这行没有任何效果！
System.out.println(money);            // 还是 100

money = money.add(new BigDecimal("50"));   // 必须接住返回值
System.out.println(money);            // 150
```

**所有 `add/subtract/multiply/divide/setScale` 都不会改变原对象。** 忘了接收返回值，就是白算。

### 5.3 保留小数：setScale

```java
BigDecimal price = new BigDecimal("123.4567");

price.setScale(2, RoundingMode.HALF_UP);   // → 123.46
price.setScale(0, RoundingMode.HALF_UP);   // → 123
price.setScale(1, RoundingMode.DOWN);      // → 123.4
```

**一定要带舍入模式**，否则遇到需要舍入的情况会抛异常。

### 5.4 六种舍入模式（只用记住前两个）

| 模式 | 含义 | `2.5` → | `-2.5` → |
|---|---|---|---|
| `HALF_UP` | 四舍五入（最常用） | `3` | `-3` |
| `HALF_EVEN` | 银行家舍入（.5 向偶数靠） | `2` | `-2` |
| `CEILING` | 向上取整 | `3` | `-2` |
| `FLOOR` | 向下取整 | `2` | `-3` |
| `DOWN` | 直接砍掉小数 | `2` | `-2` |
| `UP` | 直接进位 | `3` | `-3` |

**日常用 `HALF_UP`（四舍五入）就够了**，涉及金融统计时可能要用 `HALF_EVEN`。

### 5.5 看位数

```java
BigDecimal v = new BigDecimal("123.4567");

v.precision();   // 7  ← 总共有多少位有效数字
v.scale();       // 4  ← 小数点后面有几位
```

用途：判断精度是否超标、比较两个 BigDecimal 的小数位数是否一致。

### 5.6 去掉多余的零

```java
new BigDecimal("1.500").stripTrailingZeros();   // → 1.5
new BigDecimal("1.500").toString();             // → 1.500（没变）
```

显示金额时经常需要 `stripTrailingZeros()` 去掉尾零。

### 5.7 别让 toString 给你科学计数法

数值特别小的时候，`toString()` 会输出成 `1E-7` 这种指数形式。要拿去显示或拼接字符串，用 `toPlainString()`：

```java
new BigDecimal("1E-7").toString();        // → "1E-7"      ← 显示出来是科学计数法
new BigDecimal("1E-7").toPlainString();   // → "0.0000001" ← 正常小数点
```

---

## 6. 两者互相转换

### 6.1 BigInteger → BigDecimal

```java
BigInteger big = new BigInteger("100");
BigDecimal d = new BigDecimal(big);              // → 100
```

### 6.2 BigDecimal → BigInteger

```java
new BigDecimal("123.99").toBigInteger();          // → 123（直接砍掉小数）
new BigDecimal("123.99").toBigIntegerExact();     // ← 抛异常！有小数就不行
```

**`toBigIntegerExact()` 遇到小数会抛 `ArithmeticException: Rounding necessary`。** 想安全转换就先 `setScale(0, RoundingMode.DOWN)` 再转。

---

## 7. 实战：金额计算的标准写法

```java
// 场景：商品单价 19.99，买 3 件，打 8.8 折，保留两位小数
BigDecimal price = new BigDecimal("19.99");
BigDecimal count = new BigDecimal("3");
BigDecimal discount = new BigDecimal("0.88");

BigDecimal total = price
        .multiply(count)                          // 59.97
        .multiply(discount)                       // 52.7736
        .setScale(2, RoundingMode.HALF_UP);       // 52.77

System.out.println("总价：" + total);
```

**要点回顾**：字符串构造 → 方法代替运算符 → 每一步都接住返回值 → 最后 `setScale` 定精度。

---

## 8. 一页速查表

### 运算

| 想要 | 写法 |
|---|---|
| 加 | `a.add(b)` |
| 减 | `a.subtract(b)` |
| 乘 | `a.multiply(b)` |
| 除（整数） | `a.divide(b)` |
| 除（带小数） | `a.divide(b, 位数, RoundingMode.HALF_UP)` |
| 取余 | `a.remainder(b)` |
| 绝对值 | `a.abs()` |
| 幂 | `a.pow(n)` |

### 比较

| 想要 | 写法 |
|---|---|
| 比较大小 | `a.compareTo(b)`（返回 -1 / 0 / 1） |
| 判断相等（数值） | `a.compareTo(b) == 0` |
| 判断相等（**别用**） | `a.equals(b)` ❌ |
| 取大的 | `a.max(b)` |

### BigDecimal 常用

| 想要 | 写法 |
|---|---|
| 创建 | `new BigDecimal("0.1")` ✅ / `BigDecimal.valueOf(0.1)` ✅ / `new BigDecimal(0.1)` ❌ |
| 保留几位小数 | `a.setScale(2, RoundingMode.HALF_UP)` |
| 转字符串 | `a.toPlainString()`（避免科学计数法） |
| 去尾零 | `a.stripTrailingZeros()` |
| 位数 | `a.precision()`（总位数） / `a.scale()`（小数位数） |

### 类型转换

| 方向 | 写法 |
|---|---|
| int/long → BigInteger | `BigInteger.valueOf(100)` |
| 字符串 → BigInteger | `new BigInteger("123456789012345678901234567890")` |
| BigInteger → int | `big.intValue()` |
| BigInteger → BigDecimal | `new BigDecimal(big)` |
| BigDecimal → BigInteger | `bigDec.toBigInteger()` |

---

## 9. 常见错误速查（照着避开）

| ❌ 错误写法 | ✅ 正确写法 | 后果 |
|---|---|---|
| `big + big` | `big.add(big)` | 编译错误 |
| `big == big2` | `big.compareTo(big2) == 0` | 比的是地址，结果不可靠 |
| `bigDec.equals(bigDec2)` | `bigDec.compareTo(bigDec2) == 0` | `1.0` 和 `1.00` 判为不等 |
| `new BigDecimal(0.1)` | `new BigDecimal("0.1")` | 得到一长串误差 |
| `a.divide(b)`（除不尽时） | `a.divide(b, 10, RoundingMode.HALF_UP)` | 直接抛异常 |
| `money.add(x);`（不接返回值） | `money = money.add(x);` | 等于没算 |
| `(int) big` | `big.intValue()` | 编译错误，BigInteger 不能强转 |

---

## 10. 什么时候用哪个（决策图）

```
要算的数会超过 19 位（long 放不下）？
    └─ 是 → BigInteger（整数）或 BigDecimal（可能带小数）

要算小数（钱、利率、分数、测量值）？
    └─ 是 → BigDecimal

只是计数、下标、普通循环？
    └─ 用 int / long 就行，别用 Big 系列（慢十倍以上）
```

**性能提醒**：`BigInteger` / `BigDecimal` 比基础类型**慢很多**（因为要变长存储、要调方法）。**只在真正需要大数或精确小数的地方用**，别为了"安全"无脑全用。

---

## 附：动手练习（建议直接 jshell 里敲）

```bash
jshell        # 启动
```

```java
import java.math.*;

// 练习 1：算 50 的阶乘
BigInteger r = BigInteger.ONE;
for (int i = 2; i <= 50; i++) r = r.multiply(BigInteger.valueOf(i));
System.out.println(r);

// 练习 2：算金额 199.99 × 3 × 0.88，保留两位小数
BigDecimal t = new BigDecimal("199.99")
        .multiply(new BigDecimal("3"))
        .multiply(new BigDecimal("0.88"))
        .setScale(2, RoundingMode.HALF_UP);
System.out.println(t + " 元");

// 练习 3：故意踩坑，看看 1.0 和 1.00 的 equals 结果
System.out.println(new BigDecimal("1.0").equals(new BigDecimal("1.00")));

// 练习 4：故意踩坑，看看 new BigDecimal(0.1) 出来什么
System.out.println(new BigDecimal(0.1));
```

---

## 记忆口诀

> **大数用字符串造，运算全是方法调；**
> **比较数值用 compareTo，等于零才算相等；**
> **除法必须给精度，四舍五入 HALF_UP；**
> **结果一定接回来，不可变类型不改自己。**
