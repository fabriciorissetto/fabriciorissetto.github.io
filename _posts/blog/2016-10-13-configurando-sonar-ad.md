---
layout: post
title: "Integrando a autenticação do Sonar Qube com o AD"
modified:
categories: blog
excerpt: ""
tags: [LDAP]
image:
  feature:
date: 2016-10-13T00:58:56-02:00
---

O objetivo do post é faciliar a configuração do [Sonar Qube](https://www.sonarqube.org/) com o Active Directory para quem não estiver familizarizado com o protocolo LDAP. 

Diferente de ferramentas como o Jenkins, Team City e Octopus, a integração do Sonar Qube com o AD é feita de maneira bem artezanal e a configuração varia conforme o ambiente de cada organização.

Pra faciliar iremos utilizar a seguinte ferramenta (que é free):

* [LDAP Browser](http://www.ldapbrowser.com/download.htm).

## Sonar Qube

Faça o download do .jar do [plugin](https://docs.sonarqube.org/display/PLUG/LDAP+Plugin) e cole na pasta: 

> SONARQUBE_HOME/extensions/plugins/

Depois abra o arquivo:

> SONARQUBE_HOME/conf/sonar.properties

E nele adicione as seguintes linhas:

```shell
# CTRL C + CTRL V
sonar.security.realm=LDAP
sonar.security.savePassword=true

# A partir daqui tudo vai ser diferente conforme o AD da sua empresa
ldap.url=ldap://yourldapurl:389
ldap.bindDn=CN=fabriciorissetto,OU=OU=ACCOUNTS,DC=EMPRESA,DC=COM
ldap.bindPassword=senhaDoUsuarioAcima

ldap.user.baseDn=DC=EMPRESA,DC=COM
ldap.user.request=(&(objectClass=user)(sAMAccountName={login}))
```

O primeiro campo é o que ativa a autenticação via LDAP. O segundo (`sonar.security.savePassword`), conforme a [documentação](https://docs.sonarqube.org/display/PLUG/LDAP+Plugin) serve para:

> To save the user password in the SonarQube database. Then, users will be able to log into SonarQube even when the LDAP server is not reachable.

Os demais campos variam de configuração para configuração. Teremos que descobrir os valores corretos pro seu AD. Para facilitar isso iremos utilizar o LDAP Browser.

## LDAP Browser

Instale o LDAP Browser dentro de uma máquina que esteja no domínio do Active Directory. 

Abra ele e crie um novo profiler.

Na configuração do host, clique em "Lookup Servers..." e copie um dos servidores (algo como `YOURCOMPANY.COM:389` - pode existir mais de um) que aparecerem na listagem conforme a imagem abaixo e cole no campo "Host". *OBS: guarde esse domínio que você copiou, ele vai ser útil logo a frente.* 

![Select Host]({{ site.url }}/images/2016-10-13-configurando-sonar-ad/1.png)

No próximo passo, pra facilitar, selecione a opção "Currently logged on user" (OBS: é necessário que você esteja logado na máquina com um usuário do AD).


![Current Logged User]({{ site.url }}/images/2016-10-13-configurando-sonar-ad/2.png)

Após isso clique em "Finish".

Clique no profile que você criou e em seguida vá para o campo de pesquisa (CTRL + F3). Nesse campo digite o login (na verdade pode ser nome, email, entre outros campos) do usuário que você deseja encontrar. Exemplo:

![Pesquisa por usuário]({{ site.url }}/images/2016-10-13-configurando-sonar-ad/3.png)

No resultado, selecione o usuário que você encontrou para ver seus atributos de forma detalhada:

![Detalhes usuário]({{ site.url }}/images/2016-10-13-configurando-sonar-ad/4.png)

Esses são os atributos do usuário no AD! Vamos utilizar alguns deles pra configurar o Sonar Qube.

### Extraindo informações do AD para o arquivo sonar.properties

#### ldap.url
Cole aqui o domínio (com a porta, que normalmente é 389) que você copiou na janela de configuração do profile no LDAP Browser. Exemplo: 

> ldap.url=ldap://YOURCOMPANY.COM:389

#### ldap.bindDn

No AD, o campo `distinguishedName` representa um valor único para o registro (usuário, grupo, etc). Em uma analogia simples, é como se ele fosse um *namespace* praquele objeto. Podemos também fazer uma analogia com um *DNS*, por exemplo, o valor `CN=cwi.fabriciosilva,OU=CWI,OU=EXTERNAL,OU=ACCOUNTS,DC=COM,DC=BR` em formato de DNS representaria o site: `http://cwi.fabriciosilva.cwi.external.accounts.com.br`. *Esse [post](http://stackoverflow.com/a/33961313/890890) no stackoverflow é bem esclarecedor.*

Copie o valor desse campo e cole no nosso arquivo de configuração do Sonar, na chave `ldap.bindDn`. Exemplo:

> ldap.bindDn=CN=cwi.fabriciosilva,OU=CWI,OU=EXTERNAL,OU=ACCOUNTS,DC=COM,DC=BR

*Esse será o usuário que o Sonar Qube utilizará para se conectar no AD e verificar as credenciais de quem tenta se logar nele.*

#### ldap.bindPassword

No campo `ldap.bindPassword` coloque a senha desse usuário (você está logado na máquina com ele, esperamos que saiba a senha :P). Infelizmente a senha fica "aberta" nesse arquivo sem criptografia alguma. 

> ldap.bindPassword=senhaDoUsuario

#### ldap.user.baseDn

Voltando à analogia do *DNS*, nesse campo aqui iremos colocar o "DNS raiz" dos usuários que queremos permitir logarem no Sonar Qube. Por exemplo, os DNS's `trends.google.com` e `drive.google.com` estão dentro do DNS `google.com`. O mesmo vale praquela string maluca do LDAP. Portanto, no exemplo do distinguished name `CN=cwi.fabriciosilva,OU=CWI,OU=EXTERNAL,OU=ACCOUNTS,DC=COM,DC=BR` posso extrair apenas o final desse "namespace" e colocar no campo `ldap.user.baseDn`:

> ldap.user.baseDn=DC=COM,DC=BR

Isso quer dizer que todos usuários que possuam `DC=COM,DC=BR` no final do seu distinguished name poderão se logar no Sonar.

#### ldap.user.request

O último campo necessário pra configurar aqui é o `ldap.user.request`. Provavelmente esse valor aqui funcionará pra você:

> ldap.user.request=(&(objectClass=user)(sAMAccountName={login}))

Basicamente esse filtro é a query LDAP que será feita pelo sonar para determinar se o usuário faz parte do AD. Ele apenas substituirá a chave `{login}` pelo campo que foi digitado no formulário de login do Sonar. O campo [sAMAccountName](https://msdn.microsoft.com/en-us/library/ms679635(v=vs.85).aspx) se refere ao login no Windows. Caso queira verificar por algum outro campo a princípio basta substituir esse valor pelo desejado (não cheguei a testar).

### Conclusão

Com isso seu arquivo de configuração ficará próximo disso:

```shell
sonar.security.realm=LDAP
sonar.security.savePassword=true

ldap.url=ldap://YOURCOMPANY.COM:389
ldap.bindDn=CN=cwi.fabriciosilva,OU=CWI,OU=EXTERNAL,OU=ACCOUNTS,DC=COM,DC=BR
ldap.bindPassword=senhaDoUsuario

ldap.user.baseDn=DC=COM,DC=BR
ldap.user.request=(&(objectClass=user)(sAMAccountName={login}))

```

Não cheguei a precisar fazer a configuração baseada em grupos do AD, mas pelo que pesquisei a lógica é a mesma:

```
ldap.group.baseDn=OU=GRUPO ESPECIAL,DC=COM,DC=BR
ldap.group.request=(&(objectClass=group)(memberUid={uid}))
```

Onde `ldap.group.baseDn` é o "namespace" base do grupo e `ldap.group.request` é a query utilizada pelo Sonar para determinar se o usuário pertence ao grupo.