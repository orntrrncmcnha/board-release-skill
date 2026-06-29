# infra-release-skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development ou superpowers:executing-plans. Steps usam checkbox (`- [ ]`).

**Goal:** Construir o skill `infra-release` — fecha um board Jira gerando uma release de
stakeholders no Confluence (rascunho), um roadmap, e um texto de Slack pra postar manual,
mantendo-se read-only no Jira. Genérico: site/board/espaço vêm de um setup interativo.

**Architecture:** Skill procedural nativo do Claude Code. `SKILL.md` orquestra o setup + 5 fases
(escopo → coleta via MCP → síntese → publicação draft no Confluence → Slack manual → limpeza
guiada). Templates e guias em `templates/` e `reference/`. Sem runtime próprio: o executor é o
Claude usando o MCP do Atlassian. Config por usuário fica fora do repo.

**Tech Stack:** Markdown, HTML (template Confluence), Bash (install.sh), MCP Atlassian.
Verificação via `grep`, `bash -n`, `python3` (stdlib), e instalação real do symlink.

## Global Constraints

- **Read-only no Jira:** nunca transiciona/arquiva/edita issues.
- **Confluence draft-first:** páginas com `status: draft` até OK do usuário.
- **Slack:** nunca posta automaticamente.
- **Sem hardcode:** nenhum site/board/espaço/projeto específico no skill — tudo vem do config
  por usuário (`${XDG_CONFIG_HOME:-$HOME/.config}/infra-release/config.yaml`).
- **Status por categoria:** filtrar por `statusCategory`, nunca por nome localizado.
- **Idempotência:** se já existe página com o mesmo título, atualizar o rascunho (não duplicar).
- Repo git próprio.

## File Structure

| Arquivo | Responsabilidade |
|---------|------------------|
| `README.md` | O que é, como instalar, como rodar |
| `.gitignore` | Lixo de SO/editor + `config.yaml` local |
| `config.example.yaml` | Formato da config por usuário |
| `install.sh` | Symlink `~/.claude/skills/infra-release` → repo + validação |
| `reference/jql-recipes.md` | Queries JQL (placeholders) + paginação + jq |
| `reference/business-translation.md` | Regras técnico→negócio + temas-semente genéricos |
| `reference/board-clear-runbook.md` | Limpeza guiada (placeholders) + reacesso |
| `templates/confluence-release.html` | Template HTML (TL;DR/temas/apêndice/roadmap) |
| `templates/slack-announcement.md` | Template mrkdwn |
| `SKILL.md` | Núcleo: setup interativo + esquema de config + 5 fases |
| `docs/SPEC.md`, `docs/PLAN.md` | Design e plano |

## Tasks

### Task 1 — Scaffold + git
- [x] `git init` com identidade do repo; `.gitignore` (inclui `config.yaml`); `README.md` genérico.
- Verificação: README cita "Read-only no Jira", "draft-first", "Slack manual", "/infra-release",
  "setup interativo". Commit.

### Task 2 — Config genérico
- [x] `config.example.yaml` com todos os campos (cloudId, jiraBaseUrl, projectKey, mapa de status,
  confluence.spaceId/parentId, releaseTitlePrefix), todos vazios/default.
- Verificação: `grep` dos campos. Commit.

### Task 3 — reference/jql-recipes.md
- [x] 3 queries com `{{PROJECT_KEY}}`/`{{NEXT_STATUS}}`; nota de `statusCategory` vs nome localizado;
  paginação + exemplos de `jq`.
- Verificação: `grep` por `{{PROJECT_KEY}}`, `statusCategory = Done`, `nextPageToken`, `jq -r`. Commit.

### Task 4 — reference/business-translation.md
- [x] Regras (impacto > mecanismo, quantificar, zero jargão, agrupar por tema, nunca inventar);
  temas-semente genéricos; exemplos genéricos (sem nomes próprios).
- Verificação: `grep` por "Lidera pelo impacto", "Zero jargão", "Temas-semente", "Revisar". Commit.

### Task 5 — reference/board-clear-runbook.md
- [x] Verificação de plano + Caminho A (arquivar) + Caminho B (label `released`) + reacesso,
  com `{{PROJECT_KEY}}`/`{{JIRA_BASE_URL}}`.
- Verificação: `grep` por "Passo 0", "Caminho A", "Caminho B", "Reacesso". Commit.

### Task 6 — templates
- [x] `confluence-release.html` com marcadores `{{RELEASE_TITLE}}/{{PERIODO}}/{{TLDR_ITEMS}}/`
  `{{TEMAS}}/{{ROADMAP}}/{{APENDICE_ROWS}}/{{REVISAR}}`; `slack-announcement.md` com
  `{{RELEASE_TITLE}}/{{PERIODO}}/{{BULLETS}}/{{CONFLUENCE_URL}}`.
- Verificação: `grep` dos marcadores + parse HTML (`python3 html.parser`). Commit.

### Task 7 — SKILL.md
- [x] Frontmatter `name: infra-release` + esquema de config + Fase de Setup interativo + 5 fases
  referenciando os arquivos e os valores do config; trilhos de segurança.
- Verificação: `grep` por `^name: infra-release`, `^description:`, "Fase de Setup", "Fase 0..5",
  "status: draft", "Read-only no Jira"; todos os arquivos referenciados existem. Commit.

### Task 8 — install.sh + smoke test
- [x] Symlink `~/.claude/skills/infra-release` → repo, com validação de `SKILL.md`/`name:`,
  idempotente.
- Verificação: `bash -n`; rodar `./install.sh`; symlink existe; `SKILL.md` acessível; rerun OK. Commit.

## Verificação final

- [x] Nenhum site/board/espaço específico nem nome próprio nos arquivos versionados
  (`grep -ri` por termos específicos retorna vazio).
- [x] Skill discoverable como `infra-release`.
- [ ] **Teste de aceite E2E:** rodar `/infra-release` num board real, passar pelo setup e gerar o rascunho.
