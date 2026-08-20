# Cenários de Teste — Advantage Online Shopping

**Site testado:** https://advantageonlineshopping.com/#/
**Notação:** Gherkin (Português). A tag `@excecao` marca cenários negativos/casos de borda.

---

## Funcionalidade 1: Busca de Produtos

```gherkin
Funcionalidade: Busca de produtos
  Como um comprador
  Eu quero buscar produtos
  Para que eu possa encontrar rapidamente os itens que desejo comprar

  Contexto:
    Dado que estou na página inicial da Advantage Online Shopping

  Cenário: Buscar por um produto existente retorna resultados correspondentes
    Quando eu busco por "tablet"
    Então o cabeçalho do resultado da busca deve exibir "Search result: "tablet"
    E a quantidade de resultados deve ser maior que 0
    E todo item do resultado deve estar relacionado ao termo "tablet"
    E as opções de filtro "CATEGORIES", "PRICE" e "COLOR" devem ser exibidas

  Cenário: Item do resultado da busca navega para a página correta do produto
    Quando eu busco por "tablet"
    E eu clico no primeiro resultado
    Então eu devo ser direcionado para a página de detalhes daquele produto
    E o nome, preço, descrição, opções de cor e campo de quantidade do produto devem ser exibidos

  @excecao
  Cenário: Buscar por um produto inexistente exibe mensagem de nenhum resultado
    Quando eu busco por "zzzznotfound123"
    Então eu devo ver a mensagem "No results for \"zzzznotfound123\""
    E nenhuma lista de produtos deve ser exibida
    E um link de sugestão alternativa "SAP Fiori Demo App" deve ser exibido sem causar erro na página

  @excecao
  Cenário: Busca vazia é enviada e retorna todo o catálogo
    Dado que o campo de busca está vazio
    Quando eu envio a busca
    Então eu não devo ver um erro de validação
    E o cabeçalho do resultado da busca deve exibir um termo de busca vazio
    E todos os itens do catálogo devem ser listados

  @excecao
  Cenário: Busca contendo apenas espaços em branco se comporta como uma busca vazia
    Quando eu busco por "   "
    Então eu não devo ver um erro de validação
    E o resultado deve ser tratado da mesma forma que uma busca vazia

  @excecao
  Esquema do Cenário: Busca com caracteres especiais não quebra a página
    Quando eu busco por "<termo>"
    Então a página deve renderizar sem erro de JavaScript
    E eu devo ver resultados correspondentes ou a mensagem "No results for \"<termo>\""

    Exemplos:
      | termo       |
      | @#$%^&*     |
      | <script>    |
      | ' OR '1'='1 |
      | !!!         |
      | ../../etc   |

  Cenário: O termo de busca não diferencia maiúsculas de minúsculas
    Quando eu busco por "TABLET"
    Então os resultados devem ser os mesmos que ao buscar por "tablet"

  Cenário: A busca corresponde a palavras-chave parciais
    Quando eu busco por "tab"
    Então resultados contendo "tablet" no nome devem ser incluídos

  @excecao
  Cenário: Busca por um termo de um único caractere/curto
    Quando eu busco por "a"
    Então a página deve retornar um conjunto amplo de resultados ou a mensagem "No results"
    E a página não deve apresentar erro ou travar

  Cenário: Filtrar resultados de busca por categoria reduz o conjunto de resultados
    Dado que busquei por "tablet" e vejo 3 itens
    Quando eu marco o filtro de categoria "TABLETS"
    Então somente os itens pertencentes à categoria "TABLETS" devem permanecer visíveis

  Cenário: Filtrar resultados de busca por faixa de preço reduz o conjunto de resultados
    Dado que busquei por "tablet"
    Quando eu filtro por uma faixa de preço dentro dos valores mínimo/máximo exibidos
    Então somente itens dentro dessa faixa de preço devem permanecer visíveis

  Cenário: Todo resultado retornado deve ser genuinamente relevante ao termo buscado
    Quando eu busco por uma palavra-chave válida de produto, por exemplo "tablet"
    Então todo item na lista de resultados deve ser um produto que um usuário buscando por essa palavra-chave esperaria encontrar
    E nenhum item não relacionado a "tablet" deve estar presente nos resultados

  Cenário: Os resultados da busca não devem incluir itens que apenas contêm o termo como substring por coincidência
    Quando eu busco por uma palavra-chave que também é substring do nome de uma categoria de produto não relacionada
    Então os resultados devem incluir somente itens verdadeiramente relevantes à minha intenção de busca
    E itens não devem ser incluídos quando não possuem palavras relacionadas

  Cenário: A contagem de resultados corresponde ao número de itens genuinamente relevantes
    Quando eu busco por "tablet"
    Então a contagem de itens exibida (ex.: "3 ITEMS") deve ser igual ao número de resultados realmente relevantes para "tablet"
    E não deve incluir itens não relacionados inflando a contagem
```

