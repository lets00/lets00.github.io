---
layout: post
title:  "Byobu - um poderoso multiplexador de janelas"
date:   2020-05-11 15:04:23
categories: [linux]
tags: [byobu, ssh, tmux, server]
comments: true
image: https://i.ibb.co/0sW9sxH/Captura-de-tela-de-2020-05-11-15-25-36.png
author: lets00
toc: true
---

O **terminal de comandos** é um dos programas mais acessados pelos desenvolvedores, quanto pelos administradores para realizar suas atividades cotidianas. Talvez o cenário descrito a seguir seja familiar e cotidiano pra você: você começou a desenvolver uma tela de um sistema e precisou fazer alguma alteração no *backend* da aplicação. Você abre um novo terminal e começa a alterar o arquivo e relançar o servidor de desenvolvimento. Logo percebe que precisa atualizar o sistema operacional ou instalar uma nova biblioteca. Lá vai mais um terminal que é aberto para atualizar ou instalar. Enquanto o sistema está atualizando, precisa baixar os dados de um servidor *git*, ao mesmo tempo que acompanha todos os demais processos.

Para um administrador de sistemas, lidar com o terminal de comandos é uma atividade ainda mais comum. Diariamente, ele precisa **acessar remotamente** várias máquinas, **executar diversos comandos**, **acompanhar _logs_**, **monitorar atividades**, etc. E por mais que toda informação seja centralizada em algum lugar, um administrador ainda precisa lidar com uma série de terminais abertos.

O **Byobu** é uma ferramenta de código aberto que permite o usuário gerenciar todas as janelas de terminais que possui aberta. Ele permite o usuário realizar atividades como: criar novos terminais e navegar por eles, mudando seu foco sempre que for desejado; manter diversas conexões ssh abertas ao mesmo tempo, sem ser expulso da sessão quando fechar uma janela ou permanecer muito tempo ocioso.

Se você utiliza ferramentas como **Tmux** ou **GNU-Screen** para gerenciar múltiplos terminais, já deve está familiarizado como essas ferramentas podem trazer praticidade no dia a dia do usuário. E se esse for o caso, utilizar o *Byobu* será muito simples pois, por debaixo dos panos,um processo do *Tmux* está sendo executado. Poderíamos dizer que o *Byobu* é um tipo de *wrapper* do *tmux* com uma série de facilidades como: integração de mouse na manipulação de diversas janelas, esquemas de cores, etc.

## Instalação

O *Byobu* está presente na maioria dos repositórios de pacotes das principais distribuições Linux do mercado. Para instalá-lo, basta executar o seguinte comando:

- Em distribuições baseadas em Debian/Ubuntu:
```sh
$ sudo apt install byobu
```

- Em distribuições baseadas em Fedora/Centos:
```sh
$ sudo dnf install byobu
```

Para executar o *Byobu*, abra um único terminal e digite o comando:
```sh
$ byobu
``` 

