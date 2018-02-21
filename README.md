### Escrevendo posts

 Voce pode publicar e manter o blog simplesmente gerenciando uma pasta(_posts), para isso voce precisa somente utilizar a estrutura seguinte.

#### Pasta `_posts`

Na pasta `_posts` é onde vão ficar os posts do blog (OH!RLY?!), os arquivo deveram ser escritos em Markdown.

#### Criando o arquivo do post

Já no diretório `_posts` criar um arquivo com o nome utilizando o seguinte padrão:

```
ANO-MES-DIA-nosso-busines-e-bem-legal.md
```

#### Corpo do arquivo

No corpo do arquivo deve conter no minimo o `Layout` e o `title`.  

```
---
layout: post
title: 'Um business bem top!'
---
```
Mas você tb pode colocar o `date`, `categories` e `tags`.
```
---
layout: post
title: 'Você programa em Go?!!'
date: '2018-02-15 15:12:11'
categories: go
---
```
#### Exemplo de post
```
---
layout: post
title: 'Cachorro e computadores combinam?'
date: '2018-02-15 15:12:11'
categories: pelos
---
Cachorro e computador são orb effect? tire sua propria conclusão.

![Foto massa de um cachorro no pc]({{ "/assets/dogfofin.jpg" | absolute_url }})

um pdf top sobre cachorros [get the PDF]({{ "/assets/cachorroveespiritos.pdf" | absolute_url }})

Conclusão:
Dog é top!
```

#### E depois?

Feito o post todo bonitinho agora é só abrir um PR e ser feliz :)
