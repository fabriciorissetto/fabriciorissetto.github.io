---
layout: post
title: "A base da pirâmide de testes é um bom compilador"
modified:
categories: blog
excerpt: ""
tags: [Testes, Linguagens de Programação]
image:
  feature:
date: 2019-07-19T00:58:56-02:00
---

**Fail Early & Fail Fast**: um princípio que está constantemente presente, de uma forma ou de outra, em praticamente todas áreas profissionais. Em desenvolvimento de software ele se manifesta constantemente mesmo que muitas vezes o abracemos sem conscientemente notar que está ali. Quando adotamos práticas de Agile (como alternativa ao Waterfall) estamos abraçando esse princípio. Quando construímos um MVP, quando aplicamos práticas de Lean UX, quando escrevemos testes de unidade, estamos seguindo, mesmo sem saber ou notar, o fundamental princípio de falhar cedo (e aprender com os erros - eu espero).

## A famosa pirâmide de testes

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2019-07-11-base-piramide-testes/teste-piramide.png">
</div>

Essa pirâmide é amplamente conhecida e é uma boa abstração para transmitir a ideia de que: quanto mais tarde você testa determinada peça do seu software, maior será o custo da correção.

Mas e se eu te disser que quando escolhemos a linguagem de programação de um software, dependendo da escolha, podemos nos presentear com mais uma sólida camada sob a base dessa pirâmide? Uma camada que pode prevenir que alguns tipos de erros atinjam as de cima...

Não sou bom em desenhos, mas me permita tentar:

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2019-07-11-base-piramide-testes/teste-piramide-compilador.png">
</div>

Isso mesmo, estou falando do COMPILADOR. Mais precisamente, dos compiladores das [linguagens estaticamente tipadas](https://stackoverflow.com/q/1517582/890890).

Ter um bom compilador é a maior vantagem de se utilizar uma linguagem estaticamente tipada. Entretanto, tenho impressão de desenvolvedores nem sempre enxergam esse valor. Sem fazer julgamento, apenas uma constatação baseada em minha observação empírica: isso normalmente ocorre com desenvolvedores de linguagens dinamicamente tipadas. Principalmente com aqueles que jamais trabalharam numa linguagem estática, ou que trabalharam em linguagens estáticas mais "idosas" e verbosas, como o Java.

## Mais compilador, menos testes

> NOTA: Nesse post estou usando o termo "compilador" como uma generalização para: validação [léxica, sintática e semântica](https://johnidm.gitbooks.io/compiladores-para-humanos/content/part1/structure-of-a-compiler.html) do código em tempo de desenvolvimento (não runtime), seja ela feita por um compiler, transpiler ou qualquer outro termo similar que atenda a esse fim (em tempo de desenvolvimento!).

Ter um bom compilador no seu arsenal de ferramentas faz com que você precise escrever menos testes. Vou além: ter uma boa **linguagem de programação**, aliada a um forte **compilador** (se possível bem integrados através de uma **IDE ou editor de texto empoderado**), faz você escrever MUITO menos testes e passar MUITO menos trabalho.

Para fazer um rápido exercício dessa sinergia do compilador com a linguagem de programação, basta dar uma olhada [nesse post onde apresento algumas features de Kotlin](http://www.fabriciorissetto.com/blog/kotlin-introducao/). Veja quantas facilidades a linguagem nos dá que nos livra de escrever código boilerplate. Lembre-se: quanto mais boilerplate você escreve, mais brechas para bugs você abre, mais código você tem para testar, mais código você tem para ser lido e mantido (por você e pelos seus colegas). Como já dizia Jeff Atwood: “[The best code is no code at all](https://blog.codinghorror.com/the-best-code-is-no-code-at-all/)”.

Apesar disso, sabemos que por mais avançada e alto nível que a linguagem seja, em alguma esfera, nós vamos ter que codar. Acontece que quando programamos sem um compilador, por maior que seja o esforço e engajamento em testar a aplicação, é comum acabar descobrindo em tempo de execução erros básicos como: um typo do desenvolvedor ao chamar um método ou variável, ocasionando erros como [undefined method](https://stackoverflow.com/questions/21072355/undefined-method-nomethoderror-ruby) e [undefined local variable](https://stackoverflow.com/questions/9671259/ruby-local-variable-is-undefined).

Já trabalhei em um ambiente com diversos desenvolvedores Ruby experientes mantendo uma mesma aplicação. E mesmo sendo a comunidade de Ruby, de longe, a mais forte e qualificada em testes e grande adepta do TDD (essa aplicação tinha mais de 8 mil testes de unidade), ainda assim lá acontecia em produção erros básicos como os que acabei de citar. Não adianta, *shit happens*! Na verdade, com ou sem um compilador. Meu ponto é que quando a linguagem e a ausência de um compilador permite esse tipo de coisa, *"shit happens mais ainda"*.

Sim, é difícil separar o compilador da linguagem. Pelo simples fato de que o design de um é impactado pelo outro. Se um código pode ou não gerar bugs é uma mistura entre a permissividade que a linguagem oferece aliado ao trabalho que o compilador faz durante sua interpretação e compilação. Por exemplo, Kotlin não permite que você passe `null` para uma variável a não ser que ela explicitamente, em sua definição, aceite receber `null`.

```java
var nome = "foo" //OK, variável definida como String e inicializada com "foo"
nome = "bar" //OK, valor sobrescrito para "bar"
nome = null //BAAAM! NÃO COMPILA
```

Estamos falando basicamente da prevenção de bugs, não é mesmo? No exemplo acima, o compilador está nos ajudando a evitar um dos mais famosos de todos, o tal `Null Reference Exception`, também conhecido como [o erro de um Bilhão de Dólares](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions), dado uma estimativa (obviamente chute, e do próprio criador) do prejuízo que isso já deve ter causado em produção pras empresas ao longo dos anos.

Enfim, os exemplos são os mais variados, mas tenho certeza de que você entendeu a ideia. Quis trazer aqui uma provocação quanto aos benefícios de se utilizar o compilador como um dos pilares para a descoberta de falhas cedo, a cada linha escrita, a cada caracter.

**Um disclaimer pra finalizar**: Deve ter ficado claro no texto, dado algumas citações à linguagens dinâmicas, que eu tenho forte preferência pelas estáticas. Isso é verdade. Tive em minha carreira a oportunidade de trabalhar e aprender bastante com ambos os estilos e quis deixar aqui, documentado, uma parte dos porquês de tal preferência. Pretendo dar seguimento e aprofundar mais na comparação dos estilos em outro post.
