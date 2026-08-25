# HUs cadastradas no JIRA

## SISPREQ-13911
Implementar cadastro de tribunais

[PORTAL DE REMESSAS] Implementar cadastro e gerenciamento dos tribunais participantes

História

Como administrador,
quero gerenciar os tribunais participantes,
para controlar quais instituições podem realizar remessas de dados.

Dados mínimos

* Código;
* Sigla;
* Nome;
* Segmento;
* Situação;
* Responsável;
* Identificador utilizado no SSO;
* Data de adesão;
* Configurações específicas.

Critérios de aceite

* Deve ser possível cadastrar, consultar e alterar tribunais;
* O código e a sigla devem ser únicos;
* A exclusão física de tribunal com histórico não deve ser permitida;
* Tribunais inativos não devem criar novas remessas;
* Alterações devem ser auditadas;
* O vínculo com usuários deve ser validado;
* Apenas perfis autorizados devem gerenciar tribunais.

## SISPREQ-13912

[PORTAL DE REMESSAS] Implementar parametrização das competências de envio

História

Como administrador,
quero configurar as competências disponíveis para remessa,
para controlar os períodos autorizados para envio dos arquivos.

Escopo

* Cadastrar competência;
* Definir período de abertura;
* Definir prazo de envio;
* Definir prazo de retificação;
* Abrir ou bloquear competência;
* Associar competências aos tribunais;
* Tratar envio fora do prazo;
* Permitir exceção administrativa.

Critérios de aceite

* A competência deve seguir o formato definido, como AAAA-MM;
* Não deve existir duplicidade de competência;
* Somente competências abertas devem aceitar remessas;
* Exceções devem exigir perfil autorizado e justificativa;
* O sistema deve apresentar claramente o período disponível;
* Alterações devem ser auditadas;
* Uma competência com remessas não deve ser excluída fisicamente.

## SISPREQ-13908

[PORTAL DE REMESSAS] Definir arquitetura, modelo de domínio e contratos de integração

História

Como arquiteto de solução,
quero definir a arquitetura técnica e os contratos de integração do portal,
para garantir o desacoplamento entre a aplicação, o armazenamento e a plataforma ETL.

Escopo

* Definir componentes da solução;
* Definir responsabilidades do frontend e backend;
* Modelar o domínio de remessas;
* Definir estados e transições;
* Definir APIs REST;
* Definir integração com S3;
* Definir integração com autenticação;
* Definir integração com a plataforma ETL;
* Definir integração com o banco operacional;
* Definir segregação entre ambientes;
* Definir requisitos de desempenho e segurança.


Critérios de aceite

* A arquitetura deve estar documentada;
* O modelo conceitual de remessa deve estar definido;
* Os contratos das APIs devem estar especificados;
* Os estados e transições devem estar documentados;
* O portal não deve executar regras de processamento ETL;
* Os arquivos não devem trafegar pelo backend;
* Os ambientes devem estar segregados;
* A arquitetura deve ser aprovada pelas equipes responsáveis.

## SISPREQ-13909

[PORTAL DE REMESSAS] Implementar autenticação integrada ao SSO da PDPJ-Br

História

Como usuário de um tribunal,
quero autenticar-me por meio do serviço corporativo da PDPJ-Br,
para acessar o portal com segurança e identidade institucional validada.

Escopo

* Integração OIDC/OAuth2;
* Login e logout;
* Validação de tokens;
* Renovação de sessão;
* Tratamento de sessão expirada;
* Recuperação dos dados do usuário;
* Tratamento de falhas de autenticação;
* Redirecionamento após autenticação;
* Configuração por ambiente.

Critérios de aceite

* O usuário deve autenticar-se por meio do provedor autorizado;
* O backend deve validar assinatura, emissor, audiência e validade do token;
* Usuários não autenticados não devem acessar funcionalidades protegidas;
* Tokens não devem ser gravados em logs;
* Sessões expiradas devem exigir nova autenticação;
* O logout deve encerrar corretamente a sessão;
* Falhas devem ser apresentadas sem exposição de informações sensíveis;
* A integração deve funcionar nos ambientes autorizados.

