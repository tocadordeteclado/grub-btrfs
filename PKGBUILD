#
# Contribuidor: {{ nome_do_autor(); }}
#

pkgname=grub-btrfs
pkgver=4.12
pkgrel=2
pkgdesc='Incluir instantâneos btrfs nas opções de inicialização do GRUB'
arch=('any')
url="http://localhost/grub-btrfs"
license=('cIO')
depends=('btrfs-progs' 'grub')
optdepends=(
    'snapper: Para suporte de snapper'
    'inotify-tools: Para serviços de grub-btrfsd'
)

backup=('etc/default/grub-btrfs/config')
source=("grub-btrfs.tar.gz")
b2sums=('SKIP')

package()
{
    cd "${pkgname}-${pkgver}"
    make DESTDIR="${pkgdir}" INITCPIO=true install
}
