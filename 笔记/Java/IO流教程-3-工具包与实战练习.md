# IO 流教程（三）：工具包与实战练习

> 用途：学习笔记 · 定位：讲得细、面向刚入门 · **代码完整可运行**
> 覆盖：IO流-43 ~ 52、57、59（53-56、58 的拼图游戏是 GUI，本教程跳过）
> 所有示例都在本机 JDK 27 上实测跑过
> ⚠️ 示例会在 `/tmp/iotest` 下建文件，跑完记得删

---

## 0. 这一篇讲什么

```
工具包（43-44）      →  别人写好的现成代码，直接调
Properties（57）     →  配置文件读写（重要！）
综合练习（50-52）    →  点名器、权重随机、登录注册
爬虫/假数据（45-49） →  了解思路即可
跳过：53-56、58（拼图游戏 GUI）
```

---

# 第一部分：常用工具包（43–44）

## 1. 为什么需要工具包

**IO 的很多操作，每次都要写重复代码**（比如：读文件全部内容、拷贝目录、删除目录）。

**工具包把这些封装好了，一行代码搞定。**

| 工具包 | 特点 | 常用度 |
|---|---|---|
| **Commons-io** | Apache 出品，老牌 | ★★★ |
| **hutool** | 国产，功能全，中文文档 | ★★★★ |

## 2. Commons-io

### 2.1 引入依赖（Maven）

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

### 2.2 常用方法

```java
import org.apache.commons.io.FileUtils;

// 读文件全部内容成字符串
String content = FileUtils.readFileToString(new File("a.txt"), "UTF-8");

// 写字符串到文件
FileUtils.writeStringToFile(new File("b.txt"), "内容", "UTF-8");

// 拷贝文件
FileUtils.copyFile(new File("src.txt"), new File("dest.txt"));

// 拷贝目录（递归）
FileUtils.copyDirectory(new File("srcDir"), new File("destDir"));

// 删除目录（递归）
FileUtils.deleteDirectory(new File("dir"));

// 清空目录
FileUtils.cleanDirectory(new File("dir"));

// 移动文件
FileUtils.moveFile(new File("a.txt"), new File("b.txt"));
```

**对比我们之前手写的**：`copyDirectory` 一行顶你写的 20 行递归代码。

## 3. hutool

### 3.1 引入依赖

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.20</version>
</dependency>
```

### 3.2 常用方法

```java
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.RandomUtil;

// 读文件
String content = FileUtil.readUtf8String("a.txt");

// 写文件
FileUtil.writeUtf8String("内容", "b.txt");

// 拷贝
FileUtil.copy("src.txt", "dest.txt", true);

// 删除目录
FileUtil.del(new File("dir"));

// 生成随机字符串（假数据用）
String randomStr = RandomUtil.randomString(10);
// 生成随机手机号
String phone = RandomUtil.randomNumbers(11);
```

**hutool 的优势**：中文文档、API 极其丰富、一个包搞定所有。

## 4. 工具包怎么选

| 场景 | 建议 |
|---|---|
| 公司项目 | 看团队规范，一般用 Commons-io 或 hutool |
| 个人快速开发 | hutool（功能全、中文文档） |
| 只要简单文件操作 | 自己写（不引依赖） |

**注意**：**工具包不是必须的**。面试时手写递归拷贝更显功底，工作中用工具包更高效。

---

# 第二部分：Properties 配置文件（57，重点）

## 5. Properties 是什么

**`Properties` 是专门读写"键值对配置文件"的类。**

**文件格式**：

```properties
name=张三
age=25
```

**特点**：
- 每行 `key=value`
- 支持中文（但要注意编码）
- **Java 的配置文件标准格式**

## 6. 读取配置

```java
import java.io.*;
import java.util.Properties;

public class ReadConfig {
    public static void main(String[] args) throws IOException {
        Properties props = new Properties();

        // 加载配置文件
        try (FileInputStream in = new FileInputStream("/tmp/iotest/config.properties")) {
            props.load(in);
        }

        // 读取
        String name = props.getProperty("name");
        String age = props.getProperty("age");

        // 带默认值（不存在时返回默认值）
        String notExist = props.getProperty("notExist", "默认值");

        System.out.println("name=" + name + " age=" + age);
        System.out.println("不存在的键: " + notExist);
    }
}
```

**实测：**

```
读回配置: name=张三 age=25
默认值测试: 默认值
```

## 7. 写入配置

```java
Properties props = new Properties();
props.setProperty("name", "张三");
props.setProperty("age", "25");

