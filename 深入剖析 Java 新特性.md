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

### 09 | 异常恢复，付出的代价能不能少一点？

```java
public static Returned<Digest> of(String algorithm) {
    return switch (algorithm) {
        case "SHA-256" -> new Returned.ReturnValue(new SHA256());
        case "SHA-512" -> new Returned.ReturnValue(new SHA512());
        case null -> {
            System.getLogger("co.ivi.jus.stack.union")
                    .log(System.Logger.Level.WARNING,
                        "No algorithm is specified",
                        new Throwable("the calling stack"));
            yield new Returned.ErrorCode(-1);
        }
        default -> {
            System.getLogger("co.ivi.jus.stack.union")
                    .log(System.Logger.Level.INFO,
                    "Unknown algorithm is specified " + algorithm,
                            new Throwable("the calling stack"));
            yield new Returned.ErrorCode(-1);
        }
    };
}
```

日志记录既可以开启，又可以关闭。如果我们关闭了日志，就不用再生成调试信息了，当然它的性能影响也就消失了。当需要我们定位问题的时候，再启动日志。这时候，我们就能够把性能的影响控制到一个极小的范围内了。

错误码的调试信息使用方式，更符合调试的目的：只有需要调试的时候，才会生成调试信息。

### 10 | Flow 是异步编程的终极选择吗？

**指令式编程模型**

所谓指令式编程模型，需要我们通过代码发布指令，然后等待指令的执行以及指令执行带来的状态变化。我们还要根据目前的状态，来确定下一次要发布的指令，并且用代码把下一个指令表示出来。

指令式编程模型关注的重点就在于控制状态。

阻塞在方法的调用上，增加了系统的延迟，降低了系统能够支持的吞吐量。

C10K 问题（支持 1 万个并发用户）。

**声明式编程模型**

如果指令式编程模型的逻辑是告诉计算机“该怎么做”，那么声明式的编程模型的逻辑就是告诉计算机“要做什么”。

```java
public sealed abstract class Digest {
    public static void of(String algorithm,
        Consumer<Digest> onSuccess, Consumer<Integer> onFailure) {
        // snipped
    }

    public abstract void digest(byte[] message,
        Consumer<byte[]> onSuccess, Consumer<Integer> onFailure);
}
```

无论是回调函数的实现，还是回调函数的调用，都可以自由地选择是采用异步的模式，还是同步的模式。

不过，回调函数的设计也有着天生的缺陷。这个缺陷，就是回调地狱（Callback Hell，常被译为回调地狱）。

```java
Digest.of("SHA-256",
    md -> {
        System.out.println("SHA-256 is supported");
        md.digest("Hello, world!".getBytes(),
            values -> {
                System.out.println("SHA-256 is available");
            },
            errorCode -> {
                System.out.println("SHA-256 is not available");
            });
    },
    errorCode -> {
        System.out.println("Unsupported algorithm: SHA-256");
    });
```

逻辑上的堆积，意味着代码的深度耦合。而深度耦合，意味着代码维护困难。深度嵌套里的一点点代码修改，都可能通过嵌套层层朝上传递，最后牵动全局。

这就导致，使用回调函数的声明式编程模型有着严重的场景适应问题。我们通常只使用回调函数解决性能影响最大的模块，而大部分的代码，依然使用传统的、顺序执行的指令式模型。

**反应式编程**

反应式编程的核心是数据流和变化传递。

数据的输出

```java
@FunctionalInterface
public static interface Publisher<T> {
    public void subscribe(Subscriber<? super T> subscriber);
}
```

数据的输入

```java
public static interface Subscriber<T> {
    public void onSubscribe(Subscription subscription);

    public void onNext(T item);

    public void onError(Throwable throwable);

    public void onComplete();
}
```

数据的控制

```java
public static interface Subscription {
    public void request(long n);

    public void cancel();
}
```

数据的传递

```java
public static interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
}
```

过程的串联

