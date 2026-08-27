# Cenários de Teste — Advantage Online Shopping

**Site testado:** https://advantageonlineshopping.com/#/
**Notação:** Gherkin (Português). A tag `@excecao` marca cenários negativos/casos de borda.

---

## Funcionalidade 1: Busca de Produtos

```gherkin
Funcionalidade: Busca de produtos
  Como um comprador
  Eu quero buscar produtos pelo nome
  Para encontrar rapidamente os itens que desejo comprar

  Contexto:
    Dado que estou na página inicial da Advantage Online Shopping

  Cenário: Busca por produto existente retorna apenas resultados relevantes
    Quando eu busco por "tablet"
    Então somente produtos relacionados ao termo buscado devem ser exibidos
    E as opções de filtragem por categoria, preço e cor devem estar disponíveis

  Cenário: Resultado da busca redireciona para a página de compra do produto
    Quando eu busco por "tablet" e acesso um dos resultados
    Então eu devo estar na página do produto com todas as informações necessárias para realizar a compra

  Cenário: Busca é insensível a maiúsculas e minúsculas
    Quando eu busco por "TABLET"
    Então eu devo obter os mesmos resultados de uma busca por "tablet"

  Cenário: Busca por termo parcial retorna produtos correspondentes
    Quando eu busco por "tab"
    Então produtos cujo nome contém "tablet" devem ser incluídos nos resultados

  Cenário: Filtrar por categoria restringe os resultados ao segmento selecionado
    Dado que realizei uma busca por "tablet" e vejo resultados de múltiplas categorias
    Quando eu aplico o filtro de categoria "TABLETS"
    Então somente produtos da categoria "TABLETS" devem permanecer visíveis

  Cenário: Filtrar por faixa de preço exibe apenas produtos dentro do intervalo escolhido
    Dado que realizei uma busca por "tablet"
    Quando eu aplico um filtro de preço com valores mínimo e máximo definidos
    Então somente produtos dentro dessa faixa de preço devem ser exibidos

  @excecao
  Cenário: Busca por produto inexistente informa o comprador sem quebrar a experiência
    Quando eu busco por "zzzznotfound123"
    Então eu devo ver a mensagem de que não há resultados para o termo buscado
    E a página deve permanecer funcional

  @excecao
  Cenário: Busca vazia ou com apenas espaços retorna o catálogo completo
    Quando eu envio uma busca sem informar um termo
    Então todos os produtos do catálogo devem ser listados sem mensagem de erro

  @excecao
  Esquema do Cenário: Entradas maliciosas ou inesperadas não quebram a experiência de busca
    Quando eu busco por "<termo>"
    Então a página deve continuar funcional e exibir resultados relevantes ou a mensagem de nenhum resultado

    Exemplos:
      | termo       |
      | @#$%^&*     |
      | <script>    |
      | ' OR '1'='1 |
      | !!!         |
      | ../../etc   |

  @excecao
  Cenário: Busca por termo muito curto responde de forma adequada sem erros
    Quando eu busco por "a"
    Então a página deve exibir resultados ou a mensagem de nenhum resultado sem apresentar erros
```

---

## Funcionalidade 2: Inclusão de Produto no Carrinho de Compras

```gherkin
Funcionalidade: Inclusão de produto no carrinho de compras
  Como um comprador
  Eu quero adicionar produtos ao carrinho com cor e quantidade escolhidas
  Para que meu pedido reflita exatamente o que desejo comprar

  Contexto:
    Dado que estou na página de detalhes de um produto (ex.: "HP ElitePad 1000 G2 Tablet")

  Cenário: Produto adicionado ao carrinho reflete as escolhas do comprador
    Quando eu seleciono uma cor e defino a quantidade desejada e adiciono o produto ao carrinho
    Então o carrinho deve exibir o produto com a cor e quantidade escolhidas
    E o total do pedido deve corresponder ao preço conforme a quantidade selecionada
    E o mini-resumo do carrinho deve ser exibido com a opção de avançar para o checkout

  Cenário: Adicionar o mesmo produto duas vezes acumula a quantidade no carrinho
    Dado que o produto já está no carrinho com quantidade "1"
    Quando eu adiciono mais uma unidade do mesmo produto e cor
    Então o carrinho deve exibir uma única linha com quantidade "2"

  Cenário: O indicador do carrinho no cabeçalho reflete o total de itens em tempo real
    Dado que o carrinho está vazio
    Quando eu adiciono produtos com quantidades distintas ao carrinho
    Então o indicador do carrinho no cabeçalho deve exibir a soma total das quantidades

  Cenário: O ícone do carrinho leva à página completa de gerenciamento do pedido
    Dado que o carrinho contém pelo menos um item
    Quando eu clico no ícone do carrinho
    Então eu devo ver todos os produtos adicionados com opções para editar ou remover cada item

  @excecao
  Cenário: Quantidade inválida não é adicionada ao carrinho
    Quando eu tento adicionar um produto informando uma quantidade zero, negativa ou não numérica
    Então o carrinho não deve ser atualizado com uma quantidade inválida
    E a interface deve preservar um estado consistente

  @excecao
  Cenário: Remover o único item do carrinho exibe o estado de vazio
    Dado que o carrinho contém apenas um produto
    Quando eu removo esse produto
    Então o carrinho deve exibir a mensagem de vazio e o indicador no cabeçalho não deve mais exibir contagem

  @excecao
  Cenário: Acessar o carrinho vazio orienta o comprador a continuar navegando
    Dado que o carrinho está vazio
    Quando eu acesso a página do carrinho
    Então eu devo ver a mensagem "Your shopping cart is empty"
```

