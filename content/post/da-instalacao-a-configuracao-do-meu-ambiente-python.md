---
author: "João Maia"
date: 2020-04-28
title: "Da instalação a configuração do meu ambiente Python"
tags: [
    "python",
]
categories: [
    "linguagens de programação"
]
thumbnail: "img/da-instalacao-a-configuracao-do-meu-ambiente-python/python.png"
---

Olá,

Apesar de não estar mais usando Python nas aplicações de produção que trabalho hoje, eventualmente uso Python para outras atividades pela comodidade que encontro nele. Uma coisa que é importante nisso é sempre ter seu ambientes bem segregados para evitar surpresas de projetos que precisam versões específicas de bibliotecas e Python.

Nesse artigo vou mostrar de forma clara e direta como uso o Python hoje depois de várias idas e vindas de configurações. Acredito que essa nova solução que surgiu recentemente na parte de gestão de dependências veio a resolver uma parte que sempre tive dificuldades achar sanidade na configuração. Ainda preciso explorar mais a solução mas já tenho gostado dela.

# Instalando o Python

Na maioria das versões mais modernas de sistema operacional Linux e OS X acredito que o Python padrão já seja o Python 3+ dado que o Python 2 já foi descontinuado. Quem chega no Python agora e não é um desenvolvedor que use Python nos seus sistemas é bem comum você usar o Python do sistema operacional e ser feliz com isso. Eventualmente, você vai correr o risco de alguma coisa que você instale quebre com outra instalada mas é bem difícil de ver isso acontecer dentro do que já observei até hoje. Já se você for um desenvolvedor Python isso pode se tornar rotineiro.

