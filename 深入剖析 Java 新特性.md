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

### 05 | 类型匹配：怎么切除臃肿的强制转换？

通常，一个模式是匹配谓词和匹配变量的组合。其中，匹配谓词用来确定模式和目标是否匹配。在模式和目标匹配的情况下，匹配变量是从匹配目标里提取出来的一个或者多个变量。

对于类型匹配来说，匹配谓词用来指定模式的数据类型，而匹配变量就是一个属于该类型的数据变量。需要注意的是，对于类型匹配来说，匹配变量只有一个。

```java
static boolean isSquare(Shape shape) {
    // 匹配谓词
    if (shape instanceof Rectangle) {
        // 类型转换 & 匹配变量
        Rectangle rect = (Rectangle) shape;
        return (rect.length == rect.width);
    }
    return (shape instanceof Square);
}
```

**类型匹配**

```java
// 匹配谓词 & 类型转换 & 匹配变量
if (shape instanceof Rectangle rect) {
    return (rect.length == rect.width);
}
```

**匹配变量的作用域**

```java
// 匹配变量的作用域紧跟着类型匹配语句
public static boolean isSquare(Shape shape) {
    if (shape instanceof Rectangle rect) {
        // rect is in scope
        return rect.length() == rect.width();
    }
    // rect is not in scope here
    return shape instanceof Square;
}
```

```java
// 匹配变量的作用域远离了类型匹配语句
public static boolean isSquare(Shape shape) {
    if (!(shape instanceof Rectangle rect)) {
        // rect is not in scope here
        return shape instanceof Square;
    }
    // rect is in scope
    return rect.length() == rect.width();
}
```

```java
public static boolean isSquare(Shape shape) {
    return shape instanceof Square ||  // rect is not in scope here
          (shape instanceof Rectangle rect &&
           rect.length() == rect.width());   // rect is in scope here
}
```

```java
// 不能通过编译器的审查
public static boolean isSquare(Shape shape) {
    return shape instanceof Square ||  // rect is not in scope here
          (shape instanceof Rectangle rect ||
           rect.length() == rect.width());   // rect is not in scope here
}
```

```java
// 不能通过编译器的审查
public static boolean isSquare(Shape shape) {
    return shape instanceof Square |  // rect is not in scope here
          (shape instanceof Rectangle rect &
           rect.length() == rect.width());   // rect is in scope here
}
```

```java
// 匹配变量的作用域紧跟着类型匹配语句
public final class Shadow {
    private static final Rectangle rect = null;
    public static boolean isSquare(Shape shape) {
        if (shape instanceof Rectangle rect) {
            // Field rect is shadowed, local rect is in scope
            System.out.println("This should be the local rect: " + rect);
            return rect.length() == rect.width();
        }
        // Field rect is in scope, local rect is not in scope here
        System.out.println("This should be the field rect: " + rect);
        return shape instanceof Shape.Square;
    }
}
```

```java
// 匹配变量的作用域远离了类型匹配语句
public final class Shadow {
    private static final Rectangle rect = null;
    public static boolean isSquare(Shape shape) {
        if (!(shape instanceof Rectangle rect)) {
            // Field rect is in scope, local rect is not in scope here
            System.out.println("This should be the field rect: " + rect);
            return shape instanceof Shape.Square;
        }
        // Field rect is shadowed, local rect is in scope
        System.out.println("This should be the local rect: " + rect);
        return rect.length() == rect.width();
    }
}
```

### 06 | switch 表达式：怎么简化多情景操作？

第一个容易犯错的地方，就是在 break 关键字的使用上。

为什么 switch 语句里需要使用 break 呢？最主要的原因，就是希望能够在不同的情况下，共享部分或者全部的代码片段。

第二个容易犯错的地方，是反复出现的赋值语句。

```java
import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth;
        switch (month) {
            case Calendar.JANUARY:
            case Calendar.MARCH:
            case Calendar.MAY:
            case Calendar.JULY:
            case Calendar.AUGUST:
            case Calendar.OCTOBER:
            case Calendar.DECEMBER:
                daysInMonth = 31;
                break;
            case Calendar.APRIL:
            case Calendar.JUNE:
            case Calendar.SEPTEMBER:
            case Calendar.NOVEMBER:
                daysInMonth = 30;
                break;
            case Calendar.FEBRUARY:
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    daysInMonth = 29;
                } else {
                    daysInMonth = 28;
                }
                break;
            default:
                throw new RuntimeException(
                    "Calendar in JDK does not work");
        }

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}
```

