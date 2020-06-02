---
layout: post
title:  "Emulando propriedades de rede usando Netem"
description: "NetEm (Network Emulator) é uma ferramenta agrupada ao gerenciador de tráfego do Linux (tc) que permite o administrador modificar certos parâmetros de entrada de pacotes no sistema, emulando diversos comportamentos de rede como atrasos e perdas de pacotes."
date: 2017-09-05 12:00:00
categories: [linux]
tags: [netem]
comments: true
author: lets00
image: assets/images/2017/emulando-propriedades-de-rede-com-netem/cenario.jpg
toc: true
---

NetEm (Network Emulator) é uma ferramenta agrupada ao gerenciador de tráfego do Linux (_tc_) que permite o administrador modificar certos parâmetros de entrada de pacotes no sistema, emulando diversos comportamentos de rede como atrasos e perdas de pacotes.

Essa ferramenta é bastante útil para administradores verificarem certas operações de redes em estados diferente do tradicional. Pode ser utilizada também para testes de protocolos de rede, comunicação entre rede, IPC, etc.

Nos primórdios do Linux o netem oferecia um serviço de maneira separada do _tc_. Contudo, com o amadurecimento do _tc_ muitas funções de controle de rede foram agrupado a ele, inclusive o netem.

## Instalação

Se você está executando um kernel superior a versão 2.6, as ferramentas e dependências necessárias já estão ativadas no kernel e já está presente por padrão na sua distribuição. O netem, assim como o _tc_, faz parte da suíte iproute2 que já vem instalado na maioria das distribuições conhecidas.

Sendo assim, você não vai precisar executar nenhum comando de instalação pois muito provavelmente seu sistema já o possui instalado e configurado.

## Emulando delays de redes distantes

Quanto maior a distância física de dois hosts maior o tempo de envio de um pacote entre eles. Podemos simular atraso causado pela distância através do netem. Primeiramente, vamos medir o tempo que a máquina leva para acessar uma página (ex: www.google.com) e retornar com uma resposta. Esse tempo de ida e volta de uma solicitação é denominado Round Trip-Time. Podemos utilizar diversas ferramentas para medir esse tempo. A ferramenta que é mais utilizada para esses fins é o ping que envia um pacote utilizando o protocolo ICMP (que é um protocolo utilizado para verificar acessibilidade de máquinas à rede de maneira direta, sem passar por camadas superiores como transporte pois o ICMP é executado na camada de rede).

Na figura abaixo analisamos o RTT para a página do Google.

<figure>
  <img src="/assets/images/2017/emulando-propriedades-de-rede-com-netem/ping1.png" alt="Comando ping" text="Comando ping" loading="lazy">
  <figcaption>Ping antes do netem</figcaption>
</figure>

Podemos observar pelas estatísticas do ping que o tempo médio das solicitações realizadas foi de 16.045 com tempo mínimo e máximo de 11.656 e 29.3, respectivamente. Agora vamos simular uma rede que possui uma grande distância. O comando abaixo é responsável por adicionar um atraso a a saída de uma determinada interface.

```sh
$ sudo tc qdisc add dev wlan0 root netem delay 100ms
```
onde:

- tc qdisc add: Adiciona uma qdisc. Caso possua alguma dúvida sobre o que esse comando realiza, sugiro ler esse post.
- dev wlan0: Indica qual o dispositivo de rede receberá a qdisc.
- root: Indica que o dispositivo é uma qdisc root.
- netem delay 100ms: Adiciona a 100ms de atraso a interface wlan0.

Vamos analisar novamente o RTT para a página do Google.

<figure>
  <img src="/assets/images/2017/emulando-propriedades-de-rede-com-netem/pingdelay.png" alt="Comando ping com delay" text="Comando ping com delay" loading="lazy">
  <figcaption>Ping após o netem</figcaption>
</figure>

Agora podemos ver pelas estatísticas do ping que o RTT para acessar a página do google aumentou em média para 121.118 com mínimo e máximo de 113.135 e 176.519, respectivamente. Logo vemos que o delay de 100 ms foi devidamente aplicado.

Podemos através dos parâmetros propiciados pelo netem aproximar mais da realidade adicionando _jitter_ que é uma variação estatística do atraso. O comando edita o comando acima, permitindo adicionar ± 10ms de _jitter_.

```sh
$ sudo tc qdisc change dev wlan0 root netem delay 100ms 10ms
```

A adição do _jitter_ é randômica e embora simule bem um ambiente de rede distante, não é a melhor maneira de afirmar que o atraso corresponde com grandes chances o de uma rede real. Isso porque precisamos utilizar técnicas para inferir aproximações estatísticas e não deixar simplesmente por um processo randômico não determinístico. Podemos fazer aproximações do tipo : 25% do tráfego que passa sofrerá um atraso de _jitter_ enquanto que os demais pacotes não sofrerão. O comando abaixo adiciona uma aproximação de 25% para o _jitter_.

