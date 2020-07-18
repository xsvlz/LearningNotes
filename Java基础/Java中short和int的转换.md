# Java中short和int的转换

## 例子[^1]：

[^1]: 参考https://blog.csdn.net/allenjay11/article/details/78613862

第一种情况：
```java
short a = 1;
a = a + 1;    // 这一步会报错
System.out.print(a);
```

编译器会报错，原因如下:

![-w600](http://img.longzhuang.top/20200716164415.png)

第二种情况：
```java
short a = 1;
a += 1;
System.out.print(a);
```

这种情况不会报错。
java语言规范中关于复合赋值的解释是这样的：`E1 op= E2`等价于 
`E1=(T)(E1 op E2)`,这里的T是E1的数据类型，即**复合赋值是自带了隐式的强制类型转换的**。

第三种情况：
```java
short a = 1;
short b = 1;
short c = a + b;
```

这种情况依然会编译出错，因为Java中存在的**类型升级**，导致两个`short`类型的运算也会转换成`int`进行。

## 类型升级

在Java中，对基本数据类型做比较或者运算时，如果两边的数据类型不同，在可以比较的前提下会首先进行**类型升级**：

- 如果任一方为`double`，则另一方转换为`double`
    - 否则如果任一方为`float`，则另一方转换为`float`
        - 否则如果任一方为`long`，则另一方转换为`long`
            - 否则两边都会转换为`int`

即从高到低分别为`double`, `float`, `long`, `int`
即使是两个`short`类型运算，也会转换成`int`进行，这就是前面第三种情况出现错误的原因。

