---
layout: post
title: "Modifying Ubuntu packages: Nginx"
date: 2013-02-12 17:05
comments: true
categories: [Ubuntu]
published: false
---

## Prepare the system

as in the previous post

## Existing packages

Create a PPA, if necessary.

Use 'Copy packages' from LP.

Then add the packages to the sources:

    $ sudo apt-add-repository ppa:bcandrea/nginx-stable

Create a build directory and get the sources:

    $ mkdir -p ~/ppa/nginx-stable/nginx
    $ cd ~/ppa/nginx-stable/nginx
    $ apt-get source nginx
    $ cd nginx-1.2.4

## Add a new module to Nginx

    $ mkdir debian/modules/spnego-http-auth-nginx-module
    $ cd debian/modules/spnego-http-auth-nginx-module
    $ wget https://github.com/muhgatus/spnego-http-auth-nginx-module/archive/master.zip
    $ unzip master.zip

Modify debian/modules/README.Modules-versions.
Modify debian/rules.
Modify debian/control (Build-Depends: + libkrb5-dev)
Go to the base folder:

    $ cd ~/ppa/nginx-stable/nginx/nginx-1.2.4

Modify the changelog with `dch -v 1.2.4-2ubuntu0ppa1+spnego~precise`:

    nginx (1.2.4-2ubuntu0ppa1+spnego~precise) precise; urgency=low

      * Added Kerberos authentication by inclusion of 
        spnego-http-auth-nginx-module

     -- Andrea Bernardo Ciddio <bcandrea@gmail.com>  Tue, 12 Feb 2013 17:38:43 +0000


## Compile the deb package locally

From the root of the project

    $ debuild -S

and build:

    $ cd .. && mkdir buildresult
    $ sudo cowbuilder --build nginx_1.2.4-2ubuntu0ppa1+spnego~precise.dsc --basepath=$HOME/pbuilder/precise-base.cow --buildresult ./buildresult

all ok, then publish:

    $ dput ppa:bcandrea/nginx-stable nginx_1.2.4-2ubuntu0ppa1+spnego~precise_source.changes