```java
private static void transform(byte[] message,
          Function<byte[], byte[]> transformFunction) {
    SubmissionPublisher<byte[]> publisher =
            new SubmissionPublisher<>();
    // create the transform processor
    Transform<byte[], byte[]> messageDigest =
            new Transform<>(transformFunction);
    // create subscriber for the processor
    Destination<byte[]> subscriber = new Destination<>(
            values -> System.out.println(
                    "Got it: " + Utilities.toHexString(values)));
    // chain processor and subscriber
    publisher.subscribe(messageDigest);
    messageDigest.subscribe(subscriber);
    publisher.submit(message);
    // close the submission publisher
    publisher.close();
}
```

反应式编程，怎么解决顺序执行带来的延迟后果呢？ 反应式编程，怎么解决回调函数带来的堆挤问题呢？（过程串联）

```java
Returned<Digest> rt = Digest.of("SHA-256");
switch (rt) {
    case Returned.ReturnValue rv -> {
        if (rv.returnValue() instanceof Digest d) {
            transform("Hello, World!".getBytes(), d::digest);
        } else {
            System.out.println("Implementation error: SHA-256");
        }
    }
    case Returned.ErrorCode ec -> System.out.println("Unsupported algorithm: SHA-256");
}
```

其中最要命的缺陷，就是错误很难排查，这是异步编程的通病。而反应式编程模型的解耦设计，加剧了错误排查的难度，这会严重影响开发的效率，降低代码的可维护性。

### 11 | 矢量运算：Java 的机器学习要来了吗？

<img src="线性方程.webp" alt="线性方程" style="zoom:50%;" />

```java
static final float[] a = new float[] {0.6F, 0.7F, 0.8F, 0.9F};
static final float[] x = new float[] {1.0F, 2.0F, 3.0F, 4.0F};

private static Returned<Float> sumInScalar(float[] a, float[] x) {
    if (a == null || x == null || a.length != x.length) {
        return new Returned.ErrorCode(-1);
    }
    float[] y = new float[a.length];
    for (int i = 0; i < a.length; i++) {
        y[i] = a[i] * x[i];
    }
    float r = 0F;
    for (int i = 0; i < y.length; i++) {
        r += y[i];
    }
    return new Returned.ReturnValue<>(r);
}
```

Java 的矢量运算就是使用单个指令并行处理多个数据的一个尝试（单指令多数据，Single Instruction Multiple Data）。

```java
static final float[] a = new float[] {0.6F, 0.7F, 0.8F, 0.9F};
static final FloatVector va =
        FloatVector.fromArray(FloatVector.SPECIES_128, a, 0);
static final float[] x = new float[] {1.0F, 2.0F, 3.0F, 4.0F};
static final FloatVector vx =
        FloatVector.fromArray(FloatVector.SPECIES_128, x, 0);

private static Returned<Float> sumInVector(FloatVector va, FloatVector vx) {
    if (va == null || vx == null || va.length() != vx.length()) {
        return new Returned.ErrorCode(-1);
    }
    
    FloatVector vy = va.mul(vx);
    float[] y = vy.toArray();
    
    float r = 0F;
    for (int i = 0; i < y.length; i++) {
        r += y[i];
    }
    return new Returned.ReturnValue<>(r);
}
```

### 12 | 外部内存接口：零拷贝的障碍还有多少？

像 TensorFlow、Ignite、Flink 以及 Netty 这样的类库，往往对性能有着偏执的追求。为了避免 Java 垃圾收集器不可预测的行为以及额外的性能开销，这些产品一般倾向于使用 JVM 之外的内存来存储和管理数据。这样的数据，就是我们常说的堆外数据（off-heap data）。

使用堆外存储最常用的办法，就是使用 ByteBuffer 这个类来分配直接存储空间（direct buffer）。JVM 虚拟机会尽最大的努力在直接存储空间上执行 IO 操作，避免数据在本地和 JVM 之间的拷贝。

理想状况下，一份数据只需要一份内存空间，这就是我们常说的零拷贝。

第一个缺陷是没有资源释放的接口。一旦一个 ByteBuffer 实例化，它所占用的内存的释放，就完全依赖于 JVM 的垃圾回收机制。

第二个缺陷是存储空间的限制。ByteBuffer 存储空间的大小，是使用 Java 整数来表示的。所以，它的存储空间最多只有 2G。

**外部内存接口**

