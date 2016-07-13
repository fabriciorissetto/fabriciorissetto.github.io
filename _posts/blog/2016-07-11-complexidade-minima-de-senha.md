---
layout: post
title: "Pare de obrigar seus usuários a criarem senhas impossíveis de lembrar"
modified:
categories: blog
excerpt: ""
tags: [Segurança]
image:
  feature:
date: 2016-07-11T00:00:01-00:01
modified: 2016-07-11T00:00:01-00:01
---

<div style="text-align:center">
  <img style="text-align: center;" src="{{ site.url }}/images/complexidade-senha/usuario-confuso.png">
</div>

É isso mesmo, pare de obrigar seus usuários a criarem senhas com complexidades mirabolantes! 

Fazendo isso você:

* Não garante que a senha será segura
* Praticamente garante que ele irá esquecê-la

Durante a escrita desse post resolvi tentar criar uma nova conta no facebook com uma senha comum. Enquanto eu digitava a senha ele não me informou em momento algum sobre critérios mínimos de força. Então tentei a senha "**fabricio**". Submeti e... POWWW! *Sua conta foi criada com sucesso!!!* 

Mais uma rápida pesquisada e descobri esse [site](http://www.passwordpit.com/facebook-password-requirements/),   que informa algumas coisas sobre os requerimentos de senha do facebook. Vou reproduzir os principais aqui:

* **Tamanho mínimo**: 6 caracteres;
* **Tamanho máximo**: aparentemente não há limite de tamanho (foi testado até 500 characteres e não houve problema);
* **Complexidade mínima:** não há requerimentos especiais de complexidade. 
 
Mas vamos lá, não se sinta mal, eu também já cobrei força de senha de meus usuários e não foi só uma vez. A questão é: Qual seu objetivo ao cobrar isso de seus usuários?

## Rainbow Tables

O ataque [rainbow tables](http://stackoverflow.com/a/1012793/890890) ocorre normalmente quando a base de dados que armazena as senhas dos usuários é comprometida e vazada para as mãos erradas, [como o que aconteceu com o LinkedIn em 2012](https://www.troyhunt.com/observations-and-thoughts-on-the-linkedin-data-breach/). No rainbow tables o atacante busca **descobrir a senha do usuário em *plain text*** e não necessariamente o acesso a aplicação (embora, óbvio, possa ser usada para isso).

Nesse famoso caso que, diga-se de passagem, pegou muito mal pro LinkedIn as senhas de vários usuários foram descobertas por dois principais motivos:

1. O algoritmo de hash das senhas era o SHA1 (o SHA1 é atualmente o mais utilizado na Web, porém está sendo cada vez mais [desencorajado](https://konklone.com/post/why-google-is-hurrying-the-web-to-kill-sha-1)).
2. Eles não usaram a técnica de [Salt](https://crackstation.net/hashing-security.htm). Isso daqui foi, sem dúvida, o amadorismo mais gritante de todo o acontecimento. 

Então eu digo: se você está obrigando seu usuário a criar senhas complexas para se defender desse tipo de ataque, você está indo pelo caminho errado.

Se esse é o objetivo, dê preferência a utilizar um algoritmo de hash melhor (talvez um SHA-2) e, claro, persistir esse hash *"salteado"*.

## Brute Force 

Talvez você esteja tentando tornar seu software menos suscetível a um ataque de força bruta. Observe que se estamos falando de um sistema web a própria latência intrínseca da conexão HTTP já dificulta gigantescamente (e talvez até inviabilize) um ataque de força bruta. Mas ok, de fato a possibilidade não deve ser descartada. 

Por isso pedimos para nossos usuários digitarem senhas "mais difíceis", pedindo pra eles utilizarem **caracteres especiais, números, letras maiúsculas e minúsculas**. Pois assim dificultamos o ataque, certo? Não necessariamente.

Por exemplo, para atender a todos os requisitos acima eu poderia simplesmente utilizar a senha `F@brici0`.

O site [howsecureismypassword](https://howsecureismypassword.net/) mostra (certamente com algum nível de imprecisão) quanto tempo seria necessário para quebrar determinada senha.

Observe quanto tempo supostamente seria necessário para quebrar a senha `F@brici0`:

![Usuário]({{ site.url }}/images/complexidade-senha/password_fabricio.png)

Agora vamos ver quanto tempo levaria para quebrar a senha `fabricio lalala`:

![Usuário]({{ site.url }}/images/complexidade-senha/password_fabricio_lalala.png)

O que?! 5 milhões de anos? Mas ela não é mais simples? Não, pois ela é maior!

Isso mesmo, [o tamanho torna a senha mais segura do que a complexidade](http://crambler.com/password-security-why-secure-passwords-need-length-over-complexity/). Quanto menor a senha, menor o número de combinações possíveis para um ataque de força bruta. Portanto, quando obrigamos os usuários a escolherem senhas complexas não estamos tornando as senhas mais seguras. Na verdade o efeito é justamente o contrário, quando obrigados a escolherem senhas mais complexas os usuários tendem a criar senhas mais curtas (como `F@brici0`).

> OBS: Há diversas outras variáveis envolvidas quando falamos de segurança de senhas como o   nível de predictibilidade/aleatoriedade de tal.

O fato é que há diversas maneiras mais efetivas de se defender desse tipo de ataque sem precisar importunar seus usuários. A lista é longa e as soluções variam desde o uso de captchas até *account lockouts*. 

# Conclusão

O objetivo aqui não é ensinar os meios de se defender desses ataques. Mas sim deixar claro que simplesmente exigir uma maior complexidade nas senhas não é o caminho :)