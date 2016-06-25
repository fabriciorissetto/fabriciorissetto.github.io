---
layout: post
title: "Clean Code - Dicas"
modified:
categories: blog
excerpt: "Dicas de clean code (código limpo) baseadas no livro \"Clean Code\", de Rober Cecil Martin"
tags: [clean-code]
image:
  feature:
date: 2013-10-06T15:39:55-04:00
date: 2013-10-06T15:39:55-04:00
---

Recentemente estive lendo o livro ***“Código Limpo - Habilidades Práticas do Agile Software”*** de **Robert Cecil Martin** no qual eu buscava referências para melhorar a qualidade de meu código.

Confesso que fui surpreendido por esse livro. Sobre os humildes olhares de minha autocrítica o que me faltava eram apenas algumas medidas avançadas e luxuosas de organização de código e talvez, também, algumas práticas excêntricas de orientação a objeto, pois no demais sempre fui caprichoso com meus códigos. Mas como eu disse: fui surpreendido, eu estava enganado! O que me faltava não era nada de excêntrico, nem tampouco avançado. **Era o básico.**

É impressionante o quanto um código pode ser melhorado ao se prestar atenção em detalhes. Há certos aspectos de um código que  são ruins e o hábito acaba fazendo com que aquilo se torne algo tão natural que não notamos o quanto é prejudicial fazê-lo. Mas dai chega alguém e diz: “Cara, seria melhor se você fizesse assim…” e sua visão sobre aquilo muda completamente. Na minha opinião isso é o princípio do Clean Code, prestar atenção em detalhes em que a maioria das pessoas não dá importância.

Mas enfim, baseado no aprendizado que obtive e no fato de que eu já havia procurado na internet, sem obter sucesso, dicas sobre Clean Code, resolvi fazer esse artigo com um compilado de práticas a serem observadas, e quem sabe, serem seguidas.

