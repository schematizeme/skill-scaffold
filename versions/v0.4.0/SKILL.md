---
name: schematize-scaffold
metadata:
  version: 0.4.0
description: O blueprint de PROJETO NOVO da casa — o piso do dia 0, na ordem certa. Rege como um sistema nasce: contenção no workspace, archive antes do código, IAM PRIMEIRO como app separada em `auth.<domain>`, escolha de linguagem por fit + ADR dentro do rol sancionado, repos no padrão `<projeto>_<contexto>[_<lang>]` com o `<projeto>_ops` desde o início, observabilidade integrada (OTel + Grafana/Loki/Tempo/Prometheus), CI/CD com os gates que travam o merge, e o overdev como peça inegociável do fluxo. Entrega o checklist de bootstrap fase a fase (`/scaffold-new`) e a verificação do que já existe (`/scaffold-check`), apontando o que falta antes de escrever a primeira linha de domínio. Use SEMPRE que for começar projeto novo, montar a estrutura inicial de um repo/sistema, decidir a ordem de construção, ou revisar se um projeto existente nasceu com o piso — mesmo sem citar "scaffold". Projeto que começa torto custa mais para endireitar do que para nascer certo.
---

# O scaffolder da casa (schematize-scaffold)

Disciplina normativa, **agnóstica de linguagem**, que responde a uma pergunta só: **como um
projeto novo da casa NASCE?** A resposta não é "com um `main` que responde 200" — é **com o piso
inteiro montado no dia 0**. Esta skill é o **blueprint executável** desse nascimento: a estrutura
canônica, o piso de fábrica e o checklist de bootstrap que transformam "pasta vazia" em "sistema
da casa em pé".

É o **blueprint do dia 0** da casa: onde a `schematize-engineering`
descreve *o que* é o piso (arquitetura, IAM, ops, DoD, archive), esta skill descreve *como
instanciá-lo* num projeto do zero — e como **auditar** (`scaffold-check`) se um projeto existente
o tem. O antídoto ao "começa sem IAM/sem ops que a gente coloca depois" — porque **depois é
nunca**, e "nunca" é um sistema inseguro em produção.

**Versão:** skill `schematize-scaffold` v0.4.0. Changelog em `CHANGELOG.md`.

## Comandos (Claude Code)

Digite `/scaffold-help` pra ver todos. Em resumo:

| Comando | O que faz |
|---|---|
| `/scaffold-help` | lista todos os comandos do schematize-scaffold |
| `/scaffold-new` | o passo-a-passo de **criar projeto novo** já com o piso do dia 0: nomeia os repos por contexto, escolhe linguagem (fit + ADR), scaffolda `auth`/`authfront`/`front`/`ops`, monta CI gated, IAM baseline, observabilidade, archive/índice e liga o overdev |
| `/scaffold-check` | roda contra um **projeto existente** e aponta **o que falta do piso** (IAM app-separada? ops/isolamento? CI gated? DoD? archive/índice? overdev?) — emite checklist de saneamento e trava se faltar peça do dia 0 |
| `/scaffold-load` | carrega à força TODO o corpo normativo (estrutura, piso, linguagem, bootstrap, check) e passa a aplicá-lo |
| `/scaffold-claude` | cria ou mescla o `CLAUDE.md` sempre-on do scaffold na raiz do repo |
| `/scaffold-cc` | context compact: gera handoff no archive e roda `/compact` |
| `/scaffold-handoff` | gera o handoff (context.md + checklist.md) sem compactar |

Os comandos ficam em `assets/commands/` e são instalados em `.claude/commands/`.

## Como usar esta skill

1. **Projeto novo? plan-first sempre** (`/scaffold-new`): antes de criar pasta, fixe a
   **topologia** (quais bounded contexts, que repos, qual linguagem por serviço) e o **ADR
   inicial** de linguagem. Scaffoldar sem decidir a fronteira é montar um monólito por acidente.
   Leia `references/estrutura.md` e `references/linguagem.md`.
2. **Monte o piso do dia 0** (`references/piso.md`): IAM app-separada, ops/isolamento,
   observabilidade, CI gated, testes/pentest, DoD, archive/índice, overdev. **Nada é "pra
   depois".** O checklist de bootstrap está em `references/bootstrap.md`.
3. **Projeto existente?** (`/scaffold-check`, `references/check.md`): audite contra o piso, item
   por item, e gere o **checklist de saneamento** do que falta. Pareia com a `schematize-audit`.
