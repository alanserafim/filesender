# ÉPICO: Implementação do Portal Nacional de Remessas de Dados de Precatórios

**Objetivo**

Implementar o Portal Nacional de Remessas de Dados de Precatórios, permitindo que os tribunais realizem o envio dos arquivos CSV, acompanhem o processamento de cada remessa, consultem o histórico de envios, visualizem inconsistências identificadas durante o processamento e realizem reenvios quando necessário.
A solução será desenvolvida utilizando Angular no frontend e Java 21 + Spring Boot no backend, integrando-se ao Amazon S3.
 
**Descrição**

O Portal Nacional de Remessas será o principal ponto de interação entre os tribunais e a plataforma nacional de dados de precatórios.
A aplicação deverá oferecer uma interface segura e intuitiva para gerenciamento do ciclo completo das remessas mensais de arquivos, permitindo o acompanhamento de todo o fluxo operacional desde o envio até a conclusão do processamento.
O portal deverá atuar como camada de orquestração do processo de submissão de arquivos, sem participar do processamento dos dados propriamente dito. Os arquivos serão enviados diretamente ao Amazon S3 mediante URLs pré-assinadas, enquanto o processamento será realizado de forma assíncrona pela plataforma de ETL baseada em AWS Glue.
 
## Escopo

**Autenticação e Autorização**
* autenticação dos usuários dos tribunais via SSO da PDPJ-Br;
* gerenciamento de perfis e permissões;
* associação de usuários aos respectivos tribunais;
* controle de acesso às funcionalidades da aplicação.

**Gestão das Remessas**
* criação de novas remessas;
* envio de arquivos CSV;
* geração de URLs pré-assinadas para upload no Amazon S3;
* validação das informações da remessa;
* cancelamento de remessas antes do processamento.

**Acompanhamento do Processamento**
* consulta do status das remessas;
* visualização da etapa atual do processamento;
* consulta das datas e horários de cada etapa;
* acompanhamento do histórico de execuções.

**Consulta de Resultados**
* quantidade de registros recebidos;
* quantidade de registros aceitos;
* quantidade de registros rejeitados;
* tempo de processamento;
* mensagens de erro;
* motivo das rejeições.

**Reenvio de Arquivos**
* criação de nova versão de uma remessa;
* reenvio de arquivos corrigidos;
* manutenção do histórico das versões;
* preservação da rastreabilidade entre remessas.

**Histórico**
* consulta das remessas por competência;
* consulta por período;
* consulta por status;
* consulta por tribunal;
* visualização das versões enviadas.

**Auditoria**
* registro das operações realizadas;
* identificação do usuário responsável;
* histórico completo das alterações;
* trilha de auditoria das remessas.

**Administração**
* parametrização da aplicação;
* gerenciamento dos tribunais participantes;
* gerenciamento dos layouts aceitos;
* gerenciamento das competências disponíveis.


## Funcionalidades

O portal deverá disponibilizar, no mínimo, as seguintes funcionalidades:

* autenticação de usuários;
* envio de arquivos CSV;
* upload direto para o Amazon S3;
* consulta do histórico de remessas;
* acompanhamento em tempo real do processamento;
* visualização das estatísticas de processamento;
* consulta das inconsistências encontradas;
* download do relatório de erros;
* reenvio de arquivos corrigidos.


## Modelo Conceitual
A solução deverá ser orientada ao conceito de Remessa, representando cada submissão realizada por um tribunal para uma determinada competência.
Cada remessa deverá possuir, entre outras, as seguintes informações:

* identificador da remessa;
* tribunal remetente;
* competência;
* versão da remessa;
* arquivo enviado;
* data e hora do envio;
* usuário responsável;
* checksum do arquivo;
* status da remessa;
* quantidade de registros recebidos;
* quantidade de registros aceitos;
* quantidade de registros rejeitados;
* data de conclusão do processamento;
* mensagens de erro.


## Fluxo Operacional
O fluxo básico da solução deverá contemplar:

* criação da remessa;
* geração da URL pré-assinada;
* upload do arquivo diretamente para o Amazon S3;
* registro da remessa;
* processamento assíncrono pela plataforma ETL;
* atualização automática do status da remessa;
* disponibilização dos resultados ao tribunal.