As dicas que venho a oferecer aqui estão descritas de forma resumida e em sua maioria foram retiradas do livro de [Uncle Bob](https://www.google.com.br/search?q=uncle+bob&aq=f&oq=uncle+bob) citado anteriormente.

Vamos lá...


# Variáveis

#### Utilize variáveis de domínio

Talvez você tenha aprendido, em seu primeiro cursinho de programação, que utilizar variáveis a, b e c é a coisa mais linda do mundo. Pois bem, não é!

Veja esse código:

```javascript
var a = b + c;
```

Ao visualizar esse código isoladamente não há como saber do que se trata. Ele está somando duas variáveis com nomes que não fazem sentido e colocando seu valor em outra variável que também não faz sentido.

O que é a variável b? Uma banana, uma maçã, a taxa anual de cobrança do seguro de um automóvel?
Sempre utilize variáveis com nomes auto descritivos que se enquadram no contexto de seu código.

Exemplo:

```javascript
var lucroLiquidoProduto = valorVendaProduto - valorCustoProduto;
```

# Código auto explicativo dispensa comentários

Ao contrário do que muitos pensam, encher o código de comentários não é uma boa prática.

Veja o exemplo abaixo:

```c#
bool isTestPage = page.HasAttribute("Test");
if (isTestPage) {
    ...
}
```

Não seria necessário colocar um comentário acima do if, pois a variável isTestPage explica perfeitamente o que ele se propõe a validar. Colocar o resultado de page.hasAttribute(“Test”) dentro da variável isTestPage explica o que é necessário fazer para que uma página seja de testes (apenas colocar o atributo Test na página).

Fato: 

> A maioria dos comentários serve apenas para justificar um trecho de código ruim. São como pedidos de desculpas do desenvolvedor por não ter conseguido se expressar em código.

Portanto, pense duas vezes antes de escrever um comentário. Se você está prestes a explicar seu código com comentários é porque, provavelmente, você fracassou em se expressar codificando. Se isso for verdade, prefira refatorar. Deixe o código limpo e auto explicativo.

Não se explique com comentários:

```java
//Valida se o cliente possui benefícios
if ((cliente.Idade > 45 && cliente.Salario < enumValorSalarios.SalarioMinimo)
```

Se explique com código:

```java
if (cliente.PossuiBeneficios())
```

Entretanto, nem todos os tipos de comentários são ruins. Saber onde e quando aplicar um comentário é uma ótima *skill*. Um comentário bem colocado pode salvar vidas.

Dois exemplos de bons comentários:

1º:

```java
//Essa importação demora algumas horas. Se iniciá-la, aguarde terminar,
//caso contrário a base de dados ficará inconsistente
ExecutarImportacaoProdutos();
```

2º:

```c#
Regex regexCPF = new Regex(@"^\d{3}\.\d{3}\.\d{3}-\d{2}$"); // 999.999.99
Regex regexTelefone = new Regex(@"^\(\d{2}\)\d{4}-\d{4}$"); // (99)9999-9999
Regex regexCEP = new Regex(@"^\d{5}-\d{3}$"); // 99999-99
```

# Não usar negação no if se houver um else

Esse é sutil e eu gosto muito. Especialmente porque foi a partir dele que pela primeira vez eu ouvi falar de clean code.

É simples, você tem o seguinte código:

```java
if (!visitante.IsAdulto)
    return "You have not permissions to see this content.";
```

Ele está perfeito. Entretanto surge a necessidade de alterá-lo, pois é preciso retornar uma outra mensagem quando o visitante for adulto. Por ser a maneira mais intuitiva, normalmente essa alteração seria feita da seguinte maneira:

```c#
if (!visitante.IsAdulto)
    return "You have not permissions to see this content.";
else
    return "Enjoy the content!";
```

É um detalhe simples, talvez seja até frescura, mas não há dúvidas (a não ser que você seja um adepto de [Yoda Programming](https://en.wikipedia.org/wiki/Yoda_conditions)) que dessa maneira fica mais fácil de entender:

```java
if (visitante.IsAdulto)
    return "Enjoy the content!";
else
    return "You have not permissions to see this content.";
```

# Funções/métodos

### Não usar diferentes níveis de abstração no mesmo método

Como assim, “diferentes níveis de abstração”? Simples, veja o exemplo abaixo.

```c#
var contractText = RemoveHtml(contract);
SendContractByEmail(contractText);
contractText = contractText.Replace(".", ";").Replace("-", ":");
```

O código chama 2 métodos diferentes, um que remove o HTML de uma string (o texto de um contrato, por exemplo) e outro que envia essa string (sem HTML, somente o texto) por e-mail.
Nenhum problema até aqui, mas a próxima linha (a 3ª) possui um nível de abstração diferente das demais. Ela trata uma string diretamente, podemos dizer que essa linha está mais “baixo nível” do que as demais. 

Diferentes níveis de abstração no mesmo método podem dificultar o entendimento dele. Quando se está visualizando um método com a abstração alta das funcionalidades é possível ter um entendimento da funcionalidade como um “todo”, sem entrar em um nível profundo nos detalhes da funcionalidade, em outras palavras, queremos saber o que esse método envia por e-mail mas não nos interessa saber que ele faz um replace do cáracter “-” para “:” (se quisermos saber esses detalhes entraremos nos métodos mais baixo nível para procurar).

Outras dicas rápidas:

### Menor quantidade de parâmetros possíveis no método

Eles dificultam a assinatura. Fazer o teste unitário de um método sem parâmetros pode ser simples, já de um método com um parâmetro nem tanto. Imagine um método com 4 parâmetros, é díficil de fazer um teste unitário que cubra uma grande combinação de possibilidades para esse método.

### Apenas uma responsabilidade por método

Exemplo: Um método chamado CheckPassword não deve validar um login ou iniciar uma sessão, e sim, como o nome diz, apenas informar se a senha é válida ou não.

### Evitar nomes de classes genéricos

Não usar nomes como: Processador ou Gerdenciador. Um nome genérico torna a classe muito abrangente e isso pode gerar confusão quanto a sua responsabilidade, levando os desenvolvedores a incluírem nela métodos que não deveriam estar ali. Já nomes específicos tendem a gerar classes menores e mais organizadas.

Espero que essas dicas tenham sido tão úteis pra você quanto foram pra mim. Aconselho fortemente você a comprar o livro caso deseje ir mais a fundo na saga de tornar um melhor desenvolvedor, lá tem muito mais coisas do que eu falei aqui e explicadas com mais detalhes e embasamento. Encerro esse post com uma linda quote do uncle Bob:

> “Tornar seu código legível é tão importante quanto torná-lo executável” 
>
> -- <cite>Robert Cecil Martin</cite>