---
author: "João Maia"
date: 2020-01-22
title: "Trabalhando com dependências em Go"
tags: [
    "golang",
    "iniciante",
]
categories: [
    "linguagens de programação"
]
thumbnail: "img/trabalhando-com-dependencias-em-go/modules.png"
---

Olá,

Esse é o último artigo da minha sequência proposta para introduzir novas pessoas ao Go. Espero que tenha sido útil até aqui e hoje esse material se complete para você ter a liberdade de começar a praticar e usar o Go. Ao invés de explicar como as coisas devem ser feitas hoje vou mostrar um pouco do histórico desse contexto no Go no meu ponto de vista como usuário mostrando o que encontrei nesse caminho.

Quando comecei com Go não tinha nenhuma ferramenta de gestão de dependências em Go, então, a solução era manter ou um `make dep` que tinha todos os `go get` necessários para rodar o projeto ou algum script em bash que fazia o mesmo trabalho. Sofrido, não? Para quem vinha de de C/C++ era até melhor a experiência mas para quem já vinha de Ruby, Python e Java, por exemplo, era horrível a experiência. Não demorou muito com a aderência da comunidade pela linguagem isso ser uma reclamação constante e emergir da comunidade algumas soluções até a solução oficial da linguagem hoje, segue a minha jornada que vou abordar em mais detalhes a seguir: Godeps, govendor, Glide, go dep e go modules.

# Godeps

