---
name: infra-release
description: Use ao fechar um board Jira em ciclos de release — gera uma página de release para stakeholders no Confluence (rascunho) a partir dos cards concluídos, um roadmap do que está em andamento/selecionado, e um anúncio de Slack para postar manualmente. Read-only no Jira; faz um setup interativo na primeira vez (board, status, espaço Confluence).
---

# infra-release

Fecha um board Jira em ciclos de release: produz uma release **para stakeholders** (não para o
time técnico), um roadmap do que vem aí, e um texto de Slack pronto pra postar.

**Garantias:** Read-only no Jira (nunca transiciona/arquiva/edita). Confluence sempre em **rascunho**.
Slack **nunca** é postado automaticamente.

## Config (por usuário, fora do repo)

Arquivo: `${XDG_CONFIG_HOME:-$HOME/.config}/infra-release/config.yaml`.
Formato em `config.example.yaml`. Campos:

```yaml
cloudId: ""                       # site Atlassian (UUID)
jiraBaseUrl: ""                   # ex.: https://yoursite.atlassian.net
projectKey: ""                    # ex.: OPS
doneStatusCategory: "Done"        # categoria dos concluídos (entram na release)
inProgressStatusCategory: "In Progress"
nextStatusName: ""                # status do "vem aí" (ex.: "To Do" / "Ready"); vazio = pular
confluence:
  spaceId: ""                     # espaço onde a página é criada
  parentId: ""                    # folder ou página pai (opcional; vazio = raiz do espaço)
releaseTitlePrefix: "Release Notes"  # prefixo do título; o título final recebe " — AAAA-MM-DD"
```

Nada de site/board/espaço fica embutido no skill — tudo vem deste arquivo.

## Fase de Setup (primeira vez, ou quando o config falta/está incompleto)

Ao iniciar, leia o config. Se não existir ou faltar campo obrigatório
(`cloudId`, `projectKey`, `confluence.spaceId`), rode o setup:

1. **Site Atlassian:** chame `getAccessibleAtlassianResources`. Se houver mais de um site,
   liste e peça o usuário escolher → grave `cloudId` e `jiraBaseUrl` (a `url` do recurso).
2. **Board/projeto:** ofereça `getVisibleJiraProjects` (ou peça a chave) → grave `projectKey`.
3. **Mapa de status:** confirme `doneStatusCategory`/`inProgressStatusCategory` (defaults acima).
   Para o "vem aí", liste os status do projeto (use `getTransitionsForJiraIssue` num card de
   exemplo, com `includeUnavailableTransitions: true`) e peça qual representa "selecionado para
   entrega" → grave `nextStatusName` (ou deixe vazio para pular essa seção).
4. **Confluence:** chame `getConfluenceSpaces` → peça o espaço → grave `spaceId`. Pergunte o
   `parentId` (folder/página pai) ou deixe vazio.
5. **Título:** pergunte o `releaseTitlePrefix` (default "Release Notes").
6. Grave o `config.yaml` (crie o diretório com `mkdir -p`) e mostre o resumo da config ao usuário.

Reconfigurar = apagar o `config.yaml` e rodar de novo, ou pedir explicitamente "refazer setup".

## Procedimento (5 fases, com checkpoints ✋)

Todos os valores entre `{...}` vêm do config.

### Fase 0 — Escopo ✋
Conte os concluídos (`computeIssueCount`) e confirme: "fechar tudo que está concluído no board
`{projectKey}` agora?". Só siga com o OK.

### Fase 1 — Coleta (read-only)
Rode as 3 queries de `reference/jql-recipes.md`, substituindo os placeholders pelo config
(`{projectKey}`, `{nextStatusName}` etc.). Pagine e extraia com `jq` conforme o guia.
Filtre por `statusCategory`, nunca por nome de status localizado.

### Fase 2 — Síntese
Aplique `reference/business-translation.md`: agrupe os concluídos por tema de negócio e escreva
o TL;DR + parágrafos por tema (não-técnicos, quantificados). Monte o roadmap a partir de
EM_ANDAMENTO + SELECIONADOS. Cards sem enquadramento confiável vão para "Revisar".

### Fase 3 — Confluence (draft) ✋
Preencha `templates/confluence-release.html` (marcador `{{RELEASE_TITLE}}` = `{releaseTitlePrefix}`)
e crie a página com `createConfluencePage`: `spaceId = {confluence.spaceId}`,
`parentId = {confluence.parentId}` (omita se vazio), `contentFormat: html`, `status: draft`,
título = `"{releaseTitlePrefix} — {AAAA-MM-DD}"`.
**Idempotência:** antes de criar, procure página com o mesmo título; se existir,
`updateConfluencePage` no rascunho em vez de duplicar.
Se o `parentId` for uma folder e a API recusar, crie/use uma página índice de releases sob a folder
e aninhe a release nela.
Apresente o link do rascunho e peça revisão antes de publicar.

### Fase 4 — Slack (manual) ✋
Preencha `templates/slack-announcement.md` com a URL da página e apresente o resultado **dentro de
um bloco de código** para o usuário copiar. **Não poste.**

### Fase 5 — Limpeza guiada ✋
Entregue o conteúdo de `reference/board-clear-runbook.md` (substituindo `{projectKey}` e
`{jiraBaseUrl}`): verificação de plano + caminho A/B + reacesso. O skill **não** transiciona nem
arquiva nada.

## Trilhos de segurança
- NUNCA chamar transição/edição/arquivamento de issue no Jira.
- NUNCA postar no Slack.
- Confluence só como `draft` até OK explícito.
