# Maintainer: 7Ji <pugokughin@gmail.com>

pkgname=linux-firmware-orangepi-git
pkgver=20230828.b2809d6.20240115.9b6d0b08
pkgrel=1
pkgdesc="Firmware files for Linux - Orange Pi specific firmware"
arch=('any')
makedepends=('git')
url="https://github.com/orangepi-xunlong/firmware"
license=('GPL2' 'GPL3' 'custom')
conflicts=('linux-firmware-orangepi')
replaces=('linux-firmware-orangepi')
options=(!strip)
source=("git+${url}.git")
sha256sums=('SKIP')
depends=('linux-firmware')

pkgver() {
  cd firmware

  local _ver_linux_firmware=$(LANG=C pacman -Qi linux-firmware | grep -E '^Version' | awk '{print $3}')
  _ver_linux_firmware="${_ver_linux_firmware##*:}"
  _ver_linux_firmware="${_ver_linux_firmware%%-*}"

  # Commit date + short rev
  printf '%s.%s.%s' \
    $(TZ=UTC git show -s --pretty=%cd --date=format-local:%Y%m%d HEAD) \
    $(git rev-parse --short HEAD) \
    "${_ver_linux_firmware}"
}

package() {
  local _prefix="${pkgdir}"/usr/lib
  install -d "${_prefix}"
  pacman -Ql linux-firmware | sed -n 's|^linux-firmware /usr/lib/\(firmware/.*\)|\1|p' | sort | uniq > linux-firmware.list
  local _file _target _link
  if [[ "${CARCH}" == 'aarch64' ]]; then
    local _nocomp='yes'
  else
    local _nocomp=''
  fi
  for _file in $(find firmware/*); do
    grep -q "^${_file}\(.zst\|/\)\?$" linux-firmware.list && continue
    if [[ -L "${_file}" ]]; then
      echo "L: ${_file}"
      _target=$(readlink "${_file}")
      if [[ -z "${_nocomp}" && -f "${_file}" ]]; then
        _target+='.zst'
        _file+='.zst'
      fi
      _link="${_prefix}/${_file}"
      install -d "${_link%/*}"
      ln -s "${_target}" "${_link}"
    elif [[ -f "${_file}" ]]; then
      echo "F: ${_file}"
      if [[ "${_nocpm}" ]]; then
          install -Dm644 "${_file}" "${_prefix}/${_file}.zst"
      else
        zstd --compress --quiet --stdout "${_file}" |
          install -Dm644 /dev/stdin "${_prefix}/${_file}.zst"
      fi
    else
      echo "D: ${_file}"
      install -d "${_prefix}/${_file}"
    fi
  done
}
