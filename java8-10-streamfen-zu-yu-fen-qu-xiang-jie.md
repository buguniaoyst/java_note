上一篇我们介绍了Strem的概念与实际的一些操作，本篇我们继续来学习Stream的另一个重要操作，分组与分区。

我们在上一篇介绍Stream的操作时，会经常使用到Collectors这个类，这个类实际上是一个封装了很多常用的汇聚操作的一个工厂类。我们之前用到过



//将结果汇聚到ArrayList中

Collectors.toList\(\);

//将结果汇聚到HashSet中

Collectors.toSet\(\);

以及更为通用的



//将结果汇聚到一个指定类型的集合中

Collectors.toCollection\(Supplier&lt;C&gt; collectionFactory\);

Stream分组



在实际开发中，对于将一个集合的内容进行分组或分区这种需求也非常常见，所以我们继续学习下Collectors类中的groupingBy和partitioningBy方法。



public static Collector groupingBy\(Function&lt;? super T, ? extends K&gt; classifier\){

    //...

}

groupingBy接收一个Function类型的变量classifier，classifier被称作分类器，收集器会按着classifier作为key对集合元素进行分组，然后返回Collector收集器对象，假如现在有一个实体Student



public class Student {

    private String name;

    private int score;

    private int age;



    public Student\(String name,int score,int age\){

        this.name = name;

        this.score = score;

        this.age = age;

    }



    public String getName\(\) {

        return name;

    }



    public void setName\(String name\) {

        this.name = name;

    }



    public int getScore\(\) {

        return score;

    }



    public void setScore\(int score\) {

        this.score = score;

    }



    public int getAge\(\) {

        return age;

    }



    public void setAge\(int age\) {

        this.age = age;

    }

}

我们现在按Student的name进行分组，如果使用sql来表示就是select \* from student group by name; 再看下使用Stream的方式



Map&lt;String, List&lt;Student&gt;&gt; collect = students.stream\(\).collect\(Collectors.groupingBy\(Student::getName\)\);

这里我们使用方法引用（类名::实例方法名）替代lambda表达式（s -&gt; s.getName\(\)）的方式来指定classifier分类器，使集合按Student的name来分组。

注意到分组后的返回类型是Map&lt;String, List&lt;Student&gt;&gt;，结果集中会将name作为key，对应的Student集合作为value返回。

那如果按name分组后，想求出每组学生的数量，就需要借助groupingBy另一个重载的方法



public static Collector groupingBy\(Function&lt;? super T, ? extends K&gt; classifier,Collector&lt;? super T, A, D&gt; downstream\){

    //...

}

第二个参数downstream还是一个收集器Collector对象，也就是说我们可以先将classifier作为key进行分组，然后将分组后的结果交给downstream收集器再进行处理



//按name分组 得出每组的学生数量 使用重载的groupingBy方法，第二个参数是分组后的操作

Map&lt;String, Long&gt; collect1 = students.stream\(\).collect\(Collectors.groupingBy\(Student::getName, Collectors.counting\(\)\)\);

Collectors类这里也帮我们封装好了用于统计数量的counting\(\)方法，这里先了解一下counting\(\)就是将收集器中元素求总数即可，后续我们会再深入源码学习。



我们还可以对分组后的数据求平均值



Map&lt;String, Double&gt; collect2 = students.stream\(\).collect\(Collectors.groupingBy\(Student::getName, Collectors.averagingDouble\(Student::getScore\)\)\);

averagingDouble方法接收一个ToDoubleFunction参数



@FunctionalInterface

public interface ToDoubleFunction&lt;T&gt; {



    /\*\*

     \* Applies this function to the given argument.

     \*

     \* @param value the function argument

     \* @return the function result

     \*/

    double applyAsDouble\(T value\);

}

ToDoubleFunction实际上也是Function系列函数式接口中的其中一个特例，接收一个参数，返回Double类型（这里是接收一个Student返回score）。因为分组后的集合中每个元素是Student类型的，所以我们无法直接对Student进行求平均值



//伪代码

Collectors.averagingDouble\(Student\)\)

所以需要将Student转成score再求平均值，Collectors.averagingDouble\(Student::getScore\)\)。



Stream分区



针对上面的Student，我们现在再加一个需求，分别统计一下及格和不及格的学生（分数是否&gt;=60）

这时候符合Stream分区的概念了，Stream分区会将集合中的元素按条件分成两部分结果，key是Boolean类型，value是结果集，满足条件的key是true，我们看下示例。



Map&lt;Boolean, List&lt;Student&gt;&gt; collect3 = students.stream\(\).collect\(Collectors.partitioningBy\(student -&gt; student.getScore\(\) &gt;= 60\)\);

System.out.println\(collect3.get\(true\)\);//输出及格的Student

System.out.println\(collect3.get\(false\)\);//输出不及格的Student

partitioningBy方法接收一个Predicate作为分区判断的依据，满足条件的元素放在key为true的集合中，反之放在key为false的集合中



//partitioningBy方法

public static Collector partitioningBy\(Predicate&lt;? super T&gt; predicate\) {

    return partitioningBy\(predicate, toList\(\)\);

}

