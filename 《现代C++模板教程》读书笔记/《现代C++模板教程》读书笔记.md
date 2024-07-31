# 《现代 C++ 模板教程》

## 记在前面

- 模板本身不是代码，模板实例化后才有代码出现，才会有符号供链接器去链接。
- 模板就像是一门脚本语言，它指导编译器该怎么样生成代码。
- 模板的大多数语法特性是普遍适用的，不区分函数模板、类模板、变量模板。

## 函数模板

### 万能引用与引用折叠

- 万能引用

  “万能引用”又称转发引用，即接受左值表达式那形参类型就推导为左值引用，接受右值表达式，那就推导为右值引用。

  ```cpp
  template <typename T>
  void f(T&& t){}
  
  int a = 10;
  f(a);       // a 是左值表达式，f 是 f<int&>，t 的类型是 int&
  f(10);      // 10 是右值表达式，f 是 f<int>，t 的类型是 int&&
  ```

- 引用折叠

  通过模板或 `typedef` 中的类型操作可以形成引用的引用，此时适用“引用折叠”（reference collapsing）规则：

  右值引用的右值引用折叠成右值引用，所有其他组合均折叠成左值引用。

  - `X& &`、`X& &&`、`X&& &` 都折叠成类型 `X&`
  - `X&& &&` 折叠成 `X&&`

  ```cpp
  typedef int&  lref;
  typedef int&& rref;
  int n;
   
  lref&  r1 = n; // r1 的类型是 int&
  lref&& r2 = n; // r2 的类型是 int&
  rref&  r3 = n; // r3 的类型是 int&
  rref&& r4 = 1; // r4 的类型是 int&&
  ```

  ```cpp
  template <class Ty>
  constexpr Ty&& forward(Ty& Arg) noexcept {
      return static_cast<Ty&&>(Arg);
  }
  
  int a = 10;            // 不重要
  ::forward<int>(a);     // 返回 int&&，因为 Ty 是 int，Ty&& 就是 int&&
  ::forward<int&>(a);    // 返回 int&，因为 Ty 是 int&，Ty&& 就是 int&
  ::forward<int&&>(a);   // 返回 int&&，因为 Ty 是 int&&，Ty&& 就是 int&&
  ```

  > 由于存在引用折叠规则，对于类型是 `T&` 的参数，只能给它传递一个左值。

### 模板非类型参数

非类型模板形参有众多的规则和要求，目前，我们简单认为需要参数是“常量”即可。

```cpp
template <std::size_t N>
void f() { std::cout << N << '\n'; }

f<100>();
```

### 可变参数模板

和其他语言一样，C++ 也是支持可变参数的，但我们必须使用模板才能做到。

我们提一个简单的需求：实现一个函数 `sum`，支持接收任意类型、任意个数的参数，对它们进行求和后，以公共类型返回。

首先就要引入一个东西：“形参包”。

“模板形参包”是接受零个或更多个模板实参（非类型、类型或模板）的模板形参。

“函数形参包”是接受零个或更多个函数实参的函数形参。

```cpp
template <typename...Args>
void sum(Args...args) {}
```

这样一个函数，就可以接受任意类型的任意个数的参数调用。

`args` 是函数形参包，`Args` 是模板类型形参包，它们的名字我们可以自定义。

`args` 中存储了我们传入的全部函数参数，`Args` 中存储了我们传入的全部函数参数的类型。

---

那么，我们要如何把这些东西取出来使用呢？这就涉及到另一个知识：“形参包展开”。

```cpp
void f(const char*, int, double) { puts("="); }
void f(const char**, int*, double*) { puts("&"); }

template <typename...Args>
void sum(Args...args) {  // const char * args0, int args1, double args2
    f(args...);   // 相当于 f(args0, args1, args2)
    f(&args...);  // 相当于 f(&args0, &args1, &args2)
}

int main() {
    sum("luse", 1, 1.2);
}
```

`Args...args` 被展开为 `const char* args0, int args1, double args2`。

这里我们需要定义一个术语：“模式”。

后随省略号且其中至少有一个形参包的名字的模式，会被展开成零个或更多个以逗号分隔的模式实例。

