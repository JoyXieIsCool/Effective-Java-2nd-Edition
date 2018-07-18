通常一个类要允许客户端获取它的实例的方式就是提供一个构造函数，不过还有一个方法，它应该成为每个程序员的工具箱的一部分。一个类可以提供一个共有的静态工厂方法，即一个简单的可以返回该类对象的静态方法。下面是一个简单的例子，摘自`Boolean`（基本类型boolean装箱后的原始类型）。这个方法把一个`boolean`原始类型转为了`Boolean`对象引用：  

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

需要注意的是，一个静态工厂方法跟设计模式[Gamma95, p. 107]中的工厂方法模式是不一样的。
