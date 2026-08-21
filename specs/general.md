# Especificação de Requisitos

**1. Visão Geral do Produto**

Uma aplicação com o objetivo principal de envio de arquivos com dados sobre precatórios para um Bucket S3. O envio será realizado pelos servidores dos tribuinais estaduais e federais.

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

- **Módulo de Autenticação:** Cadastro e login básico (E-mail/Senha).
- **Módulo de envio de arquivos** Envio de arquivos CSV.

---
**5. Funcionalidades Pós-MVP (Roadmap Futuro)** _Estes itens são complexos e NÃO devem bloquear a primeira versão operacional._

- **Validação de autenticidade** Estratégia de autenticação de usuário que evite falhas de segurança ou ataques diversos como força bruta.
- **Validação de envio** Assinatura digital, hash ou outro tipo de mecanismo de segurança que garanta a autenticidade e integridade do arquivo.
- **Histórico de envio por usuário/tribunal:**  O usuário consegue verificar suas atividades de envio, e o admnistrador visualiza as atividades do sistema como um todo. 

--- 
**6. Instruções Específicas para o Agente de Código**

- Mensagens de commit enxutas no padrão conventional commits
- Usar gitflow para nomeação das branches