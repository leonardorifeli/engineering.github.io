---
layout: post
title: 'Como fazemos parse dos textos para extração de inteligência?'
date: '2018-03-30 9:00:00'
categories: bigdata
comments: true
author: leonardorifeli
---

---
layout: post
title: 'Como fazemos parse dos textos para extração de inteligência?'
date: '2018-03-30 9:00:00'
categories: bigdata
comments: true
author: leonardorifeli
---

Em programação de software o termo **parse de texto** é ainda sombrio, ainda mais em demasiado volume de texto.

Este é o primeiro post sobre o assunto aqui no blog. Falaremos sobre um desafio que enfrantamos: parse dos textos dos reviews e extração de inteligência. Então, vamos lá.

![image](https://user-images.githubusercontent.com/6767689/38067896-c067a4aa-32e4-11e8-9529-858351724902.png)

Aqui na Reviewr, prezamos muito pelo compartilhamento de conhecimento.

# Tópicos

Para não termos problemas com as expectativas é muito importante deixarmos claro os tópicos que iremos abordar neste post.

- O que é parsear?
- Introdução;
- Nossa arquitetura;
- Nossa stack;
- Como fizemos;
- Performance;
- Conclusão.

# Tá, mas o que significa parsear?

![image](https://user-images.githubusercontent.com/6767689/38067620-ccc65b84-32e3-11e8-893b-8ce79053ad6c.png)

Fonte: [dicionarioinformal.com.br/parsear](http://www.dicionarioinformal.com.br/parsear/)

# Introdução

Para dar continuídade, é muito importante você ter conhecimento sobre a Reviewr. [Aqui você encontra mais informações](http://reviewr.me/site/).

Em nossa plataforma temos uma feature com o nome de **termos mais citados**. 

![image](https://user-images.githubusercontent.com/6767689/38068105-b1e9d028-32e5-11e8-89aa-341b2aeee22f.png)

Ou seja, dado um volume de Reviews, retornamos as palavras mais citadas, neste caso mais relevantes.

Sobre isso, nossa COO (Cecília Brandão) escreveu um post sobre o assunto **[como usar termos mais citados](http://reviewr.me/blog/como-usar-termos-mais-citados/)**.

Este foi um dos nossos grandes desafios. Salvar dados classificados, fazer parse de um grande volume de dados e entregar inteligência.


# Nossa arquitetura

Nossa arquitetura é com base em microservices e seguimos (sempre que possível) os pontos abaixo:

- API First;
- [12 factors](https://12factor.net/pt_br/);
- Microservices;
- Clean architecture.

Para descrevermos a arquitetura que utilizamos para resolver este desafio, irei isolar este assunto em dois subtópicos: **coletores** e **processadores**.

### Coletores

A arquitetura da nossa plataforma é divida em duas partes principais, e uma delas chamamores de coletores. Nosso CTO (Marcelo Andrade) fala um pouco dos coletores neste post; [Princípios e estratégias de Web Scrapping](http://engineering.reviewr.tech/scrapping/2018/03/21/principios-e-estrategias-de-web-scrapping.html).

![image](https://user-images.githubusercontent.com/6767689/38068537-198df3ec-32e8-11e8-8be0-645effbc1a59.png)

Onde é feita toda a coleta dos reviews, em seguida são inseridos em uma base MongoDB e em nosso Data Warehouse (Amazon Redshift, um postgres).

Neste processo, já fazemos o pré-processamento dos textos, extraindo as palavras e claro, desconsiderando as palavras não necessárias atráves de uma blacklist.

### Processadores

Os processadores são microservices pequenos, onde externalizamos endpoints (uma API), tanto para relatórios quanto para extração de inteligência (um relatório é a junção de vários indicadores, dos quais este não tem a responsabilidade de extrair, solicitando para outro microservice efetuar essa extração).

Para prosseguirmos, analisaremos a seguinte imagem (nossa arquitetura de dados):

![image](https://user-images.githubusercontent.com/6767689/38068754-3484ea06-32e9-11e8-8d10-d58ac97ebd58.png)

É visível que nosso data warehouse é acessado por alguns microservices. Neste post, falaremos sobre o **Kowalski** e o **Wolfram-Alpha**, responsáveis por relatórios, pelo parse de texto para extração dos termos mais citados.

# Nossa stack

Para o cenário atual, nossa stack é dividida em:

- Banco NoSQL para os reviews em raw-data;
- Banco Postgres ([Redshift](https://aws.amazon.com/pt/redshift/));
- NodeJS (no Kowalski) e Golang (no Wolfram-alpha) microservice.

# Nossos microservices

Bom, como visto, temos dois microservices principais para o processo dos relatórios e extração de inteligência. O microservice **Kowalski** é responsável pela entrega dos relatórios e apenas por alguns indicadores, este é feito totalmente em NodeJS.

Já o **Wolfram-Alpha** é feito em Go (foi nosso primeiro microservice em Go). É responsável por processar um volume **X** de reviews para a extração, agregação e ordenação dos termos. Aqui utilizamos um processo de **MapReduce**. Golang possui um grande recurso para programação paralela, por isso a escolhemos para esta tarefa.

# Como fazemos o parse

Como visto na imagem sobre nossa arquitetura de dados. Até então, o **Kowalski** efetua uma request para um endpoint **XYZ** do **Wolfram-Alpha**, passando algunas informações para a extracão, tais como: período de data (data `de` e `até`), o cliente, o canal (Todos, Facebook, Google, etc), etc.

Com base nessas informações, fazemos as extrações dos reviews em nosso data warehouse. Como já temos os termos pré-processados e salvos em json (como o exemplo abaixo) precisamos reprocessa-los junto ao volume todo.

**Exemplo do json de termos pré-processados**

```json
{  
   "melhor":1,
   "shopping":1,
   "região":1,
   "peca":1,
   "questão":1,
   "estacionamento":1,
   "acho":1,
   "bem":1,
   "caro":1
}
```
Fazemos parse de todos os termos de cada review e vamos populando um `array` com os termos, incrementando a cada ocorrência. Posteriormente a isso, fazemos a orderação de ocorrências de modo decrescente. Assim, temos uma lista única de todos os termos mais citados em um volume de reviews. Claro, no parse dos termos, analisamos novamente a blacklist para disconsiderar termos não necessários, que foram processados após uma possível atualização do nosso blacklist (fica pesado refazermos os termos que já foram pré-processados).

Aí você questiona: qual a performance para analisar tudo isso? Chega mais no próximo tópico.

# Performance

Neste tópico, trago alguns indicadores de performance atingidos no decorrer do desenvolvimento. Claro, em cenários não ideiais; como a aplicação em desktop, em um Macbook PRO e o banco na AWS (Vírginia).

Indicador sem cache, com parse de `~6k reviews`:

![image](https://user-images.githubusercontent.com/6767689/38069427-9e3b2ea8-32ec-11e8-94bf-bcc73079c992.png)


Indicador com cache, com parse de `~5k reviews`:

![image](https://user-images.githubusercontent.com/6767689/38069405-7babc47e-32ec-11e8-8f53-46f26bb92758.png)

Exemplo de resultado de uma requisitação:

```json
{
   "terms":[  
      {  
         "name":"shopping",
         "total":948
      },
      {  
         "name":"lojas",
         "total":693
      },
      {  
         "name":"estacionamento",
         "total":565
      },
      {  
         "name":"bom",
         "total":537
      },
      {  
         "name":"alimentação",
         "total":448
      },
      {  
         "name":"praça",
         "total":398
      },
      {  
         "name":"ótimo",
         "total":284
      },
      {  
         "name":"lugar",
         "total":269
      },
      {  
         "name":"cinema",
         "total":251
      },
      {  
         "name":"opções",
         "total":250
      },
      {  
         "name":"grande",
         "total":214
      },
      {  
         "name":"excelente",
         "total":193
      },
      {  
         "name":"melhor",
         "total":190
      },
      {  
         "name":"compras",
         "total":163
      },
      {  
         "name":"caro",
         "total":160
      },
   ]
}
```

![image](https://media1.tenor.com/images/41bdcebc51d47375cf5513194d0d6980/tenor.gif?itemid=4308764)

# Conclusão

Tivemos um longe desafio pela frente. No inicio fizemos a nossa POC com NodeJS e Go. Obtivemos excelente indicadores de performance com nosso algoritmo totalmente rodando com Golang.

Hoje este feature é muito utilizada em nossa plataforme e os indicadores continuam muito bem. Com base nesse microservice, escrevemos outros com Golang e pretendemos tornar uma stack default em nosso back-end.

Um abraço e até o próximo post.