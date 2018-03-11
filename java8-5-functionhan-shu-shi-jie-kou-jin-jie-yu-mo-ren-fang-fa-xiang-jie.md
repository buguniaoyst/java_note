Java8-5-函数式接口进阶与默认方法详解

上一篇我们快速的借助示例演示了stream api的简单应用，体会到了使用stream api对集合处理的便捷和其与函数式接口密不可分的关系，所以为了更高效的使用stream api，有必要更熟练的掌握函数式接口。Java8中内置了大量的函数式接口，接下来我们选择一些比较常用的一起学习下。



Function接口

在之前的文章中，我们简单介绍过Function接口中apply方法的应用，除了apply这个抽象方法，Function接口中还内置了两个比较常用的默认方法（接口中增加的有具体实现的方法，扩展了接口功能，子类默认会继承该实现），看下Function接口源码



/\*\*

 \* Represents a function that accepts one argument and produces a result.

 \*

 \* &lt;p&gt;This is a &lt;a href="package-summary.html"&gt;functional interface&lt;/a&gt;

 \* whose functional method is {@link \#apply\(Object\)}.

 \*

 \* @param &lt;T&gt; the type of the input to the function

 \* @param &lt;R&gt; the type of the result of the function

 \*

 \* @since 1.8

 \*/

@FunctionalInterface

public interface Function&lt;T, R&gt; {



    R apply\(T t\);



    /\*\*

     \* @return a composed function that first applies the {@code before}

     \* function and then applies this function

     \*/

    default &lt;V&gt; Function&lt;V, R&gt; compose\(Function&lt;? super V, ? extends T&gt; before\) {

        Objects.requireNonNull\(before\);

        return \(V v\) -&gt; apply\(before.apply\(v\)\);

    }

    /\*\*

     \* @return a composed function that first applies this function and then

     \* applies the {@code after} function

     \*/

    default &lt;V&gt; Function&lt;T, V&gt; andThen\(Function&lt;? super R, ? extends V&gt; after\) {

        Objects.requireNonNull\(after\);

        return \(T t\) -&gt; after.apply\(apply\(t\)\);

    }

    /\*\*

     \* 省略

     \*/

}

Function接口定义中有两个泛型，按着接口文档说明第一个泛型是输入类型，第二泛型是结果类型。

compose方法接收一个Function参数before，该方法说明是返回一个组合的函数，首先会应用before，然后应用当前对象，换句话说就是先执行before对象的apply，再执行当前对象的apply，将两个执行逻辑串起来。

andThen方法接收一个Function参数after，与compose方法相反，它是先执行当前对象的apply方法，再执行after对象的方法。看一个应用示例



public class FunctionTest {

    public static void main\(String\[\] args\) {

        FunctionTest functionTest = new FunctionTest\(\);

        System.out.println\(functionTest.compute1\(5,i -&gt; i \* 2,i -&gt; i \* i\)\);//50

        System.out.println\(functionTest.compute2\(5,i -&gt; i \* 2,i -&gt; i \* i\)\);//100

    }



    public int compute1\(int i, Function&lt;Integer,Integer&gt; after,Function&lt;Integer,Integer&gt; before\){

        return after.compose\(before\).apply\(i\);

    }



    public int compute2\(int i, Function&lt;Integer,Integer&gt; before,Function&lt;Integer,Integer&gt; after\){

        return before.andThen\(after\).apply\(i\);

    }

}

定义了compute1和compute2两个方法，compute1方法第一个参数是要计算的数据，第二个参数是后执行的函数，第一个是先执行的函数，因为输入输出都是数字类型，所以泛型都指定为Integer类型，通过after.compose\(before\);将两个函数串联起来然后执行组合后的Funtion方法apply\(i\)。当调用compute1\(5,i -&gt; i 2,i -&gt; i i\)时，先平方再乘以2所以结果是50。而compute2方法对两个Function的调用正好相反，所以结果是100。



BiFunction接口

接下来继续看下另一个很常用的函数式接口BiFunction





/\*\*

 \* This is the two-arity specialization of {@link Function}.

 \* @param &lt;T&gt; the type of the first argument to the function

 \* @param &lt;U&gt; the type of the second argument to the function

 \* @param &lt;R&gt; the type of the result of the function

 \*

 \* @see Function

 \* @since 1.8

 \*/

@FunctionalInterface

public interface BiFunction&lt;T, U, R&gt; {



    /\*\*

     \* Applies this function to the given arguments.

     \*

     \* @param t the first function argument

     \* @param u the second function argument

     \* @return the function result

     \*/

    R apply\(T t, U u\);



    /\*\*

     \* Returns a composed function that first applies this function to

     \* its input, and then applies the {@code after} function to the result.

     \* If evaluation of either function throws an exception, it is relayed to

     \* the caller of the composed function.

     \*

     \* @param &lt;V&gt; the type of output of the {@code after} function, and of the

     \*           composed function

     \* @param after the function to apply after this function is applied

     \* @return a composed function that first applies this function and then

     \* applies the {@code after} function

     \* @throws NullPointerException if after is null

     \*/

    default &lt;V&gt; BiFunction&lt;T, U, V&gt; andThen\(Function&lt;? super R, ? extends V&gt; after\) {

        Objects.requireNonNull\(after\);

        return \(T t, U u\) -&gt; after.apply\(apply\(t, u\)\);

    }

}

实际上就是可以有两个参数的Function，同样前两个泛型代表着入参的类型，第三个代表结果类型。



public class BiFunctionTest {

    public static void main\(String\[\] args\) {

        BiFunctionTest2 biFunctionTest2 = new BiFunctionTest2\(\);

        System.out.println\(biFunctionTest2.compute\(4,5,\(a,b\) -&gt; a \* b,a -&gt; a \* 2\)\);

    }



    public int compute\(int a, int b, BiFunction&lt;Integer,Integer,Integer&gt; biFunction,

                       Function&lt;Integer,Integer&gt; function\){

        return biFunction.andThen\(function\).apply\(a,b\);

    }

}

看下compute方法，前两个参数是待计算数据，第三个是一个BiFunction，因为入参和结果都是数组所以三个泛型都定义为Integer。最后一个参数是Function。计算逻辑是先执行BiFunction然后将结果传给Funciton在计算最后返回结果，所以使用了andThen方法。我们想一下，BiFunction的andThen方法为什么接收的是Function类型的参数而不是BiFunction，答案很简单，因为BiFunction的apply方法接收两个参数，但是任何一个方法不可能有两个返回值，所以也没办法放在BiFunction前面执行，这也是为什么BiFunction没有compose方法的原因。