---

## Funcionalidade 3: Validação do Carrinho de Compras na Tela de Pagamento/Checkout

```gherkin
Funcionalidade: Validação do carrinho de compras e checkout (tela de pagamento)
  Como um comprador
  Eu quero que o carrinho e o checkout reflitam meu pedido com precisão
  Para confiar nas informações antes de finalizar a compra

  Contexto:
    Dado que adicionei pelo menos um produto ao meu carrinho de compras

  Cenário: A página do carrinho exibe o pedido com precisão
    Quando eu acesso a página do carrinho
    Então cada produto deve estar listado com a cor, quantidade e preço escolhidos
    E o total deve corresponder à soma de todos os itens
    E as opções de pagamento disponíveis devem ser exibidas

  Cenário: O comprador pode corrigir itens do pedido diretamente pelo carrinho
    Dado que um produto está listado no carrinho
    Quando eu edito esse produto
    Então eu devo poder alterar cor ou quantidade e ver o carrinho atualizado com os novos valores

  Cenário: Remover um produto atualiza o pedido corretamente
    Quando eu removo um produto do carrinho
    Então esse item não deve mais constar no pedido e o total deve ser recalculado

  @excecao
  Cenário: Remover todos os produtos encerra o pedido em andamento
    Dado que o carrinho contém exatamente um produto
    Quando eu o removo
    Então o carrinho deve exibir o estado de vazio com opção de continuar comprando

  Cenário: Comprador não logado é orientado a fazer login ou cadastro no checkout
    Dado que não estou logado
    E meu carrinho contém pelo menos um item
    Quando eu prossigo para o checkout
    Então eu devo ser apresentado às opções de login e cadastro
    E o resumo do pedido deve permanecer visível com os produtos, quantidades e total corretos

  @excecao
  Cenário: Credenciais inválidas no checkout não perdem o pedido do comprador
    Dado que estou na etapa de login do checkout
    Quando eu tento autenticar com credenciais inválidas
    Então eu devo ver uma mensagem de erro de autenticação
    E o conteúdo do meu carrinho deve ser preservado

  Cenário: Comprador logado acessa diretamente os detalhes de pagamento
    Dado que estou logado
    E meu carrinho contém pelo menos um item
    Quando eu prossigo para o checkout
    Então eu devo ser direcionado aos detalhes de pagamento sem etapa de login
    E o resumo do pedido deve listar os itens corretos com o total calculado

  Cenário: Pedido com múltiplos itens é resumido corretamente no checkout
    Dado que adicionei dois produtos com quantidades distintas
    Quando eu prossigo para o checkout
    Então o resumo deve exibir ambos os produtos com seus respectivos preços e o total correto

  @excecao
  Cenário: Acessar o checkout com carrinho vazio impede a continuação da compra
    Dado que o carrinho está vazio
    Quando eu tento acessar a página de checkout diretamente
    Então eu não devo ver um resumo de pedido válido
    E devo ser impedido de prosseguir para o pagamento
```

---

## Resumo de Cobertura

| Área | Fluxo Principal (Happy Path) | Exceções / Casos de Borda |
|---|---|---|
| Busca de Produtos | Produto existente, correspondência parcial, sem diferenciação de maiúsculas/minúsculas, filtros por categoria e preço | Busca vazia / somente espaços, termo inexistente, entradas maliciosas, termo muito curto |
| Inclusão no Carrinho | Escolha de cor e quantidade refletidas no pedido, inclusão repetida acumula, indicador do cabeçalho atualizado, acesso à página completa do carrinho | Quantidade inválida (zero, negativa, não numérica), carrinho vazio após remoção |
| Validação do Carrinho / Checkout | Pedido exibido com precisão, edição de itens, remoção atualiza total, checkout logado, múltiplos itens | Carrinho esvaziado, login inválido preserva carrinho, checkout sem login, checkout com carrinho vazio |

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
