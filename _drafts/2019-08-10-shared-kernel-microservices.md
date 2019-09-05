---
layout: post
title: "Shared Kernel Microservices"
modified:
categories: blog
excerpt: ""
tags: [Linguagens de Programação]
image:
  feature:
date: 2019-07-11T00:58:56-02:00
---

Seria muito bom se tecnologia fosse algo preto no branco, mas não é. Cada um de nós traz bagagens, experiências e cicatrizes diferentes por isso é comum que a gente discorde de coisas que as vezes parecem ser simples.

No fórum de terça eu fiquei igualmente frustrado ao mameli por a gente estar discutindo isso mais uma vez.

E frustrado pela nossa discordância ser tão grande.

Naquela conversa não estávamos preparados pra discutir aquilo, fomos pegos de surpresa pela indefinição de algo que tínhamos pensado já estar definido.

Há um tempo atrás discutimos sobre a replicação dos dados e sobre isso. Aparentenmente a replicação dos dados virou decisão e consenso. Já a questão de quem fica com cada dado (exemplo de person), não.

O objetivo aqui é ser mais objetivo, já trouxe exemplos concretos e estão distacados numa doc. Já coloquei prós e contra de cada abordagem e pedi ajuda de vcs pra complementar.

Agora quero revisar isso e a gente de forma objetiva tomar a decisão. Quando digo de forma objetiva digo: não é o que eu acho, o que u sinto. E sim olhar pra esses prós e contras, avaliar seus pesos e riscos e tomar uma decisão.

Canonical Data Models & Microservices
https://www2.deloitte.com/content/dam/Deloitte/za/Documents/ZA_Deloitte_Digita_Canonical_Schemas.pdf

Why You Should Avoid a Canonical Data Model
https://www.innoq.com/en/blog/thoughts-on-a-canonical-data-model/

Is there still room for Canonical Data Model in a Microservices architecture?
https://nl.devoteam.com/en/blog-post/is-there-still-room-for-canonical-data-model-in-a-microservices-architecture/

Why is a Canonical Data Model an Anti Pattern
https://medium.com/@teivah/why-is-a-canonical-data-model-an-anti-pattern-441b5c4cbff8

DDD - Patterns for Model Sharing
https://www.voxxed.com/2016/12/harnessing-domain-driven-design-in-distributed-systems-development-part-iii/

Enterprise Integration Patterns
https://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html

**Alta coesão, baixo acoplamento**

Com replicação de dados eu diminuo o acoplamento
Eu ficando dono de determinados dados eu diminuo o acoplamento

Eu usando dados com nomes que nao fazem sentido pra mim diminui a coesão
