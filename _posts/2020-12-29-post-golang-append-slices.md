---
title: "Golang append slice详解"
categories:
  - 学习笔记
tags:
  - Golang
  - 语言基础
---

#### 结论

1. golang append函数在append int等基本类型时作为值拷贝append，而数组等类型是引用拷贝。
2. golang在申请数组空间时是以上次申请内存的2倍来申请内存。
3. 数组每次申请额外空间会重新寻找合适的内存地址

#### 测试

```go
// 值拷贝
func valueSilence() {
    a := 123
    b := make([]int, 0)
    fmt.Printf("len(b) cap(b) address  value \n")
    for i := 0; i < 5; i++ {
        a = a + i*9
        b = append(b, a)
        fmt.Printf("%d, %6d, %6p, %d\n", len(b), cap(b), b, b)
    }
    for i := 0; i < len(b); i++ {
        fmt.Printf("%p \n", &b[i])
    }
}

/*
len(b) cap(b) address  value 
1,      1, 0xc0000a61a8, [123]
2,      2, 0xc0000a61c0, [123 132]
3,      4, 0xc0000b4060, [123 132 150]
4,      4, 0xc0000b4060, [123 132 150 177]
5,      8, 0xc0000d6180, [123 132 150 177 213]
0xc0000d6180 
0xc0000d6188 
0xc0000d6190 
0xc0000d6198 
0xc0000d61a0
*/
```

```go
// 引用拷贝
func listSilence() {
    a := []int{1, 2, 3}
    b := make([][]int, 0)
    fmt.Printf("len(b) cap(b) address  value \n")
    // 只是拷贝引用
    for i := 0; i < 5; i++ {
        for j := 0; j < 3; j++ {
            a[j] = a[j] + i*9
        }
        b = append(b, a)
        fmt.Printf("%d, %6d, %6p, %d\n", len(b), cap(b), b, b)
    }

    // 因为a长度不够，所以重新申请一个新的数组，地址改变，数值更改不会影响之前的切片
    a = append(a, 555)
    b = append(b, a)
    fmt.Printf("%d, %6d, %6p, %d\n", len(b), cap(b), b, b)

    for i := 0; i < len(b); i++ {
        fmt.Printf("%p \n", b[i])
    }
}

/*
len(b) cap(b) address  value 
1,      1, 0xc0000b21e0, [[1 2 3]]
2,      2, 0xc000092360, [[10 11 12] [10 11 12]]
3,      4, 0xc0000a2240, [[28 29 30] [28 29 30] [28 29 30]]
4,      4, 0xc0000a2240, [[55 56 57] [55 56 57] [55 56 57] [55 56 57]]
5,      8, 0xc0000e4000, [[91 92 93] [91 92 93] [91 92 93] [91 92 93] [91 92 93]]
6,      8, 0xc0000e4000, [[91 92 93] [91 92 93] [91 92 93] [91 92 93] [91 92 93] [91 92 93 555]]
0xc0000b4080 
0xc0000b4080 
0xc0000b4080 
0xc0000b4080 
0xc0000b4080 
0xc00009e030
*/
```

#### 引用

- [1][The Go Programming Language Specification](https://golang.google.cn/ref/spec#Appending_and_copying_slices)