## Integrações
A aplicação deverá integrar-se com:
* Amazon S3;
* Plataforma ETL baseada em AWS Glue;
* Glue Data Catalog (consulta de metadados, quando aplicável);
* Banco de dados operacional da aplicação;
* Serviço corporativo de autenticação (OIDC/OAuth2), quando disponível.


## Requisitos Não Funcionais
* autenticação segura;
* comunicação exclusivamente via HTTPS;
* arquitetura REST;
* upload direto para o Amazon S3 utilizando URLs pré-assinadas;
* processamento assíncrono;
* alta rastreabilidade;
* registro completo de auditoria;
* interface responsiva;
* compatibilidade com os navegadores homologados pelo CNJ.


## Entregáveis
* Aplicação Angular;
* API REST em Java/Spring Boot;
* Integração com Amazon S3;
* Integração com o mecanismo de autenticação;
* Gerenciamento das remessas;
* Consulta de status;
* Consulta do histórico;
* Relatórios de processamento;
* Documentação técnica;
* Manual do usuário.

## Critérios de Aceitação
* o tribunal consegue autenticar-se no portal;
* o envio do arquivo ocorre diretamente para o Amazon S3;
* a remessa é registrada automaticamente;
* o processamento pode ser acompanhado pelo usuário;
* o usuário consegue consultar o histórico completo das remessas;
* o sistema apresenta claramente os erros encontrados durante o processamento;
* o usuário consegue reenviar arquivos corrigidos;
todas as operações são registradas para auditoria;
* o portal apresenta desempenho adequado para o volume previsto de remessas mensais.


## Possíveis Stories

    PORTAL-01 - Implementar autenticação e autorização de usuários.
    PORTAL-02 - Implementar cadastro e gerenciamento dos tribunais.
    PORTAL-03 - Implementar gerenciamento de competências.
    PORTAL-04 - Implementar criação de remessas.
    PORTAL-05 - Implementar geração de URLs pré-assinadas para upload no Amazon S3.
    PORTAL-06 - Implementar upload direto de arquivos CSV.
    PORTAL-07 - Implementar consulta de status das remessas.
    PORTAL-08 - Implementar consulta do histórico de remessas.
    PORTAL-09 - Implementar visualização das estatísticas de processamento.
    PORTAL-10 - Implementar consulta dos registros rejeitados e download do relatório de erros.
    PORTAL-11 - Implementar reenvio de remessas.
    PORTAL-12 - Implementar auditoria das operações.
    PORTAL-13 - Implementar notificações sobre alteração de status das remessas.
    PORTAL-14 - Implementar painel administrativo da aplicação.
    
## Recomendação arquitetural
Para manter a solução desacoplada e alinhada à arquitetura já definida para o Data Lake, o portal deve ser uma camada de gestão operacional, sem incorporar lógica de ETL. Sua responsabilidade é administrar o ciclo de vida das remessas e fornecer visibilidade aos tribunais. A arquitetura sugerida é:

    Angular
        │
    REST API (Spring Boot)
        │
    ├── Autenticação (OIDC/OAuth2)
    ├── Gestão de Remessas
    ├── Histórico e Auditoria
    ├── Geração de URLs Pré-assinadas
    └── Consulta de Status
            │
            ├── Banco Operacional (PostgreSQL)
            ├── Amazon S3 (Upload direto)
            └── Plataforma ETL (AWS Glue / EventBridge / Step Functions)


Essa separação traz algumas vantagens importantes:
* o backend não manipula arquivos, apenas controla o processo de negócio;
* o upload direto ao S3 reduz carga sobre a aplicação e melhora a escalabilidade;
* a plataforma de ETL permanece independente da aplicação web, permitindo sua evolução sem impacto no portal;
* a eventual substituição da tecnologia de ETL (Glue, EMR, Databricks etc.) ou da ferramenta de BI (Qlik, Power BI, QuickSight etc.) não exige alterações significativas no Portal de Remessas, preservando seu papel como interface institucional entre os tribunais e a plataforma nacional de dados.
