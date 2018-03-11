上一篇文章中，我们简单介绍了Java8的Lambda表达式以及函数式接口的概念，接下来我们继续深入Java8函数式编程模型。



public class Test1 {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1,2,3,4,5,6,7,8,9,10\);

        list.forEach\(new Consumer&lt;Integer&gt;\(\) {

            @Override

            public void accept\(Integer integer\) {

                System.out.println\(integer\);

            }

        }\);

    }

}

这段程序很简单，首先初始化一个Integer类型的集合然后向控制台输出每个元素。其中我们注意到forEach方法，它就是Java8中新增加的默认方法。



public interface Iterable&lt;T&gt; {

    .

    .省略

    .

    default void forEach\(Consumer&lt;? super T&gt; action\) {

        Objects.requireNonNull\(action\);

        for \(T t : this\) {

            action.accept\(t\);

        }

    }

}

它被声明在Iterable接口中，并被关键字default修饰。这样任何一个该接口的子类型都可以继承forEach方法的实现，所以List接口因为是Iterable的间接子接口，所以也继承了该默认方法。Java8采用这种巧妙的方式既扩展了接口的功能，又兼容了老版本。



接下来分析下forEach的实现，首先接收了一个Consumer类型的参数action，进行非空判断，然后遍历当前所有元素交由action的accept方法进行处理。那么Consumer又是什么鬼，看源码



@FunctionalInterface

public interface Consumer&lt;T&gt; {



    /\*\*

     \* Performs this operation on the given argument.

     \*

     \* @param t the input argument

     \*/

    void accept\(T t\);

    .

    .省略

    .

}

一个接口，有且仅有一个抽象方法，被@FunctionalInterface修饰，典型的函数式接口。

ok，现在我们知道forEach接收的Consumer类型的参数是一个函数式接口，接口里唯一的accept抽象方法接收一个参数，不返回值。那通过上一篇文章我们知道，创建函数式接口类型的实例其中一种方式是使用Lambda表达式，所以可以将最上面的程序改造一下



public class Test1 {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1,2,3,4,5,6,7,8,9,10\);

        //Lambda表达式 接收一个参数 不返回值

        list.forEach\(item -&gt; System.out.println\(item\)\);

    }

}

该lambda表达式item -&gt; System.out.println\(item\)接收一个参数 不返回值，符合accept方法定义，编译通过。

也就是说如果使用lambda表达式来创建一个函数式接口实例，那这个lambda表达式的入参和返回必须符合这个函数式接口中唯一的抽象方法的定义。



接下来再对程序进行改造



public class Test1 {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1,2,3,4,5,6,7,8,9,10\);

        //方法引用

        list.forEach\(System.out::println\);

    }

}

看到out后面有两个冒号，反正当时我是凌乱了。。。这个就是函数式接口实例第二种创建方式：方法引用

方法引用的语法是 对象::方法名（只是其中一种）

同样，使用方法引用方式去创建函数式接口实例也必须遵守方法的定义，看下此处println方法源码



public void println\(Object x\) {

    String s = String.valueOf\(x\);

    synchronized \(this\) {

        print\(s\);

        newLine\(\);

    }

}

接收一个参数，并不返回值，编译通过。

最后我们来看下创建函数式接口的最后一种，第三种方式：构造方法引用 ，继续改程序



public class Test1 {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1,2,3,4,5,6,7,8,9,10\);

        //构造方法引用

        list.forEach\(Test1::new\);

    }

    

    Test1\(Integer i\){

        System.out.println\(i\);

    }

}

构造方法引用的语法是：类名::new

我们给Test1新添加了一个构造方法，该构造方法接收一个参数，不返回值，编译通过。（仅为展示构造方法引用的用法）



结合上一篇文章可以总结一下，创建函数式接口类型的三种方式：

1.lambda表达式

2.方法引用

3.构造方法引用

注意：无论是哪种方式，必须要符合抽象方法的方法定义

