## 函数式编程

> 基本概念

可以简化类的书写，简化编程模型。

> 几大函数式接口

| **特征**                                  | **函数式方法名**                                             | **示例**                                                     |
| ----------------------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| 无参数； 无返回值                         | **Runnable** (java.lang) `run()`                             | **Runnable**                                                 |
| 无参数； 返回类型任意                     | **Supplier** `get()` `getAs类型()`                           | **Supplier`<T>` BooleanSupplier IntSupplier LongSupplier DoubleSupplier** |
| 无参数； 返回类型任意                     | **Callable** (java.util.concurrent) `call()`                 | **Callable`<V>`**                                            |
| 1 参数； 无返回值                         | **Consumer** `accept()`                                      | **`Consumer<T>` IntConsumer LongConsumer DoubleConsumer**    |
| 2 参数 **Consumer**                       | **BiConsumer** `accept()`                                    | **`BiConsumer<T,U>`**                                        |
| 2 参数 **Consumer**； 1 引用； 1 基本类型 | **Obj类型Consumer** `accept()`                               | **`ObjIntConsumer<T>` `ObjLongConsumer<T>` `ObjDoubleConsumer<T>`** |
| 1 参数； 返回类型不同                     | **Function** `apply()` **To类型** 和 **类型To类型** `applyAs类型()` | **Function`<T,R>` IntFunction`<R>` `LongFunction<R>` DoubleFunction`<R>` ToIntFunction`<T>` `ToLongFunction<T>` `ToDoubleFunction<T>` IntToLongFunction IntToDoubleFunction LongToIntFunction LongToDoubleFunction DoubleToIntFunction DoubleToLongFunction** |
| 1 参数； 返回类型相同                     | **UnaryOperator** `apply()`                                  | **`UnaryOperator<T>` IntUnaryOperator LongUnaryOperator DoubleUnaryOperator** |
| 2 参数类型相同； 返回类型相同             | **BinaryOperator** `apply()`                                 | **`BinaryOperator<T>` IntBinaryOperator LongBinaryOperator DoubleBinaryOperator** |
| 2 参数类型相同; 返回整型                  | Comparator (java.util) `compare()`                           | **`Comparator<T>`**                                          |
| 2 参数； 返回布尔型                       | **Predicate** `test()`                                       | **`Predicate<T>` `BiPredicate<T,U>` IntPredicate LongPredicate DoublePredicate** |
| 参数基本类型； 返回基本类型               | **类型To类型Function** `applyAs类型()`                       | **IntToLongFunction IntToDoubleFunction LongToIntFunction LongToDoubleFunction DoubleToIntFunction DoubleToLongFunction** |
| 2 参数类型不同                            | **Bi操作** (不同方法名)                                      | **`BiFunction<T,U,R>` `BiConsumer<T,U>` `BiPredicate<T,U>` `ToIntBiFunction<T,U>` `ToLongBiFunction<T,U>` `ToDoubleBiFunction<T>`** |

> 函数的组合

| 组合方法                                         | 支持接口                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| `andThen(argument)` 根据参数执行原始操作         | **Function BiFunction Consumer BiConsumer IntConsumer LongConsumer DoubleConsumer UnaryOperator IntUnaryOperator LongUnaryOperator DoubleUnaryOperator BinaryOperator** |
| `compose(argument)` 根据参数执行原始操作         | **Function UnaryOperator IntUnaryOperator LongUnaryOperator DoubleUnaryOperator** |
| `and(argument)` 短路**逻辑与**原始谓词和参数谓词 | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |
| `or(argument)` 短路**逻辑或**原始谓词和参数谓词  | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |
| `negate()` 该谓词的**逻辑否**谓词                | **Predicate BiPredicate IntPredicate LongPredicate DoublePredicate** |

```java
public class FunctionComposition {
  static Function<String, String>
    f1 = s -> {
      System.out.println(s);
      return s.replace('A', '_');
    },
    f2 = s -> s.substring(3),
    f3 = s -> s.toLowerCase(),
    f4 = f1.compose(f2).andThen(f3);
  public static void main(String[] args) {
    System.out.println(
      f4.apply("GO AFTER ALL AMBULANCES"));
  }
}
```

> 柯里化函数

概念：将多参数转换为单参数。

在函数式编程的应用：

