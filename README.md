# Metodologia de Desenvolvimento - DPX

> Base da metodologia de desenvolvimento da equipe de tecnologia da DPX: como
> trabalhamos, quais padrões aplicamos e por quê. Serve tanto para apps de uso
> **interno** (ex.: ERP, GDX, etc) quanto para **produtos externos**
> que vão para produção, com usuários e exposição na internet.

**Versão 1.1 · Última atualização: 08/08/2026**

### Como ler este documento

Cada item é marcado conforme onde se aplica:

- **[BASE]** : vale para **todo** projeto, sempre, incluindo apps internos.
- **[EXTERNO]** : obrigatório quando o projeto terá **usuários externos ou
  exposição na internet**. Em ferramenta interna costuma ser opcional ou
  dispensável.

Onde faz sentido, cada item responde quatro perguntas: o que é, para que serve,
quando usar e quando não usar. Sempre que citamos um padrão ou ferramenta com
documentação própria, deixamos um link de referência para aprofundamento.

**Uma ressalva sobre stack.** Este documento define método, não linguagem. As
ferramentas listadas na [seção 6](#6-ferramentas-recomendadas) se dividem em duas
naturezas: as de **uso geral**, que independem da linguagem do projeto, e as que
**dependem da stack escolhida**. A escolha de linguagem, framework e bibliotecas
é particular de cada projeto e deve atender à necessidade dele.

Na prática, a maior parte do que construímos é em JavaScript e TypeScript, e por
isso os exemplos concretos citados ao longo do texto vêm desse ecossistema. Eles
estão ali como referência, não como exigência: em outra stack, o princípio é o
mesmo e a ferramenta é a equivalente.

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

- **Projetos internos**: ferramentas de uso da própria DPX (ERP, GDX e afins).
  Rodam em ambiente controlado, sem usuários externos.
- **Projetos externos ou de produção**: produtos que vão para a internet, com
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
enxergando o código e, no máximo, o banco. O ser humano decide, revisa e mantém
o controle do versionamento.

**Para que serve.** Separar o "pensar" do "fazer". O copiloto mantém a visão de
conjunto e a coerência das decisões; o executor aplica com precisão no código. A
pessoa fica no centro, validando cada etapa.

**O loop de trabalho.** O copiloto desenha a solução e escreve o prompt, a pessoa
roda esse prompt no executor, o executor implementa e emite um **relatório de
fechamento**, a pessoa cola esse retorno de volta, e o copiloto revisa contra o
relatório. Cada mudança passa por revisão humana antes de seguir.

**Quando usar.** Em mudanças de peso (novas telas, refatorações, decisões de
arquitetura), onde a visão de conjunto evita erro. **Quando não.** Em ajuste
trivial, onde o overhead dos dois papéis não compensa: faça direto.

> Este é um modelo **recomendado**, não uma norma. Cada pessoa pode adotá-lo no
> grau que fizer sentido.

#### Quem verifica o quê [BASE]

A divisão de papéis só funciona se ficar claro o que cada um consegue, de fato,
verificar:

- **O executor verifica o que é automatizável**: lint, verificação de tipos,
  build, consultas de leitura, buscas de conferência.
- **A pessoa verifica o que exige olho e dispositivo**: comportamento visual,
  interação, fluxo ponta a ponta, aparência em diferentes tamanhos de tela.
- **O agente não deve simular o que não pode verificar.** Se ele não tem como
  abrir um navegador, deve dizer isso no relatório, e a tarefa permanece
  incompleta até que alguém verifique.
- **Emulação não substitui o motor real.** O modo dispositivo do navegador roda o
  mesmo motor do desktop. Comportamento específico de outro motor exige aparelho
  real, simulador ou serviço de dispositivos remotos.

#### Relatório de fechamento (`/wrap`) [BASE]

**O que é.** Um comando (ou rotina) que o executor roda ao concluir uma tarefa,
produzindo um relatório padronizado do que foi feito. No nosso setup, vive em
`.claude/commands/wrap.md`.

**Para que serve.** Dar ao copiloto e à pessoa uma base factual para revisar e
para reconciliar a documentação. Sem ele, o changelog e o ADR são escritos a
partir da intenção, não do que de fato mudou.

**O que o relatório precisa conter:**

- **O que mudou**, com base no diff real, não no que se pretendia mudar.
- **Decisões tomadas** durante a execução, e o porquê de cada uma.
- **Desvios** do que foi pedido, se houve.
- **Dívida técnica** gerada ou resolvida.
- **O que NÃO pôde ser verificado.** O item mais importante da lista: é ele que
  impede uma tarefa de ser dada como concluída sem a verificação humana.
- **Candidatos a regra**: aprendizado durável descoberto durante a execução (ver
  [5.6](#56-rules-e-contexto-para-ia)).

#### Confiança calibrada no agente [BASE]

Agentes erram com confiança. Reportam contagens erradas, afirmam que um arquivo
está num estado que não está, e concluem por leitura de código aquilo que só o
navegador ou o banco respondem.

A postura correta: **o relatório do agente é o ponto de partida da revisão, não a
conclusão dela**. Verificação factual (contagem, estado de arquivo, existência de
coluna, comportamento em produção) se confirma na fonte, não na afirmação.

### 2.2 Fluxo de trabalho

As demandas pendentes do dia a dia vivem no **GDX**.

Princípios que já valem:

- **Confirmar o escopo antes de executar.** Deixar claro o que entra e,
  principalmente, o que **não** entra na tarefa. Escopo confirmado evita
  retrabalho. [BASE]
- **Diagnóstico antes de ação que altera estado**, sobretudo em produção (banco,
  deploy, configuração). Comandos de leitura primeiro; entender o terreno antes
  de escrever. Pontos de parada explícitos antes de passos arriscados. [BASE]

> A estrutura de branches e a regra de não desenvolver direto na principal estão
> detalhadas em [2.6](#26-estrutura-de-branches).

### 2.3 Definition of Done

Uma tarefa só está **concluída** quando os três estão prontos. Pular qualquer um
deixa a tarefa incompleta, por mais que o código funcione. [BASE]

1. **Código** implementado seguindo os padrões deste documento.

2. **Verificação**: as checagens automáticas da stack passam (em JavaScript e
   TypeScript: ESLint, verificação de tipos e os testes existentes); **o build de
   produção roda sem erro** (não só o servidor de desenvolvimento, que pode
   esconder erros que só aparecem no build); e **todos os cenários possíveis do
   que foi desenvolvido são testados antes de ir para produção**.

   A verificação segue a divisão de [2.1](#quem-verifica-o-quê): o que é
   automatizável fica com o agente, o que exige olho e dispositivo é humano.
   Quando existe **preview deploy** (ver [2.6](#26-estrutura-de-branches)), a
   verificação humana acontece nele, não em produção.

   Quando houver **testes E2E automatizados**, eles cobrem os fluxos críticos e
   reduzem o peso do teste manual, mas não o eliminam para o que acabou de ser
   desenvolvido.

3. **Documentação reconciliada**: o que mudou de estado no projeto atualiza o
   documento certo (ver [seção 5](#5-documentação-e-organização-do-repositório)).
   Decisão de arquitetura vira ADR; item de roadmap se move; entrega entra no
   changelog; gambiarra vai para a dívida técnica.

   Quando se trabalha com o modelo de
   [2.1](#21-modelo-copiloto-360--executor-recomendação), o **relatório de
   fechamento** é o insumo desta etapa: documenta-se a partir do que de fato
   mudou, não do que se pretendia mudar.

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

### 2.6 Estrutura de branches

**O que é.** A organização das branches do repositório e o caminho que uma
mudança percorre até chegar em produção. [BASE]

**Para que serve.** Garantir que nada chegue a usuários reais sem passar por
verificação, e permitir que mais de uma pessoa trabalhe no mesmo repositório sem
atropelamento.

#### O critério que define a estrutura: estar em produção

A estrutura muda em um único ponto: **se existe ou não ambiente de produção sendo
usado**. Não é sobre ter "lançado" formalmente, é sobre existir gente usando.

**Antes de estar em produção**, trabalhar direto na branch principal é aceitável.
Não há usuário para quebrar, e o overhead de PR atrasa a construção do MVP.

**A partir do momento em que existe produção com usuários**, a estrutura completa
passa a valer.

> **A transição é um evento datado, não uma percepção gradual.** Alguém precisa
> dizer "a partir de hoje trabalhamos com branches", registrar isso, e passar a
> seguir. Sem esse marco explícito, a exceção do MVP se prolonga por inércia, e o
> primeiro bug que chega a usuário real vem por aí.

#### A estrutura em produção

- **`main`** : o que está em produção. Nunca recebe commit direto.
- **`dev`** (ou `develop`) : integração do que está pronto e ainda não foi
  publicado. Opcional em time pequeno, onde as feature branches podem ir direto
  para `main` via PR.
- **`feature/*`, `fix/*`, `chore/*`** : uma branch por frente de trabalho, criada
  a partir de `dev` (ou de `main`, se não houver `dev`).

#### Como manusear

- **Uma branch por frente coesa.** Nome que descreve o que a mudança faz, não o
  número da tarefa: `fix/modal-mobile`, não `fix/tarefa-42`.
- **Branch de vida curta.** Quanto mais tempo aberta, mais divergência acumula e
  mais doloroso é o merge. Se uma frente vai levar semanas, fatie a entrega.
- **Sincronizar com frequência.** Trazer as mudanças da branch base para a sua
  antes de abrir o PR, e não no dia do merge.
- **Deletar a branch após o merge.** Repositório com dezenas de branches mortas
  esconde as que estão vivas. A limpeza é parte do fechamento.

#### Com mais de uma pessoa no mesmo repositório

- **Cada pessoa em sua própria branch**, mesmo em mudança pequena. Duas pessoas
  na mesma branch geram conflito sem revisão.
- **Fatiar por fronteira de arquivo sempre que possível.** Duas frentes que tocam
  o mesmo arquivo grande vão conflitar; combinar a ordem antes evita o conflito.
- **Nunca reescrever histórico já publicado** (`--force` sem cuidado, rebase de
  branch compartilhada). Se precisar desfazer algo já enviado, prefira um commit
  que reverte.
- **PR pequeno é revisado; PR gigante é aprovado sem ler.** O tamanho da mudança
  é o principal fator de qualidade da revisão.

#### O que o fluxo de PR habilita

- **Preview deploy**: cada branch ou PR gera um ambiente igual ao de produção,
  onde a verificação humana acontece antes do merge.
- **Portão automático**: ferramentas de análise que rodam em PR (ver
  [seção 6](#6-ferramentas-recomendadas)) passam a bloquear antes do merge, em
  vez de reportar depois.

### 2.7 Corte de versão

**O que é.** O ritual de fechar uma versão: consolidar o que mudou, publicar as
notas e marcar o ponto na história do repositório. [BASE]

**Para que serve.** Ter marcos nomeados e recuperáveis, e comunicar o que mudou
para quem usa o produto.

**A ordem dos passos** (o detalhamento de cada artefato está em
[5.3](#53-changelog-versionamento-e-releases)):

1. **CHANGELOG**: o bloco `[Unreleased]` vira `[X.Y.Z] - AAAA-MM-DD`, e um novo
   `[Unreleased]` vazio é criado acima.
2. **RELEASE-NOTES**: nova seção, **destilada do changelog**, com o recorte do
   que o usuário percebe.
3. **Revisar as premissas condicionais da dívida técnica** (ver
   [5.2](#tech-debtmd)): itens adiados com justificativa do tipo "não urgente
   porque ainda não há X" precisam ser reavaliados a cada versão, porque essas
   condições caem sem aviso.
4. **Commit e push.**
5. **Tag anotada** apontando para esse commit, e push da tag.
6. **Release** a partir da tag, com o corpo recortado do release notes.

> A ordem importa: a tag precisa apontar para o commit que **já contém** o
> changelog cortado. E a mensagem do commit descreve os arquivos
> (`docs: corta a versão X.Y.Z`), enquanto a da tag descreve a versão. São textos
> diferentes, escritos para leitores diferentes.

---

## 3. Arquitetura e boas práticas

### 3.1 Princípios gerais [BASE]

- **Nada hardcoded (centralização).** Endpoints, links, valores de configuração e
  constantes ficam em um lugar único (arquivo de constantes ou variáveis de
  ambiente), nunca espalhados como literais pelo código. Muda em um ponto,
  propaga para todos.
- **Tipagem forte.** Quando a linguagem oferece tipagem estática, usá-la de ponta
  a ponta (nos nossos projetos JS/TS, isso significa TypeScript em todo o
  caminho, não só nas bordas). O tipo é a primeira camada de defesa contra bug.
  Em linguagem de tipagem dinâmica, o equivalente é validar nas fronteiras
  (entrada, saída, integração).
- **Separação de camadas.** Manter fronteiras claras entre apresentação, regra de
  negócio e acesso a dados, na medida adequada ao tamanho do projeto.
- **Estados explícitos.** Todo fluxo trata loading, erro e estado vazio de forma
  explícita. Nunca deixar a interface presa em loading infinito.
- **Cuidado com o sucesso silencioso.** Erro que aparece é barato: alguém vê e
  corrige. Operação que **reporta sucesso sem ter agido** é cara, porque
  sobrevive ao teste, à revisão de código e a meses de produção. Confirme o
  **efeito** da operação, não apenas a ausência de erro. E dado que é gravado e
  nunca lido não existe: ao adicionar um campo, percorra o caminho inteiro até
  quem consome, com dado real.

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
- **The Twelve-Factor App** (boas práticas para apps que rodam como serviço, com
  destaque para configuração via variáveis de ambiente). Referência:
  https://12factor.net

### 3.2 Banco de dados e migrations

**Padrão de banco da DPX: Supabase** (Postgres gerenciado, com Auth e RLS).
Referência: https://supabase.com/docs

A disciplina de schema depende de uma escolha por projeto, e o ponto central é
**ser consistente com ela**:

- **Projeto disciplinado por migrations.** Toda mudança de schema (tabela,
  coluna, índice, política, trigger) entra como uma **migration versionada** via
  CLI, e ninguém edita o schema manualmente pelo painel. Nesse modo, as
  migrations **são a fonte de verdade** reprodutível do banco.
  - **Expand/contract** para mudanças destrutivas: primeiro uma migration
    **aditiva** (coluna nova nullable, etc.) que coexiste com o código antigo e
    novo; depois a migration de **contract** (drop ou aperto) só quando ninguém
    mais usa o antigo. Evita quebrar produção, onde o banco é único.
  - **Migration aplicada antes do código** que lê ou escreve a coluna nova.
    Código que grava em coluna inexistente quebra, e o erro costuma aparecer
    longe da causa.

- **Projeto editado direto no Supabase.** Em muitos casos (sobretudo interno e
  rápido) as tabelas são alteradas manualmente pelo painel. Aí as migrations
  **divergem do schema real** e **não devem ser tratadas como fonte de verdade**:
  o banco vivo é a verdade. Se em algum momento quisermos voltar à disciplina de
  migrations, re-baseamos o schema a partir de um **dump** do banco real e
  seguimos a partir dali.

A regra prática: **decida o modo no início do projeto e não misture**. O pior
cenário é o meio-termo, onde parte vai por migration e parte na mão, e ninguém
sabe o que é verdade.

#### Pré-flight antes de mudança destrutiva [BASE]

Antes de aplicar uma migration que remove ou aperta algo em produção:

- **Consulte o dado que será afetado.** Quantas linhas existem, quantas estão
  preenchidas, quais valores aparecem. Uma coluna vazia se remove sem cerimônia;
  uma com dado exige plano.
- **Confirme que não há consumidor.** Busca no código, e também nos artefatos
  fora do repositório (ver [3.5](#35-código-fora-do-repositório)).
- **Inventário vem do banco, não do arquivo.** Para saber o que existe em uma
  tabela, consulte o catálogo do banco, não o arquivo de migration. Arquivo longo
  é fácil de ler pela metade.
- **Ponto de parada explícito** antes de aplicar, com o SQL revisado por outra
  pessoa (ou pelo copiloto) antes da execução.

### 3.3 Convenções de código [BASE]

- **Contratos de dados explícitos.** Definir e documentar unidades e formatos
  (ex.: tempo em segundos ou minutos, distância em metros) em um lugar visível.
  Ambiguidade de unidade é fonte recorrente de bug.
- **Contrato declarado não é contrato verificado.** Tipagem estática não valida o
  conteúdo de campo dinâmico (JSON no banco, resposta de API externa, payload de
  webhook): a interface escrita à mão sobre isso é uma **suposição**. Ao mudar o
  contrato de um campo desses, confirme o tipo real contra o dado antes de
  confiar na declaração.
- **Opções de interface derivam do domínio.** Valores de select, enum e radio
  saem de uma constante compartilhada ou da própria restrição do banco, nunca
  escritos direto na tela. Caso contrário a interface oferece uma opção que o
  banco rejeita, e a falha só aparece quando alguém escolhe justamente aquela.
- **Lógica reutilizável centralizada.** Parse, formatação e conversões moram em
  uma biblioteca compartilhada; não reimplementar inline, o que gera duplicatas
  que divergem.
- **Naming consistente** e previsível em todo o projeto.

### 3.4 Lint, formatação e qualidade [BASE]

Toda stack tem suas ferramentas de formatação e análise estática. A escolha
específica depende da linguagem; o que não é opcional é **ter**:

- **Formatação automática**, para manter estilo uniforme sem discussão manual
  (em JS/TS, o Prettier).
- **Análise estática**, que pega padrões problemáticos e bugs prováveis antes de
  rodar o código (em JS/TS, o ESLint).
- **Complexidade ciclomática** como sinal de alerta. É a medida de quantos
  caminhos possíveis existem dentro de uma função: quanto maior, mais difícil de
  entender, testar e alterar sem quebrar. Vale configurar um limite na ferramenta
  de análise e tratar o estouro como gatilho de refatoração, não como erro fatal.
  Arquivo ou função que cresceu demais é o lugar onde os bugs se escondem.

### 3.5 Código fora do repositório

**O que é.** Lógica de produto que não mora no git: prompt de agente de IA, nó de
código em ferramenta de automação, configuração feita em painel, regra em serviço
de terceiro. [BASE]

**Por que merece atenção.** É legítimo e às vezes inevitável, mas quem trabalha
assim abre mão de quatro coisas que o repositório dá de graça:

- **Histórico e diff.** Não dá para ver o que mudou nem voltar atrás.
  **Exporte o artefato antes de qualquer alteração**, porque ele é o único
  rollback.
- **Compartilhamento de código.** Duas automações que precisam da mesma função
  não a importam: cada uma tem a própria cópia. Corrigir uma e esquecer a outra é
  o modo de falha padrão, e ele é silencioso. Registre a duplicação como dívida
  técnica e sincronize à mão de forma deliberada.
- **Atomicidade com o deploy.** Código e artefato externo não sobem juntos.
  Durante a janela entre um e outro, o código precisa tolerar as duas versões do
  contrato.
- **Alcance do agente de IA.** O executor não enxerga o que está fora do repo.
  Diagnóstico ali é trabalho humano, e a spec precisa considerar isso ao dividir
  o escopo.

**Consequência para a documentação.** A spec descreve o **contrato** do artefato
externo (o que ele recebe, o que devolve, quais regras precisa cumprir), nunca
copia o texto dele. Spec que copia prompt vira cópia desatualizada no dia
seguinte à primeira edição.

### 3.6 Observabilidade

**O que é.** A capacidade de saber o que está acontecendo em produção sem
depender de o usuário relatar. [BASE, com mais peso em EXTERNO]

- **Logging estruturado.** Log em formato consistente e parseável (JSON, com
  campos fixos: rota, identificador de usuário, ação, resultado), não texto
  livre. Serve para filtrar e correlacionar depois. **Sem PII e sem dado
  sensível** no log.
- **Captura de erro com contexto.** Erro sem contexto (usuário, entrada,
  operação) só diz que algo quebrou. Com contexto, diz o quê e por quê.
- **Ausência de sinal não é sinal.** Antes de concluir que um evento não foi
  registrado, limpe **todos** os filtros do painel de monitoramento, inclusive os
  de status e período. Um item marcado como resolvido some da visão padrão e
  parece nunca ter existido.
- **Distinguir o que é do produto do que é ruído externo.** Relatórios
  automáticos (violação de CSP, erro de script) captam também conteúdo injetado
  por navegador e extensão. Filtre o ruído externo na entrada, para que o painel
  mostre só o que é acionável.

### 3.7 Entrega progressiva e feature flags

**O que é.** Um mecanismo para ligar e desligar funcionalidade em produção sem
novo deploy. [EXTERNO]

**Para que serve.** Separar o **deploy** (o código estar em produção) da
**entrega** (a funcionalidade estar disponível). Permite mergear cedo, liberar
para um subconjunto de usuários, e desligar rápido se algo der errado, sem
reverter código.

**Quando usar.** Funcionalidade arriscada, mudança que precisa de validação com
usuários reais, ou release coordenado com outra área. **Quando não.** Projeto
pequeno e mudança de baixo risco: a flag vira complexidade sem retorno, e flag
esquecida é dívida técnica que se acumula.

> Toda flag deve ter um plano de remoção. Flag permanente é ramificação
> permanente no código.

---

## 4. Segurança

Esta é a seção onde a separação [BASE] / [EXTERNO] mais importa. Referência
geral: **OWASP** (https://owasp.org), em especial o OWASP Top Ten
(https://owasp.org/www-project-top-ten/) e os Cheat Sheets
(https://cheatsheetseries.owasp.org).

### Sempre, em todo projeto [BASE]

- **Segredos só em variáveis de ambiente.** Nunca commitar chaves, senhas ou
  tokens. Em produção, usar o cofre de variáveis da plataforma; em
  desenvolvimento, um arquivo de ambiente local fora do versionamento.
- **Validação de entrada com schema.** Toda entrada (corpo de requisição,
  parâmetros) é validada contra um schema antes de qualquer lógica, barrando dado
  malformado na porta. Em JS/TS usamos o Zod; em outra stack, a biblioteca de
  validação equivalente.
- **Tratamento de erro sem vazar detalhe sensível** para o usuário final.
- **Menor privilégio.** Cada credencial recebe o menor escopo que resolve o
  problema. Chave de serviço que ignora regras de acesso só existe no servidor,
  nunca no cliente. Ferramenta de leitura (BI, relatório) usa usuário
  somente-leitura.
- **Segurança de dependências.** Auditoria periódica das dependências do projeto.
  Nem toda vulnerabilidade reportada se aplica ao seu uso; quando a decisão for
  **aceitar**, registre o racional na dívida técnica, com o vetor analisado. Sem
  esse registro, alguém "corrige" depois aplicando a sugestão automática da
  ferramenta, que às vezes é destrutiva.

### Quando há usuários externos ou exposição na web [EXTERNO]

- **RLS (Row Level Security).** Regras no banco que garantem que cada usuário só
  acessa os próprios dados. Serve para isolar dados por usuário na origem (no
  banco), não só na aplicação. Usar sempre que houver dados por usuário
  acessíveis a partir do cliente; no Supabase, ativar RLS por tabela. Referência:
  https://supabase.com/docs/guides/database/postgres/row-level-security
- **Autorização, não só autenticação.** Saber **quem** é o usuário não basta:
  toda operação precisa verificar se ele é dono do recurso que está tocando. Um
  endpoint que recebe um identificador no corpo da requisição e confia nele está
  autenticando sem autorizar. A verificação sai da **sessão**, nunca do payload.
- **CORS (Cross-Origin Resource Sharing).** Controle de quais origens podem
  chamar a API. Restringir às origens legítimas; nunca liberar tudo em produção.
- **Rate limiting.** Limitar requisições por IP ou usuário em endpoints públicos,
  para conter abuso, brute force e custo. Aplicar nos endpoints públicos
  sensíveis (login, criação de conta, geração de conteúdo). Vale tanto pela
  segurança quanto pelo **custo**: endpoint que dispara operação cara (chamada a
  modelo de IA, envio de e-mail, processamento pesado) precisa de limite mesmo
  sem risco de vazamento, porque o abuso ali gera fatura, não incidente.
- **Honeypot.** Um campo de formulário invisível ao usuário e visível ao bot: se
  vier preenchido, a submissão é descartada. Barato, invisível e sem atrito.
  Primeira linha contra automação simples, antes de partir para captcha.
- **reCAPTCHA ou desafio anti-bot.** Verificação de que há uma pessoa do outro
  lado. Cobre o bot que respeita o rate limit e ignora o honeypot. Tem custo:
  adiciona atrito ao usuário legítimo e introduz um terceiro recebendo dado do
  visitante (implicação de política de privacidade). Avaliar honeypot antes.
- **Verificação de webhooks (assinatura, HMAC ou segredo em header).** Webhooks
  de terceiros e chamadas entre serviços devem ser autenticados: segredo
  compartilhado em header, ou HMAC com comparação em tempo constante. Nunca
  confiar em um webhook sem verificar a origem, porque o endpoint é público por
  natureza.
- **Idempotência em operação que cria recurso caro.** Retry de rede, duplo clique
  e timeout com resposta perdida geram execução dupla. Onde a operação cobra,
  gera conteúdo caro ou cria recurso único, o cliente envia uma **chave de
  idempotência** e o servidor deduplica por ela.
- **Expiração de sessão.** Definir explicitamente o tempo de vida de sessão e de
  refresh, proporcional ao uso do produto. Sessão eterna é risco; sessão curta
  demais em produto de uso esporádico é fricção sem ganho de segurança.
- **Guardrails e sanitização para LLM.** Quando há um modelo de linguagem no
  fluxo: sanitizar texto livre, isolar dados externos em delimitadores, proteger
  a system message, e validar a resposta contra um schema. Defesa em camadas
  contra prompt injection.
- **Security headers e CSP** (Content Security Policy, HSTS, X-Frame-Options,
  etc.). Reduzem a superfície de ataque no navegador. Introduzir primeiro em modo
  somente-relato e depois passar a enforcing, mas **só depois** de revisar os
  relatórios e confirmar que nenhum recurso legítimo do próprio app está sendo
  bloqueado.
- **LGPD e privacidade.** Antes de coletar dado pessoal sensível (ex.: saúde),
  documentar a base legal e a Política de Privacidade. **A política vem antes da
  coleta**, não depois. Praticar **minimização**: dado sensível e PII não vão para
  ferramentas de analytics; o identificador pseudonimizado basta para funil e
  coorte, e o cruzamento com a identidade real acontece no seu banco, não na
  ferramenta de terceiro.
- **Consentimento versionado.** Se o produto pede aceite de termos ou política,
  guarde **qual versão** foi aceita e **quando**, não um booleano. Documentos
  mudam, e sem a versão não há prova de consentimento nem gatilho de re-aceite.

### Governança de segurança [EXTERNO]

Os itens acima são controles; esta é a prática de mantê-los vivos. Segurança não
é um checklist que se cumpre uma vez.

- **Auditoria periódica** de dependências e de configuração, com cadência
  definida (não "quando lembrar").
- **Checklist antes de expor algo novo**: endpoint público, integração,
  ferramenta com acesso ao banco.
- **Triagem do que as ferramentas apontam.** Análise automática gera muito achado
  já decidido conscientemente. Suprima na própria ferramenta o que está registrado
  na dívida técnica, senão o painel vira ruído e o achado novo se perde no meio
  dos conhecidos.
- **Revisão das premissas de risco.** Item adiado com justificativa condicional
  ("não urgente porque ainda não há usuários") precisa ser reavaliado quando a
  condição muda. Ver [2.7](#27-corte-de-versão).

---

## 5. Documentação e organização do repositório

Documentação não morre por falta de criação, morre por falta de manutenção. A
regra de ouro: toda tarefa que muda o estado do projeto atualiza o documento
certo, como parte do [Definition of Done](#23-definition-of-done). [BASE]

Dois princípios sustentam isso:

- **Uma fonte por fato.** Quando a mesma informação vive em dois documentos, uma
  das cópias envelhece silenciosamente. Escolha o documento dono e faça o outro
  **apontar** para ele, em vez de repetir.
- **Premissas envelhecem.** Justificativa condicional ("não urgente porque X",
  "fazer antes de Y") deixa de valer sem aviso. Revisar as condicionais é parte
  do corte de versão.

### 5.1 Nomes e estrutura do repositório

**Nome do repositório.** Usar **kebab-case**, com nomes **intuitivos e
descritivos** do que o projeto é. Bons exemplos: `erp-financeiro`,
`gestor-de-demandas`, `site-institucional`. Evitar nomes genéricos ou opacos
(`projeto1`, `app-novo`) que não dizem nada a quem chega.

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
    ├── commands/
    │   └── wrap.md         # ritual de fechamento de tarefa
    └── rules/              # convenções carregadas pelo agente de IA
        ├── ways-of-working.md
        ├── code-conventions.md
        └── security.md
```

> A pasta `.claude/` e o `CLAUDE.md` são o exemplo concreto para o agente de IA
> que recomendamos; o nome exato depende da ferramenta adotada (ver
> [seção 6](#6-ferramentas-recomendadas)). O importante é existir um lugar
> versionado para regras, comandos e contexto da IA.

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
ele se **move** para a seção de concluídos (um registro enxuto do que foi
entregue). **Onde fica.** `docs/ROADMAP.md`.

> A seção de próximos passos precisa responder "o que fazer agora". Se os itens
> concluídos ficam nela riscados, ela deixa de responder, e daí vem a regra de
> mover. Item sem gatilho declarado ("quando houver volume", "a partir de X
> usuários") vira gaveta de "algum dia": ao registrar algo que depende de
> condição, escreva a condição.

#### `TECH-DEBT.md`
**O que é.** O registro de gambiarras, adiamentos conscientes e riscos
conhecidos, cada um com uma severidade (ex.: crítico, alto, médio, baixo).
**Para que serve.** Tornar a dívida **visível e rastreável**, em vez de
esquecida. Toda gambiarra criada se registra aqui, e idealmente deixa um marcador
`TODO` no código apontando para ela. **Quando atualizar.** Ao criar uma dívida
(registrar) e ao resolvê-la (marcar como resolvida, **sem apagar**: o histórico
tem valor). **Onde fica.** `docs/TECH-DEBT.md`.

> Diferente do roadmap, aqui o item resolvido **fica onde está**, apenas marcado.
> O motivo: a causa estrutural de uma dívida costuma continuar viva ao lado dela,
> e separar os dois perde a conexão. O roadmap é plano (o que fazer); a dívida é
> inventário de risco (o que existe).

#### `RELEASE-NOTES.md`
**O que é.** As notas de versão em **linguagem simples**, voltadas ao usuário, e
não ao desenvolvedor. **Para que serve.** Comunicar o que mudou de forma
acessível, no corte de cada versão. É sempre **destilado do changelog**, nunca
escrito do zero (o changelog é a fonte técnica; as release notes são o recorte
amigável). **Quando atualizar.** No corte de uma versão. **Onde fica.** Raiz do
repo. Detalhamento da régua de recorte em
[5.3](#53-changelog-versionamento-e-releases).

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

> Este documento descreve o que é **estável** (produto, papéis, arquitetura de
> memória). O estado vivo do projeto fica no roadmap. Não duplicar estado mutável
> aqui: envelhece e vira mentira.

#### Arquivo de contexto da IA (ex.: `CLAUDE.md`)
**O que é.** O mapa do repositório e as regras de base que o agente de IA carrega
sempre (estrutura de pastas, convenções inegociáveis, armadilhas conhecidas).
**Para que serve.** Garantir que o agente trabalhe alinhado ao projeto desde o
primeiro prompt. **Quando atualizar.** Ao mudar a estrutura do repo ou uma regra
de base. **Onde fica.** Raiz do repo (o nome depende da ferramenta de IA).

> Manter curto. Arquivo carregado em toda sessão que cresce demais deixa de ser
> lido com atenção; o detalhe vai para as rules, carregadas por escopo.

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

#### As RELEASE-NOTES e a régua de recorte

**O que é.** O recorte do changelog em linguagem de usuário, publicado no corte
de cada versão.

**A régua:** entra **só o que o usuário percebe na experiência**. Correção
interna (schema, validação, observabilidade, refactor, configuração) fica apenas
no changelog, que é o eixo técnico e mantém a rastreabilidade completa.

Consequências dessa régua:

- Uma versão só de correção interna **pode não gerar seção nova** nas release
  notes, e isso é aceitável.
- As notas nunca são escritas do zero: são sempre recorte do changelog. Se algo
  aparece nas notas e não no changelog, o processo foi invertido.

#### As duas formas de versionar (escolha por caso)

Mantemos **as duas** à disposição da equipe; a escolha depende do tipo de
projeto.

**A) SemVer (Semantic Versioning):** `MAJOR.MINOR.PATCH` (ex.: `2.4.1`). O
gatilho é **compatibilidade técnica**:
- **MAJOR**: mudança que **quebra compatibilidade** para quem consome o código.
- **MINOR**: funcionalidade nova mantendo compatibilidade.
- **PATCH**: correção de bug mantendo compatibilidade.
Referência: https://semver.org

**B) Marco de produto:** `MARCO.LANÇAMENTO.CORREÇÃO`. O gatilho é a **percepção
do usuário**, não compatibilidade:
- **MARCO**: redefinição do produto ("o produto mudou"). Raro.
- **LANÇAMENTO**: funcionalidade nova que o usuário percebe (bloco `Added` ou
  `Changed`).
- **CORREÇÃO**: bug fix, ajuste, performance, sem capacidade nova (bloco `Fixed`
  ou `Security`).

**Quando usar A ou B.** Use **SemVer (A)** quando o software é **consumido por
outros desenvolvedores**: bibliotecas, pacotes, APIs públicas, SDKs. Ali o número
é um contrato técnico de compatibilidade. Use **marco de produto (B)** quando o
software é um **produto de usuário final** (web app, app interno): ninguém
depende do seu código como dependência, então o número que comunica marco de
produto é mais útil. Um mesmo projeto pode usar B no produto e A em uma API
pública que ele venha a expor.

#### Tags e releases no repositório

**O que é.** No corte de uma versão, cria-se uma **tag git** (ex.: `v1.2.0`)
apontando para aquele commit, e publica-se um **release** no GitHub com as notas
daquela versão. **Para que serve.** Marcar um ponto fixo e nomeado na história do
projeto, fácil de localizar, comparar (`git diff v1.1.0..v1.2.0`) e comunicar.
**Quando criar.** No fechamento de uma versão, não a cada commit de correção
pequena.

O corpo do release é o recorte das **release notes** (linguagem simples,
escaneável na página de releases), com um link para o changelog para quem quiser
o detalhe técnico. Se o corte não gerou seção nas notas, o corpo é uma linha
apontando para o changelog, nunca vazio.

> **Atenção: tag não é deploy.** Dependendo da plataforma, a publicação em
> produção acontece por push na branch principal (deploy automático) ou por
> comando de CLI, não a partir da tag. Aqui, a tag e o release são um **marco de
> documentação e comunicação**, não o gatilho de publicação. Não confundir os
> dois, e deixar explícito no README de cada projeto qual é o gatilho real.

> O ritual completo, em ordem, está em [2.7](#27-corte-de-versão).

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
ficam.** `docs/adr/`, numerados em sequência, a partir de um `TEMPLATE.md`, com
um `README.md` de índice. Referência: https://adr.github.io

> Registrar as **alternativas rejeitadas** é metade do valor de um ADR. Sem isso,
> alguém propõe de novo a opção que já foi analisada e descartada.

### 5.5 Specs (Spec-Driven Development)

**O que é.** O contrato de uma feature, escrito **antes** do código: o que vai
ser construído, por quê, o escopo (o que entra e o que não entra), e os critérios
de aceite. Complementa a [seção 2.4](#24-spec-driven-development-sdd). **Para que
serve.** Alinhar todo mundo (pessoas e agentes) sobre o que será feito antes de
gastar esforço, evitando premissas divergentes e retrabalho. **Quando escrever:**
features de peso (telas complexas, mudanças de schema, refatorações grandes,
integrações). **Quando não:** bug fix e ajuste pequeno. **Ciclo de vida:**
**Proposta** (rascunho em discussão), depois **Aprovada** (contrato fechado,
implementa-se contra ela), depois **Implementada** (feita, mantida para
referência, linkando os ADRs que saíram dela). **Onde ficam.** `specs/`,
numeradas em sequência, a partir de um `TEMPLATE.md`, com um `README.md` de
índice.

Boas práticas que evitam que a spec apodreça:

- **Perguntas abertas explícitas** enquanto o status é Proposta. Se algo não foi
  decidido, escreva a pergunta em vez de assumir uma resposta.
- **Ordem obrigatória**, quando os passos têm dependência real entre si, e o
  porquê da ordem, que costuma ser a informação que se perde.
- **Pendências não escondidas atrás do "Implementada".** O que não foi verificado
  fica registrado como pendência, não some no fechamento.

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

#### As rules se auto-alimentam

O [relatório de fechamento](#relatório-de-fechamento-wrap) inclui os **candidatos
a regra**: aprendizado descoberto durante a execução (comportamento de
ferramenta, mudança de API de biblioteca, pegadinha do ambiente, convenção que
evita um erro).

Cabe a quem revisa aplicar o **filtro**, e ele tem dois critérios:

- **Durável**: vou precisar disso de novo?
- **Específico do projeto**: é conhecimento deste projeto, e não algo que
  qualquer dev reencontra na documentação oficial?

Só o que passa nos dois vira rule. **Destilar, não despejar**: rule inflada deixa
de ser lida, e aí nenhuma regra é seguida. Colocação: encaixe no arquivo do
domínio correspondente; crie um arquivo novo só quando surgir um domínio inteiro
recorrente, nunca para uma regra órfã.

---

## 6. Ferramentas recomendadas

Esta seção tem duas naturezas de item, e a diferença importa:

- **Uso geral**: ferramentas que independem da linguagem do projeto e servem em
  qualquer stack. Banco, deploy, monitoramento e análise entram aqui.
- **Dependentes da stack**: ferramentas que só existem em um ecossistema
  específico. Listamos as que usamos, mas **a escolha de linguagem, framework e
  bibliotecas é de cada projeto** e deve atender à necessidade dele. Um projeto
  em outra stack segue a mesma metodologia com as ferramentas equivalentes do seu
  ecossistema.

Cada item traz o que é, para que serve, quando faz sentido e o link da
documentação.

### Uso geral

#### Cloudflare Workers, deploy [BASE]
Plataforma de execução serverless na borda da Cloudflare. Serve para hospedar e
rodar aplicações próximas do usuário, com escala automática. É o padrão de deploy
da DPX porque os domínios da empresa estão centralizados na Cloudflare, o que
simplifica DNS, certificados e gestão. Referência:
https://developers.cloudflare.com/workers/

#### Wrangler, CLI de deploy [BASE]
A ferramenta de linha de comando da Cloudflare para desenvolver, testar e
publicar Workers. Serve para fazer o build e o deploy a partir do ambiente local.
É como, na prática, uma aplicação chega à Cloudflare quando o deploy é manual.
Referência: https://developers.cloudflare.com/workers/wrangler/

#### Cloudflare Builds, deploy automático [BASE]
Integração da Cloudflare com o repositório: a cada push, o build e o deploy
acontecem automaticamente, sem comando manual. Serve para eliminar o passo de
publicação manual, e o risco de esquecer de publicar ou de publicar de uma
máquina com estado local diferente. Combina com a
[estrutura de branches](#26-estrutura-de-branches): a branch principal publica em
produção, e as demais podem gerar ambientes de preview para verificação antes do
merge. Referência: https://developers.cloudflare.com/workers/ci-cd/builds/

> Com deploy automático, o gatilho de publicação é o **push**, não a tag. Ver a
> nota em [5.3](#tags-e-releases-no-repositório).

#### Supabase, banco de dados e autenticação [BASE]
Plataforma sobre Postgres que entrega banco de dados, autenticação, RLS e uma API
pronta. Serve como a camada de dados e de login dos projetos. É o padrão de banco
da DPX. Tem CLI para migrations e painel para administração. Referência:
https://supabase.com/docs

#### Sentry, monitoramento de erros [EXTERNO] (útil também interno)
Serviço de captura e agregação de erros em tempo real, com stack trace, contexto
e alertas. Serve para descobrir e diagnosticar falhas em produção sem depender do
usuário relatar. É crítico em produtos externos, onde um erro silencioso afeta
usuários reais; em projeto interno, ajuda mas é menos urgente. Tem SDK para a
maioria das stacks. Referência: https://docs.sentry.io

#### PostHog, analytics de produto [EXTERNO]
Plataforma de analytics de produto: eventos, funis de conversão e comportamento
de uso. Serve para entender como as pessoas usam o produto e onde abandonam. Faz
sentido sobretudo em produtos externos, onde há uma base de usuários para medir;
em ferramenta interna, raramente se justifica. Referência:
https://posthog.com/docs

> **Client-side ou server-side?** A captura no **cliente** (navegador) é rica em
> contexto de navegação (origem, campanha, dispositivo, sequência de telas) mas é
> **bloqueada por ad blockers**, com perda relevante de eventos. A captura no
> **servidor** não é bloqueada e é confiável, mas perde o contexto de navegação e
> exige enviar cada evento explicitamente. O critério prático é o **valor do
> evento**: conversão, pagamento e assinatura vão pelo servidor, porque não podem
> se perder; comportamento de navegação vai pelo cliente, onde o contexto importa
> mais que a completude. Um proxy reverso no seu próprio domínio recupera parte
> dos eventos de cliente bloqueados, e é uma alternativa ao server-side para os
> eventos que não justificam a instrumentação no backend.
>
> **Atenção à minimização**: dado sensível e PII não devem ser enviados para o
> analytics, em nenhum dos dois modos (ver [seção 4](#4-segurança)).

#### Upstash Redis, rate limiting e cache [EXTERNO]
Redis serverless, cobrado por uso, que usamos sobretudo para **rate limiting** de
endpoints públicos. Serve para conter abuso, brute force e custo em rotas
expostas na internet. Faz sentido em projetos externos; em interno, normalmente
não. Referência: https://upstash.com/docs/redis

#### SonarCloud, qualidade e manutenibilidade [BASE]
Plataforma de análise contínua de qualidade de código: duplicação, complexidade,
code smells, cobertura e pontos de segurança. Serve para acompanhar a saúde do
código ao longo do tempo, não só o erro pontual. Suporta várias linguagens e
**roda em pull request**, o que a torna a peça que transforma a revisão em portão
automático antes do merge (ver [2.6](#26-estrutura-de-branches)). Referência:
https://docs.sonarsource.com

#### Aikido, segurança do repositório [EXTERNO]
Plataforma de varredura de segurança sobre o código e as dependências: CVEs em
bibliotecas, segredos expostos no repositório, configuração insegura, padrões de
código com risco. Serve como uma segunda camada além da revisão humana. Roda em
varredura periódica, então funciona como **relatório recorrente**, não como
portão de merge. Referência: https://apidocs.aikido.dev

> Vale para SonarCloud e Aikido: ferramenta de análise aponta muita coisa já
> decidida conscientemente. Suprima na própria ferramenta o que está registrado
> na dívida técnica, com o racional, senão o painel vira ruído e o achado novo se
> perde no meio dos conhecidos.

#### Grafana e Prometheus, métricas de infraestrutura [EXTERNO]
Prometheus coleta e armazena métricas em série temporal; Grafana as visualiza em
painéis e dispara alertas. Servem para monitorar **infraestrutura própria**: uso
de CPU e memória, latência, filas, disponibilidade de serviço. Fazem sentido
quando existe servidor ou serviço self-hosted sob nossa responsabilidade (VPS,
container, automação hospedada). Em stack totalmente gerenciada (serverless mais
banco gerenciado), a métrica de infraestrutura vem do painel do provedor e essa
dupla costuma ser complexidade desnecessária. Não confundir com Sentry (erro de
aplicação) nem PostHog (comportamento de usuário): são três perguntas diferentes.
Referências: https://grafana.com/docs e https://prometheus.io/docs

### Dependentes da stack

As ferramentas abaixo são as que usamos em projetos JavaScript e TypeScript. Em
outra stack, valem os equivalentes do ecossistema, e o que importa é cumprir o
princípio, não usar a mesma ferramenta.

#### Zod, validação de schema
Biblioteca de validação de schema para TypeScript, com inferência de tipos a
partir do schema. Cumpre o princípio de "validação de entrada" da
[seção 4](#4-segurança), barrando dado malformado na porta. Referência:
https://zod.dev

#### Prettier, formatação de código
Formatador automático para JavaScript, TypeScript e formatos adjacentes. Cumpre o
princípio de formatação automática da
[seção 3.4](#34-lint-formatação-e-qualidade). Referência: https://prettier.io

#### ESLint, análise estática
Ferramenta de análise estática para JavaScript e TypeScript, que aponta padrões
problemáticos e bugs prováveis antes de rodar o código. Cumpre o princípio de
análise estática da [seção 3.4](#34-lint-formatação-e-qualidade). Referência:
https://eslint.org

---

## 7. Glossário e decisões em aberto

### Glossário

Termos citados ao longo do documento **sem explicação no próprio texto**. O que
já é definido onde aparece não se repete aqui.

- **PII** (*Personally Identifiable Information*): dado que identifica uma pessoa
  diretamente, como nome, e-mail, telefone, CPF e endereço.
- **PR** (*pull request*): proposta de incorporar as mudanças de uma branch em
  outra, aberta para revisão antes do merge.
- **Rebase**: reescrever a base de uma branch sobre outra, reaplicando os
  commits. Muda o histórico, por isso é perigoso em branch compartilhada.
- **MVP** (*Minimum Viable Product*): a menor versão do produto que já entrega
  valor e permite validar a ideia com usuários.
- **LLM** (*Large Language Model*): modelo de linguagem de grande porte, como os
  que usamos nos agentes de IA.
- **System message**: a instrução base dada ao modelo, que define seu papel e
  suas regras, separada da mensagem do usuário.
- **Prompt injection**: ataque em que texto malicioso vindo do usuário (ou de um
  dado externo) é interpretado pelo modelo como instrução, alterando seu
  comportamento.
- **HMAC**: assinatura criptográfica calculada com um segredo compartilhado,
  usada para verificar que uma mensagem veio de quem diz ter vindo e não foi
  alterada.
- **Security headers**: cabeçalhos HTTP que restringem o que o navegador pode
  fazer com a página. **CSP** limita de onde scripts, estilos e fontes podem ser
  carregados; **HSTS** obriga o uso de HTTPS; **X-Frame-Options** impede que a
  página seja embutida em um iframe de terceiro.
- **CVE** (*Common Vulnerabilities and Exposures*): identificador público de uma
  vulnerabilidade conhecida (ex.: `CVE-2024-1234`), usado pelas ferramentas de
  auditoria de dependências.
- **Code smell**: trecho de código que funciona mas indica problema de desenho,
  como duplicação, função gigante ou aninhamento excessivo. Não é bug; é sinal de
  que vai virar um.
- **Serverless**: modelo de execução em que o código roda sob demanda em
  infraestrutura gerenciada pelo provedor, sem servidor para manter, com escala
  automática e cobrança por uso.
- **Self-hosted**: serviço que rodamos em infraestrutura própria, em vez de
  contratar como serviço gerenciado. Dá controle e custa manutenção.
- **Proxy reverso**: servidor intermediário que recebe as requisições em nome de
  outro serviço. No contexto de analytics, serve para que os eventos saiam pelo
  próprio domínio, escapando de bloqueadores.
- **Pseudonimizado**: dado em que a identidade direta foi substituída por um
  identificador (ex.: um UUID), de modo que só é possível chegar à pessoa
  cruzando com outra base sob seu controle.
- **LGPD**: Lei Geral de Proteção de Dados (Lei 13.709/2018), que regula o
  tratamento de dados pessoais no Brasil.
- **Dump**: cópia do estado atual de um banco de dados (estrutura ou conteúdo),
  em arquivo, que permite recriá-lo em outro lugar.
- **kebab-case**: palavras minúsculas separadas por hífen (ex.:
  `gestor-de-demandas`).

### Decisões em aberto

Itens que a DPX ainda não fechou e vai detalhar mais adiante. Ficam listados
aqui, de forma honesta, em vez de fixar um padrão prematuro:

- **Estratégia de testes.** A ferramenta e o conceito estão indicados (testes E2E
  automatizados para fluxos críticos), mas a estratégia não: o que se testa, que
  cobertura se persegue, quando os testes rodam, e em que ponto passam a ser
  bloqueantes. Por ora, a regra continua sendo testar todos os cenários possíveis
  do que está sendo desenvolvido antes de ir para produção.
- **Regras de aprovação de PR.** A estrutura de branches está definida
  ([2.6](#26-estrutura-de-branches)), mas quem aprova, quantas aprovações são
  necessárias e quais verificações são bloqueantes ainda dependem do tamanho de
  cada time.

---

*Este documento é vivo. Ao mudar uma prática da equipe, atualize-o e registre a
data no topo.*