---

## Funcionalidade 2: Inclusão de Produto no Carrinho de Compras

```gherkin
Funcionalidade: Inclusão de produto no carrinho de compras
  Como um comprador
  Eu quero adicionar produtos ao meu carrinho com uma cor e quantidade escolhidas
  Para que eu possa comprar os itens que desejo e acompanhar o estado do carrinho pelo ícone/toggle no cabeçalho

  Contexto:
    Dado que estou na página de detalhes de um produto (ex.: "HP ElitePad 1000 G2 Tablet")

  Cenário: Adicionar um produto ao carrinho com a quantidade padrão
    Dado que o campo de quantidade exibe o valor padrão "1"
    E meu carrinho de compras está vazio
    Quando eu clico em "ADD TO CART"
    Então o carrinho deve conter o item com "QTY: 1"
    E o ícone do carrinho deve exibir um indicador com o valor "1"
    E um toggle de resumo do carrinho deve aparecer mostrando o nome do produto, "QTY: 1", cor selecionada e preço
    E o toggle deve exibir o "TOTAL" do pedido e um botão "CHECKOUT"

  Cenário: Adicionar um produto com uma seleção específica de cor
    Quando eu seleciono a cor "GRAY"
    E eu clico em "ADD TO CART"
    Então o carrinho deve conter o item com a cor "GRAY"

  Cenário: Adicionar um produto com quantidade aumentada
    Quando eu aumento a quantidade para "3" usando o controle "+" do stepper
    E eu clico em "ADD TO CART"
    Então o carrinho deve conter "QTY: 3" para aquele produto
    E o preço do item exibido deve refletir corretamente a quantidade conforme o comportamento do site

  Cenário: Adicionar o mesmo produto duas vezes acumula a quantidade no carrinho
    Dado que já adicionei 1 unidade do produto ao carrinho
    Quando eu adiciono mais 1 unidade do mesmo produto e cor ao carrinho
    Então o carrinho deve exibir um único item de linha com quantidade "2"

  @excecao
  Cenário: Tentativa de adicionar um produto com quantidade zero
    Quando eu limpo o campo de quantidade e digito "0"
    E eu clico em "ADD TO CART"
    Então o item adicionado ao carrinho deve ter uma quantidade mínima de "1"
    Ou a ação "ADD TO CART" deve ser bloqueada com uma mensagem de validação
    E o carrinho nunca deve conter um item de linha com quantidade "0"

  @excecao
  Cenário: Tentativa de adicionar um produto com quantidade negativa
    Quando eu limpo o campo de quantidade e digito "-5"
    Então o campo de quantidade deve rejeitar o valor negativo e manter/restaurar um valor positivo válido
    E clicar em "ADD TO CART" não deve adicionar ao carrinho um item com quantidade negativa

  @excecao - Bug
  Cenário: Tentativa de adicionar um produto com quantidade não numérica
    Quando eu limpo o campo de quantidade e digito "abc"
    E eu clico em "ADD TO CART"
    Então a contagem de itens do carrinho deve permanecer inalterada
    E nenhum item novo deve ser adicionado ao carrinho
    E idealmente uma mensagem de validação deve informar ao usuário que a quantidade deve ser numérica

  @excecao
  Cenário: Diminuir a quantidade abaixo de um usando o controle "-" do stepper
    Dado que o campo de quantidade exibe "1"
    Quando eu clico no controle "-" do stepper
    Então a quantidade não deve ficar abaixo de "1"

  Cenário: O indicador do ícone do carrinho corresponde à soma de todas as quantidades de itens
    Dado que adicionei o produto A com quantidade "2"
    Quando eu adiciono o produto B com quantidade "1"
    Então o indicador do ícone do carrinho deve atualizar em tempo real e exibir "3"

  Cenário: O mini-toggle do carrinho permite navegar diretamente para o checkout
    Dado que o toggle de resumo do carrinho está visível após adicionar um produto
    Quando eu clico em "CHECKOUT" dentro do toggle
    Então eu devo ser direcionado para a página de pagamento do pedido / login

  Cenário: Clicar no ícone do carrinho abre a página completa do carrinho de compras
    Dado que meu carrinho tem pelo menos um item
    Quando eu clico no ícone do carrinho de compras
    Então eu devo ser direcionado para "#/shoppingCart"
    E todos os itens do carrinho devem estar listados com as ações "EDIT" e "REMOVE"

  @excecao
  Cenário: O ícone do carrinho reflete a remoção de itens imediatamente
    Dado que meu carrinho tem "1" item
    Quando eu removo esse item na página do carrinho de compras
    Então o indicador do ícone do carrinho não deve mais exibir uma contagem

  @excecao
  Cenário: Clicar no ícone do carrinho com o carrinho vazio exibe a mensagem de carrinho vazio
    Dado que meu carrinho de compras está vazio
    Quando eu clico no ícone do carrinho de compras
    Então eu devo ser direcionado para a página do carrinho de compras
    E eu devo ver a mensagem "Your shopping cart is empty"
    E eu devo ver um link "CONTINUE SHOPPING" que me retorna à página inicial
```