Dependências: 
* disponibilidade do SSO.


## SISPREQ-13910

[PORTAL DE REMESSAS] Implementar perfis, permissões e vinculação de usuários aos tribunais

História

Como administrador da aplicação,
quero controlar as funcionalidades e os dados acessíveis por cada perfil,
para impedir acessos indevidos entre tribunais.

Perfis sugeridos

* Usuário de tribunal;
* Gestor de tribunal;
* Operador nacional;
* Administrador;
* Auditor;
* Suporte técnico.

Critérios de aceite

* Cada usuário deve possuir perfil e tribunal associados;
* Um tribunal não deve consultar remessas de outro tribunal;
* O perfil administrativo deve possuir acesso explicitamente autorizado;
* As permissões devem ser aplicadas no backend;
* A interface deve ocultar ações não permitidas;
* Tentativas de acesso indevido devem ser rejeitadas e auditadas;
* As permissões devem seguir o princípio do menor privilégio;
* A matriz de perfis e permissões deve estar documentada.

## SISPREQ-13913

[PORTAL DE REMESSAS] Implementar cadastro e versionamento dos layouts de arquivos CSV

História

Como administrador,
quero gerenciar as versões de layout aceitas pelo portal,
para garantir que cada remessa informe o formato aplicável ao arquivo enviado.

Escopo

* Cadastrar layout;
* Versionar layout;
* Definir vigência;
* Definir campos e tipos;
* Disponibilizar arquivo ou manual do layout;
* Ativar e desativar versões;
* Associar layout à competência;
* Impedir uso de versão não vigente.

Critérios de aceite

* Cada layout deve possuir identificador e versão;
* A vigência deve estar definida;
* Versões já utilizadas não devem ser excluídas;
* A remessa deve registrar a versão utilizada;
* Somente layouts vigentes devem aceitar novas remessas;
* A documentação do layout deve estar disponível para consulta;
* Alterações devem ser auditadas.


## SISPREQ-13914

[PORTAL DE REMESSAS] Implementar criação e registro de novas remessas

História

Como usuário de um tribunal,
quero criar uma remessa para determinada competência,
para iniciar o envio mensal dos dados de precatórios.

Dados mínimos

* Identificador da remessa;
* Tribunal;
* Competência;
* Versão;
* Layout;
* Nome do arquivo;
* Usuário responsável;
* Data de criação;
* Status;
* Identificador de correlação.

Critérios de aceite

* O tribunal deve ser obtido do usuário autenticado;
* O usuário não deve selecionar outro tribunal;
* A competência deve estar disponível para envio;
* O layout deve estar vigente;
* A remessa deve receber identificador único;
* A primeira remessa deve iniciar na versão definida;
* O sistema deve impedir duplicidade não controlada;
* A criação deve registrar data, usuário e tribunal;
* A remessa deve iniciar em status compatível com o fluxo;
* Nenhuma lógica de processamento ETL deve ser executada pelo portal.

## SISPREQ-13915

[PORTAL DE REMESSAS] Implementar geração segura de URL pré-assinada para upload no Amazon S3

História

Como usuário de um tribunal,
quero receber uma URL temporária para enviar o arquivo diretamente ao S3,
para realizar o upload sem transferir o arquivo pelo backend do portal.

Escopo

* Gerar chave do objeto no S3;
* Gerar URL pré-assinada;
* Definir tempo de expiração;
* Restringir método e destino;
* Definir limite de tamanho;
* Associar a URL à remessa;
* Gerar identificador de upload;
* Registrar tentativa de envio;
* Impedir reutilização indevida.

Critérios de aceite

* A URL deve possuir validade limitada;
* A URL deve permitir upload somente no caminho autorizado;
* O caminho deve identificar tribunal, competência, remessa e versão;
* O backend não deve receber o conteúdo do arquivo;
* O bucket não deve ser público;
* O usuário deve gerar URL apenas para remessa do seu tribunal;
* URLs expiradas devem ser rejeitadas;
* A geração deve ser auditada;
* Nenhuma credencial AWS deve ser exposta ao navegador;
* O nome recebido do usuário não deve controlar diretamente o caminho físico no S3.

