虽然`Object`是一个具体的类，但它主要是为扩展而设计的。它的所有非final方法(`equals`, `hashCode`, `toString`, `clone`和`finalize`)都有显式的通用约定(general contracts)，因为它们被设计用于覆盖。任何覆盖了这些方法的类都有责任遵循这些通用约定，如果没有遵循，那么其他依赖这些约定的类(例如`HashMap`和`HashSet`)就无法与这些类在功能上正确地结合起来。

本章将向你讲述应该何时及如何覆盖这些非final的`Object`方法。本章会忽略`finalize`方法，因为它已经在第7条讨论过了。`Comparable.compareTo`方法虽然不是`Object`的方法，但是也会放在本章讨论，因为它有相似的特性。