4. **Não trabalhe de memória** — a estrutura canônica, o piso e o bootstrap estão nos references.
   A implementação idiomática de cada peça (o auth em Rust, o front em Next, o ops em Go) sai da
   **skill de linguagem** escolhida (`schematize-rust`/`web`/`go`/...). Aplique os pisos abaixo
   independentemente do reference carregado.

Mapa de references — leia o que casa com a tarefa:

| Tarefa | Reference |
|---|---|
| A **estrutura canônica**: repos por contexto (`<projeto>_<ctx>_<lang>`), auth como app separada (`<projeto>_auth_<lang>` + `<projeto>_authfront`) + delegação OIDC, front separado, `<projeto>_ops`, `<projeto>_archive`, contenção no workspace | `references/estrutura.md` |
| O **piso de fábrica** que vem no dia 0: segurança, testes+pentest, IAM baseline 2FA, ops/isolamento por app, observabilidade, CI com deploy gated, DoD §DoD, archive/índice, overdev | `references/piso.md` |
| A **escolha de linguagem** por fit + o **ADR inicial** (rol sancionado Go/Rust/Elixir/C#/Zig/Ruby; front Node) | `references/linguagem.md` |
| O **checklist de bootstrap**: a ordem de criar um projeto novo com o piso, passo a passo, do workspace ao overdev ligado | `references/bootstrap.md` |
| O **scaffold-check**: auditar um projeto existente contra o piso, o que conta como "tem o piso", o checklist de saneamento e o gate | `references/check.md` |
| A **integração com a GUI**: como uma skill vira botão na aba do projeto declarando `gui.json` na raiz — o mecanismo geral (Q.A. e Pentest são instâncias dele) | `references/gui.md` |

## Pisos inegociáveis (vetam o atalho)

Independente do reference, estes limites nunca são cruzados:

1. **O piso é DIA 0 — nada de "sem IAM/sem ops pra depois".** IAM (app separada), ops
   (isolamento por app), CI gated, testes/pentest, observabilidade, DoD, archive e overdev
   **nascem com o projeto**, não entram num sprint futuro. "Depois" é "nunca"; "sem segurança
   pra ir rápido" é dívida disfarçada de produtividade. Projeto sem o piso **não é projeto da
   casa**, é protótipo — e protótipo pede ADR que dê data de virada.
2. **Auth é APP SEPARADA desde o primeiro commit.** Todo projeto nasce com
   `<projeto>_auth_<lang>` (o IdP) **+** `<projeto>_authfront`, com repo/deploy/user/systemd
   próprios, servido em `auth.<domain>`; o app principal **delega por OIDC/OAuth2.1 + PKCE**.
   **VETADO** apensar auth ao escopo principal "por enquanto". Comprometer o app principal não
   pode comprometer o IdP.
3. **Um repo = um bounded context, dentro do workspace.** Repos no padrão
   `<projeto>_<ctx>_<lang>`, **irmãos dentro da pasta do projeto** (nunca soltos no root, nunca
   `cd ..`). Front separado do backend. Nada de monólito que acopla contextos sem ADR de plano e
   **data** de quebra.
4. **`<projeto>_ops` nasce junto — control plane, não afterthought.** O sistema tem um
   `<projeto>_ops` desde o dia 0: bootstrap/instalação paralela (`nproc`), deploy destrutivo por
   seed (`/<app>/.env`), isolamento por app (user Linux + systemd hardened), fluxo de promoção
   (dev→teste→git→hml→prd). **100%** das operações passam por ele; nada de mão no servidor.
5. **Escolha de linguagem sai do ROL por fit + ADR inicial.** Backend novo em Go/Rust/Elixir/
   C#/Zig/Ruby, escolhido por **encaixe com o problema** e **registrado num ADR** no dia 0 —
   nunca por gosto. Front é Node (Next/Astro). O **piso é o mesmo em toda linguagem**; ela muda o
   "como", não o "o quê".
6. **Archive e índice desde o primeiro commit.** `<projeto>_archive/` existe no dia 0; toda
   entrega gera o `.md` de archive (§28), o `MAPA.md`/índice (§39) mora no archive, o **root fica
   limpo** (código, config, README/CLAUDE/LICENSE). Sem archive, a entrega não aconteceu.
7. **Overdev ligado e DoD como gate.** O projeto nasce com o overdev disponível (`/eng-overdev`)
   e a **Definition of Done (§35)** como gate de entrega. O scaffold não fecha "quase pronto": o
   bootstrap só termina quando o piso **inteiro** está de pé, provado (não auto-declarado).

8. **Projeto novo NASCE com o efeito externo em SINK — default de fábrica, não lembrete.** É
   aqui que o piso "efeito externo nunca sai de não-produção" (`schematize-engineering` →
   `references/efeitos-externos.md`) vira **default**: o `.env.example`/seed gerado já traz
   **`APP_ENV`**, **`MAIL_PROVIDER=sink`**, **`TEST_MAIL_DOMAIN=test.<domain>`** e
   **`MAIL_MAX_PER_RUN=50`**; o **`docker-compose` de dev já sobe o Mailpit** (UI/API `:8025` pro
   teste **ler** a caixa, SMTP `:1025`); o **guard mora dentro do provider** (destinatário fora do
   domínio de teste com `APP_ENV != prd` → **erro**, fail-closed) e nasce com o **teste que vê a
   recusa**; fixture/seed/factory/persona só emitem `<papel>+<run-id>-<n>@test.<domain>`. **VETADO**
   scaffoldar com provedor real ligado por default fora de `prd`, com chave de prd no `.env` de
   não-prd, sem cap por execução ou com `@gmail.com`/caixa real em template. O `/scaffold-check`
   confere esse piso (sink default, guard, cap, domínio de teste) como **9ª peça**, com veto duro
   em projeto que envia. **Por quê:** um laço disparou **>5.000 e-mails reais**, queimou reputação
   de IP/domínio e derrubaria o **OTP de login** de produção — projeto que nasce com o provider real
   por default já nasce com a bomba armada. Detalhe: `references/piso.md` §9, `references/check.md`.

## Relação com as outras skills

- **schematize-engineering** — a **BASE** que este scaffold materializa. Onde a engenharia
  descreve o piso (**arquitetura §2** repos/bounded context/ops, **IAM** app separada, **ops.md**
  isolamento/seed/promoção, **DoD §35**, **archive §28**, **índice/MAPA §39**, **overdev**), esta
  skill dá o **passo-a-passo de instanciá-lo do zero**.

> **Sobre o `schematize new`:** ✔ verificado em 2026-08-21 — **esse subcomando não existe** no binário `schematize` (as fronteiras do CLI hoje são `skills`, `overdev`, `archive`, `doctor`, `graph`, `db`, …). A skill dizia dar "lastro" a um comando inexistente, o que é promessa sem entrega. Se algum dia ele existir, o lastro é este documento; enquanto não existe, **o blueprint é executado por `/scaffold-new`** (aqui) e auditado por `/scaffold-check`. **Fronteira:** o CLI vive noutro projeto (`schematize_app`), fora do escopo deste catálogo.
- **schematize-audit** — o **par de auditoria**. O `scaffold-check` verifica se o **piso do dia 0**
  está lá (IAM/ops/CI/DoD/archive); a `schematize-audit` verifica se **os checklists criados foram
  sanados**. O scaffold monta o piso; o audit prova que o que foi prometido depois foi entregue.
- **schematize-go / rust / elixir / csharp / zig / ruby** — a **implementação idiomática** de cada
  peça scaffoldada. O scaffold decide *que* o auth é app separada em Rust; a `schematize-rust`
  diz *como* escrever esse auth. Cada uma traz seu `/<prefixo>-iam` e `/<prefixo>-ops` que scaffoldam a *(o prefixo NÃO se deriva do nome da skill — a tabela de `references/linguagem.md` tem a coluna; em C# é `/cs-iam`.)*
  peça na linguagem.
- **schematize-web** — o **front** (o do produto **e** o `<projeto>_authfront`): fronteira
  client/server, segredo só no servidor, a11y, CWV, i18n.
- **schematize-pentest** — o **oráculo de segurança**: o piso de IAM/authz que o scaffold monta é
  o que a pentest ataca rota por rota (rejeição, cross-tenant, IDOR/BOLA) pra provar que segura.

- **schematize-qa** — o **teste é peça do dia 0**, não coisa da fase 3. O scaffold monta o test kit
  antes da primeira feature: a pirâmide, o **smoke com self-check** (que prova que o runner sabe
  reportar FAIL — suíte que nunca falhou está cega), a matriz `simulated` de rota × persona, e os
  **gates de CI que travam o merge** desde o primeiro commit. Quem é dono do COMO é a
  `schematize-qa`; o scaffold só garante que a peça exista antes de haver dívida. Projeto que
  nasce sem gate de teste não ganha gate depois — ganha exceção.
- **schematize-archive** — a **planta do archive é dela**, não desta skill. A Fase 1 do bootstrap
  cria o `<projeto>_archive/` **rodando `/archive-init`**, com a árvore canônica da
  `schematize-archive` (ADR-0005) — inclusive o `git init` e o remote **privado**, que fazem parte
  do piso de criticidade 0. O scaffold decide *quando* (dia 0, antes do código); a
  `schematize-archive` decide *o quê* e *onde*.
