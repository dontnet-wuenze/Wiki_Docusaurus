https://github.com/stanford-cs149/asst2/blob/master/tutorial/README.md

https://segmentfault.com/a/1190000006614695

## 线程(Thread)
```cpp
#include <thread>
#include <stdio.h>

void my_func(int thread_id, int num_threads) {
	printf("Hello from spawned thread %d of %d\n", thread_id, num_threads);
}

int main(int argc, char** argv) {

  std::thread t0 = std::thread(my_func, 0, 2);
  std::thread t1 = std::thread(my_func, 1, 2);

  printf("The main thread is running concurrently with spawned threads.\n");

  t0.join();
  t1.join();

  printf("Spawned threads have terminated at this point.\n");

  return 0;
}
```

### 创建线程
使用 thread 创建一个线程, 第一个参数是执行的函数, 后面是执行函数的参数

### 销毁线程
注意线程在销毁前需要 join 或者 detach, 否则会出现错误。

A thread object does not have an associated thread (and is safe to destroy) after:

- it was default-constructed
- it was moved from
- join() has been called
- detach() has been called

## 互斥锁(Mutexes)

### mutex.lock()
直接操作 mutex，即直接调用 mutex 的 lock / unlock 函数

```cpp
#include <condition_variable>
#include <mutex>
#include <thread>

#include <stdio.h>

/*
 Wrapper class around an integer counter and a mutex.
 */
class Counter {
    public:
        int counter_;
        std::mutex* mutex_;
        Counter() {
            counter_ = 0;
            mutex_ = new std::mutex();
        }
        ~Counter() {
            delete mutex_;
        }
};

void increment_counter_fn(Counter* counter) {
    for (int i = 0; i < 10000; i++) {
        // Call lock() method to acquire lock.
        counter->mutex_->lock();
        // Since multiple threads are trying to perform an increment, the
        // increment needs to be protected by a mutex.
        counter->counter_++;
        // Call unlock() method to release lock.
        counter->mutex_->unlock();
    }
}

/*
 * Threads increment a shared counter in a tight for loop 10,000 times.
 */
void mutex_example() {
    int num_threads = 8;

    printf("==============================================================\n");
    printf("Starting %d threads to increment counter...\n", num_threads);
    std::thread* threads = new std::thread[num_threads];
    Counter* counter = new Counter();
    // `num_threads` threads will call `increment_counter_fn`, trying to
    // increment `counter`.
    for (int i = 0; i < num_threads; i++) {
        threads[i] = std::thread(increment_counter_fn, counter);
    }
    // Wait for spawned threads to complete.
    for (int i = 0; i < num_threads; i++) {
        threads[i].join();
    }
    // Verify that final counter value is (10000 * `num_threads`).
    printf("Final counter value: %d...\n", counter->counter_);
    printf("==============================================================\n");

    delete counter;
    delete[] threads;
}
```

### lock_guard

可以使用封装好的 lock_guard, 类似智能指针, 函数结束会自动释放锁。
```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>

std::mutex g_mutex;
int g_count = 0;

void Counter() {
  // lock_guard 在构造函数里加锁，在析构函数里解锁。
  std::lock_guard<std::mutex> lock(g_mutex);

  int i = ++g_count;
  std::cout << "count: " << i << std::endl;
}

int main() {
  const std::size_t SIZE = 4;

  std::vector<std::thread> v;
  v.reserve(SIZE);

  for (std::size_t i = 0; i < SIZE; ++i) {
    v.emplace_back(&Counter);
  }

  for (std::thread& t : v) {
    t.join();
  }

  return 0;
}
```

### unique_lock

在 lock 上的用法和 lock_guard 相同, 但是 unique_lock 可以配合条件锁使用。

## 条件变量(Conditional Variables)

