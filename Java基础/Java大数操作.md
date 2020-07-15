# Java大数操作

Java的Math包中提供了两个类用于对大数进行操作：
- `BigInteger`类，用于大整数的操作
- `BigDecimal`类，用于大的小数操作

## BigInteger类

Java中的基本类型中，表示整数的有`short`, `int`和`long`，其中最大的`long`也只有64位，能表示的最大正整数是`9223372036854775807`，如果超过了这个数就无法使用基本数据类型表示了。

而`BigInteger`类把数据以字符串的形式写入即计算。并且由于使用它需要创建对象，而且Java中没有提供运算符的重载，所以不能直接使用`+ - * /`这些运算符直接运算。需要进行运算操作时，可以使用该类提供的方法进行。
`BigInteger`类的二元运算需要两个操作数都为`BigInteger`，通常前一个操作数调用方法，后一个操作数作为参数。常用的方法如下：

```java
public BigInteger(String val); // 构造方法，将一个字符串变为 BigInteger类型的数据
public BigInteger add(BigInteger val)       // 加法，结果用BigInteger类型返回
public BigInteger subtract(BigInteger val)  // 减法
public BigInteger multiply(BigInteger val)  // 乘法
public BigInteger divide(BigInteger val)    // 除法
public BigInteger max(BigInteger val)       // 返回最大值
public BigInteger min(BigInteger val)       // 返回最小值

// 带余数的除法操作，数组的第一个元素表示除法的商，第二个元素表示除法的余数
public BigInteger[] divideAndRemainder(BigInteger val) 
```

示例

```java
import java.math.BigInteger;

public class Test {
    public static void main(String[] args) {
        // 用来保存 两个大数
        BigInteger b1 = new BigInteger("987654321098765432109876543210");
        BigInteger b2 = new BigInteger("123456789012345678901234567890");

        // 两个大数的运算（加减乘除、最大值、最小值）
        System.out.println("b1 + b2 = " + b1.add(b2));        // 加
        System.out.println("b1 - b2 = " + b1.subtract(b2));   // 减
        System.out.println("b1 * b2 = " + b1.multiply(b2));   // 乘
        System.out.println("b1 / b2 = " + b1.divide(b2));     // 除
        System.out.println("max: " + b1.max(b2));        // 最大值
        System.out.println("min: " + b1.min(b2));        // 最小值
        System.out.println();

        // 除法操作，数组的第一个元素是除法的商，第二个元素是除法的余数
        BigInteger[] bArr = b1.divideAndRemainder(b2);
        System.out.println("商：" + bArr[0]);
        System.out.println("余数：" + bArr[1]);
    }
}
```

## BigDecimal类

`BigDecimal`是为了解决小数的精度问题而使用的。它也是使用字符串进行写入。
`BigDecimal`类的常用方法与`BigInteger`类相同，只不过操作数变成了`BigDecimal`类。

示例：

```java
import java.math.BigDecimal;

public class Test {
    public static void main(String[] args) {
        // 用来保存 两个小数
        BigDecimal b1 = new BigDecimal("1.00");
        BigDecimal b2 = new BigDecimal("0.30");

        // 两个小数的运算（加减乘除、最大值、最小值）
        System.out.println("b1 + b2 = " + b1.add(b2));        // 加
        System.out.println("b1 - b2 = " + b1.subtract(b2));   // 减
        System.out.println("b1 * b2 = " + b1.multiply(b2));   // 乘
        // 除法，开始四舍五入模式，否则可能会报 ArithmeticException
        System.out.println("b1 / b2 = " + b1.divide(b2, BigDecimal.ROUND_HALF_UP));
        System.out.println("max: " + b1.max(b2));        // 最大值
        System.out.println("min: " + b1.min(b2));        // 最小值
        System.out.println();

        // 除法操作，数组的第一个元素是除法的商，第二个元素是除法的余数
        BigDecimal[] bArr = b1.divideAndRemainder(b2);
        System.out.println("商：" + bArr[0]);
        System.out.println("余数：" + bArr[1]);
    }
}
```

---
[参考链接: CSDN 小异常](https://blog.csdn.net/sun8112133/article/details/81291750)