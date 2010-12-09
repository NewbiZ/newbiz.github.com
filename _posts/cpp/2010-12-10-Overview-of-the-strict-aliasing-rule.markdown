---
layout: page_post
title: Overview of the strict aliasing rule
categories: cpp
---
{% highlight bash %}
g++ -S -c aliasing.cpp -fstrict-aliasing -O3
{% endhighlight %}

A simple example
----------------
{% highlight cpp %}
int function( int* a, int* b )
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
  movl  12(%ebp), %edx # %edx = b
  movl  $1, (%eax)     # *a = 1
  movl  $2, (%edx)     # *b = 2
  movl  (%eax), %eax   # return *a
  leave                #
  ret                  #
{% endhighlight %}

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
  movl  $1, %eax       # return 1
  leave                #
  ret                  #
{% endhighlight %}

A more complex example
----------------------
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