# Java File 类教程（小白版）

> 用途：学习笔记 · 定位：讲得细、面向刚入门
> 覆盖：File-01 ~ 11
> 所有示例和输出都是在本机 JDK 27 上实测跑出来的
> ⚠️ 本文示例会在 `/tmp` 下建临时目录，跑完记得删

---

## 0. File 是什么

**File 是"文件和目录的路径"的抽象表示。**

**先搞清楚一个关键认知**：

> **`File` 对象只是"一个路径"，不一定是真实存在的文件。**

```java
File f = new File("/tmp/不存在的文件.txt");

f.exists();     // false —— 路径可以指向不存在的东西
f.getName();    // "不存在的文件.txt" —— 但名字还是能取
```

**创建 File 对象 ≠ 创建文件。** 要真的建文件，得调 `createNewFile()`。

---

## 1. 三种构造方法（01）

```java
// ① 直接给路径
File f1 = new File("/tmp/filetest");

// ② 父路径（字符串）+ 子文件名
File f2 = new File("/tmp/filetest", "a.txt");

// ③ 父路径（File 对象）+ 子文件名
File f3 = new File(new File("/tmp/filetest"), "b.txt");
```

**实测结果：**

```
File(String path):                  /tmp/filetest
File(String parent, String child):  /tmp/filetest/a.txt
File(File parent, String child):    /tmp/filetest/b.txt
```

**注意**：
- 路径是**相对路径**时，相对于"当前工作目录"（不是项目根目录）
- **Windows 用 `\`，Linux/Mac 用 `/`**，跨平台建议用 `File.separator` 或直接写 `/`（Java 在 Windows 上也能识别 `/`）

---

## 2. 判断方法（02）

```java
File f = new File("/tmp/filetest");

f.exists();        // true    路径是否存在
f.isFile();        // false   是不是文件
f.isDirectory();   // true    是不是目录
f.canRead();       // true    可读吗
f.canWrite();      // true    可写吗
f.isHidden();      // 是否隐藏
```

**实测：**

```
f1.exists(): true
f1.isDirectory(): true
f1.isFile(): false
f2.exists(): false       ← 路径不存在
```

**判断逻辑顺序**：**先 `exists()`，再判断是文件还是目录**。对不存在的路径调 `isFile()` 返回 false，不会报错。

---

## 3. 获取方法（02）

```java
File f = new File("/tmp/filetest");

f.getName();           // "filetest"     文件名/目录名
f.getAbsolutePath();   // "/tmp/filetest" 绝对路径
f.getPath();           // "/tmp/filetest" 构造时给的路径
f.length();            // 字节数（文件大小）
f.getParent();         // "/tmp"          父路径字符串
f.getParentFile();     // File 对象
f.lastModified();      // 最后修改时间（毫秒时间戳）
```

**实测：**

```
getName(): filetest
getAbsolutePath(): /tmp/filetest
getPath(): /tmp/filetest
length(): 40          ← 注意！目录返回的是目录自身大小，不是内容总大小
getParent(): /tmp
```

### 3.1 坑：`length()` 对目录不返回"总大小"

```java
File dir = new File("/tmp/filetest");   // 目录里有几个文件
dir.length();    // 40  ← 不是里面文件的总和
```

**这就是练习 5「统计文件夹大小」需要递归的原因**——`length()` 只算目录项本身。

### 3.2 坑：`getPath()` 和 `getAbsolutePath()` 不一样

```java
File f = new File("a.txt");        // 相对路径

f.getPath();           // "a.txt"            ← 原样返回
f.getAbsolutePath();   // "/home/xxx/a.txt"  ← 转成绝对路径
```

**要绝对路径就用 `getAbsolutePath()`。**

---

## 4. 创建（03）

```java
// 创建文件（父目录必须存在，否则抛 IOException）
File f = new File("/tmp/filetest/a.txt");
f.createNewFile();          // true（创建成功）

f.createNewFile();          // false（已存在）
```

```java
// 创建单级目录（父目录必须存在）
File dir = new File("/tmp/filetest/sub");
dir.mkdir();                // true

