---
author: "João Maia"
date: 2021-03-07
title: "Conhecendo o Terraform"
tags: [
    "terraform",
]
categories: [
    "devops"
]
thumbnail: "img/conhecendo-o-terraform/terraform.png"
---

Olá,

venho aqui no blog compartilhando tem um certo tempo uma apresentação básica de tecnologias mdoernas voltadas a área de infraestrutura que tem ganhado muito força e atração no mercado. Sei que não são realidade também mas na minha bolha social e profissional de pessoas que já estão lidando com infraestrutura como código elas já são realidades e hoje venho falar do [Terraform](https://www.terraform.io/) que fecha esse ciclo de ferramentas que venho apresentando. Olhando de maneira breve e superficial ou vendo pessoas comentar a respeito dele temos um primeiro entendimento que o Terraform é uma ferramenta apenas de provisionar infraestrutura em lugares como [AWS](https://aws.amazon.com/), [Google Cloud](https://cloud.google.com/) e [Azure](https://azure.microsoft.com/en-us/). Isso não é uma verdade pois o objetivo dele é provisionar *qualquer* serviço de Cloud desde que ele forneça uma API para tal, dentro da terminologia do Terraform chamamos esse serviço de Cloud de `provider`.

# Comparando Terraform com outras soluções

No primeiro momento que vemos o uso de Terraform muito orientado a AWS ou outro Cloud Provider já logo imaginamos que ele faz a mesma coisa que [Ansible](https://www.ansible.com/), [Puppet](https://puppet.com/) ou [Chef](https://www.chef.io/) e são a mesma coisa, isso não é verdade, as soluções mencionadas são ferramentas de gestão de configuração. O Ansible já apresentado aqui no blog, por exemplo, tem a capacidade de provisionar serviços dentro de uma máquina virtual como, por exemplo, instalar o [NGINX](https://nginx.org/en/). O Terraform não tem esse mesmo papel do Ansible, na verdade eles se complementam pois ele pode consumir trabalhos feitos pelo Ansible para provisionar sua solução na sua Cloud, por exemplo, a sua imagem de máquina virtual de NGINX o Terraform pode consumir para criar uma solução com ela em alta disponibilidade na AWS usando multiplas zonas e regiões através de balanceadores de carga e afins tudo feito via Terraform.

Então, podemos imaginar que o Terraform é uma solução igual ao [Cloud Formation](https://aws.amazon.com/cloudformation/) para quem usa AWS pois usando ele vamos definir nossa infraestrutura na nuvem. Qual a diferença entre eles? No Terraform você tem uma única sintaxe que poderá usar a vários `providers` mas isso não significa que o mesmo código que você usa para subir uma máquina virtual na AWS seja igual ao mesmo na Azure. No Terraform, é trabalhado a fase de planejamento, ou seja, antes de aplicar sua mudança na definição de infraestrutura é possível ver como ela será afetada além de conseguir ver um grafo de dependência do mesmo. Isso é extremamente interessante pois vai te ajudar a evitar possíveis problemas se aplicado diretamente.

# Instalando o Terraform

Pessoalmente gosto mais de gestores de versões das ferramentas que uso e no caso do Terraform temos a solução do [tfenv](https://github.com/tfutils/tfenv). Outras formas de usar o terraform seria baixando o binário dele e colocando no sue `$PATH` ou usando algumas imagem Docker. A cara do `tfenv`:

```
$ tfenv 
tfenv 2.0.0-37-g0494129
Usage: tfenv <command> [<options>]

Commands:
   install       Install a specific version of Terraform
   use           Switch a version to use
   uninstall     Uninstall a specific version of Terraform
   list          List all installed versions
   list-remote   List all installable versions  
```

Listando versões disponíveis:

```
$ tfenv list-remote
0.15.0-beta1
0.15.0-alpha20210210
0.15.0-alpha20210127
0.15.0-alpha20210107
0.14.8
0.14.7
0.14.6
...
0.2.0
0.1.1
0.1.0
```

Instalando:

```
$ tfenv install 0.15.0-beta1
Installing Terraform v0.15.0-beta1
Downloading release tarball from https://releases.hashicorp.com/terraform/0.15.0-beta1/terraform_0.15.0-beta1_linux_amd64.zip
####################################################################################################################################################################################################################################################### 100.0%
Downloading SHA hash file from https://releases.hashicorp.com/terraform/0.15.0-beta1/terraform_0.15.0-beta1_SHA256SUMS
Unable to verify OpenPGP signature unless logged into keybase and following hashicorp
Archive:  tfenv_download.ArSDdY/terraform_0.15.0-beta1_linux_amd64.zip
  inflating: /home/jvrmaia/.tfenv/versions/0.15.0-beta1/terraform  
Installation of terraform v0.15.0-beta1 successful. To make this your default version, run 'tfenv use 0.15.0-beta1'
```

Usando:

```
$ tfenv use 0.15.0-beta1
Switching default version to v0.15.0-beta1
Switching completed
$ terraform -version
Terraform v0.15.0-beta1
on linux_amd64
```

# Na prática

O exemplo de códido apresentado aqui se encontra no repositório [terraform-intro-example](https://gitlab.com/jvrmaia/terraform-intro-example) na minha conta do GitLab, segue:


```terraform
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "us-east-1"
}

provider "aws" {
  region = var.aws_region
}

module "iam_user" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-user"
  version = "~> 3.0"

  name                          = "test-user"
  force_destroy                 = true
  create_iam_user_login_profile = false
}

module "iam_assumable_roles" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-assumable-roles"
  version = "~> 3.0"

  trusted_role_arns = [
    module.iam_user.this_iam_user_arn
  ]

  create_admin_role = true

  create_poweruser_role = true
  poweruser_role_name   = "developer"

  create_readonly_role       = true
  readonly_role_requires_mfa = false
}

output "access_key_id" {
  description = "test-user ACCESS_KEY_ID"
  value       = module.iam_user.this_iam_access_key_id
}

output "access_key_secret" {
  description = "test-user ACCESS_KEY_SECRET"
  value       = module.iam_user.this_iam_access_key_secret
  sensitive   = true
} 
```

Como podemos ver é um código bem simples que não faz muitas coisas pois não vou me aprofundar nesse assunto hoje, é só para ter um primeiro contato e noção da ferramenta. Nitidamente o código é voltado a AWS e logo no começo já podemos ver que o Terraform nos permite criar variáveis e configurar valores padrão nelas uma coisa bem interessante nisso é a `description` que ajuda na documentação do código.

Lembra que falei que no Terraform trabalhomos com `providers`? Então, como pode notar logo em segui temos essa declaração de `provider` definida. Em sequência, vem outro recurso muito legal do Terraform que são os módulos que nos permitem evitar repetição de código e resolver problemas complexos através de composição de soluções. Por fim, crio algumas variáveis de retorno do meu código com uma particularidade na última que marco como `sensitive` pois se trata de um segredo.

No começo comentei que o Terraform diferente de outras ferramentas tem uma parte de teste da mudança, vamos conferir como é isso:

```terraform
$ terraform plan

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  # module.iam_assumable_roles.data.aws_iam_policy_document.assume_role will be read during apply
  # (config refers to values not yet known)
 <= data "aws_iam_policy_document" "assume_role"  {
      + id   = (known after apply)
      + json = (known after apply)

      + statement {
          + actions = [
              + "sts:AssumeRole",
            ]
          + effect  = "Allow"

          + principals {
              + identifiers = [
                  + (known after apply),
                ]
              + type        = "AWS"
            }
          + principals {
              + identifiers = []
              + type        = "Service"
            }
        }
    }

  # module.iam_assumable_roles.data.aws_iam_policy_document.assume_role_with_mfa will be read during apply
  # (config refers to values not yet known)
 <= data "aws_iam_policy_document" "assume_role_with_mfa"  {
      + id   = (known after apply)
      + json = (known after apply)

      + statement {
          + actions = [
              + "sts:AssumeRole",
            ]
          + effect  = "Allow"

          + condition {
              + test     = "Bool"
              + values   = [
                  + "true",
                ]
              + variable = "aws:MultiFactorAuthPresent"
            }
          + condition {
              + test     = "NumericLessThan"
              + values   = [
                  + "86400",
                ]
              + variable = "aws:MultiFactorAuthAge"
            }

          + principals {
              + identifiers = [
                  + (known after apply),
                ]
              + type        = "AWS"
            }
          + principals {
              + identifiers = []
              + type        = "Service"
            }
        }
    }

  # module.iam_assumable_roles.aws_iam_role.admin[0] will be created
  + resource "aws_iam_role" "admin" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "admin"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role.poweruser[0] will be created
  + resource "aws_iam_role" "poweruser" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "developer"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role.readonly[0] will be created
  + resource "aws_iam_role" "readonly" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "readonly"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0] will be created
  + resource "aws_iam_role_policy_attachment" "admin" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
      + role       = "admin"
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0] will be created
  + resource "aws_iam_role_policy_attachment" "poweruser" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/PowerUserAccess"
      + role       = "developer"
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0] will be created
  + resource "aws_iam_role_policy_attachment" "readonly" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
      + role       = "readonly"
    }

  # module.iam_user.aws_iam_access_key.this_no_pgp[0] will be created
  + resource "aws_iam_access_key" "this_no_pgp" {
      + create_date          = (known after apply)
      + encrypted_secret     = (known after apply)
      + id                   = (known after apply)
      + key_fingerprint      = (known after apply)
      + secret               = (sensitive value)
      + ses_smtp_password_v4 = (sensitive value)
      + status               = (known after apply)
      + user                 = "test-user"
    }

  # module.iam_user.aws_iam_user.this[0] will be created
  + resource "aws_iam_user" "this" {
      + arn           = (known after apply)
      + force_destroy = true
      + id            = (known after apply)
      + name          = "test-user"
      + path          = "/"
      + unique_id     = (known after apply)
    }

Plan: 8 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + access_key_id     = (known after apply)
  + access_key_secret = (sensitive value)

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

Assim como usamos o `diff` no terminal para comparar dois arquivos ou `git diff` para verificar a diferença de dois commits no git o `plan` do Terraform nos mostra essa mesma visão de qual a diferença prevista no seu `provider` caso você aplique esse código. Um ponto de atenção, não é verdade dizer que já que o `plan` funcionou o `apply` vai funcionar também, podem ter outras questões envolvidas como questões de permissão que vão te impedir de ter sucesso ao aplicar o código. Vamos aplicar ele agora:

```terraform
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  # module.iam_assumable_roles.data.aws_iam_policy_document.assume_role will be read during apply
  # (config refers to values not yet known)
 <= data "aws_iam_policy_document" "assume_role"  {
      + id   = (known after apply)
      + json = (known after apply)

      + statement {
          + actions = [
              + "sts:AssumeRole",
            ]
          + effect  = "Allow"

          + principals {
              + identifiers = [
                  + (known after apply),
                ]
              + type        = "AWS"
            }
          + principals {
              + identifiers = []
              + type        = "Service"
            }
        }
    }

  # module.iam_assumable_roles.data.aws_iam_policy_document.assume_role_with_mfa will be read during apply
  # (config refers to values not yet known)
 <= data "aws_iam_policy_document" "assume_role_with_mfa"  {
      + id   = (known after apply)
      + json = (known after apply)

      + statement {
          + actions = [
              + "sts:AssumeRole",
            ]
          + effect  = "Allow"

          + condition {
              + test     = "Bool"
              + values   = [
                  + "true",
                ]
              + variable = "aws:MultiFactorAuthPresent"
            }
          + condition {
              + test     = "NumericLessThan"
              + values   = [
                  + "86400",
                ]
              + variable = "aws:MultiFactorAuthAge"
            }

          + principals {
              + identifiers = [
                  + (known after apply),
                ]
              + type        = "AWS"
            }
          + principals {
              + identifiers = []
              + type        = "Service"
            }
        }
    }

  # module.iam_assumable_roles.aws_iam_role.admin[0] will be created
  + resource "aws_iam_role" "admin" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "admin"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role.poweruser[0] will be created
  + resource "aws_iam_role" "poweruser" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "developer"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role.readonly[0] will be created
  + resource "aws_iam_role" "readonly" {
      + arn                   = (known after apply)
      + assume_role_policy    = (known after apply)
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "readonly"
      + path                  = "/"
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0] will be created
  + resource "aws_iam_role_policy_attachment" "admin" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
      + role       = "admin"
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0] will be created
  + resource "aws_iam_role_policy_attachment" "poweruser" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/PowerUserAccess"
      + role       = "developer"
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0] will be created
  + resource "aws_iam_role_policy_attachment" "readonly" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
      + role       = "readonly"
    }

  # module.iam_user.aws_iam_access_key.this_no_pgp[0] will be created
  + resource "aws_iam_access_key" "this_no_pgp" {
      + create_date          = (known after apply)
      + encrypted_secret     = (known after apply)
      + id                   = (known after apply)
      + key_fingerprint      = (known after apply)
      + secret               = (sensitive value)
      + ses_smtp_password_v4 = (sensitive value)
      + status               = (known after apply)
      + user                 = "test-user"
    }

  # module.iam_user.aws_iam_user.this[0] will be created
  + resource "aws_iam_user" "this" {
      + arn           = (known after apply)
      + force_destroy = true
      + id            = (known after apply)
      + name          = "test-user"
      + path          = "/"
      + unique_id     = (known after apply)
    }

Plan: 8 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + access_key_id     = (known after apply)
  + access_key_secret = (sensitive value)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.iam_user.aws_iam_user.this[0]: Creating...
module.iam_user.aws_iam_user.this[0]: Creation complete after 2s [id=test-user]
module.iam_assumable_roles.data.aws_iam_policy_document.assume_role: Reading...
module.iam_assumable_roles.data.aws_iam_policy_document.assume_role_with_mfa: Reading...
module.iam_user.aws_iam_access_key.this_no_pgp[0]: Creating...
module.iam_assumable_roles.data.aws_iam_policy_document.assume_role: Read complete after 0s [id=119073080]
module.iam_assumable_roles.data.aws_iam_policy_document.assume_role_with_mfa: Read complete after 0s [id=3139435766]
module.iam_assumable_roles.aws_iam_role.readonly[0]: Creating...
module.iam_assumable_roles.aws_iam_role.poweruser[0]: Creating...
module.iam_assumable_roles.aws_iam_role.admin[0]: Creating...
module.iam_user.aws_iam_access_key.this_no_pgp[0]: Creation complete after 1s [id=AKIAREMIJPA7PUSCWR34]
module.iam_assumable_roles.aws_iam_role.readonly[0]: Creation complete after 10s [id=readonly]
module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0]: Creating...
module.iam_assumable_roles.aws_iam_role.poweruser[0]: Still creating... [10s elapsed]
module.iam_assumable_roles.aws_iam_role.admin[0]: Still creating... [10s elapsed]
module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0]: Creation complete after 1s [id=readonly-20210320110254508000000001]
module.iam_assumable_roles.aws_iam_role.poweruser[0]: Creation complete after 14s [id=developer]
module.iam_assumable_roles.aws_iam_role.admin[0]: Creation complete after 14s [id=admin]
module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0]: Creating...
module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0]: Creating...
module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0]: Creation complete after 2s [id=developer-20210320110259289300000002]
module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0]: Creation complete after 2s [id=admin-20210320110259289400000003]

