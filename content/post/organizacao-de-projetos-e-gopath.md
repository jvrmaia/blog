---
author: "João Maia"
date: 2020-01-17
title: "Organização de projetos e GOPATH"
tags: [
    "golang",
    "iniciante",
]
categories: [
    "linguagens de programação"
]
thumbnail: "img/organizacao-de-projetos-e-gopath/gophers.jpg"
---

Olá,

Esse é um tópico que acaba sendo polêmico onde cada um acaba tendo uma opinião sobre e fazendo uma busca rápida no Google você pode confirmar isso. Como sempre, tem uma recomendação oficial da linguagem e diversas outras alternativas que emergem da comunidade ou formas que empresas acharam para lidar com a sua complexidade de controlar o projeto. Nesse artigo vamos navegar por essas abordagens e ver o que elas trazem de interessante ou não.

# Recomendação oficial

Em ["How to Write Go Code"](https://golang.org/doc/code.html) você vai econtrar um guia de como começar o seu projeto já usando Go Modules criando um projeto bem simples com importação de package e testes. Vamos fazer ele aqui juntos de forma mais breve?

Primeiro vou trocar meu diretório atual para `$HOME/Workspace/temp` que é um lugar que uso para testar coisas novas e pequenas demonstrações que sei que vou poder apagar depois qualquer coisa dentro sem grandes perdas. A seguir, vou criar a pasta do projeto como é dito no tutorial:

    $ mkdir hello && cd $_

Agora, já dentro da pasta do projeto que estou criando vou inicializar o projeto:

    $ go mod init github.com/jvrmaia/hello
    go: creating new go.mod: module github.com/jvrmaia/hello
    $ cat go.mod
    module github.com/jvrmaia/hello

    go 1.13

Nosso primeiro código vai ser esse, `hello.go`:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

Feito isso, podes agora testar rodar nosso código, podemos fazer de 3 formas:

## go run

    $ go run hello.go

## go build

    $ go build hello.go
    $ ./hello

## go install

    $ go install
    $ hello

Repare que na terceira opção eu não precisei passar o caminho do binário pois foi inserido no meu `$PATH`. Como isso acontece? No caso, estou usando o Go instalado pelo [Brew](https://brew.sh/) e meu `$GOROOT` e `$GOPATH` configurados pelo `gvm` da seguinte forma:

    GOPATH=/home/jvrmaia/.gvm/pkgsets/system/global
    GOROOT=/home/linuxbrew/.linuxbrew/Cellar/go/1.13.6/libexec

Porém, isso não basta para ter funcionado, o `gvm` também cuida para mim de colocar o `$GOPATH` no meu `$PATH` o que garante esse funcionamento.

Agora que já executamos uma primeira versão desse nosso projeto vamos adicionar uma biblioteca nova a ele chamada `morestrings`:

    $ mkdir morestrings
    $ cat >morestrings.go<<EOF
    // Package morestrings implements additional functions to manipulate UTF-8
    // encoded strings, beyond what is provided in the standard "strings" package.
    package morestrings

    // ReverseRunes returns its argument string reversed rune-wise left to right.
    func ReverseRunes(s string) string {
        r := []rune(s)
        for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
            r[i], r[j] = r[j], r[i]
        }
        return string(r)
    }
    EOF

Feito isso, vamos adicionar ela no nosso projeto inicial do `hello`:

```go
package main

import (
	"fmt"

	"github.com/user/hello/morestrings"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```

Executando como visto antes vamos ter a reposta: `Hello, Go!`. Agora, vamos adicionar uma biblioteca externa no nosso projeto:

```go
package main

import (
	"fmt"

	"github.com/jvrmaia/hello/morestrings"
	"github.com/google/go-cmp/cmp"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
	fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```

Executando vamos ter o seguinte resultado:

```
$ go run hello.go
go: downloading github.com/google/go-cmp v0.4.0
go: extracting github.com/google/go-cmp v0.4.0
go: finding github.com/google/go-cmp v0.4.0
Hello, Go!
  string(
- 	"Hello World",
+ 	"Hello Go",
  )
```

E consequentemente algumas mudanças no seu `go.mod` e `go.sum` utilizado pelo Go Modules:

    $ cat go.mod
    module github.com/jvrmaia/hello

    go 1.13

    require github.com/google/go-cmp v0.4.0 // indirect
    $ cat go.sum
    github.com/google/go-cmp v0.4.0 h1:xsAVV57WRhGj6kEIi8ReJzQlHHqcBYCElAvkovg3B/4=
    github.com/google/go-cmp v0.4.0/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
    golang.org/x/xerrors v0.0.0-20191204190536-9bdfabe68543/go.mod h1:I/5z698sn9Ka8TeJc9MKroUUfqBBauWjQqLJ2OPfmY0=

Por fim e não menos importante, vamos colocar testes no projeto:

    cat >morestrings/reverse_test.go<<EOF
    package morestrings

    import "testing"

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
                t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
            }
        }
    }
    EOF

Para executar os testes é bem simples:

    $ go test -v ./morestrings/
    === RUN   TestReverseRunes
    --- PASS: TestReverseRunes (0.00s)
    PASS
    ok  	github.com/jvrmaia/hello/morestrings	0.002s

Bem legal, não?

# Uma proposta de organização de projeto

