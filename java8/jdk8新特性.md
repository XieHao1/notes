 JDK8新特性

Oracle JDK和Oracle JDK：

Oracle JDK是基于Open JDK源代码的商业版本。要学习Java新技术可以去Open JDK 官网学习。



# 一.lambda表达式



##  1.lambda的标准格式

Lambda省去面向对象的条条框框，Lambda的标准格式格式由3个部分组成：

```java
(参数类型 参数名称) -> { 
	代码体; 
}
```

**格式说明：**

- (参数类型 参数名称)：参数列表
- {代码体;}：方法体
- ->：箭头，分隔参数列表和方法体

```java
new Thread(	
    ///lambda就是一个匿名函数，我们只需要将要执行的代码放到lambda表达式中
    () ->{ System.out.println("lambda"); }
	).start();
```



## 2.lambda使用的条件

1. **方法的参数或局部变量类型必须为接口才能使用Lambda**
2. **接口中有且仅有一个抽象方法**
3. lambda相当于是对接口中抽象方法的重写 

```java
public interface LambdaDemoInterface {
    public abstract void test();
}
```

方法的参数是一个接口：

```java
public static void main(String[] args) {
        lambdaTest(() -> {
            System.out.println("lambda");
        });
    }
private static void lambdaTest(LambdaDemoInterface lambdaDemoInterface){
        lambdaDemoInterface.test();
}
```

**局部变量**是一个接口：

```java
LambdaDemoInterface lambdaDemoInterface = () -> System.out.println(1);
```



## 3.Lambda省略格式

1. 小括号内参数的类型可以省略

2. 如果小括号内**有且仅有一个参数**，则小括号可以省略

3. 如果大括号内**有且仅有一个语句**，可以同时省略大括号、return关键字及语句分号（三个必须同时省略）

```java
(int a) -> { return new Person(); }
```

```java
Map map = new TreeMap<>((o1,o2)->{
    return o1.getAge() - o2.getAge(); 
});
```

省略后

```java
a -> new Person()
```

```java
Map map = new TreeMap<>((o1,o2)->
	o1.getAge() - o2.getAge() 
 );
```



## 4.函数式接口

函数式接口在Java中是指：**有且仅有一个抽象方法的接口**。

函数式接口，即适用于函数式编程场景的接口。而Java中的函数式编程体现就是Lambda，所以函数式接口就是可以适用于Lambda使用的接口。只有**确保接口中有且仅有一个抽象方法**，Java中的Lambda才能顺利地进行推导。

### **@FunctionalInterface注解**

与 @Override 注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解： @FunctionalInterface 。该注

解可用于一个接口的定义上：

```java
@FunctionalInterface
//一旦使用该注解来定义接口，
//编译器将会强制检查该接口是否确实有且仅有一个抽象方法
public interface LambdaDemoInterface {
    public abstract void test();
}
```

一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。

不过，即使不使用该注解，只要满足函数式接口的定义，这仍然是一个函数式接口，使用起来都一样。



## 5.Lambda的实现原理

匿名内部类在编译后会形成一个新的类----**类名为本类类名＋$** 如：demo01$1 1表示第一个匿名内部类

![image-20220311154246138](img.assets\image-20220311154246138.png)

使用**javap**对字节码进行反汇编

在DOS命令行输入：

```cmd
javap -c -p 文件名.class  -c：表示对代码进行反汇编  -p：显示所有类和成员
```

![image-20220311155030123](img.assets\image-20220311155030123.png)

**lambda会在这个类中新生成一个静态的私有方法**

关于于这个方法 lambda$main$0 的命名：以lambda开头，因为是在main()函数里使用了lambda表达式，所以带有$main表示，因为是第一个，所以$0。

使用断点：

![image-20220311155522333](img.assets\image-20220311155522333.png)

我们可以发现 **lambda$main$0 里面放的就是Lambda中的内容**

如何调用这个方法：**Lambda在运行的时候会生成一个内部类**

​		在运行时加上 -Djdk.internal.lambda.dumpProxyClasses ，加上这个参数后，运行时会将生成的内部类class码输出到一个文件中。

```cmd
java -Djdk.internal.lambda.dumpProxyClasses 要运行的包名.类名
```

```cmd
D:\JAVA project\JDK8NewFeatures\out\production\JDK8NewFeatures>
java -Djdk.internal.lambda.dumpProxyClasses JDK8.lambda.demo01
```

在字节码文件中生成该内部类文件

![image-20220311161237850](img.assets\image-20220311161237850.png)

```java
final class demo01$$Lambda$1 implements Runnable{
	private demo01$$Lambda$1(){}
	public void run()
	{
		demo01.lambda$main$0();
	}
}
```

所以lambda表达式相当于

```java
new Thread(new Runnable() { //生成demo01$$Lambda$1类
     @Override
     public void run() {
        demo01.lambda$main$0();//lambda$main$0 里面放的就是Lambda中的内容
      }}).start();
```

### 总结

匿名内部类在编译的时候会一个class文件

Lambda在程序运行的时候形成一个类

1. 在类中新增一个方法,这个方法的方法体就是Lambda表达式中的代码

2. 还会形成一个匿名内部类,实现接口,重写抽象方法

3. 在接口的重写方法中会调用新生成的方法



## 6.Lambda和匿名内部类对比

```java
//匿名内部类做了什么事情：
//1.定义了一个没有名字的类
//2.这个类实现了Runnable接口，重写了run方法
//3.创建了这个类的实例对象
new Thread(new Runnable() {
     @Override
     public void run() {
        System.out.println("执行线程方法");
            }
	}).start();

//lambda表达式体现的是函数式编程思想，只需要将执行的代码放到函数中
//函数就是类中的方法
//lambda就是一个匿名函数，我们只需要将要执行的代码放到lambda表达式中
   new Thread(() ->System.out.println("lambda")).start();
```

| 区别           | 匿名内部类                     | Lambda                            |
| -------------- | ------------------------------ | --------------------------------- |
| 所需的类型     | 需要的类型可以是类,抽象类,接口 | 需要的类型必须是接口              |
| 抽象方法的数量 | 所需的接口中抽象方法的数量随意 | 所需的接口只能有一个抽象方法      |
| 实现原理       | 在**编译后会形成class**        | 在**程序运行的时候动态生成class** |

当接口中只有一个抽象方法时,建议使用Lambda表达式,其他其他情况还是需要使用匿名内部类



# 二.接口新增方法

jdk以前的接口:

```java
interface 接口名 { 静态常量; 抽象方法; }
```

JDK 8对接口的增强，接口还可以有**默认方法**和**静态方法**

```java
interface 接口名 { 静态常量; 抽象方法; 默认方法; 静态方法; }
```

## 1.接口默认方法

```java
interface 接口名 { 修饰符 default 返回值类型 方法名() { 代码; } }
```

```java
public interface InterfaceDemo01 {
    default void test(){
        System.out.println("默认方法");
    }}
```

**接口默认方法的使用**

方式一：实现类直接调用接口默认方法

方式二：实现类重写接口默认方法

```java
public class demo3 implements InterfaceDemo01{
    public static void main(String[] args) {
        //实现类可以直接使用接口中的默认方法
        //也可以重写默认方法
        new demo3().test();}
    @Override
    public void test() {
        System.out.println("重写后的默认方法");
    }
}
```

## 2.接口静态方法

```java
interface 接口名 { 修饰符 static 返回值类型 方法名() { 代码; } }
```

```java
public interface InterfaceDemo01 {    
    static void test1(){
        System.out.println("接口中的静态方法");
    }
}
```

**接口静态方法的使用**

直接使用接口名调用即可：**接口名.静态方法名();**

**接口中的静态方法不能重写**

```java
InterfaceDemo01.test1();//接口中的静态方法
```

## 3.**接口默认方法和静态方法的区别**

|            | 默认方法                                           | 静态方法                               |
| ---------- | -------------------------------------------------- | -------------------------------------- |
| 调用       | 实例调用                                           | 接口名调用                             |
| 继承和重写 | 可以被继承，实现类可以直接使用接口默认方法或者重写 | 不能被继承，实现类不能重写接口静态方法 |