Dependências:  
* infraestrutura S3.

## SISPREQ-13916

[PORTAL DE REMESSAS] Implementar upload direto de arquivos CSV para o Amazon S3

História

Como usuário de um tribunal,
quero enviar o arquivo CSV diretamente ao Amazon S3,
para concluir o fornecimento dos dados da remessa de forma segura e escalável.

Escopo frontend

* Seleção do arquivo;
* Validação de extensão;
* Validação de tamanho;
* Exibição do progresso;
* Cancelamento antes da conclusão;
* Tratamento de expiração da URL;
* Confirmação do upload;
* Mensagens de sucesso e erro.

Critérios de aceite

* Apenas arquivos permitidos devem ser selecionados;
* O upload deve ocorrer diretamente entre navegador e S3;
* O progresso deve ser apresentado ao usuário;
* A falha no envio não deve confirmar a remessa;
* Deve ser possível obter nova URL após expiração;
* O sistema deve registrar nome, tamanho e tipo do arquivo;
* O upload deve utilizar exclusivamente HTTPS;
* O arquivo deve ser armazenado no caminho correspondente à remessa;
* O usuário deve receber confirmação clara do resultado;
* Arquivos enviados não devem ser tornados públicos.


## SISPREQ-13917

[PORTAL DE REMESSAS] Implementar confirmação do upload e validação de integridade do arquivo

História

Como responsável pela remessa,
quero confirmar que o arquivo foi armazenado corretamente,
para liberar a remessa ao processamento assíncrono.

Escopo

* Confirmar existência do objeto no S3;
* Validar tamanho;
* Registrar ETag ou checksum;
* Validar metadados obrigatórios;
* Atualizar status;
* Impedir alteração após submissão;
* Registrar data e hora;
* Emitir evento para processamento;
* Tratar confirmação duplicada.

Critérios de aceite

* A confirmação deve consultar o objeto no S3;
* A remessa não deve ser submetida sem arquivo válido;
* O checksum deve ser registrado;
* O mesmo arquivo não deve gerar submissões duplicadas;
* A confirmação deve ser idempotente;
* Após submissão, o arquivo não deve ser substituído;
* O status deve ser atualizado de forma transacional;
* O evento deve possuir identificador de correlação;
* Falhas na emissão do evento não devem produzir uma remessa falsamente liberada.

Dependências: 
* integração com a plataforma ETL.

## SISPREQ-13918

[PORTAL DE REMESSAS] Implementar cancelamento de remessa antes do processamento

História

Como usuário de um tribunal,
quero cancelar uma remessa ainda não processada,
para impedir o processamento de um arquivo enviado incorretamente.

Critérios de aceite

* Somente remessas em estados permitidos devem ser canceladas;
* Remessas em processamento ou concluídas não devem ser canceladas;
* O cancelamento deve exigir confirmação;
* Deve ser registrado o usuário, a data e o motivo;
* O histórico da remessa deve ser preservado;
* O cancelamento não deve apagar fisicamente os registros;
* A tentativa de cancelamento fora do estado permitido deve ser rejeitada;
* O evento de processamento pendente deve ser invalidado ou tratado.

## SISPREQ-13919

[PORTAL DE REMESSAS] Implementar integração assíncrona para atualização do processamento

História

Como usuário de um tribunal,
quero que o portal receba automaticamente as atualizações da plataforma ETL,
para acompanhar o processamento sem intervenção manual.

Escopo

* Definir contrato de atualização;
* Receber eventos ou consultar status;
* Validar origem da atualização;
* Correlacionar execução e remessa;
* Atualizar etapa e status;
* Registrar datas das transições;
* Tratar mensagens duplicadas;
* Tratar eventos fora de ordem;
* Preservar histórico de execuções;
* Registrar resultado final.

Critérios de aceite

* Toda atualização deve estar associada a uma remessa válida;
* A origem da mensagem deve ser autenticada;
* Eventos duplicados devem ser tratados de forma idempotente;
* Transições inválidas devem ser rejeitadas e registradas;
* Eventos fora de ordem não devem regredir indevidamente o status;
* As datas de início e término das etapas devem ser armazenadas;
* Falhas de integração devem gerar alerta;
* O histórico não deve ser sobrescrito;
* O contrato deve ser versionado e documentado.

