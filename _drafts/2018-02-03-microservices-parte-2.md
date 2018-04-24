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

[take your pills]({{ site.url }}/images/2018-02-13-microservices-parte-2/blue-red-pills.jpg)

Toda decisão arquitetural possui tradeoffs e com microserviços não é diferente. Sair do paradigma de uma arquitetura SOA voltada a serviços "encorpados" ou até mesmo migrar de uma estrutura monolítica para microserviços é um caminho complexo e cheio de desafios. Ao mesmo tempo que aconselho cleintes e equipes a seguirem esse caminho em passos curtos, saliento que é exigido uma preparação não trivial que depende em muito da maturidade da arquitetura da empresa e também de suas equipes técnicas. 

Nesse post listo algumas das dificuldades mais comuns que vejo times enfrentarem ao optar pela abordagem de microserviços e também alguns pontos de atenção. Vamos lá.

# DevOps

Ter uma alta granuralidade de microserviços heterogêneos numa mesma arquitetura traz diversos desafios não funcionais, tais como logs, segurança, monitoramento, conversão de protocolos (REST/SOAP). Todos esses requisitos normalmente vinham juntos no pacotão do ESB na arquitetura SOA clássica, mas dado os pontos negativos da abordagem (detalhados nos posts anteriores) talvez você esteja querendo evitá-los. Entretanto, não ter esse tipo de ferramenta central faz com que tenhamos que nos preocupar com todo esse tooling de forma descentralizada. Isso vem com um custo, e ele não é barato.

##### Monitoramento e logs

Já pensou em como documentar seus microserviços? 
* Quantos existem? 
* Onde eles estão?
* Qual a topologia de comunicação entre eles? 

E os logs...
* Quantas requests por hora no serviço X?
* Quantos erros no serviço Y?
* Os logs estão centralizados num lugar só? 
* Eles são padronizados e fáceis de entender?

Resolver esses problemas em uma arquitetura monolítica normalmente é simples. Existem diversas ferramentas de APM que você acopla na sua aplicação e que num passe de mágica disponibilizam esse tipo de visão. Algumas delas: [AppDynamics](https://www.appdynamics.com/), [New Relic](https://newrelic.com/), [DynaTrace](https://www.dynatrace.com/), [Data Dog](https://www.datadoghq.com).

O problema é que muitas dessas ferramentas, aos menos as melhores, são pagas e possuem planos que não são nada atrativos para quem possui um  grande espectro de aplicações e serviços. Por esse e outros motivos muitas equipes utilizam libs descentralizadas em conjunto com algumas ferramentas para prover esse tipo de visão. Algumas dessas ferramentas: [Graphite](http://graphite.readthedocs.org/en/latest/), [Logstash](https://github.com/logstash/logstash-logback-encoder), [Kibana](https://www.elastic.co/guide/en/kibana/current/xpack-monitoring.html), [Grafana](https://grafana.com), [Prometheus](https://prometheus.io/). **USAR SÓ FERRAMENTAS OPEN SOURCE E SEPARAR FERRAMENTAS / LIBS**

##### Automação de infraestrutura

Suas ferramentas de integração e deploy contínuo devem ser preparadas para essa maior gama de aplicações/serviços. Não só preparadas para suportar mas também prover a adição de novas aplicações com facilidade. Se a configuração de sua infraestrutura automatizada é muito "manual", isso será um slowdown na criação de novos microserviços e a longo prazo a soma desse setup todo pode ser demasiado custoso.

Além da integração contínua, não se esqueça que o setup do ambiente de desenvolvimento também deve ser ágil. Porque configurar um monolito na máquina de cada desenvolvedor pode até não ser tão custoso. Mas configurar múltiplas aplicações na qual o time depende para desenvolver e testar uma funcionalidade não é nada divertido, muito menos barato. Containers em conjunto com ferramentas como o [Docker Swarm](https://github.com/docker/swarm) ou [Docker Compose](https://docs.docker.com/compose/) ajudam muito nesse quesito.

##### Segurança e autenticação

Um monolíto normalmente sabe se autenticar sozinho, ou seja, a autenticação está lá e é apenas ele quem a utiliza. Porém, se você começa a migrar para uma estrutura de microserviços é fortemente recomendado que você construa um serviço especialista nisso (normalmente autenticação + autorização) e que esse serviço seja bem resiliente, pois diversas outras aplicações vão depender dele. 

Outra dica válida aqui é: não construa isso do zero! Utilize algum framework de mercado que possua uma forte comunidade em volta, existem vários desse tipo que são open source e gratuítos como o [Identity Server](http://identityserver.io/), por exemplo. Não reinvente a roda.

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

##### As boundaries nem sempre estão claras
botar isso?

##### Dependência entre serviços
Quanto maior a dependência entre seus serviços mais a vantagem de "deploy independente" se esvai. 


##### [API Gateway](http://www.elemarjr.com/pt/2017/06/entendendo-microservicos/) (que podemos dizer que é uma versão renomeada e "lightweight" dos antigos ESBs)

### multiplos bancos de dados e transações pode ser doloroso

## Conclusão

Toda escolha de design e arquitetura possui seus tradeoffs e com microserviços não é diferente. São muitas as complexidades que você irá introduzir na sua arquitetura ao optar pelo approach de ter uma alta granularidade de serviços. É preciso avaliar os tradeoffs com cautela.