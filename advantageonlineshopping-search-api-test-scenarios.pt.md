# Cenários de Teste de API — Busca de Produtos (Advantage Online Shopping)

**Endpoint testado:** `GET https://www.advantageonlineshopping.com/catalog/api/v1/products/search`
**Tipo de API:** REST, síncrona, sem autenticação aparente (endpoint público).
**Notação:** Gherkin (Português). Tags: `@contrato` (contract testing), `@excecao` (negativo/edge case), `@seguranca`, `@naofuncional`, `@defeito` (relacionado a um bug reportado separadamente).

> Os cenários abaixo descrevem o **comportamento esperado** do endpoint, mapeados de forma independente do resultado da execução/validação. Os defeitos encontrados durante a exploração estão documentados à parte no [Relatório de Bugs](advantageonlineshopping-search-api-bug-report.pt.md).

---

### Schema de resposta esperado (contrato)
```json
[
  {
    "categoryId": 3,
    "categoryName": "TABLETS",
    "categoryImageId": "tablets",
    "products": [
      {
        "productId": 16,
        "categoryId": 3,
        "productName": "HP ElitePad 1000 G2 Tablet",
        "price": 1009.0,
        "imageUrl": "3100"
      }
    ]
  }
]
```

---

## Funcionalidade 1: Contrato da API — Schema e Status Codes

```gherkin
Funcionalidade: Contrato da API de busca de produtos
  Como um consumidor da API (aplicação cliente/QA de contrato)
  Eu quero que o endpoint de busca respeite um contrato estável e previsível
  Para que integrações não quebrem com mudanças inesperadas de schema, status ou headers

  Contexto:
    Dado que o endpoint base é "GET /catalog/api/v1/products/search"

  @contrato
  Cenário: Busca válida retorna resposta estruturada e pronta para consumo pela aplicação cliente
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o status HTTP deve ser 200 com "Content-Type: application/json"
    E a resposta deve ser um array de categorias, cada uma contendo seus produtos com identificação, nome e preço

  @contrato
  Cenário: Campos numéricos da resposta têm o tipo correto
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o campo "price" de cada produto deve ser um número e não uma string
    E o "categoryId" de cada produto deve ser consistente com o da categoria que o contém

  @contrato @defeito
  Cenário: Busca sem resultados retorna resposta vazia e não um erro
    Quando eu envio uma requisição GET com o parâmetro "name=zzzznotfound123"
    Então o status HTTP deve ser 200
    E o corpo da resposta deve ser um array JSON vazio

  @contrato @defeito
  Cenário: Ausência do parâmetro obrigatório retorna erro estruturado
    Quando eu envio uma requisição GET sem o parâmetro "name"
    Então o status HTTP deve ser 400 com uma mensagem de erro que identifica o parâmetro ausente

  @contrato @defeito
  Cenário: Parâmetro com nome incorreto é tratado como ausência do parâmetro obrigatório
    Quando eu envio uma requisição GET usando "query", "term" ou outro nome de parâmetro em vez de "name"
    Então o servidor deve retornar 400 indicando que o parâmetro "name" é esperado

```

---

## Funcionalidade 2: Busca de Produtos — Casos Funcionais (Happy Path)

```gherkin
Funcionalidade: Busca funcional de produtos via API
  Como um consumidor da API
  Eu quero buscar produtos por nome
  Para obter resultados corretos agrupados por categoria

  @funcional
  Cenário: Busca por termo existente retorna produtos agrupados na categoria correta
    Quando eu envio "GET /catalog/api/v1/products/search?name=tablet"
    Então o status deve ser 200
    E a resposta deve conter a categoria "TABLETS" com os produtos correspondentes e preços corretos

  @funcional
  Cenário: Busca funciona por termo parcial e é insensível a maiúsculas e minúsculas
    Quando eu busco por "tab" e por "TABLET"
    Então ambas as buscas devem retornar produtos da categoria "TABLETS"
    E a busca por "TABLET" deve retornar o mesmo resultado que a busca por "tablet"

  @funcional
  Cenário: Busca com parâmetro vazio retorna o catálogo completo agrupado por categoria
    Quando eu envio "GET /catalog/api/v1/products/search?name="
    Então o status deve ser 200
    E a resposta deve conter todas as categorias do catálogo com seus respectivos produtos

  @funcional
  Cenário: Parâmetros extras são ignorados sem afetar o resultado
    Quando eu envio "GET /catalog/api/v1/products/search?name=tablet&foo=bar"
    Então o resultado deve ser idêntico ao de uma busca apenas com "name=tablet"

  @excecao
  Cenário: Busca contendo apenas espaço é tratada como busca vazia
    Quando eu envio "GET /catalog/api/v1/products/search?name=%20"
    Então o comportamento deve ser equivalente ao de uma busca com "name" vazio
```

