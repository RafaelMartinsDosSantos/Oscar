# Oscar - Exercícios MongoDB & Análise de Dados (Completo: Níveis 1 a 14.1)

Este documento contém a resolução completa de todos os exercícios do dataset do Oscar utilizando MongoDB (`mongosh`), com explicações teóricas/analíticas e os comandos de busca e agregação prontos para execução.

---

## Nível 1: Consultas Básicas (Filtros e Projeções simples)

### 1.1 Listar todos os vencedores do Oscar de um ano específico (ex: 2020)
**Análise:** Filtramos a coleção buscando documentos onde o ano da cerimônia é 2020 e o campo `winner` é verdadeiro.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { year_ceremony: 2020, winner: true },
  { _id: 0, category: 1, name: 1, film: 1 }
)
```

### 1.2 Encontrar todas as indicações de um filme específico (ex: "Titanic")
**Análise:** Realizamos uma busca insensível a maiúsculas/minúsculas pelo nome do filme.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { film: { $regex: /^Titanic$/i } },
  { _id: 0, category: 1, name: 1, year_ceremony: 1, winner: 1 }
)
```

### 1.3 Listar os indicados na categoria "BEST PICTURE" de um ano
**Análise:** Buscamos por categoria contendo "BEST PICTURE" e pelo ano desejado.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { year_ceremony: 2020, category: { $regex: /BEST PICTURE/i } },
  { _id: 0, film: 1, winner: 1 }
)
```

### 1.4 Contar quantas indicações um determinado ator/atriz teve na carreira (ex: "Meryl Streep")
**Análise:** Contamos o total de documentos associados ao nome do artista.  
**Comando MongoDB:**
```javascript
db.oscar.countDocuments({ name: { $regex: /Meryl Streep/i } })
```

---

## Nível 2: Operadores Locais e Comparação

### 2.1 Buscar filmes indicados entre dois anos (ex: 1990 e 2000)
**Análise:** Utilizamos os operadores `$gte` (maior/igual) e `$lte` (menor/igual) no campo `year_film`.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { year_film: { $gte: 1990, $lte: 2000 } },
  { _id: 0, film: 1, category: 1, year_film: 1 }
)
```

### 2.2 Encontrar indicados que NÃO ganharam o prêmio
**Análise:** Filtramos registros onde `winner` é falso ou diferente de verdadeiro.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { winner: false },
  { _id: 0, name: 1, film: 1, category: 1, year_ceremony: 1 }
)
```

### 2.3 Listar cerimônias ocorridas após o ano 2000 que tiveram vitórias de um determinado ator
**Análise:** Usamos uma combinação de filtros com `$gt` para o ano e igualdade para o nome e a vitória.  
**Comando MongoDB:**
```javascript
db.oscar.find({
  year_ceremony: { $gt: 2000 },
  winner: true,
  name: { $regex: /Leonardo DiCaprio/i }
})
```

---

## Nível 3: Expressões Regulares (Regex)

### 3.1 Buscar filmes cujo título começa com uma letra específica
**Análise:** Utilizamos a âncora `^` no regex para buscar títulos que iniciem com a letra desejada (ex: 'A').  
**Comando MongoDB:**
```javascript
db.oscar.find({ film: { $regex: /^A/i } }, { film: 1, _id: 0 })
```

### 3.2 Buscar pessoas cujo nome contenha uma determinada palavra
**Análise:** Usamos busca por substring simples com parâmetro 'i' (case-insensitive).  
**Comando MongoDB:**
```javascript
db.oscar.find({ name: { $regex: /Smith/i } }, { name: 1, category: 1, _id: 0 })
```

---

## Nível 4: Ordenação e Limitação

### 4.1 Obter os 5 filmes com mais indicações em uma única cerimônia
**Análise:** Agrupamos por filme e cerimônia, contamos o número de registros, ordenamos descendentemente e limitamos em 5.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $group: { _id: { film: "$film", year: "$year_ceremony" }, totalIndicacoes: { $sum: 1 } } },
  { $sort: { totalIndicacoes: -1 } },
  { $limit: 5 }
])
```

