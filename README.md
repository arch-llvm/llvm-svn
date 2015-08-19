# llvm-svn

This is an [Arch](https://www.archlinux.org/) Linux [PKGBUILD](https://wiki.archlinux.org/index.php/Makepkg) for the [LLVM](http://llvm.org/) compiler infrastructure, the [Clang](http://clang.llvm.org/) frontend, and the various tools associated with it. It's available in the [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository) as [llvm-svn](https://aur.archlinux.org/pkgbase/llvm-svn/).

Main development is in the master branch, while the AUR git repo is mirrored by the [aur](https://github.com/kerberizer/llvm-svn/tree/aur) branch.

## BUGS

* libLLVM.so might not be exporting all expected symbols, which, in turn, may lead to "undefined reference" build errors. See issue [#2](https://github.com/kerberizer/llvm-svn/issues/2) for more information.
