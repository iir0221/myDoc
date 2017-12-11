# lambda表达式语法
lambda表达式是一种没有名字的函数，它拥有函数体和参数。

lambda表达式的语法十分简单：**参数->主体**。

通过->来分离参数和主体。
## 参数
lambda表达式可以有零个参数，一个参数或多个参数，参数可以指定类型，在编译器可以推导出参数类型的情况下，也可以省略参数类型。 

两个参数的例子：
```java
(String first, String second)-> Integer.compare(first.length(), second.length())
```
0个参数的例子：
```java
() -> { for (int i = 0; i < 1000; i++) doWork(); }
```
关于省略参数类型，可以参考泛型省略类型来理解。从jdk7开始，泛型可以简化写成如下形式：
```java
Map<String, String> myMap = new HashMap<>(); 
```
编译器会根据变量声明时的泛型类型自动推断出实例化HashMap时的泛型类型。

同样的，如果编译器可以推导出Lambda表达式中参数的类型，也可以将其省略，例如：
```java
Comparator<String> comp = (first, second) -> Integer.compare(first.length(), second.length());
```
上例lambda创建了一个函数式接口Comparator的对象（后文将介绍函数式接口），编译器根据声明，可以推断出first和second的类型为String。此时，参数类型可省略。在只有一个参数，且可推断出其类型的情况下，可以再将括号省略： 
```java
EventHandler<ActionEvent> listener = event ->System.out.println("Thanks for clicking!");
```
同方法参数一样，表达式参数也可以添加annotations或者final修饰：
```java
(final String name) -> ...
(@NonNull String name) -> 
```
## 主体
**lambda表达式的主体一定要有返回值。**

如果主体只有一句，则可以省略大括号：
```java
Comparator<String> comp = (first, second) -> Integer.compare(first.length(), second.length());
```
多于一句的情况，需要用{}括上:
```java
(String first, String second) -> {
   if (first.length() < second.length()) return -1;
   else if (first.length() > second.length()) return 1;
   else return 0;
}
```
主体必须有返回值，只在某些分支上有返回值也是不合法的，例如:
```java
(int x) -> { if (x >= 0) return 1; }
```
 这个例子是不合法的。
# 函数式接口                                                                                                                                                                                                
只包含一个抽象方法的接口叫做函数式接口。

函数式接口可使用注解@FunctionalInterface标注（不强制，但是如果标注了，编译器就会检查它是否只包含一个抽象方法）

**可以通过lambda表达式创建函数式接口的对象，这是lambda表达式在java中做的最重要的事情**

在jdk8以前，其实已经存在着一些接口，符合上述函数式接口的定义。

## JDK 8之前已有的函数式接口
```java
java.lang.Runnable

java.util.concurrent.Callable

java.security.PrivilegedAction

java.util.Comparator

java.io.FileFilter

java.nio.file.PathMatcher

java.lang.reflect.InvocationHandler

java.beans.PropertyChangeListener

java.awt.event.ActionListener

javax.swing.event.ChangeListener
```

在jdk8以前，这些接口的使用方式与其他接口并无不同。

通过两个例子来说明lambda表达式如何创建函数式接口实例

1.创建Runnable函数式接口实例，以启动线程——jdk8以前:
```java
import java.util.*;

public class OldStyle {
    public static void main(String[] args) {
        // 启动一个线程
        Worker w = new Worker();
        new Thread(w).start();
        // 启动一个线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        }).start();

    }
}

class Worker implements Runnable {
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```
运行结果：
```
Thread-0
Thread-1
```
从代码角度来看，不管是通过内部类还是通过匿名内部类，启动线程需要编写的代码都较为繁琐，其中，由程序员自定义的仅仅是run方法中的这一句话：
```java
System.out.println(Thread.currentThread().getName());
```
lambda表达式风格的启动线程:
```java
// 启动一个线程
Runnable runner = () -> System.out.println(Thread.currentThread().getName());
runner.run(); 
```
第一行实际上创建了一个函数式接口Runnable的实例runner，可以看出，lambda表达式的实体，恰好是run方法的方法体部分。

2.创建Comparator函数式接口实例，实现根据String的长度来排序一个String数组——jdk8以前:
```java
import java.util.*;

public class OldStyle {
    public static void main(String[] args) {
        // 排序一个数组
        class LengthComparator implements Comparator<String> {
            public int compare(String first, String second) {
                return Integer.compare(first.length(), second.length());
            }
        }
        String[] strings = "Mary had a little lamb".split(" ");
        Arrays.sort(strings, new LengthComparator());
        System.out.println(Arrays.toString(strings));
    }
}
```
lambda表达式:
```java
import java.util.*;

public class LambdaStyle {
    public static void main(String[] args) {    
        // 排序一个数组
        String[] strings = "Mary had a little lamb".split(" ");
        Arrays.sort(strings, (first, second) -> Integer.compare(first.length(), second.length()));
        System.out.println(Arrays.toString(strings));
    }
}
```
可以看出，函数式接口通过lambda表达式创建实例，是如此的精简。 

