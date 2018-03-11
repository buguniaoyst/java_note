Java8被称作Java史上变化最大的一个版本。其中包含很多重要的新特性，最核心的就是增加了Lambda表达式和Stream API。这两者也可以结合在一起使用。首先来看下什么是Lambda表达式。  
Lambda表达式，维基百科上的解释是一种用于表示匿名函数和闭包的运算符，感觉看到这个解释还是觉得很抽象，接下来我们看一个例子

```java
public
class
SwingTest
 {
    
public
static
void
main
(
String[] args
) 
{
        JFrame jFrame = 
new
 JFrame(
"My JFrame"
);
        JButton jButton = 
new
 JButton(
"My JButton"
);

        jButton.addActionListener(
new
 ActionListener() {
            @
Override
            
public
void
actionPerformed
(
ActionEvent e
) 
{                
                System.
out
.println(
"Button Pressed!"
);
            } 
        }); 
        
        jFrame.
add
(jButton); jFrame.pack(); 
        jFrame.setVisible(
true
); 
        jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); 
    }
}
```

这是一段Swing编程中的代码，给Button绑定一个监听事件，当点击Button时会在控制台输出"Button Pressed!"内容。这里使用了创建了一个匿名内部类的实例来绑定到监听器，这也是以往比较常规的代码组织形式。但是仔细看一下我们会发现，实际上我们真正关注的就是一个ActionEvent类型的参数e和向控制台输出的语句System.out.println\("Button Pressed!"\);。  
如果将上段程序中以匿名内部类的方式创建接口实例的代码替换成Lambda表达式后，代码如下  
public class SwingTest {

```
public static void main(String[] args) {
    
JFrame 
jFrame 
= new 
JFrame("My 
JFrame");

JButton 
jButton 
= new 
JButton("My 
JButton");

jButton.addActionListener(e 
-
>
 System.out.println(
"Button Pressed!"
))
;
jFrame.add(jButton);

jFrame.pack();

jFrame.setVisible(true);

jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

}
```

}  
关注最中间部分代码的变化，由原来的6行代码，现在1行就可以实现了。这就是Lambda表达式的一种简单形式。  
可以看出Lambda表达式的语法是  
\(param1,param2,param3\) -&gt; {

```
//todo
```

}  
这里参数的类型程序可以根据上下文进行推断，但是并不是所有的类型都可以推断出来，此时就需要我们显示的声明参数类型，当只有一个参数时小括号可以省略。当todo部分只有一行代码时，外边的大括号可以省略。如我们上面的示例  
那么除了代码简洁了，Lambda表达式还给我们带来了什么变化吗？  
我们回忆一下，在Java中，我们是否无法将函数作为参数传递给一个方法，也无法声明返回值是一个函数的方法。在Java8之前，答案是肯定的。  
那么，在上面的例子中我们居然可以将一段代码逻辑作为参数传递给了监听器，告诉监听器事件触发时你可以这么做，而不再需要以匿名内部类的方式作为参数。这也是Java8带来的另一新特性：函数式编程。  
支持函数式编程的语言有很多，在JavaScript中，把函数作为参数传递，或者返回值是一个函数的情况非常常见，JavaScript是一门非常常见的函数式语言。  
Lambda为Java添加了缺失的函数式编程的特性，使我们能将函数当做一等公民看待。  
在函数式编程语言中，Lambda表达式的类型是函数。**而在Java中，Lambda表达式是对象，它们必须依附于一类特别的对象类型——函数式接口\(Functional Interface\)**。  
接下来我们看下函数式接口的定义：  
如果一个接口中，有且只有一个抽象的方法（Object类中的方法不包括在内），那这个接口就可以被看做是函数式接口。

```
@FunctionalInterface
public
interface
Runnable
{
    
/**
     * When an object implementing interface 
<
code
>
Runnable
<
/code
>
 is used
     * to create a thread, starting the thread causes the object's
     * 
<
code
>
run
<
/code
>
 method to be called in that separately executing
     * thread.
     * 
<
p
>

     * The general contract of the method 
<
code
>
run
<
/code
>
 is that it may
     * take any action whatsoever.
     *
     * 
@see
     java.lang.Thread#run()
     */
public
abstract
void
run
()
;
}
```

来看下Runnable接口的声明，在Java8后，Runnable接口多了一个FunctionalInterface注解，表示该接口是一个函数式接口。但是如果我们不添加FunctionalInterface注解的话，如果接口中有且只有一个抽象方法时，编译器也会把该接口当做函数式接口看待。

```
@FunctionalInterface
public
interface
MyInterface
{
    
void
test
()
;
    
String 
toString
()
;
}
```

MyInterface这也是一个函数式接口，因为toString\(\)是Object类中的方法，只是在这里进行了复写，不会增加接口中抽象方法的数量。  
\(到这里额外提一下，Java8中，接口里面的方法不仅仅只能有抽象方法，也可以有具体实现了的方法，被称作默认方法\(default method\)，这部分后面会具体介绍\)  
既然在Java中，Lambda表达式是对象。那么这个对象的类型是什么呢？我们再回顾下SwingTest程序，这里以匿名内部类的方式创建了一个ActionListener接口实例

```
jButton.addActionListener(
new
 ActionListener() {
    
@Override
public
void
actionPerformed
(ActionEvent e)
{                
        System.out.println(
"Button Pressed!"
);
    } 
}); 
```

使用Lambda表达式改进后

```
jButton
.addActionListener
(
e
-
>
System
.out
.println
("
Button
Pressed
!"));
```

也就是我们使用Lambda表达式创建了一个ActionListener接口的实例，再看下ActionListener接口的定义

```
public
interface
ActionListener
extends
EventListener
{
    
/**
     * Invoked when an action occurs.
     */
public
void
 actionPerformed(ActionEvent e);
}
```

只有一个抽象方法，虽然没添加FunctionalInterface注解，但是也符合函数式接口的定义，编译器会认为这是一个函数式接口。  
所以，使用Lambda表达式可以创建函数式接口的实例。即Lambda表达式返回的是函数式接口类型。  
实际上，函数式接口实例的创建可以有三种方式（参考自FunctionalInterface注解说明）：  
1.Lambda表达式  
2.方法引用（后续章节介绍）  
3.构造方法引用（后续章节介绍）

