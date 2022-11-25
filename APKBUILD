# Maintainer: Natanael Copa <ncopa@alpinelinux.org>
pkgname=mesa
pkgver=9999
pkgrel=15
pkgdesc="Mesa DRI OpenGL library"
url="https://www.mesa3d.org"
arch="all"
license="MIT SGI-B-2.0 BSL-1.0"

subpackages="
	$pkgname-dri-gallium:_gallium
	$pkgname-glapi
 	$pkgname-gl
	$pkgname-egl
	$pkgname-gles
	$pkgname-gbm
	"
#	$pkgname-dbg
#	$pkgname-dev
#	$pkgname-xatracker
_llvmver=15
depends_dev="
	libdrm-dev
	libxext-dev
	libxdamage-dev
	libxcb-dev
	libxshmfence-dev
	"
makedepends="
	$depends_dev
	bison
	eudev-dev
	expat-dev
	findutils
	flex
	gettext
	elfutils-dev
	glslang-dev
	libtool
	libxfixes-dev
	libva-dev
	libvdpau-dev
	libx11-dev
	libxml2-dev
	libxrandr-dev
	libxxf86vm-dev
	llvm$_llvmver-dev
	meson
	py3-mako
	python3
	vulkan-loader-dev
	wayland-dev
	wayland-protocols
	xorgproto
	zlib-dev
	zstd-dev
	mold
	"
source="
	https://gitlab.freedesktop.org/mesa/mesa/-/archive/main/mesa-main.tar.gz
	"
replaces="mesa-dricore"
options="!check" # we skip tests intentionally
builddir="$srcdir/mesa-main"

build() {
	export CFLAGS="$CFLAGS -g0 -pipe -ffat-lto-objects -fPIC"
	export CXXFLAGS="$CXXFLAGS -g0 -pipe -ffat-lto-objects -fPIC"
	export CPPFLAGS="$CPPFLAGS -g0"
	export OPT="-mtune=native -mfpu=neon -fno-semantic-interposition"

	PATH="$PATH:/usr/lib/llvm$_llvmver/bin" \
	abuild-meson \
		-Ddri-drivers-path= \
		-Dgallium-drivers=panfrost \
		-Dvulkan-drivers= \
		-Dvulkan-layers= \
		-Dplatforms=x11,wayland \
		-Dgbm=enabled \
		-Dopengl=true \
		-Dosmesa=false \
		-Dgles1=disabled \
		-Dgles2=enabled \
		-Degl=enabled \
		-Dvideo-codecs= \
		-Dshader-cache=enabled \
		-Dc_args="$OPT" \
		-Dc_link_args="$OPT -fuse-ld=mold" \
		-Dcpp_args="$OPT" \
		-Dcpp_link_args="$OPT -fuse-ld=mold" \
		-Db_lto=true \
		-Db_lto_mode=default \
		-Db_ndebug=true \
		-Doptimization=2 \
		-Db_lto_threads=4 \
		--buildtype=release \
		. output

	# Print config
	meson configure output

	ninja -C output \
		src/compiler/nir/nir_intrinsics.h \
		src/util/format/u_format_pack.h \
		$build_first

	meson compile -C output
}

package() {
	DESTDIR="$pkgdir" meson install --no-rebuild -C output

	sed -i s/-devel//g "$pkgdir"/usr/lib/pkgconfig/*.pc
}

egl() {
	pkgdesc="Mesa libEGL runtime libraries"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libEGL.so* "$subpkgdir"/usr/lib/
}

gl() {
	pkgdesc="Mesa libGL runtime libraries"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libGL.so* "$subpkgdir"/usr/lib/
}

glapi() {
	pkgdesc="Mesa shared glapi"
	replaces="$pkgname-gles=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libglapi.so.* "$subpkgdir"/usr/lib/
}

gles() {
	pkgdesc="Mesa libGLESv2 runtime libraries"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libGLES*.so* "$subpkgdir"/usr/lib/
}

xatracker() {
	pkgdesc="Mesa XA state tracker for vmware"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libxatracker*.so.* "$subpkgdir"/usr/lib/
}

