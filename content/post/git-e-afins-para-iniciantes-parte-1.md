---
author: "João Maia"
date: 2020-08-13
title: "Git e afins para iniciantes - parte 1"
tags: [
    "git",
    "github",
    "gitlab",
    "bitbucket",
]
categories: [
    "desenvolvimento"
]
thumbnail: "img/git-e-afins-para-iniciantes-parte-1/git.png"
---

Olá,

acredito que você que está chegando no mercado de tecnologia independente de ter ou estar fazendo alguma formação superior ou técnica em tecnologia já desenvolveu algum trabalho que demorou certo tempo e você precisou salvar versões dele. Esse processo é muito comum no mundo de tecnologia e o termo mais comum é versionar o projeto. Exemplificando, vamos supor aquele trabalho no ensino médio sobre história medieval que você precisou fazer, provavelmente, você usou o Word e tinha o seu `trabalho-história-mediaval.doc` e tudo virou um caos depois da primeira vez que você começou ele, né? Vamos tentar narrar a possível história desse `trabalho-história-mediaval.doc`.

Vamos lá, primeiro dia do trabalho e você escreveu a estrutura do documento e uma introdução do seu trabalho. O normal é que você salve o arquido e vá fazer outra coisa na sua rotina. Algumas pessoas vão além de salvar enviar para o seu próprio email aquela versão do trabalho para ter salva em algum lugar, outros vão criar uma versão compactada com o mesmo nome ou adicionar algo extra tipo, `trabalho-história-mediaval-primeira-versao.zip`, ou simplesmente não faz nada. Outra coisa que pode acontecer é o trabalho não ser individual e você ter ficado de escrever a introdução e o próximo capítulo ser de responsabilidade de outra pessoa. Então, você pega a sua versão `trabalho-história-medieval-joão.zip` ou `trabalho-história-medieval-introdução.zip` e manda para os colegas do seu grupo. E quando você deleta parte do texto, salva e fecha o Word e se arrepende e não tem como mais dar CTRL+Z? Nesse ciclo vamos até o fim do trabalho onde cada vez mais próximo do fim a quantidade de versões "finais" aumentam de forma exponencial, certo?

No fim disso tudo a sua pasta do trabalho já consta com diversas versões de arquivo no formato .doc ou .zip e você não tem noção nenhuma mais do que elas são e já chegando a hora de enviar para avaliação e você não sabe mais se a versão correta é `trabalho-história-medieval-final.zip`, `trabalho-história-medieval-agora-vai.zip` ou `trabalho-história-medieval-final-real.zip`. Um verdadeiro caos, né? O Git é uma das ferramentas que vieram para resolver esse problema e acredito que a que foi a mais bem aceita pela comunidade. Eventualmente na sua carreira vai encontrar pessoas que falaram que usaram SVN, Mercurial ou outros sistemas de controle de versão. Vamos deixar claro que o fato de usar Git não resolve o problema de fato pois o uso errado da ferramenta pode causar os mesmos resultados ruins do não uso.

Chega de história de trabalhos do ensino médio e vamos ao Git! =)

# Git

## História do Git