`&args...` 中 `&args` 就是模式。在展开的时候，模式，也就是省略号前面的一整个表达式，会被不停的填入对象并添加 `&`，然后以逗号分隔，直至形参包的元素被消耗完。

那么根据这个，我们就能写出一些有意思的东西，比如一次性把它们打印出来：

```cpp
template <typename...Args>
void print(const Args&...args){    // const char (&args0)[5], const int& args1, const double& args2
    using Arr = int[];
    (void)Arr{ 0, (std::cout << args << ' ' , 0)... };		// 弃值表达式
}

int main() {
    print("luse", 1, 1.2);
}
```

这里为啥要括号里加个 `, 0` 呢？这是因为逗号表达式是从左往右执行的，返回右边的值作为逗号表达式的值。也就是说：每一个 `(std::cout << arg0 << ' ' , 0)` 都会返回 `0`，这主要是为了符合语法，用来初始化数组。我们创建了一个数组，最终这些 `0` 会用来初始化这个数组。当然，这个数组本身没有用，只是为了创造合适的包展开场所。

---

我们再给出一个数组的示例：

```cpp
template <typename...Args>
void print(const Args&...args) {
    int _[]{ (std::cout << args << ' ' ,0)... };
}

template <typename T, std::size_t N, typename...Args>
void f(const T(&array)[N], Args...index) {
    print(array[index]...);
}

int main() {
    int array[10]{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    f(array, 1, 3, 5);
}
```

我们复用了之前写的 `print` 函数，我们看新的 `f` 函数即可。

`array[index]...` 是包展开，`array[index]` 是模式，实际展开的时候就是：`array[arg0], array[arg1], array[arg2]`。

---

那么回到最初的需求，实现一个 `sum`：

```cpp
#include <iostream>
#include <type_traits>

template <typename...Args, typename RT = std::common_type_t<Args...>>
RT sum(const Args&...args) {
    RT _[]{ static_cast<RT>(args)... };
    RT n{};
    for (int i = 0; i < sizeof...(args); ++i) {
        n += _[i];
    }
    return n;
    // return std::accumulate(std::begin(_), std::end(_), RT{});  // 也可以直接调用标准库的求和函数
}

int main() {
    double ret = sum(1, 2, 3, 4, 5, 6.7);
    std::cout << ret << '\n';       // 21.7
}
```

## 类模板

### 类模板参数推导

从 C++17 开始，只要传给构造函数的实参可以用来推导类模板的全部模板参数（含非类型参数），就无需显式指定类模板参数。

但有时编译器推导出来的类型与我们的期望不符，此时可以定义“推导指引”。

举个例子，我们要让一个类模板，如果推导为 `int`，就让它实际成为 `size_t`：

```cpp
template <typename T>
struct Test{
    Test(T v) : t{ v } {}
private:
    T t;
};

Test(int) -> Test<std::size_t>;

Test t(1);      // t 是 Test<size_t>
```

如果要类模板 `Test` 推导为指针类型，就变成数组呢？

```cpp
templat e<typename T>
Test(T*) -> Test<T[]>;

char* p = nullptr;

Test t(p);      // t 是 Test<char[]>
```

推导指引的语法还是简单的，如果只是涉及具体类型，那么只需要：

`模板名称(类型a)->模板名称<想要让类型a被推导为的类型>`

如果涉及的是一类类型，那么就需要加上 `template`，然后使用它的模板形参。

---

我们提一个稍微有点难度的需求：

```cpp
template <class Ty, std::size_t size>
struct array {
    Ty arr[size];
};

::array arr{1, 2, 3, 4, 5};     // Error!
```

类模板 `array` 同时使用了类型模板形参与非类型模板形参，保有了一个成员是数组。

它无法被我们直接推导出类型，此时就需要我们自己定义推导指引。

这会用到我们之前在函数模板里学习到的形参包。

```cpp
template <typename T, typename ...Args>
array(T t, Args...) -> array<T, sizeof...(Args) + 1>;
```

