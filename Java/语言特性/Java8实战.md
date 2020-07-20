# 流式API

> Java 8可以透明地把输入的不相关部分拿到几个CPU内核上去分别执行你的Stream操作流水线——这是几乎免费的并行

> 这两个要点（没有共享的可变数据，将方法和函数即代码传递给其他方法的能力）是我们平常所说的函数式编程范式的基石

> Java只允许值传递，原本的“一等值”只有值，类和方法是“二等值”。通过Stream，将方法变成“一等值”，也能进行传递。



```java

/**
 * 筛选apple
 */
@Test
public void test2() {
    ArrayList<Apple> apples = new ArrayList<>();
    apples.add(new Apple(100, "Green"));
    apples.add(new Apple(102, "Green"));
    apples.add(new Apple(105, "Blue"));
    apples.add(new Apple(107, "Red"));

    // 1.实现isGreenApple方法
    List<Apple> apples1 = filterApples(apples, Apple::isGreenApple);
    // 2.lambda函数
    List<Apple> apples2 = filterApples(apples, (Apple a)->"green".equalsIgnoreCase(a.getColor()));

    // 3.多条件的lambda函数
    List<Apple> apples3 = filterApples(apples, (Apple apple) -> "red".equalsIgnoreCase(apple.getColor()) || apple.getWeight() > 101);
   
    // 4.用原生的filter过滤器
    List<Apple> apples4 = apples.stream()
            .filter((Apple apple) -> "red".equalsIgnoreCase(apple.getColor()) || apple.getWeight() > 101)
            .collect(Collectors.toList());
    System.out.println(apples3);
}

/**
* 自定义筛选方式
* /
private List<Apple> filterApples(ArrayList<Apple> apples, Predicate<Apple> predicate) {
    ArrayList<Apple> apples1 = new ArrayList<>();
    for (Apple apple : apples) {
        if (predicate.test(apple)) {
            apples1.add(apple);
        }
    }
    return apples1;
}
```





# 通过参数化传递代码

给定一个Apple， 按照不同的方式输出apple。



## 原始传递函数方式



接口

```java
public interface AppleFormatter {
    String accept(Apple apple) ;
}
```

实现1

```java
public class AppleFancyFormatter implements AppleFormatter {
    @Override
    public String accept(Apple apple) {
        String characteristic = apple.getWeight() > 150 ? "heavy" : "light";
        return "A " + characteristic + " " + apple.getColor() + " apple";
    }
}
```

实现2

```java
public class AppleSimpleFormatter implements AppleFormatter {
    @Override
    public String accept(Apple apple) {
        return "An apple of" + apple.getWeight() + "g";
    }
}
```

测试

```java
@Test
public void test3() {

    // 用AppleFancyFormatter方式输出
    prettyPrintApple(apples, new AppleFancyFormatter());
    // 用AppleSimpleFormatter方式输出
    prettyPrintApple(apples, new AppleSimpleFormatter());
}

public void prettyPrintApple(List<Apple> inventory, AppleFormatter formatter) {
    for (Apple apple : inventory) {
        String accept = formatter.accept(apple);
        System.out.println(accept);
    }
}
```

> 优点：可以看到，我们把行为抽象出来，让代码适应需求的变化。
>
> 缺点：这个过程麻烦，因为需要声明只要实例化一次的类。



## 使用匿名类

**同时声明和实例化类**



```java
@Test
public void test3() {

    // 用AppleFancyFormatter方式输出
    prettyPrintApple(apples, new AppleFancyFormatter());
    // 用AppleSimpleFormatter方式输出
    prettyPrintApple(apples, new AppleSimpleFormatter());

    prettyPrintApple(apples, (Apple apple)->apple.getWeight() > 150?
            "A heavy " + apple.getColor() +" apple":
            "A light " + apple.getColor() +" apple");
}
```

现在我们只能用于Apple这一个类，来搞个泛型！



<hr/>

## 泛型

谓词接口

```java
public interface Predicate<T> {
    boolean test(T t) ;
}
```



泛型方法  注意需要加 <T>

```java
public <T> List<T> filter(List<T> list, Predicate<T> p) {
    ArrayList<T> ts = new ArrayList<>();
    for (T t : list) {
        if (p.test(t)) {
            ts.add(t);
        }
    }
    return ts;
}
```



测试

```java
    @Test
    public void test3() {
		
        // 我们只写了一个接口和一个test方法，具体的方法实现在调用的时候写个匿名方法传入
        List<Apple> filter = filter(apples, (Apple apple) -> "red".equalsIgnoreCase(apple.getColor()));
        for (Apple apple : filter) {
            System.out.println(apple);
        }
    }
```





 