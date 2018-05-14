---
layout: post
title: "Microservices (parte 1): Partes boas"
modified:
categories: blog
excerpt: ""
tags: [SOA,Microservices]
image:
  feature:
date: 2017-07-28T00:58:56-02:00
---

*Esse post é continuação de uma série sobre arquitetura SOA. Aqui você pode conferior o anterior, no qual falei de [ESBs](www.fabriciorissetto.com/blog/ESBs/) e contextualizei o cenário SOA antes da hype de microservices.*

* **Edit:** Como o post ficou muito grande dividi ele em dois. Nessa primeira parte vou focar nos benefícios da abordagem e na segunda focarei nas dificuldades.

## Mais hype que Game of Thrones...

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-28-microservices/one-does-not-microservice.jpg">
</div>

Antes de tudo, quero começar esclarecendo uma coisa: **O padrão de microservices não é um substituto de SOA**. Pra falar a verdade a afirmação nem faz tanto sentido. Vejo microservices como um subset de SOA, uma das N maneiras de se fazer SOA.

## Afinal, o que é?

O termo ganhou popularidade depois desse [post](http://martinfowler.com/articles/microservices.html) feito pelo Martin Fowler e James Lewis onde eles descrevem as 9 principais características do padrão de microserviços. Mas como aquele post é meio chato, e você está aqui ao invés de estar lá, me permito descrever pra você uma definição resumida utilizando minhas palavras, lá vai:

> É um monte de serviço se comunicando pra todo lado, em que cada serviço tem uma responsabilidade única e normalmente focada em uma parte específica do negócio (ou até mesmo em um aspecto não funcional) que juntos compõem um objetivo maior do "todo". É separar aquele monolítico de Ecommerce em serviços menores como "Carrinho de compras", "Pagamento", "Catalogo de Produtos", "Autenticação" (microserviço não funcional), etc.

Bonito de falar, difícil de fazer...

Pra mim, não tem maneira melhor de entender um padrão do que descrevendo as vantagens e desvantagens de se usá-lo. No final das contas a decisão de utilizar qualquer padrão passa por essa balança. Então vamos lá.

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-28-microservices/pros-cons.png">
</div>


## Algumas vantagens

#### Smart endpoints, dumb pipes

Os serviços conhecem e consomem uns aos outros. Por isso dizemos que os endpoints são "inteligentes" pois os serviços se comunicam entre si diretamente, sem um "ESB tomador de decisão" no meio. Com ESBs os desenvolvedores são encorajados a botar cada vez mais lógica e inteligência dentro dele. Ele é a única e central entidade nas quais as aplicações e serviços enxergam.

O resultado de termos serviços que se comunicam diretamente é que eles sabem da existência dos demais e por isso eles fazem parte da sau regra de negócio, bem como a comunicação entre eles.

#### Organized around Business Capabilities

Esse é um dos tópicos que eu mais gosto. Fazendo um gancho com DDD isso seria o conceito de [bounded contexts](http://www.fabriciorissetto.com/blog/ddd-bounded-context/) implementado num sentido mais amplo, no de um serviço. Isso mesmo, cada bounded context poderia ser implementado como sendo um microserviço. Normalmente a motivação de separar um bounded context num microserviço seria isolar ele do restante, por diversos motivos como:

##### Independência de linguagem
Poder desenvolver os microserviços em diferentes linguagens. Por exemplo, na [Creditas](www.creditas.com.br) existe um microserviço que utiliza uma série de conceitos de machine learning para elencar a prioridade de atendimento de um cliente (clientes com maiores chances de assinarem o contrato são atendidos primeiro). Apesar da stack da empresa estar em grande parte construída em Ruby, para esse microservice em específico foi utilizado Python devido ao fato de que haviam diversas libs prontas com implementação dos conceitos que eram necessários, além de libs de integração com ferramentas que utilizamos na área de data science. E como reinventar a roda, apesar de ser divertido, tem um custo, escolhemos utilizar a linguagem que mais nos favorecia naquela implementação.

##### Flexibilidade de arquitetura
Seguir modelos e possuir padrões dentro de sua empresa é legal. Mas ditar que todas aplicações vão possuir o mesmo padrão de arquitetura não é. Por exmeplo, existem aplicações que vão ser favorecidas pelo uso do padrão de DDD por possuírem complexidade de negócio. Mas nem todas aplicações possuem complexidade de negócio e nem todas as áreas do seu negócio são complexas. Uma aplicação ou área de negócio que é mais voltada a dados, ou seja, "CRUD intensive", poderia ser desenvolvida num estilo arquitetural como o [Smart UI](http://www.markewer.com/2015/10/21/smartui-architecture-pattern/), um padrão muito simples e fácil de se manter para casos como esse.

##### Independência de protocolo (REST, SOAP, etc)
Seu serviço vai se expor para outros serviços/aplicações de alguma maneira e porvavelmente você vai utilizar algum padrão/protocolo para isso. Se ele vai ser consumido por um código javascript em uma página web talvez seja interessante você utilizar REST e retornar um JSON para o frontend. Se seu serviço vai ser consumido por uma aplicação escrita em Java ou C#, que são linguagens estaticamente tipadas, talvez você queira utilizar o protocolo SOAP para aproveitar o boilerplate de classes criadas automaticamente ao mapear o WSDL no seu projeto.

##### Independência de deploy
A independência de deploy pode ter diversas vantagens quanto a simplicidade. Como a simplicidade de poder republicar um serviço de forma isolada sem ter que derrubar uma aplicação core da empresa. Sim, isso pode ser feito utilizando algum método de deploy [mais inteligente](https://stackoverflow.com/questions/23746038/canary-release-strategy-vs-blue-green), porém dependendo da arquitetura da sua estrutura de microserviços (reativa por exemplo) isso nem é necessário. E também simplicidade nos processos de automação de deploy na integração contínua, já que um microserviço normalmente tem menos complexidade de configuração que um monolítico que atende a várias necessidades.

##### Escalabilidade do serviço
Esse aqui talvez nem precise explicar. É a primeira coisa que se acha no google quando se pesquisa por microserviços. Se uma professora da terceira série do fundamental aplicasse em uma prova a pergunta "qual é o principal benefício de microserviços?" *e algum aluno respondessse* provavelmente a resposta seria "escalabilidade do serviço". E de fato o padrão se popularizou pelo uso em empresas com alta demanda de escalabilidade como Netflix, Amazon, Ebay. Entretanto essa é apenas uma das vantagens, que para empresas como as citadas anteriormente talvez realmente seja a principal, mas que para outras pode ser completamente irrelevante. Digo isso pois vejo muitas pessoas utilizando essa caracteríscica como o [litmus test](http://www.dictionary.com/browse/litmus-test) para implementação de microserviços, o que é um equívoco. Talvez sua empresa não precise de escalabilidade mas ainda assim precise de microserviços. Veja o motivo abaixo.

##### Escalabilidade de equipe
Complemento ao tópico anterior. Escalabilidade não se resume apenas a instâncias de sua aplicação subindo na AWS ou Azure. Escalabildiadde de **equipe** também é importante.

Isso daqui é a lei de Conway em ação!

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
-- Melvyn Conway, 1967

Times completamente diferentes trabalhando na mesma base de código (mesmo repositório) podem enfrentar maiores dificuldades de trabalho conforme suas equipes crescem. Dificuldades de source control, como número excessivo de merges e resolução de conflitos, podem tomar um tempo precioso de desenvolvimento e aumentar o lead time do processo de code review. E não podemos esquecer também de alguns dos pontos que foram citados anteriormente como a dificuldade de deploy. Tudo isso pode afetar a **escalabilidade da equipe** e ao mesmo tempo não afetar em nada a **escalabilidade aplicação**.

#### Mas nem tudo são flores

E por isso separei um post só pros "downsides" de microserviços. Muitos deles tive o prazer de sentir na pele e confesso que não foi uma experiência muito divertida. Confira o post [aqui](/blog/microservices-parte-2).
