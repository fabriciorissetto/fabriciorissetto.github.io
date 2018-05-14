---
layout: post
title: "Microservices (parte 2): Dificuldades e problemas"
modified:
categories: blog
excerpt: ""
tags: [SOA,Microservices]
image:
  feature:
date: 2018-05-10T00:58:56-02:00
---

![take your pills]({{ site.url }}/images/2018-02-13-microservices-parte-2/blue-red-pills.jpg)

Toda decisão arquitetural possui tradeoffs e com microserviços não é diferente. Sair do paradigma de uma arquitetura SOA voltada a serviços "encorpados" ou até mesmo migrar de uma estrutura monolítica para microserviços é um caminho complexo e cheio de desafios. Ao mesmo tempo que aconselho desenvolvedores a seguirem esse caminho em passos curtos, saliento que é exigido uma preparação não trivial que implica em investimento da empresa na maturidade de sua arquitetura e também de suas equipes técnicas.

Nesse post listo algumas das dificuldades mais comuns que vejo times enfrentarem ao optar pela abordagem de microserviços e também alguns pontos de atenção. Vamos lá.

# DevOps

Ter uma alta granularidade de microserviços heterogêneos numa mesma arquitetura traz diversos desafios não funcionais, tais como logs, segurança, monitoramento, conversão de protocolos (REST/SOAP). Todos esses requisitos normalmente vinham juntos no pacotão do ESB na arquitetura SOA clássica, mas dado os pontos negativos da abordagem (detalhados nos posts anteriores) talvez você esteja querendo evitá-los. Entretanto, não ter esse tipo de ferramenta central faz com que tenhamos que nos preocupar com todo esse tooling de forma descentralizada. Isso vem com um custo, e ele não é barato.

#### Monitoramento e logs

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