# 三.**常用内置函数式接口**

Lambda使用时不关心接口名，抽象方法名，**只关心抽象方法的参数列表和返回值类型**。因此为了让我们使用Lambda方便，JDK提供了大量常用的函数式接口。它们主要在 **java.util.function 包**中



## 1.Supplier接口

java.util.function.Supplier<T> 接口，它意味着"供给" , 对应的Lambda表达式需要“**对外提供**”一个符合泛型类型的对象数据

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**供给型接口**，通过Supplier接口中的get方法可以得到一个值，无参有返回的接口。

**使用Lambda表达式返回数组元素最大值**

```java
public class Demo5 {
    //使用Lambda表达式返回数组元素的最大值
    public static void main(String[] args) {
        getMax(()->{
            int[] arr = {11,12,124,2532,123};
            Arrays.sort(arr);//升序排序
            return arr[arr.length-1];
        });
    }
    private static void getMax(Supplier<Integer> supplier){
        int max = supplier.get();
        System.out.println("max = " + max);
    }
}
```

## 2.Consumer接口

java.util.function.Consumer<T> 接口则正好相反，它不是生产一个数据，而是**消费**一个数据，其数据类型由泛型参数决定。**消费型接口**

```java
@FunctionalInterface 
public interface Consumer<T> { 
    void accept(T t); 
    
	default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        //备注： java.util.Objects 的 requireNonNull 静态方法将会在参数为null时主动抛出
		//NullPointerException 异常。这省去了重复编写if语句和抛出空指针异常的麻烦。
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

**使用Lambda表达式将一个字符串转成大写和小写的字符串**

```java
public class Demo6 {
    public static void main(String[] args) {
        test(s -> System.out.println(s.toUpperCase()));
    }
    private static void test(Consumer<String> consumer){
        consumer.accept("hello world");
    }
}
```

**默认方法:andThen**

如果一个方法的参数和返回值全都是 Consumer 类型，那么就可以实现效果：消费一个数据的时候，首先做一个操作，然后再做一个操作，实现组合。而这个方法就是 Consumer 接口中的default方法 andThen 。

```java
test1(s -> System.out.println(s.toUpperCase()),s -> System.out.println(s.toLowerCase())); 

private static void test1(Consumer<String> consumer1,Consumer<String> consumer2){
    	//先执行consumer1，再执行consumer2
        consumer1.andThen(consumer2).accept("helloWorld");
    }
```



## 3.Function接口

java.util.function.Function<T,R> 接口用来**根据一个类型的数据得到另一个类型的数据**，前者称为前置条件，

后者称为后置条件。有参数有返回值。

**Function的前置条件泛型和后置条件泛型可以相同。**

```java
@FunctionalInterface
public interface Function<T, R> {
 	R apply(T t);
    
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
      
  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
}
```

**将integer转换为字符串**

```java
public static void main(String[] args) {
      test(a -> String.valueOf(a));
}
//将integer转换为字符串
private static void test(Function<Integer,String> function){
   String apply = function.apply(100);
   System.out.println(apply.length());
}
```

**使用链式将integer转换为字符串然后再转换integer**

```java
test1(a -> String.valueOf(a),a -> Integer.parseInt(a));

private static void test1(Function<Integer,String> function1,Function<String,Integer> function2){
      //先执行function1，再执行function2
      Integer apply = function1.andThen(function2).apply(100);
      System.out.println(apply+100);
}
```



## 4.Predicate接口

**对某种类型的数据进行判断**，从而得到一个boolean值结果。这时可以使用java.util.function.Predicate<T> 接口。

```java
@FunctionalInterface
public interface Predicate<T> {
    
 	boolean test(T t);
    