---

## Funcionalidade 3: Validação do Carrinho de Compras na Tela de Pagamento/Checkout

```gherkin
Funcionalidade: Validação do carrinho de compras e checkout (tela de pagamento)
  Como um comprador
  Eu quero que o carrinho e a tela de checkout reflitam com precisão meu pedido
  Para que eu possa confiar no site antes de pagar

  Contexto:
    Dado que adicionei pelo menos um produto ao meu carrinho de compras

  Cenário: A página do carrinho exibe os detalhes corretos do produto
    Quando eu abro a página do carrinho de compras
    Então cada item de linha deve exibir o nome correto do produto
    E cada item de linha deve exibir a cor selecionada correta
    E cada item de linha deve exibir a quantidade correta
    E cada item de linha deve exibir o preço unitário/da linha correto
    E o "TOTAL" deve ser igual à soma dos preços de todos os itens de linha

  Cenário: As opções de pagamento disponíveis são exibidas na página do carrinho
    Quando eu abro a página do carrinho de compras
    Então a seção "PAYMENT OPTIONS:" deve exibir os ícones dos métodos de pagamento suportados (ex.: MasterCredit, SafePay)

  Cenário: Editar um produto a partir do carrinho
    Dado que um produto está listado no carrinho de compras
    Quando eu clico em "EDIT" naquele item de linha
    Então eu devo ser direcionado para a página do produto pré-preenchida com a cor e quantidade atuais
    E eu devo conseguir alterar a quantidade e/ou cor e atualizar o carrinho de acordo

  Cenário: Remover um produto do carrinho
    Dado que pelo menos um produto está listado no carrinho de compras
    Quando eu clico em "REMOVE" naquele item de linha
    Então o item não deve mais aparecer no carrinho
    E o "TOTAL" deve ser recalculado excluindo aquele item
    E a contagem do indicador do ícone do carrinho deve diminuir de acordo

  @excecao
  Cenário: Remover todos os produtos esvazia o carrinho
    Dado que existe exatamente um produto no carrinho de compras
    Quando eu removo esse produto
    Então a página do carrinho deve exibir "Your shopping cart is empty"
    E um link "CONTINUE SHOPPING" deve ser exibido
    E o indicador do ícone do carrinho não deve mais exibir uma contagem

  Cenário: Checkout com usuário deslogado exibe opções de login e cadastro
    Dado que eu não estou logado
    E meu carrinho contém pelo menos um item
    Quando eu clico em "CHECKOUT"
    Então eu devo ser direcionado para a página de pagamento do pedido
    E eu devo ver "Already have an account?" com campos de Usuário e Senha e um botão "LOGIN"
    E eu devo ver um link "Forgot your password?"
    E eu devo ver "New user?" com um botão "REGISTRATION"
    E o painel de Resumo do Pedido deve listar o(s) produto(s) correto(s), quantidade, cor e total

  @excecao
  Cenário: Login no checkout com credenciais inválidas exibe um erro e mantém conteudo do carrinho
    Dado que estou na página de pagamento do pedido / login
    Quando eu insiro um usuário e senha inválidos
    E eu clico em "LOGIN"
    Então eu devo ver a mensagem "Incorrect user name or password."
    E eu devo permanecer na página de login
    E o conteúdo do meu carrinho deve ser preservado

  Cenário: Checkout com usuário logado segue para os detalhes de pagamento
    Dado que estou logado com uma conta registrada válida
    E meu carrinho contém pelo menos um item
    Quando eu clico em "CHECKOUT"
    Então eu devo ser levado diretamente aos detalhes de pagamento do pedido (sem solicitação de login/cadastro)
    E o Resumo do Pedido deve exibir o(s) produto(s) correto(s), cor, quantidade e total

  Cenário: O resumo do pedido reflete múltiplos itens corretamente
    Dado que adicionei 2 produtos diferentes com quantidades diferentes ao meu carrinho
    Quando eu prossigo para o checkout
    Então o Resumo do Pedido deve listar ambos os produtos com sua cor, quantidade e preço corretos
    E o total deve ser igual à soma de ambos os itens de linha (mais frete, se aplicável)

  @excecao
  Cenário: Tentativa de checkout com carrinho vazio
    Dado que meu carrinho de compras está vazio
    Quando eu tento navegar diretamente para a página de checkout/pagamento
    Então eu não devo ver um resumo de pedido válido com itens
    E eu devo ser redirecionado para a página de carrinho vazio, impedido de prosseguir para o pagamento
```

