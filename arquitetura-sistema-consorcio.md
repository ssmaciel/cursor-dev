# Documento Arquitetural - Sistema de Administradoras de Consórcio

## 1. Visão Geral

### 1.1 Objetivo
Sistema completo para gestão de administradoras de consórcio, desenvolvido com arquitetura de microserviços, deployado em Kubernetes e utilizando tecnologias modernas para garantir escalabilidade, segurança e compliance com regulamentações da CVM.

### 1.2 Características Principais
- **Multi-tenant**: Cada cliente possui instalação isolada
- **Escalável**: Arquitetura de microserviços
- **Seguro**: Compliance com LGPD e regulamentações CVM
- **Disponível**: Alta disponibilidade com Kubernetes
- **Auditável**: Rastreabilidade completa de operações

## 2. Stack Tecnológico

### 2.1 Tecnologias Core
- **.NET 8**: Framework principal para desenvolvimento
- **PostgreSQL 15+**: Banco de dados principal
- **Redis 7+**: Cache, sessões e locks distribuídos
- **RabbitMQ 3.12+**: Message broker para comunicação assíncrona
- **gRPC**: Comunicação interna entre microserviços
- **MinIO**: Object storage para documentos e arquivos

### 2.2 Infraestrutura
- **Kubernetes 1.28+**: Orquestração de containers
- **Docker**: Containerização
- **Helm**: Gerenciamento de deployments
- **NGINX Ingress**: Load balancer e proxy reverso

## 3. Arquitetura de Microserviços

### 3.1 Serviços Principais

#### 3.1.1 API Gateway
- **Tecnologia**: .NET 8 + Ocelot + gRPC Client
- **Responsabilidades**:
  - Roteamento de requisições HTTP/REST
  - Proxy para serviços gRPC internos
  - Autenticação e autorização
  - Rate limiting
  - Logging centralizado
  - CORS e segurança
  - Load balancing para gRPC

#### 3.1.2 Serviço de Autenticação (Identity Service)
- **Tecnologia**: .NET 8 + IdentityServer4/OpenIddict
- **Responsabilidades**:
  - Gerenciamento de usuários
  - Autenticação JWT
  - Autorização baseada em roles
  - Refresh tokens
  - Integração com Active Directory (opcional)

#### 3.1.3 Serviço de Consorciados
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - CRUD de consorciados via gRPC
  - Validação de dados pessoais
  - Histórico de participações
  - Integração com serviços de validação
  - Cache com Redis
  - Locks distribuídos para operações críticas

#### 3.1.4 Serviço de Grupos de Consórcio
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Gestão de grupos ativos via gRPC
  - Configuração de regras
  - Cronogramas de pagamento
  - Controle de vagas
  - Cache com Redis
  - Locks para operações de grupo

#### 3.1.5 Serviço Financeiro
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Gestão de mensalidades via gRPC
  - Controle de inadimplência
  - Relatórios financeiros
  - Integração bancária
  - Cálculos de juros e multas
  - Locks distribuídos para transações críticas

#### 3.1.6 Serviço de Sorteios e Lances
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Controle de sorteios via gRPC
  - Gestão de lances com locks distribuídos
  - Processo de contemplação
  - Auditoria de sorteios
  - Prevenção de execução simultânea de sorteios

#### 3.1.7 Serviço de Documentos
- **Tecnologia**: .NET 8 + MinIO SDK + gRPC Server
- **Responsabilidades**:
  - Upload/download de documentos via gRPC
  - Geração de contratos
  - Versionamento de documentos
  - Compliance com LGPD
  - Locks para operações de arquivo

#### 3.1.8 Serviço de Relatórios
- **Tecnologia**: .NET 8 + Dapper + Redis + gRPC Server
- **Responsabilidades**:
  - Geração de relatórios via gRPC
  - Cache de consultas complexas
  - Exportação (PDF, Excel)
  - Dashboards em tempo real
  - Locks distribuídos para relatórios pesados

#### 3.1.9 Serviço de Notificações
- **Tecnologia**: .NET 8 + RabbitMQ + gRPC Server
- **Responsabilidades**:
  - Envio de emails via gRPC
  - Notificações push
  - SMS
  - Webhooks
  - Processamento assíncrono de notificações

### 3.2 Serviços de Apoio

#### 3.2.1 Serviço de Auditoria
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Log de todas as operações via gRPC
  - Rastreabilidade de mudanças
  - Compliance com CVM
  - Relatórios de auditoria
  - Event sourcing para auditoria

#### 3.2.2 Serviço de Configuração
- **Tecnologia**: .NET 8 + Redis + gRPC Server
- **Responsabilidades**:
  - Configurações dinâmicas via gRPC
  - Feature flags
  - Parâmetros do sistema
  - Cache de configurações
  - Locks para atualizações de configuração