### 使用方法
首先获取 mutex 锁(使用 unique_lock), 然后调用 wait(lock)。这一步会使线程休眠并释放 mutex 资源。当 Conditional Variable 调用 .notify()时, 会唤醒一个线程。此时线程会重新获取到 mutex 锁(注意如果 mutex 不是空闲的, 线程依然会被唤醒, 但是要等待锁资源被其他线程释放)。

```cpp
/*
 * Wrapper class around a counter, a condition variable, and a mutex.
 */
class ThreadState {
    public:
        std::condition_variable* condition_variable_;
        std::mutex* mutex_;
        int counter_;
        int num_waiting_threads_;
        ThreadState(int num_waiting_threads) {
            condition_variable_ = new std::condition_variable();
            mutex_ = new std::mutex();
            counter_ = 0;
            num_waiting_threads_ = num_waiting_threads;
        }
        ~ThreadState() {
            delete condition_variable_;
            delete mutex_;
        }
};

void signal_fn(ThreadState* thread_state) {
    // Acquire mutex to make sure the shared counter is read in a
    // consistent state.
    thread_state->mutex_->lock();
    while (thread_state->counter_ < thread_state->num_waiting_threads_) {
        thread_state->mutex_->unlock();
        // Release the mutex before calling `notify_all()` to make sure
        // waiting threads have a chance to make progress.
        thread_state->condition_variable_->notify_all();
        // Re-acquire the mutex to read the shared counter again.
        thread_state->mutex_->lock();
    }
    thread_state->mutex_->unlock();
}

void wait_fn(ThreadState* thread_state) {
    // A lock must be held in order to wait on a condition variable.
    // This lock is atomically released before the thread goes to sleep
    // when `wait()` is called. The lock is atomically re-acquired when
    // the thread is woken up using `notify_all()`.
    std::unique_lock<std::mutex> lk(*thread_state->mutex_);
    thread_state->condition_variable_->wait(lk);
    // Increment the shared counter with the lock re-acquired to inform the
    // signaling thread that this waiting thread has successfully been
    // woken up.
    thread_state->counter_++;
    printf("Lock re-acquired after wait()...\n");
    lk.unlock();
}

/*
 * Signaling thread spins until each waiting thread increments a shared
 * counter after being woken up from the `wait()` method.
 */
void condition_variable_example() {
    int num_threads = 3;

    printf("==============================================================\n");
    printf("Starting %d threads for signal-and-waiting...\n", num_threads);
    std::thread* threads = new std::thread[num_threads];
    ThreadState* thread_state = new ThreadState(num_threads-1);
    threads[0] = std::thread(signal_fn, thread_state);
    for (int i = 1; i < num_threads; i++) {
        threads[i] = std::thread(wait_fn, thread_state);
    }
    for (int i = 0; i < num_threads; i++) {
        threads[i].join();
    }
    printf("==============================================================\n");

    delete thread_state;
    delete[] threads;
}
```

### wait 判断条件

wait 被 notify 触发时， 可以设置一个函数判断是否满足条件, 否则继续阻塞在 wait 中。

```cpp
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

mutex mutex_;
condition_variable condVar;
bool someCondition = false;

void waitThread() {
	unique_lock<mutex> lock(mutex_);
	cout << "acquire the mutex in waitThread" << endl;
	condVar.wait(lock, []{return someCondition;});
	cout << "wait end and get lock" << endl;
}

void signalThread() {
	cout << "in signalThread" << endl;
	unique_lock<mutex> lock(mutex_);
	cout << "acquire the mutex in signalThread" << endl;
	condVar.notify_all();
	cout << "notify the thread" << endl;
	cout << "hold the lock" << endl;
	std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	lock.unlock();
	someCondition = true;
	condVar.notify_all();
	return ;
}


int main()
{
	thread thread1 = thread(waitThread);
	//thread1.join();
	std::this_thread::sleep_for(std::chrono::milliseconds(1000));
	thread thread2 = thread(signalThread);
	thread1.join();
	thread2.join();
}
```