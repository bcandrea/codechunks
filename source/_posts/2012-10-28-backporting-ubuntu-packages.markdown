---
layout: post
title: "Backporting Ubuntu packages: CMake"
date: 2012-10-28 18:05
comments: true
categories: [Ubuntu]
---

Here's how I backported the Ubuntu package for CMake 2.8.5 to natty, making it available 
in a [Launchpad PPA](https://ppa.launchpad.net/bcandrea/backports). Instead of using `apt-get source`,
`debuild` and `dput` manually I opted for the handy script `backportpackage`, which can be found in the
`ubuntu-dev-utils` package.

<!-- more -->

It's actually quite easy to backport an existing package using the standard developer tools -- once you find out 
exactly how to do it, that is. Here's my little HOWTO.

And beware: rerely things go well as in this case. Sometimes the package specification needs to be modified 
to make it work (for instance, packaging CMake 2.8.5 for lucid is not trivial at all, but 
[that's another story]({% post_url 2012-10-30-cmake-2-dot-8-5-in-lucid-lynx %})).

## Launchpad

[Launchpad](https://launchpad.net) (among other things) is a good platform for generating and publishing Ubuntu 
and Debian packages. It allows users to set up private package repositories (PPAs) and hosts a farm of build servers as well.

You first need to set up an account (for free) at [this URL](https://login.launchpad.net/+new_account), as described 
[here](https://help.launchpad.net/YourAccount/NewAccount).

In order to create your first PPA, you'll need to [import your PGP key to Launchpad](https://help.launchpad.net/YourAccount/ImportingYourPGPKey)
and [sign the Ubuntu Code of Conduct](https://help.ubuntu.com/community/SigningCodeofConduct). There is a 
[nice tutorial on WikiHow](http://www.wikihow.com/Sign-the-Ubuntu-Code-of-Conduct) if you like pictures.

So far so good. You can now create your first PPA by following the link on your home page, which is something like
`https://launchpad.net/~username`. I would suggest to pick a meaningful short name for the URL (I chose `backports` for
this one) instead of just `ppa`.

## Setting up your system

Your local Ubuntu box needs to be configured (only once) before trying to backport packages. First install some prerequisites:

    $ sudo apt-get install cowbuilder ubuntu-dev-utils

Then create the base folder which will be used to setup the chrooted environment in which the package will be built:

    $ mkdir -p ~/pbuilder/
    $ sudo cowbuilder --create --distribution natty --components "main restricted universe multiverse" --basepath=$HOME/pbuilder/natty-base.cow

If you have more than one PGP key in your local keyring (I do), you may also want to specify which one to use when signing 
packages. Just list your keys with `gpg --list-keys`, and copy the key id to `/etc/devscript.conf` under the debsign stanza:

    DEBSIGN_KEYID=FDCCCD6E

Last problem to avoid: the personal package builders cowbuilder and pbuilder must be allowed to preserve the environment
when called using `sudo`. Create the file `/etc/sudoers.d/pbuilders` using the command

    $ sudo visudo -f /etc/sudoers.d/pbuilders

and put the following two lines into it:

    Cmnd_Alias PBUILDERS = /usr/sbin/pbuilder, /usr/sbin/cowbuilder
    ALL ALL=(ALL) SETENV: PBUILDERS

All done. Let's backport the package (finally).

## Building and uploading the package 

We'll work in a temporary directory, so first create it.

    $ mkdir -p ~/backports/cmake/natty

Now cd into it:

    $ cd ~/backports/cmake/natty

You should test the package build before the submission to Launchpad; it can be done by running `backportpackage` with the `-b` option:

    $ BASEPATH=~/pbuilder/natty-base.cow DEBEMAIL=bcandrea@gmail.com DEBFULLNAME="Andrea Bernardo Ciddio" backportpackage -B cowbuilder -b -s oneiric -d natty -w . cmake

A brief explanation of the options:

* `BASEPATH=~/pbuilder/natty-base.cow`: this tells `cowbuilder` where to find the base image for the chroot
* `DEBEMAIL=bcandrea@gmail.com DEBFULLNAME="Andrea Bernardo Ciddio"`: those values will appear in the generated Debian package
* `-B cowbuilder`: select `cowbuilder` to create the package (`pbuilder` is the default)
* `-b`: build the package completely and generate .deb files
* `-s oneiric -d natty`: source and destination codenames, respectively
* `-w .`: use the current path as a local build directory

Now that everything seems to be working, the package can be uploaded to our freshly created PPA:

    $ DEBEMAIL=bcandrea@gmail.com DEBFULLNAME="Andrea Bernardo Ciddio" backportpackage -s oneiric -d natty -u ppa:bcandrea/backports cmake


## References

[How-To: PPA with CMake!](http://www.simonschneegans.de/?p=346) - A useful tutorial about PPAs (it's not about _packaging_ CMake itself, tough)

[Personal Package Archives](https://help.launchpad.net/Packaging/PPA/) - Everything you always wanted to know about PPAs, but were too afraid to ask

[Ubuntu packaging guide](http://developer.ubuntu.com/packaging/html/) - The ultimate source of information about Ubuntu packages