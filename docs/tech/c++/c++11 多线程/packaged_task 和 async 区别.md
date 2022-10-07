https://stackoverflow.com/questions/18143661/what-is-the-difference-between-packaged-task-and-async/18143844#18143844

如果一个函数运行时间很长, 就能发现区别, 例如
```cpp
//! sleeps for one second and returns 1
auto sleep = [](){
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return 1;
};
```

## packaged_task

对于一个 packaged_task, 需要手动调用
```cpp
std::packaged_task<int()> task(sleep);

auto f = task.get_future();
task(); // invoke the function

// You have to wait until task returns. Since task calls sleep
// you will have to wait at least 1 second.
std::cout << "You can see this after 1 second\n";

// However, f.get() will be available, since task has already finished.
std::cout << f.get() << std::endl;
```

## std::async
首先 async 会在一个其他线程中执行函数
```cpp
auto f = std::async(std::launch::async, sleep);
std::cout << "You can see this immediately!\n";

// However, the value of the future will be available after sleep has finished
// so f.get() can block up to 1 second.
std::cout << f.get() << "This will be shown after a second!\n";
```

### 缺点

async 某些时候会阻塞在 future 的析构阶段。

下面这个例子因为 async 的返回 future 没有被接收, 所以会发生 ~future, 因此会阻塞

```cpp
std::async(do_work1); // ~future blocks
std::async(do_work2); // ~future blocks

/* output: (assuming that do_work* log their progress)
    do_work1() started;
    do_work1() stopped;
    do_work2() started;
    do_work2() stopped;
*/
```

示例2:
```cpp
这个例子在 return 的时候 pizza 要被回收, 此时执行 ~future, 发生阻塞
{
    auto pizza = std::async(get_pizza);
    /* ... */
    if(need_to_go)
        return;          // ~future will block
    else
       eat(pizza.get());
}  
```

总的来说, 使用 packaged_task 更灵活, 是一个更基础的特性。