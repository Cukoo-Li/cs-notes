# 《现代 C++ 并发编程教程》读书笔记

## 使用线程对象

在 C++ 标准中，`std::thread` 可以指代线程，我们所说的线程对象其实就是指 `std::thread` 类的对象。

### 创建线程对象

- 一旦为线程对象提供一个可调用对象，就会启动一个新线程执行该可调用对象，线程对象与该新线程关联。
- 可调用对象的参数默认会被拷贝到新线程的内存空间中。即使参数是引用类型，实际上依然是拷贝。如果不想拷贝，需要使用 `std::ref`。
- `std::thread` 不可复制，只可移动。

### 管理线程对象

启动线程后，我们必须在线程对象的生命周期结束之前，决定它的执行策略，是 `join` 还是 `detach`。调用 `join` 或 `detach` 后，线程对象不再关联某个线程。

- `join` - 等待关联的线程执行完毕。
- `detach` - 使得所关联线程独立地运行。

> `std::thread::~thread` 执行时，首先通过 `std::thread::joinable` 判断线程对象目前是否有关联线程，如果是，就调用 `std::terminate`。换言之，不能销毁一个仍关联着某个线程的对象对象。
>
> 通常非常不推荐使用 `detach`，因为程序员必须确保所有创建的线程正常退出。

`std::this_thread` 这个命名空间包含了管理当前线程的函数，常用的有：

- `get_id` - 返回当前线程id。
- `sleep_for` - 使当前线程睡眠指定时间，接受一个`std::chrono`命名空间中的时间对象。

```cpp
int main() {
    std::this_thread::sleep_for(std::chrono::seconds(3));
}
```

## 线程互斥

### 互斥锁

`std::mutex` 是一种最普通的互斥体，只允许一个线程独占访问。

- 提供 `lock` 和 `unlock` 成员函数用于上锁和解锁。
- 提供 `try_lock` 成员函数，用于尝试上锁，上锁失败不会阻塞当前线程。

### 读写锁

`std::shared_mutex` 是用于涉及频繁读、少量写场景的互斥体，允许多个线程同时读，但涉及到写操作时只允许一个线程独占访问。

- 和 `std::mutex` 一样，提供 `lock`、`unlock`、`try_lock` 成员函数，可实现写线程独占访问。
- 提供 `lock_shared`、`unlock_shared`、`try_lock_shared` 成员函数，可实现读线程同时读。

### 递归锁

`std::recursive_mutex` 允许一个线程多次对它进行上锁和解锁，只有当解锁与上锁次数相匹配时，互斥体才会被释放。

### 互斥体包装器

我们通常不会直接操作互斥体本身，而是使用特定的互斥体包装器间接地去操作它，这是更安全的。

- `std::lock_guard`
  - `std::lock_guard` 的结构非常简单，仅使用 RAII 封装了互斥体的 `lock` 和 `unlock` 操作。
  - 通常会使用 `{}` 创建一个块作用域，从而限制 `std::lock_guard` 对象的生命周期。
  - 可以在构造时额外传递一个 `std::adopt_lock` 参数，这样构造时就不会对 `std::mutex` 执行上锁操作。
- `std::unique_lock`
  - `std::unique_lock` 不仅包含 `std::lock_guard` 的基本功能，还更加灵活，使用起来也更加复杂。

  - 提供了 `lock` 和 `unlock` 成员函数。当我们手动调用 `unlock` 后，其析构函数不会再调用 `unlock`。这使得我们不再需要专门去使用 `{}` 创建一个块作用域来限制其生命周期。
  - 可以与 `std::condition_variable` 结合在一起使用。
- `std::shared_lock`
  - `std::shared_lock` 是共享互斥体的包装器，与 `std::unique_lock` 很类似，主要区别在于它使用共享模式锁定和解锁关联的共享互斥体。

