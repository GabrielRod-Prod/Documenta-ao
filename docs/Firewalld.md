---
title: Gerenciamento Firewalld
author: J.Gabriel Rod. Santos
icon: lucide/wall
description: Manual de instalaçao e gerenciamento firewalld(ArchLinux)
social:
  cards_layout_options:
    background_color: "#1fc26e"        # Cor de fundo do card
    background_image: null             # Sem imagem de fundo
    text_color: "#ffffff"              # Cor do texto
    author_color: "#c57f0f"            # Cor do nome do autor
---

# Firewalld

Manual de instalaçao e gerenciamento.

## Index

- [Firewalld](#firewalld)
  - [Index](#index)
  - [Instalaçao {#instalation}](#instalaçao-instalation)
  - [Conceitos Fundamentais {#fundamentos}](#conceitos-fundamentais-fundamentos)
  - [Gerenciamento Basico (firewall-cmd) {#managementefirewalld}](#gerenciamento-basico-firewall-cmd-managementefirewalld)
  - [Gerenciamento Avançado {#managementhigh}](#gerenciamento-avançado-managementhigh)
    - [Gerenciamento de zonas com Network Manager {#ger-net-manager}](#gerenciamento-de-zonas-com-network-manager-ger-net-manager)

## Instalaçao {#instalation}

```sh
sudo pacman -S firewalld python-firewall firewall-config firewall-applet firewalld-test bash-completion networkmanager polkit                     # Instalar firewalld a partir de repo. oficial arch linux
sudo systemctl enable --now firewalld.service # Ativa o serviço
```

___

## Conceitos Fundamentais {#fundamentos}

Para gerenciar o firewalld, é essencial entender alguns conceitos principais.

1. Zonas: Zonas são conjuntos de regras que definem o nível de confiança de uma conexão de rede. Algumas das zonas pré-definidas e seus propósitos incluem:
      1. ```block```: Rejeita todo o tráfego de entrada, com uma mensagem ```icmp-host-prohibited```.
      2. ```public```: Para uso em áreas públicas onde você não confia nos outros dispositivos da rede (é a zona padrão)
      3. ```.home```, ```work```, ```internal```: Zonas com níveis de confiança mais altos para redes domésticas, de trabalho ou internas.
      4. ```trusted```: Permite todo o tráfego de rede.
2. Configuração Runtime vs. Permanente: O firewalld separa as configurações que estão ativas no momento (runtime) das que serão preservadas entre reinicializações (permanent).
      1. ```Runtime```: Alterações são aplicadas instantaneamente, mas são perdidas após um reboot ou recarga do serviço ```firewalld```.
      2. ```Permanent```: Configurações salvas que persistem após reinicializações. Para que uma configuração permanente entre em vigor, é necessário recarregar o firewall (veja abaixo).
      3. Por padrão, a maioria dos comandos sem a opção ```--permanent``` modifica apenas a configuração em runtime.

___

## Gerenciamento Basico (firewall-cmd) {#managementefirewalld}

A principal ferramenta para interagir com o firewalld é o utilitário de linha de comando firewall-cmd.

- Comandos basicos de configuraçao

```sh
--get-active-zones                                     #Mostra as zonas atualmente ativas e as interfaces atribuidas a elas
--zone=<zone> --change-interface<interface>            #Atribui uma interface de rede a uma zona especifica. Exemplo:
sudo firewall-cmd --zone=home --change-interface=eth0 # Move a interface eth0 para a zona home
```

- Adicionando Seviços

Serviços são regras pré-definidas que liberam portas específicas (e.g., http para porta 80).

```sh
sudo firewall-cmd --add-service=http     # Adiciona o serviço HTTP na zona padrao (public) em tempo de execuçao
sudo firewall-cmd --runtime-to-permanent # Torna a mudança permanente
```

- Adicionando portas

```sh
sudo firewall-cmd --add-port=8080/tcp
sudo firewall-cmd --runtime-ti-permanent
```

- Para mudanças permanentes (inline)

```sh
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

___

## Gerenciamento Avançado {#managementhigh}

1. Regras Avançadas (Rich Rules): As "rich rules" fornecem uma linguagem mais expressiva e poderosa para criar regras complexas que vão além dos elementos básicos como serviços ou portas.
      1. **Estrutura**: As regras podem incluir elementos como endereços de origem (source), destino (destination), ações como accept, reject ou drop, além de logging.
      2. **Exemplo**: Para permitir acesso SSH (porta 22) apenas de um IP específico (e.g., 192.168.1.10) na zona public:

```sh
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.10" service name="ssh" accept'
sudo firewall-cmd --reload
```

1. **Redirecionamento de Portas (Port Forwarding)**: Permite encaminhar o tráfego de uma porta para outra porta ou até para um endereço IP diferente na rede
      1. P**ré-requisito**: O redirecionamento geralmente requer que o NAT (masquerade) esteja ativo na zona.
      2. **Exemplo**: Redirecionar o tráfego da porta 12345/tcp na interface pública para a porta 22 (SSH) do host interno 10.20.30.40:

```sh
sudo firewall-cmd --zone=public --add-masquerade --permanent
sudo firewall-cmd --zone=public --add-forward-port=port=12345:proto=tcp:toport=22:toaddr=10.20.30.40 --permanent
sudo firewall-cmd --reload
```

> Note que para remover este redirecionamento, é necessário informar a declaração completa como no comando ```--add-forward-port```

1. Criando/Editando Serviços Personalizados:
      1. Os serviços pré-definidos ficam em ```/usr/lib/firewalld/services/```. Para criar um serviço personalizado, copie um dos arquivos para ```/etc/firewalld/services/``` e o edite.
      2. **Exemplo**: Para criar um serviço para a porta 3000/TCP:

```sh
sudo cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/meuapp.xml
sudo nano /etc/firewalld/services/meuapp.xml
```

- Edite o arquivo, alterando o nome do serviço, descriçao e a tag ```<port>```:

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Minha Aplicação</short>
  <description>Serviço para minha aplicação na porta 3000.</description>
  <port protocol="tcp" port="3000"/>
</service>
```

- Depois de criar o arquivo, recarregue o firewalld:

```sh
sudo firewall-cmd --reload
```

- Agora você pode usar ```--add-service=meuapp``` para liberar essa porta.

### Gerenciamento de zonas com Network Manager {#ger-net-manager}

O firewalld pode ser integrado ao NetworkManager para que a zona do firewall mude automaticamente com base na rede à qual você se conecta.

- Listar Perfis de Conexão: Use o ```nmcli``` para listar seus perfis de conexão

```sh
nmcli connection show
```

- Atribuir zona a um perfil

```sh
sudo nmcli connection modify MeuWifi connection.zone home
```

- Verificar atribuiçao

```sh
nmcli connection show MeuWifi | grep zone
```

* Quando você se conectar ao "MeuWiFi", o firewalld automaticamente aplicará as regras da zona ```home```.

Dicas:
> Não é possível executar ```firewall-cmd``` como usuário normal: Instalações mínimas do Arch podem não ter um agente polkit em execução. Tente executar o comando com sudo ou inicie um agente polkit (como ```polkit-gnome```).


> Regras não persistem após reinicialização: Certifique-se de que usou a flag ```--permanent``` ou executou ```--runtime-to-permanent``` após suas alterações.

> Firewall não inicia: Verifique se nenhum outro serviço de firewall (como ```ufw, iptables ou nftables```) está em execução, pois eles podem conflitar com o firewalld.
