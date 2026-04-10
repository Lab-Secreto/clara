# Clara — Proposta de Linguagem

**Versão:** 0.0.1 (rascunho de trabalho)
**Autor:** Filipe Baêta
**Data:** Abril de 2026
**Status:** Documento vivo — em discussão

---

## 1. Filosofia

Clara parte de uma tese incômoda: as linguagens de programação atuais foram otimizadas para programadores experientes economizarem teclas. Nesse processo, pagaram um custo alto em clareza semântica — custo que era aceitável quando o único leitor era outro programador, mas que deixou de fazer sentido num mundo onde o código é coescrito, revisado e mantido em parceria com modelos de linguagem, e onde analistas de negócio, auditores e operadores precisam entender o que o sistema efetivamente faz.

A tese central de Clara é simples de enunciar e difícil de executar:

> **A melhor linguagem para humanos comuns e LLMs não é a que minimiza caracteres; é a que maximiza clareza semântica.**

Clara não quer ser a linguagem mais rápida, nem a mais expressiva, nem a mais elegante para o programador virtuoso. Ela quer ser a linguagem onde a *intenção* do código é indistinguível do *código*, onde um erro é uma conversa em vez de um stack trace, e onde o compilador absorve a complexidade em vez de transferi-la para quem escreve.

Isso implica três compromissos filosóficos:

**Clareza acima de concisão.** Quando há conflito entre ler rápido e escrever rápido, Clara escolhe ler rápido. Código é lido muito mais vezes do que é escrito — por humanos, por auditores, por modelos, pelo autor original seis meses depois.

**Explicitude acima de mágica.** Efeitos colaterais, dependências, condições de erro e suposições ficam visíveis na assinatura. Nada importante acontece em silêncio.

**Uma forma canônica.** Para cada coisa que Clara permite fazer, existe uma forma principal de fazer. Atalhos, açúcar sintático opcional e idiomas paralelos são rejeitados por design — não por purismo, mas porque são a principal fonte de divergência entre código escrito por humanos e por modelos.

---

## 2. Escopo

Clara é desenhada para uma faixa específica de problemas: **software de negócio, orquestração, automação, regras de domínio, pipelines de dados e sistemas auditáveis**. É a camada onde regras, decisões e efeitos encontram pessoas — operadores, analistas, clientes, reguladores.

Clara *não* tenta ser:

- Uma linguagem de sistemas (kernels, drivers, engines gráficas)
- Uma linguagem para código científico de alta performance
- Uma linguagem para metaprogramação ou DSLs internas
- Uma substituta universal para TypeScript, Python ou Rust

Clara tenta ser:

- A linguagem onde você escreve a regra de negócio que precisa ser auditada
- A linguagem onde você descreve o workflow que um operador precisa entender
- A linguagem onde humano e LLM colaboram sem atrito de sintaxe
- A fonte de verdade semântica que gera código, testes, UI e documentação

Fora desse escopo, Clara interopera com linguagens tradicionais via FFI — ela não tenta substituí-las.

---

## 3. Os Sete Pilares

### Pilar 1 — Sintaxe pequena e regular

Clara tem um vocabulário fechado de palavras-chave. Cada palavra-chave tem uma função estável e não polissêmica. Não há operadores sobrecarregáveis, não há macros, não há múltiplas formas de fazer a mesma coisa. Um formatter é parte da linguagem, não uma ferramenta externa.

### Pilar 2 — Blocos semânticos com intenção explícita

Clara não tem "função" como primitiva universal. Tem blocos com propósito: `compute`, `decision`, `effect`, `action`, `entity`, `policy`. Cada bloco carrega semântica distinta e o compilador trata cada um de forma diferente. Isso é o que permite separar o que é puro do que é efeito, o que é regra do que é execução.

### Pilar 3 — Tipos semânticos de domínio

Clara tem tipos primitivos tradicionais, mas encoraja fortemente o uso de tipos de domínio: `money_brl`, `cpf`, `email`, `cnpj`, `date`, `percentage`, `phone_br`. Unidades são cidadãs de primeira classe — você pode escrever `30 days` ou `R$ 50,00` como literais. O sistema de tipos impede operações sem sentido: você não pode somar reais com dólares, nem multiplicar CPF por data.

### Pilar 4 — Contratos explícitos