```java
public class Curry3Args {
   public static void main(String[] args) {
      Function<String,
        Function<String,
          Function<String, String>>> sum =
            a -> b -> c -> a + b + c;
      Function<String,
        Function<String, String>> hi =
          sum.apply("Hi ");
      Function<String, String> ho =
        hi.apply("Ho ");
      System.out.println(ho.apply("Hup"));
   }
}
```

> 未绑定的方法引用

```java
class X {
  String f() { return "X::f()"; }
}

interface MakeString {
  String make();
}

interface TransformX {
  String transform(X x);
}

public class UnboundMethodReference {
  public static void main(String[] args) {
    // MakeString ms = X::f; // [1]
    TransformX sp = X::f;
    X x = new X();
    System.out.println(sp.transform(x)); // [2]
    System.out.println(x.f()); // 同等效果
  }
}
```

这个用途主要是在流式编程中使用的比较多：

```java
public class StreamOfOptionals {
    public static void main(String[] args) {
        Signal.stream()
                .limit(10)
                .forEach(System.out::println);
        System.out.println(" ---");
        Signal.stream()
                .limit(10)
          // 这里使用，非常方便的流操作。
                .filter(Optional::isPresent)
                .map(Optional::get)
                .forEach(System.out::println);
    }
}
```

## 流式编程(*)

> 概念

- 集合用来做对象的存储，而流用来做计算。
- 流是懒加载的。这代表着它只在绝对必要时才计算。你可以将流看作“延迟列表”。由于计算延迟，流使我们能够表示非常大（甚至无限）的序列，而不需要考虑内存问题。

### 1. 流创建

> Stream.of()

```java
public class StreamOf {
    public static void main(String[] args) {
        Stream.of(new Bubble(1), new Bubble(2), new Bubble(3))
            .forEach(System.out::println);
        Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!")
            .forEach(System.out::print);
        System.out.println();
        Stream.of(3.14159, 2.718, 1.618)
            .forEach(System.out::println);
    }
}
```

> Stream.generate()

可以获得一个无限流的对象，需要元素的时候就执行Supplier获取。

```java

public class RandomWords implements Supplier<String> {
    List<String> words = new ArrayList<>();
    Random rand = new Random(47);
    RandomWords(String fname) throws IOException {
        List<String> lines = Files.readAllLines(Paths.get(fname));
        // 略过第一行
        for (String line : lines.subList(1, lines.size())) {
            for (String word : line.split("[ .?,]+"))
                words.add(word.toLowerCase());
        }
    }
    public String get() {
        return words.get(rand.nextInt(words.size()));
    }
    @Override
    public String toString() {
        return words.stream()
            .collect(Collectors.joining(" "));
    }
    public static void main(String[] args) throws Exception {
        System.out.println(
            Stream.generate(new RandomWords("Cheese.dat"))
                .limit(10)
                .collect(Collectors.joining(" ")));
    }
}
```

> 通过集合、数组相关api直接转换为流

```java
public class CollectionToStream {
    public static void main(String[] args) {
        List<Bubble> bubbles = Arrays.asList(new Bubble(1), new Bubble(2), new Bubble(3));
        System.out.println(bubbles.stream()
            .mapToInt(b -> b.i)
            .sum());
        
        Set<String> w = new HashSet<>(Arrays.asList("It's a wonderful day for pie!".split(" ")));
        w.stream()
         .map(x -> x + " ")
         .forEach(System.out::print);
        System.out.println();
        
        Map<String, Double> m = new HashMap<>();
        m.put("pi", 3.14159);
        m.put("e", 2.718);
        m.put("phi", 1.618);
        m.entrySet().stream()
                    .map(e -> e.getKey() + ": " + e.getValue())
                    .forEach(System.out::println);
    }
}
```

```java
Arrays.stream：将数组转换为流。
```

> 随机数流：使用Random类

使用ints、longs等方法。

> 通过流相关类的静态方法

`IntStream.range(10, 20).sum()`

> Stream.iterate()

通过迭代产生流，将第一次的结果作为下一次的参数：

```java
public class Fibonacci {
    int x = 1;
    
    Stream<Integer> numbers() {
        return Stream.iterate(0, i -> {
            int result = x + i;
            x = i;
            return result;
        });
    }
    
    public static void main(String[] args) {
        new Fibonacci().numbers()
                       .skip(20) // 过滤前 20 个
                       .limit(10) // 然后取 10 个
                       .forEach(System.out::println);
    }
}
```

