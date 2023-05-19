### Descrição:

A inicialização em um instantâneo no modo somente leitura
pode ser complicada. Uma maneira elegante é inicializar
este instantâneo usando overlayfs (incluído no kernel).

Usando overlayfs, o instantâneo inicializado se comportará
como um CD ao vivo no modo não persistente. O instantâneo
não será modificado, o sistema poderá inicializar corretamente,
pois uma pasta gravável será incluída no bloco RAM. (sem
novas falhas devido a `/var` não abrir para escrita).

Quaisquer alterações neste sistema assim iniciado serão
perdidas quando o sistema for reinicializado/desligado.

Para fazer isso, é necessário modificar a página initramfs.
Isso significa que qualquer instantâneo que não inclua esse
initramfs modificado não poderá se beneficiar dele. (exceto
para partições de inicialização separadas).

#
### Instalação:
#### Arch Linux

 1.
   `Pacman -S grub-btrfs` Ou se você usar o git:
      copie a página `overlay_snap_ro-install` para `/etc/initcpio/install/grub-btrfs-overlayfs`,
      copie a página `overlay_snap_ro-hook` para `/etc/initcpio/hooks/grub-btrfs-overlayfs`.
      Você deve renomear as páginas. (eu fiz isso acima).

      Por exemplo:
         `overlay_snap_ro-install` para `grub-btrfs-overlayfs`,
         `overlay_snap_ro-hook` para `grub-btrfs-overlayfs`.

      Lembre-se de que as páginas devem ter exatamente o mesmo
      nome para garantir uma correspondência.

 2.
   Edite a página `/etc/mkinitcpio.conf`.
   Plug adicionado em `grub-btrfs-overlayfs` no final da linha `HOOKS`.

   Por exemplo:
      `HOOKS=(base udev autodetect modconf block filesystems keyboard fsck grub-btrfs-overlayfs)`.
      Você percebe que o nome do `hook` deve corresponder ao
      nome das duas páginas instaladas. (não se esqueça disso).

 3.
   Gere novamente o seu initramfs.
   `mkinitcpio -P` (opção -P significa, todos os pré-ajustes
   presentes em `/etc/mkinitcpio.d`).

#### Outra distribuição
Consulte o manual da sua distribuição ou contribua
com este projeto para adicionar um parágrafo.
#