```cpp
class Settings {
private:
    std::map<std::string, std::string> data_;
    mutable std::shared_mutex mutex_; // “M&M 规则”：mutable 与 mutex 一起出现

public:
    void set(const std::string& key, const std::string& value) {
        std::lock_guard<std::shared_mutex> lock(mutex_);
        data_[key] = value;
    }

    std::string get(const std::string& key) const {
        std::shared_lock<std::shared_mutex> lock(mutex_);
        auto it = data_.find(key);
        return (it != data_.end()) ? it->second : ""; // 如果没有找到键返回空字符串
    }
};
```

### 死锁的解决

我们可以通过一些简单的规则，约束开发者的行为，帮助写出“无死锁”的代码。

- 避免嵌套锁

  线程获取一个锁时，就别再获取第二个锁。每个线程只持有一个锁，自然不会产生死锁。如果必须要获取多个锁，使用 `std::scoped_lock`。

  > `std::scoped_lock` 能够一次性锁住多个 `std::mutex`，没有死锁的风险。

- 使用固定顺序获取锁

## 线程同步

线程同步是指协调线程之间的工作顺序，使得某个线程必须等待另一个线程完成特定操作后，才能进行后续操作。

### 实现线程同步的笨方法

- 忙等待

  ```cpp
  bool condtion = false;
  std::mutex mtx;
  
  void wait_for_flag(){
      std::unique_lock<std::mutex> lck{mtx};
      while (!condtion){
          lck.unlock();    
          lck.lock();      
      }
  }
  ```

- 定时检查

  ```cpp
  bool condtion = false;
  std::mutex mtx;
  
  void wait_for_flag(){
      std::unique_lock<std::mutex> lck{mtx};
      while (!condtion){
          lck.unlock();    
          std::this_thread::sleep_for(std::chrono::seconds(3));
          lck.lock();      
      }
  }
  ```

### 使用条件变量

条件变量可以用于更好地实现线程同步，主要包括两个动作：一个线程因“等待条件成立”而被阻塞；另一个线程“使条件成立”，而后唤醒阻塞的线程。

类 `std::condition_variable` 是 C++ 标准库对条件变量的实现，常用方法包括：

- `void notify_one()` - 唤醒一个等待条件变量的线程

- `void notify_all()` - 唤醒所有等待条件变量的线程

- `void wait(std::unique_lock<std::mutex>& lock)` - 当前线程阻塞自身并解锁，被唤醒后重新上锁

- `void wait(std::unique_lock<std::mutex>& lock, Predicate pred)` - 等待 `pred` 为真

  ```cpp
  // wait(lock, pred) 等价于：
  while (!pred())
      wait(lock);
  ```

> 为什么条件变量总是和互斥锁结合在一起使用？
>
> 条件表达式所涉及的数据是共享数据，在对其进行读写时需要加锁。因此，条件变量总是和一个互斥锁结合在一起使用。首先将互斥锁加锁，然后查看条件是否成立，如果条件不成立，则调用条件变量的 `wait` 方法。该方法在执行时，首先将线程阻塞，然后将互斥锁解锁。阻塞和解锁是原子的，从而保证从检查条件是否成立，到调用线程被阻塞的这段时间内，其他线程没有机会修改条件的状态。当 `wait` 方法成功返回时（意味着某个线程使得条件成立，唤醒被阻塞的线程），互斥锁会再次被锁上，从而恢复到调用 `wait` 方法之前加锁的状态。

```cpp
std::mutex mtx;
std::condition_variable cv;
bool arrived = false;

void wait_for_arrival() {
    std::unique_lock<std::mutex> lck(mtx);
    cv.wait(lck, []{ return arrived; }); // 等待 arrived 变为 true
    std::cout << "到达目的地，可以下车了！" << std::endl;
}

void simulate_arrival() {
    std::this_thread::sleep_for(std::chrono::seconds(5)); // 模拟地铁到站，假设5秒后到达目的地
    {
        std::lock_guard<std::mutex> lck(mtx);
        arrived = true; // 设置条件变量为 true，表示到达目的地
    }
    cv.notify_one(); // 通知等待的线程
}
```

### std::future