jdk8的java.util.function包下，又定义了一些函数式接口以及针对基本数据类型的子接口。
```java
Predicate -- 传入一个参数，返回一个bool结果， 方法为boolean test(T t)
Consumer -- 传入一个参数，无返回值，纯消费。 方法为void accept(T t)
Function<t,r> -- 传入一个参数，返回一个结果，方法为R apply(T t)
Supplier -- 无参数传入，返回一个结果，方法为T get()
UnaryOperator -- 一元操作符， 继承Function<t,t>,传入参数的类型和返回类型相同。
BinaryOperator -- 二元操作符， 传入的两个参数的类型和返回类型相同， 继承BiFunction
```
# 方法引用   
**方法引用增强了lambda表达式的可读性**

方法表达式的三种主要情况：

* 类::静态方法
* 对象::实例方法
* 类::实例方法

方法引用将会执行该类（对象）的指定静态（实例）方法。

方法引用例1：根据字母顺序(不区分大小写)排序一个字符串数组：

```java
import java.util.*;

public class LambdaStyle {
    public static void main(String[] args) {
        // 排序一个数组
        String[] strings = "Mary had a little lamb".split(" ");     
        Arrays.sort(strings, (s1, s2) -> {
            int n1 = s1.length();
            int n2 = s2.length();
            int min = Math.min(n1, n2);
            for (int i = 0; i < min; i++) {
                char c1 = s1.charAt(i);
                char c2 = s2.charAt(i);
                if (c1 != c2) {
                    c1 = Character.toUpperCase(c1);
                    c2 = Character.toUpperCase(c2);
                    if (c1 != c2) {
                        c1 = Character.toLowerCase(c1);
                        c2 = Character.toLowerCase(c2);
                        if (c1 != c2) {
                            // No overflow because of numeric promotion
                            return c1 - c2;
                        }
                    }
                }
            }
            return n1 - n2;
        });

        System.out.println(Arrays.toString(strings));
    }
}
```
上述例子，由于lambda表达式的主体代码较长，导致代码可读性下降，通过方法引用可以解决这个问题

方法引用例2：类::静态方法
```java
import java.util.*;

public class LambdaStyle {
    public static void main(String[] args) {
        // 排序一个数组
        String[] strings = "Mary had a little lamb".split(" ");
        Arrays.sort(strings, LambdaStyle::myCompareToIgnoreCase);
        System.out.println(Arrays.toString(strings));
    }

    public static int myCompareToIgnoreCase(String s1, String s2){
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);
                    if (c1 != c2) {
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;
    }
}
```
将主体代码抽出来写到一个方法中，然后引用这个方法。

方法引用例3：对象::实例方法
```java
import java.util.*;

public class LambdaStyle {
    public static void main(String[] args) {
        // 排序一个数组
        String[] strings = "Mary had a little lamb".split(" ");
        LambdaStyle lambdaStyle = new LambdaStyle();
        Arrays.sort(strings, lambdaStyle::myCompareToIgnoreCase);
        System.out.println(Arrays.toString(strings));
    }

    public int myCompareToIgnoreCase(String s1, String s2){
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);
                    if (c1 != c2) {
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;
    }
}
```
对类::实例方法这种情况的方法引用来说，第一个参数会成为执行方法的对象。

通过一个例子来说明。在String类中实际上已经提供了不区分大小写比较字符串的方法：
```java
public int compareToIgnoreCase(String str)
```
这个方法的用法为：
```java
String s = "jdfjsjfjskd";
String ss = "dskfksdkf";
int i = s.compareToIgnoreCase(ss);
System.out.println(i);
```
方法引用例4：类::实例

```java
import java.util.*;

public class LambdaStyle {
    public static void main(String[] args) {
        // 排序一个数组
        String[] strings = "Mary had a little lamb".split(" ");
        Arrays.sort(strings, String::compareToIgnoreCase);
        System.out.println(Arrays.toString(strings));
    }
}
```
分析例4，对于函数式接口Comparator来说，它的抽象方法为：
```java
int compare(T o1, T o2);
```
这个方法有两个参数，对于例1来说，出现在lambda表达式参数中的s1,s2,实际上就是这两个参数。例2，例3中的方法myCompareToIgnoreCase的参数也是如此。

而对于例4，String的compareToIgnoreCase方法只有一个参数。这时，第一个参数将会作为执行方法的对象，（s1.compareToIgnoreCase(s2)） 

另外，也可以通过如下形式方法引用：

* this::实例方法
* super::实例方法


方法引用例5：

```java
public class SuperTest {
    public static void main(String[] args) {
        class Greeter {
            public void greet() {
                System.out.println("Hello, world!");
            }
        }

        class ConcurrentGreeter extends Greeter {
            public void greet() {
                Thread t = new Thread(super::greet);
                t.start();
            }
        }

        new ConcurrentGreeter().greet();
    }
}
```
# 构造器引用                                                                                                                                                                            
和方法引用相似，只不过通过如下方式引用：

