# Guia de tradução: técnico → impacto de negócio

Aplicar na Fase 2, ao escrever a prosa da release e do roadmap.

## Regras

1. **Lidera pelo impacto, não pelo mecanismo.** Diga o que mudou para a empresa/cliente,
   não como foi feito tecnicamente.
2. **Quantifica quando o card permite.** Puxe números reais das descrições ($/%/tempo).
3. **Zero jargão na prosa principal.** Sem nomes de ferramentas internas, siglas de infra ou
   termos de implementação. Se um termo for inevitável, traduza entre parênteses em linguagem
   comum. Jargão só sobrevive no apêndice de rastreabilidade.
4. **Agrupa por tema de negócio.** Use `labels`/`components` primeiro; fallback por palavra-chave
   no resumo/descrição.
5. **Uma frase de "por que importa"** por tema: o benefício pra empresa/cliente, não pro time.
6. **Tom:** executivo, conciso, confiante. Escreva no idioma do board (PT-BR com acentuação
   correta quando for o caso).
7. **Nunca invente impacto.** Card sem evidência de benefício → nota "Revisar" para o dono decidir.

## Temas-semente (ajuste por board)

| Tema | Emoji | Pega cards sobre… |
|------|-------|-------------------|
| Custo / Eficiência | 💰 | redução de gastos, otimização de recursos, automação que poupa trabalho |
| Segurança & Conformidade | 🔒 | controle de acesso, proteção de dados, correções de vulnerabilidade, auditoria |
| Confiabilidade | 🛡️ | estabilidade, recuperação de falhas, redução de incidentes |
| Performance | ⚡ | velocidade, latência, capacidade, tempo de resposta |
| Plataforma / Experiência | 🧱 | fundações, novas capacidades, melhorias de fluxo para usuários/times |

Estes temas são um ponto de partida. Adapte os nomes ao vocabulário do board em questão.

Heurística de fallback: se nenhum label casar, classifique pela primeira palavra-chave do resumo;
se ainda ambíguo, escolha o tema do *benefício predominante* (ex.: uma melhoria que derruba uso de
CPU é Performance, mas se o card a enquadra como economia, é Custo).

## Exemplos (técnico → negócio)

- "Padronizar etiquetagem de custos na nuvem" → "Deixamos os relatórios financeiros de
  infraestrutura mais simples e confiáveis."
- "Configurar domínio de cookie compartilhado entre subdomínios" → "O login passou a funcionar de
  forma fluida entre os diferentes endereços do produto."
- "Adicionar índice que resolve CPU a 90%" → "Eliminamos um gargalo que sobrecarregava o banco,
  deixando as consultas mais rápidas e o sistema mais estável."
