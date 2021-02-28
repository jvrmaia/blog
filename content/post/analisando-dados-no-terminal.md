---
author: "João Maia"
date: 2020-05-01
title: "Analisando dados no terminal"
tags: [
    "terminal",
    "dicas",
]
categories: [
    "produtividade",
    "devops"
]
thumbnail: "img/analisando-dados-no-terminal/terminal-tips.png"
---

Olá,

recentemente fazendo uma inspeção na aplicação precisei ver seu logs para entender com mais detalhes o que estava acontecendo em termos de comportamento e com isso para ser produtivo na atividade fiz uso de recursos presentes no meu terminal para realizar a atividade. Neste artigo vou compartilhar as ferramentas que usei e como elas podem ser úteis para você também. O interessante que depois de falar a respeito disso no [Twitter](https://twitter.com/jvrmaia/status/1255963940804313088) surgiram comentários interessantes. O primeiro foi esse [artigo](https://adamdrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html) e o segundo foi a ferramenta [datamash](https://www.gnu.org/software/datamash/).

> AVISO: não assuma que sei isso tudo de memória e não se pressione a saber, muita coisa aqui tiver que procurar no Google que não lembrava ;D

# As ferramentas

Os principais comandos que vou utilizar aqui são: [cat](http://man7.org/linux/man-pages/man1/cat.1.html), [jq](https://stedolan.github.io/jq/), [awk](https://www.gnu.org/software/gawk/manual/gawk.html), [sort](http://man7.org/linux/man-pages/man1/sort.1.html), [tail](http://man7.org/linux/man-pages/man1/tail.1.html), [grep](http://linuxcommand.org/lc3_man_pages/grep1.html), [uniq](http://man7.org/linux/man-pages/man1/uniq.1.html) e [pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html).

## cat

O `cat` é um comando bem simples que é usado no geral para mostrar no terminal as informações de uma arquivo ou mais passados via argumentos, por exemplo:

```
$ cat hello.txt
world
```

Existem alguns outros recursos interessantes que valem nota para o uso que são as opções `-n` e `-b`:

```
$ cat -n hello.txt
     1 world
$ cat linhas.txt
primeira linha

segunda linha
terceira linha
$ cat -b linhas.txt
     1	primeira linha

     2	segunda linha
     3	terceira linha
```

Ambos mostram o número da linha com a diferença que na segunda opção as linhas em branco não são consideradas.

### Bônus

Descobrir o que acontece se fizermos isso:

```
$ cat >a.txt<<EOF
> abcEOF
> 123
> EOF
```

## tail

O `tail` parece com o `cat` mas o que ele faz é retornar os dados na ordem inversa do `cat` e com limitação na quantidade de linhas. Vamos testar com o `/etc/resolv.conf`:

```
$ cat -n /etc/resolv.conf
     1	# This file is managed by man:systemd-resolved(8). Do not edit.
     2	#
     3	# This is a dynamic resolv.conf file for connecting local clients to the
     4	# internal DNS stub resolver of systemd-resolved. This file lists all
     5	# configured search domains.
     6	#
     7	# Run "systemd-resolve --status" to see details about the uplink DNS servers
     8	# currently in use.
     9	#
    10	# Third party programs must not access this file directly, but only through the
    11	# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,
    12	# replace this symlink by a static file or a different symlink.
    13	#
    14	# See man:systemd-resolved.service(8) for details about the supported modes of
    15	# operation for /etc/resolv.conf.
    16
    17	nameserver 127.0.0.53
    18	options edns0
```

Usei o `cat` aqui para termos uma visibilidade do quem no arquivo como um todo. Agora, vamos ao `tail`.

```
$ tail /etc/resolv.conf
#
# Third party programs must not access this file directly, but only through the
# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,
# replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0
```

Já deu para notar que a quantidade de dados que veio foi reduzida, sabendo que são 18 linhas, para ver todas com o `tail`, podemos fazer:

```
$ tail -n 18 /etc/resolv.conf
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "systemd-resolve --status" to see details about the uplink DNS servers
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
```

Assim como podemos fazer para ver menos linhas:

```
$ tail -n 2 /etc/resolv.conf
nameserver 127.0.0.53
options edns0
```

### Bônus

Procure algum arquivo de log com escrita constante no seu computador ou servidor e execute, por exemplo: `$ tail -f /var/log/syslog`.

## pipelines

Acho que esse usamos e acabamos nem percebendo, quem nunca viu algum projeto livre por ai que o primeiro comando não é um `curl` ou `wget` para fazer download de uma aquivo, um `|` e `bash`. Então, esse `|` é o `pipelines`. O que esse comando fazer é conectar uma cadeia de comandos:

```
comando1 | comando2 | comando3 | ... | comandoN
```

O que estamos fazendo ai é que o `comando1` vai gerar uma saída que vai ser entrada do `comando2` que por consequência vai gerar uma saída nova que será a entrada do `comando3` e assim até o `comandoN`. No próximo item, `jq`, vamos explorar melhor isso na prática.

### Bônus

Existe uma variação do comando, `|&`, que é para redirecionar as saídas de erro.

## jq

É uma ferramenta bem legal feita em Node para lidarmos com JSON/string no terminal. Aqui não vamos explorar muito ela pois acho que isso pode ser um artigo só para isso de tantas coisas interessantes que ele tem. Vamos supor o seguinte JSON de uma compra em e-commerce:

```json
{
    "id": 12345,
    "cliente": {
        "id": 3243,
        "nome": "joão da silva"
    },
    "estado": "aguardando pagamento",
    "valor_total": 300.0,
    "carrinho": {
        "valor": 300.0,
        "produtos": [
            {
                "id": 123,
                "nome": "livro do dragão",
                "quantidade": 1,
                "valor_unitario": 150.0
            },
            {
                "id": 312,
                "nome": "copos",
                "quantidade": 10,
                "valor_unitario": 10.0
            },
            {
                "id": 213,
                "nome": "camisa preta",
                "quantidade": 1,
                "valor_unitario": 50.0
            }
        ]
    },
    "entrega": {
        "endereco": {
            "rua": "rua x",
            "numero": 123,
            "complemento": "apt 101",
            "cidade": "vitoria",
            "estado": "es",
            "cep": "29000000"
        },
        "valor": 0.0
    },
    "pagamento": {
        "tipo": "boleto",
        "codigo_barra": "23790504004200050320330008109206182470000019900"
    }
}
```

### Nome do cliente

```
$ cat compra.json | jq .cliente.nome
"joão da silva"
```

### Listar produtos

```
$ cat compra.json | jq .carrinho.produtos
[
  {
    "id": 123,
    "nome": "livro do dragão",
    "quantidade": 1,
    "valor_unitario": 150
  },
  {
    "id": 312,
    "nome": "copos",
    "quantidade": 10,
    "valor_unitario": 10
  },
  {
    "id": 213,
    "nome": "camisa preta",
    "quantidade": 1,
    "valor_unitario": 50
  }
]
```

### Nome dos produtos

```
$ cat compra.json | jq .carrinho.produtos[].nome
"livro do dragão"
"copos"
"camisa preta"
```

### Valor da entrega

```
$ cat compra.json | jq .entrega.valor
0
```

### Quantidade de itens (Bônus)

```
$ cat compra.json | jq -n '[inputs | .carrinho.produtos[].quantidade] | reduce .[] as $num (0; .+$num)'
12
```

## grep

O `grep` é uma comando para procurar padrões em arquivos e imprimir suas ocorrências por linha. Vamos explorar o uso `grep` no arquivo: `/etc/sysctl.conf`.

### Todas configurações presentes relativas a IPv4

```
$ grep ipv4 /etc/sysctl.conf
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1
#net.ipv4.tcp_syncookies=1
#net.ipv4.ip_forward=1
#net.ipv4.conf.all.accept_redirects = 0
# net.ipv4.conf.all.secure_redirects = 1
#net.ipv4.conf.all.send_redirects = 0
#net.ipv4.conf.all.accept_source_route = 0
#net.ipv4.conf.all.log_martians = 1
```

### Todas as configurações ativas

Sabendo que `#` no começo da linha significa que a linha está comentada.

```
$ grep -v ^# /etc/sysctl.conf












fs.inotify.max_user_watches=524288 # nao comentada
```

Várias linhas em branco e nota-se que a última não foi pega no `grep` pois o `^` indica começo da linha, como o `#` está no meio da linha o casamento de padrão é falso.

### Bônus

```
$ grep -i --color -e 'ipv[46]' /etc/sysctl.conf
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1
# Note: This may impact IPv6 TCP sessions too
#net.ipv4.tcp_syncookies=1
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
# Uncomment the next line to enable packet forwarding for IPv6
#net.ipv6.conf.all.forwarding=1
#net.ipv4.conf.all.accept_redirects = 0
#net.ipv6.conf.all.accept_redirects = 0
# net.ipv4.conf.all.secure_redirects = 1
#net.ipv4.conf.all.send_redirects = 0
#net.ipv4.conf.all.accept_source_route = 0
#net.ipv6.conf.all.accept_source_route = 0
#net.ipv4.conf.all.log_martians = 1
```

Recomendo executar para ver a opção `--color` funcionando e entender melhor a `-i`.

## sort

Como próprio nome sugere, o `sort` vai ordenar uma entrada informada por linhas. Vamos tomar o seguinte arquivo para exemplo:

```
$ cat ordenar.txt
zambia
nigeria
áfrica do sul
moçambique
mandagascar
93213
123123
13
3215
123
!@#!@#
5$%$%

~`''`
```

Você acha que conseguiria ordenar apenas no olhar de forma igual?

```
$ sort ordenar.txt

~`''`
!@#!@#
123
123123
13
3215
5$%$%
93213
áfrica do sul
mandagascar
moçambique
nigeria
zambia
```

### Bônus

Considerando o seguinte arquivo:

```
$ cat colunas.txt
9 banana
8 maçã
7 abacate
6 uva
5 laranja
4 pêra
3 pokan
2 acerola
1 cajá
```

Entenda o comportamento de `sort -k1 colunas.txt` e `sort -k2 colunas.txt`.

## uniq

O `uniq` pelo nome já da sinais do que faça, o que ele faz é informar ou omitir linhas repetidas, vamos considerar o seguinte arquivo:

```
$ cat uniq.txt
zzzz
bbbb
zzzz
ffff
jjjj
kkkk
aaaa
zzzz
oooo
eeee
bbbb
mmmm
cccc
```

Fazendo o teste mais óbvio com o `uniq`:

```
$ uniq uniq.txt
zzzz
bbbb
zzzz
ffff
jjjj
kkkk
aaaa
zzzz
oooo
eeee
bbbb
mmmm
cccc
```

O mesmo retorno! Vamos ler a documentação na `man pages`:

> Filter adjacent matching lines from INPUT (or standard input), writing to OUTPUT (or standard output).

Então, para ser efetivo no uso do `uniq` precisamos de usar o sort antes, vejamos:

```
$ sort uniq.txt | uniq
aaaa
bbbb
cccc
eeee
ffff
jjjj
kkkk
mmmm
oooo
zzzz
```

Agora sim conseguimos ver o uso efetivo do `uniq`.

### Vendo apenas os dados repetidos

```
$ sort uniq.txt | uniq -d
bbbb
zzzz
```

### Vendo apenas os únicos

```
$ sort uniq.txt | uniq -u
aaaa
cccc
eeee
ffff
jjjj
kkkk
mmmm
oooo
```

### Bônus

Contanto as ocorrências:

```
$ sort uniq.txt | uniq -c
      1 aaaa
      2 bbbb
      1 cccc
      1 eeee
      1 ffff
      1 jjjj
      1 kkkk
      1 mmmm
      1 oooo
      3 zzzz
```

## awk

Acho que melhor do que criar uma explicação do que é o `awk`, preferi pegar a descrição do [Wikipedia](https://pt.wikipedia.org/wiki/AWK) que diz tudo:

> A linguagem de programação AWK foi criada em 1977 pelos cientistas Alfred Aho, Peter J. Weinberger e Brian Kernighan no laboratório Bell Labs. A palavra AWK é uma abreviatura das iniciais dos sobrenomes dos criadores da linguagem (Aho, Weinberger e Kernighan).
>
> A linguagem é interpretada linha por linha e tem como principal objetivo deixar os scripts de Shell em sistemas POSIX mais poderosos e com muito mais recursos sem utilizar muitas linhas de comando, podendo resolver infinidades de problemas do dia-a-dia do desenvolvedor nesses sistemas operacionais.
>
> Baseada na linguagem C, é utilizada frequentemente por desenvolvedores para processar textos e manipular arquivos. Tem como os paradigmas linguagem de script, procedural e orientada a eventos.
>
> Esta linguagem é considerada por muitos um importante marco para história da programação, tendo tido bastante influência na criação de outras linguagens de programação, como Perl e Lua.

Então, sabendo do tamanho do universo que é o `awk` é claro que esse artigo não vai cobrir ele e vamos ser pontuais no que precisamos para resolver o nosso desafio de analisar os logs.

Vamos considerar o seguinte arquivo:

```
$ cat awk1.txt
colunaA colunaB colunaC
1       a       !
2       b       @
3       c       #
4       d       $
5       e       %
6       f       &
```

### Retornando a entrada

```
$ awk '{print($0)}' awk1.txt
colunaA colunaB colunaC
1       a       !
2       b       @
3       c       #
4       d       $
5       e       %
6       f       &
```

### Retornando os dados isolados por coluna

```
$ awk '{print($1)}' awk1.txt
colunaA
1
2
3
4
5
6
```

```
$ awk '{print($2)}' awk1.txt
colunaB
a
b
c
d
e
f
```

```
$ awk '{print($3)}' awk1.txt
colunaC
!
@
#
$
%
&
```

### Usando o `split`

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); print(l[0]) }'
```

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); print(l[1]) }'
0
```

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); print(l[1]) }'
0
```

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); print(l[10]) }'
9
```

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); print(l[11]) }'

