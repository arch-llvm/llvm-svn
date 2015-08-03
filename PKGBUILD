# Maintainer: Armin K. <krejzi at email dot com>
# Contributor: Christian Babeux <christian.babeux@0x80.ca>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Anders Bergh <anders@archlinuxppc.org>
# Contributor: Tomas Wilhelmsson <tomas.wilhelmsson@gmail.com>
# Contributor: Luchesar V. ILIEV <luchesar%2eiliev%40gmail%2ecom>

pkgbase=llvm-svn
pkgname=('llvm-svn' 'llvm-libs-svn' 'llvm-ocaml-svn' 'clang-svn' 'clang-analyzer-svn' 'clang-tools-extra-svn')
_pkgname='llvm'
pkgver=3.8.0svn_r243863
pkgrel=1
arch=('i686' 'x86_64')
url="http://llvm.org"
license=('custom:University of Illinois')
makedepends=('cmake' 'subversion' 'libffi' 'python2' 'ocaml-ctypes' 'ocaml-findlib' 'python-sphinx')

# this is always the latest svn so debug info can be useful
options=('staticlibs' '!strip')

source=("${_pkgname}::svn+http://llvm.org/svn/llvm-project/llvm/trunk"
        "clang::svn+http://llvm.org/svn/llvm-project/cfe/trunk"
        "clang-tools-extra::svn+http://llvm.org/svn/llvm-project/clang-tools-extra/trunk"
        "compiler-rt::svn+http://llvm.org/svn/llvm-project/compiler-rt/trunk"
        llvm-Config-llvm-config.h)
