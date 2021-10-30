---
author: "João Maia"
date: 2021-10-30
title: "Dica de ferramenta: SDKMAN!"
tags: [
    "JVM",
    "java",
    "scala",
    "gradle"
]
categories: [
    "desenvolvimento"
]
thumbnail: "img/dica-de-ferramenta-sdkman/sdkman.png"
---

Olá,

se você trabalha com projetos que usam [Java Virtual Machine (JVM)](https://en.wikipedia.org/wiki/Java_virtual_machine), ou seja, [Java](https://www.java.com/en/), [Kotlin](https://kotlinlang.org/) e [Scala](https://scala-lang.org/) e tem ferramentas de gestão de projeto como [Maven](https://maven.apache.org/), [Gradle](https://gradle.org/) e [sbt](https://www.scala-sbt.org/) o [SDKMAN!](https://sdkman.io/) pode ser a ferramenta que você precisa para ajudar na gestão de diversos projetos com versões diferentes dessas ferramentas. Eu conheci a ferramenta a um tempo atrás pelo Twitter com o [Kico](https://twitter.com/loboweissmann), ele escreveu um [guia](https://itexto.com.br/configurando-seu-ambiente-de-desenvolvimento-java-com-sdkman/) da ferramenta, e desde então tenho usado ela para organizar os projetos aqui.

O processo de instalar e configurar o SDKMAN! é bem simples e você pode seguir o [tutorial do site oficial](https://sdkman.io/install). Depois de instalado e configurado basta chamar ele na linha de comando para entender seu funcionamento:

```
$ sdk
We periodically need to update the local cache. Please run:

  $ sdk update

==== BROADCAST =================================================================
* 2021-10-30: jreleaser 0.8.0 available on SDKMAN! https://github.com/jreleaser/jreleaser/releases/tag/v0.8.0
* 2021-10-28: micronaut 3.1.3 available on SDKMAN!
* 2021-10-26: gradle 7.3-rc-3 available on SDKMAN!
================================================================================

Usage: sdk <command> [candidate] [version]
       sdk offline <enable|disable>

   commands:
       install   or i    <candidate> [version] [local-path]
       uninstall or rm   <candidate> <version>
       list      or ls   [candidate]
       use       or u    <candidate> <version>
       config
       default   or d    <candidate> [version]
       home      or h    <candidate> <version>
       env       or e    [init|install|clear]
       current   or c    [candidate]
       upgrade   or ug   [candidate]
       version   or v
       broadcast or b
       help
       offline           [enable|disable]
       selfupdate        [force]
       update
       flush             [archives|tmp|broadcast|version]

   candidate  :  the SDK to install: groovy, scala, grails, gradle, kotlin, etc.
                 use list command for comprehensive list of candidates
                 eg: $ sdk list
   version    :  where optional, defaults to latest stable if not provided
                 eg: $ sdk install groovy
   local-path :  optional path to an existing local installation
                 eg: $ sdk install groovy 2.4.13-local /opt/groovy-2.4.13
```

O tutorial de uso também é bem simples e prático e pode ser visto [aqui](https://sdkman.io/usage).

Exemplo da instalação do meu computador pessoal:

```
$ sdk current

Using:

gradle: 6.7
java: 11.0.11.hs-adpt
kotlin: 1.5.21
$ sdk version

SDKMAN 5.12.2
```

Espero que a ferramenta te ajude, por hoje é só.

# Referências

- JVM: https://en.wikipedia.org/wiki/Java_virtual_machine
- Java: https://www.java.com/en/
- Kotlin: https://kotlinlang.org/
- Scala: https://scala-lang.org/
- Maven: https://maven.apache.org/
- Gradle: https://gradle.org/
- sbt: https://www.scala-sbt.org/
- SDKMAN!: https://sdkman.io/
- Twitter do Kico: https://twitter.com/loboweissmann
- Guia de uso do SDKMAN! da itexto: https://itexto.com.br/configurando-seu-ambiente-de-desenvolvimento-java-com-sdkman/
- Tutorial de instalação do SDKMAN!: https://sdkman.io/install
- Tutorial de uso do SDKMAN!: https://sdkman.io/usage