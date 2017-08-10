Lil' Debi
=========

This is an app to setup and manage a Debian install in parallel on an Android
device.  It can build a Debian install from scratch or use an existing image.
It manages the starting and stopping of the Debian install.

It uses cdebootstrap to build up the disk image as a chroot, and then provides
start and stop methods for handling mounting, fsck, starting/stopping sshd,
etc.

It is 100% free software. Ultimately, our aim is to have the whole
process for every bit of this app documented so that it can be freely
inspected, modified, ported, etc.  We want this app to build a trusted Debian
install on the phone, so free software is the only way to get there.  This is
currently functional alpha software, so do not rely on it to produce a trusted
Debian install.  Please do try it out, use it, and report criticisms, bugs,
improvements, etc.


Requirements
============
* Android 2.1 or higher.
* Busybox executable. Lil' Debi app no longer includes busybox executable but still depends on it.
You so need to install Busybox by your own for example with help of the installer
https://f-droid.org/packages/ru.meefik.busybox/


Installing Debian
=================

The process of installing Debian with Lil' Debi is self-explanatory, just run
the app and click the Install... button.  But it doesn't yet work on all
devices.  If the install process fails on your phone, you can still use Lil'
Debi by downloading a pre-built Debian image.  It should work with any
Debian/Ubuntu/Mint armel image file.  Here is a Debian image file that was
built by Lil' Debi:

https://github.com/guardianproject/lildebi/downloads

Download the file, uncompress it and rename it 'debian.img' and copy it to
your SD Card.  Launch Lil' Debi, and you should now see the button says "Start
Debian".  Click the button to start your new Debian install.


Build Setup
===========

Build setup on machine other than Debian-based is not officially supported.
Therefore following notes assume that your build machine is a Debian or Debian-derivatives (Ubuntu, Mint, ...).
Setup has actually been validated on Ubuntu 16.04.

Both the Android SDK and the Android NDK are required:
* SDK: https://developer.android.com/studio/index.html#linux-bundle
* NDK: https://developer.android.com/ndk/downloads/older_releases.html#ndk-11c-downloads

Download and install the Android SDK from link given just above.

Install dependencies for building of the external projects:
  ```
  sudo apt-get install autoconf automake libtool transfig wget patch \
       texinfo make openjdk-8-jdk faketime
```

Download latest NDK officially supporting Android 2.1 i.e. release 10e:
  ```
  wget https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
```

Unpack the Android NDK archive somewhere, e.g.:
  ```
  unzip android-ndk-r10e-linux-x86_64.zip -d /opt/Android/
```


Below are some tips you may want to consider to build on Mac OS X or Windows:

On Mac OS X you will need Fink, MacPorts, or Brew to install some of the build
dependencies.  For example, GNU tar is required, OS X's tar will not work.

On Windows, you will likely need to work with cygwin, mingw32 or "Ubuntu on Windows" bash.


Building
========

Building Lil' Debi is a multi-step process which consists in :
* cloning the sources,
* building the native utilities,
* and then finally building the Android app.

Here are all those steps in a form to run in the terminal:

```
  git clone https://github.com/herv/lildebi
  cd lildebi
  git submodule init
  git submodule update
  # make NDK_BASE=/path/to/your/android-ndk -C external assets
  # e.g.
  make NDK_BASE=/opt/Android/android-ndk-r10e -C external assets
  ./gradlew assembleDebug
```

When done, the debug apk of the app is ready to install, i.e.:

```
  adb install app/build/outputs/apk/app-debug.apk
```

Or simply the one-step command to assemble and install the apk
```
  ./gradlew installDebug
```


Deterministic Release (**release process is broken and requires some rework**)
---------------------

Having a deterministic, repeatable build process that produces the exact same
APK wherever it is run has a lot of benefits:

* makes it easy for anyone to verify that the official APKs are indeed
  generated only from the sources in git

* makes it possible for FDroid to distribute APKs with the upstream
  developer's signature instead of the FDroid's signature

To increase the likelyhood of producing a deterministic build of LilDebi, run
the java build with `faketime`.  The rest is already included in the
Makefiles.  This is also included in the ./make-release-build.sh
script. Running a program with `faketime` causes that program to recent a
fixed time based on the timestamp provided to `faketime`.  This ensures that
the timestamps in the files are always the same.

```
  faketime "`git log -n1 --format=format:%ai`" \
  ant clean debug
```

The actual process that is used for making the release builds is the included
`./make-release-build` script.  To reproduce the official releases, run this
script. But be aware, it will delete all changes in the git repo that it is
run in, so it is probably best to run it in a clean clone.  Then you can
compare your release build to the official release using the included
`./compare-to-official-release` script.  It requires a few utilities to work.
All of them are Debian/Ubuntu packages except for `apktool`.  Here's what to
install:

```
  apt-get install unzip meld bsdmainutils
```

Or on OSX with brew:

```
  brew install apktool unzip
```

If you want to reproduce a build and the cdebootstrap-static package is no
longer available, you can download it from snapshot.debian.org.  For example:

 * http://snapshot.debian.org/archive/debian/20141024T052403Z/pool/main/c/cdebootstrap/cdebootstrap_0.6.3_armel.deb

NDK build options
-----------------

The following options can be set from the make command line to tailor the NDK
build to your setup:

 * NDK_BASE             (/path/to/your/android-ndk)
 * NDK_PLATFORM_LEVEL   (default is 5)
 * NDK_ABI              (arm, mips, x86)
 * NDK_COMPILER_VERSION (4.4.3, 4.6, 4.7, clang3.1, clang3.2)
 * HOST                 (arm-linux-androideabi, mipsel-linux-android, x86)


Original Sources
================

cdebootstrap
-----------
http://packages.debian.org/unstable/cdebootstrap

cdebootstrap is downloaded directly from Debian, extracted, and then
tar'ed into the included tarball assets/cdebootstrap.tar. See
external/cdebootstrap/Makefile for details.

gpgv
----
git://git.gnupg.org/gnupg.git

Only `gpgv` is needed, so it is built from GnuPG v1.4.x.  It is built
statically to get around PIE vs non-PIE.  If an executable is built fully
statically, with no dynamic linking at all, then the same binary will work on
both PIE systems (android-21 and above), and systems where PIE does not work
(older than android-16).