```java
try (ResourceScope scope = ResourceScope.newConfinedScope()) {
    MemorySegment segment = MemorySegment.allocateNative(4, scope);
    for (int i = 0; i < 4; i++) {
        MemoryAccess.setByteAtOffset(segment, i, (byte)'A');
    }
}
```

ResourceScope 这个类实现了 AutoCloseable 接口，我们就可以使用 try-with-resource 这样的语句，及时地释放掉它管理的内存了。

MemorySegment 这个类，定义和模拟了一段连续的内存区域。MemoryAccess 这个类，定义了可以对 MemorySegment 执行读写操作。这两个类的寻址数据类型使用的是长整形（long）。

外部内存接口的更大使命，是和外部函数接口联系在一起的。

### 13 | 外部函数接口，能不能取代 Java 本地接口？

Java 本地接口面临的比较大的问题有两个：

一个是 C 语言编译、链接带来的问题，因为 Java 本地接口实现的动态库是平台相关的，所以就没有了 Java 语言“一次编译，到处运行”的跨平台优势；另一个问题是，因为逃脱了 JVM 的语言安全机制，JNI 本质上是不安全的。

```java
import java.lang.invoke.MethodType;
import jdk.incubator.foreign.*;

public class HelloWorld {
    public static void main(String[] args) throws Throwable {
        try (ResourceScope scope = ResourceScope.newConfinedScope()) {
            CLinker cLinker = CLinker.getInstance();
            MethodHandle cPrintf = cLinker.downcallHandle(
                    CLinker.systemLookup().lookup("printf").get(),
                    MethodType.methodType(int.class, MemoryAddress.class),
                    FunctionDescriptor.of(CLinker.C_INT, CLinker.C_POINTER));

            MemorySegment helloWorld =
                    CLinker.toCString("Hello, world!\n", scope);
            cPrintf.invoke(helloWorld.address());
        }
    }
}
```

使用外部函数接口的代码，不再需要编写 C 代码。当然，也不再需要编译、链接生成 C 的动态库了。所以，由动态库带来的平台相关的问题，也就不存在了。

可以说，使用外部函数接口的代码，是 Java 代码，因此也受到 Java 安全机制的约束。

### 用户故事 | 保持好奇心，积极拥抱变化

```java
// 假如给定一个由数字 1，2，3，4 构成的 List，要求把元素值都扩大一倍
// java8
List<Integer> result = initList.stream().map(i -> i * 2).collect(Collectors.toList());
// java11
var result = initList.stream().map(i -> i * 2).collect(Collectors.toUnmodifiableList());
// java17
var result = initList.stream().map(i -> i * 2).toList();
```

不可变数据可以让开发更加简单、可回溯、测试友好，它减少了很多可能的副作用，也就是说，减少了 Bug 的出现。

```java
// lombok 是 web 开发常用的扩展包，其中 setter 是一个可变操作，我们可以在配置文件中把它禁用掉，通过其它方式来替代
lombok.setter.flagUsage=error
lombok.data.flagUsage=error

@With
@Getter
@Builder(toBuilder = true)
@AllArgsConstructor(staticName = "of")
@NoArgsConstructor(staticName = "of")
@FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)
public class Model() {
    String id;
    String name;
    String type;
}

var model_01 = Model.of("101", "model_01", "model");

var model_02 = Model.of();

var model_03 = Model.toBuilder().id("301").name("model_03").build();

var model_04 = model_01.withName("model_04");

var model_05 = model_01.toBuilder().name("model_05").type("new_model").build();
```

这段代码使用 of() 方法构建新对象，withXX() 方法修改单个参数，toBuilder() 方法修改多个参数，修改后会返回新的对象。

```java
@With
@Builder(toBuilder = true)
public record Model(String id, String name, String type) {}
```

### 14 | 禁止空指针，该怎么避免崩溃的空指针？

我们需要检查返回值有没有可能是空指针，然后才能继续使用返回值。

**避免空指针**

在很多场景下，我们都可以使用空值来替代空指针，比如，空的字符串、空的集合。

```java
public record FullName(String firstName, String middleName, String lastName) {
    public FullName(String firstName, String middleName, String lastName) {
        this.firstName = firstName == null ? "" : firstName;
        this.middleName = middleName == null ? "" : middleName;
        this.lastName = lastName == null ? "" : lastName;
    }
}
```

