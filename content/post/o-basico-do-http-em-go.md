---
author: "João Maia"
date: 2021-03-07
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

Vamos fazer mais um teste que é na parte de lidar com a informação das requisições no seu formato mais comum hoje na Web que é o [JSON](https://www.json.org/json-pt.html). No caso para simplificar vou escrever o JSON direto na string ao invés de emular a requisição ([código](https://play.golang.org/p/LrWFLeDBWUY)):

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

Podemos ver que temos algumas configurações interessantes na estrutura do `Client`, vamos falar do `Timeout` que é onde um erro pode custar caro.

> A Timeout of zero means no timeout.

Como vimos na parte anterior o Go trabalha com valores nulos por padrão e na configuração de `Timeout` como podemos ver no trecho em destaque se o valor for nulo significa que não tem expiração da requisição, segue uma imagem na prática de quando isso acontece:

![Google Chrome Timeout](/img/o-basico-do-http-em-go/google-chrome-timeout.png)

Se ainda não ficou claro o conceito de tempo de expiração vamos aproveitar o nosso momento atual de pandemia que estamos fazendo reuniões de forma virtual e eventos online, é bem provável que você já tenha participado de alguma que não tinha membros suficientes ainda para começar e alguém dito: "Vamos esperar 5 minutos e então começamos", esse é o tempo de expiração.

Lembra que na parte de HTTP falamos que a resposta do servidor de forma simples é o estado de retorno, cabeçalho e corpo da mensagem? Então, esse configuração de expiração é para até o momento que o corpo da mensagem tenha sido completamente lido senão temos esse erro igual na imagem do Google Chrome acima.

Outra parte importante do `Client` é essa parte de `Transport` que vai ter várias configurações importantes:

```go
type Transport struct {

    // Proxy specifies a function to return a proxy for a given
    // Request. If the function returns a non-nil error, the
    // request is aborted with the provided error.
    //
    // The proxy type is determined by the URL scheme. "http",
    // "https", and "socks5" are supported. If the scheme is empty,
    // "http" is assumed.
    //
    // If Proxy is nil or returns a nil *URL, no proxy is used.
    Proxy func(*Request) (*url.URL, error)

    // DialContext specifies the dial function for creating unencrypted TCP connections.
    // If DialContext is nil (and the deprecated Dial below is also nil),
    // then the transport dials using package net.
    //
    // DialContext runs concurrently with calls to RoundTrip.
    // A RoundTrip call that initiates a dial may end up using
    // a connection dialed previously when the earlier connection
    // becomes idle before the later DialContext completes.
    DialContext func(ctx context.Context, network, addr string) (net.Conn, error) // Go 1.7

    // Dial specifies the dial function for creating unencrypted TCP connections.
    //
    // Dial runs concurrently with calls to RoundTrip.
    // A RoundTrip call that initiates a dial may end up using
    // a connection dialed previously when the earlier connection
    // becomes idle before the later Dial completes.
    //
    // Deprecated: Use DialContext instead, which allows the transport
    // to cancel dials as soon as they are no longer needed.
    // If both are set, DialContext takes priority.
    Dial func(network, addr string) (net.Conn, error)

    // DialTLSContext specifies an optional dial function for creating
    // TLS connections for non-proxied HTTPS requests.
    //
    // If DialTLSContext is nil (and the deprecated DialTLS below is also nil),
    // DialContext and TLSClientConfig are used.
    //
    // If DialTLSContext is set, the Dial and DialContext hooks are not used for HTTPS
    // requests and the TLSClientConfig and TLSHandshakeTimeout
    // are ignored. The returned net.Conn is assumed to already be
    // past the TLS handshake.
    DialTLSContext func(ctx context.Context, network, addr string) (net.Conn, error) // Go 1.14

    // DialTLS specifies an optional dial function for creating
    // TLS connections for non-proxied HTTPS requests.
    //
    // Deprecated: Use DialTLSContext instead, which allows the transport
    // to cancel dials as soon as they are no longer needed.
    // If both are set, DialTLSContext takes priority.
    DialTLS func(network, addr string) (net.Conn, error) // Go 1.4

    // TLSClientConfig specifies the TLS configuration to use with
    // tls.Client.
    // If nil, the default configuration is used.
    // If non-nil, HTTP/2 support may not be enabled by default.
    TLSClientConfig *tls.Config

    // TLSHandshakeTimeout specifies the maximum amount of time waiting to
    // wait for a TLS handshake. Zero means no timeout.
    TLSHandshakeTimeout time.Duration // Go 1.3

    // DisableKeepAlives, if true, disables HTTP keep-alives and
    // will only use the connection to the server for a single
    // HTTP request.
    //
    // This is unrelated to the similarly named TCP keep-alives.
    DisableKeepAlives bool

    // DisableCompression, if true, prevents the Transport from
    // requesting compression with an "Accept-Encoding: gzip"
    // request header when the Request contains no existing
    // Accept-Encoding value. If the Transport requests gzip on
    // its own and gets a gzipped response, it's transparently
    // decoded in the Response.Body. However, if the user
    // explicitly requested gzip it is not automatically
    // uncompressed.
    DisableCompression bool

    // MaxIdleConns controls the maximum number of idle (keep-alive)
    // connections across all hosts. Zero means no limit.
    MaxIdleConns int // Go 1.7

    // MaxIdleConnsPerHost, if non-zero, controls the maximum idle
    // (keep-alive) connections to keep per-host. If zero,
    // DefaultMaxIdleConnsPerHost is used.
    MaxIdleConnsPerHost int

    // MaxConnsPerHost optionally limits the total number of
    // connections per host, including connections in the dialing,
    // active, and idle states. On limit violation, dials will block.
    //
    // Zero means no limit.
    MaxConnsPerHost int // Go 1.11

    // IdleConnTimeout is the maximum amount of time an idle
    // (keep-alive) connection will remain idle before closing
    // itself.
    // Zero means no limit.
    IdleConnTimeout time.Duration // Go 1.7

    // ResponseHeaderTimeout, if non-zero, specifies the amount of
    // time to wait for a server's response headers after fully
    // writing the request (including its body, if any). This
    // time does not include the time to read the response body.
    ResponseHeaderTimeout time.Duration // Go 1.1

    // ExpectContinueTimeout, if non-zero, specifies the amount of
    // time to wait for a server's first response headers after fully
    // writing the request headers if the request has an
    // "Expect: 100-continue" header. Zero means no timeout and
    // causes the body to be sent immediately, without
    // waiting for the server to approve.
    // This time does not include the time to send the request header.
    ExpectContinueTimeout time.Duration // Go 1.6

    // TLSNextProto specifies how the Transport switches to an
    // alternate protocol (such as HTTP/2) after a TLS ALPN
    // protocol negotiation. If Transport dials an TLS connection
    // with a non-empty protocol name and TLSNextProto contains a
    // map entry for that key (such as "h2"), then the func is
    // called with the request's authority (such as "example.com"
    // or "example.com:1234") and the TLS connection. The function
    // must return a RoundTripper that then handles the request.
    // If TLSNextProto is not nil, HTTP/2 support is not enabled
    // automatically.
    TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper // Go 1.6

    // ProxyConnectHeader optionally specifies headers to send to
    // proxies during CONNECT requests.
    // To set the header dynamically, see GetProxyConnectHeader.
    ProxyConnectHeader Header // Go 1.8

    // GetProxyConnectHeader optionally specifies a func to return
    // headers to send to proxyURL during a CONNECT request to the
    // ip:port target.
    // If it returns an error, the Transport's RoundTrip fails with
    // that error. It can return (nil, nil) to not add headers.
    // If GetProxyConnectHeader is non-nil, ProxyConnectHeader is
    // ignored.
    GetProxyConnectHeader func(ctx context.Context, proxyURL *url.URL, target string) (Header, error) // Go 1.16

    // MaxResponseHeaderBytes specifies a limit on how many
    // response bytes are allowed in the server's response
    // header.
    //
    // Zero means to use a default limit.
    MaxResponseHeaderBytes int64 // Go 1.7

    // WriteBufferSize specifies the size of the write buffer used
    // when writing to the transport.
    // If zero, a default (currently 4KB) is used.
    WriteBufferSize int // Go 1.13

    // ReadBufferSize specifies the size of the read buffer used
    // when reading from the transport.
    // If zero, a default (currently 4KB) is used.
    ReadBufferSize int // Go 1.13

    // ForceAttemptHTTP2 controls whether HTTP/2 is enabled when a non-zero
    // Dial, DialTLS, or DialContext func or TLSClientConfig is provided.
    // By default, use of any those fields conservatively disables HTTP/2.
    // To use a custom dialer or TLS config and still attempt HTTP/2
    // upgrades, set this to true.
    ForceAttemptHTTP2 bool // Go 1.13
    // contains filtered or unexported fields
}
```

Nele é interessante notar que podemos fazer o uso de `Proxy` caso seja um requisito seu, você também pode criar conexões customizadas com as funções `Dial` e configurar o `TLS Client`. Esses recursos são legais mas acabam sendo casos bem pontuais no nosso projeto e não comuns. O que vale mais a pena ficar atento aqui são outros atributos que vão estar fortemente ligados a resiliência do nosso `HTTP Client`, são eles:
- `DisableKeepAlives`: é uma configuração que existe nos dois contextos e é bem interessante quando habilitada pois permite você saber ser a comunicação entre cliente e servidor não foi quebrada;
- `MaxIdleConns` e `MaxIdleConnsPerHost`: essa é uma configuração casada com a anterior, pois para ter essa monitoria é preciso manter uma conexão aberta entre cliente e servidor para saber da saúde desse canal, essa configuração limita a quantidade de conexões desse tipo que podem existir, isso é bem importante para você não lotar sua aplicação de conexões que não estão transferindo dados e apenas validando a contectividade;
- `IdleConnTimeout`: se você usar o recurso do TCP Keep-alive é também importante ficar atento a isso pois se o valor padrão é não ter limite e isso pode fazer sua aplicação chegar no valor máximo de conexões abertas mais rápido; e
- `ForceAttemptHTTP2`: se você pode, eu recomendo. =)

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

