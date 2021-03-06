# vgmstream build help

This document explains how to build each of vgmstream's components and libs.


## Compilation requirements

Because each module has different quirks one can't use a single tool for everything. You should be able to build most using a standard compiler (GCC/MSVC/Clang) in any typical OS (Windows/Linux/Mac) using common build helpers (scripts/CMake/autotools).

64-bit support may work but has been minimally tested, since main use of vgmstream is plugins for 32-bit players (should work but extra codec libs are included for 32-bit only ATM).


### GCC / Make
Common C compiler, most development is done with this.

On Windows you need one of these two somewhere in PATH:
- MinGW-w64 (32bit version): https://sourceforge.net/projects/mingw-w64/
  - Use this for easier standalone executables
  - Latest online installer with any config should work (for example: gcc-8.1.0, i686, win32, sjlj).
    https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe
  - Or simply download and unzip portable package:
    https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-win32/sjlj/x86_64-8.1.0-release-win32-sjlj-rt_v6-rev0.7z
- MSYS2 with the MinGW-w64_shell (32bit) package: https://msys2.github.io/

On Linux it should be included by default in the distribution, or can be easily installed using the distro's package manager (for example `sudo apt-get install gcc g++ make`).


### Microsoft's Visual C++ (MSVC) / Visual Studio / MSBuild
Alt C compiler, auto-generated builds for Windows use this.

Bundled in either:
- Visual Studio (2015/2017/latest): https://www.visualstudio.com/downloads/

Visual Studio Community should work (free), but must register after trial period. Even after trial you can still use MSBuild, command-line tool that actually does all the building. You can also find and install "Visual C++ Build Tools" without Visual Studio IDE (link tend to move around).

Older versions of MSVC (2010 and before) have limited C support, while reportedly beta/new versions aren't always very stable. Some very odd issues affecting only MSVC have been found and fixed before. Keep in mind if you run into problems.

### Clang
Reportedly works fine on Mac and may used as a replacement for GCC without issues.


### Git
Optional, used to generate version numbers:
- Git for Windows: https://git-scm.com/download/win


### CMake
Optional, can be used to compile vgmstream's modules instead of regular scripts. Needs v3.6 or later:
- https://cmake.org/download/

If you wish to use CMake, see [CMAKE.md](CMAKE.md). Some extra info is only mentioned in this doc though.


### autotools
Optional, used by some modules (mainly Audacious for Linux, and external libs).

For Windows you must include GCC, and Linux's sh tool in some form in PATH. The simplest would be installing MinGW-w64 for GCC.exe, and Git for sh.exe, and making PATH point their bin dir. 
- ex. `C:\i686-8.1.0-release-win32-sjlj-rt_v6-rev0\mingw32\bin` and `C:\Git\usr\bin`
- Both must be installed/copied in a dir without spaces (with spaces autoconf seemingly works but creates buggy files)
- If you don't have Git, try compiled GNU tools for Windows (http://gnuwin32.sourceforge.net/packages.html)

A useful trick for Windows is that you can alter PATH variable temporarly in `.bat` scripts (PATH is used to call programs in Windows without having to write full path to .exe)
```
set PATH=%PATH%;C:\i686-8.1.0-release-win32-sjlj-rt_v6-rev0\mingw32\bin
set PATH=%PATH%;C:\Git\usr\bin
gcc.exe (...)
```

For Linux, those may be included already, or install with a package manager (`sudo apt-get install autoconf automake libtool`).

Typical usage involves `./configure` (creates makefiles) + `Make` (compiles) + `Make install` (copies results), but varies slightly depending on module/lib (explained later).

External libs using autotools can be compiled on Windows too, try using `sh.exe ./configure`, `mingw32-make.exe`, `mingw32-make.exe install` instead. Also for older libs, call `sh.exe ./configure` with either  `--build=mingw32`, `--host=mingw32` or `--target-os=mingw32` (varies) for older configure. You may also need to use `mingw32-make.exe LDFLAGS="-no-undefined -static-libgcc" MAKE=mingw32-make.exe` so that `.dll` are correctly generated.


### Extra libs
Optional, add codec or extra functionalities. See *External libraries* for full info.

