---
layout: page_post
title: Return value optimization
categories: cpp
---
Introductory example
--------------------
Consider the following C++ code:

{% highlight cpp %}
struct Foo
{
  Foo()
  {
    std::cout << "Foo() @" << this << std::endl;
  }
  
  Foo( const Foo& )
  {
    std::cout << "Foo( const Foo& ) @" << this << std::endl;
  }
};

Foo function()
{
  Foo f_function; // (1)
  return f_function; // (2)
}

int main( int, char** )
{
  Foo f_main = function(); // (3)
  
  return EXIT_SUCCESS;
}
{% endhighlight %}

What would you expect this code snippet to output ?

The answer is:

    Foo() @0x11111111
    Foo( const Foo& ) @0x22222222
    Foo( const Foo& ) @0x33333333

* The first default constructor happens when `f_function` is created at (1).
* The second constructor, the copy one, is called when the return value from `function()` at (2) is copied to a temporary at (3).
* The third copy constructor is called when `f_main` is constructed from the temporary at (3).

Now if we try to build and run this snippet using `g++ test.cpp -o test && ./test`
The final output is quite disappointing:

    Foo() @0x11111111

A look at what just happened
----------------------------

The compiler just optimized away all copy constructor calls. This may seem actually quite weird, since this kind of optimization is:

* not safe (the "print" side effect of our copy constructors are not respected)
* not required (we did not ask g++ to optimize our code, it should default to `-O0`)

The C++ standard clearly states that any optimization performed by a compiler should respect the _as if_ rule (§1.9):

> An implementation is free to disregard any requirement of this International Standard as long as the result is as if the requirement had been obeyed, as far as can be determined from the observable behavior of the program.
> For instance, an actual implementation need not evaluate part of an expression if it can deduce that its value is not used and that no side effects affecting the observable behavior of the program are produced.

Let's look at the g++ assembly output using `g++ -c test.o && otool -tV test.o` (`otool` is the OSX equivalent of `objdump`):
{% highlight cpp-objdump %}
function()
{
  pushl %ebp            ; 
  movl  %esp,%ebp       ; 
  pushl %esi            ; 
  subl  $0x24,%esp      ; 
  movl  0x08(%ebp),%esi ; Address of instance on the stack
  movl  %esi,%eax       ; is loaded in eax and pushed on
  movl  %eax,(%esp)     ; the stack again (g++ thiscall convention)
  calll 0x00000230      ; Symbol stub for Foo()
  movl  %esi,%eax       ; 
  addl  $0x24,%esp      ; 
  popl  %esi            ; 
  leave                 ; 
  ret $0x0004           ; 
}

main()
{
  pushl %ebp             ; 
  movl  %esp,%ebp        ; 
  subl  $0x28,%esp       ; 
  leal  0xf7(%ebp),%eax  ; Load address of the Foo 
  movl  %eax,(%esp)      ; instance on the stack
  calll function         ; Call function()
  subl  $0x04,%esp       ; 
  movl  $function,%eax   ; 
  leave                  ; 
  ret                    ; 
}
{% endhighlight %}

I cleaned and commented the output for more clarity. The result is quite obvious: no object copying is involved, everything is done on the exact same `Foo` instance.

Some explanations
-----------------

It is now time to give some explanations on what happened here. It is one of the only exceptions to the _as if_ rule know as the _return value optimization_, or _RVO_.
It allows the compiler, in case a function returns by constructor, to reserve the instance on the stack of the caller, and pass the address to the callee.
This is somewhat similar to the following code:
{% highlight c++ %}
struct Foo
{
  Foo()
  {
    std::cout << "Foo() @" << this << std::endl;
  }
  
  Foo( const Foo& )
  {
    std::cout << "Foo( const Foo& ) @" << this << std::endl;
  }
};

Foo* function(Foo* f_function)
{
  return f_function;
}

int main( int, char** )
{
  Foo f_main;
  function(&f_main);
  
  return EXIT_SUCCESS;
}
{% endhighlight %}

Side notes
----------

Now this is just a trivial case of _RVO_. There is a more advanced version of this optimization that can be applied on named value, this is known as the _named return value optimization_, or _NRVO_.

Sometimes we really don't want g++ to perform such optimizations, hopefully, we can disallow it with the following parameter: `-fno-elide-constructors`.

Building the first example of this article with `g++ test.cpp -fno-elide-constructors -o test && ./test` now returns the expected output:

    Foo() @0xbffff79f
    Foo( const Foo& ) @0xbffff7cf
    Foo( const Foo& ) @0xbffff7ce

Things to remember
------------------

There is one really important thing to remember from all what we have just seen.

> A copy constructor shall __NOT__ have any side effect, since RVOs and NRVOs may prevent its call.

Such optimizations are not compiler-specific nor optimization-level-specific, they are granted to be possibly applied by the C++ standard. That is why we should either avoid return-by-value functions, or side-effect copy constructors. That is to be remembered when implementing RAII.