osmesa() {
	pkgdesc="Mesa offscreen rendering libraries"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libOSMesa.so.* "$subpkgdir"/usr/lib/
}

gbm() {
	pkgdesc="Mesa gbm library"
	depends="mesa=$pkgver-r$pkgrel"

	install -d "$subpkgdir"/usr/lib
	mv "$pkgdir"/usr/lib/libgbm.so.* "$subpkgdir"/usr/lib/
}

libd3dadapter9() {
	pkgdesc="Mesa directx9 adapter"
	depends="mesa=$pkgver-r$pkgrel"

	amove usr/lib/d3d/d3dadapter9.so*
}

# Move links referencing the same file to the subpackage.
# Usage: _mv_links <base directory> <example>
# where <example> is one of the libraries covered by the megadriver.
# The example is used to find other links that point to the same file.
_mv_links() {
	install -d "$subpkgdir"/$1
	find -L "$pkgdir"/$1 -samefile "$pkgdir"/$1/$2 -print0 \
		| xargs -0 -I{} mv {} "$subpkgdir"/$1/
}

_mv_vulkan() {
	local i
	install -d "$subpkgdir"/usr/lib
	install -d "$subpkgdir"/usr/share/vulkan/icd.d
	for i in "$@"; do
		mv "$pkgdir"/usr/lib/libvulkan_$i.so "$subpkgdir"/usr/lib/
		mv "$pkgdir"/usr/share/vulkan/icd.d/${i}* "$subpkgdir"/usr/share/vulkan/icd.d/
	done
}

# Mesa uses "megadrivers" where multiple drivers are linked into one shared
# library. This library is then hard-linked to separate files (one for each driver).
# Each subpackage contains one megadriver so that all the hard-links are preserved.

_gallium() {
	pkgdesc="Mesa gallium DRI drivers"
	depends="mesa=$pkgver-r$pkgrel"

	# libgallium_dri.so
	_mv_links $_dri_driverdir swrast_dri.so
}

_va() {
	local n=${subpkgname##*-va-}
	pkgdesc="Mesa $n VAAPI drivers"
	depends="mesa=$pkgver-r$pkgrel libva"

	case $n in
	gallium)
		# libgallium_drv_video.so
		_mv_links /usr/lib/dri radeonsi_drv_video.so ;;
	esac
}

_vdpau() {
	local n=${subpkgname##*-vdpau-}
	pkgdesc="Mesa $n VDPAU drivers"
	depends="mesa=$pkgver-r$pkgrel libvdpau"

	case $n in
	gallium)
		# libvdpau_gallium.so.1.0.0
		_mv_links /usr/lib/vdpau libvdpau_radeonsi.so.1.0.0 ;;
	esac
}

_vulkan() {
	local n=${subpkgname##*-vulkan-}
	pkgdesc="Mesa Vulkan API driver for $n"
	depends="mesa=$pkgver-r$pkgrel"

	case $n in
	ati)
		_mv_vulkan radeon ;;
	intel)
		_mv_vulkan intel ;;
	broadcom)
		_mv_vulkan broadcom ;;
	freedreno)
		_mv_vulkan freedreno ;;
	panfrost)
		_mv_vulkan panfrost ;;
	swrast)
		_mv_vulkan lvp ;;
	esac
}

_vulkan_layers() {
	pkgdesc="collection of vulkan layers from mesa"
	depends="python3"

	# Remove this after the release of the next stable (3.14)
	# it originally was claed layer as it only packaged the
	# overlay one but now it also packages device-select and
	# intel-nullhw (on x86*)
	provides="$pkgname-vulkan-layer=$pkgver-r$pkgrel"
	replaces="$pkgname-vulkan-layer=$pkgver-r$pkgrel"

	amove usr/share/vulkan/explicit_layer.d
	amove usr/share/vulkan/implicit_layer.d
	amove usr/lib/libVkLayer_*.so
	amove usr/bin/mesa-overlay-control.py
}

sha512sums="
efe42015f4f0f15ece6d9b67289fbcac46be10c532fddbbce439bb13f440fb7be2b831376dd404e1c100d0821fb796e1aeb6bf476814e8d33e071626b322c721  mesa-main.tar.gz
"
