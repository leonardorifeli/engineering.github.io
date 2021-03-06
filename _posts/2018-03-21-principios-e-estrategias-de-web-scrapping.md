---
layout: post
title: 'Princípios e estratégias de Web Scrapping'
date: '2018-03-21 9:00:00'
categories: scrapping
comments: true
author: marcelinhov2
---

Coletar dados nem sempre é uma tarefa simples. Ainda que hoje em dia a grande maioria das empresas "new world" optem por criar APIs para facilitar a integração com o seu sistema, existem muitas outras que não querem disponibilizar os seus dados e ainda as que simplesmente não entraram no século XXI. Casos como o do Banco do Brasil, que recentemente disponibilizou sua [API de serviços bancários](https://developers.bb.com.br/pt-br/), comprovam que as grandes corporações estão correndo atrás do tempo perdido e o futuro é muito promissor.

O objetivo desse post é apresentar um pouco das nossas estratégias para extrações de dados e coletores.

### Mas o que é Web Scrapping?

São processos automáticos que extraem dados de sistemas que não contenham nenhum tipo de integração oficial (ex.: API's). Costumam ser robôs que simulam a navegação de um usuário comum através de um browser, com o objetivo de capturar os dados mostrados na interface.

## Estratégias

O principal pré-requisito para o desenvolvimento de um coletor aqui na reviewr, é um estudo onde são levados em conta 3 possíveis caminhos a seguir para conseguir concretizar a integração: **APIs**, **browser scrapping** e **engenharia reversa**.

### APIs

Sem dúvida, o melhor dos 3 caminhos. Integrações via APIs tendem a ser de rápido desenvolvimento e costumam contar com um serviço de suporte, caso necessário. Em muitas empresas as APIs são tratadas como produtos e costumam receber muito investimento para que seus sistemas tenham alto índice de disponibilidade e sejam performáticos.

**Percepções gerais**

- **Esforço para integrar:** baixo
- **Estabilidade:** média / alta
- **Segurança:** alta
- **Custo de manutenção:** baixo
- **Performance:** alta

**Prós:**

- **Documentação:** costumam ser bem escritas e vir acompanhadas de bons exemplos de código;
- **Segurança:** a maioria conta com uma fase de autenticação, para que seus clientes possam utilizá-la de maneira segura e garantirem o acesso ao escopo de dados correto; 
- **Menos bugs:** o versionamento diminui drasticamente o risco da integração quebrar de repente;

**Contras:**

- **Manutenção:** é normal entrar nos mais diversos fórums de comunidade e encontrar issues com meses (e muitas vezes anos) de idade, sem resolução;
- **Disponibilidade:** se o sistema da empresa fica indisponível, não tem nada que possa ser feito do nosso lado, a não ser esperar ele voltar;
- **Custo:** muitas delas são pagas e caras. 

### Browser Scrapping

Mesmo sendo uma técnica bem antiga, muitas empresas contam com *scrappers* para automatizar a coleta de dados em sistemas que não disponibilizam APIs. Ao seguir por esse caminho, tenha a certeza de que existem algumas dificuldades que costumam tirar o sono de muita gente:

- Captchas
- AB Tests
- Bloqueio por user agent
- Bloqueio de IP's de nuvem (Amazon, Google Cloud, Azure...)
- Performance

O custo para manter um scrapper é alto, a manutenção é inevitável e dependendo do problema (que VAI acontecer) isso pode afetar por dias a disponibilidade do seu robô. Fora isso, é normal encontrar códigos e gambiarras de sangrar os olhos.

É uma técnica interessante para POC's e MVP's, mas por se tratar de uma simulação de uso, esses sistemas costumam ter uma performance bem baixa e com poucas possibilidades de otimização.


**Percepções gerais**

- **Esforço para integrar:** variado
- **Estabilidade:** baixa
- **Segurança:** baixa
- **Custo de manutenção:** alto
- **Performance:** baixa
 
### Engenharia reversa

*"A persistência é o melhor caminho para o êxito" - Charles Chaplin*

A principal diferença quando comparado com o **browser scrapping**, é que não precisamos de um browser. O objetivo é descobrir o endpoint resposável por retornar a informação necessária, e para isso, basta navegar até a URL alvo (com o Developer Tools aberto), abrir a aba Network, filtrar por XHR e descobrir a chamada. 

**Obs.:** *Se não achar o que precisa em nenhuma chamada, o dado pode ter sido retornado na requisição inicial.*

Três pontos chaves que nos fizeram enxergar o poder de utilizar da engenharia reversa nos nossos coletores:

**Performance**

Te permitem usufruir de todo um trabalho de otimizações feitos para garantir uma boa experiência aos usuários, permitindo atingir níveis interessantes de performance. Temos uma plataforma que conta com 2 robôs diferentes de coleta e a API de produto (não oficial) é mais rápida e traz um volume de dados muito maior do que a API oficial.

**Estabilidade e Disponibilidade**

Os endpoints de produto costumam ser core da empresa e trabalham com uma taxa altíssima de disponibilidade. Se sua empresa estiver trabalhando em integrações que vão ser disponibilizadas para seus usuários, quanto maior a disponibilidade, melhor a experiência entregue.

**Captchas**

Essa técnica permite um bypass total em qualquer tipo de validação com captchas, afinal, seu sistema vai fazer apenas requests HTTP.

<hr>

Apesar das vantagens, esse costuma ser o caminho mais difícil. Encontrar o endpoint alvo para ser utilizado é moleza, a dificuldade é que em muitos casos eles contam com algum tipo de segurança e necessitam de tokens que são gerados durante a navegação.

Mas não desanime, uma vez mapeada e automatizada a criação dos tokens de segurança (que não costumam ser alterados com muita frequência), metade do caminho vai ter sido percorrido.

O próximo passo é *parseamento* de dados. Muitas dessas APIs respondem HTMLs renderizados do lado do servidor, que acabam retornando muita *sujeira* e podem ser muito extensos. Utilizamos o ```cheerio``` para nos ajudar com essa última etapa, possibilitando o uso de seletores CSS para fazer a busca dos elementos.


**Percepções gerais**

- **Esforço para integrar:** alto
- **Estabilidade:** média
- **Segurança:** média
- **Custo de manutenção:** médio
- **Performance:** alta 

<hr>

## Nossos coletores

São desenvolvidos em Javascript e estamos migrando todos para a arquitetura de *functions*. Isso vai nos permitir escalar ao infinito com um custo mais baixo.

Cada robô conta com um MongoDb exclusivo onde ficam armazenados nossos *raw datas* e as coletas podem ser feitas de maneira instantânea ou via agendamento. Todos os reviews coletados sobem para uma fila, para garantir sua inserção no Jarvis, nosso Data Warehouse (Amazon Redshift).

![diagrama_coleta](https://user-images.githubusercontent.com/232648/37443047-6ee29bfa-27e8-11e8-8590-c02d86d34d30.png)

## Conclusão

Com muito estudo e persistência, é possível criar extratores de dados muito estáveis e poderosos. O segredo é manter ao máximo a simplicidade e reduzir o escopo da extração. Vale a pena tomar cuidado com as "soluções duvidosas" que podem surgir durante a sua batalha, isso vai evitar a construção de um frágil "castelo de cartas".

Muito obrigado pela leitura e se tiver alguma dúvida, não deixe de nos perguntar.

Acompanhe a reviewr no Facebook: [https://www.facebook.com/reviewrme](https://www.facebook.com/reviewrme)

Acompanhe a reviewr no Linkedin: [https://www.linkedin.com/company/reviewr.me/](https://www.linkedin.com/company/reviewr.me/)