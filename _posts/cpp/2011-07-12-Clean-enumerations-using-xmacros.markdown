---
layout: page_post
title: Clean enumerations using xmacros
categories: cpp
---
Introduction
------------
In this short article, we will discuss a classical C++ engineering problem: storing meta data alongside enumeration values (such as their string equivalent). The solution we will end up with is will be based on so called "xmacros", a not so well known C-inherited trick.

The problem
-----------
Let's consider a simple case: You have to store some enumerations representing colors. Something of the kind:

{% highlight cpp %}
// --- color.h
enum Color
{
  RED,
  GREEN,
  BLUE,
  PURPLE,
  YELLOW,
  _COLOR_MAX
};
{% endhighlight %}

One day or another, you will have to dump these enumerations, either for debugging or logging purpose. A typical solution
would be to store their string equivalent in an other container, and remember to update it accordingly when you add a new
color.

{% highlight cpp %}
// --- color.h
#include <string>

enum Color
{
  RED,
  GREEN,
  BLUE,
  PURPLE,
  YELLOW,
  _COLOR_MAX
};

std::string colorName( Color c );

// --- color.cpp
#include "color.h"

#include <cassert>

static const std::string colorNames[] =
{
  "RED",
  "GREEN",
  "BLUE",
  "PURPLE",
  "YELLOW"
}

std::string colorName( Color c )
{
  assert( c<_COLOR_MAX );
  return colorNames[c%_COLOR_MAX];
}
{% endhighlight %}

Maintaining these meta data is a pain. You data is spread in multiple parts, and one could easily forget to maintain either
one of these containers.

X macros
--------
The whole idea behind X macros is to store the enumerations in macros instead of directly defining enumerations. Consider
the following example:

{% highlight cpp %}
enum Color
{
#define X(code) code,
  X( RED    )
  X( GREEN  )
  X( BLUE   )
  X( PURPLE )
#undef X
};
{% endhighlight %}

