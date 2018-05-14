---
layout: post
title: "Funcional e OO na JVM: o melhor dos dois mundos com Kotlin"
modified:
categories: blog
excerpt: ""
tags: [Kotlin]
image:
  feature:
date: 2018-05-13T00:58:56-02:00
---

Faz um certo tempo que despertei o interesse por programação funcional, afinal com o Hype de linguagens funcionais quem ainda não despertou não é mesmo? Confesso que **XXXXXX** Mas uma coisa que eu ainda estava sentindo falta antes de conhecer Kotlin era a união dos dois mundos `FP` e `OOP` de maneira "harmônica".

**Haskell** é uma linguagem puramente funcional e é o que normalmente se indica para aprendizado do paradigma. Acho que faz todo sentido aprender um novo paradigma numa linguagem que não te dê espaço para fugir dele. Para programar em Haskell você é obrigado a pensar "funcionalmente". Isso é uma escolha da linguagem. Haskell se sai bem como uma linguagem puramente funcional e não tem um grande espaço no mercado corporativo justamente por não ser *"general purpose"*. **Smalltalk** teve um papel parecido na história. É uma linguagem puramente orientada à objetos e foi a grande responsável por popularizar o paradigma, que posteriormente foi incorporado por outras linguagens que até então eram basicamente imperativas.

No entando, mesmo essas novas linguagens se dizendo "multiparadigmas" elas normalmente favorecem a um dos lados.


<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2018-05-13-kotlin/fp-oop-slider.png">
</div>

C#, por exemplo, é uma linguagem que possui muitos aspectos de programação funcional mas que está mais voltada para a esquerda nesse slider. Já F# está muito mais para a direita, é uma linguagem claramente *functional-first*. Entretanto, ainda assim é possível escrever um código orientado à objetos conciso e legível em F#. Meu palpite para F# não ser tão popular quanto deveria se dá ao fato de C# ser uma boa linguagem. Já na stack JVM, acredito que um fator que influencia a popularização de linguagens alternativas seja a inconformação (justificada) ao Java. O problema é que até então essas linguagens se distanciavam em muito do paradigma comum OOP e da sintaxe das linguagens tradicionais. Scala não é uma linguagem com curva de aprendizado trivial. Clojure então não vamos nem falar...

E é aí que está o grande pulo de Kotlin. A sintaxe é muito familiar para programadores de linguagens tradicionais (Java, C# e até mesmo Ruby). O que torna sua curva de aprendizado muito pequena em relação a outras linguagens híbridas.

A imutabildiade é o default? Na verdade é você quem escolhe. Vamos pelo imutável, criando um Value Object:

```java
data class Address(val street: String,
                   val number: Int)
```

Agora uma Entidade, mutável:

```java
class Person(var name: String, var address: Address) {
    val id = UUID.randomUUID() //nativo do java

    fun sayName() {
        println("Hello my name is $name! And my id is $id")
    }
}
```
