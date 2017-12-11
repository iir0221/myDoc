# Stream API
Stream主要用于处理集合元素。

Jdk8以前，通过Iterator来遍历集合，从而处理每个item，然而这个过程是串行的，处理完一个item才能再处理下一个item。这种情况下，不能充分的利用多核处理器的高性能。

Stream提供串行和并行两种模式来高效的处理集合元素。当选择并行模式时，Stream会在多个线程中，同时处理多个item。

如果说Collections主要关注于存储item，那么streams则主要用于操作数据源的item。

Streams有如下几个特点：

* Stream并不能存储数据元素，它只是操作数据源的item而已。
* Stream可以是无限的
* Stream不能复用
* Stream不会改变数据源对象，它只会返回一个持有结果的新的Stream
* Stream的操作可能是延迟的
```java
import java.io.*;
import java.nio.charset.*;
import java.nio.file.*;
import java.util.Arrays;
import java.util.List;

/**
 * 统计一个文件中长度大于12的字符数量
 */
public class Test {
    public static void main(String[] args) throws IOException {
        String contents = new String(Files.readAllBytes(
                Paths.get("/absolute/alice.txt")), StandardCharsets.UTF_8);
        List<String> words = Arrays.asList(contents.split("[\\P{L}]+"));

        // jdk8以前
        long count = 0;
        for (String w : words) {
            if (w.length() > 12) count++;
        }
        System.out.println(count);

        // Stream串行方式
        count = words.stream().filter(s -> s.length() > 12).count();
        System.out.println(count);

        // Stream并行方式
        count = words.parallelStream().filter(w -> w.length() > 12).count();
        System.out.println(count);
    }
}
```
* Stream通过三个阶段建立一个流水线

    * 创建一个Stream (words.stream())

    * 将初始Stream转换为另一个Stream (filter(s->s.length()>12))

    * 终止操作产生结果 (count())

Stream并不按照item的调用顺序执行。在上例中，当变量count被用到时（System.out.println(count)）,才会执行Stream操作。

另外Stream API只提供了IntStream，LongStream，DoubleStream3个基本类型的流。
# 创建Stream
* 有多种方式生成 Stream：
    * Create Streams from values
    * Create Streams from Empty streams
    * Create Streams from functions
    * Create Streams from arrays
    * Create Streams from collections
    * Create Streams from String
    * Create Streams from files
    * Create Streams from other sources

## Stream.of()
```java
static <T> Stream<T> of(T t)
static <T> Stream<T> of(T... values)
```
```java
/**
 * Create Streams from values
 */
public static void createByValue() throws IOException {

    // 使用Stream<T>.of(T  t), Stream<T>.of(T...values) 创建stream.
    Stream<String> stream = Stream.of("java2s.com");
    stream.forEach(System.out::println);

    Stream<String> stream2 = Stream.of("XML", "Java", "CSS", "SQL");
    stream2.forEach(System.out::println);

    Path path = Paths.get("/absolutely/alice.txt");
    String contents = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
    String[] strArray = contents.split("[\\P{L}]+");
    Stream streamArray = Stream.of(strArray);
    streamArray.forEach(System.out::println);
}
```
类似的
```
IntStream.of(int t)
IntStream.of(int... values)
LongStream.of(long t)
LongStream.of(long... values)
DoubleStream.of(double t)
DoubleStream.of(double... values)
```
## Stream.builder()
```java
Stream<String> stream3 = Stream.<String>builder()
        .add("XML")
        .add("Java")
        .add("CSS")
        .add("SQL")
        .build();
stream3.forEach(System.out::println);
```
类似的
```java
IntStream.builder()
LongStream.builder()
DoubleStream.builder()
```