Assim como o `Client` o `Server` tem várias configurações importantes e precisamos tomar bastante cuidado com elas, aqui também vamos ter questões a serem configuradas sobre `Timeout`:
- `ReadTimeout`: esse é o tempo que seu servidor vai esperar para ler todas as informações da requisição que está recebendo e caso nulo não terá limite de tempo;
- `ReadHeaderTimeout`:  parecido com o anterior mas apenas para o Header, caso ele não seja definido usará o `ReadTimeout` caso tenha sido definido;
- `WriteTimeout`: esse é o tempo de vida total da requisição no seu servidor e caso não configurado a requisição pode ficar presa eternamente no seu servidor causando danos a ele; e
- `IdleTimeout`: é o tempo máximo que o seu servidor vai ficar disponível ao cliente sem troca de dados efetivas.

Além disso, temos como configurar o TLS através de `TLSConfig` que é muito importante ter cuidado caso sua aplicação fique acessível diretamente na Internet. Algumas outras configurações que são importante ter cuidado:
- `Addr`: aqui você configura o endereço que seu servidor vai escutar requisições, por exemplo, "127.0.0.1:3000" significa que apenas requisições vindo da própria máquina que roda o servidor na porta 3000 vão ser aceitas, o padrão é toda requisição na porta 80;
- `Handler`: nesse componentes cadastramos as rotas e métodos que esse nosso servidor vai responder; e
- `ErrorLog`: caso você tenha um sistema de agregação de logs customizados você pode definir ele aqui.

# Conclusão

É muito importante estar atento a esses detalhes do Go sobre valores padrão pois isso pode nos causar grandes problemas com comportamentos não esperados, mesmo você usando bibliotecas que fazem abstrações dessa parte nativa do HTTP do Go sofrem do mesmo problema falando das bibliotecas mais populares. Caso tenha interesse em ver isso mais na prática eu tenho um projeto no meu GitLab para testar esses comportamentos, confira ele [aqui](https://gitlab.com/jvrmaia/golang-http-playground).

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
- Go HTTP playground: https://gitlab.com/jvrmaia/golang-http-playground