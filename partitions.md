# Particionamento no SQL Server

## Filegroups

```sql
alter database tpc_h
add filegroup lineitem_Part1

alter database tpc_h
add filegroup lineitem_Part2

alter database tpc_h
add filegroup lineitem_Part3
```

## Arquivos (.ndf)

```sql
alter database tpc_h
add file
(name = part1,
filename = 'C:\Lineitem_Partitions\lineitem_P1.ndf',
size= 10MB,
filegrowth=10%
) to filegroup lineitem_Part1

alter database tpc_h
add file
(name = part2,
filename = 'C:\Lineitem_Partitions\lineitem_P2.ndf',
size= 10MB,
filegrowth=10%
) to filegroup lineitem_Part2

alter database tpc_h
add file
(name = part3,
filename = 'C:\Lineitem_Partitions\lineitem_P3.ndf',
size= 10MB,
filegrowth=10%
) to filegroup lineitem_Part3
```

## Partition Function

```sql
create partition function PF_linenumber (int)
as range left for values (1,3)
```

## Partition Scheme

```sql
create partition scheme scheme_lineitem
as partition PF_linenumber to 
(lineitem_Part1,lineitem_Part2,lineitem_Part3)
```

## Tabela Particionada

```sql
CREATE TABLE [dbo].[lineitem_P](
	[l_orderkey] [int] NULL,
	[l_partkey] [int] NULL,
	[l_suppkey] [int] NULL,
	[l_linenumber] [int] NULL,
	[l_quantity] [real] NULL,
	[l_extendedprice] [real] NULL,
	[l_discount] [real] NULL,
	[l_tax] [real] NULL,
	[l_returnflag] [char](1) NULL,
	[l_linestatus] [char](1) NULL,
	[l_shipDATE] [datetime] NULL,
	[l_commitDATE] [datetime] NULL,
	[l_receiptDATE] [datetime] NULL,
	[l_shipinstruct] [char](25) NULL,
	[l_shipmode] [char](10) NULL,
	[l_comment] [varchar](44) NULL
) 
ON scheme_lineitem (l_linenumber)
```

## Carga de Dados

```sql
insert into lineitem_P
select * from lineitem
```

---

## 2. Entregáveis da Atividade

Realize cada uma das etapas a seguir e capture os prints solicitados como evidência.

### Entregável 1: Verificação dos Arquivos Físicos no Sistema Operacional
* **Objetivo:** Comprovar a criação dos arquivos de partição no disco e sua persistência após a carga de dados.
* **O que fazer:** Acesse a pasta onde os arquivos `.ndf` foram criados e liste os arquivos.
* **Entregável 1:** Print legível do terminal ou do Explorador de Arquivos mostrando obrigatoriamente: **Caminho completo (Path)**, **Nome dos 3 arquivos** e o **Tamanho em disco de cada um**.

---

### Entregável 2: Teste de Roteamento com `$PARTITION`
* **Objetivo:** Compreender como o SQL Server calcula a partição de destino de acordo com os limites definidos em `RANGE LEFT (1, 3)`.
* **O que fazer:** Escreva e execute um comando SQL utilizando a função interna `$PARTITION.PF_linenumber(...)` para testar em qual partição cairiam para os valores de `l_linenumber`. Teste com valores negativos e positivos.
* **Entregável 2::** Print da consulta executada e da grade com os resultados retornados.

---

### Entregável 3: Contagem e Distribuição por Linha e Partição
* **Objetivo:** Praticar a escrita de consultas analíticas para auditar a distribuição física dos dados.
* **O que fazer:** Desenvolva uma consulta SQL que retorne:
  1. Cada valor distinto de `l_linenumber`;
  2. O total de linhas associado a cada um desses valores;
  3. A identificação da partição em que cada valor se encontra (você pode usar a função `$PARTITION.PF_linenumber(...)`)
* **Entregável 3:** Print do seu script SQL e do resultado retornado na tela.

---

### Entregável 4: Comparativo de Desempenho (Tabela Tradicional vs Particionada)
* **Objetivo:** Evidenciar o benefício de *Partition Pruning* (eliminação de partição) na leitura de dados.
* **O que fazer:** 
  1. Habilite a exibição de estatísticas de I/O (`SET STATISTICS IO ON;`) ou o Plano de Execução Real.
  2. Crie uma consulta com filtro (`WHERE`) na coluna de partição (`l_linenumber = 1` por exemplo).
  3. Execute a mesma consulta na tabela original (`lineitem`) e na tabela particionada (`lineitem_P`).
* **Evidência exigida (Print):** Print(s) comparando as métricas obtidas nas duas execuções (leituras lógicas na aba *Messages* ou acessos no *Execution Plan*) e também o tempo de execução, demonstrando a redução de custo na tabela particionada.

---

### Entregável 5: Limpeza e Reset do Ambiente
* **Objetivo:** Restaurar o banco de dados ao estado inicial, removendo objetos na ordem correta de dependência (Tabela -> Esquema -> Função -> Arquivos -> Filegroups).
* **O que fazer:** Execute o script de limpeza abaixo:

```sql
-- 1. Excluir a tabela que usa o esquema
DROP TABLE IF EXISTS dbo.lineitem_P;

-- 2. Excluir o esquema de partição
DROP PARTITION SCHEME scheme_lineitem;

-- 3. Excluir a função de partição
DROP PARTITION FUNCTION PF_linenumber;

-- 4. Remover os arquivos físicos do banco de dados
ALTER DATABASE tpc_h REMOVE FILE part1;
ALTER DATABASE tpc_h REMOVE FILE part2;
ALTER DATABASE tpc_h REMOVE FILE part3;

-- 5. Remover os filegroups
ALTER DATABASE tpc_h REMOVE FILEGROUP lineitem_Part1;
ALTER DATABASE tpc_h REMOVE FILEGROUP lineitem_Part2;
ALTER DATABASE tpc_h REMOVE FILEGROUP lineitem_Part3;
```

* **Evidência exigida (Print):** Print do script + execução bem-sucedida do script no SSMS (aba **Messages** com *"Command(s) completed successfully."*) ou print do terminal/explorador confirmando a remoção/desvinculação dos arquivos.


# *Informações da Entrega:** 

> **Informações da Entrega:** 
> * **Prazo:** Até sexta-feira 23/10
> * **Envio das evidências (prints):** [Upload no Google Drive https://drive.google.com/drive/u/0/folders/1FC4R2uJbOE9oI3ai6RvpvjGNe7HrWtDq](https://drive.google.com/drive/u/0/folders/1FC4R2uJbOE9oI3ai6RvpvjGNe7HrWtDq)
> * Organize os prints utilizando o número do entregável como nome do arquivo. Exemplo: E1.png, E2.png, E3.png.
> * Crie uma pasta no Google Drive para enviar as evidências, identificando-a com a matrícula do aluno ou com as matrículas dos integrantes da equipe.
> * Para entregas individuais: utilize sua matrícula como nome da pasta. Exemplo: 123456.
> * Para entregas em equipe (até 3 integrantes): utilize as matrículas de todos os integrantes, separadas por hífen (-). Exemplo: 123456-234567-345678.