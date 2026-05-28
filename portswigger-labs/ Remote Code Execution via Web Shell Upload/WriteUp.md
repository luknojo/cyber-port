# Remote Code Execution via Web Shell Upload

## Writeup

Neste laboratório, explorei uma vulnerabilidade de upload de arquivos que permitia a execução remota de código no servidor (RCE).

A aplicação possuía uma funcionalidade de upload de imagem de perfil, onde os arquivos enviados eram armazenados no servidor e posteriormente acessados via uma URL pública.

Ao interceptar as requisições com o Burp Suite, foi possível observar que não havia validação adequada do tipo de arquivo enviado, permitindo o upload de arquivos com extensão `.php`.

Após o upload de um arquivo PHP contendo um web shell simples, foi identificado que o servidor executava o arquivo ao ser acessado diretamente via URL.

O payload utilizado foi:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
