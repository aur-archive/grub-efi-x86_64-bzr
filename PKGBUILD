# Maintainer : Keshav Padram <(the.ridikulus.rat) (aatt) (gemmaeiil) (ddoott) (ccoomm)>

########################################################################
_bzrtrunk="bzr://bzr.savannah.gnu.org/grub/trunk/grub/"
# _bzrtrunk="http://bzr.savannah.gnu.org/r/grub/trunk/grub/"
# _bzrtrunk="lp:~the-ridikulus-rat/grub/trunk"
_bzrmod="grub"
########################################################################

########################################################################
_bzrtrunk_lua="bzr://bzr.savannah.gnu.org/grub-extras/lua/"
# _bzrtrunk_lua="http://bzr.savannah.gnu.org/r/grub-extras/lua/"
# _bzrtrunk_lua="lp:~the-ridikulus-rat/grub/grub-extras-lua"
_bzrmod_lua="grub-extras-lua"

_bzrtrunk_gpxe="bzr://bzr.savannah.gnu.org/grub-extras/gpxe/"
# _bzrtrunk_gpxe="http://bzr.savannah.gnu.org/r/grub-extras/gpxe/"
# _bzrtrunk_gpxe="lp:~the-ridikulus-rat/grub/grub-extras-gpxe"
_bzrmod_gpxe="grub-extras-gpxe"
########################################################################

__pkgname="grub-efi-bzr"

_UEFI_ARCH="x86_64"
_pkgname="grub-efi-${_UEFI_ARCH}-bzr"

pkgname="${_pkgname}"          ## Uncomment for grub BZR Main Branch
# pkgname="${_pkgname}-exp"    ## Uncomment for grub BZR Experimental Branch

pkgdesc="The GNU GRand Unified Bootloader (2) - ${_UEFI_ARCH} UEFI - bzr mainline branch with grub-extras"
pkgver=5011
pkgrel=1

url="https://www.gnu.org/software/grub/"
arch=('i686' 'x86_64')
license=('GPL3')

makedepends=('bzr' 'rsync' 'xz' 'bdf-unifont' 'ttf-dejavu' 'python' 'autogen' 'texinfo' 'help2man' 'gettext' 'device-mapper' 'fuse')
depends=('xz' 'freetype2' 'device-mapper' 'fuse' 'gettext' 'dosfstools' 'efibootmgr' 'sh')
optdepends=('libisoburn: provides xorriso for generating grub rescue iso using grub-mkrescue'
            'mtools: for manipulating FAT fs image files')

install="${__pkgname}.install"
backup=('boot/grub/grub.cfg' 'etc/default/grub' 'etc/grub.d/40_custom')
options=('strip' 'docs' 'zipman' 'emptydirs' '!makeflags')

conflicts=('grub-common' "grub-efi-${_UEFI_ARCH}")
provides=('grub-common' "grub-efi-${_UEFI_ARCH}")

if [[ "${_UEFI_ARCH}" == "x86_64" ]] && [[ "${CARCH}" != "x86_64" ]]; then
	echo "${pkgname} package can be built only in an x86_64 system. Exiting."
	exit 1
fi

if [[ "${pkgname}" == "${_pkgname}-exp" ]]; then
	pkgdesc="The GNU GRand Unified Bootloader (2) - ${_UEFI_ARCH} UEFI - bzr experimental branch with grub-extras"
	
	_bzrtrunk="bzr://bzr.savannah.gnu.org/grub/branches/experimental"
	# _bzrtrunk="http://bzr.savannah.gnu.org/r/grub/branches/experimental"
	# _bzrtrunk="lp:~the-ridikulus-rat/grub/experimental"
	_bzrmod="grub-exp"
fi

source=("${_bzrmod}::bzr+${_bzrtrunk}"
        "${_bzrmod_lua}::bzr+${_bzrtrunk_lua}"
        "${_bzrmod_gpxe}::bzr+${_bzrtrunk_gpxe}"
        'archlinux_grub_mkconfig_fixes.patch'
        'grub.default'
        'grub.cfg')

sha1sums=('SKIP'
          'SKIP'
          'SKIP'
          'fac9c3bbb24808f175def4d3726c2bd0503a880a'
          'dbf493dec4722feb11f0b5c71ad453a18daf0fc5'
          '76ae862a945a8848e6999adf8ad1847f0f7008b9')

pkgver() {
	cd "${srcdir}/${_bzrmod}/"
	bzr revno
}

