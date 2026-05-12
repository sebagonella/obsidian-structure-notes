# ARCHITECTURE.md

> Documentação de alto nível da arquitetura deste vault Obsidian.
> Este arquivo descreve **como** o vault é organizado, **não o que** ele contém.
> Para regras de privacidade aplicadas a este documento, ver [`CLAUDE.md`](CLAUDE.md) desta pasta.

---

## 1. Propósito

Este vault funciona como **memória persistente** do agente assistente (Claude Code) e como sistema de pensamento e referência integrado, em português, para projetos pessoais e profissionais. Os objetivos são:

- Capturar informação rapidamente, em múltiplos canais.
- Reduzir fricção entre captura, revisão e arquivamento.
- Manter contexto reutilizável entre sessões com agentes (Claude Code).
- Servir de base para automações de manutenção (logs diários, agregação de períodos, validação de metadados).

---

## 2. Taxonomia de pastas

A estrutura combina **prefixos numéricos `NN_NOME`** para ordenação determinística e uma adaptação do método **PARA** para a área de trabalho ativo:

```
vault/
├── 00_SISTEMA/                            # meta: documentação, templates, scripts e anexos
│   ├── 00_DOCUMENTACOES/
│   │   └── 01_VAULT/
│   │       ├── PUBLICO/                   # esta pasta — espelhada em repo público
│   │       │   ├── README.md
│   │       │   ├── ARCHITECTURE.md
│   │       │   └── CLAUDE.md
│   │       └── PRIVADO/                   # documentação e prompts internos
│   ├── 01_TEMPLATES/                      # templates do Obsidian (Templater)
│   ├── 02_SCRIPTS/                        # scripts Python e shell de manutenção
│   └── 03_ANEXOS/                         # binários (Canvas, Excalidraw, PDFs, áudios)
├── 10_CALENDARIO/                         # notas periódicas
│   ├── 01_DAILY/                          # YYYY/MM/YYYY-MM-DD.md
│   ├── 02_WEEKLY/                         # YYYY/YYYY-Www.md
│   ├── 03_MONTHLY/                        # YYYY/YYYY-MM.md
│   └── 04_YEARLY/                         # YYYY.md
├── 20_PROJETOS/                           # esforços com objetivo definido
│   ├── PESSOAL/<slug>/
│   ├── PROFISSIONAL/<slug>/
│   └── ARQUIVADOS/<categoria>/<YYYY>/<slug>/
├── 30_AREAS/                              # responsabilidades contínuas (sem data de fim)
│   ├── CARREIRA/                          # desenvolvimento profissional contínuo
│   ├── FAMILIA/                           # contexto familiar
│   ├── FINANCAS/                          # planejamento financeiro
│   ├── GARAGEM/                           # manutenção de bens duráveis
│   ├── SAUDE/                             # métricas, exames, rotinas, análises
│   ├── TRABALHO/                          # responsabilidades contínuas do trabalho
│   └── VAULT/                             # meta-manutenção do vault (sessões, tarefas, decisões, pesquisas)
├── 40_RECURSOS/                           # referências, biblioteca, material durável
│   ├── LEGADO/                            # destino de importações de outros sistemas
│   └── TI/                                # referências técnicas (linguagens, ferramentas)
├── 99_INBOX/                              # captura crua, antes da triagem
├── MOC-home.md                            # índice geral do vault
└── .claude/                               # commands, skills locais e subagents do Claude Code
```

**Justificativas:**

- Prefixos numéricos garantem ordenação previsível em qualquer cliente que liste alfabeticamente.
- `00_SISTEMA/` agrega o que é "sobre o vault" e separa visualmente do conteúdo de trabalho.
- Separação `20_PROJETOS/` / `30_AREAS/` segue PARA: projetos têm objetivo e fim; áreas, não.
- `99_INBOX/` único centraliza captura — toda entrada não classificada cai aqui antes de ser triada.
- Projetos arquivados são **movidos**, nunca apagados, e ganham um sufixo de ano para preservar o histórico.

---

## 3. Convenções de nomeação

### Pastas de sistema e de conteúdo

Padrão `NN_NOME` em caixa alta com underscore para o primeiro nível (`00_SISTEMA`, `10_CALENDARIO`, `20_PROJETOS`, `30_AREAS`, `40_RECURSOS`, `99_INBOX`). Subpastas dentro de cada categoria também seguem o padrão `NN_NOME`. Pastas de projeto usam `NN_slug-em-kebab-case` dentro de `20_PROJETOS/<categoria>/`.

