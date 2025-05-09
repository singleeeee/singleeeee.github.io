---
title: TypeScript类型体操
categories: Typescript
---
# TypeScript类型体操

>  TypeScript是一个带有类型的JavaScript

- 明确了类型以后，那自然可以想到类型和所做的操作要匹配才行，这就是为什么要做类型检查，**如果能保证对某种类型只做该类型允许的操作，这就叫做`类型安全`**，所以**类型检查是为了保证类型安全的**。

- 类型检查可以在运行时做，也可以运行之前的编译期做。这是两种不同的类型，前者叫做动态类型检查，后者叫做静态类型检查。

- `动态类型检查` 在源码中不保留类型信息，对某个变量赋什么值、做什么操作都是允许的，写代码很灵活。只有代码运行时才会出现报错，具有很大的安全隐患
- `静态类型检查`则是在源码中保留类型信息，声明变量要指定类型，对变量做的操作要和类型匹配，会有专门的编译器在编译期间做检查。

- **动态类型只适合简单的场景，对于大项目却不太合适，因为代码中可能藏着的隐患太多了**
- **静态类型虽然会增加写代码的成本，但是却能更好的保证代码的健壮性，减少 Bug 率**
- **大型项目注定会用静态类型语言开发。**
- **JavaScript** 是一门动态类型语言，**TypeScript** 是一门静态类型语言



## 为什么 TypeScript **类型编程**被叫做类型体操



- `类型系统`不止 TypeScript 有，别的语言 Java、C++ 等都有，为什么 TypeScript 的类型编程被叫做类型体操，而其他语言没有呢？

- TypeScript 给 JavaScript 增加了一套**静态类型系统**，通过 **TS Compiler 编译为 J**S，**编译的过程做类型检查**。



静态类型编程语言都有自己的类型系统，从简单到复杂可以分为 3 类：

1. **简单类型系统**

> 变量、函数、类等都可以声明类型，编译器会基于声明的类型做类型检查，类型不匹配时会报错。

但是这种系统比较死板

比如一个 add 函数既可以做整数加法、又可以做浮点数加法，却需要声明两个函数：

``` c
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}
```

2. **支持泛型的类型系统**

>  泛型的英文是 Generic Type，通用的类型，它可以代表任何一种类型，也叫做`类型参数`。

**声明时把会变化的类型声明成泛型（也就是类型参数），在调用的时候再确定类型。**

比如上面的 add 函数，有了泛型之后就可以这样写：

```c++
T add<T>(T a, T b) {
    return a + b;
}

add(1,2);
add(1.111, 2.2222);
```

3. **支持类型编程的类型系统**

> 对传入的类型参数（泛型）做各种`逻辑运算`，产生新的类型，这就是类型编程。

在 Java 里面，拿到了对象的类型就能找到它的类，进一步拿到各种信息，所以类型系统支持泛型就足够了。

但是在 JavaScript 里面，对象可以字面量的方式创建，还可以灵活的增删属性，拿到对象并不能确定什么，所以要支持对传入的类型参数做进一步的处理。

例如一个获取对象身上某个属性的值的函数。

``` typescript
function getPropValue<
    T extends object,
    Key extends keyof T
>(obj: T, key: Key): T[Key] {
    return obj[key];
}
```



**TypeScript 的类型系统是`图灵完备`的，也就是能描述各种可计算逻辑。简单点来理解就是循环、条件等各种 JS 里面有的语法它都有，JS 能写的逻辑它都能写。**

对类型参数的编程是 TypeScript 类型系统最强大的部分，可以实现各种复杂的类型计算逻辑，是它的优点。但同时也被认为是它的缺点，因为**除了业务逻辑外还要写很多类型逻辑**。

不过，我倒是觉得这种复杂度是不可避免的，因为 JS 本身足够灵活，要准确定义类型那类型系统必然也要设计的足够灵活。

正是因为TypeScript 类型系统的复杂性，不然也不会被大家戏称为`类型体操`了