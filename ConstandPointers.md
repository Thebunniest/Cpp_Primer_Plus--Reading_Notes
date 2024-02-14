# C++中的`const`限定符和指针

> 本文档是2023年暑假时个人阅读C++ Primer Plus时整理的读书笔记之一，现整理并发布于GitHub上，供大家参考。
>
> 鄙人不胜码力，若有不妥之处还望不吝指出！

## 0. 说在前面

### `const`限定符的使用规范

 `const`限定符修饰的变量，必须在声明/定义的同时赋初值，即必须进行**初始化**：

```c++
const int Correct = 1;//正确地初始化
const int Wrong;//编译器报错：const类型变量必须指定初值
Wrong = 1;//编译器报错2：不能修改const类型变量的值（假设上一条声明语句已经被正确地修改了）
```

### 如何正确地阅读声明？

Bjarne在*The C++ Programming Language*中写道：
>从右向左读；同时把*解释为"pointer to"。

这能解决不少问题。不过为了大家理解上的方便，我们还是看看下面的例子吧。



## 1. 不作限定

指针指向的变量可以改变，可以通过指针的解引用来修改其指向的变量的值。

```c++
int * plainPtr;
int plainVar = 0;
plainPtr = & plainVar // Zzz
```



## 2. 让指针指向常量：指向常量的指针

指针**指向常量**，这意味着**不能通过指针修改其指向的变量的值**，但**指针的指向可以改变**。

### 2.1 指向 `const 基类型`的指针

```c++
int variable = 114;//or const int variable = 114;
const int * constptr = &variable;
```

现在，不管`variable`是不是`const`量，都不可以用`*constptr = newvalue`来修改其值，因为编译器从其类型推断，`constptr`指向的是一个常量（尽管这个例子里，事实上，并非如此）。

#### *和指向`基类型`的指针作比较，其有何区别？

最大的不同：指向`基类型`的指针**不可以**指向`const`修饰符修饰的基类型变量！

```c++
const int Constant = 0;
int * normalptr = &Constant;//INVALID
```

假如上面的代码通过编译，则可以通过`*normalptr = newvalue`的方式来修改`const`变量，这显然是荒唐的。

### 2.2 二级（高级）指针和`const`

`const` 和非 `const` 类型之间的**指针类型变量**（**非**基类型变量）不可以相互赋值——**即使是为了保险，把非`const`赋给`const`类型也不行**。
假如允许这样的行为，就会产生极其荒唐的效果，请看如下代码：

```c++
const int ** pp2;//pp2 是一个指向[指向 const int 的指针(类型为 const int * )]的指针
int * p1;//p1 是一个指向普通的 int 的指针
const int n = 0;

pp2 = &p1;// 假设允许 const 和非 const 类型之间的【指针】相互赋值
*pp2 = &n;//合法，因为pp2 和 n 都是 const类型。注意，由于上一条语句，p1“暗度陈仓”地指向了n
*p1 = 1；//也是合法的，因为 p1 不是 const 类型，可以修改其指向的值
//诶，等等，现在就尴尬了——这导致 const 变量 n 的值被修改了！
```

### 2.3 `const`和函数的接口设计

基于上面的讨论，在函数传参时，C++禁止将`const typeName *`类型的实参传入到`typeName *`类型的形参中。

```c++
const int * ptr1;
***
int func(int * ptr)//接口不是 const 类型
{
    /*Do some thing to the variable that ptr points to*/
}
***
int a = func(ptr1);//编译不通过，不允许通过操作func()中的形参 ptr 来修改 ptr1 指向的变量的值
```

#### `const`和函数的接口设计：小总结

一条经验上的原则：当函数的接口包括指向**基类型**的指针，并且**不需要通过该指针修改变量的值**时，应该**尽量把形参声明为`const`类型的指针**，这样不仅可以防止意外修改数据，还可以增加接口的通用性——这样，`const`和非`const`的指针**都**可以被函数接受并通过编译。

当函数的接口包括高级指针时，`const`的混用可能带来问题，所以说不要在这里耍小聪明。

## 3. 让指针成为常量:常量指针

指针成为常量，这意味着**指针的指向不能改变**，但**可以通过这个指针修改其指向的变量的值**。

```c++
int var1 = 0;
int * const constptr = &var1;//这里constptr是一个指向int的const指针
```

吟唱咒语：

> Well let's see... Aha! The variable `constptr` is a constant pointer to an integer!

> Bjarne老爷子的咒语挺有用的，不过这种指针真的有什么用吗？

## 4. 让常量指针指向常量

```c++
const int constvar = 0;
const int * const constptr = &constvar;//指向 const int 的 const 指针 
```

吟唱咒语：

> Err... The variable `constptr`is a constant pointer to an integer constant... SERIOUSLY???

`constptr`只能解引用，别的什么也干不了。

>  天啊，我现在觉得常量指针太有用了（雾）