Dependências: 
* contrato da plataforma ETL


## SISPREQ-13920

[PORTAL DE REMESSAS] Implementar consulta e acompanhamento do processamento das remessas

História

Como usuário de um tribunal,
quero acompanhar a situação e a etapa atual das minhas remessas,
para saber se o arquivo foi recebido, processado ou rejeitado.

Informações apresentadas

* Identificador;
* Tribunal;
* Competência;
* Versão;
* Arquivo;
* Data do envio;
* Status;
* Etapa atual;
* Data de início;
* Última atualização;
* Data de conclusão;
* Tempo de processamento.

Status sugeridos

* Rascunho;
* Aguardando upload;
* Arquivo recebido;
* Aguardando processamento;
* Em validação;
* Em processamento;
* Processada com sucesso;
* Processada com rejeições;
* Rejeitada;
* Falha técnica;
* Cancelada.

Critérios de aceite

* O usuário deve visualizar apenas remessas autorizadas;
* O status deve refletir a última atualização válida;
* A etapa atual deve ser apresentada de forma clara;
* Datas e horários devem seguir o padrão institucional;
* A atualização da tela não deve alterar o processamento;
* O portal deve informar quando ocorreu a última atualização;
* Status de negócio e falha técnica devem ser diferenciados;
* A tela deve ser responsiva e acessível.


## SISPREQ-13921

[PORTAL DE REMESSAS] Implementar histórico, pesquisa e filtros de remessas

História

Como usuário de um tribunal,
quero consultar as remessas anteriores utilizando filtros,
para acompanhar os envios realizados e localizar uma submissão específica.

Filtros mínimos

* Competência;
* Período de envio;
* Status;
* Versão;
* Nome do arquivo;
* Identificador da remessa;
* Tribunal, para perfis nacionais.


Critérios de aceite

* A consulta deve ser paginada;
* Os filtros devem poder ser combinados;
* O usuário de tribunal deve visualizar apenas os seus dados;
* O perfil nacional deve consultar conforme sua permissão;
* A ordenação padrão deve apresentar as remessas mais recentes;
* As versões devem ser apresentadas de forma relacionada;
* A pesquisa deve atender ao tempo de resposta definido;
* A ausência de resultados deve ser informada claramente.

## SISPREQ-13922

[PORTAL DE REMESSAS] Implementar visualização dos resultados do processamento

História

Como usuário de um tribunal,
quero visualizar o resultado consolidado de uma remessa,
para compreender o resultado do processamento dos dados enviados.

Informações mínimas

* Registros recebidos;
* Registros aceitos;
* Registros rejeitados;
* Percentual de aproveitamento;
* Horário de início e conclusão;
* Tempo de processamento;
* Status final;
* Etapas executadas;
* Mensagens gerais;
* Versão do layout;
* Versão das regras de processamento.

Critérios de aceite

* Os totais devem corresponder ao resultado informado pela ETL;
* A soma de aceitos e rejeitados deve ser reconciliável com o total recebido;
* O portal não deve recalcular resultados produzidos pela ETL;
* A versão das regras deve ser exibida quando disponível;
* As mensagens devem ser compreensíveis;
* Informações técnicas detalhadas devem ficar restritas aos perfis autorizados;
* O resultado deve permanecer vinculado à versão da remessa.

## SISPREQ-13923

[PORTAL DE REMESSAS] Implementar consulta das inconsistências e registros rejeitados

História

Como usuário de um tribunal,
quero consultar as inconsistências encontradas na remessa,
para corrigir os dados antes de realizar um novo envio.

Informações sugeridas

* Código da inconsistência;
* Descrição;
* Severidade;
* Linha ou registro;
* Campo;
* Valor recebido;
* Regra violada;
* Orientação para correção.

Critérios de aceite