build() {
	
	rm -rf "${srcdir}/${_bzrmod}_build" || true
	cp -r "${srcdir}/${_bzrmod}" "${srcdir}/${_bzrmod}_build"
	
	mkdir -p "${srcdir}/${_bzrmod}_build/grub-extras"
	cp -r "${srcdir}/grub-extras-lua" "${srcdir}/${_bzrmod}_build/grub-extras/lua"
	cp -r "${srcdir}/grub-extras-gpxe" "${srcdir}/${_bzrmod}_build/grub-extras/gpxe"
	
	cd "${srcdir}/${_bzrmod}_build/"
	rsync -Lrtvz translationproject.org::tp/latest/grub/ "${srcdir}/${_bzrmod}_build/po" || true
	(cd "${srcdir}/${_bzrmod}_build/po" && ls *.po | cut -d. -f1 | xargs) > "${srcdir}/${_bzrmod}_build/po/LINGUAS"
	
	cd "${srcdir}/${_bzrmod}_build/"
	
	## Apply Archlinux specific fixes to enable grub-mkconfig detect Arch kernels and initramfs
	patch -Np1 -i "${srcdir}/archlinux_grub_mkconfig_fixes.patch"
	echo
	
	## fix unifont.bdf location so that grub-mkfont can create *.pf2 files
	sed 's|/usr/share/fonts/unifont|/usr/share/fonts/unifont /usr/share/fonts/misc|g' -i "${srcdir}/${_bzrmod}_build/configure.ac"
	
	## fix DejaVuSans.ttf location so that grub-mkfont can create *.pf2 files for starfield theme
	sed 's|/usr/share/fonts/dejavu|/usr/share/fonts/dejavu /usr/share/fonts/TTF|g' -i "${srcdir}/${_bzrmod}_build/configure.ac"
	
	rm -f "${srcdir}/${_bzrmod}_build/po/POTFILES.in" || true
	rm -f "${srcdir}/${_bzrmod}_build/po/POTFILES-shell.in" || true
	
	find . -iname '*.[ch]' | sort > "${srcdir}/${_bzrmod}_build/po/POTFILES.in"
	find ./util -iname '*.in' | sort | sed '/bash-completion.d/d' > "${srcdir}/${_bzrmod}_build/po/POTFILES-shell.in"
	
	sed -i 's/grub-dev.texi//g' "${srcdir}/${_bzrmod}_build/docs/Makefile.am" || true
	
	export GRUB_CONTRIB="${srcdir}/${_bzrmod}_build/grub-extras/"
	
	## ! Requires python2
	# install -D -m0755 "${srcdir}/${_bzrmod}_build/autogen.sh" "${srcdir}/${_bzrmod}_build/autogen_unmodified.sh"
	# sed 's|python |python2 |g' -i "${srcdir}/${_bzrmod}_build/autogen.sh"
	echo
	
	"${srcdir}/${_bzrmod}_build/autogen.sh"
	echo
	
	mkdir -p "${srcdir}/${_bzrmod}_build/BUILD_UEFI_${_UEFI_ARCH}"
	cd "${srcdir}/${_bzrmod}_build/BUILD_UEFI_${_UEFI_ARCH}"
	
	unset CFLAGS
	unset CPPFLAGS
	unset CXXFLAGS
	unset LDFLAGS
	unset MAKEFLAGS
	
	../configure \
		--with-platform="efi" \
		--target="${_UEFI_ARCH}" \
		--host="${CARCH}-unknown-linux-gnu" \
		--disable-efiemu \
		--enable-mm-debug \
		--enable-nls \
		--enable-device-mapper \
		--enable-cache-stats \
		--enable-grub-mkfont \
		--enable-grub-mount \
		--prefix="/usr" \
		--bindir="/usr/bin" \
		--sbindir="/usr/bin" \
		--mandir="/usr/share/man" \
		--infodir="/usr/share/info" \
		--datarootdir="/usr/share" \
		--sysconfdir="/etc" \
		--program-prefix="" \
		--with-bootdir="/boot" \
		--with-grubdir="grub" \
		--disable-werror
	echo
	
	make
	echo
	
}


package() {
	
	cd "${srcdir}/${_bzrmod}_build/BUILD_UEFI_${_UEFI_ARCH}"
	make DESTDIR="${pkgdir}/" bashcompletiondir="/usr/share/bash-completion/completions/" install
	echo
	
	## remove gdb debugging related files
	rm -f "${pkgdir}/usr/lib/grub/${_UEFI_ARCH}-efi"/*.module || true
	rm -f "${pkgdir}/usr/lib/grub/${_UEFI_ARCH}-efi"/*.image || true
	rm -f "${pkgdir}/usr/lib/grub/${_UEFI_ARCH}-efi"/{kernel.exec,gdb_grub,gmodule.pl} || true
	
	## install /etc/default/grub
	install -D -m0644 "${srcdir}/grub.default" "${pkgdir}/etc/default/grub"
	
	## install example grub config file
	install -D -m0644 "${srcdir}/grub.cfg" "${pkgdir}/boot/grub/grub.cfg"
	
}