原理很简单，我们要给出 `array` 的模板类型，那么就让模板形参单独写一个 `T` 占位，放到模板形参列表中，并且写一个模板类型形参包用来处理任意个参数；获取 `array` 的 `size` 也很简单，直接使用 `sizeof...` 获取形参包的元素个数，然后再 +1 ，因为先前我们用了一个模板形参占位。

标准库的 `std::array` 的推导指引，原理和这个一样。

### 模板模板形参

> 没看懂。

### 成员函数模板

成员函数模板基本上和普通函数模板没多大区别，唯一需要注意的是，它大致有两类：

- 类模板中的成员函数模板

  ```cpp
  template <typename T>
  struct ClassTemplate{
      template <typename... Args>
      void f(Args&&... args) {}
  };
  ```

- 普通类中的成员函数模板

  ```cpp
  struct Test{
      template <typename... Args>
      void f(Args&&... args){}
  };
  ```

> 需要注意的是，以下 `ClassTemplate` 的成员函数 `f` 不是函数模板，它就是普通的成员函数。
>
> ```cpp
> template <typename T>
> struct ClassTemplate{
>     void f(T) {}
> };
> ```

## 变量模板

变量模板是 C++14 引入的，变量模板实例化后就是一个具有静态存储期的变量，所以也不用考虑生存期的问题。

在类中也可以使用变量模板，作为类的静态数据成员。

```cpp
struct Limits{
    template<typename T>
    static const T min; // 静态数据成员模板的声明
};
 
template<typename T>
const T Limits::min{}; // 静态数据成员模板的定义
```

## 模板特例化

在某些情况下，通用模板的定义并不适用于特定类型。对此，我们可以专门为特定类型定义一个特例化版本，这被称作“模板特例化”。

模板特例化的本质是接管了编译器的工作，手动实例化一个模板。

### 模板全特化

模板全特化的核心语法在于 `template<>`，它表示我们将为原模板的所有模板参数提供实参。

```cpp
template<typename T>
struct X {
    template<typename T2>
    void f(T2) {}

    template<>
    void f<int>(int) {            // 类内特化，对于 函数模板 f<int> 的情况
        std::puts("f<int>(int)"); 
    }
};

template<>
template<>
void X<void>::f<double>(double) { // 类外特化，对于 X<void>::f<double> 的情况
    std::puts("X<void>::f<double>");
}

X<void> x;
x.f(1);    // f<int>(int)
x.f(1.2);  // X<void>::f<double>
x.f("");
```

> 上述代码中，类内对成员函数 `f` 的特化，在 gcc 无法通过编译，根据考察，这是一个很多年前就有的 bug。

### 模板偏特化

模板偏特化允许我们为具有相同特征的类模板、变量模板进行定制行为。

与模板全特化不同，我们不会为原模板的所有模板参数提供实参，模板偏特化后的本质还是一个模板。

> 只有类模板和变量模板可以偏特化，函数模板不可以偏特化。

- 类模板偏特化

  ```cpp
  template<typename T,std::size_t N>
  struct X{
      template<typename T_,typename T2>
      struct Y{};
  };
  
  template<>
  template<typename T2>
  struct X<int, 10>::Y<int, T2> {     // 对 X<int,10> 的情况下的 Y<int> 进行偏特化
      void f()const{}
  };
  
  X<int, 10>::Y<int, void>y;
  y.f();                      // OK X<int,10> 和 Y<int> 
  X<int, 1>::Y<int, void>y2;
  y2.f();                     // Error! 主模板模板实参不对
  X<int, 10>::Y<void, int>y3;
  y3.f();                     // Error！成员函数模板模板实参不对
  ```

  > 此示例无法在 gcc 通过编译，这是编译器 bug。

- 变量模板偏特化

  ```cpp
  template <typename T>
  const char* s = "?";            // 原模板
  
  template <typename T>
  const char* s<T*> = "pointer";  // 偏特化，对指针这一类类型
  
  template <typename T>
  const char* s<T[]> = "array";   // 偏特化，但是只是对 T[] 这一类类型，而不是数组类型，因为 int[] 和 int[N] 不是一个类型
  
  std::cout << s<int> << '\n';            // ?
  std::cout << s<int*> << '\n';           // pointer
  std::cout << s<std::string*> << '\n';   // pointer
  std::cout << s<int[]> << '\n';          // array
  std::cout << s<double[]> << '\n';       // array
  std::cout << s<int[1]> << '\n';         // ?
  ```


