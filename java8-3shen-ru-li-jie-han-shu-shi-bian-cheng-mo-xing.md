上一篇文章中，我们总体介绍了创建函数式接口实例的几种方式以及Java8中接口新增的默认方法特性，接下来我们来看下Java8中已经为我们提供的几种典型的函数式接口

先看一个示例



public class FunctionTest {

    public static void main\(String\[\] args\) {

        FunctionTest functionTest = new FunctionTest\(\);

        int i2 = functionTest.add2\(2\);

        int i3 = functionTest.add3\(2\);

        int i4 = functionTest.add4\(2\);

    }



    //逻辑提前定义好

    public int add2\(int i\){

        return i + 2;

    }



    //逻辑提前定义好

    public int add3\(int i\){

        return i + 3;

    }



    //逻辑提前定义好

    public int add4\(int i\){

        return i + 4;

    }

}

FunctionTest中定义了三个方法，分别把参数加2，加3，加4然后分别把结果返回，那如果以后还要取到参数的平方或者各种其他的运算，就还需要定义更多的处理方法。接下来看下如果使用Java8提供的Function接口会有哪些改进

首先看下Function接口定义



@FunctionalInterface

public interface Function&lt;T, R&gt; {



    /\*\*

     \* Applies this function to the given argument.

     \*

     \* @param t the function argument

     \* @return the function result

     \*/

    R apply\(T t\);

    .

    .省略

    .

}

该函数式接口唯一的抽象方法apply接收一个参数，有返回值。看下使用方式



public class FunctionTest {

    public static void main\(String\[\] args\) {

        FunctionTest functionTest = new FunctionTest\(\);



        int result2 = functionTest.compute\(5, num -&gt; num + 2\);

        int result3 = functionTest.compute\(5, num -&gt; num + 2\);

        int result4 = functionTest.compute\(5, num -&gt; num + 2\);

        int results = functionTest.compute\(5, num -&gt; num \* num\);



    }



    //调用时传入逻辑

    public int compute\(int i, Function&lt;Integer,Integer&gt; function\){

        Integer result = function.apply\(i\);

        return result;

    }

}

我们在FunctionTest中定义了compute方法，方法的第一个参数是要运算的数据，第二个参数是函数式接口Function的实例，当执行compute方法时，会将第一个参数交给第二个参数Function中的apply方法处理，然后返回结果。

这样我们可以将方法定义的更抽象，代码重用性也就越高，每次将要计算的数据和计算逻辑一起作为参数传递给compute方法就可以。是不是有点体验到函数式编程的灵活之处。

注：因为表达式只有一行语句 num -&gt; num + 2 可以省略了return 关键字 如果为了更加直观可以写成 num -&gt; return num + 2



Supplier接口，默认抽象方法get不接收参数，有返回值



@FunctionalInterface

public interface Supplier&lt;T&gt; {



    /\*\*

     \* Gets a result.

     \*

     \* @return a result

     \*/

    T get\(\);

}

类似工厂模式



public class SupplierTest {

    public static void main\(String\[\] args\) {

        Supplier&lt;String&gt; supplier = String::new;

        String s = supplier.get\(\);

    }

}

这里使用构造方法引用的方式创建Supplier实例，通过get直接返回String对象



Predicate接口



@FunctionalInterface

public interface Predicate&lt;T&gt; {



    /\*\*

     \* Evaluates this predicate on the given argument.

     \*

     \* @param t the input argument

     \* @return {@code true} if the input argument matches the predicate,

     \* otherwise {@code false}

     \*/

    boolean test\(T t\);

    .

    .省略

    .

}

接收一个参数，返回布尔类型，使用方式



public class PredicateTest {

    public static void main\(String\[\] args\) {

        Predicate&lt;String&gt; predicate = s -&gt; s.length\(\) &gt; 5;

        System.out.println\(predicate.test\("hello"\)\);

    }

}

定义了一个接收一个参数返回布尔值的lambda表达式，赋值给predicate，就可以直接对传入参数进行校验



再看一个Predicate例子



public class PredicateTest {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1, 2, 3, 4, 5, 6, 7, 8, 9, 10\);

        PredicateTest predicateTest = new PredicateTest\(\);

        List&lt;Integer&gt; result = predicateTest.conditionFilter\(list, integer -&gt; integer &gt; 5\);

        result.forEach\(System.out::println\);

    }



    public List&lt;Integer&gt; conditionFilter\(List&lt;Integer&gt; list, Predicate&lt;Integer&gt; predicate\){

        return list.stream\(\).filter\(predicate\).collect\(Collectors.toList\(\)\);

    }

}

这段程序的逻辑是找到集合里大于5的数据，打印到控制台。我们具体分析一下conditionFilter方法，第一个参数是待遍历的集合，第二个参数是Predicate类型的实例，还记得Predicate接口中的抽象方法定义吗，接收一个参数返回布尔类型。list.stream\(\)是创建了一个针对此集合的Stream对象（先简单认识一下），然后调用Stream的filter方法对结果进行过滤，过滤的条件就是Predicate接口的实现，最后collect\(Collectors.toList\(\)\);是将过滤后的结果收集起来放置到一个新的集合中并返回。

其中涉及到Srteam api的知识点我们会在后续章节中详细介绍，现在只关心Predicate的使用就可以。



接下来调用conditionFilter方法



List&lt;Integer&gt; result = predicateTest.conditionFilter\(list, integer -&gt; integer &gt; 5\);

list是待遍历的集合

integer -&gt; integer &gt; 5 lambda表达式是对Predicate接口的具体实现



result.forEach\(System.out::println\);

最后，方法引用实例化一个Consumer对象，把结果输出到控制台。



可以看出，lambda表达式和stream api结合起来使用让代码更简洁更加语义化，即使之前没接触过stream api也能大概看出conditionFilter方法的功能。