### Notas periódicas

| Tipo    | Caminho                              | Padrão            |
|---------|--------------------------------------|-------------------|
| Daily   | `10_CALENDARIO/01_DAILY/YYYY/MM/`    | `YYYY-MM-DD.md`   |
| Weekly  | `10_CALENDARIO/02_WEEKLY/YYYY/`      | `YYYY-Www.md`     |
| Monthly | `10_CALENDARIO/03_MONTHLY/YYYY/`     | `YYYY-MM.md`      |
| Yearly  | `10_CALENDARIO/04_YEARLY/`           | `YYYY.md`         |

### Notas de sessão

- **Sessão de manutenção do vault:** `30_AREAS/VAULT/sessoes/YYYY-MM-DD-slug.md`.
- **Sessão de projeto técnico:** `20_PROJETOS/<categoria>/<slug>/sessoes/YYYY-MM-DD-slug.md`. Cada projeto também possui um arquivo-índice `_PROJETO.md` na raiz da pasta, que carrega contexto do projeto para o agente.

### Slugs

- Kebab-case ASCII puro, sem caracteres reservados (`/ \ : * ? " < > |`).
- Exemplo *fictício*: `2025-03-14-revisao-trimestral.md`.

---

## 4. Frontmatter

Toda nota processada pelo Claude Code recebe ou preserva frontmatter YAML mínimo:

```yaml
---
data: 2025-03-14
data_atualizacao: 2025-03-14 09:30
tipo: daily | weekly | monthly | yearly | projeto | sessao | sessao-vault |
      decisao | pesquisa | tarefa | saude | analise-saude | legado |
      webclipper | moc | documentacao
status: ideia | planejamento | execucao | revisar | concluido | arquivado
tags: []
---
```

**Campos por tipo de nota:**

- **Daily:** acrescenta `dia_semana`.
- **Sessão de projeto:** acrescenta `hora`, `session_id`, `branch`, `mensagens`, `projeto: <slug>`.
- **Notas dentro de `20_PROJETOS/...`:** **obrigatoriamente** `projeto: <slug>` no frontmatter. Slug = nome da pasta mais profunda do projeto. Tarefas inline (checkbox) usam as tags `#projeto/<slug>` + `#tipo/tarefa`.
- **Notas dentro de `30_AREAS/...`:** **obrigatoriamente** `area: <slug>` no frontmatter. Slug = nome da subpasta de área em minúsculas (ex.: `area: saude`, `area: vault`). Regra simétrica à de `projeto: <slug>`; aplica-se a notas operacionais (tarefas, decisões, pesquisas) dentro da área. MOCs e sub-MOCs auto-gerados ficam isentos.
- **Pesquisa:** acrescenta `notebook_id`, `fontes`.
- **Saúde:** notas da área de saúde possuem campos numéricos para acompanhamento longitudinal (métricas físicas, sinais vitais, atividade). Os campos específicos não aparecem nesta documentação por convenção de privacidade.

**Princípio:** o frontmatter é a fonte de queries (Dataview, Tasks, Tracker). Tags hierárquicas servem para filtragem visual; valores únicos por nota ficam no frontmatter, sem duplicação.

---

## 5. Fluxos de captura

```
                                       ┌──────────────────────────┐
                                       │  Editor Obsidian         │
                                       └─────────────┬────────────┘
                                                     │
                                       ┌──────────────────────────┐
                                       │  Web clipper             │
                                       └─────────────┬────────────┘
                                                     │
                                       ┌──────────────────────────┐
                                       │  Pesquisa externa        │
                                       │  (NotebookLM)            │
                                       └─────────────┬────────────┘
                                                     │
                                       ┌──────────────────────────┐
                                       │  Sessão do agente        │
                                       │  (Claude Code)           │
                                       └─────────────┬────────────┘
                                                     │
                                                     ▼
                                       ┌──────────────────────────┐
                                       │       99_INBOX/          │
                                       │   (captura sem destino)  │
                                       └─────────────┬────────────┘
                                                     │
                                                  triagem
                                                     │
                                                     ▼
                                       ┌──────────────────────────┐
                                       │   destino na taxonomia   │
                                       │   (10/20/30/40)          │
                                       └──────────────────────────┘
```

### 5.1 Captura manual no Obsidian

