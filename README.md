# 🚀 Desafio DIO - Workflows Automatizados com AWS Step Functions

## 📋 Sobre o Projeto

Este repositório documenta minha jornada de aprendizado sobre **AWS Step Functions**, consolidando conhecimentos adquiridos durante o bootcamp da DIO sobre automação de workflows na AWS.

## 🎯 Objetivos do Desafio

- ✅ Aplicar conceitos de AWS Step Functions em ambiente prático
- ✅ Documentar processos técnicos de forma clara e estruturada
- ✅ Utilizar GitHub como ferramenta de compartilhamento de conhecimento

## 📚 O que é AWS Step Functions?

AWS Step Functions é um serviço de orquestração serverless que permite coordenar múltiplos serviços AWS em workflows visuais. Com ele, é possível criar aplicações distribuídas, automatizar processos e orquestrar microserviços de forma escalável e resiliente.

### Principais Características

- **Orquestração Visual**: Interface gráfica para visualizar e editar fluxos de trabalho
- **Integração Nativa**: Conecta facilmente com Lambda, ECS, SNS, SQS, DynamoDB e outros serviços AWS
- **Gerenciamento de Estado**: Mantém o estado da execução automaticamente
- **Tratamento de Erros**: Retry automático, catching de erros e fallback
- **Escalabilidade**: Executa milhares de workflows simultaneamente

## 🔧 Conceitos Fundamentais

### 1. State Machine (Máquina de Estados)

Uma state machine define o workflow através de estados conectados. Cada estado representa uma etapa do processo.

**Tipos de Estados:**

- **Task**: Executa uma ação (invocar Lambda, enviar mensagem SNS, etc.)
- **Choice**: Adiciona lógica condicional (if/else)
- **Parallel**: Executa múltiplos branches simultaneamente
- **Wait**: Adiciona delay no workflow
- **Succeed/Fail**: Estados terminais de sucesso ou falha
- **Pass**: Passa dados sem processamento
- **Map**: Itera sobre arrays de dados

### 2. Amazon States Language (ASL)

Linguagem JSON usada para definir state machines. Exemplo básico:

```json
{
  "Comment": "Exemplo simples de workflow",
  "StartAt": "HelloWorld",
  "States": {
    "HelloWorld": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:HelloWorld",
      "End": true
    }
  }
}
```

### 3. Tipos de Workflow

#### Standard Workflows
- **Duração**: Até 1 ano
- **Uso**: Workflows de longa duração
- **Preço**: Por transição de estado
- **Execução**: Exatamente uma vez

#### Express Workflows
- **Duração**: Até 5 minutos
- **Uso**: Alto volume, event processing
- **Preço**: Por duração e memória
- **Execução**: Pelo menos uma vez

## 💡 Casos de Uso Práticos

### 1. Pipeline de Processamento de Dados

```
Receber Arquivo S3 → Validar Dados → Processar com Lambda → 
Salvar no DynamoDB → Notificar via SNS
```

### 2. Automação de Deploy

```
Iniciar Build → Executar Testes → Deploy Staging → 
Aprovação Manual → Deploy Production → Notificação
```

### 3. Processamento de Pedidos

```
Receber Pedido → Verificar Estoque → Processar Pagamento → 
Enviar Email → Atualizar Inventário
```

### 4. ETL Pipeline

```
Extract (S3) → Transform (Lambda/Glue) → Load (Redshift/RDS) → 
Validar Qualidade → Enviar Relatório
```

## 🎨 Padrões de Design

### 1. Try-Catch-Finally Pattern

Implementa tratamento de erros robusto:

```json
{
  "Type": "Task",
  "Resource": "arn:aws:lambda:...",
  "Retry": [{
    "ErrorEquals": ["States.TaskFailed"],
    "IntervalSeconds": 2,
    "MaxAttempts": 3,
    "BackoffRate": 2.0
  }],
  "Catch": [{
    "ErrorEquals": ["States.ALL"],
    "Next": "HandleError"
  }]
}
```

### 2. Saga Pattern

Para transações distribuídas com compensação:

```
Operação 1 → Operação 2 → Operação 3
    ↓            ↓            ↓
Compensar 1  Compensar 2  Compensar 3
```

### 3. Fan-Out/Fan-In Pattern

Processamento paralelo com agregação:

```json
{
  "Type": "Parallel",
  "Branches": [
    {"StartAt": "Process1", ...},
    {"StartAt": "Process2", ...},
    {"StartAt": "Process3", ...}
  ],
  "Next": "AggregateResults"
}
```