Apply complete! Resources: 8 added, 0 changed, 0 destroyed.

Outputs:

access_key_id = "AKIAREMIJPA7PUSCWR34"
access_key_secret = <sensitive>
```

Como podemos ver ele mostra o resultado `plan` novamente e pede a confirmação para dar continuidade no `apply`, confirmando ele exibe o log do que acontece e por fim exibe os `outputs` definidos. Depois de executar vai reparar que no mesmo diretório foi criado o `terraform.tfstate` que é o estado final com os valores dos recursos na AWS, isso vai nos ajudar na evolução do projeto, nele o dado que marcamos como sensível vai estar de forma aberta também. Por fim, podemos também desfazer tudo que foi feito com o comando de `destroy`:

```terraform
$ terraform destroy

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # module.iam_assumable_roles.aws_iam_role.admin[0] will be destroyed
  - resource "aws_iam_role" "admin" {
      - arn                   = "arn:aws:iam::078132115518:role/admin" -> null
      - assume_role_policy    = jsonencode(
            {
              - Statement = [
                  - {
                      - Action    = "sts:AssumeRole"
                      - Condition = {
                          - Bool            = {
                              - aws:MultiFactorAuthPresent = "true"
                            }
                          - NumericLessThan = {
                              - aws:MultiFactorAuthAge = "86400"
                            }
                        }
                      - Effect    = "Allow"
                      - Principal = {
                          - AWS = "arn:aws:iam::078132115518:user/test-user"
                        }
                      - Sid       = ""
                    },
                ]
              - Version   = "2012-10-17"
            }
        ) -> null
      - create_date           = "2021-03-20T11:02:55Z" -> null
      - force_detach_policies = false -> null
      - id                    = "admin" -> null
      - managed_policy_arns   = [] -> null
      - max_session_duration  = 3600 -> null
      - name                  = "admin" -> null
      - path                  = "/" -> null
      - unique_id             = "AROAREMIJPA7HRCB6FDJT" -> null

      - inline_policy {}
    }

  # module.iam_assumable_roles.aws_iam_role.poweruser[0] will be destroyed
  - resource "aws_iam_role" "poweruser" {
      - arn                   = "arn:aws:iam::078132115518:role/developer" -> null
      - assume_role_policy    = jsonencode(
            {
              - Statement = [
                  - {
                      - Action    = "sts:AssumeRole"
                      - Condition = {
                          - Bool            = {
                              - aws:MultiFactorAuthPresent = "true"
                            }
                          - NumericLessThan = {
                              - aws:MultiFactorAuthAge = "86400"
                            }
                        }
                      - Effect    = "Allow"
                      - Principal = {
                          - AWS = "arn:aws:iam::078132115518:user/test-user"
                        }
                      - Sid       = ""
                    },
                ]
              - Version   = "2012-10-17"
            }
        ) -> null
      - create_date           = "2021-03-20T11:02:55Z" -> null
      - force_detach_policies = false -> null
      - id                    = "developer" -> null
      - managed_policy_arns   = [] -> null
      - max_session_duration  = 3600 -> null
      - name                  = "developer" -> null
      - path                  = "/" -> null
      - unique_id             = "AROAREMIJPA7IECKWDPBQ" -> null

      - inline_policy {}
    }

  # module.iam_assumable_roles.aws_iam_role.readonly[0] will be destroyed
  - resource "aws_iam_role" "readonly" {
      - arn                   = "arn:aws:iam::078132115518:role/readonly" -> null
      - assume_role_policy    = jsonencode(
            {
              - Statement = [
                  - {
                      - Action    = "sts:AssumeRole"
                      - Effect    = "Allow"
                      - Principal = {
                          - AWS = "arn:aws:iam::078132115518:user/test-user"
                        }
                      - Sid       = ""
                    },
                ]
              - Version   = "2012-10-17"
            }
        ) -> null
      - create_date           = "2021-03-20T11:02:51Z" -> null
      - force_detach_policies = false -> null
      - id                    = "readonly" -> null
      - managed_policy_arns   = [] -> null
      - max_session_duration  = 3600 -> null
      - name                  = "readonly" -> null
      - path                  = "/" -> null
      - unique_id             = "AROAREMIJPA7PSOGAMO4N" -> null

      - inline_policy {}
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0] will be destroyed
  - resource "aws_iam_role_policy_attachment" "admin" {
      - id         = "admin-20210320110259289400000003" -> null
      - policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess" -> null
      - role       = "admin" -> null
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0] will be destroyed
  - resource "aws_iam_role_policy_attachment" "poweruser" {
      - id         = "developer-20210320110259289300000002" -> null
      - policy_arn = "arn:aws:iam::aws:policy/PowerUserAccess" -> null
      - role       = "developer" -> null
    }

  # module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0] will be destroyed
  - resource "aws_iam_role_policy_attachment" "readonly" {
      - id         = "readonly-20210320110254508000000001" -> null
      - policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess" -> null
      - role       = "readonly" -> null
    }

  # module.iam_user.aws_iam_access_key.this_no_pgp[0] will be destroyed
  - resource "aws_iam_access_key" "this_no_pgp" {
      - create_date          = "2021-03-20T11:02:45Z" -> null
      - id                   = "AKIAREMIJPA7PUSCWR34" -> null
      - secret               = (sensitive value)
      - ses_smtp_password_v4 = (sensitive value)
      - status               = "Active" -> null
      - user                 = "test-user" -> null
    }

  # module.iam_user.aws_iam_user.this[0] will be destroyed
  - resource "aws_iam_user" "this" {
      - arn           = "arn:aws:iam::078132115518:user/test-user" -> null
      - force_destroy = true -> null
      - id            = "test-user" -> null
      - name          = "test-user" -> null
      - path          = "/" -> null
      - unique_id     = "AIDAREMIJPA7EUM7QNEOJ" -> null
    }

