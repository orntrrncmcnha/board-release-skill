# JQL recipes — coleta da Fase 1

Substitua `{{PROJECT_KEY}}` e `{{NEXT_STATUS}}` pelos valores do config
(`projectKey`, `nextStatusName`). Use o `cloudId` do config nas chamadas.

> **Sempre filtre por categoria de status, não por nome.** Em sites localizados, buscar pelo nome
> traduzido do status (ex.: `status = "Em andamento"`) pode retornar vazio. `statusCategory` é estável.

## Queries

**CONCLUÍDOS** (entram na release):
```
project = {{PROJECT_KEY}} AND statusCategory = Done ORDER BY resolutiondate DESC
```

**EM_ANDAMENTO** (roadmap — "está rolando"):
```
project = {{PROJECT_KEY}} AND statusCategory = "In Progress" ORDER BY updated DESC
```

**SELECIONADOS** (roadmap — "vem aí"; só se `nextStatusName` estiver definido):
```
project = {{PROJECT_KEY}} AND status = "{{NEXT_STATUS}}" ORDER BY updated DESC
```

## Campos a pedir

`["key","summary","description","issuetype","labels","components","assignee","resolutiondate","parent","updated"]`

Body em markdown: passar `responseContentFormat: "markdown"` para descrições legíveis.

## Paginação + extração

1. Chamar `searchJiraIssuesUsingJql` com `maxResults: 100`.
2. Se a resposta estourar o limite de tokens do tool, ela é salva em arquivo automaticamente —
   ler do arquivo com `jq` em vez de despejar no contexto.
3. Repetir com `nextPageToken` até `pageInfo.hasNextPage = false`.

Exemplos de extração com jq sobre o arquivo salvo `$F`:
```bash
# inventário compacto: chave + tipo + resumo
jq -r '.issues.nodes[] | "\(.key)\t\(.fields.issuetype.name)\t\(.fields.summary)"' "$F"
# labels por card (pra tematização)
jq -r '.issues.nodes[] | "\(.key)\t\(.fields.labels | join(","))"' "$F"
# contagem por categoria de status (sanidade)
jq -r '.issues.nodes[] | .fields.status.statusCategory.name' "$F" | sort | uniq -c
```

## Contagem (quando precisar do total)

Usar `computeIssueCount: true` com `maxResults: 1` numa query só pra contar — não combinar com paginação real.
