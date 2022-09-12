https://murphypei.github.io/blog/2019/04/cpp-std-ref

std::ref 用来取某个变量的引用, 这是因为某些c++的方法传参默认是拷贝, 所以需要 std::ref 来获取引用

std::bind 就是对参数进行拷贝

示例1:
```cpp
#include <functional>
#include <iostream>

void f(int& n1, int& n2, const int& n3)
{
    std::cout << "In function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    ++n1; // increments the copy of n1 stored in the function object
    ++n2; // increments the main()'s n2
    // ++n3; // compile error
}

int main()
{
    int n1 = 1, n2 = 2, n3 = 3;
    std::function<void()> bound_f = std::bind(f, n1, std::ref(n2), std::cref(n3));
    n1 = 10;
    n2 = 11;
    n3 = 12;
    std::cout << "Before function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    bound_f();
    std::cout << "After function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
}
```

输出
```
Before function: 10 11 12
In function: 1 11 12
After function: 10 12 12
```


std::thread 也同样是对参数进行拷贝

示例2:
```cpp
#include<thread>
#include<iostream>
#include<string>

void threadFunc(std::string &str, int a)
{
    str = "change by threadFunc";
    a = 13;
}

int main()
{
    std::string str("main");
    int a = 9;
    std::thread th(threadFunc, std::ref(str), a);

    th.join();

    std::cout<<"str = " << str << std::endl;
    std::cout<<"a = " << a << std::endl;

    return 0;
}
```

输出
```
str = change by threadFunc
a = 9
```