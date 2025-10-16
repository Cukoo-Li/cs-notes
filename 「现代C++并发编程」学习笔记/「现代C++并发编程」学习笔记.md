# 「现代 C++ 并发编程」学习笔记

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

- `get_id` - 返回当前线程 id。
- `sleep_for` - 使当前线程睡眠指定时间，接受一个 `std::chrono` 命名空间中的时间对象。

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
        std::unique_lock<std::shared_mutex> lock(mutex_);
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
    std::this_thread::sleep_for(std::chrono::seconds(5)); // 模拟地铁到站，假设 5 秒后到达目的地
    {
        std::lock_guard<std::mutex> lck(mtx);
        arrived = true; // 设置条件变量为 true，表示到达目的地
    }
    cv.notify_one(); // 通知等待的线程
}
```

### 获取异步任务返回值

当 `std::thread` 对象所关联的线程执行结束时，我们没有办法直接获取其返回值。

如果想获取返回值，我们可以使用 `std::packaged_task`、`std::async`、`std::future`。

模板类 `std::future` 用于表示异步操作的结果。

#### std::async

使用 `std::async` 启动一个异步任务，它会返回一个 `std::future` 对象，这个对象和任务关联，将持有最终计算出来的结果。当需要任务执行完的结果的时候，只需要调用 `get` 成员函数，就会阻塞直到 `future` 为就绪为止（即任务执行完毕），返回执行结果。`valid` 成员函数检查 `future` 当前是否关联共享状态，即是否当前关联任务。还未关联，或者任务已经执行完（调用了 `get` 或 `wait`），都会返回 `false`。

与 `std::thread` 一样，`std::async` 支持任意可调用对象，以及传递调用参数。包括支持使用 `std::ref` ，以及支持只能移动的类型。

`std::async` 的执行策略：

- `std::launch::async` 在不同线程上执行异步任务。
- `std::launch::deferred` 惰性求值，不创建线程，等待 `future` 对象调用 `wait` 或 `get` 成员函数的时候执行任务。

而我们先前一直没有写明这个参数，是因为 `std::async` 函数模板有两个重载，不给出执行策略就是以：`std::launch::async | std::launch::deferred` 调用另一个重载版本，此策略表示由实现选择到底是否创建线程执行异步任务。典型情况是，如果系统资源充足，并且异步任务的执行不会导致性能问题，那么系统可能会选择在新线程中执行任务。但是，如果系统资源有限，或者延迟执行可以提高性能或节省资源，那么系统可能会选择延迟执行。

> 如果从 `std::async` 获得的 `std::future` 没有被移动或绑定到引用，那么在完整表达式结尾，`std::future` 的析构函数将阻塞，直到到异步任务完成。因为临时对象的生存期就在这一行，而对象生存期结束就会调用调用析构函数。
>
> ```cpp
> std::async(std::launch::async, []{ f(); }); // 临时量的析构函数等待 f()
> std::async(std::launch::async, []{ g(); }); // f() 完成前不开始
> ```
>
> 这并不能创建异步任务，它会阻塞，然后逐个执行。

#### std::packaged_task

类模板 `std::packaged_task` 可以包装任何可调用对象（与 `std::function` 类似），允许异步获取其结果，它将可调用对象的结果或所抛异常传递给一个 `std::future` 对象。

它通常会和 `std::future` 一起使用，不过也可以单独使用：

```cpp
std::packaged_task<double(int, int)> task([](int a, int b){
    return std::pow(a, b);
});
task(10, 2); // 执行传递的 lambda，但无法获取返回值
```

> 它有 `operator()` 的重载，它会执行我们传递的可调用对象，不过这个重载的返回类型是 `void`，没办法获取返回值。

如果想要异步的获取返回值，我们需要在调用 `operator()` 之前，让它和 `std::future` 关联，然后使用 `std::future::get`：

```cpp
std::packaged_task<double(int, int)> task([](int a, int b){
    return std::pow(a, b);
});
std::future<double>future = task.get_future();
task(10, 2); // 此处执行任务
std::cout << future.get() << '\n'; // 不阻塞，此处获取返回值
```

如果想在线程中执行异步任务，再获取返回值，可以这么做：

```cpp
std::packaged_task<double(int, int)> task([](int a, int b){
    return std::pow(a, b);
});
std::future<double> future = task.get_future();
std::thread t{ std::move(task),10,2 }; // 任务在线程中执行
// todo.. 幻想还有许多耗时的代码
t.join();

std::cout << future.get() << '\n'; // 并不阻塞，获取任务返回值罢了
```

## 原子操作

原子操作即不可分割的操作，要么都完成，要么都不完成。系统中的所有线程，不可能观察到原子操作只完成了一半的情况。

在 C++ 中，实现原子操作主要有两种方式：

- 使用互斥量实现“逻辑上的原子操作”
- 使用原子类型 `std::atomic`

### std::atomic

类模板 `std::atomic` 保证在某类型上的某些操作是原子的，其内部可能使用了 CPU 层面上的原子指令来实现，也可能使用了锁来实现，这一点可以通过成员函数 `is_lock_free` 来查询。

在 C++17 中，所有原子类型都有一个 `static constexpr` 的数据成员 `is_always_lock_free`，它指明了当前环境上的原子类型 X 是否一定是无锁的。

模板 `std::atomic` 可用任何满足可平凡复制的类型 `T` 实例化，可以使用 `load`、`store`、`exchange`、`compare_exchange_weak` 和 `compare_exchange_strong` 等成员函数对 `std::atomic` 进行操作。

> 如果是整数类型的特化，还支持 `++`、`--`、`+=`、`-=`、`&=`、`|=`、`^=` 、`fetch_add`、`fetch_sub` 等操作方式。

### std::atomic\<std::shared_ptr>

若多个线程同时操作同一个 `std::shared_ptr` 对象，且这些操作使用了 `std::shared_ptr` 的非 `const` 成员函数，则将出现数据竞争，除非通过 `std::atomic<std::shared_ptr>` 的实例进行所有操作。
