# Gerenciamento de Banco de Dados 

  

## 1. **Criar um Banco de Dados** 

   ```sql 

   CREATE DATABASE nome_do_banco; 

   ``` 

   - **Descrição:** Cria um novo banco de dados vazio com o nome especificado.   

   - **Exemplo:** `CREATE DATABASE empresa;` 

  

## 2. **Excluir um Banco de Dados** 

   ```sql 

   DROP DATABASE nome_do_banco; 

   ``` 

   - **Descrição:** Remove o banco de dados especificado e todos os seus dados. **Atenção:** Essa operação é irreversível. 

   - **Exemplo:** `DROP DATABASE empresa;` 

  

## 3. **Selecionar um Banco de Dados** 

   ```sql 

   USE nome_do_banco; 

   ``` 

   - **Descrição:** Define o banco de dados a ser utilizado para as operações subsequentes. 

   - **Exemplo:** `USE empresa;` 

  

## 4. **Criar uma Tabela** 

   ```sql 

   CREATE TABLE nome_da_tabela ( 

       coluna1 tipo_de_dado, 

       coluna2 tipo_de_dado, 

       ... 

   ); 

   ``` 

   - **Descrição:** Cria uma nova tabela dentro do banco de dados ativo, especificando colunas e tipos de dados. 

   - **Exemplo:**  

     ```sql 

     CREATE TABLE funcionarios ( 

         id INT PRIMARY KEY, 

         nome VARCHAR(100), 

         salario DECIMAL(10, 2) 

     ); 

     ``` 

  

## 5. **Excluir uma Tabela** 

   ```sql 

   DROP TABLE nome_da_tabela; 

   ``` 

   - **Descrição:** Remove a tabela especificada do banco de dados, excluindo todos os seus dados. **Atenção:** Essa operação é irreversível. 

   - **Exemplo:** `DROP TABLE funcionarios;` 

  

## 6. **Renomear uma Tabela** 

   ```sql 

   ALTER TABLE nome_antigo RENAME TO nome_novo; 

   ``` 

   - **Descrição:** Altera o nome de uma tabela existente para um novo nome. 

   - **Exemplo:** `ALTER TABLE funcionarios RENAME TO colaboradores;` 

  

## 7. **Adicionar Coluna a uma Tabela** 

   ```sql 

   ALTER TABLE nome_da_tabela ADD nome_da_coluna tipo_de_dado; 

   ``` 

   - **Descrição:** Adiciona uma nova coluna a uma tabela existente. 

   - **Exemplo:** `ALTER TABLE funcionarios ADD data_contratacao DATE;` 

  

## 8. **Remover Coluna de uma Tabela** 

   ```sql 

   ALTER TABLE nome_da_tabela DROP COLUMN nome_da_coluna; 

   ``` 

   - **Descrição:** Remove uma coluna específica de uma tabela existente. 

   - **Exemplo:** `ALTER TABLE funcionarios DROP COLUMN data_contratacao;` 

  

## 9. **Excluir Todos os Dados de uma Tabela** 

   ```sql 

   TRUNCATE TABLE nome_da_tabela; 

   ``` 

   - **Descrição:** Remove todos os dados de uma tabela, mas mantém a estrutura da tabela. 

   - **Exemplo:** `TRUNCATE TABLE funcionarios;` 

  

## 10. **Excluir Registros Específicos de uma Tabela** 

   ```sql 

   DELETE FROM nome_da_tabela WHERE condição; 

   ``` 

   - **Descrição:** Exclui registros específicos de uma tabela com base em uma condição. Se nenhuma condição for fornecida, todos os registros serão excluídos. 

   - **Exemplo:** `DELETE FROM funcionarios WHERE id = 5;` 

  

## 11. **Alterar o Tipo de Dados de uma Coluna** 

   ```sql 

   ALTER TABLE nome_da_tabela  

   MODIFY COLUMN nome_da_coluna novo_tipo_de_dado; 

   ``` 

   - **Descrição:** Modifica o tipo de dados de uma coluna existente na tabela. 

   - **Exemplo:** `ALTER TABLE funcionarios MODIFY COLUMN salario DECIMAL(15, 2);` 

 

## 12. **Atualizar Dados em uma Tabela** 

   ```sql 

   UPDATE nome_da_tabela  

   SET nome_da_coluna = novo_valor  

   WHERE condição; 

   ``` 

   - **Descrição:** Atualiza os dados de uma ou mais colunas em registros específicos de uma tabela, com base em uma condição. Se a condição não for especificada, todos os registros serão atualizados. 

   - **Exemplo:**  

     ```sql 

     UPDATE funcionarios  

     SET salario = salario * 1.10  

     WHERE id = 3; 

     ``` 

 

--- 

  

Esse comando é fundamental para modificar os dados existentes em uma tabela sem a necessidade de excluir e reinserir registros. Caso precise de mais detalhes ou novos comandos, estou à disposição! 