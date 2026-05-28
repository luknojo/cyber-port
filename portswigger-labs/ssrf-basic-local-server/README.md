# SSRF Basic - Access to Local Server

## Visão geral
Este laboratório explora uma vulnerabilidade de Server-Side Request Forgery (SSRF), onde o servidor realiza requisições HTTP controladas pelo usuário, permitindo acesso a recursos internos da aplicação que não estariam expostos externamente.

## Objetivo
Explorar a funcionalidade de verificação de estoque para forçar o servidor a realizar requisições para endpoints internos da aplicação.

## Ferramentas utilizadas
- Burp Suite

## Análise da aplicação
Ao interagir com a funcionalidade "Check stock", foi interceptada uma requisição contendo o parâmetro `stockApi`, que define a URL que será acessada pelo servidor para buscar informações de estoque.

## Exploração
Foi identificado que o parâmetro `stockApi` é totalmente controlado pelo cliente. Ao modificar seu valor para `http://localhost/admin`, o servidor realizou uma requisição interna para o próprio ambiente e retornou o conteúdo da página administrativa.

## Enumeração interna
Através da resposta retornada, foi possível identificar um endpoint sensível de administração: `http://localhost/admin/delete?username=carlos`.

## Resultado
A vulnerabilidade SSRF permitiu acesso a recursos internos da aplicação, incluindo painel administrativo e descoberta de endpoint sensível com capacidade de exclusão de usuário.

## Impacto de segurança
SSRF pode ser explorado para acessar serviços internos não expostos publicamente, como painéis administrativos, serviços de infraestrutura e metadados em ambientes cloud. Em cenários reais, isso pode levar a comprometimento crítico da aplicação.

## Conclusão
A aplicação não valida corretamente o parâmetro `stockApi`, permitindo requisições arbitrárias pelo servidor, caracterizando uma vulnerabilidade de SSRF explorável.
