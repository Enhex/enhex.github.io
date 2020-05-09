---
layout: post
title: "Premake IDEs"
description: A review of available IDEs for Premake.
share: true
tags:
 - C++
 - Premake
 - CLion
 - QtCreator
 - CodeLite
 - CodeBlocks
 - KDevelop
 - Eclipse
 - SlickEdit
---

Recently I've built a new PC and wanted to give Linux a try as the main OS.
One of the problems I had was to choose an IDE that can work with Premake, and I ended up trying all of them.

# Windows

Visual Studio is the best option.
<br/>

# Linux

On Linux the choice is less straight forward.
Here are some of the pros and cons of each IDE.
<br/>

## Codelite
pros:
- official Premake support.
- easy to build from source.

cons:
- find definition/decleration doesn't work.
- slow "do nothing" builds.
- hard to get working on Windows.
- "press any key to continue" makes it a bit tricky to copy from console because pressing ctrl+C closes the window.
- OS package isn't latest version, and no package for some distros.
- no "create definition".
<br/>

## CodeBlocks
pros:
- works as intended.
- Premake addon available.

cons:
- can't open more than one workspace at a time.
- defaults to xterm terminal which may not be installed, thus no console window will open.
- building from source for Arch linux based distros isn't straight forward because of a missing package:
http://wiki.codeblocks.org/index.php/Installing_Code::Blocks_from_source_on_Arch_Linux
- doesn't ship with dark theme.
- OS package isn't latest version.
- builds workspace's projects one by one instead of in parallel, slowing build times.
- need to manually find a process's PID to attach to it.
- freezes on some builds, requiring to kill the process and clean the project.
- attaching to process to debug a shared lib it uses doesn't seem to work.
<br/>

## CLion
pros:
- great IDE.

cons:
- only works with CMake.
CMake addon repo that's listed on Premake wiki was removed.
	- I went ahead and created a [CLion generator](https://github.com/Enhex/premake-clion).
<br/>

## KDevelop
pros:
- seems to be a very good IDE (I don't have enough experience with it to be sure).
- OS package is recent version.

cons:
- doesn't seem to have proper support out-of-source builds.
- no Premake addon available, possible to make one.
<br/>

## Eclipse
cons:
- no ctrl+arrows naviation.
- Premake addon is broken, fails to generate.
<br/>

## SlickEdit
cons:
- Premake addon is broken, fails to find libraries (missing libdirs).
	- might be possible to fix: https://community.slickedit.com/index.php?topic=1541.0
<br/>

## QtCreator
pros:
- Premake addon available.
- dark theme.

cons:
- precompiled headers don't work (need to patch the addon).
- when first opening the project Qt Creator defines default build directories that mess up paths.
- qmake got no ability to specify working directory different than DESTDIR.

To complete the generator it's possible to also generate the Qt Creator file that defines the default build dirs and working dirs.
Though it's a big file, and I'm not sure if things other than what we want to set dynamically change between 
projects or not.
<br/>

# Conclusions
For Linux the best paid option is CLion.

For a free option currently the best option that works out of the box is CodeBlocks, though it isn't that great.
Improving the Qt Creator generator will result a much better option.