Editor → template do tipo correspondente → grava em `99_INBOX/` quando a classificação ainda não está clara, ou direto na pasta de destino quando o tipo já é conhecido. Templates carregam frontmatter mínimo já preenchido.

### 5.2 Captura via web clipper

Extensão de navegador grava em `99_INBOX/` com `tipo: webclipper` e `tags: [origem/webclipper]`, preservando URL de origem e timestamp.

### 5.3 Captura via pesquisa externa (NotebookLM)

Um script orquestra `pergunta → notebook → sumarização → nota Markdown` no projeto correspondente, criando `pesquisas/YYYY-MM-DD-slug.md` com `tipo: pesquisa` e `tags: [origem/notebooklm]`. A interação com o serviço externo fica no script; o vault só guarda o resultado em texto.

### 5.4 Captura por sessão de trabalho

Toda sessão do agente (manutenção do vault ou desenvolvimento de projeto técnico) gera duas escritas:

1. Uma nota de sessão padronizada (`tipo: sessao`), em pasta apropriada à modalidade.
2. Uma linha no Log do Dia da nota daily correspondente.

Skills locais do vault (`/vault-start`, `/vault-end`, `/vault-log`) e skills globais (`/session-start`, `/session-end`) automatizam o ciclo.

### 5.5 Importação de legados

Utilitários pontuais migram conteúdo de outros sistemas de notas para `40_RECURSOS/LEGADO/`, preservando data e origem em frontmatter (`tipo: legado`, `origem_sistema: <nome>`). Esses scripts rodam sob demanda, fora do fluxo diário.

---

## 6. Templates

`00_SISTEMA/01_TEMPLATES/` contém templates parametrizados para os tipos mais comuns:

| Template                       | `tipo`           | Papel                                                       |
|--------------------------------|------------------|-------------------------------------------------------------|
| `DAILY.md`                     | `daily`          | Nota diária; abre/encerra o dia                             |
| `WEEKLY.md`                    | `weekly`         | Resumo semanal agregando dailies                            |
| `MONTHLY.md`                   | `monthly`        | Resumo mensal agregando weeklies                            |
| `YEARLY.md`                    | `yearly`         | Resumo anual agregando monthlies                            |
| `PROJETO.md`                   | `projeto`        | Arquivo-índice `_PROJETO.md` de cada projeto                |
| `SESSAO-PROJETO.md`            | `sessao`         | Sessão de trabalho num projeto técnico                      |
| `SESSAO-VAULT.md`              | `sessao`         | Sessão de manutenção do próprio vault                       |
| `DECISAO.md`                   | `decisao`        | ADR — decisão arquitetural com alternativas e consequências |
| `PESQUISA.md`                  | `pesquisa`       | Resultado consolidado de pesquisa externa                   |
| `TAREFA.md`                    | `tarefa`         | Tarefa-nota com prazo e critério de pronto                  |
| `SAUDE-DIARIO.md`              | `saude`          | Métricas e relato diário da área de saúde                   |
| `LEGADO.md`                    | `legado`         | Stub para conteúdo importado de outro sistema               |
| `MOC_PROJETO_GERAL.md`         | `moc`            | Mapa geral de projetos por contexto                         |
| `MOC_PROJETO_INDIVIDUAL.md`    | `projeto`        | Mapa individual de projeto com tarefas e queries            |
| `CLAUDE-PROJETO-TECNICO.md`    | `documentacao`   | Modelo de `.claude/CLAUDE.md` para projetos técnicos        |

**Convenção:** Templater (`<% %>`) é o padrão atual; alguns templates legados ainda usam placeholders `{{VARIAVEL}}` aceitos por um carregador embutido nos scripts. Migração é oportunística.

**Princípio:** templates são a **fonte da verdade do frontmatter**. Mudanças de convenção alteram o template antes das notas existentes.

**Saúde — fluxo dual.** Métricas longitudinais da área de saúde têm duas origens: (1) o template manual `SAUDE-DIARIO.md` (registro pontual com `tipo: saude`); (2) um pipeline auto-gerado fora deste vault que escreve diretamente em `30_AREAS/SAUDE/METRICAS/<subpasta-de-integração>/` mantendo o mesmo `tipo: saude` + tag granular `saude/<subarea>`. Os campos numéricos específicos não aparecem nesta documentação por convenção de privacidade.

