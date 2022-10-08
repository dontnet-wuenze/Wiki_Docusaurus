# C++ Header File Include Patterns

Large software projects require a careful header file management even when programming in C. When developers move to C++, header file management becomes even more complex and time consuming. Here we present a few header file inclusion patterns that will simplify this chore.

## Header File Inclusion Rules

Here, we discuss the basic rules of C++ header file inclusion needed to simplify header file management.

- A header file should be included only when a forward declaration would not do the job.
- The header file should be so designed that the order of header file inclusion is not important. 
  - This is achieved by making sure that x.h is the first header file in x.cpp
- The header file inclusion mechanism should be tolerant to duplicate header file inclusions.
The following sections will explain these rules with the help of an example.

## Header File Inclusion Example

The following example illustrates different types of dependencies. Assume a class A with code stored in a.cpp and a.h.

**a.h**
```cpp
#ifndef _a_h_included_
#define _a_h_included_
#include "abase.h"
#include "b.h"

// Forward Declarations
class C;
class D;

class A : public ABase
{
  B m_b;
  C *m_c;
  D *m_d;
  
public:
  void SetC(C *c);
  C *GetC() const;
  
  void ModifyD(D *d);
};
#endif
```

**a.cpp**
```cpp
#include "a.h"
#include "d.h"

void A::SetC(C* c)
{
  m_c = c;
}

C* A::GetC() const
{
  return m_c;
}

void A::ModifyD(D* d)
{
  d->SetX(0);
  d->SetY(0);
  m_d = d;
}
```

## File Inclusion Analysis
Lets analyze the header file inclusions, from the point of view of classes involved in this example, i.e. ABase, A, B, C and D.

| |  |
| :-----| :-----|
| Class ABase | ABase is the base class, so the class declaration is required to complete the class declaration. The compiler needs to know the size of ABase to determine the total size of A. In this case **abase.h** should be included explicitly in **a.h**. |
| Class B | Class A contains Class B by value , so the class declaration is required to complete the class declaration. The compiler needs to know the size of B to determine the total size of A. In this case **b.h** should be included explicitly in **a.h**. | 
| Class C | Class C is included only as a pointer reference. The size or actual content of C are not important to **a.h** or **a.cpp**. Thus only a forward declaration has been included in **a.h**. Notice that c.h has not been included in either **a.h** or **a.cpp**. | 
| Class D | Class D is just used as a pointer reference in **a.h**. Thus a forward declaration is sufficient. But **a.cpp** uses class D in substance so it explicitly includes **d.h**. | 


## Key Points

- Header files should be included only when a forward declaration will not do the job. By not including c.h and d.h other clients of class A never have to worry about c.h and d.h unless they use class C and D by value.
- a.h has been included as the first header file in a.cpp This will make sure that a.h does not expect a certain header files to be included before a.h. As a.h has been included as the first file, successful compilation of a.cpp will ensure that a.h does not expect any other header file to be included before a.h.
  - If this is followed for all classes, (i.e. x.cpp always includes x.h as the first header) there will be no dependency on header file inclusion.
- a.h includes the check on preprocessor definition of symbol \_a_h_included_. This makes it tolerant to duplicate inclusions of a.h.

## Cyclic Dependency

Cyclic dependency exists between class X and Y in the following example. This dependency is handled by using forward declarations.

**x.h and y.h**
```cpp
/* ====== x.h ====== */
// Forward declaration of Y for cyclic dependency
class Y;

class X 
{
    Y *m_y;
    ...
};

/* ====== y.h ====== */
// Forward declaration of X for cyclic dependency
class X;

class Y 
{
    X *m_x;
    ...
};
```