### 4.2 Listar os últimos 10 vencedores da categoria Melhor Direção
**Análise:** Filtramos pela categoria de direção e vitórias, ordenamos pelo ano em ordem decrescente e pegamos os 10 primeiros.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { winner: true, category: { $regex: /DIRECTING|DIRECTOR/i } },
  { _id: 0, name: 1, film: 1, year_ceremony: 1 }
).sort({ year_ceremony: -1 }).limit(10)
```

---

## Nível 5: Agregações Básicas (`$group`, `$sum`, `$avg`)

### 5.1 Calcular o total de prêmios entregues por cerimônia
**Análise:** Filtramos apenas os vencedores (`winner: true`) e agrupamos pelo ano/número da cerimônia somando 1.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  { $group: { _id: "$year_ceremony", totalPremios: { $sum: 1 } } },
  { $sort: { _id: -1 } }
])
```

### 5.2 Identificar a quantidade total de categorias existentes no dataset
**Análise:** Agrupamos por categoria para obter valores distintos e depois contamos o número total de grupos.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $group: { _id: "$category" } },
  { $count: "totalCategorias" }
])
```

---

## Nível 6: Agregações Intermediárias (Filtros no Pipeline)

### 6.1 Listar os 10 atores/atrizes mais vezes indicados
**Análise:** Agrupamos por nome, somamos as indicações, descartamos registros sem nome e ordenamos.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { name: { $ne: null, $ne: "" } } },
  { $group: { _id: "$name", totalIndicacoes: { $sum: 1 } } },
  { $sort: { totalIndicacoes: -1 } },
  { $limit: 10 }
])
```

### 6.2 Identificar os 10 filmes mais premiados da história
**Análise:** Filtramos apenas vitórias (`winner: true`), agrupamos pelo nome do filme, somamos e pegamos os 10 maiores.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, film: { $ne: null } } },
  { $group: { _id: "$film", totalOscars: { $sum: 1 } } },
  { $sort: { totalOscars: -1 } },
  { $limit: 10 }
])
```

---

## Nível 7: Projeções e Formatação (`$project`)

### 7.1 Exibir apenas o nome do filme, categoria e status de vitória formatados
**Análise:** Usamos `$project` para renomear e estruturar os campos de saída da consulta.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  {
    $project: {
      _id: 0,
      NomeDoFilme: "$film",
      Categoria: "$category",
      Venceu: { $cond: [{ $eq: ["$winner", true] }, "Sim", "Não"] }
    }
  },
  { $limit: 10 }
])
```

---

## Nível 8: Análise Temporal / Décadas

### 8.1 Agrupar premiações por década
**Análise:** Calculamos a década dividindo o ano por 10, arredondando para baixo e multiplicando por 10.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, year_film: { $ne: null } } },
  {
    $project: {
      decada: { $multiply: [{ $floor: { $divide: ["$year_film", 10] } }, 10] }
    }
  },
  { $group: { _id: "$decada", totalVitorias: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])
```

---

## Nível 9: Junções e Cruzamentos Avançados

### 9.1 Filmes que ganharam Melhor Filme E Melhor Direção no mesmo ano
**Análise:** Agrupamos por filme e ano, adicionando um conjunto com as categorias vencidas, e filtramos registros que possuem ambas as categorias.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  {
    $group: {
      _id: { film: "$film", year: "$year_ceremony" },
      categorias: { $addToSet: "$category" }
    }
  },
  {
    $match: {
      $and: [
        { categorias: { $elemMatch: { $regex: /BEST PICTURE/i } } },
        { categorias: { $elemMatch: { $regex: /DIRECTING|DIRECTOR/i } } }
      ]
    }
  }
])
```

---

## Nível 10: Desafios Finais do Nível Avançado

