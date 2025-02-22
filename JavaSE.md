## 1. 接口、普通类和抽象类有什么异同

- 普通类用于描述具体的对象
- 抽象类用于定义具有共性的基类
- 接口则用于定义行为规范

### 定义上的区别

- 普通类是完完整整的类，可以实例化对象
- 抽象类不能实例化对象，一般用作基类，包括抽象方法（未实现的方法）和具体方法（已实现的方法）
- 接口则是完完全全的抽象结构，只包含抽象方法

### 继承关系上的区别

- 普通类单继承，只能有一个父类
- 抽象类也是单继承，一个类只能继承一个抽象类
- 接口支持多实现，一个类可以实现多个接口

### 成员变量上的区别

- 普通类和抽象类都可以有各种类型的成员变量（实例变量，静态变量）
- 接口只能有各种常量



## 2. 深拷贝与浅拷贝

- 浅拷贝是指：新建一个对象，但是该对象的字段仍指向原对象的内存地址，修改新对象会影响原对象的值。
- 深拷贝则是完全的新对象，不会影响到原对象

深拷贝

```java
// 需要手动深拷贝引用类型
class Person implements Cloneable {
    String name;
    Address address;

    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone();
        cloned.address = (Address) this.address.clone(); // 手动深拷贝引用类型
        return cloned;
    }
}

class Address implements Cloneable {
    String city;

    public Address(String city) {
        this.city = city;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Address address = new Address("New York");
        Person person1 = new Person("Alice", address);
        Person person2 = (Person) person1.clone();

        System.out.println(person1.address.city); // 输出: New York
        person2.address.city = "Los Angeles";
        System.out.println(person1.address.city); // 输出: New York
    }
}
```

浅拷贝

```java
class Person implements Cloneable {
    String name;
    Address address;

    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone(); // 默认是浅拷贝
    }
}

class Address {
    String city;

    public Address(String city) {
        this.city = city;
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Address address = new Address("New York");
        Person person1 = new Person("Alice", address);
        Person person2 = (Person) person1.clone();

        System.out.println(person1.address.city); // 输出: New York
        person2.address.city = "Los Angeles";
        System.out.println(person1.address.city); // 输出: Los Angeles
    }
}
```

## 3. 自动拆箱 / 装箱

- 自动装箱是指将基本数据类型（如 int、double、boolean 等）自动转换为对应的包装类对象（如 Integer、Double、Boolean 等）。这个过程由编译器自动完成，无需手动调用包装类的构造方法或静态方法。
  - 获取包装类对象：使用包装类的 `valueOf()` 方法，如 `Integer num = Integer.valueOf(123);`

- 自动拆箱是指将包装类对象（如 Integer、Double、Boolean 等）自动转换为对应的基本数据类型（如 int、double、boolean 等）。同样，这个过程也是由编译器自动完成的。

  - 当需要基本数据类型是，可使用包装类的 `xxxValue() 方法获取对应值`

  - ```java
    Integer num = new Integer(1);
    double v = num.doubleValue();
    ```

### 注意事项

1. 性能问题，频繁的自动装箱和拆箱可能会导致额外的性能开销，因为每次都需要创建或转换对象。
   - 比如 Integer 的缓存范围为 `-128 - 127`，超过这个范围会 创建新对象
2. 空指针异常，如果对一个 null 的包装类对象进行自动拆箱操作，会抛出 `NullPointerException`。
3. 缓存机制，某些包装类（如 Integer、Boolean 等）会对常用值进行缓存。



## 4. 重载与重写的区别

### 发生位置不同

- 重载发生在同一个类中
- 重写发生在父子类之间

### 方法签名不同

- 重载要求方法名相同，参数不同
- 重写要求全部相同

### 返回值类型不同

- 重载返回值类型可以不同
- 重写返回值类型必须和父类方法完全相同

### 访问修饰符不同

- 重载对访问修饰符没有限制
- 重写的访问修饰符，子类不能比父类更严格

### 异常声明不同

- 重载对异常声明没有限制
- 重写子类方法抛出的异常不能比父类方法抛出的异常范围更大

### 绑定关系不同

- 重载是静态绑定，编译时确定调用哪个方法
- 重写是动态绑定，运行时根据对象的实际类型决定调用哪个方法



## 5. == 和 equals() 的区别

### 比较内容

- == 比较的是内存地址（引用类型）或实际值（基本数据类型）
- 而 equals() 比较的是逻辑上的相等性，具体取决于 类是否重写的 equals() 方法

### 适用范围

- == 可用于基本数据类型和引用数据类型
- equals 只能用于引用数据类型

### 默认行为

- == 始终比较的是内存地址或实际值
- equals 在未重写时与 == 行为一致，但在某些类中（String、Integer 等）重写了，是内容比较

### 可扩展性

- == 无法修改或扩展
- equals 可以自定义比较逻辑

### 性能

- == 性能更高，因为直接标价哦内存地址或值
- equals 性能可能较低，尤其是复杂对象中需要逐个比较属性值



## 6. 泛型

> 反省是一种在**编译时**提供的安全检查机制，将**类型**作为参数传递给类、接口、方法，从而避免硬编码具体的类型。

作用：

1. 提高代码复用性，允许编写与类型无关的通用代码
2. 增强类型安全性，比如集合默认存储的时 Object，取出元素需要手动转换，容易引发 `ClassCastException`，而泛型在编译时就会进行类型检查，避免运行时的类型错误。
3. 简化代码，无需显式地进行类型转换，减少冗余代码，提高代码可读性。
4. 支持复杂的类型约束，可通过通配符 (? extends T 和 ? super T) 实现更复杂的类型限制

## 7. 反射

反射是一种在运行时动态获取类信息的能力，通过反射，我们可以在程序运行时动态地获取类的信息并操作类的属性、方法和构造器。甚至可以调用类的方法或修改字段值。

应用场景：

1. 框架开发
2. 动态代理，AOP
3. 注解处理
4. 插件化开发
5. 测试工具



## 8. StringBuffer

> StringBuffer 是 Java 中用于处理可变字符串的重要类，在多线程环境中表现出色，能够高效地进行字符串拼接和修改操作。与 String 不同，StringBuffer 的内容是可以被修改的，核心特点是**线程安全、搞笑的字符串操作**

特点：

1. 具有可变性，可以在原有对象上直接修改字符串内容，无需创建新的对象
2. 线程安全，StringBuffer 所有方法都通过 `synchronized` 关键字修饰，因此他是线程安全的。
3. 性能相对较好，StringBuffer 内部使用一个可扩容的字符数据来存储数据，当容量不足时会自动扩展。相比于 String 的不可变性(每次修改都会产生新对象)，StringBuffer 在频繁修改字符串时性能更高。而相比于非线程安全的**StringBuilder**，性能略低。
4. 包含丰富的 API，如：`append()`： 追加内容至字符串末尾。`insert*()`：在指定位置插入内容。`delete()`：删除指定范围的内容。`reverse()`：反转字符串内容。`toString()`：将 StringBuffer 转为 String。
