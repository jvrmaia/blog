---
author: "João Maia"
date: 2020-02-17
title: "Hashing e criptografia"
tags: [
    "hashing",
    "criptografia",
    "iniciante",
]
categories: [
    "algoritmos"
]
mathjax: true
thumbnail: "img/hashing-e-criptografia/criptografia.jpeg"
---

Olá,

A ideia desse artigo é mostrar a diferenção entre as duas técnicas para quem ainda tem dúvidas sem se aprofundar muito no assunto, isso podemos explorar em artigos mais a frente aqui no blog. A motivação é que tive uma percepção no último ano de algumas pessoas misturando o conceitos e aplicações de ambos. Que tal começar com uma revisão de matemática?

# Visitando a matemática

Acredito que a maioria das pessoas que estão lendo esse artigo tenho completado os estudos até o ensino médio pelo menos. Então, vou assumir que todos tenham estudado teoria de conjuntos e funções. Se tem dúvidas sobre os dois assuntos segue uma [playlist](https://www.youtube.com/watch?v=2g0o0lzQin8&list=PL83s8LGM84J6Ew0fa1Ue2YZlNEqRXCBQo) do canal [Matemática Rio com Prof. Rafael Procopio](https://www.youtube.com/channel/UCjIPRjJZtGhzWD2LrEKOHMA) que mostra tudo sobre o assunto.

De forma breve, vamos revisitar algumas definições de forma simplificada:

- Função injetora: nunca aponta elementos distintos do seu domínio para o mesmo elemento de seu contradomínio;
- Função sobrejetora: é quando o seu contradomínio é igual a sua imagem;
- Função bijetora: é quando a função é injetora e sobrejetora ao mesmo tempo;
- Função inversa: sendo `f` uma função que mapeia valores do domínio `A` para o conjunto `B`, uma função inversa `g` é aquela que mapeio o domínio `B` para o contradomínio `A`.

![Tipos de funções](/img/hashing-e-criptografia/tipos-de-funcoes.jpg)

# Relacionamento entre matemática e computação

Independente da linguagem que você tenha experiência, de alguma forma é possível criar funções com ela, vamos escolher o Python aqui como exemplo por conta de sua simplicidade, segue um exemplo de função que calcula o dobre de um valor:

```python
>>> def dobro(a: int) -> int:
...     return 2 * a
...
>>> dobro(1)
2
>>>
```

A presentação matemática poderia ser nesse caso:

$$
\begin{equation}f: \mathbb{Z} \to \mathbb{Z}, y = f(x) = x * 2\end{equation}
$$

Onde `f` representa a função `dobro` do Python e `a` representa o `x` e o domínio da função é um número inteiro. Essa é uma representação bem simples mas muito importante para o que vamos ver mais a seguir. Outro exemplo interessante:

```python
>>> def quadrado(a: float) -> float:
...     return a * a
...
>>> quadrado(3.0)
9.0
>>>
```

A presentação matemática poderia ser nesse caso:

$$
\begin{equation}g: \mathbb{R} \to \mathbb{R} ^ + , y = g(x) = x ^ 2\end{equation}
$$

Olhando as funções `f` e `g` dado as suas definições de domínios, sabe dizer o que elas são? A resposta é simples, `f` no caso é injetora pois os números impares do contradomínio não estão presentes na imagem da função e na função `g` ela é uma função sobrejetora pois, por exemplo, para valores de `x` iguais a 3 ou -3 o valor é 9 e para todo valor de seu domínio vai ter um contradomínio correpondente, logo sua imagem vai ser o contradomínio.

Ficou faltando apenas uma das funções dentre as mencionadas no tópico anterior: bijetora. Você pode se perguntar por que a função `g` não é? Simples, não existe valor $$\mathbb{R} ^ +$$ cuja multiplicação por ele mesmo seja um número negativo. Então, para termos um exemplo simes de uma função bijetora, vamos ao caso mais trivial:

```python
>>> def identidade(a: int) -> int:
...     return a
...
>>> identidade(6)
6
>>>
```

$$
\begin{equation}h: \mathbb{Z} \to \mathbb{Z}, y = h(x) = x\end{equation}
$$

Agora que já fizemos uma introdução da matemática e um breve paralelo com a computação podemos dar continuidade ao tópico principal.

# Hashing

Um função de hash é aquela que transforma o seu valor de entrada em uma saída de tamanho fixo, em geral pequena, usualmente de 256 a 512 bits. Uma boa função de hash tem que ter algumas características para seu uso:

- a computação do seu valor deve ser eficiente para qualquer entrada;
- usando um valor de hash calculado deve ser extramamente complexo chegar no seu valor de entrada;
- deve ser complexo de mudar a entrada sem mudar o resultado do hash; e
- deve ser difícli encontrar duas entradas com o mesmo valor de hash.

São exemplos de função hash: MD5, SHA-1, SHA-256, SHA-3, and BLAKE2. As duas primeiras você programador que já utilizou git ou fez download de uma ISO linux ou software já devem ter visto elas antes. O MD5 é uma função de hash muito simples usada para você validar que fez o download de um arquivo correto, como isso funciona?

1. Vou criar um arquivo chamado `a.txt` com o conteúdo: `teste`;
2. Executo o `md5sum a.txt` para validar saber seu MD5: `1ca308df6cdb0a8bf40d59be2a17eac1`;
3. Envio o arquivo para uma outra pessoa qualquer;
4. A pessoa executa o mesmo comando do passo `2`; e
5. Avalia resultado:
    - Se o valor for diferente significa que o arquivo chegou corrompido ou adulterado no meio do caminho; ou
    - Se o valor for o mesmo, o arquivo foi recebido sem problemas.

E o git? Já reparou que toda vez que você faz um commit um valor fica associado aquela mudança? Exemplo, no último commit que fiz nesse blog de mensagem, `update`, ficou associado a ele um hash `92259c5e7f2ac0cce49c971312fd548c4dbbe07d` que é um resultado da função SHA-1.

# Criptografia

Criptografia não tem uma definição clara se você fizer uma busca mas olhando o significado da palavra vinda do grego temos: kryptós, "escondido", e gráphein, "escrita". Então, podemos encarar esse significa que refleto no uso como sua defição, a criptografia é largamente usada onde precisamos de criar uma comunicação segura, por exemplo, o HTTPS que usamos para abrir páginas na internet. Todo esse processo consiste em duas etapas que são: encriptação de decriptação. Na encriptação é a fase que pegados os dados puros e transformamos ele em dados embaralhados e a decriptação é o processo que reverte esse embaralhamento. Se você viu o filme [O Jogo da Imitação](https://www.imdb.com/title/tt2084970/) deve ter entendido do que estou falando, outro exemplo, quem na escola nunca criou alguma linguagem nova para se comunicar com os amigos para não serem entendidos tipo a [Língua do P](https://pt.wikipedia.org/wiki/L%C3%ADngua_do_P)?

Dentro desse universo de algoritmos a criptografia se divide em dois grupos: simétricas e assimétricas. Criptografia simétrica é aquela que uma chave de segredo será usada pelas duas partes da comunição, por exemplo, suponha um cofre que você e outra pessoa tenham a chave dele, então, vocês decidem conversar através de um bilhete que para cada um escrever e ler é preciso usar a chave para abrir o cofre, logo, vocês conseguem estabelecer uma comunicação restrita que existe apenas uma chave para abrir o cofre onde tem a informação. Na assimétrica o processo é diferente, nesse caso os comunicadores vão gerar duas chaves, uma privada e outra pública, os dois comunicadores vão fazer as trocas dessa chave pública para conseguirem se comunicar. Então, cada comunicador vai fazer a encriptação da mensagem com a chave pública do comunicador para que ele possa com a chave privada dele abrir a mensagem.

Nesse caso a criptografia se parece muito com a função bijetora para ter sua funcionalidade atendida mas isso não é verdade pois como podemos ver no parágrafo anterior existe um elementro extra que é a chave de criptografia. O que nos daria algo do tipo:

$$
\begin{equation}encrypt: (\mathbb{X},\mathbb{X}) \to \mathbb{X}, encrypt(msg, key) = crypt\\_msg\end{equation}
$$

$$
\begin{equation}decrypt: (\mathbb{X},\mathbb{X}) \to \mathbb{X}, decrypt(crypt\\_msg, key) = msg\end{equation}
$$

$$
\begin{equation}\mathbb{X}\ é\ um\ texto\ qualquer\end{equation}
$$

Exemplos de criptografias:
- Data Encryption Standard (DES) por Horst Feistel
- Advanced Encryption Standard (AES) por Rijndael
- Pretty Good Privacy (PGP) por Phil Zimmermann
- X.509 por International Telecommunications Union
- Transport Layer Security (TLS) por  Internet Engineering Task Force (IETF)

Vamos testar? Exemplo usando a [pycrypto](https://pypi.org/project/pycrypto/):

    >>> from Crypto.Cipher import AES
    >>> obj = AES.new('This is a key123', AES.MODE_CBC, 'This is an IV456')
    >>> message = "The answer is no"
    >>> ciphertext = obj.encrypt(message)
    >>> ciphertext
    '\xd6\x83\x8dd!VT\x92\xaa`A\x05\xe0\x9b\x8b\xf1'
    >>> obj2 = AES.new('This is a key123', AES.MODE_CBC, 'This is an IV456')
    >>> obj2.decrypt(ciphertext)
    'The answer is no'

# Conclusão

Como podemos ver hashing e criptografia não a mesma coisa e o assunto é bem complexo, um dos motivos de demorar a ter escrito esse artigo também. O importante é a mensagem que coloquei no começo de não escrever a sua criptografia ou hashing para uso na vida real se não tem a experiência devida. O legal é fazer para uso de estudos e caso se interessar pelo assunto contribuir com algum projeto que lide com isso, segue a [lista](https://en.wikipedia.org/wiki/Comparison_of_cryptography_libraries) de vários projetos de criptografia.
