# SQL Injection – WHERE Clause (Hidden Data Retrieval)

## Writeup

Neste laboratório, explorei uma vulnerabilidade de injeção SQL presente no filtro de categoria de produtos.

A aplicação realiza consultas SQL com base na categoria selecionada pelo usuário, utilizando uma query semelhante a:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`


Foi identificado que o parâmetro de categoria não era devidamente sanitizado antes de ser incluído na consulta SQL, permitindo a manipulação direta da query.

---

## Exploração

Ao interceptar a requisição com o Burp Suite, foi possível modificar o valor do parâmetro de categoria para injetar uma condição SQL adicional, alterando o comportamento da consulta.

O objetivo foi contornar a restrição `released = 1`, permitindo a exibição de produtos não liberados.

A injeção fez com que a lógica da query fosse modificada, retornando também registros ocultos no banco de dados.

---

## Resultado

Foi possível recuperar produtos que não estavam disponíveis normalmente na aplicação, explorando a falha de injeção SQL na cláusula WHERE.

---

## Conclusão

A vulnerabilidade ocorre devido à concatenação direta de entrada do usuário em uma consulta SQL sem validação adequada, permitindo manipulação da lógica da query e acesso a dados não autorizados (SQL Injection).
