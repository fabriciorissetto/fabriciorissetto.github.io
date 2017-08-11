---
layout: post
title: "Microservices"
modified:
categories: blog
excerpt: ""
tags: [SOA]
image:
  feature:
date: 2017-07-28T00:58:56-02:00
---

*Esse post é a parte 2 de 3 tratando sobre arquitetura SOA. No anterior falei de [ESBs](www.fabriciorissetto.com/blog/ESBs/) e contextualizei o cenário SOA antes da hype de microservices. No próximo falarei sobre API Gateways...*

## Mais hype que Game of Thrones...

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-28-microservices/one-does-not-microservice.jpg">
</div>

Antes de tudo, quero começar esclarecendo uma coisa: **O padrão de microservices não é um substituto de SOA**. Pra falar a verdade a afirmação nem faz sentido, pelo menos não pra mim. Vejo microservices como um subset de SOA, uma das N maneiras de se fazer SOA.

Alguns dizem até que é um paradigma evolucionário do SOA "default". Dizem que microserviços é o *"SOA feito da maneira certa"*. O que eu também não concordo 100% e provavelmente você vai entender o porquê até o final do post.

## Afinal, o que é?

O termo ganhou popularidade depois desse [post](http://martinfowler.com/articles/microservices.html) feito pelo Martin Fowler e James Lewis onde eles descrevem as 9 principais características do padrão de microserviços. Sinceramente, não gosto da classificação macro dividida dentre esses 9 itens porque alguns se confundem (e poderiam ser fundidos) e acho que faltam alguns outros aspectos "macro" bem importantes. 

Me permito descrever uma definição resumida utilizando minhas palavras:

> É um monte de servicinho se chamando pra todo lado, em que cada serviço tem uma responsabilidade única e normalmente focada em uma pequena parte do negócio (ou até mesmo em um aspecto não funcional) que juntos compõem um objetivo maior de negócio. É separar aquele monolítico de Ecommerce em serviços menores como "Carrinho de compras", "Pagamento", "Catalogo de Produtos", "Autenticação" (microserviço não funcional), etc. 

Bonito de falar, difícil de fazer...

Pra mim, não tem maneira melhor de entender um padrão do que descrevendo as vantagens e desvantagens de se usá-lo. No final das contas a decisão de utilizar qualquer padrão passa por essa balança. Então vamos lá.

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-28-microservices/pros-cons.png">
</div>


## Vantagens

#### Smart endpoints and dumb pipes

Os serviços conhecem e consome uns aos outros. 

With the ESB, developers are encouraged to put more and more logic and intelligence into the central infrastructure (the pipes) and less and less into the services and the applications (the endpoints) that consume them. 

ESBs deliver some very important, in some cases critical, capabilities, especially for non-functional requirements like security, monitoring (what's happening in a distributed system.), and mediation

how to inform the service consumer of which standard it needs to comply with. This lines up with the challenge of letting the consumer know where to find the various endpoint(s) for the microservices, the latter being especially important where services use dynamic scaling models.

pra isso chegamos num novo tipo de ESB, os chamados de API Gateways

#### Organized around Business Capabilities (classificação com selinho Martin Fowler)

When looking to split a large application into parts, often management focuses on the technology layer, leading to UI teams, server-side logic teams, and database teams. When teams are separated along these lines, even simple changes can lead to a cross-team project taking time and budgetary approval. A smart team will optimise around this and plump for the lesser of two evils - just force the logic into whichever application they have access to. Logic everywhere in other words. This is an example of Conway's Law in action.

Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

-- Melvyn Conway, 1967

-> aligning microservices with DDD bounded contexts

Inclusive o ideal é começar dessa maneira, e só depois, caso realmente precise, evoluir esses contextos para microserviços (falo mais sobre isso logo abaixo). 

# Outras coiasas / Vantagens

No commitment to a unique technology

Greater flexibility of architecture

Escalabilidade

# Cuidados e [possíveis] desvantagens

##### Infrastructure Automation

Integração contínua, entrega contínua (teste e deploy), Docker,
(ambiente desenvolvimento)

##### Ferramentas de monitoramento e logs
Common mistakes microservices
    ESBs resolviam requisitos não funcionais como "Autenticação, monitoramento, logs centaliazdos, mediação"
-> Com microservices esses requisitos não funcionais muitas vezes são negligenciados.

multiplos bancos de dados e transações pode ser doloroso

-> Documentação dos serviços existentes (onde encontrar?)
-> Monitoramento. Os serviços estão distribuídos, ótimo! Mas qual a topologia de comunicação entre eles? Quantas requests por dia pro serviço X? Quantos erros no serviço Y?
-> Log de erros individuais e centralizados

##### Protocolo de comunicação
  SOAP pra comunicação interna
  REST é de fato o mais utilizado (isso por sí só poderia ser um post)

##### Versionamento API?
Fica dificil evoluir os serviços sem um versionamento de API. O versionamento de api permite uma maior liberdade de evolução dos serviços e diminui o acoplamento.

##### Sincrono vs Assincorno

Transacao e eventual consistency (With a monolith, you can update a bunch of things together in a single transaction.)

Não que no mundo monolítico isso seja uma maravilha. Já vi cenários em que TODAS as aplicações do cliente ficavam indisponíveis quando usuários de um sistema específico executavam um relatório que tinha uma consulta pesada e mal escrita na qual atolava o banco de dados. 

In the market today, RabbitMQ and Apache Kafka are both commonly used message bus technologies for asynchronous communication between microservices.

##### Service registry e service discovery
Each microservice handles registration within the central service registry. They typically register during the startup and periodically update the registry with current information. When the microservice gets terminated, it needs to be unregistered from the registry. The registry plays a critical role in orchestrating microservices.

Consul, etcd and Apache Zookeeper are examples of commonly used registries for microservices. 

-> Se nao tiver tem que fazer escalabildiade via servidor web ou através de um reverse proxy


### Você provavelmente não precisa
Amazon, Netflix, and eBay
Motivação principal é escalabidade

The additional complexity that comes from distributed systems requires an additional level of maturity and investment

Microservices Hell / service-disoriented architecture

# Monolítico não é ruim
Nem todo monolítico é ruim, apesar da conotação negativa no senso comum do termo "monolítico".

-> MonilithFirst
Martin Fowler diz [nesse post](https://martinfowler.com/bliki/MonolithFirst.html) que os exemplos de sucesso de microservices que ele notou começaram por monolíticos que evoluiram para microserviços. 

Any refactoring of functionality between services is much harder than it is in a monolith. But even experienced architects working in familiar domains have great difficulty getting boundaries right at the beginning. By building a monolith first, you can figure out what the right boundaries are, before a microservices design brushes a layer of treacle over them. It also gives you time to develop the MicroservicePrerequisites you need for finer-grained services.  


# Api Gateway (outro post?)

http://www.elemarjr.com/pt/2017/06/entendendo-microservicos/