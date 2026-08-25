# Especificação de Requisitos e Arquitetura — Portal Nacional de Remessas de Dados de Precatórios

> **Documento normativo para o agente de código.** Palavras‑chave em maiúsculas (`DEVE`, `NÃO DEVE`, `OBRIGATÓRIO`, `PROIBIDO`) têm força de requisito e não são opcionais. Onde houver conflito entre este documento e conhecimento prévio, este documento prevalece.

- **Versão:** 1.0 (refinada a partir do épico e das 28 HUs — projeto `SISPREQ`)
- **Público‑alvo:** Agente de código (planejamento + execução) e time técnico
- **Escopo deste documento:** MVP operacional + fronteiras explícitas de fora‑de‑escopo

---

## 1. Visão Geral do Produto

Aplicação web (Frontend + Backend) que constitui o **Portal Nacional de Remessas de Dados de Precatórios**, permitindo que **servidores de tribunais estaduais e federais** enviem, de forma segura, arquivos **CSV** com dados de precatórios para um **bucket AWS S3**, e acompanhem o ciclo de vida de cada envio.

O portal é uma **camada de gestão operacional**. Ele **orquestra o ciclo de vida das remessas** (criação, upload, confirmação, acompanhamento, cancelamento) e **NÃO executa processamento de dados (ETL)**. O processamento é realizado de forma **assíncrona por plataforma externa (AWS Glue)**, fora do escopo desta aplicação.

### 1.1 Princípios arquiteturais inegociáveis
1. **O conteúdo dos arquivos NÃO trafega pelo backend.** O upload ocorre **diretamente do navegador para o S3 via URL pré-assinada**.
2. **O portal não recalcula nem produz resultados de processamento.** Ele apenas registra, exibe e dá visibilidade.
3. **Isolamento horizontal por tribunal.** Um usuário de um tribunal **NUNCA** acessa dados de outro tribunal.
4. **Segurança por padrão.** Credenciais nunca no código; validação sempre no backend; tráfego só por HTTPS.

---

## 2. Registro de Decisões Arquiteturais (Decision Log)

Estas decisões resolvem divergências entre o rascunho inicial e o épico/HUs oficiais. **O agente DEVE seguir a coluna "Decisão".** Caso o Product Owner queira reverter alguma, deve fazê-lo antes do início da implementação.

| # | Tema | Rascunho inicial | Épico/HUs oficiais | **Decisão adotada (MVP)** |
|---|------|------------------|--------------------|---------------------------|
| D1 | **Fluxo de upload** | Backend recebe `multipart/form-data` e reenvia ao S3 | Upload **direto ao S3 via URL pré-assinada**; arquivos não trafegam pelo backend | **URL pré-assinada (direto navegador→S3).** O backend nunca recebe o conteúdo do arquivo. |
| D2 | **Autenticação** | E-mail/senha + JWT local | SSO PDPJ-Br (OIDC/OAuth2) | **E-mail/senha + JWT no MVP**, encapsulado atrás de uma interface `AuthenticationProvider` para permitir plugar OIDC/PDPJ-Br sem refatorar. |
| D3 | **Multi-tenancy** | Não previsto no MVP | Isolamento por tribunal é requisito de segurança | **Incluído no MVP.** Todo acesso a Remessa é filtrado pelo `tribunal_id` do usuário autenticado, validado **no backend**. |
| D4 | **Versão do Java** | Java 17 | Java 21 | **Java 21.** |
| D5 | **Modelo de domínio** | "Upload genérico" | Domínio orientado a **Remessa** | **Remessa** é a entidade central (competência, versão, checksum, status). |
| D6 | **Integração ETL** | — | Processamento assíncrono via Glue | **Fora do MVP.** Após a confirmação do upload, a remessa transita para `AGUARDANDO_PROCESSAMENTO` e o fluxo do portal encerra. Um endpoint administrativo de simulação de status é fornecido apenas para teste. |

---

## 3. Stack Tecnológico

