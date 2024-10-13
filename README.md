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

Notes: root may not find the toolchain, so when switching to root remember to set your path again.

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
* libexpat (NOT libxml2 as I failed to link it)
* zlib
* libsqlite3
* pkg-config
  
### Nettle ###

* Download src from GNU

```
$ wget https://ftp.gnu.org/gnu/nettle/nettle-3.10.tar.gz -O nettle.tar.gz
$ tar zxvf nettle.tar.gz
$ rm nettle.tar.gz
```

* Configure

```
$ cd nettle-*
$ ./configure --host=aarch64-w64-mingw32 --disable-shared --enable-static --prefix=/usr/local/aarch64-w64-mingw32
```

* Compile

```
$ make && sudo make install
```

###  LibGMP, LibSSH2-1, c-ares, LibExpat ###

* Download src from official

```
$ wget https://gmplib.org/download/gmp/gmp-6.3.0.tar.xz -O gmp.tar.gz
$ wget https://libssh2.org/download/libssh2-1.11.0.tar.xz -O libssh2-1.tar.xz
$ wget https://github.com/c-ares/c-ares/releases/download/v1.34.1/c-ares-1.34.1.tar.gz -O c-ares.tar.gz
$ wget https://github.com/libexpat/libexpat/releases/download/R_2_6_3/expat-2.6.3.tar.xz -O expat.tar.xz
$ tar zxf gmp.tar.gz
$ tar xf libssh2-1.tar.xz
$ tar zxf c-ares.tar.gz
$ tar xf expat.tar.xz
$ rm gmp.tar.gz libssh2-1.tar.xz c-ares.tar.gz expat.tar.xz
```

### ZLIB ###

* Download src from official

```
$ wget https://zlib.net/current/zlib.tar.gz -O zlib.tar.gz
$ tar zxf zlib.tar.gz
$ rm zlib.tar.gz
```

* Configure

ZLib doesn't take cross-compiling into consideration. (though it runs everywhere without modification)

Make some patches to it.

```
$ cd zlib-*
$ ./configure --prefix=/usr/local/aarch64-w64-mingw32/ --static
$ sed -i "s/CC=gcc/CC=aarch64-w64-mingw32-gcc/" Makefile
$ sed -i "s/CPP=/CPP=aarch64-w64-mingw32-g++/" Makefile
$ sed -i "s/AR=ar/AR=aarch64-w64-mingw32-ar/" Makefile
$ sed -i "s/LDSHARED=gcc/LDSHARED=aarch64-w64-mingw32-gcc/" Makefile
$ sed -i "s/RANLIB=ranlib/RANLIB=aarch64-w64-mingw32-ranlib/" Makefile
```

* Build

```
$ make && sudo make install
```