Plan: 0 to add, 0 to change, 8 to destroy.

Changes to Outputs:
  - access_key_id     = "AKIAREMIJPA7PUSCWR34" -> null
  - access_key_secret = (sensitive value)

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0]: Destroying... [id=developer-20210320110259289300000002]
module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0]: Destroying... [id=admin-20210320110259289400000003]
module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0]: Destroying... [id=readonly-20210320110254508000000001]
module.iam_user.aws_iam_access_key.this_no_pgp[0]: Destroying... [id=AKIAREMIJPA7PUSCWR34]
module.iam_assumable_roles.aws_iam_role_policy_attachment.readonly[0]: Destruction complete after 1s
module.iam_user.aws_iam_access_key.this_no_pgp[0]: Destruction complete after 1s
module.iam_assumable_roles.aws_iam_role_policy_attachment.poweruser[0]: Destruction complete after 1s
module.iam_assumable_roles.aws_iam_role.readonly[0]: Destroying... [id=readonly]
module.iam_assumable_roles.aws_iam_role_policy_attachment.admin[0]: Destruction complete after 1s
module.iam_assumable_roles.aws_iam_role.poweruser[0]: Destroying... [id=developer]
module.iam_assumable_roles.aws_iam_role.admin[0]: Destroying... [id=admin]
module.iam_assumable_roles.aws_iam_role.admin[0]: Destruction complete after 2s
module.iam_assumable_roles.aws_iam_role.readonly[0]: Destruction complete after 2s
module.iam_assumable_roles.aws_iam_role.poweruser[0]: Destruction complete after 2s
module.iam_user.aws_iam_user.this[0]: Destroying... [id=test-user]
module.iam_user.aws_iam_user.this[0]: Destruction complete after 7s

