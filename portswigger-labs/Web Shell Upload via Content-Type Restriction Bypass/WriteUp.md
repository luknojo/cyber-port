# Web Shell Upload via Content-Type Restriction Bypass

## Writeup

Neste laboratório, explorei uma vulnerabilidade em uma funcionalidade de upload de avatar que realizava validação baseada apenas no header `Content-Type`, permitindo o bypass da restrição de tipo de arquivo.

Ao tentar realizar o upload de um arquivo PHP contendo um web shell, a aplicação inicialmente bloqueou o envio, aceitando apenas tipos MIME como `image/jpeg` e `image/png`.

No entanto, ao interceptar a requisição com o Burp Suite, foi identificado que a validação do tipo de arquivo era feita apenas com base no valor do header `Content-Type`, sem validação real do conteúdo do arquivo.

Foi possível então modificar a requisição de upload, alterando o `Content-Type` para `image/jpeg`, enquanto mantinha o payload PHP no corpo do arquivo.

Após o upload bem-sucedido, o arquivo foi armazenado no servidor e acessível via `/files/avatars/`.

Ao acessar diretamente o arquivo enviado com extensão `.php`, o servidor executou o código PHP, permitindo a leitura do conteúdo sensível do arquivo `/home/carlos/secret`.

## Resultado

Consegui fazer upload de um arquivo malicioso contornando a validação de tipo MIME e executar código no servidor, obtendo acesso a um arquivo interno sensível.

## Conclusão

A vulnerabilidade ocorre devido à confiança incorreta no header `Content-Type` fornecido pelo cliente, permitindo bypass de validação de upload e execução remota de código (RCE).