Então, como resolver esse problema? Um solução que já uso tem uns anos e não vejo motivos para mudar é o [pyenv](https://github.com/pyenv/pyenv). O `pyenv` é um gestor e instalador de versões de Python no seu sistema. O `README` do projeto é bem claro sobre a instalação dele e acredito que você não vai ter dificuldades nessa parte, um adicional que faço é nesse ponto do [Homebrew](https://github.com/pyenv/pyenv#homebrew-on-macos), o Homebrew também funciona no Linux e recomendo o uso, mais detalhes a respeito podem ser vistos [aqui](https://docs.brew.sh/Homebrew-on-Linux).

Vamos seguir agora assumindo que o `pyenv` já está 100% configurado na sua máquina, a primeira coisa que precisamos fazer é lista quais versões de Python estão disponíveis:

```
$ pyenv install -l
Available versions:
  2.1.3
  2.2.3
  ...
  2.7.16
  2.7.17
  3.0.1
  3.1.0
  ...
  3.8.1
  3.8.2
  3.9.0a5
  3.9-dev
  activepython-2.7.14
  activepython-3.5.4
  activepython-3.6.0
  anaconda-1.4.0
  anaconda-1.5.0
  ...
  anaconda-4.0.0
  anaconda2-2.4.0
  ...
  anaconda2-2019.07
  anaconda3-2.0.0
  ...
  anaconda3-2019.10
  ironpython-dev
  ...
  ironpython-2.7.7
  jython-dev
  ...
  jython-2.7.1
  micropython-dev
  ...
  micropython-1.12
  miniconda-latest
  miniconda-2.2.2
  miniconda-3.0.0
  ...
  miniconda-3.18.3
  miniconda2-latest
  miniconda2-3.18.3
  ...
  miniconda2-4.7.12
  miniconda3-latest
  ...
  miniconda3-4.7.12
  pypy-c-jit-latest
  pypy-c-nojit-latest
  pypy-dev
  pypy-stm-2.3
  pypy-stm-2.5.1
  pypy-1.5-src
  ...
  pypy-1.9
  pypy-2.0-src
  ...
  pypy-2.6.1
  pypy-4.0.0-src
  ...
  pypy-4.0.1
  pypy-5.0.0-src
  ...
  pypy-5.7.1
  pypy2-5.3-src
  ...
  pypy2.7-7.3.0
  pypy3-dev
  ...
  pypy3.6-7.3.0
  pyston-0.5.1
  pyston-0.6.0
  pyston-0.6.1
  stackless-dev
  stackless-2.7-dev
  ...
  stackless-2.7.14
  stackless-3.2.2
  ...
  stackless-3.5.4

```

Para instalar agora é simples: `pyenv install <versão>`. Aqui no meu caso tenho instalado algumas versões apenas:

```
$ pyenv versions
  system
  3.6.9
  * 3.7.5 (set by /home/jvrmaia/.pyenv/version)
  3.8.0
  3.8.0/envs/flask-docker-example
  flask-docker-example
```
Como pode perceber a versão 3.7.5 tem um `*` na frente pois é a versão atual. Para listar a versão atual apenas:

```
$ pyenv version
3.7.5 (set by /home/jvrmaia/.pyenv/version)
```

Outro fato curioso é o `system` na lista de versões de Python, é assim que o `pyenv` se refere a versão instalada pelo sistema operacional e não ele. Agora, com os Python's que você precisa instalado você precisa escolher qual vai usar, tem duas formas de selecionar e inspecionar isso: `global` e `local`. O `global` como o próprio nome já sugere é para a configuração global do Python, no meu caso como visto anteriormente é o 3.7.5. Isso pode ser conferido rodando o comando `pyenv global`, no caso de você passar algum argumento extra e se esse argumento for outra versão instalada, essa versão vai assumir o papel de versão global. O `local` segue o mesmo comportamento da `global` só que no escopo de onde você executa o comando, supondo que o seu diretório atual seja `/tmp/app_py` e você execute `pyenv local 3.8.0`, a versão local vai ser alterada para a 3.8.0 e para persistir essa informação será criado um arquivo `.python-version` com o valor 3.8.0 para que caso você entre dentro dessa árvore dos seus diretórios o `pyenv` troque o Python do `$PATH` automaticamente.

# Ambientes virtuais do Python

Como disse antes é um risco compartilhar o Python do sistema para diversos propósitos, na comunidade do Python já é bem conhecido o uso de `virtualenvs` para uma mesma versão do Python ter diversos projetos com bibliotecas iguais em versões distintas e garantir esse isolamento para não ter surpresas. O `pyenv` tem suporte a plugins e existe um para gerenciar esses ambientes: [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv).

O uso dele é bem simples, é adicionado ao `pyenv` um novo comando `virtualenv` para criar esses ambientes, então podemos fazer o seguinte comando: `pyenv virtualenv 3.8.0 app_py`. Agora, na nossa pasta `/tmp/app_py` podemos configurar com `pyenv local` para ele usar o ambiente virtual criado chamado `app_py` assim o projeto consegue ter um isolamento das suas bibliotecas da instalação do Python 3.8.0.

# Lidando com dependências do seu projeto

Acho que se você já lidou um pouco mais com Python e precisou instalar alguma biblioteca já deve ter esbarrado com o `pip`. O problema do `pip` puro é que ele não entrega tudo que precisamos para uma boa gestão de dendências de forma amigável e confortável ao desenvolvedor. Algumas soluções surgiram durante todo esse tempo e algumas frustações geradas com isso. Recentemente, vendo videos no canal [Code Show](https://codeshow.com.br/) do [Bruno Rocha](https://twitter.com/rochacbruno) acabei conhecendo o [Poetry](https://python-poetry.org/) e gostei bastante dele e me lembrou diversas outra soluções que vemos por ai para gestão de dependências. Nada melhor que uma ferramenta que tem a mesma experência de usário de outras ferramentas que você usa.

> "I built Poetry because I wanted a single tool to manage my Python projects from start to finish. I wanted something reliable and intuitive that the community could use and enjoy." Sébastien Eustace

Como podemos ver na declaração do criador o Poetry vai além de gerenciar suas dependências mas aqui vou ficar apenas até essa parte que é o que faço uso hoje. No meu Github tenho um projeto chamado [flask-docker-example](https://github.com/jvrmaia/flask-docker-example/tree/master/app) que é um exemplo de como colocar uma aplicação [Flask](https://flask.palletsprojects.com/en/1.1.x/) em Docker:

```
(flask-docker-example) => ~/Workspace/jvrmaia/flask-docker-example/app
$ tree .
.
├── __init__.py
├── poetry.lock
├── pyproject.toml
└── whoami.py

```
Essa é a estrutura do projeto na parte de código, a estrutura inicial  que criei com o `poetry init` mas antes de entrar nisso vamos instalar o Poetry, nesse caso recomendo instalar o Poetry fora do seu ambiente virtual de forma que o acesso fique global com o `pip`: `pip install poetry`. A "interface" do Poetry:

```
$ poetry
Poetry version 1.0.5

USAGE
  poetry [-h] [-q] [-v [<...>]] [-V] [--ansi] [--no-ansi] [-n] <command> [<arg1>] ... [<argN>]

ARGUMENTS
  <command>              The command to execute
  <arg>                  The arguments of the command

GLOBAL OPTIONS
  -h (--help)            Display this help message
  -q (--quiet)           Do not output any message
  -v (--verbose)         Increase the verbosity of messages: "-v" for normal output, "-vv" for more verbose output and "-vvv" for debug
  -V (--version)         Display this application version
  --ansi                 Force ANSI output
  --no-ansi              Disable ANSI output
  -n (--no-interaction)  Do not ask any interactive question

AVAILABLE COMMANDS
  about                  Shows information about Poetry.
  add                    Adds a new dependency to pyproject.toml.
  build                  Builds a package, as a tarball and a wheel by default.
  cache                  Interact with Poetry's cache
  check                  Checks the validity of the pyproject.toml file.
  config                 Manages configuration settings.
  debug                  Debug various elements of Poetry.
  env                    Interact with Poetry's project environments.
  export                 Exports the lock file to alternative formats.
  help                   Display the manual of a command
  init                   Creates a basic pyproject.toml file in the current directory.
  install                Installs the project dependencies.
  lock                   Locks the project dependencies.
  new                    Creates a new Python project at <path>.
  publish                Publishes a package to a remote repository.
  remove                 Removes a package from the project dependencies.
  run                    Runs a command in the appropriate environment.
  search                 Searches for packages on remote repositories.
  self                   Interact with Poetry directly.
  shell                  Spawns a shell within the virtual environment.
  show                   Shows information about packages.
  update                 Update the dependencies as according to the pyproject.toml file.
  version                Shows the version of the project or bumps it when a valid bump rule is provided.
```

Para adicionar dependências no seu projeto basta fazer como é mostrado no exemplo do site:

```
$ poetry add pendulum
Using version ^2.0.5 for pendulum

Updating dependencies
Resolving dependencies..........

Writing lock file

Package operations: 4 installs, 0 updates, 0 removals

  - Installing six (1.13.0)
  - Installing python-dateutil (2.8.1)
  - Installing pytzdata (2019.3)
  - Installing pendulum (2.0.5)
```

Caso você tenha feito apenas um clone de um projeto e queira instalar elas é só executar o comando `install`. Agora, vamos entender os dois arquivos gerandos que cuidam da gestão desse ambiente: 1) `pyproject.toml`: esse guarda as informações do seu projeto; e 2) `poetry.lock` que em detalhes versiona as suas depências e as dependências de suas dependências. Isso é muito importante para evitar que uma pessoa que fez a configuração do projeto após um certo tempo de existência não instale uma subdependência com versão superior a do momento da criação com quebras de API quebrando o projeto.

`pyproject.toml`

```
 [tool.poetry]
 name = "flask-docker-example"
 version = "0.1.0"
 description = "Flask dockerize example"
 license = "MIT"
 authors = [
     "João Maia <joao@joaovrmaia.com>"
 ]
 readme = "README.md"
 repository = "https://github.com/jvrmaia/flask-docker-example"
 homepage = "https://github.com/jvrmaia/flask-docker-example"


[tool.poetry.dependencies]
python = "3.8.2"
click = "==7.0"
flask = "==1.1.1"
itsdangerous = "==1.1.0"
jinja2 = "==2.11.1"
markupsafe = "==1.1.1"
python-decouple = "==3.3"
werkzeug = "==1.0.0"
gunicorn = "^20.0.4"



[tool.poetry.dev-dependencies]
pylint = "^2.4.4"
[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"
```

`poetry.lock`

```
[[package]]
category = "dev"
description = "An abstract syntax tree for Python with inference support."
name = "astroid"
optional = false
python-versions = ">=3.5.*"
version = "2.3.3"

[package.dependencies]
lazy-object-proxy = ">=1.4.0,<1.5.0"
six = ">=1.12,<2.0"
wrapt = ">=1.11.0,<1.12.0"

[[package]]
category = "main"
description = "Composable command line interface toolkit"
name = "click"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*"
version = "7.0"

[[package]]
category = "dev"
description = "Cross-platform colored terminal text."
marker = "sys_platform == \"win32\""
name = "colorama"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
version = "0.4.3"

[[package]]
category = "main"
description = "A simple framework for building complex web applications."
name = "flask"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
version = "1.1.1"

[package.dependencies]
Jinja2 = ">=2.10.1"
Werkzeug = ">=0.15"
click = ">=5.1"
itsdangerous = ">=0.24"

[package.extras]
dev = ["pytest", "coverage", "tox", "sphinx", "pallets-sphinx-themes", "sphinxcontrib-log-cabinet", "sphinx-issues"]
docs = ["sphinx", "pallets-sphinx-themes", "sphinxcontrib-log-cabinet", "sphinx-issues"]
dotenv = ["python-dotenv"]

[[package]]
category = "main"
description = "WSGI HTTP Server for UNIX"
name = "gunicorn"
optional = false
python-versions = ">=3.4"
version = "20.0.4"

[package.dependencies]
setuptools = ">=3.0"

[package.extras]
eventlet = ["eventlet (>=0.9.7)"]
gevent = ["gevent (>=0.13)"]
setproctitle = ["setproctitle"]
tornado = ["tornado (>=0.2)"]

[[package]]
category = "dev"
description = "A Python utility / library to sort Python imports."
name = "isort"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*"
version = "4.3.21"

[package.extras]
pipfile = ["pipreqs", "requirementslib"]
pyproject = ["toml"]
requirements = ["pipreqs", "pip-api"]
xdg_home = ["appdirs (>=1.4.0)"]

[[package]]
category = "main"
description = "Various helpers to pass data to untrusted environments and back."
name = "itsdangerous"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*"
version = "1.1.0"

[[package]]
category = "main"
description = "A very fast and expressive template engine."
name = "jinja2"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
version = "2.11.1"

[package.dependencies]
MarkupSafe = ">=0.23"

[package.extras]
i18n = ["Babel (>=0.8)"]

[[package]]
category = "dev"
description = "A fast and thorough lazy object proxy."
name = "lazy-object-proxy"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*"
version = "1.4.3"

[[package]]
category = "main"
description = "Safely add untrusted strings to HTML/XML markup."
name = "markupsafe"
optional = false
python-versions = ">=2.7,!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*"
version = "1.1.1"

[[package]]
category = "dev"
description = "McCabe checker, plugin for flake8"
name = "mccabe"
optional = false
python-versions = "*"
version = "0.6.1"

[[package]]
category = "dev"
description = "python code static checker"
name = "pylint"
optional = false
python-versions = ">=3.5.*"
version = "2.4.4"

[package.dependencies]
astroid = ">=2.3.0,<2.4"
colorama = "*"
isort = ">=4.2.5,<5"
mccabe = ">=0.6,<0.7"

[[package]]
category = "main"
description = "Strict separation of settings from code."
name = "python-decouple"
optional = false
python-versions = "*"
version = "3.3"

[[package]]
category = "dev"
description = "Python 2 and 3 compatibility utilities"
name = "six"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*"
version = "1.14.0"

[[package]]
category = "main"
description = "The comprehensive WSGI web application library."
name = "werkzeug"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
version = "1.0.0"

[package.extras]
dev = ["pytest", "coverage", "tox", "sphinx", "pallets-sphinx-themes", "sphinx-issues"]
watchdog = ["watchdog"]

[[package]]
category = "dev"
description = "Module for decorators, wrappers and monkey patching."
name = "wrapt"
optional = false
python-versions = "*"
version = "1.11.2"

[metadata]
content-hash = "6dffc365a6d8b9d1ba2ce40d09661212452c759d2102e38efa6b0bcb997f07db"
python-versions = "3.8.2"

[metadata.files]
astroid = [
    {file = "astroid-2.3.3-py3-none-any.whl", hash = "sha256:840947ebfa8b58f318d42301cf8c0a20fd794a33b61cc4638e28e9e61ba32f42"},
    {file = "astroid-2.3.3.tar.gz", hash = "sha256:71ea07f44df9568a75d0f354c49143a4575d90645e9fead6dfb52c26a85ed13a"},
]
click = [
    {file = "Click-7.0-py2.py3-none-any.whl", hash = "sha256:2335065e6395b9e67ca716de5f7526736bfa6ceead690adf616d925bdc622b13"},
    {file = "Click-7.0.tar.gz", hash = "sha256:5b94b49521f6456670fdb30cd82a4eca9412788a93fa6dd6df72c94d5a8ff2d7"},
]
colorama = [
    {file = "colorama-0.4.3-py2.py3-none-any.whl", hash = "sha256:7d73d2a99753107a36ac6b455ee49046802e59d9d076ef8e47b61499fa29afff"},
    {file = "colorama-0.4.3.tar.gz", hash = "sha256:e96da0d330793e2cb9485e9ddfd918d456036c7149416295932478192f4436a1"},
]
flask = [
    {file = "Flask-1.1.1-py2.py3-none-any.whl", hash = "sha256:45eb5a6fd193d6cf7e0cf5d8a5b31f83d5faae0293695626f539a823e93b13f6"},
    {file = "Flask-1.1.1.tar.gz", hash = "sha256:13f9f196f330c7c2c5d7a5cf91af894110ca0215ac051b5844701f2bfd934d52"},
]
gunicorn = [
    {file = "gunicorn-20.0.4-py2.py3-none-any.whl", hash = "sha256:cd4a810dd51bf497552cf3f863b575dabd73d6ad6a91075b65936b151cbf4f9c"},
    {file = "gunicorn-20.0.4.tar.gz", hash = "sha256:1904bb2b8a43658807108d59c3f3d56c2b6121a701161de0ddf9ad140073c626"},
]
isort = [
    {file = "isort-4.3.21-py2.py3-none-any.whl", hash = "sha256:6e811fcb295968434526407adb8796944f1988c5b65e8139058f2014cbe100fd"},
    {file = "isort-4.3.21.tar.gz", hash = "sha256:54da7e92468955c4fceacd0c86bd0ec997b0e1ee80d97f67c35a78b719dccab1"},
]
itsdangerous = [
    {file = "itsdangerous-1.1.0-py2.py3-none-any.whl", hash = "sha256:b12271b2047cb23eeb98c8b5622e2e5c5e9abd9784a153e9d8ef9cb4dd09d749"},
    {file = "itsdangerous-1.1.0.tar.gz", hash = "sha256:321b033d07f2a4136d3ec762eac9f16a10ccd60f53c0c91af90217ace7ba1f19"},
]
jinja2 = [
    {file = "Jinja2-2.11.1-py2.py3-none-any.whl", hash = "sha256:b0eaf100007721b5c16c1fc1eecb87409464edc10469ddc9a22a27a99123be49"},
    {file = "Jinja2-2.11.1.tar.gz", hash = "sha256:93187ffbc7808079673ef52771baa950426fd664d3aad1d0fa3e95644360e250"},
]
lazy-object-proxy = [
    {file = "lazy-object-proxy-1.4.3.tar.gz", hash = "sha256:f3900e8a5de27447acbf900b4750b0ddfd7ec1ea7fbaf11dfa911141bc522af0"},
    {file = "lazy_object_proxy-1.4.3-cp27-cp27m-macosx_10_13_x86_64.whl", hash = "sha256:a2238e9d1bb71a56cd710611a1614d1194dc10a175c1e08d75e1a7bcc250d442"},
    {file = "lazy_object_proxy-1.4.3-cp27-cp27m-win32.whl", hash = "sha256:efa1909120ce98bbb3777e8b6f92237f5d5c8ea6758efea36a473e1d38f7d3e4"},
    {file = "lazy_object_proxy-1.4.3-cp27-cp27m-win_amd64.whl", hash = "sha256:4677f594e474c91da97f489fea5b7daa17b5517190899cf213697e48d3902f5a"},
    {file = "lazy_object_proxy-1.4.3-cp27-cp27mu-manylinux1_x86_64.whl", hash = "sha256:0c4b206227a8097f05c4dbdd323c50edf81f15db3b8dc064d08c62d37e1a504d"},
    {file = "lazy_object_proxy-1.4.3-cp34-cp34m-manylinux1_x86_64.whl", hash = "sha256:d945239a5639b3ff35b70a88c5f2f491913eb94871780ebfabb2568bd58afc5a"},
    {file = "lazy_object_proxy-1.4.3-cp34-cp34m-win32.whl", hash = "sha256:9651375199045a358eb6741df3e02a651e0330be090b3bc79f6d0de31a80ec3e"},
    {file = "lazy_object_proxy-1.4.3-cp34-cp34m-win_amd64.whl", hash = "sha256:eba7011090323c1dadf18b3b689845fd96a61ba0a1dfbd7f24b921398affc357"},
    {file = "lazy_object_proxy-1.4.3-cp35-cp35m-manylinux1_x86_64.whl", hash = "sha256:48dab84ebd4831077b150572aec802f303117c8cc5c871e182447281ebf3ac50"},
    {file = "lazy_object_proxy-1.4.3-cp35-cp35m-win32.whl", hash = "sha256:ca0a928a3ddbc5725be2dd1cf895ec0a254798915fb3a36af0964a0a4149e3db"},
    {file = "lazy_object_proxy-1.4.3-cp35-cp35m-win_amd64.whl", hash = "sha256:194d092e6f246b906e8f70884e620e459fc54db3259e60cf69a4d66c3fda3449"},
    {file = "lazy_object_proxy-1.4.3-cp36-cp36m-manylinux1_x86_64.whl", hash = "sha256:97bb5884f6f1cdce0099f86b907aa41c970c3c672ac8b9c8352789e103cf3156"},
    {file = "lazy_object_proxy-1.4.3-cp36-cp36m-win32.whl", hash = "sha256:cb2c7c57005a6804ab66f106ceb8482da55f5314b7fcb06551db1edae4ad1531"},
    {file = "lazy_object_proxy-1.4.3-cp36-cp36m-win_amd64.whl", hash = "sha256:8d859b89baf8ef7f8bc6b00aa20316483d67f0b1cbf422f5b4dc56701c8f2ffb"},
    {file = "lazy_object_proxy-1.4.3-cp37-cp37m-macosx_10_13_x86_64.whl", hash = "sha256:1be7e4c9f96948003609aa6c974ae59830a6baecc5376c25c92d7d697e684c08"},
    {file = "lazy_object_proxy-1.4.3-cp37-cp37m-manylinux1_x86_64.whl", hash = "sha256:d74bb8693bf9cf75ac3b47a54d716bbb1a92648d5f781fc799347cfc95952383"},
    {file = "lazy_object_proxy-1.4.3-cp37-cp37m-win32.whl", hash = "sha256:9b15f3f4c0f35727d3a0fba4b770b3c4ebbb1fa907dbcc046a1d2799f3edd142"},
    {file = "lazy_object_proxy-1.4.3-cp37-cp37m-win_amd64.whl", hash = "sha256:9254f4358b9b541e3441b007a0ea0764b9d056afdeafc1a5569eee1cc6c1b9ea"},
    {file = "lazy_object_proxy-1.4.3-cp38-cp38-manylinux1_x86_64.whl", hash = "sha256:a6ae12d08c0bf9909ce12385803a543bfe99b95fe01e752536a60af2b7797c62"},
    {file = "lazy_object_proxy-1.4.3-cp38-cp38-win32.whl", hash = "sha256:5541cada25cd173702dbd99f8e22434105456314462326f06dba3e180f203dfd"},
    {file = "lazy_object_proxy-1.4.3-cp38-cp38-win_amd64.whl", hash = "sha256:59f79fef100b09564bc2df42ea2d8d21a64fdcda64979c0fa3db7bdaabaf6239"},
]
markupsafe = [
    {file = "MarkupSafe-1.1.1-cp27-cp27m-macosx_10_6_intel.whl", hash = "sha256:09027a7803a62ca78792ad89403b1b7a73a01c8cb65909cd876f7fcebd79b161"},
    {file = "MarkupSafe-1.1.1-cp27-cp27m-manylinux1_i686.whl", hash = "sha256:e249096428b3ae81b08327a63a485ad0878de3fb939049038579ac0ef61e17e7"},
    {file = "MarkupSafe-1.1.1-cp27-cp27m-manylinux1_x86_64.whl", hash = "sha256:500d4957e52ddc3351cabf489e79c91c17f6e0899158447047588650b5e69183"},
    {file = "MarkupSafe-1.1.1-cp27-cp27m-win32.whl", hash = "sha256:b2051432115498d3562c084a49bba65d97cf251f5a331c64a12ee7e04dacc51b"},
    {file = "MarkupSafe-1.1.1-cp27-cp27m-win_amd64.whl", hash = "sha256:98c7086708b163d425c67c7a91bad6e466bb99d797aa64f965e9d25c12111a5e"},
    {file = "MarkupSafe-1.1.1-cp27-cp27mu-manylinux1_i686.whl", hash = "sha256:cd5df75523866410809ca100dc9681e301e3c27567cf498077e8551b6d20e42f"},
    {file = "MarkupSafe-1.1.1-cp27-cp27mu-manylinux1_x86_64.whl", hash = "sha256:43a55c2930bbc139570ac2452adf3d70cdbb3cfe5912c71cdce1c2c6bbd9c5d1"},
    {file = "MarkupSafe-1.1.1-cp34-cp34m-macosx_10_6_intel.whl", hash = "sha256:1027c282dad077d0bae18be6794e6b6b8c91d58ed8a8d89a89d59693b9131db5"},
    {file = "MarkupSafe-1.1.1-cp34-cp34m-manylinux1_i686.whl", hash = "sha256:62fe6c95e3ec8a7fad637b7f3d372c15ec1caa01ab47926cfdf7a75b40e0eac1"},
    {file = "MarkupSafe-1.1.1-cp34-cp34m-manylinux1_x86_64.whl", hash = "sha256:88e5fcfb52ee7b911e8bb6d6aa2fd21fbecc674eadd44118a9cc3863f938e735"},
    {file = "MarkupSafe-1.1.1-cp34-cp34m-win32.whl", hash = "sha256:ade5e387d2ad0d7ebf59146cc00c8044acbd863725f887353a10df825fc8ae21"},
    {file = "MarkupSafe-1.1.1-cp34-cp34m-win_amd64.whl", hash = "sha256:09c4b7f37d6c648cb13f9230d847adf22f8171b1ccc4d5682398e77f40309235"},
    {file = "MarkupSafe-1.1.1-cp35-cp35m-macosx_10_6_intel.whl", hash = "sha256:79855e1c5b8da654cf486b830bd42c06e8780cea587384cf6545b7d9ac013a0b"},
    {file = "MarkupSafe-1.1.1-cp35-cp35m-manylinux1_i686.whl", hash = "sha256:c8716a48d94b06bb3b2524c2b77e055fb313aeb4ea620c8dd03a105574ba704f"},
    {file = "MarkupSafe-1.1.1-cp35-cp35m-manylinux1_x86_64.whl", hash = "sha256:7c1699dfe0cf8ff607dbdcc1e9b9af1755371f92a68f706051cc8c37d447c905"},
    {file = "MarkupSafe-1.1.1-cp35-cp35m-win32.whl", hash = "sha256:6dd73240d2af64df90aa7c4e7481e23825ea70af4b4922f8ede5b9e35f78a3b1"},
    {file = "MarkupSafe-1.1.1-cp35-cp35m-win_amd64.whl", hash = "sha256:9add70b36c5666a2ed02b43b335fe19002ee5235efd4b8a89bfcf9005bebac0d"},
    {file = "MarkupSafe-1.1.1-cp36-cp36m-macosx_10_6_intel.whl", hash = "sha256:24982cc2533820871eba85ba648cd53d8623687ff11cbb805be4ff7b4c971aff"},
    {file = "MarkupSafe-1.1.1-cp36-cp36m-manylinux1_i686.whl", hash = "sha256:00bc623926325b26bb9605ae9eae8a215691f33cae5df11ca5424f06f2d1f473"},
    {file = "MarkupSafe-1.1.1-cp36-cp36m-manylinux1_x86_64.whl", hash = "sha256:717ba8fe3ae9cc0006d7c451f0bb265ee07739daf76355d06366154ee68d221e"},
    {file = "MarkupSafe-1.1.1-cp36-cp36m-win32.whl", hash = "sha256:535f6fc4d397c1563d08b88e485c3496cf5784e927af890fb3c3aac7f933ec66"},
    {file = "MarkupSafe-1.1.1-cp36-cp36m-win_amd64.whl", hash = "sha256:b1282f8c00509d99fef04d8ba936b156d419be841854fe901d8ae224c59f0be5"},
    {file = "MarkupSafe-1.1.1-cp37-cp37m-macosx_10_6_intel.whl", hash = "sha256:8defac2f2ccd6805ebf65f5eeb132adcf2ab57aa11fdf4c0dd5169a004710e7d"},
    {file = "MarkupSafe-1.1.1-cp37-cp37m-manylinux1_i686.whl", hash = "sha256:46c99d2de99945ec5cb54f23c8cd5689f6d7177305ebff350a58ce5f8de1669e"},
    {file = "MarkupSafe-1.1.1-cp37-cp37m-manylinux1_x86_64.whl", hash = "sha256:ba59edeaa2fc6114428f1637ffff42da1e311e29382d81b339c1817d37ec93c6"},
    {file = "MarkupSafe-1.1.1-cp37-cp37m-win32.whl", hash = "sha256:b00c1de48212e4cc9603895652c5c410df699856a2853135b3967591e4beebc2"},
    {file = "MarkupSafe-1.1.1-cp37-cp37m-win_amd64.whl", hash = "sha256:9bf40443012702a1d2070043cb6291650a0841ece432556f784f004937f0f32c"},
    {file = "MarkupSafe-1.1.1-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:6788b695d50a51edb699cb55e35487e430fa21f1ed838122d722e0ff0ac5ba15"},
    {file = "MarkupSafe-1.1.1-cp38-cp38-manylinux1_i686.whl", hash = "sha256:cdb132fc825c38e1aeec2c8aa9338310d29d337bebbd7baa06889d09a60a1fa2"},
    {file = "MarkupSafe-1.1.1-cp38-cp38-manylinux1_x86_64.whl", hash = "sha256:13d3144e1e340870b25e7b10b98d779608c02016d5184cfb9927a9f10c689f42"},
    {file = "MarkupSafe-1.1.1-cp38-cp38-win32.whl", hash = "sha256:596510de112c685489095da617b5bcbbac7dd6384aeebeda4df6025d0256a81b"},
    {file = "MarkupSafe-1.1.1-cp38-cp38-win_amd64.whl", hash = "sha256:e8313f01ba26fbbe36c7be1966a7b7424942f670f38e666995b88d012765b9be"},
    {file = "MarkupSafe-1.1.1.tar.gz", hash = "sha256:29872e92839765e546828bb7754a68c418d927cd064fd4708fab9fe9c8bb116b"},
]
mccabe = [
    {file = "mccabe-0.6.1-py2.py3-none-any.whl", hash = "sha256:ab8a6258860da4b6677da4bd2fe5dc2c659cff31b3ee4f7f5d64e79735b80d42"},
    {file = "mccabe-0.6.1.tar.gz", hash = "sha256:dd8d182285a0fe56bace7f45b5e7d1a6ebcbf524e8f3bd87eb0f125271b8831f"},
]
pylint = [
    {file = "pylint-2.4.4-py3-none-any.whl", hash = "sha256:886e6afc935ea2590b462664b161ca9a5e40168ea99e5300935f6591ad467df4"},
    {file = "pylint-2.4.4.tar.gz", hash = "sha256:3db5468ad013380e987410a8d6956226963aed94ecb5f9d3a28acca6d9ac36cd"},
]
python-decouple = [
    {file = "python-decouple-3.3.tar.gz", hash = "sha256:55c546b85b0c47a15a47a4312d451a437f7344a9be3e001660bccd93b637de95"},
]
six = [
    {file = "six-1.14.0-py2.py3-none-any.whl", hash = "sha256:8f3cd2e254d8f793e7f3d6d9df77b92252b52637291d0f0da013c76ea2724b6c"},
    {file = "six-1.14.0.tar.gz", hash = "sha256:236bdbdce46e6e6a3d61a337c0f8b763ca1e8717c03b369e87a7ec7ce1319c0a"},
]
werkzeug = [
    {file = "Werkzeug-1.0.0-py2.py3-none-any.whl", hash = "sha256:6dc65cf9091cf750012f56f2cad759fa9e879f511b5ff8685e456b4e3bf90d16"},
    {file = "Werkzeug-1.0.0.tar.gz", hash = "sha256:169ba8a33788476292d04186ab33b01d6add475033dfc07215e6d219cc077096"},
]
wrapt = [
    {file = "wrapt-1.11.2.tar.gz", hash = "sha256:565a021fd19419476b9362b05eeaa094178de64f8361e44468f9e9d7843901e1"},
]
```

Não vou me extender mais nesse assunto mas como viram não tem grandes complicações na configuração do ambiente e do projeto. Fico aberto a susgestões de mudanças no que tenho feito e aberto a tirar dúvidas caso tenha alguma. Ainda preciso usar mais essa configuração de ambiente para ver todos os seus pontos fortes e fracos mas já me agradou a primeira vista.

Até!
