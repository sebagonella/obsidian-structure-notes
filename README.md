# Vault Architecture — Public Documentation

> Documentação pública da arquitetura de um vault Obsidian pessoal,
> mantido como sistema de pensamento e referência para projetos pessoais e profissionais.

Este repositório contém **apenas a estrutura, convenções e fluxos** do vault — nunca o conteúdo das notas, nomes de pessoas, clientes ou projetos internos.

---

## Por que existe

Um vault de Obsidian usado a sério vira, com o tempo, um sistema com regras próprias: convenções de nomeação, taxonomia, automações, integrações. Manter essas decisões documentadas em separado serve a três propósitos:

1. **Memória institucional pessoal.** Lembrar daqui a um ano *por que* cada decisão foi tomada.
2. **Contexto persistente para agentes.** O Claude Code lê estes arquivos a cada sessão e opera dentro das convenções estabelecidas, em vez de improvisar.
3. **Compartilhamento honesto.** Permite mostrar o sistema a outras pessoas (ou empregadores, recrutadores, comunidade) sem expor conteúdo privado.

---

## O que tem aqui

| Arquivo            | Conteúdo                                                                |
|--------------------|-------------------------------------------------------------------------|
| `ARCHITECTURE.md`  | Visão geral: taxonomia, convenções, fluxos de captura, integrações     |
| `CLAUDE.md`        | Regras de privacidade aplicadas a esta pasta — usadas pelo Claude Code |
| `README.md`        | Este arquivo                                                            |

---

## O que **não** tem aqui

- Notas reais ou trechos delas
- Nomes de pessoas, empresas, clientes, projetos internos
- Endpoints, webhooks, tokens, chaves ou credenciais
- Capturas de tela do vault em uso

A separação não é casual: a pasta `PUBLICO/` vive dentro de um vault maior cuja maior parte é privada por design. As regras em `CLAUDE.md` deste diretório existem justamente para que essa fronteira não seja cruzada por engano.

---

## Como é mantido

A documentação é mantida em conjunto com o vault, usando [Claude Code](https://www.anthropic.com/claude-code). Três slash commands automatizam o ciclo:

- `/doc-update` — detecta mudanças arquiteturais no vault e propõe edições aqui
- `/doc-audit` — auditoria de privacidade antes de qualquer push
- `/doc-sync` — dry-run do script de sync, varredura final por padrões de risco e proposta de commit message

Um subagent dedicado (`privacy-reviewer`) opera em contexto isolado para auditar conteúdo candidato a publicação, reduzindo o risco de vazamento por inferência.

A regra-gatilho declarada no `CLAUDE.md` raiz do vault é: **toda mudança estrutural propaga para `ARCHITECTURE.md`**, antes de qualquer sync.

---

## Para quem chega de fora

Se você está montando um sistema de notas próprio, esta documentação pode servir como referência de:

- Como adaptar PARA com prefixos numéricos para ordenação determinística
- Como estruturar captura multi-canal (manual, web clipper, pesquisa externa, importação de legados) com um inbox único
- Como dividir documentação técnica entre uma camada pública e uma privada no mesmo vault
- Como instrumentar um vault para trabalhar em conjunto com um agente de IA sem perder controle sobre privacidade

---

## Licença

A documentação neste repositório está sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — pode ser usada, adaptada e redistribuída desde que com atribuição.

> Esta licença cobre **apenas** os arquivos deste repositório público. O vault em si, suas notas e qualquer conteúdo derivado permanecem privados e fora do escopo desta licença.
