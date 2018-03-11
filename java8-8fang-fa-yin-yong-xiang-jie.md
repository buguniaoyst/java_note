上一篇我们详细介绍了Optional类用来避免空指针问题，本篇我们全面了解一下Java8中的方法引用特性。

方法引用是lambda表达式的一种特殊形式，如果正好有某个方法满足一个lambda表达式的形式，那就可以将这个lambda表达式用方法引用的方式表示，但是如果这个lambda表达式的比较复杂就不能用方法引用进行替换。实际上方法引用是lambda表达式的一种语法糖。

在介绍方法引用使用方式之前，先将方法引用分下类

方法引用共分为四类：

1.类名::静态方法名

2.对象::实例方法名

3.类名::实例方法名 

4.类名::new



首先来看下第一种 类名::静态方法名 为了演示我们自定义了一个Student类



public class Student {

    private String name;

    private int score;



    public Student\(\){



    }



    public Student\(String name,int score\){

        this.name = name;

        this.score = score;

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



    public static int compareStudentByScore\(Student student1,Student student2\){

        return student1.getScore\(\) - student2.getScore\(\);

    }

}

Student类有两个属性name和score并提供了初始化name和score的构造方法，并且在最下方提供了两个静态方法分别按score和name进行比较先后顺序。

接下来的需求是，按着分数由小到大排列并输出，在使用方法引用前，我们先使用lambda表达式的方式进行处理



Student student1 = new Student\("zhangsan",60\);

Student student2 = new Student\("lisi",70\);

Student student3 = new Student\("wangwu",80\);

Student student4 = new Student\("zhaoliu",90\);

List&lt;Student&gt; students = Arrays.asList\(student1,student2,student3,student4\);



students.sort\(\(o1, o2\) -&gt; o1.getScore\(\) - o2.getScore\(\)\);

students.forEach\(student -&gt; System.out.println\(student.getScore\(\)\)\);

sort方法接收一个Comparator函数式接口，接口中唯一的抽象方法compare接收两个参数返回一个int类型值，下方是Comparator接口定义



@FunctionalInterface

public interface Comparator&lt;T&gt; {

    int compare\(T o1, T o2\);

}

我们再看下Student类中定义的compareStudentByScore静态方法



public static int compareStudentByScore\(Student student1,Student student2\){

    return student1.getScore\(\) - student2.getScore\(\);

}

同样是接收两个参数返回一个int类型值，而且是对Student对象的分数进行比较，所以我们这里就可以 使用类名::静态方法名 方法引用替换lambda表达式



students.sort\(Student::compareStudentByScore\);

students.forEach\(student -&gt; System.out.println\(student.getScore\(\)\)\);

第二种 对象::实例方法名

我们再自定义一个用于比较Student元素的类



public class StudentComparator {

    public int compareStudentByScore\(Student student1,Student student2\){

        return student2.getScore\(\) - student1.getScore\(\);

    }

}

StudentComparator中定义了一个非静态的，实例方法compareStudentByScore，同样该方法的定义满足Comparator接口的compare方法定义，所以这里可以直接使用 对象::实例方法名 的方式使用方法引用来替换lambda表达式



StudentComparator studentComparator = new StudentComparator\(\);

students.sort\(studentComparator::compareStudentByScore\);

students.forEach\(student -&gt; System.out.println\(student.getScore\(\)\)\);

第三种 类名::实例方法名 

这种方法引用的方式较之前两种稍微有一些不好理解，因为无论是通过类名调用静态方法还是通过对象调用实例方法这都是符合Java的语法，使用起来也比较清晰明了。那我们带着这个疑问来了解一下这个比较特殊的方法引用。

现在再看一下Student类中静态方法的定义



public static int compareStudentByScore\(Student student1,Student student2\){

    return student1.getScore\(\) - student2.getScore\(\);

}

虽然这个方法在语法上没有任何问题，可以作为一个工具正常使用，但是有没有觉得其在设计上是不合适的或者是错误的。这样的方法定义放在任何一个类中都可以正常使用，而不只是从属于Student这个类，那如果要定义一个只能从属于Student类的比较方法下面这个实例方法更合适一些



public int compareByScore\(Student student\){

    return this.getScore\(\) - student.getScore\(\);

}

接收一个Student对象和当前调用该方法的Student对象的分数进行比较即可。现在我们就可以使用 类名::实例方法名 这种方式的方法引用替换lambda表达式了



students.sort\(Student::compareByScore\);

students.forEach\(student -&gt; System.out.println\(student.getScore\(\)\)\);

这里非常奇怪，sort方法接收的lambda表达式不应该是两个参数么，为什么这个实例方法只有一个参数也满足了lambda表达式的定义（想想这个方法是谁来调用的）。这就是 类名::实例方法名 这种方法引用的特殊之处：当使用 类名::实例方法名 方法引用时，一定是lambda表达式所接收的第一个参数来调用实例方法，如果lambda表达式接收多个参数，其余的参数作为方法的参数传递进去。

结合本例来看，最初的lambda表达式是这样的



students.sort\(\(o1, o2\) -&gt; o1.getScore\(\) - o2.getScore\(\)\);

那使用 类名::实例方法名 方法引用时，一定是o1来调用了compareByScore实例方法，并将o2作为参数传递进来进行比较。是不是就符合了compareByScore的方法定义。



第四种 类名::new

也称构造方法引用，和前两种类似只要符合lambda表达式的定义即可，回想下Supplier函数式接口的get方法，不接收参数有返回值，正好符合无参构造方法的定义

@FunctionalInterface

public interface Supplier&lt;T&gt; {



/\*\*

 \* Gets a result.

 \*

 \* @return a result

 \*/

T get\(\);

}



Supplier&lt;Student&gt; supplier = Student::new;

上面就是使用了Student类构造方法引用创建了supplier实例，以后通过supplier.get\(\)就可以获取一个Student类型的对象，前提是Student类中存在无参构造方法。



小结：本篇全面介绍了方法引用的四种使用方式，且每种方式都有对应一个示例来帮助大家理解。当我们使用lambda表达式进行函数式编程时，如果某个方法正好满足lambda的定义，也满足实际需求的逻辑，就可以使用方法引用的方式来替换lambda表达式。接下来我们将真正开始学习stream api，并结合前面学习的内容体验stream api的强大之处。

