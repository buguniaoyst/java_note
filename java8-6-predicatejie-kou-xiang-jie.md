上一篇我们学习了Function和BiFunction函数式接口，本篇继续了解下其他常用的函数式接口。

先来看下Predicate

Predicate函数式接口的主要作用就是提供一个test方法，接受一个参数返回一个布尔类型，Predicate在stream api中进行一些判断的时候非常常用。



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

}

使用泛型T指定传入的参数类型，我们通过一个根据不同条件取出不同数据的例子来看下Predicate具体应用



public class PredicateTest {

    public static void main\(String\[\] args\) {

        List&lt;Integer&gt; list = Arrays.asList\(1, 2, 3, 4, 5, 6, 7, 8, 9, 10\);

        PredicateTest predicateTest = new PredicateTest\(\);

        //输出大于5的数字

        List&lt;Integer&gt; result = predicateTest.conditionFilter\(list, integer -&gt; integer &gt; 5\);

        result.forEach\(System.out::println\);

        System.out.println\("-------"\);

        //输出大于等于5的数字

        result = predicateTest.conditionFilter\(list, integer -&gt; integer &gt;= 5\);

        result.forEach\(System.out::println\);

        System.out.println\("-------"\);

        //输出小于8的数字

        result = predicateTest.conditionFilter\(list, integer -&gt; integer &lt; 8\);

        result.forEach\(System.out::println\);

        System.out.println\("-------"\);

        //输出所有数字

        result = predicateTest.conditionFilter\(list, integer -&gt; true\);

        result.forEach\(System.out::println\);

        System.out.println\("-------"\);

    }

    //高度抽象的方法定义，复用性高

    public List&lt;Integer&gt; conditionFilter\(List&lt;Integer&gt; list, Predicate&lt;Integer&gt; predicate\){

        return list.stream\(\).filter\(predicate\).collect\(Collectors.toList\(\)\);

    }

}

我们只定义了一个conditionFilter方法，stream\(\)会将当前list作为源创建一个Stream对象，collect\(Collectors.toList\(\)\)是将最终的结果封装在ArrayList中（这部分会在后续stream学习中详细介绍，这里只关注filter即可），filter方法接收一个Predicate类型参数用于对目标集合进行过滤。里面并没有任何具体的逻辑，提供了一种更高层次的抽象化，我们可以把要处理的数据和具体的逻辑通过参数传递给conditionFilter即可。理解了这种设计思想后，再看上面的例子就很容易理解，本身逻辑并不复杂，分别取出小于5、大于等于5、小于8的元素，最后一个总是返回true的条件意味着打印出集合中所有元素。

除此之外，Predicate还提供了另外三个默认方法和一个静态方法



default Predicate&lt;T&gt; and\(Predicate&lt;? super T&gt; other\) {

    Objects.requireNonNull\(other\);

    return \(t\) -&gt; test\(t\) && other.test\(t\);

}



default Predicate&lt;T&gt; or\(Predicate&lt;? super T&gt; other\) {

    Objects.requireNonNull\(other\);

    return \(t\) -&gt; test\(t\) \|\| other.test\(t\);

}



default Predicate&lt;T&gt; negate\(\) {

    return \(t\) -&gt; !test\(t\);

}



static &lt;T&gt; Predicate&lt;T&gt; isEqual\(Object targetRef\) {

    return \(null == targetRef\)

            ? Objects::isNull

            : object -&gt; targetRef.equals\(object\);

}

and方法接收一个Predicate类型，也就是将传入的条件和当前条件以并且的关系过滤数据。or方法同样接收一个Predicate类型，将传入的条件和当前的条件以或者的关系过滤数据。negate就是将当前条件取反。看下具体使用方式



public List&lt;Integer&gt; conditionFilterNegate\(List&lt;Integer&gt; list, Predicate&lt;Integer&gt; predicate\){

    return list.stream\(\).filter\(predicate.negate\(\)\).collect\(Collectors.toList\(\)\);

}



public List&lt;Integer&gt; conditionFilterAnd\(List&lt;Integer&gt; list, Predicate&lt;Integer&gt; predicate,Predicate&lt;Integer&gt; predicate2\){

    return list.stream\(\).filter\(predicate.and\(predicate2\)\).collect\(Collectors.toList\(\)\);

}



public List&lt;Integer&gt; conditionFilterOr\(List&lt;Integer&gt; list, Predicate&lt;Integer&gt; predicate,Predicate&lt;Integer&gt; predicate2\){

    return list.stream\(\).filter\(predicate.or\(predicate2\)\).collect\(Collectors.toList\(\)\);

}



//大于5并且是偶数

result = predicateTest.conditionFilterAnd\(list, integer -&gt; integer &gt; 5, integer1 -&gt; integer1 % 2 == 0\);

result.forEach\(System.out::println\);//6 8 10

System.out.println\("-------"\);



//大于5或者是偶数

result = predicateTest.conditionFilterOr\(list, integer -&gt; integer &gt; 5, integer1 -&gt; integer1 % 2 == 0\);

result.forEach\(System.out::println\);//2 4 6 8 9 10

System.out.println\("-------"\);



//条件取反

result = predicateTest.conditionFilterNegate\(list,integer2 -&gt; integer2 &gt; 5\);

result.forEach\(System.out::println\);// 1 2 3 4 5

System.out.println\("-------"\);

我们分别借助Predicate的三个默认方法定义了conditionFilterAnd、conditionFilterOr和conditionFilterNegate方法。然后再下方调用这三个方法，根据传入的判断条件观察输出结果。



最后再来看一下Predicate接口中的唯一一个静态方法,Java8中接口中除了增加了默认方法也可以定义静态方法。



/\*\*

 \* Returns a predicate that tests if two arguments are equal according

 \* to {@link Objects\#equals\(Object, Object\)}.

 \*

 \* @param &lt;T&gt; the type of arguments to the predicate

 \* @param targetRef the object reference with which to compare for equality,

 \*               which may be {@code null}

 \* @return a predicate that tests if two arguments are equal according

 \* to {@link Objects\#equals\(Object, Object\)}

 \*/

static &lt;T&gt; Predicate&lt;T&gt; isEqual\(Object targetRef\) {

    return \(null == targetRef\)

            ? Objects::isNull

            : object -&gt; targetRef.equals\(object\);

}

isEqual方法返回类型也是Predicate，也就是说通过isEqual方法得到的也是一个用来进行条件判断的函数式接口实例。而返回的这个函数式接口实例是通过传入的targetRef的equals方法进行判断的。我们看一下具体用法



System.out.println\(Predicate.isEqual\("test"\).test\("test"\)\);//true

这里会用第一个"test"的equals方法判断与第二个"test"是否相等，结果true。

