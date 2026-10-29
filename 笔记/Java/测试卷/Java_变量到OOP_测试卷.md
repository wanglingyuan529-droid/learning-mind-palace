# Java 程序设计阶段测试卷

**考核范围：变量与数据类型 → 面向对象（类与对象、封装、继承、多态）**
**满分：100 分　　建议用时：90 分钟　　题型：单项选择 / 多项选择 / 判断 / 代码分析**

---

## 考生须知

1. 本卷共四大题，共 36 小题，闭卷作答。
2. 多选题**错选、少选、多选均不得分**。
3. 代码分析题请写出**运行结果**（输出内容），有多个输出请按行依次写出，注意空格与换行。
4. 答题卡在卷末，请在答题卡上作答，或将答案按题号依次回复。

---

## 一、单项选择题（本大题共 12 小题，每小题 3 分，共 36 分）

**1.** 下列关于 Java 标识符的说法，正确的是（a）

- A. `2num` 是合法标识符
- B. `user-name` 是合法标识符
- C. `_count` 是合法标识符
- D. `class` 可以作为变量名使用

**2.** 下列赋值语句中，能够**直接通过编译**的是（a）

- A. `float f = 3.14;`
- B. `byte b = 128;`
- C. `long l = 100;`
- D. `char c = "A";`

**3.** 在 Java 中，局部变量与成员变量的区别，下列说法**错误**的是（d）

- A. 成员变量有默认初始值，局部变量没有默认初始值
- B. 局部变量必须先赋值再使用
- C. 局部变量的作用范围是其所在的方法或语句块内部
- D. 成员变量只能在构造方法中赋值

**4.** 执行 `int x = 5; int y = x++ + ++x;` 后，`x` 和 `y` 的值分别是（a）

- A. `x = 7, y = 12`
- B. `x = 7, y = 13`
- C. `x = 6, y = 12`
- D. `x = 6, y = 13`

**5.** 执行下列代码后，输出结果是（b）

