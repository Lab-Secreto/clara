# Clara

> Uma linguagem experimental para a era da colaboração entre humanos e LLMs.
> Sua tese: a melhor linguagem para pessoas comuns e modelos de linguagem não é a que minimiza caracteres — é a que maximiza clareza semântica.

![status](https://img.shields.io/badge/status-experimental-orange)
![version](https://img.shields.io/badge/version-0.0.1-blue)
![license](https://img.shields.io/badge/license-Apache--2.0-green)
![stage](https://img.shields.io/badge/stage-design_proposal-lightgrey)

---

> ⚠️ **Status: Proposta conceitual.** Clara ainda não tem compilador, parser ou interpretador. Este repositório contém a proposta de design da linguagem e é um documento vivo em discussão. Contribuições, críticas e experimentos mentais são bem-vindos.

---

## A cara da linguagem

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

Um humano entende. Um LLM gera sem erro. Um auditor audita. A mesma fonte gera documentação, testes, schemas de API e formulários.

## Por que Clara existe

Linguagens tradicionais foram otimizadas para programadores experientes economizarem teclas. Isso fazia sentido quando o único leitor do código era outro programador. Hoje, código é coescrito com LLMs, revisado por auditores e mantido por equipes mistas de tecnologia, produto e operação. O custo de clareza mudou de lado.

Clara aposta em três compromissos:

- **Clareza acima de concisão.** Código é lido muito mais vezes do que escrito.
- **Explicitude acima de mágica.** Efeitos, dependências e falhas ficam visíveis na assinatura.
- **Uma forma canônica.** Uma maneira principal de fazer cada coisa, não dez atalhos equivalentes.

## Os sete pilares

1. **Sintaxe pequena e regular** — vocabulário fechado, zero ambiguidade.
2. **Blocos semânticos com intenção** — `compute`, `decision`, `effect`, `action`, `entity`, `policy`.
3. **Tipos semânticos de domínio** — `money_brl`, `cpf`, `email`, `percentage`, unidades nativas.
4. **Contratos explícitos** — `reads:`, `writes:`, `calls:`, `raises:` verificados pelo compilador.
5. **Erros como valores** — nada de exceções silenciosas, todos os caminhos visíveis.
6. **Exemplos embutidos** — especificação executável dentro do código.
7. **Explainability nativa** — `explain <bloco> for <público>` é parte da linguagem.

## Para quem é

Clara é desenhada para **software de negócio, orquestração, automação, regras de domínio, pipelines de dados e sistemas auditáveis**. A camada onde regras, decisões e efeitos encontram pessoas.

Clara **não é** para kernels, drivers, engines gráficas ou código científico de altíssima performance. Para isso, continue usando Rust, C++ ou a linguagem que já resolve bem.

## Documentação

- 📘 [**DESIGN.md**](./DESIGN.md) — Proposta completa: filosofia, fundamentos técnicos, sintaxe, explainability, interop e riscos conhecidos.
- 🗺️ [**ROADMAP.md**](./ROADMAP.md) — Fases de implementação e o que vem a seguir.
- 🤝 [**CONTRIBUTING.md**](./CONTRIBUTING.md) — Como participar da discussão e contribuir com o design.

## Como acompanhar

Clara está em fase de design, não de implementação. A forma mais útil de contribuir agora é:

- ⭐ Marcar o repositório para acompanhar a evolução
- 💬 Abrir uma issue com críticas, dúvidas ou propostas de design
- 🧪 Experimentar escrever um problema real em Clara e compartilhar o resultado
- 📖 Ler o [DESIGN.md](./DESIGN.md) e apontar contradições, lacunas ou ideias melhores

## Licença

Apache License 2.0. Veja [LICENSE](./LICENSE) e [NOTICE](./NOTICE).

---

*Clara é um documento vivo. Cada seção é um convite à discussão, não uma decisão tomada.*
