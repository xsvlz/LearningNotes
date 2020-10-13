# Java比较两个浮点数

> 浮点数的基本数据类型不能用`==`比较，包装数据类型不能用 `equals` 比较

## 浮点数的表示

在计算机系统中，浮点数采用 **符号+阶码+尾数** 进行表示。在Java中，单精度浮点数`float`类型占32位，它的二进制表示方式为：
- 符号位：1位，0表示正数； 1表示负数
- 指数位：8位，用来表示指数（要加上偏移量）
- 小数位：23位，用来表示小数

实际上计算机中的浮点数表示方法和科学技术法类似，小数的位数决定了浮点数的精度。当一个十进制的小数转换成二进制时，很有可能无法使用这么多小数位表示它。因此使用浮点数的时候，实际存储的尾数是被截取或者舍入之后的值。因此使用浮点数进行计算的时候就不得不考虑精度问题，即浮点数无法精确计算。

## 如何比较浮点数

### 1. 不能使用 `==`

对于浮点数，我们无法使用`==`或者包装类型的`equals()`来精确比较。例如:
```java
// 对f1执行11次加0.1的操作
float f1 = 0.0f;
for (int i = 0; i < 11; i++) {
    f1 += 0.1f;
}
// f2是0.1*11的结果
float f2 = 0.1f * 11;
System.out.println(f1 == f2);   // 结果为false
```
执行上述代码，会得到二者不相等，同时如果打印f1和f2，会得到如下结果:
```java
f1 = 1.1000001
f2 = 1.1
```

可以看到，在浮点数的运算过程中，确实出现了精度问题。

### 2. 规定误差范围

尽管无法做到精确比较，但是我们可以确定一个误差范围，当小于这个误差范围的时候就认为这两个浮点数相等。例如：
```java
final float THRESHOLD = 0.000001;   // 设置最大误差不超过0.000001
float f1 = 0.0f;
for (int i = 0; i < 11; i++) {
    f1 += 0.1f;
}

float f2 = 0.1f * 11;

if (Math.abs(f1 - f2) < THRESHOLD) {
    System.out.println("f1 equals f2");
}
```

### 3. 使用BigDecimal

`BigDecimal`是一个不可变的、能够表示大的十进制整数的对象。注意使用`BigDecimal`时，要使用参数为`String`的构造方法，而不要使用参数为`double`的构造方法，防止产生精度丢失。使用`BigDecimal`进行运算，使用它的`compareTo()`方法比较即可。
示例：
```java
private void compareByBigDecimal() {
    BigDecimal f1 = new BigDecimal("0.0");
    BigDecimal pointOne = new BigDecimal("0.1");
    for (int i = 0; i < 11; i++) {
        f1 = f1.add(pointOne);
    }

    BigDecimal f2 = new BigDecimal("0.1");
    BigDecimal eleven = new BigDecimal("11");
    f2 = f2.multiply(eleven);

    System.out.println("f1 = " + f1);
    System.out.println("f2 = " + f2);

    if (f1.compareTo(f2) == 0) {
        System.out.println("f1 and f2 are equal using BigDecimal");
    } else {
        System.out.println("f1 and f2 are not equal using BigDecimal");
    }
}
```

---
[参考链接](https://isuperqiang.cn/post/java-bi-jiao-fu-dian-shu-de-zheng-que-fang-shi/)