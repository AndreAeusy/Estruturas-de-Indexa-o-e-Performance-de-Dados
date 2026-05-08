# Estruturas-de-Indexa-o-e-Performance-de-Dados
# Introdução

A indexação é um recurso utilizado em bancos de dados para acelerar buscas e consultas. Sem índices, o sistema precisa percorrer todos os registros de uma tabela para encontrar uma informação, o que aumenta muito o tempo de resposta. Os índices funcionam como um catálogo de biblioteca, organizando os dados para que sejam encontrados de forma rápida e eficiente.

---

# 1. Fundamentos e Estruturas

## O que são índices?

Índices são estruturas de dados criadas para melhorar a velocidade de consultas em bancos de dados. Eles armazenam referências organizadas dos dados de uma tabela, permitindo localizar informações sem precisar ler todos os registros.

Exemplo:
Sem índice → o banco faz uma varredura completa na tabela (Full Scan).

Com índice → o banco encontra o dado diretamente pelo caminho organizado do índice.

---

## Estrutura B-Tree (Árvore B)

A B-Tree é a estrutura de índice mais utilizada em bancos relacionais como o PostgreSQL. Ela organiza os dados em forma de árvore balanceada.

### Características:

* Mantém os dados ordenados.
* Excelente para:

  * buscas exatas;
  * intervalos de valores;
  * ordenações (`ORDER BY`);
  * buscas por prefixo.

---

## Analogia com Dicionário

Uma Árvore B funciona como um dicionário físico.

Imagine procurar a palavra “Banco”:

* Você não lê página por página.
* Primeiro abre aproximadamente no meio.
* Depois escolhe a seção correta.
* Continua dividindo até encontrar a palavra.

A Árvore B faz exatamente isso:

* divide os dados em blocos organizados;
* elimina grandes partes da busca;
* encontra rapidamente o valor desejado.

---

## Estrutura Hash

O índice Hash funciona através de uma função matemática chamada “hash”, que transforma um valor em um endereço específico.

### Características:

* Muito rápido para igualdade:

  ```sql
  WHERE id = 10
  ```
* Não funciona bem para:

  * intervalos;
  * ordenações;
  * buscas por prefixo.

---

## Diferença entre B-Tree e Hash

| Característica      | B-Tree            | Hash        |
| ------------------- | ----------------- | ----------- |
| Busca exata         | Excelente         | Excelente   |
| Busca por intervalo | Sim               | Não         |
| Ordenação           | Sim               | Não         |
| Prefixo de texto    | Sim               | Não         |
| Estrutura           | Árvore balanceada | Tabela Hash |

---

# 2. Densidade de Índices

## Índice Denso

No índice denso existe uma entrada para cada registro da tabela.

### Exemplo:

| ID | Nome   |
| -- | ------ |
| 1  | Ana    |
| 2  | Carlos |
| 3  | João   |

Índice:

| Chave | Ponteiro |
| ----- | -------- |
| 1     | Linha 1  |
| 2     | Linha 2  |
| 3     | Linha 3  |

### Vantagens:

* Busca extremamente rápida.

### Desvantagens:

* Maior uso de armazenamento.

---

## Índice Esparso

No índice esparso existem entradas apenas para alguns blocos da tabela.

### Exemplo:

| Chave | Ponteiro |
| ----- | -------- |
| 1     | Bloco A  |
| 100   | Bloco B  |
| 200   | Bloco C  |

### Vantagens:

* Menor espaço em disco.

### Desvantagens:

* Busca um pouco mais lenta.

---

## Quando utilizar?

| Tipo    | Uso Ideal                               |
| ------- | --------------------------------------- |
| Denso   | Sistemas com consultas muito frequentes |
| Esparso | Grandes volumes de dados organizados    |

---

# 3. Tomada de Decisão e Cardinalidade

## Chave Primária (ID)

### Melhor índice:

B-Tree.

### Justificativa:

* IDs normalmente são únicos.
* Precisam de buscas rápidas.
* Permite ordenação e intervalos.
* É o padrão do PostgreSQL.

