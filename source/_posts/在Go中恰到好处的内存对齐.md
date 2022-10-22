---
title: 在 Go 中恰到好处的内存对齐
date: 2022-10-22 11:53:37
categories:
- Golang 编程
- 内存对齐
- 参考文章
tags:
---

# 问题
在开始之前，希望你计算一下 Part1 共占用的大小是多少呢？
```Golang
type Part1 struct {
	a bool
	b int32
	c int8
	d int64
	e byte
}

```
运行如下代码
```Golang
func main() {
	fmt.Printf("bool size: %d\n", unsafe.Sizeof(bool(true)))
	fmt.Printf("int32 size: %d\n", unsafe.Sizeof(int32(0)))
	fmt.Printf("int8 size: %d\n", unsafe.Sizeof(int8(0)))
	fmt.Printf("int64 size: %d\n", unsafe.Sizeof(int64(0)))
	fmt.Printf("byte size: %d\n", unsafe.Sizeof(byte(0)))
	fmt.Printf("string size: %d\n", unsafe.Sizeof("EDDYCJY"))
}
```
输出结果：
```Bash
bool size: 1
int32 size: 4
int8 size: 1
int64 size: 8
byte size: 1
string size: 16
```
这么一算，Part1 这一个结构体的占用内存大小为 1+4+1+8+1 = 15 个字节。相信有的小伙伴是这么算的，看上去也没什么毛病

真实情况是怎么样的呢？我们实际调用看看，如下：
```Golang
type Part1 struct {
	a bool
	b int32
	c int8
	d int64
	e byte
}

func main() {
	part1 := Part1{}

	fmt.Printf("part1 size: %d, align: %d\n", unsafe.Sizeof(part1), unsafe.Alignof(part1))
}
```
输出结果：
```Bash
part1 size: 32, align: 8
```
最终输出为占用 32 个字节。这与前面所预期的结果完全不一样。这充分地说明了先前的计算方式是错误的。为什么呢？

在这里要提到 “内存对齐” 这一概念，才能够用正确的姿势去计算，接下来我们详细的讲讲它是什么

# 内存对齐
有的小伙伴可能会认为内存读取，就是一个简单的字节数组摆放

{% asset_img 微信图片_20221022120129.png  %}

上图表示一个坑一个萝卜的内存读取方式。但实际上 CPU 并不会以一个一个字节去读取和写入内存。相反 CPU 读取内存是一块一块读取的，块的大小可以为 2、4、6、8、16 字节等大小。块大小我们称其为内存访问粒度。如下图：

在样例中，假设访问粒度为 4。CPU 是以每 4 个字节大小的访问粒度去读取和写入内存的。这才是正确的姿势

## 为什么要关心对齐
- 你正在编写的代码在性能（CPU、Memory）方面有一定的要求

- 你正在处理向量方面的指令

- 某些硬件平台（ARM）体系不支持未对齐的内存访问

另外作为一个工程师，你也很有必要学习这块知识点哦 :)

## 为什么要做对齐
- 平台（移植性）原因：不是所有的硬件平台都能够访问任意地址上的任意数据。例如：特定的硬件平台只允许在特定地址获取特定类型的数据，否则会导致异常情况

- 性能原因：若访问未对齐的内存，将会导致 CPU 进行两次内存访问，并且要花费额外的时钟周期来处理对齐及运算。而本身就对齐的内存仅需要一次访问就可以完成读取动作

在上图中，假设从 Index 1 开始读取，将会出现很崩溃的问题。因为它的内存访问边界是不对齐的。因此 CPU 会做一些额外的处理工作。如下：

1. CPU 首次读取未对齐地址的第一个内存块，读取 0-3 字节。并移除不需要的字节 0

2. CPU 再次读取未对齐地址的第二个内存块，读取 4-7 字节。并移除不需要的字节 5、6、7 字节

3. 合并 1-4 字节的数据

4. 合并后放入寄存器

从上述流程可得出，不做 “内存对齐” 是一件有点 “麻烦” 的事。因为它会增加许多耗费时间的动作

而假设做了内存对齐，从 Index 0 开始读取 4 个字节，只需要读取一次，也不需要额外的运算。这显然高效很多，是标准的空间换时间做法

在不同平台上的编译器都有自己默认的 “对齐系数”，可通过预编译命令`#pragma pack(n)`进行变更，n 就是代指 “对齐系数”。一般来讲，我们常用的平台的系数如下：

