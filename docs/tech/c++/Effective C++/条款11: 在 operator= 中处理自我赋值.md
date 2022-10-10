## 证同测试

通过证同测试保证自我赋值的安全性

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
    if (this == &rhs) return *this;
    // ...

    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

## 异常安全性

上面的代码如果在 `delete pb` 之后，`new Bitmap(*rhs.pb)` 抛出异常，那么 `pb` 就会指向一个已经被释放的内存，这是不安全的。

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap* pOrig = pb;         // 记住原先的 pb
    pb = new Bitmap(*rhs.pb);   // 令 pb 指向一个新的 Bitmap
    delete pOrig;               // 删除原先的 pb
    return *this;
}
```

## copy and swap 技术

```cpp
class Widget {
    ...
    void swap(Widget& rhs);
    ...
};
Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs);   // 临时 Widget
    swap(temp);         // 交换 *this 和 temp
    return *this;
}
```

另一种写法利用了以下事实:

(1) 某 class 的 copy assignment 操作符可能被声明为 " 以 by value 方式接受实参 ";

(2) 以 by value 方式传递东西会造成一份复件/副本 :


```cpp
Widget& Widget::operator=(Widget rhs)
{
    swap(rhs);
    *this = temp;       // 交换 *this 和 temp
    return *this;
}
```

作者认为这种方法牺牲了清晰性, 但从函数本体移至"函数参数构造阶段" 却可令编译器生成更高效的代码。