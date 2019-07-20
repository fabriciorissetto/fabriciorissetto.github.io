---
layout: post
title: "Linguagens dinâmicas vs estáticas"
modified:
categories: blog
excerpt: ""
tags: [Linguagens de Programação]
image:
  feature:
date: 2019-07-11T00:58:56-02:00
---

## Porque linguagens dinâmicas surgiram

A moivação desse post partiu dessa [resposta no stackoverflow](https://stackoverflow.com/a/47558186/890890).

Deixei minha resposta lá e aqui aprofundo com um pouco mais de detalhes (e em português).

## Checkagem de tipos

Não é a toa que o suporte para checagem estática de tipos foi adicionado no [Python](https://docs.python.org/3/library/typing.html) e será também adicionado em [Ruby](https://codesmithdev.com/ruby-3-will-introduce-types/). É muito útil ter a possibilidade de poder informar para uma ferramenta, seja ela um compilador, uma IDE ou plugin qualquer, sobre o tipo de uma determinada variável, parâmetro ou retorno de método a fim de GARANTIR que os desenvolvedores (inclusive eu) não irão passar tipos errados (ou nulos) e ocasionar erros de [undefined local variable](https://stackoverflow.com/questions/9671259/ruby-local-variable-is-undefined), [undefined method](https://stackoverflow.com/questions/21072355/undefined-method-nomethoderror-ruby).

## IDE’s
Ainda sobre dinamismo...

## Unit testing
Esse é o porquê do editor VIM ser tão popular entre os desenvolvedores Ruby.

# Frameworks
dependency injection
object mappers
ORMs

Isso tudo acima se resume a refactorings. Se você está escrevendo código ou testando algo que algum framework ou o próprio compilador poderia fazer por você, em troca de nenhum benefício extra, você está apenas ficando com os drawbacks da ferramenta. Considere trocar de ferramenta.

“[The best code is no code at all](https://blog.codinghorror.com/the-best-code-is-no-code-at-all/)” - Jeff Atwood

Citar dinamic proxy em linguagens estáticas

# Monads sem tipos?

# Vantagens dinâmicas

"interface implícita" / "Polimorfismo sem cerimônias" com ducktyping as vezes é util pra nao ter que pensar em classes
https://youtu.be/XwjvEMLdjco?list=PLoxsuhBe-jd8MRbv_NYq_Bdx774e0Q3Xd&t=373

# Open Classes?
extension functions (praticamente toda linguagem estaticamente tipada, C#, F#, Kotlin)

method_missing (dynamic object em C#)