![Byobu em execução](https://i.ibb.co/30Npchy/Captura-de-tela-de-2020-05-11-17-17-33.png "Byobu em execução" loading="lazy")
Figura 1- Byobu em execução

Pronto. A partir de agora todo terminal que precisarmos executar, será gerenciado nessa janela aberta pelo *Byobu*. Por padrão, o *Byobu* apresenta na parte inferior da janela algumas informações do sistema como:

- Sistema operacional
- Versão do sistema operacional
- Janelas
- _uptime_ do sistema
- número de núcleos do processador e velocidade
- data e hora


## Esqueleto de um gerenciador de janelas

A arquitetura básica de um multiplexador de janelas consiste em 3 partes básicas: **programa de terminal**, **janela** e o **terminal**.

O programa de terminal é a aplicação que executamos quando queremos abrir um novo terminal em sistemas que possuem uma Interface gráfica (GUI). Por exemplo, no Ubuntu podemos abrir o **gnome-terminal** ou outros como o **tilix**, **tilda**, **terminator**, etc. Esse programa contém uma **janela** e dentro da janela um **terminal**. Programas como o **gnome-terminal**, permitem o usuário criar múltiplas janelas e, em cada janela, um novo terminal é executado. Então pra resumir:

1 programa de terminal → 1 janela → 1 terminal

O *Byobu* é um multiplexador de janelas. Isso significa que ele é responsável por navegar em diversas janelas e, em cada janela pode existir 1 ou mais terminais, posicionados em diversos lugares da tela. A figura abaixo mostra uma janela aberta com 3 terminais estão em execução.

![Byobu gerenciando uma janela com 3 terminais](https://i.ibb.co/0sW9sxH/Captura-de-tela-de-2020-05-11-15-25-36.png "Byobu gerenciando uma janela com 3 terminais" loading="lazy")
Figura 2- Byobu gerenciando uma janela com 3 terminais

Os terminais são os elementos que nos permitem executar comandos. Podemos perceber em qual terminal estamos através das bordas alaranjadas.

## Criando/Apagando um terminal

Um terminal pode ser criado em duas posições: **horizontalmente** ou **verticalmente**.
Para criar um terminal **horizontal**, utiliza-se o atalho **_Shift + F2_**. A figura abaixo demonstra o novo terminal criado.

![Novo terminal na horizontal](https://i.ibb.co/PmkhFnm/byobu-horizontal.png "Novo terminal na horizontal" loading="lazy")
Figura 3- Novo terminal na horizontal

Para **mover o cursor** para o novo terminal, pressionamos a teclas **_Shift_** + as setas direcionais apontando para onde desejamos movimentar o cursor (↑↓← →). No exemplo acima, o novo terminal foi criado horizontalmente para baixo. Então, para colocar o cursor nesse terminal pressionaremos as teclas **_Shift_ + ↓** .

Para criar um terminal **vertical**, utiliza-se o atalho **_Ctrl + F2_**. A figura abaixo mostra a aparência do novo terminal criado. Para mover o cursor para o novo terminal criado a direita, utiliza-se o atalho **_Shift_ + →** .

![Novo terminal na vertical](https://i.ibb.co/5kr90vt/Captura-de-tela-de-2020-05-11-15-23-45.png "Novo terminal na vertical" loading="lazy")
Figura 4- Novo terminal na vertical

Em alguns casos, precisamos que o **foco de um terminal** específico ocupe 100% do tamanho da janela (_Fullscreen_). Através do atalho **_Shift + F11_**, o foco da janela passa a ser exclusivamente aquele terminal. Note que os demais terminais não foram fechados e podemos observá-los novamente através do atalho **_Shift + F11_**.

![Terminal com foco](https://i.ibb.co/By4Jpfg/Captura-de-tela-de-2020-05-11-15-29-04.png "Terminal em foco" loading="lazy")
Figura 5- Terminal com foco. Observe a letra Z na janela (parte inferior) indicando que existe Zoom aplicado.

Quando um terminal não é mais útil, podemos **fechar** o terminal selecionado através do atalho **_Ctrl + D_**.

## Criando/Apagando uma janela

Até o momento, todos os terminais estão sendo executados na mesma janela. Através de janelas nós podemos navegar entre vários contextos. Em cada janela podemos utilizar um conjunto de terminais específicos para um determinado fim e isso facilita a **organização das atividades** que estão sendo executadas. Para criar uma nova janela, pressione a tecla **_F2_**. A nova janela é criada a direita e podemos navegar entre elas utilizando o atalho **_Alt + (←→)_**.

![Nova janela](https://i.ibb.co/Yyp0wdL/Captura-de-tela-de-2020-05-11-17-17-21.png "Nova janela" loading="lazy")
Figura 6- Nova janela

Na parte inferior aparece a janela que estamos trabalhando no momento e podemos ver a **quantidade de janelas aberta**. Perceba que as janelas não possuem nomes definidos (apenas um número com a ordem que foram criadas). Para **renomear** uma janela, vá a qualquer terminal da janela e digite o seguinte comando:

```sh
$ tmux rename-window -t "${TMUX_PANE}" "Janela X"
```

E a janela atual terá o nome _Janela X_, tornando mais fácil identificar o que trata cada janela.

![Janela renomeada](https://i.ibb.co/61gXLtW/Captura-de-tela-de-2020-05-11-17-17-21.png "Janela renomeada" loading="lazy")
Figura 7- Janela Renomeada

Para **fechar uma janela**, precisamos fechar todos os terminais abertos através do atalho **_Ctrl + D_** ou **_Ctrl + F6_**. 

## Conclusão

Trabalhar com multiplexadores de janelas poupa bastante tempo, principalmente quando precisamos de vários terminais abertos em sessões remotas ou quando precisamos lidar com muitos terminais executando diferentes tarefas ao mesmo tempo.

Para mais informações, consulte a página manual da ferramenta:

```sh
$ man byobu
```
