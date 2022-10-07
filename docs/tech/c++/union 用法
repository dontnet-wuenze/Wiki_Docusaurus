https://en.cppreference.com/w/cpp/language/union

Union 的主要目的应该是节省空间
一个 Union 中的变量共用一块空间


## 用法
``` cpp
#include <iostream>
#include <cstdint>
union S
{
    std::int32_t n;     // occupies 4 bytes
    std::uint16_t s[2]; // occupies 4 bytes
    std::uint8_t c;     // occupies 1 byte
};                      // the whole union occupies 4 bytes
 
int main()
{
    S s = {0x12345678}; // initializes the first member, s.n is now the active member
    // at this point, reading from s.s or s.c is undefined behavior
    std::cout << std::hex << "s.n = " << s.n << '\n';
    s.s[0] = 0x0011; // s.s is now the active member
    // at this point, reading from n or c is UB but most compilers define it
    std::cout << "s.c is now " << +s.c << '\n' // 11 or 00, depending on platform
              << "s.n is now " << s.n << '\n'; // 12340011 or 00115678
}
```

## Lifetime

当一个 Union 变量被赋值的时候这个变量就是 active member, 也就是说, 读取其他 member 的值是未定义的行为。
如果 Union 中的其他变量被 active 了, 之前的变量也就结束了。

``` cpp
union A { int x; int y[4]; };
struct B { A a; };
union C { B b; int k; };
int f() {
  C c;               // does not start lifetime of any union member
  c.b.a.y[3] = 4;    // OK: "c.b.a.y[3]", names union members c.b and c.b.a.y;
                     // This creates objects to hold union members c.b and c.b.a.y
  return c.b.a.y[3]; // OK: c.b.a.y refers to newly created object
}
 
struct X { const int a; int b; };
union Y { X x; int k; };
void g() {
  Y y = { { 1, 2 } }; // OK, y.x is active union member
  int n = y.x.a;
  y.k = 4;   // OK: ends lifetime of y.x, y.k is active member of union
  y.x.b = n; // undefined behavior: y.x.b modified outside its lifetime,
             // "y.x.b" names y.x, but X's default constructor is deleted,
             // so union member y.x's lifetime does not implicitly start
}
```

## Anonymous unions
这是一种特殊的用法, 不指定 Union 的名字, 所以可以直接用 Union 中的变量

``` cpp
int main()
{
    union
    {
        int a;
        const char* p;
    };
    a = 1;
    p = "Jennifer";
}
```

## Union-like classes
配合上面的 Anonymous union 来使用

```cpp
#include <iostream>
 
// S has one non-static data member (tag), three enumerator members (CHAR, INT, DOUBLE), 
// and three variant members (c, i, d)
struct S
{
    enum{CHAR, INT, DOUBLE} tag;
    union
    {
        char c;
        int i;
        double d;
    };
};
 
void print_s(const S& s)
{
    switch(s.tag)
    {
        case S::CHAR: std::cout << s.c << '\n'; break;
        case S::INT: std::cout << s.i << '\n'; break;
        case S::DOUBLE: std::cout << s.d << '\n'; break;
    }
}
 
int main()
{
    S s = {S::CHAR, 'a'};
    print_s(s);
    s.tag = S::INT;
    s.i = 123;
    print_s(s);
}
```
