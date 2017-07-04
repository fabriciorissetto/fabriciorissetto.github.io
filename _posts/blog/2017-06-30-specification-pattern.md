---
layout: post
title: "Usos equivocados do Specification Pattern"
modified:
categories: blog
excerpt: ""
tags: [Patterns, DDD]
image:
  feature:
date: 2017-06-30T00:58:56-02:00
---

Imagine essa classe:

```csharp
public class Livro
{
    public string Titulo { get; private set; }
    public Genero Genero { get; private set; }
    public Autor Autor { get; private set; }
    public DateTime? DataPublicacao { get; private set; }

    public Livro(string titulo, Genero genero, Autor autor)
    {
        Titulo = titulo;
        Genero = genero;
        Autor = genero;
    }

    public void Publicar(Revisor revisor) 
    {
        DataPublicacao = DateTime.Now; 
        Revisor = revisor; 
    }
}
```

E esse specification:

```csharp
public class LivroEstaPublicadoSpecification : Specification<Livro>
{
    public override bool IsSatisfiedBy(Livro livro)
    {
       return livro.DataRevisao != null && livro.Revisor != null;
    }
}
```

Algum problema nisso? Pode ser que não. Na verdade a resposta está na seguinte pergunta: "É responsabilidade do livro informar se ele está publicado?". Se a resposta for sim, então há um problema em usar especificação para isso. 

Portanto,

## Não retire da entidade regras que pertencem a ela

> Não é objetivo da especificação **absorver regras que pertencem à entidade**. E sim **absorver regras que NÃO pertencem à entidade** (explico melhor isso logo em seguida).

Ou seja, se é responsabilidade do Livro informar seu status de publicação então é lá que essa regra deve estar (dentro de uma propriedade get ou método).

Tenho visto em muitos projetos a utilização equivocada do Specification Pattern e esse exemplo acima é um desses casos. 

## Cuidado ao utilizar specification para "validação"

Outro caso que tenho visto também é esse:

```csharp
public class ContratoEstaValidoSpecification : Specification<Contrato>
{
    public override bool IsSatisfiedBy(Contrato contrato)
    {
            return contrato.Cliente != null &&
                   contrato.PrimeiraParcelaQuitada == true &&
                   contrato.Valor > 0;
    }
}
```

Isso dá pano pra manga...