	default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

**判断字符串长度**

```java
public static void main(String[] args) {
        test(a->a.length()>3,"VVVVV");
    }
    //判断字符串长度
    private static void test(Predicate<String> predicate ,String str){
        boolean test = predicate.test(str);
        System.out.println(test);
    }
```

**默认方法：and，or，negate**，对应与、或、非三种常见的逻辑关系

```java
 testAnd(a->a.length()>5,a->a.contains("W"),"agasgWWxzgsdhds");

 testNegate(a->a.length()>5);

 testOr(a->a.length()>100,a->a.contains("s"));
 
//and
 private static void testAnd(Predicate<String> predicate1,Predicate<String> predicate2,String str){
   
        boolean test = predicate1.and(predicate2).test(str);
        System.out.println(test);
    }
	//取反
  private static void testNegate(Predicate<String> predicate){
        boolean qwer = predicate.negate().test("qwer");
        System.out.println(qwer);
    }
  //or
  private static void testOr(Predicate<String> predicate1,Predicate<String> predicate2){
        boolean asgasgsdga = predicate1.or(predicate2).test("Asgasgsdga");
        System.out.println(asgasgsdga);
    }
```



# 四.方法引用

方法引用是对Lambda表达式符合特定情况下的一种缩写，它使得我们的Lambda表达式更加的精简，也可以理解为Lambda表达式的缩写形式 , 不过要注意的是**方法引用只能"引用"已经存在的方法!**



## 1.方法引用的格式

**符号表示** :  **`::`** 双冒号 :: 写法，这被称为“**方法引用**”，是一种新的语法。

**符号说明** **:** 双冒号为方法引用运算符，而它所在的表达式被称为**方法引用**。

**应用场景** **:** 如果Lambda所要实现的方案 , 已经有其他方法存在相同方案，那么则可以使用方法引用。

**方法引用的注意事项:**

1. 被引用的方法，参数要和接口中抽象方法的参数一样

2. 当接口抽象方法有返回值时，被引用的方法也必须有返回值

```java
	public static void main(String[] args) {
        //使用传统lambda表达式
        consumerSum(a->sum(a));
        //使用方法引用
        //相当于让这个指定的方法去重写接口的抽象方法
        //调用接口的抽象方法就是调用传递过去的方法
        consumerSum(Demo10::sum);
    }
    private static void sum(int[] arr){
        int sum = 0;
        for (int a : arr){
            sum += a;
        }
        System.out.println(sum);
    }
    private static void consumerSum(Consumer<int[]> consumer){
        int[] arr = {11,12,334,354,67457};
        consumer.accept(arr);
    }
```



## 2.**常见引用方式**

1. `instanceName::methodName 对象::方法名`

2. `ClassName::staticMethodName 类名::静态方法`

3. `ClassName::methodName 类名::普通方法`

4. `ClassName::new 类名::new 调用的构造器`

5. `TypeName[]::new String[]::new 调用数组的构造器`



### (1).对象名::引用成员方法

最常见的一种用法，如果一个类中已经存在了一个成员方法，则可以通过对象名引用成员方法。

```java
public static void main(String[] args) {
        Date date = new Date();
        Supplier<Long> supplier1 = ()-> date.getTime();
        //使用方法引用
        Supplier<Long> supplier2 = date::getTime;
        //date::getTime ----相当于--->  ()-> date.getTime()
        //date::getTime相当于重写接口中的抽象方法，
        System.out.println(supplier1.get());
        System.out.println(supplier2.get());
}
```



### (2).类名::引用静态方法

```java
public static void main(String[] args) {
        //使用lambda表达式
        Supplier<Long> supplier1 = () -> System.currentTimeMillis();
        System.out.println(supplier1.get());
        //使用方法引用
        Supplier<Long> supplier2 = System::currentTimeMillis;
        System.out.println(supplier2.get());
}
```



### (3).类名::引用实例方法

Java面向对象中，类名只能调用静态方法，类名引用实例方法是有前提的，实际上是**拿第一个参数作为方法的调用**

**者，后面的参数作为方法的参数。**

```java
public static void main(String[] args) {
        //使用lambda表达式
        Function<String,Integer> function1 = (a) -> a.length();
        System.out.println(function1.apply("HelloWord!"));
        
    	//使用方法引用
        Function<String,Integer> function2 = String::length;
        //拿第一个参数作为方法的调用着
        //相当于"HelloWord!".length
        System.out.println(function2.apply("HelloWord!"));
        
    	BiFunction<String,Integer,String> biFunction = String::substring;
        //相当于"HelloWord!".substring(3)
        System.out.println(biFunction.apply("HelloWord!",3));
}
```



### (4).类名::new引用构造器

**根据赋于的方法不同选择调用有参或者无参的构造方法**

```java
public static void main(String[] args) {
        //使用lambda表达式
        Supplier<Person> supplier1 = ()->new Person("zs");
        System.out.println(supplier1.get().getName());
        //使用方法引用
        //根据赋于的方法不同选择调用有参或者无参的构造方法
        //调用无参构造
        Supplier<Person> supplier2= Person::new;
        //get()无参，调用无参构造
        System.out.println(supplier2.get().getName());
        //调用有参构造
        Function<String,Person> function = Person::new;
        //apply("zs")有参，调用有参数的构造方法
        System.out.println(function.apply("zs").getName());
}
class Person{
    private String name;
    public Person() {}
    public Person(String name) { this.name = name; }
    public String getName() { return name; }
}
```



### (5).数组::new引用数组构造器

数组也是 Object 的子类对象，所以同样具有构造器，只是语法稍有不同。

```java
public static void main(String[] args) {
        //使用lambda表达式
        Supplier<int[]> supplier1 = ()-> new int[5];
        System.out.println(supplier1.get().length);
        //数组需要传递数组长度
        Function<Integer,int[]> function1 = a->new int[a];
        System.out.println(function1.apply(5).length);
        //使用方法引用
        Function<Integer,int[]> function2 = int[]::new;
        System.out.println(function2.apply(5).length);
}
```



# 五.集合的Stream流

**Stream和IO流(InputStream/OutputStream)没有任何关系**

Stream流式思想类似于工厂车间的“**生产流水线**”，Stream流不是一种数据结构，不保存数据，而是对数据进行加工处理。Stream可以看作是流水线上的一个工序。在流水线上，通过多个工序让一个原材料加工成一个商品。

Stream API能让我们快速完成许多复杂的操作，如筛选、切片、映射、查找、去除重复，统计，匹配和归约。



## 1.获取Stream流常用的两种方式

java.util.stream.Stream<T> 是JDK 8新加入的流接口。

获取一个流非常简单，有以下几种常用的方式：

​	所有的 Collection 集合都可以通过 stream 默认方法获取流；

​	Stream 接口的静态方法 of 可以获取数组对应的流。

### (1).使用Collection获取流

java.util**.Collection 接口**中**加入了default方法 stream** 用来获取流，所以其所有实现类均可获取流。

java.util.Map 接口不是 Collection 的子接口，所以获取对应的流需要分key、value或entry等情况：

```java
public interface Collection { default Stream<E> stream() }
```

```java
 List<String> list = new ArrayList<>();
 Collections.addAll(list,"zs","ls","ww","zl");
 //获取Stream流
 Stream<String> stream = list.stream();
        
 Set<String> set = new HashSet<>();
 Stream<String> stream1 = set.stream();
        
 //获取Map接口的Stream流
 Map<String,String> map = new HashMap<>();
 Stream<Map.Entry<String, String>> stream2 = map.entrySet().stream();
```

### (3).Stream中的静态方法of获取流

由于数组对象不可能添加默认方法，所以 Stream 接口中提供了静态方法 of 。

```java
public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }
    
//常用
public static<T> Stream<T> of(T... values) {
    //of 方法的参数其实是一个可变参数，所以支持数组。
        return Arrays.stream(values);
    }
```

```java
Stream<String> streamString = Stream.of("zs", "ls", "ww", "zl");
//可变参数类型可以传入一个数组
String[] str = {"zs", "ls", "ww", "zl"};
Stream<String> str1 = Stream.of(str);

//基本数据类型数组Stream会将整个数据当作一个元素进行操作
int[] a = {11,12,13};
Stream<int[]> a1 = Stream.of(a);
//引用数据类型的数据是对数组中的元素进行操作
Integer[] a2 = {11,12,13};
Stream<Integer> a21 = Stream.of(a2);
```



## 2.Stream常用方法

| **方法名** | **方法作用** | **返回值类型** | 方法种类 |
| ---------- | ------------ | -------------- | -------- |
| count      | 统计个数     | long           | 终结     |
| forEach    | 逐一处理     | void           | 终结     |
| fifilter   | 过滤         | Stream         | 函数拼接 |
| limit      | 取用前几个   | Stream         | 函数拼接 |
| skip       | 跳过前几个   | Stream         | 函数拼接 |
| map        | 映射         | Stream         | 函数拼接 |
| concat     | 组合         | Stream         | 函数拼接 |

**终结方法**：返回值类型不再是 Stream 类型的方法，不再支持链式调用。

**非终结方法**：返回值类型仍然是 Stream 类型的方法，支持链式调用。（除了终结方法外，其余方法均为非终结

方法。）

**Stream方法注意事项(重要)**:

1. Stream只能操作一次(Stream流用完一次就会关闭)

   ```java
   tream<Integer> integerStream = Stream.of(11, 12, 13);
   //第一次调用没有问题
   long count = integerStream.count();
   //第二次调用
   //stream has already been operated upon or closed
   long count1 = integerStream.count();
   ```

2. Stream方法返回的是新的流

   ```java
   Stream<Integer> integerStream = Stream.of(11, 12, 13);
   Stream<Integer> limit = integerStream.limit(1);
   System.out.println("integerStream------->"+integerStream);
   //integerStream------->java.util.stream.ReferencePipeline$Head@4554617c
   System.out.println("limit-------->"+limit);
   //limit-------->java.util.stream.SliceOps$1@74a14482
   ```

3. Stream不调用终结方法，中间的操作不会执行

   ```java
   Stream<Integer> integerStream = Stream.of(11, 12, 13);
   //'Stream.filter()' 的结果被忽略
   integerStream.filter(a->{
   //以下代码不会执行
      System.out.println(a);
      return true;
   });
   ```

   

###  (1).forEach

forEach 用来遍历流中的数据

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个 Consumer 接口函数，会将**每一个流元素**交给该函数进行处理

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
//实例::实例方法
integerStream.forEach(System.out::println);
```



### (2).**count**

Stream流提供 count 方法来统计其中的元素个数：

```java
long count();
```

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
long count = integerStream.count();
System.out.println(count);
```



### (3).filter

fifilter用于过滤数据，返回符合过滤条件的数据

```java
Stream<T> filter(Predicate<? super T> predicate);
```

可以通过 filter 方法将一个流转换成另一个子集流

该接口接收一个 Predicate 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件。

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
//filter方法返回一个新流，若没有终结方法，则中间的操作不会执行
integerStream.filter(a -> a >= 12).forEach(System.out::print);
```



### (4).limit

limit 方法可以对流进行截取，只取用前n个。

```java
Stream<T> limit(long maxSize);
```

参数是一个long型，如果集合**当前长度大于参数则进行截取**。否则不进行操作。

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
integerStream.limit(2).forEach(System.out::println);
```



### (5).**skip**

跳过前几个元素，可以使用 skip 方法获取一个截取之后的新流

```java
Stream<T> skip(long n);
```

如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
integerStream.skip(1).forEach(System.out::println);
```



### (6).map

将流中的元素映射到另一个流中，可以使用 map 方法。

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

该接口需要一个 Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

```java
Stream<Integer> integerStream = Stream.of(11, 12, 13);
//将int转换为字符串
integerStream.map(String::valueOf).forEach(System.out::println);
```



### (7).sorted

如果需要将数据排序，可以使用 sorted 方法。

```java
Stream<T> sorted();  //根据元素的自然顺序排序
Stream<T> sorted(Comparator<? super T> comparator); //根据比较器指定的规则排序
```

```java
Stream<Integer> integerStream = Stream.of(13, 12, 11);
//升序排序
integerStream.sorted().forEach(System.out::println);
Stream<String> stringStream = Stream.of("afac","afag","awggsv","egs");
//自定义排序规则
stringStream.sorted(String::compareTo).forEach(System.out::println);
```



### (8).distinct

去除重复数据，可以使用 distinct 方法

```java
Stream<T> distinct();
```

```java
Stream<Integer> integerStream = Stream.of(13, 12, 11,11,23,11);
//去除重复数据
integerStream.distinct().forEach(System.out::println);
```

自定义类型是根据对象的hashCode和equals来去除重复元素

```java
 //对自定义类进行排除
Stream.of(new Person("zs",13),
          new Person("zs",14),
          new Person("zs",13),
          new Person("ll",14)).distinct().forEach(System.out::println);

//重写equls和hashcode
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Person person = (Person) o;
    return age == person.age && Objects.equals(name, person.name);
}

