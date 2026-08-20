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
  Cenário: Requisição válida retorna status 200 e Content-Type JSON
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o status HTTP da resposta deve ser 200
    E o header "Content-Type" deve estar presente e conter "application/json"

  @contrato
  Cenário: O corpo da resposta segue o schema esperado (array de categorias)
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o corpo da resposta deve ser um array JSON
    E cada item do array deve conter os campos "categoryId" (inteiro), "categoryName" (string) e "categoryImageId" (string)
    E cada item deve conter um campo "products" do tipo array
    E cada produto dentro de "products" deve conter "productId" (inteiro), "categoryId" (inteiro), "productName" (string), "price" (número) e "imageUrl" (string)
    E nenhum campo adicional não documentado deve estar presente (validação estrita de schema)

  @contrato
  Cenário: O tipo de dado do campo "price" é numérico e não uma string
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o campo "price" de cada produto deve ser um número (float), nunca uma string como "1009.00"

  @contrato
  Cenário: O campo "categoryId" do produto é consistente com o "categoryId" da categoria pai
    Quando eu envio uma requisição GET com o parâmetro "name=tablet"
    Então o "categoryId" de cada produto dentro de uma categoria deve ser igual ao "categoryId" da categoria que o contém

  @contrato @defeito
  Cenário: Busca sem resultados deve retornar um array JSON vazio
    Quando eu envio uma requisição GET com o parâmetro "name=zzzznotfound123" (termo sem correspondência)
    Então o status HTTP da resposta deve ser 200
    E o corpo da resposta deve ser um array JSON vazio ("[]")

  @contrato @defeito
  Cenário: Parâmetro obrigatório ausente deve retornar 400
    Quando eu envio uma requisição GET para o endpoint sem o parâmetro "name" (obrigatório)
    Então o status HTTP esperado é 400 (Bad Request) com uma mensagem de erro estruturada

  @contrato @defeito
  Cenário: Nome de parâmetro inesperado deve retornar 400
    Quando eu envio uma requisição GET usando um nome de parâmetro diferente de "name" (ex.: "query", "term", "viewAll", "searchTerm")
    Então o servidor deve validar a ausência do parâmetro "name" e retornar 400 informando o parâmetro esperado

  @contrato
  Cenário: Content-Type da requisição não deveria impactar uma requisição GET simples
    Quando eu envio uma requisição GET com o parâmetro "name=tablet" sem header "Content-Type"
    Então a resposta deve ser 200 normalmente, já que GET não deveria exigir corpo/Content-Type

```

---

## Funcionalidade 2: Busca de Produtos — Casos Funcionais (Happy Path)

```gherkin
Funcionalidade: Busca funcional de produtos via API
  Como um consumidor da API
  Eu quero buscar produtos por nome
  Para obter resultados corretos agrupados por categoria

  @funcional
  Cenário: Buscar por um termo existente retorna produtos da categoria correta
    Quando eu envio "GET /catalog/api/v1/products/search?name=tablet"
    Então o status deve ser 200
    E a resposta deve conter a categoria "TABLETS"
    E a categoria "TABLETS" deve conter ao menos os produtos "HP ElitePad 1000 G2 Tablet", "HP Elite x2 1011 G1 Tablet" e "HP Pro Tablet 608 G1"
    E os preços retornados devem corresponder aos preços reais desses produtos

  @funcional
  Cenário: Busca por termo parcial (substring) retorna produtos correspondentes
    Quando eu envio "GET /catalog/api/v1/products/search?name=tab"
    Então o status deve ser 200
    E a categoria "TABLETS" deve ser retornada, já que "tab" é um prefixo de "Tablet"

  @funcional
  Cenário: Busca é case-insensitive
    Quando eu envio "GET /catalog/api/v1/products/search?name=TABLET"
    E eu envio "GET /catalog/api/v1/products/search?name=tablet"
    Então ambas as respostas devem ser idênticas em conteúdo

  @funcional
  Cenário: Parâmetro "name" vazio retorna o catálogo completo agrupado por categoria
    Quando eu envio "GET /catalog/api/v1/products/search?name="
    Então o status deve ser 200
    E a resposta deve conter múltiplas categorias (ex.: "LAPTOPS", "TABLETS", "HEADPHONES", "MICE", etc.)
    E o número total de produtos deve ser igual ao total do catálogo

  @funcional
  Cenário: Parâmetros extras e desconhecidos são ignorados sem quebrar a busca
    Quando eu envio "GET /catalog/api/v1/products/search?name=tablet&foo=bar"
    Então o status deve ser 200
    E o resultado deve ser idêntico ao de uma busca apenas com "name=tablet"

  @excecao
  Cenário: Busca contendo apenas um espaço deve ser tratada como busca vazia
    Quando eu envio "GET /catalog/api/v1/products/search?name=%20" (um único espaço)
    Então o status deve ser 200
    E o comportamento deve ser equivalente ao de uma busca com o parâmetro "name" vazio