## IntStream.range && IntStream.rangeClosed
```java
static IntStream range(int startInclusive,
                       int endExclusive)
static IntStream rangeClosed(int startInclusive,
                             int endInclusive)
```
```java
// 使用IntStream.range(int startInclusive, int endExclusive) 创建 IntStream
// 不包括上限
IntStream oneToFiveExclusive = IntStream.range(1, 6);
oneToFiveExclusive.forEach(System.out::println);

// 使用使用IntStream.rangeClosed(int startInclusive, int endInclusive) 创建 IntStream
// 包括上限
IntStream oneToFiveInclusive = IntStream.rangeClosed(1, 5);
oneToFiveInclusive.forEach(System.out::println);

// LongStream也是如此
LongStream LongoneToLongsixExclusix = LongStream.range(1l, 7l);
LongoneToLongsixExclusix.forEach(System.out::println);

LongStream LongoneToLongsixInclusix = LongStream.rangeClosed(1l, 6l);
LongoneToLongsixInclusix.forEach(System.out::println);
```
## Strea.empty()
```java
static <T> Stream<T> empty()
```
```java
/**
 * Create Streams from Empty streams
 */
public static void createEmptyStream() throws IOException {
    
    // 使用Stream.empty()创建空Stream
    Stream<String> streamEmpty = Stream.empty();
    streamEmpty.forEach(System.out::println);

    // 创建空的 IntStream,LongStream,DoubleStream
    IntStream intStreamEmpty = IntStream.empty();
    LongStream longStreamEmpyt = LongStream.empty();
    DoubleStream doubleStreamEmpty = DoubleStream.empty();
}
```
## Create infinite Streams from functions
The following two static methods from Stream interface generates an infinite stream from a function.
```java
<T> Stream<T> iterate(T  seed, UnaryOperator<T>  f)
<T> Stream<T> generate(Supplier<T> s)
```
* iterate() method creates a sequential ordered stream.
* generate() method creates a sequential unordered stream.

java.util.Random 提供的 ints(), longs(), doubles()方法可以创建无限长度的 IntStream, LongStream, and DoubleStream
```java
public IntStream ints()
public IntStream ints(long streamSize)
public IntStream ints(long streamSize,
                      int randomNumberOrigin,
                      int randomNumberBound)
public IntStream ints(int randomNumberOrigin,
                      int randomNumberBound)

public LongStream longs(long streamSize)
public LongStream longs()
public LongStream longs(long streamSize,
                        long randomNumberOrigin,
                        long randomNumberBound)
public LongStream longs(long randomNumberOrigin,
                        long randomNumberBound)

public DoubleStream doubles(long streamSize)
public DoubleStream doubles()
public DoubleStream doubles(long streamSize,
                            double randomNumberOrigin,
                            double randomNumberBound)
public DoubleStream doubles(double randomNumberOrigin,
                            double randomNumberBound)
```
```java
public static void createByFunctions() {
    // <T> Stream<T> iterate(T  seed, UnaryOperator<T>  f)
    // 创建无限长度的stream
    // seed是stream的第一个元素
    // 第二个元素由指定的函数作用于第一个元素生成f(seed)
    // 第三个元素由指定的函数作用于第二个元素生成f(f(seed))
    // 因此所有的元素依次是:
    // seed, f(seed), f(f(seed)), f(f(f(seed)))....

    Stream<Long> tenNaturalNumbers = Stream.iterate(1L, n -> n + 1)
            .limit(10);
    tenNaturalNumbers.forEach(System.out::println);

    IntStream.iterate(1, n -> n + 1).limit(10).forEach(System.out::println);


    // <T> Stream<T> generate(Supplier<T> s)
    // 创建无限长度的stream，通过指定的函数生成元素
    Stream.generate(Math::random)
            .limit(5)
            .forEach(System.out::println);

    IntStream.generate(() -> 0)
            .limit(5)
            .forEach(System.out::println);

    Stream.generate(new Random()::nextInt)
            .limit(5)
            .forEach(System.out::println);

    IntStream.generate((new Random()::nextInt)).
            limit(5).
            forEach(System.out::println);

    // java.util.Random 提供的 ints(), longs(), doubles()方法
    // 可以创建无限长度的 IntStream, LongStream, and DoubleStream
    new Random().ints()
            .limit(5)
            .forEach(System.out::println);
}
```
## Collection#stream() && Arrays.stream()
```java
default Stream<E> stream()
default Stream<E> parallelStream()

static DoubleStream	stream(double[] array)
static DoubleStream	stream(double[] array, int startInclusive, int endExclusive)
static IntStream stream(int[] array)
static IntStream stream(int[] array, int startInclusive, int endExclusive)
static LongStream stream(long[] array)
static LongStream stream(long[] array, int startInclusive, int endExclusive)
static <T> Stream<T> stream(T[] array)
static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)
```
```java
/**
 * create streams from Arrays or Collections
 * @throws IOException
 */
public static void createStreamByArraysOrCollections() throws IOException {
    Path path = Paths.get("/absolutely/alice.txt");
    String contents = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
    String[] strArray = contents.split("[\\P{L}]+");
    List<String> list = Arrays.asList(strArray);

    Stream streamArray2 = Arrays.stream(strArray);
    streamArray2.forEach(System.out::println);

    Stream<String> sequentialStream = list.stream();
    sequentialStream.forEach(System.out::println);

    Stream<String> parallelStream = list.parallelStream();
    parallelStream.forEach(System.out::println);
}
```