```sh
$ sudo tc qdisc change dev wlan0 root netem delay 100ms 10ms 25%
```

Perfeito! Mas caso não possua uma aproximação definida (o que muitas vezes é difícil de inferir estatisticamente) podemos utilizar uma distribuição de probabilidade que simule o comportamento do _jitter_ em uma rede normal. A distribuição mais utilizada para atrasos e perda de pacotes é a distribuição normal (ou gaussiana). Esse tipo de distribuição é bastante utilizada em análises estatísticas e utiliza duas variáveis: uma média e um desvio-padrão. A média nada mas é do que o delay aplicado a rede (em nosso exemplo assume o valor 100 ms) e o desvio-padrão é o _jitter_. O comando abaixo aplica uma distribuição normal ao tráfego de rede.

```sh
$ sudo tc qdisc change dev wlan0 root netem delay 100ms 10ms distribution normal
```

Podemos utilizar outras distribuições de probabilidade (pareto, uniforme e paretonormal) dependendo do tipo de tráfego que circulará na nossa rede. Como no nosso exemplo são apenas pacotes ICMP onde o tamanho dos pacotes são invariáveis, o uso da distribuição normal é excelente e simula com uma grande precisão uma rede real. O conhecimento de técnicas estatísticas ajuda bastante a entender como o tráfego que circulará na rede se comportará (Uma adentro: Recomendo o livro do [Montgomery](http://www.amazon.com/Applied-Statistics-Probability-Engineers-Montgomery/dp/1118539710)!).

## Emulando perdas de pacotes

Uma variável importante existente em redes é a perda de pacotes. A perda de pacotes acontece quando um pacote enviado por algum motivo não alcança o destino. Isso também pode ser simulado utilizando o netem. O comando abaixo aplica uma perda de pacotes de 0.1%.

```sh
$ sudo tc qdisc change dev eth0 root netem loss 0.1%
```

## Emulando pacotes duplicados

Em algumas situações um pacote pode ser enviado mais de uma vez. Alguns protocolos de transporte utilizam métodos de identificação e correção desses pacotes (vide TCP) e outros simplesmente ignoram (vide UDP). Em ambos os casos é interessante emular o impacto que pacotes duplicados podem ter em uma rede de longa distância. O comando abaixo adiciona uma porcentagem de 2% na duplicação de pacotes.

```sh
$ sudo tc qdisc change dev eth0 root netem duplicate 2%
```

## Emulando pacotes corrompidos

Pacotes corrompidos são pacotes que são modificado no meio enviado devido a fatores externos como interferências eletromagnéticas e sujeiras microscópicas que gera uma grande ou pequena alteração. O comando abaixo adiciona uma porcentagem de 0.1% de corrupção.

```sh
$ sudo tc qdisc change dev wlan0 root netem corrupt 0.1%
```

## Conclusão

Netem é uma ferramenta incrível que permite alterar parâmetros cruciais de rede que podem levar a estados que naturalmente podem demorar a ocorrer. Os casos de uso do netem são bastante específicos é interessante conhecer bem como tirar proveito das suas funcionalidades. Dependendo do custo-benefício de tratamento de determinadas situações o netem pode ser um salva-vidas por permitir entender o comportamento da sua rede em um caso que e difícil de reproduzir naturalmente.

Uma experiência que tive com o netem foi em um trabalho no mestrado que consistia em verificar o comportamento de protocolos (SIP e SOP) desenvolvido por um grupo de pesquisa dirigido pelo meu orientador em uma rede multimídia com taxas de atraso normal e perdas baseada em processos probabilísticos de Gilbert-Eliot. O trabalho consistia em verificar o comportamento do protocolo em alguns casos difíceis de ocorrer com frequência em uma rede real mas que ocasionalmente ocorrem.

A dica que posso passar para aqueles que passem por uma experiência parecida é ter um bom conhecimento de estatística para conseguir emular a rede desejada com o mínimo de erro possível.

## Referências

1. [http://www.linuxfoundation.org/collaborate/workgroups/networking/netem](http://www.linuxfoundation.org/collaborate/workgroups/networking/netem)

2. [http://manpages.ubuntu.com/manpages/raring/man8/tc-netem.8.html](http://manpages.ubuntu.com/manpages/raring/man8/tc-netem.8.html)

3. [https://omf.mytestbed.net/projects/omf/wiki/NetEM_examples_of_rules](https://omf.mytestbed.net/projects/omf/wiki/NetEM_examples_of_rules)