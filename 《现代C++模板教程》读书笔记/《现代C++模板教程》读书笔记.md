# 《现代 C++ 模板教程》

模板本身不是代码，模板实例化后才有代码出现。

## 函数模板

### 万能引用与引用折叠

- 万能引用

  「万能引用」又称转发引用，即接受左值表达式那形参类型就推导为左值引用，接受右值表达式，那就推导为右值引用。

  ```cpp
  template <typename T>
  void f(T&&t){}
  
  int a = 10;
  f(a);       // a 是左值表达式，f 是 f<int&>，t 的类型是 int&
  f(10);      // 10 是右值表达式，f 是 f<int>，t 的类型是 int&&
  ```

- 引用折叠

  通过模板或 `typedef` 中的类型操作可以形成引用的引用，此时适用「引用折叠」（reference collapsing）规则：

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

首先就要引入一个东西：「形参包」

「模板形参包」是接受零个或更多个模板实参（非类型、类型或模板）的模板形参。

「函数形参包」是接受零个或更多个函数实参的函数形参。

```cpp
template <typename...Args>
void sum(Args...args) {}
```

这样一个函数，就可以接受任意类型的任意个数的参数调用。

`args` 是函数形参包，`Args` 是模板类型形参包，它们的名字我们可以自定义。

`args` 中存储了我们传入的全部函数参数，`Args` 中存储了我们传入的全部函数参数的类型。

---

那么，我们要如何把这些东西取出来使用呢？这就涉及到另一个知识：「形参包展开」。

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

这里我们需要定义一个术语：「模式」。

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

这里为啥要括号里加个 `, 0` 呢？这是因为逗号表达式是从左往右执行的，返回右边的值作为逗号表达式的值。也就是说：每一个 `(std::cout << arg0 << ' ' , 0)` 都会返回 `0`，这主要是为了符合语法，用来初始化数组。我们创建了一个数组 `int _[]`，最终这些 `0` 会用来初始化这个数组。当然，这个数组本身没有用，只是为了创造合适的包展开场所。

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
