# infra-release-skill

Skill do Claude Code para **fechar um board Jira em ciclos de release** e produzir uma
**release para stakeholders** no Confluence, um **roadmap** do que vem aí, e um **texto de Slack**
pronto para postar — sem passar card a card e sem virar relatório técnico.

Funciona com **qualquer board Jira**: na primeira vez, o skill faz um **setup interativo**
(descobre o site Atlassian, pergunta o board, os status e o espaço do Confluence) e grava a
configuração num arquivo por usuário. Nada de board/site/espaço fica embutido no skill.

## O que ele faz

1. Lê os cards **concluídos** do board (read-only, via MCP Atlassian).
2. Traduz o trabalho técnico em **impacto de negócio**, agrupado por tema.
3. Cria uma **página de release** no Confluence como **rascunho**.
4. Gera um **anúncio de Slack** (mrkdwn) para você postar manualmente.
5. Entrega um **runbook de limpeza guiada** do board + como **reacessar** os cards depois.

## Garantias

- **Read-only no Jira:** nunca transiciona, arquiva ou edita issues.
- **Confluence draft-first:** a página nasce como rascunho até seu OK.
- **Slack manual:** o skill nunca posta sozinho.

## Instalação

```bash
./install.sh
```
Cria um symlink em `~/.claude/skills/infra-release`. Reinicie a sessão do Claude Code.

## Uso

```
/infra-release
```
Na primeira vez, responda o setup. A config fica em
`${XDG_CONFIG_HOME:-$HOME/.config}/infra-release/config.yaml` (veja `config.example.yaml`).
Para reconfigurar, apague esse arquivo ou peça "refazer setup".

## Estrutura

- `SKILL.md` — procedimento + setup + esquema de config
- `config.example.yaml` — formato da configuração por usuário
- `reference/` — queries JQL, guia de tradução, runbook de limpeza
- `templates/` — HTML da página Confluence, mrkdwn do Slack
- `docs/` — SPEC e PLAN