Toda `action` declara na assinatura seus efeitos: `reads:`, `writes:`, `calls:`, `raises:`. Isso não é documentação — é verificado pelo compilador. Se uma ação escreve em um banco sem declarar, o código não compila. Isso torna refatoração segura e torna o contexto local suficiente para raciocinar sobre qualquer bloco.

### Pilar 5 — Erros como valores, não como exceções

Clara não tem `try/catch`. Operações que podem falhar retornam um tipo `Result[T, Error]` e o compilador obriga você a lidar com ambos os casos. Todos os caminhos de falha ficam visíveis no código. Um LLM lendo o código vê onde algo pode dar errado sem precisar simular execução.

### Pilar 6 — Exemplos embutidos como especificação executável

Todo bloco pode conter um `examples:` que serve simultaneamente como documentação, teste e especificação. Exemplos são executados na compilação — se algum falhar, o build quebra. Um LLM gerando uma função pode usar os exemplos como ground truth antes mesmo do código existir.

### Pilar 7 — Explainability nativa

Clara tem uma operação de primeira classe: `explain <bloco> for <público>`. Ela gera uma descrição em linguagem natural do comportamento do bloco, usando os metadados do próprio código (tipos semânticos, efeitos declarados, exemplos, pré-condições). Isso não é um plugin — é parte da especificação da linguagem.

---

## 4. Fundamentos Técnicos

### 4.1 Modelo de compilação em duas camadas

Clara não compila diretamente para binário nem para uma linguagem-alvo única. Ela compila para uma **Intermediate Representation (IR)** tipada e autocontida. Dessa IR, múltiplos backends produzem artefatos:

```
código Clara
    ↓
parser → AST semântica
    ↓
verificador de tipos + efeitos + exemplos
    ↓
Clara IR (tipada, sem ambiguidade)
    ↓
┌──────────────┬────────────────┬─────────────┐
↓              ↓                ↓             ↓
TypeScript     Go              SQL          LLVM → binário
(frontend)  (serviços)      (queries)      (performance)
```

A decisão de qual backend usar é por módulo, não por projeto inteiro. Um módulo de cálculo puro pode compilar para binário nativo; um módulo de workflow pode rodar em runtime gerenciado com explainability ativa.

**Fase 1 (v0.1):** apenas backend TypeScript. Suficiente para validar a linguagem com casos reais.

**Fase 2 (v0.5):** backend Go + SQL para queries geradas.

**Fase 3 (v1.0):** backend LLVM via MLIR para módulos `pure`.

### 4.2 Separação pureza / efeito / decisão

Clara trata três categorias de código como cidadãs distintas:

- **`compute`** — função pura. Sem efeitos, sem I/O, sem dependência de tempo. O compilador pode memoizar, paralelizar, compilar para nativo.
- **`decision`** — regra de negócio expressa declarativamente. Retorna um valor a partir de condições. Auditável e explicável por construção.
- **`effect`** — operação que toca o mundo externo. Banco, rede, arquivo, log. Declara tudo que faz na assinatura.

Uma `action` é a composição orquestrada dos três. Ela pode chamar `compute`, consultar `decision` e executar `effect`, mas sempre de forma explícita.

### 4.3 Erros como valores

```clara
result = try fetch_customer(id: "123")

match result:
  ok(customer)  -> continue with customer
  error(reason) -> return error(reason)
```

Não há exceções. Não há `null`. Valores que podem estar ausentes usam `optional[T]` e o compilador força o tratamento do caso vazio.

### 4.4 Imutabilidade por padrão

Valores são imutáveis. Mutação é explícita e local, usando `mutate` dentro de um bloco `effect`. Isso elimina uma classe inteira de bugs e torna o raciocínio sobre estado linear.

### 4.5 Pattern matching exaustivo

`match` é exaustivo e verificado em compile-time. Se você adicionar um novo valor a um `one_of`, todos os `match` que dependem dele quebram o build até serem atualizados. Isso é o oposto do `switch` de JavaScript — é uma ferramenta de segurança, não de conveniência.

---

## 5. Sintaxe — Primeiro Esboço

### 5.1 Entidades

```clara
entity Customer
  fields:
    id: text
    name: text
    email: email
    tier: one_of(regular, premium, vip)
    created_at: datetime

  invariants:
    name must_not_be empty
    email must_be_valid
```

