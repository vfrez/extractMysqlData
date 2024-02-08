Importar com dump

referencias 
https://idiallo.com/blog/mysql-dump-table-where
https://www.digitalocean.com/community/tutorials/how-to-import-and-export-databases-in-mysql-or-mariadb

-- Exporta todos os registros 
mysqldump -u root -p importer --tables PESSOA > dumpPessoa2.sql
-- Exporta registros com where
mysqldump -u root -p importer --tables PESSOA --where="NOME = 'Cleiton' " > dumpPessoa10.sql


-- Importa os registros 
mysql -u root -p importer2  < dumpPessoa3.sql