---
layout: post
title: "Deno - Um novo interpretador Javascript"
description: "Nesse artigo vamos conhecer o novo interpretador javascript (Deno) que vem chamando atenção de várias pessoas da comunidade Javascript"
date: 2020-05-20 12:00:00
categories: [javascript]
tags: [deno, javascript]
comments: true
image: https://deno.land/v1_wide.jpg
author: lets00
featured: true
hidden: true
toc: true
---

No dia 13 de maio de 2020, foi lançada a versão 1.0 do Deno, um executor de javascript e typescript fora do navegador.  O nome da linguagem é um a inversão das sílabas da palavra node (no-de, de-no). E você deve está se perguntando: “porque criar mais uma plataforma para executar javascript se já existe o Node?”. Nesse artigo irei apresentar quais as principais diferenças entre Node e o Deno e quando é interessante utilizar o Deno no lugar do Node.


## Um pouco de história

Primeiramente, não devemos achar que o Deno veio para substituir o Node. Embora possuam funções semelhantes (runtime de javascript fora dos navegadores), ambos possuem características distintas. O próprio autor do Deno,  Ryan Dahl (que inclusive é o mesmo criador do Node), explica que não se deve pensar que tudo deve ser migrado para Deno e deve-se abandonar o Node.

O Node é uma plataforma robusta, com milhares de pessoas, bibliotecas e sistemas que utilizam-no para diversos fins. É uma plataforma que ganha vários adeptos pela sua facilidade de utilização e principalmente pela sua praticidade, sendo utilizado por diversos cases de sucessos pela internet. Mas, devemos lembrar que o Node foi desenvolvido em 2009, quando o Javascript era uma linguagem bastante diferente dos dias atuais. O Node criou diferentes conceitos para permitir a execução de código Javascript fora dos navegadores, e com isso criou também uma série de problemas de design que não são fáceis de resolver.

Quando o Node foi criado, não era utilizado Promises, ou async/await para as chamadas assíncronas do sistema. Então muitas bibliotecas padrões passaram a adotar a utilização de callbacks, que com o passar do tempo mostrou-se ineficiente para o desenvolvimento de grandes projetos devido a confusão causada pelas inúmeras chamadas de callbacks (vide o problema de callback hell que são amenizados pelo uso de async/await). Mudar as bibliotecas para utilizarem esses novos padrões utilizados pelo Javascript podem quebrar diversas aplicações que estão sendo utilizadas na internet. Esse foi somente um exemplo de funcionalidade que torna difícil e lento a evolução do Node.
Outros problemas são quanto a questão de permissões de acessos dos módulos, não padronização do uso de módulos entre os navegadores e o Node, dentre outras coisas. O Deno é uma tentativa de resolver esses problemas em um novo runtime.

## Principais características

O Deno é um **executor (runtime) de Javascript e Typescript seguro** fora dos browsers, utilizando o motor javascript do Chrome (o **V8**) e construído em **Rust**.
Por padrão implementa uma série de **medidas de segurança** que permite os programas/módulos acessarem apenas o que foi implicitamente liberado pelo usuário (arquivos, diretórios, rede, variáveis de ambiente, etc), o que lembra a política de proteção dos firewalls (proíbe tudo, e vai liberando os recursos de acordo com a necessidade). Isso evita problemas de módulos acessem indevidamente arquivos que por teoria não deveriam acessar (chaves SSH privadas, arquivos de senha do sistema, etc).

O Deno possui **suporte nativo à Typescript**: um superset do Javascript que trás muita produtividade principalmente em grandes projetos. O uso de Typescript vem crescendo bastante na comunidade de Javascript pelas facilidades que ela trás no desenvolvimento de software. Cada vez mais, diversas bibliotecas estão dando suporte ao Typescript.

A **inexistência de um gerenciador de pacotes** é uma das principais diferenças entre os dois runtimes. Os módulos são obtidos na declaração de importação, ou seja, quando o programador declara que vai utilizar o módulo, o Deno já identifica e faz o download do módulo necessário, colocando-o em um diretório de cache que será utilizado pela aplicação. Isso pode parecer estranho inicialmente, mas diversas outras linguagens utilizam esse mesmo conceito para obter os pacotes que serão necessário (a exemplo: Golang) . Por enquanto, oficialmente, não existe uma maneira de integrar o NPM ao Deno.

A instalação é um **arquivo único** e simples, autocontido, que já trás tudo o que é necessário para executar as aplicações que serão desenvolvidas. Isso facilita bastante o processo de instalação em diversas plataforma.

Um **inspetor de dependências** (_deno info_) e **formatador de código** (_deno fmt_), que permite todo código produzido em Deno ter um estilo de código padrão.

## Instalação

Como o Deno é autocontido, tudo que precisamos fazer é baixar o executável e configurar as variáveis de ambien

No **Linux (ou Mac)**, execute o comando abaixo:

```sh
$ curl -fsSL https://deno.land/x/install/install.sh | sh
```

```
######################################################################## 100,0%##O=#                                         ######################################################################## 100,0%
Archive:  /home/ubuntu/.deno/bin/deno.zip
  inflating: deno                    
Deno was installed successfully to /home/ubuntu/.deno/bin/deno
Manually add the directory to your $HOME/.bash_profile (or similar)
  export DENO_INSTALL="/home/ubuntu/.deno"
  export PATH="$DENO_INSTALL/bin:$PATH"
Run '/home/ubuntu/.deno/bin/deno --help' to get started
```
A saída do comando acima mostra que o binário foi baixado e extraído no diretório `~/.deno/bin/deno`.
Para torná-lo um comando que possa ser localizado pelo shell, adicione o binário do Deno na variável _PATH_ do arquivo de profile do seu interpretador de comandos. No exemplo, utilizaremos o `bash`. No final do arquivo `~/.bashrc` adicione as seguintes linhas:

Arquivo ~/.bashrc 
```sh
export DENO_INSTALL="/home/ubuntu/.deno"
export PATH="$DENO_INSTALL/bin:$PATH"
```

Pronto, abra um novo terminal para que a variável _PATH_ seja atualizada. A partir de agora o terminal passa a enxergar os binários do Deno diretamente (sem a necessidade de apontar o diretório do binário). Vamos testar executando o seguinte comando:

```sh
$ deno --help
deno 1.0.1
A secure JavaScript and TypeScript runtime

Docs: https://deno.land/manual
Modules: https://deno.land/std/ https://deno.land/x/
Bugs: https://github.com/denoland/deno/issues
...
```

Se a saída do comando foi semelhante a de cima, tudo foi configurado corretamente e já podemos utilizar o runtime do Deno para executar nossos códigos Javascript/Typescript.
Em um próximo post mostrarei um _"Hello World"_ e o sistema de segurança do Deno em ação.

## Referências

- Página oficial do Deno: [https://deno.land/](https://deno.land/)
- Blog do Deno: [https://deno.land/v1](https://deno.land/v1)