**强制性检查**

**不尽人意的 Optional**

设计 Optional 的目的，是希望开发者能够先调用它的 Optional.isPresent 方法，然后再调用 Optional.get 方法获得目标对象。

```java
if (x.y().isPresent()) {
    return x.y().get();
}
```

遗憾的是，我们也可以不按照预期的方式使用它。这不在设计者的预期之内，但这是合法的代码。

```java
return x.y().get();
```

**新特性带来的新希望**

我们希望返回值的检查是强制性的。如果不检查，就没有办法得到返回值指代的真实对象。

```java
public sealed interface Returned {
    Returned.Undefined UNDEFINED = new Undefined();

    record ReturnValue<T>(T returnValue) implements Returned {
    }

    record Undefined() implements Returned {
    }
}
```

```java
public Returned<String> middleName() {
    if (middleName == null) {
        return Returned.UNDEFINED;
    }
    return new Returned.ReturnValue<>(middleName);
}
```

```java
private static boolean hasMiddleName(FullName fullName, String middleName) {
    return switch (fullName.middleName()) {
        case Returned.Undefined undefined -> false;
        case Returned.ReturnValue rv -> {
            String returnedMiddleName = (String) rv.returnValue();
            yield returnedMiddleName.equals(middleName);
        }
    };
}
```

### 15 | 现代密码：你用的加密算法过时了吗？

任何一个密码学的算法，都有它的生命周期。

密钥算法的废弃，总是会带来这样或者那样的兼容性问题。当安全性和兼容性相遇的时候，我们应该毫不犹豫地选择选择安全性，及时解决掉兼容性问题。

**应该抛弃的算法**

MD2 / MD5 / SHA-1 / DES / 3DES / RC4 / SSL 3.0 / TLS 1.0 / TLS 1.1 / 密钥小于 1024 位的 RSA 算法 / 密钥小于 1024 位的 DSA 算法 / 密钥小于 1024 位的 Diffie-Hellman 算法 / 密钥小于 256 位的 EC 算法

**应该退役的算法**

密钥大于 1024 位小于 2048 位的 RSA 算法 / 密钥大于 1024 位小于 2048 位的 DSA 算法 / 密钥大于 1024 位小于 2048 位的 Diffie-Hellman 算法 / RSA 签名算法 / 基于 RSA 的密钥交换算法 / 128 位的 AES 算法

**推荐使用的算法**

256 位的 AES 算法 / SHA-256、SHA-512 单向散列函数 / RSASSA-PSS 签名算法 / X25519/X448 密钥交换算法 / EdDSA 签名算法

### 16 | 改进的废弃，怎么避免使用废弃的特性？

在 JDK 中，公开接口的废弃需要使用两种不同的机制，也就是 Deprecated 注解（annotation）和 Deprecated 文档标记（JavaDoc tag）。

如果一段程序使用了废弃接口，在编译的时候就会提出警告。但是，有很多编译环境把编译警告看作是编译错误。为了解决这样的问题，JDK 还提供了“消除使用废弃接口的编译警告”选项，也就是 SuppressWarnings 注解。

为什么删除废弃的公开接口这么困难呢？如果从废弃机制本身的角度来思考，下面几个问题延迟了废弃接口使用者的迁移意愿和努力。

第一个问题，也是最重要的问题，就是 SuppressWarnings 注解的使用。SuppressWarnings 注解的本意是消除编译警告，保持向后的编译兼容性。可是一旦编译警告消除，SuppressWarnings 注解也就抵消了 Deprecated 注解的功效。

第二个问题，就是废弃接口的使用者并不担心使用废弃接口。虽然我们都知道不应该使用废弃的接口，但是因为一些人认为没有紧急迁移的必要性，也不急着制定代码迁移的时间表，所以倾向于先使用 SuppressWarnings 注解把编译警告消除了。

第三个问题，就是废弃接口的使用者并不知道接口废弃了多久。在接口使用者的眼里，废弃了十年和废弃了一年的接口没有什么区别。可是，在接口维护者的眼里，废弃了十年的接口应该可以放心地删除了。

**改进的废弃**

