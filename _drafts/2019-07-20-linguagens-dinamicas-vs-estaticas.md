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


Num [post anterior](http://www.fabriciorissetto.com/blog/base-piramide-testes/) falei sobre os benefícios de se ter um compilador e citei algumas questões sobre linguagens estaticas e dinâmicas. Nesse post aqui pretendo aprofundar um pouco mais nessa comparação.

Mas antes de começar tal comparação é muito interessante


## Porque linguagens dinâmicas surgiram

Python is a dynamically typed language, it is also a strongly-typed language. You cannot concatenate a Duck and a float in Python. Javascript is a weakly-typed language that allows 3 + “quacks.”

Good type systems do much more than define records and can also remove boilerplate and defensive coding that even dynamic languages are prone to.

## Type inference
Type inference removes verbosity C# tem inference em alguns momentos (exemplo uso do var, ou de chamar um método genérico sem ter que dizer o tipo). Mas ainda assim não é tao avançado quando o de F# por exemplo que infere os parametros do método a partir do momento em que ele é chamado.

Generics bring flexibility to static languages. Generics chegou em java em 2004 e em 2005 pro C# (com uma implementação muito mais elegante, sem parameter hiding)

## DDD em linguagens dinâmicas
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

https://medium.com/@jessedickey/tdd-is-for-programmers-who-dont-program-46f431f06bbd
https://medium.com/@jessedickey/to-type-or-not-to-type-that-is-not-the-question-9d89f82349b1

Reconheço que minha carreira pode ter criado uma visão limitada baseado nas experiencias pelas quais passei. Não pode ser que eles são burros e eu inteligente. Não me acho alguem inteligente. Nunca fui o cisne dourado da turma. Sou esforçado e bom em detectar padrões. Como detectar padrões se eu não tive oportunidades de ver todos os contextos? Portanto, se você tem uma visão diferente, algo relevante que acha que eu deixei passar, por favor comente e me ajude a evoluir esse pensamento. Só não espere que eu receba seus argumentos pelo simples fato de aceitar. Me comprometo emf azer uma análise crítica e o mais imparcial possível do seu argumento.

Comentário Regis no [post do Robson](https://robsoncastilho.com.br/2019/11/16/linguagens-estaticamente-ou-dinamicamente-tipadas/) sobre dinamicas:

> Um problema com tipagem estática que me deparei apenas recentemente (talvez por estar acostumado com o fluxo de trabalho de tipagem dinâmica), é que às vezes ela dificulta a adoção de baby steps. Por exemplo, se eu altero uma classe para receber uma dependência a mais, eu preciso alterar todos os lugares que usam esta classe antes de rodar os testes. Mas eu gostaria de poder rodar apenas um testezinho para ver como fica e depois alterar o resto. Mas como o código não compila, isso não é possível.