@Override
public int hashCode() {
     return Objects.hash(name, age);
}
```



### (9).match

要判断数据是否匹配指定的条件，可以使用 Match 相关方法

Match 相关方法是一个终结方法,和filter方法不同，**match侧重于判断,集合中元素要满足指定的条件，若满足，则返回true，否则返回false**，filter则是过滤，对集合中的元素进行过滤，返回一个新流

```java
boolean allMatch(Predicate<? super T> predicate);//元素是否全部满足条件
boolean anyMatch(Predicate<? super T> predicate); //元素是否任意有一个满足条件
boolean noneMatch(Predicate<? super T> predicate);//元素是否全部不满足条件
```

```java
boolean b = Stream.of(5, 3, 6, 1) 
	// .allMatch(e -> e > 0); // allMatch: 元素是否全部满足条件 
	// .anyMatch(e -> e > 5); // anyMatch: 元素是否任意有一个满足条件 
	.noneMatch(e -> e < 0); // noneMatch: 元素是否全部不满足条件 
	System.out.println("b = " + b);
```



### (10).find

找到某些数据，可以使用 find 相关方法

```java
//找第一个数据
//findFirst保证总能取到第一个元素
Optional<T> findFirst(); 
//findAny在并行的条件下，不能保证取到的是第一个
Optional<T> findAny();
```

```java
Optional<String> first = Stream.of("123", "12345", "13548231", "asasg").findFirst();
System.out.println(first.get());

Optional<String> any = Stream.of("123", "12345", "13548231", "asasg").findAny();
System.out.println(any.get());
```



### (11).max和min

要获取最大和最小值，可以使用 max 和 min 方法

```java
Optional<T> max(Comparator<? super T> comparator); 
Optional<T> min(Comparator<? super T> comparator);
```

```java
Optional<String> max = Stream.of("123", "12345", "13548231", "asasg", "qqqwww").
                                 max(String::compareTo);
System.out.println(max.get());

Optional<String> min = Stream.of("123", "12345", "13548231", "asasg", "qqqwww").
                                 min(String::compareTo);
System.out.println(min.get());
```



### (12).reduce

将所有数据归纳得到一个数据，可以使用 reduce 方法。

```java
T reduce(T identity, BinaryOperator<T> accumulator);
//identity默认值
//accumulator 如何对数据进行处理
//BinaryOperator<T> 继承了BiFunction接口

Optional<T> reduce(BinaryOperator<T> accumulator);
```

```java
interface BinaryOperator<T> extends BiFunction<T,T,T>
```

对数据进行求和：

```java
Integer reduce = Stream.of(1, 2, 3, 4, 5).reduce(0, Integer::sum);
//identity默认值
//accumulator 如何对数据进行处理
ystem.out.println(reduce);//15

//获取最大值
String reduce2 = Stream.of("123", "234", "zbc", "accc").reduce("",
                          (x, y) -> x.compareTo(y) > 0 ? x : y);
```

如何实现:

第一次将默认做赋值给x, 取出第一个元素赋值给y,进行操作 

第二次,将第一次的结果赋值给x, 取出二个元素赋值给y,进行操作 

第三次,将第二次的结果赋值给x, 取出三个元素赋值给y,进行操作 

第四次,将第三次的结果赋值给x, 取出四个元素赋值给y,进行操作 

```java
nteger reduce1 = Stream.of(1, 2, 3, 4, 5).reduce(0, (x, y) -> {
            System.out.println("x=" + x + ",y=" + y + ",x+y=" + (x+y));
            //x=0,y=1,x+y=1
            //x=1,y=2,x+y=3
            //x=3,y=3,x+y=6
            //x=6,y=4,x+y=10
            //x=10,y=5,x+y=15
            return x + y;
        });
```

![image-20220315222603374](img.assets\image-20220315222603374.png)



#### map和reduce组合

```java
//求所有年龄的和
Integer reduce = Stream.of(new Person("zs", 13),
          		new Person("zs", 14),
                new Person("zs", 13),
                new Person("ll", 14))
    			//获取所有的年龄，将对象转换为其中的年龄
                .map(Person::getAge)
    			//求所有年龄的和
                .reduce(0, Integer::sum);
//求年龄的最大值
Integer reduce = Stream.of(new Person("zs", 13),
          		new Person("zs", 14),
                new Person("zs", 13),
                new Person("ll", 14))
    			//获取所有的年龄，将对象转换为其中的年龄
                .map(Person::getAge)
    			//求所有年龄的和
                .reduce(0, Integer::max);
				//.max((o1, o2) -> o1 - o2).get();

// 统计 字母a 出现的次数
Integer reduce = Stream.of("a", "v", "a", "u", "a", "s", "a", "t", "w")
                //将为a的映射为1，不是a的映射为0
                .map(a -> "a".equals(a) ? 1 : 0)
                .reduce(0, Integer::sum);
//使用fifter和count实现
long count = Stream.of("a", "v", "a", "u", "a", "s", "a", "t", "w")
                        .filter("a"::equals)
                        .count();
```



### (13).mapToInt

将Stream中的Integer类型数据转成int类型，可以使用 mapToInt 方法。

```java
IntStream mapToInt(ToIntFunction<? super T> mapper);
```

```java
public interface ToIntFunction<T> {
    int applyAsInt(T value);
}
```

```java
long time1 = System.currentTimeMillis();
//Integer占用的内存比int多，在Stream流中会自动装箱和拆箱
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4);
integerStream.filter(a-> a > 3).forEach(System.out::println);
long time2 = System.currentTimeMillis();
System.out.println("所需要的时间"+(time2-time1));

//为了节约内存,增加运行速度，使用 mapToInt 方法
Stream.of(1,2,3,4).mapToInt(Integer::intValue)
                  .filter(a-> a > 3)
                  .forEach(System.out::println);

long time3 = System.currentTimeMillis();
System.out.println("所需要的时间"+(time3-time2));

//4
//所需要的时间42
//4
//所需要的时间3
```

IntStream:内部操作的是int数据类型，可以节约内存，减少自动装箱和查询

![image-20220315231008996](img.assets\image-20220315231008996.png)



### (14).concat

如果有两个流，希望合并成为一个流，那么可以使用 Stream 接口的静态方法 concat 

```java
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
//备注：这是一个静态方法，与 java.lang.String 当中的 concat 方法是不同的。
```

```java
Stream<String> zs = Stream.of("zs");
Stream<String> ls = Stream.of("ls");
//将两个流合并为一个流
Stream.concat(zs,ls).forEach(System.out::println);
```



## 3.收集Stream流中的结果

对流操作完成之后，如果需要将流的结果保存到数组或集合中，可以收集流中的数据

### (1).Stream流中的结果到集合中

Stream流提供 collect 方法，其参数需要一个 java.util.stream.Collector<T,A, R> 接口对象来指定收集到哪

种集合中。java.util.stream.Collectors 类提供一些方法，可以作为 Collector接口的实例：

```java
public static <T> Collector<T, ?, List<T>> toList() ：//转换为 List 集合。
public static <T> Collector<T, ?, Set<T>> toSet() ：//转换为 Set 集合。
```

```java
//将流中的数据转换为list集合
List<String> collect = Stream.of("12", "as", "wq").collect(Collectors.toList());
System.out.println(collect);
//将流中得到数据转换为set集合
Set<String> as = Stream.of("12", "as", "21").collect(Collectors.toSet());
System.out.println(as);
        
