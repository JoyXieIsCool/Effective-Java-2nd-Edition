覆盖`equals`方法看起来很简单，但是有很多方式会导致错误，而且后果非常严重。避免这个问题的最简单方法就是不覆盖`equals`方法，这样类的每个实例都只和它自己相等。如果满足下面任何一个条件的话，那么这样做就是对的：  

- **类的每个实例本质上都是唯一的。**对于像`Thread`这种代表活动项而不是值的类来说，它满足这个条件，而`Object`提供的`equals`实现对这些类来说正好提供了正确的行为。

- **你并不关心该类是否提供“逻辑相等性”的判断**。例如，`java.util.Random`可以覆盖`equals`来检查两个`Random`实例是否会提供相同的随机数序列，但是设计者并不认为客户会需要或想要这个功能。在这种情况下，从`Object`继承的`equals`实现就可以满足需求了。

- **一个父类已经覆盖了`equals`，并且父类的行为适用于当前子类。**例如，大部分的`Set`实现都从`AbstractSet`继承了`equals`实现，`List`实现继承于`AbstractList`，`Map`实现继承于`AbstractMap`。

- **类是私有的或包私有的，并且你确定它的`equals`方法永远不会被调用。**在这些情况下，`equals`方法应该这样覆盖以防被错误调用： 
   
```java  
@Override public boolean equals(Object o) {    throw new AssertionError(); // Method is never called}
```

所以什么时候应该覆盖`Object.equals`方法呢？如果类有逻辑相等(logical equality)的概念(不是对象相同)，并且它的父类没有覆盖`equals`方法来实现需要的行为，那么就需要覆盖`equals`方法。这通常是“值类”(value classes)的情形，一个值类简单来说就是代表一个值的类，例如`Integer`或`Date`。程序员使用`equals`方法来比较值对象的引用时，希望知道它们是否是逻辑相等的，而不关心它们是否指向同一个对象。覆盖`equals`方法不仅满足了程序员的需求，它也允许类的实例称为map的key、set的元素，并且符合可预期的行为。

