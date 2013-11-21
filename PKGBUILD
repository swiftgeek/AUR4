# Contributor && Maintarner: Swift Geek <swiftgeek ɐt gmail døt com>
# Contributor: Gaetan Bisson <bisson@archlinux.org>
# Contributor: hokasch <hokasch at operamail dot com>
# Contributor: Dennis Brendel <buddabrod at gmail dot com>
# Contributor: Mathias Buren <mathias.buren at gmail dot com>
# Contributor: Benjamin Mtz (Cruznick) <cruznick at archlinux dot us>

_runkernver=$(uname -r)
_shortkernver=${_runkernver%.*}

pkgname=backports-patched
pkgver=3.12_1
_upver="${pkgver//_/-}"
pkgrel=6
pkgdesc='Backports provides drivers released on newer kernels backported for usage on older kernels. Patched flavor'
url='https://backports.wiki.kernel.org/index.php/Main_Page'
arch=('i686' 'x86_64')
license=('GPL')
depends=('linux')
makedepends=('linux-api-headers' "linux-headers>=$_shortkernver")
optdepends=('backports-frag+ack: wl-frag+ack patch')
install=backports.install
# Stable and rc? TODO: Check with rc :D | Double %% cuts to the first, single % cuts to the last
source=("http://www.kernel.org/pub/linux/kernel/projects/backports/stable/v${_upver%-*}/backports-${_upver}.tar.xz")
# Snapshot:
#source=("http://www.kernel.org/pub/linux/kernel/projects/backports/${pkgver:0:4}/${pkgver:4:2}/${pkgver:6:2}/backports-${pkgver}.tar.xz")
sha256sums=('9833d43dc676eb0029b452e13a3eb24d89b5a5c065e39b9d71b1a023cc3c2b84')

# Check for daily pkgver eg. 20370718
date -d "$pkgver" > /dev/null 2>&1
if [[ $? == 0 ]]; then
  source=("http://www.kernel.org/pub/linux/kernel/projects/backports/${pkgver:0:4}/${pkgver:4:2}/${pkgver:6:2}/backports-${pkgver}.tar.xz")
  sha256sums=('SKIP')
  warning "Skipping checksum check for snapshots"
fi

_extramodules=extramodules-${_shortkernver}-ARCH
_kernver=$(cat /usr/lib/modules/${_extramodules}/version) # TODO make this a lower boundary and utilize in reality pacman to get freshest paths. Or make it for specific kernels. Or multiply it over specific kernels ? :3

_cfgdir="/etc/makepkg.d/${pkgname}/"
_patchdir="${_cfgdir}/patches/"

countdown() {
  local i 
  for ((i=$1; i>=1; i--)); do
    [[ ! -e /proc/$$ ]] && exit
    echo -ne "\rPress [i] to start interactive config in $i second(s) "
    sleep 1
  done
}

prepare() {
  cd "${srcdir}/backports-${_upver}"
  # modprobe -l dropped in kmod
  sed 's:modprobe -l \([^ )`]*\):find /usr/lib/modules/*/kernel -name "\1.ko*" | sed "s|.*/kernel||":' -i scripts/*
  sed 's:\$(MODPROBE) -l \([^ )`]*\):find /usr/lib/modules/*/kernel -name "\1.ko*" | sed "s|.*/kernel||":' -i Makefile

  # rfkill.h does not use compat-3.1.h # TODO : IS THIS RLY NEEDED ?
  #echo '#define br_port_exists(dev) (dev->priv_flags & IFF_BRIDGE_PORT)' >> net/wireless/core.h

  # Patch time!
  # WARNING - PART BELOW IS UNTESTED: current format - 00-name for cfgfile (to export variables) and 12-3-name for patches (3 stands for -p3)
  if [ -d "${_cfgdir}" ]; then
    _CFGLIST=$(ls -1 "${_cfgdir}" | awk ' /^[0-9][0-9]-.*/ {print "source '${_cfgdir}'"$0";"}')
    eval $_CFGLIST
    unset _CFGLIST
    if [ -d "${_patchdir}" ]; then
      _PATCHLIST=$(ls -1 "${_patchdir}" | awk ' /^[0-9][0-9]-[0-9]-.*/ {print $0}')
      for _patch in $_PATCHLIST; do
        msg "Merging patch: ${_patch}"
        patch -p${_patch:3:1} -i ${_patchdir}/${_patch}
      done
      unset _PATCHLIST
    fi
  fi

  # Patch for sane install
cd "${srcdir}/backports-${_upver}"
patch -p0 <<'EOF'
--- Makefile.real	2013-07-13 18:50:46.000000000 +0200
+++ Makefile.real.fixed	2013-07-28 01:52:51.922779881 +0200
@@ -87,15 +87,6 @@
 	@$(MAKE) -C $(KLIB_BUILD) M=$(BACKPORT_PWD)			\
 		INSTALL_MOD_DIR=$(KMODDIR) $(KMODPATH_ARG)		\
 		modules_install
-	@./scripts/blacklist.sh $(KLIB)/ $(KLIB)/$(KMODDIR)
-	@./scripts/compress_modules.sh $(KLIB)/$(KMODDIR)
-	@./scripts/check_depmod.sh
-	@./scripts/backport_firmware_install.sh
-	@/sbin/depmod -a
-	@./scripts/update-initramfs.sh $(KLIB)
-	@echo
-	@echo Your backported driver modules should be installed now.
-	@echo Reboot.
 	@echo
 
 .PHONY: modules_install
EOF

}

