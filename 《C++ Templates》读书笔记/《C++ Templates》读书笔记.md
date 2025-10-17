# 《C++ Templates》读书笔记

## 基础知识

- 显式指定一个空模板实参列表（`<>`），此语法表明只有模板可以解析此次调用，但所有模板参数都应该从调用实参中推导出来。
- 如果两个函数模板只存在尾部参数包的差别，则首选没有尾部参数包的函数模板。


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



## 深入了解模版



## 模板和设计