---

## Campo Nome

### Melhor índice:

B-Tree.

### Justificativa:

Funciona muito bem para buscas por prefixo:

```sql
WHERE nome LIKE 'Ana%'
```

A árvore consegue navegar alfabeticamente.

---

## Campo Cidade

### Situação:

Milhões de registros, mas poucas cidades.

### Análise:

Nesse caso a cardinalidade é baixa.

Exemplo:

* 10 milhões de registros;
* apenas 20 cidades diferentes.

### Resultado:

O índice pode não valer a pena, porque o banco pode preferir ler toda a tabela ao invés de usar o índice.

---

## Campo Sexo

### Cardinalidade:

Muito baixa.

Exemplo:

* M
* F

### Conclusão:

Normalmente não é recomendado criar índice.

### Motivo:

O índice teria pouca eficiência, pois metade da tabela provavelmente teria o mesmo valor.

Além disso:

* aumenta custo de escrita;
* aumenta espaço em disco;
* dificulta atualizações.

---

# 4. Ecossistema PostgreSQL

## B-Tree

Mais comum no PostgreSQL.

### Utilizado para:

* igualdade;
* intervalos;
* ordenação;
* prefixos.

---

## GIN (Generalized Inverted Index)

Ideal para:

* arrays;
* JSON;
* Full Text Search.

Muito utilizado em buscas textuais complexas.

---

## GIST (Generalized Search Tree)

Utilizado para:

* dados geográficos;
* coordenadas;
* buscas espaciais.

Muito usado com PostGIS.

---

## BRIN (Block Range Index)

Ideal para tabelas gigantes.

### Funcionamento:

Armazena resumos de blocos de dados ao invés de cada linha.

### Vantagem:

* ocupa pouco espaço;
* ótimo para tabelas ordenadas por data.

---

# Criando um índice no PostgreSQL

## Exemplo SQL

```sql
CREATE INDEX idx_email
ON usuarios(email);
```

---

# Verificando uso do índice

```sql
EXPLAIN ANALYZE
SELECT * FROM usuarios
WHERE email = 'teste@email.com';
```

### Resultado esperado:

Se o índice estiver sendo utilizado, aparecerá algo como:

```sql
Index Scan using idx_email
```

---

# 5. Alta Performance com Elasticsearch

## Índice Invertido

O Elasticsearch utiliza Índice Invertido.

### Funcionamento:

Ao invés de organizar registros em árvore, ele organiza palavras.

Exemplo:

| Palavra | Documentos |
| ------- | ---------- |
| banco   | 1, 5, 8    |
| dados   | 2, 5, 9    |

Assim, ao pesquisar “banco”, ele encontra imediatamente os documentos relacionados.

---

## Quando usar Elasticsearch?

O Elasticsearch é recomendado quando temos:

* buscas textuais avançadas;
* grande volume de documentos;
* necessidade de relevância;
* autocomplete;
* análise textual;
* alta velocidade em pesquisas.

---

## Quando manter no PostgreSQL?

O PostgreSQL é melhor para:

* transações;
* integridade dos dados;
* relacionamentos;
* operações CRUD tradicionais.

---

# Parte 2 — Apresentação Oral

# Possível Pergunta do Professor

## “Por que não podemos indexar todas as colunas?”

### Resposta:

Porque índices possuem custo.

Embora acelerem consultas, eles:

* aumentam o uso de armazenamento;
* deixam INSERT, UPDATE e DELETE mais lentos;
* exigem manutenção constante;
* podem piorar a performance se usados em colunas inadequadas.

Por isso, é necessário analisar:

* frequência das consultas;
* cardinalidade;
* tipo de busca realizada.

---

# Conclusão

A indexação é essencial para otimizar bancos de dados, mas deve ser aplicada estrategicamente. Estruturas como B-Tree, Hash e Índice Invertido possuem objetivos diferentes e cada cenário exige uma escolha adequada. Entender cardinalidade, custo e tipo de consulta é fundamental para garantir alta performance e eficiência no sistema.
