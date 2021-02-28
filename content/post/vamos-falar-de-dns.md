---
author: "João Maia"
date: 2020-08-28
title: "Vamos falar de DNS?"
tags: [
    "dns",
    "infraestrutura"
]
categories: [
    "rede de computadores"
]
thumbnail: "img/vamos-falar-de-dns/dns.png"
---

Olá,

você já deve ter visto essa sigla alguma vez na sua vida. Talvez já tenha até configurado isso na sua máquina sem bem entender direito para que serve, né? Nesse texto pretendo fazer um resumão de DNS para ajudar qualquer pessoa que queira entender mais sobre o assunto. A definição formal tá aqui na [rfc1034](https://www.ietf.org/rfc/rfc1034.txt), sobre a implementação [rfc1035](https://www.ietf.org/rfc/rfc1035.txt) e para quem tiver mais curiosidade a [lista completa](https://en.wikipedia.org/wiki/Domain_Name_System#RFC_documents).

Se ficarem dúvidas ou tiverem sugestões de continuidade no assunto é só comentar no fim do artigo ou entrar em contato comigo por algum meio dos informados na página inicial do blog.

# O que é DNS?

## Introdução a rede de computadores

Vamos falar de forma breve e direta para nivelar todo mundo aqui. Para que seja possível você ler esse artigo é preciso que o servidor onde tem esse site hospedado te envie os dados dele para o seu navegador renderizar e ler. Para que isso ocorra é preciso que um conjunto de protocolos sejam respeitados para que essa entrega aconteça e ainda assim pode falhar e um desses protocolos é o DNS. Vamos fazer um pequeno paralelo para ficar mais claro. Por conta da pandemia estamos fazendo mais compras online que o normal. Com isso, para garantir que sua compra chegue até você é coletado de você algumas informações a respeito de onde deve ser feita a entrega. Essas informações fazem parte da exigência da transportadora que vai receber o pacote e cuidar que ele chegue até você. Tudo dando certo o pacote chega até você sem problemas mas pode ser que aconteça de algum acidente acontecer no meio do caminho e o pacote não chegar, assim como sua internet pode cair antes do site carregar.

Da [Wikipédia](https://pt.wikipedia.org/wiki/Modelo_OSI):

> O Modelo OSI (acrônimo do inglês Open System Interconnection) é um modelo de rede de computador referência da ISO dividido em camadas de funções, criado em 1971 e formalizado em 1983, com objetivo de ser um padrão, para protocolos de comunicação entre os mais diversos sistemas em uma rede local (Ethernet), garantindo a comunicação entre dois sistemas computacionais (end-to-end).

![Modelo OSI](/img/vamos-falar-de-dns/modelo_osi.png)

Detalhando essas camadas:

1. **Física**: Transmissão e recepção dos bits brutos através do meio físico de transmissão;
2. **Enlace**: Detecção de erros;
3. **Rede**: Roteamento de pacotes em uma ou várias redes;
4. **Transporte**: Oferece métodos para a entrega de dados ponto-a-ponto;
5. **Sessão**: Negociação e conexão com outros nós, analogia;
6. **Apresentação**: Formatação dos dados, conversão de códigos e caracteres; e
7. **Aplicação**: Funções especialistas (transferência de arquivos, envio de e-mail, terminal virtual).

Voltando para nossa analogia da compra, o site que você fez a compra seria a camda de aplicação, na camada de apresentação os seus dados vão ser formatadas para ficarem de acordo com a transportadora que por sua vez entre na camada de sessão onde ela começa a fazer parte desse fluxo da entreta. Na camada de transporte vamos definir qual vai ser a modalidade de entrega da transportadora que vamos usar e a camada de rede seria a inteligência de logistica dessa empresa e por fim na camada de enlace teriamos as vias de transporte e na física o transportador.

Tá bom, e onde entra o DNS nisso? Já reparou que no geral sempre ao cadastrar um endereço podemos dar apelidos para eles ou eles assumem que a pessoa fazendo a compra é o apelido? Esse ato de apelidar é o DNS. Imagine que você more com mais pessoas, toda a parte formal do seu endereço vai ser igual para você e demais pessoas que moram com você e por isso apelidos são importantes, assim podemos garantir que sua entrega vai ser feita a você e não para outra pessoa do seu apartamento ou casa.

## Utilidades do DNS

Até então ao que parece o DNS serve apenas para dar apelidos a entidades mas não é só isso. Vamos nesse artigo explorar três utilidades do DNS:
1. Nomear servidores;
3. Distribuição de carga; e
2. Descoberta de serviços.

### Nomear servidores

Essa é uma funcionalidade que já conversamos aqui um pouco. Vamos explorar um pouco mais essa funcionalidade, vamos entender um pouco mais sobre o `google.com` e para isso vamos usar o comando `host` do Linux:

```
$ host google.com
google.com has address 172.217.29.110
google.com has IPv6 address 2800:3f0:4004:807::200e
google.com mail is handled by 30 alt2.aspmx.l.google.com.
google.com mail is handled by 20 alt1.aspmx.l.google.com.
google.com mail is handled by 10 aspmx.l.google.com.
google.com mail is handled by 50 alt4.aspmx.l.google.com.
google.com mail is handled by 40 alt3.aspmx.l.google.com.
```

Veja que interessante, ele converte o DNS do Google em IP e ainda tem outra informações interessantes. Repare que ele forneceu duas versões de IP. Uma pausa para esse assunto, hoje temos duas versões do protocolo IP em uso na Internet, o [IPv4](https://pt.wikipedia.org/wiki/IPv4) e [IPv6](https://pt.wikipedia.org/wiki/IPv6), por conta de uma limitação de combinações diferentes de IP e a explosão de dispositivos e serviços que hoje conseguem ter interface com a rede de computadores o protocolo se tornou limitado. Com isso, surgiu o IPv6 que veio atender a essa nova demanda de IPs. Além disso, ele também identificou que nesse domínio do Google tem um serviços de email como podemos ver. Olhe agora como podemos explorar ainda mais o `host` no estudo do domínio do Google:

```
$ host -a google.com
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43764
;; flags: qr rd ra; QUERY: 1, ANSWER: 12, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	ANY

;; ANSWER SECTION:
google.com.		296	IN	AAAA	2800:3f0:4004:808::200e
google.com.		111294	IN	NS	ns1.google.com.
google.com.		111294	IN	NS	ns3.google.com.
google.com.		111294	IN	NS	ns2.google.com.
google.com.		111294	IN	NS	ns4.google.com.
google.com.		2	IN	SOA	ns1.google.com. dns-admin.google.com. 326790814 900 900 1800 60
google.com.		272	IN	A	172.217.29.110
google.com.		443	IN	MX	40 alt3.aspmx.l.google.com.
google.com.		443	IN	MX	30 alt2.aspmx.l.google.com.
google.com.		443	IN	MX	20 alt1.aspmx.l.google.com.
google.com.		443	IN	MX	10 aspmx.l.google.com.
google.com.		443	IN	MX	50 alt4.aspmx.l.google.com.

Received 298 bytes from 127.0.0.53#53 in 74 ms
```

Veja que interessantes quanta informação a mais temos, nesse momento vamos olhar apenas essas:

``` 
google.com.		111294	IN	NS	ns1.google.com.
google.com.		111294	IN	NS	ns3.google.com.
google.com.		111294	IN	NS	ns2.google.com.
google.com.		111294	IN	NS	ns4.google.com.
```

Vamos falar sobre isso nos próximos tópicos mas esses são os servidores do Google que fazem o papel de serviço de DNS deles, ou seja, são essas máquinas que sabem traduzir tudo que for `*.google.com` em um IP.

Curiosamente rodei o primeiro comando mais uma vez depois de um certo espaço de tempo e veja o resultado:

```
$ host google.com
google.com has address 216.58.222.78
google.com has IPv6 address 2800:3f0:4004:804::200e
google.com mail is handled by 10 aspmx.l.google.com.
google.com mail is handled by 50 alt4.aspmx.l.google.com.
google.com mail is handled by 20 alt1.aspmx.l.google.com.
google.com mail is handled by 40 alt3.aspmx.l.google.com.
google.com mail is handled by 30 alt2.aspmx.l.google.com.
```

Repare que o IP mudou e isso nos leva ao nosso próximo tópico! =)

### Distribuição de carga

Expor um servidor ao seu público pode ser algo complicado se o público aumentar de forma descontrolada e seu processamento por requisição não for extremamente rápido ou fazer atualizações nesse sem impactar o público são coisas complicadas de serem feitas por conta desse acesso direto. Um das boas práticas de arquitetura de software é colocar um distribuidor de carga na frente de serviços, são exemplos disso o [Apache](https://www.apache.org/), [NGINX](https://www.nginx.com/) e [HAProxy](http://www.haproxy.org/) de opções que você pode executar em sua infraestrutura de forma livre e outras soluções que já serão amarradas ao seu provedor de nuvem, por exemplo, [Elastic Load Balancing](https://aws.amazon.com/pt/elasticloadbalancing/) da AWS, [Cloud Load Balancing](https://cloud.google.com/load-balancing) do Google Cloud e [Load Balancer](https://azure.microsoft.com/pt-br/services/load-balancer) da Azure. Todas essas soluções vão criar para você uma interface única de comunicação com seus servidores. 

Vamos imaginar um cenário de ecommerce pequeno, minhaloja.com, com poucos acessos durante o ano e com uma máquina apenas consiga atender a demanda. Então, é época de Black Friday e nesse momento em especial do ano o acesso do site aumenta muito e um servidor não da conta e para isso o time de infraestrutura precisa colocar duas novas instâncias para sustentar a carga dado o estudo de capacidade feito durante o ano e tipo de hardware disponível. Como vimos na parte anterior o Google consegue responder diferentes IPs para o mesmo domínio, isso acontece pois o DNS dele está fazendo o papel de distribuir carga entre dois servidores. Para fazer isso no nosso ecommerce vamos inserir o NGINX na nossa arquitetura. Na *Arquitetura Normal* vamos ter apenas o NGINX e uma instância do ecommerce e na *Arquitetura Black Friday* vamos ter o NGINX com três instâncias do ecommerce, veja imagem abaixo. Qual a nossa estratégia com essa solução? Vamos delegar o DNS `minhaloja.com` ao nosso servidor de NGINX pois com isso ele fica delegado em distribuir a carga entre os servidores do ecommerce pois apesar de possível não seria o ideal aqui de fazer isso apenas com o DNS.

![Arquitetura Normal](/img/vamos-falar-de-dns/arquitetura_normal.png) ![Arquitetura Black Friday](/img/vamos-falar-de-dns/arquitetura_black_friday.png)

### Descoberta de serviços

Outro recurso muito interessante e extremamente útil nos tempos atuais com sistemas distribuídos é o uso do DNS para descoberta de serviços. Vamos voltar no caso anterior do ecommerce e agora introduzir um banco de dados na arquitetura que foi omitido na parte anterior. Imagina o problema que é se todos os três serviços apenas se comunicarem através de configuração de IP fixo, um troca de servidor programada ou não poderia causar algum problema no sistema por indisponibilidade de algum desses serviços devido a troca do IP configurado. Agora, vamos imaginar um quarto serviço nessa arquitetura que você possa ir nele e registrar quem é você, quase um cartório. Então, o nosso banco de dados poderia, por exemplo, se registrar como `db.internal.minhaloja.com` ao IP `10.20.30.40` assim nossa aplicação web só vai precisar saber esse domínio cadastrado, pois sempre que ele for interagir com o banco de dados ele vai perguntar a esse serviço central que é `db.internal.minhaloja.com` e caso o IP mude não será problema. Um ferramenta que faz isso e você pode estudar a respeito é o [Consul](https://www.consul.io/) da [HashiCorp](https://www.hashicorp.com/).
 
# Como o DNS funciona? 

## Resolvendo DNS no Linux

Vamos supor que você está no Linux e vai fazer acesso de algum site no seu navegador, esse blog, por exemplo. Como já sabemos o `blog.joaovrmaia.com` é apenas um apelido para abstrair algum IP na rede mundial de computadores, como já diria o Jefferson da [LinuxTips](https://www.linuxtips.io/), e vamos precisar desse IP para chegar lá e trocar dados com ele.

No Linux vai ocorrer a chamada [`gethostbyname()`](https://man7.org/linux/man-pages/man3/gethostbyname.3.html) quem vai começar a jornada pela resolução desse IP. Essa nossa chamada vai gerar uma consulta que será feita na Internet mas como nossa máquina lança essa consulta na Internet? No nosso sistema temos um arquivo, `/etc/resolv.conf`, que é gerenciado pelo sistema operacional e alimentado pela sua rede caso não tenha feito alguma modificação manual nele. Os valores desse arquivo no geral vem através do serviço de [DHCP](https://pt.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) que podemos falar a respeito em outro momento. Veja como é o arquivo:

```conf
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs must not access this file directly, but only through the
# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,
# replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0
search br
```

Então, nosso DNS está rodando para esse IP `127.0.0.53` que na verdade é uma abstração do [systemd](https://pt.wikipedia.org/wiki/Systemd). Então, vamos executar o comando sugerido no arquivo de configuração:

```
$ resolvectl status
Global
       LLMNR setting: no                  
MulticastDNS setting: no                  
  DNSOverTLS setting: no                  
      DNSSEC setting: no                  
    DNSSEC supported: no                  
          DNSSEC NTA: 10.in-addr.arpa     
                      16.172.in-addr.arpa 
                      168.192.in-addr.arpa
                      17.172.in-addr.arpa 
                      18.172.in-addr.arpa 
                      19.172.in-addr.arpa 
                      20.172.in-addr.arpa 
                      21.172.in-addr.arpa 
                      22.172.in-addr.arpa 
                      23.172.in-addr.arpa 
                      24.172.in-addr.arpa 
                      25.172.in-addr.arpa 
                      26.172.in-addr.arpa 
                      27.172.in-addr.arpa 
                      28.172.in-addr.arpa 
                      29.172.in-addr.arpa 
                      30.172.in-addr.arpa 
                      31.172.in-addr.arpa 
                      corp                
                      d.f.ip6.arpa        
                      home                
                      internal            
                      intranet            
                      lan                 
                      local               
                      private             
                      test                

...

Link 3 (wlp4s0)
      Current Scopes: DNS                      
DefaultRoute setting: yes                      
       LLMNR setting: yes                      
MulticastDNS setting: no                       
  DNSOverTLS setting: no                       
      DNSSEC setting: no                       
    DNSSEC supported: no                       
  Current DNS Server: fe80::96ea:eaff:feb5:aa35
         DNS Servers: 192.168.15.1             
                      fe80::96ea:eaff:feb5:aa35
          DNS Domain: ~.                       
                      br

...
```

Omitindo algumas partes esse são nossos reais servidores de DNS: 192.168.15.1 e fe8a0::96ea:eaff:feb5:aa35 nas duas versões de IP. Esse valor é do meu roteador e provedor de Internet que é o segundo passo nessa jornada para buscar o IP do meu blog. Antes de continuar a jornada, teria como eu evitar todo esse caminho em busca do IP? Algumas vez você já tenha passado por problemas de DNS em algum lugar e precisou colocar o DNS de forma manual na sua máquina, para isso precisou apenas editar o `/etc/hosts` correto? Como o sistema sabe como dele deve buscar ali? Existe outro recurso no nosso sistema que diz a prioridade de onde buscar essa informação, é o [`nsswitch`](https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html):

```
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files mdns4_minimal [NOTFOUND=return] dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

Como podemos ver em `hosts:          files mdns4_minimal [NOTFOUND=return] dns` o primeiro local a ser consultado é o o arquivo do sistema. Vamos fazer um teste! Vou editar no `/etc/hosts` o endereço do meu blog para localhost.

```
$ ping blog.joaovrmaia.com
PING jvrmaia.github.io (185.199.110.153) 56(84) bytes of data.
64 bytes from 185.199.110.153 (185.199.110.153): icmp_seq=1 ttl=56 time=20.7 ms
...
$ vim /etc/hosts
$ ping blog.joaovrmaia.com
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.061 ms
...
```

Agora, já sabemos como nasce a requisição para resolver um nome pelo DNS. Veja bem, olhando o seu `/etc/hosts`, reparou algo?

```
127.0.0.1	localhost 
127.0.1.1	avell

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Se consultar no [`man pages`](https://pt.wikipedia.org/wiki/Man_(Unix)) vai ver que depois da informação do IP podemos colocar vários nomes. Então, podemos imaginar que temos uma [banco de dados chave-valor](https://en.wikipedia.org/wiki/Key%E2%80%93value_database) que a chave é o IP e o valor é uma lista de nomes ou que cada nome se torna uma chave e o valor é o IP. A implementação aqui não importa no momento mas podemos entender que existe um conceito de banco de dados. Também podemos imaginar que o mesmo conceito se aplica na sequência da nossa consulta ao se propagar pela Internet. Então, podemos imaginar que o DNS é o maior e mais antigo banco de dados distribuído em execução.

## Desenhando a ideia da implementação

Essa consulta do DNS precisa de alguma forma ser propagada, por exemplo, quando vamos abrir um site no geral fazemos um acesso [HTTP](https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol)/[HTTPS](https://pt.wikipedia.org/wiki/Hyper_Text_Transfer_Protocol_Secure) na porta 80/443 do protocolo [TCP](https://pt.wikipedia.org/wiki/Transmission_Control_Protocol). No caso do DNS, o processo é diferente pois ele usa o protocolo [UDP](https://pt.wikipedia.org/wiki/User_Datagram_Protocol) na porta 53. Podemos acreditar que a escolha foi feita pela velocidade e baixo risco no caso de falha e os motivos disso podemos ter em algum outro aritgo discutindo sobre TCP x UDP. Agora que já temos um conhecimento de como essa consulta é feita, vamos voltar na estrutura dos dados do DNS.

![DNS Tree](/img/vamos-falar-de-dns/dns-tree.png)

Como podemos ver o DNS tem um estrutura de árvore onde tudo começa na `root zone` que são os grandes [ISP's](https://pt.wikipedia.org/wiki/Fornecedor_de_acesso_%C3%A0_internet) que fazem a composição principal da Internet e abaixo deles temos alguns domínios conhecidos por muitos de nós como o `.com` e `.br`. Podemos explorar ainda mais isso, por exemplo, vamos ver o meu blog que é `blog.joaovrmaia.com`. Então, temos o seguinte caminho: root zone, .com, .joaovrmaia e por fim blog. São três saltos na árvore que precisam ser feitos para chegar na folha da árvore que representa meu IP. Agora que sabemos dessa estrutura em árvore seria interessante ser perguntar o motivo disso, vamos considerar duas vertentes: algoritmos e segurança.

Vamos começar pelo ponto de vista de segurança. Imagina o caos se todos tivessem a liberdade de sair criando domínio como bem entender? Vamos analisar o meu caso novamente usando o blog. Meu blog é `blog.joaovrmaia.com`, o blog nada mais é que um apelido para o [Github Pages](https://pages.github.com/) da minha conta no [Github](https://github.com/) que administro na [AWS](https://aws.amazon.com/pt/) usando o serviço de DNS deles chamado [Route53](https://aws.amazon.com/pt/route53/). Esse meu domínio `joaovrmaia.com` hoje está na AWS porque ao comprar no `register.com` deleguei a responsabilidade da resolução dele para a AWS que por sua vez roda por baixo do `.com` que está por baixo dos `root servers`. Então, é assim que temos uma primeira visão de segurança pois cada nível dessa árvore do DNS a responsabilidade está associada a alguma entidade que se torna responsável pelo que executa no domínio mas em caso de infrações e não retratamento a camada superior pode interferir.

Agora vamos ver a questão do algoritmo. Eu tinha dado a ideia de um banco de dados chave-valor mas imagine todos os domínios da Internet dentro desse banco, inviável, né? Então, esse modelo de árvore e ser distribuído começa a fazer ainda mais sentido pois a carga será bem mais dividida. Vamos exemplorar dois exemplos para entender melhor, no primeiro caso queremos em uma rede local acessar outro serviço. Supondo que estou no `srv01.minharede.local` e quero chegar no `srv04.minharede.local` é bem simples pois a nossa requisição de DNS vai depender apenas do ambiente local, ou seja, precisamos ir apenas um nível acima na árvore para descobrir o IP. Agora, no segundo exemplo vamos imaginar que você está no seu PC pessoal em casa querendo compartilhar algo com alguém via [ngrok](https://ngrok.com/), por exemplo. Essa sua requisição vai ter que subir mais passos na árvore e uma hora voltar, imagina o quanto de possibilidades são evitas pelo simples fato do domínio ser <algumacoisa>.ngrok.io, ou seja, antes de chegar nos serviços internos do ngrok um universo de possibilidades vão ser removidas com as restrições do .io e .ngrok.

## Como funciona na prática

Como já tinha dito a borda da Internet se conversa através de grandes ISP. Essas entidades são os nossos `Root DNS servers` que tem como responsabilidade saber quem são nossos `Top-level domain (TLD) servers` que são as entidades responsáveis por domínios como o .com, .org, .net e afins além de todos os países como .nr, .uk, .fr e outros, [aqui](https://root-servers.org/) você pode visualizar os `Root DNS servers` igual na imagem abaixo os TLD servers estão nessa [lista](https://www.iana.org/domains/root/db). Vamos explorar um pouco mais os TLD servers observando o [.br](https://www.iana.org/domains/root/db/br.html) e [.google](https://www.iana.org/domains/root/db/google.html), veja que em ambos possuem a lista de servidores de domínio que fazem a gestão do DNS e seus respectivos contados administrativo e técnico. Por fim, os TLD servers vão saber indentificar os `Authoritative DNS servers` que são os domínios que usamos, por exemplo, joaovrmaia.com. Então, nesse nível a responsabilidade de resolução do domínio passa a ser do dono saindo dessa estrutura central da Internet. Nesse nível já estamos falando de configurar DNS para hospedar um site, ter seu serviço de e-mail e afins. Podemos dizer que existe ainda mais uma camada nessa estrutura que é o `Local DNS server`, por exemplo, em ambientes coorporativos isso é bem comum de se ter um DNS interno que só funciona para as máquina da rede local para abstrair o IP para facilitar o acesso a uma impressora remota na rede ou servidor de armazenamento compartilhado.

![Root DNS server SP](/img/vamos-falar-de-dns/root-dns-server-sp.png)

Então, agora que já sabemos a estrutura vamos imaginar o caso de você novamente em uma rede local que tem DNS interno e precisa resolver algum domínio. Nesse processo vamos dar ínicio a uma busca recursiva nessa árvore e o primeiro local a ser consultado vai ser o seu `Local DNS server`, estou considerando que não trabalhamos com caching, que por sua vez vai identificar se ele pode responder sobre aquela consulta e caso contrário redirecionar a três opções: `Root DNS server`, `TLD DNS server` ou `Authoritative DNS server`, dependendo da sua consulta.

### Recebendo a resposta da sua consulta

Agora que já sabemos muito mais a respeito da estrutura e propagação da requisição da resolução de domínio vamos entender as resposas que recebemos que nada mais é que um tupla com quatro valores: `Name`, `Value`, `Type` e `TTL`. O Type vai definir que resposta vamos obter em Name e Value, por exemplo, se você apenas consultou qual é o IP de blog.joaovrmaia.com o Name vai conter `blog.joaovrmaia.com` e o Value um IP, se o Type for A o valor vai ser IPv4 e se for AAA vai ser IPv6. Se o Type for NS ele vai responder quem é o Authoritative DNS server daquele domínio, para o caso de CNAME o apelido daquele domínio e por fim MX vai dizer quem é servidor de email daquele domínio.

Lista completa de tipos:

| Type  | Significado             | Valor                        |
|-------|-------------------------|------------------------------|
| SOA   | Authority               | Parâmetros da zona           |
| A     | IPv4                    | 32 bits                      |
| AAAA  | IPv6                    | 128 bits                     |
| MX    | Mail exchange           | Domínio a ser aceito o email |
| NS    | Name server             | Servidor DNS do domínio      |
| CNAME | Canonical Name          | Apelido                      |
| PTR   | Pointer                 | Apelido para IP              |
| SPF   | Sender policy framework | Regra de antispam do MX      |
| SRV   | Service                 | Controlador de domínio       |
| TXT   | Text                    | Texto ASCII                  |

Existem muitas outras coisas que podemos ainda falar sobre o funcionamento do DNS mas acredito que nesse texto podemos encerrar por aqui a conversa técnica. Espero que tenha conseguido passar uma visão geral sobre algo tão importante em nossa vida digital hoje.

# Obtendo um domínio

Existem diversas formas de obter um domínio, no caso de querer um .br recomendo usar o [registro.br](https://registro.br/) que é o que faço uso quando preciso. Para outros que não são tenho usado o [register.com](http://register.com/) ou o [Namecheap](https://www.namecheap.com/). Não tenho muitas opiniões a respeito disso mas só digo para que você tome cuidado na escolha de onde comprar para saber se a empresa presta um bom serviço em termos de disponibilidade do serviço e suporte. Algumas empresas fazem venda casada de subir um site pronto e domínio para o cliente e quando você decide sair o site do serviço deles a transferência desse DNS pode ser um problema.

# Implementações

Aqui vou listar algumas opções de serviços de DNS para você administrar seu `Authoritative DNS servers` ou `Local DNS server`

## Open Source

- [BIND](https://en.wikipedia.org/wiki/BIND)
- [PowerDNS](https://www.powerdns.com/)
- [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq)

## Comerciais

- [Route53](https://aws.amazon.com/pt/route53/)
- [Cloudflare](https://www.cloudflare.com/pt-br/dns/)
- [ns1](https://ns1.com/)

# Referências do artigo

- RFC's do DNS:
    - https://www.ietf.org/rfc/rfc1034.txt
    - https://www.ietf.org/rfc/rfc1035.txt
    - https://en.wikipedia.org/wiki/Domain_Name_System#RFC_documents
- Modelo OSI: https://pt.wikipedia.org/wiki/Modelo_OSI
- IPv4: https://pt.wikipedia.org/wiki/IPv4
- IPv6: https://pt.wikipedia.org/wiki/IPv6
- Apache: https://www.apache.org/
- NGINX: https://www.nginx.com/
- HAProxy: http://www.haproxy.org/
- Elastic Load Balancing: https://aws.amazon.com/pt/elasticloadbalancing/
- Cloud Load Balancing: https://cloud.google.com/load-balancing
- Load Balancer: https://azure.microsoft.com/pt-br/services/load-balancer
- Consul: https://www.consul.io/
- HashiCorp: https://www.hashicorp.com/
- LinuxTips: https://www.linuxtips.io/
- gethostbyname(): https://man7.org/linux/man-pages/man3/gethostbyname.3.html
- DHCP: https://pt.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol
- systemd: https://pt.wikipedia.org/wiki/Systemd
- nsswitch: https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html
- man pages: https://pt.wikipedia.org/wiki/Man_(Unix)
- Banco de dados chave-valor: https://en.wikipedia.org/wiki/Key%E2%80%93value_database
- HTTP: https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol
- HTTPS: https://pt.wikipedia.org/wiki/Hyper_Text_Transfer_Protocol_Secure
- TCP: https://pt.wikipedia.org/wiki/Transmission_Control_Protocol
- UDP: https://pt.wikipedia.org/wiki/User_Datagram_Protocol
- ISP: https://pt.wikipedia.org/wiki/Fornecedor_de_acesso_%C3%A0_internet
- Github Pages: https://pages.github.com/
- Github: https://github.com/
- AWS: https://aws.amazon.com/pt/
- Route53: https://aws.amazon.com/pt/route53/
- ngrok: https://ngrok.com/
- Root DNS servers: https://root-servers.org/
- Top-Level Domain servers: https://www.iana.org/domains/root/db
- registro.br: https://registro.br/
- register.com: http://register.com/
- Namecheap: https://www.namecheap.com/
- BIND: https://en.wikipedia.org/wiki/BIND
- PowerDNS: https://www.powerdns.com/
- dnsmasq: https://en.wikipedia.org/wiki/Dnsmasq
- Cloudflare: https://www.cloudflare.com/pt-br/dns/
- ns1: https://ns1.com/
