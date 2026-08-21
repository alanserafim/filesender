# Especificação de Requisitos

**1. Visão Geral do Produto**

Uma aplicação com o objetivo principal de envio de arquivos com dados sobre precatorios para um Bucket S3. O envio será realizado pelos servidores dos tribuinais estaduais e federais.

---
**2. Stack Tecnológico e Arquitetura**

- **Frontend:** React
- **Backend:**  Spring Boot em Java
- **Banco de Dados :** MySQL
- **Testes:** Junit

---
**3. Perfis de Acesso**

- **Usuário Padrão:** Servidores dos tribunais que enviam os arquivos
- **Administrador:** Gerencia usuários

---
**4. Escopo do MVP (Minimum Viable Product)** 

- **Autenticação:** Cadastro e login básico (E-mail/Senha).
- **Envio de Arquivos CSV**

---
**5. Funcionalidades Pós-MVP (Roadmap Futuro)** _Estes itens são complexos e NÃO devem bloquear a primeira versão operacional._

- **Validação de autenticidade**
- **Validação de envio**
- **Histórico de envio por usuário/tribunal:** 

--- 
**6. Instruções Específicas para o Agente de Código**

- Mensagens de commit enxutas no padrão conventional commits
- Usar estratégio de gitflow na nomeação das branches