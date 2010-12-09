---
layout: page_post
title: Overview of the strict aliasing rules
categories: cpp
---
Understanding aliasing and restriction
--------------------------------------
We will start our journey by digging an obscure paragraph from the _ISO C89 standard_ (propagated to the following C and C++ standards). This paragraph (6.5 §7) describes what are known as the _strict aliasing rules_:
> An object shall have its stored value accessed only by an lvalue expression that has one of the following types:
> 
> * a type compatible with the effective type of the object, 
> * a qualified version of a type compatible with the effective type of the object, 
> * a type that is the signed or unsigned type corresponding to the effective type of the object, 
> * a type that is the signed or unsigned type corresponding to a qualified version of the effective type of the object, 
> * an aggregate or union type that includes one of the aforementioned types among its members (including, recursively, a member of a subaggregate or contained union), or 
> * a character type. 

At first, it may seem a bit obscure. What it really means, is that you cannot freely alias (or interleave, overlap) pointers and dereference them. For instance, the following code is illegal:
{% highlight cpp %}
uint16_t a;
uint8_t bytes = (uint8_t*)&a;
bytes[0] = 1;
bytes[1] = 1;
{% endhighlight %}
Accessing parts of a `uint16_t` from `uint8_t` does not fall in any strict-aliasing rules mentioned above. This is what is called _type punning_.

If we list the rules one after the other, here are some of the allowed ways to access our `uint16_t`:

- `uint16_t*`
- `uint16_t* const`
- `uint16_t* volatile`
- `uint16_t* const volatile`
- `int16_t*`
- `int16_t* const`
- `int16_t* volatile`
- `int16_t* const volatile`
- `union { uint16_t a; const int16_t b; }*`
- `char*`
- `unsigned char*`

Okay, so now that we have some knowledge on the subject, there is a direct consequence of the strict aliasing rules:

> Two pointers which types does not fall in the strict aliasing rules shall not alias.

This implication is __very__ important for understanding the _restrict pointers_ concept. The compiler will constantly make assumptions about pointer aliasing to try to perform optimizations. The more information are available, the more optimizations will be possible. Thus, the worsts cases are when we are manipulating pointers of _aliasable_ types - `char*` and `unsigned char*` being aliasable with everything, they are to be avoided.

Hopefully, ISO C99 introduced the `restrict` keyword:

> An object that is accessed through a _restrict-qualified_ pointer has a special association with that pointer. This association requires that all accesses to that object use, directly or indirectly, the value of that particular pointer. The intended use of the _restrict qualifier_ is to promote optimization, and deleting all instances of the qualifier from all preprocessing translation units composing a conforming program does not change its meaning (i.e., _observable behavior_).

As a side note, C++ does not (at he time this article was written) supports `restrict` pointers (since C99 is not part of C++98/03). You can nevertheless use `__restrict__` or `__restrict` on most modern compilers (GCC/MSVC/ICC) without any problem.

A simple example
----------------
Let's have a look at a very simple example:
{% highlight cpp %}
int function( int* a, int* b )
{
  *a = 1;
  *b = 2;
  return *a;
}
{% endhighlight %}
Two cases are possible here:

- `a` and `b` may point to the same memory location. Then we cannot know what to return without actually loading back `*a` from memory.
- `a` and `b` points to different memory locations. Then we would like the function to constantly return 1.

Building the code with `g++ -S -c aliasing.cpp -fstrict-aliasing -O3` should produce the following output (g++ 4.0.1)
{% highlight gas %}
function:
  pushl %ebp           #
  movl  %esp, %ebp     #
  movl  8(%ebp), %eax  # %eax = a
  movl  12(%ebp), %edx # %edx = b
  movl  $1, (%eax)     # *a = 1
  movl  $2, (%edx)     # *b = 2
  movl  (%eax), %eax   # Reload *a in %eax (return register)
  leave                #
  ret                  #
{% endhighlight %}
The output clearly illustrates the case 1 mentioned above. Without aliasing information about `a` and `b` the compiler cannot produce optimal code.

