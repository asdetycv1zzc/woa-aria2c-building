# woa-aria2c-building

## 0. Get Cross-Compiling Toolchain && Set up Environment ##

### Environment ###

I chose WSL2 Debian 12 distro x86_64 as the compiling host. If anyone succeeds to build with MSYS2 please PR, thx a lot.

If you are under ARM64 environment like Surface or Huawei Matebook E GO, the LLVM/Clang should be ```arch64``` instead of ```x86_64```. However I won't compile ARIA2 on such device.

For Debian/Ubuntu users, run ``` sudo apt update && sudo apt install binutils gcc wget curl tar unzip autoconf automake make -y ```

For other linux distros, install the variants of packages above.

### Acquire Compiling Toolchain ###

GCC currently has no official support for aarch64-w64-mingw32. Let's choose LLVM/Clang instead.

LLVM official release: https://github.com/mstorsjo/llvm-mingw

Until writing this tutor, 20241001 is the latest version of LLVM/Clang. I cannot guarantee newer/older version will work as expected.

* Download LLVM/Clang & Extract it to ``` /opt ```

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

We are building Windows-on-ARM aria2 so we don't need to compile LibGNUTls anymore :)

Packages below are what we need for the build:

* libnettle (maybe we don't need it?)
* libgmp
* libssh2-1
* libc-ares
* libexpat (NOT libxml2 as I failed to link it)
* zlib
* libsqlite3
  
### Nettle ###

* Download src from GNU

```
$ wget https://ftp.gnu.org/gnu/nettle/nettle-3.10.tar.gz -O libnettle.tar.gz
$ tar zxvf libnettle.tar.gz
$ rm libnettle.tar.gz
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

###  LibGMP, LibSSH2-1, c-ares, LibExpat, LibSQLite3 ###

* Download src from official

```
$ wget https://gmplib.org/download/gmp/gmp-6.3.0.tar.xz -O gmp.tar.gz
$ wget https://libssh2.org/download/libssh2-1.11.0.tar.xz -O libssh2.tar.xz
$ wget https://github.com/c-ares/c-ares/releases/download/v1.34.1/c-ares-1.34.1.tar.gz -O c-ares.tar.gz
$ wget https://github.com/libexpat/libexpat/releases/download/R_2_6_3/expat-2.6.3.tar.xz -O expat.tar.xz
$ wget https://www.sqlite.org/2024/sqlite-autoconf-3460100.tar.gz -O sqlite.tar.gz
$ tar zxf gmp.tar.gz
$ tar xf libssh2.tar.xz
$ tar zxf c-ares.tar.gz
$ tar xf expat.tar.xz
$ tar zxf sqlite.tar.gz
$ rm gmp.tar.gz libssh2.tar.xz c-ares.tar.gz expat.tar.xz sqlite.tar.gz
```

* Configure & Compile

  The same as Nettle.

### ZLIB ###

* Download src from official

```
$ wget https://zlib.net/current/zlib.tar.gz -O zlib.tar.gz
$ tar zxf zlib.tar.gz
$ rm zlib.tar.gz
```

* Configure

```
$ cd zlib-*
$ ./configure --prefix=/usr/local/aarch64-w64-mingw32/ --static
```

  ZLib doesn't take cross-compiling into consideration. (though it runs everywhere without modification)

  Make some patches to its Makefile.

```
$ sed -i "s/CC=gcc/CC=aarch64-w64-mingw32-gcc/" Makefile
$ sed -i "s/CPP=/CPP=aarch64-w64-mingw32-g++/" Makefile
$ sed -i "s/AR=ar/AR=aarch64-w64-mingw32-ar/" Makefile
$ sed -i "s/LDSHARED=gcc/LDSHARED=aarch64-w64-mingw32-gcc/" Makefile
$ sed -i "s/RANLIB=ranlib/RANLIB=aarch64-w64-mingw32-ranlib/" Makefile
```

* Compile

```
$ make && sudo make install
```

## 2. Compile ARIA2 ##

* Download src from official

```
$ wget https://github.com/aria2/aria2/releases/download/release-1.37.0/aria2-1.37.0.tar.xz -O aria2.tar.xz
$ tar xf aria2.tar.xz
$ rm aria2.tar.xz
```

* Configure

```
$ cd aria2-*
$ ./configure --host=aarch64-w64-mingw32 --enable-shared --enable-static --without-appletls --without-libxml2 --prefix=/your/path/to/build ARIA2_STATIC=yes
```

* Compile

```
$ make -j8 && sudo make install
```

Now you should be able to see aria2c.exe in your build path. Enjoy.
