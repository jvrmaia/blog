---
author: "João Maia"
date: 2020-10-10
title: "O básico do HTTP em Go"
tags: [
    "golang",
    "iniciante",
    "http"
]
categories: [
    "http"
]
thumbnail: "img/o-basico-do-http-em-go/golang-http.png"
---

Olá,

nesse novo artigo venho mostra o básico do que você precisa saber de Go quando for trabalhar com Go quando o assunto for [HTTP](https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol), seja do lado do servidor ou cliente. São dicas simples que podem te fazer evitar dores de cabeça no futuro caso não tenha cuidado com isso. Estou escrevendo esse artigo para ter uma referência alternativa em português a dois artigos da Cloudflare muito bons: [So you want to expose Go on the Internet](https://blog.cloudflare.com/exposing-go-on-the-internet/) e [The complete guide to Go net/http timeouts](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/).

# HTTP

Se você já trabalha com desenvolvimento Web acredito que já tenha domínio no assunto mas de forma bem simplificada podemos dizer que é uma padronização no formato da comunicação na Web que podemos dividir em dois contextos, quando você emite uma comunicação e quando você responde. Simplificando, quando você emite uma comunicação você define um verbo no qual vai se comunicar e o mais comum que vemos são GET, POST, PUT e DELETE, um cabeçalho mais conhecido pelo termo em inglês, Header, e por fim, o corpo da mensagem mais conhecido pelo termo em inglês, Body. Na parte da resposta temos basicamente os mesmos campos só trocando o verbo pelo código de retorno nas conhecido pelo termo em inglês, Status Code, que é um valor numérico que para um conjunto de valores tem um significado associado.

# Go

Já escrevi alguns outros artigos sobre Go no blog, para listar todos veja [aqui](https://blog.joaovrmaia.com/tags/golang/). Aqui só quero contextualizar algumas curiosidades sobre Go que são importantes para não só para trabalhar com HTTP mas vão te ajudar em outros momentos mais quando usar Go.

Vamos testar o seguinte código que você conferir online [aqui](https://play.golang.org/p/F3st816reYz):

```go
package main

import (
	"fmt"
	"time"
)

type exemplo struct {
	a int
	b string
	c bool
	d map[string]string
	e interface{}
	f float32
	g time.Duration
	h time.Time
	i *int
	j *string
	k *bool
	l *map[string]string
	m *interface{}
	n *float32
	o *time.Duration
	p *time.Time
}

func main() {
	fmt.Println(exemplo{})
}
```

Tente imaginar o que é esperado de retorno dessa função e compare com a resposta abaixo:

```
{0  false map[] <nil> 0 0 {0 0 <nil>} <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>}
```

Repare que para todos os tipos ele retornou o valor padrão, no caso nulo, do tipo e no caso quando era um ponteiro do valor retornou `nil` e detalhes entre um espaço maior no começo entre `0` e `false` que é por conta da `string` vazia. [Aqui](https://golang.org/ref/spec#The_zero_value) tem mais sobre como é isso em Go.

Vamos faze mais um teste que é na parte de lidar com a informação das requisições no seu formato mais comum hoje na Web que é o [JSON](https://www.json.org/json-pt.html). No caso para simplificar vou escrever o JSON direto na string ao invés de emular a requisição ([código](https://play.golang.org/p/LrWFLeDBWUY)):

```go
package main

import (
	"fmt"
	"encoding/json"
)

type Usuario struct {
	UUID    string `json:"uuid"`
	Name    string `json:"name"`
	Email   string `json:"email"`
	Twitter string `json:"twitter,omitempty"`
}

func main() {
	rawPayload := "{}"
	var u Usuario
	err := json.Unmarshal([]byte(rawPayload), &u)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(rawPayload)
	fmt.Println(u)

	fmt.Println("")

	rawPayload = "{\"uuid\":\"3954567d-67c2-4a1c-85b9-764a73bf3d5c\"}"
	err = json.Unmarshal([]byte(rawPayload), &u)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(rawPayload)
	fmt.Println(u)

	fmt.Println("")

	u = Usuario{
		UUID: "3954567d-67c2-4a1c-85b9-764a73bf3d5c",
		Name: "João",
	}
	body, err := json.Marshal(u)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(u)
	fmt.Println(string(body))
}
```

Resultado:

```
{}
{   }

{"uuid":"3954567d-67c2-4a1c-85b9-764a73bf3d5c"}
{3954567d-67c2-4a1c-85b9-764a73bf3d5c   }

{3954567d-67c2-4a1c-85b9-764a73bf3d5c João  }
{"uuid":"3954567d-67c2-4a1c-85b9-764a73bf3d5c","name":"João","email":""}
```

Como podemos ver no primeiro resultado como o corpo da mensagem veio vazio só foi gerado espaços em branco. No segundo como passamos o valor do UUID de acordo com o esperado ele foi impresso sem problemas. Por fim, invertemos, agora vamos montar o objeto e imprimir o corpo da mensagem e como podemos ver todos os campos informados foram impressos com um detalhes curioso entre `email` e `twitter`, o `email` foi impresso vazio e o `twitter` nem apareceu, isso foi por conta do `omitempty` declarado na definição da estrutura do objeto.

# Go e HTTP

Agora que já tivemos essa pré-conversa sobre HTTP e Go podemos juntar as duas coisas para conversar sobre como Go implementa o `package net/http` onde a documentação se encontra [aqui](https://golang.org/pkg/net/http/). Como já adiantamos a conversa vai ser em torno do lado [Client](https://golang.org/pkg/net/http/#Client) e do [Server](https://golang.org/pkg/net/http/#Server).

## Client

```go
type Client struct {
    // Transport specifies the mechanism by which individual
    // HTTP requests are made.
    // If nil, DefaultTransport is used.
    Transport RoundTripper

    // CheckRedirect specifies the policy for handling redirects.
    // If CheckRedirect is not nil, the client calls it before
    // following an HTTP redirect. The arguments req and via are
    // the upcoming request and the requests made already, oldest
    // first. If CheckRedirect returns an error, the Client's Get
    // method returns both the previous Response (with its Body
    // closed) and CheckRedirect's error (wrapped in a url.Error)
    // instead of issuing the Request req.
    // As a special case, if CheckRedirect returns ErrUseLastResponse,
    // then the most recent response is returned with its body
    // unclosed, along with a nil error.
    //
    // If CheckRedirect is nil, the Client uses its default policy,
    // which is to stop after 10 consecutive requests.
    CheckRedirect func(req *Request, via []*Request) error

    // Jar specifies the cookie jar.
    //
    // The Jar is used to insert relevant cookies into every
    // outbound Request and is updated with the cookie values
    // of every inbound Response. The Jar is consulted for every
    // redirect that the Client follows.
    //
    // If Jar is nil, cookies are only sent if they are explicitly
    // set on the Request.
    Jar CookieJar

    // Timeout specifies a time limit for requests made by this
    // Client. The timeout includes connection time, any
    // redirects, and reading the response body. The timer remains
    // running after Get, Head, Post, or Do return and will
    // interrupt reading of the Response.Body.
    //
    // A Timeout of zero means no timeout.
    //
    // The Client cancels requests to the underlying Transport
    // as if the Request's Context ended.
    //
    // For compatibility, the Client will also use the deprecated
    // CancelRequest method on Transport if found. New
    // RoundTripper implementations should use the Request's Context
    // for cancellation instead of implementing CancelRequest.
    Timeout time.Duration // Go 1.3
}
```

Podemos ver que temos algumas configurações interessantes na estrutura do `Client` mas hoje vamos falar apenas do `Timeout` que é onde um erro pode custar caro.

> A Timeout of zero means no timeout.

Como vimos na parte anterior o Go trabalha com valores nulos por padrão e na configuração de `Timeout` como podemos ver no trecho em destaque se o valor for nulo significa que não tem expiração da requisição, segue uma imagem na prática de quando isso acontece:

![Google Chrome Timeout](/img/o-basico-do-http-em-go/google-chrome-timeout.png)

Se ainda não ficou claro o conceito de tempo de expiração vamos aproveitar o nosso momento atual de pandemia que estamos fazendo reuniões de forma virtual e eventos online, é bem provável que você já tenha participado de alguma que não tinha membros suficientes ainda para começar e alguém dito: "Vamos esperar 5 minutos e então começamos", esse é o tempo de expiração.

Lembra que na parte de HTTP falamos que a resposta do servidor de forma simples é o estado de retorno, cabeçalho e corpo da mensagem? Então, esse configuração de expiração é para até o momento que o corpo da mensagem tenha sido completamente lido senão temos esse erro igual na imagem do Google Chrome acima.

## Server

```go
type Server struct {
    // Addr optionally specifies the TCP address for the server to listen on,
    // in the form "host:port". If empty, ":http" (port 80) is used.
    // The service names are defined in RFC 6335 and assigned by IANA.
    // See net.Dial for details of the address format.
    Addr string

    Handler Handler // handler to invoke, http.DefaultServeMux if nil

    // TLSConfig optionally provides a TLS configuration for use
    // by ServeTLS and ListenAndServeTLS. Note that this value is
    // cloned by ServeTLS and ListenAndServeTLS, so it's not
    // possible to modify the configuration with methods like
    // tls.Config.SetSessionTicketKeys. To use
    // SetSessionTicketKeys, use Server.Serve with a TLS Listener
    // instead.
    TLSConfig *tls.Config

    // ReadTimeout is the maximum duration for reading the entire
    // request, including the body.
    //
    // Because ReadTimeout does not let Handlers make per-request
    // decisions on each request body's acceptable deadline or
    // upload rate, most users will prefer to use
    // ReadHeaderTimeout. It is valid to use them both.
    ReadTimeout time.Duration

    // ReadHeaderTimeout is the amount of time allowed to read
    // request headers. The connection's read deadline is reset
    // after reading the headers and the Handler can decide what
    // is considered too slow for the body. If ReadHeaderTimeout
    // is zero, the value of ReadTimeout is used. If both are
    // zero, there is no timeout.
    ReadHeaderTimeout time.Duration // Go 1.8

    // WriteTimeout is the maximum duration before timing out
    // writes of the response. It is reset whenever a new
    // request's header is read. Like ReadTimeout, it does not
    // let Handlers make decisions on a per-request basis.
    WriteTimeout time.Duration

    // IdleTimeout is the maximum amount of time to wait for the
    // next request when keep-alives are enabled. If IdleTimeout
    // is zero, the value of ReadTimeout is used. If both are
    // zero, there is no timeout.
    IdleTimeout time.Duration // Go 1.8

    // MaxHeaderBytes controls the maximum number of bytes the
    // server will read parsing the request header's keys and
    // values, including the request line. It does not limit the
    // size of the request body.
    // If zero, DefaultMaxHeaderBytes is used.
    MaxHeaderBytes int

    // TLSNextProto optionally specifies a function to take over
    // ownership of the provided TLS connection when an ALPN
    // protocol upgrade has occurred. The map key is the protocol
    // name negotiated. The Handler argument should be used to
    // handle HTTP requests and will initialize the Request's TLS
    // and RemoteAddr if not already set. The connection is
    // automatically closed when the function returns.
    // If TLSNextProto is not nil, HTTP/2 support is not enabled
    // automatically.
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler) // Go 1.1

    // ConnState specifies an optional callback function that is
    // called when a client connection changes state. See the
    // ConnState type and associated constants for details.
    ConnState func(net.Conn, ConnState) // Go 1.3

    // ErrorLog specifies an optional logger for errors accepting
    // connections, unexpected behavior from handlers, and
    // underlying FileSystem errors.
    // If nil, logging is done via the log package's standard logger.
    ErrorLog *log.Logger // Go 1.3

    // BaseContext optionally specifies a function that returns
    // the base context for incoming requests on this server.
    // The provided Listener is the specific Listener that's
    // about to start accepting requests.
    // If BaseContext is nil, the default is context.Background().
    // If non-nil, it must return a non-nil context.
    BaseContext func(net.Listener) context.Context // Go 1.13

    // ConnContext optionally specifies a function that modifies
    // the context used for a new connection c. The provided ctx
    // is derived from the base context and has a ServerContextKey
    // value.
    ConnContext func(ctx context.Context, c net.Conn) context.Context // Go 1.13
    // contains filtered or unexported fields
}
```

# Referências

- HTTP: https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol
- So you want to expose Go on the Internet: https://blog.cloudflare.com/exposing-go-on-the-internet/
- The complete guide to Go net/http timeouts: https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/
- Artigos antigos de Go no Blog: https://blog.joaovrmaia.com/tags/golang/
- Primeiro código de exemplo: https://play.golang.org/p/F3st816reYz
- Zero Value me Go: https://golang.org/ref/spec#The_zero_value
- JSON: https://www.json.org/json-pt.html
- Código do exemplo de JSON: https://play.golang.org/p/LrWFLeDBWUY
- Documentação do `net/http` Go package: https://golang.org/pkg/net/http/
- Go `net/http` Client: https://golang.org/pkg/net/http/#Client
- Go `net/http` Server: https://golang.org/pkg/net/http/#Server