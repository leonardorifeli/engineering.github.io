---
layout: post
title: 'Como fazemos parse dos textos para extração de inteligência?'
date: '2018-03-30 9:00:00'
categories: bigdata
comments: true
author: leonardorifeli
---

Em programação de software o termo **parse de texto** é ainda sombrio, ainda mais em grande quantidade. Aqui na Reviewr não é diferente.

Este é o primeiro post sobre o assunto aqui no blog. Falaremos sobre um desafio que enfrantamos: parse dos textos dos reviews e extração de inteligência.

Antes de tudo.

![image](https://user-images.githubusercontent.com/6767689/38067896-c067a4aa-32e4-11e8-9529-858351724902.png)

Aqui na Reviewr, prezamos muito pelo compartilhamento de conhecimento.

# Tópicos

Muito importante deixarmos claro os tópicos que iremos abordar neste post, para não termos problemas com as expectativas.

- O que é parsear?
- Introdução;
- Nossa arquitetura;
- Nossa stack;
- Performance;
- Conclusão.

# Tá, mas o que significa parsear?


![image](https://user-images.githubusercontent.com/6767689/38067620-ccc65b84-32e3-11e8-893b-8ce79053ad6c.png)

Fonte: [dicionarioinformal.com.br/parsear](http://www.dicionarioinformal.com.br/parsear/)

# Introdução

Para dar continuídade neste post, é muito importante você ter conhecimento sobre a Reviewr. [Aqui você encontra mais informações](http://reviewr.me/site/).

Em nossa plataforma temos uma feature que chamamos de **termos mais citados**. 

![image](https://user-images.githubusercontent.com/6767689/38068105-b1e9d028-32e5-11e8-89aa-341b2aeee22f.png)

Ou seja, dado um volume de Reviews, retornamos as palavras mais relevantes e mais citadas.

Para se ter mais ideia, nossa COO (Cecília Brandão) escreveu um post sobre o assunto **[como usar termos mais citados](http://reviewr.me/blog/como-usar-termos-mais-citados/)**.

Este foi um dos nossos grandes desafios. Salvar dados classificados, fazer parse de um grande volume de dados e entregar inteligência com estes dados.

# Nossa arquitetura

Nossa arquitetura é microservices e seguimos quase à risca os pontos abaixo:

- API First;
- [12 factors](https://12factor.net/pt_br/);
- Microservices;
- Clean architecture.

Vou isolar este assunto em dois subtópicos; **coletores** e **processadores**.

### Coletores

A arquitetura da plataforma é divida em duas partes principais, e uma delas, chamamores de coletores, onde nosso CTO (Marcelo Andrade) abordou neste post, sobre: [Princípios e estratégias de Web Scrapping](http://engineering.reviewr.tech/scrapping/2018/03/21/principios-e-estrategias-de-web-scrapping.html).

![image](https://user-images.githubusercontent.com/6767689/38068537-198df3ec-32e8-11e8-8be0-645effbc1a59.png)

Onde basicamente é feito a coleta dos reviews e são salvos em uma base MongoDB e em sequência, salvos em nosso Data Warehouse (Amazon Redshift).

Neste processo, já fazemos o pré-processamento dos textos, extraindo as palavras e claro, desconsiderando as palavras não necessárias atráves de uma blacklist.

### Processadores

Com base em tudo que falamos até aqui. Os processadores são microservices pequenos, onde externalizamos endpoints (uma API), tanto para relatórios quanto para extração de inteligência (um relatório é a junção de vários indicadores, dos quais este não tem a responsabilidade de extrair, solicitando para outro microservice efetuar essa extração).

Para prosseguirmos, analisaremos a seguinte imagem (nossa arquitetura de dados):

![image](https://user-images.githubusercontent.com/6767689/38068754-3484ea06-32e9-11e8-8d10-d58ac97ebd58.png)

É visível que nosso data warehouse é acessado por alguns microservices. Neste post, falaremos sobre o **Wolfram-Alpha**, responsável pelo parse de texto e extração dos termos mais citados.


# Nossa stack

Para o cenário atual, nossa stack se divide em:

- Banco NoSQL para os reviews em raw;
- Banco Postgres ([Redshift](https://aws.amazon.com/pt/redshift/));
- Golang no microservice.

Ou seja, nosso microservice é totalmente desenvolvimento em Go, onde ele se comunica somente com o Jarvis (apelido carinhoso para nosso data warehouse) para extrair os dados e iniciar o parse.

Claro, não entraremos em detalhes dos nossos algoritmos para o parse. Mas, por que utilizamos Go? Sobre isso que falaremos no próximo post.

# Performance

Neste tópico, trago alguns indicadores de performance atingidos no decorrer do desenvolvimento. Claro, em cenários não ideiais; como a aplicação em desktop, em um Macbook PRO e o banco na AWS (Vírginia).

Indicador sem cache, com parse de `~6k reviews`:

![image](https://user-images.githubusercontent.com/6767689/38069427-9e3b2ea8-32ec-11e8-94bf-bcc73079c992.png)


Indicador com cache, com parse de `~5k reviews`:

![image](https://user-images.githubusercontent.com/6767689/38069405-7babc47e-32ec-11e8-8f53-46f26bb92758.png)

Exemplo de resultado de uma requisitação:

![image](https://user-images.githubusercontent.com/6767689/38069453-daec5660-32ec-11e8-928d-545cb8e9a300.png)

![image](https://media1.tenor.com/images/41bdcebc51d47375cf5513194d0d6980/tenor.gif?itemid=4308764)

# Conclusão

Tivemos um longe desafio pela frente. No inicio fizemos a nossa POC com NodeJS e Go. Obtivemos excelente indicadores de performance com nosso algoritmo totalmente rodando com Golang.

Hoje este feature é muito utilizada em nossa plataforme e os indicadores continuam muito bem. Com base nesse microservice, escrevemos outros com Golang e pretendemos tornar uma stack default em nosso back-end.

Um abraço e até o próximo post.