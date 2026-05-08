# CLAUDE.md — Vault Obsidian (raiz)

Este vault é o "cérebro" pessoal e profissional do usuário. Contém notas, projetos, referências e contextos sensíveis. Claude Code opera neste vault com **dois modos de trabalho** que precisam ser distinguidos a cada interação:

1. **Modo privado** (default): qualquer caminho exceto `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/`. Claude tem acesso total aos conteúdos reais.
2. **Modo público**: ao tocar em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/`, valem as regras adicionais de `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/CLAUDE.md`. Sempre leia esse arquivo antes de editar lá.

Por brevidade, o restante deste documento usa o alias `<PUBLICO>` para `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/`. **No conteúdo escrito (não em referências de comando), sempre expandir para o caminho completo.**

---

## Estrutura geral

A taxonomia real do vault, com pastas de primeiro/segundo nível e convenções de nomeação, está descrita em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/ARCHITECTURE.md` (§2 e §3). Recorte mínimo relevante para esta pasta:

```
vault/
└── 00_SISTEMA/
    ├── 00_DOCUMENTACOES/
    │   └── 01_VAULT/
    │       ├── PUBLICO/        # esta pasta — sincronizada com repo público no GitHub
    │       │   ├── README.md
    │       │   ├── ARCHITECTURE.md
    │       │   └── CLAUDE.md
    │       └── PRIVADO/        # documentação interna; nunca sai do vault
    │           └── PROMPTS/    # biblioteca de prompts (ex.: bootstrap-publico.md)
    └── 02_SCRIPTS/
        └── sync-publico.sh     # PUBLICO/ → clone local do repo público
```

Para o restante (`10_CALENDARIO/`, `20_PROJETOS/`, `30_AREAS/`, `40_RECURSOS/`, `99_INBOX/`, `.claude/`), consultar `ARCHITECTURE.md` em vez de duplicar aqui — esta seção fica em sincronia automática.

---

## Princípios de operação

- **Não inventar estrutura.** Antes de criar pastas/arquivos novos, varrer o vault e propor onde encaixar.
- **Frontmatter consistente.** Toda nota gerada/modificada por Claude preserva ou propõe frontmatter YAML (created, tags, type, status quando aplicável).
- **Templates como fonte da verdade.** Mudanças em fluxos de captura propagam para `templates/`.

---

## Regra-gatilho: arquitetura ↔ documentação pública

**Sempre que houver mudança em qualquer dos itens abaixo, propor (não aplicar automaticamente) edição correspondente em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/ARCHITECTURE.md`:**

- Taxonomia de pastas de primeiro e segundo nível
- Convenções de nomeação de arquivos ou frontmatter
- Tipos de templates e o papel de cada um
- Integrações externas (serviços, automações, APIs)
- Convenções de tags ou MOCs

Antes de editar `ARCHITECTURE.md`, executar a auditoria de privacidade descrita em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/CLAUDE.md`.

---

## Limites de privacidade (válidos em todo o vault)

Mesmo no modo privado, Claude **nunca**:

- Resume ou cita conteúdo de notas privadas em respostas que serão claramente coladas em contextos públicos (ex.: README, issues do GitHub público, posts).
- Sugere subir o vault inteiro para serviço externo sem confirmação explícita.
- Inclui em commits qualquer arquivo fora de `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PUBLICO/` quando a operação alvo for o repo público.

---

## Setup esperado

O sync para o repo público depende de duas variáveis de ambiente fora do vault:

- `VAULT_ROOT` — caminho absoluto da raiz deste vault
- `PUBLIC_REPO` — caminho absoluto de um clone local do repositório público (que vive **fora** do vault e fora do Google Drive)

Sem essas variáveis, `/doc-sync` aborta na pré-condição. O script real é `00_SISTEMA/02_SCRIPTS/sync-publico.sh` (rsync com dry-run obrigatório, sem auto-commit/push).

---

## Workflow recomendado

- **Mudanças estruturais** → atualizar primeiro o vault, depois rodar `/doc-update` para propor edições em `ARCHITECTURE.md`.
- **Antes de qualquer push para o repo público** → rodar `/doc-audit` e depois `/doc-sync`. Só executar o sync real (sem `--dry-run`) após revisar o dry-run.
- **Dúvida sobre o que pode virar público** → assumir privado e perguntar.
- **Bootstrap inicial de PUBLICO/** → usar o prompt em `00_SISTEMA/00_DOCUMENTACOES/01_VAULT/PRIVADO/PROMPTS/bootstrap-publico.md`.
