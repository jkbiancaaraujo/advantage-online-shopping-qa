# Relatório Final de Bugs — API de Busca de Produtos (Advantage Online Shopping)

**Endpoint testado:** `GET https://www.advantageonlineshopping.com/catalog/api/v1/products/search`
**Data da execução:** 2026-08-18
**Método:** Testes exploratórios de API, validando os cenários descritos em [advantageonlineshopping-search-api-test-scenarios.pt.md](advantageonlineshopping-search-api-test-scenarios.pt.md)
**Ambiente:** Produção pública (`www.advantageonlineshopping.com`)

---

## Resumo Executivo

| Severidade | Quantidade |
|---|---|
| 🔴 Alta (Contrato/Erro 5xx indevido) | 4 |
| � Baixa (Cabeçalhos/Boas práticas) | 2 |
| ✅ Pontos positivos de segurança | 5 |

**Total de defeitos encontrados: 6**

---

## BUG-01 — Parâmetro obrigatório ausente retorna 500 em vez de 400

- **Severidade:** 🔴 Alta
- **Cenário relacionado:** `@contrato @defeito` — "Parâmetro obrigatório ausente deve retornar 400" (Funcionalidade 1)
- **Passos para reproduzir:**
  ```
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  ```
- **Resultado esperado:** `400 Bad Request` com corpo JSON estruturado indicando que o parâmetro `name` é obrigatório.
- **Resultado obtido:** `500 Internal Server Error`, com corpo em HTML (página padrão de erro do Apache Tomcat).
- **Impacto:** Erros de entrada do cliente (4xx) são reportados como falhas do servidor (5xx), o que confunde monitoramento/alerting, dificulta o diagnóstico por consumidores da API e viola a semântica HTTP.
- **Categoria:** Contract Testing / Tratamento de erros.

---

## BUG-02 — Nome de parâmetro incorreto também gera 500

- **Severidade:** 🔴 Alta
- **Cenário relacionado:** `@contrato @defeito` — "Nome de parâmetro inesperado deve retornar 400" (Funcionalidade 1)
- **Passos para reproduzir:**
  ```
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search?query=tablet"
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search?term=tablet"
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search?viewAll=tablet"
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search?searchTerm=tablet"
  ```
- **Resultado esperado:** `400 Bad Request`, já que nenhum desses nomes é reconhecido e `name` (obrigatório) está ausente.
- **Resultado obtido:** `500 Internal Server Error` para todos os casos.
- **Impacto:** Mesma causa raiz do BUG-01 — qualquer requisição sem o parâmetro `name` reconhecido derruba a aplicação com erro genérico.
- **Categoria:** Contract Testing.

---

## BUG-03 — Resposta sem resultados retorna corpo vazio em vez de `[]`

- **Severidade:** 🔴 Alta
- **Cenário relacionado:** `@contrato @defeito` — "Busca sem resultados deve retornar um array JSON vazio" (Funcionalidade 1)
- **Passos para reproduzir:**
  ```
  curl -i -G --data-urlencode "name=zzzznotfound123" "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  ```
- **Resultado esperado:** `200 OK`, header `Content-Type: application/json`, corpo `[]`.
- **Resultado obtido:** `200 OK`, **sem** o header `Content-Type` e corpo completamente vazio (0 bytes) — não é um JSON válido.
- **Impacto:** Qualquer cliente que tente fazer `JSON.parse()`/deserializar a resposta terá uma exceção de parsing em vez de tratar uma lista vazia. Quebra silenciosa de contrato para o caso mais comum de "sem resultados".
- **Categoria:** Contract Testing.

---

## BUG-04 — Métodos HTTP não suportados (POST/PUT/DELETE) retornam 500 em vez de 405

- **Severidade:** 🔴 Alta
- **Cenário relacionado:** `@contrato @defeito` — "Métodos de escrita não suportados devem retornar 405 Method Not Allowed" (Funcionalidade 4)
- **Passos para reproduzir:**
  ```
  curl -i -X POST -G --data-urlencode "name=tablet" "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  curl -i -X PUT  -G --data-urlencode "name=tablet" "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  curl -i -X DELETE -G --data-urlencode "name=tablet" "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  ```
- **Resultado esperado:** `405 Method Not Allowed`.
- **Resultado obtido:** `500 Internal Server Error` para os três verbos.
- **Impacto:** Mesmo problema de semântica HTTP incorreta; dificulta diagnósticos automatizados e ferramentas de contract testing que validam verbos permitidos.
- **Categoria:** Contract Testing.

---

## BUG-05 — Erro 500 expõe página de erro interna (potencial exposição de informação)

- **Severidade:** 🟡 Baixa
- **Cenário relacionado:** `@seguranca @defeito` — "Erro de requisição inválida não deve expor stack trace/detalhes internos" (Funcionalidade 3)
- **Passos para reproduzir:**
  ```
  curl -i "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  ```
- **Resultado esperado:** Corpo de erro genérico e controlado, sem detalhes de implementação.
- **Resultado obtido:** Página HTML padrão de erro do Apache Tomcat, com título "HTTP Status 500 – Internal Server Error" e estrutura padrão do servlet container.
- **Impacto:** Risco baixo a médio de exposição de informações sobre a stack tecnológica (OWASP API Security Top 10 — *Improper Error Handling*), útil para reconhecimento por um atacante.
- **Categoria:** Segurança (informativo).

---

## BUG-06 — Header `Content-Type` não inclui `charset` explícito

- **Severidade:** 🟡 Baixa
- **Cenário relacionado:** Observação de contrato (Funcionalidade 1)
- **Passos para reproduzir:**
  ```
  curl -i -G --data-urlencode "name=tablet" "https://www.advantageonlineshopping.com/catalog/api/v1/products/search"
  ```
- **Resultado esperado:** `Content-Type: application/json; charset=UTF-8`.
- **Resultado obtido:** `Content-Type: application/json` (sem `charset`).
- **Impacto:** Baixo — pode gerar ambiguidade de decodificação em clientes mais estritos, especialmente para nomes de produtos com acentuação.
- **Categoria:** Contract Testing (boas práticas).

---

## Pontos Positivos (Não são bugs)

| # | Item | Observação | Cenário relacionado |
|---|---|---|---|
| ✅ 1 | Proteção contra SQL Injection | `name=' OR '1'='1` não retorna dados indevidos nem expõe erros de banco de dados. | Teste exploratório, sem cenário formal correspondente |
| ✅ 2 | Proteção contra XSS refletido | `name=<script>alert(1)</script>` não é ecoado na resposta. | Teste exploratório, sem cenário formal correspondente |
| ✅ 3 | Headers de segurança presentes | `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security` presentes em todas as respostas testadas. | "Headers de segurança obrigatórios estão presentes" (Funcionalidade 3) |
| ✅ 4 | Busca case-insensitive | `TABLET` e `tablet` retornam resultados idênticos. | "Busca é case-insensitive" (Funcionalidade 2) |
| ✅ 5 | Aceita Unicode/acentuação e entradas muito longas sem falhar | Nenhum erro 500 nesses casos. | "Caracteres Unicode/acentuados são aceitos sem erro" / "Entrada extremamente longa não derruba o serviço" (Funcionalidade 3) |

---