### 5.2 Compute (função pura)

```clara
compute final_price
  input:
    base_price: money_brl
    discount_percentage: percentage

  output:
    money_brl

  formula:
    base_price - (base_price * discount_percentage / 100)

  examples:
    when base_price = R$ 100,00 and discount_percentage = 10%
    result = R$ 90,00

    when base_price = R$ 200,00 and discount_percentage = 0%
    result = R$ 200,00
```

### 5.3 Decision (regra declarativa)

```clara
decision discount_rate_for_customer
  input:
    customer: Customer

  output:
    percentage

  rules:
    when customer.tier is vip      -> 20%
    when customer.tier is premium  -> 10%
    otherwise                      -> 0%

  examples:
    when customer.tier = vip      -> 20%
    when customer.tier = premium  -> 10%
    when customer.tier = regular  -> 0%
```

### 5.4 Effect (operação com mundo externo)

```clara
effect save_customer
  input:
    customer: Customer

  writes:
    Customer in database

  raises:
    DatabaseError
    ValidationError

  steps:
    mutate database:
      upsert Customer where id = customer.id with customer
```

### 5.5 Action (orquestração)

```clara
action approve_order
  purpose:
    "Approve a submitted order and calculate final total."

  input:
    order_id: text

  reads:
    Order, Customer from database

  writes:
    Order in database

  raises:
    OrderNotFound
    OrderNotSubmitted

  preconditions:
    order must_exist
    order.status must_be submitted

  steps:
    order     = fetch Order where id = order_id
    customer  = fetch Customer where id = order.customer_id
    rate      = decide discount_rate_for_customer with customer
    discount  = compute final_price with
                  base_price = order.subtotal
                  discount_percentage = rate

  effects:
    update Order where id = order_id:
      status = approved
      final_total = discount

  output:
    order_id: text
    final_total: money_brl
```

### 5.6 Data pipelines

```clara
compute approved_revenue
  input:
    orders: list[Order]

  output:
    money_brl

  pipeline:
    orders
      |> filter order.status is approved
      |> map order.final_total
      |> sum
```

### 5.7 Policy (conjunto de regras nomeado)

```clara
policy refund_policy
  when order.age <= 7 days and order.status is delivered:
    allow full_refund

  when order.age <= 30 days and order.status is delivered:
    allow partial_refund at 50%

  otherwise:
    deny
```

---

## 6. Explainability

```clara
explain approve_order for business_user
```

Saída gerada pelo compilador a partir dos metadados do bloco:

> A ação "Aprovar Pedido" transforma um pedido submetido em um pedido aprovado, calculando seu total final.
>
> **Pré-condições:** o pedido precisa existir e estar com status "submitted".
>
> **O que ela faz:** busca o cliente do pedido, determina a taxa de desconto conforme o nível do cliente (VIP 20%, Premium 10%, Regular 0%), calcula o total final e atualiza o pedido para "approved".
>
> **O que ela modifica:** o registro do pedido no banco de dados.
>
> **O que pode dar errado:** pedido não encontrado, pedido não submetido.

A mesma ação pode ser explicada `for junior_developer`, `for auditor`, `for test_writer` — gerando respectivamente um passo a passo técnico, um checklist de auditoria ou casos de teste executáveis.

---

## 7. Documentação e Tooling

Clara trata documentação como artefato gerado, não escrito à mão. A partir do mesmo código:

- **Referência de API** — gerada a partir de assinaturas e `purpose:`
- **Documentação de negócio** — gerada via `explain ... for business_user`
- **Testes** — gerados a partir de `examples:`
- **Diagramas de fluxo** — gerados a partir de `steps:` em actions
- **Schemas de API** — gerados a partir de `entity` e `action`
- **Formulários** — gerados a partir de `input:` com tipos semânticos

O compilador é também o gerador de documentação. Isso elimina a divergência entre "o que o código faz" e "o que a doc diz que o código faz" — a doc *é* o código visto de outro ângulo.

---

## 8. Interoperabilidade

Clara não vive sozinha. Todo módulo pode:

