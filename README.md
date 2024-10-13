# woa-aria2c-building

## 0. Get Cross-Compiling Toolchain && Set up Environment ##

### Environment ###

I chose WSL2 Debian 12 distro as the compiling host. If anyone succeeds to build with MSYS2 please PR, thx a lot.

For Debian/Ubuntu users, run ``` sudo apt update && sudo apt install binutils gcc wget curl tar unzip autoconf automake make -y ```

For other linux distros, install the variants of packages above.

### Acquire Compiling Toolchain ###

GCC currently has no official support for aarch64-w64-mingw32. Let's choose LLVM/Clang instead.

LLVM official release: https://github.com/mstorsjo/llvm-mingw

Until writing this tutor, 20241001 is the latest version of LLVM/Clang. I cannot guarantee newer/older version will work as expected.

* Download LLVM & Extract it to ``` /opt ```

```
$ wget https://github.com/mstorsjo/llvm-mingw/releases/download/20241001/llvm-mingw-20241001-ucrt-ubuntu-20.04-x86_64.tar.xz -O llvm-mingw-20241001-ucrt-ubuntu-20.04-x86_64.tar.xz
$ sudo tar xf llvm-mingw-20241001-ucrt-ubuntu-20.04-x86_64.tar.xz -C /opt 
```

* Then add it to your PATH variable.

```
$ export PATH=/opt/llvm-mingw-20241001-ucrt-ubuntu-20.04-x86_64/bin:$PATH
```

* Check if the compiler can be found.

```
$ aarch64-w64-mingw32-gcc --version
```

* Prepare an isolated folder for installing to avoid messing up the host

```
$ sudo mkdir /usr/local/aarch64-w64-mingw32
```

## 1. Compile needed external pkgs ##

We are building Windows-on-ARM aria2 so we don't need to compile LibGNUTls or other TLSlibs anymore :)

Packages below are what we need for the build:

* nettle
* libgmp
* libssh2-1
* c-ares
* expat (NOT libxml2 as I failed to link it)
* zlib
* libsqlite3
* pkg-config
  
### Nettle ###

* Download src from GNU

```
$ wget https://ftp.gnu.org/gnu/nettle/nettle-3.10.tar.gz -O nettle.tar.gz
$ tar zxvf nettle.tar.gz
$ cd nettle
```

* Configure

```
$ ./configure --host=aarch64-w64-mingw32 --disable-shared --enable-static --prefix=/usr/local/aarch64-w64-mingw32
```

* Compile

Notes: root may not find the toolchain, so when switching to root remember to set your path again.

```
$ make && sudo make install
```