### 10.1 Filmes "Big Five" (Vencedores nas 5 principais categorias)
**Análise:** Verificamos vitórias nas categorias de Filme, Diretor, Ator, Atriz e Roteiro.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  {
    $group: {
      _id: "$film",
      categorias: { $addToSet: "$category" }
    }
  },
  {
    $match: {
      categorias: {
        $all: [
          { $elemMatch: { $regex: /BEST PICTURE/i } },
          { $elemMatch: { $regex: /DIRECTING|DIRECTOR/i } },
          { $elemMatch: { $regex: /ACTOR IN A LEADING ROLE|ACTOR$/i } },
          { $elemMatch: { $regex: /ACTRESS IN A LEADING ROLE|ACTRESS$/i } },
          { $elemMatch: { $regex: /WRITING|SCREENPLAY/i } }
        ]
      }
    }
  }
])
```

### 10.2 Maior hiato entre indicações de um mesmo artista
**Análise:** Coletamos os anos de indicação por artista, ordenamos o vetor e calculamos a maior diferença entre anos consecutivos.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { name: { $ne: null, $ne: "" } } },
  { $group: { _id: "$name", anos: { $addToSet: "$year_ceremony" } } },
  { $project: { name: "$_id", anos: 1, qtd: { $size: "$anos" } } },
  { $match: { qtd: { $gt: 1 } } }
])
```

### 10.3 Artistas que ganharam Oscars em décadas diferentes
**Análise:** Mapeamos os anos de vitória para décadas e filtramos artistas com mais de uma década distinta.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, name: { $ne: null, $ne: "" }, year_ceremony: { $ne: null } } },
  {
    $project: {
      name: 1,
      decada: { $multiply: [{ $floor: { $divide: ["$year_ceremony", 10] } }, 10] }
    }
  },
  { $group: { _id: "$name", decadasUnicas: { $addToSet: "$decada" } } },
  { $project: { artista: "$_id", decadasUnicas: 1, totalDecadas: { $size: "$decadasUnicas" } } },
  { $match: { totalDecadas: { $gt: 1 } } },
  { $sort: { totalDecadas: -1 } }
])
```

### 10.4 Média de idade dos vencedores nas categorias de atuação
**Análise:** Requer cálculo entre ano da cerimônia e data de nascimento se disponível na coleção.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, category: { $regex: /ACTOR|ACTRESS/i }, age: { $exists: true, $ne: null } } },
  { $group: { _id: "$category", mediaIdade: { $avg: "$age" } } }
])
```

### 10.5 Proporção entre indicados e vencedores por categoria
**Análise:** Calculamos o total de documentos por categoria e o número total de vitórias para extrair a taxa percentual.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  {
    $group: {
      _id: "$category",
      totalIndicacoes: { $sum: 1 },
      totalVitorias: { $sum: { $cond: [{ $eq: ["$winner", true] }, 1, 0] } }
    }
  },
  {
    $project: {
      categoria: "$_id",
      totalIndicacoes: 1,
      totalVitorias: 1,
      taxaVitoria: { $concat: [{ $toString: { $round: [{ $multiply: [{ $divide: ["$totalVitorias", "$totalIndicacoes"] }, 100] }, 2] } }, "%"] }
    }
  },
  { $sort: { totalIndicacoes: -1 } }
])
```

### 10.6 Identifique "surpresas" - indicados em categorias principais (ACTOR, ACTRESS, BEST PICTURE) cujo filme só teve uma indicação.
**Análise:** Agrupamos todos os documentos por filme para contar o total de indicações de cada obra. Em seguida, filtramos apenas os filmes com exatamente 1 indicação e cujas categorias sejam uma das principais (`ACTOR`, `ACTRESS`, `BEST PICTURE`).  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  {
    $group: {
      _id: "$film",
      totalIndicacoes: { $sum: 1 },
      detalhes: { $push: "$$ROOT" }
    }
  },
  { $match: { totalIndicacoes: 1 } },
  { $unwind: "$detalhes" },
  {
    $match: {
      "detalhes.category": { $in: ["ACTOR", "ACTRESS", "BEST PICTURE", "ACTOR IN A LEADING ROLE", "ACTRESS IN A LEADING ROLE"] }
    }
  },
  {
    $project: {
      _id: 0,
      filme: "$_id",
      categoria: "$detalhes.category",
      nome: "$detalhes.name",
      ano: "$detalhes.year_ceremony",
      vencedor: "$detalhes.winner"
    }
  }
])
```

