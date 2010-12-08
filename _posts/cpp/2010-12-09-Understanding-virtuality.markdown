---
layout: page_post
title: Understanding virtuality
categories: cpp
---
The simple case
---------------
![Example 1](/files/vtable1.png "Example 1")
{% highlight cpp %}
class Foo
{
public:
  Foo() : a(1), b(2), c(3) {}

public:
  void f() { std::cout << "Foo::f()" << std::endl; }
  void g() { std::cout << "Foo::g()" << std::endl; }

public:
  int a;
  int b;
  int c;
};

int main( int, char** )
{
  Foo foo;
  
  std::cout << "sizeof(Foo)= " << sizeof(Foo) << std::endl;
  std::cout << std::endl;
  std::cout << "(int*)&foo+0=    " << (int*)&foo+0    << std::endl;
  std::cout << "&foo.a=          " << &foo.a          << std::endl;
  std::cout << "foo.a=           " << foo.a           << std::endl;
  std::cout << "*((int*)&foo+0)= " << *((int*)&foo+0) << std::endl;
  std::cout << std::endl;
  std::cout << "&foo.b=          " << &foo.b          << std::endl;
  std::cout << "(int*)&foo+1=    " << (int*)&foo+1    << std::endl;
  std::cout << "foo.b=           " << foo.b           << std::endl;
  std::cout << "*((int*)&foo+1)= " << *((int*)&foo+1) << std::endl;
  std::cout << std::endl;
  std::cout << "&foo.c=          " << &foo.c          << std::endl;
  std::cout << "(int*)&foo+2=    " << (int*)&foo+2    << std::endl;
  std::cout << "foo.c=           " << foo.c           << std::endl;
  std::cout << "*((int*)&foo+2)= " << *((int*)&foo+2) << std::endl;
  
  return EXIT_SUCCESS;
}
{% endhighlight %}
Output:
    sizeof(Foo)= 12
    
    (int*)&foo+0=    0xbffff7c4
    &foo.a=          0xbffff7c4
    foo.a=           1
    *((int*)&foo+0)= 1
    
    &foo.b=          0xbffff7c8
    (int*)&foo+1=    0xbffff7c8
    foo.b=           2
    *((int*)&foo+1)= 2
    
    &foo.c=          0xbffff7cc
    (int*)&foo+2=    0xbffff7cc
    foo.c=           3
    *((int*)&foo+2)= 3

Introducing virtuality
----------------------
![Example 2](/files/vtable2.png "Example 2")
{% highlight cpp %}
class Foo
{
public:
  Foo() : a(1), b(2), c(3) {}

public:
  virtual void f() { std::cout << "Foo::f()" << std::endl; }
  virtual void g() { std::cout << "Foo::g()" << std::endl; }

public:
  int a;
  int b;
  int c;
};

int main( int, char** )
{
  Foo foo;
  
  std::cout << "sizeof(Foo)= " << sizeof(Foo) << std::endl;
  std::cout << std::endl;
  std::cout << "(int (***)(...))&foo+0=    " << (int (***)(...))&foo+0    << std::endl;
  std::cout << "&foo._vptr=                " << &foo._vptr                << std::endl;
  std::cout << "foo._vptr=                 " << foo._vptr                 << std::endl;
  std::cout << "*((int (***)(...))&foo+0)= " << *((int (***)(...))&foo+0) << std::endl;
  std::cout << std::endl;
  std::cout << "(int*)&foo+1=    " << (int*)&foo+1    << std::endl;
  std::cout << "&foo.a=          " << &foo.a          << std::endl;
  std::cout << "foo.a=           " << foo.a           << std::endl;
  std::cout << "*((int*)&foo+1)= " << *((int*)&foo+1) << std::endl;
  std::cout << std::endl;
  std::cout << "&foo.b=          " << &foo.b          << std::endl;
  std::cout << "(int*)&foo+2=    " << (int*)&foo+2    << std::endl;
  std::cout << "foo.b=           " << foo.b           << std::endl;
  std::cout << "*((int*)&foo+2)= " << *((int*)&foo+2) << std::endl;
  std::cout << std::endl;
  std::cout << "&foo.c=          " << &foo.c          << std::endl;
  std::cout << "(int*)&foo+3=    " << (int*)&foo+3    << std::endl;
  std::cout << "foo.c=           " << foo.c           << std::endl;
  std::cout << "*((int*)&foo+3)= " << *((int*)&foo+3) << std::endl;
  
  return EXIT_SUCCESS;
}
{% endhighlight %}

