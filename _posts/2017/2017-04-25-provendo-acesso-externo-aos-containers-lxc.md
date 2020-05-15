---
layout: post
title:  "Provendo acesso externo aos containers LXC"
date:   2017-04-25 12:00:00
categories: [linux]
tags: [namespaces]
comments: true
image: https://www.hostnet.com.br/blog/wp-content/uploads/2018/01/linux-containers-lxc-hostnet.jpg
author: lets00
toc: true
---

Containers estão cada vez mais sendo utilizado para diversos fins como produção, desenvolvimento, testes, etc. Assim, é comum os containers precisarem acessar o mundo externo, seja para adquirir conteúdo como pacotes e programas ou para permitir acessá-los em qualquer lugar do globo.

Da mesma maneira que as máquinas virtuais conseguem acessar a internet utilizando artifícios como bridges, os containers podem também usufruir dessa tecnologia. Nesse post iremos mostrar os tipos de bridges existentes e como interligar seus containers para prover acesso externo.

## Tipos de bridge

### bridge direta

A maneira mais rápida e simples de permitir acesso externo aos
containers é inserir a interface física que promove acesso externo a
mesma bridge que está sendo utilizada pelos containers, retirando o
endereço lógico da interface física e adicionando-o na bridge.
Entretanto, essa abordagem trás uma série de problemas como:

-   Expõe a rede utilizada pelas máquinas físicas nos containers, o que,
    por questões de segurança e outras limitações não é o ideal.
    Imaginem possuir um servidor dhcp na sua rede e um servidor dnsmasq
    interligada a responder solicitações provenientes da bridge (Seria
    uma bagunça pois ambos os servidores iriam responder uma solicitação
    *discover* dhcp!);
-   Mistura o ambiente físico com o virtual;
-   Alguns switches não realizam redirecionamento de frames a mesma
    porta com MACs diferentes. Switches com suporte a VEPA (*Virtual
    Ethernet Port Agregator*) utilizam como base o protocolo IEEE
    802.11Qbg e modo hairpin não sofrem esse tipo de problema, pois são
    capazes de preencher a tabela MAC com mais de um endereço MAC por
    porta;
-   Interfaces Wlan não podem ser adicionadas a uma bridge devido um
    Access point não poder responder a solicitações de MACs que não
    solicitaram conexão diretamente ao ao mesmo.

Se essas limitações citadas acima não são realmente um problema no seu
cenário, você pode adicionar a interface de acesso externo a bridge dos
container. Como exemplo, suponhamos que a interface ens1 possua o IP
192.168.1.1/24 e que seja configurado como interface de gateway e a
interface do container seja a lxcbr0. Se a bridge foi criada utilizando
linux-bridge (que é o método padrão quando se instala o lxc), adicione a
interface ens1 através do comando:

``` {style="text-align:justify;"}
# brctl addif lxcbr0 ens1
```

Se a bridge foi criada utilizando openvswitch,  adicione a interface
ens1 através do comando:

``` {style="text-align:justify;"}
# ovs-vsctl add-port lxcbr0 ens1
```

### bridge nateada

Uma bridge nateada realiza NAT (Network Address Translate) para o
endereço da interface com acesso externo, ou seja, modifica todas as
solicitações provindas dos containers interligados à bridge em
solicitações com endereço da interface de acesso externo, identificando
cada endereço do container por um mapeamento de portas. Essa técnica é
também chamada de mascaramento, pois o tráfego que deixa os containers é
sempre mascarado (modificado). As principais características dessa
abordagem são:

-   Não é necessário adicionar a interface de acesso externo a bridge
    dos containers;
-   A rede utilizada pelos containers é isolada da rede das máquinas
    físicas;
-   Permite acesso mesmo a *switches* antigos sem modo *hairpin*.

Para natear a bridge dos containers, você precisa adicionar uma regra de
mascaramento na tabela de tratamento de pacotes do *firewall*. Como
exemplo, suponhamos que a bridge dos containers possua endereço
10.0.3.1/24. Então, vamos adicionar uma regra de mascaramento na cadeia
de pós roteamento (ou seja, após a máquina finalizar toda ação de
roteamento, definindo onde o pacote deverá ser encaminhado) na tabela de
nateamento do *firewall* através do comando:

    # iptables -t nat -A POSTROUTING -s 10.0.3.0/24 ! -d 10.0.3.0/24 -j MASQUERADE

O comando acima pode ser interpretado da seguinte maneira: se um pacote
pertencer a rede 10.0.3.0/24 (-s 10.0.3.0/24) quiser acessar uma rede
diferente dela mesma (! -d 10.0.3.0/24), mascare esse pacote (-j
MASQUERADE) com o ip da interface de acesso externo.

Referências
-----------

[http://www.networkworld.com/article/2197460/tech-primers/vepa–an-answer-to-virtual-switching.html](http://www.networkworld.com/article/2197460/tech-primers/vepa–an-answer-to-virtual-switching.html)

[https://www.flockport.com/lxc-networking-guide/](https://www.flockport.com/lxc-networking-guide/)