## Create Streams from String:CharSequence#chars() && CharSequence#splitAsStream(CharSequence input)
```java
default IntStream chars()
```
```java
/**
 * create Streams from String
 */
public static void createStreamByString() {
    String str = "5 123,123,qwe,1,123, 25";

    // chars()方法可用于String, StringBuilder, StringBuffer生成stream
    str.chars()
            .filter(n -> !Character.isDigit((char) n) && !Character.isWhitespace((char) n))
            .forEach(n -> System.out.print((char) n));

    // java.util.regex.Pattern.splitAsStream(CharSequence input)
    // 根据正则表达式生成stream
    Pattern.compile(",")
            .splitAsStream(str)
            .forEach(System.out::println);
}
```
## Create Streams from files
```java
/**
 * create Streams from files
 * @throws IOException
 */
public static void createStreamByFile() throws IOException {

    // 通过stream读取文件内容
    Path path = Paths.get("/absolutely/alice.txt");
    try (Stream<String> lines = Files.lines(path)) {
        lines.forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }

    // 通过stream读取路径信息
    Path dir = Paths.get(".");
    System.out.printf("%nThe file tree for %s%n",
            dir.toAbsolutePath());
    try (Stream<Path> fileTree = Files.walk(dir)) {
        fileTree.forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }

    // 通过stream读取url
    URL example = new URL("http://localhost:8080/example/faces/index.xhtml");
    try (InputStream in = example.openStream();) {
        BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        Stream<String> lines = reader.lines();
        lines.forEach(System.out::println);
    }
}
```

## 其他方式创建stream
```java
BitSet.stream()
JarFile.stream()
java.util.Spliterator
```
# stream operations
The commonly used stream operations are listed as follows.

* Distinct
    * Intermediate Operation
    * Returns a stream consisting of the distinct elements by checking equals() method.
* filter
    * Intermediate Operation
    * Returns a stream that match the specified predicate.
* flatMap
    * Intermediate Operation
    * Produces a stream flattened.
* limit
    * Intermediate Operation
    * truncates a stream by number.
* map
    * Intermediate Operation
    * Performs one-to-one mapping on the stream
* peek
    * Intermediate Operation
    * Applies the action for debugging.
* skip
    * Intermediate Operation
    * Discards the first n elements and returns the remaining stream. If this stream contains fewer than requested, an empty stream is returned.
* sorted
    * Intermediate Operation
    * Sort a stream according to natural order or the specified Comparator. For an ordered stream, the sort is stable.
* allMatch
    * Terminal Operation
    * Returns true if all elements in the stream match the specified predicate, false otherwise. Returns true if the stream is empty.
* anyMatch
    * Terminal Operation
    * Returns true if any element in the stream matches the specified predicate, false otherwise. Returns false if the stream is empty.
* findAny
    * Terminal Operation
    * Returns any element from the stream. Returns an empty Optional object for an empty stream.
* findFirst
    * Terminal Operation
    * Returns the first element of the stream.
        * For an ordered stream, it returns the first element; 
        * for an unordered stream, it returns any element.
* noneMatch
    * Terminal Operation
    * Returns true if no elements in the stream match the specified predicate, false otherwise. Returns true if the stream is empty.
* forEach
    * Terminal Operation
    * Applies an action for each element in the stream.
* reduce
    * Terminal Operation
    * Applies a reduction operation to computes a single value from the stream.
## Streams Peek
We can debug a stream by using the peek(Consumer<? super T> action) method of the Stream<T> interface.

