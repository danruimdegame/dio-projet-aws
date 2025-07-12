# RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS

Data: 15/07/2025  
Empresa: Abstergo Industries  
Responsável: Daniel Tomaz

## Introdução

Este documento apresenta o plano de adoção de serviços AWS para a Abstergo Industries, empresa farmacêutica que distribui medicamentos a clientes B2B e realiza P&D. O objetivo é criar uma infraestrutura em nuvem do zero, eliminando servidores ociosos, automatizando fluxos de trabalho e garantindo compliance regulatória, com foco na redução de custos operacionais.

## Descrição do Projeto

A arquitetura proposta abrange três domínios principais: armazenamento de dados (objetos), processamento de eventos e banco de dados relacional. Em cada fase, selecionamos serviços com modelo pay-as-you-go e recursos de escalabilidade automática.

### Etapa 1  
- Nome da ferramenta: Amazon S3 (Intelligent-Tiering)  
- Foco da ferramenta: armazenamento de objetos com movimentação automática entre camadas conforme padrão de acesso  
- Descrição de caso de uso:  
  1. Centralizar no S3 todos os registros de lotes farmacêuticos, protocolos de ensaio clínico e relatórios de estabilidade.  
  2. Marcar objetos com tags (‘lote’, ‘ensaioclínico’, ‘documentovalidação’) e criar regras de ciclo de vida que movem automaticamente relatórios não acessados por 30 dias para camadas de menor custo.  
  3. Ativar criptografia SSE-KMS para proteger dados sensíveis e habilitar AWS CloudTrail para rastrear todos os acessos aos buckets, atendendo a requisitos de auditoria regulatória.  
  4. Integrar Amazon Macie para identificação de informações confidenciais (PII ou segredos de P&D) e geração de alertas de segurança.

### Etapa 2  
- Nome da ferramenta: AWS Lambda  
- Foco da ferramenta: execução de lógica de negócio sem servidor em resposta a eventos  
- Descrição de caso de uso:  
  1. Processar automaticamente uploads de dados analíticos de laboratório: functions Lambda acionadas por eventos S3 convertem CSV de cromatografia e carregam registros validados no AWS Glue Data Catalog.  
  2. Gerar thumbnails de imagens de microscopia e gráficos de lotes: Lambda integra-se ao Amazon Rekognition e Matplotlib para criar versões otimizadas e armazena em bucket dedicado.  
  3. Validar esquemas de dados de fornecedores (formulários de matéria-prima) e publicar notificações via Amazon SNS quando inconsistências forem detectadas, agilizando correções.  
  4. Orquestrar relatórios trimestrais de conformidade: EventBridge dispara Lambda que compila logs do CloudTrail, gera PDF dos relatórios e distribui internamente via Amazon WorkMail.

### Etapa 3  
- Nome da ferramenta: Amazon Aurora Serverless v2  
- Foco da ferramenta: banco de dados relacional que escala automaticamente sob demanda  
- Descrição de caso de uso:  
  1. Armazenar metadados de lotes, histórico de inspeções de qualidade e tabelas de formulação em um cluster Aurora compatível com PostgreSQL.  
  2. Definir endpoints de leitura para dashboards de produção no Amazon QuickSight, garantindo baixa latência mesmo em picos de consulta.  
  3. Configurar capacidade mínima de ACUs para períodos ociosos e escalonamento automático até a capacidade máxima em janelas de alta demanda, eliminando custos de instâncias subutilizadas.  
  4. Aplicar backups contínuos com retenção de 90 dias e failover multi-AZ para assegurar RTO e RPO compatíveis com normas de GMP.  
  5. Gerenciar credenciais via AWS Secrets Manager e acesso granular com IAM, segmentando perfis de R&D, produção e auditoria.

## Conclusão

A combinação das três etapas gera economia em três frentes distintas:

1. Armazenamento (S3 Intelligent-Tiering)  
   - Ao mover automaticamente objetos pouco acessados para camadas de custo reduzido, a Abstergo elimina pagamentos desnecessários por GB/mês em armazenamento premium.  
   - A automação reduz horas dedicadas a análises manuais de ciclo de vida, liberando equipe de operações para tarefas de maior valor.  
   - Criptografia gerenciada e auditoria contínua com CloudTrail evitam custos indiretos de conformidade e riscos de multas regulatórias.

2. Processamento (AWS Lambda)  
   - A cobrança por milissegundos de execução trim elimina a necessidade de instâncias EC2 em standby, reduzindo a fatura de computação em até 70% em comparação a servidores dedicados.  
   - Ao acionar funções somente quando há eventos (upload, notificação, agendamento), a empresa paga apenas pelo uso efetivo—nenhuma hora de máquina ociosa é faturada.  
   - A padronização de rotinas serverless ainda reduz custos de manutenção de sistemas operacionais, patching e licenciamento.

3. Banco de Dados (Aurora Serverless v2)  
   - A escalabilidade automática de ACUs faz com que, em períodos de baixa demanda (por exemplo: finais de semana ou feriados), o custo do cluster caia para o nível mínimo configurado.  
   - Não há cobrança por instâncias reservadas nem necessidade de provisionar capacidade para picos, eliminando superfaturamento de recursos subutilizados.  
   - O modelo serverless unifica alta disponibilidade e backup contínuo, poupando a aquisição e gerenciamento de instâncias de réplica e sistemas de replicação cross-region.

Ao migrar para esse modelo, a Abstergo Industries ganha uma infraestrutura que ajusta gastos em tempo real, elimina custos fixos e oferece previsibilidade orçamentária. Recomenda-se monitorar métricas-chave no AWS Cost Explorer e definir alertas de orçamentação no AWS Budgets, revisando políticas de ciclo de vida e thresholds de escalonamento trimestralmente para maximizar a economia.

## Anexos

- Amazon S3 Intelligent-Tiering  
  https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html  
- AWS Lambda Developer Guide  
  https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html  
- Amazon Aurora Serverless v2 User Guide  
  https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.html  

## Assinatura do ResponsÃ¡vel pelo Projeto:
  Daniel Tomaz
