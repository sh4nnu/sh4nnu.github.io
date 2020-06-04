---
title: "Getting Started with Mozilla."
date: "2018-06-12"
coverImage: "m.png"
---

![mozilla](images/mozilla.png)

Hello readers, I would like to share my experience while contributing to the Mozilla the Open Source organization. Mozilla is very broad organization and developers in the organization are very helpful. I am inspired by the commitment of the developers I can surely say Mozilla is one of the best places where all the people around us help to develop our skills and help when we were struck anywhere.

This blog post lets you know how my journey with Mozilla is started and also how a complete beginner can start their contribution to this organization.

Getting started with Mozilla there are many areas where one can contribute to this organization. You can refer the link [Developer-guide](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide) any time for building, contributing, submitting patches etc., This blog post mainly focusses on **Firefox Desktop** for Linux (the Linux-distribution used here is _Ubuntu 16.04_).

## **Let's get started:**

- Make sure your system satisfies all the requirements before Building.
    - These Requirements for Unix/Linux.
        - 2G RAM with lots of available swap space. Additional RAM will significantly decrease build time.
        - For debug builds: at least 8 GB free disk space
        - For optimized builds: at least 1 GB free disk space (6 GB recommended)
    - The following procedure is for 64 bit Linux Operating System. To check which bit processor you use, use the command `uname -m` in the terminal which gives `x86_64` denoting 64-bit processor.
    - You will need python 2.6 or 2.7 installed, which you can check with `python --version.`
    - Finally, a reasonably fast internet connection and 30GB of free disk space.

For more detailed list visit [build-requirements](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Linux_Prerequisites).

With everything set up let's jump into building firefox.

Create a directory named "src" under your home directory:

```
mkdir src && cd src
```

Next download [the bootstrap.py script](https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py) and save it in the "**src/**" directory.

Now you can start the bootstrapper using python 2.7 or 2.6 in the terminal

```
python bootstrap.py
```

Running this command leads you to some prompts and then follow them for building in the way you prefer if you are new to Mozilla and not sure what to choose? go with the default.

## **Community Access:**

For contributing to any organization you need to get access to the developer community, IRC, mailing list, Bug-Tracker.

[Bugzilla](https://bugzilla.mozilla.org/) is the Mozilla's issue tracker, you will need an account in this tracker to log into and comment or submit a patch for a bug. You can use GitHub account also for making an account.

Most of the discussion in the community takes place in the IRC. You can ask for guidance and help during the build or fixing an error/bug in the IRC. I prefer the IRC client [IRCCLOUD](https://www.irccloud.com/) for chatting in the IRC. Follow this [documentation](https://wiki.mozilla.org/IRC) to know how to use IRC. Some of the important channel names which may help you in the beginning are

_`#introduction, #build, #coding, #newbies, #qa` _ these channels will help you by providing support and help.

## Getting the source code:

If you didn't allow the bootstrapper to download the Mozilla-unified you can clone it using Mercurial (a VCS which Mozilla community uses).

```
hg clone https://hg.mozilla.org/mozilla-central 
```

**\*\*NOTE\*\*:** If your network is not fast enough or having any interruptions in the connection cloning will give some issues and source code will not be downloaded properly. For this try

(If cloning is completed without any errors you can skip this step).

**Mercurial Bundle:**

The advantage of building the source code using bundles is unlike the `hg clone` you can pause and resume your download whenever there is an interruption to the connection. For this download the desired bundle of source code, you are interested in working and download it.

Up-to-date bundles of some repositories can be found in [https://hg.mozilla.org/](https://hg.mozilla.org/) which are available on a CDN (Content Delivery Network) at [https://hg.cdn.mozilla.net/](https://hg.cdn.mozilla.net/) you can download the Mozilla-central repository from here. Generally, **zstd bundles** are preferred they are smaller in size and faster to decompress but they are supported on the modern version of mercurial clients only. To avoid confusion rename the downloaded file as "**bundle.hg**".

Follow the below steps:

```html
mkdir mozilla-central
hg init mozilla-central
```

```html
cd mozilla-central
hg unbundle /...../bundle.hg
 hg config --edit or   EDITOR= hg config --edit 
```

```html
[paths]Let
default = https://hg.mozilla.org/mozilla-central/
```

```html
hg pull
```

```html
hg update
```

Here we initialized mercurial in the Mozilla-central directory. and configured .hgrc for default path, which will pull or push to the default link. Try to fix any errors if you encounter any in these steps. Once everything works properly, we successfully downloaded the source code for Firefox. The commands "**hg pull"** and "**hg update"** will help you to keep the source up-to-date.

## **Building the Source Code:**

Building a source code is similar to compiling your program before execution here it is more simple. But if you are building Mozilla-central for the first time it's better to grab a cup of coffee and a novel to read, because it takes a lot of time for building. You can anytime build your code using :

```
./mach build
```

If you encounter any errors related to LLVM/CLANG install the version asked for and again run "**./mach build"**.

## Let the  fun begin:

With everything set you may now run your code and visualize the project when you run the code, using `./mach run` which will launch the nightly version of Firefox. With this you are ready to hack into the firefox and explore, fix-issues, create new features ...etc., You can now start communicating to people via #introduction take up a "**good-first-bug"** and start contributing.

Have a happy contribution...

See you all later with another blog post.
