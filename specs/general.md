# Especificação de Requisitos

**1. Visão Geral do Produto**
Uma aplicação web (Frontend + Backend) com o objetivo principal de permitir o envio seguro de arquivos (CSV) contendo dados sobre precatórios para um Bucket AWS S3. O envio será realizado por servidores dos tribunais estaduais e federais brasileiros.

---
**2. Stack Tecnológico e Arquitetura**
*   **Frontend:** React (usando Functional Components e Hooks), TypeScript, Vite (bundler), Axios (para requisições HTTP).
*   **Backend:** Spring Boot (Java 17+), Spring Security, Spring Web, Spring Data JPA.
*   **Banco de Dados:** MySQL (com Flyway para versionamento do schema).
*   **Testes:** JUnit 5 e Mockito (Backend), Jest e React Testing Library (Frontend).
*   **Comunicação:** API REST (JSON).
*   **Infraestrutura/Configuração:** As credenciais de banco de dados e AWS S3 DEVEM ser injetadas exclusivamente via Variáveis de Ambiente (`.env` ou `application.properties/yml`). NUNCA hardcode credenciais.

---
**3. Perfis de Acesso (RBAC - Role Based Access Control)**
*   `ROLE_USER` (Usuário Padrão): Servidores dos tribunais. Podem fazer login e enviar arquivos CSV.
*   `ROLE_ADMIN` (Administrador): Pode fazer login, visualizar a lista de usuários e cadastrar novos usuários.

---
**4. Escopo do MVP (Minimum Viable Product - Foco de Execução Atual)**

*   **Módulo de Autenticação e Segurança:**
    *   Login via E-mail e Senha.
    *   Geração e validação de token JWT (JSON Web Token) no backend.
    *   O Frontend deve armazenar o JWT de forma segura e enviá-lo no header `Authorization: Bearer <token>` em rotas protegidas.
*   **Módulo de Gestão de Usuários (Apenas Admin):**
    *   Endpoint/Tela para listar usuários existentes (ID, Nome, Email, Tribunal, Role).
    *   Endpoint/Tela para o Admin criar novos `ROLE_USER`. (No MVP, usuários não se auto-cadastram).
*   **Módulo de Envio de Arquivos:**
    *   Upload de arquivos restrito à extensão `.csv`.
    *   Tamanho máximo do arquivo: 50MB.
    *   **Fluxo de Upload:** O Frontend envia o arquivo via `multipart/form-data` para o Backend. O Backend (Spring Boot) recebe, gera um nome único (UUID + timestamp + nome original) e faz o upload para o AWS S3 usando o AWS SDK para Java. O backend deve retornar sucesso apenas se o S3 confirmar o upload.

---
**5. Funcionalidades Pós-MVP (Roadmap Futuro) - IGNORAR NA FASE ATUAL**
*Estes itens são para contexto do domínio e NÃO devem ser implementados ou bloquear a primeira versão operacional.*
*   Mecanismos anti-brute force ou 2FA.
*   Assinatura digital de arquivos ou validação de hash (checksums).
*   Telas de histórico de envios e relatórios de auditoria.
*   Validação de estrutura interna (colunas) do CSV.

---
**6. Modelagem de Dados Inicial (Sugestão)**
*   **Entidade `User`**: `id` (UUID), `name` (String), `email` (String, unique), `password` (String, hashed com BCrypt), `tribunal` (String), `role` (Enum/String).

---
**7. Instruções Estritas para o Agente de Código**
*   **Boas Práticas Git:**
    *   Siga estritamente o padrão de commits `Conventional Commits` (ex: `feat: add S3 upload service`, `fix: correct JWT expiration`).
    *   Utilize o fluxo `Gitflow` para nomeação de branches (ex: `feature/s3-integration`, `bugfix/login-error`).
*   **Clean Code e Padrões:**
    *   No Spring Boot, utilize a arquitetura em camadas: `Controller` -> `Service` -> `Repository`.
    *   No React, separe componentes de UI da lógica de requisição (ex: use services ou custom hooks para chamadas de API).
    *   Tratamento de erros padronizado na API (retornar códigos HTTP semânticos: 400, 401, 403, 404, 500) com um payload JSON descrevendo o erro.
*   **Documentação:** Mantenha um arquivo `README.md` na raiz atualizado com instruções de como rodar o backend, o frontend e as variáveis de ambiente necessárias.