## 模板显式实例化

模板实例化有隐式实例化和显式实例化两种。

- 隐式实例化

  我们平时粗略上说的“模板只有使用了才会生成实际代码”，其实就是隐式实例化，它是指编译器检测到我们使用到模板之后，会在当前翻译单元中查找模板定义，根据模板定义生成模板实例代码。

- 显式实例化

  我们可以在模板定义可见的地方，使用“显式实例化定义”，让编译器在此处生成模板实例代码。这样一来，当前翻译单元就有相应的符号供外部链接了。

由此可见，显式实例化可以解决模板分文件的问题。我们可以将模板声明放在头文件中，将模板定义放在源文件中，源文件中还需要提供显示实例化定义。这样，外部使用模板的翻译单元只需要包含带有模板声明的头文件，或者使用 `extern` 模板声明（显式实例化声明），就可以正常链接了。

```cpp
// 显式实例化定义
template 类关键词 类模板名<模板实参列表>;
template 返回类型 函数模板名<模板实参列表>(形参列表);
// 显式实例化声明
extern template 类关键词 类模板名<模板实参列表>;
extern template 返回类型 函数模板名<模板实参列表>(形参列表); 
```

## 折叠表达式

折叠表达式（C++17）能够让我们更优雅地进行形参包展开。

### 语法

```
( 形参包 运算符 ... )              (1) 一元右折叠
( ... 运算符 形参包 )              (2) 一元左折叠
( 形参包 运算符 ... 运算符 初值 )    (3) 二元右折叠
( 初值 运算符 ... 运算符 形参包 )    (4) 二元左折叠
```

折叠表达式的实例化按以下方式展开成表达式：

1. 一元右折叠 `(E 运算符 ...)` 成为 `(E1 运算符 (... 运算符 (EN-1 运算符 EN)...))`
2. 一元左折叠 `(... 运算符 E)` 成为 `((...(E1 运算符 E2) 运算符 ...) 运算符 EN)`
3. 二元右折叠 `(E 运算符 ... 运算符 I)` 成为 `(E1 运算符 (... 运算符 (EN−1 运算符 (EN 运算符 I))...))`
4. 二元左折叠 `(I 运算符 ... 运算符 E)` 成为 `((...((I 运算符 E1) 运算符 E2) 运算符 ...) 运算符 EN)`

> 这里的形参包 `E` 实际上是指含有形参包的表达式。

总结：

- 折叠表达式时左折叠还是右折叠，取决于 `...` 是在形参包的左边还是右边。
- 右折叠就是先算右边，左折叠就是先算左边。
- 判断一个折叠表达式是否是二元的，只需要看一点：`运算符 ... 运算符` 这种形式就是二元的。
- 二元折叠表达式必然有一个“初值”，是先计算的。
- 对于一些运算符来说，例如逗号运算符，一元左折叠和一元右折叠没有区别。

### 示例

- 一元折叠

  ```cpp
  template <typename...Args>
  void print_right(const Args&...args) {
      ((std::cout << args << ' '), ...);
  }
  
  template <typename...Args>
  void print_left(const Args&...args) {
      (..., (std::cout << args << ' '));
  }
  ```

- 二元折叠

  ```cpp
  template<typename... Args>
  void print(Args&&... args){
      (std::cout << ... << args) << '\n';
  }
  
  // 二元右折叠
  template<int...I>
  constexpr int v = (I + ... + 10);    // 1 + (2 + (3 + (4 + 10)))
  // 二元左折叠
  template<int...I>
  constexpr int v2 = (10 + ... + I);   // (((10 + 1) + 2) + 3) + 4
  
  std::cout << v<1, 2, 3, 4> << '\n';  // 20
  std::cout << v2<1, 2, 3, 4> << '\n'; // 20
  ```

  



