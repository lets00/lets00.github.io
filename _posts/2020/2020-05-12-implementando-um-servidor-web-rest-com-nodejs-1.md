---
layout: post
title:  "Implementando um servidor Web REST com NodeJS - #01 (Introdução)"
date:   2020-05-13 12:00:00
categories: [javascript]
tags: [nodejs, javascript]
comments: true
image:
  path: https://cdn-images-1.medium.com/max/1200/1*Jx8EXy-c6XKnIEQgPGpz7Q.jpeg
  height: 100
  width: 100
---

![Logo do NodeJS](https://cdn-images-1.medium.com/max/1200/1*Jx8EXy-c6XKnIEQgPGpz7Q.jpeg "Logo do NodeJS" loading="lazy")

Esse é o **Artigo 1** de uma série de artigos sobre como implementar um servidor _web_ REST usando o NodeJs. Os demais artigos podem ser acessados através dos links abaixo:

Implementando um servidor Web REST com NodeJS- parte 01 (Introdução)

---

## Introdução

Nesse artigo vamos explorar uma das atividades que é muito recorrente no desenvolvimento de **aplicações back-end**: A implementação de um servidor _web_ que recebe uma requisição provinda do front-end, processa-a e responde com sucesso ou com uma mensagem de erro.

Nosso objetivo será **desenvolver** uma aplicação responsável por cadastrar, ler, atualizar e remover (o CRUD básico) um usuário de um banco de dados NoSQL. A aplicação front-end comunicará com o back-end utilizando o protocolo HTTP. Logo, nossa aplicação terá que ser capaz de receber a requisição, analisar o que o front-end demanda, processar e respondê-la com o objeto solicitado. Esse objeto será modelado através de um padrão aberto simples denominado **JSON**.

## Infraestrutura básica

Para tratar a requisição do usuário, vamos precisar inicialmente da seguinte **infraestrutura**:
- Servidor _web_ (NodeJS)
- Um banco de dados NoSQL (MongoDB)
- Um gerenciador GUI pra manipular o banco de dados (Mongo-Express)

Iremos utilizar a linguagem Javascript para construir o servidor _web_. A plataforma que permite executar javascript fora do navegador é o NodeJS.

O banco NoSQL que utilizaremos é o **MongoDB**, um banco de dados que trabalha com uma linguagem de manipulação de dados muito semelhante ao javascript (diferente da linguagem SQL) e que nos permite trabalhar utilizando uma abordagem denominada de _schemaless_ (sem esquema).

O gerenciamento dos dados armazenados no banco de dados será realizado pelo **Mongo-Express**, uma interface _web_ que permite administrar o MongoDB pelo navegador, facilitando a interação com os documentos armazenados e diminuindo a curva de aprendizado do mesmo.

### Porque utilizar _schemaless_?

_Schemaless_ não obriga que os dados sejam armazenados seguindo uma série de especificações como os parâmetros que devem ser utilizados e os tipos de dados de cada um desses parâmetros.

O processo de esquematização de como os dados devem ser armazenados e manipulados é denominado **modelagem** e precisa ser realizado antes de iniciarmos as operações sobre um banco de dados. Na **modelagem** define-se as **tabelas**, os **relacionamentos** entre as mesmas, os **tipos de dados**, **chaves primárias**, **secundárias**, **estrangeiras**, entre outros. Após a modelagem, outro processo denominado **normalização**, **simplifica a modelagem** e busca **eficiência** no acesso aos dados reduzindo redundâncias e possíveis inconsistências.

Os bancos de dados que trabalham com essa abordagem não precisam realizar a modelagem e normalização dos dados. Em vez disso, o programador no momento em que realiza o armazenamento de dados precisa se preocupar como serão armazenados, modelados, etc. O processo de _schemaless_ transfere a obrigação do banco de dados de verificar os dados antes de manipulá-lo (armazenar, ler) para a aplicação.

No nosso caso de uso, optamos utilizar essa abordagem, pois não temos definido ainda uma estrutura de como os dados serão armazenados e manipulados. Vamos deixar para a aplicação back-end que iremos construir a responsabilidade de modelar o comportamento dos dados. É bom enfatizar que existem diversas outras razões que nos fazem cooptar por um banco de dados sem esquema, porém, no momento, foge dos nossos objetivos com essa aplicação.

## Instalação

Instalaremos a última versão LTS do NodeJS via **NVM** (_Node Version Manager_). Caso não possua o NVM instalado na sua máquina, leia esse [artigo](https://lets00.github.io/posts/gerenciando-versoes-do-node-com-nvm/) para saber como instalá-lo na sua máquina. Para instalar o NodeJS, execute o comando abaixo:

```sh
$ nvm install --lts 
```

O próximo passo é instalar o MongoDB e o Mongo-Express. Uma forma simples é através de um **container Docker**, pois dispensa preocupação com dependências, bibliotecas, etc. O container trará o serviço configurado e podemos focar nossa atenção na implementação do servidor _web_. 

### Instalando o Docker

Para instalar o Docker no Ubuntu, execute os passos abaixo:

```sh
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
$ sudo apt install
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Se o sistema operacional que você estiver utilizando for diferente do Ubuntu, veja a [documentação oficial do Docker](https://docs.docker.com/engine/install/) para saber qual o procedimento precisa ser realizado no seu sistema.

Depois de instalar o docker, precisaremos baixar a imagem do container do MongoDB e o Mongo-Express. O processo tradicional para executar um container é: **informar as imagens**, **adicionar os volumes** necessários, **exportar as portas** que serão utilizadas para acesso externo ao container, **configurar variáveis** de ambientes, **conectar** o container do Mongo e o container do Mongo-Express para que o mesmo possa consultar os documentos que serão armazenados no banco.

Realizar esse procedimento via linha de comando (CLI) gera **comandos gigantescos** e nada prático quando precisamos adicionar vários outros serviços a uma aplicação. Pensando nisso, a equipe do docker desenvolveu uma ferramenta para definir e executar **múltiplos containers** utilizando uma configuração mais simples para humanos. A ferramenta em questão chama-se `docker-compose` e nela podemos definir quais containers serão executados e quais atividades precisam realizar através de um **arquivo YAML**.

### Instalando o docker-compose

Para instalar o `docker-compose`, execute o comando abaixo:

``` sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Vamos criar a **pasta do nosso projeto**, onde tudo referente à construção do servidor Web será salvo.

```sh
$ mkdir servidor-web
$ cd servidor-web/
```

No diretório, criaremos um arquivo YAML com as definições dos containers mencionados anteriormente. Abra o editor de texto de sua preferência e crie um arquivo com o nome `docker-compose.yml` com as informações descritas abaixo:

```yaml
version: '3'
services:
  mongo:
    image: mongo:latest
    volumes:
      - ./volumes/mongo:/data/db
    ports:
      - 27017:27017
  mongo-express:
    image: mongo-express
    ports:
      - 8081:8081
    links:
      - mongo
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: root
```

O arquivo acima define dois serviços que serão criados: **mongo** e **mongo-express**. Cada um dos serviços tem sua própria imagem, exporta as portas padrões para serem acessadas fora dos containers, cria uma conexão entre o mongo e o mongo-express, define o volume que será utilizado pelo mongo e as variáveis com senha e usuário para acessar o mongo-express. O serviço do mongo exporta a pasta `/data/db` (onde ficam armazenados todos os documentos do MongoDB) para a pasta `./volumes/mongo`. Isso garante a persistência dos dados quando o serviço do mongo for interrompido ou o container for destruído. Lembre-se que os containers criados pelo docker são por natureza efêmeros, ou seja, os dados não são persistidos quando o container é destruído.

Precisamos agora criar a pasta `./volumes/mongo` para que não haja erros quando criarmos os containers.

```sh
$ mkdir -p ./volumes/mongo
```

Pronto, agora podemos criar os serviços através do docker-compose:

```sh
$ sudo docker-compose up -d
```
O comando acima vai preparar todo o ambiente que definimos no arquivo YAML. Acessaremos o `mongo-express` e verificar se tudo foi levantado corretamente.

## Acessando o mongo-express

Abra o navegador de sua preferência e digite na barra de navegação: [http://localhost:8081](http://localhost:8081). O resultado deve ser semelhante ao apresentado abaixo:

![Navegador acessando o mongo-express](/assets/images/2020/implementando-um-servidor-web-rest-com-nodejs-1/mongo-express.png "Navegador acessando o mongo-express" loading="lazy")
Figura 1 - Acesso ao mongo-express

Podemos ver que existem 3 bases de dados(_admin_, _config_ e _local_). Essas são as bases criadas automaticamente quando instalamos o MongoDB.

## Conclusão

No próximo tutorial aprenderemos como iniciar um servidor _web_ e como tratar as primeiras requisições recebidas.

Dúvidas, comentários ou sugestões, deixem abaixo.
