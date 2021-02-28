---
author: "João Maia"
date: 2020-01-10
title: "Instalação do Go"
tags: [
    "golang",
    "iniciante",
]
categories: [
    "linguagens de programação"
]
thumbnail: "img/instalacao-do-go/go.jpeg"
---

Olá,

esse artigo faz parte da sequência que pretendo escrever sobre Go para que sua jornada inicial na linguagem seja uma experiência muito positiva e proveitosa. Nesse artigo vou abordar em especial a questão de como instalar o Go na sua máquina. Como sou usuário de Linux vou dar foco nesse sistema, usuários do OSX provavelmente vão conseguir usar os mesmos passos mas infelizmente não vou conseguir cobrir o caso do Windows por não seu um usuário do sistema. Minha recomendação para os usuários de Windows é usar o [WSL](https://docs.microsoft.com/pt-br/windows/wsl/install-win10) e usar algum distribuição Linux.

# Documentação oficial

O ideal é sempre olhar a documentação oficial e seguir suas instruções, no caso de Go essas instruções se encontram [aqui](https://golang.org/doc/install). O problema dessa abordagem da documentação é oficial que sabemos que na vida real os projetos rodam em versões diversas. Então, fica complicado usar essa abordagem pois você dificuldades para lidar com isso já que vai instalar no sistema de forma global. Caso você escolha seguir nessa linha só faço algumas sugestões, no passo:

    tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz

recomendo renomear a pasta do `go` que vai ser criada por `go$VERSION` e usar um link simbólico com `ln` para criar a pasta `go` sempre para a versão corrente que você estiver usando. Ainda assim, não é legal essa abordagem pois você vai precisar de `sudo` nesse caso para configurar o Go. Então, minha segunda sugestão nessa metodologia é na sua `$HOME` uma pasta para colocar todas as instalações do Go, pode ser uma oculta, por exemplo, `$HOME/.go-versions` ou uma não oculta com o mesmo nome. Siga o procedimento de descompactar nela as versões e use o recurso do `ln` para fazer o mesmo padrão que citei no começo. Não se esqueça de ajudar o `PATH`:

    export PATH=$PATH:$HOME/.go-versions/go/bin

Caso você escolha essa caminho comente ai o motivo da escolha e como é a experiência.

# GVM

Essa é a forma que uso no meu computador pessoal e de trabalho, o [gvm](https://github.com/moovweb/gvm) é um gerenciador de versões do Go feito em bash que pode ser instalado apenas com um um comando, seguindo o tutorial dele:

    bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

ou via Home Brew caso vocÊ seja usuário, lembrando que o Home Brew já funciona também no Linux apesar de ser original do OSX, mais informações você pode conferir no README.md do projeto.

A cara do `gvm`:

    $ gvm
    Usage: gvm [command]

    Description:
    GVM is the Go Version Manager

    Commands:
    version    - print the gvm version number
    get        - gets the latest code (for debugging)
    use        - select a go version to use (--default to set permanently)
    diff       - view changes to Go root
    help       - display this usage text
    implode    - completely remove gvm
    install    - install go versions
    uninstall  - uninstall go versions
    cross      - install go cross compilers
    linkthis   - link this directory into GOPATH
    list       - list installed go versions
    listall    - list available versions
    alias      - manage go version aliases
    pkgset     - manage go packages sets
    pkgenv     - edit the environment for a package set

Não tem muito segredo aqui, só peço que antes você faço os passos contidos [aqui](https://github.com/moovweb/gvm#linux-requirements) de acordo com o seu sistema operacional. Para instalar é bem simples, vamos primeiro listar as versões que o `gvm` possui:

    $ gvm listall

    gvm gos (available)

        go1
        go1.0.1
        go1.0.2
        go1.0.3
        ...
        go1.13.5
        go1.13.6
        go1.14beta1
        release.r56
        release.r57
        ...
        release.r60.2
        release.r60.3

Agora, sabendo quais opções você tem é só escolher uma para instalação, vamos viver na emoção e escolher a versão `go1.14beta1`. Existem duas formas de proceder daqui, a primeira você vai fazer a instalação fazendo o download do projeto e compilando ele e a segunda é apenas pegando os binários e instalando:

    # compilando
    gvm install go1.14beta1

    # apenas binários
    gvm install go1.14beta1 -B

Simples, não? Agora, você para fazer uso dessa instalação basta fazer:

    gvm use go1.14beta1

Pronto! Go instalado e configurado no seu ambiente para uso. Esse artigo se encerra por aqui e no próximo vamos mostrar como seguir daqui para começar a ficar mais próximo de estar trabalhando em algum projeto. Se quiser saber quais versões você já instalou é só fazer:

    $ gvm list

    gvm gos (installed)

        go1.13.4
        go1.8.3
        go1.9
        system

# Outros métodos

- [ASDF](https://github.com/kennyp/asdf-golang)
- [g](https://github.com/stefanmaric/g)
- Instalador de pacotes da sua distribuição ou Home Brew

# Próximos artigos

Nos próximos artigos, que pretendo escrever o mais breve possível, vou ainda cobrir alguns tópicos básicos:
- Organização de projetos e GOPATH
- Trabalhando com dependências
