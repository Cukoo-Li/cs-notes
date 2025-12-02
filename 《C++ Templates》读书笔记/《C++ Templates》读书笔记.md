# 《C++ Templates》读书笔记

## 基础知识

### 零碎的知识

- 显式指定一个空模板实参列表（`<>`），此语法表明只有模板可以解析此次调用，但所有模板参数都应该从调用实参中推导出来。
- 如果两个函数模板只存在尾部参数包的差别，则首选没有尾部参数包的函数模板。
- 因为函数模板不支持偏特化，所以如果想要基于某些约束条件来改变函数实现的话，可以使用带有静态函数的类、`std::enable_if<>`、`SFINAE` 以及 `ifconstexpr` 。
- 偏特化、SFINAE 和 `std::enable_if<>` 允许我们在启用或禁用特定的模板实现。

### 类型特征

- `std::decay<>` 允许手动退化按引用传递的模板参数类型


### enable_if<>

`std::enable_if<>` 可以用于在特定编译期条件下禁用函数模板。它会对作为其（第 1 个）模板实参传入的、给定的编译期表达式求值，其行为如下：

- 如果表达式结果为 `true`，则其类型成员 `type` 返回一个类型：如果没有传递第 2 个模板实参，则类型为 `void`，否则，该类型为第 2 个模板实参的类型。
- 如果表达式为 `false`，则其类型成员 `type` 是未定义的。由于 SFINAE，其具有忽略带有 `std::enable_if<>` 表达式的函数模板的效果。

自 C++14 以来，所有类型特征的萃取所得都是类型，所以有相应的别名模板，其允许省略 `typename` 和 `::type`。

```cpp
template<bool _Cond, typename _Tp = void>
using enable_if_t = typename enable_if<_Cond, _Tp>::type;
```

使用 `std::enable_if<>` 的常用方法是使用带有默认值的附加函数模板实参：

```cpp
template<typename T, typename Ret = std::enable_if_t<sizeof(T > 4), int>>
Ret foo() {
    return 42;
}
```

### 处理返回值

使用模板参数 `T` 作为返回类型并不能保证它不是引用，因为 `T` 有可能会被隐式推导或显示指定为引用。安全起见，有两种选择：

- 使用类型特征 `std::remove_reference<>` 将类型 `T` 转换为非引用类型。
- 通过仅声明返回类型为 `auto`，让编译器来推导返回类型，因为 `auto` 总会导致类型退化。

### 推荐的模板参数声明方法

- 默认情况下，将参数声明为按值传递，即 `T`。

  > 这种方法比较简单，即使是字符串字面量通常也可以正常工作。对于小型实参、右值对象来说性能很好。当传递左值大型对象时，调用者有时可以使用 `std::ref` 来避免高昂的拷贝成本。

- 如果需要 out 或 inout 参数，就按非常量引用来传递实参，即 `T&`。

- 如果实参拷贝成本很高，是只读访问且不需要本地副本，就使用常量引用，即 `const T&`。

- 如果模板是用来转发实参的，就使用完美转发。即使用 `T&&` 和 `std::forward<>`。

### 编译期编程

- 模板提供了在编译期进行计算的能力（使用递归进行迭代和使用偏特化或运算符 `?:` 来做选择）。

- 通过 `constexpr` 函数，能将大多数编译期计算替换为编译期上下文中可调用的“普通函数”。

- 通过偏特化，可以基于某些编译期约束条件在不同的类模板实现之间进行选择。

  ```cpp
  template <int SZ, bool = isPrime(SZ)>
  struct Helper;
  
  template <int SZ>
  struct Helper<SZ, false> {
      // ...
  }
  
  template <int SZ>
  struct Helper<SZ, true> {
      // ...
  }
  ```

- 可以通过 `decltype` 来 SFINAE 掉表达式。

  ```cpp
  template <typename T>
  auto len(const T& t) -> decltype((void)(t.size()), T::size_type()) {
      return t.size();
  }
  ```

- 编译期 `if` 允许我们根据编译期条件来启用或者丢弃某些语句。



## 深入了解模版



## 模板和设计