**Ownership híbrido de skills.** Algumas skills locais (`.claude/skills/<nome>/`) têm fonte canônica fora do vault, em repositórios técnicos do próprio usuário. Nesses casos, a cópia versionada no vault carrega header `AUTO-GERADO` apontando para o repo de origem, e a propagação acontece via script de sync com detecção de drift por checksum (sem auto-commit). Edições devem ser feitas no repo de origem; o vault recebe a cópia revisada.

---

## 7. Integrações externas

| Componente                                | Papel                                                                  |
|-------------------------------------------|------------------------------------------------------------------------|
| Obsidian + Local REST API (plugin)        | Editor primário e endpoint local consumido pelo agente                 |
| Obsidian — Templater                      | Engine de templates                                                    |
| Obsidian — Dataview                       | Queries em MOCs e listas dinâmicas                                     |
| Obsidian — Tasks                          | Indexação de tarefas inline                                            |
| Obsidian — Tracker                        | Gráficos longitudinais a partir de frontmatter                         |
| Obsidian — Contribution Graph             | Heatmaps anuais (consistência de hábitos, contagem por dia)            |
| Obsidian — Linter                         | Mantém `data_atualizacao` consistente                                  |
| Obsidian — Smart Connections              | Indexação semântica local (estado fora de versão)                      |
| Google Drive                              | Sync entre dispositivos do vault                                       |
| Claude Code (CLI) + MCP Obsidian          | Manutenção do vault e desta documentação via REST API local            |
| Subagent `privacy-reviewer`               | Auditor isolado de privacidade para conteúdo candidato a publicação    |
| NotebookLM                                | Pesquisa externa cujo resultado é consolidado como nota                |
| GitHub (repositório público)              | Espelho de `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/`             |

