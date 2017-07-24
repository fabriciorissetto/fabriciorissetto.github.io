---
layout: post
title: "Proxy Pattern e Proxies dinâmicos em .NET com DynamicObject"
modified:
categories: blog
excerpt: ""
tags: [.NET,Patterns]
image:
  feature:
date: 2017-07-21T00:58:56-02:00
---

O Proxy é um padrão de uso relativamente específico, mas que quando bem aplicado resolve problemas de maneira muito sexy.

A imagem abaixo, retirada do site [dofactory.com](http://www.dofactory.com/net/proxy-design-pattern), exemplifica de uma maneira ~~que não dá pra entender~~ como o padrão funciona:

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/2017-07-21-dynamic-object/proxy-pattern.png">
</div>

Mas não esquenta em entender a imagem, eu coloquei ela aqui só pra citar o site dofactory, que é uma ótima referência pro aprendizado de patterns. 

Portanto, vou explicar o padrão com um exemplo real em que pude aplicá-lo recentemente. Vamos lá.

## O Problema: Parte 1

Imagine um **job**, que em .NET poderia ser até mesmo uma Console Application, no qual é executado de forma recorrente para fazer determinadas tarefas no banco de dados durante a madrugada. Porque na madrugada? Poruqe essas tarefas são pesadas e causariam uma lentidão nos sistemas que dependem desse banco de dados. Esse job possui algumas **configurações** como, o horário que deve iniciar a execução, horário de parada, intervalo de tempo em que deve ficar parado (Idle) entre uma execução e outra, etc. 

Um trecho de código bem escrito vale mais que mil palavras, então vamos fazer o esboço disso.

```csharp
public class Job
{
    public Config Config { get; set; }

    public void Execute()
    {
        //...
    }
}
```

*Detalhe: Esses exeplos de código seguem uma versão simplificada por questões didáticas. A classe acima poderia ser abstrata bem como os modificadores de acesso de suas propriedades serem mais restritivos, o Config por exemplo, poderia ter o set privado e ser atribuído durante a construção da classe.*

Os dados de configuração do job estão na classe **Config**: 

```csharp
public class Config
{
    public Config(/* ... */) { /* ... */ }

    public DateTime? StartTime { get; private set; }
    public DateTime? EndTime { get; private set; }
    public int IdleInterval { get; private set; }
    public int ExecutionInterval { get; private set; }
    public bool Enabled { get; private set; }
}
```

Esse código poderia ser consumida por uma console application da seguinte forma:

```csharp
class Program
{
    public void Main()
    {
        var job = new JobImportacaoExcel(); //Suposta classe que herda de Job

        if (job.Config.Enabled)
        {
            while (job.Config.StartTime >= /* ... */)
            {
                job.Execute();
            }
        }
    }
}
```

## O Problema: Parte 2
Até aí tudo bem. Agora imagine que temos diversos Jobs como esse executando, todos eles dependendo dessa classe Config, e o cliente pede a seguinte modificação:

> No fim de semana os jobs deverão rodar durante o dia também (e não somente de madrugada), além disso o intervalo entre uma parada e outra deverá ser menor. Para atender isso gostaríamos que os jobs obedecessem um config diferente durante os finais de semana. Ou seja, os valores de StartTime, EndTime, IdleInterval, e qualquer outro, poderiam ser diferentes no sábado e domingo.

Existem diversas maneiras de atender isso, uma das mais elegantes é utilizando um Proxy. Enfim chegamos a ele! 

## Solução 1: Proxy Pattern

```csharp
public class ConfigProxy
{
    private Config ConfigWeekdays { get; set; }
    private Config ConfigWeekend { get; set; }

    public ConfigProxy(Config configWeekdays, Config configWeekend) { /* */ }
    
    private Config CurrentConfig
    {
        get
        {
            var today = DateTime.Now.DayOfWeek;
            var todayIsWeekend = today == DayOfWeek.Saturday || today == DayOfWeek.Sunday;

            return todayIsWeekend ? ConfigWeekend : ConfigWeekdays;
        }
    }

    public DateTime? StartTime { get { return CurrentConfig.StartTime; } }
    public DateTime? EndTime { get { return CurrentConfig.EndTime; } }
    public int IdleInterval { get { return CurrentConfig.IdleInterval; } }
    public int ExecutionInterval { get { return CurrentConfig.ExecutionInterval; } }
    public bool Enabled { get { return CurrentConfig.Enabled; } }
}
```

Note que ele possui dentro dele os dois configs, um pra ser utilizado durante a semana e outro no fim de semana. O mais interessante é que ele expõe as mesmas propriedades da classe Config. Ele imita a classe config, porém internamente modifica o comportamento de cada propriedade ou método.

> *OBS: Se a classe Config implementasse uma interface a solução seria ainda mais interessante, pois o proxy poderia implementar essa mesma interface e, aí sim, a imitação de ser um config real seria ainda mais transparente.*

O legal dessa solução é que com uma mudança simples na classe Job, apenas mudando o tipo da propriedade `Config` para `ConfigProxy`:

```csharp
public class Job
{
    public ConfigProxy Config { get; set; }

    public void Execute()
    {
        //...
    }
}
```

A nosso "consumidor", a console application, não seria impactado em absolutamente nada. Nenhum caracter alterado:

```csharp
class Program
{
    public void Main()
    {
        var job = new JobImportacaoExcel(); 

        if (job.Config.Enabled)
        {
            while (job.Config.StartTime >= /* ... */)
            {
                job.Execute();
            }
        }
    }
}
```

> Um outro exemplo de uso do Proxy Pattern, talvez até mais interessante do que o demonstrado acima, é esse [daqui](https://github.com/fabriciorissetto/design-patterns-examples/blob/master/2.%20Structural%20Patterns/Proxy%20Pattern/ConsoleApplication1/SerasaProxy.cs) onde eu crio um proxy pra otimizar e restringir chamadas a um serviço pago do Serasa.

## Solução 2: Proxy dinâmico utilizando DynamicObject

Apesar da solução apresentada ser bacana e simples de entender (isso é imporantíssimo), o que aconteceria se uma nova propriedade ou método fosse criado dentro da classe real `Config`? 

O resultado é que esses membros não seriam refletidos no nosso proxy (`ConfigProxy`) até que alguém fosse lá e manualmente tratasse esses novos campos.

Para evitar essa reescrita dentro do `ConfigProxy` poderíamos utilizar um recurso do framework chamado [DynamicObject](https://msdn.microsoft.com/en-us/library/system.dynamic.dynamicobject.aspx). 

Vejam só que bacana:

```csharp
public class DynamicConfigProxy : DynamicObject
{
    private Config ConfigWeekdays { get; set; }
    private Config ConfigWeekend { get; set; }

    public ConfigProxy(Config configWeekdays, Config configWeekend) { /* */ }

    private Config CurrentConfig
    {
        get
        {
            var today = DateTime.Now.DayOfWeek;
            var todayIsWeekend = today == DayOfWeek.Saturday || today == DayOfWeek.Sunday;

            return todayIsWeekend ? ConfigWeekend : ConfigWeekdays;
        }
    }

    public override bool TryGetMember(GetMemberBinder propertyCalledByUser, out object result)
    {
        result = typeof(Config)
                    .GetProperty(propertyCalledByUser.Name)
                    .GetValue(this.CurrentConfig);

        return true;
    }
}
```

O método `TryGetMember` é acionado toda vez que o objeto recebe uma chamada pra um membro que não existe. E ali dentro chamamos a propriedade do respectivo config através de reflection.

Feito isso, apenas teríamos que modificar a classe Job para tratar a propriedade como dynamic.

```csharp
public class Job
{
    //A instância seria do tipo DynamicConfigProxy (sendo setada via contrutor - que foi omitido dos exemplos por questão de legibilidade)
    public dynamic Config { get; set; } 

    public void Execute()
    {
        //...
    }
}
```

O interessante dessa solução é que, novamente, nenhuma mudança seria necessária no nosso consumidor:

```csharp
class Program
{
    public void Main()
    {
        var job = new JobImportacaoExcel(); 

        if (job.Config.Enabled)
        {
            while (job.Config.StartTime >= /* ... */)
            {
                job.Execute();
            }
        }
    }
}
```

*O lado ruim é que pelo campo ser um "dynamic" não teríamos autocomplete :(*

### DynamicObject no framework: SignalR

Se você já utilizou o SignalR provavelmente passou pelo DynamicObject e nem se deu conta. O código está [aqui](https://github.com/aspnet/SignalR/blob/8ba29b578da537af40d3e4942da6580963d7812c/src/Microsoft.AspNetCore.SignalR/DynamicClientProxy.cs), e você o utiliza quando faz isso:

```csharp
public class ChatHub : Hub
{
    public void Send(string name, string message)
    {
        Clients.All.addNewMessageToPage(name, message);
    }
}
```

Aquela propriedade `All` é um DynamicObject e o `addNewMessageToPage()` é a função que você quer que seja chamada no JavaScript de seus clients conectados no Hub. 

## Outras maneiras de criar Proxies Dinâmicos (Solução 3)

Embora a classe DynamicObject seja uma das formas mais simples de implementar proxies dinâmicos em .NET, existem diversas outras maneiras. Se você estiver procurando algo mais robusto, não pode deixar de dar uma olhada no projeto [Castle Dynamic Proxy](http://www.castleproject.org/projects/dynamicproxy/). Um dos únicos pré-requisitos de utilizá-lo é que os membros que você quer "interceptar" devem possuir o modificador de acesso como `virutal`, o que pode não ser um problema.