Eu recomendo bastante a leitura desse [paper](https://www.martinfowler.com/apsupp/spec.pdf) de apenas 19 páginas escrito de forma colaborativa pelo Eric Evans e Martin Fowler (e não precisa tentar entender aqueles códigos Smalltalk, só entender o conceito e visualizar os diagramas é o suficiente). Nesse paper eles descrevem 3 principais problemas que podem ser solucionados com Specification:

* **Selection**: You need to select a subset of objects based on some criteria, and to refresh the selection at various times

* **Validation**: You need to check that only suitable objects are used for a certain purpose

* **Construction-to-order**: You need to describe what an object might do, without explaining the details of how the object does it, but in such a way that a candidate might be built to fulfill the requirement.

Note que esse segundo item, o de **validação**, não trata sobre o tipo de validação que acabei de expor ali em cima. Naquele meu exemplo estamos verificando se a entidade está em um estado válido, e existem alguns problemas em relação a isso. Lá vai.

##### Primeiro: 

As entidades de domínio devem estar sempre em estado válido. Se para um Contrato ser construído é necessário que ele tenha o campo X isso deve ser validado no seu construtor e impedir que a entidade seja criada caso o campo X não seja informado ou esteja incorreto.

Voltando ao exemplo do contrato onde a regra de validação, ao invés de ficar no construtor, foi parar numa Specification. Isso quer dizer que durante um breve momento, após a classe ter sido instanciada e antes do Specification ser utilizado para validá-la, nesse breve momento, alguém pode fazer a seguinte chamada:

```csharp
contrato.Aprovar();
```

**Resultado**: um contrato que estava num estado inválido foi aprovado.

***Ah Fabrício, mas eu poderia ter validado aqueles campos dentro do método Aprovar né?*** Sim, poderia. Mas imagine que essa validação poderia, além de estar dentro do método `Aprovar()`, estar no método `EnviarParaAnalise()`, no método `CalcularJuros()`, e dentro dos próximos N métodos que possam vir a existir  e que dependam que esses campos estejam com dados válidos. Não é bacana ter essa validação espalhada em todos os cantos, correto? A não ser que faça parte de um fluxo de negócio que essas informações estejam em algum momento vazias ou com alguma inconsistência, devemos implementar o modelo de forma a isso não ocorrer.

Ainda fugindo um pouco do tópico de Specifications, outro exemplo é quando buscamos uma entidade do banco de dados. Se eu obtive um objeto `Contrato` do banco de dados e ele tem um campo `Cliente` vazio, isso me dá a enteder de que faz parte da regra de negócio que um contrato possa ser criado sem possuir um cliente associado a ele. *As vezes o desenvolvedor, por alguma questão de performance ou outros motivos, não traz a informação do banco de dados, o que se tratando de desenvolvimento orientado a domínio, não é correto. Existem diversas formas elegantes de driblar dificuldades como essa sem ferir o modelo (CQRS, Bounded Contexts, etc).*

##### Segundo:

Ufa, demorei mais do que eu gostaria no primeiro. 

Se o propósito é fazer validações de input do usuário, existem outras formas, que não o specification, que tratam isso de maneira mais adequada, uma das que estou gostando muito é o [Notification Pattern](https://github.com/andrebaltieri/FluentValidator/blob/master/FluentValidator.Tests/FluentValidatorTests.cs). Mas anteção, esse tipo de validação não deve ficar no domínio. Validar se o nome que o usuário digitou tem menos de 200 caracteres normalmente não é uma regra de negócio...

# Bons exemplos de Specification

Bom, já citei alguns exemplos em que considero equivocado o uso de Specification, agora vamos citar alguns exemplos legais. Na verdade, o exemplo que mais gosto é o que está no paper que citei, **sobre a responsabiliade de definir se uma carga está apta a ser transportada por determinado container**.

O exemplo é o seguinte: Estamos desenvolvendo um sistema de administração de cargas para uma transportadora. Essa transportadora tem diversos tipos de containers, com tamanhos e caracteríscas bem distintas. Nesses containers ela transporta cargas dos mais variados tipos.

Nessa breve descrição podemos visualizar duas entidades claras no nosso modelo: `Carga` e `Container`.

Agora algumas regras de negócio pra temperar ele:

Existem cargas que possuem restrição de temperatura, como: Frango Congelado, Pizza Congelada, Iogurte, Queijo, Ovo de Páscoa. E existem containers resfriados que são específicos para esse tipo de carga e que são higienizados. Mesmo entre os containers resfriados existe uma distinção devido a capacidade de temperaturas mínimas que cada um tem (alguns conseguem atingir a temperatura mínima para transportar iogurte, mas não frango congelado), além de que nem todos esses containers resfriados são higienizados e apropriados para acomodar alimentos.

Existem containers estofados para carregar cargas frágeis, como: vasos de cêramica, taças de cristal, etc.

Existem containers com blindagem e medidas de segurança específicas para transportar cargas perigosas como gazes, explosivos, produtos inflamáveis, etc.

Agora a pergunta:

**Container**: É responsabilidade do container saber quais cargas ele pode carregar? É responsabilidadde dele saber que existe um iogurte e que no futuro talvez poderá existir uma carga de um novo tipo no qual ele ainda não conhece? Imagine um container quadradão xingando um funcionário: *"Eeeepa!! Pera aí... requeijão eu não posso transportar não fiii! Sai sai sai sai, tira isso daqui."*

**Carga**: E o contrário? É responsabilidade da carga saber em quais containers ela pode ou não ser transportada? Uma carga tem um peso, uma validade, uma temperatura mínima para conservação, agora, a responsabilidade de saber se ela pode ou não ser transportada no *"Container Tabajara X 9800 Plus"* deveria ser dela?

Nesses casos o uso de Specification é um golaço, pois não estamos retirando das entidades reponsabilidades que pertencem a elas, e ao mesmo tempo estamos definindo regras dentro do nosso domínio de uma maneira clara, encapsulada, testável e reutilizável. Além de evitar um acoplamento desnecessário entre aquelas duas entidades.

Então, que tal então Specifications como esses:

```
CargaNecessitaContainerLimpoSpecification()
CargaNecessitaContainerResfriadoSpecification()
CargaNecessitaContainerLimpoEResfriadoSpecification()  
CargaNecessitaContainerEstofadoSpecification
CargaNecessitaContainerBlindadoSpecification()
```

No final das contas o uso de specifications, pelo menos nos projetos em que eu tenho atuado, não se faz algo assim "tão comum". Com perdão da cacofonia, o uso de specifications é bem específico e você provavelmente não irá construir um sistema que já na concepção tem uma arquitetura que envolva isso. Ele é algo que irá surgir durante a modelagem, em casos específicos e que provavelmente será claro para todos de que ele é "o padrão" que vai resolver aquele problema. Assim como funciona para todos outros patterns. 

Afinal, não começamos o projeto dizendo "Bom, essa pastinha aqui é onde vamos colocar os Strategies, aqui os Builders, nessa os Proxies, e naquela outra os Decorators", não é mesmo? Os patterns se aplicam e se adequam aos projetos e não o contrário.


