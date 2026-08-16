# Changelog — schematize-scaffold

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versionamento semântico.

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
