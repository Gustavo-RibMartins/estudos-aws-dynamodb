# Estudos AWS DynamoDB

Estudos do AWS DynamoDB.

> **Sumário**
>
> - [1. Introdução](#1-introdução)
> - [2. Conceitos e Componentes do DynamoDB](#2-conceitos-e-componentes-do-dynamodb)
> - [3. Índices Secundários](#3-índices-secundários)

---

### 1. Introdução

Banco de dados NoSQL de latência extremamente baixa (< 10 milissegundos) para aplicações de alto desempenho. Totalmente gerenciado e escalável. Muito recomendado para utilização com App Web/ Mobile e Games. Possui replicação entre multiplas AZs.

**NoSQL Databases**

* NoSQL Databases são não-relacionais e distribuídos (horizontal scale);
* Não suportam query joins (ou possuem suporte limitado);
* Todos os dados necessários para a query estão presentes em uma linha;
* Não performam agregações como SUM, AVG, ...

**Performance**

* Milhões de requisições por segundo;
* Trilhões de linhas;
* Centenas de TB de storage;
* Rápido e consistente em performance (baixa latência em retrieval).

Dynamo DB oferece criptografia em repouso, te permite aumentar e diminuir a taxa de transferência da tabela, oferece recursos de backup sob demanda e backups contínuos com recuperação point-in-time (rollback para timestamp nos últimos 35 dias) e te permite excluir automaticamente itens exporados da tabela (TTL - Time to Live).

Os dados do DynamoDB são armazenados em discos SSD e automaticamente replicados entre várias AZs (>= 3 AZs). Você pode usar **tabelas globais** para manter as tabelas do DynamoDB sincronizadas.

---

### 2. Conceitos e Componentes do DynamoDB

- **Tabela**: Coleção de itens. No DynamoDB você não cria "bancos de dados", cria diretamente uma Tabela. A Tabela no DynamoDB tem o mesmo conceito que uma tabela no BD Relacional.

- **Itens**: Coleção de atributos. Cada item é identificado exclusivamente por uma **Chave Primária (PK)** informada no momento de criação da tabela. O item é o equivalente a uma linha no BD Relacional. No DynamoDB, não há limites para o número de itens que a sua tabela pode ter.

- **Atributos**: elemento de dados fundamental (nome, data de nascimento, funcional, squad). Os atributos são o equivalente das colunas do BD Relacional, porém mais poderosos pois aceitam valores aninhados (nested attributes). Os atributos podem ser de dois tipos:

    - Atributo Escalar: possui apenas um valor;
    - Atributo Aninhado/ Composto: possui mais de um valor. DynamoDB suporta atributos aninhados com até 32 níveis de profundidade. É como se fosse um mapa do Python.

> Obs.: o tamanho máximo de um único item (row) é 400KB.

Data types suportados para os atributos:

- Tipos escalares:
    
    - String
    - Number
    - Binary
    - Boolean
    - Null

- Tipo de documento:

    - Lista
    - Mapa

- **Chave Primária (PK)**: ao criar a tabela, você deve obrigatoriamente especificar a PK. DynamoDB é compatível com dois tipos de chaves primárias:

    - **Partition Key**: chave primária simples, formada por um atributo escalar. Deve ser única para cada item;
    - **Partition Key + Sort Key**: chave primária composta, formada por dois atributos escalares. A combinação da PK e da SK precisa ser única pra cada item.

![](/Imagens/primary-keys.png "Exemplo de uma tabela de usuários com Primary Key Simples")

![](/Imagens/primary-keys-2.png "Exemplo de uma tabela de user-games com Primary Key Composta")

DynamoDB utiliza a Partition Key como entrada para uma função de hash interna. A saída dessa função de hash determina a partição (onde o item com aquele partition key será armazenado fisicamente no DynamoDB). Todos os itens com o mesmo partition key são armazenados juntos e ordenados pelo sort key.

Cada atributo da chave primária precisa ser um atributo escalar (string, number, boolean) e uma boa prática, é escolher um partition key que distribua os itens o máximo possível de mode que evite que muitos itens sejam armazenados juntos em uma mesma partição. Por exemplo, escolher ID_SQUAD como partition key é uma escolha ruim, pois todos os membros vão ser armazenados na mesma partição e suqads muito grandes sentirão o impacto do problema de **Hot Key** (partição acessada muitas vezes). Uma boa escolha seria usar a FUNCIONAL como Partition Key.

**Table Class**

- Standard: default table para propósitos gerais para tabelas que armazenam dados acessados com frequência;
- Standart-IA: para tabelas que armazenam dados que não são acessados com frequência. Possui custo de armazenamento menor.

**Read/Write Capacity**

- On-demand: capacidade de leitura/ escrita se adapta as necessidades da aplicação;
- Provisionada: o usuário define quanto irá alocar de capacidade de leitura/ escrita para a tabela.

**Criptografia**

É possível criptografar os dados em repouso utilizando:

- Amazon DynamoDB Key: uma chave do KMS gerenciada pelo DynamoDB. Não há cobrança adicional por usá-la;
- AWS Managed Key: chave do KMS informada pelo usuário. Há custos do KMS envolvidos;
- Key gerenciada pelo cliente: Chave KMS criada e gerenciada pelo cliente. Há custos do KMS envolvidos.

---

### 3. Índices Secundários