O problema é que muitas dessas ferramentas, aos menos as melhores, são pagas e possuem planos que não são nada atrativos para quem possui um  grande espectro de aplicações e serviços (acredito que isso irá mudar). Por esse e outros motivos muitas equipes utilizam libs descentralizadas em conjunto com algumas ferramentas para prover esse tipo de visão. Algumas dessas ferramentas: [Graphite](http://graphite.readthedocs.org/en/latest/), [Logstash](https://github.com/logstash/logstash-logback-encoder), [Kibana](https://www.elastic.co/guide/en/kibana/current/xpack-monitoring.html), [Grafana](https://grafana.com), [Prometheus](https://prometheus.io/), [Zipkin](https://zipkin.io/).

Quando utilizando essas ferramentas temos o desafio de como nos integrar com elas. Seria diretamente no código? Algumas delas requerem isso e o que muitas empresas fazem é criar uma "lib interna" ou um "template de arquitetura de microserviço" no qual já tenha esse tipo de coisa configurada. O problema desse approach é que "mudar" coisas nessa lib ou nesse template requer mudanças em microserviços que já existem. Além do mais, ao criar isso normalmente as empresas acabam se amarrando em uma stack de tecnologia "X", fazendo com que (ou ao menos encorajando) todos os novos microserviços sigam aquele template que já foi criado.

Um approach que tende a facilitar nossa vida nesse quesito é o conceito de **Service Mash** no qual o [Istio](https://istio.io/) implementa. O Istio é um projeto recente criado pela Google em parceria com a IBM e a Lyft e que parece ser muito promissor. Com ele não é necessário nenhuma modificação no código da sua aplicação e nem mesmo no container no qual ela está hospedado. Além do Istio existem outras iniciativas parecidas voltadas a container, como o [Linkerd](https://linkerd.io/).

#### Automação de infraestrutura

Suas ferramentas de integração e deploy contínuo devem ser preparadas para essa maior gama de aplicações/serviços. Não só preparadas para suportar mas também prover a adição de novas aplicações com facilidade. Se a configuração de sua infraestrutura automatizada é muito "manual", isso será um slowdown na criação de novos microserviços e a longo prazo a soma desse setup todo pode ser demasiado custoso.

Além da integração contínua, não se esqueça que o setup do ambiente de desenvolvimento também deve ser ágil. Porque configurar um monolito na máquina de cada desenvolvedor pode até não ser tão custoso. Mas configurar múltiplas aplicações na qual o time depende para desenvolver e testar uma funcionalidade não é nada divertido, muito menos barato. Containers em conjunto com ferramentas como o [Docker Swarm](https://github.com/docker/swarm) ou [Docker Compose](https://docs.docker.com/compose/) ajudam muito nesse quesito.

#### Segurança e autenticação

Um monolito normalmente sabe se autenticar sozinho, ou seja, a autenticação está lá, foi construída nele e é apenas ele quem a utiliza. Porém, se você começa a migrar para uma estrutura de microserviços é fortemente recomendado que você construa um serviço especialista nisso (normalmente autenticação + autorização) e que esse serviço seja bem resiliente, pois diversas outras aplicações vão depender dele.

Outra dica válida aqui é: não construa isso do zero! Utilize algum framework de mercado que possua uma forte comunidade em volta, existem vários desse tipo que são open source e gratuítos como o [Identity Server](http://identityserver.io/), por exemplo.  Não reinvente a roda, principalmente quando envolve segurança.

#### Service registry e service discovery

Service Registry é uma espécie de database que contém informações sobre a localidade dos serviços na rede, onde toda vez que um serviço sobe, ele ativamente (ou de maneira dinâmcia) informa o servicy registry. Com isso, quando um serviço `A` quer chamar um serviço `B` ele não precisa saber diretamente seu IP ou DNS exato. Ele pode fazer o "discovery" do endereço de `B` através de uma query no service registry.

Com isso você adquire diversos benefícios: O endereço físico (e até mesmo DNS) do seu serviço pode variar sem afetar o código de quem depende dele. Você também ganha um healthcheck dos seus serviços, pois uma vez desconectados do service registry você saberá e terá o log disso, além de que os serviços dependentes também podem ser informados. Outro benefício é a facilidade de balanceamento de carga, já que o service registry pode assumir essa role, utilizando uma estratégia de, por exemplo, Round Robin pra escolher qual instância do serviço `B` (contando que existam várias) ele irá disponibilizar para o solicitante.

Alguns exemplos desse tipo de ferramenta são: [Consul](https://consul.io/), [etcd](https://github.com/coreos/etcd), [Zookeeper](http://zookeeper.apache.org/) e [Eureka](https://github.com/Netflix/eureka). Além disso, há outras formas de se resolver esse problema com: load palancers, reverse proxy e com soluções nativas de nuvem. Esse [link](https://docs.aws.amazon.com/aws-technical-content/latest/microservices-on-aws/service-discovery.html) da AWS mostra diversas abordagens que você pode seguir utilizando serviços da AWS e no final ainda indica algumas soluções third party como os citados acima (Consul, Eureka, etc).

# Alguns outros pontos de atenção

## Síncrono vs Assíncrono

Tome MUITO cuidado com chamadas síncronas entre serviços, elas não escalam. Prefira a abordagem [reativa](https://www.reactivemanifesto.org/pt-BR). Quanto mais assíncrona e reativa sua arquitetura mais dinâmica e escalável ela é. Eu sou um grande fã da abordagem "Event Driven", que aplicada a microserviços significa a comunicação assíncrona entre eles utilizando eventos. Nessa abordagem, além da facilidade de escalabilidade, é menos doloroso e arriscado modelar incertezas do domínio como, dependência entre [bounded contexts](http://www.fabriciorissetto.com/blog/ddd-bounded-context/) e em relação a ordem de processo/fluxo desses contextos. Ferramentas como RabbitMQ, Apache Kafka ou serviços gerenciados como Amazon SQS/SNS são tecnologias muito utilizadas para realizar essa comunicação assíncrona entre serviços utilizando eventos como forma de mensagem.

## Consistência Eventual

Ao utilizar uma abordagem de comunicação assíncrona nos serviços você passa a ter que tratar do que é chamado de "consistência eventual". Em um monolito você pode fazer uma série de alterações e encapsular tudo dentro de uma transaction para garantir a consistência, afinal, tudo está no mesmo banco de dados e no mesmo processo. Com microserviços isso geralmente não é possível, apesar de que existir padrões como o [WS Atomic Transaction](https://en.wikipedia.org/wiki/WS-Atomic_Transaction) que possibilita criar transações ACID em microserviços, implementar isso não é trivial. Sendo assim, temos que tratar a consistência de outras maneiras, como [compensating transactions](https://docs.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction). Esse tipo de abordagem impacta não só a maneira como você escreve suas aplicações mas também a iteração dela com os usuários.

## Versionamento de API

Fica dificil evoluir os serviços sem um versionamento de API. Um bom versionamento de API permite uma maior liberdade de evolução dos serviços e diminui as dores do acoplamento. Apesar de eu não ser um grande fã, uma abordagem que vem ganhando espaço para desacoplar APIs de seus consumidores é o uso do padrão HATEOAS implementado por ferramentas como [HAL](http://stateless.co/hal_specification.html), [Siren](https://github.com/kevinswiber/siren) e [JSON API](http://jsonapi.org/). Caso sua API tenha diversos consumidores, que se interessam em diferentes dados da sua aplicação e que se beneficiariam em poder compor suas queries, o [GrapQL](https://graphql.org/) pode ser uma boa opção também.

# Monolito não é ruim

Por último, mas não menos importante, a dica que quero dar é: monolítico (ou monolito) nem sempre é ruim, apesar da conotação negativa no senso comum do termo. O Martin Fowler tem um ótimo post falando da estratégia ["Monilith First"](https://martinfowler.com/bliki/MonolithFirst.html), no qual ele cita que os exemplos de sucesso de microservices na experiência dele começaram por monolitos antes de evoluirem para microserviços.

> Any refactoring of functionality between services is much harder than it is in a monolith. But even experienced architects working in familiar domains have great difficulty getting boundaries right at the beginning. By building a monolith first, you can figure out what the right boundaries are, before a microservices design brushes a layer of treacle over them. It also gives you time to develop the Microservice Prerequisites you need for finer-grained services.

Isso está fortemente atrelado ao conceito de [bounded contexts](www.fabriciorissetto.com/blog/ddd-bounded-context/) aplicado a microserviços. Você sabe exatamente quais são as boundaries e responsabilidades do microservice que você está criando? Não ter isso bem definido é um risco. E esse risco é menor quando você está trabalhando dentro de um monolito (quando construído com design desacoplado) pois mover uma funcionalidade de um contexto para outro dentro de uma mesma aplicação é mais simples do que mover conteúdo de um serviço `A` para um `B`. Em serviços isso impacta até mesmo sua API, que pode já estar sendo consumida por alguém. E se o microserviço `A` foi escrito numa linguagem diferente do `B`? Isso torna o refactoring ainda mais custoso, sem falar da migração de database... Enfim, é sensato começar por um monolito caso haja dúvidas quanto as boundaries dos seus componentes. Iniciar nessa abordagem te permite ir migrando as partes que fizerem sentido para microserviços conforme a necessidade, isso pode ser feito com planejamento e menos dor. O caso contrário geralmente é mais custoso.

## Conclusão

Toda escolha de design e arquitetura possui seus tradeoffs e com microserviços não é diferente. São muitas as complexidades que você irá introduzir na sua arquitetura ao optar pelo approach de ter uma alta granularidade de serviços. É preciso avaliar os tradeoffs com cautela.
