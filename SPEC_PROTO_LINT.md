# Especificação do intent-lint

## Objetivo

O `intent-lint` é uma ferramenta de linting que valida arquivos de protocolo (.md) e garante que eles estejam em conformidade com as especificações do Protocol-Driven Development. Ele deve ser executado no CI/CD e **quebrar o build** quando encontra erros críticos.

## Tipos de Erros (Quebram o Build)

### 1. **Erros de Sintaxe (CRÍTICO)**
- **Frontmatter inválido**: YAML malformado ou ausente
- **JSON Schema inválido**: Schema malformado ou com sintaxe incorreta
- **Markdown malformado**: Estrutura de markdown quebrada

```bash
✗ performance_analyst.md
  Error: Frontmatter YAML inválido - linha 5: mapping values are not allowed
  Build: FAILED
```

### 2. **Campos Obrigatórios Ausentes (CRÍTICO)**
- **Version**: Campo version ausente ou inválido
- **Model**: Campo model ausente ou vazio
- **Author**: Campo author ausente ou vazio
- **Schema**: Campo schema ausente ou inválido

```bash
✗ auth_guard.md
  Error: Campo obrigatório 'schema' ausente
  Error: Campo 'version' deve seguir semver (X.Y.Z)
  Build: FAILED
```

### 3. **Slots Inconsistentes (CRÍTICO)**
- **Slots não declarados**: Slots usados no markdown mas não declarados na seção Slots
- **Slots não utilizados**: Slots declarados mas nunca usados (warning em modo strict)
- **Tipos de slots inválidos**: Tipos que não estão na lista permitida

```bash
✗ db_optimizer.md
  Error: Slot '{{table_name}}' usado mas não declarado
  Error: Slot '{{index_type}}' declarado mas não utilizado
  Build: FAILED
```

### 4. **Validação de Schema (CRÍTICO)**
- **Schema JSON inválido**: Não conforme com JSON Schema Draft 7
- **Ciclos de referência**: Referências circulares em $ref
- **Tipos desconhecidos**: Tipos não suportados pelo JSON Schema

```bash
✗ data_validator.md
  Error: Schema JSON inválido: Additional properties not allowed: 'unkownField'
  Build: FAILED
```

### 5. **Versionamento Semântico (CRÍTICO)**
- **Formato inválido**: Não segue formato semver X.Y.Z
- **Números negativos**: Versões com números negativos
- **Zeros à esquerda**: Números com zeros à esquerda (01.02.03)

```bash
✗ api_analyzer.md
  Error: Version '2.1' inválida - deve seguir formato X.Y.Z
  Error: Version '01.2.3' inválida - não use zeros à esquerda
  Build: FAILED
```

## Tipos de Warnings (Não quebram o build, mas emitem alertas)

### 1. **Boas Práticas (WARNING)**
- **Descrição ausente**: Campo description vazio ou ausente
- **Tags ausentes**: Campo tags ausente
- **Timeout padrão**: Usando timeout padrão (30s) sem especificar

### 2. **Consistência (WARNING)**
- **Slots não utilizados**: Slots declarados mas não usados (modo normal)
- **Metadata vazio**: Campo metadata presente mas vazio
- **Temperatura padrão**: Usando temperatura padrão (0.7) sem especificar

## Comandos e Opções

### Comando Básico
```bash
intent-lint [arquivos/diretórios]
```

### Opções
```bash
--strict              # Modo strict: warnings viram erros
--format json         # Output em formato JSON
--format table        # Output em formato tabela (padrão)
--quiet               # Silencioso: apenas erros
--verbose             # Verbose: mostra detalhes de validação
--config .intent.yaml   # Arquivo de configuração
```

### Exemplos de Uso
```bash
# Lint único arquivo
intent-lint performance_analyst.md

# Lint diretório recursivo
intent-lint protocols/

# Modo strict (warnings = errors)
intent-lint --strict protocols/

# Output JSON para CI
intent-lint --format json protocols/ > lint-results.json

# Config customizada
intent-lint --config custom.intent protocols/
```

## Arquivo de Configuração (.intent.yaml)

# .intent.yaml
rules:
  required-fields:
    - version
    - model
    - author
    - schema
  
  slot-validation: strict
  
  schema-validation: strict
  
  warnings-as-errors: false  # true para modo strict
  
  ignore-patterns:
    - "*.draft.md"
    - "experiments/*"
  
  custom-rules:
    min-description-length: 10
    max-timeout: 300
    allowed-models:
      - "anthropic/*"
      - "openai/*"
      - "google/*"

output:
  format: table  # json, table, compact
  colors: true
  verbose: false
```

## Integração com CI/CD

### GitHub Actions
```yaml
name: Protocol Lint
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install intent-lint
        run: pip install intent-lint
      - name: Run lint
        run: intent-lint --strict --format json protocols/
      - name: Upload results
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: lint-results
          path: lint-results.json
```

### GitLab CI
```yaml
protocol-lint:
  stage: validate
  script:
    - pip install intent-lint
    - intent-lint --strict protocols/
  allow_failure: false  # Quebra o build em caso de erro
```

## Formatos de Output

### Formato Tabela (padrão)
```
✓ performance_analyst.md    slots: 3 declared, 3 used, schema: valid
✓ auth_guard.md              slots: 2 declared, 2 used, schema: valid
✗ db_optimizer.md            slot {{table_name}} declared but never used
✗ summarizer.md              no output schema defined — required in v2+

2 errors · 0 warnings · fix before deploy
```

### Formato JSON
```json
{
  "summary": {
    "total": 4,
    "valid": 2,
    "errors": 2,
    "warnings": 0
  },
  "files": [
    {
      "file": "performance_analyst.md",
      "status": "valid",
      "errors": [],
      "warnings": []
    },
    {
      "file": "db_optimizer.md",
      "status": "invalid",
      "errors": ["slot {{table_name}} declared but never used"],
      "warnings": []
    }
  ],
  "exit_code": 1
}
```

### Formato Compacto (para scripts)
```
performance_analyst.md: OK
auth_guard.md: OK
db_optimizer.md: ERROR - slot {{table_name}} declared but never used
summarizer.md: ERROR - no output schema defined
```

## Códigos de Saída
- **0**: Sucesso (nenhum erro)
- **1**: Erros encontrados (build quebrado)
- **2**: Erros de configuração/sistema
- **3**: Warnings em modo strict (se warnings-as-errors: true)

## Implementação Técnica

### Dependências
```python
# requirements.txt
pyyaml>=6.0
jsonschema>=4.0
click>=8.0
colorama>=0.4.0
```

### Estrutura do Código
```
intent_lint/
├── __init__.py
├── cli.py              # Interface de linha de comando
├── validator.py        # Lógica de validação
├── config.py           # Carregamento de config
├── formatters.py       # Formatadores de output
├── rules/              # Regras de validação
│   ├── __init__.py
│   ├── base.py
│   ├── frontmatter.py
│   ├── slots.py
│   ├── schema.py
│   └── versioning.py
└── utils/
    ├── __init__.py
    └── helpers.py
```

### Performance
- Deve processar 100 protocolos em < 2 segundos
- Deve usar cache para schemas já validados
- Deve ser capaz de rodar em paralelo (multi-thread)

## Testes
- Unit tests para cada regra de validação
- Integration tests para CLI
- Performance benchmarks
- Testes com exemplos reais de protocolos