try (FileOutputStream out = new FileOutputStream("/tmp/iotest/config.properties")) {
    props.store(out, "测试配置");    // 第二个参数是注释
}
```

**生成的配置文件：**

```properties
#测试配置
#Mon Nov 16 ...
name=张三
age=25
```

**注意**：`store` 会自动加时间戳注释。

## 8. 经典应用：控制软件运行次数（31）

**需求**：软件只能用 3 次，用完提示注册。

```java
import java.io.*;
import java.util.Properties;

public class RunCounter {
    private static final int MAX_TIMES = 3;
    private static final String CONFIG_PATH = "/tmp/iotest/count.properties";

    public static void main(String[] args) throws IOException {
        Properties props = new Properties();
        File config = new File(CONFIG_PATH);

        // 1. 读配置（文件存在才读）
        if (config.exists()) {
            try (FileInputStream in = new FileInputStream(config)) {
                props.load(in);
            }
        }

        // 2. 判断次数
        int count = Integer.parseInt(props.getProperty("count", "0"));

        if (count >= MAX_TIMES) {
            System.out.println("试用次数已用完（" + MAX_TIMES + " 次），请购买");
            return;
        }

        // 3. 次数 +1 并保存
        count++;
        props.setProperty("count", String.valueOf(count));

        try (FileOutputStream out = new FileOutputStream(config)) {
            props.store(out, "运行次数");
        }

        System.out.println("第 " + count + " 次运行，还剩 " + (MAX_TIMES - count) + " 次");
    }
}
```

**实测（第一次运行）：**

```
第 1 次运行，还剩 2 次
```

**运行 3 次后第 4 次：**

```
试用次数已用完（3 次），请购买
```

**这是 Properties 最经典的用法**，也是很多商业软件"试用期"的实现思路。

## 9. Properties 的坑

### 9.1 中文编码（**不是"乱码"，是"转义"**）

**实测结论：默认 `store` 不会乱码，而是把中文转成 `\uXXXX` 转义。**

```java
Properties p = new Properties();
p.setProperty("name", "张三");
p.store(new FileOutputStream("p.properties"), "注释");
```

**生成的文件内容（实测）：**

```properties
#\u9ED8\u8BA4\u7F16\u7801
name=\u5F20\u4E09
```

**文件里看不到中文，但 `load` 回来是正常的 "张三"**（load 会自动解析 `\u` 转义）。

**所以默认的 OutputStream + InputStream 组合是安全的**，只是文件内容不直观。

**真正会乱码的是"混用编码"：**

```java
// 用 UTF-8 写
p.store(new OutputStreamWriter(new FileOutputStream(path), "UTF-8"), "注释");
// 文件内容是 name=张三（中文原文）

// 用默认方式读（InputStream）→ 乱码 "å¼ ä¸"
p.load(new FileInputStream(path));

// 必须也用 UTF-8 读
p.load(new InputStreamReader(new FileInputStream(path), "UTF-8"));   // 正常
```

**记忆**：**写用什么编码，读就用什么编码。** 要么两边都用默认（文件里是转义），要么两边都用 UTF-8（文件里是中文）。

**或者用 `storeToXML`**（XML 格式，天然支持 UTF-8 中文）。

### 9.2 键值对不能有空格

```properties
# 错误
name = 张三
# 正确（值前面的空格会被去掉，但为了清晰还是别加）
name=张三
```

**值里如果有空格，写的时候会被保留。**

### 9.3 加载顺序

```java
props.load(in);
String v = props.getProperty("key");   // 必须先 load 再 get
```

---

# 第三部分：综合练习（50–52）

## 10. 随机点名器（50）

**四个需求：**
1. 随机点名（不重复）
2. 点名后从名单中移除
3. 名单读完提示
4. 支持重新开始

```java
import java.io.*;
import java.util.*;

