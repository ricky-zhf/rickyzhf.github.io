---
title: "go栈内存"
date: 2023-02-11
lastmod: 2023-02-12
author: "ricky"
summary: "go栈内存" #摘要
tags: ["go"]
categories: ["go"]
lightgallery: false
autonumber: true
#featuredImage: "/img/" #文章头+预览
#featuredImagePreview: "/img/" #预览
---



应用程序中的内存一般会分为堆和栈两个部分，程序要运行期间可以主动向堆内存申请内存空间，这些内存由内存分配器分配并由垃圾回收期回收。<mark>而栈内存一般由编译器自动分配和释放</mark>。其中主要存储函数局部变量、入参、返回值等数据，这些数据一般会随着函数一起创建和销毁。

## 栈结构

传统的程序虚拟空间分布如下：

<img src="https://img-blog.csdnimg.cn/img_convert/b4f882b9447760ce5321de109276ec23.png" alt="虚拟内存空间划分" style="zoom: 67%;" />

对于传统程序而言，程序计数器的缺省值为0，当运行完当前指令则加1，因此代码段数据段文件自然从地地址向搞高地址进行分配。heap从低地址向高地址扩展，是因为做内存管理相对要简单些。

而为了避免栈空间和堆空间、代码段等冲突，并最大利用地址空间，就会选择把栈底设置在高地址区间，然后让栈向下增长。这样栈底地址是固定的（高地址），只需要移动栈顶指针即可实现栈的扩展。如果将栈从地址开始分配，则栈底地址并不固定（可能面临堆空间快扩容的场景），此时就可能需要将整个栈在内存中移动。<mark>因此对于栈而言，是由高地址向低地址进行分配。</mark>

<mark>所以对于程序空间中的栈而言，是一个倒着的栈，栈基在上面（内存高地址），栈顶在下面（内存地地址）。</mark>

## Go栈内存空间

对于Go程序中的栈而言，也是由高地址向低地址进行分配，由一个个栈桢组成，当调用某个函数时，则将一个栈帧由栈顶压入栈。当被调函数执行完毕后，会则撤销当前函数对应的栈桢，也即出栈。对于栈桢而言，有两个包含 BP 和 SP 两个栈寄存器，它们分别存储了栈基地址和栈顶地址，两个地址之间的内容就是一个栈桢。

<img src="https://cdn.jsdelivr.net/gh/ricky-zhf/images/img/202303111455168.png" alt="stack-registers" style="zoom: 67%;" />



## 栈桢

Go 语言中函数栈帧布局由高到低（栈底到栈顶），以下面的代码为例，当程序运行到t函数第2行时如下：

1. 调用方的栈基地址，用于函数返回后获得调用函数的栈帧基地址：main函数的栈基地址。
2. 局部变量：a1， b1；
3. 被调函数返回值：res1，res2；
4. 被调用函数参数：a1，b1；
5. 返回地址，保存被调用函数返回后的程序地址，即本函数调用被调用函数的下一条指令地址：res1++指令的地址。
6. tt的栈桢：就是重复上面5个部分了。

```go
func t(a, b int) (int, int) {
  a1, b1 := a, b
  res1, res2:= tt(a1, b1)
  res1++
	return res1, res2
}

func tt(a, b int) (int, int) {
  res1, res2 := a + b, a - b
	return res1, res2
}

func main() {
  i, j := t(1, 2)
  fmt.Println(i, j)
}
```

