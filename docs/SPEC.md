# SPEC — infra-release-skill

**Status:** implementado
**Repositório:** repo git independente (skill publicável do Claude Code)

---

## 1. Problema

Boards Jira acumulam cards na coluna de concluídos. Antes de limpar o board, queremos:

1. Saber, como admin, **o que fazer pros cards finalizados sumirem** do board.
2. Saber **como reacessar** esses cards depois que sumirem.
3. Produzir um **texto de fechamento (release)** dos concluídos, para stakeholders —
   entendimento da **empresa**, não do time técnico. Sem passar card a card.
4. Dar um **overview do que está em andamento e do que está selecionado para entrega**.

Tudo isso como um **skill reutilizável em qualquer board** — sem nada embutido sobre um
board, site ou espaço específico.

## 2. Decisões de design

| # | Decisão | Escolha |
|---|---------|---------|
| Escopo de cada rodada | Quais concluídos entram | **Tudo que está concluído no momento** (sem watermark persistente) |
| Destino do texto | Onde publica | **Confluence** (oficial) + **Slack** (texto que o usuário posta; skill nunca posta) |
| Sumir do board | Mecanismo | **Limpeza manual guiada** — skill é **read-only no Jira**; entrega runbook, usuário executa |
| Construção | Abordagem | **Skill procedural puro** (SKILL.md + templates + reference) |
| Configuração | Como obtém site/board/espaço | **Setup interativo na 1ª vez**, persistido em config por usuário fora do repo |
| Estrutura do conteúdo | 1 ou 2 páginas | **Uma página só**: Entregamos → Em andamento → Vem aí |
| Plano Jira (arquivar) | Premium? | **Runbook verifica em runtime** e ramifica (arquivar vs fallback) |

## 3. Restrições técnicas conhecidas

- O MCP do Atlassian transiciona issues, roda JQL e cria/edita páginas Confluence
  (`createConfluencePage` aceita `parentId` e `status: draft`). **NÃO** mexe em config de board
  (colunas/sub-filtros/sprints) nem arquiva issues. → por isso "sumir do board" é guiado.
- Buscar status por **nome localizado** pode retornar vazio em sites traduzidos. **Mitigação:**
  filtrar por `statusCategory`.
- O MCP **não cria espaços** no Confluence — o espaço alvo precisa existir.

## 4. Arquitetura do skill

Skill procedural. Estrutura do repo:

```
infra-release-skill/
├── SKILL.md                         # núcleo: setup + esquema de config + procedimento das 5 fases
├── README.md
├── install.sh                       # symlink p/ ~/.claude/skills/infra-release + validação
├── config.example.yaml              # formato da config por usuário
├── templates/
│   ├── confluence-release.html      # template HTML da página
│   └── slack-announcement.md        # template mrkdwn do post
├── reference/
│   ├── jql-recipes.md               # queries (placeholders {{PROJECT_KEY}}/{{NEXT_STATUS}})
│   ├── business-translation.md      # guia técnico → impacto de negócio
│   └── board-clear-runbook.md       # limpeza guiada + reacesso (placeholders)
└── docs/
    ├── SPEC.md                      # este documento
    └── PLAN.md
```

**Config por usuário** (fora do repo, em `${XDG_CONFIG_HOME:-$HOME/.config}/infra-release/config.yaml`):
`cloudId`, `jiraBaseUrl`, `projectKey`, `doneStatusCategory`, `inProgressStatusCategory`,
`nextStatusName`, `confluence.spaceId`, `confluence.parentId`, `releaseTitlePrefix`.

## 5. Fluxo de execução

**Setup (1ª vez ou config incompleto):** descobre o site via `getAccessibleAtlassianResources`,
oferece projetos via `getVisibleJiraProjects`, confirma o mapa de status, lista espaços via
`getConfluenceSpaces`, pergunta o prefixo de título — e grava o config.

**5 fases (com checkpoints ✋):**

| Fase | Ação | Escreve? | Checkpoint |
|------|------|----------|-----------|
| 0. Escopo | Conta concluídos e confirma o fechamento | — | ✋ confirma |
| 1. Coleta | JQL concluídos + em andamento + selecionados; pagina; extrai com `jq` | não | — |
| 2. Síntese | Agrupa por tema de negócio; escreve release + roadmap (não-técnico) | não | — |
| 3. Confluence | Cria a página como **rascunho** no espaço/folder do config | sim (draft) | ✋ revisa antes de publicar |
| 4. Slack | Gera texto mrkdwn (manchete + bullets + link) num bloco de código | não (só gera) | ✋ posta manualmente |
| 5. Limpeza guiada | Entrega runbook + receita de reacesso | não | ✋ executa no Jira |

## 6. Conteúdo (uma página)

1. **Resumo executivo (TL;DR)** — 4-6 marcadores de impacto.
2. **Entregas por tema** — 1 parágrafo não-técnico por tema, com os cards dobrados dentro.
3. **Apêndice — rastreabilidade** — tabela `Card | Resumo | Tema` com link.
4. **O que vem aí (roadmap)** — "Em andamento" + "Selecionado pra entrega", por tema.
5. **Notas "Revisar"** (se houver) — cards que precisam de decisão humana de enquadramento.

Regras de tradução técnico→negócio em `reference/business-translation.md`. Temas-semente são
ponto de partida, ajustáveis por board.

## 7. Limpeza guiada + reacesso

Runbook em `reference/board-clear-runbook.md`: verificação de plano → arquivar em massa
(recomendado) ou fallback com label `released`. Reacesso pela página do Confluence (canônica),
pela view de *Archived issues* e pelas URLs de cada card.

## 8. Fora de escopo (YAGNI)

- Watermark/estado persistente entre rodadas (decisão: "tudo concluído agora").
- Skill transicionar/arquivar cards automaticamente (decisão: limpeza guiada).
- Skill postar no Slack automaticamente (decisão: usuário posta).
- Criação de espaço Confluence (MCP não cria espaços).

## 9. Critérios de aceite

- [x] Skill é **read-only no Jira** em toda a execução.
- [x] Setup interativo grava config por usuário; nada específico embutido no skill.
- [x] Página criada como **rascunho** no espaço/folder configurados, título carimbado por data.
- [x] Página tem TL;DR + entregas por tema + apêndice + roadmap, em linguagem não-técnica.
- [x] Texto de Slack gerado em mrkdwn, num bloco de código, **sem** postagem automática.
- [x] Runbook de limpeza com verificação de plano + caminhos A/B + reacesso.
- [x] Re-rodar não duplica página (atualiza o rascunho).
- [x] `install.sh` disponibiliza o skill em `~/.claude/skills/`.
