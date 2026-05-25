---
title: Gerenciamento de Usuários e Grupos no Linux
author: J.Gabriel Rod. Santos
icon: lucide/code
description: Aprenda a criar, modificar e gerenciar usuários e grupos no Linux – comandos essenciais, boas práticas e dicas de segurança.
social:
  cards_layout_options:
    background_color: "#1fc26e"        # Cor de fundo do card
    background_image: null             # Sem imagem de fundo
    text_color: "#ffffff"              # Cor do texto
    author_color: "#f5a623"            # Cor do nome do autor
---
# Disk Commands

## Resumo

* [Verificaçao De Disco(EXT4/EXT3/EXT2)](#ext4);
* [Verificaçao De Disco(FAT32)](#fat32);
* [SWAP Passo-a-Passo](#swap);

___

## Checagem de disco
>
> Desmonte a partiçao antes de usar!
>
> ```sh
> sudo umount /dev/sdXY # Exemplo: sudo umount /dev/sda1
> ```
>
___

### EXT4(ext3/ext2) Verificaçao de Disco {#ext4}

* Verificaçao simples (somente leitura)

```sh
sudo e2fsck -n /dev/sdXY
```
  
* Verificaçao e reparo auto. (responde "sim" para todas as perguntas)

```sh
sudo e2fsck -y /dev/sdXY
```

* Verificaçao e reparo iterativo (pergunta antes de cada açao)

```sh
sudo e2fsck /dev/sdXY
```

> Opçoes Uteis:
>
> * ```-p```: Reparo automatico, mas menos agressivo que o ```-y```. Corrige apenas erros considerados seguros.
> * ```-c```: Verifica e marca blocos defeituosos (bad blocks).Use ```-c -c``` para um teste nao-destrutivo mais completo.
> * ```-f```:Força a verificaçao completa, mesmo que o sistema de arquivos pareça estar limpo.

Casos Especificos:

* Recuperar Superbloco Corrompido: Se o e2fsck nao conseguir encontrar um superbloco valido, voce pode tentar usar um backup:

```sh
# Tentar usar um superbloco alternativo (ex: bloco 32768)
sudo e2fsck -b 32768 /dev/sdXY
```

Para encontrar a localizaçao de todos os superblocos de backup, use o comando ```sudo mke2fs -n /dev/sdXY```(o ```-n``` simula a operaçao sem escrever no disco).
___

### FAT32 Verificaçao de Disco {#fat32}

> FAT32 e comumente usado em pendrives e cartoes de memoria. O comando especifico e o ```fsck.vfat```.

* Verificaçao completa e reparo auto. (recomendado)

```sh
sudo fsck.vfat -a /dev/sdXY
```

* Verificaçao e reparo interativo

```sh
sudo fsck.vfat -r /dev/sdXY
```

* Modo "verbose" (saida detalhada do processo)

```sh
sudo fsck.vfat -v /dev/sdXY
```

> Opçoes Uteis
>
> * ```-a```: Reaparo auto. (sem interaçao com o user).
> * ```-r```: Reparo interativo (pergunta antes de corrigir cada erro).
> * ```-V```: Modo "verbose" (saida detalhada do processo).
> * ```-t```: Marca clusters nao utilizados como nao limpos.
>
>> Nota:O ```fsck.vfat``` tambem pode ser chamado pelos comandos ```fsck.fat``` ou ```dosfsck```, que sao sinonimos ou atalhos para a mesma ferramenta.
>
___

### SWAP Verificaçao de Disco {#swap}

```sh
# 1. Verificar se a swap esta ativa e qual dispositivo esta sendo usado
sudo swapon --show
# Saida esperada: mostra o caminho do dispositivo ( ex: /dev/sda2), seu tamanho e prioridade.

# 2. Ver o mesmo que o coman acima, acessando diretamente o sistema de arquivos /proc
cat /proc/swaps
# Saida esperada: exibe as mesmas informaçoes de uma forma mais "bruta".

# 3. Verificar o uso de memoria RAM e swap
free -h
# Saida esperada: mostra a qtdade total, usada e livre de memoria (RAM e a swap).

# 4. Para forçar a verificaçao do dispositivo de swap, voce pode desativa-lo e reativa-lo
sudo swapoff /dev/sdXY # Desativa a swap no dispositivo especificado
sudo swapon /dev/sdXY # Reativa a swap e testa se o dispositivo esta funcionando

5. Outros 
sudo swapoff -a # Desativa  todas as partiçoes e arquivos de swap ativados.
sudo swapon -a # Ativa  todas as partiçoes e arquivos de swap ativados.
```
