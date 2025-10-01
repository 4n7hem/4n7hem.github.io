---
layout: post
title: "Criando um site do GitHub Pages"
date: 2025-10-01 14:15:00 -0300
categories:
---

Então. Enquanto eu estava criando esse site, eu fui lendo a [Documentação oficial](https://docs.github.com/en/pages) do GitHub sobre como montar, mas várias dúvidas não documentadas ficaram surgindo.

Considerem esse artigo tanto uma tradução da documentação original quanto um detalhamento das respostas de obtive enquanto programava o site.

Eu vou assumir que você é familiarizado tanto em GitHub quanto em como escrever Markdown.

## Mas como eu começo?

Copiando verbatim o que está na documentação, tudo o que você precisa fazer é criar um repositório que se chame **SEU-NOME-DE-USUÁRIO.github.io**. Não há problema se o seu nome de usuário usa números (que nem o meu) e eu acredito que as limitações de escolha de username do GitHub impedem casos de exceção para isso. Inicialize já com um Markdown, para facilitar.

Após isso, vá na aba de "Configurações (Settings)" -> "Pages" -> Visitar Site (Visit Site). Ao que tudo indica, o GitHub já sobe sozinho a página se você seguiu o padrão de nome do repositório.

E parabéns, tens uma página de graça.

A parte importante aqui é frisar que isso é *absolutamente tudo o que você precisa.* Não é necessário clonar o de alguém já de começo, e todas as próximas configurações que você quiser fazer, você manualmente adiciona as pastas e arquivos que precisa no repositório, e escreve eles do zero.

## Beleza, mas o site é hiper feio

É porque Markdown nunca foi *exatameeeeente* feito para ser um site. Sua função principal sempre foi ser uma ferramenta versátil de texto, que se integra muito bem com repositórios Git e permite tudo, como links, imagens, etc.

Uma das premissas é que os sites que você estará fazendo nas pages são exclusivamente estáticos. Sem JavaScript ou frameworks de Frontend necessários. Até é possível fazer algumas coisas dinâmicas, porém é necessários hospedá-las fora do GitHub (além do mais, senão teria muito abuso disso). Para mais detalhes, segue um vídeo de um cara que fez exatamente isso: (perdi o link)

Para você deixar o site com mais a sua cara, é necessário se utilizar (pelo menos, para o jeito mais simples) dos templates do Jekyll. Este próprio site está usando o estilo Minimalist, um dos estilos padrão.

Adicione um arquivo chamado `_config.yml` na raíz do repositório, e escreva o seguinte dentro dele:

```
remote_theme: pages-themes/minimal@v0.2.0
plugins:
- jekyll-remote-theme
```

Isto será aplicado conforme o pipeline padrão do GitHub sobe a sua página. Sem mais detalhes necessários.
Para uma lista de todos os temas disponibilizados, consultar a documentação oficial.

## O site está mais bonito. Como eu escrevo coisas nele??