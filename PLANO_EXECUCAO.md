# Plano de Execução — intent-compiler

## Posicionamento do produto

> **intent-compiler é um compilador de intenção humana em Markdown.**
> Markdown进来了 → estruturas determinísticas出去了.

O produto resolve um problema real: **Markdown virou interface universal de dev** ( READMEs, specs, docs, prompts), mas não tem governança. intent-compiler transforma Markdown em **artefatos determinísticos validados** — não é editor, não é "IA interpretando texto", é um **resolvedor de intenção**.

### Frases de позиционирование (usar em todo lugar)
- "Markdown that compiles to X"
- "Intent compiler for structured protocols"
- "Deterministic Markdown → structured artifacts"

### Modelo de output do MVP
O toolchain do MVP gera:
- **JSON Schema validado** (saída do `SchemaValidator`)
- **Relatório de lint** (table/compact/json via CLI)
- **Exit codes** (0 = válido, 1 = erro, 2 = argumento inválido)

Futuros outputs planejados:
- UI renderizada (componentes)
- API mocks
- prompts estruturados
- código gerado

### Conceito central: Resolver
> **Resolver** = engine que transforma Markdown em artefato executável/validável, de forma determinística.

---

## Frente 1 — Essencial para MVP

### Escopo do ciclo completo
- Usuário escreve Markdown com frontmatter, slots e schema.
- CLI valida estrutura e gera artefatos determinísticos (schema, relatório).
- Saída legível + JSON para CI.
- Exit code bloqueia pipeline em caso de erro.
- Repositório inclui exemplos e testes mínimos.

> ⚠️ **Nota de позиционирование**: o MVP não é "um linter". É o primeiro ciclo completo do resolver — de Markdown a artefato validado.

### Checklist de implementação

#### 1) Núcleo de parsing e validação
- [x] Parser de protocolo Markdown (`parser.py`)
- [x] Frontmatter YAML com chaves normalizadas (case-insensitive)
- [x] Validação de consistência de slots declarados vs usados
- [x] Parsing de `## Schema` com suporte a code fences
- [x] SchemaValidator baseado em JSON Schema Draft 7
- [x] ProtocolValidator integrando parser + schema validator
- [x] Validação determinística de schema com `check_schema`

#### 2) Produto utilizável via CLI (Resolver CLI)
- [x] CLI `intent lint` executável por arquivo e diretório
- [x] Formatos de saída: table, compact e json
- [x] Modo `--strict` (warnings viram erro)
- [x] Códigos de saída consistentes para CI
- [x] `proto resolve` — comando que transforma Markdown em artefato (JSON/YAML, exit codes)

#### 3) Qualidade mínima de entrega
- [x] Suite de testes automatizados para parser/validator/CLI
- [x] Exemplos de protocolo válido e inválido
- [x] Deploy básico de documentação/site no GitHub Pages
- [x] Pipeline CI baseline (syntax + testes + smoke da CLI)

### Definition of Done do MVP
- [x] Comando `intent lint` funciona em diretório com múltiplos `.md`
- [x] Erros críticos retornam exit code `1`
- [x] Modo sem erros retorna exit code `0`
- [x] Formato JSON consumível por pipeline CI
- [x] Testes automatizados passam no repositório
- [x] `proto resolve` gera artefato determinístico a partir de Markdown (JSON e YAML)

---

## Frente 2 — Improvements / Next Steps

### Positioning e naming
- [x] Rename de `proto-md` para `intent-compiler` ✅
- [x] Atualizar tagline em todos os pontos de contato para фокус em "compilação" não "lint" ✅

### Produto e DX
- [x] **Hero example com wow moment**: mostrar Markdown → UI/JSON/artefato em tempo real ✅
- [x] Configuração via `.intent.yaml` com regras customizáveis ✅
- [x] Mensagens de erro com contexto de linha e seção
- [x] Melhorias visuais de output (cores ANSI, tabela rica)

### Output model expandido
- [x] `proto resolve` — comando que transforma Markdown em artefato ✅
- [x] Suporte a múltiplos backends de output (JSON, YAML, **TypeScript interfaces**) ✅
- [x] Geração de prompts estruturados a partir de slots (`--prompt`) ✅
- [ ] Geração de mocks de API a partir de schema

### Ecossistema
- [ ] SDK Python completo (publicação em PyPI)
- [ ] SDK TypeScript completo (publicação em npm)
- [ ] VS Code Extension com hover, autocomplete e lint inline
- [ ] Ferramentas auxiliares (`proto-init`, `proto-generate`, `proto-test`)

### Qualidade e hardening
- [ ] Cobertura de testes > 90%
- [ ] Pipeline de qualidade completo (lint/typecheck/test/security)
- [ ] Testes de integração com provedores LLM
- [ ] Hardening de segurança (SAST, dependabot, etc.)

---

## Próxima prioridade imediata (após MVP)
1. Hero example com wow moment ✅ (live demo no index.html com parsing client-side)
2. `proto resolve` como primer comando de output do resolver ✅ (implementado)
4. Configuração via `.intent.yaml` com regras customizáveis
