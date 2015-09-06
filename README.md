# llvm-svn

This is an [Arch](https://www.archlinux.org/) Linux [PKGBUILD](https://wiki.archlinux.org/index.php/Makepkg) for the [LLVM](http://llvm.org/) compiler infrastructure, the [Clang](http://clang.llvm.org/) frontend, and the various tools associated with it. It's available in the [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository) as [llvm-svn](https://aur.archlinux.org/pkgbase/llvm-svn/). Unlike the [official package](https://www.archlinux.org/packages/?sort=&repo=Extra&q=llvm-libs&maintainer=&flagged=), which provides the latest stable release, this one builds the code straight from the SVN source repository, where development is constantly taking place. Thus, it brings all the latest bells and whistles, but also tends to bring and all the latest bugs. __You've been warned.__

Main development is in the master branch, while the AUR git repo is mirrored by the [aur](https://github.com/kerberizer/llvm-svn/tree/aur) branch.

## BUGS

* libLLVM.so might not be exporting all expected symbols, which, in turn, may lead to "undefined reference" build errors. See issue [#2](https://github.com/kerberizer/llvm-svn/issues/2) for more information.

* When an older or generally different version of llvm-ocaml{,-svn} is installed on the build system, the build would likely fail with _inconsistent assumptions over interface_ errors. The PKGBUILD will detect such situation and spit out an appropriate suggestion: namely, to either uninstall any currently installed llvm-ocaml* package before building, or, __preferably__, to build in a clean chroot, as [described](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot) on the Arch Linux wiki.

* [LLDB](http://lldb.llvm.org/) is not being built. The  [enh/lldb-svn](https://github.com/kerberizer/llvm-svn/tree/lldb-svn) branch does include it, but it is being held back for the time being, because of a conflicting package on AUR, [lldb-svn](https://aur.archlinux.org/packages/lldb-svn/).
