---
author: "João Maia"
date: 2020-03-07
title: "Verificando vulnerabilidades com Clair e Circle CI"
tags: [
    "docker",
    "clair",
    "circleci",
    "devops",
    "devsecops",
    "segurança",
]
categories: [
    "devops",
    "devsecops",
    "segurança"
]
thumbnail: "img/verificando-vulnerabilidades-com-clair-e-circleci/clair.png"
---

Olá,

nesse artigo vou compartilhar de forma breve a minha experiência de configurar uma ferramenta de análise estática de vulnerabilidades em um projeto usando [Clair](https://github.com/quay/clair) e [Circle CI](https://circleci.com/). Na jornada de adicionar mais segurança no processo de entregas de nossas aplicações assim como fazemos análises estáticas em nosso código podemos e devemos também fazer na nossa infraestrutura. Com essa necessidade acabou até surgindo um novo termo na área que é o DevSecOps, ou seja, adicionando segurança (Sec de sercurity) no DevOps.

# Clair

O Clair é um dos projetos no escopo na [Cloud Native Computing Foundation](https://www.cncf.io/) (CNCF) e se encontra no área [Security & Compliance](https://landscape.cncf.io/selected=clair).

Descrição do projeto:

> Clair is an open source project for the static analysis of vulnerabilities in application containers (currently including appc and docker).

Objetivos do projeto e origem do nome:

> Our goal is to enable a more transparent view of the security of container-based infrastructure. Thus, the project was named Clair after the French term which translates to clear, bright, transparent.

# Circle CI

![Circle CI](/img/verificando-vulnerabilidades-com-clair-e-circleci/circleci.png)

O Circle CI é uma empresa para criação de fluxos de projetos para integração, entrega e deploy contínuos. [Aqui](https://circleci.com/docs/2.0/about-circleci/#section=welcome) você pode ler mais sobre o que o Circle Ci faz, segue uma citação do site deles:

> CircleCI allows teams to rapidly build quality projects, at scale. Our mission is to give people everywhere the power to build and deliver software at the speed of imagination.

# O projeto

Quando esbarro em dúvidas em alguma entrega no trabalho e preciso explorar alguns recursos com mais agilidade ou buscar apoio na comunidade tenho por hábito por fazer algum projeto aberto no meu Github para isso. Então, para testar o Clair e Circle CI peguei um projeto antigo meu que usei para ajudar uma pessoa a como rodar [Flask em Docker](https://github.com/jvrmaia/flask-docker-example). O código não tem nada de complexo, apenas executa uma aplicação Flask que tem apenas um recurso que informa o nome da máquina e versão do projeto.

```python
import socket
from flask import Flask
from decouple import config

app = Flask(__name__)

VERSION = config('VERSION', 'dev', cast=str)

@app.route("/")
def index():
    return socket.gethostname() + " - " + VERSION

if __name__ == '__main__':
    app.run(debug=True,host='0.0.0.0')
```

A criação do Docker dele é bem simples também:

```Dockerfile
FROM python:3.8

LABEL maintainer "Joao Maia <joao@joaovrmaia.com>"

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt

ENV VERSION=0.0.1

EXPOSE 5000

ENTRYPOINT ["python"]

CMD ["app.py"]
```

Como podemos ver, é bem simples e sem grandes riscos de segurança já que usamos uma imagem Docker oficial do Docker Hub, né?

# Configurando o Circle CI

O Circle CI é bem simples de configurar, você primeiro precisa conectar seu Github no Circle CI para ele conseguir criar as chaves no seu repositório para se conectar. Feito isso, você ainda nele precisa ativar o repositório para ser executado pelo Circle CI, por fim criar o arquivo no seu projeto `.circleci/config.yml`. No meu caso, o arquivo ficou assim:

```yaml
version: 2.1

orbs:
    clair-scanner: ovotech/clair-scanner@1.6.0

jobs:
    build:
        machine: true
        steps:
            - checkout

            - run:
                name: Build flask-docker-example image
                command: docker build -t jvrmaia/flask-docker-example:latest .

            - run:
                name: Login Docker Hub
                command: docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_PASS

            - run:
                name: Push image to Docker Hub
                command: docker push jvrmaia/flask-docker-example:latest

    scan_images:
        executor: clair-scanner/default
        steps:
            - clair-scanner/scan:
                image: jvrmaia/flask-docker-example:latest

workflows:
    version: 2
    build_and_scan:
        jobs:
            - build
            - scan_images
```

Nesse caso estou usando 2 variáveis de ambientes no Circle CI que você deve configurar no seu também: `$DOCKER_HUB_USER` e `$DOCKER_HUB_PASS`. Além disso, é preciso ir nas configurações da sua organização e habilitar a opção de segurança de executar `orbs` de terceiros pois o `ovotech/clair-scanner@1.6.0` não é um `orb` oficial do Circle CI.

# Resultados

![Build falhando](/img/verificando-vulnerabilidades-com-clair-e-circleci/circleci-fail.png)

Pois é, falhou! Pelo visto mesmo a imagem sendo oficial não podemos confiar nela para evitar problemas de segurança em nossas aplicações. Sobre os problemas encontrados:

```
Status: Downloaded newer image for *******/flask-docker-example:latest
docker.io/*******/flask-docker-example:latest
2020/03/07 10:19:57 [INFO] ▶ Start clair-scanner
2020/03/07 10:20:12 [INFO] ▶ Server listening on port 9279
2020/03/07 10:20:12 [INFO] ▶ Analyzing 833c6daa5e441f05bf0861abf69385f4336a84579e106a4e10df96c16345238c
2020/03/07 10:20:13 [INFO] ▶ Analyzing 4841da9e53306739896736018799725127bf022debe92a6fba5849e462df6d7c
2020/03/07 10:20:14 [INFO] ▶ Analyzing 74a44e872bf076416afabf8c6ad9a44267469fcf7406ca59efd971b9f36ec2e0
2020/03/07 10:20:14 [INFO] ▶ Analyzing 08956d8cfb4a5c3911c05bbcccdee8ea6c5ee78e2a557705b186d093e10e3c8d
2020/03/07 10:20:14 [INFO] ▶ Analyzing 2418f849fdd6aa84613bf622029f9c13f8af63369b1bfb9fe37d1c91c5f8014b
2020/03/07 10:20:15 [INFO] ▶ Analyzing 665107ae66eecfa7a5ad25f3a3fea690a7b39d947435aefafb73d2276f6b1e1a
2020/03/07 10:20:15 [INFO] ▶ Analyzing 81fbbe995a7a6c8de2069875be5441038ec5d6f7b79048a2ffc6ab48e2ac5c61
2020/03/07 10:20:15 [INFO] ▶ Analyzing ce041cd84cdf830252387144de7b13594132831baf70deb81b9db077e2394b16
2020/03/07 10:20:15 [INFO] ▶ Analyzing c3bc4598a93d965a92519a886c6ec7e8ad05a73bb59da5e12dc091fe6de90e05
2020/03/07 10:20:15 [INFO] ▶ Analyzing 57ef9d9cfc8a93dfa86b31274c6327f2d297eef8402742324c6f4e96bceffcba
2020/03/07 10:20:15 [INFO] ▶ Analyzing 8dc77558fa31b6eaf473e1e89ae568041b232a95a31d53d41d66e25fce4f357d
2020/03/07 10:20:15 [WARN] ▶ Image [*******/flask-docker-example:latest] contains 354 total vulnerabilities
2020/03/07 10:20:15 [ERRO] ▶ Image [*******/flask-docker-example:latest] contains 8 unapproved vulnerabilities
```

Um exemplo de parte da lista detalhada:

```
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
| STATUS     | CVE SEVERITY                | PACKAGE NAME    | PACKAGE VERSION           | CVE DESCRIPTION                                                  |
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
| Unapproved | Critical CVE-2019-19816     | linux           | 4.19.98-1                 | In the Linux kernel 5.0.21, mounting a crafted btrfs             |
|            |                             |                 |                           | filesystem image and performing some operations                  |
|            |                             |                 |                           | can cause slab-out-of-bounds write access in                     |
|            |                             |                 |                           | __btrfs_map_block in fs/btrfs/volumes.c, because a value         |
|            |                             |                 |                           | of 1 for the number of data stripes is mishandled.               |
|            |                             |                 |                           | https://security-tracker.debian.org/tracker/CVE-2019-19816       |
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
| Unapproved | Critical CVE-2019-19814     | linux           | 4.19.98-1                 | In the Linux kernel 5.0.21, mounting a crafted f2fs              |
|            |                             |                 |                           | filesystem image can cause __remove_dirty_segment                |
|            |                             |                 |                           | slab-out-of-bounds write access because an                       |
|            |                             |                 |                           | array is bounded by the number of dirty types                    |
|            |                             |                 |                           | (8) but the array index can exceed this.                         |
|            |                             |                 |                           | https://security-tracker.debian.org/tracker/CVE-2019-19814       |
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
| Unapproved | Critical CVE-2019-19813     | linux           | 4.19.98-1                 | In the Linux kernel 5.0.21, mounting a crafted btrfs             |
|            |                             |                 |                           | filesystem image, performing some operations, and then           |
|            |                             |                 |                           | making a syncfs system call can lead to a use-after-free         |
|            |                             |                 |                           | in __mutex_lock in kernel/locking/mutex.c. This is related       |
|            |                             |                 |                           | to mutex_can_spin_on_owner in kernel/locking/mutex.c,            |
|            |                             |                 |                           | __btrfs_qgroup_free_meta in fs/btrfs/qgroup.c, and               |
|            |                             |                 |                           | btrfs_insert_delayed_items in fs/btrfs/delayed-inode.c.          |
|            |                             |                 |                           | https://security-tracker.debian.org/tracker/CVE-2019-19813       |
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
| Unapproved | High CVE-2020-8492          | python3.7       | 3.7.3-2+deb10u1           | Python 2.7 through 2.7.17, 3.5 through 3.5.9, 3.6                |
|            |                             |                 |                           | through 3.6.10, 3.7 through 3.7.6, and 3.8 through 3.8.1         |
|            |                             |                 |                           | allows an HTTP server to conduct Regular Expression              |
|            |                             |                 |                           | Denial of Service (ReDoS) attacks against a client               |
|            |                             |                 |                           | because of urllib.request.AbstractBasicAuthHandler               |
|            |                             |                 |                           | catastrophic backtracking.                                       |
|            |                             |                 |                           | https://security-tracker.debian.org/tracker/CVE-2020-8492        |
+------------+-----------------------------+-----------------+---------------------------+------------------------------------------------------------------+
```

O restante do relatório pode ser visto [aqui](https://gist.github.com/jvrmaia/6dbe0dfa1c193ff3371e2860fd230e5f). Agora, você pega esse relatório e procura o seu time de segurança e pede ajuda! =)

# Conclusão

É muito importante hoje pensar em segurança na entrega de nossas aplicações e não devemos negligenciar isso! Por isso, é muito importante que não se tenham barreiras e exista um bom relacionamento com o time de segurança para que juntos vocês consigam entregar melhores aplicações e seguras.

E nunca confie que você está seguro mesmo seguindo as recomendações de projeto! ;)

# Atualizações

- 07/03/2019: Contribuições do Flávio do Telegram Devops BR
