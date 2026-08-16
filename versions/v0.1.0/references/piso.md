# O piso de fábrica — o que vem montado no dia 0

> "Projeto da casa" quer dizer **projeto que já nasce com o piso inteiro**. Este reference lista,
> peça por peça, o que o `/scaffold-new` monta antes de escrever a primeira feature — e o que o
> `/scaffold-check` procura num projeto existente. Nada aqui é "pra depois". A base normativa de
> cada item está nas references da `schematize-engineering`.

## 0. A regra que rege tudo: dia 0, não sprint futuro

O anti-padrão que esta skill mata é **"começa simples e a gente coloca segurança/ops/testes
depois"**. Depois é nunca. O piso não é um marco no roadmap — é a **fundação**; a feature vem
**em cima** dele. Um projeto sem o piso não é um projeto da casa: é um protótipo, e protótipo
exige **ADR com data de virada** (§27), não um encolher de ombros.

## 1. Segurança (piso comum, inegociável)

Vem montado, valha qual linguagem for:

- **Segredo nunca no cliente** (nem em `NEXT_PUBLIC_`/`VITE_`/`PUBLIC_`, nem no bundle, nem no
  repo). Segredo real via secret manager referenciado pelo seed do ops.
- **Consulta sempre parametrizada** — sem SQL/query por concatenação/interpolação.
- **Auth decidida no servidor** (deny-by-default); token em cookie HttpOnly, nunca localStorage.
- **Validação de input** na fronteira; sem erro engolido; sem coerção silenciosa de tipo.
- Detalhe em `schematize-engineering/references/seguranca.md` e `anti-padroes.md` (§37).

## 2. Testes de verdade + pentest de rejeição

- **Test kit** que roda pelo ops, saída machine-readable, categorias (unit/componente/e2e/
  integração). "Verde de verdade" = a suíte roda **hoje**, sem `.skip`/assert comentado.
- **Pentest prova rejeição, rota por rota, campo por campo:** nunca 500 por input hostil, nunca
  coerção de tipo, nunca eco sem escape, nunca vazamento cross-tenant. Base em
  `schematize-engineering/references/testes.md` + `testes-execucao.md` (§22); arsenal na
  `schematize-pentest`.
- O scaffold cria a **estrutura de testes** e o alvo de CI no dia 0 — não um `TODO: add tests`.

## 3. IAM baseline 2FA (app separada — o coração do piso)

O auth (`<projeto>_auth_<lang>` + `<projeto>_authfront`, `references/estrutura.md` §4) nasce com o
**baseline** da casa:

- **Identidade ≠ autorização.** `sub` = **ID interno imutável e opaco** (ULID/UUIDv7). **Email e
  telefone NUNCA são ID** — são identificadores, cada um com estado de verificação; **1..N por
  usuário** (ter mais de um email é incentivado).
- **≥ 2 fatores por desenho, desde o cadastro.** Senha (**argon2id** + checagem contra vazadas/
  HIBP) **+ Email OTP** (Resend, always-on) **já é 2FA baseline válido**. Fator forte é
  incentivado just-in-time (step-up), **nunca muro pré-login**.
- **Passkey/WebAuthn é núcleo** (phishing-resistant, "2 fatores num"), não roadmap. Twilio pra
  SMS/voz; provedores plugáveis (`EmailProvider`/`SmsProvider`/`PushProvider`).
- **Recuperação tão forte quanto o login** (nada de reset por 1 email que passa por cima do 2FA);
  **SSO nunca é ponto único de falha** (força ≥1 fator de recuperação local); account-linking
  explícito e anti-takeover.
- **ReBAC multi-tenant granular, deny-by-default, enforcement sempre no servidor.** Sessão longa
  com logout irreversível.
- Delegação **OIDC/OAuth2.1 + PKCE**; chave só no auth, validação por **JWKS**.
- Base completa em `schematize-engineering/references/iam.md`. O `/<slug>-iam` da skill de
  linguagem scaffolda a implementação.

## 4. Ops / isolamento por app + CI com deploy gated

O `<projeto>_ops` (control plane, `references/estrutura.md` §6) nasce com os invariantes:

- **Fluxo de promoção fixo, sem atalho:** `dev local → teste local (verde) → GitHub → hml → prd`.
  Nada pula etapa; **VETADO editar código direto no servidor**; hotfix segue o mesmo fluxo
  acelerado. Servidor é **imutável por edição manual** — só recebe artefato com **proveniência
  git** (commit SHA).
