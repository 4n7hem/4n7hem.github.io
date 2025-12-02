---
layout: post
title: "Gerenciando seu cluster do OpenSearch / ElasticStack"
date: 2025-10-17 18:00:00 -0300
categories:
author: Caio
---

Tanto o OpenSearch quanto o ElasticStack são ótimos bancos de dados não relacionais. Eles dão já fora-da-caixa um stack completo para qualquer coisa que se espera para um banco: APIs para armazenamento e consumo inteligente da informação, ferramentas de dashboard integradas, sintáxe inteligente e própria para queries detalhadas, gestão de acesso customizável e integrável, funcionalidades de distribuição e redundância nativas da ferramenta, você escolhe. Provavelmente tem.

Se você é familiarizado com o Solr, ambos são razoavelmente semelhantes. Ambos utilizam Lucene por trás dos panos. Porém, os que estou apresentando aqui definitivamente são mais bonitos.

## Um pouco de background

Eu estou juntando ambos em um singular post, pois o OpenSearch é um fork aberto do ElasticStack. As diferenças entre os dois estão concentradas basicamente no ElasticStack oferecer suporte pago para empresas e mais funcionalidades de dashboards.

Tirando isso, basicamente os dois são a mesma coisa.

Caso você esteja tentando estudar independentemente como utilizar o ElasticSearch, ou tentar experimentar na prática, para a sua empresa, como o ElasticSearch pode resolver problema X, use o OpenSearch.

Sempre glória às alternativas abertas.

## Subindo seu cluster

Vou assumir que você saiba como interagir especificamente com o ambiente de cluster que está nas suas mãos. Isso pode variar de nuvem a até on-premises, ou tentar subir o cluster com um somente nó.

O importante é saber quais são os papéis que os nós precisam exercer e deixar que a documentação te guie. Para os fins mais fáceis, a própria documentação oficial já oferece um exemplo de Docker Compose para iniciar o seu cluster:

https://docs.opensearch.org/latest/tuning-your-cluster/
https://docs.opensearch.org/latest/install-and-configure/install-opensearch/docker/#complete-docker-compose-example-with-custom-configuration

O importante é que estejam bem separados os nós que carregam dados, e o nó do Kibana, que faz o grosso de toda a UI.

## Criando seus índices

Um índice, para efeitos de comparação, é como se fosse uma tabela de SQL. Lá, estão guardadas entradas que compartilham *majoritariamente* os mesmos campos. Apenas usuários com elevação (gerenciada por acesso) possuem a permissão de criar índices.

Para criar um novo índice, para que você possa começar a enviar dados, vá em:

`Index Management > Indexes > Create Index`

Aqui, você poderá definir o nome do índice (extremamente importante por causa do próximo tópico! escolha um bom nome padronizado) e diversas coisas importantes.

Para efeitos de resiliência dos dados, é importante configurar as shards e as replicas dos dados conforme a capacidade do seu cluster sugere. Isso virá em conta em casos de falha.

Caso você esteja manipulando tipos de dados problemáticos (também conhecidos como: datas), declare os nomes dos campos e seus tipos manualmente na criação do índice. Com bastante frequência, a tipagem automática das ferramentas erra e datas, especificamente, precisam ser enviadas sob um específico padrão para que o OpenSearch reconheça elas. Senão, você terá vários dados que nunca poderão se utilizar dos Dashboards com filtros de data. **Isso não pode ser alterado uma vez que o índice é criado. Faça com sabedoria.**

Com o índice criado, você já é capaz de inserir informação no banco através da API, ou manualmente através do Dev Tools. Mas a teoria de algo antes é mais importante:


## Criando seus padrões de índices (para Dashboards e Discover)

Com um índice criado, ele é basicamente uma enorme caixa preta. Dá para ter quantos teras de informação lá dentro que você quiser, que não dá para ver direito.

Para conseguir olhar lá dentro, é necessário criar um padrão de índice. Ele é criado em:

`Dashboards Management > Index Patterns > Create Index Pattern`

O comentário para escolher um bom nome do índice é por causa do jeito que os padrões funcionam. Para criar o pattern, você só é perguntado duas coisas: A string que serve como padrão e que campo de data você quer usar para visualização no Discover.

A string é a parte importante. Imagine que você tem dois índices chamados `bolinha1` e `bolinha2`. Caso você crie um index pattern chamado `bolinha*`, todas as entradas de ambos os índices irão aparecer no Discover, misturadas umas das outras. Isso pode ser útil quando você quer separar fisicamente informações diferentes, mas juntá-las de forma fácil, através do pattern, para análise posterior. Senão, você precisa criar dois patterns: `bolinha1*` e `bolinha2*`.

O campo de data, use se possível, mas é opcional.

Após isso, agora você é capaz de ver a informação dentro do índice. Mas já que você criou o índice do zero agora, ele está vazio né. Não tem o que ver. O próximo passo é adicionar sua informação lá.

## A documentação da API

Antes de sequer entrar na explicação de como de fato inserir informação lá, é importante introduzir à documentação da API.
**É** necessário se utilizar da API para fazer inserções automáticas, em massa, e bonitas do sistema. Eu não posso escrever os métodos que normalmente se utiliza, pois é informação que ficará datada a partir do tempo. Então, cheque a própria documentação oficial e os métodos da API em:

https://docs.opensearch.org/latest/api-reference/document-apis/index/

Mas, na média, você vai querer o endpoint da inserção em bulk e de pesquisa. O CRUD completo está disponível, e bem mais. 

## Inserindo seus dados

O que é tranquilamente a parte mais difícil de utilizar essas ferramentas. A inserção falha completamente caso os dados inseridos estejam fora do padrão. Como, por exemplo, um campo é uma string tentando ser colocada no campo mapeado como int? Não passa. Data formatada errada? Também não. Chave primária duplicada? Não entra.

E a parte difícil: Nas operações de bulk, um estar errado faz tudo do bulk não entrar junto. Quando estiver desenhando suas chamadas à API usando _bulk, você quer multiprocessar e separar em blocos bem pequenos para evitar grande perda de informação.

Caso o futuro não mude isso, o formato aceito de data pela API é uma string que segue:
```
"%Y-%m-%dT%H:%M:%SZ"
```
Sim. Com o T e o Z junto.
A API aceitará somente JSONs passados em formato de texto. Talvez seja integrado com os objetos do JavaScript, mas em outras linguagens, você terá que rasterizar o JSON para texto antes de enviar.

Separando com `\n`s, o corpo da requisição deve ser um monte de:

```
{ "create": { "_index": indice_opensearch , "_id": id} }
{# na linha seguinte, o objeto JSON literal.}
```

`_id` é opcional. Caso você não providencie, o OpenSearch fará algo aleatório para preencher este campo. Porém, você deveria preencher esse campo. Tente sempre se utilizar de algo que seja reproduzível através dos dados para que inserções duplicadas falhem e que você rapidamente encontre uma entrada para alterá-la ou apagá-la.

Uma coisa que definitivamente falta (pelo menos, quando eu escrevo esse artigo) são bibliotecas de suporte (conectores) para a inserção de informação. Ela é bastante conturbada, na maior parte do tempo, e eu tive que escrever algo genérico o suficiente para ser utilizado. Talvez haja para outros ecossistemas que não sejam do Python, mas converter um DataFrame para o formato do OpenSearch é confuso e complexo.

## Como pesquisar informação

## O Dev Tools

## Plugins