**switch 表达式**

```java
import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth = switch (month) {
            case Calendar.JANUARY,
                 Calendar.MARCH,
                 Calendar.MAY,
                 Calendar.JULY,
                 Calendar.AUGUST,
                 Calendar.OCTOBER,
                 Calendar.DECEMBER -> 31;
            case Calendar.APRIL,
                 Calendar.JUNE,
                 Calendar.SEPTEMBER,
                 Calendar.NOVEMBER -> 30;
            case Calendar.FEBRUARY -> {
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    yield 29;
                } else {
                    yield 28;
                }
            }
            default -> throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
                "There are " + daysInMonth + " days in this month.");
    }
}
```

switch 代码块出现在了赋值运算符的右侧。

一个 case 语句，可以处理多个情景。

新的情景操作符 “->”，它是一个箭头标识符。如果要匹配的情景有两个或者两个以上，就要使用逗号 “,” 分隔符把它们分割开来。

箭头标识符右侧的数值，代表的就是该匹配情景下，switch 表达式的数值。箭头标识符右侧可以是表达式、代码块或者异常抛出语句，而不能是其他的形式。如果只需要一个语句，这个语句也要以代码块的形式呈现出来。

新的关键字 “yield”。为了便于理解，我们可以把 yield 语句产生的值看成是 switch 表达式的返回值。

在 switch 表达式里，所有的情景都要列举出来，不能多、也不能少（这也就是我们常说的穷举）。

**改进的 switch 语句**

```java
private static int daysInMonth(int year, int month) {
    int daysInMonth = 0;
    switch (month) {
        case Calendar.JANUARY,
             Calendar.MARCH,
             Calendar.MAY,
             Calendar.JULY,
             Calendar.AUGUST,
             Calendar.OCTOBER,
             Calendar.DECEMBER -> daysInMonth = 31;
        case Calendar.APRIL,
             Calendar.JUNE,
             Calendar.SEPTEMBER,
             Calendar.NOVEMBER -> daysInMonth = 30;
        case Calendar.FEBRUARY -> {
            if (((year % 4 == 0) && !(year % 100 == 0))
                    || (year % 400 == 0)) {
                daysInMonth = 29;
                break;
            }
            daysInMonth = 28;
        }
        // default -> throw new RuntimeException(
        //        "Calendar in JDK does not work");
    }
    return daysInMonth;
}
```

switch 语句可以使用箭头标识符，也可以使用 break 语句，也不需要列出所有的情景。

使用箭头标识符的 switch 语句不再需要 break 语句来实现情景间的代码共享了。

不过，使用箭头标识符的 switch 语句并没有禁止 break 语句，而是恢复了它本来的意义：从代码片段里抽身，就像它在循环语句里扮演的角色一样。

### 07 | switch 匹配：能不能适配不同的类型？

给定了一个形状的对象，我们该怎么判断这个对象是不是一个正方形呢？

（子类扩充出现的兼容性问题）

```java
public sealed interface Shape
        permits Shape.Circle, Shape.Rectangle, Shape.Square {
    /**
     * @since 1.0
     */
    record Circle(double radius) implements Shape {
    }

    /**
     * @since 1.0
     */
    record Square(double side) implements Shape {
    }

    /**
     * @since 2.0
     */
    record Rectangle(double length, double width) implements Shape {
    }
}
```

```java
/**
 * @since 1.0
 */
public static boolean isSquare(Shape shape) {
    return (shape instanceof Shape.Square);
}

/**
 * @since 2.0
 */
public static boolean isSquare(Shape shape) {
    if (shape instanceof Shape.Rectangle rect) {
        return (rect.length() == rect.width());
    }
    return (shape instanceof Shape.Square);
}
```

意识到扩展类库需要更改，并不是一件容易的事情。

