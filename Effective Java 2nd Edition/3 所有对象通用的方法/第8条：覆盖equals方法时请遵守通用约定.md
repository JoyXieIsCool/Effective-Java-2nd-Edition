覆盖`equals`方法看起来很简单，但是有很多方式会导致错误，而且后果非常严重。避免这个问题的最简单方法就是不覆盖`equals`方法，这样类的每个实例都只和它自己相等。如果满足下面任何一个条件的话，那么这样做就是对的：  

- **类的每个实例本质上都是唯一的。**对于像`Thread`这种代表活动项而不是值的类来说，它满足这个条件，而`Object`提供的`equals`实现对这些类来说正好提供了正确的行为。

- **你并不关心该类是否提供“逻辑相等性”的判断**。例如，`java.util.Random`可以覆盖`equals`来检查两个`Random`实例是否会提供相同的随机数序列，但是设计者并不认为客户会需要或想要这个功能。在这种情况下，从`Object`继承的`equals`实现就可以满足需求了。

- **一个父类已经覆盖了`equals`，并且父类的行为适用于当前子类。**例如，大部分的`Set`实现都从`AbstractSet`继承了`equals`实现，`List`实现继承于`AbstractList`，`Map`实现继承于`AbstractMap`。

- **类是私有的或包私有的，并且你确定它的`equals`方法永远不会被调用。**在这些情况下，`equals`方法应该这样覆盖以防被错误调用： 
   
```java  
@Override public boolean equals(Object o) {    throw new AssertionError(); // Method is never called}
```

所以什么时候应该覆盖`Object.equals`方法呢？如果类有逻辑相等(logical equality)的概念(不是对象相同)，并且它的父类没有覆盖`equals`方法来实现需要的行为，那么就需要覆盖`equals`方法。这通常是“值类”(value classes)的情形，一个值类简单来说就是代表一个值的类，例如`Integer`或`Date`。程序员使用`equals`方法来比较值对象的引用时，希望知道它们是否是逻辑相等的，而不关心它们是否指向同一个对象。覆盖`equals`方法不仅满足了程序员的需求，它也允许类的实例称为map的key、set的元素，并且符合可预期的行为。

有一种值类不需要覆盖`equals`方法，那就是使用了实例控制(见第1条)来保证每个值最多有一个对象存在的类，枚举类型(见第30条)就属于这种。对于这些类来说，逻辑相等和对象等同是一样的，因此`Object`的`equals`方法在功能上等同于逻辑的`equals`方法。

当覆盖`equals`方法时，你必须遵守它的通用约定。下面是从`Object`的规范中[JavaSE6]引用的约定：  
`equals`方法实现了一个相等关系，它是：  
- ***自反性(Reflexive)***：对于任何非null的引用值x，`x.equals(x)`必须返回`true`。   
- ***对称性(Symmetric)***：对于任何非null的引用值x和y，当且仅当`y.equals(x)`返回true时，`x.equals(y)`返回true。  
- ***传递性(Transitive)***：对于任何非null的引用值x、y、z，若`x.equals(y)`返回true，且` y.equals(z)`返回true，那么`x.equals(z)`必须返回true。  
- ***一致性(Consistent)***：对于任何非null的引用值x和y，只要`equals`的比较对象信息没有被修改，多次调用`x.equals(y)`必须一致地返回true或一致地返回false。
- 对于任何非null的引用值x，`x.equals(null)`必须返回`false`。 

除非你对数学很感兴趣，否则这些约定看起来有点吓人，但是绝对不要忽视它！如果你违反了它们，那么你会发现你的程序行为很不确定甚至崩溃，并且很难找到失败的源头。用John Donne的话说，没有一个类是孤岛。一个类的实例会频繁地传递给其他类的实例，包括集合类在内的很多类都依赖于传递给它们的对象是否遵守了`equals`约定。

你已经清楚违反`equals`约定的危险了，现在我们继续深入了解这些约定的细节。好消息是，这些约定看起来很复杂，但实际上并没有看起来那么吓人。只要你理解了它，那么遵守它们并不困难。现在让我们逐个了解一下这5个要求：  

- ***自反性(Reflexive)***：第一个要求仅仅说了对象必须等于它本身。很难想象会在无意中违反这个要求。假如你违反了这条约定并将类的实例添加到一个集合中，集合的`contains`方法会立即告诉你当前集合并不包含你刚刚添加的实例。  
- ***对称性(Symmetric)***：第二条要求是任何两个对象必须对equals调用返回相同的结果。与第一个要求不同的是，并不难想象你会不小心违反它。例如考虑下面这个类，它实现了一个大小写敏感的string类，字符串由`toString`保存，但是在比较时忽略了：   

```java
// Broken - 违反了对称性！
public final class CaseInsensitiveString {
    private final String s;
    public CaseInsensitiveString(String s) {
        if (s == null)
            throw new NullPointerException();
        this.s = s;
    }

    // Broken - 违反了对称性！
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 单向可用(One-way interoperability)！
            return s.equalsIgnoreCase((String) o);
        return false;
    }
       
    ...  // 其他方法忽略
}
```

这个类中的`equals`方法意图是很好的，它企图简单地与普通字符串互相操作，假设我们有一个大小写不敏感的字符串和一个普通字符串：  

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish"); 
String s = "polish";
```

如预期的一样，`cis.equals(s)`返回`true`。但问题是，`CaseInsensitiveString`中的`equals`方法可以察觉到普通字符串，而`String`中的`equals`方法并不知道`CaseInsensitiveString`的存在。因此`s.equals(cis)`会返回`false`，这明显违反了对称性。假如你将一个大小写不敏感的`CaseInsensitiveString`字符串放入了一个集合：  

```java
List<CaseInsensitiveString> list =       new ArrayList<CaseInsensitiveString>();list.add(cis);
```

此时`list.contains(s)`会返回什么结果呢？没有人知道。在Sun的当前实现中它刚好返回`false`，但这只是一个特定的实现。在其它的实现中，它可能会返回`true`或者抛出运行时异常。**一旦你违反了`equals`约定，当其它对象面对你的对象时，你完全不知道这些对象的行为会是什么样的。**

要解决这个问题只需要把企图与String互相操作的这段代码从`equals`方法中移除即可。只要你这么做了，你就可以重构这个方法让它成为一条单行的返回语句：  

```java
@Override public boolean equals(Object o) {    return o instanceof CaseInsensitiveString &&        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);}
```

- ***传递性(Transitive)***：`equals`约定的第三条要求是，如果一个对象等于第二个对象并且第二个对象等于第三个对象，那么第一个对象必须等于第三个对象。重申一下，无意识地违反这条规则的情况并不难以想象。考虑这样一种情况，子类添加了一个新的值组件(value component)到它的父类中，换句话说，子类添加了一块新的信息，它会影响`equals`的比较结果。我们先从一个简单的不可变的二维整数型Point类开始：  

```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
	    this.x = x;
	    this.y = y; 
	}

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ...  // Remainder omitted
}
```

假设你想扩展这个类，给Point添加颜色的概念：  

```java
public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ...  // Remainder omitted
}
```

`equals`方法会怎样呢？如果你完全不管它，那么它会从`Point`类中继承并在`equals`比较时忽略颜色信息。虽然这没有违反`equals`约定，但明显是不可接受的。假设你写了一个`equals`方法，只有当它的参数是另一个颜色点且拥有与自身相同的位置和颜色时才返回`true`：  

```java
// Broken - violates symmetry!
@Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```