Credenciais, tokens, hosts e endpoints internos **não** aparecem nesta documentação. Configurações sensíveis vivem em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PRIVADO/` e em variáveis de ambiente fora do vault.

**Espelhamento da pasta pública:** um script `rsync` (`00_SISTEMA/02_SCRIPTS/sync-publico.sh`) copia esta pasta para um clone local do repositório público, mantido **fora** do vault e fora do Google Drive. O script roda obrigatoriamente em `--dry-run` antes de qualquer cópia real, e nunca executa `git commit` ou `git push` — a confirmação humana e a publicação ficam fora do script. A separação física do clone reduz risco de leak por filtro de path mal configurado em estratégias de submodule ou subtree.

---

## 8. Convenções de tags

- Tags em **português brasileiro**, minúsculas, **sem acento**.
- Hierarquia por `/`: `contexto/profissional`, `tipo/projeto`, `ti/python`.
- Famílias estabelecidas:
  - `contexto/{pessoal, profissional}`
  - `origem/{claude-code, manual, webclipper, notebooklm, ...}`
  - `tipo/{...}` — espelha o frontmatter `tipo`
  - `status/{...}` — espelha o frontmatter `status`
  - `ti/{python, obsidian, ...}`
  - `pessoa/{...}`, `entidade/{...}`
  - `projeto/{slug}`
- Tags servem para filtragem visual; valores únicos por nota ficam no **frontmatter**, sem duplicação.

**Convenção interna por hub.** Áreas com muitas subdivisões podem usar uma família de tag local fora das oficiais — o caso atual é `saude/<subarea>` (notas em `30_AREAS/SAUDE/METRICAS/`), com subáreas como `diario`, `sono`, `treino`, `peso`, `pressao`, `glicose`, `analise`. Funciona como recorte visual; queries quantitativas se apoiam em `tipo` + path scope, não em tag.

---

## 8.5 MOCs

Cada pasta de primeiro nível possui um arquivo-índice `MOC-<nome>.md` na sua raiz, e um `MOC-home.md` global vive na raiz do vault. Os MOCs concentram navegação, queries Dataview/Tasks e seções manuais.

| MOC                                | Papel                                                      |
|------------------------------------|------------------------------------------------------------|
| `MOC-home.md` (raiz)               | Navegação geral entre os hubs de 1º nível                  |
| `00_SISTEMA/MOC-sistema.md`        | Navegação dos meta-recursos (templates, scripts, docs)     |
| `10_CALENDARIO/MOC-calendario.md`  | Atalhos para dailies/weeklies/monthlies/yearlies recentes  |
| `20_PROJETOS/MOC-projetos.md`      | Projetos por contexto e status                             |
| `30_AREAS/MOC-areas.md`            | Hub das áreas contínuas                                    |
| `40_RECURSOS/MOC-recursos.md`      | Biblioteca/Zettelkasten                                    |
| `99_INBOX/MOC-inbox.md`            | Resumo, tarefas de triagem e instruções de fluxo           |

Hubs de área seguem o mesmo padrão `MOC-<nome>.md` na raiz da subpasta. Exemplos vigentes: `30_AREAS/SAUDE/MOC-saude.md` (com sub-MOCs auto-gerados `_DASHBOARD.md` e `_INDICE.md` na subpasta de integração de `METRICAS/`); `30_AREAS/VAULT/MOC-vault.md` (hub de meta-manutenção do vault, com queries para tarefas/decisões/pesquisas internas e listagem das sessões de manutenção em `30_AREAS/VAULT/sessoes/`). A estrutura interna de subpastas fica livre por área — cada uma organiza conforme a natureza do conteúdo (subáreas temáticas em SAUDE; subpastas por tipo de nota em VAULT, incluindo `sessoes/` própria da área).

---

## 9. Manutenção

### Rotina recomendada

- **Diária:** triagem do `99_INBOX/` → mover para destino correto; encerrar o dia com nota de sessão e log.
- **Semanal/Mensal/Anual:** rodar os reviews periódicos via skills locais (`/weekly-review`, `/monthly-review`, `/yearly-review`), que agregam o nível imediatamente abaixo.
- **Sob demanda — área de saúde:** skill local `/analise-saude` gera relatório informativo (não-diagnóstico) de janelas pré-definidas (7d/30d/6m/1y/all) a partir das métricas longitudinais da área. Output em `30_AREAS/SAUDE/METRICAS/ANALISES/`.
- **Sob demanda:** validação de metadados em `20_PROJETOS/` (script de validação roda em dry-run por padrão); arquivamento de projetos concluídos via skill local `/projeto-arquivar`, que move a pasta para `20_PROJETOS/ARQUIVADOS/<categoria>/<YYYY>/<slug>/` e marca `status: arquivado`.
- **Trimestral:** auditoria de convenções; rodar `/doc-update` (propõe edições nesta `ARCHITECTURE.md`) e `/doc-audit` (auditoria de privacidade).

### Papel do Claude Code

- Sugere classificação para itens em `99_INBOX/`.
- Mantém `_PROJETO.md` atualizado a cada sessão de projeto técnico.
- Propõe atualizações nesta `ARCHITECTURE.md` quando detecta mudanças estruturais (regra-gatilho descrita no `CLAUDE.md` raiz do vault).
- **Não move ou exclui notas sem confirmação explícita.** Reclassificações enviam para `99_INBOX/`, não deletam.

### Sincronização com o repositório público

Ciclo padrão para publicar mudanças desta pasta:

1. `/doc-update` — propõe edições em `ARCHITECTURE.md` a partir de mudanças no vault.
2. Aplicar as edições aprovadas, seção a seção.
3. `/doc-audit` — auditoria de privacidade. Bloqueia se houver achados críticos.
4. `/doc-sync` — dry-run do script de sync, varredura final por padrões de risco e proposta de commit message.
5. Execução manual: rodar `sync-publico.sh` sem `--dry-run`, revisar `git diff` no clone do repo público, commitar e dar push.

---

## 10. Decisões registradas

Lista enxuta de decisões arquiteturais. Mudanças significativas geram nova entrada em vez de reescrever o histórico.

| Data       | Decisão                                                                                       | Motivo                                                                              |
|------------|-----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| 2025-12-14 | Adoção de PARA com prefixos `NN_NOME` em maiúsculas                                           | Ordenação previsível e separação visual entre conteúdo de trabalho e meta-pastas    |
| 2026-05-08 | Documentação do vault sob `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/`                             | Manter docs do vault dentro do próprio vault, com camada pública separada           |
| 2026-05-08 | Pasta `PUBLICO/` em repo separado, sincronizada por cópia via `rsync`, sem auto-commit/push   | Defesa em camadas: separação física do clone + revisão humana antes de cada push    |
| 2025-12-14 | `_PROJETO.md` por projeto em vez de uma única lista global                                    | Cada projeto carrega contexto próprio para o agente, com escopo isolado             |
| 2026-04-12 | Metadado `projeto: <slug>` obrigatório em todas as notas dentro de `20_PROJETOS/`             | Garante rastreabilidade e queries determinísticas; vale também para arquivados      |
| 2026-05-06 | Estrutura de arquivados `20_PROJETOS/ARQUIVADOS/<categoria>/<YYYY>/<slug>/`                   | Preserva histórico de projetos concluídos sem poluir a área ativa                   |
| 2025-12-14 | Templates como fonte da verdade do frontmatter; mudanças no padrão começam pelo template      | Evita drift entre convenção e notas existentes                                      |
| 2026-05-03 | Tags hierárquicas em português, minúsculas, sem acento                                        | Compatibilidade entre clientes e consistência visual                                |
| 2025-12-14 | Skills separadas para sessão de vault (`/vault-*`) e sessão de projeto técnico (`/session-*`) | Manutenção do vault e desenvolvimento de software têm contextos distintos           |
| 2026-05-08 | Subagent dedicado de privacidade para conteúdo candidato a publicação                         | Auditor opera em contexto isolado, sem ler notas privadas que o agente principal viu |
| 2026-05-09 | Frontmatter dos relatórios `analise-saude` em PT, alinhado ao padrão do vault                 | Coerência com convenção PT do vault; caminho mais simples para queries futuras       |
| 2026-05-10 | `tipo: saude` uniforme em toda a área + tags granulares `saude/<subarea>` para discriminação  | Reusa enum oficial sem propor expansão; granularidade fica nas tags + path scope     |
| 2026-05-11 | Manutenção e governança do próprio vault tratadas como **área contínua** em `30_AREAS/VAULT/`; campo `area: <slug>` torna-se obrigatório em notas operacionais sob `30_AREAS/...` (simétrico a `projeto: <slug>`) | Trabalho recorrente, sem data de fim; sessões pontuais vivem em `30_AREAS/VAULT/sessoes/`. Frontmatter explícito viabiliza queries determinísticas por área no Dataview/Tasks |
| 2026-05-11 | Sessões de manutenção do vault migradas de `10_CALENDARIO/01_DAILY/sessoes-vault/` para `30_AREAS/VAULT/sessoes/`, com `area: vault` normalizado em todas as sessões existentes | Coerência com o modelo de área contínua: tudo ligado à governança do vault concentra-se no mesmo hub, simétrico a `20_PROJETOS/<slug>/sessoes/` |

---

## 11. Changelog

- **2026-05-08** — primeira versão pública desta `ARCHITECTURE.md`. Substitui o esqueleto inicial pela arquitetura real do vault.
- **2026-05-10** — atualização de §2 (subáreas reais de `30_AREAS/` e `40_RECURSOS/`), §4 (`analise-saude` no enum `tipo`), §6 (notas sobre fluxo dual de saúde e ownership híbrido de skills), §7 (Contribution Graph e subagent `privacy-reviewer`), §8 (formalização da convenção `saude/<subarea>`), §8.5 nova (convenção de MOCs), §9 (skill `/analise-saude`), §10 (decisões 2026-05-09 e 2026-05-10). Auditoria de privacidade: path concreto da subpasta de integração de saúde substituído por placeholder em §6 e §8.5 para reduzir inferência sobre a fonte de dados.
- **2026-05-11** — adição de `30_AREAS/VAULT/` como subárea contínua (§2), `MOC-vault.md` no padrão de hub de área (§8.5) e regra obrigatória de `area: <slug>` em notas operacionais sob `30_AREAS/...` (§4). Nova decisão em §10. Auditoria de privacidade: conteúdo estritamente meta-estrutural — sem dados pessoais ou de terceiros. Convenções `area/<slug>` em tags (§8) e padronização de subpastas em áreas ficaram postergadas por falta de precedente em mais de uma área.
- **2026-05-11** — sessões de manutenção do vault movidas de `10_CALENDARIO/01_DAILY/sessoes-vault/` para `30_AREAS/VAULT/sessoes/` (§3, §8.5) e nova entrada em §10 registrando a mudança. Frontmatter de todas as sessões existentes normalizado com `area: vault`. Auditoria de privacidade: mudança puramente estrutural — sem PII, sem credenciais, sem caminhos sensíveis novos.

---

*Este documento é mantido em conjunto com [`CLAUDE.md`](CLAUDE.md) desta pasta. Mudanças estruturais no vault devem gerar atualizações aqui antes do próximo sync com o repositório público.*
