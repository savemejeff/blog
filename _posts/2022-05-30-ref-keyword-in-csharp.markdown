---
layout: post
title:  "C#中的ref关键字"
date:   2022-05-30 12:19:00 +0800
categories: Jiang Jingwei
---
首先看微软的文档中是怎么说的。

> `ref` 关键字指示按引用传递值。 它用在四种不同的上下文中：

那么什么叫按引用传递值呢？

在C#中，所有类型的变量（无论值类型还是引用类型），都是按值传递的。让我们先看看按值传递是怎么工作的。

```csharp
int a = 10;
int b = a;
b = 20;
Console.WriteLine($"a: {a}");
Console.WriteLine($"b: {b}");
// Output: 
// a: 10
// b: 20
```
值传递时，改变b的值并不会影响a。

下面看看按引用传递是如何工作的。

```csharp
int a = 10;
ref int b = ref a;
b = 20;
Console.WriteLine($"a: {a}");
Console.WriteLine($"b: {b}");
// Output: 
// a: 20
// b: 20
```
按引用传递时，对b的赋值会影响变量a，因为b是一个指向变量a的引用。

学过C语言的同学是不是有种似曾相识的感觉？

```C
int a = 10;
int* b = &a;
*b = 20;
printf("a: %d", a);
printf("b: %d", *b);
// Output:
// a: 20
// b: 20
```
虽然不是相同的概念，但是ref的功能其实与C语言中的指针十分相似。

当然，使用`ref`关键字去引用在同一个作用域中的变量是没有任何意义的。

```csharp
int a = 10;
// ref int b = ref a;
// b = 20; 
a = 20; // 你应该这样写。
```

正确的使用方式如下所示：

> 在方法签名和方法调用中，按引用将参数传递给方法。

```csharp
void Method(ref int num)
{
    n += 1;
}

int n = 10;
Method(ref n);
// 执行Method方法后，能够改变实参n的值。
```

> 在方法签名中，按引用将值返回给调用方。

```csharp
ref SomeStruct GetStruct()
{
    SomeStruct s = new SomeStruct();
    return ref s;
}

ref var struct = ref GetStruct();
// 按引用将值返回一个复杂的结构体，能够有效减小开销。
```
下面这两个用法还没看明白，等看明白了再补一个。

> 在成员正文中，指示引用返回值是否作为调用方欲修改的引用被存储在本地。 或指示局部变量按引用访问另一个值。

> 在 `struct` 声明中，声明 `ref struct` 或 `readonly ref struct`。

另外还有两个关键字 `in` 和 `out` ，与 `ref` 比较相似。

不同之处在于 `in` 只能向被调用的方法传递引用，而在被调用的方法中，被`in`修饰的参数是只读的。而 `out` 只能从被调用的方法向方法的调用方传递引用。

```csharp
int n = 10;
void InMethod(in int num)
{
    num = 1;
}
InMethod(in n);
// error CS8331: Cannot assign to variable 'in int' because it is a readonly variable
// in 参数被视为只读。

void OutMethod(out int num)
{
    num++;
}
OutMethod(out n);
// error CS0269: Use of unassigned out parameter 'num'
// out 参数被视为未初始化的变量。
```