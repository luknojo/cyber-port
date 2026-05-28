# SQL Injection – Login Bypass (Administrator Access)

## Writeup

Neste laboratório, explorei uma vulnerabilidade de injeção SQL na funcionalidade de login da aplicação.

A aplicação constrói consultas SQL para autenticação de usuários sem sanitizar corretamente a entrada fornecida pelo usuário.

---

## Exploração

Ao interceptar a requisição de login com o Burp Suite, foi identificado que os parâmetros `username` e `password` eram utilizados diretamente na consulta SQL de autenticação.

Foi então possível manipular o parâmetro `username` para alterar a lógica da query e contornar a verificação de senha.

O payload utilizado foi:

`administrator'--`


O operador `--` foi utilizado para comentar o restante da consulta SQL, removendo a verificação da senha.

---

## Resultado

Foi possível realizar login como o usuário `administrator` sem conhecer a senha, explorando a vulnerabilidade de SQL Injection na autenticação.

---

## Conclusão

A vulnerabilidade ocorre devido à concatenação direta de entrada do usuário em uma consulta SQL de autenticação, permitindo manipulação da lógica de login e bypass de credenciais (SQL Injection em login form).

<img width="1035" height="610" alt="image" src="https://github.com/user-attachments/assets/a4ea57f8-e3f7-4c49-878b-a21fe4b0ed5a" />
<img width="1347" height="486" alt="image" src="https://github.com/user-attachments/assets/c6a996ad-e073-4c08-b4b3-ad595370d2c9" />

