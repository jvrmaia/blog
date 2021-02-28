---
author: "João Maia"
date: 2020-05-23
title: "Conhecendo o Packer"
tags: [
    "packer",
]
categories: [
    "devops"
]
thumbnail: "img/conhecendo-o-packer/packer.png"
---

Olá,

eventualmente no seu trabalho ou projeto pessoal será preciso subir uma instância de sistema operacional para realizar alguma tarefa ou hospedar algum tipo de serviço, por exemplo, seu computador pessoal não tem um hardware muito robusto e você precisa fazer um processamento pesado nele e opta por criar uma máquina robusta na AWS para executar a tarefa. Hoje, as três principais provedoras de serviços de nuvem oferecem isso com diversos tipos de imagens e suporte para você criar as suas: [AWS](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/AMIs.html), [Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/) e [Google Cloud](https://cloud.google.com/compute/docs/images).

E o que o Packer tem a ver com isso? Vamos a descrição oficial:

> Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image.

Como podemos ver, o Packer vem justamente para você poder criar ou extender essas imagens de sistemas operacionais e como bem dito, isso não é para substituir o Chef, Puppet e Ansible, o Packer e essas ferramentas trabalham juntas para automatizar ainda mais a criação da sua infraestrutura. Não se assuste com Chef, Puppet e Ansible caso não tenha conhecimento ainda, o bom e velho Shell Script também é uma opção.

# Instalando o Packer

O processo de instalação é bem simples, caso prefira seguir a documentação oficial basta ver [aqui](https://www.packer.io/intro/getting-started/). Não vou dar todos os passos mas apenas o direcionamento das formas que podem ser feitas, segue:
- *Instalador do sistema operacional*: se você usa Ubuntu 18.04+ pode instalar via `apt`, acredito que o mesmo seja possível no ambientes `rpm` e `pacman` seja por repositórios oficiais ou alternativos;
- *Homebrew*: caso use essa [ferramenta](https://brew.sh/index_pt-br) pode instalar da mesma forma no Linux ou OSX com o `brew`;
- *Manualmente*: como o Packer é feito em Go, ele se torna um binário simples que pode ser rodado em qualquer ambiente dado uma arquitetura que se tenha o binário. Então, basta fazer o [download desse binário](https://www.packer.io/downloads/) e certificar que ele esteja localizado em algum lugar que seja coberto pelo seu `$PATH`;
- *Docker*: caso você tenha Docker e não tenha interesse de instalar no seu sistema pode usar uma imagem pronta, ver [aqui](https://hub.docker.com/r/hashicorp/packer); e
- *Compilando o fonte*: essa é a forma mais complexa, você pode ir no [repositório oficial](https://github.com/hashicorp/packer) e fazer um clone local para compilar e colocar o binário no seu `$PATH`.

De forma a garantir a reprodutibilidade do artigos vamos usar Docker aqui.

```
$ docker pull hashicorp/packer:1.5.6
1.5.6: Pulling from hashicorp/packer
cbdbe7a5bc2a: Already exists
05cf1cc5a546: Pull complete
1cb511b2b0d4: Pull complete
26885a5064ff: Pull complete
b2e3e25f1b68: Pull complete
d7512d83be57: Pull complete
ad12109a8809: Pull complete
Digest: sha256:013754a8f5f8916dbf225192a8c0f2a55bb7b8851e45138c9f00a4bfaaa955b5
Status: Downloaded newer image for hashicorp/packer:1.5.6
docker.io/hashicorp/packer:1.5.6
$ docker run --rm hashicorp/packer:1.5.6
Usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build       build image(s) from template
    console     creates a console for testing variable interpolation
    fix         fixes templates from old versions of packer
    inspect     see components of a template
    validate    check that a template is valid
    version     Prints the Packer version
```

# Primeiros passos

Com o Packer você pode criar images estáticas para uma diversidade de destinos que na documentação oficial vai encontrar com o nome de [Builders](https://www.packer.io/docs/builders/). Nesse artigo vou me restringir ao ecossistema que já tenho conhecimento que é o uso na [AWS](https://aws.amazon.com/). Então, caso você tenha interesse em reproduzir o que estou abordando nesse artigo faça uma pausa agora para criar a sua conta na AWS agora. Com a conta na AWS criada, vamos agora precisar que você entre no recurso [IAM](https://aws.amazon.com/pt/iam/) da AWS para criar um usuário de preferência sem acesso ao painel online e apenas com credenciais de acesso via API, para facilitar coloque ele como administrador mas lembre-se de destruir o usuário depois dos testes e não salve essas credencias em lugar nenhum que seja inseguro, você pode acabar tendo surpresas desagradáveis na sua fatura da AWS.

Com a credenciais crie um arquivo `credentials` com o seguinte formato:

```env
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

Os arquivos fontes de tudo que será mostrado a seguir se encontra no meu [Github](https://github.com/jvrmaia/packer-playground). Segue o `Makefile` que usarei para melhor entendimento dos prṕximos passos:

```make
.PHONY: build validate

PCKCMD=docker run -it --rm --env-file credentials -v ${PWD}:/opt -w /opt hashicorp/packer:1.5.6

build:
	${PCKCMD} build ${TPLPATH}

validate:
	${PCKCMD} validate ${TPLPATH}
```

## Criando o primeiro manifesto

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY`}}",
        "aws_secret_key": "{{env `AWS_SECRET_KEY`}}",
        "aws_region": "{{env `AWS_REGION`}}"
    },
    "builders": [{
        "type": "amazon-ebs",
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "{{user `aws_region`}}",
        "source_ami": "ami-085925f297f89fce1",
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "hello-world-packer",
        "subnet_id": "subnet-025e117e9c9cd0f91"
    }]
}
```

Nosso arquivo de manifesto consiste nesse exemplo em duas partes, na primeira declaramos variáveis a serem usadas no manifesto e informo que ela vão ser informadas através de variáveis de ambiente no meu sistema. Na segunda parte, temos uma lista de `Builders`, no caso, vamos fazer apenas com o da AWS como dito anteriormente. Entrando em detalhes de cada item:
- `type`: aqui informo o tipo de `Builder`, no caso, AWS;
- `access_key` e `secret_key`: são as credenciais de acesso que o Packer precisa para executar a operação na AWS;
- `region`: a AWS possui várias regiões e cada região tem suas próprias imagens, logo é preciso especificar qual vamos usar;
- `source_ami`: qual imagem de base vamos usar, aqui fui simples e direto na imagem que queria do Ubuntu;
- `instance_type`: para provisionar a imagem é preciso executar uma instância na AWS e com isso vocÊ precisa dizer qual tipo de imagem pretende usar, no nosso caso podemos usar essa bem simples já que não tem grandes mudanças, talvez seja interessante você variar esse tipo caso tenha instalações mais complexas;
- `ssh_username`: para executar o provisionamento o Packer faz uso do SSH para entrar na instância e executar suas atividades;
- `ami_name`: o nome final que será dado a sua nova imagem; e
- `subnet_id`: tive que adicionar essa informação pois estava dando erro aqui sem ela e não vi nenhuma explicação clara na documentação do Packer a respeito, experimente sem ela e caso tenha problemas também é preciso ir no recurso [VPC](https://aws.amazon.com/pt/vpc/) da sua conta e pegar alguma subnet pública para adicionar nessa configuração, lembrando que isso é individual por conta.

### Validando o arquivo de manifesto do Packer

```
$ make validate TPLPATH=./hello-world-packer/template.json
docker run -it --rm --env-file credentials -v /home/jvrmaia/Workspace/jvrmaia/packer-playground:/opt -w /opt hashicorp/packer:1.5.6 validate ./hello-world-packer/template.json
Template validated successfully.
```

Sem problemas com nosso arquivo de configuração. Podemos agora aplicar ele para execução.

### Executando a criação da imagem

```
$ make build TPLPATH=./hello-world-packer/template.json
docker run -it --rm --env-file credentials -v /home/jvrmaia/Workspace/jvrmaia/packer-playground:/opt -w /opt hashicorp/packer:1.5.6 build ./hello-world-packer/template.json
amazon-ebs: output will be in this color.

==> amazon-ebs: Prevalidating any provided VPC information
==> amazon-ebs: Prevalidating AMI Name: hello-world-packer
    amazon-ebs: Found Image ID: ami-085925f297f89fce1
==> amazon-ebs: Creating temporary keypair: packer_5ec90c3b-1c5a-0b0d-c703-5679a8d818d7
==> amazon-ebs: Creating temporary security group for this instance: packer_5ec90c44-555e-9721-0b32-0ae7906fa326
==> amazon-ebs: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: i-0a76f2665b8c6632a
==> amazon-ebs: Waiting for instance (i-0a76f2665b8c6632a) to become ready...
==> amazon-ebs: Using ssh communicator to connect: 54.160.68.147
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Stopping the source instance...
    amazon-ebs: Stopping instance
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating AMI hello-world-packer from instance i-0a76f2665b8c6632a
    amazon-ebs: AMI: ami-0d054dd9d276b95f7
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Cleaning up any extra volumes...
==> amazon-ebs: No volumes to clean up, skipping
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-east-1: ami-0d054dd9d276b95f7
```

É interessante observar a saída do Packer para entender como o processo todo ocorre:
- Validação das informações dadas do provedor (AWS) e nome da imagem;
- Procura a imagem que vai usar de base;
- Criar temporarimente uma chave de acesso para a instância e uma política se segurança para acessar a mesma;
- Cria a instância e coloca o nome de "Packer Builder";
- Aguarda ela ficar disponível para acesso;
- Executa o que precisa, nesse caso nada;
- Desliga a instância e gera a imagem dela;
- Apaga a imamgem e disco usado para executar ela;
- Apaga as credenciais e políticas de acesso; e
- Por fim, mostra o ID da nova AMI gerada.

Caso queira conferir no painel da AWS:

![painel da aws com a imagem criada](/img/conhecendo-o-packer/hello-world-packer-ami.png)

# Provisionando com o Packer

Agora, além de criar nossa instância vamos querer fazer algo mais, vamos fazer modificações na imagem e validar elas. Então, no nosso arquivod de manifesto vamos criar uma nova entrada de informações para isso, no Packer são chamados de [Provisioners](https://www.packer.io/docs/provisioners/). Assim como no `Builders` é possível passar uma lista de `Provisioners`.

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY`}}",
        "aws_secret_key": "{{env `AWS_SECRET_KEY`}}",
        "aws_region": "{{env `AWS_REGION`}}"
    },
    "builders": [{
        "type": "amazon-ebs",
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "{{user `aws_region`}}",
        "source_ami": "ami-085925f297f89fce1",
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "nginx-with-packer",
        "subnet_id": "subnet-025e117e9c9cd0f91"
    }],
    "provisioners": [
        {
            "type": "shell",
            "inline": ["sudo apt install -y nginx"]
        },
        {
            "type": "file",
            "source": "/opt/nginx/index.html",
            "destination": "/tmp/index.html"
        },
        {
            "type": "shell",
            "inline": ["sudo cp /tmp/index.html /var/www/html/index.html"]
        },
        {
            "type": "shell",
            "inline": ["sudo rm /var/www/html/index.nginx-debian.html"]
        }
    ]
}
```

Vamos entender aqui o que foi feito de novo:
- `provisioners`: lista de operações a serem feitas de provisionamento;
- provisionamento do tipo `shell`: o que fizemos foi passar uma linha de comando simples, nota, nos dois últimos comandos poderia ter feito os dois juntos separando por `,` dentro do `array` de `inline`; e
- provisionamento do tipo `file`: que faz uma simples cópia do arquivo local para a instância remota.

## Validando o manifesto

```
$ make validate TPLPATH=./nginx/template.json
docker run -it --rm --env-file credentials -v /home/jvrmaia/Workspace/jvrmaia/packer-playground:/opt -w /opt hashicorp/packer:1.5.6 validate ./nginx/template.json
Template validated successfully.
```

## Executando o manifesto

```
$ make build TPLPATH=./nginx/template.json [A
docker run -it --rm --env-file credentials -v /home/jvrmaia/Workspace/jvrmaia/packer-playground:/opt -w /opt hashicorp/packer:1.5.6 build ./nginx/template.json
amazon-ebs: output will be in this color.

==> amazon-ebs: Prevalidating any provided VPC information
==> amazon-ebs: Prevalidating AMI Name: nginx-with-packer
    amazon-ebs: Found Image ID: ami-085925f297f89fce1
==> amazon-ebs: Creating temporary keypair: packer_5ec93625-91b9-eb54-e0eb-563019fdffa0
==> amazon-ebs: Creating temporary security group for this instance: packer_5ec9362e-e6f6-4aab-ec29-58d0e5b9120e
==> amazon-ebs: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: i-0d5b35d6607c22bda
==> amazon-ebs: Waiting for instance (i-0d5b35d6607c22bda) to become ready...
==> amazon-ebs: Using ssh communicator to connect: 100.25.111.123
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell961822417
==> amazon-ebs:
==> amazon-ebs: WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
==> amazon-ebs:
    amazon-ebs: Reading package lists...
    amazon-ebs: Building dependency tree...
    amazon-ebs: Reading state information...
    amazon-ebs: The following additional packages will be installed:
    amazon-ebs:   fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0
    amazon-ebs:   libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip
    amazon-ebs:   libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter
    amazon-ebs:   libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 nginx-common
    amazon-ebs:   nginx-core
    amazon-ebs: Suggested packages:
    amazon-ebs:   libgd-tools fcgiwrap nginx-doc ssl-cert
    amazon-ebs: The following NEW packages will be installed:
    amazon-ebs:   fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0
    amazon-ebs:   libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip
    amazon-ebs:   libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter
    amazon-ebs:   libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 nginx
    amazon-ebs:   nginx-common nginx-core
    amazon-ebs: 0 upgraded, 18 newly installed, 0 to remove and 0 not upgraded.
    amazon-ebs: Need to get 2462 kB of archives.
    amazon-ebs: After this operation, 8210 kB of additional disk space will be used.
    amazon-ebs: Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libjpeg-turbo8 amd64 1.5.2-0ubuntu5.18.04.3 [110 kB]
    amazon-ebs: Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 fonts-dejavu-core all 2.37-1 [1041 kB]
    amazon-ebs: Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 fontconfig-config all 2.12.6-0ubuntu2 [55.8 kB]
    amazon-ebs: Get:4 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 libfontconfig1 amd64 2.12.6-0ubuntu2 [137 kB]
    amazon-ebs: Get:5 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 libjpeg8 amd64 8c-2ubuntu8 [2194 B]
    amazon-ebs: Get:6 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 libjbig0 amd64 2.1-3.1build1 [26.7 kB]
    amazon-ebs: Get:7 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libtiff5 amd64 4.0.9-5ubuntu0.3 [153 kB]
    amazon-ebs: Get:8 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 libwebp6 amd64 0.6.1-2 [185 kB]
    amazon-ebs: Get:9 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic/main amd64 libxpm4 amd64 1:3.5.12-1 [34.0 kB]
==> amazon-ebs: debconf: unable to initialize frontend: Dialog
==> amazon-ebs: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
==> amazon-ebs: debconf: falling back to frontend: Readline
    amazon-ebs: Get:10 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgd3 amd64 2.2.5-4ubuntu0.4 [119 kB]
    amazon-ebs: Get:11 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx-common all 1.14.0-0ubuntu1.7 [37.4 kB]
==> amazon-ebs: debconf: unable to initialize frontend: Readline
    amazon-ebs: Get:12 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-geoip amd64 1.14.0-0ubuntu1.7 [11.2 kB]
==> amazon-ebs: debconf: (This frontend requires a controlling tty.)
==> amazon-ebs: debconf: falling back to frontend: Teletype
==> amazon-ebs: dpkg-preconfigure: unable to re-open stdin:
    amazon-ebs: Get:13 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-image-filter amd64 1.14.0-0ubuntu1.7 [14.6 kB]
    amazon-ebs: Get:14 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-http-xslt-filter amd64 1.14.0-0ubuntu1.7 [13.0 kB]
    amazon-ebs: Get:15 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-mail amd64 1.14.0-0ubuntu1.7 [41.8 kB]
    amazon-ebs: Get:16 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnginx-mod-stream amd64 1.14.0-0ubuntu1.7 [63.7 kB]
    amazon-ebs: Get:17 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx-core amd64 1.14.0-0ubuntu1.7 [413 kB]
    amazon-ebs: Get:18 http://us-east-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 nginx all 1.14.0-0ubuntu1.7 [3596 B]
    amazon-ebs: Fetched 2462 kB in 0s (9432 kB/s)
    amazon-ebs: Selecting previously unselected package libjpeg-turbo8:amd64.
    amazon-ebs: (Reading database ... 56588 files and directories currently installed.)
    amazon-ebs: Preparing to unpack .../00-libjpeg-turbo8_1.5.2-0ubuntu5.18.04.3_amd64.deb ...
    amazon-ebs: Unpacking libjpeg-turbo8:amd64 (1.5.2-0ubuntu5.18.04.3) ...
    amazon-ebs: Selecting previously unselected package fonts-dejavu-core.
    amazon-ebs: Preparing to unpack .../01-fonts-dejavu-core_2.37-1_all.deb ...
    amazon-ebs: Unpacking fonts-dejavu-core (2.37-1) ...
    amazon-ebs: Selecting previously unselected package fontconfig-config.
    amazon-ebs: Preparing to unpack .../02-fontconfig-config_2.12.6-0ubuntu2_all.deb ...
    amazon-ebs: Unpacking fontconfig-config (2.12.6-0ubuntu2) ...
    amazon-ebs: Selecting previously unselected package libfontconfig1:amd64.
    amazon-ebs: Preparing to unpack .../03-libfontconfig1_2.12.6-0ubuntu2_amd64.deb ...
    amazon-ebs: Unpacking libfontconfig1:amd64 (2.12.6-0ubuntu2) ...
    amazon-ebs: Selecting previously unselected package libjpeg8:amd64.
    amazon-ebs: Preparing to unpack .../04-libjpeg8_8c-2ubuntu8_amd64.deb ...
    amazon-ebs: Unpacking libjpeg8:amd64 (8c-2ubuntu8) ...
    amazon-ebs: Selecting previously unselected package libjbig0:amd64.
    amazon-ebs: Preparing to unpack .../05-libjbig0_2.1-3.1build1_amd64.deb ...
    amazon-ebs: Unpacking libjbig0:amd64 (2.1-3.1build1) ...
    amazon-ebs: Selecting previously unselected package libtiff5:amd64.
    amazon-ebs: Preparing to unpack .../06-libtiff5_4.0.9-5ubuntu0.3_amd64.deb ...
    amazon-ebs: Unpacking libtiff5:amd64 (4.0.9-5ubuntu0.3) ...
    amazon-ebs: Selecting previously unselected package libwebp6:amd64.
    amazon-ebs: Preparing to unpack .../07-libwebp6_0.6.1-2_amd64.deb ...
    amazon-ebs: Unpacking libwebp6:amd64 (0.6.1-2) ...
    amazon-ebs: Selecting previously unselected package libxpm4:amd64.
    amazon-ebs: Preparing to unpack .../08-libxpm4_1%3a3.5.12-1_amd64.deb ...
    amazon-ebs: Unpacking libxpm4:amd64 (1:3.5.12-1) ...
    amazon-ebs: Selecting previously unselected package libgd3:amd64.
    amazon-ebs: Preparing to unpack .../09-libgd3_2.2.5-4ubuntu0.4_amd64.deb ...
    amazon-ebs: Unpacking libgd3:amd64 (2.2.5-4ubuntu0.4) ...
    amazon-ebs: Selecting previously unselected package nginx-common.
    amazon-ebs: Preparing to unpack .../10-nginx-common_1.14.0-0ubuntu1.7_all.deb ...
    amazon-ebs: Unpacking nginx-common (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package libnginx-mod-http-geoip.
    amazon-ebs: Preparing to unpack .../11-libnginx-mod-http-geoip_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking libnginx-mod-http-geoip (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package libnginx-mod-http-image-filter.
    amazon-ebs: Preparing to unpack .../12-libnginx-mod-http-image-filter_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking libnginx-mod-http-image-filter (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package libnginx-mod-http-xslt-filter.
    amazon-ebs: Preparing to unpack .../13-libnginx-mod-http-xslt-filter_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking libnginx-mod-http-xslt-filter (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package libnginx-mod-mail.
    amazon-ebs: Preparing to unpack .../14-libnginx-mod-mail_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking libnginx-mod-mail (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package libnginx-mod-stream.
    amazon-ebs: Preparing to unpack .../15-libnginx-mod-stream_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking libnginx-mod-stream (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package nginx-core.
    amazon-ebs: Preparing to unpack .../16-nginx-core_1.14.0-0ubuntu1.7_amd64.deb ...
    amazon-ebs: Unpacking nginx-core (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Selecting previously unselected package nginx.
    amazon-ebs: Preparing to unpack .../17-nginx_1.14.0-0ubuntu1.7_all.deb ...
    amazon-ebs: Unpacking nginx (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up libjbig0:amd64 (2.1-3.1build1) ...
    amazon-ebs: Setting up fonts-dejavu-core (2.37-1) ...
    amazon-ebs: Setting up nginx-common (1.14.0-0ubuntu1.7) ...
    amazon-ebs: debconf: unable to initialize frontend: Dialog
    amazon-ebs: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    amazon-ebs: debconf: falling back to frontend: Readline
    amazon-ebs: Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
    amazon-ebs: Setting up libjpeg-turbo8:amd64 (1.5.2-0ubuntu5.18.04.3) ...
    amazon-ebs: Setting up libnginx-mod-mail (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up libxpm4:amd64 (1:3.5.12-1) ...
    amazon-ebs: Setting up libnginx-mod-http-xslt-filter (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up libnginx-mod-http-geoip (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up libwebp6:amd64 (0.6.1-2) ...
    amazon-ebs: Setting up libjpeg8:amd64 (8c-2ubuntu8) ...
    amazon-ebs: Setting up fontconfig-config (2.12.6-0ubuntu2) ...
    amazon-ebs: Setting up libnginx-mod-stream (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up libtiff5:amd64 (4.0.9-5ubuntu0.3) ...
    amazon-ebs: Setting up libfontconfig1:amd64 (2.12.6-0ubuntu2) ...
    amazon-ebs: Setting up libgd3:amd64 (2.2.5-4ubuntu0.4) ...
    amazon-ebs: Setting up libnginx-mod-http-image-filter (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up nginx-core (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Setting up nginx (1.14.0-0ubuntu1.7) ...
    amazon-ebs: Processing triggers for systemd (237-3ubuntu10.39) ...
    amazon-ebs: Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
    amazon-ebs: Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...
    amazon-ebs: Processing triggers for ureadahead (0.100.0-21) ...
    amazon-ebs: Processing triggers for libc-bin (2.27-3ubuntu1) ...
==> amazon-ebs: Uploading /opt/nginx/index.html => /tmp/index.html
index.html 18 B / 18 B [=======================================================================================================] 100.00% 9s
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell936662298
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell378584502
==> amazon-ebs: Stopping the source instance...
    amazon-ebs: Stopping instance
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating AMI nginx-with-packer from instance i-0d5b35d6607c22bda
    amazon-ebs: AMI: ami-01915a10c5eebc69e
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Cleaning up any extra volumes...
==> amazon-ebs: No volumes to clean up, skipping
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-east-1: ami-01915a10c5eebc69e
```

Sucesso ao criar a imagem! =)

![ami do nginx](/img/conhecendo-o-packer/imagem-nginx-criada.png)

O que vale mostrar aqui de diferente nos outros passos são os momentos que nosso provisionamento é feito:

```
...
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell961822417
...
==> amazon-ebs: Uploading /opt/nginx/index.html => /tmp/index.html
index.html 18 B / 18 B [=======================================================================================================] 100.00% 9s
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell936662298
==> amazon-ebs: Provisioning with shell script: /tmp/packer-shell378584502
...
```

Repare que o Packer transforma nossos comandos do Shell em scripts e executa eles.

## Validando a imagem criada

Na imagem criada sobreescri o HTML padrão do NGINX por:

```
$ cat nginx/index.html
hello from packer
```

Vamos conferir se funcionou executando a imagem e acessando ela via navegador.

### Executando a instância

![painel da aws com instância rodando](/img/conhecendo-o-packer/instancia-nginx-rodando.png)

### Acessando a página do NGINX

![acessando página do nginx](/img/conhecendo-o-packer/nginx-funcionando.png)

# Conclusão

Vimos que o Packer é uma execelente e simples solução para criação de imagens estáticas na AWS ou outro provedor que você use e tenha suporte. Com o que foi visto aqui já podemos aplicar para diversos casos por ai em situações reais. Melhorias que podem ser feitas ainda e fica como sugestão de investigação sua e para próximos artigos:
- provisionar via `bastion host` em subnet privada para não criar superfície de ataque na sua infra durante a execução;
- usar algum provisionador de instância ao invés do shell como ansible, por exemplo;
- provisionar em múltiplos lugares ao mesmo tempo;
- melhores práticas nos arquivos de manifesto;
- criando testes para seus manifestos; e
- criando um ciclo de entrega contínua de imagens.
