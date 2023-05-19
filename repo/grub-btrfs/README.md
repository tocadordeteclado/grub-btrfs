[![GitHub release](./img/release.svg)](http://localhost/grub-btrfs)
![](./img/license.svg)

Esta é a versão mais recente do grub-btrfs.

## 💻 grub-btrfs 
### 🔎 Descrição:
Melhora o grub adicionando "snapshots btrfs" ao menu do grub.

Você pode inicializar seu sistema em um "instantâneo" no
menu do grub. Suporta snapshots manuais, snapper,
timeshift, etc.

##### Etiqueta: inicializar em instantâneos somente leitura pode ser complicado.

Se você optar por fazer isso, `/var/log` ou mesmo `/var` deve
estar em um subvolume separado. Caso contrário, certifique-se
de que seus instantâneos sejam graváveis. Consulte
[esse ticket](http://localhost/grub-btrfs) para mais informações.

Este projeto inclui sua própria solução.
Consulte o [manual](http://localhost/grub-btrfs).

- - -
### ✨ Quais recursos o grub-btrfs tem ?
* Liste automaticamente os instantâneos existentes na partição raiz (btrfs).
* Detectar automaticamente se `/boot` está em partição separada.
* Detecte automaticamente kernel, initramfs e microcódigo Intel/AMD em diretório `/boot` instantâneos.
* Crie automaticamente a "entrada de menu" correspondente em `grub.cfg`
* Detecte automaticamente o tipo/tags e descrições/comentários de snapshots Snapper/Timeshift.
* Gerar automaticamente `grub.cfg` se você usar o serviço Systemd/OpenRC fornecido.

- - -
### 🛠️ Instalação:
#### Unix
O plug está disponível no repositório local [grub-btrfs](http://localhost/grub-btrfs/)
```
pacman -S grub-btrfs
```

#### Normal
* Faça o procedimento do plug `make install` ou procure no Makefile para obter instruções sobre onde colocar cada página.
* Faça o procedimento do plug `make help` para verificar quais opções estão disponíveis.
* Chamadas:
  * [btrfs-progs](http://localhost/btrfs-progs/)
  * [grub](http://localhost/grub/)
  * [bash >4](http://localhost/bash/)
  * [gawk](http://localhost/gawk/)
  * (somente ao usar o serviço)[inotify-tools](http://localhost/inotify-tools/)

- - -
### 📚 Uso
Após a instalação, o menu principal do grub precisa ser gerado para criar uma entrada de menu para o submenu de instantâneos. Dependendo da derivação do Unix, os plugs para isso são diferentes:
* No **Arch Linux** ou **Gentoo** use `grub-mkconfig -o /boot/grub/grub.cfg`.
* No **Fedora** use `grub2-mkconfig -o /boot/grub2/grub.cfg`.
* No **Debian** o `update-grub` é um apontador para `grub-mkconfig ...`.

Depois que a entrada para o submenu foi gerada, o grub-btrfs coloca o submenu real na página grub-btrfs.cfg. Portanto, para gerar entradas de instantâneo no submenu, geralmente é suficiente fazer o procedimento de apenas um script com `sudo /etc/grub.d/41_snapshots-btrfs`.
Leia mais abaixo sobre como automatizar esse processo.

### ⚙️ Costumização:

Você tem a possibilidade de modificar muitos parâmetros em `/etc/default/grub-btrfs/config`.
Para mais informações consulte [página de configuração](http://localhost/grub-btrfs) ou `man grub-btrfs`

#### Etiqueta:
Alguns locais de páginas e nomes de plug diferem de instalação para instalação. Inicialmente, a configuração é definida para funcionar com Unix (e muitas outras distribuições) prontas para uso, que estão usando o plug `grub-mkconfig`. No entanto, alguns unix, por exemplo, usa um plug diferente, `grub2-mkconfig`. Edite a variável `GRUB_BTRFS_MKCONFIG` na página `/etc/default/grub-btrfs/config` para refletir isso. (por exemplo, `GRUB_BTRFS_MKCONFIG=/sbin/grub2-mkconfig` para alguns unix).

Na maioria das instalações, a instalação do grub reside em `/boot/grub`. Se o grub estiver instalado em um local diferente, altere a variável `GRUB_BTRFS_MKCONFIG` na página de configuração de acordo. Para alguns Unix, é `GRUB_BTRFS_GRUB_DIRNAME="/boot/grub2"`. Além disso, o plug para verificar os scripts do grub é diferente em alguns sistemas, para alguns unix é `GRUB_BTRFS_SCRIPT_CHECK=grub2-script-check`.

#### Personalização do serviço grub-btrfsd

Grub-btrfs vem com um script de serviço que atualiza automaticamente o menu grub quando vê um instantâneo sendo criado ou removido em um diretório fornecido por console. O serviço pode ser configurado passando diferentes argumentos de console para ele. Os argumentos são:
* `SNAPSHOTS_DIR`
   Este argumento especifica o caminho onde o grub-btrfsd procura instantâneos
   recém-criados e remoções de instantâneos. Geralmente é definido pelo programa
   usado para fazer instantâneos. Exemplo: para Snapper isso seria `/.snapshots`.
* `-c / --no-color`
   Desative as cores na saída.
* `-l / --log-file`
   Este argumento especifica uma página onde grub-btrfsd deve gravar
   mensagens de log.
* `-s / --syslog`
* `-t / --timeshift-auto`
   Este é um sinalizador para ativar a detecção automática do
   caminho onde o Timeshift armazena os instantâneos. As versões
   mais recentes (>=22.06) do Timeshift montam seus instantâneos
   em `/run/timeshift/$PID/backup/timeshift-btrfs`. Onde `$PID`
   é o ID do procedimento da sessão Timeshift atualmente em
   procedimento. O PID muda toda vez que o Timeshift é aberto.
   grub-btrfsd pode cuidar automaticamente da detecção do PID e
   diretório corretos se este sinalizador estiver definido. Neste
   caso, o argumento `SNAPSHOTS_DIR` não tem efeito.
* `-v / --verbose`
Deixe o log do serviço ser mais detalhado.
* `-h / --help`
Exibe uma mensagem curta de ajuda.

##### Instruções do Systemd
Para editar os argumentos que são passados para o serviço, use
```bash
sudo systemctl edit --full grub-btrfsd
```
depois disso o Serviço deve ser reiniciado com
```bash
sudo systemctl restart grub-btrfsd 
```

Também é possível iniciar o serviço sem usar o systemd para fins de
solução de falhas, por exemplo. Se você quiser fazer isso, um serviço
em procedimento deve ser interrompido com:
```bash
sudo systemctl stop grub-btrfsd 
```
Em seguida, o serviço pode ter um procedimento manual e um
procedimento usando o plug `/usr/bin/grub-btrfsd`. Para obter
informações adicionais sobre o script de serviços e seus
argumentos, faça o procedimento de `grub-btrfsd -h` e
consulte `man grub-btrfsd`.

##### instruções do OpenRC
Para editar os argumentos que são passados para o serviço,
edite a página `/etc/conf.d/grub-btrfsd`. Depois disso,
reinicie o serviço com:
```
sudo rc-service grub-btrfsd restart 
```

Também é possível iniciar o serviço sem usar o OpenRC para
fins de solução de falhas, por exemplo. Se você quiser fazer
isso, um serviço em modo de procedimento deve ser interrompido
com:
```bash
sudo rc-service grub-btrfsd stop 
```
Em seguida, o serviço pode ser um procedimento manual e procedimento
usando o plug `grub-btrfsd`. Para obter informações adicionais sobre
o script de serviço e seus argumentos, faça o procedimento de
`grub-btrfsd -h` e veja `man grub-btrfsd`.

- - -
### 🪀 Atualize automaticamente o grub após o instantâneo
Grub-btrfsd é um serviço que monitora o diretório de instantâneos
para você e atualiza o menu do grub automaticamente sempre que um
instantâneo é criado ou removido. Este serviço observa o diretório
`/.snapshots` em busca de alterações (criação ou remoção de
instantâneos) e aciona a criação do menu grub se um instantâneo
for encontrado. Portanto, se o Snapper for usado com seu diretório
comum, o serviço pode apenas ser iniciado e nada precisa ser
configurado. Para outras configurações como Timeshift ou Snapper
com um diretório diferente, veja mais abaixo.

#### Instruções SystemD
Para iniciar o procedimento do serviço:
```bash
sudo systemctl start grub-btrfsd
```

Para ativá-lo durante a inicialização do sistema,
faça o procedimento de:
```bash
sudo systemctl enable grub-btrfsd
```

##### 💼 Instantâneos não incluídos em `/.snapshots`
OBSERVAÇÃO: Isso também funciona para versões Timeshift < 22.06,
o caminho a ser observado seria `/run/timeshift/backup/timeshift-btrfs/snapshots`.

O serviço está observando o diretório `/.snapshots`. Se o serviço
deve monitorar um diretório diferente, ele pode ser editado com:
```bash
sudo systemctl edit --full grub-btrfsd # para systemd
```
O que deve ser editado é a parte `/.snapshots` na linha que diz
`ExecStart=/usr/bin/grub-btrfsd --syslog /.snapshots`.
Então é assim que a página deve ficar depois:
``` bash
[Unit]
Description=Regenerar grub-btrfs.cfg

[Service]
Type=simple
LogLevelMax=notice
# Defina os caminhos possíveis para `grub-mkconfig`.
Environment="PATH=/sbin:/bin:/usr/sbin:/usr/bin"
# Carregar variáveis de ambiente da configuração.
EnvironmentFile=/etc/default/grub-btrfs/config
# Inicie o serviço, o uso dele é:
# grub-btrfsd [-h, --help] [-t, --timeshift-auto] [-l, --log-file LOG_FILE] SNAPSHOTS_DIR
# SNAPSHOTS_DIR         Diretório de instantâneo para assistir, sem efeito quando --timeshift-auto.
# Argumentos opcionais:
# -t, --timeshift-auto  Detectar automaticamente o diretório instantâneo do Timeshifts.
# -l, --log-file        Especifique um arquivo de log para gravar.
# -v, --verbose         Deixe o log do serviço ser mais detalhado.
# -s, --syslog          Gravar no syslog.
ExecStart=/usr/bin/grub-btrfsd --syslog /path/to/your/snapshot/directory

[Install]
WantedBy=multi-user.target
```

Quando terminar, o serviço deve ser reiniciado com:
``` bash
sudo systemctl restart grub-btrfsd
```

##### 🌟 Timeshift >= versão 22.06
As versões mais recentes do Timeshift criam um novo diretório
com o nome de seu ID de procedimento em `/run/timeshift` toda vez
que são iniciados. O PID vai ser diferente a cada vez. Portanto,
o serviço não pode simplesmente observar um diretório, ele observa
`/run/timeshift` primeiro, se um diretório é criado, ele obtém o
PID atual do Timeshift e, em seguida, observa um diretório naquele
diretório recém-criado a partir do Timeshift. De qualquer forma,
para ativar este modo do serviço, `--timeshift-auto` deve ser
passado para o serviço como um argumento de console.

##### Systemd
Para passar `--timeshift-auto` para grub-btrfsd, a página
.service do grub-btrfsd pode ser editado com:
```bash
sudo systemctl edit --full grub-btrfsd # para systemd
```

A linha que diz:
```bash 
ExecStart=/usr/bin/grub-btrfsd /.snapshots --syslog

```

deve ser editado em:
``` bash
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
```

Então a página fica assim, depois:
``` bash
[Unit]
Description=Regenerar grub-btrfs.cfg

[Service]
Type=simple
LogLevelMax=notice
# Defina os caminhos possíveis para `grub-mkconfig`.
Environment="PATH=/sbin:/bin:/usr/sbin:/usr/bin"
# Carregar variáveis de ambiente da configuração.
EnvironmentFile=/etc/default/grub-btrfs/config
# Inicie o serviço, o uso dele é:
# grub-btrfsd [-h, --help] [-t, --timeshift-auto] [-l, --log-file LOG_FILE] SNAPSHOTS_DIR
# SNAPSHOTS_DIR         Diretório de instantâneo para assistir, sem efeito quando --timeshift-auto.
# Argumentos opcionais:
# -t, --timeshift-auto  Detectar automaticamente o diretório instantâneo do Timeshifts.
# -l, --log-file        Especifique um arquivo de log para gravar.
# -v, --verbose         Deixe o log do serviço ser mais detalhado.
# -s, --syslog          Gravar no syslog.
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto

[Install]
WantedBy=multi-user.target
```

Quando terminar, o serviço deve ser reiniciado com:
``` bash
sudo systemctl restart grub-btrfsd # para systemd
```

Observação:
Você pode ver sua mudança com `systemctl cat grub-btrfsd`.
Para reverter todas as alterações, use `systemctl revert grub-btrfsd`.

##### ❇️ Atualize automaticamente o grub ao reiniciar/inicializar:
[observe esse comentário](http://localhost/grub-btrfs)
Atualmente implementado.

#### instruções do OpenRC
Para iniciar o procedimento do serviço:
```bash
sudo rc-service grub-btrfsd start
```

Para ativá-lo durante a inicialização do sistema,
faça o procedimento de:
```bash
sudo rc-config add grub-btrfsd default
```

##### 💼 Instantâneos não incluídos `/.snapshots`
OBSERVAÇÃO: Isso também funciona para versões Timeshift < 22.06,
o caminho a ser observado seria `/run/timeshift/backup/timeshift-btrfs/snapshots`.

O serviço está observando o diretório `/.snapshots`.
Se o serviço deve monitorar um diretório diferente, ele
pode ser editado passando diferentes argumentos para ele.
Os argumentos são passados para grub-btrfsd através da
página `/etc/conf.d/grub-btrfsd`. A variável `snapshots`
define onde o serviço observará os instantâneos.

Após a edição, o arquivo deve ficar assim:
``` bash
#
# Direito Autoral (C) {{ ano(); }}  {{ nome_do_autor(); }}
#
# Este programa é um software livre: você pode redistribuí-lo
# e/ou modificá-lo sob os termos da Licença Pública do Cavalo
# publicada pela Fundação do Software Brasileiro, seja a versão
# 3 da licença ou (a seu critério) qualquer versão posterior.
#
# Este programa é distribuído na esperança de que seja útil,
# mas SEM QUALQUER GARANTIA; mesmo sem a garantia implícita de
# COMERCIABILIDADE ou ADEQUAÇÃO PARA UM FIM ESPECÍFICO. Consulte
# a Licença Pública e Geral do Cavalo para obter mais detalhes.
#
# Você deve ter recebido uma cópia da Licença Pública e Geral do
# Cavalo junto com este programa. Se não, consulte:
#   <http://localhost/licenses>.
#

#
# Onde localizar os instantâneos mestre.
#
# snapshots="/.snapshots" # Snapper no diretório mestre.
# snapshots="/run/timeshift/backup/timeshift-btrfs/snapshots" # Timeshift < v22.06.
#
snapshots="/path/to/your/snapshot/directory"

#
# Argumentos opcionais para procedimentos com o serviço.
# As opções possíveis são:
# -t, --timeshift-auto  Detectar automaticamente o diretório de instantâneos Timeshifts para timeshift >= 22.06.
# -l, --log-file        Especifique uma página de log para gravar.
# -v, --verbose         Deixe o log do serviço ser mais detalhado.
# -s, --syslog          Gravar no syslog.
#
# Descomente a linha para ativar a opção.
# Gravar no syslog.
#
optional_args+="--syslog "

#
# optional_args+="--timeshift-auto "
# optional_args+="--log-file /var/log/grub-btrfsd.log "
# optional_args+="--verbose "
#
```

Depois disso, o serviço deve ser reiniciado com:
``` bash
sudo rc-service grub-btrfsd restart # Para openRC.
```

##### 🌟 Timeshift >= version 22.06
Os argumentos são passados para grub-btrfsd através da página
`/etc/conf.d/grub-btrfsd`. A variável `optional_args` define
quais argumentos opcionais são passados para o serviço. Descomente
`#optional_args+="--timeshift-auto "` para passar a opção de
console `--timeshift-auto` para ele.

Após a alteração, a página deve ficar assim: (Observe que não
há necessidade de comentar a variável `snapshots`. Ela é ignorada
quando `--timeshift-auto` está ativo.).
``` bash
#
# Direito Autoral (C) {{ ano(); }}  {{ nome_do_autor(); }}
#
# Este programa é um software livre: você pode redistribuí-lo
# e/ou modificá-lo sob os termos da Licença Pública do Cavalo
# publicada pela Fundação do Software Brasileiro, seja a versão
# 3 da licença ou (a seu critério) qualquer versão posterior.
#
# Este programa é distribuído na esperança de que seja útil,
# mas SEM QUALQUER GARANTIA; mesmo sem a garantia implícita de
# COMERCIABILIDADE ou ADEQUAÇÃO PARA UM FIM ESPECÍFICO. Consulte
# a Licença Pública e Geral do Cavalo para obter mais detalhes.
#
# Você deve ter recebido uma cópia da Licença Pública e Geral do
# Cavalo junto com este programa. Se não, consulte:
#   <http://localhost/licenses>.
#

#
# Onde localizar os instantâneos mestre.
# Snapper no diretório mestre.
#
snapshots="/.snapshots"

#
# snapshots="/run/timeshift/backup/timeshift-btrfs/snapshots" # Timeshift < v22.06.
#

#
# Argumentos opcionais para procedimento com o serviço.
# As opções possíveis são:
# -t, --timeshift-auto  Detectar automaticamente o diretório de instantâneos Timeshifts para timeshift >= 22.06.
# -l, --log-file        Especifique uma página de log para gravar.
# -v, --verbose         Deixe o log do serviço ser mais detalhado.
# -s, --syslog          Gravar no syslog.
#
# Descomente a linha para ativar a opção.
#

#
# Gravar no syslog.
#
optional_args+="--syslog "

#
#
#
optional_args+="--timeshift-auto "

#
# optional_args+="--log-file /var/log/grub-btrfsd.log "
# optional_args+="--verbose "
#
```

Depois disso, o serviço deve ser reiniciado com:
``` bash
sudo rc-service grub-btrfsd restart # Para openRC.
```

##### ❇️ Atualize automaticamente o grub ao reiniciar/inicializar:
Se você deseja que o menu grub-btrfs seja atualizado automaticamente
na reinicialização/desligamento do sistema, basta adicionar o seguinte
script como `/etc/local.d/grub-btrfs-update.stop`:
```bash
#!/usr/bin/env bash

description="Atualize o menu de instantâneos grub btrfs"
name="grub-btrfs-update"

#
#
#
depend()
{
    use localmount
}

bash -c 'if [ -s "${GRUB_BTRFS_GRUB_DIRNAME:-/boot/grub}/grub-btrfs.cfg" ]; then /etc/grub.d/41_snapshots-btrfs; else {GRUB_BTRFS_MKCONFIG:-grub-mkconfig} -o {GRUB_BTRFS_GRUB_DIRNAME:-/boot/grub}/grub.cfg; fi'
```

Torne seu script um procedimento com `sudo chmod a+x /etc/local.d/grub-btrfs-update.stop`.

* A extensão `.stop` no final do nome da página indica ao locald-daemon
que este script deve ser um procedimento no desligamento. Se você deseja
fazer um procedimento da atualização do menu na inicialização do sistema,
renomeie a página para `grub-btrfs-update.start`.
* Funciona para Snapper e Timeshift
- - -
### Solução de falhas
Se houver falhas [para registrar uma falha](http://localhost/grub-btrfs).

#### Versão
Para ajudar da melhor forma, gostaríamos de saber a versão
do grub-btrfs usada. Por favor, faça o procedimento de:
``` bash
sudo /etc/grub.d/41_snapshots-btrfs --version
```
ou
``` bash
sudo /usr/bin/grub-btrfsd --help
```
para obter a versão atualmente em procedimento do grub-btrfs.

#### Modo detalhado do serviço
Se você tiver falhas com o serviço, você pode fazer o procedimento
com o sinalizador `--verbose`. Para fazer isso, você pode fazer o
procedimento de:
``` bash
sudo /usr/bin/grub-btrfsd --verbose --timeshift-auto` (for timeshift)
# ou
sudo /usr/bin/grub-btrfsd /.snapshots --verbose` (for snapper)
```
Ou passe `--verbose` para o serviço usando a página Systemd
.service ou a página OpenRC conf.d respectivamente. (veja as
instruções de instalação do serviço sobre como fazer isso).
