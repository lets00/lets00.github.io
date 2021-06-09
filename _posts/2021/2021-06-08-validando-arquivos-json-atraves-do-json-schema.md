---
layout: post
title: "Validando arquivos JSON através do JSON Schema"
description: "O que é JSON Schema e como podemos utilizá-lo para validar nossos arquivos JSON?"
date: 2021-06-08 12:00:00
categories: [json]
tags: [json]
comments: true
image: https://videohive.img.customer.envatousercontent.com/files/28a8c552-427e-4904-a155-fe2df03ab856/inline_image_preview.jpg?auto=compress%2Cformat&fit=crop&crop=top&max-h=8000&max-w=590&s=22488591411d0f04d1d05bc862551b94
author: lets00
toc: true
---

O JSON Schema é uma especificação para o formato JSON que permite descrever e validar um arquivo JSON. Ele é bastante utilizado em ferramentas de teste ou lint para verificar se a requisição/resposta de um servidor é a esperada. Um JSON Schema pode definir o tipo de objeto esperado e como cada campo deve ser preenchido.

## Introdução

Suponhamos que temos uma REST API que retorna um produto após o usuário consultar a rota de produto com um determinado identificador (ID). Vamos utilizar como exemplo o seguinte arquivo JSON:

```json
{
  "productId": 1,
  "productName": "A green door",
  "price": 12.50,
  "tags": [ "home", "green" ]
}
```

O arquivo JSON acima descreve um produto e cada campo armazena um tipo de informação diferente. Antes de aprofundarmos, podemos realizar o seguinte questionamento sobre os campos acima:

1- O que é `productID`? Qual o valor que ele manipula?
1- O `productName` é obrigatório?
1- O `price` pode ser zero?
1- Todas as `tags` são strings?

Para responder estas questões e validar se a resposta/requisição de um servidor obedecerá sempre a regra que definimos, é necessário definir um arquivo de metadados sobre o comportamento de cada campo. O JSON Schema é um arquivo de metadados proposto pela IETF que define regras de campos e valores que um arquivo JSON deverá possuir.

## Estrutura básica de um JSON Schema

Todo arquivo de schema possui a seguinte estrutura padrão:

* $schema: Define qual o *draft* utilizado. O IETF lançou uma série de *drafts* com o passar dos anos, adicionando novas estruturas de verificação e validação. No exemplo abaixo, utilizaremos o último *draft* lançado (2020-12).

* $id: Define uma URI do schema, geralmente uma URL com o arquivo de schema.

* title: O título do schema.

* description: Uma breve descrição do que o schema valida.

* type: O tipo do cabeçalho do schema. Neste caso será um JSON Object (object)

O cabeçalho do arquivo JSON Schema será:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product in the catalog",
  "type": "object"
}
```

## Definindo as properties

Agora precisamos definir as propriedades (campos) que serão definidos no arquivo JSON. O primeiro campo, `productId`, identifica o produto. Sobre este campo, podemos então definir:

* É um campo **obrigatório**, pois não existe um produto sem um identificador,
* É um inteiro único

Sabendo disto, podemos então definir o seguinte JSON Schema para representá-lo:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    }
  },
  "required": [ "productId" ]
}
```
O objeto **properties** descreve todos os campos que um arquivo poderá possuir. Cada campo é um objeto e poderá ter uma **description** para explicar melhor o que é aquele campo. O valor **type** define qual o tipo que o campo possuirá. Os tipos básicos são:

* string
* number
* integer
* object
* array
* boolean
* null

Estes tipos são análogos na maioria das linguagens de programação, logo não é difícil associá-los no arquivo JSON. O valor **required** define um array com todos os campos que são obrigatórios.

Continuando o exemplo acima, vamos fazer o mesmo procedimento com o campo `productName`.
* Uma string que descreve o que é o produto;
* Obrigatório.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    }
  },
  "required": [ "productId", "productName" ]
}
```
O campo `price` possui as seguintes propriedades:
* Um número que permite casas decimais;
* Obrigatório;
* O valor deverá ser maior que 0, pois não existe um produto com preço gratuito neste nosso exemplo.

Com essas informações, podemos definir o seguinte JSON Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```
O tipo **number** permite representar valores de ponto flutuante. Para valores inteiros, o tipo mais apropriado é o **integer**, O valor **exclusiveMinimun** define para um campo que ele espera lidar apenas com valores maiores que zero, excluindo o zero. Se quisermos incluir o número zero, podemos utilizar o valor **minumum**.

O campo `tag` possui as seguintes informações:
* Um array;
* Não é obrigatório;
* Contém apenas strings;
* Se estiver presente, deve possuir pelo menos um item;
* Se estiver presente, não deve possuir itens repetidos.

O JSON Schema para o campo tag ficará:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

O tipo **array** define que o campo possuirá um array cujo os itens (definidos no objeto **items** serão **string**. A quantidade mínima de itens é definido pelo valor **minItems** e a propriedade de não possuir valores repetidos dentro do array é definido pelo valor **uniqueItems**.

## Conclusão

Embora bastante simples, este JSON Schema valida perfeitamente nossa regra de negócio e qualquer arquivo JSON que não a siga a regra levantará um erro de validação. Claro que se pode utilizar outros valores e objetos par adicionar mais regras de validação para um JSON e isso será abordado em outros artigos (Pretendo transformar esse artigo em uma série!)

Você pode gerar automaticamente (de maneira mais limitada) um JSON Schema ou validar um JSON através deste [site](https://extendsclass.com/json-schema-validator.html). A maioria das linguagens de programação possui alguma biblioteca de validação de JSON Schema. Para citar algumas:

**Python**:
```sh 
$ pip install jsonschema
```

**Javascript**:
```sh
$ npm install jsonschema
```

## Referências
- https://json-schema.org/learn/getting-started-step-by-step
- https://restfulapi.net/json-schema/
- http://json-schema.org/understanding-json-schema/index.html