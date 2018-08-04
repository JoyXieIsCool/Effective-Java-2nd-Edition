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