//将集合中的数据收集到指定的集合中
ArrayList<String> as1 = Stream.of("12", "as", "21")
                                .collect(Collectors.toCollection(ArrayList::new));
TreeSet<String> as2 = Stream.of("12", "as", "21")
                                .collect(Collectors.toCollection(TreeSet::new));
```



### (2).Stream流中的结果到数组中

Stream提供 toArray 方法来将结果放到一个数组中，返回值类型是Object[]的：

```java
Object[] toArray();
<A> A[] toArray(IntFunction<A[]> generator);
```

```java
@FunctionalInterface
public interface IntFunction<R> {  R apply(int value); }
```

```java
//将流中的元素转换为数组
//转成object数组不方便使用
Object[] objects = Stream.of(1, 2, 3, 4, 5).toArray();
//指定要转换为的数组
Integer[] as3 = Stream.of(1, 2, 3, 4, 5).toArray(Integer[]::new);
int[] ints = Stream.of(1, 2, 3, 4, 5).mapToInt(Integer::intValue).toArray();
```



### (3).对流中数据进行聚合计算

当我们使用Stream流处理数据后，可以像数据库的聚合函数一样对某个字段进行操作。比如获取最大值，获取最小值，求总和，平均值，统计数量。

```java
collect(Collectors.minBy()) //求最小值
collect(Collectors.maxBy()) //求最大值
collect(Collectors.summingInt()) //求和
collect(Collectors.averagingDouble()) //求平均值
collect(Collectors.counting()) //统计数量
```

```java
List<Student> list = new ArrayList<>();
Collections.addAll(list,new Student("张三",15,750),
                        new Student("李四",18,469),
                        new Student("王五 ",19,260),
                        new Student("赵六",25,666),
//获取最大值
int age = list.stream().max((o1, o2) -> o1.getAge() - o2.getAge()).get().getAge();
int age1 = list.stream()
               .collect(Collectors.maxBy((o1, o2) -> o1.getAge() - o2.getAge()))
                //collect(maxBy())' 可以替换为 'max()' （可能会改变语义）
               .get()
               .getAge();
System.out.println(age + "--------------" + age1);

//获取最小值
int age2 = list.stream().min((o1, o2) -> o1.getAge() - o2.getAge()).get().getAge();
int age3 = list.stream()
               .collect(Collectors.minBy((o1, o2) -> o1.getAge() - o2.getAge()))
               .get()
               .getAge();
System.out.println(age2 + "--------------" + age3);

//求和
Integer sum = list.stream().map(Student::getScore).reduce(0, Integer::sum);
int sum1 = list.stream().mapToInt(Student::getScore).sum();
Integer sum2 = list.stream().collect(Collectors.summingInt(Student::getScore));
//'collect(summingInt())' can be replaced with 'mapToInt().sum()'
System.out.println(sum +"------"+sum1+"-----"+sum2);

//求平均值
double average1 = list.stream().mapToInt(Student::getScore).average().getAsDouble();
double average2 = list.stream().collect(Collectors.averagingDouble(Student::getScore));
System.out.println(average1 + "------ " + average2);

//统计个数
long count = list.size();
long count1 = list.stream().count();
long count2 = list.stream().collect(Collectors.counting()).longValue();
System.out.println(count+"----"+count1+"------"+count2);
```



### (4).对流中数据进行分组

当我们使用Stream流处理数据后，可以根据某个属性将数据分组

```java
//进行一次分组
public static <T, K> Collector<T, ?, Map<K, List<T>>>
groupingBy(Function<? super T, ? extends K> classifier) {
    return groupingBy(classifier, toList());
}

//经行两次分组
public static <T, K, A, D>
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                          Collector<? super T, A, D> downstream) {
return groupingBy(classifier, HashMap::new, downstream);
}
```

```java
//根据年龄分组
Map<Integer, List<Student>> collect = list.stream().collect(Collectors.groupingBy(Student::getAge));
//map集合的key为groupingBy返回的数据
collect.forEach((k,v) -> System.out.println(k +"::"+ v));

//按指定分数分组
Map<String, List<Student>> collect1 = list.stream()
//map集合的key为groupingBy返回的数据
 			.collect(Collectors.groupingBy(a -> a.getScore() > 500 ? "及格" : "不及格"));
collect1.forEach((k,v) -> System.out.println(k +"::"+ v));
//不及格::[Student{name='李四', age=18, score=469}, Student{name='王五 ', age=18, score=260}]
//及格::[Student{name='张三', age=15, score=750}, Student{name='赵六', age=25, score=666}, 
//      Student{name='七七', age=66, score=658}]
```

```java
//先根据年龄分组，再根据分数分组
Map<Integer, Map<Integer, List<Student>>> collect2 =list.stream()
                .collect(Collectors.groupingBy(Student::getAge,                                         	 Collectors.groupingBy(Student::getScore)));
collect2.forEach((k,v) -> System.out.println(k +"::"+ v));
```



### (5).对流中数据进行分区

Collectors.partitioningBy 会根据值是否为true，把集合分割为两个列表，一个true列表，一个false列表。

**分区只有两个区，一个为true，一个为false**

**分组可以根据字段分成多个组**

![image-20220316135950820](img.assets\image-20220316135950820.png)

```java
//年龄大于18的为一组，key为true或false
Map<Boolean, List<Student>> collect3 = list.stream()
             .collect(Collectors.partitioningBy(a -> a.getAge() > 18));
collect3.forEach((k,v) -> System.out.println(k +"::"+ v));

//false::[Student{name='张三', age=15, score=750}, Student{name='李四', age=18, score=260}, 
//true::[Student{name='赵六', age=25, score=666}, Student{name='七七', age=66, score=666}]
```



### (6).对流中数据进行拼接

Collectors.joining 会根据指定的连接符，将所有元素连接成一个字符串。

```java
Collector<CharSequence, ?, String> joining() //直接拼接
static Collector<CharSequence, ?, String> joining(CharSequence delimiter) //按指定字符拼接
static Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                                  CharSequence prefix,
                                                  CharSequence suffix) //有前缀和后缀
```

```java
//将名字用--拼接
String collect4 = list.stream().map(Student::getName).collect(Collectors.joining("--"));
System.out.println(collect4);
//张三--李四--王五--赵六--七七

String collect5 = list.stream().map(Student::getName).collect(Collectors.joining("-", "<", ">"));
System.out.println(collect5);
//<张三-李四-王五-赵六-七七>
```



### (7).小结

收集Stream流中的结果

到集合中: Collectors.toList()/Collectors.toSet()/Collectors.toCollection()

到数组中: toArray()/toArray(int[]::new)

聚合计算:

Collectors.maxBy/Collectors.minBy/Collectors.counting/Collectors.summingInt/Collectors.averagingInt

分组: Collectors.groupingBy

分区: Collectors.partitionBy

拼接: Collectors.joing



## 4.并行的Stream流

**串行的Stream流**

目前我们使用的Stream流是串行的，就是在一个线程上执行。

```java
public static void main(String[] args) {
        Stream.of(1,2,3,4,5,6).forEach(a->{
            System.out.println(Thread.currentThread().getName()+"::"+a);
            //main::1
			//main::2 ……
        });
 }
