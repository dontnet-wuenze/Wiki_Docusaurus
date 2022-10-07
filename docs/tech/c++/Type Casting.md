https://cplusplus.com/doc/oldtutorial/typecasting/

我们首先有两种简单的类型转换的办法

## 隐式转换

如果一个类有相应的构造函数, 则会发生隐式转换
    
``` cpp
class A {};
class B { public: B (A a) {} };

A a;
B b=a;
```

## 显式转换

可以使用 C 风格或者函数式的转换

``` cpp
short a=2000;
int b;
b = (int) a;    // c-like cast notation
b = int (a);    // functional notation 
```

c++ 提供了四种转换方法

分别是
dynamic_cast <new_type> (expression)

reinterpret_cast <new_type> (expression)

static_cast <new_type> (expression)

const_cast <new_type> (expression)

## dynamic_cast

dynamic_cast只能用于指针和对象的引用。

把一个子类转换成基类是可以的
``` cpp
class CBase { };
class CDerived: public CBase { };

CBase b; CBase* pb;
CDerived d; CDerived* pd;

pb = dynamic_cast<CBase*>(&d);     // ok: derived-to-base
pd = dynamic_cast<CDerived*>(&b);  // wrong: base-to-derived 
```

只有当一个类是多态的, 才有可能将一个基类转换成子类, 否则像上面的第二句会出现编译错误。

当一个类是多态的，dynamic_cast在运行时进行特殊的检查，以确保表达式产生一个有效的完整的请求类的对象。

```cpp
// dynamic_cast
#include <iostream>
#include <exception>
using namespace std;

class CBase { virtual void dummy() {} };
class CDerived: public CBase { int a; };

int main () {
  try {
    CBase * pba = new CDerived;
    CBase * pbb = new CBase;
    CDerived * pd;

    pd = dynamic_cast<CDerived*>(pba);
    if (pd==0) cout << "Null pointer on first type-cast" << endl;

    pd = dynamic_cast<CDerived*>(pbb);
    if (pd==0) cout << "Null pointer on second type-cast" << endl;

  } catch (exception& e) {cout << "Exception: " << e.what();}
  return 0;
}
```

## static_cast

static_cast 可以执行从基类到子类的转换而不进行安全检查。

```cpp
class CBase {};
class CDerived: public CBase {};
CBase * a = new CBase;
CDerived * b = static_cast<CDerived*>(a);
```

上面的代码会导致运行时错误。

## reinterpret_cast

reinterpret_cast将任何指针类型转换为任何其他指针类型，甚至是不相关的类的指针。操作结果是将一个指针的值简单地二进制复制到另一个指针。所有的指针转换都是允许的：所指向的内容和指针类型本身都不被检查。

``` cpp
class A {};
class B {};
A * a = new A;
B * b = reinterpret_cast<B*>(a);
```

## const_cast

const_cast 操作一个对象的常量，要么转换成常量，要么去除常量。例如，为了将一个常量参数传递给一个非常量参数的函数。

```cpp
// const_cast
#include <iostream>
using namespace std;

void print (char * str)
{
  cout << str << endl;
}

int main () {
  const char * c = "sample text";
  print ( const_cast<char *> (c) );
  return 0;
}
```

## typeid

查看变量的类型

```cpp
// typeid
#include <iostream>
#include <typeinfo>
using namespace std;

int main () {
  int * a,b;
  a=0; b=0;
  if (typeid(a) != typeid(b))
  {
    cout << "a and b are of different types:\n";
    cout << "a is: " << typeid(a).name() << '\n';
    cout << "b is: " << typeid(b).name() << '\n';
  }
  return 0;
}
```

当typeid被应用于类时，typeid使用RTTI来跟踪动态对象的类型。当typeid被应用于类型为多态类的表达式时，其结果是最派生的完整对象的类型。

```cpp
// typeid, polymorphic class
#include <iostream>
#include <typeinfo>
#include <exception>
using namespace std;

class CBase { virtual void f(){} };
class CDerived : public CBase {};

int main () {
  try {
    CBase* a = new CBase;
    CBase* b = new CDerived;
    cout << "a is: " << typeid(a).name() << '\n';
    cout << "b is: " << typeid(b).name() << '\n';
    cout << "*a is: " << typeid(*a).name() << '\n';
    cout << "*b is: " << typeid(*b).name() << '\n';
  } catch (exception& e) { cout << "Exception: " << e.what() << endl; }
  return 0;
}
```

注意, 当类中有虚函数时, 使用的是运行时动态的类型检查。如果类中没有虚函数, 则是静态的编译器的类型。