Output:
    sizeof(Foo)= 16
    
    (int (***)(...))&foo+0=    0xbffff7c0
    &foo._vptr=                0xbffff7c0
    foo._vptr=                 0x2040
    *((int (***)(...))&foo+0)= 0x2040
    
    (int*)&foo+1=    0xbffff7c4
    &foo.a=          0xbffff7c4
    foo.a=           1
    *((int*)&foo+1)= 1
    
    &foo.b=          0xbffff7c8
    (int*)&foo+2=    0xbffff7c8
    foo.b=           2
    *((int*)&foo+2)= 2
    
    &foo.c=          0xbffff7cc
    (int*)&foo+3=    0xbffff7cc
    foo.c=           3
    *((int*)&foo+3)= 3

{% highlight cpp %}
class Foo
{
public:
  Foo() : a(1), b(2), c(3) {}

public:
  virtual void f() { std::cout << "Foo::f()" << std::endl; }
  virtual void g() { std::cout << "Foo::g()" << std::endl; }

public:
  int a;
  int b;
  int c;
};

union MemberPtrMixin
{
  void (Foo::*pfn)(); // The real type of our Foo methods
  int (*pvt)(...);    // The global type of a vtable entry
};

int main( int, char** )
{
  Foo foo;
  
  std::cout << "foo.f()= "; (foo.*(&Foo::f))();
  std::cout << "foo.g()= "; (foo.*(&Foo::g))();
  
  std::cout << std::endl;
  
  std::cout << "(foo.*(&Foo::f))()= "; (foo.*(&Foo::f))();
  std::cout << "(foo.*(&Foo::g))()= "; (foo.*(&Foo::g))();
  
  std::cout << std::endl;
  
  MemberPtrMixin pmb_f;
  MemberPtrMixin pmb_g;
  pmb_f.pvt = foo._vptr[0];
  pmb_g.pvt = foo._vptr[1];
  std::cout << "(foo.*pmb_f.pfn)()= "; (foo.*pmb_f.pfn)();
  std::cout << "(foo.*pmb_g.pfn)()= "; (foo.*pmb_g.pfn)();
  
  return EXIT_SUCCESS;
}
{% endhighlight %}
Output:
    foo.f()= Foo::f()
    foo.g()= Foo::g()
    
    (foo.*(&Foo::f))()= Foo::f()
    (foo.*(&Foo::g))()= Foo::g()
    
    (foo.*pmb_f.pfn)()= Foo::f()
    (foo.*pmb_g.pfn)()= Foo::g()

Final example
-------------
![Example 3](/files/vtable3.png "Example 3")
{% highlight cpp %}
class Foo
{
public:
  Foo() : a(1), b(2), c(3) {}

public:
  virtual void f() { std::cout << "Foo::f()" << std::endl; }
  virtual void g() { std::cout << "Foo::g()" << std::endl; }

public:
  int a;
  int b;
  int c;
};

class Bar : public Foo
{
public:
  virtual void g() { std::cout << "Bar::g()" << std::endl; }
};

union MemberPtrMixin
{
  void (Bar::*pfn)(); // The real type of our Bar methods
  int (*pvt)(...);    // The global type of a vtable entry
};

int main( int, char** )
{
  Bar bar;
  
  std::cout << "bar.f()= "; (bar.*(&Bar::f))();
  std::cout << "bar.g()= "; (bar.*(&Bar::g))();
  
  std::cout << std::endl;
  
  std::cout << "(bar.*(&Bar::f))()= "; (bar.*(&Bar::f))();
  std::cout << "(bar.*(&Bar::g))()= "; (bar.*(&Bar::g))();
  
  std::cout << std::endl;
  
  MemberPtrMixin pmb_f;
  MemberPtrMixin pmb_g;
  pmb_f.pvt = bar._vptr[0];
  pmb_g.pvt = bar._vptr[1];
  std::cout << "(bar.*pmb_f.pfn)()= "; (bar.*pmb_f.pfn)();
  std::cout << "(bar.*pmb_g.pfn)()= "; (bar.*pmb_g.pfn)();
  
  return EXIT_SUCCESS;
}
{% endhighlight %}
Output:
    bar.f()= Foo::f()
    bar.g()= Bar::g()
    
    (bar.*(&Bar::f))()= Foo::f()
    (bar.*(&Bar::g))()= Bar::g()
    
    (bar.*pmb_f.pfn)()= Foo::f()
    (bar.*pmb_g.pfn)()= Bar::g()
