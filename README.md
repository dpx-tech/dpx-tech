# Metodologia de Desenvolvimento — DPX

> Base da metodologia de desenvolvimento da equipe de tecnologia da DPX: como
> trabalhamos, quais padrões aplicamos e por quê. Serve tanto para apps de uso
> **interno** (ex.: ERP, GDX, etc) quanto para **produtos externos**
> que vão para produção, com usuários e exposição na internet.

**Versão 1.0 · Última atualização: 22/06/2026**

### Como ler este documento

Cada item é marcado conforme onde se aplica:

- **[BASE]** : vale para **todo** projeto, sempre, incluindo apps internos.
- **[EXTERNO]** : obrigatório quando o projeto terá **usuários externos ou
  exposição na internet**. Em ferramenta interna costuma ser opcional ou
  dispensável.

Onde faz sentido, cada item responde quatro perguntas: o que é, para que serve,
quando usar e quando não usar. Sempre que citamos um padrão ou ferramenta com
documentação própria, deixamos um link de referência para aprofundamento.

---

## Índice

1. [Apresentação](#1-apresentação)
2. [Base da metodologia](#2-base-da-metodologia)
3. [Arquitetura e boas práticas](#3-arquitetura-e-boas-práticas)
4. [Segurança](#4-segurança)
5. [Documentação e organização do repositório](#5-documentação-e-organização-do-repositório)
6. [Ferramentas recomendadas](#6-ferramentas-recomendadas)
7. [Glossário e decisões em aberto](#7-glossário-e-decisões-em-aberto)

---

## 1. Apresentação

Este documento reúne a base da metodologia de desenvolvimento da DPX. O objetivo
é transformar prática tácita (o que está na cabeça de quem desenvolve e nos
hábitos do dia a dia) em um ativo compartilhado pela equipe: um lugar único onde
quem entra no time entende como trabalhamos, e quem já está alinha o método.

Não é um regulamento rígido. É uma referência viva, que evolui com a equipe. As
recomendações aqui não são leis: são o caminho que consideramos bom hoje, e que
ajustamos quando aprendemos algo melhor.

A metodologia cobre **dois perfis de projeto**, e a diferença entre eles é o eixo
que organiza boa parte do documento:

- **Projetos internos**: ferramentas de uso da própria DPX (ERP, GDX e afins). Rodam em ambiente controlado, sem usuários externos.
- **Projetos externos / produção**: produtos que vão para a internet, com
  usuários reais e exposição pública. Aqui o nível de exigência (sobretudo em
  segurança) sobe.

A marcação [BASE] / [EXTERNO] em cada item diz a qual perfil ele se aplica.

---

## 2. Base da metodologia

### 2.1 Modelo copiloto 360° + executor (recomendação)

**O que é.** Um modo de trabalhar com IA dividido em dois papéis. Um agente atua
como **copiloto 360°**: enxerga o projeto inteiro (arquitetura, infraestrutura,
integrações), ajuda na estratégia, escreve specs e ADRs, e gera os prompts de
implementação. Outro agente atua como **executor**: implementa no repositório,
enxergando o código (e, no máximo, o banco). O ser humano decide, revisa e
mantém o controle do versionamento.

**Para que serve.** Separar o "pensar" do "fazer". O copiloto mantém a visão de
conjunto e a coerência das decisões; o executor aplica com precisão no código. A
pessoa fica no centro, validando cada etapa.

**O loop de trabalho.** O copiloto desenha a solução e escreve o prompt, a pessoa
roda esse prompt no executor, cola o retorno de volta, e o copiloto revisa. Cada
mudança passa por revisão humana antes de seguir.

**Quando usar.** Em mudanças de peso (novas telas, refatorações, decisões de
arquitetura), onde a visão de conjunto evita erro. **Quando não.** Em ajuste
trivial, onde o overhead dos dois papéis não compensa: faça direto.

> Este é um modelo **recomendado**, não uma norma. Cada pessoa pode adotá-lo no
> grau que fizer sentido.

### 2.2 Fluxo de trabalho

As demandas pendentes do dia a dia vivem no **GDX**. Hoje não usamos tracker formal nem revisão por pares obrigatória; a
estrutura mais completa de PR review e convenção de branches será definida mais
adiante (ver [decisões em aberto](#7-glossário-e-decisões-em-aberto)).

Princípios que já valem:

- **Confirmar o escopo antes de executar.** Deixar claro o que entra e,
  principalmente, o que **não** entra na tarefa. Escopo confirmado evita
  retrabalho. [BASE]
- **Não desenvolver direto na branch principal quando a aplicação está em
  produção.** Updates são feitos em uma branch secundária, validados localmente
  ou em preview, e só então mergeados na principal. **Exceção:** durante o
  desenvolvimento do MVP, enquanto a aplicação **ainda não está em produção**,
  trabalhar direto na branch principal é aceitável. A regra passa a valer a
  partir do momento em que existe ambiente de produção com usuários. [BASE]

### 2.3 Definition of Done

Uma tarefa só está **concluída** quando os três estão prontos. Pular qualquer um
deixa a tarefa incompleta, por mais que o código funcione. [BASE]

1. **Código** implementado seguindo os padrões deste documento.
2. **Verificação**: lint e typecheck passam; **o build local roda sem erro**
   (não só o servidor de desenvolvimento, que pode esconder erros que só
   aparecem no build de produção); e **todos os cenários possíveis do que foi
   desenvolvido são testados antes de ir para produção** (ver
   [Testes](#7-glossário-e-decisões-em-aberto)).
3. **Documentação reconciliada**: o que mudou de estado no projeto atualiza o
   documento certo (ver [seção 5](#5-documentação-e-organização-do-repositório)).
   Decisão de arquitetura vira ADR; item de roadmap se move; entrega entra no
   changelog; gambiarra vai para a dívida técnica.

### 2.4 Spec-Driven Development (SDD)

**O que é.** Para features de peso, escrever uma **spec** (o contrato da feature:
o que vamos construir e por quê) **antes** do código. A spec define o "o quê" e o
"porquê"; o plano de execução define o "como".

**Para que serve.** Alinhar premissas antes de gastar esforço, sobretudo quando
mais de uma pessoa (ou agente) trabalha na mesma coisa. É barato comparado a
refazer.

**Quando usar.** Telas complexas novas, mudanças de schema relevantes,
refatorações grandes, integrações externas. **Quando não.** Bug fix, ajuste de
copy, mudança pequena e óbvia: não precisam de spec.

> O formato, o ciclo de vida e a localização das specs estão detalhados em
> [Documentação e organização do repositório](#55-specs).

### 2.5 Convenções de commit

**O que é.** Um padrão de mensagem de commit que comunica a natureza da mudança
(`feat`, `fix`, `docs`, `refactor`, `chore`, etc.). [BASE]

**Para que serve.** Histórico legível, fácil de auditar, e que pode alimentar a
geração de changelog e notas de versão. Recomendamos o padrão **Conventional
Commits**. Referência: https://www.conventionalcommits.org

---

## 3. Arquitetura e boas práticas

### 3.1 Princípios gerais [BASE]

- **Nada hardcoded (centralização).** Endpoints, links, valores de configuração e
  constantes ficam em um lugar único (arquivo de constantes ou variáveis de
  ambiente), nunca espalhados como literais pelo código. Muda em um ponto,
  propaga para todos.
- **Tipagem forte.** Em projetos TypeScript, tipar de ponta a ponta. O tipo é a
  primeira camada de defesa contra bug.
- **Separação de camadas.** Manter fronteiras claras entre apresentação, regra de
  negócio e acesso a dados, na medida adequada ao tamanho do projeto.
- **Estados explícitos.** Todo fluxo trata loading, erro e estado vazio de forma
  explícita. Nunca deixar a interface presa em loading infinito.

Apoiamos esses princípios em convenções consagradas, que valem a leitura quando
houver dúvida sobre como estruturar:

- **Clean Code** (código legível, nomes claros, funções pequenas, baixa
  duplicação). Referência: livro *Clean Code*, de Robert C. Martin.
- **SOLID** (cinco princípios de design orientado a objetos para reduzir
  acoplamento e facilitar manutenção). Referência:
  https://en.wikipedia.org/wiki/SOLID
- **Clean Architecture** (organização em camadas com as dependências apontando
  para dentro, isolando regra de negócio de framework e infraestrutura). Aplicar
  de forma **gradual** e proporcional ao projeto, sem over-engineering em algo
  pequeno. Referência:
  https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- **The Twelve-Factor App** (boas práticas para apps que rodam como serviço,
  com destaque para configuração via variáveis de ambiente). Referência:
  https://12factor.net

### 3.2 Banco de dados e migrations

**Padrão de banco da DPX: Supabase** (Postgres gerenciado, com Auth e RLS).
Referência: https://supabase.com/docs

A disciplina de schema depende de uma escolha por projeto, e o ponto central é
**ser consistente com ela**:

- **Projeto disciplinado por migrations.** Toda mudança de schema (tabela,
  coluna, índice, política, trigger) entra como uma **migration versionada** via
  CLI, e ninguém edita o schema manualmente pelo painel. Nesse modo, as migrations
  **são a fonte de verdade** reprodutível do banco.
  - **Expand/contract** para mudanças destrutivas: primeiro uma migration
    **aditiva** (coluna nova nullable, etc.) que coexiste com o código antigo e
    novo; depois a migration de **contract** (drop/aperto) só quando ninguém mais
    usa o antigo. Evita quebrar produção, onde o banco é único.
  - **Migration aplicada antes do código** que lê/escreve a coluna nova. Código
    que grava em coluna inexistente quebra, e o erro costuma aparecer longe da
    causa.

- **Projeto editado direto no Supabase.** Em muitos casos (sobretudo interno e
  rápido) as tabelas são alteradas manualmente pelo painel. Aí as migrations
  **divergem do schema real** e **não devem ser tratadas como fonte de verdade**:
  o banco vivo é a verdade. Se em algum momento quisermos voltar à disciplina de
  migrations, re-baseamos o schema a partir de um **dump** do banco real e
  seguimos a partir dali.

A regra prática: **decida o modo no início do projeto e não misture**. O pior
cenário é o meio-termo, onde parte vai por migration e parte na mão, e ninguém
sabe o que é verdade.

### 3.3 Convenções de código [BASE]

- **Contratos de dados explícitos.** Definir e documentar unidades e formatos
  (ex.: tempo em segundos vs. minutos, distância em metros) em um lugar visível.
  Ambiguidade de unidade é fonte recorrente de bug.
- **Lógica reutilizável centralizada.** Parse, formatação e conversões moram em
  uma lib compartilhada; não reimplementar inline (gera duplicatas que divergem).
- **Naming consistente** e previsível em todo o projeto.

### 3.4 Lint e formatação [BASE]

- **Prettier** para formatação automática, garantindo estilo uniforme sem
  discussão manual. Referência: https://prettier.io
- **ESLint** para análise estática: pega padrões problemáticos e
  bugs prováveis, não só formatação. Vale adotar como complemento ao Prettier.
  Referência: https://eslint.org

---

## 4. Segurança

Esta é a seção onde a separação [BASE] / [EXTERNO] mais importa. Referência geral:
**OWASP** (https://owasp.org), em especial o OWASP Top Ten
(https://owasp.org/www-project-top-ten/) e os Cheat Sheets
(https://cheatsheetseries.owasp.org).

### Sempre, em todo projeto [BASE]

- **Segredos só em variáveis de ambiente.** Nunca commitar chaves, senhas ou
  tokens. Em produção, usar o cofre de variáveis da plataforma; em
  desenvolvimento, um arquivo de ambiente local fora do versionamento.
- **Validação de entrada com schema.** Toda entrada (corpo de requisição,
  parâmetros) é validada contra um schema antes de qualquer lógica. A ferramenta
  recomendada é o **Zod** (ver [Ferramentas](#6-ferramentas-recomendadas)).
- **Tratamento de erro sem vazar detalhe sensível** para o usuário final.

### Quando há usuários externos / exposição na web [EXTERNO]

- **RLS (Row Level Security).** Regras no banco que garantem que cada usuário só
  acessa os próprios dados. Serve para isolar dados por usuário na origem (no
  banco), não só na aplicação. Usar sempre que houver dados por usuário acessíveis
  a partir do cliente; no Supabase, ativar RLS por tabela. Referência:
  https://supabase.com/docs/guides/database/postgres/row-level-security
- **CORS (Cross-Origin Resource Sharing).** Controle de quais origens podem
  chamar a API. Restringir às origens legítimas; nunca liberar tudo em produção.
- **Rate limiting.** Limitar requisições por IP ou usuário em endpoints públicos,
  para conter abuso, brute force e custo. Aplicar nos endpoints públicos
  sensíveis (login, criação de conta, geração de conteúdo).
- **Guardrails e sanitização para LLM.** Quando há um modelo de linguagem no
  fluxo: sanitizar texto livre, isolar dados externos em delimitadores, proteger
  a system message, e validar a resposta contra um schema. Defesa em camadas
  contra prompt injection.
- **Security headers / CSP** (Content Security Policy, HSTS, X-Frame-Options,
  etc.). Reduzem a superfície de ataque no navegador. Introduzir primeiro em modo
  somente-relato e depois passar a enforcing.
- **Verificação de webhooks (assinatura / HMAC).** Webhooks de terceiros e
  chamadas entre serviços devem ser autenticados (segredo no header, ou HMAC com
  comparação em tempo constante). Nunca confiar em um webhook sem verificar a
  origem.
- **LGPD / privacidade.** Antes de coletar dado pessoal sensível (ex.: saúde),
  documentar a base legal e a Política de Privacidade. Praticar **minimização**:
  dado sensível não vai para ferramentas de analytics.

---

## 5. Documentação e organização do repositório

Documentação não morre por falta de criação, morre por falta de manutenção. A
regra de ouro: toda tarefa que muda o estado do projeto atualiza o documento
certo, como parte do [Definition of Done](#23-definition-of-done). [BASE]

### 5.1 Nomes e estrutura do repositório

**Nome do repositório.** Usar **kebab-case** (palavras minúsculas separadas por
hífen), com nomes **intuitivos e descritivos** do que o projeto é. Bons exemplos:
`erp-financeiro`, `gestor-de-demandas`, `site-institucional`. Evitar nomes
genéricos ou opacos (`projeto1`, `app-novo`) que não dizem nada a quem chega.

**Onde os documentos vivem.** A estrutura de pastas recomendada, para que todo
projeto tenha a documentação no mesmo lugar:

```
repositorio/
├── README.md               # porta de entrada do projeto
├── CHANGELOG.md            # histórico técnico por versão
├── RELEASE-NOTES.md        # versão em linguagem simples (destilada do changelog)
├── CLAUDE.md               # contexto/mapa do repo para o agente de IA
├── .env.example            # variáveis de ambiente necessárias (sem valores)
├── docs/
│   ├── ROADMAP.md          # estado vivo do projeto
│   ├── TECH-DEBT.md        # dívida técnica
│   ├── COPILOT-BRIEFING.md # briefing do projeto para o copiloto
│   └── adr/
│       ├── README.md       # índice dos ADRs
│       ├── TEMPLATE.md     # modelo de ADR
│       └── 0001-titulo.md  # um arquivo por decisão
├── specs/
│   ├── README.md           # índice das specs
│   ├── TEMPLATE.md         # modelo de spec
│   └── 0001-titulo.md      # um arquivo por feature
└── .claude/
    └── rules/              # convenções carregadas pelo agente de IA
        ├── ways-of-working.md
        ├── code-conventions.md
        └── security.md
```

> A pasta `.claude/` e o `CLAUDE.md` são o exemplo concreto para o agente de IA
> que recomendamos; o nome exato depende da ferramenta adotada (ver
> [seção 6](#6-ferramentas-recomendadas)). O importante é existir um lugar
> versionado para regras e contexto da IA.

### 5.2 Os documentos do projeto

#### `README.md`
**O que é.** A porta de entrada do repositório, a primeira coisa que se lê.
**Para que serve.** Explicar o que o projeto é, como rodá-lo localmente, onde
estão as coisas e como contribuir. **Quando atualizar.** Sempre que mudar algo
estrutural (como rodar, dependências, scripts). **Onde fica.** Raiz do repo.

> Observação: este próprio documento de metodologia é um `README.md` (o do
> repositório de perfil da organização), por isso aparece no perfil da DPX.

#### `ROADMAP.md`
**O que é.** O estado vivo do projeto: o que já está feito, o que está em
andamento e o que está pendente, na ordem acordada. **Para que serve.** Dar a
qualquer pessoa, a qualquer momento, a visão de onde o projeto está e para onde
vai, sem precisar perguntar. **Quando atualizar.** Ao fechar ou mudar um item:
ele se **move** para a seção de concluídos (mantém-se um registro enxuto do que
foi entregue). **Onde fica.** `docs/ROADMAP.md`.

#### `TECH-DEBT.md`
**O que é.** O registro de gambiarras, adiamentos conscientes e riscos
conhecidos, cada um com uma severidade (ex.: crítico, alto, médio, baixo).
**Para que serve.** Tornar a dívida **visível e rastreável**, em vez de
esquecida. Toda gambiarra criada se registra aqui, e idealmente deixa um marcador
`TODO` no código apontando para ela. **Quando atualizar.** Ao criar uma dívida
(registrar) e ao resolvê-la (marcar como resolvida, **sem apagar**: o histórico
tem valor). **Onde fica.** `docs/TECH-DEBT.md`.

#### `RELEASE-NOTES.md`
**O que é.** As notas de versão em **linguagem simples**, voltadas ao usuário, e
não ao desenvolvedor. **Para que serve.** Comunicar o que mudou de forma
acessível, no corte de cada versão. É sempre **destilado do changelog**, nunca
escrito do zero (o changelog é a fonte técnica; as release notes são o recorte
amigável). **Quando atualizar.** No corte de uma versão. **Onde fica.** Raiz do
repo. Relaciona-se diretamente com o [CHANGELOG](#53-changelog-versionamento-e-releases).

#### `WAYS-OF-WORKING`
**O que é.** O documento que descreve **como o time trabalha**, independente do
arquivo que está sendo tocado: o Definition of Done, a disciplina de
documentação, os papéis, a triagem de quando algo vira ADR. **Para que serve.**
Ser a referência de processo que vale em toda tarefa. **Quando atualizar.** Ao
mudar uma prática de trabalho da equipe. **Onde fica.** Junto das regras da IA
(ex.: `.claude/rules/ways-of-working.md`), por ser carregado como contexto em
toda sessão.

#### `COPILOT-BRIEFING`
**O que é.** Um briefing consolidado do projeto para o agente de IA: visão geral,
stack, decisões em vigor, particularidades. **Para que serve.** Dar ao copiloto
um onboarding rápido em uma sessão nova, sem precisar reconstruir o contexto do
zero a cada vez. **Quando atualizar.** Quando o panorama do projeto muda de forma
relevante. **Onde fica.** `docs/COPILOT-BRIEFING.md`.

#### Arquivo de contexto da IA (ex.: `CLAUDE.md`)
**O que é.** O mapa do repositório e as regras de base que o agente de IA carrega
sempre (estrutura de pastas, convenções inegociáveis, armadilhas conhecidas).
**Para que serve.** Garantir que o agente trabalhe alinhado ao projeto desde o
primeiro prompt. **Quando atualizar.** Ao mudar a estrutura do repo ou uma regra
de base. **Onde fica.** Raiz do repo (o nome depende da ferramenta de IA).

#### `.env.example`
**O que é.** A lista de todas as variáveis de ambiente que o projeto precisa, com
os nomes mas **sem os valores** reais. **Para que serve.** Permitir que outra
pessoa reproduza o ambiente, sabendo exatamente o que precisa configurar, sem
expor segredos. **Quando atualizar.** Sempre que uma variável de ambiente nova
passa a ser necessária. **Onde fica.** Raiz do repo.

### 5.3 CHANGELOG, versionamento e releases

#### O CHANGELOG e o formato Keep a Changelog
**O que é.** O histórico técnico das mudanças do projeto, organizado por versão.
**Para que serve.** Saber o que mudou, quando e por quê, em ordem. Adotamos o
formato **Keep a Changelog**: as mudanças se organizam em seções (`Added`,
`Changed`, `Fixed`, `Security`) e se acumulam em um bloco `[Unreleased]` até o
corte da versão. **Onde fica.** Raiz do repo. Referência:
https://keepachangelog.com

#### As duas formas de versionar (escolha por caso)

Mantemos **as duas** à disposição da equipe; a escolha depende do tipo de projeto.

**A) SemVer (Semantic Versioning):** `MAJOR.MINOR.PATCH` (ex.: `2.4.1`). O gatilho
é **compatibilidade técnica**:
- **MAJOR**: mudança que **quebra compatibilidade** para quem consome o código.
- **MINOR**: funcionalidade nova mantendo compatibilidade.
- **PATCH**: correção de bug mantendo compatibilidade.
Referência: https://semver.org

**B) Marco de produto:** `MARCO.LANÇAMENTO.CORREÇÃO`. O gatilho é a **percepção do
usuário**, não compatibilidade:
- **MARCO**: redefinição do produto ("o produto mudou"). Raro.
- **LANÇAMENTO**: funcionalidade nova que o usuário percebe (bloco `Added` /
  `Changed`).
- **CORREÇÃO**: bug fix, ajuste, performance, sem capacidade nova (bloco `Fixed` /
  `Security`).

**Quando usar A ou B.** Use **SemVer (A)** quando o software é **consumido por
outros desenvolvedores**: bibliotecas, pacotes, APIs públicas, SDKs. Ali o número
é um contrato técnico de compatibilidade. Use **marco de produto (B)** quando o
software é um **produto de usuário final** (web app, app interno): ninguém depende
do seu código como dependência, então o número que comunica marco de produto é
mais útil. Um mesmo projeto pode usar B no produto e A em uma API pública que ele
venha a expor.

#### Tags e releases no repositório

**O que é.** No corte de uma versão, cria-se uma **tag git** (ex.: `v1.2.0`)
apontando para aquele commit, e publica-se um **release** no GitHub com as notas
daquela versão (destiladas do changelog / release-notes). **Para que serve.**
Marcar um ponto fixo e nomeado na história do projeto, fácil de localizar e
comunicar. **Quando criar.** No fechamento de uma versão (sobe o MARCO/LANÇAMENTO,
ou MAJOR/MINOR), não a cada commit de correção pequena.

> **Atenção (Cloudflare Workers).** No nosso padrão de deploy, criar uma tag ou um
> release **não publica em produção**. O deploy de Workers é feito via Wrangler
> (comando, ou automação futura), não a partir de tag/release do GitHub. Aqui, a
> tag e o release são um **marco de documentação e comunicação** (registrar a
> versão e suas notas), não o gatilho de deploy. Não confundir os dois.

### 5.4 ADRs (Architecture Decision Records)

**O que é.** Um registro curto e padronizado de uma decisão de arquitetura, com
contexto, a decisão tomada, as alternativas consideradas e as consequências.
**Para que serve.** Responder, daqui a seis meses ou para quem chega no time, a
pergunta "por que está assim?". A decisão e o seu porquê ficam explícitos, não
perdidos na memória de quem decidiu. **Quando escrever** (triagem): quando há
**alternativas plausíveis** e o porquê **não é óbvio** pelo código (ex.: escolher
uma ferramenta em vez de outra, uma ordem de validação, um workaround). **Quando
não:** bug fix trivial, ajuste de copy, mudança óbvia sem alternativa real.
**Regra de ouro:** ADRs são **append-only**, não se edita um ADR aceito; quando a
decisão muda, cria-se um novo que marca o anterior como substituído. **Onde
ficam.** `docs/adr/`, numerados em sequência, a partir de um `TEMPLATE.md`, com um
`README.md` de índice. Referência: https://adr.github.io

### 5.5 Specs (Spec-Driven Development)

**O que é.** O contrato de uma feature, escrito **antes** do código: o que vai ser
construído, por quê, o escopo (o que entra e o que não entra), e os critérios de
aceite. Complementa a [seção 2.4](#24-spec-driven-development-sdd). **Para que
serve.** Alinhar todo mundo (pessoas e agentes) sobre o que será feito antes de
gastar esforço, evitando premissas divergentes e retrabalho. **Quando escrever:**
features de peso (telas complexas, mudanças de schema, refatorações grandes,
integrações). **Quando não:** bug fix e ajuste pequeno. **Ciclo de vida:**
**Proposta** (rascunho em discussão) → **Aprovada** (contrato fechado, implementa-se
contra ela) → **Implementada** (feita, mantida para referência, linkando os ADRs
que saíram dela). **Onde ficam.** `specs/`, numeradas em sequência, a partir de um
`TEMPLATE.md`, com um `README.md` de índice.

### 5.6 Rules e contexto para IA

**O que é.** Arquivos de convenção que o agente de IA carrega como contexto em
toda sessão de trabalho. São o que faz o agente seguir os padrões do projeto sem
precisar ser lembrado a cada prompt. **Para que serve.** A memória de longo prazo
do método vive em **documentos versionados**, não na cabeça de uma pessoa nem em
um chat que se perde. Quando a prática muda, muda-se a rule, e o agente passa a
seguir a nova. **Exemplos de arquivos:** `ways-of-working.md` (como o time
trabalha), `code-conventions.md` (convenções de código), `security.md` (regras de
segurança), `migrations.md` (disciplina de banco), e quantos mais o projeto
precisar. **Onde ficam.** Em um diretório de regras do agente, por exemplo
`.claude/rules/` (o caminho exato depende da ferramenta de IA adotada).

---

## 6. Ferramentas recomendadas

Os princípios ficam nas seções acima; aqui vão as ferramentas concretas que
recomendamos, com o que cada uma é, para que serve, quando faz sentido e o link da
documentação. As marcadas como padrão DPX são as que a empresa adota por default.

### Cloudflare Workers — deploy (padrão DPX) [BASE]
Plataforma de execução serverless na borda da Cloudflare. Serve para hospedar e
rodar aplicações próximas do usuário, com escala automática. É o padrão de deploy
da DPX porque os domínios da empresa estão centralizados na Cloudflare, o que
simplifica DNS, certificados e gestão. Referência:
https://developers.cloudflare.com/workers/

### Wrangler — CLI de deploy (padrão DPX) [BASE]
A ferramenta de linha de comando da Cloudflare para desenvolver, testar e
publicar Workers. Serve para fazer o build e o deploy a partir do ambiente local
(hoje, via VS Code). É como, na prática, uma aplicação chega à Cloudflare.
Referência: https://developers.cloudflare.com/workers/wrangler/

### Supabase — banco de dados e autenticação (padrão DPX) [BASE]
Plataforma sobre Postgres que entrega banco de dados, autenticação, RLS e uma API
pronta. Serve como a camada de dados e de login dos projetos. É o padrão de banco
da DPX. Tem CLI para migrations e painel para administração. Referência:
https://supabase.com/docs

### Zod — validação de schema [BASE]
Biblioteca de validação de schema para TypeScript, com inferência de tipos a
partir do schema. Serve para validar toda entrada de dados (corpo de requisição,
parâmetros) antes da lógica, barrando dado malformado na porta. É a forma concreta
de aplicar o princípio de "validação de entrada" da [seção 4](#4-segurança). Faz
sentido em qualquer projeto que recebe dados. Referência: https://zod.dev

### Sentry — monitoramento de erros [EXTERNO] (útil também interno)
Serviço de captura e agregação de erros em tempo real, com stack trace, contexto e
alertas. Serve para descobrir e diagnosticar falhas em produção sem depender do
usuário relatar. É crítico em produtos externos, onde um erro silencioso afeta
usuários reais; em projeto interno, ajuda mas é menos urgente. Referência:
https://docs.sentry.io

### PostHog — analytics de produto [EXTERNO]
Plataforma de analytics de produto: eventos, funis de conversão e comportamento de
uso. Serve para entender como as pessoas usam o produto e onde abandonam. Faz
sentido sobretudo em produtos externos, onde há uma base de usuários para medir;
em ferramenta interna, raramente se justifica. Atenção à minimização de dados
(dado sensível não deve ser enviado para o analytics). Referência:
https://posthog.com/docs

### Upstash Redis — rate limiting e cache [EXTERNO]
Redis serverless, cobrado por uso, que usamos sobretudo para **rate limiting** de
endpoints públicos. Serve para conter abuso, brute force e custo em rotas expostas
na internet. Faz sentido em projetos externos; em interno, normalmente não.
Referência: https://upstash.com/docs/redis

### Prettier — formatação de código [BASE]
Formatador automático de código. Serve para manter um estilo uniforme no projeto
inteiro sem discussão manual sobre formatação. Roda no editor e/ou num passo de
verificação. Vale em todo projeto. Referência: https://prettier.io

### ESLint — análise estática [BASE]
Ferramenta de análise estática que aponta padrões problemáticos e bugs prováveis,
indo além da formatação. Serve como uma rede de segurança que pega erro antes de
rodar. Recomendado como complemento ao Prettier em todo projeto. Referência:
https://eslint.org

---

## 7. Glossário e decisões em aberto

### Glossário

- **[BASE] / [EXTERNO]**: marcação de onde um item se aplica (todo projeto vs.
  projetos com usuários externos).
- **ADR**: Architecture Decision Record, registro de decisão de arquitetura.
- **Spec / SDD**: contrato de feature escrito antes do código (Spec-Driven
  Development).
- **DoD**: Definition of Done, a definição de quando uma tarefa está concluída.
- **Expand/contract**: estratégia de mudança de schema em duas etapas (aditiva,
  depois destrutiva) para não quebrar produção.
- **RLS**: Row Level Security, regras de acesso por linha no banco.
- **CSP**: Content Security Policy, política de segurança no navegador.
- **HMAC**: assinatura criptográfica para verificar a origem de uma mensagem.
- **SemVer**: Semantic Versioning, versionamento por compatibilidade técnica.
- **kebab-case**: palavras minúsculas separadas por hífen (ex.: `gestor-de-demandas`).

### Decisões em aberto

Itens que a DPX ainda não fechou e vai detalhar mais adiante. Ficam listados aqui,
de forma honesta, em vez de fixar um padrão prematuro:

- **Convenção de branches e PR review.** Hoje trabalhamos em branch secundária e
  validamos antes do merge na principal (com a exceção do MVP, ver
  [2.2](#22-fluxo-de-trabalho)), mas sem um fluxo formal de revisão por pares. A
  estrutura completa (nomes de branch, regras de PR, quem aprova) será definida.
- **Estratégia de testes.** Por ora, a regra é testar todos os cenários possíveis
  do que está sendo desenvolvido antes de ir para produção. Uma estrutura mais
  completa (testes automatizados, cobertura, CI) virá depois.
- **CI/CD.** O deploy hoje é feito via Wrangler no VS Code. Automatizar
  build/deploy (e quando vale) é uma decisão futura.

---

*Este documento é vivo. Ao mudar uma prática da equipe, atualize-o e registre a
data no topo.*