---

## Resumo de Cobertura

| Área | Fluxo Principal (Happy Path) | Exceções / Casos de Borda |
|---|---|---|
| Busca de Produtos | Produto existente, correspondência parcial, sem diferenciação de maiúsculas/minúsculas, filtros, relevância/precisão dos resultados | Busca vazia, apenas espaços em branco, termo inexistente, caracteres especiais, termo de um único caractere, **defeitos de relevância** (`"phone"` → fones de ouvido, `"top"` → laptops) |
| Inclusão no Carrinho (inclui ícone/toggle) | Quantidade padrão, quantidade customizada, seleção de cor, inclusão repetida, indicador do ícone, resumo no toggle, navegação toggle→checkout, navegação ícone→página completa do carrinho | Quantidade = 0, quantidade negativa, quantidade não numérica, decremento abaixo de 1, cor não selecionada, indicador removido após esvaziar o carrinho, mensagem de vazio ao clicar no ícone |
| Validação do Carrinho / Checkout | Nome/cor/quantidade/preço corretos, edição, remoção, resumo do pedido, opções de pagamento | Carrinho esvaziado, login exigido quando deslogado, botão LOGIN desabilitado, credenciais inválidas, checkout com carrinho vazio |

> Observações:
> - Endpoint de busca: `#/search/?viewAll=<termo>`. Uma busca vazia retorna **todos os itens do catálogo** (394 itens observados) em vez de um erro ou estado vazio.
> - Busca sem correspondências mostra `No results for "<termo>"` além de um link para uma demo não relacionada da SAP (redirecionamento externo — vale um cenário para confirmar que isso NÃO gera erro silencioso).
> - O campo de quantidade na página do produto é um **campo de texto simples** (não é do tipo number): digitar `0`, um número negativo, ou letras é rejeitado/resetado pela interface ou simplesmente ignorado ao clicar em "ADD TO CART" (nenhum item é adicionado, nenhuma mensagem de erro visível) — um caso de exceção importante.
> - Adicionar um produto abre um **toggle/dropdown de mini-carrinho** sob o ícone do carrinho no cabeçalho, mostrando nome do produto, quantidade, cor, preço, total do pedido e um botão CHECKOUT.
> - Clicar no próprio ícone do carrinho navega para `#/shoppingCart` (página completa), que mostra as colunas `PRODUCT NAME | COLOR | QUANTITY | PRICE`, ações `EDIT | REMOVE` por linha, `PAYMENT OPTIONS:` (ícones MasterCredit, SafePay), `TOTAL`, e botão `CHECKOUT`.
> - Remover o último item mostra **"Your shopping cart is empty"** com um link `CONTINUE SHOPPING`, tanto na página do carrinho quanto ao clicar no ícone do carrinho depois.
> - O Checkout (`#/login`), quando o usuário **não está logado**, mostra duas opções lado a lado: "Already have an account?" (formulário de Login com Usuário/Senha, botão LOGIN desabilitado até os campos serem preenchidos, link "Forgot your password?") e "New user?" (botão REGISTRATION) — não foi encontrada opção de checkout como convidado (guest checkout). Um painel de Resumo do Pedido é exibido ao lado com produto(s), quantidade, cor, preço, frete e total.
> - Drodpdown de busca não desaparece após resultado.
> - Carrinho e pagina de checkout só exibe o total dos produtos e nenhuma especificação por preço unitario.
> - Credenciais de login inválidas mostram a mensagem: `Incorrect user name or password.`