| Camada | Tecnologia |
|--------|-----------|
| **Frontend** | Angular (versão LTS atual), Angular Material, UIKit, HTML5 |
| **Backend** | **Java 21**, Spring Boot (Web, Security, Data JPA, Validation) |
| **Documentação de API** | SpringDoc OpenAPI (Swagger UI) — **OBRIGATÓRIO** para todos os endpoints |
| **Banco de dados** | PostgreSQL, com **Flyway** para versionamento do schema |
| **Armazenamento de arquivos** | AWS S3 (bucket **privado**, sem acesso público) via AWS SDK for Java v2 |
| **Infraestrutura local** | Docker: `Dockerfile` para cada aplicação + `docker-compose.yml` para orquestração local (banco + serviços) |

### 3.1 Configuração e segredos
- As credenciais e configurações da AWS (chave, região, nome do bucket) **DEVEM** ser injetadas **exclusivamente via variáveis de ambiente**.
- **PROIBIDO** hardcode de qualquer credencial, segredo ou chave no código, em arquivos de configuração versionados ou em logs.
- O `docker-compose.yml` **DEVE** ler segredos de variáveis de ambiente / arquivo `.env` **não versionado** (fornecer `.env.example` como modelo).

---

## 4. Arquitetura da Solução

### 4.1 Fluxo de upload (canônico — Decisão D1)

```
┌───────────┐   1. POST /remessas                 ┌───────────┐
│           │ ──────────────────────────────────▶ │           │
│  Angular  │   2. POST /remessas/{id}/upload-url  │  Spring   │
│ (browser) │ ◀────────── URL pré-assinada ──────  │  Boot     │
│           │                                       │  Backend  │
│           │   3. PUT arquivo (bytes)              └─────┬─────┘
│           │ ─────────────────────────────┐             │ grava metadados
│           │                              ▼             │ (nunca os bytes)
│           │                        ┌───────────┐       │
│           │ ◀───── ETag ────────── │  AWS S3   │ ◀─────┘
│           │                        │ (privado) │
│           │   4. POST /remessas/{id}/confirmar   │
│           │ ────────────────────────────────────▶ (HEAD no S3, valida
└───────────┘                                        tamanho, registra ETag,
                                                      status → AGUARDANDO_PROCESSAMENTO)
```

**Regras do fluxo:**
1. `POST /remessas` — cria a remessa em status `RASCUNHO`/`AGUARDANDO_UPLOAD`. O `tribunal_id` **DEVE** vir do usuário autenticado, **nunca** do payload do cliente.
2. `POST /remessas/{id}/upload-url` — gera **URL pré-assinada de PUT** com: método restrito a `PUT`, `Content-Type` restrito a `text/csv`, **limite de tamanho (50 MB)**, expiração curta (ex.: 5–15 min), e **chave (key) do objeto controlada pelo backend**.
3. O navegador faz `PUT` **direto ao S3**. O backend **não** intermedia os bytes.
4. `POST /remessas/{id}/confirmar` — o backend faz `HEAD` no objeto, valida existência e tamanho, **registra o ETag/checksum**, atualiza o status de forma **transacional** e é **idempotente** (confirmação duplicada não gera efeitos colaterais).

**Estrutura da key no S3 (definida pelo backend):**
```
tribunal/{tribunal_id}/competencia/{AAAA-MM}/remessa/{remessa_id}/v{versao}/{uuid}-{timestamp}.csv
```
- O **nome original informado pelo usuário NÃO controla o caminho físico**; é armazenado apenas como metadado (`nome_arquivo_original`).
- O bucket **NÃO DEVE** ser público. É **OBRIGATÓRIO** configurar **CORS** no bucket para permitir `PUT` apenas a partir da origem do portal.

### 4.2 Arquitetura em camadas (Backend)
`Controller` → `Service` → `Repository`. Mapeamento **manual e limpo** entre Entidades e DTOs (nenhuma entidade JPA exposta na API). Tratamento de erros centralizado via `@ControllerAdvice`.

### 4.3 Arquitetura Frontend
Chamadas de API isoladas em serviços `@Injectable`. Estado de componentes gerenciado de forma limpa. Interceptor HTTP para anexar o JWT. Angular Material + UIKit conforme padrão visual.

---

## 5. Modelo de Domínio (Dados)

> O schema **DEVE** ser criado e versionado via **Flyway**. Os campos marcados como *(forward-compatible)* entram no schema desde o MVP para evitar migrations disruptivas futuras, mesmo que não sejam totalmente exercitados agora.

