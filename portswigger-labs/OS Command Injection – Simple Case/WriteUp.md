# OS Command Injection – Simple Case

## Writeup

Neste laboratório, explorei uma vulnerabilidade de injeção de comandos no sistema operacional (OS Command Injection) presente na funcionalidade de verificação de estoque de produtos.

Ao interceptar a requisição com o Burp Suite, foi identificado que o parâmetro `storeID` era utilizado diretamente na construção de um comando executado no sistema operacional, sem validação adequada da entrada do usuário.

Durante os testes, foi possível injetar comandos adicionais utilizando o caractere `|`, permitindo a execução de comandos arbitrários no servidor.

O payload utilizado foi:

`2|whoami`

Como resultado, o servidor executou o comando `whoami` e retornou o nome do usuário do sistema na resposta HTTP.

## Resultado

Foi possível executar comandos arbitrários no sistema operacional através da injeção de comandos, obtendo o usuário atual do servidor.

## Conclusão

A vulnerabilidade ocorre devido à concatenação direta de entrada do usuário em comandos do sistema operacional sem sanitização ou validação adequada, permitindo execução remota de comandos (OS Command Injection).