Curiosamente o Git é mais uma das ferramentas criadas na história da computação pois o criador precisa resolver outro problema que ele considerava maior e não tinha ferramental suficiente. Nesse caso, estamos falando o Linux Torvalds e o time do Linux Kernel. Outro caso semelhante é o LaTeX criado pelo Donald Knuth para poder escrever os clássicos livros "The Art of Programming". O Linux Kernel começou a ser desenvolvido de forma colaborativa usando o BitKeeper porém a empresa dona do software resolveu cancelar a gratuidade da ferramenta. O processo de criação começou no começo do ano de 2005 e sua primeira versão foi lançada ainda no mesmo ano ao fim dele. A história com mais detalhes pode ser lida de forma breva na [Wikipédia](https://pt.wikipedia.org/wiki/Git#Hist%C3%B3ria_Inicial_do_git).

## Obtendo o Git

O Git é muito fácil de ser instalado pois vem em todas as distribuições comuns Linux, possui instalador para Windows e OSX e você pode compilar o fonte original. Na [página de downloads](https://git-scm.com/downloads) você consegue obter o instalador para o seu sistema operacional e no Linux é facilmente instalado, por exemplo, via `apt` em distribuições Debian-like. [Aqui](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) tem mais detalhes sobre o processo de instalação.

## Iniciando o uso do Git no projeto

Vamos criar uma pasta no seu computador chamada de `comecando-com-git` e entrar nela. Para eventuais problemas nos passos seguintes vamos validar a versão de Git que estamos usando, no meu caso:

```
$ git version
git version 2.17.1
```

É normal em projetos de tecnologia existir na raíz do projeto um arquivo chamado de README com ou sem extensão de arquivo, vai depender muito do contexto de comunidade e idade do projeto. Hoje, o mais tradicional é o padrão Markdown que é o mesmo utilizado nesse [blog](https://blog.joaovrmaia.com) com o auxílio do [Hugo](https://gohugo.io). Então, vamos criar o nosso README.md escrito apenas:

> Tutorial de uso do Git

Agora, com o nosso primeiro arquivo no projeto vamos dar ínicio ao versionamento dele. Um projeto Git para ser inicializado basta apenas um simples comando que vai fazer a instalação e configuração base do seu projeto:

```
$ git init
Initialized empty Git repository in /home/jvrmaia/Workspace/temp/comecando-com-git/.git/
$ tree -a .
.
├── .git
│   ├── branches
│   ├── config
│   ├── description
│   ├── HEAD
│   ├── hooks
│   │   ├── applypatch-msg.sample
│   │   ├── commit-msg.sample
│   │   ├── fsmonitor-watchman.sample
│   │   ├── post-update.sample
│   │   ├── pre-applypatch.sample
│   │   ├── pre-commit.sample
│   │   ├── prepare-commit-msg.sample
│   │   ├── pre-push.sample
│   │   ├── pre-rebase.sample
│   │   ├── pre-receive.sample
│   │   └── update.sample
│   ├── info
│   │   └── exclude
│   ├── objects
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       └── tags
└── README.md

10 directories, 16 files
```

Esse é o básico do que o Git precisa para funcionar no seu projeto mas ainda não começamos de fato a versionar pois como podemos ver na mensagem de incialização ele disse estar vazio. Vamos verificar esse estado:

```
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md

nothing added to commit but untracked files present (use "git add" to track)
```

Como podemos ver na mensagem `No commits yet`, ainda não temos `commits` no projeto e isso diz que nada foi adicionado, removido ou modificado no histórico do projeto confirmando com a mensagem de inicialização que o projeto começou vazio. Porém, `Untracked files` nos mostra que ele detectou algo no contexto do projeto que não foi adicionado a ele e por fim ele da uma dica de como colocar o nosso README.md no projeto. Vamos seguir com a dica:

```
$ git add 
Nothing specified, nothing added.
Maybe you wanted to say 'git add .'?
```

Errei de forma proposital para mostrar como o Git é nosso amigo, veja que ele além de me informar que fiz a ação de forma incorreta ainda me deu outra sugestão. Seguindo a nova sugestão:

```
$ git add .
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   README.md
```

O nosso arquivo README.md agora foi adicionado ao versionamento do projeto no estado de novo arquivo mas ainda não foi adicionado de fato pois não foi feito o commit. Vamos fazer o commit então dessa mudança no histórico do projeto:

```
$ git commit
<VIM>
adicionando o README

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
#
# Initial commit
#
# Changes to be committed:
#       new file:   README.md
#
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
~                                                                                                                                      
                                                                                                                     1,13          All
</VIM>
[master (root-commit) 0595db5] adicionando o README
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
$ git status
On branch master
nothing to commit, working tree clean
```

Finalmente, temos o nosso README.md adicionado no versionamento e aqui vemos algumas informações interessantes:
- `<VIM>...</VIM>`: é o momento de edição do nosso commit e no meu caso como uso o `vim` como editor padrão ele abriu nele;
- `[master (root-commit) 0595db5] adicionando o README`:
    - o `master` é referente a branch que vou fazer o commit;
    - o `root-commit` é por ser o primeiro do projeto;
    - o `0595db5` é a versão curta do hash do commit;
    - `adicionando o README` é a mensagem do nosso commit.

Uma observação muito importante aqui, muitos devem ter acompanhado o movimento #BlackLivesMatter recentemente em todas as redes sociais e sites de notícia por conta do caso do Floyd nos EUA. Por conta de todo esse movimento na área de tecnologia surgiu um movimento de tirar nomes dos projetos que fazem referência ao racismo e como a escravidão é fortemente ligada ao racismo o termo `master` não é adequado de se usar. O Git lançou uma nova versão que agora o novo padrão de branch principal é a `main`. Então, se o seu Git não é atualizado ou o projeto que você está trabalhando ainda usa branch `master`, seguem os passos para modificar isso:

```
$ git branch -m master main
```

Esse comando vai renomear a branch `master` por `main`.

### Evoluindo o projeto

Vamos agora introduzir melhorias no nosso README.md e salvar essas alterações no nosso controle de versão, novo README.md:

```markdown
Tutorial de uso do Git
----

Esse repositório faz parte de um tutorial para instrodução ao uso do Git.

```

Agora, vamos ver o novo estado do nosso projeto:

```
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

Podemos ver que o Git mais uma vez vem nos ajudar com dicas de como dar continuidade com o nosso projeto. Ele nos mostra duas opções: 1) adicionar as modificações; e 2) remover as modificações. Caso você tenha dúvidas de quais são essas modificações, o Git pode nos ajudar nisso vendo o que foi modificado, vamos conferir:

```diff
$ git diff
diff --git a/README.md b/README.md
index 4f207b2..ed36177 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,5 @@
 Tutorial de uso do Git
+----
+
+Esse repositório faz parte de um tutorial para instrodução ao uso do Git.
+
```

Veja que interessante, como estamos apenas adicionando informação nova no README.md ele só tem essas linhas verdes com o `+` na frente indicando a adição de coisas novas, se tivesse algo sendo removido ou alterado teriamos `-` em vermelho também. Agora, vamos adicionar as modificações e comitar elas de uma única vez:

```
$ git commit -am "melhoria no README"
[main fb30dec] melhoria no README
 1 file changed, 4 insertions(+)
```

Vamos agora fazer duas modificação no projeto para ver variações no comando do `diff` e `status`, vou introduzir um `.gitignore` para evitar arquivos do VSCode no projeto e alterar o README.md, veja como ficou:

```diff
$ git diff
diff --git a/README.md b/README.md
index ed36177..e1abf02 100644
--- a/README.md
+++ b/README.md
@@ -1,5 +1,4 @@
-Tutorial de uso do Git
+Começando com o Git
 ----
 
 Esse repositório faz parte de um tutorial para instrodução ao uso do Git.
-
```

Como podemos ver, mudei o título do README e removi a linha em branco no final. Agora, cadê o arquivo que falei que iria adicionar?

```
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore

no changes added to commit (use "git add" and/or "git commit -a")
```

Como o arquivo não existia, ele não era observado pelo Git logo não tem alterações. Então, vamos adicionar todas essas mudanças primeiro adicionando elas:

```
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   .gitignore
	modified:   README.md
```

Agora sim, estamos com as modificações prontas a serem comitadas no projeto:

```
$ git commit -m "adicionando .gitignore e melhoria no README"
[main e0d67bb] adicionando .gitignore e melhoria no README
 2 files changed, 2 insertions(+), 2 deletions(-)
 create mode 100644 .gitignore
```

Vamos agora convesar sobre boas práticas no Git! Repare que essa mensagem de commit ficou longa e a ação feita no projeto são totalmente independentes, correto? Então, teria sido de melhor prática ter feito o commit de cada uma das alterações de forma separada.

### Dividir para conquistar

É bem provável que você venha a compartilhar um projeto com mais pessoas e não seria saudável todos ficarem adicionando suas modificações no linha principal do projeto, imagina que você e mais quartro pessoas tem apenas uma folha A4 e uma caneta cada e vocês devem ao mesmo tempo copiar um parágrafo diferente um dos outros de um texto qualquer. Seria uma loucura certo? Vamos imaginar agora que cada um pode copiar o texto no seu computador no seu editor preferido e depois todos sincronizarem de organizar em um arquivo só tudo. É mais ou menos isso que conseguimos fazer com o Git. Vamos criar uma nova linha no tempo no nosso projeto para adicionar um código simples em C para imprimir "Olá mundo!" no terminal.

```
$ git checkout -b ola-mundo-c
Switched to a new branch 'ola-mundo-c'
```

O arquivo `ola-mundo.c`:

```c
#include <stdio.h>

int main(int argc, char **argv) {
  printf("Olá mundo!\n");

  return 0;
}
```

Vamos copilar e executar ele agora:

```
$ gcc ola-mundo.c 
$ ./a.out 
Olá mundo!
```

Perfeito, tudo funcionando! Vamos agora ver no que isso refletiu no nosso projeto:

```
$ git status
On branch ola-mundo-c
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	a.out
	ola-mundo.c

nothing added to commit but untracked files present (use "git add" to track)

$ git diff
<VAZIO>
```

Esse arquivo `a.out` não faz sentido existir no nosso projeto pois é um binário e dado o fonte pode ser gerado por outra pessoa sem problemas. Então, vamos primeiro adicionar a mudança para ele não ser observado pelo nosso projeto:

```diff
$ git diff
diff --git a/.gitignore b/.gitignore
index 35215c6..9005441 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1 +1,2 @@
 .code/
+a.out
$ git add .gitignore 
$ git commit -m "ignora binários do C"
[ola-mundo-c 092f671] ignora binários do C
 1 file changed, 1 insertion(+)
$ git status
On branch ola-mundo-c
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	ola-mundo.c

nothing added to commit but untracked files present (use "git add" to track)
```

Perfeito, agora podemos adicionar o nosso projeto em C:

```
$ git add . 
$ git commit -m "adiciona projeto em C"
[ola-mundo-c e7acbfd] adiciona projeto em C
 1 file changed, 7 insertions(+)
 create mode 100644 ola-mundo.c
$ git status
On branch ola-mundo-c
nothing to commit, working tree clean
```

Agora, queremos voltar para linha principal do projeto com nossas mudanças e primeiro vamos conferir o quanto ela mudou desde a última vez que usamos ela. Agora podemos uma usar um recurso que de uma das utilidades do Git que já vimos aqui, o `diff`, veja:

```diff
$ git diff main
diff --git a/.gitignore b/.gitignore
index 35215c6..9005441 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1 +1,2 @@
 .code/
+a.out
diff --git a/ola-mundo.c b/ola-mundo.c
new file mode 100644
index 0000000..f3ecd97
--- /dev/null
+++ b/ola-mundo.c
@@ -0,0 +1,7 @@
+#include <stdio.h>
+
+int main(int argc, char **argv) {
+  printf("Olá mundo!\n");
+
+  return 0;
+}
```

Como podemos ver a diferença entre a nossa linha e a principal são apenas as modificações que fizemos. Com isso podemos voltar para a linha principal e aplicar nossas alterações nela, vamos fazer isso:

```
$ git checkout main
Switched to branch 'main'
$ git merge ola-mundo-c 
Updating e0d67bb..e7acbfd
Fast-forward
 .gitignore  | 1 +
 ola-mundo.c | 7 +++++++
 2 files changed, 8 insertions(+)
 create mode 100644 ola-mundo.c
$ git branch -D ola-mundo-c 
Deleted branch ola-mundo-c (was e7acbfd).
```

O que fiz aqui? Primeiro, voltei meu estado atual para o da linha principal e depois fiz a união das minhas alterações nela. Por fim, deletei a branch que desenvolvi meu código para manter mais limpo o meu projeto. No começo tinha falado sobre trabalho em equipe mas da forma que demonstrei não tive a partiçapação de outras pessoas no processo. Como funcionaria isso?

### Git como serviço

Você já deve ter escutado falar sobre Github, Gitlab, Bitbucket e afins se fez algumas pesquisas sobre a área de tecnologia. Todos esses são serviços de hospedagem de projetos Git. Esses serviços são muito importantes para o trabalho coletivo mas a solução no se limita a eles e acho que esse papo não cabe nesse momento. Então, minha sugestão é que você faça uma conta em qualquer um desses serviços e comece a navegar nele para aprender um pouco mas fique tranquilo que pretendo abordar eles aqui pois eles tem recursos e semânticas diferentes interessantes. Sem dúvidas hoje o mais popular é o Github mas existem polêmicas a respeito por conta de ser da Microsoft hoje e por conta do ICE. Não vou entrar nessa discussão agora pois não é o momento. O Gitlab é um projeto aberto e empresa, ou seja, você pode ter um Gitlab seu rodando por sua própria administração ou usar eles pagando ou não. O Bitbucket é da empresa Atlassian que tem diversas outras soluções para times de tecnologia. Uma coisa que você pode ficar tranquilo que em todos os três você vai poder criar projetos públicos e privados sem custos e acho que isso é o suficiente para o momento. Esse blog, por exemplo, eu uso o Github para salvar o código, texto e imagens dele usando um gerador de sites estáticos chamado Hugo e um recurso bem legal do Github que é o Github Pages.

O próximo artigo aqui vamos começar desse ponto, colaborando via Git!

## Considerações finais

Nesse primeiro artigo meu objetivo era aprensentar o Git e aprender a dar os primeiros passos com ele e já criar familiaridade com alguns termos comuns em projetos que usam ele. Espero que agora você sempre ao iniciar um projeto já criar seu projeto em qualquer um dos serviços de Git que comentei e manter tudo de forma organizada e com histórico. Sofri bastante com a falta desse conhecimento na graduação e perdi muita coisa que fiz por conta disso. Esse foi apenas o primeiro de uma sequência sem tamanho definido ainda que espero escrever que vão ter guiar a saber tudo que você precisa saber de Git para lidar com a maiorida dos seus problemas que se tornarão cotidianos trabalhando com tecnologia.