# llvm-svn

This is an [Arch](https://www.archlinux.org/) Linux [PKGBUILD](https://wiki.archlinux.org/index.php/Creating_packages) for the [LLVM](http://llvm.org/) compiler infrastructure, the [Clang](http://clang.llvm.org/) frontend, and the various tools associated with it. It's available in the [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository) as [llvm-svn](https://aur.archlinux.org/pkgbase/llvm-svn/).

Main development is in the master branch, while the AUR git repo is mirrored by the [aur](https://github.com/kerberizer/llvm-svn/tree/aur) branch.

## IMPORTANT INFORMATION

**PLEASE READ THIS ONE CAREFULLY**

This is a fairly complex package. The only recommended and supported method of building is in a clean chroot as [described on the Arch Wiki](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot). A crude example is also provided further below. The use of [AUR helpers](https://wiki.archlinux.org/index.php/AUR_helpers) (yaourt, pacaur, etc.) in particular is discouraged; it may or may not work for you.

Also, unlike the [official packages](https://www.archlinux.org/packages/?sort=&repo=Extra&q=llvm&maintainer=&flagged=), which provide the latest **stable** releases, this one builds the code straight from the SVN source repository, where development is constantly taking place. Thus, it brings all the latest bells and whistles, but also tends to bring and all the **latest bugs**. It is therefore strongly recommended to use this LLVM/Clang build only for testing. Use in production should only be reserved for cases where you do need a particular feature (or a fix for some bug), which are not yet available in the stable releases.

### On failing regression tests

Note that failing regression tests do not necessarily indicate a problem with the package. Such failures are fairly normal for an actively developed code (i.e. SVN trunk or Git master). If this happens, wait for some time before trying the build again: a few hours to a day or two at most should be enough. If you desperately need the package built right away, you may also comment out the `make check` and `make check-clang` lines or append `|| true` to them, but do this only if you really know what you're doing and why.

## Binary packages

Pre-built, binary packages are available from two unofficial repositories:

- _lordheavy_'s [mesa-git](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#mesa-git), which may be particularly useful for those who need LLVM solely as a Mesa dependency. Note that the packages are built against the [testing] repos. _lordheavy_ is an Arch Linux developer and trusted user (TU).

- _kerberizer_'s [llvm-svn](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#llvm-svn), which is automatically rebuilt every 6 hours from this PKGBUILD and the latest SVN code. The packages are built against the [core/extra] repos. _kerberizer_ is the current maintainer.

Both repos provide `x86_64` and `multilib` packages. _kerberizer_'s repo is also PGP signed.

### Signature "unknown trust" error

For PGP signed unofficial repositories to work correctly their signing key needs to be added to Pacman's keyring. The process is [described here](https://wiki.archlinux.org/index.php/Pacman/Package_signing#Adding_unofficial_keys). For the `llvm-svn` repo in particular, it boils down to:
1. Fetch the necessary key from a keyserver:
```
# pacman-key -r 0x76563F75679E4525
```
2. Verify the key fingerprint; it must be __exactly__ `D16C F22D 27D1 091A 841C 4BE9 7656 3F75 679E 4525`:
```
$ pacman-key -f 0x76563F75679E4525
```
3. If the fingerprint matches, sign the key locally:
```
# pacman-key --lsign-key 0x76563F75679E4525
```

## If using LLVM as Mesa dependency

You may find helpful the topic "[mesa-git - latest videodrivers & issues](https://bbs.archlinux.org/viewtopic.php?id=212819)" on the Arch Linux forums.

## Building in a clean chroot example

If you need a more detailed and specific example on how to build this package in a clean chroot, a crude excerpt from the build script of the _kerberizer_'s binary repo is presented here. You can also check the [full script](https://github.com/kerberizer/pkg-rebuild-llvm).

It is meant to allow building `lib32-llvm-svn` too, hence why `gcc-multilib` is used. The code takes advantage of multiple cores when building and compressing; the example here is tailored to an 8-core/threads system. The user's ccache cache is utilised as well, so frequent rebuilds can be much faster. If you don't sign your packages, omit the lines mentioning `PACKAGER` and `GPGKEY`, otherwise they need to be set correctly. The chroot (`${x86_64_chroot}`) is best set up in `/tmp`, but this requires a lot of RAM (most likely at least 32 GB, since `/tmp` is by default half the size of the physical RAM detected); second best solution is on an SSD. The latter goes for `~/.ccache` as well. Note that the latest versions of systemd mount `/tmp` with the nosuid flag. You need to turn this flag off before building on `/tmp`, or else the build will fail.

```shell
cd /path/to/where/llvm-svn/is/cloned

x86_64_chroot="/chroot/x86_64"

sudo mkdir -p "${x86_64_chroot}/root"

sudo /usr/bin/mkarchroot \
    -C /usr/share/devtools/pacman-multilib.conf \
    -M /usr/share/devtools/makepkg-x86_64.conf \
    -c /var/cache/pacman/pkg \
    "${x86_64_chroot}/root" \
    base-devel ccache

sudo /usr/bin/arch-nspawn "${x86_64_chroot}/root" /bin/bash -c "yes | pacman -Sy gcc-multilib"

sudo /usr/bin/arch-nspawn "${x86_64_chroot}/root" /bin/bash -c \
    "echo -e \"CCACHE_DIR='/.ccache'\nXZ_DEFAULTS='--threads=8'\" >>/etc/environment ; \
     sed \
        -e 's/^#MAKEFLAGS=.*$/MAKEFLAGS=\"-j9\"/' \
        -e '/^BUILDENV=/s/\!ccache/ccache/' \
        -e 's/^#PACKAGER=.*$/PACKAGER=\"Some One <someone@somewhere.com>\"/' \
        -e 's/^#GPGKEY=.*$/GPGKEY=\"0x0000000000000000\"/' \
        -i /etc/makepkg.conf"

sudo /usr/bin/makechrootpkg -c -d ~/.ccache:/.ccache -r "${x86_64_chroot}"
```

It's advisable to always start this from scratch, i.e. don't reuse the old chroot, but create it anew for each build (it uses the local pacman cache, so doesn't waste bandwidth, and if located in /tmp or on an SSD, is pretty fast).

## Bugs

* When an older or generally different version of llvm-ocaml{,-svn} is installed on the build system, the build would likely fail with _inconsistent assumptions over interface_ errors. The PKGBUILD will detect such situation and spit out an appropriate suggestion: namely, to either uninstall any currently installed llvm-ocaml* package before building, or, __preferably__, to build in a clean chroot, as [described](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot) on the Arch Linux wiki.