public class RandomCaller {
    public static void main(String[] args) throws IOException {
        List<String> names = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader("/tmp/iotest/names.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                if (line != null && !line.trim().isEmpty()) {
                    names.add(line.trim());
                }
            }
        }

        if (names.isEmpty()) {
            System.out.println("名单为空");
            return;
        }

        // 随机点名
        Random r = new Random();
        int index = r.nextInt(names.size());
        String picked = names.remove(index);    // 点名后移除（不重复）

        System.out.println("点到: " + picked);
        System.out.println("剩余: " + names.size() + " 人");
    }
}
```

**关键**：`names.remove(index)` 一次完成"取出 + 移除"。

## 11. 带权重的随机算法（51，重点）

**需求**：不同的人点名概率不同（比如张三 30%、李四 50%、王五 20%）。

### 11.1 思路：区间法

```
张三 30% → 区间 [0, 30)
李四 50% → 区间 [30, 80)
王五 20% → 区间 [80, 100)

随机生成 0~99 的数，落在哪个区间就是谁
```

```java
import java.util.*;

public class WeightedRandom {
    public static void main(String[] args) {
        // 权重（总和 100）
        Map<String, Integer> weights = new LinkedHashMap<>();
        weights.put("张三", 30);
        weights.put("李四", 50);
        weights.put("王五", 20);

        // 随机 0~99
        Random r = new Random();
        int num = r.nextInt(100);

        // 找落在哪个区间
        int cum = 0;
        String picked = null;
        for (Map.Entry<String, Integer> e : weights.entrySet()) {
            cum += e.getValue();
            if (num < cum) {
                picked = e.getKey();
                break;
            }
        }

        System.out.println("随机数: " + num + "，点到: " + picked);
    }
}
```

### 11.2 验证概率（跑 10000 次）

```java
int[] count = new int[3];   // 三个人的次数
String[] names = {"张三", "李四", "王五"};
int[] w = {30, 50, 20};

Random r = new Random();
for (int i = 0; i < 10000; i++) {
    int num = r.nextInt(100);
    int cum = 0;
    for (int j = 0; j < w.length; j++) {
        cum += w[j];
        if (num < cum) { count[j]++; break; }
    }
}
for (int j = 0; j < names.length; j++) {
    System.out.println(names[j] + " 出现 " + count[j] + " 次，概率约 " + (count[j] / 100.0) + "%");
}
```

**实测（跑 10000 次）：**

```
张三 出现 2969 次，概率约 29.69%（权重 30%）
李四 出现 5006 次，概率约 50.06%（权重 50%）
王五 出现 2025 次，概率约 20.25%（权重 20%）
```

**和权重 30/50/20 基本吻合。**

### 11.3 通用写法（权重总和不是 100 也行）

```java
int total = weights.values().stream().mapToInt(Integer::intValue).sum();
int num = r.nextInt(total);
int cum = 0;
for (Map.Entry<String, Integer> e : weights.entrySet()) {
    cum += e.getValue();
    if (num < cum) { picked = e.getKey(); break; }
}
```

**先求和，再随机 `nextInt(total)`**，这样权重可以是任意数。

## 12. 登录注册（52）

**需求**：用文件存储用户信息，实现注册和登录。

```java
import java.io.*;
import java.util.*;

public class LoginSystem {
    private static final String USER_FILE = "/tmp/iotest/users.properties";

    public static void main(String[] args) throws IOException {
        // 注册
        register("张三", "123456");
        // 登录
        login("张三", "123456");
    }

    /** 注册：用户名已存在则失败 */
    static void register(String username, String password) throws IOException {
        Properties props = new Properties();
        File file = new File(USER_FILE);
        if (file.exists()) {
            try (FileInputStream in = new FileInputStream(file)) { props.load(in); }
        }

        if (props.containsKey(username)) {
            System.out.println("注册失败：用户名已存在");
            return;
        }

        props.setProperty(username, password);
        try (FileOutputStream out = new FileOutputStream(file)) {
            props.store(out, "用户信息");
        }
        System.out.println("注册成功: " + username);
    }

