---
layout: post
title: "Bizarrices de números reais na JVM"
modified:
categories: blog
excerpt: ""
tags: [Kotlin]
image:
  feature:
date: 2018-09-17T00:58:56-02:00
---

Nesse rápido post não pretendo entrar em muitos detalhes técnicos por trás das características dos valores de ponto flutuante, pois é bem simples de achar em respostas como [essa](https://stackoverflow.com/a/1661313/890890) no StackOverflow que dão um ótimo "deep dive" na explicação científica por trás de pontos flutuantes na representação binária em geral.

Portanto vou resumidamente compartilhar alguns aprendizados que tive com meu recente contato com a JVM utilizando Kotlin, focando em exemplos de código.


### E lá vamos nós

Fazer operações utilizando floats e doubles na JVM resulta em valores não arredondados como no seguinte exemplo:

```java
0.1 + 0.2
//0.30000000000000004
```

Então é muito comum achar em [respostas](https://stackoverflow.com/a/3413493/890890) do StackOverflow a recomendação de utilizar o `BigDecimal`
para essas operações:

```java
BigDecimal(0.1) + BigDecimal(0.2)
//0.3000000000000000166533453693773481063544750213623046875
```

Ué!? Mas deu uma coisa zuada também! Claro, você ainda não tinha descoberto a segunda dica da comunidade, a de construir o BigDecimal utilizando o construtor que recebe uma `String`:

```java
BigDecimal("0.1") + BigDecimal("0.2")
//0.3
```

***Aeeee! Finalmente. Agora sabemos fazer cálculos com números reais na JVM! Bora fazer uns testes de unidade:***

```java
BigDecimal("0") shouldEqual BigDecimal("0.0")
//false
```

Opa, falso!? Vamos tirar as aspas de novo e vamos ver o que dá...

```java
BigDecimal(0) == BigDecimal(0.0)
// true
```

Agora foi. Eita...

## Precisão interna do BigDecimal

Isso acontece pois o BigDecimal possui internamente uma informação de **precisão**. BigDecimals inciados com o construtor Double contendo os valores `0` e `0.0` possuem ambos unidade **0** e escala **0**. Já BigDecimal inciados com o cosntrutor String contendo os valores `"0"` e `"0.0"` possuem a unidade **0** mas o segundo possui escala **1**! Por isso a igualdade resulta em falso.

Se você utilizar, ao invés do `equals()`, o `compareTo()`, a comparação ignora a diferença de precisão.

```java
BigDecimal("0").compareTo(BigDecimal("0.0"))
//0
```

## A importância da precisão para as operações

O cálculo `1 / 3` resulta em uma [dizima periodica](https://pt.wikipedia.org/wiki/D%C3%ADzima_peri%C3%B3dica). Com double o resultado é limitado pela sua precisão pré-definida:

```java
1.0 / 3.0
//0.3333333333333333
```

Já com BigDecimal, se você não definir a precisão que você deseja, a expressão `1 / 3` dá erro:


```java
BigDecimal(1.0).divide(BigDecimal(3.0))
//Error: ArithmeticException
```

Portanto defina o tamanho da escala e o tipo de arredondamento:

```java
BigDecimal(1.0).divide(BigDecimal(3.0), 80, RoundingMode.HALF_EVEN))
//0.33333333333333333333333333333333333333333333333333333333333333333333333333333333
```

Note que aqui utilizei o método `divide` quando no exemplo anterior utilizei o **operador** `+` para a soma (ao invés do método `add()`). No Java o BigDecimal não possui overload de operador pois lá não são permitidos overloads de tipos complexos (apenas primitivos). Portanto o `/` é uma extensão de Kotlin sobre o BigDecimal, que por baixo dos panos utiliza uma o método `divide` porém com alguns parâmetros padrões que podem causar efeitos inesperados. Exemplo:

```java
BigDecimal("1") / BigDecimal("3.0") //Utiliza a escala 0 inferida através do primeiro BigDecimal
//0
BigDecimal("1.0") / BigDecimal("3") //Utiliza a escala 1 inferida através do primeiro BigDecimal
//0.3
```

Portanto eu aconselho a sempre utilizar os **métodos** originais do BigDecimal ao invés dos **operadores** criados no Kotlin, para manter um padrão no código e fazer você pensar na quantidade de precisão que espera para seus cálculos.

> Por hoje foi isso, mas as bizarrices de cálculos com números reais na JVM me surpreendem a cada dia, talvez eu complemente esse post no futuro :)