---

## Funcionalidade 3: Entradas Inválidas, Segurança e Robustez

```gherkin
Funcionalidade: Robustez e segurança do endpoint de busca
  Como QA especializado em segurança e robustez de API
  Eu quero validar que o endpoint trata entradas maliciosas ou inesperadas com segurança
  Para reduzir riscos de vulnerabilidades e falhas de serviço

  @seguranca
  Cenário: Respostas incluem os headers de segurança obrigatórios
    Quando eu envio qualquer requisição válida ao endpoint
    Então a resposta deve conter os headers "X-Content-Type-Options", "X-Frame-Options" e "Strict-Transport-Security"

  @seguranca @defeito
  Cenário: Erros não expõem detalhes internos da implementação
    Quando eu envio uma requisição sem o parâmetro obrigatório "name"
    Então o status deve ser 400
    E o corpo da resposta não deve conter stack traces, nomes de classes internas ou versão do servidor

  @excecao
  Cenário: Caracteres Unicode são aceitos sem erro de encoding
    Quando eu envio "GET /catalog/api/v1/products/search?name=tâblet"
    Então o status deve ser 200 e a aplicação não deve retornar erro de encoding

  @excecao
  Cenário: Entrada extremamente longa não derruba o serviço
    Quando eu envio uma busca com um valor de "name" de 5000 caracteres
    Então o serviço deve responder com 200, 400 ou 413 de forma controlada, sem retornar 500 nem travar

  @naofuncional
  Cenário: A política de cache não serve resultados desatualizados entre buscas diferentes
    Quando eu realizo buscas consecutivas por termos distintos
    Então o header "Cache-Control" deve refletir a política correta do endpoint
    E os resultados de uma busca não devem ser servidos como resposta de outra
```

---

## Funcionalidade 4: Verbos HTTP e Negociação de Método

```gherkin
Funcionalidade: Suporte e restrição correta de verbos HTTP
  Como consumidor da API
  Eu quero que apenas os métodos HTTP suportados sejam aceitos
  E que métodos não suportados retornem o status correto (405), não um erro genérico

  @contrato @defeito
  Esquema do Cenário: Métodos de escrita não suportados devem retornar 405 Method Not Allowed
    Quando eu envio "<metodo> /catalog/api/v1/products/search?name=tablet"
    Então o status deve ser 405 (Method Not Allowed)

    Exemplos:
      | metodo |
      | POST   |
      | PUT    |
      | DELETE |

```
---

## Resumo de Cobertura

| Área | Casos Funcionais | Contrato | Exceção / Segurança | Não Funcional |
|---|---|---|---|---|
| Busca por nome | Termo existente com categoria correta, parcial + case-insensitive (unificados), vazio retorna catálogo, parâmetros extras ignorados | Estrutura e tipos da resposta, status 200 + Content-Type | Espaço como busca vazia, Unicode, entrada muito longa | — |
| Tratamento de erro | — | Parâmetro ausente (400), parâmetro com nome incorreto (400) | Stack trace não exposto em erros | — |
| Verbos HTTP | — | Métodos não suportados retornam 405 | — | — |
| Resposta vazia | — | Busca sem resultados retorna array vazio com status 200 | — | — |
| Cache | — | — | — | Política de `Cache-Control` entre buscas distintas |
| Segurança | — | — | Headers de segurança obrigatórios presentes | — |

---

Consulte o [Relatório de Bugs](advantageonlineshopping-search-api-bug-report.pt.md) para o resultado da execução destes cenários e os defeitos encontrados.