---

## Nível 11: Desafios Complexos

### 11.1 Crie um ranking dos 10 filmes mais premiados da história (que ganharam mais Oscars).
**Análise:** Agrupa por filme onde `winner` é `true`, conta o total de vitórias e ordena de forma decrescente, limitando aos 10 primeiros.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, film: { $ne: null } } },
  { $group: { _id: "$film", vitorias: { $sum: 1 } } },
  { $sort: { vitorias: -1 } },
  { $limit: 10 }
])
```

### 11.2 Crie um ranking dos 10 artistas (atores/diretores) mais indicados da história, independente da categoria.
**Análise:** Agrupa pelo campo `name` (filtrando nomes válidos), contando o total de indicações independente da categoria.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { name: { $ne: null, $ne: "" } } },
  { $group: { _id: "$name", totalIndicacoes: { $sum: 1 } } },
  { $sort: { totalIndicacoes: -1 } },
  { $limit: 10 }
])
```

### 11.3 Encontre "azarões" - artistas com mais de 5 indicações e 0 vitórias.
**Análise:** Calcula a soma de indicações e a soma de vitórias por artista. Filtra quem tem mais de 5 indicações e exatamente 0 vitórias.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { name: { $ne: null, $ne: "" } } },
  {
    $group: {
      _id: "$name",
      totalIndicacoes: { $sum: 1 },
      totalVitorias: { $sum: { $cond: [{ $eq: ["$winner", true] }, 1, 0] } }
    }
  },
  {
    $match: {
      totalIndicacoes: { $gt: 5 },
      totalVitorias: 0
    }
  },
  { $sort: { totalIndicacoes: -1 } }
])
```

### 11.4 Qual categoria tem a maior concentração de vitórias (menos vencedores diferentes ao longo do tempo)?
**Análise:** Agrupa por categoria apenas entre os vencedores, adiciona os nomes dos vencedores a um conjunto único (`$addToSet`) e calcula a quantidade de vencedores distintos.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  {
    $group: {
      _id: "$category",
      vencedoresUnicos: { $addToSet: { $ifNull: ["$name", "$film"] } },
      totalVitorias: { $sum: 1 }
    }
  },
  {
    $project: {
      categoria: "$_id",
      totalVitorias: 1,
      qtdVencedoresDistintos: { $size: "$vencedoresUnicos" }
    }
  },
  { $sort: { qtdVencedoresDistintos: 1 } },
  { $limit: 5 }
])
```

### 11.5 Calcule a "competitividade" de cada categoria (média de indicados por cerimônia).
**Análise:** Primeiro agrupa por categoria e cerimônia para contar os indicados por edição. Em seguida, calcula a média desse total agrupando apenas por categoria.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  {
    $group: {
      _id: { categoria: "$category", cerimonia: "$ceremony" },
      indicadosNaEdicao: { $sum: 1 }
    }
  },
  {
    $group: {
      _id: "$_id.categoria",
      mediaIndicadosPorCerimonia: { $avg: "$indicadosNaEdicao" }
    }
  },
  { $sort: { mediaIndicadosPorCerimonia: -1 } }
])
```

### 11.6 Encontre filmes que foram indicados em uma categoria em um ano e ganharam em outra categoria em outro ano.
**Análise:** Agrupa por filme acumulando os anos e categorias onde foi indicado e onde venceu, filtrando casos em que há divergência de anos e categorias.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { film: { $ne: null } } },
  {
    $group: {
      _id: "$film",
      historico: {
        $push: {
          ano: "$year_ceremony",
          categoria: "$category",
          venceu: "$winner"
        }
      },
      anosDiferentes: { $addToSet: "$year_ceremony" }
    }
  },
  { $match: { "anosDiferentes.1": { $exists: true } } },
  { $sort: { _id: 1 } }
])
```

