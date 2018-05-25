---
layout: post
title: "Using Conan with Premake"
description: A sane way to manage C++ projects.
permalink: /:title
share: true
tags:
 - C++
 - Conan
 - Premake
---

# Why Premake?
Conan defaults to CMake, but CMake is bad, because:
- **Awful** domain specific scripting language that sends me on *long* quests for syntax examples, even for the most basic & trivial things, since the documentation is quite bad and ambiguous with its syntax templates, and doesnt provide code examples.
Since it's a DSL learning it is completely useless for anything but writing CMake scripts, so it's quite a waste of time.
In addition it's ugly and looks like a bunch of macro hacks (which it probably evolved from).
- It's slow and can take several seconds to generate a project, or even **minutes** in projects that heavily use CMake.

Premake doesn't have these downsides:
- Premake uses Lua, which is *simple* and *fast* general purpose scripting language, with proper documentation and rich ecosystem.
Maybe you [already know Lua](https://en.wikipedia.org/wiki/List_of_applications_using_Lua), so you don't need to spend time & effort learning a new language.
- Premake is fast, and generates a project in few milliseconds.
- Premake is also extensible, and has some [community modules](https://github.com/premake/premake-core/wiki/Modules).


# Adding Premake generator to Conan
Conan doesn't have built-in support for Premake, though it does provide an example Premake generator in the [custom generator tutorial](https://docs.conan.io/en/latest/howtos/custom_generators.html).
The example generator is lacking, so I added some missing feature in [my own fork](https://github.com/Enhex/conan-premake).

Once you have the Premake generator recipe you can use the `conan create` command to add it as a package, for example:
`conan create . enhex/stable`, then the package reference will be `premake_generator/master@enhex/stable`.


# Building generated projects
Premake's scope is limited to project generation, so it doesn't provide a way to build the projects it generates, and Conan needs a way to build generated projects for its packages.
CMake has the command `cmake --build`, but it depends on having a CMake cache so it can't be used independently of CMake.
I couldn't find a standalone tool that can build different project types, so I created one myself: [Build Tool Abstraction](https://github.com/Enhex/Build-Tool-Abstraction).


# Premake script setup
The Conan Premake generator will create a file called `conanpremake.lua` that defines variables for the list of include directories, libraries, and such.
All you need to do is include that file and plug in these variables where you want them.
In my generator fork I provide `conan_basic_setup()` function which you can call from your workspace/project scope and it will setup the variables for you.

Example script:
```lua
newoption {
	trigger     = "location",
	value       = "./",
	description = "Where to generate the project.",
}

if not _OPTIONS["location"] then
	_OPTIONS["location"] = "./"
end

-- `conan install` into the same directory premake will generate into
include(_OPTIONS["location"] .. "conanpremake.lua")

workspace("example")
	location(_OPTIONS["location"])
	conan_basic_setup()

	project("example")
		kind "StaticLib"
		language "C++"
		cppdialect "C++17"
		targetdir = "bin/%{cfg.buildcfg}"

		files {"src/**"}

		filter "configurations:Debug"
			defines { "DEBUG" }
			symbols "On"

		filter "configurations:Release"
			defines { "NDEBUG" }
			optimize "On"
```


# Conan recipe setup
Premake and Build Tool Abstraction executables need to be added to your path environment variable.
You need to require the Premake generator package in order to use Premake as your generator.
For example:

consumer's `conanfile.txt`:
```
[requires]
premake_generator/master@enhex/stable

[generators]
premake
```

package's `conanfile.py`:
```python
# ...
generators = "premake"
exports = "premake5.lua" # located in the project's root directory
requires = (
    "premake_generator/master@enhex/stable"
)

# ...

def build(self):
    from premake import run_premake # need to import here after the package was required
    run_premake(self) # automatically chooses premake generator
    self.run('build') # run Build Tool Abstraction, that will detect the build tool and use it
# ...
```


# Conclusion
Now you can use Premake to consume and create Conan packages!
Hopefully you'll have more pleasant time writing build scripts, and faster builds.

Please note that both my Premake generator fork and Build Tool Abstraction are very new projects and are not mature, and I developed them based on my needs.
Currently I've only added basic support for Visual Studio and Make, though adding support for other build systems should be trivial.