# copy函数的坑

这段代码，s1切片的输出结果始终为`[]`

```go
package main

import (
    "fmt"
)

func main() {
    s1 := make([]int, 0, 3) // 创建容量为3的空切片
    fmt.Println("s1:", s1)

    s2 := []int{1, 2, 3, 4, 5} // 创建一个长度为5的切片
    copy(s1, s2) // 复制s2到s1
    fmt.Println("s1 after copy:", s1)

    s3 := []int{1, 2} // 创建一个长度为2的切片
    copy(s1, s3) // 复制s3到s1
    fmt.Println("s1 after copy:", s1)
}

```

> The copy built-in function copies elements from a source slice into a destination slice. (As a special case, it also will copy bytes from a string to a slice of bytes.) The source and destination may overlap. Copy returns the number of elements copied, ==which will be the minimum of len(src) and len(dst).==

向切片copy时，需要保证切片有足够的`len`，而不止是`cap`