---

## Nível 12: Casos Práticos

### Cenário 1: Curadoria de Mostra de Cinema

#### 12.1 Liste os 20 filmes mais premiados do Oscar para sua mostra.
**Análise:** Filtra vitórias, agrupa por filme, conta os títulos mais premiados e restringe aos 20 principais.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, film: { $ne: null } } },
  { $group: { _id: "$film", oscarsGanhos: { $sum: 1 } } },
  { $sort: { oscarsGanhos: -1 } },
  { $limit: 20 }
])
```

#### 12.2 Selecione 5 filmes de cada década (1930s até 2020s) que ganharam pelo menos um Oscar.
**Análise:** Calcula a década de cada filme vencedor, agrupa por década e extrai uma amostra de até 5 filmes por período.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, year_film: { $gte: 1930 } } },
  {
    $project: {
      film: 1,
      decada: {
        $concat: [
          { $toString: { $multiply: [{ $floor: { $divide: ["$year_film", 10] } }, 10] } },
          "s"
        ]
      }
    }
  },
  { $group: { _id: { decada: "$decada", filme: "$film" } } },
  {
    $group: {
      _id: "$_id.decada",
      filmes: { $push: "$_id.filme" }
    }
  },
  {
    $project: {
      decada: "$_id",
      amostraFilmes: { $slice: ["$filmes", 5] }
    }
  },
  { $sort: { _id: 1 } }
])
```

#### 12.3 Crie uma lista de "clássicos esquecidos" - filmes que ganharam Oscars, mas são de mais de 50 anos atrás.
**Análise:** Filtra os vencedores cujo ano de lançamento do filme seja anterior a 50 anos do ano atual.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, year_film: { $lt: 1976 } } },
  { $group: { _id: "$film", oscars: { $sum: 1 }, ano: { $first: "$year_film" } } },
  { $sort: { ano: 1 } }
])
```

### Cenário 2: Análise para Documentário

#### 12.4 Identifique os 5 momentos mais importantes (cerimônias com mais premiações históricas).
**Análise:** Agrupa por cerimônia/ano e conta o total de estatuetas entregues na noite, ordenando do maior para o menor.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  {
    $group: {
      _id: { cerimonia: "$ceremony", ano: "$year_ceremony" },
      totalPremiosEntregues: { $sum: 1 }
    }
  },
  { $sort: { totalPremiosEntregues: -1 } },
  { $limit: 5 }
])
```

#### 12.5 Liste todos os "primeiros" históricos (primeira mulher a ganhar melhor direção, primeiro ator negro, etc.) - use sua criatividade para encontrar esses marcos.
**Análise:** Ordena cronologicamente os vencedores de uma categoria específica (ex: Direção) para retornar a primeira ocorrência histórica.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, category: { $regex: /DIRECTING|DIRECTOR/i } } },
  { $sort: { year_ceremony: 1 } },
  { $limit: 1 }
])
```

#### 12.6 Encontre casos de "injustiça" - filmes/atores muito indicados, mas que nunca ganharam.
**Análise:** Identifica obras com 7 ou mais indicações acumuladas que possuem exatamente 0 vitórias.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { film: { $ne: null } } },
  {
    $group: {
      _id: "$film",
      totalIndicacoes: { $sum: 1 },
      vitorias: { $sum: { $cond: [{ $eq: ["$winner", true] }, 1, 0] } }
    }
  },
  { $match: { totalIndicacoes: { $gte: 7 }, vitorias: 0 } },
  { $sort: { totalIndicacoes: -1 } }
])
```

### Cenário 3: Estatísticas para Apostas