> Stream.builder().build();

流的建造者模式，可以在build之前讲元素添加进去：

```java
public class FileToWordsBuilder {
    Stream.Builder<String> builder = Stream.builder();
    
    public FileToWordsBuilder(String filePath) throws Exception {
        Files.lines(Paths.get(filePath))
             .skip(1) // 略过开头的注释行
             .forEach(line -> {
                  for (String w : line.split("[ .?,]+"))
                      builder.add(w);
              });
    }
    
    Stream<String> stream() {
        return builder.build();
    }
    
    public static void main(String[] args) throws Exception {
        new FileToWordsBuilder("Cheese.dat")
            .stream()
            .limit(7)
            .map(w -> w + " ")
            .forEach(System.out::print);
    }
}
```

> 通过Pattern正则表达式将字符串转换为流

```java
return Pattern
        .compile("[ .,?]+").splitAsStream(all)
```

### 2.中间操作

> 跟踪和调试

peek()：查看元素，不改变流操作。

```java
class Peeking {
    public static void main(String[] args) throws Exception {
        FileToWords.stream("Cheese.dat")
        .skip(21)
        .limit(4)
        .map(w -> w + " ")
        .peek(System.out::print)
        .map(String::toUpperCase)
        .peek(System.out::print)
        .map(String::toLowerCase)
        .forEach(System.out::print);
    }
}
```

> 排序

sorted(Compator c)：排序，送入相应的比较器

```java
import java.util.*;
public class SortedComparator {
    public static void main(String[] args) throws Exception {
        FileToWords.stream("Cheese.dat")
        .skip(10)
        .limit(10)
        .sorted(Comparator.reverseOrder())
        .map(w -> w + " ")
        .forEach(System.out::print);
    }
}
```

> 消除重复元素

distinct()

> 过滤元素

filter()

```java
public class Prime {
    public static Boolean isPrime(long n) {
        return rangeClosed(2, (long)Math.sqrt(n))
        .noneMatch(i -> n % i == 0);
    }
    public LongStream numbers() {
        return iterate(2, i -> i + 1)
        .filter(Prime::isPrime);
    }
    public static void main(String[] args) {
        new Prime().numbers()
        .limit(10)
        .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        new Prime().numbers()
        .skip(90)
        .limit(10)
        .forEach(n -> System.out.format("%d ", n));
    }
}
```

> 转换元素

map()

> 组合流(扁平化流)

flatMap()：将流中流扁平化为一个流。

```java
import java.util.stream.*;
public class FlatMap {
    public static void main(String[] args) {
        Stream.of(1, 2, 3)
        .flatMap(i -> Stream.of("Gonzo", "Fozzie", "Beaker"))
        .forEach(System.out::println);
    }
}
```

### 3.Optional类

> 作用

1. 对流中的元素进行包裹的类，主要是为了防止空元素。
2. 可以作为终端的操作。

> 如何拿到Optional

