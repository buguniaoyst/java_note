上一篇系统学了方法引用的几种类型及应用场景，本篇开始我们正式学习Stream。

Java8中的Stream与lambda表达式可以说是相伴相生的，通过Stream我们可以更好的更为流畅更为语义化的操作集合。Stream api都位于java.util.stream包中。其中就包含了最核心的Stream接口，一个Stream实例可以串行或者并行操作一组元素序列，官方文档中给出了一个示例



 \* &lt;pre&gt;{@code

 \*     int sum = widgets.stream\(\)//创建一个流

 \*                      .filter\(w -&gt; w.getColor\(\) == RED\)//取出颜色是红色的元素

 \*                      .mapToInt\(w -&gt; w.getWeight\(\)\)//返回每个红色元素的重量

 \*                      .sum\(\);//重量求和

 \* }&lt;/pre&gt;

Java8中，所有的流操作会被组合到一个 stream pipeline中，这点类似linux中的pipeline概念，将多个简单操作连接在一起组成一个功能强大的操作。一个 stream pileline首先会有一个数据源，这个数据源可能是数组、集合、生成器函数或是IO通道，流操作过程中并不会修改源中的数据；然后还有零个或多个中间操作，每个中间操作会将接收到的流转换成另一个流（比如filter）；最后还有一个终止操作，会生成一个最终结果（比如sum）。流是一种惰性操作，所有对源数据的计算只在终止操作被初始化的时候才会执行。



总结一下流操作由3部分组成

1.源

2.零个或多个中间操作

3.终止操作 （到这一步才会执行整个stream pipeline计算）



创建流的几种方式



//第一种 通过Stream接口的of静态方法创建一个流

Stream&lt;String&gt; stream = Stream.of\("hello", "world", "helloworld"\);

//第二种 通过Arrays类的stream方法，实际上第一种of方法底层也是调用的Arrays.stream\(values\);

String\[\] array = new String\[\]{"hello","world","helloworld"};

Stream&lt;String&gt; stream3 = Arrays.stream\(array\);

//第三种 通过集合的stream方法，该方法是Collection接口的默认方法，所有集合都继承了该方法

Stream&lt;String&gt; stream2 = Arrays.asList\("hello","world","helloworld"\).stream\(\);

接下来我们看一个简单的需求：将流中字符全部转成大写返回一个新的集合



List&lt;String&gt; list = Arrays.asList\("hello", "world", "helloworld"\);

List&lt;String&gt; collect = list.stream\(\).map\(s -&gt; s.toUpperCase\(\)\).collect\(Collectors.toList\(\)\);

这里我们使用了Stream的map方法，map方法接收一个Function函数式接口实例，这里的map和Hadoop中的map概念完全一致，对每个元素进行映射处理。然后传入lambda表达式将每个元素转换大写，通过collect方法将结果收集到ArrayList中。



&lt;R&gt; Stream&lt;R&gt; map\(Function&lt;? super T, ? extends R&gt; mapper\);//map函数定义

那如果我们想把结果放到Set中或者替他的集合容器，也可以这样



list.stream\(\).map\(s -&gt; s.toUpperCase\(\)\).collect\(Collectors.toSet\(\)\);//放到Set中

或者更为通用的



list.stream\(\).map\(s -&gt; s.toUpperCase\(\)\).collect\(Collectors.toCollection\(TreeSet::new\)\);//自定义容器类型

我们可以自己制定结果容器的类型Collectors的toCollection接受一个Supplier函数式接口类型参数，可以直接使用构造方法引用的方式。



Stream中除了map方法对元素进行映射外，还有一个flatMap方法



&lt;R&gt; Stream&lt;R&gt; flatMap\(Function&lt;? super T, ? extends Stream&lt;? extends R&gt;&gt; mapper\);

flatMap从方法命名上可以解释为扁平的map





map方法是将一个容器里的元素映射到另一个容器中。



flatMap方法，可以将多个容器的元素全部映射到一个容器中，即为扁平的map。

看一个求每个元素平方的例子



Stream&lt;List&lt;Integer&gt;&gt; listStream =

                Stream.of\(Arrays.asList\(1\), Arrays.asList\(2, 3\), Arrays.asList\(4, 5, 6\)\);

List&lt;Integer&gt; collect1 = listStream.flatMap\(theList -&gt; theList.stream\(\)\).

                map\(integer -&gt; integer \* integer\).collect\(Collectors.toList\(\)\);

首先我们创建了一个Stream对象，Stream中的每个元素都是容器List&lt;Integer&gt;类型，并使用三个容器list初始化这个Stream对象，然后使用flatMap方法将每个容器中的元素映射到一个容器中，这时flatMap接收的参数Funciton的泛型T就是List&lt;Integer&gt;类型，返回类型就是T对应的Stream。最后再对这个容器使用map方法求出买个元素的平方。



然后介绍一个用于获取统计信息的方法



//同时获取最大 最小 平均值等信息

