Pipeline de Processamento de Logs e Métricas (AWS Step Functions)

Este repositório contém a definição da arquitetura de uma AWS Step Function projetada para processar dados de logs de forma distribuída, gerar métricas em tempo real e armazenar resumos estruturados, finalizando com uma consolidação diária via AWS Lambda.
📐 Fluxo da Arquitetura

O fluxo é dividido em três fases principais: processamento em lote (Map State), persistência/telemetria e consolidação.
Code snippet

graph TD
    Start([Start]) --> Map[Map: ProcessLogsUnderPrefix]
    subgraph Map State (S3 Input Bucket)
        Pass[Pass: AnalyzeLogData] --> CW[CloudWatch: PutMetricData]
        CW --> DB[DynamoDB: PutItem]
    end
    Map --> Export[Export to S3 Result Bucket]
    Export --> Lambda[Lambda: GenerateDailySummary]
    Lambda --> End([End])

1. Processamento Distribuído (ProcessLogsUnderPrefix)

    Tipo: Map State (Inline ou Distributed).

    Origem: Itens lidos de um bucket S3 de entrada (<S3_INPUT_BUCKET>).

    Função: Itera sobre as chaves dos objetos de log encontrados sob um prefixo específico para processá-los individualmente ou em lotes menores.

2. Execução Interna por Item (Sub-fluxo)

Para cada log identificado pelo Map state, as seguintes ações são executadas sequencialmente:

    AnalyzeLogData (Pass State): Fase de transformação, filtragem ou passagem dos dados do log para o formato esperado pelas integrações seguintes.

    StoreMetrics (CloudWatch: PutMetricData): Publica métricas customizadas extraídas dos logs diretamente no Amazon CloudWatch para monitoramento e alertas.

    StoreSummary (DynamoDB: PutItem): Grava um registro/resumo do log processado em uma tabela do Amazon DynamoDB para consultas rápidas ou auditoria.

3. Consolidação e Encerramento

    Exportação: O resultado do processamento em lote é exportado para um bucket S3 de resultados (<S3_RESULT_BUCKET>).

    GenerateDailySummary (Lambda: Invoke): Uma função AWS Lambda é invocada logo após o fim do mapeamento para consolidar os resultados do dia, gerar relatórios finais ou disparar notificações.

🛠️ Tecnologias Utilizadas

    AWS Step Functions: Orquestração do fluxo de trabalho baseado em estados.

    Amazon S3: Armazenamento de objetos (entrada de logs e saída de resultados).

    Amazon CloudWatch: Coleta e rastreamento de métricas de desempenho/negócio.

    Amazon DynamoDB: Banco de dados NoSQL chave-valor para armazenamento rápido dos resumos.

    AWS Lambda: Computação serverless para execução da lógica de consolidação diária.

🚀 Como Executar ou Implantar
Pré-requisitos

    AWS CLI configurado com permissões de IAM adequadas para Step Functions, Lambda, DynamoDB, CloudWatch e S3.

    Infraestrutura básica criada (Buckets S3 e Tabela DynamoDB).

Definição ASL (Amazon States Language)

Para atualizar ou implantar este fluxo via Terraform, CloudFormation ou console da AWS, use o arquivo JSON correspondente à estrutura visual:
Bash

# Exemplo de comando caso use AWS CLI para atualizar a State Machine
aws stepfunctions update-state-machine \
    --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:ProcessLogsPipeline" \
    --definition file://statemachine.json