```

Fique atento, o `awk` não da erro ao acessar uma posição do `array` que não possui dados.

### Bônus

```
$ echo "0,1,2,3,4,5,6,7,8,9" | awk '{ split($0,l,","); c = l[3]; print c; if (c < 2) { print("a") } else if (c == 2) { print("b") } else print("c") }'
```

Qual é a resposta?

# Gerador aleatório de logs

Para demonstrar o uso combinado das ferramentas era preciso gerar um data set de logs. Hoje, é bem comum boa parte das aplicações escrevem seus logs no formato de JSON e esse era o meu caso. Então, no nosso arquivo de log vai conter apenas um JSON, por exemplo:

```
{"remote_ip_addr": "82.58.153.167", "access_token": "secret_a7f49337ecff4c50a97c3d23e7386e23", "x_tid": "d8d3d5b0-bd8d-4386-b381-c760a70b9f49", "user_agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)", "method": "GET", "path": "/users/1", "timestamp": "2020-04-30 03:58:21", "status_code": 201}
{"remote_ip_addr": "188.235.234.165", "access_token": "secret_df60122168e048ba846b81fed1b09a1d", "x_tid": "d60291e2-e5a2-4222-a038-67f4adc88012", "user_agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)", "method": "DELETE", "path": "/robots.txt", "timestamp": "2020-04-30 03:58:45", "status_code": 200}
{"remote_ip_addr": "51.203.201.123", "access_token": "secret_b36d6af8a0d542f1ae5be86ac38688a0", "x_tid": "74016c10-be59-45ff-b13c-ffd8732919d8", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36", "method": "DELETE", "path": "/users/12", "timestamp": "2020-04-30 03:58:48", "status_code": 400}
{"remote_ip_addr": "59.1.220.255", "access_token": "secret_2eb5ec132dac4fecaeb32ef12cfd7cd8", "x_tid": "64659fa4-f9d6-4e10-98d5-fb5178a2cd1c", "user_agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)", "method": "PUT", "path": "/security.txt", "timestamp": "2020-04-30 03:58:57", "status_code": 500}
{"remote_ip_addr": "199.18.137.169", "access_token": "secret_289c094aa4784d60987873f914b326a6", "x_tid": "af1d4979-de7e-4f9d-95f5-9de310d570b7", "user_agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)", "method": "PUT", "path": "/health", "timestamp": "2020-04-30 03:59:33", "status_code": 301}
{"remote_ip_addr": "24.109.83.239", "access_token": "secret_e34f7166be1f40c2be3754aafffe334b", "x_tid": "47cfaca0-0345-441a-bb03-670285f29854", "user_agent": "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)", "method": "POST", "path": "/robots.txt", "timestamp": "2020-04-30 04:00:01", "status_code": 500}
{"remote_ip_addr": "62.112.131.39", "access_token": "secret_b3351fdeb0cb40279c6d1f48febf5838", "x_tid": "60b8714d-4cdc-4948-932c-19f594b8e8eb", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36", "method": "GET", "path": "/robots.txt", "timestamp": "2020-04-30 04:00:21", "status_code": 301}
{"remote_ip_addr": "50.243.130.83", "access_token": "secret_f9c8f3495f324707b2a49878ce253844", "x_tid": "7ab5d9d4-2629-423c-b9b7-309fcab1958b", "user_agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)", "method": "POST", "path": "/", "timestamp": "2020-04-30 04:00:39", "status_code": 500}
{"remote_ip_addr": "17.180.155.70", "access_token": "secret_bae6bfd17dbc4ec9b7c9d6b2974f9ef6", "x_tid": "4a3beff7-bc2c-4535-bba0-9b9906f846bc", "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:53.0) Gecko/20100101 Firefox/53.0", "method": "GET", "path": "/users/35", "timestamp": "2020-04-30 04:00:54", "status_code": 400}
```

Entrando nos detalhes dos campos do log do nosso serviço acima:
- `remote_ip_addr`: uma string com endereço IP da origem da requisição;
- `access_token`: o token usado para acessar a API;
- `x_tid`: um ID para identificar a requisição;
- `user_agent`: ferramenta de onde partiu a requisição;
- `method`: método HTTP usado na requisição;
- `path`: caminho no meu serviço onde chegou a requisição
- `timestamp`: instante no tempo que a requisição chegou; e
- `status_code`: código HTTP de retorno dado pelo serviço.

## Código do gerador de log

```python
import random
import socket
import struct
import json
import uuid
import datetime
from random import randrange


def generate_ip() -> str:
    return socket.inet_ntoa(struct.pack('>I', random.randint(1, 0xffffffff)))

IP_DATASET_SIZE = 100
IP_DATASET = [generate_ip() for _ in range(IP_DATASET_SIZE)]
LOG_SIZE = 1000
USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:53.0) Gecko/20100101 Firefox/53.0",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.79 Safari/537.36 Edge/14.14393",
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)",
    "Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)",
    "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)",
    "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)",
    "curl/7.35.0",
    "Wget/1.15 (linux-gnu)"
]
STATUSES = [200, 201, 301, 400, 401, 500]
PATHS = [
    "/",
    "/health",
    "/security.txt",
    "/robots.txt",
    "/users",
    "/users/1",
    "/users/12",
    "/users/35",
    "/users/100",
    "/admin/users"
]
METHODS = [
    "GET",
    "POST",
    "PUT",
    "DELETE"
]

