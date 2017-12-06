# OIS: Object oriented Input System

[![Software License](https://img.shields.io/badge/license-zlib%2Flibpng-green.svg)](LICENSE.md)
[![Build status](https://ci.appveyor.com/api/projects/status/grsgdeij6soi88bs?svg=true)](https://ci.appveyor.com/project/andrewfenn/ois)
[![Build Status](https://travis-ci.org/andrewfenn/OIS.svg?branch=master)](https://travis-ci.org/andrewfenn/OIS)

Cross Platform Object Oriented Input Lib System. Meant to be very robust and
compatiable with many systems and operating systems.

## Modifications
This library has been improved and modified by Bitgate, Inc. to fix bugs, improve performance or otherwise add new features to keep this library moving forward, as it is still the advised way to handle input by [Ogre3D](https://www.ogre3d.org/). We've also attempted to improve the usability of the library by tweaking the CMake files.

## Todo

- Merge pull requests/patches send in over time
- Cleanup and remove dead-code
- Start implementing support for new platforms and backends

### Linux

Dependencies: X11

To build and install:

    cmake -H. -B./build
    cd ./build
    make
    make install

### Windows

Dependencies: DirectX 9 (may downloaded from [Microsoft's official page](https://www.microsoft.com/en-us/download/details.aspx?id=6812))

For Visual Studio:

    cmake -H. -B./build
    msbuild "build\OISFFConsoleApp.sln"

### OSX

For XCode:

    cmake -H. -B./build
    cd ./build
    make

## License

The zlib/libpng License

Copyright © 2015 Andrew Fenn

Copyright © 2005-2010 Phillip Castaneda (pjcast -- www.wreckedgames.com)

This software is provided 'as-is', without any express or implied warranty. In no
event will the authors be held liable for any damages arising from the use of this
software.

Permission is granted to anyone to use this software for any purpose, including
commercial applications, and to alter it and redistribute it freely, subject to the
following restrictions:

    1. The origin of this software must not be misrepresented; you must not claim that
		you wrote the original software. If you use this software in a product,
		an acknowledgment in the product documentation would be appreciated
		but is not required.

    2. Altered source versions must be plainly marked as such, and must not be
		misrepresented as being the original software.

    3. This notice may not be removed or altered from any source distribution.