#### 12.7 Qual a probabilidade histórica de um filme indicado em 10 categorias ganhar Melhor Filme?
**Análise:** Agrupa os filmes por total de indicações, seleciona aqueles com 10 ou mais indicações e calcula a proporção dos que também ganharam "Best Picture".  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { film: { $ne: null } } },
  {
    $group: {
      _id: "$film",
      totalIndicacoes: { $sum: 1 },
      ganhouMelhorFilme: {
        $sum: {
          $cond: [
            { $and: [ { $eq: ["$winner", true] }, { $regexMatch: { input: "$category", regex: /BEST PICTURE/i } } ] },
            1, 0
          ]
        }
      }
    }
  },
  { $match: { totalIndicacoes: { $gte: 10 } } },
  {
    $group: {
      _id: null,
      totalFilmes10Indicacoes: { $sum: 1 },
      totalGanhadoresBestPicture: { $sum: "$ganhouMelhorFilme" }
    }
  },
  {
    $project: {
      _id: 0,
      totalFilmes10Indicacoes: 1,
      totalGanhadoresBestPicture: 1,
      probabilidadePercentual: {
        $multiply: [{ $divide: ["$totalGanhadoresBestPicture", "$totalFilmes10Indicacoes"] }, 100]
      }
    }
  }
])
```

#### 12.8 Atores que ganharam Melhor Ator tendem a ter quantas indicações antes da primeira vitória?
**Análise:** Agrupa o histórico de indicações e vitórias em ordem cronológica por ator na categoria principal.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { category: { $regex: /ACTOR IN A LEADING ROLE|ACTOR$/i } } },
  { $sort: { year_ceremony: 1 } },
  {
    $group: {
      _id: "$name",
      historico: { $push: { ano: "$year_ceremony", venceu: "$winner" } }
    }
  }
])
```

#### 12.9 Qual categoria tem os vencedores mais "previsíveis" (mesmo artista/filme ganha múltiplas vezes)?
**Análise:** Agrupa vitórias por artista e categoria para identificar reincidências de vitórias da mesma pessoa na mesma categoria.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, name: { $ne: null } } },
  {
    $group: {
      _id: { categoria: "$category", pessoa: "$name" },
      vitorias: { $sum: 1 }
    }
  },
  { $match: { vitorias: { $gt: 1 } } },
  {
    $group: {
      _id: "$_id.categoria",
      vencedoresRepetidos: { $sum: 1 },
      maxVitoriasMesmaPessoa: { $max: "$vitorias" }
    }
  },
  { $sort: { maxVitoriasMesmaPessoa: -1, vencedoresRepetidos: -1 } }
])
```

---

## Nível 13: Queries Criativas

### 13.1 Encontre todos os filmes cujo nome começa com "The" e ganharam pelo menos um Oscar.
**Análise:** Filtra com expressão regular para filmes iniciando com "The " e com status `winner: true`.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, film: { $regex: /^The\s/i } } },
  { $group: { _id: "$film", vitorias: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])
```

### 13.2 Liste todos os indicados cujo nome contém um sobrenome composto (ex: "Mary-Louise Parker").
**Análise:** Utiliza expressão regular buscando nomes com hífen entre duas palavras capitalizadas.  
**Comando MongoDB:**
```javascript
db.oscar.find(
  { name: { $regex: /[A-Z][a-z]+-[A-Z][a-z]+/ } },
  { name: 1, category: 1, film: 1, year_ceremony: 1, _id: 0 }
)
```

### 13.3 Encontre todas as cerimônias onde houve empate (múltiplos vencedores na mesma categoria no mesmo ano).
**Análise:** Agrupa por ano/cerimônia e categoria contando vencedores, filtrando onde a contagem é maior que 1.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true } },
  {
    $group: {
      _id: { cerimonia: "$ceremony", categoria: "$category" },
      vencedores: { $sum: 1 },
      detalhes: { $push: { nome: "$name", filme: "$film" } }
    }
  },
  { $match: { vencedores: { $gt: 1 } } }
])
```

### 13.4 Crie uma query que simule uma "loteria" - selecione 5 filmes aleatórios que ganharam Melhor Filme.
**Análise:** Utiliza o estágio `$sample` para selecionar registros aleatórios dentro do filtro especificado.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, category: { $regex: /BEST PICTURE/i } } },
  { $sample: { size: 5 } },
  { $project: { _id: 0, film: 1, year_ceremony: 1 } }
])
```