第一个改进是添加了一个新的工具，jdeprscan。有了这个工具，就可以扫描编译好的 Java 类或者包，看看有没有使用废弃的接口了。即使代码使用了 SuppressWarnings 注解，jdeprscan 的结果也不受影响。

另外，如果我们使用第三方的类库，或者已经编译好的类库，发现对废弃接口的依赖关系很重要。如果将来废弃接口被删除，使用废弃接口的类库将不能正常运行。而 jdeprscan 允许我们在使用一个类库之前进行废弃依赖关系检查，提前做好风险的评估。

第二个改进是给 Deprecated 注解增加了一个 forRemoval 属性。如果这个属性设置为 true，那就表示删除这个废弃接口已经提上了日程。这样的改进，强调了代码迁移的紧急性，它给了使用者一个明确的提示。

第三个改进是给 Deprecated 注解增加了一个 since 属性。这个属性会说明这个接口是在哪一个版本废弃的。

```java
public sealed abstract class Digest {
    /**
     * -- snipped
     *
     * @deprecated This method is not performance friendly. Use
     *             {@link #digest(byte[], byte[]) instead.
     */
    @Deprecated(since = "1.4", forRemoval = true)
    public abstract byte[] digest(byte[] message);

    // snipped
    public void digest(byte[] message, byte[] digestValue) {
        // snipped
    }
}
```

如果在 Deprecated 注解里新加入 forRemoval 属性，并且设置为 true，那么以前的 SuppressWarnings 就会失去效果。要想消除掉编译警告，我们需要使用新的选项。

```java
@SuppressWarnings("removal")
public static void main(String[] args) {
    try {
        Digest.of("SHA-256")
              .digest("Hello, world!".getBytes());
    } catch (NoSuchAlgorithmException ex) {
        // ignore
    }
}
```

**废弃三部曲**

第一步，废弃一个接口，标明废弃的版本号，并且描述替代方案。

第二步，添加 forRemoval 属性，把删除的计划提上日程。

第三步，删除废弃的接口。

### 17 | 模块系统：为什么 Java 需要模块化？

**缺失的访问控制**

JDK 9 之前的 Java 语言没有描述和定义包之间的依赖关系，也没有描述和定义基于包的依赖关系的访问控制规则。

**松散的使用合约**

按照 JDK 的期望，一个开发者应该只使用公开接口，而不能使用实现细节的内部接口。无论是公开接口，还是内部接口，都可以使用 public 修饰符。那么，该怎么判断一个接口是公开接口，还是内部接口呢？

解决的办法，是依靠 Java 接口的使用规范这样的纪律性合约，而不是依靠编译器强制性的检查。在 JDK 里，以 java 或者 javax 命名开头的 Java 包是公开的接口，其他的包是内部的接口。按照 Java 接口的使用规范，一个开发者应该只使用公开的接口，而不能使用内部的接口。不过，这是一个完全依靠自觉的纪律性约束，Java 的编译器和解释器并不会禁止开发者使用内部接口。

**手工的依赖管理**

Java 的编译器只需要知道公开接口的规范，并不会去检查实现的代码，也不会去链接实现的代码。可是，Java 在运行时，不仅需要知道公开接口的字节码，还需要知道实现的字节码，这就导致了编译和运行的脱节。一个能通过编译的代码，运行时可能也会遇到找不到实现代码的错误。

**缓慢的类加载**

由于 Jar 文件里没有描述类的依赖关系的内容，Java 的类加载器只能线性地搜索 class path 下的 Jar 文件，直到发现了给定的类和方法。

更糟糕的是，这种线性搜索的方式，还带来了不可预测的副作用。其中，影子类（Shadowing classes）和版本冲突是最常见的两个副作用。

**总结**

Java 语言没有描述和定义包之间的依赖关系，这个缺失，导致了无法有效地封闭实现的细节；无法有效地管理应用的部署；无法精准地控制类的检索和加载，也影响了应用启动的效率。

### 18 | 模块系统：怎么模块化你的应用程序？