* As inconsistências devem estar vinculadas à remessa e à versão;
* Deve ser possível filtrar por código, severidade e campo;
* A consulta deve ser paginada;
* O sistema deve diferenciar erro impeditivo de alerta;
* O usuário deve visualizar somente inconsistências do seu tribunal;
* Informações sensíveis devem ser protegidas ou mascaradas;
* O portal deve exibir os resultados produzidos pela ETL sem alterar seu conteúdo;
* Grandes volumes não devem comprometer a navegação.

## SISPREQ-13924

[PORTAL DE REMESSAS] Implementar download do relatório de inconsistências

História

Como usuário de um tribunal,
quero baixar o relatório com as inconsistências encontradas,
para realizar a correção dos dados de maneira estruturada.

Critérios de aceite

* O relatório deve ser gerado ou disponibilizado pela execução correspondente;
* O arquivo deve identificar remessa, competência e versão;
* O formato deve seguir o padrão definido, preferencialmente CSV;
* O conteúdo deve corresponder às inconsistências apresentadas no portal;
* O download deve exigir autenticação e autorização;
* O acesso deve possuir validade limitada quando utilizar URL pré-assinada;
* O download deve ser auditado;
* O relatório deve preservar corretamente caracteres e acentuação;
* O arquivo não deve expor registros de outro tribunal.


Dependências: 
* disponibilização do relatório pela ETL.

## SISPREQ-13925

[PORTAL DE REMESSAS] Implementar reenvio de arquivos corrigidos e versionamento das remessas

História

Como usuário de um tribunal,
quero reenviar um arquivo corrigido,
para resolver as inconsistências mantendo a rastreabilidade dos envios anteriores.

Escopo

* Criar nova versão;
* Vincular à remessa anterior;
* Preservar arquivos e resultados anteriores;
* Gerar nova URL pré-assinada;
* Registrar justificativa;
* Submeter nova execução;
* Apresentar histórico das versões;
* Identificar versão vigente.


Critérios de aceite

* O reenvio deve criar nova versão;
* A versão anterior não deve ser alterada ou excluída;
* Todas as versões devem permanecer consultáveis;
* O relacionamento entre as versões deve ser preservado;
* Cada versão deve possuir arquivo, checksum, usuário e resultado próprios;
* O sistema deve identificar a versão mais recente;
* O reenvio deve respeitar a competência e os prazos;
* A nova versão deve passar por todo o fluxo de processamento;
* O reenvio não deve duplicar ou sobrescrever a execução anterior.

## SISPREQ-13926

[PORTAL DE REMESSAS] Implementar trilha de auditoria das operações

História

Como auditor,
quero consultar as operações realizadas no portal,
para identificar quem realizou cada ação, quando e sobre qual recurso.

Eventos mínimos

* Autenticação;
* Criação de remessa;
* Geração de URL;
* Confirmação do upload;
* Submissão;
* Cancelamento;
* Reenvio;
* Download de relatório;
* Alteração de parâmetros;
* Alteração de tribunal, competência ou layout;
* Acesso administrativo;
* Tentativas de acesso indevido.


Critérios de aceite

* O registro deve conter usuário, data, ação e recurso;
* Deve identificar tribunal, remessa e versão quando aplicável;
* Os registros não devem ser alteráveis por usuários comuns;
* O histórico deve ser pesquisável;
* Dados sensíveis e credenciais não devem constar nos logs;
* O horário deve possuir padrão único e rastreável;
* A auditoria deve estar disponível apenas para perfis autorizados.


## SISPREQ-13927

[PORTAL DE REMESSAS] Implementar notificações sobre alterações de status

História

Como usuário de um tribunal,
quero receber notificações sobre eventos relevantes da remessa,
para acompanhar o processamento sem consultar continuamente o portal.

Eventos sugeridos

* Arquivo recebido;
* Processamento iniciado;
* Processamento concluído;
* Remessa processada com rejeições;
* Remessa rejeitada;
* Falha técnica;
* Competência próxima do encerramento.

Critérios de aceite

* Os eventos notificáveis devem ser configuráveis;
* A notificação deve identificar remessa, competência e status;
* O usuário não deve receber dados de outro tribunal;
* Notificações duplicadas devem ser evitadas;
* Falha de notificação não deve alterar o status da remessa;
* O envio deve ser registrado;
O conteúdo não deve expor informações sensíveis;
O canal inicialmente habilitado deve estar documentado;
* Deve ser possível consultar as notificações no portal, caso esse canal seja adotado.

