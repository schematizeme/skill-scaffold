# Changelog — schematize-scaffold

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versionamento semântico.

## [0.2.0] — 2026-08-20

Propagação do piso **"efeito externo NUNCA sai de não-produção"** no recorte de **SCAFFOLD** — o
ponto em que o piso deixa de depender de alguém lembrar e vira **default de fábrica**: projeto
novo **nasce** com o envio em sink. Normativa: `schematize-engineering` →
`references/efeitos-externos.md` (origem: um laço de teste disparou **>5.000 e-mails reais** e
queimou a reputação de IP/domínio, derrubando o **OTP de login** de produção).

### Adicionado
- **SKILL.md** — piso inegociável **8** ("Projeto novo NASCE com o efeito externo em SINK"):
  `.env.example`/seed com `APP_ENV` + `MAIL_PROVIDER=sink` + `TEST_MAIL_DOMAIN` + `MAIL_MAX_PER_RUN`,
  Mailpit no compose, guard dentro do provider com teste da recusa, fixtures só no domínio de
  teste; menção ao piso na `description` do frontmatter.
- **references/piso.md** §9 — "Efeito externo: SINK por default — o piso vira default de FÁBRICA":
  o **`.env.example`/seed gerado** (com o bloco pronto e o motivo de cada chave), o
  **`docker-compose` de dev com Mailpit** pinado por digest (`:8025` UI/API pro teste **ler** a
  caixa, `:1025` SMTP), o guard dentro do provider + o teste que vê o vermelho, fixtures na forma
  `<papel>+<run-id>-<n>@test.<domain>` (com a lista de vetados), o gate
  `check-external-effects.sh` no CI do dia 0 e o item de **DNS em rota nula** aberto para a
  `schematize-infra`.
- **references/bootstrap.md** — passo **18** da Fase 5 (sink de fábrica) e **2 linhas novas no
  checklist de bootstrap** (config+compose+guard+fixtures+gate; domínio de teste provado por `dig`).
- **references/check.md** — **9ª peça** do check com prova de presença e bandeiras vermelhas
  (provedor real por default fora de prd, chave de prd em não-prd, envio sem cap, caixa real em
  fixture, guard no chamador), **3 itens novos** no checklist de saneamento e **veto duro** no gate
  para projeto que envia.
- **assets/CLAUDE.md** — piso sempre-on **8** com as chaves de fábrica, o Mailpit e os vetos.

### Mudado
- **references/piso.md** — a tabela-resumo do piso passa de §9 para **§10** e ganha a **9ª peça**
  ("Efeito externo (sink de fábrica)"); referências em `check.md` e `assets/CLAUDE.md` atualizadas
  de "8 peças" para "9 peças".

## [0.1.0] — 2026-08-15

Primeira versão da skill de **scaffold** da casa — o blueprint executável de "projeto novo da
casa", lastro do futuro comando `schematize new <projeto>`. Materializa o piso descrito na
`schematize-engineering` (arquitetura §2, IAM, ops, DoD §35, archive §28, índice §39, overdev) num
passo-a-passo de criar projeto do zero, e traz o `scaffold-check` que audita esse piso num projeto
existente (par da `schematize-audit`).

### Adicionado
- **SKILL.md** com 7 pisos inegociáveis (o piso é dia 0; auth app separada desde o 1º commit; um
  repo = um bounded context no workspace; `<projeto>_ops` desde o dia 0; linguagem por fit + ADR
  inicial; archive/índice desde o 1º commit; overdev + DoD como gate) + mapa de references +
  relação com engineering/audit/linguagem/web/pentest.
- **references/**:
  - `estrutura.md` — a estrutura canônica: workspace/contenção, repos por contexto
    (`<projeto>_<ctx>_<lang>`), **auth como app separada** (`<projeto>_auth_<lang>` +
    `<projeto>_authfront`) + delegação OIDC, front separado, `<projeto>_ops`, `<projeto>_archive`,
    independência de runtime, root limpo.
  - `piso.md` — o piso de fábrica do dia 0: segurança, testes+pentest, IAM baseline 2FA,
    ops/isolamento por app + CI gated, observabilidade, DoD (§35), archive/índice (§39), overdev;
    tabela-resumo das 8 peças que o check procura.
  - `linguagem.md` — rol sancionado (Go/Rust/Elixir/C#/Zig/Ruby; front Node), guia de fit, o **ADR
    inicial** obrigatório por serviço, e "o piso não muda com a linguagem".
  - `bootstrap.md` — o passo-a-passo (Fase 0 decidir → archive → auth primeiro → contextos+front →
    ops → CI/DoD/observabilidade → overdev) com o checklist de bootstrap derivável pro overdev.
  - `check.md` — o scaffold-check: as 8 peças com prova de presença, como conduzir (enumerar →
    provar → classificar), o checklist de saneamento, o gate (IAM/segurança = prioridade 0) e
    onde gravar.
- **assets/commands/**: `/scaffold-help`, `/scaffold-new` (criar projeto novo), `/scaffold-check`
  (auditar o piso), `/scaffold-load`, `/scaffold-claude`, `/scaffold-cc`, `/scaffold-handoff`.
- **assets/CLAUDE.md** — regra sempre-on: o piso é dia 0; auth app separada (OIDC); um repo = um
  bounded context; ops/archive/DoD desde o começo.