```java
package co.ivi.jus.crypto;

import java.util.ServiceLoader;

public interface Digest {
    byte[] digest(byte[] message);

    static Returned<Digest> of(String algorithm) {
        ServiceLoader<DigestManager> serviceLoader =
                ServiceLoader.load(DigestManager.class);
        for (DigestManager cryptoManager : serviceLoader) {
            Returned<Digest> rt = cryptoManager.create(algorithm);
            switch (rt) {
                case Returned.ReturnValue rv -> {
                    return rv;
                }
                case Returned.ErrorCode ec -> {
                    continue;
                }
            }
        }
        return Returned.UNDEFINED;
    }
}
```

```java
package co.ivi.jus.crypto;

public interface DigestManager {
    Returned<Digest> create(String algorithm);
}
```

```java
package co.ivi.jus.impl;

// snipped

public final class DigestManagerImpl implements DigestManager {
    @Override
    public Returned<Digest> create(String algorithm) {
        return switch (algorithm) {
            case "SHA-256" -> Sha256.returnedSha256;
            case "SHA-512" -> Sha512.returnedSha512;
            default -> Returned.UNDEFINED;
        };
    }
}
```

```java
package co.ivi.jus.impl;

import co.ivi.jus.crypto.Digest;
import co.ivi.jus.crypto.Returned;

final class Sha256 implements Digest {
    static final Returned.ReturnValue<Digest> returnedSha256;
    // snipped
    private Sha256() {
        // snipped
    }

    @Override
    public byte[] digest(byte[] message) {
        // snipped
    }
}
```

由于 ServiceLoader 需要使用 public 修饰的桥梁接口，所以我们不能使用除了 public 以外的访问修饰符。也就是说，如果应用程序加载了这个 Java 包，它就可以直接使用 DigestManagerImpl 类。

**使用 Java 模块**

把公开接口部分 co.ivi.jus.crypto 封装到模块 jus.crypto

```java
module jus.crypto {
    // 模块的公开接口
    exports co.ivi.jus.crypto;
    // 使用 DigestManager 定义的服务接口
    uses co.ivi.jus.crypto.DigestManager;
}
```

把内部接口部分 co.ivi.jus.impl 封装到模块 jus.crypto.impl

```java
module jus.crypto.impl {
    // 这个模块需要使用 jus.crypto 模块
    requires jus.crypto;
    // 这个模块实现了 DigestManager 定义的服务接口
    provides co.ivi.jus.crypto.DigestManager with co.ivi.jus.impl.DigestManagerImpl;
}
```

模块化应用程序

```java
package co.ivi.jus.use;

import co.ivi.jus.crypto.Digest;
import co.ivi.jus.crypto.Returned;

public class UseCase {
    public static void main(String[] args) {
        Returned<Digest> rt = Digest.of("SHA-256");
        switch (rt) {
            case Returned.ReturnValue rv -> {
                Digest d = (Digest) rv.returnValue();
                d.digest("Hello, world!".getBytes());
            }
            case Returned.ErrorCode ec ->
                    System.getLogger("co.ivi.jus.stack.union")
                        .log(System.Logger.Level.INFO,
                              "Failed to get instance of SHA-256");
        }
    }
}
```

```java
module jus.crypto.use {
    requires jus.crypto;
}
```

**模块的编译和运行**

```shell
$ cd jus.crypto
$ javac --enable-preview --release 17 \
      -d classes src/main/java/co/ivi/jus/crypto/* \
      src/main/java/module-info.java
$ jar --create --file ../jars/jus.crypto.jar -C classes .

$ cd ../jus.crypto.impl
$ javac --enable-preview --release 17 \
      --module-path ../jars -d classes \
      src/main/java/co/ivi/jus/impl/* \
      src/main/java/module-info.java
$ jar --create --file ../jars/jus.crypto.impl.jar -C classes .

$ cd ../jus.crypto.use
$ javac --enable-preview --release 17 \
      --module-path ../jars -d classes \
      src/main/java/co/ivi/jus/use/* \
      src/main/java/module-info.java 
$ jar --create --file ../jars/jus.crypto.use.jar \
      --main-class co.ivi.jus.use.UseCase \
      -C classes .

$ java --enable-preview --module-path ../jars --module jus.crypto.use
```

[java-9-modules-cheat-sheet](java-9-modules-cheat-sheet.pdf)

### 结束语 | Java 的未来，依然是星辰大海
