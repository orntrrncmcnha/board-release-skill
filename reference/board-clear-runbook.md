# Runbook — limpeza guiada do board + reacesso

Substitua `{{PROJECT_KEY}}` e `{{JIRA_BASE_URL}}` pelos valores do config.
O skill é **read-only no Jira**: ele só entrega estes passos; **você (admin) executa**.

## Passo 0 — verificar o plano (faça 1 vez)

No issue navigator, abra `project = {{PROJECT_KEY}} AND statusCategory = Done` e cheque se aparece
**Bulk change → Archive issues**.
- **Aparece** → use o **Caminho A** (Premium/Enterprise).
- **Não aparece** → plano Standard, use o **Caminho B**.

## Caminho A — arquivar em massa (recomendado)

1. Issue navigator → `project = {{PROJECT_KEY}} AND statusCategory = Done`.
2. Selecionar tudo → **Bulk change → Archive issues** → confirmar.

Por que arquivar (não deletar): issues arquivados somem do board **e** das buscas normais, mas
continuam **recuperáveis**. Bônus: como saem do JQL `statusCategory = Done`, o **próximo fechamento
não os repesca** — é o que faz o modelo "fechar tudo que está concluído agora" funcionar sem
precisar guardar estado entre rodadas.

## Caminho B — fallback (Standard, sem arquivamento)

1. Bulk edit: adicionar o label `released` a todos os concluídos.
2. Board settings → ajustar o **sub-filtro** para esconder `statusCategory = Done` com esse label
   (ou esconder concluídos por idade).
3. Nas próximas rodadas, a query de concluídos vira:
   `project = {{PROJECT_KEY}} AND statusCategory = Done AND labels != released`.

## Reacesso (depois que sumirem)

1. **Página do Confluence** — fonte canônica: por tema, com link de cada card.
2. **Jira** — nada é deletado:
   - *Archived issues* (view de admin) para ver/restaurar (Caminho A).
   - Qualquer card abre por `{{JIRA_BASE_URL}}/browse/{{PROJECT_KEY}}-XXX`, mesmo arquivado.
   - Caminho B: `project = {{PROJECT_KEY}} AND labels = released`.
3. **Índice de releases** no Confluence — histórico de todos os fechamentos.
