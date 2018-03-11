上一篇我们详细介绍了Predicate函数式接口中主要的一些方法使用，本篇介绍的Optional虽然并不是一个函数式接口，但是也是一个极其重要的类。



Optional并不是我们之前介绍的一系列函数式接口，它是一个class，主要作用就是解决Java中的NPE（NullPointerException）。空指针异常在程序运行中出现的频率非常大，我们经常遇到需要在逻辑处理前判断一个对象是否为null的情况。



if\(null != person\){

    Address address = person.getAddress\(\);

    if\(null != address\){

        ......

    }

}

实际开发中我们经常会按上面的方式进行非空判断，接下来看下使用Optional类如何避免空指针问题



String str = "hello";

Optional&lt;String&gt; optional = Optional.ofNullable\(str\);

optional.ifPresent\(s -&gt; System.out.println\(s\)\);//value为hello，正常输出

首先，ofNullable方法接收一个可能为null的参数，将参数的值赋给Optional类中的成员变量value，ifPresent方法接收一个Consumer类型函数式接口实例，再将成员变量value交给Consumer的accept方法处理前，会校验成员变量value是否为null，如果value是null，则什么也不会执行，避免了空指针问题。下方是ifPresent源码



/\*\*

 \* If a value is present, invoke the specified consumer with the value,

 \* otherwise do nothing.

 \*

 \* @param consumer block to be executed if a value is present

 \* @throws NullPointerException if value is present and {@code consumer} is

 \* null

 \*/

public void ifPresent\(Consumer&lt;? super T&gt; consumer\) {

    if \(value != null\)

        consumer.accept\(value\);

}

如果传入的内容是空，则什么也不会执行，也不会有空指针异常



String str = null;

Optional&lt;String&gt; optional = Optional.ofNullable\(str\);

optional.ifPresent\(s -&gt; System.out.println\(s\)\);//不会输出任何内容

如果为空时想返回一个默认值



String str = null;

Optional&lt;String&gt; optional = Optional.ofNullable\(str\);

System.out.println\(optional.orElseGet\(\(\) -&gt; "welcome"\)\);

orElseGet方法接收一个Supplier，还记得前面介绍的Supplier么，不接受参数通过get方法直接返回结果，类似工厂模式，上面代码就是针对传入的str变量，如果不为null那正常输出，如果为null，那返回一个默认值"welcome"



orElseGet方法源码



/\*\*

 \* Return the value if present, otherwise invoke {@code other} and return

 \* the result of that invocation.

 \*

 \* @param other a {@code Supplier} whose result is returned if no value

 \* is present

 \* @return the value if present otherwise the result of {@code other.get\(\)}

 \* @throws NullPointerException if value is not present and {@code other} is

 \* null

 \*/

public T orElseGet\(Supplier&lt;? extends T&gt; other\) {

    return value != null ? value : other.get\(\);

}

