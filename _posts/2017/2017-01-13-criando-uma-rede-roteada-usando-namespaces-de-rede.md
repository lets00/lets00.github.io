---
layout: post
title:  "Criando uma rede roteada usando namespaces de rede"
date:   2017-01-13 12:00:00
categories: [linux]
tags: [namespaces]
comments: true
image:
  path: /assets/images/2017/criando-uma-rede-roteada-usando-namespaces-de-rede/cenario.jpg
  height: 100
  width: 100
---

<img src="/assets/images/2017/criando-uma-rede-roteada-usando-namespaces-de-rede/cenario.jpg" alt="Cenário inicial" text="Cenário inicial" loading="lazy">

## Introdução

Alguns conceitos de redes são complexos de explicarem a leigos apenas utilizando os métodos tradicionais de ensino (lousa, livros, etc). Se uma imagem vale mais que mil palavras, uma animação/vídeo vale mais que mil imagens. Sabendo disso, quando ministra-se alguma disciplina de configuração de redes, nada mais direto e que apresenta melhor absorção do que apresentar algo na prática, como se utiliza no mundo real. E é por isso que muitos optam por utilizarem simuladores e/ou emuladores de rede, com a expectativa de reduzir a diferença entre o mundo teórico e o mundo prático. Para a apresentação de temas mais abstratos essa é com certeza uma boa maneira de aprender. Contudo, simuladores por mais perfeitos que sejam ainda são simuladores, possuem suas peculiaridades, seu grau de imersão com a realidade, suas dificuldades, e muitas vezes podem complicar mais do que ajudar.

Uma característica interessante que surgiu a poucos anos no kernel linux é a possibilidade de criar isolamento de certos recursos de uma máquina, permitindo que esses ambientes isolados não interfiram diretamente no ambiente concreto, sendo conceito chave para a expansão do uso de ferramentas como containers, etc. Essa característica de criação de ambientes isolados é denominado namespace e aqui no blog já escrevi alguns artigos que tratam desse tema [1][2]. VER OS LINKS QUE ELE REFERENCIA!!!!!!!!!!!!!!!!

O uso de um container ou máquina virtual poderia ser uma solução mais realista em termos de aproximação com a realidade quando comparado com um simulador, mas mesmo esses modelos apresentam desperdício de recursos quando queremos apenas apresentar um conceito simples como roteamento, pois, em um container, por exemplo, temos isolamento de disco, rede, uts, processos, etc, quando seria necessário apenas um ambiente de rede isolado do atual.

Dessa maneira, podemos preferir criar apenas um namespace de rede e utilizá-lo para demonstrar alguma configuração do que criar todo um ambiente isolado que não será necessário.

Nesse artigo irei demonstrar como duas máquinas, em redes diferentes conversam entre si através de um roteador linux usando apenas namespaces de rede.

## Cenário e ferramentas necessárias

A Figura 1 apresenta o cenário que será utilizado nesse post para criar uma rede e demonstrar o comportamento de um roteador linux. Precisaremos de 3 namespaces de rede, um será o routeador e os demais, máquinas em redes diferentes tentando trocar informação entre elas usando o roteador como intermediário dos pacotes. Além disso precisaremos também de dois enlaces para interligar os namespaces que farão papel de clientes ao namespace que fará o papel de roteador.

<figure>
  <img src="/assets/images/2017/criando-uma-rede-roteada-usando-namespaces-de-rede/cenario.jpg" alt="Cenário inicial" text="Cenário inicial" loading="lazy">
  <figcaption>Figura 1 - Cenário</figcaption>
</figure>

Lembre-se que para utilizar o namespace de rede é necessário que seu kernel seja superior a versão 3.0. Sugiro que, para futuras atividades, seu kernel esteja atualizado com a versão mais estável lançada. Utilizaremos nesse exemplo duas redes de classe C (192.168.1.0/24 e 192.168.2.0/24).

## Criando os namespaces de rede

O primeiro passo é criarmos os namespaces de redes que utilizaremos para construção desse cenário. O comando ip netns permite o usuário gerenciar todos os namespaces de rede que estão sendo presente em seu sistema. Antes de criarmos, listaremos os namespaces já existentes através do comando:

```sh
# ip netns
```

Como não criamos nenhum namespace e o sistema geralmente não configura nenhum automaticamente, o comando irá retornar vazio, indicando que não existe nenhum namespace no sistema. Para criar os namespaces dos clientes e do router, usaremos o comando:

```sh
# ip netns add n1
# ip netns add n2
# ip netns add router
```
Após criado, podemos listar os namespaces existentes no sistema:

```sh
# ip netns
router
n2
n1
```

## Criando os enlaces

Após a criação dos namespaces, vamos criar os enlaces necessários para interligá-los, formando assim uma rede interconectada de namespaces. Para isso, vamos criar interfaces do tipo veth (virtual ethernet). Uma veth cria um par de interfaces interligadas por um enlace . Esse par de interfaces só existem juntas, ou seja, é impossível criar uma veth apenas com uma única interface. Tentar deletar uma das interfaces deleta também a outra ponta. Por nomenclatura vamos criar duas interfaces veth, a primeira será nomeada com o nome blueX(0 e 1), e a segunda com o nome redX(o e 1).

```sh
# ip link add dev blue0 type veth peer name blue1
# ip link add dev red0 type veth peer name red1
```

## Conectando os enlaces aos namespaces

Criados os enlaces e interfaces, vamos interligá-las aos namespaces dos clientes e do router. Para manter uma lógica, vamos interligar as interfaces que terminam com o numeral 1 no namespace do roteador e as de numeral 0 nos respectivos clientes. É bom ressaltar que a ordem de interligação não altera o funcionamento. A veth blue será utilizada para interligar o namespace n1 ao router. O comando abaixo realiza esse procedimento:

