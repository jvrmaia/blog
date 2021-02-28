---
author: "João Maia"
date: 2020-08-28
title: "Git e afins para iniciantes - parte 2"
tags: [
    "git",
    "github",
    "gitlab",
    "bitbucket",
]
categories: [
    "desenvolvimento"
]
thumbnail: "img/git-e-afins-para-iniciantes-parte-2/git.png"
---

Olá,

no [último artigo](https://blog.joaovrmaia.com/post/git-e-afins-para-iniciantes-parte-1/) encerramos na parte do Git como serviço que é onde começamos a explorar o melhor recurso dele que é o poder pessoas contribuirem umas com as outras. Neste novo artigo vou comentar sobre as opções mais conhecidas hoje serviços de Git, mostrar alguns padrões e formas de uso e dar minha opinião a respeito disso e a polêmica questão se o Github é ou não o seu curriculo.

# Git como serviço

O que estou querendo dizer com essa expressão? Vamos fazer um comparativo simples, o [Wordpress](https://wordpress.org/) que é o sistema mais famoso de blogs é de códio aberto e com ele você pode criar seu blog a sua maneira mas você também pode usar o [Wordpress como serviço](https://wordpress.com/) pois por trás do programa existe uma empresa que fornece toda uma infraestrutura e conhecimento sobre o sistema para você ter seu site sem precisar saber muito sobre o assunto e mais diversos outros componentes que agregam valor ao ecossistema. Então, quando dizemos Git como serviço estamos falando do mesmo Git que você usa na sua máquina só que agora em um local onde mais pessoas vão ter acesso fácil e teram diversos outros componentes enriquecendo seu ecossistema, por exemplo, esse blog é um dos componentes desse ecossitema que nele é chamado de [Github Pages](https://pages.github.com/) onde um repositório é convertido em uma página web com algumas limitações.

Nesse artigo vou me limitar a falar apenas das mais conhecidas e que tive experiência prática: [Github](https://github.com/), [Gitlab](https://about.gitlab.com/) e [Bitbucket](https://bitbucket.org/).

## Github

![Github](/img/git-e-afins-para-iniciantes-parte-2/github.png)

Podemos dar crédito ao Github por essa transformação da forma de versionar código e consolidação do Git como solução para isso. Com um modelo de negócio onde respositórios abertos poderiam ser criados de maneira gratuita acabou sendo usado por muitos projetos de código aberto o que aliado com uma boa experiência do usuário fez com que se tornasse popular. Hoje o Github é da [Microsoft](https://www.microsoft.com/pt-br) que comprou em 2018, uma empresa construída em 2007 usando o um framework de muito sucesso na época que ainda é amplamente utilizado ainda hoje, [Ruby On Rails](https://rubyonrails.org/). Com a chegada da Microsoft e movimentação dos concorrentes o Github nos últimos anos tem dados grandes passos para ser muito mais que um serviço de repositório de código. Hoje, ele já consta com hospedagm de sites estáticos, repositório de bibliotecas, ferramentas de integração e entrega contínua e muito mais.

## Gitlab

![Gitlab](/img/git-e-afins-para-iniciantes-parte-2/gitlab.png)

Um concorrente ao Github que surgiu em 2013 de código fonte aberto e livre também com raízes de desenvolvimento da época feito em [Ruby](https://www.ruby-lang.org/pt/). O projeto foi dívido em dois, uma edição comunitária e outra corporativa. Afinal, todo mundo tem seus boletos para pagar e esse modelo de oferecer uma versão aberta que você pode gerenciar completamente e outra comercial onde a carga de gerenciar recursos e afim é de um terceiro já não era novidade. O Gitlab além de repositório de códigos também tem recursos semelhantes ao Github mas o que é mais conhecido são as ferramentas de integração e entrega contínua, acredito que além do fato de poder rodar na sua própria infraestrutura esse é um fator de grande peso do uso do Gitlab em muitas empresas. Um curiosidade, o Gitlab teve alguns momentos de grandes migração de projetos e pessoas para seu produto na história, os mais relevantes: 1) fim do [Gitorious](https://en.wikipedia.org/wiki/Gitorious); 2) migração do projeto [Gnome](https://www.gnome.org/); e 3) a compra da Github pela Microsoft.

## Bitbucket

![Bitbucket](/img/git-e-afins-para-iniciantes-parte-2/bitbucket.png)

O Bitbucket nasceu em 2008 e ao contrário dos seus concorrentes já apresentados aqui começou em [Python](https://www.python.org/) e hoje é em [Java](https://www.java.com/pt_BR/). Outra cuiriosidade, na época os projetos da comunidade de Python usavam mais o [Mercurial](https://www.mercurial-scm.org/) como mecanismo de controle de versão e o Bitbucket não foi diferente nisso, era apenas para projetos Mercurial. Em 2010, eles foram comprados pela [Atlassian](https://www.atlassian.com/br) e em 2011 o suporte ao Git foi lançado e em 2015 o produto [Stash](https://en.wikipedia.org/wiki/Stash_(software)) que era o Bitbucket vendido pela Atlassian foi renomeado para Bitbucket Server. O Bitbucket também possui ferramentas de integração e entrega contínua mas por conta da Atlassian ser uma empresa que no seu portifólio conter vários produtos que juntos entregam todo um sistema de ferramentas básicas para empresas isso acaba ficando descentralizado na ferramenta.

# As importâncias sociais e econômicas

Depois de contar um pouco de história, vamos falar agora de outros aspectos de serviços como esses na nossa sociedade. Vamos falar do ponto social primeiro, com os projetos abertos é possível um programador aqui do sudeste do Brasil como eu ter contato com programadores de todo o Brasil e mundo. Isso é fenomenal pois imagina que você mora em uma região onde o cenário tecnológico ainda não é forte, você vai poder se conectar com pessoas e trocar muito conhecimento, e mesmo você sendo tímido isso aindo pode acontecer pelo simples fato de você pode ler o trabalho dos outros e aprender a partir disso. Além disso, não é só a troca de conhecimento que pode ser obtida através desses serviços, você também vai poder juntar com outras pessoas e construir projetos que podem impactar a vida de muitas pessoas, o próprio [Linux](https://www.kernel.org/) que já falei lá no primeiro artigo que gerou a necessidade do Git existir é um exemplo disso.

Vendo agora do ponto de vista economico, imagine um mundo sem software livre. Teriamos soluções mais fechadas a grandes empresas e monopólios, o processo de inovação seria péssimo, o re-trabalho seria enorme e muito mais. Felizmente, o mundo não é assim e temos diversos softwares livres, veja esse blog, por exemplo, não tenho muita habilidade com design, criação de páginas web e afins mas sou capaz de ter esse blog com um visuál agradável com extrema felicidade pois pessoas fizeram um trabalho por mim de criar isso e deixar de forma livre. O site é gerado por um sistema de criação de sites estáticos chamado [Hugo](https://gohugo.io/) e o tema, [Mainroad](https://github.com/Vimux/Mainroad), foi criado por outra pessoa que também deixou de forma livre no Github. Além disso, imagina a quantidade de empresas e órgão de governos que no ano conseguem economizar dinheiro para realizar suas atividades pois tem nos seus pilares o uso de software livre evitando a compra de licença de software. Sensacional!

# Trabalhando de forma colaborativa com o Git

![Git flows](/img/git-e-afins-para-iniciantes-parte-2/git-flows.png)

Agora vamos voltar ao assunto técnico e começar a entender como o Git funciona para trabalhar de forma colaborativa. Na imagem acima ilustra bem tudo que acontece e adiciona mais informações que não tinhamos abordados antes no primeiro artigo. Vamos imaginar que nosso projeto consiste de camadas como uma lasanha ou bolo para quem prefere doce e quem prefere agridoce coloca no comentário a opção. A primeira camada que vem ali na base de apoio é o seu `Workspace` que é onde você trabalha ativamente modificando o projeto. Quando você cria uma arquivo e adicionar ele no repositório com um `git add` esse arquivo vai para uma camada acima que é o `Staging` seria como ter feito um acordo verbal com seu repositório mas não no papel de forma oficial. Pensando ainda nesse arquivo, quando você faz o commit do que está feito em `Staging` suas alterações são de fato adicionadas a história do projeto mas apenas de forma local. Então, chega a hora de começar a trabalhar de forma colaborativa, vamos precisar conectar esse nosso repositório local a um repositório remoto.

Vamos observar esse próprio blog que é um repositório no Github, para isso vamos precisar observar um aquivo de configuração que fica dentro do projeto em `.git/config`:

```
$ cat .git/config 
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = git@github.com:jvrmaia/blog.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "published"]
	remote = origin
	merge = refs/heads/published
```

Como podemos ver temos uma configuração de `remote` no projeto que faz referência a um projeto remoto e assim fazemos o enlace entre o projeto local e remoto. Como disse, no meu caso uso o Github para isso e por isso consta aquela configuração de [URL](https://pt.wikipedia.org/wiki/URL). Repare que não é bem uma URL como estamos acostumados com sites mas é por conta do protocolo usado para me comunicar com o Github ser o [SSH](https://pt.wikipedia.org/wiki/Secure_Shell). Caso tivesse colocado uma configuração de URL usando [HTTP](https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol) eu teria que em todo o processo de enviar ou pegar código no Github ter que passar pelo processo de login, com o SSH conseguimos criar essa facilidade no processo.

Vamos pegar esse blog de exemplo para entender os fluxos que mostrei no começo. Supondo que você tenha achado algum erro na escrita do texto ou queria fazer alguma melhoria no design a primeira coisa que você precisa fazer é obter o [projeto](https://github.com/jvrmaia/blog). A primeira coisa que você precisa fazer é o chamado `Fork` do projeto que pode ser feito no Github, basta entrar na página do repositório desse blog e clicar no botão de Fork e criar um na sua conta. Veja um exemplo disso meu fazendo um Fork no projeto do Linux Kernel:

![Página do projeto com destaque no Fork](/img/git-e-afins-para-iniciantes-parte-2/linux-fork.png)

Fork feito é hora de copiar para sua máquina o repositório. Para isso precisamos pegar a URL para clonar o projeto conforme demonstrado na figura abaixo no caso para o meu blog.

![Clonando o projeto do blog](/img/git-e-afins-para-iniciantes-parte-2/git-clone.png)

Vamos executar o clone do repositório:

```
$ git clone git@github.com:jvrmaia/blog.git
Cloning into 'blog'...
remote: Enumerating objects: 394, done.
remote: Counting objects: 100% (394/394), done.
remote: Compressing objects: 100% (241/241), done.
remote: Total 2985 (delta 187), reused 320 (delta 123), pack-reused 2591
Receiving objects: 100% (2985/2985), 6.21 MiB | 794.00 KiB/s, done.
Resolving deltas: 100% (1525/1525), done.
```

Agora, você pode executar modificações a vontade para enviar para alteração. Veja o estado atual do repo enquanto escrevo o artigo:

```
$ git status
On branch post/git-2
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   content/post/git-e-afins-para-iniciantes-parte-2.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	static/img/git-e-afins-para-iniciantes-parte-2/git-clone.png
	static/img/git-e-afins-para-iniciantes-parte-2/linux-fork.png

no changes added to commit (use "git add" and/or "git commit -a")
```

Repare que estou escrevendo ele em uma branch separada chamada "post/git-2", já tenho um arquivo alterado que é o [Markdown](https://pt.wikipedia.org/wiki/Markdown) do artigo e duas imagens não presentes ainda no histórico do Git desse projeto, esse é o meu Workspace. Vamos colocar agora essas alterações em Staging:

```
$ git add  .
$ git status
On branch post/git-2
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   content/post/git-e-afins-para-iniciantes-parte-2.md
	new file:   static/img/git-e-afins-para-iniciantes-parte-2/git-clone.png
	new file:   static/img/git-e-afins-para-iniciantes-parte-2/linux-fork.png
```

Pronto, agora temos as alterações salvas em Staging e podemos fazer o commit delas no repositório local, veja:

```
$ git commit -m "trabalho em progresso"
[post/git-2 cd8e683] trabalho em progresso
 3 files changed, 91 insertions(+), 5 deletions(-)
 create mode 100644 static/img/git-e-afins-para-iniciantes-parte-2/git-clone.png
 create mode 100644 static/img/git-e-afins-para-iniciantes-parte-2/linux-fork.png
 ```

Por fim, podemos agora enviar as alterações de volta para o Github que é o repositório remoto do projeto:

```
$ git push origin post/git-2 
Enumerating objects: 17, done.
Counting objects: 100% (17/17), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 101.54 KiB | 590.00 KiB/s, done.
Total 10 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
To github.com:jvrmaia/blog.git
   1d4dc6f..cd8e683  post/git-2 -> post/git-2
```

Alterações enviadas e agora vamos no Github ver como ficou e termos uma surpresa:

![Interface do Github sugerindo PR](/img/git-e-afins-para-iniciantes-parte-2/sugestao-pr.png)

O Github conseguiu entender que poderia ser um `Pull Request` (PR) essa atualização da branch e já fez a sugestão criando um link para facilitar o processo, vamos apertar e ver o que acontece:

![Tela do PR](/img/git-e-afins-para-iniciantes-parte-2/tela-do-pr.png)

No seu caso vai ser um pouco diferente pois a base vai ser o meu repositório e não o seu Fork mas a ideia é a mesma. Seu PR vai precisar de um título e um corpo de texto, o Github fornece recurso de arquivos de exemplo para isso mas não é o meu caso aqui pois não faço uso mas no trabalho fazemos para ajudar a manter um padrão. Repare também que é possível adicionar quem vai revisar, quem é o resposável pelo PR, rótulos e associar a uma entrega de projeto. Como disse, o Github hoje é muito mais que um local para compartilhar repositórios.

Para fechar essa primeira conversa sobre Git como serviço ficaram faltando dois comandos também interessantes mas que vou deixar aqui para você executar pois só vou comentar o que ele fazem:

- `git pull origin post/git-2`: supondo que em outro computador eu tenha esse mesmo projeto com essa branch lá, vou poder pegar todas as alterações feitas na branch desde a última vez que atualizei ela; e
- `git fetch --all --prune`: esse é um comando que rodo com frequência aqui todas as vezes que vou começar a trabalhar em um projeto no dia pois ele pega tudo que aconteceu no repositório remoto e me atualiza.

Acredito que agora juntando com o que foi visto no primeiro artigo você já tenha informação suficiente para começar a entrar nesse mundo de código colaborativo. Agora, vamos fechar o texto com um pouco de polêmica.

# Github/Gitlab/Bitbucket e afins são seu currículo?

A resposta curta e direta é não! Vou apresentar meus pontos que justificam isso.

Seu currículo é sua carta de apresentação para uma empresa, ou seja, para cada empresa vai ter uma forma certa de você se apresentar. Então, dizer que uma ferramenta que apenas exibe o seu perfil de uso dela e alguns códigos  e discussões com o seu envolvimento sem nenhum contexto a respeito disso não pode ser usado para definir quem você é.

O seu Github ou outro serviço semelhante qualquer apesar de não serem seu currículo podem jogar a seu favor em entrevistas pois é natural serem olhadas em um processo seletivo quando chega na parte das pessoas com que você vai trabalhar te avaliarem. Se uma vaga está precisando de uma pessoa que tenha habilidades com [Go](https://golang.org/) e você tiver projetos pessoais em Go ou tiver no seu histórico contribuições feitas em projetos abertos em Go isso vai te dar alguns pontos positivos no processo, por exemplo. Não precisa ser só código também, se envolver em projetos reportando problemas e ajudando pessoas com dúvidas vão somar mais uns pontos também por demonstrar habilidades de trabalho colaborativo e comunicação.

Por fim, procure mais a respeito e não considere a minha opinião uma verdade absoluta, escute o outro lado que acredita nisso de ser um currículo sim e tire suas próprias conclusões. O Paulo Vasconcellos fez um [artigo](https://paulovasconcellos.com.br/como-mentir-com-data-science-4881cd91fc35) bem interessante sobre viés em Data Science que ajuda a entender porque buscar mais opiniões e o importante, opiniões diversas.

# Referências

- Git e afins para iniciantes - parte 1: https://blog.joaovrmaia.com/post/git-e-afins-para-iniciantes-parte-1
- Wordpress software - https://wordpress.org/
- Wordpress empresa - https://wordpress.com/
- Github Pages - https://pages.github.com/
- Github: https://github.com/
- Gitlab: https://about.gitlab.com/
- Bitbucket: https://bitbucket.org/
- Microsoft: https://www.microsoft.com/pt-br
- Ruby On Rails: https://rubyonrails.org/
- Ruby: https://www.ruby-lang.org/pt/
- Gitorious: https://en.wikipedia.org/wiki/Gitorious
- Gnome: https://www.gnome.org/
- Python: https://www.python.org/
- Java: https://www.java.com/pt_BR/
- Mercurial: https://www.mercurial-scm.org/
- Stash: https://en.wikipedia.org/wiki/Stash_(software)
- Linux: https://www.kernel.org/
- Hugo: https://gohugo.io/
- URL: https://pt.wikipedia.org/wiki/URL
- SSH: https://pt.wikipedia.org/wiki/Secure_Shell
- HTTP: https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol
- Página no Github do blog: https://github.com/jvrmaia/blog
- Markdown: https://pt.wikipedia.org/wiki/Markdown
- Go: https://golang.org/
- Artigo do Paulo Vasconcellos: https://paulovasconcellos.com.br/como-mentir-com-data-science-4881cd91fc35