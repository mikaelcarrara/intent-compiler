# Especificação do Frontmatter - Protocol-Driven Development

## Campos Obrigatórios

### 1. **Version** (string, semver)
- Formato: `X.Y.Z` (major.minor.patch)
- Exemplo: `2.1.0`
- Responsabilidade: Define a versão do protocolo para tracking de mudanças

### 2. **Model** (string)
- Formato: `provider/model-name` ou `model-name`
- Exemplos: `anthropic/claude-sonnet-4`, `gpt-4`, `gemini-pro`
- Responsabilidade: Especifica qual modelo AI usar

### 3. **Author** (string)
- Formato: Nome do time ou pessoa responsável
- Exemplo: `platform-team`, `john.doe@company.com`
- Responsabilidade: Ownership e accountability

### 4. **Schema** (object/JSON)
- Formato: JSON Schema válido ou objeto de definição
- Exemplo:
```json
{
  "type": "object",
  "properties": {
    "score": {"type": "number", "minimum": 0, "maximum": 100},
    "issues": {"type": "array", "items": {"type": "object"}},
    "summary": {"type": "string"}
  },
  "required": ["score", "issues", "summary"]
}
```
- Responsabilidade: Validação do output do modelo

## Campos Opcionais

### 5. **Description** (string)
- Descrição breve do propósito do protocolo
- Máximo: 280 caracteres

### 6. **Tags** (array)
- Lista de tags para organização e descoberta
- Exemplo: `["analysis", "security", "performance"]`

### 7. **Timeout** (number)
- Tempo máximo de execução em segundos
- Padrão: 30
- Máximo: 300

### 8. **Temperature** (number)
- Controle de criatividade do modelo
- Range: 0.0 - 2.0
- Padrão: 0.7

### 9. **MaxTokens** (number)
- Limite máximo de tokens na resposta
- Padrão: 4096

### 10. **Metadata** (object)
- Metadados adicionais customizados
- Exemplo: `{"environment": "production", "team": "ml"}`

## Estrutura do Arquivo .md

```markdown
---
# [Protocol] Nome do Protocolo
Version: 1.0.0
Model: anthropic/claude-sonnet-4
Author: platform-team
Description: Analisa performance de código
Tags: ["analysis", "performance"]
Timeout: 60
Temperature: 0.5
MaxTokens: 2048
Schema:
  type: object
  properties:
    score: {type: number, minimum: 0, maximum: 100}
    issues: {type: array, items: {type: object}}
    summary: {type: string}
  required: [score, issues, summary]
Metadata:
  environment: production
  team: platform
---

## Context
[Você é um especialista em...]

## Slots
{{code_snippet}}  string — código para analisar
{{strictness}}   int(1..10) — nível de rigor

## Constraints
1. Output DEVE ser JSON válido
2. Nunca sugira bibliotecas externas
3. Strictness < 5: apenas issues críticas

## Schema
{...}
```

## Validações Obrigatórias

1. **Version**: Deve ser semver válido
2. **Model**: Não pode estar vazio
3. **Author**: Não pode estar vazio
4. **Schema**: Deve ser JSON Schema válido
5. **Slots declarados**: Todos os slots usados no markdown devem ser declarados
6. **Tipos de slots**: Devem ter tipos válidos (string, int, float, bool, array, object)

## Exemplos de Tipos de Slots

- `{{nome}} string` — string simples
- `{{idade}} int(0..120)` — inteiro com range
- `{{preco}} float(0.0..9999.99)` — float com range
- `{{ativo}} bool` — booleano
- `{{tags}} array[string]` — array de strings
- `{{config}} object` — objeto complexo