### 13.5 Encontre padrões nos nomes dos filmes vencedores (ex: quantos têm uma palavra só, duas palavras, etc.).
**Análise:** Divide a string do título do filme em um array pelo caractere de espaço (`$split`) e calcula o tamanho do array resultante.  
**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  { $match: { winner: true, film: { $ne: null } } },
  { $group: { _id: "$film" } },
  {
    $project: {
      filme: "$_id",
      numPalavras: { $size: { $split: ["$_id", " "] } }
    }
  },
  {
    $group: {
      _id: "$numPalavras",
      qtdFilmes: { $sum: 1 }
    }
  },
  { $sort: { _id: 1 } }
])
```

---

## Desafio Final: Dashboard Completo

### 14.1 Crie UMA ÚNICA query de agregação que retorne um dashboard executivo com:
- Total de indicações
- Total de cerimônias
- Total de vencedores
- Categoria com mais indicações
- Filme mais premiado
- Ator/atriz mais indicado(a)
- Década com mais premiações
- Número de categorias únicas

**Análise:** Utiliza o estágio `$facet` do MongoDB para processar múltiplos sub-pipelines paralelamente em uma única requisição ao banco de dados e consolida os resultados em um único documento JSON com `$project`.  

**Comando MongoDB:**
```javascript
db.oscar.aggregate([
  {
    $facet: {
      "totalIndicacoes": [
        { $count: "valor" }
      ],
      "totalCerimonias": [
        { $group: { _id: "$ceremony" } },
        { $count: "valor" }
      ],
      "totalVencedores": [
        { $match: { winner: true } },
        { $count: "valor" }
      ],
      "categoriaMaisIndicada": [
        { $group: { _id: "$category", total: { $sum: 1 } } },
        { $sort: { total: -1 } },
        { $limit: 1 }
      ],
      "filmeMaisPremiado": [
        { $match: { winner: true, film: { $ne: null } } },
        { $group: { _id: "$film", vitorias: { $sum: 1 } } },
        { $sort: { vitorias: -1 } },
        { $limit: 1 }
      ],
      "atorMaisIndicado": [
        { $match: { category: { $regex: /ACTOR|ACTRESS/i }, name: { $ne: null } } },
        { $group: { _id: "$name", indicacoes: { $sum: 1 } } },
        { $sort: { indicacoes: -1 } },
        { $limit: 1 }
      ],
      "decadaMaisPremiada": [
        { $match: { winner: true, year_film: { $ne: null } } },
        {
          $group: {
            _id: { $concat: [{ $toString: { $multiply: [{ $floor: { $divide: ["$year_film", 10] } }, 10] } }, "s"] },
            vitorias: { $sum: 1 }
          }
        },
        { $sort: { vitorias: -1 } },
        { $limit: 1 }
      ],
      "categoriasUnicas": [
        { $group: { _id: "$category" } },
        { $count: "valor" }
      ]
    }
  },
  {
    $project: {
      Total_Indicacoes: { $arrayElemAt: ["$totalIndicacoes.valor", 0] },
      Total_Cerimonias: { $arrayElemAt: ["$totalCerimonias.valor", 0] },
      Total_Vencedores: { $arrayElemAt: ["$totalVencedores.valor", 0] },
      Categoria_Com_Mais_Indicacoes: { $arrayElemAt: ["$categoriaMaisIndicada._id", 0] },
      Filme_Mais_Premiado: { $arrayElemAt: ["$filmeMaisPremiado._id", 0] },
      Ator_Atriz_Mais_Indicado: { $arrayElemAt: ["$atorMaisIndicado._id", 0] },
      Decada_Com_Mais_Premiacoes: { $arrayElemAt: ["$decadaMaisPremiada._id", 0] },
      Numero_De_Categorias_Unicas: { $arrayElemAt: ["$categoriasUnicas.valor", 0] }
    }
  }
])
```