### 5.1 `Tribunal`
| Campo | Tipo | Regras |
|-------|------|--------|
| `id` | UUID (PK) | |
| `codigo` | String | **único** |
| `sigla` | String | **único** |
| `nome` | String | |
| `segmento` | String | Estadual/Federal |
| `situacao` | Enum (`ATIVO`/`INATIVO`) | Tribunais inativos **NÃO** criam remessas |
| `identificador_sso` | String | *(forward-compatible; nullable no MVP)* |
| `criado_em` / `atualizado_em` | Timestamp | |

> Exclusão física de tribunal **com histórico** é **PROIBIDA** (usar situação/soft state).

### 5.2 `User`
| Campo | Tipo | Regras |
|-------|------|--------|
| `id` | UUID (PK) | |
| `nome` | String | |
| `email` | String | **único** |
| `senha` | String | Hash **BCrypt** — nunca em texto puro, nunca em logs |
| `tribunal_id` | UUID (FK → Tribunal) | Obrigatório para `ROLE_USER` |
| `role` | Enum | Ver seção 6 |
| `ativo` | Boolean | |
| `criado_em` | Timestamp | |

### 5.3 `Remessa` (entidade central)
| Campo | Tipo | Regras |
|-------|------|--------|
| `id` | UUID (PK) | Identificador único |
| `tribunal_id` | UUID (FK) | Do usuário autenticado |
| `competencia` | String `AAAA-MM` | Validação de formato |
| `versao` | Integer | Inicia em 1 |
| `remessa_pai_id` | UUID (FK, nullable) | *(forward-compatible: versionamento/reenvio)* |
| `layout_id` | UUID (nullable) | *(forward-compatible: versionamento de layout)* |
| `nome_arquivo_original` | String | Metadado; não controla a key do S3 |
| `object_key` | String | Caminho físico no S3 (gerado pelo backend) |
| `tamanho_bytes` | Long | Validado na confirmação |
| `checksum_etag` | String | Registrado na confirmação |
| `status` | Enum | Ver máquina de estados (5.4) |
| `criado_por` | UUID (FK → User) | |
| `criado_em` | Timestamp | |
| `submetido_em` | Timestamp (nullable) | |
| `cancelado_em` / `motivo_cancelamento` | Timestamp / String (nullable) | |
| `correlacao_id` | UUID | Para rastreabilidade/eventos futuros |

### 5.4 Máquina de estados da Remessa (MVP)
Status suportados no MVP:
```
RASCUNHO → AGUARDANDO_UPLOAD → ARQUIVO_RECEBIDO → AGUARDANDO_PROCESSAMENTO
   │                                                        
   └──────────────► CANCELADA (a partir de estados pré-processamento)
```
- Status **pós-processamento** (`EM_PROCESSAMENTO`, `PROCESSADA_COM_SUCESSO`, `PROCESSADA_COM_REJEICOES`, `REJEITADA`, `FALHA_TECNICA`) **DEVEM existir no enum** *(forward-compatible)*, mas **não são transicionados pelo portal no MVP** (dependem da ETL — fora de escopo).
- Transições inválidas **DEVEM** ser rejeitadas com HTTP 409/422 e registradas.

### 5.5 `AuditLog`
| Campo | Tipo |
|-------|------|
| `id` | UUID (PK) |
| `usuario_id` | UUID |
| `acao` | String (ex.: `CRIAR_REMESSA`, `GERAR_URL`, `CONFIRMAR_UPLOAD`, `CANCELAR`, `LOGIN`) |
| `recurso_tipo` / `recurso_id` | String / UUID |
| `tribunal_id` | UUID (quando aplicável) |
| `data_hora` | Timestamp (padrão único, UTC) |
| `detalhes` | JSON/Text (sem dados sensíveis) |

> Registros de auditoria **NÃO DEVEM** ser alteráveis por usuários comuns. Credenciais e dados pessoais **PROIBIDOS** nos logs.

---

## 6. Perfis de Acesso (RBAC) e Multi-tenancy

