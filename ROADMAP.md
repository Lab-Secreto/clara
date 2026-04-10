# Clara Roadmap

Clara é construída em fases deliberadamente pequenas. Cada fase precisa validar algo antes da próxima começar.

## Fase 0 — Design (atual)

**Objetivo:** estabilizar a proposta conceitual e validar premissas com casos reais.

- [x] Proposta inicial v0.0.1 (DESIGN.md)
- [ ] Validar com caso real: escrever uma regra de negócio não trivial em Clara, TypeScript e Python, e comparar compreensão com três públicos (engenheiro, operador, analista)
- [ ] Testar geração por LLM: pedir a um modelo para gerar 20 exemplos nas três linguagens a partir de descrições em português; medir erro, consistência e tempo
- [ ] Fechar decisões em aberto do Apêndice A do DESIGN.md
- [ ] Rascunhar gramática EBNF mínima

**Critério para avançar:** Clara precisa ganhar claramente em compreensão e confiança de modificação nos testes com humanos. Se não ganhar, o design volta para a prancheta.

## Fase 1 — Interpretador v0.1

**Objetivo:** rodar o exemplo `approve_order` end-to-end.

- [ ] Parser em TypeScript
- [ ] AST tipada
- [ ] Verificador de tipos semânticos básicos (`money_brl`, `percentage`, `email`)
- [ ] Interpretador direto (sem IR ainda)
- [ ] Execução de `examples:` como testes
- [ ] Primeira versão de `explain <bloco> for <público>`

## Fase 2 — Transpilador para TypeScript

**Objetivo:** Clara gerando código TS utilizável em projetos reais.

- [ ] Backend TypeScript
- [ ] Integração com Zod ou equivalente para validação em runtime
- [ ] Geração de schemas JSON e OpenAPI a partir de `entity`
- [ ] Primeiro caso real em produção (interno, baixo risco)

## Fase 3 — Clara IR e múltiplos backends

**Objetivo:** separar frontend semântico de backends executáveis.

- [ ] Especificação da Clara IR
- [ ] Refatorar transpilador TS para usar a IR
- [ ] Backend Go
- [ ] Backend SQL (para queries declarativas)

## Fase 4 — Compilação nativa

**Objetivo:** performance séria para módulos puros.

- [ ] Backend LLVM via MLIR
- [ ] Compilação seletiva: `compute` puro vira nativo, resto fica em runtime gerenciado
- [ ] Benchmarks contra Go e Rust para cálculos puros

## Fases não comprometidas

Coisas que podem ou não acontecer, dependendo do que Clara provar ser:

- Ecossistema de pacotes
- LSP e integração com editores
- Ferramentas de auditoria e diff semântico
- Runtime distribuído para workflows
- Integração profunda com LLMs (geração assistida como parte do tooling)

---

**Princípio do roadmap:** nenhuma fase começa antes da anterior validar sua premissa. Se a Fase 0 mostrar que Clara não é melhor que TypeScript bem tipado para humanos e LLMs, o projeto para — e isso é um resultado válido.