```

**并行的Stream流**

**parallelStream**其实就是一个并行执行的流。它通过默认的ForkJoinPool，可能提高多线程任务的速度。



### (1).获取并行Stream流的两种方式

1. 直接获取并行流: **parallelStream()**
2. 将串行流转成并行流: **parallel()**

```java
List<Integer> list = new ArrayList<>();
//直接获取并行的流
Stream<Integer> integerStream = list.parallelStream();
//将串行流转成并行流
Stream<Integer> parallel = list.stream().parallel();
```

```java
Stream.of(1,2,3,4).parallel().forEach(a->{
            System.out.println(Thread.currentThread().getName()+"--"+a);
            //main--3
            //main--4
            //ForkJoinPool.commonPool-worker-9--2
            //ForkJoinPool.commonPool-worker-2--1});
```

**并行和串行Stream流的效率对比**

```java
private static final long TIMES = 500000000L;
long start = System.currentTimeMillis();
        long sum = 0;
        //使用for循环 所需时间=135
		//for (long i =0; i<=TIMES;i++){
        //sum += i;
       	// }

        //使用串行Stream流 所需时间=227
        //long reduce = LongStream.rangeClosed(0, TIMES).reduce(0, Long::sum);

        //使用串行Stream流 所需时间=110
        //long reduce = LongStream.rangeClosed(0, TIMES).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("所需时间="+(end-start));
```

parallelStream的效率最高

Stream并行处理的过程会分而治之，也就是将一个大任务切分成多个小任务，这表示每个任务都是一个操作。



### (2).parallelStream线程安全问题

```java
List<Integer> list = new ArrayList<>();
IntStream.rangeClosed(0,1000).parallel().forEach(list::add);
System.out.println(list.size());
//949
```

往集合中添加1001个元素，而实际上只有949个元素。出现了线程安全问题

**解决线程安全问题方法一：加锁**

```java
List<Integer> list = new ArrayList<>();
//解决线程安全问题方法一：加锁
IntStream.rangeClosed(0,1000).parallel().forEach(a->{
          synchronized (list){
                list.add(a);
            }
        });
System.out.println(list.size());
//1001
```

**解决线程安全问题方法二：使用线程安全的集合**

```java
List<Integer> list = new Vector<>();
IntStream.rangeClosed(0,1000).parallel().forEach(list::add);
System.out.println(list.size());
//1001

List<Integer> list = new ArrayList<>();
//将集合转换为线程安全的集合
List<Integer> list1 = Collections.synchronizedList(list);
IntStream.rangeClosed(0,1000).parallel().forEach(list1::add);
System.out.println(list.size());//1001
```

**解决线程安全问题方法三：调用Stream的 toArray() / collect()** 

```java
public final Stream<Integer> boxed() {
      return mapToObj(Integer::valueOf);
}
```

```java
List<Integer> list = new ArrayList<>();   IntStream.rangeClosed(0,1000).parallel().boxed()
    //boxed() 将操作对象转换为IntStream
    .collect(Collectors.toList()).forEach(list::add);
System.out.println(list.size());//1001

//简写
List<Integer> list = IntStream.rangeClosed(0,1000).parallel().boxed().collect(Collectors.toList());
System.out.println(list.size());//1001
```



### (3).Fork/Join框架

**parallelStream使用的是Fork/Join框架。**Fork/Join框架自JDK 7引入。Fork/Join框架可以将一个大任务拆分为很多小任务来异步执行。 Fork/Join框架主要包含三个模块：

1. 线程池：ForkJoinPool

2. 任务对象：ForkJoinTask

3. 执行任务的线程：ForkJoinWorkerThread

![image-20220316152412627](img.assets\image-20220316152412627.png)



#### Fork/Join原理-分治法

ForkJoinPool主要用来使用分治法(Divide-and-Conquer Algorithm)来解决问题。典型的应用比如快速排序算法，

ForkJoinPool需要使用相对少的线程来处理大量的任务。比如要对1000万个数据进行排序，那么会将这个任务分割成两个500万的排序任务和一个针对这两组500万数据的合并任务。以此类推，对于500万的数据也会做出同样的分割处理，到最后会设置一个阈值来规定当数据规模到多少时，停止这样的分割处理。比如，当元素的数量小于10时，会停止分割，转而使用插入排序对它们进行排序。那么到最后，所有的任务加起来会有大概2000000+个。问题的关键在于，对于一个任务而言，只有当它所有的子任务完成之后，它才能够被执行。

![image-20220316152634089](img.assets\image-20220316152634089.png)



#### Fork/Join原理-工作窃取算法

Fork/Join最核心的地方就是利用了现代硬件设备多核，在一个操作时候会有空闲的cpu，那么如何利用好这个空闲的cpu就成了提高性能的关键，而这里我们要提到的工作窃取（work-stealing）算法就是整个Fork/Join框架的核心理念Fork/Join工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。

![image-20220316152808644](img.assets\image-20220316152808644.png)

工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。



对于ForkJoinPool通用线程池的线程数量，通常使用默认值就可以了，即运行时计算机的处理器数量。可以通过设置系统属性：java.util.concurrent.ForkJoinPool.common.parallelism=N （N为线程数量），来调整ForkJoinPool的线程数量，可以尝试调整成不同的参数来观察每次的输出结果。



### (4).Fork/Join案例

需求：使用Fork/Join计算1-10000的和，当一个任务的计算数量大于3000时拆分任务，数量小于3000时计算。

![image-20220316153143679](img.assets\image-20220316153143679.png)

```java
public class Demo13 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        //将任务放入线程池中
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //创建一个任务,计算1——10000
        SumRecursiveTask sumRecursiveTask = new SumRecursiveTask(1,100000);
        //将任务提交到线程池中
        Long invoke = forkJoinPool.invoke(sumRecursiveTask);
        long end = System.currentTimeMillis();
        System.out.println("任务执行时间:"+(end-start) +",任务执行结果:"+invoke);
    }
}
//创建一个求和的任务继承RecursiveTask<>类
//RecursiveTask:一个任务
class SumRecursiveTask extends RecursiveTask<Long> {
    //设置临界点
    private static final long THRESHOLD =3000L;
    //起始值
    private final long start;
    //结束值
    private final long end;
    public SumRecursiveTask(long start, long end) {
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute() {
        long length = end - start;
        //如果小于临界值，则计算
        if(length < THRESHOLD){
            //计算
            long sum = 0;
            for (long i = start ; i<end;i++){
                sum += i;
            }
            return sum;
        }else {
            //拆分
            long middle = (end+start) / 2;
            //第一个任务是从起始值到中间值
            SumRecursiveTask sumRecursiveTaskLeft = new SumRecursiveTask(start,middle);
            //fork()表示开启任务
            sumRecursiveTaskLeft.fork();
            //开启右边的任务
            SumRecursiveTask sumRecursiveTaskRight = new SumRecursiveTask(middle+1,end);
            sumRecursiveTaskRight.fork();
            //合并任务的结果
            return sumRecursiveTaskLeft.join() + sumRecursiveTaskRight.join();
        }
    }
}
```



### (5).小结

1. parallelStream是线程不安全的

2. parallelStream适用的场景是CPU密集型的，只是做到别浪费CPU，假如本身电脑CPU的负载很大，那还到处用并行流，那并不能起到作用

3. I/O密集型 磁盘I/O、网络I/O都属于I/O操作，这部分操作是较少消耗CPU资源，一般并行流中不适用于I/O密集型的操作，就比如使用并流行进行大批量的消息推送，涉及到了大量I/O，使用并行流反而慢了很多

4. 在使用并行流的时候是无法保证元素的顺序的，也就是即使你用了同步集合也只能保证元素都正确但无法保证

   其中的顺序



# 六.Optional类

Optional是一个没有子类的工具类，Optional是一个可以为null的容器对象。它的作用主要就是为了解决避免Null检查，防止NullPointerException。

Optional是一个可以为null的容器对象。orElse，ifPresent，ifPresentOrElse，map等方法避免对null的判断，写出更加优雅的代码。



## 1.Optional类的创建方式

```java
Optional.of(T t) : 创建一个 Optional 实例 
Optional.empty() : 创建一个空的 Optional 实例 
Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
```

```java
//创建optional实例
//of只能传入具体的值，不能传入null
Optional<String> optional = Optional.of("optional");
//ofNullable可以传递null，也可以传入具体的值
Optional<Object> o = Optional.ofNullable(null);
//empty:存入的为null
Optional<Object> empty = Optional.empty();
```



## 2.Optional类的常用方法

```java
isPresent() : //判断是否包含值,包含值返回true，不包含值返回false 
ifPresent(Consumer<? super T> consumer)://如果有值，则调用lambda表达式,无值则不调用
get() : //如果Optional有值则将其返回，否则抛出NoSuchElementException 
orElse(T t) : //如果调用对象包含值，返回该值，否则返回参数t 
orElseGet(Supplier s) ://如果调用对象包含值，返回该值，否则返回 s 获取的值
map(Function f): //如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()
```

isPresent() ,ifPresent(),get()

```java
//判断是否有值,有就返回true
//没有就返回false
System.out.println(empty.isPresent());
System.out.println(optional.isPresent());
//获取值,如果有值则返回具体的值，如果没有值，则抛出异常
System.out.println(optional.get());
//联合使用
if(optional.isPresent()){
    System.out.println(optional.get());
}else {
    System.out.println("没有值");
}

optional.ifPresent(a-> System.out.println("有值"+a));
//有值optional
empty.ifPresent(a-> System.out.println("无值"));
//无值不调用lambda表达式
```

orElse(T t   )如果调用对象包含值，返回该值，否则返回参数t 

```java
optional.orElse("若有值，则返回对象中的值");
//optional
System.out.println(empty.orElse("若无值，则使用参数的值"));
//若无值，则使用参数的值
```

orElseGet(Supplier s) 如果调用对象包含值，返回该值，否则返回 s 获取的值

```java
System.out.println(empty.orElseGet(() -> "如果调用对象包含值，返回该值，否则返回 s 获取的值"));
// 如果调用对象包含值，返回该值，否则返回 s 获取的值
```

map(Function f):  如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }}
```

```java
public static void main(String[] args) {
        User user = new User("zs",20);
        //定义一个方法将名字转换为大写
        Optional<User> user1 = Optional.of(user);
        String upperName = getUpperUserName(user1);
        System.out.println(upperName);
    }
private static String getUpperUserName(Optional<User> optional) {
    //return optional.map(a ->optional.get().getUserName().toUpperCase()).orElse(null);
    return optional.map(User::getUserName).map(String::toUpperCase).orElse(null);
} 
```



# 七.**新的日期和时间** API

**旧版日期时间** **API** **存在的问题**

1. 设计很差： 在java.util和java.sql的包中都有日期类，java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期。此外用于格式化和解析的类在java.text包中定义。

2. 非线程安全：java.util.Date ,java.util.SimpleDateFormat是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。

3. 时区处理麻烦：日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和

