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

