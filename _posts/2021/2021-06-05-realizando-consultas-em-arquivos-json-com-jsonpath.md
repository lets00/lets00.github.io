---
layout: post
title: "Realizando consultas em arquivos JSON usando JSONPath"
description: "Nesse artigo vamos conhecer um pouco mais sobre a linguagem de consulta JSONPath"
date: 2021-06-05 12:00:00
categories: [json]
tags: [json]
comments: true
image: https://cdn.cinepop.com.br/2019/08/friday-the-13th-jason-voorhees-696x391.jpg
author: lets00
featured: true
hidden: false
toc: true
---

Muitas linguagens de programação já trazem incluídas em suas bibliotecas integradas alguma ferramenta de manipulação de arquivos JSON(_Javascript Object Notation_). Hoje em dia, o JSON é um dos formatos para troca de informações entre aplicações mais comum e utilizado, devido ao tamanho reduzido e o tempo de para interpretar um arquivo em uma estrutura de dados (parse) em comparação a outros formatos como XML.


## Introdução

A consulta e manipulação das informações contidas em um arquivo JSON são realizadas pela estrutura de dados que foi gerada após o arquivo ser interpretado. Algumas linguagens representam um JSON como um dicionário, o que permite acessar as informações da mesma maneira que se acessa qualquer outro dicionário nessa linguagem. Outras linguagens o representa como um objeto, contendo alguns métodos especiais para lidar com o acesso aos dados do mesmo. E isso torna a manipulação de um arquivo JSON diferente dependendo como a linguagem trate esse tipo de arquivo.

O JSONPath vem como uma tentativa de padronizar acesso aos dados contidos em um arquivo JSON, funcionando como uma **linguagem de consulta** (similar ao XPath para o XML) e removendo a dependência de como uma linguagem lida com ele. Ele é usado desde uma consulta simples (acesso direto a um valor de um nó) a uma consulta complexa (consultas a vários nós, com estruturas condicionais).


## Estrutura de uma consulta

Toda consulta começa pelo símbolo **$** (root member object), que representa a **raiz de um arquivo JSON**. O acesso aos nós pode ser realizado de duas formas: da notação de colchetes(1) ou através da notação de ponto (2).

Ex: Acessar o título do primeiro livro:
```js
$.store.book[0].title (1)
$.['store']['book'][0]['title'] (2)
```

A consulta acima é similar ao que naturalmente fazemos quando acessamos um dicionário ou objeto em Python ou Javascript, e, neste caso, seria melhor utilizar a própria estrutura de acesso fornecidas pela linguagem do que utilizar JSONPath. 

Mas em casos mais complexos, provavelmente teríamos que utilizar estruturas de repetição, condicionais ou até mesmo bibliotecas ou estratégias nativas oferecidas pela linguagem (como por exemplo, acessar todos os nós de um  nó raiz e verificar internamente em cada um deles se existe um determinado campo ou não). Abaixo encontramos uma tabela com todos os **símbolos** utilizados:

| Símbolo | Descrição |
| :-----: | :-------: |
| $ | Elemento Raiz |
| @ | Elemento atual |
| . ou [] | Operador de acesso a nó filho |
| .. | Recursivo descendente |
| * | Todos os nós |
| [] | Operador de Array |
| [x,y] | Seleção de elementos de um Array |
| [0:1:2] |Array slice |
| ?() | Expressão boleana|
| () | Script expression |

## Exemplos de uso

Para aprender como realizar uma consulta com JSONPath, consideremos o JSON abaixo:

```json
{ "store": {
    "book": [ 
        { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      },
        { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      },
        { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      },
        { "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99
      }
    ],
    "bicycle": {
      "color": "red",
      "price": 19.95
    }
  }
}
```

Para selecionar o **título do primeiro livro**, pode-se utilizar as seguintes consultas:

```js
$.store.book[0].title (notação de ponto)
ou
$.[‘store’][‘book’][0][‘title’] (notação de colchetes)
```
Ambas as expressões retornarão:
```js
[ "Sayings of the Century" ]
```

Toda consulta sempre retornará um array com 0 ou n elementos, caso a consulta esteja correta ou erro caso a consulta esteja incorreta. Os próximos exemplos utilizarão a notação de ponto para não misturar ambas as notações.

Para obter **todos os autores**:

```js
$.store.book[*].author
ou
$..author
```

O retorno da expressão:
```js
[
  "Nigel Rees",
  "Evelyn Waugh",
  "Herman Melville",
  "J. R. R. Tolkien"
]
```

A segunda expressão procurará dentro de todos os nós recursivamente, pelo campo author.

Para obter o **livro de número 4**:

```js 
$.store.book[3]
ou
$..book[3]
```

```js
[  
   {  
      "category":"fiction",
      "author":"J. R. R. Tolkien",
      "title":"The Lord of the Rings",
      "isbn":"0-395-19395-8",
      "price":22.99
   }
]
```

Para obter **todos os preços**:

```js
$.store..price
ou
$..price
```

Para obter **todos os preços dos livros**:
```js
$.store.book..price
ou
$.store.book[*].price
```

Obter o **último livro**:

```js
$..book[-1]
ou
$..book[@.length – 1]
```
O @ referencia a expressão anterior, ou seja, ele é o mesmo que ```$..book```.

Para obter **os dois primeiros livros**:

```js
$..book[1,2]
ou
$..book[:2]
```

Para obter apenas os livros que **possuem um ISBN definido**:
```js
$..book[?(@.isbn)]
```

Para obter os livros que **não possuem um ISBN definido**:
```js
$..book[?(! @.isbn)]
```

Para obter os **livros cujo preço é maior que 9**:
```js
$..book[?(! @.price > 9)]
```

## Testando as consultas

Para testar se suas consultas estão corretas, você pode utilizar o site [JSONPath](https://jsonpath.com/) que permite definir um JSON, a consulta desejada e verificar a saída referente a consulta.

A maioria das linguagens implementam alguma biblioteca para interpretar consultas JSONPath. Abaixo é possível ver em algumas linguagens o nome dos pacotes e como instalá-los:

**Javascript/Node**
```sh
$ npm install JSONPath 
```

**Python**
```sh
$ pip install jsonpath-rw 
```

**Java (maven)**
```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.0.0</version>
</dependency>
```

## Referências
- https://restfulapi.net/json-jsonpath/
- https://jsonpath.com/