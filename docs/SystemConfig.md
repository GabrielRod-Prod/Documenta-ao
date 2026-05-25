---
title: Instalaçao e configuraçao Base do sistema - Arch
author: J.Gabriel Rod. Santos
icon: lucide/rocket
description: Aprenda a criar, modificar e gerenciar usuários e grupos no Linux – comandos essenciais, boas práticas e dicas de segurança.
social:
  cards_layout_options:
    background_color: "#1fc26e"        # Cor de fundo do card
    background_image: null             # Sem imagem de fundo
    text_color: "#ffffff"              # Cor do texto
    author_color: "#f5a623"            # Cor do nome do autor
---

# System Config

Manual de configuraçao do sistema.

## Index

- [System Config](#system-config)
  - [Index](#index)
  - [Instalaçao {#instalation}](#instalaçao-instalation)
    - [Reflector {#reflector}](#reflector-reflector)
    - [YAY](#yay)
    - [Cockpit {#cockpit}](#cockpit-cockpit)

## Instalaçao {#instalation}

1. Atualizar o sistema:

```sh
sudo pacman -Syu
```

1. Instalando ferramentas essenciais

```sh
sudo pacman -S --needed git base-devel linux linux-firmware NetworkManager firewalld wget tree plocate
```

### Reflector {#reflector}

```sh
sudo pacman -S reflector rsync curl
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.$(date +%d%m%Y).bak  # Cria um backup do mirrorlist atual com data e extensao .bak
sudo reflector --verbose -l NÚMERO --sort rate --save /etc/pacman.d/mirrorlist # Comando geral
sudo reflector --verbose --country 'Brazil' --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist  # Otimizado p/Brazil
```

- Atualizaçao periodica de espelhos

```sh
sudo systemctl enable reflector.timer
sudo systemctl start reflector.timer
```

>A configuração padrão do timer pode ser visualizada/editada em ```/etc/systemd/system/reflector.timer```. Para ajustar os parâmetros do ```reflector```, edite o arquivo ```/etc/xdg/reflector/reflector.conf```.
>
### YAY

```sh
mkdir -p ~/Downloads/AUR
cd ~/Downloads/AUR
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
yay --version
```

### Cockpit {#cockpit}

```sh
sudo pacman -S cockpit 
yay -S cockpit-machines cockpit-podman cockpit-storaged cockpit-packagekit cockpit-files cockpit-pcp cockpit-pacman-git cockpit-files
```

- Ativaçao

```sh
sudo systemctl enable --now cockpit.socket
systemctl status cockpit.socket
```

- Acesse em **<https://localhost:9090>**

- Limitar acesso apenas à máquina local (localhost):
Crie um arquivo de configuração do socket para restringir a escuta apenas ao endereço ```127.0.0.1```

```sh
sudo mkdir -p /etc/systemd/system/cockpit.socket.d/
sudo nano /etc/systemd/system/cockpit.socket.d/listen.conf
```

- Adicione o seguinte conteúdo:

```conf
[Socket]
ListenStream=
ListenStream=127.0.0.1:9090
FreeBind=yes
```

Reiniciar Systemd e Socket

```sh
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

- Instalar e ativar plugins, ferramentas e dep.

```sh
sudo pacman -S libvirt podman
sudo systemctl enable --now libvirtd.service podman 
podman-auto-update.service podman.socket podman-kube@.service
```