## 🛠️ Integrações com Serviços AWS

### Serviços Integrados Nativamente

| Serviço | Uso Comum |
|---------|-----------|
| **Lambda** | Processamento de lógica de negócio |
| **DynamoDB** | Operações CRUD em banco NoSQL |
| **SNS/SQS** | Notificações e mensageria |
| **ECS/Fargate** | Execução de containers |
| **Glue** | Processamento ETL |
| **SageMaker** | Machine Learning workflows |
| **Batch** | Jobs batch de longa duração |
| **EventBridge** | Event-driven workflows |

### Exemplo de Integração com DynamoDB

```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::dynamodb:putItem",
  "Parameters": {
    "TableName": "MyTable",
    "Item": {
      "id": {"S.$": "$.orderId"},
      "status": {"S": "PROCESSING"}
    }
  },
  "Next": "NextState"
}
```

## 📊 Monitoramento e Logs

### CloudWatch Integration

- **Logs de Execução**: Rastreamento detalhado de cada step
- **Métricas**: ExecutionTime, ExecutionsFailed, ExecutionsSucceeded
- **Alarmes**: Configuração de alertas para falhas

### X-Ray Tracing

Ativa rastreamento distribuído para análise de performance:

```json
{
  "TracingConfiguration": {
    "Enabled": true
  }
}
```

## 💰 Otimização de Custos

### Boas Práticas

1. **Use Express Workflows** para alto volume e curta duração
2. **Minimize Transições de Estado** consolidando lógica quando possível
3. **Implemente Wait States** eficientemente para reduzir polling
4. **Use Map State** ao invés de múltiplas iterações paralelas
5. **Configure Timeouts** adequados para evitar execuções penduradas

### Exemplo de Cálculo

**Standard Workflow:**
- 4.000 transições de estado por execução
- 1.000 execuções por mês
- Total: 4 milhões de transições
- Custo: ~$100/mês (primeiros 4k grátis no free tier)

## 🔐 Segurança e Compliance

### IAM Roles e Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "lambda:InvokeFunction",
      "dynamodb:PutItem",
      "sns:Publish"
    ],
    "Resource": "*"
  }]
}
```

### Encryption

- **Em Trânsito**: TLS/HTTPS automático
- **Em Repouso**: KMS integration para dados sensíveis
- **Logs**: CloudWatch Logs encryption

## 📖 Recursos de Aprendizado

### Documentação Oficial
- [AWS Step Functions Documentation](https://docs.aws.amazon.com/step-functions/)
- [Amazon States Language Specification](https://states-language.net/spec.html)
- [AWS Step Functions Workshop](https://catalog.workshops.aws/stepfunctions)

### Tutoriais Práticos
- [Building Serverless Workflows](https://aws.amazon.com/getting-started/hands-on/create-a-serverless-workflow-step-functions-lambda/)
- [Step Functions Samples](https://github.com/aws-samples/aws-stepfunctions-examples)

### Cursos e Certificações
- AWS Certified Developer - Associate
- AWS Certified Solutions Architect

## 🚀 Próximos Passos

- [ ] Implementar projeto prático completo
- [ ] Explorar Step Functions com EventBridge
- [ ] Criar workflows de Machine Learning com SageMaker
- [ ] Estudar padrões avançados de orquestração
- [ ] Preparar para certificação AWS

## 📝 Anotações Pessoais

### Insights Importantes

1. **Idempotência é Crucial**: Sempre projete funções Lambda para serem idempotentes
2. **Trate Erros Adequadamente**: Use Retry e Catch em todos os Task states
3. **Visualize Primeiro**: Use o Workflow Studio para prototipar rapidamente
4. **Teste Localmente**: Use Step Functions Local para desenvolvimento
5. **Monitore Sempre**: Configure CloudWatch Alarms desde o início

### Desafios Encontrados

- **Limite de Payload**: 256KB - Use S3 para dados maiores
- **Timeout Padrão**: Configure timeouts apropriados para cada task
- **Versionamento**: Mantenha versões das state machines para rollback

### Melhores Práticas Aprendidas

- Sempre adicione comentários nas definições JSON
- Use variáveis de ambiente para ARNs de recursos
- Implemente circuit breakers para serviços externos
- Documente as decisões de arquitetura
- Faça code reviews das definições de workflow

## 🤝 Contribuições

Este repositório faz parte do meu processo de aprendizado. Sugestões e feedback são sempre bem-vindos!