Going one step further, we could extract the `X(...)̀  declarations in a clean macro list outside the enumerations scope.

{% highlight cpp %}
#define COLOR_LIST \
  X( RED    )      \
  X( GREEN  )      \
  X( BLUE   )      \
  X( PURPLE )

enum Color
{
#define X(code) code,
  COLOR_LIST
#undef X
};
{% endhighlight %}

You should start to understand what we are doing here. By exporting the color list outside of the enumeration scope, we
now have the possibility to declare a similar container for color string equivalents!

{% highlight cpp %}
#define COLOR_LIST \
  X( RED    )      \
  X( GREEN  )      \
  X( BLUE   )      \
  X( PURPLE )

enum Color
{
#define X(code) code,
  COLOR_LIST
#undef X
};

static const std::string colorNames[] =
{
#define X(code) #code,
  COLOR_LIST
#undef X
};
{% endhighlight %}

Possible improvement: definition files
--------------------------------------
The C++ standard does not allow arbitrary length lines to be parsed. In fact, compliant compilers are only required to be
able to parse _at least_ 4k characters. This directly implies that we will not be able to store very long enumerations in
the same list macro. Most of the time, this problem is solved by exporting all xmacros in a separate `.def` file that you
can later include where you need it:

{% highlight cpp %}
// --- color.def
X( RED    )
X( BLUE   )
X( GREEN  )
X( PURPLE )
X( YELLOW )

// --- color.h
enum Color
{
#define X(code) code,
#  include "color.def"
#endif
};
{% endhighlight %}

Possible improvement: multiple meta data
----------------------------------------
Keep in mind that you do not need to deduce all needed meta data from a single xmacro argument, you can store whatever you need,
and tweak the xmacro definition to your likings:

{% highlight cpp %}
// --- color.def
X( RED,    "Red",   0xFF0000 )
X( GREEN,  "Green", 0x00FF00 )
X( BLUE,   "Blue",  0x0000FF )

// --- color.h
enum Color
{
#define X(code,name,mask) code,
#  include "color.def"
#endif
};

static const std::string colorNames[] =
{
#define X(code,name,mask) name,
#  include "color.def"
#undef X
};

static unsigned int colorMasks[] =
{
#define X(code,name,mask) mask,
#  include "color.def"
#undef X
};
{% endhighlight %}

A complete example
------------------
Here is a complete example of the use of xmacros. We use the _NARGS trick_
(https://groups.google.com/d/msg/comp.std.c/d-6Mj5Lko_s/5R6bMWTEbzQJ) to retrieve the number of colors without having to
pollute the `Color` enumeration with _MAX_COLORS.

{% highlight cpp %}
// --- color.h
#ifndef COLOR_H
#define COLOR_H

#include <string>

#define COLOR_LIST \
  X( RED    )      \
  X( GREEN  )      \
  X( BLUE   )      \
  X( PURPLE )      \
  X( ORANGE )      \
  X( YELLOW )

enum Color
{
#define X(code) code,
  COLOR_LIST
#undef X
};

/**
 * Returns the string equivalent of a color code
 */
std::string colorName( Color c );

/**
 * Returns true if c maps to a valid color code, false otherwise
 */
bool colorValid( unsigned int c );

/**
 * Returns the total count of existing color codes
 */
std::size_t colorCount();

#endif // COLOR_H

// --- color.cpp
#include "color.h"

#define NARG(...)  NARG_(__VA_ARGS__,RSEQ_N())
#define NARG_(...) ARG_N(__VA_ARGS__)
#define ARG_N(                             \
   _1, _2, _3, _4, _5, _6, _7, _8, _9,_10, \
  _11,_12,_13,_14,_15,_16,_17,_18,_19,_20, \
  _21,_22,_23,_24,_25,_26,_27,_28,_29,_30, \
  _31,_32,_33,_34,_35,_36,_37,_38,_39,_40, \
  _41,_42,_43,_44,_45,_46,_47,_48,_49,_50, \
  _51,_52,_53,_54,_55,_56,_57,_58,_59,_60, \
  _61,_62,_63,N,...) N

#define RSEQ_N()                \
 63,62,61,60,                   \
 59,58,57,56,55,54,53,52,51,50, \
 49,48,47,46,45,44,43,42,41,40, \
 39,38,37,36,35,34,33,32,31,30, \
 29,28,27,26,25,24,23,22,21,20, \
 19,18,17,16,15,14,13,12,11,10, \
  9, 8, 7, 6, 5, 4, 3, 2, 1, 0

static const std::string colorNames[] =
{
#define X(code) #code,
  COLOR_LIST
#undef X
};

std::string colorName( Color c )
{
  return colorNames[c];
}

bool colorValid( unsigned int c )
{
  return c<colorCount();
}

std::size_t colorCount()
{
#define X(code) code,
  return NARG(COLOR_LIST)-1;
#undef X
}

// --- main.cpp
#include <iostream>
#include <cstdlib>

#include "color.h"

int main( int argc, char** argv )
{
  std::cout << "Red   code is " << RED   << std::endl;
  std::cout << "Green code is " << GREEN << std::endl;
  std::cout << "Blue  code is " << BLUE  << std::endl;
  std::cout << std::endl;
  std::cout << "Red   name is " << colorName(RED  ) << std::endl;
  std::cout << "Green name is " << colorName(GREEN) << std::endl;
  std::cout << "Blue  name is " << colorName(BLUE ) << std::endl;
  std::cout << std::endl;
  std::cout << "Code 0 is valid? " << colorValid(0) << std::endl;
  std::cout << "Code 5 is valid? " << colorValid(5) << std::endl;
  std::cout << "Code 6 is valid? " << colorValid(6) << std::endl;
  std::cout << std::endl;
  std::cout << "Total color count: " << colorCount() << std::endl;
  
  return EXIT_SUCCESS;
}
{% endhighlight %}