**类::new**

构造器引用可以生成一个类的实例

例1
```java
Stream<Button> stream = labels.stream().map(Button::new);
Button[] buttons4 = stream.toArray(Button[]::new);
```
# 变量作用域
lambda表达式引用值，而不是变量。

**lambda表达式中引用的局部变量必须是：显示声明为final的，或者虽然没有被声明为final，但实际上也算是有效的final的。**

在Java中与其相似的是匿名内部类关于局部变量的引用。

例1：匿名内部类引用局部变量——jdk8以前

```java
public class Outter {

    public static void main(String[] args) {
        final String  s1 = "Hello ";
        new Inner() {
            @Override
            public void printName(String name) {
                System.out.println(s1 + name);
            }
        }.printName("Lucy");

    }
}

interface Inner{
    public void printName(String name);

};
```
如例1所示，在jdk8以前，匿名内部类引用外部类定义的局部变量，则该变量必须是final的。

**jdk8将这个条件放宽，匿名内部类也可以访问外部类有效的final局部变量——即这个变量虽然没有显示声明为final，但定义后也没有再发生变化。**

例2：匿名内部类引用局部变量——jdk8

```java
public class Outter {

    public static void main(String[] args) {
        String  s1 = "Hello ";
        new Inner() {
            @Override
            public void printName(String name) {
                System.out.println(s1 + name);
            }
        }.printName("Lucy");

    }
}

interface Inner{
    public void printName(String name);

};
```
匿名内部类引用的外部类变量s1可以不显示定义为final。但是s1必须在初始化后不再改变。

**lambda表达式对于引用局部变量的规则同jdk8中的匿名内部类一样：显示声明为final的，或者虽然没有被声明为final，但实际上也算是有效的final的**

```java
import java.io.*;
import java.nio.charset.*;
import java.nio.file.*;
import java.util.*;
import java.util.stream.*;

public class VariableScope {
    public static void main(String[] args) {
        repeatMessage("Hello", 100);
    }
    
    public static void repeatMessage(String text, int count) {
        Runnable r = () -> {
            for (int i = 0; i < count; i++) {
                System.out.println(text);
                Thread.yield();
            }
        };
        new Thread(r).start();
    }

    public static void repeatMessage2(String text, int count) {
        Runnable r = () -> {
            while (count > 0) {
                // count--; // Error: Can't mutate captured variable
                System.out.println(text);
                Thread.yield();
            }
        };
        new Thread(r).start();
    }

    public static void countMatches(Path dir, String word) throws IOException {
        Path[] files = getDescendants(dir);
        int matches = 0;
        for (Path p : files)
            new Thread(() -> {
                if (contains(p, word)) {
                    // matches++;
                    // ERROR: Illegal to mutate matches
                }
            }).start();
    }


    private static int matches;

    public static void countMatches2(Path dir, String word) {
        Path[] files = getDescendants(dir);
        for (Path p : files)
            new Thread(() -> {
                if (contains(p, word)) {
                    matches++;
                    // CAUTION: Legal to mutate matches, but not threadsafe
                }
            }).start();
    }

    // Warning: Bad code ahead
    public static List<Path> collectMatches(Path dir, String word) {
        Path[] files = getDescendants(dir);
        List<Path> matches = new ArrayList<>();
        for (Path p : files)
            new Thread(() -> {
                if (contains(p, word)) {
                    matches.add(p);
                    // CAUTION: Legal to mutate matches, but not threadsafe
                }
            }).start();
        return matches;
    }

    public static Path[] getDescendants(Path dir) {
        try {
            try (Stream<Path> entries = Files.walk(dir)) {
                return entries.toArray(Path[]::new);
            }
        } catch (IOException ex) {
            return new Path[0];
        }
    }

    public static boolean contains(Path p, String word) {
        try {
            return new String(Files.readAllBytes(p),
                    StandardCharsets.UTF_8).contains(word);
        } catch (IOException ex) {
            return false;
        }
    }
}
```
# 默认方法
默认方法的意义：**为了兼容以前的版本**如果Collection接口要添加新的方法，例如forEach。那么每个实现了Collection接口的自定义类就必须都实现这个方法。这在JAVA中是完全无法接受的。
>Java8中，forEach方法已经被添加到了Iterable接口中。

* 如果一个类实现的接口有一个默认方法而该类继承的父类拥有同样的方法，则使用父类的方法。
    * >类优先原则
* 如果一个类实现的两个接口中拥有同样的默认方法，则该类必须重写这个默认方法以避免冲突。

# 接口中的静态方法
接口可以提供静态方法
```java
public interface Path {
    public static Path(String first,String... more) {
        return FileSystems.getDefault().getPath(first,more);
    }
}
```