---
layout: page_post
title: Clean project management with CMake
categories: tutorials
---
{% highlight cpp %}
code
{% endhighlight %}
<pre class="console">
output
</pre>

Introduction
------------
This article will discuss the use of CMake to manage your application or library. I will go through the steps of building, documenting, testing and deploying your project.

CMake
-----
> CMake is a unified, cross-platform, open-source build system that allows developers to build, test and package software by specifying build parameters in simple, portable text files. It works in a compiler-independent manner and the build process works in conjunction with native build environments, such as _Make_, _Xcode_, _Code Blocks_ and _Visual Studio_. It also has minimal dependencies, C++ only. CMake is open source software and is developed by Kitware.
> 
> CMake is a robust, versatile tool that can:
> 
> * Create libraries
> * Generate wrappers
> * Compile source code
> * Build executables in arbitrary combinations

The most important thing to note is that CMake is IDE agnostic. You don't have to force your co-workers to use a specific IDE or compiler, CMake will generate a project file for any compiler, any IDE, and on any platform that one may think of.

Application overview
--------------------

First, let's define our awesome application. It is basically composed of two classes: SimpleProject1 and SimpleProject2.

__sampleproject1.h__:
{% highlight cpp %}
#ifndef SAMPLEPROJECT1_H
#define SAMPLEPROJECT1_H

class SampleProject1
{
public:
  SampleProject1();
  ~SampleProject1();
  
public:
  void function();
};

#endif // SAMPLEPROJECT1_H
{% endhighlight %}

__sampleproject1.cpp__:
{% highlight cpp %}
#include "sampleproject1.h"

#include <iostream>

SampleProject1::SampleProject1()
{
  std::cout << "SampleProject1::SampleProject1()" << std::endl;
}

SampleProject1::~SampleProject1()
{
  std::cout << "SampleProject1::~SampleProject1()" << std::endl;
}

void SampleProject1::function()
{
  std::cout << "SampleProject1::function()" << std::endl;
}
{% endhighlight %}

__sampleproject2.h__:
{% highlight cpp %}
#ifndef SAMPLEPROJECT2_H
#define SAMPLEPROJECT2_H

class SampleProject2
{
public:
  SampleProject2();
  ~SampleProject2();
  
public:
  void function();
};

#endif // SAMPLEPROJECT2_H
{% endhighlight %}

__sampleproject2.cpp__:
{% highlight cpp %}
#include "sampleproject2.h"

#include <iostream>

SampleProject2::SampleProject2()
{
  std::cout << "SampleProject2::SampleProject2()" << std::endl;
}

SampleProject2::~SampleProject2()
{
  std::cout << "SampleProject2::~SampleProject2()" << std::endl;
}

void SampleProject2::function()
{
  std::cout << "SampleProject2::function()" << std::endl;
}
{% endhighlight %}

__main.cpp__:
{% highlight cpp %}
#include <cstdlib>
#include "sampleproject1.h"
#include "sampleproject2.h"

int main( int, char** )
{
  SampleProject1 sp1;
  SampleProject2 sp2;
  
  sp1.function();
  sp2.function();
  
  return EXIT_SUCCESS;
}
{% endhighlight %}

Nothing fancy here, we have five files: _sampleproject1.h_, _sampleproject1.cpp_, _sampleproject2.h_, _sampleproject2.cpp_, and main.cpp.

Our project directory should look like the following:

* SampleProject
* * sampleproject1.h
* * sampleproject1.cpp
* * sampleproject2.h
* * sampleproject2.cpp
* * main.cpp


Building with CMake
-------------------

It is now time to look at how to build this project.
CMake uses configuration files named _CMakeLists.txt_ containing variables and instructions to build an application.

### A first attempt

Create a file 

Handling platform-specific issues
---------------------------------

### OS-specific configuration
### Compiler-specific configuration

Managing resources
------------------

Documenting with Doxygen
------------------------

Testing with CTest
------------------

### Using boost.unit

Deploying with CPack
--------------------

