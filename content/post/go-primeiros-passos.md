---
author: "João Maia"
date: 2020-01-03
title: "Go: primeiros passos"
tags: [
    "golang",
    "iniciante",
]
categories: [
    "linguagens de programação"
]
thumbnail: "img/go-primeiros-passos/golang.jpg"
---

Olá,

esse primeiro post sobre [Go](https://golang.org/) é para mostrar a quem não sabe nada sobre a linguagem algumas características e minhas opiniões a respeito dela. Caso discorde de algo e/ou tenha alguma correção a ser feita no texto pode me apontar nos comentários ou no meu [e-mail](mailto:joao@joaovrmaia.com) que darei toda atenção desde que faça de forma educada.

# Senta que lá vem a História

![Rá-Tim-Bum: Senta que lá vem a História](/img/go-primeiros-passos/historia.jpg)

Do [Wikipedia](https://pt.wikipedia.org/wiki/Go_(linguagem_de_programa%C3%A7%C3%A3o)):

> Go é uma linguagem de programação criada pela Google e lançada em código livre em novembro de 2009. É uma linguagem compilada e focada em produtividade e programação concorrente, baseada em trabalhos feitos no sistema operacional chamado Inferno. O projeto inicial da linguagem foi feito em setembro de 2007 por Robert Griesemer, Rob Pike e Ken Thompson. Atualmente, há implementações para Windows, Linux, Mac OS X e FreeBSD.

Como muitas empresas que começaram antes da era dos microserviços o Google é também um monorepo com diversos sistemas e seus legados. Para combater essa complexidade espalhada em sistemas escritos em C++ e Java nasceu Go com várias promessas de entrega em facilidades de criação de código, desempenho e distribuição. Um exemplo disso foi compartilhado pelo [Brad Fitzpatrick](https://twitter.com/bradfitz) nos links a seguir:
- Google Groups: [dl.google.com now served by Go](https://groups.google.com/forum/#!msg/golang-nuts/BNUNbKSypE0/E4qSfpx9qI8J)
- Talk: [dl.google.com: Powered by Go](https://talks.golang.org/2013/oscon-dl.slide#1)

# Características da linguagem

![Gophers](/img/go-primeiros-passos/gophers.jpg)

## Tipagem estática

Se você vem de linguagens como C/C++, Java e C# das mais tradicionais, provavelmente, já deve saber do que se trata isso, diferente de quem vem de Javascript, Ruby e Python que são linguagens de tipagem dinâmica. Linguagens de tipagem estática são aquelas que suas variáveis após declaradas ou incializadas com algum valor assumem um contrato com o tipo atribuido a ela e não mudam ao longo do seu escopo, caso tenha uma tentativa de violação disso o mesmo será sinalizado pelo compilador da linguagem.

- [Exemplo mostrando a declaração de variáveis](https://play.golang.org/p/YYJRlWZebZf)
- [Tipos primitivos de Go](https://tour.golang.org/basics/11)

## Não tem uma máquina virtual

Você já deve ter escutado o termo `JVM`, Java Virtual Machine, e agora deve suspeitar do que se trata. Sim, é a máquina virtual do Java mas não somente ele, algumas outras linguagens conseguem rodar na JVM mas isso não é papo para agora. O fato de Go não ter uma máquina virtual faz com que o ínicio de sua aplicação seja mais rápido pois não tem a inicialização da máquina virtual, você vai escutar por ai: "Go tem footprint baixo", e agora você sabe o que é isso.

- [Artigo sobre máquina virtual do Wikipedia](https://pt.wikipedia.org/wiki/M%C3%A1quina_virtual)

## Coletor de lixo

O nome é uma tradução direta da expressão `Garbage Collector` que as vezes será referenciado apenas por `GC`. O `GC` é responsável por pegar memórias reservadas por sua aplicação que não são mais usadas e devolver para o sistema operacional. Quem vem de linguagens como Java, Python e Ruby já deve ter familiaridade com o assunto pois todas tem. Quem vem do C/C++ aprendeu a gerenciar isso no código e provavelmente ter sofrido bastante, por isso algumas linguagens adotam o uso de um `GC` removendo essa complexidade do programador. Só que já teve que lidar com `mallocs` e `free` do C entende esse sofrimento.

- [Artigo sobre coletor de lixo do Wikipedia](https://pt.wikipedia.org/wiki/Coletor_de_lixo_(inform%C3%A1tica))

## Distribuição em binário

Go é uma linguagem que ao ser compilada um binário é gerado que pode ser executado sem grandes problemas dado a diferença que vimos de outra linguagens no tópico de máquina virtual. Esse binário provavelmente poderá ser executado em diversas plataformas mudando as variáveis `GOOS` e `GOARCH`, [documentação](https://golang.org/pkg/go/build/#Context), não vou entrar em detalhes nisso agora mas se tiver curiosidade esse [post](https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04) mostra como funciona isso, o termo é `crosscompiling` pois permite do Linux gerar um binário Windows, por exemplo.

- [Exemplo de binários para diversas plataformas: Terraform](https://www.terraform.io/downloads.html)

## A saga de gestão de dependências

Go nasceu sem um gerenciador de dependências e ficou por muito tempo sem um oficial até surgir o [Go Modules](https://blog.golang.org/using-go-modules) que resolveu isso, antes tivemos várias alternaticas de solução como [govendor](https://github.com/kardianos/govendor), [glide](https://github.com/Masterminds/glide) e [go dep](https://github.com/golang/dep), por exemplo. A única coisa que nunca mudou foi a localização delas no seu projeto que é na pasta `vendor` do seu repositório. Isso foi alvo de muitas polêmicas e motivador para muitos não adotarem Go dado que diversas outras linguagens tem isso já resolvido como o [Maven](https://maven.apache.org/) do Java, [npm](https://www.npmjs.com/) do Javascript e [PyPI](https://pypi.org/) do Python, por exemplo. Outro detalhe da solução é que ela não conta com um grande servidor central de pacotes, os pacotes no seu projeto vão ser sempre referenciados ao fonte do mesmo, ou seja, se todos os projetos do mundo em Go estivessem no Github, poderiamos dizer que ele era praticamente o Maven, npm, PyPI do Go.

## Formatação de código

Em Go essas polêmicas não tem vez, o `go fmt` dita as regras da organização do seu código, ou você respeita ou ele não compila. Aqui quem manda é o `Tab` e o `{` não abre em uma nova linha. Então, fica a dica, configure no seu editor favorito para sempre que salvar um arquivo editado ele rodar sozinho o `go fmt` antes de salvar, vai te economizar um tempo bom na revisão de código do seu time.

- [Package fmt](https://golang.org/pkg/fmt/)

## Goroutines

![Goroutine](/img/go-primeiros-passos/goroutine.jpeg)

Essa é a característica que acredito ser a mais famosa da linguagem, `goroutines`, que é um formas qu Go lida com concorrência e paralelismo, usando troca de mensagens via `channel`, `communicating sequential processes` ou `CSP`. Uma outra opção de trabalhar com concorrência em Go é através de memória compartilhada, `shared memory multithreading`. As pessoas tendem a achar que `goroutines` são threads mas não são pois elas não são mapeadas diretamente ao sistema operacional como thread igual acontece no Java. Seguem dois textos sobre o assunto:

- [Goroutine vs Thread](https://www.geeksforgeeks.org/golang-goroutine-vs-thread/)
- [Concorrência: Goroutines x Threads](https://gopher.net.br/concorrencia-threads-x-goroutines/)

# Comunidade

A comunidade de Go é bastante grande devido ao grande sucesso de várias empresas que adotaram e resolveram seus problemas de escalabilidade e afins com a adoção. Além disso, vários projetos de código aberto foram criados usando Go, vale o destaque para o mais comentado no momento que é o [Kubernetes](https://kubernetes.io/) e por grandes empresas que desenvolvem produtos abertos quase todos em Go como a [HashiCorp](https://www.hashicorp.com/), detalhe, o único projeto deles que não é em Go foi o [Vagrant](https://www.vagrantup.com/) por não existir o Go na época. No Brasil também temos projetos abertos com Go, por exemplo, o [tsuru](https://tsuru.io/) da Globo.com e o [pREST](https://postgres.rest/) da nuveo.

No Brasil hoje temos a nossa [conferência nacional](https://2020.gopherconbr.org/) que já tem data para 2020, o grupo no [Telegram](https://t.me/go_br) e o [Slack](https://invite.slack.golangbridge.org/) que conta com dois canais brasileiros: `#brazil` e `#brazilian-go-studies`, o segundo é um grupo de estudo que se reune todas as quintas-feira via hangouts às 22:00h pilotado por [Cesar Gimenes](https://twitter.com/crgimenes). Além disso, diversos meetups regionais:

- [Golang SP](https://www.meetup.com/golangbr/)
- [GO Belo Horizonte](https://www.meetup.com/go-belo-horizonte/)
- [Floripa Gophers](https://www.meetup.com/Floripa-Gophers/)
- [GopheRio](https://www.meetup.com/GopheRio/)
- [GolangCWB](https://www.meetup.com/GolangCWB/)
- [Joinville Go Meetup](https://www.meetup.com/Joinville-Go-Meetup/)
- [Porto Alegre GOlang Meetup](https://www.meetup.com/Porto-Alegre-GOlang-Meetup/)
- [Women Who Go Sampa](https://www.meetup.com/Women-Who-Go-Sampa/)
- [Women Who Go CWB](https://www.meetup.com/Women-Who-Go-CWB/)
- [Golang Campinas](https://www.meetup.com/Golang-Campinas/)

# Empresas

Dificilmente vamos estudar uma tecnologia que não tem mercado de trabalho, né? Sem problemas quanto a isso com Go. Como já mostrei no tópico anterior tem a HashiCorp e Google que criou. Além disso, várias outras empresas dentro e fora do Brasil usam Go, por exemplo, aqui na [pagar.me](https://pagar.me/) onde trabalho, na [Globo.com](https://www.globo.com/) e na [nuveo](https://www.nuveo.ai/), [Mercado Livre](https://www.mercadolivre.com.br/) entre outras. Fora do Brasil, temos a [Digital Ocean](https://www.digitalocean.com/), [Form3](https://form3.tech/), [SoundCloud](https://soundcloud.com/), [Salesforce](https://www.salesforce.com/), [Uber](https://www.uber.com/), [Twitch.tv](https://www.twitch.tv) e outras. Outras vagas você pode ver [aqui](https://www.golangprojects.com/), um site especializado em vagas para Go. Na [wiki](https://github.com/golang/go/wiki/GoUsers) do repositório de Go vocÊ pode encontrar mais empresa que usam.

- Mercado Livre: [O céu é o limite na utilização de Golang](https://imasters.com.br/apis-microsservicos/o-ceu-e-o-limite-na-utilizacao-de-golang)

# Fontes de estudo

## Livros

- [Programando em Go: Crie aplicações com a linguagem do Google](https://www.amazon.com.br/Programando-Go-aplica%C3%A7%C3%B5es-linguagem-Google-ebook/dp/B06XDSVH8G/)
- [A Linguagem de Programação Go](https://www.amazon.com.br/Linguagem-Programa%C3%A7%C3%A3o-Go-Alan-Donovan/dp/8575225464)
- [Go em Ação](https://www.amazon.com.br/Go-em-A%C3%A7%C3%A3o-William-Kennedy/dp/8575225065)
- [Introdução à Linguagem Go: Crie Programas Escaláveis e Confiáveis](https://www.amazon.com.br/Introdu%C3%A7%C3%A3o-Linguagem-Programas-Escal%C3%A1veis-Confi%C3%A1veis/dp/8575224891)
- [Mastering Go: Create Golang production applications using network libraries, concurrency, machine learning, and advanced data structures, 2nd Edition](https://www.amazon.com.br/Mastering-production-applications-concurrency-structures-ebook/dp/B07WC24RTQ)
- [Aprenda Go com desenvolvimento orientado a testes](https://github.com/larien/learn-go-with-tests)

## YouTube

- [grupo de estudos de golang](https://www.youtube.com/channel/UCxRoRvJi7NbC2boKAV70t_g)
- [Introduction - Go Lang Practical Programming Tutorial](https://www.youtube.com/watch?v=G3PvTWRIhZA&list=PLQVvvaa0QuDeF3hP0wQoSxpkqgRcgxMqX)
- [GopherCon UK](https://www.youtube.com/channel/UC9ZNrGdT2aAdrNbX78lbNlQ)
- [GopherCon Brasil](https://www.youtube.com/channel/UCGFVA_XvkUoMWpKVH0IrjUA)
- [Gopher Academy](https://www.youtube.com/channel/UCx9QVEApa5BKLw9r8cnOFEA)
- [Golang SP](https://www.youtube.com/channel/UCvqY9VI9LREcklZokv_SNEg)

## Cursos online

- [Alura](https://www.alura.com.br/cursos-online-programacao/golang)
- [Lista de cursos em inglês](https://digitaldefynd.com/best-golang-courses-training-tutorial-online/#1_Programming_with_Google_Go_Certification_Course_Coursera)

## Podcasts

- [Go Time](https://changelog.com/gotime)

# Editores e IDE's

Vou apenas comentar minha experiência pessoal, eu uso basicamente 3 editores, para escrever no blog e algumas coisas mais rápidas uso o [Code](https://code.visualstudio.com/) que tem um ótimo suporte para Go, por conta do passado com Java também uso o [IDEA](https://www.jetbrains.com/idea/) que também conta com plugin para Go e por fim, e não menos importante, o [vim](https://www.vim.org/) que foi meu editor principal por anos e hoje apenas quando estou no terminal e preciso editar algo sem precisar sair do terminal, o vim também tem um bom suporte a Go.

# Go é para mim?

Só você pode decidir isso, eu apresentei algumas característica no começo e acredito com sua própria experiência aprendendo vai ter os seus julgamentos que podem ser diferentes do meu. Eu gosto bastante da linguagem pois me lembra C/C++ que foram linguagens me ajudaram na minha formação como desenvolvedor na graduação e algumas facilidades do Java. Acho que é uma linguagem muito boa para criar aplicações de linha de comando como o exemplo que dei do Terraform e serviços de mais baixo nível como o Kubernetes mas nada impede de ser usado para criar aplicações Web's que podia ser feitas com Rails ou django. A sua familiriadade com a linguagem e do seu time que vão definir mais se ele se ajusta a sua situação.

# Próximos artigos

Nos próximos artigos, que pretendo escrever o mais breve possível, vou ainda cobrir alguns tópicos básicos:
- Instalação do Go
- Organização de projetos e GOPATH
- Trabalhando com dependências

# Bônus

- [Go 2, here we come!](https://blog.golang.org/go2-here-we-come)
- Site com mais referências sobre Go: [go.dev](https://go.dev/)
- Coletânea de material para estudos: [Go Wiki | Learn](https://github.com/golang/go/wiki/Learn)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Practical Go: Real world advice for writing maintainable Go programs](https://dave.cheney.net/practical-go/presentations/qcon-china.html)

# Atualizações

- 05/01/2019: Contribuições do Felipe Henrique do Telegram Go Brasil
- 05/01/2019: Contribuições do Daniel Fernandes do Telegram Go Brasil
- 06/01/2019: Contribuições do Francisco Oliveira do Telegram Go Brasil
