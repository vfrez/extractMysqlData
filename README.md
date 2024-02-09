# Extrair dados de uma tabela em banco Mysql.

**Databases sem comunicação direta.**
Exemplos de opções para realização do processo 

## Referencias
https://www.mysqltutorial.org/mysql-basics/mysql-export-table-to-csv/
https://stackoverflow.com/questions/2675323/mysql-load-null-values-from-csv-data

## Requisitos
 - Mysql - client ou docker
 - IDE para facilitar(DBeaver, MySQL Workbench, etc)

## Extrair dados em formato CSV e importa-los para a nova base.

Para este exemplo precisaremos de dois databases, um para extrair os dados e outro para importar.

### Preparação de ambiente

Criar a seguinte tabela em ambos os databases
```sql
CREATE TABLE `PESSOA` (
  `ID` binary(16) NOT NULL,
  `NOME` varchar(100) NOT NULL,
  `SOBRENOME` varchar(250) DEFAULT NULL,
  `DATA_NASCIMENTO` date DEFAULT NULL,
  `DATA_CADASTRO` datetime NOT NULL,
  `DATA_ATUALIZACAO` datetime DEFAULT NULL,
  `OBSERVACAO` varchar(1000) DEFAULT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
> OBS: criamos o ID como **binary** para deixar o exemplo mais completo.
> Para salvar dados **binary** no mysql podemos usar '0x' no inicio do um hexadecimal, ou usar a função `HEX()` na exportação do dado e `UNHEX()` na importação.

Em uma delas insira alguns registros, como por exemplo:

```sql
INSERT INTO PESSOA
(ID, NOME, SOBRENOME, DATA_NASCIMENTO, DATA_CADASTRO, DATA_ATUALIZACAO, OBSERVACAO)
VALUES(0x2EF4C83FEB1D46E2AF2F9317490283B1, 'João', 'Pé de Feijão', '1990-01-01', '2024-01-24 10:00:00', '2024-01-24 01:00:00', 'Texto de observação com acentuação');

INSERT INTO PESSOA
(ID, NOME, SOBRENOME, DATA_NASCIMENTO, DATA_CADASTRO, DATA_ATUALIZACAO, OBSERVACAO)
VALUES(0x2EF4C83FEB1D46E2AF2F9317490283B2, 'Cleiton', 'Rasta', '2001-10-01', '2024-01-24 23:05:09', NULL, NULL);
```
### Extrair dados
Para extrair os dados precisamos saber onde os dados poderão ser armazenados dentro do servidor.
Para isso pode-se usar a variável do Mysql `secure_file_priv` que armazena o diretório onde é possível armazenar dados. Usando o comando abaixo:
```sql
SHOW VARIABLES LIKE "secure_file_priv"
```

Para sistemas Linux o valor pode ser `/var/lib/mysql-files/`, para Windows dependerá de onde está instalado o Mysql 

Para realizar a extração, é utilizado de `SELECT` necessário e as informações indicando como será o arquivo na saída e o local de armazenamento
segue exemplo :

```sql
SELECT HEX(ID) ,NOME,SOBRENOME,DATA_NASCIMENTO,DATA_CADASTRO, IFNULL(DATA_ATUALIZACAO, "~NULL~") DATA_ATUALIZACAO, IFNULL(OBSERVACAO, "~NULL~") OBSERVACAO FROM PESSOA p
INTO OUTFILE '/var/lib/mysql-files/pessoa.csv' 
FIELDS ENCLOSED BY '"' 
TERMINATED BY ';' 
ESCAPED BY '"' 
LINES TERMINATED BY '\r\n';
```

O uso da função `HEX()` é necessário para facilitar a decodificação da coluna de tipo `binary` na importação

O uso da função `IFNULL` é necessário para colunas que são opcionais na tabela, pois caso seja extraída uma coluna com valor nulo, no CSV estará o caractere 'N' no lugar, e isso não deu erros em testes locais. 
Então para contornar este problema substituiremos os valores nulos com o texto `~NULL~`, ajudará a identificar estes registros na importação.


### Importar dados
Para importar usamos o comando exemplo abaixo
Muito parecido com o comando para extração

O valor de `LOAD DATA INFILE` deve conter o diretório com o arquivo CSV gerado na extração.

```sql
LOAD DATA INFILE '/var/lib/mysql-files/pessoa.csv' 
INTO TABLE PESSOA2
FIELDS ENCLOSED BY '"' 
TERMINATED BY ';' 
ESCAPED BY '"' 
LINES TERMINATED BY '\r\n'
(@vId,NOME,SOBRENOME,DATA_NASCIMENTO,DATA_CADASTRO, @vDtAt,  @vObs)
SET 
ID = unhex(@vId),
DATA_ATUALIZACAO = NULLIF(@vDtAt,'~NULL~'),
OBSERVACAO = NULLIF(@vObs,'~NULL~');
```

> OBS: É importante que os valores **ENCLOSE**, **TERMINATED**, **ESCAPED**, **TERMINATED** sejam iguais aos usados na extração. Assim garantimos a compatibilidade na importação.

Usando SET conseguimos substituir os valores em tempo de importação. Assim conseguimos indicar o uso da função `UNHEX()` para a coluna de tipo `binary`, assim como os registros com valor os valor `~NULL~`.
