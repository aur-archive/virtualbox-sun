# $Id:$
# Contributor: Balwinder S "bsd" Dheeman (bdheeman AT gmail.com)
# Credits: thotypous <matiasΘarchlinux-br·org>
# Maintainer: Balwinder S "bsd" Dheeman (bdheeman AT gmail.com)

pkgname=virtualbox-sun
_pkgver=4.1.14
_pkgbld=77440
pkgrel=1
pkgver=4.1.14.77440
pkgdesc="A general-purpose full virtualizer for x86 hardware"
url=http://www.virtualbox.org/
license=('GPL2')
arch=('i686' 'x86_64')
depends=('gcc' 'make' 'kernel26-headers' 'libxi' 'libxcursor' 'libxrandr' 'libxmu')
makedepends=('python2' 'qt' 'sdl')
optdepends=('alsa-lib: for ALSA support'
    'dkms: for building and loading vboxhost modules'
    'libgl: for shared OpenGL support'
    'libxt: for shared clipboard support'
    'mesa: for OpenGL support'
    'pulseaudio: for PulseAudio support'
    'python2: for VirtualBox SDK'
    'qt: for Oracle VirtualBox QT4 GUI on X-Window System'
    'sdl: for Oracle VBoxSDL and VirtualBox GUI on console'
    'vfuse: for mounting VBox (VDI/VMDK/VHD) disk images')

provides=("virtualbox=${_pkgver}")
conflicts=('virtualbox_bin' 'virtualbox-ose' 'virtualbox-modules')
backup=('etc/conf.d/vboxdrv')
install=${pkgname%-*}.install

md5sums=('4b408d6c7ced82edf3c1284b494a47bd'
         '87c2c827f9a0432039500d37f6d0477e'
         'd6b5f065118f145095a013da0e987c17'
         '28334ec9f6da56d1094e94f7796ff31a')

case "$CARCH" in
    i686|i[3-5]86) _arch='x86';;
    x86_64|amd64) _arch='amd64'; md5sums[0]='97d0dc8d74bfdc710a3cd8f4e5a994dd';;
    # The following should not happen; provided you're using 'makepkg' ;)
    *) error "Unknown or invalid CARCH=$CARCH"; exit 1
esac

source=(http://download.virtualbox.org/virtualbox/${_pkgver}/VirtualBox-${_pkgver}-${_pkgbld}-Linux_${_arch}.run
	'etc_conf.d_vboxdrv'
	'etc_rc.d_vboxdrv'
	"${pkgname%-*}.desktop")

