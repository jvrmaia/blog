---
author: "João Maia"
date: 2020-12-02
title: "E esse tal de BGP?"
tags: [
    "bgp",
    "infraestrutura"
]
categories: [
    "rede de computadores"
]
thumbnail: "img/e-esse-tal-de-bgp/bgp.png"
---

Olá,

nesse artigo pretendo explicar um pouco sobre o que é o [Border Gateway Protocol (BGP)](https://tools.ietf.org/html/rfc4271) mas antes disso é preciso passar por alguns outros conceitos para no fim entender porque muitas das grandes falhas da internet mundial tem essa sigla envolvida e como isso também é usado na rede da sua empresa.

# Contextualizando

O primeiro passo é entender em qual nível estamos falando dentro do [Modelo OSI](https://pt.wikipedia.org/wiki/Modelo_OSI), no caso, a camada de aplicação onde também temos o [DNS](https://pt.wikipedia.org/wiki/Sistema_de_Nomes_de_Dom%C3%ADnio) que é amplamente conhecido, o [DHCP](https://pt.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) que é o responsável por distribuir [IP](https://pt.wikipedia.org/wiki/Endere%C3%A7o_IP). Entrando mais detalhes, o BGP vai trabalhar no plano de controle.

Fazendo uma analogia para melhor entender isso, vamos imaginar um estado do Brasil, Espírito Santo minha terra, e algumas de suas cidades como, por exemplo, Cariacica, Vila Velha, Vitória e Serra. Quando você precisa ir de Cariacica para Vitória, por exemplo, é normal que você use o Google Maps ou Waze para te guiar, isso é o papel do plano de controle, e como vai chegar, ou seja, se vai de bicicleta ou carro é o plano de dados.

Agora, voltando a nossa realidade de roteamento de internet precisamos de explicar mais um elemento dessa estrutura que são os [sistemas autônomos (AS)](https://pt.wikipedia.org/wiki/Sistema_aut%C3%B4nomo_(Internet)) que são prefixos de IP gerenciados por alguma entidade pública ou privada. Vamos ao exemplo, eu comecei a trabalhar com infraestrutura na UFES durante a graduação, eu era do time que cuidava do laboratório da graduação do departamento de informática e da infraestrutura do prédio dos professores e site do departamento. Isso tudo ficava dentro do domínio `inf.ufes.br` e era de nosso cuidado a rede `200.137.65.0/24`, esse era o nosso AS.

Dado essa introdução podemos dizer que o BGP é o protocolo que conecta os AS de forma descentralizada e assíncrona fazendo dele um dos protolos mais importantes e desafiadores da internet.

# Entendendo como funciona

![rede de exemplo](/img/e-esse-tal-de-bgp/bgp-funcionamento-1.png)

Considere uma rede com três AS: AS1, AS2 e AS3 de acordo com a imagem acima. Como podemos ver, a AS3 tem também uma subrede com prefixo X. Para cada AS temos um `Gateway Router` ou um `Internal Router`. O `Gateway Router` são os que tem capacidade de conectar um AS com outro e o `Internal Router` são os que se conectam a dispositivos de rede. Olhando no AS1, os roteadores 1a, 1b e 1d são `Internal Router`e o 1c é `Gateway Router`, por exemplo.

Agora, vamos pegar a considerar a tarefa de anúnciar a rede X para todos os roteadores da imagem. Primeiro, o AS3 vai enviar uma mensagem BGP para o AS2 falando que a rede X existe no AS3. Então, o AS2 vai repassar a mensagem para o AS1 avisando que o AS3 contém a rede X e para chegar nela pode passar por ele. Com isso, não só a existência da rede X é conhecida agora como também o seu caminho. Além disso, podemos classificar essas comunicações como externas (eBGP) e internas (iBGP) como podemos ver na imagem abaixo. Tudo isso sendo feito via TCP na porta 179.

![rede de exemplo](/img/e-esse-tal-de-bgp/bgp-funcionamento-2.png)

Então, podemos narrar a progagação da rede X:
1. 3d envia uma mensagem iBGP para 3c e 3a falando que tem X;
2. 3a envia uma mensagem eBGP para 2c falando que o AS3 tem X;
3. 2c envia para 2b que envia para 2a uma mensagem iBGP que AS3 tem X;
4. 2a envia uma mensage eBGP para 1c que AS3 tem X; e
5. por fim 1c terminar de propagar X no AS1 com iBGP.

Agora, todos sabem onde está X mas como saber agora exatamente qual a rota para alguem no AS1 ou no AS2 chegar em X? Temos agora que introduzir mais um componente no BGP que são seus atributos, são eles: `AS-PATH` e `NEXT-HOP`. O `AS-PATH` contém uma lista com todos os AS que por onde essa mensagem passou, isso além de ser usado para achar o caminho é feito para evitar loop. O `NEXT-HOP` é o IP do roteador que começa o `AS-PATH`.

Pensando com uma cabeça de programador com o que sabemos agora a mensagem de anúncio do BGP é basicamente uma pilha onde vamos colocando cada AS que a mensagem para e o valor `NEXT-HOP` seria o ponteiro para o próximo nó dessa estrutura de grafo da rede. Com isso, já podemos discutir como é o algoritmo de roteamento do BGP. Pensando de forma simples, dentre todas as mensagens que chegaram basta pegar a de menor pilha e processar ela. Essa algoritmo é conhecido como `hot potato routing` (roteamento batata quente) pois a ideia é que image uma batata quente na sua mão que precise chegar na de outra pessoa, a ideia é passar o mais rápido possível para o outra mais próxima de você até chegar no destino rápido. Também existe a versão contrária que é o `cold potato routing` (roteamento batata fria), e você deve ser perguntar as razões disso, veja a figura abaixo:

![rede de exemplo](/img/e-esse-tal-de-bgp/potato-routing.png)

Na figura temos 2 AS's com diversos roteadores esses dois AS possuem duas interfaces de comunicação. Considerando que temos agora o tarefa de levar um informação do `s1` para o `s2`. Se escolhermos o `hot` o que vai acontecer é que vamos passar pela `rota 1` pois queremos logo chegar no destino mas se escolhermos o `cold` vamos transmitir pelo AS1 até sair pela `rota 2`. E qual o motivo disso? Imagina a situação que você usuário de AWS que tem o `s1` rodando na região de SP e o `s2` rodando em Virginia e ambos estão em redes privadas e que a `rota 1` precisa passar por caminhos públicos e a `rota 2` é uma totalmente interna. Nesse caso, faz todo sentido o uso do algoritmo `cold` pois não queremos dados privados rodando em rede pública.

Outro ponto interessante, supondo que você seja o administrador do AS1 e um terceiro do AS2, não é porque você escolheu o modelo `cold` que a comunicação de `s1` e `s2` vão ser sempre assim. Como disse no começo, o BGP é descentralizado e assimétrico. Então, o administrado do BGP do AS2 pode manipular a tabela de rotas para usar o algoritmo `hot`.

O que acontece na prática? Nenhum dos dois. O algoritmo usado na prática é o seleção de rota. Cada roteador vai ter uma configuração de preferência local e isso pode ser configurado por seu auto-aprendizado ou pode aprender através de outros roteadores do seu AS. A forma desse aprendizado é controlada através de políticas criadas pelo administrador da rede e as isso é feito em forma de rank onde mais com maior rank são as escolhidas. Depois de varrer a preferência local o roteamento dos prefixos restantes é feito através da escolha do menor caminho que é feito pelo [`Distance-Vector (DV) Routing Algorithm`](https://en.wikipedia.org/wiki/Distance-vector_routing_protocol). Sobrando ainda mais prefixos a escolha de roteamento é feita pelo `hot potato` e por fim BGP identifiers.

# Verificando rotas de BGP

```
$ telnet route-views.saopaulo.routeviews.org
Trying 2001:12ff:0:6192::217...
Connected to route-views.saopaulo.routeviews.org.
Escape character is '^]'.

Hello, this is Quagga (version 0.99.24.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

route-views.saopaulo.routeviews.org> show ip bgp
BGP table version is 0, local router ID is 187.16.216.223
Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
              i internal, r RIB-failure, S Stale, R Removed
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  0.0.0.0          187.16.217.113                         0 53013 3356 i
*                   187.16.218.235           0             0 53013 i
*>                  187.16.216.232           0             0 28329 i
*  1.0.0.0/24       187.16.217.113                         0 53013 6762 13335 i
*                   187.16.218.235                         0 53013 16735 13335 i
*                   187.16.219.111                         0 263075 13335 i
*                   187.16.216.20                          0 28571 1251 13335 i
*                   187.16.219.111                         0 28329 13335 i
*>                  187.16.219.111                         0 52863 13335 i
*  1.0.4.0/22       187.16.221.197                         0 263075 6939 4826 38803 i
*                   187.16.221.197        1714             0 6939 4826 38803 i
*                   187.16.221.197        1714             0 6939 4826 38803 i
*>                  187.16.221.197        1714             0 6939 4826 38803 i
*                   187.16.217.113                         0 53013 3356 4826 38803 i
*                   187.16.218.235                         0 53013 16735 4826 38803 i
*                   187.16.221.197                         0 28571 6939 4826 38803 i
*                   187.16.221.197                         0 28329 6939 4826 38803 i
*                   187.16.221.197                         0 52863 6939 4826 38803 i
*  1.0.4.0/24       187.16.221.197                         0 263075 6939 4826 38803 i
*                   187.16.221.197        1714             0 6939 4826 38803 i
*                   187.16.221.197        1714             0 6939 4826 38803 i
*>                  187.16.221.197        1714             0 6939 4826 38803 i
*                   187.16.217.113                         0 53013 3356 4826 38803 i
*                   187.16.218.235                         0 53013 16735 4826 38803 i
*                   187.16.221.197                         0 28571 6939 4826 38803 i
 --More-- 
```

Como tinha dito, na primera coluna temos o `Network` que é o prefixo e dentro dele temos alguns `NEXT-HOP` para te direcionar ao AS da sua escolha presente no `AS-PATH`. Então, para você chegar em algum IP da rede `1.0.4.0/24`, por exemplo, é muito provável que você passe pelo AS 6939 pois podemos ver que ele é frequente nos `AS-PATH`. Isso é claro se seu pacote passar em `route-views.saopaulo.routeviews.org` pois essa é a tabela de rotas dele.

Como você pode ver não falamos de nenhum método de autenticação ou validação dos pares ao criar uma rede BGP e isso se da pelo fato de não existir de fato. A rede é formada através das operadoras formando pares e compartilhando anúncios de BGP. Vamos falar mais sobre isso agora no próximo tópico sobre os desafios do BGP.

# Desafios

Dado a simplicidade do BGP aparecem alguns desafios que vão desde problemas técnicos de configuração quanto de segurança. Então, vamos abordar de forma superficial aqui quais são esses desafios.

## Problemas

![desafios do bgp](/img/e-esse-tal-de-bgp/desafios-bgp.png)

Na figura acima temos o AS1 e o AS2 com suas redes de prefixo X e Y respectivamente. Se a rede X não tiver interseção com a rede Y essa configuração de BGP funciona normalmente dado que X e Y são anúnciadas pelo seus BGP's. Porém, se tiver isso vai causar sérios problemas de conectividade na sua rede e isso é bem comum. Levando para o lado real, é normal que empresas usando AWS com o tempo tenham várias VPC's e contas diferentes de AWS e em algum momento essas redes precisem conversar, caso não tenha sido feito um bom projeto de redes antes para organizar essa infraestrutura pode ser bem problemático conectar elas e na maioria dos casos a resolução é usar o recurso da prioridade da tabela de rotas do BGP e criar rotas estáticas, por exemplo, na rede X temos a 192.164.1.0/24 e na rede Y também e por acidente a Y anúncia essa rede em X, a solução é ensinar na rede X que essa rede pertence ao seu VPC e não procurar no BGP.

## Segurança

Como vimos no tópico anterior existe uma sensibilidade bem grande no BGP para falhas e um ataque conhecido é o [hijacking](https://pt.wikipedia.org/wiki/Session_hijacking) de sessão de BGP onde o atacante faz anúncios falsos de prefixo para roubar tráfego de dados. Uma solução para isso são os [RPKI (Resource Public Key Infrastructure)](https://en.wikipedia.org/wiki/Resource_Public_Key_Infrastructure) que é uma infraestrutura de chave pública que utiliza criptografia para que um bloco de endereços enviado por meio de um AS seja validado. Existem também outras soluções e técnicas para resolver isso que curiosamente na manutenção de uma dessas formas a pouco tempo atrás levou a internet a sofrer um apagão parcial de alguns sites: ["'IP outage' on CenturyLink network caused by Flowspec mitigation, says Cloudflare CEO"](https://www.datacenterdynamics.com/en/news/ip-outage-centurylink-network-caused-flowspec-mitigation-says-cloudflare-ceo/).

# Referências

- BGP: https://tools.ietf.org/html/rfc4271
- Modelo OSI: https://pt.wikipedia.org/wiki/Modelo_OSI
- DNS: https://pt.wikipedia.org/wiki/Sistema_de_Nomes_de_Dom%C3%ADnio
- DHCP: https://pt.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol
- IP: https://pt.wikipedia.org/wiki/Endere%C3%A7o_IP
- ICMP: https://pt.wikipedia.org/wiki/Internet_Control_Message_Protocol
- Sistemas autônomos: https://pt.wikipedia.org/wiki/Sistema_aut%C3%B4nomo_(Internet)
- Distance-Vector (DV) Routing Algorithm: https://en.wikipedia.org/wiki/Distance-vector_routing_protocol
- Hijacking: https://pt.wikipedia.org/wiki/Session_hijacking
- RPKI (Resource Public Key Infrastructure): (https://en.wikipedia.org/wiki/Resource_Public_Key_Infrastructure