dt = datetime.datetime(2020, 4, 30, 0, 0)
for i in range(LOG_SIZE):
    random_ip_pos = randrange(-1, IP_DATASET_SIZE, 1)
    random_user_agent_pos = randrange(-1, len(USER_AGENTS), 1)
    random_status_pos = randrange(-1, len(STATUSES), 1)
    random_path_pos = randrange(-1, len(PATHS), 1)
    random_method_pos = randrange(-1, len(METHODS), 1)
    payload = {
        "remote_ip_addr": IP_DATASET[random_ip_pos],
        "access_token": "secret_" + uuid.uuid4().hex,
        "x_tid": str(uuid.uuid4()),
        "user_agent": USER_AGENTS[random_user_agent_pos],
        "method": METHODS[random_method_pos],
        "path": PATHS[random_path_pos],
        "timestamp": dt.strftime("%Y-%m-%d %H:%M:%S"),
        "status_code": STATUSES[random_status_pos],
    }
    dt += datetime.timedelta(seconds=randrange(randrange(1, 60, 1)))
    print(json.dumps(payload))
```

# Analisando os logs

## Contando a quantidade de status_code

```
$ python generate.py | jq .status_code | sort | uniq --count
  14257 200
  14179 201
  14411 301
  14360 400
  14069 401
  28724 500