build() {
    # Unpack the binary package
    msg2 "Unpacking the Oracle binaries..."
    echo yes | sh "VirtualBox-${_pkgver}-${_pkgbld}-Linux_${_arch}.run" --target "$srcdir" \
        --nox11 --noexec &>/dev/null || return 1
    mkdir -p "$pkgdir/opt/VirtualBox"
    cd "$pkgdir/opt/VirtualBox"
    tar xf "$srcdir/VirtualBox.tar.bz2" || return 1

    msg2 "Fixing permissions..."
    # Mark set-user-ID-on-execution, if release is marked as a hardened build
    if egrep '^HARDENED="1"' "$srcdir/install.sh" &> /dev/null; then
        chmod 4511 "VirtualBox" "VBoxSDL" "VBoxHeadless" "VBoxNetDHCP"
        for _n in "VBoxVMM.so" "VBoxREM.so" "VBoxRT.so" "VBoxDDU.so" "VBoxXPCOM.so"; do
            ln -sf "/opt/VirtualBox/${_n}" "components"
        done
        chmod go-w .
    fi
    # VBoxNetAdpCtl needs to be suid root, even if this is not a hardened build
    chmod 4511 "VBoxNetAdpCtl"

    msg2 "Making it nice..."
    # Replace VirtualBox built-in Qt by system Qt libraries
    # Fixes problems with some KDE themes reported by users
    ln -fs "/usr/lib/libQtCore.so.4" "libQtCoreVBox.so.4"
    ln -fs "/usr/lib/libQtGui.so.4" "libQtGuiVBox.so.4"
    ln -fs "/usr/lib/libQtNetwork.so.4" "libQtNetworkVBox.so.4"
    ln -fs "/usr/lib/libQtOpenGL.so.4" "libQtOpenGLVBox.so.4"

    # Patch "vboxshell.py" to use Python 2.x instead of Python 3
    sed -i 's#/usr/bin/python#\02#' "$pkgdir/opt/VirtualBox/vboxshell.py"

    # Install the SDK
    cd "$pkgdir/opt/VirtualBox/sdk/installer"
    VBOX_INSTALL_PATH="/opt/VirtualBox" python2 vboxapisetup.py install --root "${pkgdir}" || return 1
    rm -Rf build
    cd "$pkgdir/opt/VirtualBox"

    # Symlink the launchers
    mkdir -p "$pkgdir/usr/bin"
    for _app in "VirtualBox" "VBoxHeadless" "VBoxManage" "VBoxSDL" "VBoxSVC" \
                "VBoxTunctl" "VBoxNetAdpCtl" "rdesktop-vrdp"; do
        ln -s "/opt/VirtualBox/${_app}" "$pkgdir/usr/bin/${_app}"
    done

    # Symlink the VBoxGuestAdditions.iso into /usr/share/virtualbox, to make it
    # a Debian/Ubuntu and, or Fedora compatible installation :)
    mkdir -p "$pkgdir/usr/share/virtualbox/"
    ln -s "/opt/VirtualBox/additions/VBoxGuestAdditions.iso" \
	"$pkgdir/usr/share/virtualbox/VBoxGuestAdditions.iso"

    # Install rc scripts and defaults
    install -Dm644 "$srcdir/etc_conf.d_vboxdrv" "$pkgdir/etc/conf.d/vboxdrv"
    echo "MODULES=($(cd $pkgdir/opt/VirtualBox/src/vboxhost && echo vbox*))" \
	>> "$pkgdir/etc/conf.d/vboxdrv"
    echo "VBOX_VERSION=\"${_pkgver}\"" >> "$pkgdir/etc/conf.d/vboxdrv"
    install -Dm755 "$srcdir/vboxballoonctrl-service.sh" "$pkgdir/etc/rc.d/vboxballoonctrl-service"
    install -Dm755 "$srcdir/etc_rc.d_vboxdrv" "$pkgdir/etc/rc.d/vboxdrv"
    install -Dm755 "$srcdir/vboxweb-service.sh" "$pkgdir/etc/rc.d/vboxweb-service"

    # Install desktop and desktop and icon appropriately
    install -m644 "$srcdir/virtualbox.desktop" "$pkgdir/opt/VirtualBox/"
    mkdir -p "$pkgdir/usr/share/applications"
    ln -s "/opt/VirtualBox/virtualbox.desktop" "$pkgdir/usr/share/applications/VirtualBox.desktop"
    mkdir -p "$pkgdir/usr/share/pixmaps"
    ln -s "/opt/VirtualBox/VBox.png" "$pkgdir/usr/share/pixmaps/VBox.png"

    # Replace some init scripts by simplified stuff
    sed -i -e 's,/etc/vbox/vbox.cfg,/etc/vbox.cfg,g' "$pkgdir/opt/VirtualBox/VBox.sh" $pkgdir/etc/rc.d/vbox*-service
    sed -i -e 's,sudo /etc/init.d/vboxdrv restart,sudo /etc/rc.d/vboxdrv restart,g' "$pkgdir/opt/VirtualBox/VBox.sh"
    sed -i -e 's,sudo /etc/init.d/vboxdrv setup,sudo /etc/rc.d/vboxdrv setup,g' "$pkgdir/opt/VirtualBox/VBox.sh"
    sed -i -e "s,/etc/init.d/vboxdrv setup' as root\x00,/etc/rc.d/vboxdrv start' as root\x00\x00\x00,g" "$pkgdir/opt/VirtualBox/VBoxVMM.so"
    sed -i -e "s,/etc/init.d/vboxdrv setup',/etc/rc.d/vboxdrv start'  ,g" "$pkgdir/opt/VirtualBox/VirtualBox.so"

    # Add udev rules
    mkdir -p "$pkgdir/etc/udev/rules.d"
    echo 'KERNEL=="vboxdrv", NAME="vboxdrv", OWNER="root", GROUP="vboxusers", MODE="0660"' \
        > "$pkgdir/etc/udev/rules.d/10-vboxdrv.rules"
    echo 'SUBSYSTEM=="usb_device", GROUP="vboxusers", MODE="0664"' \
        >> "$pkgdir/etc/udev/rules.d/10-vboxdrv.rules"
    echo 'SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", GROUP="vboxusers", MODE="0664"' \
        >> "$pkgdir/etc/udev/rules.d/10-vboxdrv.rules"

    # Point the installation directory to vbox
    echo '# VirtualBox installation directory' > "$pkgdir/etc/vbox.cfg"
    echo 'INSTALL_DIR="/opt/VirtualBox"' >> "$pkgdir/etc/vbox.cfg"

    # Link the license
    #mkdir -p "$pkgdir/usr/share/licenses/$pkgname"
    #ln -s "/opt/VirtualBox/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/PUEL"

    # create symlinks for dkms, too
    mkdir -p $pkgdir/usr/src
    . $pkgdir/etc/conf.d/vboxdrv
    ln -s /opt/VirtualBox/src/vboxhost $pkgdir/usr/src/vboxhost-$VBOX_VERSION
}

# vim:set ts=4 sw=4 et:
