---
author: "João Maia"
date: 2020-06-15
title: "AWS IAM direto ao ponto"
tags: [
    "aws",
    "iam",
    "security"
]
categories: [
    "aws",
    "devops"
]
thumbnail: "img/aws-iam-direto-ao-ponto/aws-iam.png"
---

Olá,

sou usuário de AWS tanto da forma pessoal quanto profissional tem alguns anos e uma das coisas que deveriam ser suas preocupações ao criar uma conta na AWS é o Identity and Access Management (IAM). Tenho um sentimento que esse recurso da AWS é sempre usado de forma muito rasa e errada na maioria dos lugares pois as permissões configuradas são sempre brandas demais. Vou tentar nesse artigo passar uma visão geral sobre o recurso e minha opinião a respeito de como fazer um bom uso do mesmo.

# Modelo de segurança da AWS

Ao usar a AWS e não ter sua própria infraestrutura desde ambiente físico até o de aplicações entramos um modelo de responsabilidade compartilhada. A AWS no caso vai entrar com a responsabilidade física total para você da sua infraestrutura e de forma compartilhada na parte de software. O IAM é um dos principais recursos para gerenciar essa responsabilidade compartilhada. Podemos resumir todo o modelo de segurança da AWS em uma frase mas se tiver interesse em saber mais veja [aqui](https://aws.amazon.com/pt/compliance/shared-responsibility-model/).

> Por padrão tudo não é permitido, é sua responsabilidade liberar as permissões.

E assim começa esse modelo de responsabilidade compartilhada.

# Começando com segurança na AWS

Considerando que você está começando agora ou nunca tenha olhado com mais carinho para a segurança da sua conta na AWS, vamos conferir essa lista antes de dar continuidade no artigo:

1. Conta `root` sem credenciais de API: a conta `root` é a conta que criou a conta na AWS e é importante não ativar credenciais de acesso nela, use outra conta para isso com responsabilidades mais bem definidas;
2. Use e abuse de criação de regras, grupos e usuários para ter acessos a sua conta da AWS de forma bem espcializada para fazer apenas o que for preciso ser feito;
3. Política de senhas: é possível cofigurar uma política de senha para todos que tiverem uma conta na sua organização da AWS, por exemplo, tamanho mínimo, variedade de elementos e rotação da senha;
4. CloudTrail habilitado pois ele faz log que tudo que acontece na sua conta e é muito importante para auditoria ou criação de monitorias e alertas; e
5. Autenticação de dois fatores, é importante pedir que todos os usuários façam a configuração ou simplesmente impor via AWS Config.

# Amazon Resource Names (arn)

Trabalhando via interface web na AWS esse recurso as vezes passa despercebido apesar de preciso em alguns momentos para integrar alguns serviços. Quem lida com AWS através de código seja via API direto ou Terraform já deve estar mais familiarizado com o termo. O `arn` é o identificador universal de objetos na AWS, ou seja, seu `bucket` no s3, sua instância no EC2 e por ai vai tudo possui um `arn`. Saber esse valor é muito importante para o momento de criar `Policy` conseguir ser o mais granular possível e menos genéricas as suas regras.

# Visão geral do IAM

![Visão geral do IAM](/img/aws-iam-direto-ao-ponto/iam-visao-geral.jpg)

Na imagem acima podemos ver de forma simples como é a estrutura do IAM. O IAM como já foi dito é um recurso dentro da sua conta da AWS que dentro dele temos `Group`, `User`, `Role` e `Policy`, apesar de não estar associada de forma direta ao IAM na imagem ela faz parte do recurso mas ela tem apenas valor quando associada a `Group`, `User` ou `Role`.

![AWS IAM visão web](/img/aws-iam-direto-ao-ponto/aws-iam-web.png)

## Policy

`Policy` é a forma mais granular de criar permissões na AWS, basicamente você vai dizer se algo pode ou não, o que é essa ação e ao que dentro da AWS ela será aplicada. Vamos exemplificar, uma `Policy` poder ser a permissão de fazer um `GetObject` em um `bucket` específico do `s3`, se essa `Policy` for dada a um usuário ele terá o poder de pegar qualquer arquivo desse `bucket` e fazer um cópia local em sua máquina dado que ele saiba da existência do mesmo pois a permissão dada não permite ele listar os itens do `bucket`.

### Exemplo

A `Policy` é descrita em formato `JSON`, segue um exemplo:

```json
{
    "Statement": [
        {
            "Action": [
                "s3:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "*"
            ]
        },
        {
            "Action": [
                "s3:*"
            ],
            "Effect": "Deny",
            "Resource": [
                "arn:aws:s3:::db-backups"
            ]
        }
    ]
}
```

No exemplo acima estou permitindo tudo no `s3` no primeiro momento mas depois me certifico que no `bucket` de backups de um banco de dados nada é permitido.

Políticas dinâmicas:

```json
{
   "Statement":[{
      "Effect":"Allow",
      "Action":"s3:*",
      "Resource":"arn:aws:s3:::db-backups",
      "Condition":{
         "Bool":{
            "aws:SecureTransport":"true"
            }
         }
      }
   ]
}
```

Você pode aprender mais sobre isso [aqui](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html).

### Como funciona

Como o IAM executa a `Policy`:

- todas as policies de usuário e grupo são combinadas na hora da requisição
- se tiver alguma regra de negação explicita a requisição é negada
- se tiver alguma regra de aceite explicita a requisição é aprovada
- se não tiver alguma regra não explicita permitindo é negada

### Limitações

- Nem tudo na AWS tem `arn`, por exemplo, EC2 ativas;
    - O pessoal do Nubank deu uma [palestra interessante na QConSP 2019](https://www.infoq.com/br/presentations/como-nubank-automatiza-seguranca-aws/) sobre segurança e acredito que a criação de função Lambdas seja uma das soluções deles para contornar isso.
- Representar a sua estrutura organizacional na AWS pode trazer diversas dúvidas, [Conway's law](https://en.wikipedia.org/wiki/Conway%27s_law) diz mais a respeito disso. Vamos ver alguns desses modelos de organização:
    - Conta única da AWS: Gerenciamento centralizado com sobrecarga mínima. Proteger com usuários personalizados limitados no acesso a ações e recursos;
    - Contas separadas de produção, desenvolvimento e teste da AWS: Contempla os recursos de uma conta única com separação entre várias contas da AWS. Esforço adicional necessário para disponibilizar recursos entre contas;
    - Várias contas da AWS, uma por departamento: O acesso a ações e recursos pode seguir procedimentos radicalmente diferentes para cada organização. A cooperação em recursos compartilhados privados é marginalmente mais complexa; e
    - Várias contas da AWS, uma por função: Gerenciamento funcionalmente centralizado com contas diferentes para DNS, DBMS, CDN, CMS ou qualquer outro serviço.

## Role

`Role` é uma composição de uma ou mais `Policy`, o ideal é manter `Policy` de maneira exuta para facilitar a gestão. Para entender melhor o a `Role` vamos exemplificar. Supondo que você trabalhe em um local que tenha passar por auditorias constantes e o auditor precise de acesso a AWS para realizar seu trabalho. Um forma de resolver isso é criar `Policy` de leituras a tudo que está dentro do escopo da audiotira por recursos da AWS, por exemplo, uma para `s3`, uma para `VPC` e outra para `EC2`. Com isso, pode ser agrupar todas em uma `Role` e atribuir ela ao auditor. Outro exemplo, como pode ver na figura que é através de `Role` que conseguimos fazer acessos cruzados entre contas da AWS.

Além disso, se parecem com o IAM User mas sem ter um acesso direto pois não possuem senha ou credenciais de acesso. Um exemplo disso é o uso em EC2 com o ec2-profile, com ele você pode dar permissões a uma instância em execução como se fosse um usuário com isso qualquer pessoa ou software executando dentro da instância pode performar regras de uma `Policy` sem possuir ela diretamente.

[Exemplo da AWS](https://docs.aws.amazon.com/pt_br/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) de como fazer acesso cruzado entre contas usando Role.


## IAM User e Group

`User` e `Group` não tem grandes mistérios, eles funcionam de maneira normal como é em todo sistema de gestão de usuários e assumem `Policy` e/ou `Role`. Se vovê entende como funcionam usuários e grupos no Unix ou Windows você já entende o conceito básico do IAM User e Group.

Usuários podem ser pessoas que fazem acesso ao console web e API ou apenas automações que usam apenas a API sem acesso a parte web. Para quem vem de gestão de usuários como LDAP e AD o IAM possui o recurso de criar pastas para segregar usuários igual esses sistemas possuem.

# Conclusão

O AWS IAM é um grande universo mas não complexo, talvez a sua maior dificuldade vai ser na hora de criar as `Policy` de maneira granular pois como são muitas as ações disponíveis é complicado as vezes saber quais são as que você precisa exatamente.

O mais importante aqui é ficar com a mensagem de não negligenciar o uso do IAM e dar acessos deliberados em sua organização pois isso pode custar caro no futuro e se possível crie automações para configurar isso e não se tornar uma dor de cabeça no futuro. A falta de automação as vezes pode ser a porta de entrada para falhas de segurança quando ela encontra a ~~agilidade~~ pressa em entregar.

Existem outros pontos do AWS IAM que não foram abordados como podem ver na imagem da interface web deles nesse artigo mas acho que o material aqui já tem um grande valor para começar, podemos pensar em novos artigos para abordar o que ficou faltando. Fico no aguardo de melhorias e opiniões a respeito do que escrevi aqui. =)
