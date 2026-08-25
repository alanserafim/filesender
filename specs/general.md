# Especificação de Requisitos e Arquitetura

**1. Visão Geral do Produto**
Uma aplicação web (Frontend + Backend) com o objetivo principal de permitir o envio seguro de arquivos (CSV) contendo dados sobre precatórios para um Bucket AWS S3. O envio será realizado por servidores dos tribunais estaduais e federais brasileiros.

---
**2. Stack Tecnológico e Arquitetura**
*   **Frontend:** Angular, Angular Material, UIKIT, HTML 5. 
*   **Backend:** Java 21, Spring Boot.
*   **Documentação de API:** SpringDoc OpenAPI.
*   **Banco de Dados:** PostgreSQL (com Flyway para versionamento do schema).
*   **Infraestrutura:** Docker (O projeto deve conter `Dockerfile` para as aplicações e `docker-compose.yml` para orquestração local do banco e serviços). As credenciais da AWS S3 DEVEM ser injetadas exclusivamente via Variáveis de Ambiente. NUNCA hardcode credenciais.

---
**3. Estratégia e Stack de Testes**
A aplicação deve possuir uma base preparada para uma estratégia de testes robusta cobrindo os requisitos técnicos e de negócio:
*   **Testes de Unidade:** JUnit 5 e Mockito (Backend) / Jasmine e Karma ou Jest (Frontend Angular).
*   **Testes Automatizados (API e UI):** O agente deve preparar a estrutura para testes E2E (ex: Cypress) e testes de integração de API (ex: RestAssured ou próprio MockMvc do Spring).
*   **Testes Funcionais, Carga/Performance e Segurança:** Planejados para execução manual e por ferramentas externas (ex: JMeter, SonarQube). O código deve ser escrito seguindo boas práticas de segurança da OWASP.

---
**4. Perfis de Acesso (RBAC - Role Based Access Control)**
*   `ROLE_USER` (Usuário Padrão): Servidores dos tribunais. Podem fazer login e enviar arquivos CSV.
*   `ROLE_ADMIN` (Administrador): Pode fazer login, visualizar a lista de usuários e cadastrar novos usuários.

---
**5. Escopo do MVP (Minimum Viable Product - Foco de Execução Atual)**

*   **Módulo de Autenticação e Segurança:**
    *   Login via E-mail e Senha.
    *   Geração e validação de token JWT (JSON Web Token) no backend.
    *   O Frontend deve armazenar o JWT de forma segura e enviá-lo via interceptor HTTP no header `Authorization: Bearer <token>` em rotas protegidas.
*   **Módulo de Gestão de Usuários (Apenas Admin):**
    *   Endpoint/Tela para listar usuários existentes (ID, Nome, Email, Tribunal, Role).
    *   Endpoint/Tela para o Admin criar novos `ROLE_USER`. (No MVP, usuários não se auto-cadastram).
*   **Módulo de Envio de Arquivos:**
    *   Upload de arquivos restrito à extensão `.csv`.
    *   Tamanho máximo do arquivo: 50MB.
    *   **Fluxo de Upload:** O Frontend Angular envia o arquivo via `multipart/form-data`. O Backend Spring recebe, gera um nome único (UUID + timestamp + nome original) e faz o upload para o AWS S3 usando o AWS SDK para Java. Retornar sucesso apenas se o S3 confirmar o upload.

---
**6. Funcionalidades Pós-MVP (Roadmap Futuro) - IGNORAR NA FASE ATUAL**
*Estes itens são para contexto do domínio e NÃO devem ser implementados ou bloquear a primeira versão operacional do MVP.*
*   **Integração de Autenticação (SSO):** Autenticação com o Serviço de Autenticação (SSO) do CNJ para login único dos servidores.
*   Mecanismos anti-brute force ou 2FA.
*   Assinatura digital de arquivos ou validação de hash (checksums).
*   Telas de histórico de envios e relatórios de auditoria.
*   Validação de estrutura interna (colunas) do CSV.

---
**7. Modelagem de Dados Inicial (Sugestão)**
*   **Entidade `User`**: `id` (UUID), `name` (String), `email` (String, unique), `password` (String, hashed com BCrypt), `tribunal` (String), `role` (Enum/String).

---
**8. Instruções Estritas para o Agente de Código**
*   **Planejamento primeiro:** Antes de escrever qualquer código, gere um plano de ação passo a passo de como os módulos Angular, serviços Spring e a infraestrutura Docker serão criados. Aguarde minha aprovação.
*   **Boas Práticas Git:**
    *   Siga estritamente o padrão de commits `Conventional Commits` (ex: `feat: add S3 upload service`, `fix: correct JWT expiration`).
    *   Utilize o fluxo `Gitflow` para nomeação de branches (ex: `feature/s3-integration`, `bugfix/login-error`).
*   **Clean Code e Padrões:**
    *   No Backend: Utilize a arquitetura em camadas (`Controller` -> `Service` -> `Repository`). Faça o mapeamento entre Entidades e DTOs de forma manual e limpa. Todos os endpoints DEVEM estar documentados via SpringDoc OpenAPI.
    *   No Frontend: Siga as diretrizes oficiais de estilo do Angular. Isole as chamadas de API em serviços (`@Injectable`) e gerencie o estado dos componentes de forma limpa. Integre o Angular Material e UIKIT conforme especificado.
    *   Tratamento de erros: Padronizado na API via `@ControllerAdvice`, retornando códigos HTTP semânticos (400, 401, 403, 404, 500) com payload JSON descritivo.
*   **Documentação:** Crie e mantenha um `README.md` detalhado explicando como subir o ambiente local completo utilizando o `docker-compose up`.