Destroy complete! Resources: 8 destroyed.
```

Isso é bem importante pois caso você estaja apenas testando algo na AWS você vai garantir que não terá nenhum custo imprevisto com algo que esqueceu ligado. =)

Antes de finalizar, em nenhum momento mostrei as configurações de acesso para modificar a AWS, tudo isso foi porque já tenho na minha máquina o AWS CLI configurado e ele usou a configuração dele para ter acesso.

# Concluindo

O Terraform é uma ferramenta bem legal e poderosa para essa nossa jornada de infraestrutura como código mas ela não é perfeita e nunca será como todos as outras ferramentas que fazem o mesmo trabalho ou parecido. Cabe a você entender o seu contexto e saber como aplicar ele da melhor forma. Segue uma lista de lugares onde você pode aprender mais sobre Terraform:

- [Canal da Linux Tips](https://www.youtube.com/c/LinuxTips/featured)
- [Terraform Up & Running](https://www.amazon.com.br/Terraform-Running-Writing-Infrastructure-English-ebook/dp/B07XKF258P/)
- [The Terraform Book](https://www.amazon.com.br/Terraform-Book-English-James-Turnbull-ebook/dp/B01MZYE7OY)

# Referências

- Terraform: https://www.terraform.io/
- AWS: https://aws.amazon.com/
- Google Cloud: https://cloud.google.com/
- Azure: https://azure.microsoft.com/en-us/
- Ansible: https://www.ansible.com/
- Puppet: https://puppet.com/
- Chef: https://www.chef.io/
- NGINX: https://nginx.org/en/
- AWS Cloud Formation: https://aws.amazon.com/cloudformation/
- tfenv: https://github.com/tfutils/tfenv
- Canal da Linux Tips: https://www.youtube.com/c/LinuxTips/featured
- Terraform Up & Running: https://www.amazon.com.br/Terraform-Running-Writing-Infrastructure-English-ebook/dp/B07XKF258P/
- The Terraform Book: https://www.amazon.com.br/Terraform-Book-English-James-Turnbull-ebook/dp/B01MZYE7OY
