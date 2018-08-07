静态工厂方法和构造器都有个限制：它们无法扩展到大量的可选参数。思考一下这样的一个类，它表示包装食物上面的营养成分标签。这些标签有一些必须的属性：每份含量，每罐含量和每份的卡路里，另外还有超过20个可选的属性：总脂肪量，饱和脂肪量，转化脂肪，胆固醇，纳等等。大部分产品只在一小部分可选属性上有非零值。

对于这种类，你应该编写那种类型的构造器或静态工厂呢？通常，程序员们习惯使用折叠构造器(telescoping constructor)模式，这种模式你可以提供一个只接收必选参数的构造器，再提供一个接收单一可选参数的构造器，再提供一个接收两个可选参数的构造器，以此类推，最后一个构造器接收所有可选参数。下面是实际代码看起来的样子，为了简单起见，只显示4个可选属性：

```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts { 
    private final int servingSize;   // (mL)            必选
    private final int servings;      // (per container) 必选
    private final int calories;      //                 可选
    private final int fat;           // (g)             可选
    private final int sodium;        // (mg)            可选
    privatefinalintcarbohydrate;     // (g)             可选

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
        int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

当你想创建实例时，你选用包含所有你需要参数的最短参数列表构造器：

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

通常这种构造器调用需要很多你并不想设置的参数，但是无论如何你都被迫传入这些值。在上面的例子中，我们给`fat`参数传了一个值为0。如果“只有”6个参数的话，这看起来没有那么糟，但随着参数数量的增加它很快就会变得不可控。

简而言之，**折叠构造器模式可以正常运行，但是当有许多参数时，客户端代码会很难编写，更别说阅读它了。**代码阅读者能做的就是了解所有这些值代表什么，并且还要仔细地数它们的位置才能知道。相同类型的长参数列表会导致很微妙的bug，如果客户不小心调换了两个这样的参数，编译器是不会报错的，但是程序在运行时无法满足预期行为。(见第40条)

面对大量构造器参数时，第二种替代方案是`JavaBean`模式，这种模式下你可以调用一个无参构造器来创建对象，然后调用setter方法来设置每个必要的参数和感兴趣的可选参数：  

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
    // 参数初始化为默认值 (如果有的话)
    private int servingSize = -1; // 必选；无默认值
    private int servings    = -1; // 必选；无默认值
    private int calories
    private int fat
    private int sodium
    private int carbohydrate = 0;

    public NutritionFacts() { }

    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}

```

这种模式没有折叠构造器模式的任何缺点，它创建对象非常简单，但稍有一点冗长，并且产生的代码也很容易阅读：  

```java
NutritionFacts cocaCola = new NutritionFacts();cocaCola.setServingSize(240);cocaCola.setServings(8);cocaCola.setCalories(100);cocaCola.setSodium(35);cocaCola.setCarbohydrate(27);
```

不幸的是，JavaBean模式有它自身的严重缺点。因为构造过程被分到了多个调用中，**一个JavaBean也许会在构造过程中处于一个不一致的状态。**类无法仅仅通过检查构造器参数的有效性来保证一致性。当一个对象处于不一致状态时，企图使用它可能会导致失败，并且这种失败与包含错误的代码远远不同，因此非常难以调试。还有一个相关的缺点是**JavaBean模式阻止了把类变为不可变类的可能性**(见第15条)，并且要求程序员付出额外的努力去保证线程安全。

你可以通过在构造完成时手工地“冻结”对象并且在冻结之前不允许使用的方式来弥补这些缺点，但是这种方式太笨重且在实践中很少使用。此外，它还可能在运行时产生错误，因为编译器没法保证程序员在使用对象之前调用它的冻结方法。 

幸运的是，还有第三种可选方案，它结合了折叠构造器模式的安全性和JavaBean模式的可读性，这就是Builder模式[Gamma95, p. 97]。客户不再直接构造对象，而是使用所有必选参数调用构造器（或静态工厂）得到一个builder对象，然后客户在builder对象上调用类似setter的方法来设置每个他感兴趣的可选参数，最后客户调用一个无参数的`build`方法来创建对象，这个对象是不可变的。builder是创建对象类型的一个静态成员类，下面是它的示例：  

```java
// Builder模式
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 必选参数
        private final int servingSize;
        private final int servings;

        // 可选参数 - 初始化为默认值
        private int calories      = 0;
        private int fat           = 0;
        private int carbohydrate  = 0;
        private int sodium        = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        } 
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    } 
}
```

注意`NutritionFacts`是不可变的，并且所有参数的默认值都放在同一个地方。builder的setter方法返回它本身，所以调用可以是链式的。下面是示例代码：  

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).     calories(100).sodium(35).carbohydrate(27).build();
```

这个客户端代码非常容易编写，更重要的是容易阅读。**Builder模式了命名的可选参数**，就像Ada和Python一样。

像构造器一样，builder可以给它的参数强加约束条件。build方法可以检查这些约束条件，这是非常重要的，从builder拷贝参数到对象中以后可以被检查是非常重要的，并且它们可以在对象的属性上而不是builder的属性上被检查(见第39条)。如果违反了任何约束条件，`build`方法应该抛出一个`IllegalStateException`异常(见第60条)，且异常的明细应该指出它违反了哪个约束(见第63条)。

另一种对多个参数强加约束条件的方法是让setter方法接收一组参数，组里包含所有要强加约束的所有参数。如果不满足约束，setter方法就抛出一个`IllegalArgumentException`异常。这种方式的优点是当不合法参数被传递时能尽早地检测到，而不需要等待`build`方法被调用。

builder对比构造器还有一个小优点，它可以拥有多个可变参数。构造器跟方法一样只能有一个可变参数，而因为builder使用了不同的方法来设置每个参数，所以它可以拥有任意个数的可变参数，但最多是每个setter方法一个。

Builder模式是很灵活的，单个builder可以用来创建多个对象。builder的参数可以在对象创建时调整，也可以随不同的对象而调整。builder还可以自动地填充某些属性的值，例如每次对象创建时自动递增的序列号。

设置了参数的builder变成了一个很好的抽象工厂(Abstract Factory)[Gamma95, p. 87]。换而言之，客户可以给方法传递一个这样的builder，并且允许方法为客户创建一个或多个对象。想要使用这种用法的话你需要一个类型来表示builder，如果你正在使用JDK1.5或之后的版本，只要一个单一泛型就能满足所有的builder了，无需关心它们正在创建的类型：  

```java
// A builder for objects of type Tpublic interface Builder<T> {    public T build();}
```

注意，我们的`NutritionFacts.Builder`类可以声明为实现了`Builder<NutritionFacts>`接口。

