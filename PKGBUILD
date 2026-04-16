# Maintainer: next-epic-scrooge <next-epic-scrooge@duck.com>
#
# Based on the official Arch `gamescope` package PKGBUILD by:
#   Sefa Eyeoglu <contact@scrumplex.net>
#   Bouke Sybren Haarsma <boukehaarsma23 at gmail dot com>
#   Maxime Gauduin <alucryd@archlinux.org>
#   Giancarlo Razzolini <grazzolini@archlinux.org>
# Adjustments for this fork (EWA Lanczos / Hermite downscaling, bilateral
# denoise and hdeband post-passes, plus the wlroots-libinput patch shipped
# under patches/) by next-epic-scrooge.
#
# This fork is a drop-in replacement for `gamescope`. It provides and
# conflicts with the upstream package; pacman will refuse to install both
# at the same time.

_pkgname=gamescope-lanczos-downscaling
pkgname=${_pkgname}-git
pkgver=r2548.g87ba8e6
pkgrel=1
pkgdesc='Fork of gamescope adding EWA Lanczos/Hermite downscaling + bilateral/hdeband post-passes'
arch=(x86_64)
url=https://github.com/next-epic-scrooge/gamescope-lanczos-downscaling
# gamescope itself is BSD-2-Clause; the four shader files added by this
# fork are LGPL (see README.md -> "Licensing" for the breakdown).
license=(
    'BSD-2-Clause'
    'LGPL-2.1-or-later'
    'LGPL-3.0-or-later'
)
depends=(
    gcc-libs
    glibc
    glm
    hwdata
    lcms2
    libavif
    libcap.so
    libdecor
    libdrm
    libinput
    libpipewire-0.3.so
    libx11
    libxcb
    libxcomposite
    libxdamage
    libxext
    libxfixes
    libxkbcommon
    libxmu
    libxrender
    libxres
    libxtst
    libxxf86vm
    luajit
    seatd
    sdl2
    vulkan-icd-loader
    wayland
    xcb-util-wm
    xcb-util-errors
    xorg-server-xwayland
)
makedepends=(
    benchmark
    cmake
    git
    glslang
    meson
    ninja
    vulkan-headers
    wayland-protocols
)
provides=(gamescope "${_pkgname}")
conflicts=(gamescope "${_pkgname}")

# The main source is this fork. Every git submodule used by the fork is
# mirrored as a separate `git+` source so makepkg fetches them once into
# $srcdir; `prepare()` then points the in-tree submodule .git/config at
# those local clones and runs `git submodule update`. This matches the
# approach used by the upstream Arch `gamescope-git` PKGBUILD.
source=(
    "${_pkgname}::git+${url}.git"
    'git+https://github.com/Joshua-Ashton/wlroots.git'
    'git+https://gitlab.freedesktop.org/emersion/libliftoff.git'
    'git+https://github.com/Joshua-Ashton/vkroots.git'
    'git+https://gitlab.freedesktop.org/emersion/libdisplay-info.git'
    'git+https://github.com/ValveSoftware/openvr.git'
    'git+https://github.com/Joshua-Ashton/reshade.git'
    'git+https://github.com/KhronosGroup/SPIRV-Headers.git'
)

b2sums=(
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
)

pkgver() {
    cd "${_pkgname}"
    printf 'r%s.g%s' \
        "$(git rev-list --count HEAD)" \
        "$(git rev-parse --short=7 HEAD)"
}

prepare() {
    cd "${_pkgname}"

    # Wire every submodule to the sibling clone makepkg already fetched.
    local _sm
    for _sm in \
        subprojects/wlroots::wlroots \
        subprojects/libliftoff::libliftoff \
        subprojects/vkroots::vkroots \
        subprojects/libdisplay-info::libdisplay-info \
        subprojects/openvr::openvr \
        src/reshade::reshade \
        thirdparty/SPIRV-Headers::SPIRV-Headers
    do
        local _path="${_sm%%::*}"
        local _src="${_sm##*::}"
        git submodule init "${_path}"
        git config "submodule.${_path}.url" "${srcdir}/${_src}"
    done

    git -c protocol.file.allow=always submodule update --recursive

    # Apply the wlroots libinput-switch-default patch shipped with the repo
    # (same command the README documents).  The `apply --check` guard makes
    # this a no-op if an upstream wlroots bump already merged the fix.
    if git -C subprojects/wlroots apply --check \
            ../../patches/wlroots-libinput-switch-default.patch 2>/dev/null
    then
        git -C subprojects/wlroots apply \
            ../../patches/wlroots-libinput-switch-default.patch
    fi

    # Pull the meson-wrap-based subprojects (glm, stb).
    meson subprojects download
}

build() {
    # openvr's bundled CMake requires an ancient minimum; newer CMake
    # refuses by default.  Same workaround upstream Arch uses.
    export CMAKE_POLICY_VERSION_MINIMUM=3.5

    cd "${_pkgname}"
    arch-meson . build \
        -Dforce_fallback_for=stb,wlroots,vkroots,libliftoff,glm,libdisplay-info \
        -Dpipewire=enabled
    ninja -C build
}

package() {
    cd "${_pkgname}"
    meson install -C build --skip-subprojects --destdir="${pkgdir}"

    # Ship every licence file the repo carries so /usr/share/licenses/
    # reflects the mixed licence of the shader additions.
    install -Dm644 LICENSE            "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
    install -Dm644 COPYING.GPL.v3     "${pkgdir}/usr/share/licenses/${pkgname}/COPYING.GPL.v3"
    install -Dm644 COPYING.LESSER.v2.1 "${pkgdir}/usr/share/licenses/${pkgname}/COPYING.LESSER.v2.1"
    install -Dm644 COPYING.LESSER.v3  "${pkgdir}/usr/share/licenses/${pkgname}/COPYING.LESSER.v3"
}

# vim: ts=4 sw=4 et:
