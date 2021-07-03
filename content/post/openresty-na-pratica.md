---
author: "João Maia"
date: 2021-07-02
title: "OpenResty na prática"
tags: [
    "openresty",
    "nginx",
    "lua"
]
categories: [
    "desenvolvimento",
    "infraestrutura"
]
thumbnail: "img/openresty-na-pratica/openresty.png"
---

Olá,

[OpenResty](https://openresty.org/en/) é uma plataforma completa para trabalhar como uma primeira camada na sua infraestrutura de aplicações fazendo um papel de [Proxy ou Load Balancer](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/). O projeto é um fork do [NGINX](https://nginx.org/en/) e evoluido com o uso do [LuaJIT](https://luajit.org/) para enriquecer a plataforma de entregar uma solução completa. Caso venha a se interessar no projeto e tenha interesse em recursos mais avançados existe uma [empresa](https://openresty.com/en/) por trás do projeto que vende soluções pagas da ferramenta com mais funcionalidades.

# Hello World

Como o foco aqui é ir na prática vamos começar pelo exemplo mais trivial da programação que é o "Hello World".

## Arquivos

### docker-compose.yml

```yaml
version: '3.2'

services:
  openresty:
    build: .
    ports:
      - 8080:8080
```

### Dockerfile 

```Dockerfile
FROM openresty/openresty:1.19.3.2-1-alpine

LABEL maintener="joão maia <joao@joaovrmaia.com>"
LABEL project="openresty-na-pratica"

COPY src/nginx/nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY src/nginx/server.conf /usr/local/openresty/nginx/conf/server.conf
COPY src/lua /opt/app

EXPOSE 8080
```

### nginx.conf

```bash
worker_processes  1;
error_log logs/error.log;

events {
    worker_connections 1024;
}


http {
	include server.conf;
}
```

### server.conf

```bash
server {
	listen 8080;
	location / {
		default_type text/html;
		content_by_lua_block {
			ngx.say("<p>hello, world</p>")
		}
	}
}
```

Aqui é a única parte que tem algo de diferente para quem já usa nginx pois tem esse bloco que conseguimos invocar um bloco de código em Lua.

## Executando

```
$ docker-compose up -d
$ curl -i localhost:8080
HTTP/1.1 200 OK
Server: openresty/1.19.3.2
Date: Fri, 02 Jul 2021 22:40:58 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive

<p>hello, world</p>
```

## Refatorando

Escrever seu código Lua dentro de um arquivo do NGINX não é muito produtivo e gera problemas para criar testes então vamos refatorar o projeto para isolar o código em Lua em um arquivo:

### nginx.conf

```bash
worker_processes  1;
error_log logs/error.log;

events {
    worker_connections 1024;
}


http {
    lua_package_path '/opt/app/?.lua;;';
	include server.conf;
}
```

Adiciona-se uma entrada para dizer onde estão os arquivos Lua.

###  server.conf

```bash
server {
	listen 8080;
	location / {
		default_type text/html;
		content_by_lua_block {
			require('app').hello_world()
		}
	}
}
```

Importa o módulo e executa sua função exportada.

# app.lua

```lua
_M = {}

function _M.hello_world() 
    ngx.say("<p>hello, world</p>")
end

return _M
```

# Contador de requisições

Vamos agora fazer algo mais além de apenas imprimir algo fixo e vamos retorar a quantidade de requisições recebidas pelo seu servidor. Para isso vamos mudar apenas nosso arquivo Lua do projeto anterior:

## app.lua

```lua
_M = {}

counter = 0

function _M.hello_world() 
    counter = counter + 1;
    ngx.say(counter);
end

return _M
```

## Executando

```
$ for i in `seq 1 10`; do curl localhost:8080; done
1
2
3
4
5
6
7
8
9
10
```

Interessante, não?

# Conclusão

Como podemos ver é bem fácil e simples de começar a usar Lua no NGINX com o OpenResty mas ai está longe de ter tudo que o OpenResty pode nos oferecer. Por isso, recomendo fortemente que olhe no site deles para ver se tem algo mais que te interesse e no Github deles, pois tem projetos interessantes para turbinar seu Proxy ou Load Balancer como limitador de requests.

Vou no futuro voltar com mais textos sobre o assunto pois tive uma experiência bem interessante lidando com um projeto real usando essa tecnologia em produção e tendo a oportunidade na prática de aprender Lua que só conhecia de nome.

# Referências

- Sife oficial do OpenResty: https://openresty.org/en/
- "What is a Reverse Proxy vs. Load Balancer?": https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/
- Site oficial do NGINX: https://nginx.org/en/
- LuaJIT: https://luajit.org/
- Empresa do OpenResty: https://openresty.com/en/