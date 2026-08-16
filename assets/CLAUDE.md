# CLAUDE.md — Projeto Novo da Casa / Scaffold (sempre on)

> Copie para a **raiz do repositório** e ajuste `<project>`. Fica pinado no contexto de toda
> tarefa e garante o piso mesmo quando a skill `schematize-scaffold` não dispara sozinha. Em repo
> multi-skill, use **junto** com os `CLAUDE.md` das skills de engenharia (rode `/scaffold-claude`
> que mescla, sem sobrescrever os outros blocos).

## Regra mestre

Um projeto da casa **nasce com o piso inteiro no dia 0** — não com um `main` que responde 200 e
"segurança/ops depois". A skill `schematize-scaffold` rege *como* um projeto novo é instanciado
(estrutura canônica + piso de fábrica) e como se **audita** (`/scaffold-check`) se um projeto
existente o tem. Em conflito entre "começa simples, coloca depois" e este piso, **o piso vence**.
Consulte o reference antes de agir — não trabalhe de memória.

## Pisos inegociáveis (VETADO — sem exceção)

1. **O piso é DIA 0, não sprint futuro.** IAM (app separada), ops (isolamento por app), CI gated,
   testes+pentest, observabilidade, DoD, archive/índice e overdev **nascem com o projeto**.
   "Depois" é "nunca"; "sem segurança pra ir rápido" é dívida disfarçada. Projeto sem o piso é
   protótipo — e protótipo exige **ADR com data de virada**, não um encolher de ombros.
2. **Auth é APP SEPARADA desde o primeiro commit.** `<project>_auth_<lang>` (IdP) **+**
   `<project>_authfront`, repo/deploy/user/systemd próprios, em `auth.<domain>`; o app principal
   **delega por OIDC/OAuth2.1 + PKCE**. VETADO apensar login ao escopo principal "por enquanto".
   Baseline: ID≠email, ≥2 fatores (senha + Email OTP já conta; passkey núcleo), ReBAC
   multi-tenant, deny-by-default, chave só no auth + JWKS. **IAM é prioridade 0.**
3. **Um repo = um bounded context, dentro do workspace.** Padrão `<project>_<ctx>_<lang>`, repos
   **irmãos dentro da pasta do projeto** (nunca soltos no root, nunca `cd ..`). Front separado do
   backend. Nada de monólito que acopla contextos sem ADR de plano **e data** de quebra.
4. **`<project>_ops` desde o dia 0 — control plane.** Bootstrap + instalação paralela (`nproc`),
   deploy destrutivo por seed (`/<app>/.env`), isolamento por app (user Linux + systemd hardened),
   fluxo de promoção (dev→teste→git→hml→prd). **100%** das operações passam por ele; nada de mão
   no servidor.
5. **Linguagem por fit + ADR inicial.** Backend novo do rol (Go/Rust/Elixir/C#/Zig/Ruby),
   escolhido por encaixe e **registrado em ADR** no dia 0. Front é Node (Next/Astro). O piso é o
   mesmo em toda linguagem — ela muda o "como", não o "o quê".
6. **Archive e índice desde o primeiro commit.** `<project>_archive/` existe no dia 0; toda
   entrega gera o `.md` de archive; `MAPA.md`/índice (§39) mora no archive; **root limpo**.
7. **Overdev + DoD como gate.** O projeto nasce com `/eng-overdev` disponível e a DoD (§35) como
   gate de entrega. O bootstrap só fecha quando o piso **inteiro** está de pé, **provado** (não
   auto-declarado).

## Como se faz aqui

- **Projeto novo (`/scaffold-new`):** decide topologia + linguagem/ADR (Fase 0), cria o archive,
  scaffolda o **auth primeiro** (app separada + OIDC), depois contextos + front, o `ops`, amarra
  CI gated + DoD + observabilidade, liga o overdev e fecha com `/scaffold-check` verde.
- **Projeto existente (`/scaffold-check`):** audita as 8 peças do piso **com prova de hoje**,
  aponta `missing`/`partial` com origem, gera o **checklist de saneamento** (candidato a
  `/eng-overdev`) e **trava** se faltar peça do dia 0 (IAM/segurança/ops/archive). Relatório em
  `<project>_archive/scaffold/`.

## Relação com as outras skills

- **schematize-engineering** — a base que este scaffold materializa (arquitetura §2, IAM, ops.md,
  DoD §35, archive §28, índice §39, overdev). O scaffold dá o passo-a-passo de instanciá-la.
- **schematize-audit** — o par: o `/scaffold-check` audita o **piso**; o audit audita se os
  **checklists criados** foram sanados.
- **schematize-<lang>** (go/rust/elixir/csharp/zig/ruby) — a implementação idiomática de cada peça
  (`/<slug>-iam`, `/<slug>-ops`). **schematize-web** — os fronts. **schematize-pentest** — o
  oráculo que ataca o IAM/authz montado.

## Gestão de contexto (sessões longas)

Ao se aproximar do teto de contexto: **PARE e** gere o handoff em `<project>_archive/context/`
(topologia + peças do piso FEITO vs EM ABERTO) **antes** de compactar (`/scaffold-cc`).
