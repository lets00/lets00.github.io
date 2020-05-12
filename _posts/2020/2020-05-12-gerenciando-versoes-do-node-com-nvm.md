---
layout: post
title:  "Gerenciando versões do Node com NVM"
date:   2020-05-12 12:00:00
categories: [node, javascript]
tags: [node, javascript]
comments: true
---

![nvm](https://miro.medium.com/max/1050/0*csTuUtvi1VdLS4le.jpg "NVM logo")

O **NVM (Node Version Manager)** é um script que permite o usuário instalar um gerenciador de versões do Node, permitindo o usuário trabalhar com múltiplas versões do Node. Isso é bastante útil quando uma aplicação precisa suportar múltiplas versões do Node ou quando queremos desacoplar o Node dos diretórios padrões do sistema (já que o NVM cria uma nova estrutura de diretórios onde as versões são instaladas e atualizadas).

Como o NVM é um script, está diretamente atrelado a qual interpretador de comando estamos utilizando em nossa máquina. Os interpretadores suportados são: `sh`, `dash`, `zsh`, `ksh`, e o `bash`.

O NVM é projetado pra ser **isolado por usuário e por terminal**, ou seja, o comandos de gerenciamento de versões do Node apenas é configurado no usuário que instalou a ferramenta, e, em cada terminal aberto pode-se ter executando uma versão diferente do Node. Isso é bastante útil quando precisa-se testar uma aplicação em diversas versões ao mesmo tempo, sem a necessidade de fechar a aplicação que está em execução para modificar a versão do Node e depois reexecutar.

O gerenciamento realizado pelo NVM permite:
- Baixar novas versões do Node
- Instalar/apagar versões antigas do Node
- Isolar um biblioteca em uma versão específica do Node, permitindo que diferentes versões do Node executem diferentes bibliotecas

## Instalação

Vamos instalar o NVM dentro da pasta do usuário que está logado no sistema atualmente. Para isso, acesse o diretório `/home/<usuário>/` com o comando abaixo

```sh
$ cd ~
```
Se você utiliza o interpretador **bash**, baixe e execute o script `install.sh` através do comando abaixo:

```sh
$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash 
```

O script que foi executado clona o repositório oficial do NVM na pasta `~/.nvm` e adiciona a referência da pasta na no arquivo de profile do interpretador do sistema (`~/.bashrc`).

Feito isso, é necessário reiniciar o interpretador para que ele carregue as informações contidas no profile e, para isso, basta abrirmos um novo interpretador:

```sh
$ bash
``` 

Nesse novo interpretador, foi adicionado o caminho para os executáveis do NVM. Podemos testar se tudo está funcionando, executando o comando abaixo:

```sh
$ nvm ls
iojs -> N/A (default)
node -> stable (-> N/A) (default)
unstable -> N/A (default)
```

Se o comando apresentar alguma saída diferente de “comando não encontrado”, então temos o ambiente pronto pra começarmos a instalar as versões do Node. Na saída acima vemos que não existe nenhuma versão do Node instalada em nossa máquina (N/A).

## Instalando o Node via NVM

Primeiramente, vamos **listar todas as versões LTS** (_Long Time Support_) do Node que podemos instalar em nossa máquina com o comando abaixo:

```sh
$ nvm ls-remote –lts
  v4.2.0   (LTS: Argon)
  v4.2.1   (LTS: Argon)
  …
  v12.16.1   (LTS: Erbium)
  v12.16.2   (LTS: Erbium)
  v12.16.3   (Latest LTS: Erbium)
```

A lista é longa e todas essas versões podem ser instaladas e gerenciadas pelo NVM. Vamos **instalar** as duas últimas LTS versões (nesse exemplo será a versão 12.16.3 e a 10.20.1)

```sh
$ nvm install 12.16.3
$ nvm install 10.20.1
```

O NVM vai acessar o site do Node, baixar e descomprimir o arquivo no diretório que é gerenciado pelo NVM.

Agora podemos listar quais as **versões que estão instaladas** na nossa máquina e qual está sendo utilizada no momento.

```sh
$ nvm ls
->     v10.20.1
       v12.16.3
default -> lts/* (-> v12.16.3)
node -> stable (-> v12.16.3) (default)
stable -> 12.16 (-> v12.16.3) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/erbium (-> v12.16.3)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.20.1
lts/erbium -> v12.16.3
```

Podemos ver agora que temos 2 versões do Node instaladas e através da → podemos saber qual versão está em uso no momento (a última instalada). Outra maneira de verificar qual **versão do Node** estamos utilizando é através do comando:

```sh
$ node -v
v10.20.1
```

Além do Node, o NVM também **instala o NPM** (Node Package Manager), que é responsável de gerenciar as bibliotecas que serão utilizadas pelas aplicações Node. Podemos verificar a versão do NPM instalada através do comando

```sh
$ npm -v
6.14.4
```

## Trocando entre versões do Node

Para trocar da versão 10 para a 12, executamos o comando abaixo:
```sh
$ nvm use 12
Now using node v12.16.3 (npm v6.14.4)
```

Observe que não foi necessário passar a versão completa (versão descritiva). O NVM é capaz de identificar qual a versão que queremos utilizar (já que temos apenas um Node de versão 12 instalado).

Talvez você esteja se perguntando onde fica armazenado o NPM e o Node que foram baixados pelo NVM. Todas as versões são armazenadas no diretório `~/.nvm/versions/node`.


## Desinstalando uma versão do Node

Para **desinstalar uma versão** do Node, execute o comando abaixo:

```sh
$ nvm uninstall 10
Uninstalled node v10.20.1
```
O NVM não permite você desisntalar uma versão que esteja em uso. Por isso, antes de desinstalar procure ver qual a versão que está sendo utilizada no momento.

## Instalando bibliotecas em uma versão do Node

O NPM é instalado juntamente do Node e é carregado quando você utiliza uma versão via NVM. Todos os **pacotes** são instalado no **diretório** `.nvm/versions/node/<versão do node>/lib/node_modules` e isso nos trás o **isolamento de bibliotecas** dependendo da versão do Node que esteja em execução. Uma vantagem é que esse diretório possui permissão de leitura e escrita do usuário, então **não** é necessário utilizar o comando `sudo` quando precisamos instalar um  módulo globalmente. Por exemplo:

```sh
$ npm i @hapi/hapi -g
+ @hapi/hapi@19.1.1
added 33 packages from 3 contributors in 5.562s
```

## Conclusão

O uso do NVM mostrado aqui é o básico que o desenvolvedor precisa. Se você ficou curioso em quais outras opções o NVM pode oferecer, execute o comando sem nenhum parâmetro e veja a saída com todas as opções possíveis.

```sh
$ nvm
…
```

Dúvidas, comentários ou sugestão, por favor, escreva abaixo.
