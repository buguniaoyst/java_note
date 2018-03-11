上一篇文章中，我们介绍了几个Java8内置的函数式接口的特点和使用方式，并在最后引出了stream api的知识点，接下来我们开始学习Java8中的stream api。

先假设一个简单的需求，存在一个字符串集合，我们想把所有长度大于5的字符串转换成大写输出到控制台，之前我们可能会直接这么做



public class Test3 {

    public static void main\(String\[\] args\) {

        List&lt;String&gt; list = Arrays.asList\("hello","world","helloworld"\);



        for \(int i = 0; i &lt; list.size\(\); i++\) {

            if\(list.get\(i\).length\(\) &gt; 5\){

                System.out.println\(list.get\(i\).toUpperCase\(\)\);

            }

        }

    }

}

现在换成使用stream api看下



public class Test3 {

    public static void main\(String\[\] args\) {

        List&lt;String&gt; list = Arrays.asList\("hello","world","helloworld"\);

        list.stream\(\).filter\(s -&gt; s.length\(\) &gt; 5\).map\(s -&gt; s.toUpperCase\(\)\).forEach\(System.out::println\);

    }

}

1行代码直接搞定，而且这种链式编程风格从语义上看逻辑很清晰。

stream方法先构造了一个该集合的Stream对象，filter方法取出长度大于5的字符串，map方法将所有字符串转大写，forEach输出到控制台。我们可以对照Stream接口的源码片段看下这几个stream api的定义



public interface Stream&lt;T&gt; extends BaseStream&lt;T, Stream&lt;T&gt;&gt; {

    

    Stream&lt;T&gt; filter\(Predicate&lt;? super T&gt; predicate\);

    

    &lt;R&gt; Stream&lt;R&gt; map\(Function&lt;? super T, ? extends R&gt; mapper\);

    

    void forEach\(Consumer&lt;? super T&gt; action\);

    .

    .省略

    .

}

filter方法，接收一个Predicate函数式接口类型作为参数，并返回一个Stream对象，从上一篇我们知道可以由一个接收一个参数返回布尔类型的lambda表达式来创建Predicate函数式接口实例，所以看到filter接收的参数是s -&gt; s.length\(\) &gt; 5



map方法，接收Function函数式接口类型，接收一个参数，有返回值s -&gt; s.toUpperCase\(\) 正是做了这件事情



forEach方法，接收Consumer函数式接口类型，接收一个参数，不返回值 这里使用方法引用的其中一种形式System.out::println来创建了Consumer实例。



所以通过上面的例子可以看出函数式编程和stream api结合的非常紧密。大家应该也注意到了在介绍每个方法时，我们提到了有中间操作和终止操作，终止操作意味着我们需要一个结果了，当程序遇到终止操作时才会真正执行。中间操作是指在终止操作之前所有的方法，这些方法以方法链的形式组织在一起处理一些列逻辑，如果只有中间操作而没有终止操作的话即使运行程序，代码也不会执行的。



实际上map方法中可以使用另一种方法引用的形式来处理，类方法引用。语法：类名::方法名



public class Test3 {

    public static void main\(String\[\] args\) {

        List&lt;String&gt; list = Arrays.asList\("hello","world","helloworld"\);

        list.stream\(\).filter\(s -&gt; s.length\(\) &gt; 5\).map\(String::toUpperCase\).forEach\(System.out::println\);

    }

}

之前介绍的System.out::println这种属于对象方法引用，类方法引用的应用也是很广泛的。只是理解起来需要花费些时间，map方法接收一个Function函数式接口的实现，那就肯定需要一个输入并且有一个输出，但是我们看下toUpperCase方法的定义



public String toUpperCase\(\) {

    return toUpperCase\(Locale.getDefault\(\)\);

}

有返回值，但是没有入参，乍一看也不符合Function接口中apply方法的定义啊。这也是类方法引用的特点，虽然toUpperCase没有明确的入参，因为此时toUpperCase的输入是调用它的那个对象，编译器会把调用toUpperCase方法的那个对象当做参数，也就是lambda表达式s -&gt; s.toUpperCase\(\)中的s参数。所以也满足一个输入一个输出的定义。



小结：本篇简单介绍了函数式编程与stream api应用及类方法引用的使用，lambda表达式让老版本的代码更简洁，方法引用让lambda表达式更简洁，实际上就是lambda表达式的一种语法糖。

