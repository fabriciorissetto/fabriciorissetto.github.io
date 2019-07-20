---
layout: post
title: "Kotlin: Introdução a algumas das principais features da linguagem"
modified:
categories: blog
excerpt: ""
tags: [Kotlin, Linguagens de Programação]
image:
  feature:
date: 2018-09-14T00:58:56-02:00
---

Na [Creditas](https://www.creditas.com.br/) estamos começando a utilizar Kotlin para modelar nossas aplicações que são *Core Business* e que exigem padrões de projeto mais complexos como o DDD. Até então estávamos utilizando majoritariamente Ruby para esse fim. Porém, dado a inúmeras insatisfações com a stack Ruby para esse caso de uso em específico, resovemos testar Kotlin e o resultado tem sido muito bom. Talvez no futuro eu faça algum post falando mais detalhes por trás das motivações dessa migração. Neste aqui pretendo focar em algumas simples features da linguagem que inicialmente nos chamaram atenção e nos fizeram avaliar com seriedade essa linguagem que hoje estamos utilizando.

Uma das coisas que tem vindo bem forte no movimento de linguagens funcionais é a **imutabilidade**. Kotlin, diferente de algumas linguagens como C#, Java, Ruby e Python, tem uma série de facilidades para lidar com imutabilidade de forma simples e sem verbosidade. Um exemplo são os `data classes`:

## Criando Value Objects/DTOs

```java
data class Endereco(val rua: String, val numero: Int)
```

Ao usar a keyword `data class` você ganha diversas coisas como, por exemplo, um construtor padrão, um lindo método `equals()` **que compara os valores dos atributos ao invés da referência da instância**, além de outros métodos como `copy()`, `toString()`.

```java
val instancia1 = Endereco("foo", 1)
val instancia2 = Endereco("foo", 1)

instancia1 == instancia2
// true!
```

São funcionalidades perfeitas para atender implementações de [Value Objects do DDD](https://imasters.com.br/back-end/aprendendo-sobre-objetos-de-valor) e [DTOs](https://pt.wikipedia.org/wiki/Objeto_de_Transfer%C3%AAncia_de_Dados).

## Criando uma classe

Uma classe comum contendo um atributo `nome` imutável (val) e `endereco` mutável (var):

```java
class Pessoa(val nome: String, var endereco: Endereco) {
    fun tamanhoDoNome() {
        println("Olá, meu nome tem ${nome.length} letras!")
    }
}
```

## Diferença entre `val` e `var`
Eles na verdade são açucares sintáticos para criação de métodos getters e setters. O `val` equivale a uma propriedade com apenas getter, o `var` equivale a uma propriedade com getter e setter.

A seguinte classe:

```java
class Animal(var nome: String)
```

É equivalente a isso em Java:

```java
public final class Animal {
  private String nome

  public Animal(String nome) {
    this.nome = nome;
  }

  public String getNome() {
    return this.nome;
  }

  public void setNome(string value) {
      this.nome = value;
  }
}
```

Muito menos verboso, não é mesmo? Durante a migração de um dos nossos projetos de Ruby para Kotlin conseguimos diminuir quase pela metade a quantidade de linhas de código. Nunca se esqueça: quanto mais boilerplate você escreve, mais brechas para bugs você abre, mais código você tem para testar, mais código você tem para ser lido (por você e pelos seus colegas) e mantido. Complexidade, seja lá de qual forma se manifeste, [é custo](https://www.elemarjr.com/pt/2018/02/complexidade-e-custo/).

**Observação**: Ambos `val` e `var` podem ser utilizados tanto em classes normais `class` quanto em classes de dados `data class`. As pessoas normalmente entendem o `data class` como sendo por si só uma classe imutável, quando na verdade o que garante que seus campos não sejam modificados é o uso de atributos do tipo `val`. O `data class` por si só apenas facilita a criação do construtor, método `equals()` e etc (como citado anteriormente).

## Fechado por padrão

Note o `final` na tradução para Java no código anterior. Isso mesmo, em Kotlin todas as classes são seladas por padrão (***"Design and document for inheritance or else prohibit it"***). Caso vocẽ queria estender uma classe você terá que marcá-la como `open`. Gosto desse comportamento pois se formos analisar em nossos projetos a esmagadora maioria das classes não possuem herança e não foram projetadas para isso.

## Interoperabilidade

Outra coisa legal, em relação a interoperabilidade, é que se você consumir em Java um arquivo `.jar` que foi gerado a partir de Kotlin, você vai poder chamar os métodos setters e getters normalmente sem perceber nada de diferente do que está acostumado.

## Variáveis locais imutáveis

Não apenas um açucar sintático para criação dos getters, setters e parâmetros de construtor, o `val` é uma keyworkd que também pode ser utilizada no escopo de funções para definir "variáveis" imutáveis (que nesse caso não deveriam ser chamadas de variáveis, já que não variam, anyway... é assim que a propria [documentação](https://kotlinlang.org/docs/reference/basic-syntax.html#using-type-checks-and-automatic-casts) chama):

```java
val animal = Animal("Scooby")
animal = Animal("Marley") //Error: Val cannot be reassigned!
```

Das linguagens que citei anteriormente C#, Ruby e Python simplesmente não possuem essa feature (possuem apenas para tipos primitivos, através do uso de constantes hardcoded). Java possui através da junção das keyworkds `final` e `var`, exemplo: `final var animal = new Animal("Scooby")`.

**Observação 2**: Na verdade o `val` não garante a imutabilidade de fato. [MIND BLOW] O `val` apenas garante que aquela variável vai ser *atribuída uma única vez*. Ou seja, a instância para qual aquela variável está apontando pode sofrer mudanças internas e isso está além do escopo do val.

Por exemplo, imaginando que a classe `Animal` tenha um método que altera estado do objeto, adicionando uma coleira no animal:

```java
val animal = Animal("Scooby")
animal.adicionarColeira(coleira)
//A instância do objeto foi modificada, porém a variável não foi reatribuida, então Sucesso!
```

## Null safety

A proteçao contra nulos é a feature que eu mais gosto em Kotlin e não está presente em nenhuma das linguagens citadas anteriormente.

> O sistema de tipos de Kotlin foi desenhado visando eliminar o perigo de exceções de `NullReference` no código, também conhecido como o [Erro de um Bilhão de Dólares](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions).

Em Kotlin os tipos nulos e não nulos são tratados como coisas distintas. O default é ser **não nulo**. Caso você queira que seja nulo você precisa dizer isso explicitamente, e o compilador vai te **OBRIGAR** a tratar essa variável em todo lugar. Isso acaba por desencorajar o uso de tipos nulos e nos obriga a refletir onde eles realmente são necessários, o que é ótimo.

Por exemplo, se no código anterior alterarmos o atributo `nome` para ser nullable, o seguinte código **não compila**:

```java
class Pessoa(val nome: String?, var endereco: Endereco) {
    fun tamanhoDoNome() {
        println("Olá, meu nome tem ${nome.length} letras!")
    }
}
```

É preciso fazer o tratamento da possibilidade de null reference no trecho `${nome.lenght}`! Uma das formas de tratar isso seria utilizando o operador `?`, também conhecido como safe call operator, exemplo:`${nome?.length}`. Entretanto nem sempre é necessário, pois o compilador é inteligente e consegue identificar se você já tratou a variável em determinado escopo, veja o exemplo abaixo:

```java
if (nome != null) {
    // Aqui dentro a variável nome é automaticamente convertida para 'não nula'
    // Portanto não é necessário tratar o null reference, o compilador sabe que não vai acontecer!
    println("Olá, meu nome tem ${nome.length} letras!")
}
```

O nome dessa feature é **smart cast** e não serve apenas para tipos nullables.

## Smart Cast

```java
fun funcaoQualquer(x: Any) {
    if (x is String) {
        // x automaticamente convertido para String nesse escopo!
        print(x.length)
    }
}
```

E funciona com o pattern matching do `when` também:

```java
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

## Lidando com coleções
Esse [post](https://medium.freecodecamp.org/my-favorite-examples-of-functional-programming-in-kotlin-e69217b39112) mostra ótimos exemplos ao se trabalhar com coleções em Kotlin. Vou reproduzir um bem simples aqui:

```java
class Student(
    val name: String,
    val surname: String,
    val passing: Boolean,
    val averageGrade: Double
)
```

Considerando uma lista que contem vários estudantes (classe definida acima), é possível fazer operações nessa lista utilizando as funções da standard library do Kotlin:

```java
students.filter { it.passing && it.averageGrade > 4.0 }
    .sortedBy { it.averageGrade }
    .take(10)
    .sortedWith(compareBy({ it.surname }, { it.name }))
```

O código é auto explicativo pra maioria das pessoas que vem de outras linguagens, já que quase todas dão suporte pra esses tipos de operações. C# dá suporte pra isso há 10 anos, Java introduziu as Streams no Java 8 há 4 anos (porém com uma sintaxe muito mais suja do que o exemplo acima), Ruby, Python, Javascript e praticamente todas as linguagens mainstreams de mercado dão suporte, umas de forma mais clean e completa do que outras, mas todas possuem suporte em algum nível.

## Monads e outras abstrações funcionais

Kotlin não possui na sua standard library um grande acervo para abstrações e estruturas de dados comuns em linguagens funcionais. Entretanto, é possível utilizar esse tipo de feature com o auxílio de libs de extensão como o [Arrow](https://arrow-kt.io/). Esse tipo de lib é bem comum mesmo em linguagens que possuem certo suporte para estruturas de dados funcionais. Scala, por exemplo, apesar de possuir algumas estruturas já implementadas na sua standard library, possui libs fortes que complementam essas funcionalidades como a [Scalaz](https://github.com/scalaz/scalaz) e [ScalaCats](https://github.com/typelevel/cats).

## Outras features não mencionadas

Nesse post citei apenas algumas features para não estender tanto, mas tem muito mais coisas interessantes do que mostrei aqui. Aqui vai mais algumas:

* Funções soltas em nível de package (fora de classes)
* Tipos de dados algébricos
* Pattern matching **exaustivos** com `when`
* Ótimo suporte para criação de [DSLs](https://kotlinlang.org/docs/reference/type-safe-builders.html#a-type-safe-builder-example) poderosas
* Delegated properties
* Extension functions

A [documentação oficial da linguagem](http://kotlinlang.org/docs/reference/) é muito boa e considero um ótimo começo de estudo pra quem tiver interesse em se aprofundar mais!