// 创建多级目录（父目录不存在也能建）
File deep = new File("/tmp/filetest/a/b/c");
deep.mkdirs();              // true
```

**`mkdir()` vs `mkdirs()`：**

| 方法 | 能建多级吗 | 用哪个 |
|---|---|---|
| `mkdir()` | ❌ 只能建一级 | 确定父目录存在时 |
| `mkdirs()` | ✅ 父目录不存在也建 | **推荐，省心** |

**实测：**

```
mkdir(): true     ← /tmp/filetest/sub
mkdirs(): true    ← /tmp/filetest/a/b/c
```

---

## 5. 删除（03）

```java
File f = new File("/tmp/filetest/a.txt");
f.delete();        // true（删除成功）
```

**坑：只能删空目录**

```java
File dir = new File("/tmp/filetest/sub1");   // 里面有文件
dir.delete();      // false  ← 删不掉！
```

**实测：**

```
delete() 非空目录: false
```

**要删非空目录，必须递归删除**（练习 4）。

**另一个坑**：`delete()` 是**永久删除**，不进回收站，删了找不回来。

---

## 6. 遍历（04–05）

### 6.1 list()：返回文件名

```java
File dir = new File("/tmp/filetest2");
String[] names = dir.list();
// [b.java, a.txt, empty, sub1]
```

**返回的是 `String[]`（只有名字，没有路径）。**

### 6.2 listFiles()：返回 File 对象

```java
File[] files = dir.listFiles();
for (File f : files) {
    System.out.println(f.getName() + " 是文件?" + f.isFile());
}
```

**实测：**

```
b.java (文件?true)
a.txt (文件?true)
empty (文件?false)
sub1 (文件?false)
```

**推荐用 `listFiles()`**，因为拿到的是 File 对象，可以直接调 `isFile()`、`length()` 等方法。

### 6.3 传过滤器：只要特定文件

```java
// 只要 .txt 文件
File[] txts = dir.listFiles(f -> f.getName().endsWith(".txt"));
// [/tmp/filetest2/a.txt]
```

**注意**：`listFiles` 的 lambda 参数是 `File`，返回 `boolean`。`true` 表示保留。

### 6.4 坑：listFiles() 可能返回 null

```java
File dir = new File("/tmp/filetest2");
File[] files = dir.listFiles();

for (File f : files) { }   // 如果 files 是 null，会 NPE
```

**什么时候返回 null？**
- 路径不存在
- 路径不是目录（是文件）
- 没有权限

**标准写法：**

```java
File[] files = dir.listFiles();
if (files == null) return;      // 先判空
for (File f : files) { ... }
```

---

## 7. 六个综合练习（06–11）

> **这六个练习的核心都是"递归"**，是 File 类最有价值的部分。

### 练习 1：创建文件

```java
File f = new File("/tmp/filetest/a.txt");
f.createNewFile();
```

**注意**：父目录不存在会抛 `IOException`，所以要先 `mkdirs()`。

### 练习 2：单个文件夹查找文件（不递归）

**只查当前这一层：**

```java
ArrayList<File> findInDir(File dir, String suffix) {
    ArrayList<File> result = new ArrayList<>();
    File[] files = dir.listFiles();
    if (files == null) return result;

    for (File f : files) {
        if (f.isFile() && f.getName().endsWith(suffix)) {
            result.add(f);
        }
    }
    return result;
}

findInDir(new File("/tmp/filetest2"), ".txt");
// [/tmp/filetest2/a.txt]
```

### 练习 3：遍历所有层级查找（递归）

**子文件夹也要查：**

```java
ArrayList<File> findAll(File dir, String suffix) {
    ArrayList<File> result = new ArrayList<>();
    File[] files = dir.listFiles();
    if (files == null) return result;

    for (File f : files) {
        if (f.isFile() && f.getName().endsWith(suffix)) {
            result.add(f);
        } else if (f.isDirectory()) {
            result.addAll(findAll(f, suffix));     // ← 递归
        }
    }
    return result;
}

findAll(new File("/tmp/filetest2"), ".java");
// [b.java, sub1/sub2/d.java]   子目录里的也找到了
```

**递归三要素**（所有递归题都适用）：
1. **出口**：`files == null` 时返回
2. **自己做**：判断当前文件
3. **交给子问题**：`findAll(f)` 递归

### 练习 4：删除文件夹（递归）

```java
void deleteDir(File dir) {
    File[] files = dir.listFiles();

    // ① 先把里面的都删掉
    if (files != null) {
        for (File f : files) {
            if (f.isFile()) {
                f.delete();          // 文件直接删
            } else {
                deleteDir(f);        // 目录递归删
            }
        }
    }

    // ② 最后删自己
    dir.delete();
}
```

**实测：**

```
删除前存在: true
删除后存在: false
```

**关键**：`dir.delete()` 必须放在**最后**，否则目录非空删不掉。

### 练习 5：统计文件夹大小（递归）

```java
long getSize(File dir) {
    File[] files = dir.listFiles();
    if (files == null) return 0;

    long total = 0;
    for (File f : files) {
        if (f.isFile()) {
            total += f.length();      // 文件：累加大小
        } else {
            total += getSize(f);      // 目录：递归统计
        }
    }
    return total;
}
```

**实测：22 字节**（4 个文件大小之和）

**为什么必须递归？** 因为 `length()` 对目录只返回目录项本身的大小，不是内容总和。

### 练习 6：统计各种文件数量（递归）

```java
void countFiles(File dir, Map<String, Integer> map) {
    File[] files = dir.listFiles();
    if (files == null) return;

    for (File f : files) {
        if (f.isFile()) {
            String name = f.getName();
            String ext = name.contains(".") ? name.substring(name.lastIndexOf(".") + 1) : "无后缀";
            map.put(ext, map.getOrDefault(ext, 0) + 1);
        } else {
            countFiles(f, map);       // 递归子目录
        }
    }
}

