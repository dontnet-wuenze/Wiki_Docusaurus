可以通过 {} 控制作用域

因为 c++11 引入了智能指针, 会在作用域结束自动销毁, 所以我们可以利用 {} 控制对象的生命周期
例如 mutex
```cpp
void f()
{
   //some code - MULTIPLE threads can execute this code at the same time

   {
       scoped_lock lock(mutex); //critical section starts here

       //critical section code
       //EXACTLY ONE thread can execute this code at a time

   } //mutex is automatically released here

  //other code  - MULTIPLE threads can execute this code at the same time
}
```