```

---

## Funcionalidade 3: Entradas Inválidas, Segurança e Robustez

```gherkin
Funcionalidade: Robustez e segurança do endpoint de busca
  Como QA especializado em segurança e robustez de API
  Eu quero validar que o endpoint trata entradas maliciosas ou inesperadas com segurança
  Para reduzir riscos de vulnerabilidades e falhas de serviço


  @seguranca
  Cenário: Headers de segurança obrigatórios estão presentes
    Quando eu envio qualquer requisição válida ao endpoint
    Então a resposta deve conter o header "X-Content-Type-Options: nosniff"
    E a resposta deve conter o header "X-Frame-Options: DENY"
    E a resposta deve conter o header "Strict-Transport-Security"

  @seguranca @defeito
  Cenário: Erro de requisição inválida não deve expor stack trace/detalhes internos
    Quando eu envio uma requisição sem o parâmetro obrigatório "name"
    Então o status retornado deve ser 400
    E o corpo da resposta de erro não deve conter detalhes de implementação (stack traces, nomes de classes internas, versão do servidor de aplicação)

  @excecao
  Cenário: Caracteres Unicode/acentuados são aceitos sem erro
    Quando eu envio "GET /catalog/api/v1/products/search?name=tâblet"
    Então o status deve ser 200
    E a aplicação não deve retornar erro de encoding

  @excecao
  Cenário: Entrada extremamente longa não derruba o serviço
    Quando eu envio "GET /catalog/api/v1/products/search?name=<string de 5000 caracteres>"
    Então o status deve ser 200 ou um 400/413 controlado
    E o serviço não deve travar, nem retornar 500, nem demorar excessivamente

  @naofuncional
  Cenário: Cache não deve servir resultados desatualizados incorretamente
    Quando eu envio "GET /catalog/api/v1/products/search?name=tablet"
    Então o header "Cache-Control" deve refletir a política real de cache do endpoint (ex.: "no-store")
    E resultados não devem ficar em cache indevidamente entre buscas por termos diferentes
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
| Busca por nome | Termo existente, parcial, case-insensitive, vazio, parâmetros extras | Schema do array/objeto, tipos de campo, status 200, Content-Type | Busca contendo apenas espaço tratada como vazia | — |
| Entradas inválidas/maliciosas | — | — | Unicode, string longa | — |
| Tratamento de erro | — | Parâmetro ausente, parâmetro inesperado | Exposição de detalhes internos em erros | — |
| Verbos HTTP | — | Método não suportado (405) | — | — |
| Contrato de resposta vazia | — | Array vazio (`[]`), Content-Type presente | — | — |
| Cache | — | — | — | Política de `Cache-Control` entre buscas |

---

Consulte o [Relatório de Bugs](advantageonlineshopping-search-api-bug-report.pt.md) para o resultado da execução destes cenários e os defeitos encontrados.
