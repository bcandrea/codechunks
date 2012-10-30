---
layout: post
title: "CMake 2.8.5 in Lucid Lynx"
date: 2012-10-30 16:56
comments: true
categories: [Ubuntu]
published: false
---

Sometimes backporting a package is not a trivial task (quite often, actually). When the package in the source 
distribution has dependencies that cannot be satisfied in the destination, tools such as `backportpackage` 
(see [my previous post]({% post_url 2012-10-28-backporting-ubuntu-packages %})) are of little use.

<!-- more -->

## CMake, again

I tried to follow the [steps described here]({% post_url 2012-10-28-backporting-ubuntu-packages %})
to backport CMake 2.8.5 to Lucid Lynx, with no luck.

    The following packages have unmet dependencies:
      pbuilder-satisfydepends-dummy: Depends: debhelper (>= 8) but it is not installable

Tried to edit debian files, no luck

Create /etc/apt/sources.list.d/lucid.list:

    deb-src http://gb.archive.ubuntu.com/ubuntu/ lucid main restricted
    deb-src http://gb.archive.ubuntu.com/ubuntu/ lucid-backports main restricted


    $ apt-get source cmake=2.8.1-4~lucid1

    $ tar xzvf cmake_2.8.5.orig.tar.gz # taken from the other backported package

    $ tar xzvf ../cmake_2.8.1-4~lucid1.debian.tar.gz
    
    $ rm debian/patches -r


Problem: deps:

    -- Could NOT find BZip2 (missing:  BZIP2_LIBRARIES BZIP2_INCLUDE_DIR) 
    -- Could NOT find LibArchive (missing:  LibArchive_LIBRARY LibArchive_INCLUDE_DIR) 
    CMake Error at CMakeLists.txt:326 (MESSAGE):
      CMAKE_USE_SYSTEM_LIBARCHIVE is ON but LibArchive is not found!
    Call Stack (most recent call first):
      CMakeLists.txt:520 (CMAKE_BUILD_UTILITIES)


Added dependencies in debian/control: 
    
    Build-Depends: .... libbz2-dev, libarchive-dev

Test failed:

    The following tests FAILED:
      134 - CTestTestUpload (Failed)
    Errors while running CTest
    make[2]: *** [test] Error 8

Changed debian/rules (mind the TABs, this is a Makefile):

    override_dh_auto_test:
      # disable CTestTestUpload as it requires internet access
      HOME="`pwd`/Build" dh_auto_test -- ARGS="-E CTestTestUpload"

And the changelog

    $ dch -v 2.8.5-1ubuntu1~lucid1~ppa1

    cmake (2.8.5-1ubuntu1~lucid1~ppa1) lucid; urgency=low

      * New upstream release packaged for lucid. Removed previous patches.
      * Disabled CTestTestUpload test as it requires internet access.

     -- Andrea Bernardo Ciddio <bcandrea@gmail.com>  Tue, 30 Oct 2012 15:59:08 +0000

Now `debuild -S` from the cmake-2.8.5 folder

And build the package:

    $ cd .. && mkdir buildresult
    $ sudo cowbuilder --build cmake_2.8.5-1ubuntu1~lucid1~ppa1.dsc --basepath=$HOME/pbuilder/lucid-base.cow --buildresult ./buildresult

Upload:

    $ dput ppa:bcandrea/backports cmake_2.8.5-1ubuntu1~lucid1~ppa1_source.changes