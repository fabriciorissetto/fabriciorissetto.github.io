---
layout: post
title: "Domain Model vs Persistence Model"
modified:
categories: blog
excerpt: ""
tags: [DDD]
image:
  feature:
date: 2016-06-26T15:39:55-04:01
---

## O que é um modelo de persistência (persistence model)?

Quando utilizamos algum tipo de ORM como o Entity Framework e Hibernate precisamos mapear as tabelas do banco de dados para as classes da aplicação. Essas classes são os nossos "modelos de persistência". Antigamente era algo mais ou menos assim:

```c#
public class Livro 
{ 
    [Key] 
    public int Isbn { get; set; }
    [Required]      
    [MaxLength(10)]
    public string Nome { get; set; }  
    public DateTime DataPublicacao { get; set; }
}
```

Com a evolução dos ORM's passou a ser possível fazer essas configurações sem a necessidade do uso de data annotations, através de arquivos que chamamos de `mappings` (tanto Entity Framework como NHibernate possuem interfaces fluent para fazermos esses mapeamentos de forma fácil). Assim conseguimos criar essas classes de persistência de maneira POCO (Plain Old CLR Object), o que é muito mais bonito:

```c#
public class Livro 
{ 
    public int Isbn { get; set; }    
    public string Nome { get; set; }
    public DateTime DataPublicacao { get; set; }  
}
```

Tendo essa possibilidade virou algo muito comum usar essas classes no domínio das aplicações, ou seja, a mesma classe que é utilizada para modelar a persistência é utilizada também para modelar o domínio.

## Mas o que é um modelo de domínio (domain model)?

São as classes que representam o domínio da sua aplicação. Elas expressam não só dados mas também (e principalmente) comportamentos:


```c#
public class Livro 
{ 
    public int Isbn { get; private set; }    
    public string Nome { get; private set; }
    public DateTime DataPublicacao { get; private set; }
    public Usuario Revisor { get; private set; }

    public void Publicar(Usuario revisor)
    {
        if (revisor == null)
            throw new ArgumentException("É obrigatório ter um revisor para publicar!");
        
        this.DataPublicacao = DateTime.Now;        
        this.Revisor = revisor;
    }

    ...
}
```

> **Domain model** na definição raiz do DDD é na verdade um conceito mais amplo do que isso. Um domain model pode ser representado de outras maneiras além do código, afinal, como o nome já diz é apenas um modelo. Um modelo da solução de um determinado problema. O objetivo de um modelo é representar algo. Orientação a objetos é uma maneira de representar algo, portanto, é uma peça do modelo. 

# Mesclar ou não mesclar, eis a questão!

Uma das coisas mais comuns e intuitivas quando iniciando uma aplicação em DDD é mesclar esses dois modelos em um só. Ou seja, você pega aquela classe `Livro` do último exemplo acima (domain model) e mapeia ela em seu ORM. 

Ela tem métodos que o ORM não vai utilizar (já que para o ORM só interessam as propriedades) mas isso não vai gerar problema algum. Nem mesmo os campos com `private set`, pois os ORMs atribuem os valores das propriedades por reflection.

Então a conclusão é: 

> **Podemos sim utilizar a mesma classe para representar o modelo de domínio e também mapeá-la no ORM. Dois pelo preço de um!**

Porém, **cuidado**, isso é uma decisão que deve ser tomada com muita prudência pois pode trazer problemas a longo prazo. 

Respondi sobre isso [nessa](http://stackoverflow.com/a/34436709/890890) questão do stackoverflow e pretendo reproduzir aqui os *prós* e *contras* de cada um desses approachs.

## Persistence e Domain model numa classe só

Se você está construindo uma aplicação nova na qual se tem o conhecimento de que o domínio dificilmente sofrerá modificações drásticas e se o banco de dados está sendo construído com o único propósito de atender a essa aplicação, provavelmente essa seja a maneira mais prática a se seguir. Entretanto quando maior seja a ambição de vida longa do projeto (em matéria de manutenções e continuidade de desenvolvimento) maior o risco desse approach.

***Vantagens:***

* Velocidade extremamente rápida no desenvolvimento

***Desvantagens:***

* Os desenvolvedores tendem a ser mais receosos com refatorações e mudanças estruturais no domínio, pois elas afetam diretamente os mapeamentos do ORM. Esse medo não é bom para o domíno.
* Se o domínio começa a divergir muito do banco de dados certamente surgirão dificuldades para mantê-lo em harmonia com o ORM. Aquela produtividade ganha no início do desenvolvimento começa a se degradar e atingir a ponta da parábola. Quanto mais próximo do domínio mais difícil fica configurar o ORM. Quanto mais próximo do ORM pior fica o domínio.

## Persistence e Domain model como duas classes separadas

Essa abordagem traz a principal vantagem de modelar o domínio com liberdade e sem medo de refactorings. Nenhuma limitação proveniente do ORM ou qualquer outro aspecto de persistência dos dados. Ou seja, não importa se o banco de dados é relacional, orientado a documentos, grafos, ou se você salva tudo num arquivo de texto, nada disso afetará a maneira como você modela seu domínio. 

Eu tendo a sugerir essa separação para projetos utilizam base dados legadas. E, principalemente, para projetos que ambicionam vida longa.  

***Vantagens:***

* Maior liberade para modelar o domínio.
* Banco de dados é tecnologia. Tecnologia muda com muito mais frequencia do que o negócio. Dessa maneira ganhamos liberdade não só no domínio mas também no repositório, pois os detalhes de persistência podem mudar drasticamente sem afetar o domínio.
* Maior facilidade para implementar outros conceitos importantes do DDD como o [Bounded Context](http://martinfowler.com/bliki/BoundedContext.html).

***Desvantagens:***

* Necessidade de um esforço maior nas conversões de dados (mapeamentos) entre as camadas de repositório e domínio. O que resulta em um desenvolvimento um pouco mais trabalhoso e também numa pequena perda (provavelmente desconsiderável) de velocidade em runtime devido a uma conversão extra.
* Dessa maneira, no final das contas, acabamos perdendo diversos benefícios de utilizar um ORM ([tracking changes](https://msdn.microsoft.com/library/dd456848(v=vs.100).aspx), por exemplo). 

Enfim, essa é a visão que tenho e na qual sempre pondero no momento de conceber a arquitetura das minhas aplicações. 