sha256sums=('SKIP'
            'SKIP'    
            'SKIP'
            'SKIP'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')

_ocamlver()
{
    pacman -Q ocaml | awk '{print $2}' | cut -d - -f1 | cut -d . -f1,2,3
}

pkgver()
{
    cd "${srcdir}/${_pkgname}"

    # This will almost match the output of `llvm-config --version` when the
    # LLVM_APPEND_VC_REV cmake flag is turned on. The only difference is
    # dash being replaced with underscore because of Pacman requirements.
    echo $(sed -n '/^AC_INIT/s|^.*,\[\([[:digit:]\.]\+svn\)\],.*$|\1|p' \
        autoconf/configure.ac)_r$(svnversion | tr -d [A-z])
}

prepare() {
    cd "${srcdir}/${_pkgname}"

    svn export --force "${srcdir}/clang" tools/clang
    svn export --force "${srcdir}/clang-tools-extra" tools/clang/tools/extra
    svn export --force "${srcdir}/compiler-rt" projects/compiler-rt

    # Fix docs installation directory
    sed -e 's|^\([[:blank:]]*DESTINATION[[:blank:]]\+\)docs/html|\1share/doc|' \
        -e 's|^\([[:blank:]]*DESTINATION[[:blank:]]\+\)docs/ocaml/html|\1share/doc/ocaml|' \
        -i docs/CMakeLists.txt

    mkdir -p "${srcdir}/build"
}

build() {
    cd "${srcdir}/build"

    export PKG_CONFIG_PATH="/usr/lib/pkgconfig"
    _ffi_include_flags=$(pkg-config --cflags-only-I libffi)
    _ffi_libs_flags=$(pkg-config --libs-only-L libffi)

    # LLVM_BUILD_LLVM_DYLIB: Build the dynamic runtime libs (e.g. libLLVM.so)
    # LLVM_BINUTILS_INCDIR: Must be set for the LLVMgold plugin to be built.
    cmake -G "Unix Makefiles" \
        -DCMAKE_BUILD_TYPE:STRING=Release \
        -DCMAKE_INSTALL_PREFIX:PATH=/usr \
        -DLLVM_APPEND_VC_REV:BOOL=ON \
        -DLLVM_ENABLE_RTTI:BOOL=ON \
        -DLLVM_ENABLE_FFI:BOOL=ON \
        -DFFI_INCLUDE_DIR:PATH=${_ffi_include_flags#-I} \
        -DFFI_LIBRARY_DIR:PATH=${_ffi_libs_flags#-L} \
        -DLLVM_BUILD_DOCS:BOOL=ON \
        -DLLVM_ENABLE_SPHINX:BOOL=ON \
        -DSPHINX_OUTPUT_HTML:BOOL=ON \
        -DSPHINX_OUTPUT_MAN:BOOL=ON \
        -DSPHINX_WARNINGS_AS_ERRORS:BOOL=OFF \
        -DLLVM_BUILD_LLVM_DYLIB:BOOL=ON \
        -DLLVM_BINUTILS_INCDIR:PATH=/usr/include \
        ../${_pkgname}

    make ocaml_doc
    make
}

package_llvm-svn() {
    pkgdesc="Low Level Virtual Machine"
    depends=("llvm-libs-svn=$pkgver-$pkgrel" 'perl')
    provides=('llvm')
    replaces=('llvm')
    conflicts=('llvm')

    cd "${srcdir}/build"

    # Exclude the clang directory, since it'll be installed in a separate package
    sed -i \
        "s|^\([[:blank:]]*include(\"${srcdir}/build/tools/clang/cmake_install.cmake\")\)$|#\1|" \
        tools/cmake_install.cmake

    make DESTDIR="${pkgdir}" install

    # The runtime libraries get installed in llvm-libs-svn
    rm -f "${pkgdir}"/usr/lib/lib{LLVM,LTO}.so{,.*}
    mv -f "${pkgdir}"/usr/lib/{BugpointPasses,LLVMgold}.so "${srcdir}/"

    # Clang libraries and OCaml bindings go to separate packages
    rm -rf "${srcdir}"/{clang,ocaml.{doc,lib}}
    mv "${pkgdir}/usr/lib/clang" "${srcdir}/clang"
    mv "${pkgdir}/usr/lib/ocaml" "${srcdir}/ocaml.lib"
    mv "${pkgdir}/usr/share/doc/ocaml" "${srcdir}/ocaml.doc"

    # Get rid of example Hello transformation
    rm -f "${pkgdir}"/usr/lib/*LLVMHello.*

    if [[ $CARCH == x86_64 ]]; then
        # Needed for multilib (https://bugs.archlinux.org/task/29951)
        # Header stubs are taken from Fedora
        #for _header in config llvm-config; do
        mv "${pkgdir}/usr/include/llvm/Config/llvm-config"{,-64}.h
        cp "${srcdir}/llvm-Config-llvm-config.h" "${pkgdir}/usr/include/llvm/Config/llvm-config.h"
    fi

    # Install Python bindings
    install -m755 -d "${pkgdir}/usr/lib/python2.7/site-packages"
    cp -r "${srcdir}/llvm/bindings/python/llvm" \
        "${pkgdir}/usr/lib/python2.7/site-packages/"
    python2 -m compileall "${pkgdir}/usr/lib/python2.7/site-packages/llvm"
    python2 -O -m compileall "${pkgdir}/usr/lib/python2.7/site-packages/llvm"

    # Clean up documentation
    rm -rf "${pkgdir}/usr/share/doc/llvm/html/_sources"

    install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_llvm-libs-svn() {
    pkgdesc="Low Level Virtual Machine (runtime library)"
    depends=('gcc-libs' 'zlib' 'libffi' 'ncurses')
    provides=('llvm-libs')
    replaces=('llvm-libs')
    conflicts=('llvm-libs')

    cd "${srcdir}/build"

    make DESTDIR="${pkgdir}" install-{LLVM,LTO}

    # Moved from the llvm-svn package here
    mv "${srcdir}"/{BugpointPasses,LLVMgold}.so "${pkgdir}/usr/lib/"

    # Ref: http://llvm.org/docs/GoldPlugin.html
    install -m755 -d "${pkgdir}/usr/lib/bfd-plugins"
    ln -s {/usr/lib,"${pkgdir}/usr/lib/bfd-plugins"}/LLVMgold.so

    # Must have a symlink that corresponds to the output of `llvm-config --version`.
    # Without it, some builds, e.g. Mesa, might fail for "lack of shared libraries".
    for _shlib in lib{LLVM,LTO} ; do
        # libLLVM.so.3.8.0svn-r123456
        ln -s "${_shlib}.so.$(echo ${pkgver} | cut -d _ -f 1)" \
            "${pkgdir}/usr/lib/${_shlib}.so.$(echo ${pkgver} | tr _ -)"
        # libLLVM-3.8.0svn-r123456.so
        ln -s "${_shlib}.so.$(echo ${pkgver} | cut -d _ -f 1)" \
            "${pkgdir}/usr/lib/${_shlib}-$(echo ${pkgver} | tr _ -).so"
    done

    install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_llvm-ocaml-svn() {
    pkgdesc="OCaml bindings for LLVM"
    depends=("llvm-svn=$pkgver-$pkgrel" "ocaml=$(_ocamlver)" 'ocaml-ctypes')
    provides=('llvm-ocaml')
    replaces=('llvm-ocaml')
    conflicts=('llvm-ocaml')

    cd "${srcdir}/build"

    install -m755 -d "${pkgdir}/usr/lib"
    install -m755 -d "${pkgdir}/usr/share/doc"
    cp -a "${srcdir}/ocaml.lib" "${pkgdir}/usr/lib/ocaml"
    cp -a "${srcdir}/ocaml.doc" "${pkgdir}/usr/share/doc/ocaml"

   install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_clang-svn() {
    pkgdesc="C language family frontend for LLVM"
    url="http://clang.llvm.org/"
    depends=("llvm-svn=$pkgver-$pkgrel" 'gcc')
    provides=('clang')
    replaces=('clang')
    conflicts=('clang')

    cd "${srcdir}/build/tools/clang"

    # Exclude the extra directory, since it'll be installed in a separate package
    sed -i \
        "s|^\([[:blank:]]*include(\"${srcdir}/build/tools/clang/tools/extra/cmake_install.cmake\")\)$|#\1|" \
        tools/cmake_install.cmake

    make DESTDIR="${pkgdir}" install

    # Install clang-format editor integration files (FS#38485)
    # Destination paths are copied from clang-format/CMakeLists.txt
    install -m755 -d "$pkgdir/usr/share/clang"
    (
        cd "${srcdir}/llvm/tools/clang/tools/clang-format"
        cp clang-format-diff.py \
            clang-format-sublime.py \
            clang-format.el \
            clang-format.py \
            "${pkgdir}/usr/share/clang/"

        cp git-clang-format "${pkgdir}/usr/bin/"

        sed -e 's|/usr/bin/python$|&2|' \
            -i "${pkgdir}/usr/bin/git-clang-format" \
            -i "${pkgdir}/usr/share/clang/clang-format-diff.py"
    )

    # Install Python bindings
    install -m755 -d "${pkgdir}/usr/lib/python2.7/site-packages"
    cp -r "${srcdir}/llvm/tools/clang/bindings/python/clang" \
        "${pkgdir}/usr/lib/python2.7/site-packages/"
    python2 -m compileall "${pkgdir}/usr/lib/python2.7/site-packages/clang"
    python2 -O -m compileall "${pkgdir}/usr/lib/python2.7/site-packages/clang"

    # Install html docs
    cp -r docs/html/* "${pkgdir}/usr/share/doc/clang/html/"
    rm -r "${pkgdir}/usr/share/doc/clang/html/_sources"

    install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_clang-analyzer-svn() {
    pkgdesc="A source code analysis framework"
    url="http://clang-analyzer.llvm.org/"
    depends=("clang-svn=$pkgver-$pkgrel" 'python2')
    provides=('clang-analyzer')
    replaces=('clang-analyzer')
    conflicts=('clang-analyzer')

    cd "${srcdir}/llvm/tools/clang"

    install -m755 -d "${pkgdir}"/usr/{bin,lib/clang-analyzer}
    for _tool in scan-{build,view}; do
        cp -r tools/${_tool} "${pkgdir}/usr/lib/clang-analyzer/"
        ln -s /usr/lib/clang-analyzer/${_tool}/${_tool} "${pkgdir}/usr/bin/"
    done

    # scan-build looks for clang within the same directory
    ln -s /usr/bin/clang "${pkgdir}/usr/lib/clang-analyzer/scan-build/"

    # Relocate man page
    install -m755 -d "${pkgdir}/usr/share/man/man1"
    mv "${pkgdir}/usr/lib/clang-analyzer/scan-build/scan-build.1" \
       "${pkgdir}/usr/share/man/man1/"

    # Use Python 2
    sed -e 's|env python$|&2|' \
        -e 's|/usr/bin/python$|&2|' \
        -i "${pkgdir}/usr/lib/clang-analyzer/scan-view/scan-view" \
           "${pkgdir}/usr/lib/clang-analyzer/scan-build/set-xcode-analyzer"

    # Compile Python scripts
    python2 -m compileall "${pkgdir}/usr/lib/clang-analyzer"
    python2 -O -m compileall "${pkgdir}/usr/lib/clang-analyzer"

    install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

package_clang-tools-extra-svn() {
    pkgdesc="Extra tools built using Clang's tooling APIs"
    url="http://clang.llvm.org/"
    depends=("clang-svn=$pkgver-$pkgrel")
    provides=('clang-tools-extra')
    replaces=('clang-tools-extra')
    conflicts=('clang-tools-extra')

    cd "${srcdir}/build/tools/clang/tools/extra"

    make DESTDIR="${pkgdir}" install

    install -Dm644 "${srcdir}/${_pkgname}/LICENSE.TXT" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

# vim:set ts=4 sts=4 sw=4 et:
