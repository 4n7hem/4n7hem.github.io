---
layout: post
title: "Criando um site do GitHub Pages"
date: 2025-12-02 14:15:00 -0300
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

Uma das premissas é que os sites que você estará fazendo nas pages são exclusivamente estáticos. Eles ainda são capazes de se utilizar de JavaScript ou frameworks, contanto que não seja necessário processamento server-side. Até é possível fazer algumas coisas dinâmicas, porém é necessários hospedá-las fora do GitHub (além do mais, senão teria muito abuso disso). Para mais detalhes, segue um vídeo de um cara que fez exatamente isso: (perdi o link)

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

Se você sabe Markdown, todas as estruturas normais de Markdown funcionam pelo site. Então, no caso, eu vou focar em contar sobre como estruturar seu site como um site de verdade.
Em tanto páginas quanto posts, é necessário colocar o que chamam de "front-matter" no texto em si. O front-matter deste post é algo assim:
```yaml
---
layout: post
title: "Criando um site do GitHub Pages"
date: 2025-10-01 14:15:00 -0300
categories:
---
```
Isso será necessário para várias coisas, como:
 - Definir qual modelo de layout do seu site você quer que a página use;
 - O título que aparece na aba do navegador;
 - A data (que alguns formatos se utilizam para escrever dentro dos posts);
 - O caminho da página (aplicável somente para páginas, não posts).

Também é útil colocar outras metainformações, como `tags:` , `excerpt:`, `permalink:`, `author:` e `description:`, que pode interagir com personalizações ou ajudar SEOs.
### Criando novas páginas

As páginas devem ser os "hubs" centrais da informação do seu site. Eles devem ser colocados na raíz do projeto, junto da página inicial.

Para definir por onde a página pode ser acessada, dentro do frontheader, você usará o campo `permalink`, que constrói o caminho a partir da raíz do site. Ou seja, no .md da minha página de posts, está escrito:

```yaml
---
title: "Meus artigos"
permalink: /posts
---
```

E isso leva ao caminho
[4n7h3m.github.io/posts](./posts)

### Criando posts

Você pode criar uma pasta chamada `_posts` na raíz do projeto. Ela carregará todos os posts que o seu site pode ter.
Posts tem uma regra de nomenclatura, que é utilizada (provavelmente) pelo pipeline automático de implementação do site. A regra é:

```yaml
ano-mes-dia-nome_do_post.md
```
Quando você segue essa regra, o post poderá ser acessado em:

```yaml
./ano/mes/dia/nome_do_post.html
```

### Hyperlinkando uma na outra

Imaginemos um fluxo de usuário simples:
Alguém entra na sua página, vê que você possui uma aba de posts, e vai tentar ler algum dos seus posts.

Não é necessário escrever todo o caminho em extenso. Você pode apenas escrever o caminho relativo:

`(./posts)`

Isso se aplica tanto a páginas quanto posts.


## Tá. Mas eu vi sites mais bonitos mundo afora. Como eu faço eles exatamente?

Então. Aqui não é completamente anti-HTML como você possa imaginar. Na verdade, para conseguir verdadeira customização, é necessário se utilizar de HTML e CSS para modificar a cara do site.

Vamos começar pelo simples: Provavelmente o seu site vai querer um header e um footer, como todos os outros. Para isso você **vai** precisar de HTML.

Crie uma pasta chamada `_layouts` e outra chamada `_includes`. Ambas servirão funções diferentes.

A pasta de layouts serve para que os arquivos definidos nela sejam importados pelo frontheader das páginas principais. Quando você escrever:

```yaml
---
layout: algo
---
```

A compilação do site está, na verdade, procurando por `./_layouts/algo.html` e, se não existir, cai para as definições do tema. Por isso alguns desses funcionam ***baseados no tema***.

A pasta de includes serve um diferente propósito. Enquanto você estiver escrevendo HTML, você pode (graças às maravilhas de bons padrões de código) reutilizar código escrito em vários lugares do site. Esta pasta é para especificamente isso. Você pode escrever o seguinte:


```yaml
<div>
 {include algum_bagulho.html}
</div>
```

Ele procurará por `./_includes/algum_bagulho.html` e colocará o que estiver escrito nessa posição.


## Como testar seu site localmente

Antes de enviar suas mudanças para o GitHub, é muito útil você rodar o Jekyll localmente para garantir que o que está sendo enviado já está correto. A documentação oficial do GitHub recomenda que se use o Bundler para instalar o Jekyll. Então, os passos em ordem para configurar seu ambiente são:

### Instale Ruby

O Jekyll é escrito em Ruby, então, é necessário instalar seu SDK antes. Em Linux, normalmente ele já vem instalado, e em Windows, ele pode ser configurado pelo _RubyInstaller_.

### Adicione o Bundler

Com o Ruby configurado, use seu gerenciador de pacotes para instalar o Bundler:

```sh
gem install bundler
```

### Instale as dependências do Jekyll

Na raiz do seu projeto, crie um arquivo chamado **Gemfile**, contendo:

```
source 'https://rubygems.org'

gem 'github-pages'

gem "webrick", "~> 1.7"
```

E, após isso, só rode:

```sh
bundle install
```

Isso deve criar uma pasta `vendor/` com todos os plugins, incluindo o Jekyll. Além disso, como gerenciadores de pacotes, isso irá gerar um `Gemfile.lock` Lembre-se de adicionar ambos no seu `.gitignore` !

### Só rodar

Para testar o site, escreva no terminal, na pasta relevante:

```sh
bundle exec jekyll serve
```

Que vai iniciar um servidor padrão na máquina host. Como todo servidor de desenvolvimento web, salvar alterações em arquivos fará o site se recompilar automaticamente e processar as mudanças.