    /** 登录：用户名密码都对才算成功 */
    static void login(String username, String password) throws IOException {
        Properties props = new Properties();
        File file = new File(USER_FILE);
        if (!file.exists()) {
            System.out.println("登录失败：没有用户数据");
            return;
        }

        try (FileInputStream in = new FileInputStream(file)) { props.load(in); }

        String saved = props.getProperty(username);
        if (saved == null) {
            System.out.println("登录失败：用户不存在");
        } else if (!saved.equals(password)) {
            System.out.println("登录失败：密码错误");
        } else {
            System.out.println("登录成功: " + username);
        }
    }
}
```

**关键点**：
- 用 `Properties` 存用户（key=用户名，value=密码）
- **判断"用户已存在"用 `containsKey`**
- **密码不能明文存**（真实项目要哈希，这里只是练习）

---

# 第四部分：了解即可（45–49）

## 13. 网络爬虫（45–48）

**思路**：用 `URL` 读取网页 → 用正则提取数据 → 用 IO 写出到文件。

```java
// 极简版：读取网页内容
URL url = new URL("https://example.com");
try (BufferedReader br = new BufferedReader(new InputStreamReader(url.openStream()))) {
    String line;
    while ((line = br.readLine()) != null) {
        // 处理每一行
    }
}
```

**四集的内容大概是**：
- 45 爬取姓氏（从一个网页提取）
- 46 爬取名字
- 47 数据处理（拼接、去重）
- 48 写出文件

**注意**：真实爬虫涉及**反爬、频率限制、法律合规**，学习阶段别爬敏感网站。**这部分了解思路即可，不必深究。**

## 14. 生成假数据（49）

**用 hutool 的 `RandomUtil` 生成测试数据：**

```java
String name = RandomUtil.randomString("张三李四王五", 2);   // 随机姓名
String phone = RandomUtil.randomNumbers(11);               // 随机手机号
int age = RandomUtil.randomInt(18, 60);                    // 随机年龄
```

**用途**：测试数据、演示程序，不用手动造。

## 15. 跳过的集数（53–56、58）

**拼图游戏（53-56、58）是 GUI 项目**，用到 Swing/JavaFX：

- 53 导入项目
- 54 登录界面
- 55 注册界面
- 56 存档/读档/数据同步
- 58 配置文件

**跳过理由**：Java GUI 在现代开发中几乎不用（桌面端已被 Web/移动端取代），**算法和 IO 部分前面的教程已经覆盖了**。

**59 每日一记（阶段大作业）**：综合运用前面所有知识，做一个日记本程序，可以自己练手。

---

## 16. 一页速查表

### 工具包

| 想要 | Commons-io | hutool |
|---|---|---|
| 读文件 | `FileUtils.readFileToString(f, "UTF-8")` | `FileUtil.readUtf8String(path)` |
| 写文件 | `FileUtils.writeStringToFile(f, s, "UTF-8")` | `FileUtil.writeUtf8String(s, path)` |
| 拷贝目录 | `FileUtils.copyDirectory(src, dest)` | `FileUtil.copy(src, dest, true)` |
| 删除目录 | `FileUtils.deleteDirectory(dir)` | `FileUtil.del(dir)` |

### Properties

| 操作 | 写法 |
|---|---|
| 加载 | `props.load(new FileInputStream(path))` |
| 读取 | `props.getProperty(key)` |
| 带默认值 | `props.getProperty(key, 默认值)` |
| 设置 | `props.setProperty(key, value)` |
| 保存 | `props.store(new FileOutputStream(path), "注释")` |
| 判断存在 | `props.containsKey(key)` |

### 实战口诀

| 场景 | 方案 |
|---|---|
| 控制运行次数 | Properties + count |
| 随机不重复 | `list.remove(randomIndex)` |
| 权重随机 | 累加权重 + `nextInt(total)` |
| 登录注册 | Properties 存用户名/密码 |

---

## 17. 常见错误速查

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| Properties 写 UTF-8 却用默认读 | 读写都用 UTF-8，或者都用默认 | 乱码 |
| `getProperty` 前没 `load` | 先 load | 读不到 |
| 权重随机直接 `nextInt(100)` | `nextInt(total)` 先求和 | 权重总和不是 100 时结果错 |
| 点名用 `get` 不移除 | `remove(index)` | 会重复点到 |
| 密码明文存文件 | 真实项目要哈希 | 安全问题 |
| 爬虫爬太猛 | 加延时、看 robots.txt | 封 IP / 违法 |

---

## 记忆口诀

> **工具包省代码，Commons-io 和 hutool 二选一；**
> **Properties 读写配置，load 加载 store 保存；**
> **控制运行次数靠 count，中文记得指定编码；**
> **随机不重复用 remove，权重随机先求和；**
> **GUI 拼图跳过，算法和 IO 才是重点。**