```java
/**
 * @since 1.0
 */
public static boolean isSquare(Shape shape) {
    return switch (shape) {
        case null, Shape.Circle c -> false;
        case Shape.Square s -> true;
    };
}

/**
 * @since 2.0
 */
public static boolean isSquare(Shape shape) {
    return switch (shape) {
        case null, Shape.Circle c -> false;
        case Shape.Square s -> true;
        case Shape.Rectangle r -> r.length() == r.width();
    };
}
```

具有模式匹配能力的 switch，提升了 switch 的数据类型匹配能力。switch 要匹配的数据，现在可以是整型的原始类型（数字、枚举、字符串），或者引用类型。

具有模式匹配能力的 switch，支持空引用的匹配。

类型匹配出现在了匹配情景中。也就是说，你既可以检查类型，还可以获得匹配变量。

使用 switch 表达式，穷举出所有的情景。这种提前暴露问题的方式，大大地降低了代码维护的难度，让我们有更多的精力专注在更有价值的问题上。

**什么时候使用 default？**

```java
public static boolean isSquare(Shape shape) {
    return switch (shape) {
        case Shape.Square s -> true;
        case null, default -> false;
    };
}
```

使用了 default，也就意味着这样的 switch 表达式总是能够穷举出所有的情景。遗憾的是，这样的代码丧失了检测匹配情景有没有变更的能力。

所以，一般来说，只有我们能够确信，待匹配类型的升级，不会影响 switch 表达式的逻辑的时候，我们才能考虑使用缺省选择情景。

### 08 | 抛出异常，是不是错误处理的第一选择？

通常情况下，我们谈到异常的时候，除非有特别的声明，不然指的都是运行时异常或者检查型异常。

我们还知道，异常状况的处理会让代码的效率变低，所以我们不应该使用异常机制来处理正常的状况。一个流畅的业务，理想的情况是，在执行代码时没有任何异常发生。否则，业务执行的效率就会大打折扣。

```java
public static Digest of(String algorithm) throws NoSuchAlgorithmException {
    return switch (algorithm) {
        case "SHA-256" -> new SHA256();
        case "SHA-512" -> new SHA512();
        default -> throw new NoSuchAlgorithmException();
    };
}
```

```java
public static Coded<Digest> of(String algorithm) {
    return switch (algorithm) {
        case "SHA-256" -> new Coded(sha256, 0);
        case "SHA-512" -> new Coded(sha512, 0);
        default -> new Coded(null, -1);
    };
}
```

需要更多的代码 / 丢弃了调试信息 / 易碎的数据结构

我们能够通过异常的调用堆栈，清楚地看到代码的执行轨迹，快速找到出问题的代码。

生成一个 Coded 的实例，需要遵守两条纪律。第一条纪律是错误码的数值必须一致，0 代表没有错误，如果是其他的值表示出现了错误；第二条纪律是不能同时设置返回值和错误码。但是，这两条纪律需要编写代码的人自觉实现，编译器不会帮助我们检查错误。

使用错误码，也有一条铁的纪律：必须首先检查错误码，然后才能使用返回值。同样，编译器也不会帮助我们检查违反纪律的错误。

**改进方案：共用错误码**

```java
public sealed interface Returned<T> {
    record ReturnValue<T>(T returnValue) implements Returned {
    }
    
    record ErrorCode(Integer errorCode) implements Returned {
    }
}    
```

```java
public static Returned<Digest> of(String algorithm) {
    return switch (algorithm) {
        case "SHA-256" -> new ReturnValue(new SHA256());
        case "SHA-512" -> new ReturnValue(new SHA512());
        case null, default -> new ErrorCode(-1);
    };
}
```

生成 Coded 实例需要遵守的两条纪律，在这里也不需要了。因为，返回 ReturnValue 这个许可类，就表示没有错误；返回 ErrorCode 这个许可类，就表示出现错误。

```java
Returned<Digest> rt = Digest.of("SHA-256");
switch (rt) {
    case ReturnValue rv -> {
            Digest d = (Digest) rv.returnValue();
            d.digest("Hello, world!".getBytes());
        }
    case ErrorCode ec ->
            System.out.println("Failed to get instance");
}
```

你要使用返回值，就必须检查它是不是一个 ReturnValue 的实例。这种情况下，使用 Coded 档案类编写代码需要遵守的纪律，也就是必须先检查错误码，在这里也不需要了。

这种方式仍然具有一些缺陷，例如它本身没有携带调试信息。

