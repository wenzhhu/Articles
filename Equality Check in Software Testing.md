# 前言
相等性（Equality）检查可以说是软件测试中最常见的操作。一个典型的测试场景一般由以下三个步骤组成

1. 设置测试环境。

2. 发起一个需要测试的操作，获取返回值。

3. 将实际返回值（Actual Value）与期望值（Expected Value）进行相等性比较来确定测试通过与否。

本文将探讨步骤三，即如何正确，有效地进行相等性检查。在这里，我会使用Java来展示一些实际的例子，但是这里的概念将适用于其他任何面向对象的语言。

# 例子
为了方便讨论，我们假设有一个图书管理系统。一个图书对象由三个字段组成
- 编号（id）是一个内部生成的，随机的数字，用来全局唯一的标识一种图书。
- 书名（title）是一个字符串
- 版本号 （version）是一个自然数
相应的Java类定义如下
```Java
public class Book {
    private final long id;
    private final String title;
    private final int version;

    public Book(long id, String title, int version) {
		super();
		this.id = id;
		this.title = title;
		this.version = version;
	}
...
}
```

# 返回值的类型
根据需要被测试的软件不同，返回值的类型可以被分为两大类。

1. Java类型

在传统的库（Library）或工具软件（Utiilty）的测试中，实际返回值一般都是Java类型。典型的例子有JDK自带的IO库，Math库等等。Java类型可以被分为基本类型（Primitive Type）和组合类型（Composite Type）。基本类型的相等性检查很简单，在Java中通过操作符“=”实现。这里的我们需要着重探讨的是组合类型，也就是Java中的类（Class）。本文中的图书对象就是一个Java类。比如《Effective Java 第二版》就可以表示为
 ```
new Book(123, "Effective Java", 2) // 123是一个随机生成的，全局唯一的图书号
```
组合类型的相等性检查在Java中一般通过equals() 方法实现。

2. 对象的序列化类型

在Web Service的测试中，实际返回值一般由HTTP的body部分表示。它代表了我们所关心的实际返回值对象的序列化的形式。典型的例子有RESTful API中常用的JSON或XML表示形式。同样用《Effective Java 第二版》举例：
- JSON

```
{
  "id" : 123,
  "title" : "Effective Java",
  "version" : 2
}
```
- XML
```
<Book>
  <id>123</id>
  <title>Effective Java</title>
  <version>2</version>
</Book>
```

通过合适的反序列化，我们可以由JSON或XML字符串得到Java对象。因此，对于这种情况，我们可以认为实际的返回值类型也是Java类型。

有时，在问题域中，并没有现成的或合适的Java类型可以用来反序列化JSON或XML字符串。所以，我们需要直接对这些字符串进行相等性检查。注意，由于以下一些原因，我们并不能直接使用String类的equals方法。（String的equals方法实现使用严格的逐字符比较）

- JSON或XML是自由格式的（Free Format），也就是说其中的空格，Tab键，甚至换行符都是可以被忽略的。这些字符不应该参与相等性比较。

- JSON或XML有时候会被用来表示集合（Set），即其中的元素在一个序列中是没有先后顺序的。元素的顺序不应该参与相等性比较。

- 对于问题域来说，有些字段（field）不应该参与相等性比较。比如图书的id是服务器端随机生成的，我们无法事先得知，因此也不可能构造一个期望的id来进行相等性检查。

对于这种情况，我们需要借助一些第三方JSON或XML的工具库来进行相等性检查。在某些情况下，我们甚至需要自己来实现一些专用的相等性检查工具。

# Java对象的相等性检查
几乎所有的Java测试框架都依赖于Java对象的equals方法来进行相等性检查。典型的有JUnit，Hamcrest，AssertJ等等。Equals方法是Java最基本类Object所定义的方法。一般而言，问题域中的类型都会覆盖该方法，但往往这种覆盖并不符合测试的要求。因为从测试的观点来看， 只有当对象的实际返回值与期望值在问题域的范畴中逐字段相等才被认为实际返回值与期望值相等。而从系统实现的角度来看，开发人员默认程序实现是无错的，因此只要使用可以区分两个不同的对象的字段即可。比如说，图书的id字段。进一步说，即使仅仅从性能角度出发（假设完整的图书对象含有作者，出版社，出版年份，价格等数十个其他字段），在系统内部进行逐字段相等性检查也是不可接受的。

（待续...)