On Windows most libs are pre-compiled and included to simplify building (since they can be quite involved to compile).

On Linux you usually need dev packages of each (for example `libao-dev` for vgmstream123, `audacious-dev` for Audacious, and so on) and they should be picked by CMake/autotool scripts.



## Compiling modules

### CLI (test.exe/vgmstream-cli) / Winamp plugin (in_vgmstream) / XMPlay plugin (xmp-vgmstream)

**With GCC**: use the *./Makefile* in the root folder, see inside for options. For compilation flags check the *Makefile* in each folder. You may need to manually rebuild if you change a *.h* file (use *make clean*).

In Linux, Makefiles can be used to cross-compile with the MingW headers, but may not be updated to generate native code at the moment. It should be fixable with some effort. 

*Autotools* should build and install it as `vgmstream-cli`, this is explained in detail in the Audacious section. It also build with (some) extra codecs. Some Linux distributions like Arch Linux include pre-patched vgmstream with most libraries, you may want that instead:
https://aur.archlinux.org/packages/vgmstream-kode54-git/

You may try CMake instead as it may be simpler and handle libs better.


Windows CMD example:
```
prompt $P$G$_$S
set PATH=C:\Program Files (x86)\Git\usr\bin;%PATH%
set PATH=C:\Program Files (x86)\mingw-w64\i686-5.4.0-win32-sjlj-rt_v5-rev0\mingw32\bin;%PATH%

cd vgmstream

mingw32-make.exe vgmstream_cli -f Makefile ^
 VGM_ENABLE_FFMPEG=1 ^
 SHELL=sh.exe CC=gcc.exe AR=ar.exe STRIP=strip.exe DLLTOOL=dlltool.exe WINDRES=windres.exe
```

**With MSVC**: To build in Visual Studio, run *./init-build.bat*, open *./vgmstream_full.sln* and compile. To build from the command line, run *./build.bat*.

The build script will automatically handle obtaining dependencies and making the project changes listed in the foobar2000 section (you may need to get some PowerShell .NET packages).

You can also call MSBuild directly in the command line (see the foobar2000 section for dependencies and examples)