Dependências: 
* definição do canal institucional.

## SISPREQ-13928

[PORTAL DE REMESSAS] Implementar painel administrativo da aplicação

História

Como administrador,
quero gerenciar as configurações operacionais do portal,
para manter tribunais, competências, layouts e parâmetros atualizados.

Funcionalidades

* Gerenciar tribunais;
* Gerenciar competências;
* Gerenciar layouts;
* Configurar limites de arquivo;
* Configurar prazos;
* Consultar remessas nacionais;
* Consultar falhas de integração;
* Gerenciar parâmetros não sensíveis;
* Consultar auditoria;
* Controlar ativações.


Critérios de aceite

* O acesso deve ser restrito aos perfis autorizados;
* Alterações devem exigir validação;
* Operações críticas devem exigir confirmação;
* Todas as alterações devem ser auditadas;
* Segredos e credenciais não devem ser administrados pela interface;
* Parâmetros devem ser segregados por ambiente;
* A exclusão física de registros com histórico não deve ser permitida;
* O painel deve seguir o padrão visual e de acessibilidade do portal.


## SISPREQ-13929

Implementar observabilidade e monitoramento

Funcionalidades

* Tempo de resposta;
* Erros por endpoint;
* Geração e confirmação de URLs;
* Falhas de integração;
* Atualizações de status pendentes;
* Health checks;
* Identificadores de correlação;
* Alertas;
* Painel operacional.

Critérios de aceite

* Requisições devem possuir identificador de correlação;
* Falhas devem identificar serviço e operação;
* Credenciais e dados pessoais não devem aparecer nos logs;
* Health checks devem estar disponíveis para componentes autorizados;
* Alertas devem ser emitidos para falhas críticas;
* Deve ser possível relacionar remessa, execução ETL e evento;
* O histórico de métricas deve seguir a retenção definida;
* O monitoramento deve ser segregado por ambiente.

Dependência: 
* transversal.

## SISPREQ-13930

[PORTAL DE REMESSAS] Implementar e validar requisitos de segurança

Escopo

* HTTPS;
* CORS;
* CSRF conforme arquitetura;
* Validação de entrada;
* Proteção contra injeção;
* Controle de upload;
* Limitação de requisições;
* Gestão de segredos;
* Headers de segurança;
* Dependências vulneráveis;
* Testes de autorização;
* Proteção contra acesso entre tribunais.


Critérios de aceite

* Todo tráfego deve utilizar HTTPS;
* Credenciais não devem estar no código;
* Entradas devem ser validadas no backend;
* O sistema deve impedir acesso horizontal entre tribunais;
* As URLs pré-assinadas devem ser restritas e temporárias;
* O backend não deve confiar apenas nas validações do frontend;
* Dependências críticas vulneráveis devem ser tratadas;
* Erros não devem expor detalhes internos;
* Os testes de segurança devem possuir evidências;
* Pendências críticas devem ser resolvidas antes da produção.


Dependências: 
* funcionalidades principais implementadas.

## SISPREQ-13931

[PORTAL DE REMESSAS] Validar desempenho, escalabilidade e capacidade do portal

Cenários

* Acessos simultâneos;
* Criação simultânea de remessas;
* Geração de URLs;
* Consulta de histórico;
* Grandes volumes de inconsistências;
* Atualização simultânea de status;
* Upload de arquivos no limite autorizado;
* Reenvios;
* Indisponibilidade temporária de uma integração.

Critérios de aceite

* Os tempos máximos devem estar previamente definidos;
* As APIs devem atender ao volume esperado;
* A consulta de histórico deve permanecer paginada;
* O upload direto não deve consumir recursos do backend;
* A aplicação deve recuperar-se de falhas transitórias;
* Os resultados devem indicar volume, duração e taxa de erro;
* Gargalos devem ser documentados;
* Pendências críticas de capacidade devem ser resolvidas antes da produção.

Dependências: 
* fluxo principal integrado.