- 使用Stream流的时候，当出现那种可能出现空值的情况：**[findFirst](itss://chm/java/util/stream/Stream.html#findFirst--)**() 、**[max](itss://chm/java/util/stream/Stream.html#max-java.util.Comparator-)**([Comparator](itss://chm/java/util/Comparator.html)<? super [T](itss://chm/java/util/stream/Stream.html)> comparator)等。
- 可以通过Optional.**[empty](itss://chm/java/util/Optional.html#empty--)**()、of、ofNullAble()等方法拿到。

> 主要使用场景

1. 在执行流的某些终端操作的时候：max()、findFirst()等，为了避免拿到空元素，使用Optional包裹，并且也可以通过Optional的包裹进一步对元素进行流的操作（例如map、filter等，但是仅仅是对Optional中那一个元素操作）。

```java
class OptionalFilter {
    static String[] elements = {
            "Foo", "", "Bar", "Baz", "Bingo"
    };
    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }
    static void test(String descr, Predicate<String> pred) {
        System.out.println(" ---( " + descr + " )---");
        for(int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst()
              //对Optional中元素也能进行filter等流的操作，但是它仅仅是对一个元素操作。
                            .filter(pred));
        }
    }
    public static void main(String[] args) {
        test("true", str -> true);
        test("false", str -> false);
        test("str != \"\"", str -> str != "");
        test("str.length() == 3", str -> str.length() == 3);
        test("startsWith(\"B\")",
                str -> str.startsWith("B"));
    }
}
```

2. 在某些时候，因为流中的元素可能为空，我们可以使用map将元素映射称为Optional，然后再将里面元素判断提取：

```java
public class Signal {
    private final String msg;
    public Signal(String msg) { this.msg = msg; }
    public String getMsg() { return msg; }
    @Override
    public String toString() {
        return "Signal(" + msg + ")";
    }
    static Random rand = new Random(47);
    public static Signal morse() {
        switch(rand.nextInt(4)) {
            case 1: return new Signal("dot");
            case 2: return new Signal("dash");
            default: return null;
        }
    }
    public static Stream<Optional<Signal>> stream() {
        return Stream.generate(Signal::morse)
                .map(signal -> Optional.ofNullable(signal));
    }
    
        public static void main(String[] args) {
        Signal.stream()
                .limit(10)
                .forEach(System.out::println);
        System.out.println(" ---");
        Signal.stream()
                .limit(10)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .forEach(System.out::println);
    }
}
```

### 4.终端操作

> foreach：循环遍历所有的元素

> toArray：转换为数组输出

> collect(Collector c)：将流转换为相应的集合

- Collector的使用：需要实现里面的三个方法：一个是返回一个集合、一个是往集合中添加元素、一个是集合中添加集合。

- 使用Collectors工具类中的一些方法，例如`toList()` `toSet()` `toCollection()`等：

举例：去除一些重复点，并且按某种方式进行排序：

```java
List<HistoryPoint.DataBean> collects = dataBeans.stream().collect(Collectors.collectingAndThen(
        Collectors.toCollection(
                () -> new TreeSet<>((o1, o2) -> {
                    if (DataHandlerUtil.isEqual(o1.getLatitude(), o2.getLatitude()) &&
                            DataHandlerUtil.isEqual(o1.getLongitude(), o2.getLongitude())) {
                        return 0;
                    } else {
                        return o1.getCreateDate().compareTo(o2.getCreateDate());
                    }
                }))
        , ArrayList::new)
);
```

- 使用自己实现的，其实也就是直接实现Collector的三个方法，它再帮你封装一下：

```java
public class SpecialCollector {
    public static void main(String[] args) throws Exception {
        ArrayList<String> words =
                FileToWords.stream("Cheese.dat")
                        .collect(ArrayList::new,
                                ArrayList::add,
                                ArrayList::addAll);
        words.stream()
                .filter(s -> s.equals("cheese"))
                .forEach(System.out::println);
    }
}
```

> 组合操作：reduce

其实就是将流中的所有元素进行一个处理：reduce(BinaryOperator)

```java
class Frobnitz {
    int size;
    Frobnitz(int sz) { size = sz; }
    @Override
    public String toString() {
        return "Frobnitz(" + size + ")";
    }
    // Generator:
    static Random rand = new Random(47);
    static final int BOUND = 100;
    static Frobnitz supply() {
        return new Frobnitz(rand.nextInt(BOUND));
    }
}
public class Reduce {
    public static void main(String[] args) {
        Stream.generate(Frobnitz::supply)
                .limit(10)
                .peek(System.out::println)
          // Lambda 表达式中的第一个参数 fr0 是 reduce() 中上一次调用的结果。而第二个参数 fr1 是从流传递过来的值。
                .reduce((fr0, fr1) -> fr0.size < 50 ? fr0 : fr1)
                .ifPresent(System.out::println);
    }
}

// 因为最后的结果可能不存在，所以会返回一个Optional，通过最后的判断来提取出元素。
```

> 匹配

- `allMatch(Predicate)` ：如果流的每个元素提供给 **Predicate** 都返回 true ，结果返回为 true。在第一个 false 时，则停止执行计算。
- `anyMatch(Predicate)`：如果流的任意一个元素提供给 **Predicate** 返回 true ，结果返回为 true。在第一个 true 是停止执行计算。
- `noneMatch(Predicate)`：如果流的每个元素提供给 **Predicate** 都返回 false 时，结果返回为 true。在第一个 true 时停止执行计算。

> Optional

包括了查找操作、最大值检索等

就是前面的findFirst() findAny() max()等，返回一个Optional。

> Stream的子类

例如`IntStream`等，可以有更多的操作，例如max()等。