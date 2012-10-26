---
layout: post
title: "A Makefile for the simplest shared library"
date: 2012-10-24 17:00
comments: true
categories: [C]
---

The problem: writing a very simple Makefile to build a C shared library.
Let's say we have just coded the most wonderful library `foo`, which -- not too surprisingly -- has 
greetings as its main purpose. 

<!-- more -->

{% gist 3947197 foo.h %}

{% gist 3947197 foo.c %}

(Yeah, pretty impressive). We may also want to push it further, and test it using a quick C executable:

{% gist 3947197 foo_test.c %}

Here's a simple Makefile with some nice features:

* the name and version of the library are just parameters
* it creates a shared object with a proper soname
* it compiles the library with debugging symbols, optimization and strict error checking

{% gist 3947197 basic.mk %}

Nice. But what about our little test program? Let's add some clutter to the minimal Makefile:

{% gist 3947197 complete.mk %}

The complete version has a `test` target which builds the `foo_test` executable (only if needed) and 
launches it (always). There is an intermediate step (the one involving `ldconfig`) which takes care of
creating some symlinks needed to link against our `foo` library. 

## References
[ Program Library HOWTO ](http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html) - The section about shared libraries

[ Shared objects for the object disoriented! ](http://www.ibm.com/developerworks/library/l-shobj/) - Quite old (2001) but very informative article

[ Simple makefile for compiling a shared object library file ](http://stackoverflow.com/questions/10803109/simple-makefile-for-compiling-a-shared-object-library-file) - Found on Stackoverflow (the library is written in C++, but the Makefile is adaptable to C with few modifications)