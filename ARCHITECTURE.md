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
├── 40_RECURSOS/                           # referências, biblioteca, material durável
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

- **Sessão de manutenção do vault:** `10_CALENDARIO/01_DAILY/sessoes-vault/YYYY-MM-DD-slug.md`.
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
      decisao | pesquisa | tarefa | saude | legado | webclipper |
      moc | documentacao
status: ideia | planejamento | execucao | revisar | concluido | arquivado
tags: []
---
```

**Campos por tipo de nota:**

- **Daily:** acrescenta `dia_semana`.
- **Sessão de projeto:** acrescenta `hora`, `session_id`, `branch`, `mensagens`, `projeto: <slug>`.
- **Notas dentro de `20_PROJETOS/...`:** **obrigatoriamente** `projeto: <slug>` no frontmatter. Slug = nome da pasta mais profunda do projeto. Tarefas inline (checkbox) usam as tags `#projeto/<slug>` + `#tipo/tarefa`.
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

---

## 7. Integrações externas

| Componente                                | Papel                                                                  |
|-------------------------------------------|------------------------------------------------------------------------|
| Obsidian + Local REST API (plugin)        | Editor primário e endpoint local consumido pelo agente                 |
| Obsidian — Templater                      | Engine de templates                                                    |
| Obsidian — Dataview                       | Queries em MOCs e listas dinâmicas                                     |
| Obsidian — Tasks                          | Indexação de tarefas inline                                            |
| Obsidian — Tracker                        | Gráficos longitudinais a partir de frontmatter                         |
| Obsidian — Linter                         | Mantém `data_atualizacao` consistente                                  |
| Obsidian — Smart Connections              | Indexação semântica local (estado fora de versão)                      |
| Google Drive                              | Sync entre dispositivos do vault                                       |
| Claude Code (CLI) + MCP Obsidian          | Manutenção do vault e desta documentação via REST API local            |
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

---

## 9. Manutenção

### Rotina recomendada

- **Diária:** triagem do `99_INBOX/` → mover para destino correto; encerrar o dia com nota de sessão e log.
- **Semanal/Mensal/Anual:** rodar os reviews periódicos via skills locais (`/weekly-review`, `/monthly-review`, `/yearly-review`), que agregam o nível imediatamente abaixo.
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

---

## 11. Changelog

- **2026-05-08** — primeira versão pública desta `ARCHITECTURE.md`. Substitui o esqueleto inicial pela arquitetura real do vault.

---

*Este documento é mantido em conjunto com [`CLAUDE.md`](CLAUDE.md) desta pasta. Mudanças estruturais no vault devem gerar atualizações aqui antes do próximo sync com o repositório público.*
