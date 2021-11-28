# 深入剖析 Java 新特性

### 开篇词 | 拥抱 Java 新特性，像设计者一样工作和思考

### 01 | JShell：怎么快速验证简单的小问题？

我们使用 JShell 工具的主要目的之一，就是观察评估我们编写的代码片段。

```bash
jshell -v
jshell> System.out.println("Hello World!")
jshell> /exit
```

JShell 是一个有状态的工具，这样我们就能够很方便地处理多个有关联的语句了。

变量的声明可以重复，也可以转换类型，就像上一个声明并不存在一样。

```bash
jshell> String greeting;
jshell> String language = "English";
jshell> greeting = switch (language) {
   ...>     case "English" -> "Hello";
   ...>     case "Spanish" -> "Hola";
   ...>     case "Chinese" -> "NiHao";
   ...>     default -> throw new RuntimeException("Unsupported language");};
jshell> System.out.println(greeting);
jshell> String greeting = "Hola";
jshell> Integer greeting;
```

JShell 支持表达式的输入。有了独立的表达式，我们就可以直接评估表达式，而不再需要把它附着在一个语句上了。

```bash
jshell> 1 + 1
jshell> "Hello, world" == new String("Hello, world");
```

### 02 | 文字块：怎么编写所见即所得的字符串？

**所见即所得的文字块**

文字块是使用一个新的形式，而不是传统的形式，来表达字符串的。通过这个新的形式，文字块尝试消除换行、连接符、转义字符的影响，使得文字对齐和必要的占位符更加清晰，从而简化多行文字字符串的表达。

需要注意的是，开始分隔符必须单独成行；三个双引号字符后面的空格和换行符都属于开始分隔符。所以，一个文字块至少有两行代码。即使是一个空字符，结束分隔符也不能和开始分隔符放在同一行代码里。

同样需要注意的是，结束分隔符只有一个由三个双引号字符组成的序列。结束分隔符之前的字符，包括换行符，都属于文字块的有效内容。

```bash
jshell> String s = """
   ...> OneLine""";
s ==> "OneLine"
jshell> String s = """
   ...> TwoLines
   ...> """
s ==> "TwoLines\n"
```

**文字块的编译过程**

在编译期，文字块要顺序通过如下三个不同的编译步骤：

1. 为了降低不同平台间换行符的表达差异，编译器把文字内容里的换行符统一转换成 LF（\u000A）；

2. 为了能够处理 Java 源代码里的缩进空格，要删除所有文字内容行和结束分隔符共享的前导空格，以及所有文字内容行的尾部空格；

3. 最后处理转义字符，这样开发人员编写的转义序列就不会在第一步和第二步被修改或删除。

使用传统方式声明的字符串和使用文字块声明的字符串，它们的内容是一样的，而且指向的是同一个对象。

这就说明了，文字块是在编译期处理的，并且在编译期被转换成了常量字符串，然后就被当作常规的字符串了。

**巧妙的结束分隔符**

我们使用小数点号"."表示编译期要删除的前导空格，使用叹号"!"表示编译期要删除的尾部空格。

```java
String textBlock = """
........<!DOCTYPE html>
........<html>
........    <body>
........        <h1>"Hello World!"</h1>!!!!
........    </body>
........</html>
........""";

<!DOCTYPE html>
<html>
    <body>
        <h1>"Hello World!"</h1>
    </body>
</html>
```

```java
String textBlock = """
....    <!DOCTYPE html>
....    <html>
....        <body>
....            <h1>"Hello World!"</h1>!!!!
....        </body>
....    </html>
....""";

    <!DOCTYPE html>
    <html>
        <body>
            <h1>"Hello World!"</h1>
        </body>
    </html>
```

```java
String textBlock = """
........<!DOCTYPE html>
........<html>
........    <body>
........        <h1>"Hello World!"</h1>!!!!
........    </body>
........</html>
........!!!!""";

<!DOCTYPE html>
<html>
    <body>
        <h1>"Hello World!"</h1>
    </body>
</html>
```

**尾部空格还能回来吗？**

"\s" 空格转义符

```java
String textBlock = """
........<!DOCTYPE html>    \s!!!!
........<html>             \s
........    <body>!!!!!!!!!!
........        <h1>"Hello World!"</h1>
........    </body>
........</html>
........""";

<!DOCTYPE html>     
<html>              
    <body>
        <h1>"Hello World!"</h1>
    </body>
</html>
```

**该怎么表达长段落？**

"\\" 换行转义符

```java
String textBlock = """
        <!DOCTYPE html>
        <html>
            <body>
                <h1>"Hello \
        World!"</h1>
            </body>
        </html>
        """;

<!DOCTYPE html>
<html>
    <body>
        <h1>"Hello World!"</h1>
    </body>
</html>
```

### 03 | 档案类：怎么精简地表达不可变数据？

Java 档案类是用来表示不可变数据的透明载体。

天生的多线程安全 / 简化的代码

```java
public final class Circle implements Shape {
    public final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

声明档案类

```java
public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

```java
Circle circle = new Circle(10.0);
double radius = circle.radius();
```

档案类内置了这些方法的缺省实现：构造方法、equals 方法、hashCode 方法、toString 方法、不可变数据的读取方法。

```java
public record Circle(double radius) implements Shape {
    public Circle {
        if (radius < 0) {
            throw new IllegalArgumentException(
                "The radius of a circle cannot be negative [" + radius + "]");
        }
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

### 04 | 封闭类：怎么刹住失控的扩展性？

面向对象编程的最佳实践之一，就是要把可扩展性限制在可以预测和控制的范围内，而不是无限的可扩展性。

由于继承的安全问题，我们在设计 API 时，有两个要反省思考的点：一个类，有没有真实的可扩展需求，能不能使用 final 修饰符？一个方法，子类有没有重写的必要性，能不能使用 final 修饰符？

限制住不可预测的可扩展性，是实现安全代码、健壮代码的一个重要目标。

JDK 17 之前的 Java 语言，限制住可扩展性只有两个方法，使用私有类或者 final 修饰符。显而易见，私有类不是公开接口，只能内部使用；而 final 修饰符彻底放弃了可扩展性。

**怎么声明封闭类**

封闭类这个概念，涉及到两种类型的类。第一种是被扩展的父类，第二种是扩展而来的子类。通常地，我们把第一种称为封闭类，第二种称为许可类。

封闭类的声明使用 sealed 类修饰符，然后在所有的 extends 和 implements 语句之后，使用 permits 指定允许扩展该封闭类的子类。

由 permits 关键字指定的许可子类（permitted subclasses），必须和封闭类处于同一模块（module）或者包空间（package）里。

```java
public abstract sealed class Shape permits Circle, Rectangle, Square {
    public final String id;

    public Shape(String id) {
        this.id = id;
    }

    public abstract double area();
}
```

**怎么声明许可类**

许可类必须和封闭类处于同一模块（module）或者包空间（package）里，也就是说，在编译的时候，封闭类必须可以访问它的许可类；

许可类必须是封闭类的直接扩展类；

许可类必须声明是否继续保持封闭：许可类可以声明为终极类（final），从而关闭扩展性；许可类可以声明为封闭类（sealed），从而延续受限制的扩展性；许可类可以声明为解封类（non-sealed）, 从而支持不受限制的扩展性。

**总结**

可扩展性的限定方法有四个：使用私有类；使用 final 修饰符；使用 sealed 修饰符；不受限制的扩展性。

在我们日常的接口设计和编码实践中，使用这四个限定方法的优先级应该是由高到低的。最优先使用私有类，尽量不要使用不受限制的扩展性。

