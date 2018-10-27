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

O propósito desse post é ser rápido, portanto não pretendo entrar em muitos detalhes teóricos por trás das características dos valores de ponto flutuante na JVM, pois é bem simples de achar em respostas como [essa](https://stackoverflow.com/a/1661313/890890) no StackOverflow que dão um ótimo "deep dive" na explicação científica por trás de pontos flutuantes na representação binária em geral.

### E lá vamos nós

Veja o resultado da operação abaixo:

```java
0.1 + 0.2
//0.30000000000000004
```

Isso é o comum na maioria das linguagens devido a incapacidade de representar [alguns números decimais em binário](http://cs.furman.edu/digitaldomain/more/ch6/dec_frac_to_bin.htm).

Em Java, ou melhor, no munto JVM (Kotlin, Scala, etc),é muito comum achar em [respostas](https://stackoverflow.com/a/3413493/890890) do StackOverflow a recomendação de se utilizar o `BigDecimal`
para esse tipo de operação. No entanto ao passar um `double` como parâmetro para um BigDecimal você continua com o mesmo problema:

```java
BigDecimal(0.1) + BigDecimal(0.2)
//0.3000000000000000166533453693773481063544750213623046875
```

Isso acontece porque o `double` já perdeu a precisão exata antes mesmo do `BigDecimal` ter sido construído. E aí as coisas começam a ficar estranhas pra quem nunca trabalhou com `BigDecimal`, pois logo você descobre que utilizando o construtor que recebe uma `String` tudo funciona como o esperado:

```java
BigDecimal("0.1") + BigDecimal("0.2")
//0.3
```

***Aeeee! Finalmente. Agora o BigDecimal funciona com suas completas capacidades e com precisão exata (sem a treta dos pontos flutuantes reprentados em binário).***

Mas algumas outras coisas podem intrigar você logo em seguida:

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

Isso acontece porque o BigDecimal possui internamente uma informação de **precisão**. BigDecimals inciados com o construtor Double contendo os valores `0` e `0.0` possuem ambos unidade **0** e escala **0**. Já BigDecimals inciados com o cosntrutor String contendo os valores `"0"` e `"0.0"` possuem a unidade **0** mas o segundo possui escala **1**! Por isso a igualdade resulta em falso.

Se você utilizar, ao invés do `equals()`, o `compareTo()`, a comparação ignora a diferença de precisão.

```java
BigDecimal("0").compareTo(BigDecimal("0.0"))
//0
```

## A importância da precisão para as operações

O cálculo `1 / 3` resulta em uma [dízima periodica](https://pt.wikipedia.org/wiki/D%C3%ADzima_peri%C3%B3dica). Com double o resultado é limitado pela sua precisão pré-definida:

```java
1.0 / 3.0
//0.3333333333333333
```

Já com BigDecimal, se você não definir a precisão que deseja, a expressão `1 / 3` dá erro (o que faz todo sentido):

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