### foobar2000 plugin (foo\_input\_vgmstream)
Requires MSVC (foobar/SDK only links to MSVC C++ DLLs) and these dependencies:
- foobar2000 SDK (2018), in *(vgmstream)/dependencies/foobar/*: http://www.foobar2000.org/SDK
- (optional) FDK-AAC, in *(vgmstream)/dependencies/fdk-aac/*: https://github.com/kode54/fdk-aac
- (optional) QAAC, in *(vgmstream)/dependencies/qaac/*: https://github.com/kode54/qaac
- WTL (if needed), in *(vgmstream)/dependencies/WTL/*: http://wtl.sourceforge.net/
- may need to install ATL and MFC libraries (can be installed in Visual Studio Installer)

The following project modifications are required:
- For *foobar2000_ATL_helpers* add *../../../WTL/Include* to the compilers's *additional includes*

FDK-AAC/QAAC can be enabled adding *VGM_USE_MP4V2* and *VGM_USE_FDKAAC* in the compiler/linker options and the project dependencies, otherwise FFmpeg is used instead to support .mp4. Support is limited so FFmpeg is recommended.

You can also manually use the command line to compile with MSBuild, if you don't want to touch the .vcxproj files, register VS after trial, get PowerShell dependencies for the build script, or only have VC++/MSBuild tools.

Windows CMD example for foobar2000:
```
prompt $P$G$_$S
set PATH=C:\Program Files (x86)\Git\usr\bin;%PATH%
set PATH=C:\Program Files (x86)\MSBuild\14.0\Bin;%PATH%

cd vgmstream

set CL=/I"C:\projects\WTL\Include"
set LINK="C:\projects\foobar\foobar2000\shared\shared.lib"

msbuild fb2k/foo_input_vgmstream.vcxproj ^
 /t:Clean

msbuild fb2k/foo_input_vgmstream.vcxproj ^
 /t:Build ^
 /p:Platform=Win32 ^
 /p:PlatformToolset=v140 ^
 /p:Configuration=Release ^
 /p:DependenciesDir=../..
```

### Audacious plugin
Requires the dev version of Audacious (and dependencies), automake/autoconf or CMake, and gcc/make (C++11). It must be compiled and installed into Audacious, where it should appear in the plugin list as "vgmstream".

The plugin needs Audacious 3.5 or higher. New Audacious releases can break plugin compatibility so it may not work with the latest version unless adapted first.

libvorbis/libmpg123/libspeex will be used if found, while FFmpeg and other external libraries aren't enabled at the moment, thus some formats won't work (build scripts need to be fixed).

Windows builds aren't supported at the moment (should be possible but there are complex dependency chains).

If you get errors during the build phase we probably forgot some `#ifdef` needed for Audacious, notify and should be quickly fixed.

Take note of other plugins stealing extensions (see README). To change Audacious's default priority for vgmstream you can make with CFLAG `AUDACIOUS_VGMSTREAM_PRIORITY n` (where `N` is a number where 10=lowest)


Terminal example, assuming a Ubuntu-based Linux distribution:
```
# build setup

# default requirements
sudo apt-get update
sudo apt-get install gcc g++ make
sudo apt-get install autoconf automake libtool
sudo apt-get install git
# vgmstream dependencies
sudo apt-get install libmpg123-dev libvorbis-dev libspeex-dev
#sudo apt-get install libavformat-dev libavcodec-dev libavutil-dev libswresample-dev
# Audacious player and dependencies
sudo apt-get install audacious
sudo apt-get install audacious-dev libglib2.0-dev libgtk2.0-dev libpango1.0-dev
# vgmstream123 dependencies (optional)
sudo apt-get install libao-dev

# check Audacious version >= 3.5
pkg-config --modversion audacious
```
```
# base vgmstream build
git clone https://github.com/losnoco/vgmstream
cd vgmstream

# main vgmstream build (if you get errors here please report)
./bootstrap
./configure
make -f Makefile.autotools

# copy to audacious plugins (note that this will also install "libvgmstream",
# vgmstream-cli and vgmstream123, so they can be invoked from the terminal)
sudo make -f Makefile.autotools install

# update global libvgmstream.so.0 refs
sudo ldconfig

# start audacious in verbose mode to check if it was installed correctly
audacious -V

# if all goes well no "ERROR (something) referencing libvgmstream should show 
# in the terminal log, then go to menu services > plugins > input tab and check
# vgmstream is there (you can start audacious normally next time)
```
```
# uninstall if needed
sudo make -f Makefile.autotools uninstall

# optional post-cleanup
make -f Makefile.autotools clean
find . -name ".deps" -type d -exec rm -r "{}" \;
./unbootstrap
## WARNING, removes *all* untracked files not in .gitignore
git clean -fd
```
To update vgmstream it's probably easiest to remove the `vgmstream` folder and start again from *base vgmstream build* step, since updates often require a full rebuild anyway.

Instead of autotools you can try building with CMake. Some older distros may not work though (CMake version needs to recognize FILTER command). You may need to install resulting artifacts manually.
```
sudo apt-get update
sudo apt-get install -y libmpg123-dev libvorbis-dev libspeex-dev
sudo apt-get install -y libavformat-dev libavcodec-dev libavutil-dev libswresample-dev
sudo apt-get install -y libao-dev audacious-dev
sudo apt-get install -y cmake

cmake .
make
```

### vgmstream123 player
Should be buildable with Autotools by following the same steps as listen in the Audacious section (requires libao-dev).

Windows builds are possible with `libao.dll` and `libao` includes (found elsewhere) through the `Makefile`, but some features are disabled.

libao is licensed under the GPL v2 or later.


## External libraries
Support for some codecs is done with external libs, instead of copying their code in vgmstream. There are various reasons for this:
- each lib may have complex or conflicting ways to compile that aren't simple to duplicate
- their sources can be quite big and undesirable to include in full
- libs usually only compile with either GCC or MSVC, while vgmstream supports both compilers, so linking to the generated binary is much easier
- not all licenses used by libs may allow to copy their code
- simplifies maintenance and updating

They are compiled in their own sources, and the resulting binary is linked by vgmstream using a few of their symbols.

Currently only Windows builds can use external libraries, as vgmstream only includes generated 32-bit DLLs, but it should be fixable for others systems with some effort (libs are enabled on compile time). Ideally vgmstream could use libs compiled as static code (thus eliminating the need of DLLs), but involves a bunch of changes.

Below is a quick explanation of each library and how to compile binaries from them (for Windows, for Linux modules should include them automatically). Unless mentioned, their latest version should be ok to use, though included DLLs may be a bit older.

MSVC needs a .lib helper to link .dll files, but libs below usually only create .dll (and maybe .def). Instead, those .lib are automatically generated during build step in `ext_libs.vcxproj` from .dll+.def, using lib.exe tool.


### libvorbis
Adds support for Vorbis (inside Ogg and custom containers).
- Source: http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.6.zip
- DLL: `libvorbis.dll`
- licensed under the 3-clause BSD license.

Should be buildable with MSVC (in /win32 dir are .sln files) or autotools (use `autogen.sh`).


### mpg123
Adds support for MPEG (MP1/MP2/MP3).
- Source: https://sourceforge.net/projects/mpg123/files/mpg123/1.25.10/
- Builds: http://www.mpg123.de/download/win32/1.25.10/
- DLL: `libmpg123-0.dll`
- licensed under the LGPL v2.1

Must use autotools (sh configure, make, make install), though some scripts simplify the process: `makedll.sh`, `windows-builds.sh`.


### libg719_decode
Adds support for ITU-T G.719 (standardization of Polycom Siren 22).
- Source: https://github.com/kode54/libg719_decode
- DLL: `libg719_decode.dll`
- unknown license (possibly invalid and Polycom's)

Use MSVC (use `g719.sln`). It can be built with GCC too, but you'll need to manually create scripts/makefiles.


### FFmpeg
Adds support for multiple codecs: ATRAC3, ATRAC3plus, XMA1/2, WMA v1, WMA v2, WMAPro, AAC, Bink, AC3/SPDIF, Opus, Musepack, FLAC, etc (also Vorbis and MPEG for certain cases).
- Source: https://github.com/FFmpeg/FFmpeg/
- DLLs: `avcodec-vgmstream-58.dll`, `avformat-vgmstream-58.dll`, `avutil-vgmstream-56.dll`, `swresample-vgmstream-3.dll`
- primarily licensed under the LGPL v2.1 or later, with portions licensed under the GPL v2

vgmstream's FFmpeg builds remove many unnecessary parts of FFmpeg to trim down its gigantic size, and are also built with the "vgmstream-" preffix (to avoid clashing with other plugins). Current options can be seen in `ffmpeg_options.txt`.

For GCC simply use autotools (configure, make, make install), passing to `configure` the above options.

For MSCV it can be done through a helper: https://github.com/jb-alvarado/media-autobuild_suite

Both may need yasm somewhere in PATH to properly compile: https://yasm.tortall.net


### LibAtrac9
Adds support for ATRAC9.
- Source: https://github.com/Thealexbarney/LibAtrac9
- DLL: `libatrac9.dll`
- licensed under the MIT license

Use MSCV and `libatrac9.sln`, or GCC and the Makefile included.


### libcelt
Adds support for FSB CELT versions 0.6.1 and 0.11.0.
- Source (0.6.1): http://downloads.us.xiph.org/releases/celt/celt-0.6.1.tar.gz
- Source (0.11.0): http://downloads.xiph.org/releases/celt/celt-0.11.0.tar.gz
- DLL: `libcelt-0061.dll`, `libcelt-0110.dll`
- licensed under the MIT license

FSB uses two incompatible, older libcelt versions. Both libraries export the same symbols so normally can't coexist together. To get them working we need to make sure symbols are renamed first. This may be solved in various ways:
- using dynamic loading (LoadLibrary) but for portability it isn't an option
- It may be possible to link+rename using .def files
- Linux/Mingw's objcopy to (supposedly) rename DLL symbols
- Use GCC's preprocessor to rename functions on compile
- Rename functions in the source code directly.

To compile we'll use autotools with GCC preprocessor renaming:
- in the celt-0.6.1 dir:
  ```
  # creates Makefiles with Automake
  sh.exe ./configure --build=mingw32 --prefix=/c/celt-0.11.0/bin/  --exec-prefix=/c/celt-0.11.0/bin/

  # LDFLAGS are needed to create the .dll (Automake whinning)
  # CFLAGS rename a few CELT functions (we don't import the rest so they won't clash)
  mingw32-make.exe clean
  mingw32-make.exe LDFLAGS="-no-undefined" AM_CFLAGS="-Dcelt_decode=celt_0061_decode -Dcelt_decoder_create=celt_0061_decoder_create -Dcelt_decoder_destroy=celt_0061_decoder_destroy -Dcelt_mode_create=celt_0061_mode_create -Dcelt_mode_destroy=celt_0061_mode_destroy -Dcelt_mode_info=celt_0061_mode_info"
  ```
- in the celt-0.11.0 dir:
  ```
  # creates Makefiles with Automake
  sh.exe ./configure --build=mingw32 --prefix=/c/celt-0.11.0/bin/  --exec-prefix=/c/celt-0.11.0/bin/

  # LDFLAGS are needed to create the .dll (Automake whinning)
  # CFLAGS rename a few CELT functions (notice one is different vs 0.6.1), CUSTOM_MODES is also a must.
  mingw32-make.exe clean
  mingw32-make.exe LDFLAGS="-no-undefined" AM_CFLAGS="-DCUSTOM_MODES=1 -Dcelt_decode=celt_0110_decode -Dcelt_decoder_create_custom=celt_0110_decoder_create_custom -Dcelt_decoder_destroy=celt_0110_decoder_destroy -Dcelt_mode_create=celt_0110_mode_create -Dcelt_mode_destroy=celt_0110_mode_destroy -Dcelt_mode_info=celt_0110_mode_info"
  ```
- take the .dlls from ./bin/bin, and rename libcelt.dll to libcelt-0061.dll and libcelt-0110.dll respectively.
- you need to create a .def file for those DLL with the renamed simbol names above
- finally the includes. libcelt gives "celt.h" "celt_types.h" "celt_header.h", but since we renamed a few functions we have a simpler custom .h with minimal renamed symbols.


### libspeex
Adds support for Speex (inside custom containers).
- Source: http://downloads.us.xiph.org/releases/speex/speex-1.2.0.tar.gz
- DLL: `libspeex.dll`
- licensed under the Xiph.Org variant of the BSD license.
  https://www.xiph.org/licenses/bsd/speex/

Should be buildable with MSVC (in /win32 dir are .sln files, but not up to date and may need to convert .vcproj to vcxproj) or autotools (use `autogen.sh`, or script below).

You can also find a release on Github (https://github.com/xiph/speex/releases/tag/Speex-1.2.0). It has newer timestamps and some different helper files vs Xiph's release, but actual lib should be the same. Notably, Github's release *needs* `autogen.sh` that calls `autoreconf` to generate a base `configure` script, while Xiph's pre-includes `configure`. Since getting autoreconf working on Windows can be quite involved, Xiph's release is recommended.

Windows CMD example:
```
set PATH=%PATH%;C:\Git\usr\bin
set PATH=%PATH%;C:\i686-8.1.0-release-win32-sjlj-rt_v6-rev0\mingw32\bin

sh ./configure --host=mingw32 --prefix=/c/celt-0.11.0/bin/  --exec-prefix=/c/celt-0.11.0/bin/
mingw32-make.exe LDFLAGS="-no-undefined -static-libgcc" MAKE=mingw32-make.exe
mingw32-make.exe MAKE=mingw32-make.exe install
```
If all goes well, use generated .DLL in ./bin/bin (may need to rename to libspeex.dll) and ./win32/libspeex.def, and speex folder with .h in bin/include.


### maiatrac3plus
This lib was used as an alternate for ATRAC3PLUS decoding. Now this is handled by FFmpeg, though some code remains for now.

It was a straight-up decompilation from Sony's libs, without any clean-up or actual reverse engineering, thus legally and morally dubious.

It doesn't do encoder delay properly, but on the other hand decoding is 100% accurate unlike FFmpeg (probably inaudible though).

So, don't use it unless you have a very good reason.
