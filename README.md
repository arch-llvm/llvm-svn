# llvm-svn

This is a fork of the already nice [llvm-svn](https://aur4.archlinux.org/packages/llvm-svn/) PKGBUILD from Arch Linux AUR. The fork makes the switch to cmake in lieu of autotools, and supports out-of-source builds.

___IMPORTANT:___ _This PKGBUILD uses a different version scheme, which tries to be as close to the output of `llvm-config --version` as possible. Instead of being simply the SVN revision number, it also adds the LLVM version number itself, e.g. `llvm-svn 3.8.0svn_r243840-1`._

## TODO

* Lots of things need refining. Some nasty bugs ~~may be~~ are still lurking around.

## BUGS

* libLLVM.so doesn't export all expected symbols, which leads to "undefined reference" build errors in e.g. Mesa. See issue [#2](https://github.com/kerberizer/llvm-svn/issues/2) for more information.