## 4. Arquitetura de Dados

### 4.1 Estratégia de Banco de Dados
- **Database per Service**: Cada microserviço possui seu próprio banco
- **Saga Pattern**: Para transações distribuídas
- **Event Sourcing**: Para auditoria e rastreabilidade
- **CQRS**: Separação de comandos e consultas

### 4.2 Estrutura de Dados

#### 4.2.1 Banco de Consorciados
```sql
-- Tabelas principais
- Consorciados
- Enderecos
- Contatos
- Documentos
- HistoricoParticipacoes
```

#### 4.2.2 Banco de Grupos
```sql
-- Tabelas principais
- GruposConsorcio
- RegrasGrupo
- Cronogramas
- Vagas
- ParticipantesGrupo
```

#### 4.2.3 Banco Financeiro
```sql
-- Tabelas principais
- Mensalidades
- Pagamentos
- Inadimplencias
- Taxas
- MovimentacoesFinanceiras
```

#### 4.2.4 Banco de Sorteios
```sql
-- Tabelas principais
- Sorteios
- Lances
- Contemplacoes
- ResultadosSorteio
- AuditoriaSorteios
```

### 4.3 Redis - Cache e Locks Distribuídos

#### 4.3.1 Estrutura de Cache
```json
{
  "cache:consorciado:{id}": "dados_consorciado",
  "cache:grupo:{id}": "dados_grupo",
  "cache:relatorio:{tipo}:{filtros}": "dados_relatorio",
  "session:{token}": "dados_sessao",
  "config:{chave}": "valor_configuracao"
}
```

#### 4.3.2 Locks Distribuídos
```json
{
  "lock:sorteio:{grupo_id}": "executando_sorteio",
  "lock:lance:{consorciado_id}": "processando_lance",
  "lock:pagamento:{id}": "processando_pagamento",
  "lock:contemplacao:{grupo_id}": "processando_contemplacao",
  "lock:relatorio:{tipo}": "gerando_relatorio"
}
```

#### 4.3.3 Implementação de Locks
```csharp
// Exemplo de implementação de lock distribuído
public class DistributedLockService
{
    private readonly IDatabase _database;
    private readonly TimeSpan _defaultExpiry = TimeSpan.FromMinutes(5);

    public async Task<bool> AcquireLockAsync(string key, string value, TimeSpan? expiry = null)
    {
        var expiryTime = expiry ?? _defaultExpiry;
        return await _database.StringSetAsync($"lock:{key}", value, expiryTime, When.NotExists);
    }

    public async Task<bool> ReleaseLockAsync(string key, string value)
    {
        const string script = @"
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end";
        
        var result = await _database.ScriptEvaluateAsync(script, 
            new RedisKey[] { $"lock:{key}" }, 
            new RedisValue[] { value });
        
        return (int)result == 1;
    }

    public async Task<T> ExecuteWithLockAsync<T>(string key, string value, Func<Task<T>> operation)
    {
        if (await AcquireLockAsync(key, value))
        {
            try
            {
                return await operation();
            }
            finally
            {
                await ReleaseLockAsync(key, value);
            }
        }
        throw new InvalidOperationException($"Could not acquire lock for key: {key}");
    }
}
```

#### 4.3.4 Casos de Uso para Locks
- **Sorteios**: Evitar execução simultânea de sorteios no mesmo grupo
- **Lances**: Processar lances de forma sequencial
- **Pagamentos**: Evitar processamento duplicado de pagamentos
- **Contemplações**: Garantir processamento único de contemplações
- **Relatórios**: Evitar geração simultânea de relatórios pesados

## 5. Comunicação entre Serviços

### 5.1 Padrões de Comunicação
- **gRPC**: Comunicação interna síncrona entre microserviços
- **HTTP/REST**: APIs públicas e integrações externas
- **Assíncrona**: RabbitMQ para processamento em background
- **Event-Driven**: Eventos para notificações e auditoria

### 5.2 gRPC - Comunicação Interna

#### 5.2.1 Serviços com gRPC
- **Consorciado Service**: Operações CRUD e consultas complexas
- **Financeiro Service**: Cálculos financeiros e validações
- **Grupo Service**: Gestão de grupos e regras
- **Sorteio Service**: Processamento de sorteios e lances
- **Auditoria Service**: Logging e rastreabilidade

