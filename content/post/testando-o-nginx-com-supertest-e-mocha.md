---
author: "João Maia"
date: 2020-03-18
title: "Testando o NGINX com SuperTest e Mocha"
tags: [
    "devops",
    "teste",
    "nginx",
    "supertest",
    "node"
]
categories: [
    "devops",
    "testes"
]
thumbnail: "img/testando-o-nginx-com-supertest-e-mocha/nginx-logo.png"
---

Olá,

testar a infraestrutura não é um sonho mas sim uma realidade, se você ainda não faz nada a respeito saiba que já está atrasado. Com a transformação da infraestrutura em código acabando com os processos manuais e nem sempre documentados é preciso também trazer as boas práticas do mundo de desenvolvimento de software para a infraestrutura. Graças ao Docker subir uma representação da sua infraestrutura ficou mais fácil realizar esse processo apesar de ainda não ser 100% fiel ao ambiente de produção.

Muitos já devem ter usado ou usam o [NGINX](https://www.nginx.com/) como balanceador de carga da sua infraestrutura ou até para outros recursos mais avançados que ele permite com suas integrações de plugins, por exemplo, o projeto [OpenResty](https://openresty.org/en/). Se o seu NGINX tem diversos recursos e configurações particulares é preciso que você tenha controle disso e apenas comentar na configuração do arquivo talvez não seja o suficiente caso se lembre de fazer isso. Com testes conseguimos garantir melhor o entendimento e propósito dessas configurações. Então, como escrever testes para isso?

Procurando alguma solução para execução de testes de chamadas HTTP acabei encontrando como opção o [SuperTest](https://github.com/visionmedia/supertest) que é feito em JavaScript e podemos facilmente executar no nosso ambiente de integração contínua com [Node.js](https://nodejs.org/en/). Apesar de, à primeira, vista ele ser um framework voltado ao [Express.js](https://expressjs.com/pt-br/), ele pode ser executado para testar qualquer aplicação Web sem se importar com o que seja, examente o que precisamos.

Além disso, precisamos de alguma ferramenta que vai gerenciar as execuções dos testes e fazer as asserções e gerar o relatório final, nessa caso, escolhi o [Mocha](https://mochajs.org/) que já tinha familiaridade trabalhando em projetos de Node.js no passado. Quem já programou em [Ruby](https://www.ruby-lang.org/pt/) e escreveu testes com [RSpec](https://rspec.info/) vai achar bem parecido.

Como forma de validar essa solução fiz um pequeno [projeto](https://github.com/jvrmaia/nginx-tests-with-supertest) para validação das ferramentas e conceito do que buscava. No desenho abaixo mostro como vai funcionar a estrutura do projeto. Explicando os componentes:
- Test runner: é o Docker que vai rodar a bateria de testes com o SuperTest;
- Gateway: é o NGINX (Docker) que queremos testar que está fazendo um papel de proxy reverso; e
- Echo: uma aplicação que responde uma resposta padrão para simular um backend.

```
+-----------------+              +-----------------+                   +------------------+
|                 |              |                 |                   |                  |
|  TEST           |              |                 |                   |                  |
|                 +--------------> GATEWAY (NGINX) +-------------------> ECHO (BACKEND)   |
|  RUNNER         |              |                 |                   |                  |
|                 |              |                 |                   |                  |
+-----------------+              +-----------------+                   +------------------+
```

# Infraestrutura dos testes

Agora que já sabemos a estutura da infraestrutura que vamos testar precimos tornar ela real. A escolha foi usar [docker-compose](https://docs.docker.com/compose/) que é uma ferramenta muito usada em ambiente de desenvolvimento hoje para subir o ambiente completo:

```yaml
version: '3.2'

services:
  gateway:
    hostname: gateway
    build:
      context: ./gateway
      dockerfile: Dockerfile
    depends_on:
      - echo
    links:
      - echo
    ports:
      - 8080:80
    networks:
      - fake-vpc

  echo:
    hostname: echo
    image: hashicorp/http-echo
    command: -listen=:3000 -text="hello world"
    ports:
      - 3000:3000
    networks:
      - fake-vpc

  test-runner:
    build:
      context: ./tests
      dockerfile: Dockerfile
    environment:
      - GATEWAY_HOST=gateway
    depends_on:
      - gateway
    links:
      - gateway
    networks:
      - fake-vpc

networks:
  fake-vpc:
```

Não temos novidades aqui para quem já está acostumado com o uso, declaramos nossos 3 serviços e fizemos as configurações devidas de rede para resolverem por nome dentro da rede virtualizada pelo Docker e exportamos as portas necessárias para testar de forma local o SuperTest ou simplesmente um `curl`.

# Gateway

O nosso gateway não tem muitas customizações, basicamente adicionei o proxy reverso para o `echo` e adicionei uma configuração para `gzip`:

```nginx
gzip  on;
```

Adicionando o `echo`:

```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    location /echo {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://echo:3000;
    }

    // omitido o restante
}
```

# Os testes

Não tem muito mistério, primeiro você tem que inicializar o executor do SuperTest para fazer as chamadas HTTP no Gateway e por fim descrever os testes usando o Mocha:

```javascript
'use strict';

const request = require('supertest');
const assert = require('assert');

const GATEWAY_HOST = process.env.GATEWAY_HOST || 'http://localhost:8080'
const runner = request(GATEWAY_HOST);


describe('Gateway tests', () => {

    it('Root path must return HTTP/200', (done) => {
        runner
            .get('/')
            .expect(200)
            .end(done);
    });

    it('Non existis path must return HTTP/404', (done) => {
        runner
            .get('/404')
            .expect(404)
            .end(done);
    });

    describe('Headers configurations', (done) => {

        it('Server must be NGINX', (done) => {
            runner
                .get('/')
                .expect((res) => {
                    assert.equal(res.header.server, 'nginx/1.17.9');
                })
                .end(done);
        });

        it('Content-Type must be HTML', (done) => {
            runner
                .get('/')
                .expect((res) => {
                    assert.equal(res.header['content-type'], 'text/html');
                })
                .end(done);
        });

        it('Connection must be closed', (done) => {
            runner
                .get('/')
                .expect((res) => {
                    assert.equal(res.header.connection, 'close');
                })
                .end(done);
        });

        it('Content encoding must be gzip', (done) => {
            runner
                .get('/')
                .expect((res) => {
                    assert.equal(res.header['content-encoding'], 'gzip');
                })
                .end(done);
        });

    });

    describe('Application path', (done) => {

        it('/echo must return to application', (done) => {
            runner
                .get('/echo')
                .expect((res) => {
                    assert.equal(res.text, 'hello world\n');
                })
                .end(done);
        });

    })

});
```

Olhando em detalhes para entender melhor:

```javascript
it('Content encoding must be gzip', (done) => {
    runner
        .get('/')
        .expect((res) => {
            assert.equal(res.header['content-encoding'], 'gzip');
        })
        .end(done);
});
```

O nosso executor, no caso a variável `runner`, executa uma chamad HTTP/GET na raíz do nosso Gateway e de resultado esperamos que o `Content-Encoding` seja `gzip` como bem definimos no arquivo de configuração e por fim fechamos a `Promise` do teste com o `.end(done)`. Detalhe importante, lembre-se que as chaves dos valores do `Header` não são sensíveis ao caso, o que isso significa, você pode enviar sua requisição escrito `Content-Encoding` ou `content-encoding` que não faz diferença, são tratados iguais. Afim de evitar confusão, o SuperTest padroniza tudo no minúsculo.

# Colocando no CI

Tenho usado o Circle CI para isso pois é fácil o uso e para projetos abertos não tem custo e usamos ele na empresa. Acho ele bem completo para ajudar nisso mas acho que falta melhorar na parte de granularidade de acessos e permissões da organização e projetos. Aproveitando a brincadeira coloquei de quebra um teste no CI para ver ser o Dockerfile tá seguindo as boas práticas também com o [Hadolint](https://github.com/hadolint/hadolint):

```yaml
version: 2.1

jobs:
    build:
        machine: true
        steps:
            - checkout

            - run:
                name: Dockerfile lint 'gateway'
                command:
                    docker run --rm -i hadolint/hadolint < ./gateway/Dockerfile

            - run:
                name: Test 'gateway' image
                command: |
                    docker-compose up -d echo gateway
                    docker-compose up test-runner
```

Será que funcionou? Veja a imagem do CI:

![Circle CI](/img/testando-o-nginx-com-supertest-e-mocha/circleci.png)

Acho que era isso que queria passar nesse artigo, até o próximo.