> This is an opinionated Go project template you can use as a starting point for your project. It doesn't include any code generation, so you'll need to replace the placeholder variables/values/names with your own.
>
> Clone the repository, keep what you need and delete everything else! Feel free to replace the parts that don't align with your use cases (e.g., you may prefer/use a different vendoring tool).
>
> See [Go Project Layout](https://github.com/golang-standards/project-layout) for a more generic and a less opinionated starting point for your project.

Essa é uma proposta de organização que me agrada bastante pois tem uma Makefile para ajudar a estruturar tarefas básicas como compilar, testar e preparar o empacotamento do projeto. As pastas são escolhidas de formas claras a entender onde estão os códigos e suas finalidades. No tópico seguinte vamos ver estrutura semelhante a essa em projetos de código aberto.

# Olhando projetos de código aberto

Vou demonstrar dois projetos abertos, um nacional e outro internacional que já estão com Go Modules. No meu github tem um código antigo que usa [go dep](https://github.com/jvrmaia/golang-testing-examples). Nos exemplos abaixo vamos ver uma s

## Tsuru

O repositório deles ficam [aqui](https://github.com/tsuru/tsuru). Como podem ver, já usam Go Modules e na raíz do projeto já começa o código com os packages separados por pastas com a pasta `vendor`.

## Kubernetes

O repositório deles ficam [aqui](https://github.com/kubernetes/kubernetes). Como podem ver, já usam Go Modules e na raíz do projeto já começa o código com os packages separados por pastas com a pasta `vendor`.

# O caso da Digital Ocean

> [Cthulhu: Organizing Go Code in a Scalable Repo](https://blog.digitalocean.com/cthulhu-organizing-go-code-in-a-scalable-repo/)

Como eles menos já definem no começo "mono repo"! Como vinha mostrando aqui no artigo a ideia é que cada projeto seu tenha o seu repositório. Nesse caso da Digital Ocean a estratégia foi juntar todos os projetos no mesmo repositório e compartilhar as dependências. Vale a pena ler e entender se os problemas e soluções fazem sentido ao seu contexto. Diga o que achou nos comentários!

# Um pouco mais sobre o $GOPATH

Na primeira parte do artigo no projeto `hello` fizemos um package e nessa mesma instalação e configuração de Go acabei testando algumas outras coisas, só lembrando o meu `$GOPATH`:

    GOPATH=/home/jvrmaia/.gvm/pkgsets/system/global

Agora, vamos inspecionar o que tem nele:

    $ ls /home/jvrmaia/.gvm/pkgsets/system/global
    bin/  pkg/  src/

Inspecionando mais afundo:

    $ tree -L 3 /home/jvrmaia/.gvm/pkgsets/system/global
    /home/jvrmaia/.gvm/pkgsets/system/global
    ├── bin
    │   └── hello
    ├── pkg
    │   └── mod
    │       ├── cache
    │       ├── github.com
    │       ├── gopkg.in
    │       └── go.uber.org
    └── src
        └── github.com
            ├── go-chi
            ├── golang
            ├── gorilla
            ├── go-sql-driver
            ├── miguelpragier
            └── tinrab

    15 directories, 1 file

Veja que tem várias outras não relacionadas ao `hello` e isso pode ser um problema no seu projeto se ficar compartilhando dependências entre projetos pois se estiverem em versões diferentes pode causar sérios problemas no seu projeto. Então, como resolver isso?

Como uso o gvm, eu gosto de usar um recurso que já vem nele de isolamento de projetos:

    $ gvm pkgset
    = gvm pkgset

    * http://github.com/moovweb/gvm

    == DESCRIPTION:

    GVM pkgset is used to manage various Go packages

    == Usage

    gvm pkgset Command

    == Command

    create     - create a new package set
    delete     - delete a package set
    use        - select where gb and goinstall target and link
    empty      - remove all code and compiled binaries from package set
    list       - list installed go packages

Como usar:

    $ gvm pkgset create hello
    $ gvm pkgset use hello
    Now using version system@hello
    $ env | grep GO
    GOPATH=/home/jvrmaia/.gvm/pkgsets/system/hello:/home/jvrmaia/.gvm/pkgsets/system/global
    GOROOT=/home/linuxbrew/.linuxbrew/Cellar/go/1.13.6/libexec

Agoram vamos repetir o passo de instalação do projeto igual vimos na primeira parte:

    $ go install
    go: downloading github.com/google/go-cmp v0.4.0
    go: extracting github.com/google/go-cmp v0.4.0
    go: finding github.com/google/go-cmp v0.4.0

Repare que ele instalou novamente a biblioteca `go-cmp`, por quê?

    $ ls /home/jvrmaia/.gvm/pkgsets/system/hello/pkg/mod/github.com/google/
    'go-cmp@v0.4.0'/

Porque agora ele foi instalada no ambiente que você criou com o `gvm`. Legal, não?

Nova localização do binário do `hello`:

    $ whereis hello
    hello: /home/jvrmaia/.gvm/pkgsets/system/hello/bin/hello /home/jvrmaia/.gvm/pkgsets/system/global/bin/hello

Mais informações sobre como funciona o `$PATH` podem ser lidas [aqui](https://medium.com/@jalendport/what-exactly-is-your-shell-path-2f076f02deb4).

# Outros artigos relacionados que valem a pena ler

- [How to structure a Go project?](https://vsupalov.com/go-folder-structure/)
- [Golang project layout](https://manfred.life/golang-project-layout)
- [Go Project Structure Best Practices](https://tutorialedge.net/golang/go-project-structure-best-practices/)
- [Simple Go project layout with modules](https://eli.thegreenplace.net/2019/simple-go-project-layout-with-modules/)
- [Examples for my talk on structuring go apps](https://github.com/katzien/go-structure-examples)
- [LondonGophers 19/09/2018: Kat Zien - How Do You Structure Your Go Apps?](https://www.youtube.com/watch?v=B5oQnECDJ8g&feature=emb_logo)

# Atualizações

- 17/01/2019: Contribuições do Joelson do Telegram Go Brasil
