# Maintainer: Achilleas Michailidis <achmichail@gmail.com>
# Contributor: Gabriel Souza Franco <Z2FicmllbGZyYW5jb3NvdXphQGdtYWlsLmNvbQ==>
# Contributor: Nico Rumpeltin <$forename at $surname dot de>
# Contributor: Matthias Blaicher <matthias at blaicher dot com>
# Contributor: Danny Dutton <duttondj@vt.edu>
#
# NOTE: If you plan on using the usbblaster make sure you are member of the plugdev group.
#
pkgname=quartus
# Keep dot in _patchver
_mainver=20.1; _patchver=.1; _buildver=720
# Latest HLS compiler was only released with Pro numbering
_promain=20.3; _propatch=.0; _probuild=158; _prover=${_promain}${_propatch}.${_probuild}
pkgver=${_mainver}${_patchver}.${_buildver}
pkgrel=1
pkgdesc="Quartus Prime Lite design software for Intel FPGAs"
arch=('x86_64')
url="http://fpgasoftware.intel.com/?edition=lite"
license=('custom')

_alteradir="/opt/intelFPGA/${_mainver}"

# According to the installer script, these dependencies are needed for the installer
depends=('ld-lsb' 'lib32-expat' 'lib32-fontconfig' 'lib32-freetype2' 'lib32-glibc'
         'lib32-gtk2' 'lib32-libcanberra' 'lib32-libpng' 'lib32-libice' 'lib32-libsm'
         'lib32-util-linux' 'lib32-ncurses' 'lib32-ncurses5-compat-libs' 'lib32-zlib'
         'lib32-libx11' 'lib32-libxau' 'lib32-libxdmcp' 'lib32-libxext' 'lib32-libxft'
         'lib32-libxrender' 'lib32-libxt' 'lib32-libxtst')
optdepends=("eclipse: For Nios II EDS")

_base_url="https://download.altera.com/akdlm/software/acdsinst"
source=("${_base_url}/${_mainver}std${_patchver/.0/}/${_buildver}/ib_installers/QuartusLiteSetup-${pkgver}-linux.run"
        'quartus.sh' 'quartus.desktop' '51-usbblaster.rules')
md5sums=('3a5ca38169bcff285611789850b5af83'
         '60fbfafbaa565af5e97b2904914e41e7'
         'c5a8f6310ade971f07e5ee6c4e338054'
         'f5744dc4820725b93917e3a24df13da9')
options=(!strip !debug) # Stripping will takes ages, I'd avoid it
PKGEXT=".pkg.tar.zst" # ZSTD is fast enough for compression

prepare() {
    echo "Notice: Requires around 16GB of free space during package building!"
    echo "Notice: The package files also requires around 9GB of free space"

    chmod +x QuartusLiteSetup-${pkgver}-linux.run
}

package() {

    DISPLAY="" ./QuartusLiteSetup-${pkgver}-linux.run \
        --disable-components quartus_help,devinfo,modelsim_ase,modelsim_ae \
        --mode unattended \
        --unattendedmodeui none \
        --accept_eula 1 \
        --installdir "${pkgdir}${_alteradir}"

    # Remove uninstaller and install logs since we have a working package management
    rm -r "${pkgdir}${_alteradir}/uninstall"
    rm -r "${pkgdir}${_alteradir}/logs"

    # Remove useless unzip binaries
    find "${pkgdir}${_alteradir}" \( -name "unzip" -or -name "unzip32" \) -delete

    # Remove duplicated file from help
    rm -r "${pkgdir}${_alteradir}/quartus/common/help/webhelp"

    # Fix missing permissions
    find "${pkgdir}${_alteradir}" \! -perm /o+rwx -exec chmod o=g {} \;

    # Replace altera directory in integration files
    sed -i "s,_alteradir,${_alteradir},g" quartus.sh
    sed -i "s,_alteradir,${_alteradir},g" quartus.desktop

    # Remove pkgdir reference in sopc_builder
    sed -i "s,${pkgdir},,g" "${pkgdir}${_alteradir}/quartus/sopc_builder/.sopc_builder"

    # Fix world writable permissions
    find "${pkgdir}${_alteradir}/nios2eds/documents" -perm -o+w -exec chmod go-w {} \+
    find "${pkgdir}${_alteradir}/quartus/common/tcl" -perm -o+w -exec chmod go-w {} \+
    find "${pkgdir}${_alteradir}/quartus/linux64" -perm -o+w -exec chmod go-w {} \+
    find "${pkgdir}${_alteradir}/quartus/sopc_builder/bin/europa" -perm -o+w -exec chmod go-w {} \+

    # Copy license file
    install -D -m644 "${pkgdir}${_alteradir}/licenses/license.txt" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

    # Install integration files
    install -D -m755 quartus.sh "${pkgdir}/etc/profile.d/quartus.sh"
    install -D -m644 51-usbblaster.rules "${pkgdir}/etc/udev/rules.d/51-usbblaster.rules"
    install -D -m644 quartus.desktop "${pkgdir}/usr/share/applications/quartus.desktop"
}