build() {
  cd "${srcdir}/backports-${_upver}"
  # unset _selected_drivers # WARNING! DEBUGGING UNSET - MAKE SURE THAT THIS LINE IS COMMENTED
  # Get config - not in prepare beause interactive part is using cc
  if [ -n "${_selected_drivers}" ]; then
    msg "Config detected"
    make "${_selected_drivers[@]/#/defconfig-}"
  else # TODO: else if not that try showing up dialog menu with checkboxes based on available defconfigs ;)
    warning "Config undetected"
    # WAIT 10s FOR INTEACTIVE PART - PRESS I FOR INTERACTIVE CONFIG
    tty -s # Checks if user input is accesssible, otherwise fail
    countdown 10 & countdown_pid=$!
    read -s -n 1 -t 10 ikey || true
    kill $countdown_pid
    echo -e -n "\n"
    [[ "$ikey" != "i" ]] && false
    # BEGIN INTERACTIVE PART TODO: ADD OLDCONFIG OPTION WITH FILE SELECT
    cfgway=$(dialog --clear --backtitle "$pkgname" --radiolist 'Choose method to configure' 0 0 0 defconfig 'desc' off "menuconfig" 'desc' off 2>&1 >/dev/tty)
    case "$cfgway" in
      defconfig)
        for i in $(ls ./defconfigs/); do
          list_opts+=("$i" "desc" "off")
        done
        echo "${list_opts[@]}"
        _selected_drivers=$(dialog --clear --backtitle "$pkgname" --checklist 'Choose driver groups to compile' 0 0 0 "${list_opts[@]}" 2>&1 >/dev/tty)
        make "${_selected_drivers[@]/#/defconfig-}"
        ;;
      menuconfig)
        make menuconfig
        ;;
      *)
        echo Break out from the PKGBUILD
        false
        ;;
    esac
    # END OF THE INTERACTIVE PART
  fi
  [[ -n "$_selected_drivers" ]] && msg "» $_selected_drivers «" # CONVERT TO MESSAGE AND ONLY IF VAR IS -n

  # Actual build
  msg "Starting actual build"
  make KLIB=/usr/lib/modules/"${_kernver}" # should be make modules
}

package() {
  cd "${srcdir}/backports-${_upver}"
  mkdir ${pkgdir}/usr/
  make INSTALL_MOD_PATH="${pkgdir}/usr" KMODDIR=../"${_extramodules}" install
  find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;

  #install -d "${pkgdir}"/usr/bin
  #install scripts/*{enable,load} "${pkgdir}"/usr/bin

  #install -d "${pkgdir}"/usr/lib/compat-drivers
  #install scripts/modlib.sh "${pkgdir}"/usr/lib/compat-drivers

  install -d "${pkgdir}"/usr/lib/udev/rules.d
  install udev/compat_firmware.sh	"${pkgdir}"/usr/lib/udev
  install udev/50-compat_firmware.rules "${pkgdir}"/usr/lib/udev/rules.d

  # Preparation for future
  install -d "${pkgdir}"/etc/makepkg.d/"${pkgname}"/patches
}
