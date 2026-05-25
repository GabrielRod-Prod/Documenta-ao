---
title: Gerenciamento de Usuários e Grupos no Linux
author: J.Gabriel Rod. Santos
icon: lucide/user
description: Aprenda a criar, modificar e gerenciar usuários e grupos no Linux – comandos essenciais, boas práticas e dicas de segurança.
social:
  cards_layout_options:
    background_color: "#1fc26e"        # Cor de fundo do card
    background_image: null             # Sem imagem de fundo
    text_color: "#ffffff"              # Cor do texto
    author_color: "#f5a623"            # Cor do nome do autor
---

# Gerenciamento de Usuarios, Grupos, Acesso(Arquivos) e Quotas

## Resumo

1. [Usuarios](#users)
      1. [Contas](#accounts)
      2. [Arquivos(Permissoes)](#archine_acess)
      3. [Limite de Recursos(Quotas)](#quotas)
           1. [Configuraçao inicial](#init_config_quotas)
2. [Grupos](#groups)
3. [Boas Praticas](#bonus)

___

### Usuarios {#users}

* Criar usuarios

```sh
useradd nome_user # Cria usuario com opçoes padrao
adduser nome_user # Versao interativa (Deb/Ubuntu)
```

* Definir ou alterar senha

```sh
passwd nome_user # Define ou altera a senha
```

* Listar Usuarios

```sh
cat /etc/passwd  # Lista todos os usuarios
groups nome_user # Mostra grupos do usuario
id nome_user     # Mostra UID, GID e grupos
```

* Remover usuario

```sh
userdel nome_user    # Remove usuario, mas mantem seu diretorio home
userdel -r nome_user # Remove usuario + home + mail spool 
```

* Modificar atributos de usuario(```usermod```)

```sh
usermod -c "Nome Completo" user # Comentario (GECOS)
usermod -d /novo/home user      # Altera diretorio home
usermod -l novo_login user      # Muda o nome de login
usermod -s /bin/zsh user        # Altera shell de login
usermod -u 2000 user            # Muda UID
usermod -g group_primary user   # Altera grupo primario
usermod -G group1,group2 user   # Define grupos secundarios ( sobrescreve )
```

> Logs de autenticaçao:
>
> ```sh
> last    # Ultimos logins
> lastlog # Ultimo login de cada usuario
> who     # Usuarios logados no momento
> w       # Mais detalhes (atividade)
> ```
>
#### Contas {#accounts}

* Controle de expiraçao de senha e conta

```sh
chage -l user            # Mostra politica de senhas
chage -M 90 user         # Senha expira em 90 dias
chage -E 2026-12-31 user # Conta expira em data especifica
passwd -e user           # Força troca de senha no proximo login
```

* Bloquear/Desbloquear conta

```sh
passwd -l user  # Lock
passwd -u user  # Unlock
usermod -L user # Lock
usermod -U user # Unlock
```

#### Controle de acesso a arquivos - ACLs(alem de permissoes UNIX) {#archine_acess}

```sh
setfacl -m u:user:rwx arquivo   # Concede permissao especifica a um usuario
setfacl -m g:group:rx arquivo   # Concede permissao especifica a um grupo
setfacl -x u:user arquivo       # Remove ACL de usuario
getfacl arquivo                 # Visualiza ACLs
setfacl -b arquivo              # Remove todas as ACLs
```

> Boas práticas: Use ACLs apenas quando as permissões tradicionais (ugo+rwx) forem insuficientes. Para diretórios compartilhados, lembre-se da herança com d:.
>
#### Limites de Recursos(quotas) por user/group {#quotas}

##### Configuraçao Inicial {#init_config_quotas}

1. Editar ```/etc/fstab``` adicionando ```usrquota,grpquota``` nas opçoes da partiçao.
2. Remontar a partiçao: ```mount -o remount /home```
3. Criar arquivos de quota: ```quotacheck -avugm```
4. Ativar quotas: ```quotaon -avug```

* Comandos Principais

```sh
quotacheck -avugm                     # Verifica e cria/atualiza aquota.user/group
edquota -u usuario                    # Edita limites (soft/hard) para um usuário
edquota -g grupo                      # Edita limites para um grupo
quota -u usuario                      # Exibe uso atual e limites
repquota -a                           # Relatório completo de todas as partições
quotaon -avug                         # Ativa quotas
quotaoff -avug                        # Desativa quotas
```

### Grupos {#groups}

* Criar grupos

```sh
groupadd nome_group 
```

* Listar grupos

```sh
cat /etc/group # Lista todos os grupos
```

* Adicionar usuario a um grupo secundario.

```sh
usermod -aG group user # -a e importante para nao remover outros grupos
gpasswd -a user group  # Alternativa mais simples
```

* Modificar grupo (groupmod)

```sh
groupmod -n novo_nome group_antigo  # Renomei grupo
groupmod -g 3000 group              # Altera GID
```

* Gerenciamento de grupos com gpasswd(ADMs de grupo)

```sh
gpasswd -A user group        # Define um ADM para o grupo
gpasswd -M user1,user2 group # Define lista de membros(sobrescreve)
gpasswd group                # Define senha para o grupo (raro)
```

* Remover grupo

```sh
groupdel nome_group
```

* Remover usuario de um grupo

```sh
gpasswd -d user group
```

### Boas práticas {#bonus}
> * **UIDs reservados: 0 (root), 1–999 (usuários de sistema), ≥1000 (usuários comuns).**
> * **Após modificar grupos de um usuário, ele precisa reconectar para que as novas associações tenham efeito.**
> * **Sempre use -aG no usermod ao adicionar grupos secundários.**
> * **Para scripts de criação em lote, combine useradd -m com chpasswd.**
> * **Monitore logs de autenticação regularmente (/var/log/auth.log ou /var/log/secure).**
> * **Prefira adduser e deluser em distribuições Debian/Ubuntu para maior interatividade.**