#### 5.2.2 Definições de Serviços gRPC
```protobuf
// ConsorciadoService.proto
service ConsorciadoService {
  rpc CriarConsorciado(CriarConsorciadoRequest) returns (ConsorciadoResponse);
  rpc BuscarConsorciado(BuscarConsorciadoRequest) returns (ConsorciadoResponse);
  rpc AtualizarConsorciado(AtualizarConsorciadoRequest) returns (ConsorciadoResponse);
  rpc ListarConsorciados(ListarConsorciadosRequest) returns (stream ConsorciadoResponse);
  rpc ValidarConsorciado(ValidarConsorciadoRequest) returns (ValidacaoResponse);
}

// FinanceiroService.proto
service FinanceiroService {
  rpc CalcularMensalidade(CalcularMensalidadeRequest) returns (MensalidadeResponse);
  rpc ProcessarPagamento(ProcessarPagamentoRequest) returns (PagamentoResponse);
  rpc VerificarInadimplencia(VerificarInadimplenciaRequest) returns (InadimplenciaResponse);
  rpc GerarRelatorioFinanceiro(RelatorioRequest) returns (stream RelatorioResponse);
}

// SorteioService.proto
service SorteioService {
  rpc ExecutarSorteio(ExecutarSorteioRequest) returns (SorteioResponse);
  rpc ProcessarLance(ProcessarLanceRequest) returns (LanceResponse);
  rpc ContemplarConsorciado(ContemplarRequest) returns (ContemplacaoResponse);
  rpc AuditorarSorteio(AuditoriaRequest) returns (AuditoriaResponse);
}
```

#### 5.2.3 Vantagens do gRPC
- **Performance**: Até 7x mais rápido que REST
- **Type Safety**: Contratos definidos com Protocol Buffers
- **Streaming**: Suporte a streaming bidirecional
- **Load Balancing**: Suporte nativo a load balancing
- **Compressão**: Compressão automática de dados

### 5.3 Eventos Principais
```csharp
// Exemplos de eventos
- ConsorciadoCriado
- PagamentoRealizado
- SorteioExecutado
- ContemplacaoProcessada
- DocumentoUploadado
- InadimplenciaDetectada
```

### 5.4 Message Queues (RabbitMQ)
- **consorciado.queue**: Eventos de consorciados
- **financeiro.queue**: Eventos financeiros
- **sorteio.queue**: Eventos de sorteios
- **notificacao.queue**: Eventos de notificação
- **auditoria.queue**: Eventos de auditoria

## 6. Segurança

### 6.1 Autenticação e Autorização
- **JWT Tokens**: Autenticação stateless
- **RBAC**: Role-Based Access Control
- **OAuth 2.0**: Para integrações externas
- **Multi-Factor Authentication**: Opcional

### 6.2 Criptografia
- **TLS 1.3**: Comunicação entre serviços
- **AES-256**: Criptografia de dados sensíveis
- **RSA-2048**: Chaves assimétricas
- **Hashing**: Senhas com bcrypt

### 6.3 Compliance
- **LGPD**: Proteção de dados pessoais
- **CVM**: Regulamentações específicas
- **Auditoria**: Logs completos e imutáveis
- **Backup**: Estratégia de backup e recuperação

## 7. Deploy no Kubernetes

### 7.1 Estrutura de Namespaces
```yaml
# Namespaces por ambiente
- consorcio-prod
- consorcio-staging
- consorcio-dev
- consorcio-monitoring
```

### 7.2 ConfigMaps e Secrets
```yaml
# ConfigMaps para configurações
- app-config
- database-config
- redis-config
- rabbitmq-config

# Secrets para dados sensíveis
- database-credentials
- jwt-secrets
- api-keys
- ssl-certificates
```

### 7.3 Deployments e Services
```yaml
# Exemplo de deployment com gRPC
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consorciado-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: consorciado-service
  template:
    spec:
      containers:
      - name: consorciado-service
        image: consorcio/consorciado-service:latest
        ports:
        - containerPort: 80
          name: http
        - containerPort: 50051
          name: grpc
        env:
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: connection-string
        - name: Redis__ConnectionString
          valueFrom:
            secretKeyRef:
              name: redis-credentials
              key: connection-string
        - name: Grpc__Port
          value: "50051"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: consorciado-service
spec:
  selector:
    app: consorciado-service
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: grpc
    port: 50051
    targetPort: 50051
  type: ClusterIP
```

### 7.4 Ingress e Load Balancing
```yaml
# NGINX Ingress para HTTP/REST
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: consorcio-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/grpc-backend: "true"
spec:
  rules:
  - host: api.consorcio.com
    http:
      paths:
      - path: /consorciados
        pathType: Prefix
        backend:
          service:
            name: consorciado-service
            port:
              number: 80
---
# Service para gRPC interno
apiVersion: v1
kind: Service
metadata:
  name: consorciado-grpc
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  selector:
    app: consorciado-service
  ports:
  - name: grpc
    port: 50051
    targetPort: 50051
  type: ClusterIP
```

