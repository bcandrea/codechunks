---
layout: post
title: "Writing shared libraries with Autotools and Check"
date: 2012-11-01 14:58
comments: true
categories: [C]
published: false
---


Skeleton:

    $ sudo apt-get install automake libtool check
    $ mkdir -p libfoo/{build,build-aux,lib,m4,tests}
    $ cd libfoo
    $ echo "foo - A wonderful library." > README

./configure.ac:

    AC_INIT([libfoo], [0.1], [bcandrea@gmail.com])
    AC_CONFIG_AUX_DIR([build-aux])
    AC_CONFIG_MACRO_DIR([m4])
    AM_INIT_AUTOMAKE([foreign -Wall -Werror])
    LT_INIT
    AC_PROG_CC
    AM_PROG_CC_C_O
    PKG_CHECK_MODULES([CHECK], [check >= 0.9.4])
    AC_CONFIG_HEADERS([config.h])
    AC_CONFIG_FILES([Makefile lib/Makefile tests/Makefile])
    AC_OUTPUT

./Makefile.am:

    SUBDIRS = lib . tests
    ACLOCAL_AMFLAGS = -I m4

./lib/Makefile.am:

    lib_LTLIBRARIES = libfoo.la
    libfoo_la_SOURCES = foo.c foo.h

./tests/Makefile.am:

    TESTS = check_foo
    check_PROGRAMS = check_foo
    check_foo_SOURCES = check_foo.c $(top_builddir)/lib/foo.h
    check_foo_CFLAGS = @CHECK_CFLAGS@
    check_foo_LDADD = $(top_builddir)/lib/libfoo.la @CHECK_LIBS@

./lib/foo.h:

    #ifndef FOO_H
    #define FOO_H

    #endif /* FOO_H */

./lib.foo.c:

    // nothing

./tests/check_foo.c:

    int main (void)
    {
      return 0;
    }

    $ autoreconf --install
    $ cd build
    $ ../configure
    $ make check

    PASS: check_foo
    =============
    1 test passed
    =============

Implementing checks:


check_foo.c:

    #include <stdlib.h>
    #include <check.h>
    #include "../lib/foo.h"

    START_TEST(test_do_something)
    {
      fail_unless(do_something() == 42);
    }
    END_TEST

    Suite* foo_suite(void)
    {
      Suite *s = suite_create("Foo");

      /* Core test case */
      TCase *tc_core = tcase_create("Core");
      tcase_add_test(tc_core, test_do_something);
      suite_add_tcase(s, tc_core);

      return s;
    }

    int main(void)
    {
      int number_failed;
      Suite *s = foo_suite();
      SRunner *sr = srunner_create(s);
      srunner_run_all(sr, CK_NORMAL);
      number_failed = srunner_ntests_failed(sr);
      srunner_free(sr);
      return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
    }

Then add code to foo.h:

    #ifndef FOO_H
    #define FOO_H

    int do_something();

    #endif /* FOO_H */

And make it pass in foo.c:

    #include "foo.h"

    int do_something()
    {
        return 42;
    }


## References

http://check.sourceforge.net/doc/check_html/index.html
http://www.lrde.epita.fr/~adl/autotools.html