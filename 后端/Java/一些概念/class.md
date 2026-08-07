## 类中类

### 1. 成员内部类（Member Inner Class）

最"正统"的内部类，**持有外部类实例的引用**。
```java
public class Outer {
    private int x = 10;
    
    // 非 static → 绑定外部类实例
    class Inner {
        void print() {
            System.out.println(x); // ✅ 直接访问外部私有成员
        }
    }
}

// ⚠️ 创建时必须先有外部类实例
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```
> 本质：编译器会生成 Outer$Inner.class，并自动注入一个 this$0 引用指向外部实例。
> 类比：类似 TS 中闭包捕获了外层 this，但 Java 是语言层面显式支持的。

### 2. 静态嵌套类（Static Nested Class）
用 `static` 修饰，**不持有外部类实例引用**。
```java
public class Outer {
    private static int count = 0;
    
    // static → 不依赖外部实例
    static class Nested {
        void print() {
            System.out.println(count); // ✅ 只能访问外部 static 成员
            // System.out.println(x);  // ❌ 不能访问非 static 成员
        }
    }
}

// ✅ 不需要外部类实例
Outer.Nested nested = new Outer.Nested();
```
> **这是最常用的内部类形式**，本质上就是一个放在另一个类命名空间里的普通类。  
> **类比**：完全等价于 NestJS 中在同一文件里定义但不导出的辅助类，只是多了命名空间的组织作用。

### 3. 局部内部类（Local Inner Class）

定义在**方法内部**，作用域仅限该方法。
```java
public class Outer {
    void method() {
        final String msg = "hello"; // 必须是 effectively final
        
        class LocalClass {
            void say() {
                System.out.println(msg); // ✅ 可访问方法的 final 变量
            }
        }
        
        new LocalClass().say();
    }
}
```
> **限制**：不能有访问修饰符（public/private），不能用 static。  
> **实际使用极少**，几乎都被 Lambda / 匿名类替代了。

| 特性            | 成员内部类         | 静态嵌套类          | 局部内部类                 | 匿名内部类     |
| :------------ | :------------ | :------------- | :-------------------- | :-------- |
| `static`      | ❌             | ✅              | ❌                     | ❌         |
| 访问外部非static成员 | ✅             | ❌              | ✅ (final)             | ✅ (final) |
| 需要外部实例创建      | ✅             | ❌              | ❌                     | ✅         |
| 可有访问修饰符       | ✅             | ✅              | ❌                     | ❌         |
| 生成class文件     | `Outer$Inner` | `Outer$Nested` | `Outer$1Method$Local` | `Outer$1` |
| 实际使用频率        | ⭐⭐            | ⭐⭐⭐⭐⭐          | ⭐                     | ⭐⭐⭐       |