## 8. Monitoramento e Observabilidade

### 8.1 Logging
- **Serilog**: Logging estruturado
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Correlation IDs**: Rastreamento de requisições
- **gRPC Logging**: Logs específicos para chamadas gRPC
- **Redis Logging**: Logs de operações de cache e locks

### 8.2 Métricas
- **Prometheus**: Coleta de métricas
- **Grafana**: Dashboards e alertas
- **Health Checks**: Verificação de saúde dos serviços
- **gRPC Metrics**: Métricas de performance gRPC
- **Redis Metrics**: Métricas de cache e locks distribuídos

### 8.3 Tracing
- **OpenTelemetry**: Distributed tracing
- **Jaeger**: Visualização de traces
- **Performance Monitoring**: APM
- **gRPC Tracing**: Rastreamento de chamadas gRPC
- **Redis Tracing**: Rastreamento de operações de cache

## 9. CI/CD Pipeline

### 9.1 Estrutura do Pipeline
```yaml
# Exemplo de pipeline GitLab CI/CD
stages:
  - build
  - test
  - security
  - package
  - deploy

build:
  stage: build
  script:
    - dotnet restore
    - dotnet build
    - dotnet publish

test:
  stage: test
  script:
    - dotnet test
    - dotnet test --collect:"XPlat Code Coverage"

security:
  stage: security
  script:
    - dotnet list package --vulnerable
    - security-scan

package:
  stage: package
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy:
  stage: deploy
  script:
    - helm upgrade --install consorcio ./helm-charts
```

### 9.2 Estratégia de Deploy
- **Blue-Green**: Para produção
- **Rolling Update**: Para staging
- **Canary**: Para testes de carga

## 10. Backup e Disaster Recovery

### 10.1 Estratégia de Backup
- **PostgreSQL**: Backup diário com WAL archiving
- **MinIO**: Replicação automática
- **Redis**: RDB + AOF persistence
- **RabbitMQ**: Backup de configurações

### 10.2 Disaster Recovery
- **RTO**: 4 horas
- **RPO**: 1 hora
- **Multi-region**: Backup em região diferente
- **Automated Recovery**: Scripts de recuperação

## 11. Escalabilidade

### 11.1 Horizontal Scaling
- **HPA**: Horizontal Pod Autoscaler
- **VPA**: Vertical Pod Autoscaler
- **Cluster Autoscaler**: Escalabilidade do cluster

### 11.2 Performance
- **Redis**: Cache distribuído
- **Connection Pooling**: Pool de conexões
- **Async/Await**: Programação assíncrona
- **CDN**: Para arquivos estáticos

## 12. Considerações de Compliance

### 12.1 LGPD
- **Anonimização**: Dados pessoais anonimizados
- **Consentimento**: Gestão de consentimentos
- **Direito ao Esquecimento**: Exclusão de dados
- **Portabilidade**: Exportação de dados

### 12.2 CVM
- **Auditoria**: Logs imutáveis
- **Relatórios**: Relatórios obrigatórios
- **Transparência**: Acesso a informações
- **Compliance**: Verificação contínua

## 13. Roadmap de Implementação

### 13.1 Fase 1 - MVP (3 meses)
- Serviços core (Consorciados, Grupos, Financeiro) com gRPC
- API Gateway com proxy gRPC
- Autenticação básica
- Redis para cache e locks básicos
- Deploy em K8s

### 13.2 Fase 2 - Funcionalidades Avançadas (2 meses)
- Serviço de Sorteios com locks distribuídos
- Relatórios com cache Redis
- Notificações
- Documentos
- Implementação completa de gRPC

### 13.3 Fase 3 - Otimização (1 mês)
- Monitoramento gRPC e Redis
- Performance e escalabilidade
- Segurança avançada
- Compliance completo
- Locks distribuídos avançados

## 14. Conclusão

Esta arquitetura fornece uma base sólida para o desenvolvimento de um sistema robusto e escalável para administradoras de consórcio, atendendo aos requisitos de compliance, segurança e performance necessários para o domínio financeiro.

A escolha das tecnologias (.NET, PostgreSQL, Redis, RabbitMQ, MinIO, gRPC) garante:
- **Performance**: Stack otimizada com gRPC para comunicação interna ultra-rápida
- **Escalabilidade**: Arquitetura de microserviços com locks distribuídos
- **Segurança**: Compliance com regulamentações e locks para operações críticas
- **Manutenibilidade**: Código limpo com contratos gRPC bem definidos
- **Observabilidade**: Monitoramento completo incluindo gRPC e Redis
- **Concorrência**: Locks distribuídos para evitar condições de corrida
- **Consistência**: Garantia de operações atômicas com Redis locks