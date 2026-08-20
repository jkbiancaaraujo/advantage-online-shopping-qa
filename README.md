# Advantage Online Shopping — QA

Repositório de documentação de QA para o site [Advantage Online Shopping](https://advantageonlineshopping.com/#/), cobrindo tanto testes funcionais de UI quanto testes de API do endpoint de busca de produtos.

## Conteúdo

| Arquivo | Descrição |
|---|---|
| [advantageonlineshopping-test-scenarios.pt.md](advantageonlineshopping-test-scenarios.pt.md) | Cenários de teste (Gherkin) para a UI do site, incluindo a funcionalidade de busca de produtos. |
| [advantageonlineshopping-search-api-test-scenarios.pt.md](advantageonlineshopping-search-api-test-scenarios.pt.md) | Cenários de teste (Gherkin) para a API pública de busca de produtos (`GET /catalog/api/v1/products/search`), cobrindo contrato, casos negativos, segurança e aspectos não funcionais. |
| [advantageonlineshopping-search-api-bug-report.pt.md](advantageonlineshopping-search-api-bug-report.pt.md) | Relatório de bugs encontrados durante a validação dos cenários de API, com passos de reprodução e severidade. |
| [advantageonlineshopping-search-api.postman_collection.json](advantageonlineshopping-search-api.postman_collection.json) | Coleção Postman com a automação dos cenários de teste da API de busca. |

## Escopo testado

- **UI:** busca de produtos na página inicial do site (resultados válidos, navegação para detalhes do produto, casos de borda como buscas vazias, com espaços, caracteres especiais, etc.).
- **API:** endpoint `GET https://www.advantageonlineshopping.com/catalog/api/v1/products/search`, cobrindo schema de resposta, status codes, tratamento de erros, segurança e métodos HTTP não suportados.

## Como usar

1. Consulte os arquivos de cenários (`*-test-scenarios.pt.md`) para entender a cobertura de testes planejada, escrita em notação Gherkin.
2. Importe o arquivo `advantageonlineshopping-search-api.postman_collection.json` no [Postman](https://www.postman.com/) para executar os testes automatizados da API.
3. Veja o [relatório de bugs](advantageonlineshopping-search-api-bug-report.pt.md) para os defeitos encontrados durante a execução, com severidade e passos de reprodução via `curl`.

### Como usar a coleção Postman

**Via interface do Postman:**

1. Abra o Postman e clique em **Import**.
2. Selecione o arquivo [advantageonlineshopping-search-api.postman_collection.json](advantageonlineshopping-search-api.postman_collection.json) (ou arraste-o para a janela de importação).
3. A coleção "Advantage Online Shopping - Search API" será adicionada com suas pastas (`Funcionalidade 1`, `Funcionalidade 2`, etc.), organizadas pelos mesmos temas dos cenários de teste em Gherkin.
4. A variável de coleção `baseUrl` já vem configurada com `https://www.advantageonlineshopping.com/catalog/api/v1/products/search` — não é necessário criar um Environment separado.
5. Execute uma requisição individualmente (aba **Send**) para ver o resultado e os testes (aba **Test Results**), ou use o **Collection Runner** (botão direito na coleção → *Run collection*) para rodar todos os cenários de uma vez.
6. Cada requisição é independente e pode ser executada isoladamente, em qualquer ordem — requisições que comparam respostas (ex.: case-insensitive) buscam sua própria baseline internamente via `pm.sendRequest`.

**Via linha de comando (Newman):**

```powershell
npm install -g newman
newman run advantageonlineshopping-search-api.postman_collection.json
```

O Newman executa todos os requests e testes da coleção e imprime um resumo com o total de asserts que passaram/falharam, útil para integração em pipelines de CI.

## Notação

- Os cenários seguem o formato **Gherkin** em português.
- Tags utilizadas: `@excecao` (casos negativos/de borda), `@contrato` (contract testing), `@seguranca`, `@naofuncional` e `@defeito` (cenário relacionado a um bug reportado).
