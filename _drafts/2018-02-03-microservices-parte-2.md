---
layout: post
title: "Microservices (parte 2): Dificuldades e problemas"
modified:
categories: blog
excerpt: ""
tags: [SOA,Microservices]
image:
  feature:
date: 2017-07-28T00:58:56-02:00
---

# Imagem legal

No post anterior expliquei o que são microserviços e algumas de suas vantagens. Mas como toda rosa surge com espinhos que devem ser removidos cuidadosamente para você não furar o dedinho da sua pessoa amada, nesse post vou focar nas suas desvantagens e pontos de atenção necessários para desenvolver microserviços sem levar sua empresa a falência nem você ao suicídio.

Ter uma alta granuralidade de serviços na sua estrutura traz diversos desafios não funcionais como: logs, segurança, monitoramento, conversão de protocolos (caso os serviços utilizem diferentes tipos), etc. Todos esses requisitos normalmente vem junto no pacote do ESB. Embora alguns desses problemas possam ser endereçados também por API Gateways (falo mais no final do post) não ter esse tipo de ferramenta central faz com que tenhamos que nos preocupar com todo esse tooling de forma descentralizada. Isso vem com um custo e não é nada simples. 

Portanto, pense bem antes de entrar nesse mundo e não se deixe enganar apenas pelos pontos positivos que ele irá te trazer. Junto com os benefícios virão dificuldades e é melhor você estrar preparado. Vai demandar estudo

# Coisas a se pensar 

### Você provavelmente não precisa
Amazon, Netflix, and eBay
Motivação principal é escalabidade

The additional complexity that comes from distributed systems requires an additional level of maturity and investment

Microservices Hell / service-disoriented architecture

# Monolítico não é ruim [Post]
Nem todo monolítico é ruim, apesar da conotação negativa no senso comum do termo "monolítico".

-> MonilithFirst
Martin Fowler diz [nesse post](https://martinfowler.com/bliki/MonolithFirst.html) que os exemplos de sucesso de microservices que ele notou começaram por monolíticos que evoluiram para microserviços. 

Any refactoring of functionality between services is much harder than it is in a monolith. But even experienced architects working in familiar domains have great difficulty getting boundaries right at the beginning. By building a monolith first, you can figure out what the right boundaries are, before a microservices design brushes a layer of treacle over them. It also gives you time to develop the MicroservicePrerequisites you need for finer-grained services.  

**Atenção**: Não recomendo iniciar um contexto dessa maneira (em microserviço) a não ser que se tenha extrema convicção dos motivos citados acima. Acho muito mais sensato começar no monolítico (com um design bom e desacoplado) e ir migrando para microserviços algumas partes conforme a necessidade. Fazer o contrário é muito mais difícil. Além do mais, enquanto no monolítico, o refactoring se torna muito menos doloroso do que quando envolve a comunicação entre serviços.


# Pontos de atenção / Cuidados necessários

##### Sincrono vs Assincorno

Transacao e eventual consistency (With a monolith, you can update a bunch of things together in a single transaction.)

Não que no mundo monolítico isso seja uma maravilha. Já vi cenários em que TODAS as aplicações do cliente ficavam indisponíveis quando usuários de um sistema específico executavam um relatório que tinha uma consulta pesada e mal escrita na qual atolava o banco de dados. 

In the market today, RabbitMQ and Apache Kafka are both commonly used message bus technologies for asynchronous communication between microservices.

##### Service registry e service discovery
Each microservice handles registration within the central service registry. They typically register during the startup and periodically update the registry with current information. When the microservice gets terminated, it needs to be unregistered from the registry. The registry plays a critical role in orchestrating microservices.

Consul, etcd and Apache Zookeeper are examples of commonly used registries for microservices. 

-> Se nao tiver tem que fazer escalabildiade via servidor web ou através de um reverse proxy

##### Versionamento API?
Fica dificil evoluir os serviços sem um versionamento de API. O versionamento de api permite uma maior liberdade de evolução dos serviços e diminui o acoplamento.

##### Escolha do protocolo de comunicação
  SOAP pra comunicação interna
  REST é de fato o mais utilizado (isso por sí só poderia ser um post)

##### Infrastructure Automation

Integração contínua, entrega contínua (teste e deploy), Docker,
(ambiente desenvolvimento)

##### As boundaries nem sempre estão claras
botar isso?

##### Dependência entre serviços
Quanto maior a dependência entre seus serviços mais a vantagem de "deploy independente" se esvai. 

##### Ferramentas de monitoramento e logs
Common mistakes microservices
    ESBs resolviam requisitos não funcionais como "Autenticação, monitoramento, logs centaliazdos, mediação"

* Com microservices esses requisitos não funcionais muitas vezes são negligenciados.

multiplos bancos de dados e transações pode ser doloroso

* Documentação dos serviços existentes (onde encontrar?)
* Monitoramento. Os serviços estão distribuídos, ótimo! Mas qual a topologia de comunicação entre eles? Quantas requests por dia pro serviço X? Quantos erros no serviço Y?
* Log de erros individuais e centralizados

Dependendo da ferramenta de APM pode ser mais caro.

# Api Gateway (outro post?)

http://www.elemarjr.com/pt/2017/06/entendendo-microservicos/