   java.util.TimeZone类，但他们同样存在上述所有的问题。



**新日期时间 API**

JDK 8中增加了一套全新的日期时间API,新的日期及时间API位于 java.time 包中,是线程安全的

下面是一些关键类:

```java
LocalDate ：//表示日期，包含年月日，格式为 2019-10-16

LocalTime ：//表示时间，包含时分秒，格式为 16:38:54.158549300

LocalDateTime ：//表示日期时间，包含年月日，时分秒，格式为 2018-09-06T15:33:56.750

DateTimeFormatter ：//日期时间格式化类。

Instant：//时间戳，表示一个特定的时间瞬间。

Duration：//用于计算2个时间(LocalTime，时分秒)的距离

Period：//用于计算2个日期(LocalDate，年月日)的距离

ZonedDateTime ：//包含时区的时间
```

Java中使用的历法是ISO 8601日历系统，它是世界民用历法，也就是我们所说的公历。平年有365天，闰年是366

天。此外Java 8还提供了4套其他历法，分别是：

ThaiBuddhistDate：泰国佛教历

MinguoDate：中华民国历

JapaneseDate：日本历

HijrahDate：伊斯兰历



## 1.JDK 8的日期和时间类

LocalDate、LocalTime、LocalDateTime类的实例是不可变的对象，分别表示使用 ISO-8601 日历系统的日期、时

间、日期和时间。它们提供了简单的日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。

```java
//自定义时间
LocalDate localDate1 = LocalDate.of(2020,3,16);
System.out.println(localDate1);
//2020-03-16
LocalTime localTime1 = LocalTime.of(8,59,59,999);
System.out.println(localTime1);
//08:59:59.000000999
```

**静态方法now()获取当前时间**

```java
//获取当前的年月日
LocalDate localDate = LocalDate.now();
System.out.println(localDate);
//2022-03-16
//获取年，月，日
System.out.println
    (localDate.getYear()+"."+localDate.getMonthValue()+"."+localDate.getDayOfMonth());

//获取当前的时分秒，纳秒
LocalTime localTime = LocalTime.now();
System.out.println(localTime);
//19:24:12.679
System.out.println
       (localTime.getHour()+"."+localTime.getMinute()+"."+localTime.getSecond());
//19.24.12

//获取当前的年月日，时分秒
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime);
//2022-03-16T19:14:05.990;
```



对日期时间的修改，对已存在的LocalDate对象，创建它的修改版，最简单的方式是**使用withXXX方法**。

withAttribute方法会创建对象的一个副本，并按照需要修改它的属性。以下所有的方法都返回了一个修改属性的对象，他们不会影响原来的对象。

```java
//修改当前时间
LocalDate now = LocalDate.now();
//修改时间不会改变原来的时间，而是生成一个新的LocalDate
LocalDate localDate2 = now.withYear(2018).withMonth(10).withDayOfMonth(1);
System.out.println("原来的时间:"+now);
System.out.println("修改后的时间:"+localDate2);
//原来的时间:2022-03-16
//修改后的时间:2018-10-01
LocalDateTime localDateTime1 = localDateTime.
										withYear(2020).withMonth(8).withDayOfMonth(1);
System.out.println(localDateTime1);
//2020-08-01T19:18:21.072
```



**plusXXX()方法**，在当前对象的基础上加上指定的时间。

**minusXXX()方法**,在当前对象的基础减少上指定的时间。

```java
LocalDate now = LocalDate.now();
//增加时间
//在当前时间的基础上增加一年
LocalDate localDate = now.plusYears(1);
System.out.println("增加后的时间:"+localDate);
//增加后的时间:2023-03-16

//在增加后的时间的基础上减少一年
LocalDate localDate1 = localDate.minusYears(1);
System.out.println("减少后的时间:"+localDate1);
//减少后的时间:2022-03-16
```



日期时间的比较

在JDK8中，LocalDate类中使用**isBefore()、isAfter()、isEqual()、equals()**方法来比较两个日期，可直接进行比较。

```java
LocalDate of = LocalDate.of(2020, 12, 9);
LocalDate now = LocalDate.now();
//比较当前时间
System.out.println(now.isBefore(of));//false
System.out.println(now.isAfter(of));//true
System.out.println(now.isEqual(of));//false
```



## 2.时间格式化与解析

通过 **java.time.format.DateTimeFormatter** 类可以进行日期时间解析与格式化。

```java
LocalDateTime localDateTime = LocalDateTime.now();
//解析当前时间
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss SSS");
//将日期时间格式转换为字符串
String format = dtf.format(localDateTime);
System.out.println(format);
//2022-03-16 19:53:29 228

//将字符串解析为时间
//字符串中的分隔符要和解析中的分隔符保持一致，否则会报错
LocalDateTime parse = LocalDateTime.parse("2022-03-16 19:53:29 228", dtf);
System.out.println(parse);
//2022-03-16T19:53:29.228

DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
//字符串中的时间的位数也要和解析字符串中的位数保持一致
LocalDate parse1 = LocalDate.parse("2020-01-01", dateTimeFormatter);
System.out.println(parse1);
//2020-01-01
```



## 3.Instant类

Instant 时间戳/时间线，内部保存了从1970年1月1日 00:00:00以来的秒和纳秒。

相关方法和上述日期和时间类类似

```java
//instant内部保存了秒和纳秒，方便程序员对程序做一些统计
Instant instant = Instant.now();
System.out.println("当前的时间戳:"+instant);
//在当前时间上加上20秒
Instant instant1 = instant.plusMillis(20);
System.out.println(instant1);

// 获取从1970年1月1日 00:00:00的秒
System.out.println("获取纳秒:"+instant.getNano());
System.out.println("获取秒:"+instant.getEpochSecond());
//1647432942 省略了System.currentTimeMillis()的后三位
System.out.println(System.currentTimeMillis());
//1647432942130
```



## 4.计算日期时间差类

Duration/Period类: 计算日期时间差。

1. Duration：用于计算2个时间(LocalTime，时分秒)的距离

2. Period：用于计算2个日期(LocalDate，年月日)的距离

   