- **Importar** funções de TypeScript, Go ou Python via FFI tipada
- **Exportar** suas actions como endpoints HTTP, GraphQL ou gRPC
- **Emitir** schemas JSON, OpenAPI, Protobuf a partir de suas entidades
- **Gerar** migrations SQL a partir de mudanças em entidades

A expectativa não é que um sistema seja 100% Clara. A expectativa é que a camada semântica — regras, decisões, workflows auditáveis — seja Clara, e o resto continue sendo TypeScript, Go ou o que fizer sentido.

---

## 9. O que Clara explicitamente rejeita

- Herança de classes
- Exceções não declaradas
- `null` / `undefined`
- Sobrecarga de operadores
- Macros e metaprogramação em user-space
- Múltiplas formas sintáticas para a mesma operação
- Inferência de tipos invisível em assinaturas públicas
- Efeitos colaterais implícitos
- Mutação global

Cada uma dessas rejeições é uma escolha custosa — remove expressividade para ganhar previsibilidade. Clara aposta que previsibilidade é o recurso mais escasso na interface entre humanos, código e modelos.

---

## 10. Riscos Conhecidos

**Adoção.** Linguagens novas quase sempre falham, independentemente de mérito técnico. Ecossistema vence design nove em dez vezes. Clara só sobrevive se resolver um problema tão doloroso que justifique o custo de adoção — provavelmente começando como linguagem interna de um domínio específico (regras de negócio auditáveis em indústrias reguladas) antes de tentar ser geral.

**Verbosidade.** O equilíbrio entre "explícito o suficiente para ser claro" e "verboso demais para caber em contexto" ainda não está resolvido. A aposta de Clara é ser verbosa nas bordas (contratos, efeitos) e compacta no miolo (cálculos, pipelines).

**Compilador como gargalo.** Uma linguagem com IR própria, múltiplos backends e verificação semântica densa é cara de construir. A estratégia em três fases (TS → Go/SQL → LLVM) existe justamente para não depender do compilador perfeito antes de validar a ideia.

**Cemitério histórico.** AppleScript, HyperTalk, Inform 7, Wolfram, Eve, Dark, Unison. Várias linguagens semânticas já tentaram algo parecido e travaram em nicho. Clara precisa articular claramente o que é diferente *agora* — e a resposta mais honesta é: a presença de LLMs como co-autores muda a equação de custo/benefício, mas isso ainda é uma aposta, não uma certeza.

---

## 11. Próximos Passos

1. **Validar com um caso real.** Pegar uma regra de negócio não trivial do Portos Rio ou do Mira (por exemplo, uma transição de estado de ativo ou uma barreira de Bowtie) e escrevê-la em Clara, em TypeScript e em Python. Mostrar as três para três públicos distintos (engenheiro, operador, analista) e medir compreensão e confiança de modificação.

2. **Testar geração por LLM.** Pedir a um modelo para gerar 20 exemplos de cada versão a partir de descrições em português. Medir taxa de erro, consistência e tempo de iteração.

3. **Rascunhar a gramática formal.** Transformar esta proposta em uma gramática EBNF mínima e implementar um parser de brinquedo em TypeScript.

4. **Construir o interpretador v0.1.** Sem compilação, sem IR, sem backends. Apenas parser + AST + interpretador em TypeScript. Objetivo: rodar o exemplo `approve_order` end-to-end.

5. **Escrever a especificação da Clara IR.** Uma vez que o interpretador valide a semântica, definir a IR como contrato estável entre frontend e backends futuros.

---

## Apêndice A — Decisões em aberto

- Nome. "Clara" é um marcador. Alternativas: IntentScript, Civic, Plain, Lucid.
- Idioma das palavras-chave. Inglês por convenção de ecossistema, ou português para reduzir fricção com usuários não técnicos brasileiros? Possível suporte a ambos via layer de tradução.
- Como lidar com concorrência. Async/await? Actors? Structured concurrency? Nenhum dos três é trivial sob a filosofia de Clara.
- Generics / tipos paramétricos. Potencialmente necessários para coleções, mas adicionam complexidade significativa ao sistema de tipos.
- Módulos e versionamento. Como uma `action` evolui sem quebrar consumidores?
- Como integrar com o ecossistema JavaScript/TypeScript sem vazar seus problemas de volta.

---

*Este é um documento vivo. Cada seção é um convite à discussão, não uma decisão tomada.*
