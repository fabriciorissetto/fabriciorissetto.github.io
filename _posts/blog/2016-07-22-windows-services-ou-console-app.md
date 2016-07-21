---
layout: post
title: "Windows Services ou Console APP + Task Scheduler"
modified:
categories: blog
excerpt: ""
tags: [.NET]
image:
  feature:
date: 2016-07-22T15:39:55-04:01
---

Algo que frequentemente vejo no ambiente .NET é a utilização de Windows Services para qualquer tarefa que deva ser executada com recorrência. O que em muitos casos é um desperdício de esforço, pois poderiam ser utilizadas **Console Applications** scheduladas no Windows.

![Task Scheduler]({{ site.url }}/images/windows-services-ou-task-scheduler/task-scheduler.png)

[Como agendar uma tarefa no Windows com Task Scheduler?](http://atendimento.redehost.com.br/hc/pt-br/articles/210816987-Agendando-tarefas-no-Windows-Server-2012)

Minha *rule of thumb* é: 

> Se você não **precisa** usar Windows Service, use Task Scheduler.

### Windows Service

Para tarefas muito recorrentes. Exemplo: uma task que deve ficar infinitamente realizando um processo como chamar uma API pra obter alguma informação. Ou uma task com uma recorrência muito baixa, como de 1 em 1 minuto. Nesse caso, se for usar Windows Service com um "timer" (que roda de tempos em tempos) não faça seu próprio Scheduler "na mão". Utilize frameworks como o [Quarts.NET](http://www.quartz-scheduler.net).

Windows Services no geral são mais poderosos, uma feature que gosto muito é a de poder sobrescerver os eventos de "Start", "Pause" e "Stop". Por exemplo uma aplicação que fique infinitamente (`while (true) { }`) fazendo determinados processos (cálculos, chamdas de APIs, etc), mas que você queira ter a possibilidade de interrmpê-lo *sem que isso seja feito de forma abrupta*. Nesse caso, você pode dar override no evento de "Stop" do Windows Service e criar uma lógica para informá-lo que após a conclusão do processo corrente ele não deve inciar um novo, e sim dar exit.

### Console Applicatin + Task Scheduler

Para tarefas não tão recorrentes (como um processo que deve rodar uma vez por dia ou toda terça-feira às 17 horas, por exemplo).

É simples de criar e manter. Possui temporizador *built in* (com user interface).

Uma vez tendo uma console application, migrar pra Windows Service não é uma tarefa difícil, principalmente se utilizarmos ferramentas como o [TopShelf](http://topshelf-project.com).
Outro ponto que muitas pessoas desconhecem é a possibilidade de configurar o Task Scheduler pra rodar a [Console App em uma janela invisível](http://www.howtogeek.com/tips/how-to-run-a-scheduled-task-without-a-command-window-appearing) (completamente em background).
