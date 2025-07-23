# 《C++ Templates》读书笔记

## 基础知识

### 函数模板

显式指定一个空模板实参列表（`<>`），此语法表明只有模板可以解析此次调用，但所有模板参数都应该从调用实参中推导出来。

### 类模板

- 别名模板

  ```cpp
  namespace std {
      template <typename T>
      using add_const_t = typename add_const<T>::type;
  }
  ```

63

## 深入了解模版

## 模板和设计

