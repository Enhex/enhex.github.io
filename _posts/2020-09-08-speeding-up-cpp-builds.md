---
layout: post
title: "Speeding up C++ builds"
description: Ways to make your C++ projects build faster.
share: true
tags:
 - C++
 - Build
 - Clang
 - GCC
---

# Tools

- prefer [ninja](https://ninja-build.org/) over make when possible.

- if using make, use the flags `-r` and ``` -j`nproc` ```.
	- `-r` greatly improves "do nothing" build speed.
	When I started using it "do nothing" went from `0.6` seconds to `0.02`, eliminating 
	an annoying lag when launching via IDE.
<br/>
- use LLVM's [LLD linker](https://lld.llvm.org/).
compiler flag: `-fuse-ld=lld`
	- if you're using precompiled headers it [only works with Clang's precompiled headers](https://bugs.llvm.org/show_bug.cgi?id=42446).
		- if GCC gives you better performance, you can use Clang for development only.
	- if you can't use LLD, alternatively you can use [GOLD linker](https://en.wikipedia.org/wiki/Gold_(linker)), which slower than LLD but faster than the default LD.
	compiler flag: `-fuse-ld=gold`
<br/>
- use Clang [ThinLTO](https://clang.llvm.org/docs/ThinLTO.html) with caching.
In my testing it's faster than not using LTO at all.
compiler flags: `-flto=thin -Wl,--thinlto-cache-dir=...`
	- in my project it resulted x2.3 speedup.
<br/>
- use `-pipe` compiler flag.
make sure you have enough RAM for big projects.


# Code

- use forward declarations to move `#include`s from headers to `.cpp` files.
This can help eliminate dependencies and reduce incremental build time.


# Hardware

- Fast storage can make compilation several times faster.
In my experience switching from HDD to SSD resulted ~x10 speedup.

- More cores for parallel builds.

- RAM consumption can scale up with parallel builds, especially when using `-pipe`.
I've reached ~30GB RAM usage while compiling a medium sized project, with 16c/32t CPU.
So at least 1GB per CPU thread should be a good heuristic.