List&lt;Integer&gt; list1 = Arrays.asList\(1, 3, 5, 7, 9, 11\);

IntSummaryStatistics statistics = list1.stream\(\).filter\(integer -&gt; integer &gt; 2\).mapToInt\(i -&gt; i \* 2\).skip\(2\).limit\(2\).summaryStatistics\(\);

System.out.println\(statistics.getMax\(\)\);//18

System.out.println\(statistics.getMin\(\)\);//14

System.out.println\(statistics.getAverage\(\)\);//16

将list1中的数据取出大于2的，每个数进行平方计算，skip\(2\)忽略前两个，limit\(2\)再取出前两个，summaryStatistics对取出的这两个数计算统计数据。mapToInt接收一个ToIntFunction类型，也就是接收一个参数返回值是int类型。



接下来看一下Stream中的一个静态方法，generate方法



/\*\*

 \* Returns an infinite sequential unordered stream where each element is

 \* generated by the provided {@code Supplier}.  This is suitable for

 \* generating constant streams, streams of random elements, etc.

 \*

 \* @param &lt;T&gt; the type of stream elements

 \* @param s the {@code Supplier} of generated elements

 \* @return a new infinite sequential unordered {@code Stream}

 \*/

public static&lt;T&gt; Stream&lt;T&gt; generate\(Supplier&lt;T&gt; s\) {

    Objects.requireNonNull\(s\);

    return StreamSupport.stream\(

            new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef&lt;&gt;\(Long.MAX\_VALUE, s\), false\);

}

generate接收一个Supplier，适合生成连续不断的流或者一个全部是随机数的流



Stream.generate\(UUID.randomUUID\(\)::toString\).findFirst\(\).ifPresent\(System.out::println\);

使用UUID.randomUUID\(\)::toString 方法引用的方式创建了Supplier，然后取出第一个元素，这里的findFirst返回的是 Optional，因为流中有可能没有元素，为了避免空指针，在使用前 ifPresent 进行是否存在的判断。



最后再学习一下另一个静态方法，iterate



/\*\*

 \* Returns an infinite sequential ordered {@code Stream} produced by iterative

 \* application of a function {@code f} to an initial element {@code seed},

 \* producing a {@code Stream} consisting of {@code seed}, {@code f\(seed\)},

 \* {@code f\(f\(seed\)\)}, etc.

 \*

 \* &lt;p&gt;The first element \(position {@code 0}\) in the {@code Stream} will be

 \* the provided {@code seed}.  For {@code n &gt; 0}, the element at position

 \* {@code n}, will be the result of applying the function {@code f} to the

 \* element at position {@code n - 1}.

 \*

 \* @param &lt;T&gt; the type of stream elements

 \* @param seed the initial element

 \* @param f a function to be applied to to the previous element to produce

 \*          a new element

 \* @return a new sequential {@code Stream}

 \*/

public static&lt;T&gt; Stream&lt;T&gt; iterate\(final T seed, final UnaryOperator&lt;T&gt; f\) {

    Objects.requireNonNull\(f\);

    final Iterator&lt;T&gt; iterator = new Iterator&lt;T&gt;\(\) {

        @SuppressWarnings\("unchecked"\)

        T t = \(T\) Streams.NONE;



        @Override

        public boolean hasNext\(\) {

            return true;

        }



        @Override

        public T next\(\) {

            return t = \(t == Streams.NONE\) ? seed : f.apply\(t\);

        }

    };

    return StreamSupport.stream\(Spliterators.spliteratorUnknownSize\(

            iterator,

            Spliterator.ORDERED \| Spliterator.IMMUTABLE\), false\);

}

iterate方法有两个参数，第一个是seed也可以称作种子，第二个是一个UnaryOperator，UnaryOperator实际上是Function的一个子接口，和Funciton区别就是参数和返回类型都是同一种类型



@FunctionalInterface

public interface UnaryOperator&lt;T&gt; extends Function&lt;T, T&gt; {



}

iterate方法第一次生成的元素是UnaryOperator对seed执行apply后的返回值，之后所有生成的元素都是UnaryOperator对上一个apply的返回值再执行apply，不断循环。

f\(f\(f\(f\(f\(f\(n\)\)\)\)\)\)......



//从1开始，每个元素比前一个元素大2，最多生成10个元素

Stream.iterate\(1,item -&gt; item + 2\).limit\(10\).forEach\(System.out::println\);

我们在使用stream api时也要注意一些陷阱，比如下面这个例子



//Stream陷阱 distinct\(\)会一直等待产生的结果去重，将distinct\(\)和limit\(6\)调换位置，先限制结果集再去重就可以了

IntStream.iterate\(0,i -&gt; \(i + 1\) % 2\).distinct\(\).limit\(6\).forEach\(System.out::println\);

如果distinct\(\)一直等待那程序会一直执行不断生成数据，所以需要先限制结果集再去进行去重操作就可以了。

