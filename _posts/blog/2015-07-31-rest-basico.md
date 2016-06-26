---
layout: post
title: "API REST Básico – Rotas, Verbos e Status Codes"
modified:
categories: blog
excerpt: ""
tags: [rest]
image:
  feature:
date: 2015-07-31T15:39:55-04:00
---

# GET

Utilizado para consultas. *Se modifica algum recurso no servidor, como banco de dados, envio de e-mail, sessão, etc, não deve ser GET!*

**Status Codes normalmente retornados**

* 200 (OK)
* 404 (Not Found)

<div class="spacing"></div>

| ROTAS | RESULTADO |
|:---|:---|
| /usuarios/2 | retorna o usuário com ID igual a 2 |
| /usuarios/2/telefones	| retorna os telefones do usuário com ID igual a 2 |
| /usuarios	| retorna todos usuários |
| /usuarios?ativo=true&lastName=Silva | retorna uma lista de usuários ativos com o sobrenome “Silva” |

Observe que o ID principal do recurso é enviado como parte do path (/usuários/{id}), porém para consulta que não é feita através do identificador da entidade os parâmetros de consulta devem ser enviados via querystring (/usuarios?ativo=true&lastName=Silva).

---

# POST

Cria recursos no servidor.​

**Status Codes normalmente retornados**

* 201 (Created)
* 409 (Conflict) se o recurso já existe

<div class="spacing"></div>

| ROTAS | RESULTADO |
|:---|:---|
| /usuarios | cria um novo usuário (conteúdo enviado no body da request) |

---

# PUT

Atualiza recursos no servidor.

**Status Codes normalmente retornados**

* 200 (OK)
* 404 (Not Found)


<div class="spacing"></div>

| ROTAS | RESULTADO |
|:---|:---|
| /usuarios/2 | atualiza o usuário com ID igual a 2 (o ID é enviado na URL mas as informações do usuário em si são enviadas no body da request)|

---

# PATCH

Há momentos em que é necessário a atualização parcial de um recurso, como por exemplo, atualizar apenas o “status” de um cliente. Em casos como esse o verbo mais apropriado seria o PATCH.

**Status Codes normalmente retornados**

* 200 (OK)
* 404 (Not Found)

*A implementação do PATCH varia muito e não há um “padrão único” a ser seguido, um exemplo poderia ser esse:

| ROTAS | RESULTADO |
|:---|:---|
| /usuarios/2 | atualiza o status do usuário cujo ID é igual a 2 (nesse caso o corpo da requisição teria apenas uma propriedade, exemplo: “Status: Inativo”) |
 
---

# DELETE

Remove recursos do servidor

**Status Codes normalmente retornados**

* 200 (OK)
* 404 (Not Found)

<div class="spacing"></div>

| ROTAS | RESULTADO |
|:---|:---|
| /usuarios/2 | remove o usuário com ID igual a 2 |
| /usuarios/2/telefones/1 | remove apenas o telefone com ID “1″ do usuário com ID igual a “2″ |
 
Para casos menos curriqueiros a vezes surge a necessidade de se utilizar status codes mais específicos. Uma regra de ouro que facilita na busca de status codes mais apropriados é que os 4xx são erros do cliente e os 5xx são erros do servidor.