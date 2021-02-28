---
author: "João Maia"
date: 2020-04-23
title: "Desenhando infraestrutura com código"
tags: [
    "devops",
    "diagrama",
    "python",
    "documentação"
]
categories: [
    "devops",
    "documentação"
]
thumbnail: "img/desenhando-infraestrutura-com-codigo/advanced_web_service_with_on-premise.png"
---

Olá,

Documentar é um processo importante no processo de criação da sua infraestrutura pois ajuda a todos da empresa terem visibilidade da arquitetura atual e pensar em possibilidades de melhorias e ajudar nos momentos de incidentes. Criar documentação é um processo chato para a maioria das pessoas da área de tecnologia e nem todos gostam de ferramentas visuais que muito fácil pode virar uma confusão total. Várias soluções para diagramas existem por ai sendo algumas delas o [Lucidchart](https://www.lucidchart.com/), [Cloudcraft](https://cloudcraft.co/) e [draw.io](https://app.diagrams.net/).

Como desenvolvedores gostam de programar, por que não desenhar a infraestrutura programando? Pensando nessa possibilidade procurei ferramentas para isso e acabei encontrando o [Diagrams](https://diagrams.mingrammer.com/), um projeto feito em Python que possibilita desenhar sua infraestrutura para as principais clouds do mercado ([AWS](https://diagrams.mingrammer.com/docs/nodes/aws), [GCP](https://diagrams.mingrammer.com/docs/nodes/gcp), [Azure](https://diagrams.mingrammer.com/docs/nodes/azure) e [AlibabaCloud](https://diagrams.mingrammer.com/docs/nodes/alibabacloud)) e outras coisas mais ([k8s](https://diagrams.mingrammer.com/docs/nodes/k8s) e [OnPremise](https://diagrams.mingrammer.com/docs/nodes/onprem)).

![Diagrams](/img/desenhando-infraestrutura-com-codigo/diagrams.png)

# Objetos

O `Diagrams` possui quatro objetos principais para desenhar diagramas: `Diagram`, `Node`, `Cluster` e `Edge`.

## Diagram

Esse é o seu desenho final que vai ser exportado em imagem. Ele pode ser modificado para exportar o diagrama nos seguintes formatos: png, jpg, svg, e pdf.

```python
from diagrams import Diagram
...

with Diagram("Minha infra", outformat="jpg"):
    ...
```

## Node

São os componentes do seu diagrama, por exemplo, EC2, RDS, Docker e outros.

```python
from diagrams import Diagram
from diagrams.aws.compute import EC2
from diagrams.aws.database import RDS
from diagrams.aws.network import ELB
from diagrams.aws.storage import S3

with Diagram("Web Services", show=False):
    ELB("lb") >> EC2("web") >> RDS("userdb") >> S3("store")
    ELB("lb") >> EC2("web") >> RDS("userdb") << EC2("stat")
    (ELB("lb") >> EC2("web")) - EC2("web") >> RDS("userdb")
```

Os objetos do tipo `Node` possuem operadores especiais: `>>`, `<<` e `-`. Eles são usados para mostrar a direção da conexão dos componentes, no caso do `-` a direção é nos 2 sentidos.

## Cluster

É a forma que agrupamos os componentes da nossa infraestrutura.

```python
from diagrams import Cluster, Diagram
from diagrams.aws.compute import ECS
from diagrams.aws.database import RDS
from diagrams.aws.network import Route53

with Diagram("Simple Web Service with DB Cluster", show=False):
    dns = Route53("dns")
    web = ECS("service")

    with Cluster("DB Cluster"):
        db_master = RDS("master")
        db_master - [RDS("slave1"),
                     RDS("slave2")]

    dns >> web >> db_master
```

## Edge

São as operações especiais de conexões entre os componentes que vimos em `Node`. Aqui tornamos ela explícita para customizações.

```python
from diagrams import Cluster, Diagram, Edge
from diagrams.onprem.analytics import Spark
from diagrams.onprem.compute import Server
from diagrams.onprem.database import PostgreSQL
from diagrams.onprem.inmemory import Redis
from diagrams.onprem.logging import Fluentd
from diagrams.onprem.monitoring import Grafana, Prometheus
from diagrams.onprem.network import Nginx
from diagrams.onprem.queue import Kafka

with Diagram(name="Advanced Web Service with On-Premise (colored)", show=False):
    ingress = Nginx("ingress")

    metrics = Prometheus("metric")
    metrics << Edge(color="firebrick", style="dashed") << Grafana("monitoring")

    with Cluster("Service Cluster"):
        grpcsvc = [
            Server("grpc1"),
            Server("grpc2"),
            Server("grpc3")]

    with Cluster("Sessions HA"):
        master = Redis("session")
        master - Edge(color="brown", style="dashed") - Redis("replica") << Edge(label="collect") << metrics
        grpcsvc >> Edge(color="brown") >> master

    with Cluster("Database HA"):
        master = PostgreSQL("users")
        master - Edge(color="brown", style="dotted") - PostgreSQL("slave") << Edge(label="collect") << metrics
        grpcsvc >> Edge(color="black") >> master

    aggregator = Fluentd("logging")
    aggregator >> Edge(label="parse") >> Kafka("stream") >> Edge(color="black", style="bold") >> Spark("analytics")

    ingress >> Edge(color="darkgreen") << grpcsvc >> Edge(color="darkorange") >> aggregator
```

# Exemplos

O fonte dos exemplos estão [aqui](https://github.com/jvrmaia/diagram-as-code).

## Blog

### Código

```python
from diagrams import Cluster, Diagram
from diagrams.aws.network import VPC
from diagrams.aws.database import RDS
from diagrams.aws.network import Route53
from diagrams.onprem.network import Nginx
from diagrams.onprem.container import Docker


with Diagram("Blog stack", show=False):
    with Cluster("AWS"):
        dns = Route53("meu.blog.br")

        with Cluster("us-east-1"):
            blog_vpc = VPC("blog_vpc")

            dns >> blog_vpc

            with Cluster("blog_vpc"):
                nginx = Nginx("proxy")
                docker = Docker("blog")
                postgres = RDS("postgres")

                blog_vpc >> nginx >> docker >> postgres
```

### Desenho

![blog stack](/img/desenhando-infraestrutura-com-codigo/blog_stack.png)

## Elastic Stack

### Código

```python
from diagrams import Cluster, Diagram
from diagrams.onprem.database import Cassandra
from diagrams.onprem.monitoring import Kibana
from diagrams.onprem.search import Elasticsearch
from diagrams.onprem.logging import Logstash


with Diagram("ELK", show=False):
    with Cluster("cassandra"):
        cassandra_cluster = [Cassandra("c01"),
                             Cassandra("c02"),
                             Cassandra("c03")]

    logstash = Logstash("logstash")
    elasticsearch = Elasticsearch("elasticsearch")
    kibana = Kibana("kibana")

    cassandra_cluster >> logstash >> elasticsearch >> kibana
```

### Desenho

![elk](/img/desenhando-infraestrutura-com-codigo/elk.png)

# Conclusão

Como vimos podemos organizar nossos diagramas no git e com isso manter um versionamento da nossa infraestrutura a medida que ela for evoluindo. Ainda estou explorando a ferramenta e estou gostando bastante da experiência. Uma dica é sempre que tiver os objetos já instânciados para serem conectados faça isso, meu primeiro diagrama fui fazer tudo no final e ficou um completo caos o desenho final. Agora, preciso ver se tem como escrever testes para o nosso desenho já que ele virou código.

O que você achou?
