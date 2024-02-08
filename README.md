Extrair dados de base Mysql para outra base Mysql
Bases sem comunicação direta 
Referencias 
https://www.mysqltutorial.org/mysql-basics/mysql-export-table-to-csv/



Exemplos de opções para realizar processo 

Requisitos 
Mysql

Extrair dados de uma das bases em formato CSV e importalos diretamente do CSV para a nova base.

Vamos utilizar uma tabela de exemplo 

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

E inserir dados na mesma 
-- OBS: criamos o ID como Bynary para deixar o exemplo mais completo
Para salvar dados binary no mysql podemos usar '0x' no inicio do um id hexadecimal, ou usar a função HEX() na saída do dado e UNHEX() na entrada
 --
 
INSERT INTO PESSOA
(ID, NOME, SOBRENOME, DATA_NASCIMENTO, DATA_CADASTRO, DATA_ATUALIZACAO, OBSERVACAO)
VALUES(0x2EF4C83FEB1D46E2AF2F9317490283B1, 'João', 'Pé de Feijão', '1990-01-01', '2024-01-24 10:00:00', '2024-01-24 01:00:00', 'Texto de observação com acentuação');

INSERT INTO PESSOA
(ID, NOME, SOBRENOME, DATA_NASCIMENTO, DATA_CADASTRO, DATA_ATUALIZACAO, OBSERVACAO)
VALUES(0x2EF4C83FEB1D46E2AF2F9317490283B2, 'Cleiton', 'Rasta', '2001-10-01', '2024-01-24 23:05:09', NULL, NULL);
 

Agora criar uma segunda tabela igual a primeira, para exemplificar a exportação dos dados
para facilitar, utilizei o comando 
CREATE TABLE PESSOA_NOVA
SELECT * FROM PESSOA

TRUNCATE PESSOA_NOVA


-----------

Para extrair os dados precisamos saber onde os dados poderão ser armazenados dentro do servidor
a variavel secure_file_priv apresenta este local, então podemos usar o comando:
SHOW VARIABLES LIKE "secure_file_priv"

Para sistemas linux o valor pode ser /var/lib/mysql-files/, para windows dependerá de onde está instalado o Mysql 

------------

Para relizar a extração, é utilizado o comando de select necessário e as informações indicando como será o arquivo na saída e o local de armazenamento
segue exemplo  

SELECT HEX(ID) ,NOME,SOBRENOME,DATA_NASCIMENTO,DATA_CADASTRO, IFNULL(DATA_ATUALIZACAO, "~NULL~") DATA_ATUALIZACAO, IFNULL(OBSERVACAO, "~NULL~") OBSERVACAO FROM PESSOA p
INTO OUTFILE '/var/lib/mysql-files/pessoa.csv' 
FIELDS ENCLOSED BY '"' 
TERMINATED BY ';' 
ESCAPED BY '"' 
LINES TERMINATED BY '\r\n';

O uso de HEX() é necessário para facilitar a decodificação da coluna bynary na importação
O uso de IFNULL é necessário para colunas que são opcionais na tabela, pois caso seja extraída uma coluna NULL, o CSV colocará um caractere 'N' onde deveria ser NULL. Então precisamos substituir os valores nulos com '~NULL~', pois este valor ajudará a identificar estes registros na importação.


Para importar usamos o comando exemplo abaixo


LOAD DATA INFILE '/var/lib/mysql-files/pessoa41.csv' 
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

Usando SET conseguimos substituir os valor ~NULL~, e tambem fazer o unhex do ID que codificamos anteriormente com HEX



