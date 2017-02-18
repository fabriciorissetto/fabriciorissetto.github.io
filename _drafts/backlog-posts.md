POST BLOG:

# Porque você precisa de CQRS na sua aplicação?
Post simples e ao mesmo tempo ótimo.

# Microservices VS ESBs
https://googleweblight.com/?lite_url=https://www.voxxed.com/blog/2015/01/good-microservices-architectures-death-enterprise-service-bus-part-one

Ótimo post: http://openstack.sys-con.com/node/3334106
Martin Fowler: http://martinfowler.com/articles/microservices.html

Componentization via Services
Organized around Business Capabilities - fazer analogia com pessoas especialistas  (nerdzinho de oculos - DBA, Josicleison), cara barbudo (programador Adão), mulher numa sala colorida (designer Janaina) vs agentes especiais (em que cada um sabe tudo mas tem uma especialidade em algo) - foto modafucker de agentes da CIA

Products not Projects
Smart endpoints and dumb pipes
Decentralized Governance
Decentralized Data Management
Infrastructure Automation
Design for failure
Evolutionary Design

Microservices is an evolutionary paradigm born out of the need for simplicity (i.e., get away from the ESB) and alignment with agile (think DevOps) and scalable (think Containerization) development and deployment architectures.

Common mistakes microservices
    ESBs resolviam requisitos não funcionais como "Autenticação, monitoramento, logs centaliazdos, mediação"
-> Com microservices esses requisitos não funcionais muitas vezes são negligenciados.
-> Documentação dos serviços existentes (onde encontrar?)
-> Monitoramento. Os serviços estão distribuídos, ótimo! Mas qual a topologia de comunicação entre eles? Quantas requests por dia pro serviço X? Quantos erros no serviço Y?
-> Log de erros individuais e centralizados

Microservice por necessidade

# Web API vs WCF em ambiente corporativo

# Service Discovery: Uma necessidade para microservices

# A invasão do paradigma funcional (como isso está vazando para linguagens como C#)

# Programador Fashionweek (Falar sobre os devs gostarem de coisa novas e que �mais facil de contatar eles com isso)

# Programação assíncrona 
https://msdn.microsoft.com/en-us/magazine/jj991977.aspx
(e depois de estudar responder essa pergunta: http://stackoverflow.com/questions/38150804/how-can-i-implement-async-methods-in-n-layer-or-ddd-net)

# Database as a queue anti pattern
http://mikehadlow.blogspot.com.br/2012/04/database-as-queue-anti-pattern.html
Contra ponto: https://blog.jooq.org/2014/09/26/using-your-rdbms-for-messaging-is-totally-ok/