### 6.1 Perfis no MVP
| Perfil | Permissões |
|--------|-----------|
| `ROLE_USER` (Servidor de tribunal) | Login; criar/consultar/cancelar remessas **do próprio tribunal**; gerar URL e confirmar upload **do próprio tribunal**; consultar histórico **do próprio tribunal**. |
| `ROLE_ADMIN` (Administrador) | Login; listar/criar usuários; gerenciar tribunais (mínimo necessário para vincular usuários); consultar remessas em nível nacional. **NÃO** cria remessas por si só. |

> Perfis adicionais do épico (Gestor de tribunal, Operador nacional, Auditor, Suporte) são **fora de escopo do MVP**, mas o enum de roles **DEVE** ser extensível.

### 6.2 Regras de multi-tenancy (OBRIGATÓRIAS — Decisão D3)
- Toda consulta/ação sobre `Remessa` **DEVE** ser filtrada pelo `tribunal_id` do usuário autenticado, **no backend**.
- Um `ROLE_USER` **NÃO DEVE**, em hipótese alguma, acessar remessas de outro tribunal (proteção contra acesso horizontal).
- A interface **DEVE** ocultar ações não permitidas, mas a **autorização real DEVE ser aplicada no backend** (o backend não confia no frontend).
- Tentativas de acesso indevido **DEVEM** ser rejeitadas (HTTP 403) e **auditadas**.
- Princípio do **menor privilégio**.

---

## 7. Escopo do MVP (Foco de execução atual)

### 7.1 Módulo de Autenticação e Segurança
- Login via **e-mail e senha**; geração e validação de **JWT** no backend.
- **Abstração `AuthenticationProvider`** (Decisão D2): a implementação local (e-mail/senha) é uma das implementações; a arquitetura **DEVE** permitir plugar OIDC/PDPJ-Br futuramente **sem alterar controllers/services**.
- Frontend armazena o JWT de forma segura e o envia via **interceptor HTTP** no header `Authorization: Bearer <token>` em rotas protegidas.
- Tokens **NÃO DEVEM** ser gravados em logs.

**Endpoints:**
| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/auth/login` | Autentica e retorna JWT |
| `POST` | `/auth/logout` | Encerra a sessão (invalidação conforme estratégia adotada) |
| `GET` | `/auth/me` | Dados do usuário autenticado |

### 7.2 Módulo de Gestão de Usuários e Tribunais (Apenas Admin)
| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/admin/usuarios` | Lista usuários (ID, Nome, Email, Tribunal, Role) |
| `POST` | `/admin/usuarios` | Cria novo `ROLE_USER` (vinculado a um tribunal). Sem autocadastro no MVP. |
| `GET` | `/admin/tribunais` | Lista tribunais |
| `POST` | `/admin/tribunais` | Cadastra tribunal (mínimo: código, sigla, nome, segmento, situação) |

