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

这种方法的问题是，当你将一个普通点与一个颜色点比较时可能会得到不同的结果，反之亦然。前一种比较忽略了颜色信息，而后者则总是返回false，因为参数的类型不正确。为了清晰地说明这点，我们创建一个普通点和一个颜色点：  

```java
Point p = new Point(1, 2);ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

然后`p.equals(cp)`返回`true`，但`cp.equals(p)`返回`false`。你可能会想在做“混合比较”时在`ColorPoint.equals`中忽略颜色，并通过这种方式来解决这个问题：

```java
// Broken - violates transitivity!
@Override public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```

这种方式确实满足了对称性，但牺牲了传递性：

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);Point p2 = new Point(1, 2);ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

现在`p1.equals(p2)`和`p2.equals(p3)`都返回`true`，但是`p1.equals(p3)`返回`false`，这明显违反了传递性。前两种比较不考虑颜色信息，第三种则考虑了颜色。

那么正确的方案是什么呢？其实这是面向对象语言中关于相等关系的一个基本问题。**你没有办法在扩展一个可实例化类的同时，既添加值组件又保持`equals`约定，**除非你愿意抛弃面向对象抽象带来的好处。

你可能听说在扩展可实例化类并添加值组件时，在`equals`方法中使用`getClass`测试来取代`instanceof`测试，通过这种方式来保留`equals`约定：  

```java
// Broken - violates Liskov substitution principle (page 40)
@Override public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

这使得只有当对象是相同实现类时才满足相等性，虽然这个方法看起来没有那么糟，但是结果却是无法接受的。

假设我们想写一个方法来判断一个整数点是否在单元环(unit circle)上，下面是一种可能的实现方式：  

```java
// Initialize UnitCircle to contain all Points on the unit circle private static final Set<Point> unitCircle;
static {
    unitCircle = new HashSet<Point>();
    unitCircle.add(new Point( 1,  0));
    unitCircle.add(new Point( 0,  1));
    unitCircle.add(new Point(-1,  0));
    unitCircle.add(new Point( 0, -1));
}

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```

虽然这可能不是实现这种功能的最快方式，但是它运行起来没有问题。假设你用某种微妙的方式扩展了`Point`类且没有添加值组件，比方说，在它的构造器中记录一共创建了多少个实例：  

```java
public class CounterPoint extends Point {
    private static final AtomicInteger counter =
        new AtomicInteger();
        
    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }
    public int numberCreated() { return counter.get(); }
}
```

里氏替换原则(Liskov substitution principle)认为，一个类型的任何重要属性也应该被它的子类持有，这样任何为该类所编写的方法在它的子类中也能同样地有效运行[Liskov87]。但是现在假设我们传递了一个`CounterPoint`实例给`onUnitCircle`方法，如果`Point`类使用了基于`getClass`的`equals`方法，那么无论`CounterPoint`实例的`x`和`y`值是什么，`onUnitCircle`方法都会返回`false`。这是因为`onUnitCircle`方法使用了像`HashSet`这样的集合来判断是否包含一个元素，并且没有任何`CounterPoint`实例等于任何一个`Point`实例。然而如果你在`Point`类中使用了合适的、基于`instanceof`的`equals`方法，那么同样的`onUnitCircle`方法在遇到`CounterPoint`实例时就能正常运行。

虽然没有一种令人完全满意的方式来扩展可实例化类并添加值组件，但并不是没有变通方法。遵循第16条的建议，“组合优先于继承”(Favor composition over inheritance)。与其让`CounterPoint`继承`Point`，我们不如给`CounterPoint`一个私有的`Point`属性，并且添加一个公有的`view`方法(见第5条)返回一个与颜色点相同位置的普通点：  

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        if (color == null)
            throw new NullPointerException();
        point = new Point(x, y);
        this.color = color;
    }

    /**
     * Returns the point-view of this color point.
     */
    public Point asPoint() { 
        return point;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
         ColorPoint cp = (ColorPoint) o;
         return cp.point.equals(point) && cp.color.equals(color);
    }
    ...  // Remainder omitted
}
```

在Java平台的类库中，也有一些类扩展了可实例化的类并添加了值组件。例如，`java.sql.Timestamp`继承了`java.util.Date`且添加了一个`nanoseconds`属性。`Timestamp`的`equals`实现确实违反了对称性，并且当`Timestamp`和`Date`对象在同一个集合中使用或其他方式混合在一起时，则可能会造成不稳定的行为。`Timestamp`类有一个免责声明，提醒程序员不要混合使用`Date`和`Timestamp`对象。只要你将它们隔离开就不会遇到任何麻烦，但并没有什么方法可以阻止你这么做，而这导致的后果可能会难以调试。因此`Timestamp`类的行为是一个错误，并不值得模仿。

注意，你可以给抽象类的子类添加一个值组件且不会违反`equals`约定。对于遵循第20条建议“类层次(class hierarchies)优于标签类(tagged class)”得到的类层次来说，这一点很重要。例如，你可以有一个抽象的`Shape`类，它不包含任何值组件，一个添加了`radius`属性的`Circle`子类，一个添加了`length`和`width`属性的`Rectangle`子类。由于无法直接创建父类实例，所以上面展示的问题都不会发生。

- ***一致性(Consistency)***—`equals`约定的第四个要求是：如果两个对象相等，那么它们必须一直保持相等，除非其中一个(或者两个)对象被修改了。换而言之，可变对象可以在不同时间等于不同的对象，而不可变对象则不能。当你编写一个类时，请仔细考虑它是否应该不可变(见第15条)。如果你的结论是它应该不可变，那就必须保证你的`equals`方法满足这种限制：所有相等对象一直保持相等，所有不相等对象一直保持不相等。  
无论一个类是否不可变，都不要编写依赖不可靠资源的`equals`方法。如果你违反了这条禁令那么将很难满足一致性。例如，`java.net.URL`的`equals`方法依赖于URL中主机的IP地址比较，将主机名翻译为IP地址需要网络访问，并且并不保证每次都能得到相同的结果。这可能会使得`URL`的`equals`方法违反`equals`约定，并在实践中产生问题(遗憾的是，因为兼容性的要求，这个行为无法被改变)。除了极少数例外的情况，`equals`方法对应该对内存常驻对象执行确定的计算。

- ***非空性(Non-nullity)***—最后一条约定，由于它没有名字，我暂且称之为“非空性”，它要求所有对象都不能等于`null`。虽然很难想象调用`o.equals(null)`时意外地返回`true`，但是并不难想象意外地抛出`NullPointerException`异常。通用约定并不允许抛出异常，有很多类使用了显式的null判断来防止这种情况：  

```java
@Override public boolean equals(Object o) {    if (o == null)        return false;    ...}
```

这种判断是没有必要的。