   Duration.between()  后面的时间-前面的时间

```java
//计算时间的距离
LocalTime localTime = LocalTime.now();
LocalTime of = LocalTime.of(18, 22, 16);
//localTime - of 后减前
Duration duration = Duration.between(of, localTime);
System.out.println("相差的天数:" + duration.toDays());
System.out.println("相差的小时:" + duration.toHours());
System.out.println("相差的分钟:" + duration.toMinutes());
System.out.println("相差的秒数:" + duration.toMillis());
System.out.println("相差的纳秒数:" + duration.toNanos());
```

Period.between()

```java
//计算日期的距离
LocalDate localDate = LocalDate.now();
LocalDate localDate1 = LocalDate.of(2020, 9, 13);
Period period = Period.between(localDate1, localDate);
System.out.println("相差的年数:"+period.getYears());
System.out.println("相差的月数:"+period.getMonths());
System.out.println("相差的日数:"+period.getDays());
```



## 5.时间校正器

有时我们可能需要获取例如：将日期调整到“下一个月的第一天”等操作。可以通过时间校正器来进行。

TemporalAdjuster : 时间校正器。

TemporalAdjusters : 该类通过静态方法提供了大量的常用TemporalAdjuster的实现。

```java
@FunctionalInterface
public interface TemporalAdjuster {
	Temporal adjustInto(Temporal temporal);
}
```

```java
LocalDateTime localDateTime = LocalDateTime.now();
//日期调整到“下一个月的第一天”
//设置时间调整器
TemporalAdjuster temporalAdjuster = (temporal) -> {
			LocalDateTime localDateTime1 = (LocalDateTime) temporal;
			//调整到“下一个月的第一天”
            return localDateTime1.plusMonths(1).withDayOfMonth(1);
        };
//将时间调整器的实例传入with方法中
LocalDateTime temporalTime = localDateTime.with(temporalAdjuster);
System.out.println(temporalTime);
//2022-04-01T20:40:00.604
```

TemporalAdjusters中的常用调整器：

|   firstDayOfMonth()   | 返回"每月的第一天"调整器，该调整器返回设置为当前月份第一天的新日期。 |
| :-------------------: | ------------------------------------------------------------ |
| firstDayOfNextMonth() | 返回"下个月的第一天"调整器，该调整器返回设置为下个月第一天的新日期。 |
| firstDayOfNextYear()  | 返回"明年的第一天"调整器，该调整器返回设置为下一年第一天的新日期。 |
|   firstDayOfYear()    | 返回"一年中的第一天"调整器，该调整器返回设置为当前年份第一天的新日期。 |
|   lastDayOfMonth()    | 返回"每月最后一天"调整器，该调整器返回设置为当前月份最后一天的新日期。 |
|    lastDayOfYear()    | 返回"一年中的最后一天"调整器，该调整器返回设置为当前年份最后一天的新日期。 |

```java
//设置为今年第一天
TemporalAdjuster temporalAdjuster1 = TemporalAdjusters.firstDayOfYear();
LocalDateTime with = localDateTime.with(temporalAdjuster1);
System.out.println(with);
//2022-01-01T20:54:10.837
```



## 6.设置日期时间的时区

Java8 中加入了对时区的支持，LocalDate、LocalTime、LocalDateTime是不带时区的，带时区的日期时间类分别

为：ZonedDate、ZonedTime、ZonedDateTime。

其中每个时区都对应着 ID，ID的格式为 “区域/城市” 。例如 ：Asia/Shanghai 等。

ZoneId：该类中包含了所有的时区信息

```java
//获取所有的时区ID
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
availableZoneIds.forEach(System.out::println);

//不带时区的类 中国在东八区
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime);
//2022-03-16T21:07:23.451

//操作带时区的类
//Clock.systemUTC() 创建世界标准时间
ZonedDateTime zonedDateTime = ZonedDateTime.now(Clock.systemUTC());
//中国的东八区比时间标准时间早8个小时
System.out.println(zonedDateTime);
//2022-03-16T13:07:23.451Z

//不加参数将带上计算机时间默认时区
ZonedDateTime zonedDateTime1 = ZonedDateTime.now();
System.out.println(zonedDateTime1);
//2022-03-16T21:07:23.451+08:00[Asia/Shanghai]

//使用指定的时区创造时间
ZonedDateTime zonedDateTime2 = ZonedDateTime.now(ZoneId.of("America/Cuiaba"));
System.out.println(zonedDateTime2);
//2022-03-16T09:09:33.509-04:00[America/Cuiaba]

//修改时区  withZoneSameInstant既修改时区，也修改时间
ZonedDateTime zonedDateTime3 = zonedDateTime2
    							.withZoneSameInstant(ZoneId.of("Asia/Shanghai"));
System.out.println(zonedDateTime3);
//2022-03-16T21:10:59.874+08:00[Asia/Shanghai]

//withZoneSameLocal 只该时区，不该时间
ZonedDateTime zonedDateTime4 = zonedDateTime2
                               .withZoneSameLocal(ZoneId.of("Asia/Shanghai"));
System.out.println(zonedDateTime4);
//2022-03-16T09:12:42.525+08:00[Asia/Shanghai]
```



## 7.小结

1.localDate表示日期,包含年月日,LocalTime表示时间,包含时分秒,LocalDateTime = LocalDate + LocalTime,时间的格式化和解析,通过DateTimeFormatter类型进行。

2.nstant类,方便操作秒和纳秒,一般是给程序使用的。

3.Duration/Period计算时间或日期的距离。

4.时间调整器（TemporalAdjuster）方便的调整时间。

5.带时区的3个类ZoneDate/ZoneTime/ZoneDateTime



**JDK 8新的日期和时间 API的优势：**

1. 新版的日期和时间API中，日期和时间对象是不可变的。操纵的日期不会影响老值，而是新生成一个实例。

2. 新的API提供了两种不同的时间表示方式，有效地区分了人和机器的不同需求。

3. TemporalAdjuster可以更精确的操纵日期，还可以自定义日期调整器。

4. 是线程安全的



# 八.重复注解与类型注解

## 1.重复注解

JDK 8引入了重复注解的概念，允许在同一个地方多次使用同一个注解。在JDK 8中使用**@Repeatable**注解定义重复注解。

1. 定义重复的注解容器注解

```java
@Retention(RetentionPolicy.RUNTIME) 
@interface MyTests { MyTest[] value(); }
```

2. 定义一个可以重复的注解

```java
@Retention(RetentionPolicy.RUNTIME) 
@Repeatable(MyTests.class) 
@interface MyTest { String value(); }
```

3. 配置多个重复的注解

```java
@MyTest("tbc") 
@MyTest("tba") 
@MyTest("tba") 
public class Demo01 { 
@MyTest("mbc") 
@MyTest("mba") 
public void test() throws NoSuchMethodException { } 
	}
```

## 2.类型注解

JDK 8为@Target元注解新增了两种类型： TYPE_PARAMETER ， TYPE_USE 。 

TYPE_PARAMETER ：表示该注解能写在类型参数的声明语句中。 类型参数声明如： <T> 、 

TYPE_USE ：表示注解可以再任何用到类型的地方使用。



TYPE_PARAMETER:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE_PARAMETER})
//ElementType.TYPE_PARAMETER表示该方法可以作用于泛型
public @interface Demo1 {
}
```

```java
public class Demo2 <@Demo1 T>{
    //注解作用在泛型上
    public <@Demo1 E> void test(){}
    public <@Demo1 R extends Object> void test1(){}
}
```

TYPE_USE :

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
//TYPE_USE可以作用在任何使用在类型的地方
public @interface Demo3 {}
```

```java
//注解使用在类型使用的地方
private @Demo3 int a = 10;
public void test2(@Demo3 String name){
     @Demo3 double s = 10.0;
}
```

通过@Repeatable元注解可以定义可重复注解， TYPE_PARAMETER 可以让注解放在泛型上， TYPE_USE 可以让注解放在类型的前面
