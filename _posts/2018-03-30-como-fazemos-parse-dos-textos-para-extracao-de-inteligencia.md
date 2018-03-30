---
layout: post
title: 'Como fazemos parse dos textos para extração de inteligência?'
date: '2018-03-30 9:00:00'
categories: bigdata
comments: true
author: leonardorifeli
---

Em programação de software, o termo **parse de texto** continua sombrio, principalmente considerando um alto volume de dados. Este é o primeiro post sobre o assunto aqui no blog e falarei sobre um desafio que enfrentamos: parse dos textos dos reviews e extração de inteligência. 

Antes de começar:

![image](https://user-images.githubusercontent.com/6767689/38067896-c067a4aa-32e4-11e8-9529-858351724902.png)

Aqui na Reviewr, prezamos muito pelo compartilhamento de conhecimento.

Vamos lá?

# Tópicos

Para não termos problemas com as expectativas é importante deixarmos claro os tópicos que iremos abordar neste post.

- O que é *parsear*?
- Introdução;
- Nossa arquitetura;
- Nossa stack;
- Nossos microserviços;
- Como fazemos o parse;
- Performance;
- Conclusão.

# Tá, mas o que significa parsear?

![image](https://user-images.githubusercontent.com/6767689/38067620-ccc65b84-32e3-11e8-893b-8ce79053ad6c.png)a

Fonte: [dicionarioinformal.com.br/parsear](http://www.dicionarioinformal.com.br/parsear/)

# Introdução

Para melhor compreensão, é importante você ter conhecimento sobre o que fazemos aqui na Reviewr. [Aqui você encontra mais informações](http://reviewr.me/site/).

Em nossa plataforma temos uma *feature* chamada **termos mais citados**. 

![image](https://user-images.githubusercontent.com/6767689/38068105-b1e9d028-32e5-11e8-89aa-341b2aeee22f.png)

Dada uma massa de reviews retornamos as palavras com maior número de ocorrências, neste caso, mais relevantes.

Este foi um dos nossos grandes desafios. Salvar dados classificados, fazer parse de um grande volume de dados e entregar inteligência.

Você pode obter mais informações sobre essa funcionalidade no post **[Como usar os termos mais citados](http://reviewr.me/blog/como-usar-termos-mais-citados/)** da nossa COO Cecília Brandão.

# Nossa arquitetura

Aqui na Reviewr, utlizamos a arquitetura de microserviços e seguimos (sempre que possível) os pontos abaixo:

- API First;
- [12 factors](https://12factor.net/pt_br/);
- Single Responsability;
- Clean architecture.

Para uma melhor compreensão das nossas escolhas de arquitetura, vou isolar este assunto em dois subtópicos: **coletores** e **processadores**.

### Coletores

A arquitetura da nossa plataforma é dividida em duas partes principais, uma delas é a de coleta de reviews. Nosso CTO (Marcelo Andrade) fala um pouco dos coletores neste post: [Princípios e estratégias de Web Scrapping](http://engineering.reviewr.tech/scrapping/2018/03/21/principios-e-estrategias-de-web-scrapping.html).

![image](https://user-images.githubusercontent.com/6767689/38068537-198df3ec-32e8-11e8-8be0-645effbc1a59.png)

Nossos coletores extraem os reviews das plataformas integradas e em seguida insere em uma base `MongoDB`. Posteriormente, esses dados são agregados em nosso `Data Warehouse` (Amazon Redshift/Postgres).

Neste processo é feito um pré-processamento dos textos, extraindo os termos e desconsiderando as palavras pré definidas atráves de uma *blacklist*. Tudo isso é salvo de maneira estruturada para que fique fácil e dinâmico a forma de análise desses dados.

### Processadores

São resposáveis por extrair inteligência de toda essa massa de reviews coletada, e cada API tem uma única responsabilidade. Esses microserviços costumam solicitar para outros que processem algo que este não tenha a responsabilidade de resolver. 

Exemplificando: um relatório é a consolidação de várias métricas. O microserviço responsável por gerar o relatório pode precisar de métricas que são geradas por outros microserviços. Ao invés de replicar a lógica de extração da métrica, esse microserviço solicita para outro que processe e retorne para ele o resultado.

Para prosseguirmos, analisaremos a seguinte imagem (nossa arquitetura de dados):

![image](https://user-images.githubusercontent.com/6767689/38068754-3484ea06-32e9-11e8-8d10-d58ac97ebd58.png)

Nosso Data Warehouse é acessado por vários microserviços e falarei um pouco abaixo sobre dois: o **Kowalski** e o **Wolfram-Alpha**, respectivamente responsáveis por geração de relatórios e parse de texto dos termos mais citados.

# Nossa stack

Para o cenário atual, nossa stack é dividida em:

- Banco `NoSQL` (neste caso `MongoDB`) para os reviews em `raw-data `;
- Banco `Postgres` ([Redshift](https://aws.amazon.com/pt/redshift/));
- NodeJS (`Kowalski`) e Golang (`Wolfram-alpha`) microservice.

# Nossos microserviços

O **Kowalski**, microserviço feito com `NodeJS`, é responsável pela entrega dos relatórios e alguns indicadores.

O **Wolfram-Alpha**, nosso primeiro microserviço em `golang`, é responsável por processar um volume **XY** de reviews para a extração, agregação e ordenação dos termos. Para isso, utilizamos um processo de **MapReduce** e o `golang` possui uma infinidade de recursos para processamento de dados de maneira paralela. Isso faz com que a perfomance da aplicação aumente drásticamente e foi o fator chave que nos fez escolher a linguagem para realizar essa tarefa.

# Como fazemos o parse

O **Kowalski** efetua uma request para um endpoint **XYZ** do **Wolfram-Alpha**, passando algunas informações para a extracão, tais como: `range de data`, `cliente`, `canal` (Todos, Facebook, Google, etc), etc.

Com base nessas informações, fazemos as extrações dos reviews em nosso `data warehouse`. Como já temos os termos pré-processados e salvos em json (como o exemplo abaixo) precisamos reprocessa-los junto a todo o volume de texto.

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

Fazemos parse de todos os termos de cada review e populamos um `array`, incrementando as ocorrências. Posteriormente, fazemos a ordernação de ocorrências de modo decrescente. Assim, temos uma lista única de todos os termos mais citados de um determinado volume de reviews. No parse dos termos, analisamos novamente a blacklist para desconsiderar termos desnecessários, que foram processados antes de alguma atualização da nossa blacklist.

Tá, mas como fica a performance disso tudo? Chega mais no próximo tópico.

`P.S.:` Estamos evoluindo para análise de sinônimos, pluralidade, e alguns indicadores para entregarmos ainda mais inteligência.

# Performance

Neste tópico, trago alguns indicadores de performance atingidos no decorrer do desenvolvimento. 

*Esses testes foram realizados rodando a aplicação em ambiente local e o banco na AWS (na N. Virgínia).*

**Indicador sem cache, com parse de `~6k reviews`:**

![image](https://user-images.githubusercontent.com/6767689/38069427-9e3b2ea8-32ec-11e8-94bf-bcc73079c992.png)

**Indicador com cache, com parse de `~6k reviews`:**

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
         "name":"alimentação",
         "total":448
      },
      {  
         "name":"praça",
         "total":398
      },
      {  
         "name":"cinema",
         "total":251
      },
      {  
         "name":"excelente",
         "total":193
      }
   ]
}
```

![image](https://media1.tenor.com/images/41bdcebc51d47375cf5513194d0d6980/tenor.gif?itemid=4308764)

# Conclusão

Tivemos um longe desafio pela frente. Fizemos duas POCs e obtivemos indicadores de performance esmagadores quando comparamos o algoritmo feito em `golang` vs `NodeJS`.

Hoje esta feature é muito utilizada em nossa plataforma e os indicadores continuam muito satisfatórios. Com base nesse microserviço, escrevemos outros com `golang` e pretendemos tornar uma stack default em nosso back-end.

Se você possui algum case como este, fique a vontade para compartilhar conosco! Abraço e até o nosso próximo post.