### 7.3 Módulo de Remessas
| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/remessas` | Cria remessa (tribunal do usuário autenticado; competência `AAAA-MM`) |
| `POST` | `/remessas/{id}/upload-url` | Gera URL pré-assinada de PUT (Content-Type `text/csv`, ≤ 50MB, expiração curta) |
| `POST` | `/remessas/{id}/confirmar` | Confirma upload: HEAD no S3, valida tamanho, registra ETag, transita status. **Idempotente.** |
| `POST` | `/remessas/{id}/cancelar` | Cancela remessa em estado pré-processamento (exige confirmação e motivo) |
| `GET` | `/remessas` | Lista/histórico paginado, com filtros (ver 7.4), **restrito ao tribunal do usuário** |
| `GET` | `/remessas/{id}` | Detalhe/acompanhamento de uma remessa |

**Regras de upload (MVP):**
- Extensão restrita a **`.csv`**; tamanho máximo **50 MB** (validado no frontend **e** reforçado na URL pré-assinada e na confirmação).
- Frontend **DEVE** exibir progresso, permitir cancelamento antes da conclusão, tratar expiração da URL (permitir gerar nova) e exibir mensagens claras de sucesso/erro.
- Falha no upload **NÃO DEVE** confirmar a remessa.

### 7.4 Histórico e filtros (MVP)
Filtros combináveis: `competencia`, `periodo_envio`, `status`, `versao`, `nome_arquivo`, `id_remessa`. Consulta **paginada**, ordenação padrão pelas mais recentes. `ROLE_USER` vê apenas seu tribunal; `ROLE_ADMIN` pode filtrar por tribunal.

### 7.5 Cancelamento (MVP)
- Apenas estados pré-processamento (`RASCUNHO`, `AGUARDANDO_UPLOAD`, `ARQUIVO_RECEBIDO`, `AGUARDANDO_PROCESSAMENTO`).
- Exige confirmação; registra usuário, data e motivo; **preserva histórico** (sem exclusão física).

### 7.6 Auditoria básica (MVP)
Registro obrigatório dos eventos: `LOGIN`, `CRIAR_REMESSA`, `GERAR_URL`, `CONFIRMAR_UPLOAD`, `CANCELAR`, `CRIAR_USUARIO`, `CRIAR_TRIBUNAL`.

---

## 8. Fora de Escopo do MVP (Roadmap) — NÃO IMPLEMENTAR AGORA

> Itens de contexto de domínio que **NÃO DEVEM** ser implementados nem bloquear a primeira versão operacional. O modelo de dados já é forward-compatible para acomodá-los.

- **SSO PDPJ-Br (OIDC/OAuth2)** — plugável via `AuthenticationProvider` (SISPREQ-13909).
- **Integração assíncrona com plataforma ETL (AWS Glue)** e atualização automática de status (SISPREQ-13919).
- **Resultados do processamento** (registros recebidos/aceitos/rejeitados, % aproveitamento) (SISPREQ-13922).
- **Consulta de inconsistências / registros rejeitados** e **download do relatório de erros** (SISPREQ-13923, 13924).
- **Reenvio versionado de arquivos corrigidos** (SISPREQ-13925) — depende dos resultados da ETL. *(Campos `versao`/`remessa_pai_id` já preparados.)*
- **Parametrização de competências** (janelas de abertura/prazo/retificação) (SISPREQ-13912).
- **Cadastro/versionamento de layouts CSV** (SISPREQ-13913) — *(campo `layout_id` já preparado.)*
- **Validação da estrutura interna (colunas) do CSV.**
- **Notificações** de mudança de status (SISPREQ-13927).
- **Painel administrativo completo** (SISPREQ-13928) e **observabilidade/monitoramento** (SISPREQ-13929).
- **Testes de carga/performance e escalabilidade** como execução (SISPREQ-13931).
- **2FA, anti‑brute‑force, assinatura digital de arquivos.**

---

## 9. Requisitos Não Funcionais e Segurança

- **HTTPS obrigatório** em todo o tráfego.
- **Bucket S3 privado**; nenhum arquivo público; CORS restrito à origem do portal.
- **URLs pré-assinadas** restritas (método, content-type, tamanho, caminho) e **temporárias**; URLs expiradas rejeitadas; sem exposição de credenciais AWS ao navegador.
- Validação de entrada **no backend** (não confiar no frontend); proteção contra injeção; senhas com **BCrypt**.
- Segredos apenas via variáveis de ambiente; **nunca** em código ou logs.
- Erros **NÃO DEVEM** expor detalhes internos (stack traces, dados sensíveis).
- Boas práticas **OWASP**.
- Interface **responsiva e acessível**; datas/horários em padrão único (UTC internamente).

---

## 10. Estratégia e Stack de Testes

- **Unidade — Backend:** JUnit 5 + Mockito. **Frontend:** Jasmine/Karma ou Jest.
- **Integração de API:** MockMvc (Spring) ou RestAssured — estrutura **preparada**.
- **E2E:** estrutura preparada para Cypress.
- **Testes de autorização OBRIGATÓRIOS:** garantir que um tribunal não acessa dados de outro (multi-tenancy).
- **Funcional, carga/performance e segurança:** planejados para execução manual/externa (JMeter, SonarQube) — **não** bloqueiam o MVP.

---

## 11. Instruções Estritas para o Agente de Código

1. **Planejamento primeiro.** Antes de escrever qualquer código, gere um **plano de ação passo a passo** (módulos Angular, serviços Spring, schema Flyway, infraestrutura Docker, ordem de implementação por módulo) e **aguarde aprovação** antes de executar.
2. **Git — Conventional Commits.** Ex.: `feat: add s3 presigned url service`, `fix: correct jwt expiration`.
3. **Git — Gitflow para branches.** Ex.: `feature/s3-presigned-upload`, `bugfix/login-error`.
4. **Backend — Clean Code.** Arquitetura em camadas (`Controller → Service → Repository`); mapeamento manual Entidade↔DTO; **todos** os endpoints documentados via **SpringDoc OpenAPI**.
5. **Backend — Erros.** `@ControllerAdvice` com códigos HTTP semânticos (400, 401, 403, 404, 409/422, 500) e payload JSON descritivo padronizado.
6. **Frontend — Padrões oficiais do Angular.** APIs isoladas em serviços `@Injectable`; interceptor JWT; Angular Material + UIKit.
7. **Documentação.** `README.md` detalhado explicando como subir o ambiente local completo com `docker-compose up` (incluindo `.env.example` e variáveis obrigatórias da AWS).
8. **Não implementar itens da Seção 8.** Se um item de fora‑de‑escopo for tocado, apenas deixar o **ponto de extensão preparado** (interface/campo), sem lógica funcional.

---

## 12. Rastreabilidade (Épico/HU → Escopo)

| HU (JIRA) | Item | Status neste MVP |
|-----------|------|------------------|
| SISPREQ-13908 | Arquitetura/domínio/contratos | ✅ Seções 4, 5 |
| SISPREQ-13909 | Auth SSO PDPJ-Br | 🔜 Preparado (D2), impl. futura |
| SISPREQ-13910 | Perfis/permissões/vínculo tribunal | ✅ Seção 6 (subconjunto) |
| SISPREQ-13911 | Cadastro de tribunais | ✅ Mínimo (7.2) |
| SISPREQ-13912 | Parametrização de competências | 🔜 Fora de escopo |
| SISPREQ-13913 | Versionamento de layouts | 🔜 Fora de escopo (campo preparado) |
| SISPREQ-13914 | Criação de remessas | ✅ Seção 7.3 |
| SISPREQ-13915 | URL pré-assinada | ✅ Seções 4.1, 7.3 |
| SISPREQ-13916 | Upload direto ao S3 | ✅ Seções 4.1, 7.3 |
| SISPREQ-13917 | Confirmação/integridade | ✅ Seção 7.3 |
| SISPREQ-13918 | Cancelamento pré-processamento | ✅ Seção 7.5 |
| SISPREQ-13919 | Integração assíncrona ETL | 🔜 Fora de escopo |
| SISPREQ-13920 | Acompanhamento de processamento | ✅ Parcial (estados pré-ETL) |
| SISPREQ-13921 | Histórico/filtros | ✅ Seção 7.4 |
| SISPREQ-13922 | Resultados do processamento | 🔜 Fora de escopo |
| SISPREQ-13923 | Inconsistências | 🔜 Fora de escopo |
| SISPREQ-13924 | Download de relatório | 🔜 Fora de escopo |
| SISPREQ-13925 | Reenvio/versionamento | 🔜 Fora de escopo (campos preparados) |
| SISPREQ-13926 | Trilha de auditoria | ✅ Básica (7.6, 5.5) |
| SISPREQ-13927 | Notificações | 🔜 Fora de escopo |
| SISPREQ-13928 | Painel administrativo | ✅ Mínimo (7.2) |
| SISPREQ-13929 | Observabilidade | 🔜 Fora de escopo |
| SISPREQ-13930 | Requisitos de segurança | ✅ Seção 9 (subconjunto MVP) |
| SISPREQ-13931 | Desempenho/escalabilidade | 🔜 Fora de escopo |

---

### Decisões que requerem confirmação do Product Owner
1. **D2 — Autenticação:** manter e-mail/senha + JWT no MVP (com SSO plugável)? Se o SSO PDPJ-Br já estiver disponível em ambiente de dev, pode-se antecipá-lo.
2. **Estratégia de logout/JWT:** stateless com expiração curta vs. blacklist de tokens.
3. **Gestão de tribunais/competência no MVP:** confirmar o nível mínimo aceitável (aqui adotado: tribunal com CRUD mínimo; competência como campo `AAAA-MM` validado, sem janelas de prazo).