The IntStream, LongStream, and DoubleStream also contain a peek() method that takes a IntConsumer, a LongConsumer, and a DoubleConsumer as an argument.

We can use a lambda expression with the peek() method to log elements.

The following code uses the peek() method to print the elements passing through the stream pipeline:
```java
import java.util.stream.Stream;
//ww w  . j a v a 2  s  .co  m
public class Main {
  public static void main(String[] args) {
    int sum = Stream.of(1, 2, 3, 4, 5)
        .peek(e -> System.out.println("Taking integer: " + e))
        .filter(n -> n % 2 == 1)
        .peek(e -> System.out.println("Filtered integer: " + e))
        .map(n -> n * n).peek(e -> System.out.println("Mapped integer: " + e))
        .reduce(0, Integer::sum);
    System.out.println("Sum = " + sum);

  }
}
```

## Streams ForEach
The forEach operation takes an action for each element of the stream.

The Stream<T> interface contains two methods to perform the forEach operation:
```java
void  forEach(Consumer<? super  T> action)
void  forEachOrdered(Consumer<? super  T> action)
```
IntStream, LongStream, and DoubleStream contain the same methods.

**The forEach() method does not guarantee the order in which the action for each element in the stream is applied.**

**The forEachOrdered() method performs the action in the order of elements defined by the stream.**

forEachOrdered() method may slow down processing in a parallel stream.

The following code prints the details of females in the employee list:
```java
import java.time.LocalDate;
import java.time.Month;
import java.util.Arrays;
import java.util.List;
//  ww  w.j  av a2 s.co m
public class Main {
  public static void main(String[] args) {

    Employee.persons()
            .stream()
            .filter(Employee::isFemale)
            .forEach(System.out::println);
  }
}

class Employee {
  public static enum Gender {
    MALE, FEMALE
  }

  private long id;
  private String name;
  private Gender gender;
  private LocalDate dob;
  private double income;

  public Employee(long id, String name, Gender gender, LocalDate dob,
      double income) {
    this.id = id;
    this.name = name;
    this.gender = gender;
    this.dob = dob;
    this.income = income;
  }
  public boolean isFemale() {
    return this.gender == Gender.FEMALE;
  }

  public static List<Employee> persons() {
    Employee p1 = new Employee(1, "Jake", Gender.MALE, LocalDate.of(1971,
        Month.JANUARY, 1), 2343.0);
    Employee p2 = new Employee(2, "Jack", Gender.MALE, LocalDate.of(1972,
        Month.JULY, 21), 7100.0);
    Employee p3 = new Employee(3, "Jane", Gender.FEMALE, LocalDate.of(1973,
        Month.MAY, 29), 5455.0);
    Employee p4 = new Employee(4, "Jode", Gender.MALE, LocalDate.of(1974,
        Month.OCTOBER, 16), 1800.0);
    Employee p5 = new Employee(5, "Jeny", Gender.FEMALE, LocalDate.of(1975,
        Month.DECEMBER, 13), 1234.0);
    Employee p6 = new Employee(6, "Jason", Gender.MALE, LocalDate.of(1976,
        Month.JUNE, 9), 3211.0);

    List<Employee> persons = Arrays.asList(p1, p2, p3, p4, p5, p6);

    return persons;
  }
}
```
## Java Streams - Java Stream Filter
The filter operation produces a filtered stream, a subset of the input stream, whose elements evaluate to true for the specified predicate.

A predicate is a function that accepts an element and returns a boolean value.

The filtered stream has the same type as the input stream.

If the predicate evaluates to false for all elements, it produces an empty stream.
```java
import java.time.LocalDate;
import java.time.Month;
import java.util.Arrays;
import java.util.List;

public class Main {
  public static void main(String[] args) {
    Employee.persons()
    .stream()
    .filter(Employee::isMale)
    .filter(p ->  p.getIncome() > 5000.0)
    .map(Employee::getName)
    .forEach(System.out::println);
  }
}
```
We can combine the two filters to one with AND operation.
```java
public class Main {
  public static void main(String[] args) {
    Employee.persons()
    .stream()
    .filter(p ->  p.isMale() &&   p.getIncome() > 5000.0)
    .map(Employee::getName)
    .forEach(System.out::println);
  }
}
```