Map<String, Integer> map = new HashMap<>();
countFiles(new File("/tmp/filetest2"), map);
// {txt=2, java=2}
```

**这段代码用到了前面学的两个东西**：
- `getOrDefault`（Map 的计数技巧）
- 递归（处理子目录）

### 7.1 六个练习的递归套路

```java
// 通用模板
void 处理(File dir) {
    File[] files = dir.listFiles();
    if (files == null) return;              // ① 出口

    for (File f : files) {
        if (f.isFile()) {
            // 处理文件
        } else {
            处理(f);                          // ② 递归子目录
        }
    }
}
```

**所有递归题都是这个骨架**，区别只在 `if (f.isFile())` 里干什么。

---

## 8. 一页速查表

### 构造

| 方式 | 写法 |
|---|---|
| 路径 | `new File("path")` |
| 父+子（字符串） | `new File(parent, child)` |
| 父+子（File） | `new File(parentFile, child)` |

### 判断

| 作用 | 方法 |
|---|---|
| 存在吗 | `exists()` |
| 是文件吗 | `isFile()` |
| 是目录吗 | `isDirectory()` |
| 可读/可写 | `canRead()` / `canWrite()` |

### 获取

| 作用 | 方法 |
|---|---|
| 文件名 | `getName()` |
| 绝对路径 | `getAbsolutePath()` |
| 原路径 | `getPath()` |
| 大小（字节） | `length()` |
| 父路径 | `getParent()` / `getParentFile()` |
| 修改时间 | `lastModified()` |

### 创建/删除

| 作用 | 方法 | 注意 |
|---|---|---|
| 创建文件 | `createNewFile()` | 父目录要存在 |
| 创建单级目录 | `mkdir()` | 父目录要存在 |
| 创建多级目录 | `mkdirs()` | 推荐 |
| 删除 | `delete()` | **只能删空目录** |

### 遍历

| 作用 | 方法 |
|---|---|
| 文件名数组 | `list()` |
| File 对象数组 | `listFiles()` |
| 过滤 | `listFiles(filter)` |

---

## 9. 常见错误速查

| ❌ 错法 | ✅ 对法 | 后果 |
|---|---|---|
| `new File(...)` 以为就建了文件 | 调 `createNewFile()` | 文件不存在 |
| `createNewFile()` 不先建父目录 | 先 `mkdirs()` | IOException |
| `delete()` 删非空目录 | 递归删除 | 返回 false，删不掉 |
| `listFiles()` 不判 null | 先 `if (files == null) return` | NPE |
| 用 `length()` 求文件夹大小 | 递归累加文件 | 结果只是目录本身大小 |
| `getPath()` 当绝对路径 | 用 `getAbsolutePath()` | 可能是相对路径 |
| 递归忘写出口 | `if (files == null) return` | StackOverflowError |
| `delete()` 以为能恢复 | 没有回收站 | 永久丢失 |

---

## 10. 动手练习（直接 jshell 敲）

```bash
jshell
```

```java
import java.io.*;
import java.util.*;

// ① 建多级目录和文件
File dir = new File("/tmp/demo");
dir.mkdirs();
File f1 = new File(dir, "a.txt");
f1.createNewFile();
System.out.println("创建成功: " + f1.exists());

// ② 判断和获取
System.out.println(f1.getName() + " / " + f1.length() + " 字节");

// ③ 遍历
File[] files = dir.listFiles();
System.out.println("目录下文件数: " + (files == null ? 0 : files.length));

// ④ 递归统计（改成一行写）
long size = 0;
for (File f : dir.listFiles()) size += f.length();
System.out.println("总大小: " + size);

// ⑤ 清理
f1.delete();
dir.delete();
```

---

## 记忆口诀

> **File 只是路径，不一定真存在；**
> **mkdir 建一级，mkdirs 建多级；**
> **delete 只能删空目录，非空要递归；**
> **length 对目录不算总大小，统计要递归；**
> **listFiles 可能返回 null，先判空；**
> **递归三要素：出口、自己干、交给子问题。**
