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

Alguns dizem até que é um paradigma evolucionário do SOA, que é o *"SOA feito da maneira certa"*. O que eu também não concordo 100% e provavelmente você vai entender o porquê até o final do post.

## Afinal, o que é?

O termo ganhou popularidade depois desse [post](http://martinfowler.com/articles/microservices.html) feito pelo Martin Fowler e James Lewis onde eles descrevem as 9 principais características do padrão de microserviços. Mas como aquele post é meio chato, e você está aqui ao invés de estar lá, me permito descrever pra você uma definição resumida utilizando minhas palavras, lá vai:

> É um monte de servicinho se chamando pra todo lado, em que cada serviço tem uma responsabilidade única e normalmente focada em uma parte específica do negócio (ou até mesmo em um aspecto não funcional) que juntos compõem um objetivo maior do "todo". É separar aquele monolítico de Ecommerce em serviços menores como "Carrinho de compras", "Pagamento", "Catalogo de Produtos", "Autenticação" (microserviço não funcional), etc. 

Bonito de falar, difícil de fazer...

Pra mim, não tem maneira melhor de entender um padrão do que descrevendo as vantagens e desvantagens de se usá-lo. No final das contas a decisão de utilizar qualquer padrão passa por essa balança. Então vamos lá.

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-28-microservices/pros-cons.png">
</div>


## Algumas vantagens

#### Smart endpoints and dumb pipes

Os serviços conhecem e consomem uns aos outros. Por isso dizemos que os endpoints são "inteligentes" pois os serviços se comunicam entre si diretamente, sem um "ESB tomador de decisão" no meio. Com ESBs os desenvolvedores são encorajados a botar cada vez mais lógica e inteligência dentro dele. Ele é a única e central entidade nas quais as aplicações e serviços enxergam.

O resultado de termos serviços que se comunicam diretamente é que eles sabem da existência dos demais e por isso eles fazem parte da sau regra de negócio, bem como a comunicação entre eles. 

#### Organized around Business Capabilities 

Esse é um dos tópicos que eu mais gosto. Fazendo um gancho com DDD isso seria o conceito de [bounded contexts](http://www.fabriciorissetto.com/blog/ddd-bounded-context/) implementado num sentido mais amplo, no de um serviço. Isso mesmo, cada bounded context poderia ser implementado como sendo um microserviço. Normalmente a motivação de separar um bounded context num microserviço seria isolar ele do restante, por diversos motivos como: 

##### Independência de linguagem 
Poder desenvolver os microserviços em diferentes linguagens. Por exemplo, na [Creditas](www.creditas.com.br) existe um microserviço que utiliza uma série de conceitos de machine learning para elencar a prioridade de atendimento de um cliente (clientes com maiores chances de assinarem o contrato são atendidos primeiro). Apesar da stack da empresa estar em grande parte construída em Ruby, para esse microservice em específico foi utilizado Python devido ao fato de que haviam diversas libs prontas com implementação dos conceitos que eram necessários, além de libs de integração com ferramentas que utilizamos na área de data science. E como reinventar a roda, apesar de ser divertido, tem um custo, escolhemos utilizar a linguagem que mais nos favorecia naquela implementação.

##### Flexibilidade de arquitetura
Seguir modelos e possuir padrões dentro de sua empresa é legal. Mas ditar que todas aplicações vão possuir a mesma arquitetura não é. Por exmeplo, existem aplicações que vão ser favorecidas pelo uso do padrão de DDD por possuírem complexidade de negócio. Mas nem todas aplicações possuem complexidade de negócio e nem todas as áreas do seu negócio são complexas. Uma aplicação ou área de negócio que é mais voltada a dados, ou seja, "CRUD intensive", poderia ser desenvolvida num estilo arquitetural como o [Smart UI](http://www.markewer.com/2015/10/21/smartui-architecture-pattern/), um padrão muito simples e fácil de se manter em casos como esse.

##### Independência de protocolo (REST, SOAP, etc)
Seu serviço vai se expor para outros serviços/aplicações de alguma maneira e porvavelmente você vai utilizar algum padrão/protocolo para isso. Se ele vai ser consumido por um código javascript em uma página web talvez seja interessante você utilizar REST e retornar um JSON para o frontend. Se seu serviço vai ser consumido por uma aplicação escrita em Java ou C#, que são linguagens estaticamente tipadas, talvez você queira utilizar o protocolo SOAP para aproveitar o boilerplate de classes criadas automaticamente ao mapear o WSDL no seu projeto. 

##### Independência de deploy
A independência de deploy pode ter diversas vantagens quanto a simplicidade. As vezes a simplicidade de poder republicar um serviço sem ter que, em casos de uma aplicação monolítica, derrubar a aplicação core da empresa por estar tudo na mesma base de código (argumento meio *"meeh"*, já que isso pode ser feito utilizando algum método de deploy [mais inteligente](https://stackoverflow.com/questions/23746038/canary-release-strategy-vs-blue-green)). E também simplicidade nos processos de automação de deploy na integração contínua, já que um microserviço normalmente tem menos complexidade e dependências que um grande monolítico.

##### Escalabilidade do serviço
Esse aqui não precisa explicar. É a primeira coisa que se acha no google quando se pesquisa por microserviços. Se uma professora da terceira série do fundamental aplicasse em uma prova a pergunta "qual é o principal benefício de microserviços?" *e algum aluno respondessse* provavelmente a resposta seria "escalabilidade do serviço". E de fato o padrão se popularizou pelo uso em empresas com alta demanda de escalabilidade como Netflix, Amazon, Ebay. Entretanto essa é apenas uma das vantagens, que para empresas como as citadas anteriormente talvez realmente seja a principal, mas que para outras talvez seja completamente irrelevante. Digo isso pois vejo muitas pessoas utilizando essa caracteríscica como o [litmus test](https://en.wikipedia.org/wiki/Litmus_test_(politics)) para implementação de microserviços, o que é um equívoco. Talvez sua empresa não precise de escalabilidade mas ainda assim precise de microserviços.

##### Escalabilidade de equipe
Complemento ao tópico anterior. Escalabilidade não é apenas instâncias da sua aplicação subindo na AWS. Escalabildiadde de equipe também é importante.

Isolamento dos times de desenvolvimento 

É a lei de Conway em ação!

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
-- Melvyn Conway, 1967

# Pontos de atenção / Cuidados necessários

##### Versionamento API?
Fica dificil evoluir os serviços sem um versionamento de API. O versionamento de api permite uma maior liberdade de evolução dos serviços e diminui o acoplamento.

##### Escolha do protocolo de comunicação
  SOAP pra comunicação interna
  REST é de fato o mais utilizado (isso por sí só poderia ser um post)

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

# Monolítico não é ruim [Post]
Nem todo monolítico é ruim, apesar da conotação negativa no senso comum do termo "monolítico".

-> MonilithFirst
Martin Fowler diz [nesse post](https://martinfowler.com/bliki/MonolithFirst.html) que os exemplos de sucesso de microservices que ele notou começaram por monolíticos que evoluiram para microserviços. 

Any refactoring of functionality between services is much harder than it is in a monolith. But even experienced architects working in familiar domains have great difficulty getting boundaries right at the beginning. By building a monolith first, you can figure out what the right boundaries are, before a microservices design brushes a layer of treacle over them. It also gives you time to develop the MicroservicePrerequisites you need for finer-grained services.  

**Atenção**: Não recomendo iniciar um contexto dessa maneira (em microserviço) a não ser que se tenha extrema convicção dos motivos citados acima. Acho muito mais sensato começar no monolítico (com um design bom e desacoplado) e ir migrando para microserviços algumas partes conforme a necessidade. Fazer o contrário é muito mais difícil. Além do mais, enquanto no monolítico, o refactoring se torna muito menos doloroso do que quando envolve a comunicação entre serviços.

# Algumas desvantagens

### Mas nem tudo são flores...

Ter uma alta granuralidade de serviços na sua estrutura traz diversos desafios não funcionais como: logs, segurança, monitoramento, conversão de protocolos (caso os serviços utilizem diferentes tipos), etc. Todos esses requisitos normalmente vem junto no pacote do ESB. Não ter um ESB faz com que tenhamos que nos preocupar com todo esse tooling de forma descentralizada.

Alguns desses desafios podem ser solucionados através de um API Gateway (que é uma espécie de ESB, só que não) no qual pretendo falar num próximo post.

##### Infrastructure Automation

Integração contínua, entrega contínua (teste e deploy), Docker,
(ambiente desenvolvimento)

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