Now if we are sure that `a` and `b` will never alias, we can declare them as `restrict`:
{% highlight cpp %}
int function( int* __restrict__ a, int* __restrict__ b )
{
  *a = 1;
  *b = 2;
  return *a;
}
{% endhighlight %}
Assembly output:
{% highlight gas %}
function:
  pushl %ebp           #
  movl  %esp, %ebp     #
  movl  8(%ebp), %eax  # %eax = a
  movl  $1, (%eax)     # *a = 1
  movl  12(%ebp), %eax # %eax = b
  movl  $2, (%eax)     # *b = 2
  movl  $1, %eax       # return 1 in %eax (return register)
  leave                #
  ret                  #
{% endhighlight %}
This is what we expected, the compiler was able to remove the useless load of `*a` from memory.

A more complex example
----------------------
Let's now consider a more subtle example. You should know that the worst speed efficient operations possible are those involving loads and stores from and to memory. This means that to obtain the best possible results, we should try to follow the _Load -> Compute -> Store_ principle that maximizes prefetching and prevents pipeline stalls.

In the following example, we are looping reading and writing aliasable pointers. Without further restrict information, the compiler will have to perform a load _during each loop iteration_, this is the worst possible scenario.
{% highlight cpp %}
void function( int* a, int* b )
{
  for ( int i=0; i<10; ++i )
    *a += *b;
}
{% endhighlight %}
Assembly output:
{% highlight gas %}
function:
  pushl %ebp           # backup stack pointer
  xorl  %edx, %edx     # loop register %edx = 0
  movl  %esp, %ebp     # setup new stack
  pushl %esi           # backup %esi
  movl  8(%ebp), %esi  # %esi = a (from stack)
  movl  12(%ebp), %ecx # %ecx = b (from stack)
  movl  (%esi), %eax   # %eax = *a
loop:                  #
  addl  (%ecx), %eax   # Load *b and add it to %eax
  incl  %edx           # increment loop register %edx
  cmpl  $10, %edx      # comapre %edx to 10
  movl  %eax, (%esi)   # store %eax to *a
  jne   loop           # loop if %edx!=10
  popl  %esi           # restore %esi
  leave                #
  ret                  #
{% endhighlight %}

Now if we know `a` and `b` won't ever alias, we can try the following code out:
{% highlight cpp %}
void function( int* __restrict__ a, int* __restrict__ b )
{
  for ( int i=0; i<10; ++i )
    *a += *b;
}
{% endhighlight %}
Assembly output:
{% highlight gas %}
function:
  pushl %ebp              # backup stack pointer
  movl  %esp, %ebp        # setup new stack
  movl  12(%ebp), %eax    # %eax = b
  pushl %edi              # backup %edi
  movl  8(%ebp), %edi     # %edi = a
  pushl %esi              # backup %esi
  xorl  %esi, %esi        # loop register %esi = 0
  movl  (%eax), %eax      # %eax = *b
  movl  (%edi), %ecx      # %ecx = *a
loop:                     #
  incl  %esi              # increment loop register %esi
  leal  (%ecx,%eax), %edx # %edx = %ecx+%eax
  cmpl  $10, %esi         # compare %esi to 10
  movl  %edx, %ecx        # %edx = %ecx
  jne   loop              # loop if %esi!=10
  movl  %edx, (%edi)      # store %edx to *a
  popl  %esi              # restore %esi
  popl  %edi              # restore %edi
  leave                   #
  ret                     #
{% endhighlight %}
We were clearly able to extract the store from the loop, following the _Load -> Compute -> Store_ principle.

Conclusion
----------
Strict aliasing rules allows the compiler to perform some optimizations by assuming that some pointers do not alias. However, the programmer has to handle special cases when dealing with aliasable types. This involves tricky specification of restricted and non-restricted pointers. Matrix multiplications are a typical example where left and right hand side pointers will not alias, and agressive restrict pointer optimizations will be very efficient.