```

## Rank do user-agents

```
$ python generate.py | jq .user_agent | sort | uniq --count | sort -k1 -n -r
  20113 "Wget/1.15 (linux-gnu)"
  10030 "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)"
  10008 "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
   9993 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.79 Safari/537.36 Edge/14.14393"
   9992 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"
   9988 "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:53.0) Gecko/20100101 Firefox/53.0"
   9988 "curl/7.35.0"
   9963 "Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)"
   9925 "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)"
```

## Descobrindo se teve conflitos de X-TID

```
$ python generate.py | jq .x_tid | sort | uniq --count | awk '{ if ($1 > 1) print($0) }'
$ echo $?
0
```

Como não teve retorno e a execução foi um sucesso, significa que não teve conflitos de X-TID.

## Rank das classes de IP

Primeiro, vamos entender a decomposição decimal do IP, 172.16.254.1, por exemplo.

```
172 = 10101100 (8 bits)
16  = 00010000 (8 bits)
254 = 11111110 (8 bits)
1   = 00000001 (8 bits)

total de 32 bitss (4 bytes)
```

Agora, quais são as classes de IP:
- Classe A: Primeiro bit é *0* (zero)
    - começa: 0.0.0.0
    - termina: 127.255.255.255
- Classe B: Primeiros dois bits são *10*
    - começa: 128.0.0.0
    - termina: 191.255.255.255
- Classe C: Primeiros três bits são *110*
    - começa: 192.0.0.0
    - termina: 223.255.255.255
- Classe D: Primeiros quatro bits são: *1110*
    - começa: 224.0.0.0
    - termina: 239.255.255.255
- Classe E: Primeiros quatro bits são *1111*
    - começa: 240.0.0.0
    - termina: 255.255.255.254

Detalhes sobre as classes de IP [aqui](https://pt.wikipedia.org/wiki/Endere%C3%A7o_IP#Classes_de_endere.C3.A7os).

```
$ python generate.py | jq '.remote_ip_addr|split(".")[0]|tonumber' | awk '{ if ($1 <= 127) { print "A"; } else if ($1 <= 191) { print "B"; } else if ($1 <= 223) { print "C"; } else if ($1 <= 239) { print "D"; }  else { print "E"; } }' | sort | uniq -c | sort -k2
  49540 A
  23552 B
  10068 C
   8991 D
   7849 E
```

# Conclusão

O terminal e suas ferramentas são extremamante poderosos mas é preciso conhecer e praticar para ter produtividade além de muito Google e Stack Overflow. Quais ferramentas que você usa e como?