- **Deploy destrutivo por seed:** o ops provisiona em `/<app>/` clonando os repos dentro;
  `/<app>/.env` é o **seeder global** (fonte única de config); todo redeploy **apaga a
  implantação anterior e recria um clone zerado** só com o seed (idempotente, sem drift) —
  **preservando os dados** (migration reversível com `down`; `ops reset` de dados só em dev/hml).
- **Isolamento por app:** **um user Linux dedicado por serviço** (nunca dois no mesmo, nunca
  `root`) + **um systemd unit hardened** (`NoNewPrivileges`, `ProtectSystem=strict`,
  `ProtectHome`, `PrivateTmp`, `ReadWritePaths` mínimo, `CapabilityBoundingSet` mínimo). Blast
  radius contido: comprometer um serviço não vaza pros outros nem pro host.
- **100% das operações passam pelo ops** (`bootstrap`/`install`/`redeploy`/`update`/`config`/
  `migrate`/`health`/`rollback`/`logs`/`reset`/`test`), idempotente, com `--help`. Nada de
  `ssh` + comando ad-hoc.
- **Instalação SEMPRE paralela** (grau = `nproc`); falha no paralelo = **bug de independência**
  (prioridade máxima), corrigido de verdade — **nunca** mascarado serializando.
- **CI com deploy gated:** o pipeline roda o test kit + gate da DoD e **só promove o que passou**;
  a promoção pra hml/prd é gated pelo fluxo acima. Base em
  `schematize-engineering/references/ops.md` + `operacao.md` (§21).

## 5. Observabilidade integrada

Todo serviço — e o próprio ops — sobe com **observabilidade integrada** (logs estruturados,
métricas, tracing; stack tipo Grafana/LGTM+). Endpoint de saúde responde; dashboard/alerta
versionado como código. Não é "instrumenta depois de dar problema"; nasce instrumentado. Base em
`schematize-engineering/references/observabilidade.md` (§16).

## 6. Definition of Done (§DoD) como gate

Toda entrega passa pela **Definition of Done (§35)**: testes verdes, sem anti-padrão (§37),
archive gerado, índice atualizado, gate de review (`/<slug>-review`) passa. O scaffold instala a
DoD como **gate de PR/entrega** no dia 0 — não fecha "quase pronto". Pisos de código junto:
arquivos **≤ 750 linhas** (~500 úteis + ~250 comentário; flag > 300 úteis), **uma unidade lógica
por arquivo**, **toda função com doc-comment**. Base em
`schematize-engineering/references/entrega.md` (§35) + `padroes-codigo.md`.

## 7. Archive / índice (MAPA §39)

`<projeto>_archive/` existe **no primeiro commit**. Toda entrega gera o `.md` de archive (§28); o
`MAPA.md` + índice de microfunções (§39) mora no archive (`/eng-index` regenera dos doc-comments);
o **root fica limpo**. Sem archive, a entrega não aconteceu. Base em
`schematize-engineering/references/operacao.md` (§28, §39).

## 8. Overdev ligado

O projeto nasce com o **overdev disponível** (`/eng-overdev`): Fase 0 (decisões→grafo→plano
pesado→checklist) e o laço que não deixa parar com item aberto. O bootstrap é, ele mesmo, um bom
primeiro alvo de overdev — o piso inteiro vira um checklist exaustivo que só fecha **provado**.
Base em `schematize-engineering/references/overdev.md`.

## 9. Tabela-resumo do piso (o que o check procura)

| # | Peça do piso | Presença mínima no dia 0 |
|---|---|---|
| 1 | Segurança | segredo fora do cliente, query parametrizada, auth server-side, validação na borda |
| 2 | Testes + pentest | test kit rodando + estrutura de pentest de rejeição |
| 3 | IAM baseline 2FA | `<projeto>_auth_<lang>` + `authfront` separados, ID≠email, senha+OTP, passkey, ReBAC, OIDC/JWKS |
| 4 | Ops / isolamento / CI gated | `<projeto>_ops` com promoção, seed, user+systemd por app, `nproc`, CI que só promove verde |
| 5 | Observabilidade | logs/métricas/tracing + health endpoint desde o boot |
| 6 | DoD (§DoD) | gate de entrega (review + testes + archive) ligado |
| 7 | Archive/índice | `<projeto>_archive/` + MAPA/índice (§39), root limpo |
| 8 | Overdev | `/eng-overdev` disponível; bootstrap como checklist |
