---
layout: post
title: "Bizarrices do BigDecimal na JVM"
modified:
categories: blog
excerpt: ""
tags: [Kotlin]
image:
  feature:
date: 2018-10-11T00:58:56-02:00
---

fazer operações matemáticas com double e float resulta nuns numero quebrado nada a ver

```
0.1 + 0.2 //0.30000000000000004
```

Então pra fazer calculos a galera instrui a usar o `BigDecimal`
que deveria ser bom

mas é quase tao ruim quanto KKKKK

```
BigDecimal(0.1) + BigDecimal(0.2)
0.3000000000000000166533453693773481063544750213623046875
```

ou seja, tbm dá com valores zuados... mas ué? pq?!?! Eles nao falaram que era bom?! :thinking_face:
se tu passar uma string no construtor ele trata certo:

```
BigDecimal("0.1") + BigDecimal("0.2")
0.3
```

Então se liguem nisso, usem o construtor string sempre
Mais um easter egg:

```
BigDecimal("0") == BigDecimal("0.0") //false
```

```
BigDecimal(0) == BigDecimal(0.0) // true
```

Isso acontece pq o `equals` do java vai comparar no primeiro caso com String "0.0" == "0" //false
Já com double 0.0 == 0 //true