[Esse](https://github.com/tools/godep) foi o primeiro projeto que usei para gerir dependências em Go. Foi uma experiência de uso de ferramenta bem ruim e ai começava a saga de comitar a pasta `vendor` no repositório para não correr risco de quebrar a integração contínua.

# govendor

Esse ainda consta no Github [aqui](https://github.com/kardianos/govendor). Foi o primeiro que usei depois do Godeps e ajudou bastante com a interface de linha de comandos dele, ajudou o controlar melhor e ficar mais com cara de sistema de gestão de dependências iguais ao que eu tinha familiaridade de outras linguagens.

# Glide

Ainda sentindo com os problemas do govendor, achei o [Glide](https://github.com/Masterminds/glide) que ajudou muito com o conceito de arquivo de `lock` que versionava as minhas dependências. Agora sim, estava satisfeito pois tinha chegado a um nível próximo ao do que vinha usando no Python e Ruby, o que faltava era ter um serviço central para cuidar das dependências.

# go dep

Estava feliz com meus projetos migrados para o Glide e me deparo com a chegada do [go dep](https://github.com/golang/dep), pronto, mais uma vez lá vamos nós migrar por conta que parece que a solução oficial da linguagem vai ser essa. Acabou que a migração não aconteceu e entrei em um hiato de trabalhar com Go e não tive a experiência de usar.

# go modules

Afastado da programação em Go, eis que surge o [Go Modules](https://blog.golang.org/using-go-modules) e parece que a saga da gestão de dependências chega ao fim. Agora, voltando a programar em Go tenho como meta usar ele mas ainda estamos aqui com umas outras pendências mais importantes no projeto antes de migrar, estou ansioso para testar mas nos projetos que vi no Github que usam e testei a experiência ficou bem melhor. Nos [último artigo](https://blog.joaovrmaia.com/post/organizacao-de-projetos-e-gopath/) acabei dando `spoiler` de como usar `go modules` no projeto `hello`. Como é a cara desse `go modules`:

    $ go mod
    Go mod provides access to operations on modules.

    Note that support for modules is built into all the go commands,
    not just 'go mod'. For example, day-to-day adding, removing, upgrading,
    and downgrading of dependencies should be done using 'go get'.
    See 'go help modules' for an overview of module functionality.

    Usage:

        go mod <command> [arguments]

    The commands are:

        download    download modules to local cache
        edit        edit go.mod from tools or scripts
        graph       print module requirement graph
        init        initialize new module in current directory
        tidy        add missing and remove unused modules
        vendor      make vendored copy of dependencies
        verify      verify dependencies have expected content
        why         explain why packages or modules are needed

    Use "go help mod <command>" for more information about a command.

Alterando o projeto `hello` para ver como funciona o `go modules`:

{{< highlight golang >}}
package morestrings

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestReverseRunes(t *testing.T) {
    cases := []struct {
        in, want string
    }{
        {"Hello, world", "dlrow ,olleH"},
        {"Hello, 世界", "界世 ,olleH"},
        {"", ""},
    }
    for _, c := range cases {
        got := ReverseRunes(c.in)
        if got != c.want {
            assert.Equal(t, got, c.want)
        }
    }
}
{{< /highlight >}}

Executando agora vamos ter o seguinte resultado:

    $ go test ./...
    go: finding github.com/stretchr/testify v1.4.0
    go: downloading github.com/stretchr/testify v1.4.0
    go: extracting github.com/stretchr/testify v1.4.0
    go: downloading gopkg.in/yaml.v2 v2.2.2
    go: downloading github.com/davecgh/go-spew v1.1.0
    go: downloading github.com/pmezard/go-difflib v1.0.0
    go: extracting gopkg.in/yaml.v2 v2.2.2
    go: extracting github.com/pmezard/go-difflib v1.0.0
    go: extracting github.com/davecgh/go-spew v1.1.0
    go: finding github.com/pmezard/go-difflib v1.0.0
    go: finding github.com/davecgh/go-spew v1.1.0
    go: finding gopkg.in/yaml.v2 v2.2.2
    ?   	github.com/jvrmaia/hello	[no test files]
    ok  	github.com/jvrmaia/hello/morestrings	0.002s

Repare que ainda não temos uma pasta `vendor` trabalhando dessa forma no projeto:

    $ ls
    go.mod  go.sum  hello.go  morestrings/

Para criar a estrutura do `vendor` basta usar o comando `go mod vendor`, veja o resultado:

    $ tree -L 3 .
    .
    ├── go.mod
    ├── go.sum
    ├── hello.go
    ├── morestrings
    │   ├── morestrings.go
    │   └── reverse_test.go
    └── vendor
        ├── github.com
        │   ├── davecgh
        │   ├── google
        │   ├── pmezard
        │   └── stretchr
        ├── gopkg.in
        │   └── yaml.v2
        └── modules.txt
    $ cat vendor/modules.txt
    # github.com/davecgh/go-spew v1.1.0
    github.com/davecgh/go-spew/spew
    # github.com/google/go-cmp v0.4.0
    github.com/google/go-cmp/cmp
    github.com/google/go-cmp/cmp/internal/diff
    github.com/google/go-cmp/cmp/internal/flags
    github.com/google/go-cmp/cmp/internal/function
    github.com/google/go-cmp/cmp/internal/value
    # github.com/pmezard/go-difflib v1.0.0
    github.com/pmezard/go-difflib/difflib
    # github.com/stretchr/testify v1.4.0
    github.com/stretchr/testify/assert
    # gopkg.in/yaml.v2 v2.2.2
    gopkg.in/yaml.v2

Existem outras coisas mais a serem exploradas no `go modules` mas recomendo que agora você tente ir por você mesmo nessa descoberta e para ajudar no próximo tópico recomendo alguns artigos sobre essa parte da história de gestão de dependências em Go e sobre o uso de `go modules`.

Espero que com essa sequência de artigos tenha ajudado você a começar a sua jornada em Go.

# Artigos relacionados ao tema

- [Our Software Dependency Problem](https://research.swtch.com/deps)
- [Wiki do Go sobre ferramentes de gestão de dependências](https://github.com/golang/go/wiki/PackageManagementTools)
- [An intro to dep: How to manage your Golang project dependencies](https://www.freecodecamp.org/news/an-intro-to-dep-how-to-manage-your-golang-project-dependencies-7b07d84e7ba5/)
- [Package Management With Go Modules: The Pragmatic Guide](https://medium.com/@adiach3nko/package-management-with-go-modules-the-pragmatic-guide-c831b4eaaf31)