```java
String s1 = "hello";
String s2 = new String("hello");
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

- A. `true` 和 `true`
- B. `true` 和 `false`
- C. `false` 和 `true`
- D. `false` 和 `false`

**6.** 已知某类中已定义方法 `public void show(int a)`，下列方法中能与其构成**合法重载**的是（　c　）

- A. `public int show(int a)`
- B. `public void show(String a)`
- C. `private void show(int a)`
- D. `public void show(int b)`

**7.** 关于构造方法，下列说法正确的是（）

- A. 构造方法可以声明返回值类型为 `void`
- B. 如果一个类没有显式定义构造方法，编译时不会生成任何构造方法
- C. 构造方法名必须与类名相同，且不能有返回值类型声明
- D. 构造方法只能被 `new` 调用一次，不能被其他构造方法调用

**8.** Java 中关于类之间继承的说法，正确的是（　　）

- A. 一个类可以同时继承多个父类
- B. 子类可以继承父类中被 `private` 修饰的成员变量
- C. 子类继承父类后，可以直接访问父类中 `private` 修饰的成员
- D. 构造方法可以被继承

**9.** 下列代码的运行结果是（　　）

```java
class Animal {
    public void eat() { System.out.println("动物吃东西"); }
}
class Dog extends Animal {
    public void eat() { System.out.println("狗吃骨头"); }
}
public class Test {
    public static void main(String[] args) {
        Animal a = new Dog();
        a.eat();
    }
}
```

- A. `动物吃东西`
- B. `狗吃骨头`
- C. 编译错误
- D. 运行时报错

**10.** 关于 `static` 关键字，下列说法正确的是（　　）

- A. 静态方法中可以直接访问本类的非静态成员变量
- B. 静态变量被该类的所有对象共享，可以通过 `类名.变量名` 访问
- C. 静态方法中可以使用 `this` 关键字
- D. 被 `static` 修饰的变量，每个对象都有一份独立副本

**11.** 关于修饰符可见范围，下列说法正确的是（　　）

- A. `private` 修饰的成员，同一个包中的其他类可以访问
- B. 默认（不写修饰符）的成员，任何包中的子类都可以访问
- C. `protected` 修饰的成员，不同包中的非子类可以访问
- D. `public` 修饰的成员，任何位置都可以访问

**12.** 下列属于**编译期错误**（无法通过编译）的是（　　）

- A. 子类重写父类方法时，把 `public` 改为 `private`
- B. 子类重写父类方法时，返回类型改为父类返回类型的子类
- C. 子类中定义与父类同名的成员变量
- D. 子类中定义与父类方法名相同但参数列表不同的方法

---

## 二、多项选择题（本大题共 6 小题，每小题 4 分，共 24 分；错选、少选、多选不得分）

**13.** 下列属于 Java 基本数据类型的是（　　）

- A. `int`
- B. `String`
- C. `boolean`
- D. `double`
- E. `char`

**14.** 关于方法重载（Overload）与重写（Override），说法正确的有（　　）

- A. 重载发生在同一个类中，方法名相同、参数列表不同
- B. 重写发生在子类与父类之间，方法签名相同
- C. 重载可以改变返回值类型
- D. 重写时子类方法的访问权限不能小于父类方法的访问权限
- E. 重写时子类方法可以抛出比父类更宽的受检异常

**15.** 关于 `final` 关键字，说法正确的有（　　）

- A. `final` 修饰的类不能被继承
- B. `final` 修饰的方法不能被子类重写
- C. `final` 修饰的成员变量必须在声明时或构造方法中完成赋值
- D. `final` 修饰的引用类型变量，其指向对象的内部属性也不能被修改
- E. `final` 和 `abstract` 可以同时修饰同一个方法

**16.** 下列关于抽象类与抽象方法，说法正确的有（　　）

- A. 抽象类不能被实例化（不能用 `new` 创建对象）
- B. 抽象类中可以有非抽象方法
- C. 抽象类中可以有构造方法
- D. 一个类只要包含抽象方法，就必须声明为抽象类
- E. 子类继承抽象类后，可以不实现父类的抽象方法而直接使用

**17.** 关于多态，说法正确的有（　　）

- A. 多态的前提是继承（或接口实现）、方法重写、父类引用指向子类对象
- B. 通过父类引用调用被重写的方法时，执行的是子类重写后的方法
- C. 通过父类引用调用时，访问到的成员变量是父类的成员变量
- D. 父类引用可以直接调用子类特有的方法，无需类型转换
- E. 多态可以提高程序的扩展性，降低耦合

**18.** 关于封装，说法正确的有（　　）

- A. 封装通常做法是把成员变量用 `private` 修饰
- B. 通过提供 `public` 的 `getter` / `setter` 方法访问私有属性
- C. 封装的作用之一是可以在 `setter` 中对数据做合法性校验
- D. 封装意味着一个类中不能有 `public` 方法
- E. 封装可以提高代码的安全性和可维护性

---

## 三、判断题（本大题共 10 小题，每小题 2 分，共 20 分；正确画 √，错误画 ×）

**19.** 在一个方法中定义的局部变量，可以在该方法之外的其他方法中被访问。（　　）

**20.** `int` 类型变量可以直接赋值给 `double` 类型变量，属于自动类型转换。（　　）

**21.** `byte b = (byte) 200;` 编译可以通过，但会发生精度损失。（　　）

**22.** `String` 是基本数据类型，所以可以用 `==` 比较两个字符串的内容是否相同。（　　）

**23.** 子类构造方法中，如果没有显式调用 `super(...)`，系统会自动调用父类的无参构造方法。（　　）

**24.** 一个类的所有对象共享同一个 `static` 变量。（　　）

**25.** 被 `final` 修饰的方法可以被子类重写。（　　）

**26.** 多态中，通过父类引用访问成员变量时，取的是子类的值。（　　）

**27.** 创建子类对象时，先执行父类构造方法，再执行子类构造方法。（　　）

**28.** 抽象类中的抽象方法可以有方法体。（　　）

---

## 四、代码分析题（本大题共 8 小题，每小题 2.5 分，共 20 分；请写出程序运行结果）

**29.** 写出下列代码的输出结果。

```java
public class Q29 {
    public static void main(String[] args) {
        int a = 10;
        int b = a;
        b = 20;
        System.out.println(a + " " + b);
    }
}
```

**30.** 写出下列代码的输出结果。

```java
class A {
    public void print(int i) {
        System.out.println("int: " + i);
    }
}
class B extends A {
    public void print(double d) {
        System.out.println("double: " + d);
    }
}
public class Q30 {
    public static void main(String[] args) {
        A a = new B();
        a.print(5);
    }
}
```

**31.** 写出下列代码的输出结果。

```java
class Counter {
    static int count = 0;
    Counter() {
        count++;
    }
}
public class Q31 {
    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
        Counter c3 = new Counter();
        System.out.println(Counter.count);
    }
}
```

**32.** 写出下列代码的输出结果。

```java
public class Q32 {
    public static void main(String[] args) {
        String s1 = "java";
        String s2 = "java";
        String s3 = new String("java");
        System.out.println(s1 == s2);
        System.out.println(s1 == s3);
        System.out.println(s1.equals(s3));
    }
}
```

**33.** 写出下列代码的输出结果。

```java
class Person {
    public Person() {
        System.out.println("Person");
    }
}
class Student extends Person {
    public Student() {
        System.out.println("Student");
    }
}
public class Q33 {
    public static void main(String[] args) {
        Student s = new Student();
    }
}
```

**34.** 写出下列代码的输出结果。

```java
class Father {
    int num = 10;
    public void show() {
        System.out.println("Father show");
    }
}
class Son extends Father {
    int num = 20;
    public void show() {
        System.out.println("Son show");
    }
}
public class Q34 {
    public static void main(String[] args) {
        Father f = new Son();
        System.out.println(f.num);
        f.show();
    }
}
```

**35.** 写出下列代码的输出结果。

```java
public class Q35 {
    public static void main(String[] args) {
        int i = 1;
        int j = i++ + i++ + ++i;
        System.out.println("i = " + i);
        System.out.println("j = " + j);
    }
}
```

**36.** 写出下列代码的输出结果。

```java
public class Q36 {
    public static void change(int x, int[] arr) {
        x = 100;
        arr[0] = 99;
    }
    public static void main(String[] args) {
        int a = 10;
        int[] nums = {1, 2, 3};
        change(a, nums);
        System.out.println(a + " " + nums[0]);
    }
}
```

---

## 答题卡

> 请按题号作答；也可以直接把答案按题号依次回复给我，我来批改。

**一、单项选择题（每题 3 分）**

| 题号 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 答案 |   |   |   |   |   |   |   |   |   |    |    |    |

**二、多项选择题（每题 4 分，错选/少选/多选均不得分）**

| 题号 | 13 | 14 | 15 | 16 | 17 | 18 |
|:---:|:--:|:--:|:--:|:--:|:--:|:--:|
| 答案 |    |    |    |    |    |    |

**三、判断题（每题 2 分，√ 或 ×）**

| 题号 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 |
|:---:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 答案 |    |    |    |    |    |    |    |    |    |    |

**四、代码分析题（每题 2.5 分）**

| 题号 | 输出结果 |
|:---:|:---|
| 29 |  |
| 30 |  |
| 31 |  |
| 32 |  |
| 33 |  |
| 34 |  |
| 35 |  |
| 36 |  |

---

**得分统计（阅卷后填写）**

| 题型 | 一、单选 | 二、多选 | 三、判断 | 四、代码分析 | 总分 |
|:---|:---:|:---:|:---:|:---:|:---:|
| 满分 | 36 | 24 | 20 | 20 | 100 |
| 得分 |  |  |  |  |  |
