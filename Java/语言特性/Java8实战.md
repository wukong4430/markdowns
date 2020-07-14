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