```sh
# ip link set dev blue0 netns n1
# ip link set dev blue1 netns router
```

A veth red será utilizada para interligar o namespace n2 ao router:

```sh
# ip link set dev red0 netns n2
# ip link set dev red1 netns router
```

Agora todos os namespaces estão interligados, mas por padrão suas interfaces são conectadas em estado DOWN. Precisamos definir as interfaces no estado UP para utilizá-las. Existe duas maneiras de realizar essa atividade. A primeira e mais intuitiva é definindo um endereço IP para a interface, a segunda é utilizando o utilitário ip para subir manualmente sem definir endereços IP. Nesse exemplo iremos utilizar o segundo método, executando o comando:

```sh
# ip netns exec router ip link set blue1 up
```

Esse comando na realidade é a união de dois comandos: o primeiro comando (ip netns exec router) é para indicar que queremos executar um comando de rede em um namespace (nesse caso, no namespace router) e o segundo comando (ip link set blue1 up) é o comando de rede que será executado naquele namespace. Sabendo disso, conduziremos o estado de todas as interfaces para o estado UP:

```sh
# ip netns exec router ip link set red1 up
# ip netns exec n1 ip link set blue0 up
# ip netns exec n2 ip link set red0 up
```

Por padrão, todas as interfaces de um namespace de rede estão no estado DOWN. Para que um namespace possa realizar testes de ping a ela mesma através de um IP que definiremos numa de suas interfaces, precisaremos levantar a interface de loopback. Podemos usar a mesma lógica aprendida anteriormente e usarmos o comando:

```sh
# ip netns exec router ip link set lo up
# ip netns exec n1 ip link set lo up
# ip netns exec n2 ip link set lo up
```

Obs: Se quisermos que todos os comandos relacionados a redes digitados no terminal seja relacionado a um determinado namespace de rede com o intuito de evitar digitar longos comandos, podemos entrar no bash especial que carrega as informações de rede do namespace. Isso pode ser realizado através do comando:

```sh
# ip netns exec router bash
```

Agora, todo comando de rede será relacionado ao namespace router. Tomem cuidado pois, diferentemente de um container ou máquina virtual, onde as variáveis de terminal PS1, Ps2 são alteradas e isso facilita identificar qual o ambiente está controlando, em um namespace de rede apenas a parte de rede é modificada, logo todas as demais coisas serão do ambiente físico, dificultando a identificação do usuário ao ambiente que se encontra (se encontra-se na máquina sem isolamento ou no namespace de rede isolado). Tenha como prática, após o término das configurações no ambiente do namespace, deixar aquele ambiente e voltar ao ambiente do sistema. Para isso, execute o comando:

```sh
# exit
```

## Definindo redes e rotas

Vamos definir os endereços IPs de cada namespace. O roteador possuirá em cada interface sempre o último endereço da rede válido, pois essa interface atuará como gateway da rede. Cada namespace possuirá em sua interface o primeiro endereço da rede válido. Para isso, começaremos com o namespace n1.

```sh
# ip netns exec n1 bash
# ifconfig blue0 192.168.1.1/24
# route add default gw 192.168.1.254 blue0 # Configura o gateway padrão
# exit
```

Configurando o namespace n2:

```sh
# ip netns exec n2 bash
# ifconfig red0 192.168.2.1/24
# route add default gw 192.168.2.254 red0 # Configura o gateway padrão
# exit
```

Configurando o router:

```sh
# ip netns exec router bash
# ifconfig blue1 192.168.1.254/24
# ifconfig red1 192.168.2.254/24
```

Ainda no router, garanta que o redirecionamento de pacotes está ativo. Uma observação importante é que essa configuração foge do escopo do namespace de rede, o que ativará sua máquina a também realizar redirecionamento de pacotes (o que em muitos casos já é a opção padrão).

```sh
# echo 1 > /proc/sys/net/ipv4/ip_forward
```

Quando você adiciona um IP a uma determinada interface, por padrão o utilitário ip já cria também uma rota para aquela interface. Podemos verificar a tabela de roteamento a partir do comando:

```sh
# route -n
Tabela de Roteamento IP do Kernel
Destino         Roteador        MáscaraGen.    Opções Métrica Ref   Uso Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 blue1
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 red1
``` 

## Testando conectividade

Tudo pronto! Nosso roteador é capaz de realizar o roteamento de pacotes e podemos acessar qualquer namespace. Tente pingar através do namespace n1 o namespace n2:

```sh
# ip netns exec n1 ping 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
64 bytes from 192.168.2.1: icmp_seq=1 ttl=63 time=0.070 ms
64 bytes from 192.168.2.1: icmp_seq=2 ttl=63 time=0.059 ms
...
```

## Conclusão

Vimos através desse exemplo que podemos utilizar namespaces para realizar criação e configuração de rede com o intuito de substituir o uso de simuladores e até mesmo máquinas virtuais e containers quando precisamos realizar uma demonstração do funcionamento de uma rede simples. O isolamento provindo do uso do namespace de rede foi capaz de sozinho demonstrar como configurar um router linux simples quando desejamos realizar roteamento entre redes. A grande vantagem é que todos os comandos de rede realizados são os mesmos usados em um ambiente real, sem a necessidade de instalação de ferramentas por fora, usando features disponíveis no próprio kernel linux (o que torna independente de distribuição).

## Referências

1. [http://man7.org/linux/man-pages/man7/namespaces.7.html](http://man7.org/linux/man-pages/man7/namespaces.7.html)