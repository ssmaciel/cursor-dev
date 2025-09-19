# Arquitetura de Sistema Personal Trainer

## Índice
1. [Visão Geral do Sistema](#1-visão-geral-do-sistema)
2. [Arquitetura de Alto Nível](#2-arquitetura-de-alto-nível)
3. [Arquitetura de Backend](#3-arquitetura-de-backend)
4. [Banco de Dados](#4-banco-de-dados)
5. [Integração e Mensageria](#5-integração-e-mensageria)
6. [Cache e Performance](#6-cache-e-performance)
7. [Frontend Web (Next.js)](#7-frontend-web-nextjs)
8. [Mobile (Flutter)](#8-mobile-flutter)
9. [Segurança](#9-segurança)
10. [Multi-tenancy e Personalização](#10-multi-tenancy-e-personalização)
11. [Sistema de Pagamentos](#11-sistema-de-pagamentos)
12. [Monitoramento e Observabilidade](#12-monitoramento-e-observabilidade)
13. [Deploy e DevOps](#13-deploy-e-devops)
14. [Testes](#14-testes)
15. [Documentação Técnica](#15-documentação-técnica)

---

## 1. Visão Geral do Sistema

### 1.1 Objetivos e Escopo

O sistema Personal Trainer é uma plataforma completa que conecta Personal Trainers e seus alunos, oferecendo funcionalidades de gestão de treinos, dietas, acompanhamento de evolução e comunicação. A plataforma suporta diferentes modelos de negócio através de planos de assinatura escalonados.

**Objetivos Principais:**
- Facilitar a gestão completa de clientes para Personal Trainers
- Fornecer ferramentas avançadas de acompanhamento nutricional e físico
- Suportar múltiplos modelos de monetização
- Garantir escalabilidade e performance
- Manter alta disponibilidade (99.9%)

### 1.2 Stakeholders

**Primários:**
- Personal Trainers (usuários premium)
- Alunos/Clientes (usuários finais)
- Administradores da plataforma

**Secundários:**
- Desenvolvedores e equipe técnica
- Equipe de suporte
- Parceiros de pagamento

### 1.3 Requisitos Funcionais

#### Gestão de Perfis
- Cadastro e autenticação de Personal Trainers e alunos
- Perfis personalizáveis com informações profissionais
- Upload e gestão de fotos/documentos
- Sistema de verificação de credenciais

#### Gestão de Treinos
- Criação de treinos personalizados
- Banco de exercícios com vídeos e instruções
- Programação de treinos por período
- Acompanhamento de execução e progresso
- Sistema de notas e observações

#### Gestão Nutricional
- Criação de planos alimentares
- Banco de alimentos com informações nutricionais
- Cálculo automático de macronutrientes
- Acompanhamento de consumo vs. meta
- Relatórios nutricionais

#### Comunicação
- Sistema de mensagens em tempo real
- Notificações push
- Agendamento de sessões
- Sistema de lembretes

#### Gestão de Assinaturas
- Planos escalonados (Gratuito, Básico, Premium, Enterprise)
- Sistema de pagamentos recorrentes
- Gestão de limites por plano
- Apps personalizados por Personal Trainer

### 1.4 Requisitos Não Funcionais

#### Performance
- Tempo de resposta < 200ms para APIs
- Suporte a 10.000+ usuários simultâneos
- Sincronização offline em mobile
- Cache eficiente para dados frequentes

#### Escalabilidade
- Arquitetura horizontalmente escalável
- Microserviços independentes
- Load balancing automático
- Auto-scaling baseado em demanda

#### Segurança
- Autenticação JWT com refresh tokens
- Criptografia de dados sensíveis (AES-256)
- Compliance LGPD/GDPR
- Auditoria completa de ações

#### Disponibilidade
- SLA 99.9% uptime
- Redundância em múltiplas regiões
- Backup automático diário
- Disaster recovery < 4 horas

---

## 2. Arquitetura de Alto Nível

### 2.1 Stack Tecnológica

**Backend:**
- .NET 8 (LTS) - Framework principal
- ASP.NET Core Web API - APIs RESTful
- Entity Framework Core - ORM
- MediatR - CQRS pattern
- AutoMapper - Object mapping

**Message Broker:**
- RabbitMQ - Comunicação assíncrona
- MassTransit - Abstração de mensageria

**Cache:**
- Redis 7.x - Cache distribuído
- StackExchange.Redis - Cliente .NET

**Banco de Dados:**
- PostgreSQL 15+ - Banco principal
- TimescaleDB - Extensão para dados temporais

**Frontend Web:**
- Next.js 14 - Framework React
- TypeScript - Linguagem tipada
- Tailwind CSS - Styling
- Zustand - State management

**Mobile:**
- Flutter 3.16+ - Framework multiplataforma
- Dart 3.0+ - Linguagem
- Riverpod - State management
- SQLite - Banco local

**Infraestrutura:**
- Docker - Containerização
- Kubernetes - Orquestração
- NGINX - Load balancer/Reverse proxy
- GitHub Actions - CI/CD

### 2.2 Princípios Arquiteturais

1. **Domain-Driven Design (DDD)**
   - Bounded contexts bem definidos
   - Aggregate roots para consistência
   - Value objects para imutabilidade

2. **Clean Architecture**
   - Separação clara de responsabilidades
   - Dependency inversion
   - Testabilidade garantida

3. **Microservices Architecture**
   - Serviços independentes e desacoplados
   - Comunicação via APIs REST e mensageria
   - Deploy independente

4. **Event-Driven Architecture**
   - Eventos de domínio para comunicação
   - Processamento assíncrono
   - Resilência e escalabilidade

### 2.3 Componentes Principais

```
┌─────────────────┐    ┌─────────────────┐
│   Web Frontend  │    │  Mobile Apps    │
│    (Next.js)    │    │   (Flutter)     │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          └──────────┬───────────┘
                     │
          ┌──────────▼───────────┐
          │   API Gateway        │
          │   (NGINX/Ocelot)     │
          └──────────┬───────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼────┐    ┌─────▼─────┐    ┌────▼─────┐
│ User   │    │ Training  │    │ Nutrition│
│Service │    │ Service   │    │ Service  │
└───┬────┘    └─────┬─────┘    └────┬─────┘
    │               │               │
    └───────────────┼───────────────┘
                    │
          ┌─────────▼─────────┐
          │  Message Broker   │
          │   (RabbitMQ)      │
          └─────────┬─────────┘
                    │
    ┌───────────────┼───────────────┐
    │               │               │
┌───▼────┐    ┌────▼─────┐    ┌────▼─────┐
│ Cache  │    │Database  │    │Payment   │
│(Redis) │    │(Postgres)│    │Service   │
└────────┘    └──────────┘    └──────────┘
```

---

## 3. Arquitetura de Backend

### 3.1 Estrutura de Microserviços

O sistema será dividido em microserviços especializados, cada um com responsabilidade única:

#### 3.1.1 User Service
**Responsabilidades:**
- Autenticação e autorização
- Gestão de perfis de usuários
- Controle de acesso baseado em roles
- Gestão de sessões e tokens

**Tecnologias:**
- ASP.NET Core Web API
- JWT Bearer Authentication
- Identity Framework
- Redis para sessões

#### 3.1.2 Training Service
**Responsabilidades:**
- Gestão de exercícios e treinos
- Programação de treinos
- Acompanhamento de execução
- Relatórios de progresso

**Tecnologias:**
- ASP.NET Core Web API
- Entity Framework Core
- MediatR (CQRS)
- AutoMapper

#### 3.1.3 Nutrition Service
**Responsabilidades:**
- Banco de alimentos
- Cálculo nutricional
- Planos alimentares
- Acompanhamento nutricional

**Tecnologias:**
- ASP.NET Core Web API
- Entity Framework Core
- MediatR (CQRS)
- Cálculos matemáticos para macros

#### 3.1.4 Communication Service
**Responsabilidades:**
- Sistema de mensagens
- Notificações push
- Agendamento de sessões
- WebSocket para tempo real

**Tecnologias:**
- ASP.NET Core Web API
- SignalR para WebSocket
- Background services
- Firebase Cloud Messaging

#### 3.1.5 Payment Service
**Responsabilidades:**
- Processamento de pagamentos
- Gestão de assinaturas
- Webhooks de pagamento
- Faturamento e cobrança

**Tecnologias:**
- ASP.NET Core Web API
- Stripe/PayPal SDK
- Background services
- Entity Framework Core

#### 3.1.6 Notification Service
**Responsabilidades:**
- Notificações push
- Emails transacionais
- SMS (opcional)
- Templates de mensagens

**Tecnologias:**
- ASP.NET Core Web API
- Firebase Cloud Messaging
- SendGrid/Mailgun
- Background services

### 3.2 Padrões Arquiteturais

#### 3.2.1 Clean Architecture

```
src/
├── PersonalTrainer.Domain/           # Entidades e regras de negócio
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Interfaces/
│   └── Events/
├── PersonalTrainer.Application/      # Casos de uso e lógica de aplicação
│   ├── Commands/
│   ├── Queries/
│   ├── Handlers/
│   ├── DTOs/
│   └── Interfaces/
├── PersonalTrainer.Infrastructure/   # Implementações externas
│   ├── Data/
│   ├── Services/
│   ├── Repositories/
│   └── External/
└── PersonalTrainer.API/              # Interface web
    ├── Controllers/
    ├── Middleware/
    ├── Configuration/
    └── Program.cs
```

#### 3.2.2 CQRS com MediatR

**Command Example:**
```csharp
public class CreateTrainingCommand : IRequest<TrainingDto>
{
    public Guid TrainerId { get; set; }
    public Guid ClientId { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public List<ExerciseDto> Exercises { get; set; }
}

public class CreateTrainingHandler : IRequestHandler<CreateTrainingCommand, TrainingDto>
{
    private readonly ITrainingRepository _repository;
    private readonly IMapper _mapper;
    private readonly IMediator _mediator;

    public async Task<TrainingDto> Handle(CreateTrainingCommand request, CancellationToken cancellationToken)
    {
        var training = new Training(
            request.TrainerId,
            request.ClientId,
            request.Name,
            request.Description
        );

        foreach (var exerciseDto in request.Exercises)
        {
            training.AddExercise(new Exercise(
                exerciseDto.Name,
                exerciseDto.Sets,
                exerciseDto.Repetitions,
                exerciseDto.Weight,
                exerciseDto.RestTime
            ));
        }

        await _repository.AddAsync(training);
        
        // Publish domain event
        await _mediator.Publish(new TrainingCreatedEvent(training.Id), cancellationToken);
        
        return _mapper.Map<TrainingDto>(training);
    }
}
```

#### 3.2.3 Repository Pattern

```csharp
public interface ITrainingRepository
{
    Task<Training> GetByIdAsync(Guid id);
    Task<IEnumerable<Training>> GetByTrainerIdAsync(Guid trainerId);
    Task<IEnumerable<Training>> GetByClientIdAsync(Guid clientId);
    Task AddAsync(Training training);
    Task UpdateAsync(Training training);
    Task DeleteAsync(Guid id);
}

public class TrainingRepository : ITrainingRepository
{
    private readonly TrainingDbContext _context;

    public async Task<Training> GetByIdAsync(Guid id)
    {
        return await _context.Trainings
            .Include(t => t.Exercises)
            .FirstOrDefaultAsync(t => t.Id == id);
    }
    
    // ... other implementations
}
```

### 3.3 APIs e Contratos

#### 3.3.1 RESTful API Design

**Training Service Endpoints:**
```
GET    /api/trainings                    # Listar treinos
GET    /api/trainings/{id}              # Obter treino específico
POST   /api/trainings                   # Criar novo treino
PUT    /api/trainings/{id}              # Atualizar treino
DELETE /api/trainings/{id}              # Deletar treino
GET    /api/trainings/trainer/{id}      # Treinos por personal
GET    /api/trainings/client/{id}       # Treinos por cliente
POST   /api/trainings/{id}/execute      # Registrar execução
```

**Nutrition Service Endpoints:**
```
GET    /api/foods                       # Listar alimentos
GET    /api/foods/search?q={term}       # Buscar alimentos
GET    /api/foods/{id}                  # Obter alimento específico
POST   /api/meal-plans                  # Criar plano alimentar
PUT    /api/meal-plans/{id}             # Atualizar plano
GET    /api/meal-plans/client/{id}      # Planos por cliente
POST   /api/nutrition/calculate         # Calcular macros
```

#### 3.3.2 API Versioning

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class TrainingsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public async Task<ActionResult<IEnumerable<TrainingDto>>> GetTrainings()
    {
        // Implementation
    }
}
```

### 3.4 Estrutura de Projeto

```
PersonalTrainer.Training.Service/
├── src/
│   ├── PersonalTrainer.Training.Domain/
│   │   ├── Entities/
│   │   │   ├── Training.cs
│   │   │   ├── Exercise.cs
│   │   │   └── TrainingExecution.cs
│   │   ├── ValueObjects/
│   │   │   ├── ExerciseDetails.cs
│   │   │   └── SetDetails.cs
│   │   ├── Interfaces/
│   │   │   └── ITrainingRepository.cs
│   │   └── Events/
│   │       └── TrainingCreatedEvent.cs
│   ├── PersonalTrainer.Training.Application/
│   │   ├── Commands/
│   │   │   ├── CreateTraining/
│   │   │   └── UpdateTraining/
│   │   ├── Queries/
│   │   │   ├── GetTrainingById/
│   │   │   └── GetTrainingsByTrainer/
│   │   ├── DTOs/
│   │   │   ├── TrainingDto.cs
│   │   │   └── ExerciseDto.cs
│   │   └── Interfaces/
│   │       └── ITrainingService.cs
│   ├── PersonalTrainer.Training.Infrastructure/
│   │   ├── Data/
│   │   │   ├── TrainingDbContext.cs
│   │   │   ├── Configurations/
│   │   │   └── Repositories/
│   │   ├── Services/
│   │   │   └── TrainingService.cs
│   │   └── External/
│   │       └── MessageBus/
│   └── PersonalTrainer.Training.API/
│       ├── Controllers/
│       ├── Middleware/
│       ├── Configuration/
│       └── Program.cs
├── tests/
│   ├── PersonalTrainer.Training.UnitTests/
│   ├── PersonalTrainer.Training.IntegrationTests/
│   └── PersonalTrainer.Training.E2ETests/
└── docs/
    ├── api/
    └── architecture/
```

---

## 4. Banco de Dados

### 4.1 Modelagem de Dados

#### 4.1.1 Entidades Principais

**Users (Usuários)**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    user_type user_type_enum NOT NULL, -- 'TRAINER', 'CLIENT'
    phone VARCHAR(20),
    birth_date DATE,
    gender gender_enum,
    profile_image_url TEXT,
    is_verified BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    subscription_plan subscription_plan_enum DEFAULT 'FREE',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Trainers (Personal Trainers)**
```sql
CREATE TABLE trainers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    specialties TEXT[],
    certifications JSONB,
    experience_years INTEGER,
    hourly_rate DECIMAL(10,2),
    max_clients INTEGER DEFAULT 1,
    current_clients_count INTEGER DEFAULT 0,
    rating DECIMAL(3,2) DEFAULT 0.0,
    total_reviews INTEGER DEFAULT 0,
    is_featured BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Clients (Alunos)**
```sql
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    trainer_id UUID REFERENCES trainers(id) ON DELETE SET NULL,
    height_cm INTEGER,
    weight_kg DECIMAL(5,2),
    activity_level activity_level_enum,
    fitness_goals TEXT[],
    medical_conditions TEXT[],
    dietary_restrictions TEXT[],
    emergency_contact JSONB,
    subscription_status subscription_status_enum DEFAULT 'ACTIVE',
    subscription_start_date DATE,
    subscription_end_date DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Trainings (Treinos)**
```sql
CREATE TABLE trainings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trainer_id UUID REFERENCES trainers(id) ON DELETE CASCADE,
    client_id UUID REFERENCES clients(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    difficulty_level difficulty_level_enum,
    estimated_duration_minutes INTEGER,
    target_muscle_groups TEXT[],
    is_active BOOLEAN DEFAULT TRUE,
    scheduled_date DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Exercises (Exercícios)**
```sql
CREATE TABLE exercises (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    training_id UUID REFERENCES trainings(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    muscle_groups TEXT[],
    equipment_needed TEXT[],
    video_url TEXT,
    image_url TEXT,
    instructions TEXT[],
    tips TEXT[],
    order_index INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Exercise Sets (Séries de Exercícios)**
```sql
CREATE TABLE exercise_sets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    exercise_id UUID REFERENCES exercises(id) ON DELETE CASCADE,
    sets INTEGER NOT NULL,
    repetitions INTEGER,
    weight_kg DECIMAL(5,2),
    duration_seconds INTEGER, -- Para exercícios de tempo
    rest_seconds INTEGER,
    distance_meters DECIMAL(8,2), -- Para cardio
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Food Database (Banco de Alimentos)**
```sql
CREATE TABLE foods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    brand VARCHAR(100),
    category food_category_enum,
    serving_size DECIMAL(8,2) NOT NULL,
    serving_unit VARCHAR(50) NOT NULL,
    calories_per_serving DECIMAL(8,2) NOT NULL,
    protein_g DECIMAL(8,2) NOT NULL,
    carbs_g DECIMAL(8,2) NOT NULL,
    fat_g DECIMAL(8,2) NOT NULL,
    fiber_g DECIMAL(8,2) DEFAULT 0,
    sugar_g DECIMAL(8,2) DEFAULT 0,
    sodium_mg DECIMAL(8,2) DEFAULT 0,
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Meal Plans (Planos Alimentares)**
```sql
CREATE TABLE meal_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trainer_id UUID REFERENCES trainers(id) ON DELETE CASCADE,
    client_id UUID REFERENCES clients(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    total_calories DECIMAL(8,2),
    total_protein_g DECIMAL(8,2),
    total_carbs_g DECIMAL(8,2),
    total_fat_g DECIMAL(8,2),
    start_date DATE NOT NULL,
    end_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Meals (Refeições)**
```sql
CREATE TABLE meals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    meal_plan_id UUID REFERENCES meal_plans(id) ON DELETE CASCADE,
    meal_type meal_type_enum NOT NULL, -- 'BREAKFAST', 'LUNCH', 'DINNER', 'SNACK'
    name VARCHAR(200) NOT NULL,
    scheduled_time TIME,
    order_index INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Meal Items (Itens das Refeições)**
```sql
CREATE TABLE meal_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    meal_id UUID REFERENCES meals(id) ON DELETE CASCADE,
    food_id UUID REFERENCES foods(id) ON DELETE CASCADE,
    quantity DECIMAL(8,2) NOT NULL,
    unit VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### 4.1.2 Enums e Tipos

```sql
-- Enums
CREATE TYPE user_type_enum AS ENUM ('TRAINER', 'CLIENT');
CREATE TYPE gender_enum AS ENUM ('MALE', 'FEMALE', 'OTHER');
CREATE TYPE subscription_plan_enum AS ENUM ('FREE', 'BASIC', 'PREMIUM', 'ENTERPRISE');
CREATE TYPE subscription_status_enum AS ENUM ('ACTIVE', 'INACTIVE', 'CANCELLED', 'EXPIRED');
CREATE TYPE activity_level_enum AS ENUM ('SEDENTARY', 'LIGHT', 'MODERATE', 'ACTIVE', 'VERY_ACTIVE');
CREATE TYPE difficulty_level_enum AS ENUM ('BEGINNER', 'INTERMEDIATE', 'ADVANCED', 'EXPERT');
CREATE TYPE food_category_enum AS ENUM ('PROTEIN', 'CARBOHYDRATE', 'FAT', 'VEGETABLE', 'FRUIT', 'DAIRY', 'GRAIN', 'BEVERAGE');
CREATE TYPE meal_type_enum AS ENUM ('BREAKFAST', 'LUNCH', 'DINNER', 'SNACK');
```

### 4.2 Estratégias de Indexação

#### 4.2.1 Índices Primários
```sql
-- Índices para consultas frequentes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_user_type ON users(user_type);
CREATE INDEX idx_trainers_user_id ON trainers(user_id);
CREATE INDEX idx_clients_trainer_id ON clients(trainer_id);
CREATE INDEX idx_trainings_trainer_client ON trainings(trainer_id, client_id);
CREATE INDEX idx_trainings_scheduled_date ON trainings(scheduled_date);
CREATE INDEX idx_exercises_training_id ON exercises(training_id);
CREATE INDEX idx_foods_name ON foods USING gin(to_tsvector('portuguese', name));
CREATE INDEX idx_foods_category ON foods(category);
CREATE INDEX idx_meal_plans_client_id ON meal_plans(client_id);
CREATE INDEX idx_meal_plans_active ON meal_plans(is_active) WHERE is_active = true;
```

#### 4.2.2 Índices Compostos
```sql
-- Índices para consultas complexas
CREATE INDEX idx_trainings_trainer_date ON trainings(trainer_id, scheduled_date);
CREATE INDEX idx_exercise_sets_exercise_id ON exercise_sets(exercise_id);
CREATE INDEX idx_meal_items_meal_food ON meal_items(meal_id, food_id);
```

### 4.3 Backup e Recuperação

#### 4.3.1 Estratégia de Backup
```bash
#!/bin/bash
# Script de backup automático

# Backup completo diário
pg_dump -h localhost -U postgres -d personal_trainer_db \
  --format=custom \
  --compress=9 \
  --file="/backup/personal_trainer_$(date +%Y%m%d).dump"

# Backup incremental (WAL)
# Configuração no postgresql.conf:
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /backup/wal/%f'

# Retenção de 30 dias
find /backup -name "*.dump" -mtime +30 -delete
```

#### 4.3.2 Disaster Recovery
```sql
-- Procedimento de recuperação
-- 1. Parar aplicação
-- 2. Restaurar backup mais recente
pg_restore -h localhost -U postgres -d personal_trainer_db_new \
  --clean --if-exists \
  /backup/personal_trainer_20240101.dump

-- 3. Aplicar WAL files se necessário
-- 4. Validar integridade dos dados
-- 5. Atualizar DNS/load balancer
```

---

## 5. Integração e Mensageria

### 5.1 RabbitMQ Configuration

#### 5.1.1 Estrutura de Filas

```yaml
# docker-compose.yml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: personal-trainer-rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: password
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - personal-trainer-network

volumes:
  rabbitmq_data:

networks:
  personal-trainer-network:
    driver: bridge
```

#### 5.1.2 Exchanges e Filas

```csharp
// Configuração de exchanges
public static class RabbitMQConfig
{
    public const string TRAINING_EXCHANGE = "training.exchange";
    public const string NUTRITION_EXCHANGE = "nutrition.exchange";
    public const string NOTIFICATION_EXCHANGE = "notification.exchange";
    public const string PAYMENT_EXCHANGE = "payment.exchange";
    
    // Filas
    public const string TRAINING_CREATED_QUEUE = "training.created.queue";
    public const string TRAINING_EXECUTED_QUEUE = "training.executed.queue";
    public const string MEAL_PLAN_CREATED_QUEUE = "meal.plan.created.queue";
    public const string PAYMENT_PROCESSED_QUEUE = "payment.processed.queue";
    public const string NOTIFICATION_SEND_QUEUE = "notification.send.queue";
}
```

#### 5.1.3 MassTransit Configuration

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMassTransit(x =>
    {
        x.AddConsumer<TrainingCreatedConsumer>();
        x.AddConsumer<MealPlanCreatedConsumer>();
        x.AddConsumer<PaymentProcessedConsumer>();
        
        x.UsingRabbitMq((context, cfg) =>
        {
            cfg.Host("localhost", "/", h =>
            {
                h.Username("admin");
                h.Password("password");
            });
            
            // Training Service
            cfg.Message<TrainingCreatedEvent>(e => e.SetEntityName(RabbitMQConfig.TRAINING_EXCHANGE));
            cfg.Publish<TrainingCreatedEvent>(p => p.ExchangeType = ExchangeType.Fanout);
            
            // Nutrition Service
            cfg.Message<MealPlanCreatedEvent>(e => e.SetEntityName(RabbitMQConfig.NUTRITION_EXCHANGE));
            cfg.Publish<MealPlanCreatedEvent>(p => p.ExchangeType = ExchangeType.Fanout);
            
            // Notification Service
            cfg.Message<NotificationRequestedEvent>(e => e.SetEntityName(RabbitMQConfig.NOTIFICATION_EXCHANGE));
            cfg.Publish<NotificationRequestedEvent>(p => p.ExchangeType = ExchangeType.Direct);
            
            cfg.ConfigureEndpoints(context);
        });
    });
}
```

### 5.2 Event-Driven Architecture

#### 5.2.1 Domain Events

```csharp
// Training Domain Event
public class TrainingCreatedEvent : INotification
{
    public Guid TrainingId { get; }
    public Guid TrainerId { get; }
    public Guid ClientId { get; }
    public string TrainingName { get; }
    public DateTime CreatedAt { get; }

    public TrainingCreatedEvent(Guid trainingId, Guid trainerId, Guid clientId, string trainingName)
    {
        TrainingId = trainingId;
        TrainerId = trainerId;
        ClientId = clientId;
        TrainingName = trainingName;
        CreatedAt = DateTime.UtcNow;
    }
}

// Nutrition Domain Event
public class MealPlanCreatedEvent : INotification
{
    public Guid MealPlanId { get; }
    public Guid TrainerId { get; }
    public Guid ClientId { get; }
    public decimal TotalCalories { get; }
    public DateTime CreatedAt { get; }

    public MealPlanCreatedEvent(Guid mealPlanId, Guid trainerId, Guid clientId, decimal totalCalories)
    {
        MealPlanId = mealPlanId;
        TrainerId = trainerId;
        ClientId = clientId;
        TotalCalories = totalCalories;
        CreatedAt = DateTime.UtcNow;
    }
}
```

#### 5.2.2 Event Consumers

```csharp
// Training Created Consumer
public class TrainingCreatedConsumer : IConsumer<TrainingCreatedEvent>
{
    private readonly INotificationService _notificationService;
    private readonly ILogger<TrainingCreatedConsumer> _logger;

    public async Task Consume(ConsumeContext<TrainingCreatedEvent> context)
    {
        var message = context.Message;
        
        try
        {
            // Send notification to client
            await _notificationService.SendTrainingCreatedNotification(
                message.ClientId, 
                message.TrainingName
            );
            
            _logger.LogInformation("Training created notification sent for training {TrainingId}", 
                message.TrainingId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process training created event for training {TrainingId}", 
                message.TrainingId);
            throw;
        }
    }
}
```

### 5.3 Message Patterns

#### 5.3.1 Request-Response Pattern

```csharp
// Request
public class CalculateNutritionRequest
{
    public Guid ClientId { get; set; }
    public List<MealItemDto> Meals { get; set; }
}

// Response
public class CalculateNutritionResponse
{
    public decimal TotalCalories { get; set; }
    public decimal TotalProtein { get; set; }
    public decimal TotalCarbs { get; set; }
    public decimal TotalFat { get; set; }
}

// Consumer
public class CalculateNutritionConsumer : IConsumer<CalculateNutritionRequest>
{
    public async Task Consume(ConsumeContext<CalculateNutritionRequest> context)
    {
        var request = context.Message;
        var response = await _nutritionService.CalculateNutrition(request);
        
        await context.RespondAsync(response);
    }
}
```

#### 5.3.2 Publish-Subscribe Pattern

```csharp
// Publisher
public class TrainingService
{
    private readonly IPublishEndpoint _publishEndpoint;

    public async Task CreateTrainingAsync(CreateTrainingCommand command)
    {
        // Create training logic...
        
        await _publishEndpoint.Publish(new TrainingCreatedEvent(
            training.Id,
            training.TrainerId,
            training.ClientId,
            training.Name
        ));
    }
}

// Subscribers
public class NotificationServiceConsumer : IConsumer<TrainingCreatedEvent>
{
    public async Task Consume(ConsumeContext<TrainingCreatedEvent> context)
    {
        // Send push notification
        await _pushNotificationService.SendAsync(context.Message);
    }
}

public class AnalyticsServiceConsumer : IConsumer<TrainingCreatedEvent>
{
    public async Task Consume(ConsumeContext<TrainingCreatedEvent> context)
    {
        // Update analytics
        await _analyticsService.RecordTrainingCreatedAsync(context.Message);
    }
}
```

---

## 6. Cache e Performance

### 6.1 Redis Configuration

#### 6.1.1 Redis Setup

```yaml
# docker-compose.yml
services:
  redis:
    image: redis:7-alpine
    container_name: personal-trainer-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass password
    volumes:
      - redis_data:/data
    networks:
      - personal-trainer-network

volumes:
  redis_data:
```

#### 6.1.2 StackExchange.Redis Configuration

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddStackExchangeRedisCache(options =>
    {
        options.Configuration = "localhost:6379";
        options.ConfigurationOptions = new ConfigurationOptions
        {
            EndPoints = { "localhost:6379" },
            Password = "password",
            AbortOnConnectFail = false,
            ConnectRetry = 3,
            ConnectTimeout = 5000
        };
    });
    
    services.AddSingleton<IConnectionMultiplexer>(provider =>
    {
        var config = new ConfigurationOptions
        {
            EndPoints = { "localhost:6379" },
            Password = "password",
            AbortOnConnectFail = false
        };
        return ConnectionMultiplexer.Connect(config);
    });
}
```

### 6.2 Cache Strategies

#### 6.2.1 Cache-Aside Pattern

```csharp
public class TrainingService : ITrainingService
{
    private readonly ITrainingRepository _repository;
    private readonly IDistributedCache _cache;
    private readonly ILogger<TrainingService> _logger;

    public async Task<TrainingDto> GetTrainingByIdAsync(Guid id)
    {
        var cacheKey = $"training:{id}";
        
        // Try to get from cache
        var cachedData = await _cache.GetStringAsync(cacheKey);
        if (!string.IsNullOrEmpty(cachedData))
        {
            return JsonSerializer.Deserialize<TrainingDto>(cachedData);
        }
        
        // Get from database
        var training = await _repository.GetByIdAsync(id);
        if (training == null)
            return null;
            
        var trainingDto = _mapper.Map<TrainingDto>(training);
        
        // Store in cache
        var cacheOptions = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30),
            SlidingExpiration = TimeSpan.FromMinutes(10)
        };
        
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(trainingDto), cacheOptions);
        
        return trainingDto;
    }
}
```

#### 6.2.2 Write-Through Cache

```csharp
public class FoodService : IFoodService
{
    private readonly IFoodRepository _repository;
    private readonly IDistributedCache _cache;

    public async Task<FoodDto> CreateFoodAsync(CreateFoodCommand command)
    {
        var food = new Food(command.Name, command.Calories, command.Protein);
        await _repository.AddAsync(food);
        
        var foodDto = _mapper.Map<FoodDto>(food);
        var cacheKey = $"food:{food.Id}";
        
        // Write to cache immediately
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(foodDto));
        
        return foodDto;
    }
}
```

### 6.3 Cache Invalidation

#### 6.3.1 Event-Based Invalidation

```csharp
public class CacheInvalidationHandler : INotificationHandler<TrainingUpdatedEvent>
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<CacheInvalidationHandler> _logger;

    public async Task Handle(TrainingUpdatedEvent notification, CancellationToken cancellationToken)
    {
        var cacheKey = $"training:{notification.TrainingId}";
        
        try
        {
            await _cache.RemoveAsync(cacheKey);
            _logger.LogInformation("Cache invalidated for training {TrainingId}", notification.TrainingId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to invalidate cache for training {TrainingId}", notification.TrainingId);
        }
    }
}
```

#### 6.3.2 TTL-Based Expiration

```csharp
public static class CacheKeys
{
    public static class Training
    {
        public const string ById = "training:{0}";
        public const string ByTrainer = "trainings:trainer:{0}";
        public const string ByClient = "trainings:client:{0}";
    }
    
    public static class Food
    {
        public const string ById = "food:{0}";
        public const string Search = "foods:search:{0}";
        public const string ByCategory = "foods:category:{0}";
    }
}

public static class CacheExpiration
{
    public static readonly TimeSpan Training = TimeSpan.FromMinutes(30);
    public static readonly TimeSpan Food = TimeSpan.FromHours(2);
    public static readonly TimeSpan UserProfile = TimeSpan.FromMinutes(15);
    public static readonly TimeSpan SearchResults = TimeSpan.FromMinutes(5);
}
```

---

## 7. Frontend Web (Next.js)

### 7.1 Arquitetura do Frontend

#### 7.1.1 Estrutura de Projeto

```
personal-trainer-web/
├── src/
│   ├── app/                          # App Router (Next.js 13+)
│   │   ├── (auth)/                   # Route groups
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/
│   │   │   ├── trainer/
│   │   │   │   ├── clients/
│   │   │   │   ├── trainings/
│   │   │   │   ├── nutrition/
│   │   │   │   └── analytics/
│   │   │   └── client/
│   │   │       ├── workouts/
│   │   │       ├── nutrition/
│   │   │       └── progress/
│   │   ├── api/                      # API routes
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/                   # Reusable components
│   │   ├── ui/                       # Base UI components
│   │   ├── forms/                    # Form components
│   │   ├── charts/                   # Chart components
│   │   └── layout/                   # Layout components
│   ├── lib/                          # Utilities and configurations
│   │   ├── api/                      # API client
│   │   ├── auth/                     # Authentication
│   │   ├── utils/                    # Utility functions
│   │   └── validations/              # Zod schemas
│   ├── hooks/                        # Custom React hooks
│   ├── store/                        # Zustand stores
│   └── types/                        # TypeScript types
├── public/                           # Static assets
├── tailwind.config.js
├── next.config.js
└── package.json
```

#### 7.1.2 Tecnologias e Dependências

```json
{
  "dependencies": {
    "next": "14.0.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "@next-auth/prisma-adapter": "^1.0.7",
    "next-auth": "^4.24.0",
    "zustand": "^4.4.0",
    "react-query": "^3.39.0",
    "react-hook-form": "^7.47.0",
    "@hookform/resolvers": "^3.3.0",
    "zod": "^3.22.0",
    "tailwindcss": "^3.3.0",
    "@headlessui/react": "^1.7.0",
    "@heroicons/react": "^2.0.0",
    "recharts": "^2.8.0",
    "date-fns": "^2.30.0",
    "clsx": "^2.0.0",
    "lucide-react": "^0.290.0"
  }
}
```

### 7.2 State Management (Zustand)

#### 7.2.1 Auth Store

```typescript
// store/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  userType: 'TRAINER' | 'CLIENT';
  subscriptionPlan: 'FREE' | 'BASIC' | 'PREMIUM' | 'ENTERPRISE';
}

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateUser: (user: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isLoading: false,
      isAuthenticated: false,

      login: async (email: string, password: string) => {
        set({ isLoading: true });
        try {
          const response = await authApi.login(email, password);
          set({
            user: response.user,
            token: response.token,
            isAuthenticated: true,
            isLoading: false
          });
        } catch (error) {
          set({ isLoading: false });
          throw error;
        }
      },

      logout: () => {
        set({
          user: null,
          token: null,
          isAuthenticated: false
        });
      },

      updateUser: (userData) => {
        const currentUser = get().user;
        if (currentUser) {
          set({ user: { ...currentUser, ...userData } });
        }
      }
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated
      })
    }
  )
);
```

#### 7.2.2 Training Store

```typescript
// store/trainingStore.ts
interface Training {
  id: string;
  name: string;
  description: string;
  exercises: Exercise[];
  difficultyLevel: 'BEGINNER' | 'INTERMEDIATE' | 'ADVANCED';
  estimatedDuration: number;
  targetMuscleGroups: string[];
  isActive: boolean;
}

interface TrainingState {
  trainings: Training[];
  currentTraining: Training | null;
  isLoading: boolean;
  error: string | null;
  fetchTrainings: (trainerId?: string, clientId?: string) => Promise<void>;
  fetchTrainingById: (id: string) => Promise<void>;
  createTraining: (training: Omit<Training, 'id'>) => Promise<void>;
  updateTraining: (id: string, training: Partial<Training>) => Promise<void>;
  deleteTraining: (id: string) => Promise<void>;
}

export const useTrainingStore = create<TrainingState>((set, get) => ({
  trainings: [],
  currentTraining: null,
  isLoading: false,
  error: null,

  fetchTrainings: async (trainerId?, clientId?) => {
    set({ isLoading: true, error: null });
    try {
      const trainings = await trainingApi.getTrainings({ trainerId, clientId });
      set({ trainings, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  fetchTrainingById: async (id: string) => {
    set({ isLoading: true, error: null });
    try {
      const training = await trainingApi.getTrainingById(id);
      set({ currentTraining: training, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  createTraining: async (trainingData) => {
    set({ isLoading: true, error: null });
    try {
      const newTraining = await trainingApi.createTraining(trainingData);
      set(state => ({
        trainings: [...state.trainings, newTraining],
        isLoading: false
      }));
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  updateTraining: async (id: string, trainingData) => {
    set({ isLoading: true, error: null });
    try {
      const updatedTraining = await trainingApi.updateTraining(id, trainingData);
      set(state => ({
        trainings: state.trainings.map(t => t.id === id ? updatedTraining : t),
        currentTraining: state.currentTraining?.id === id ? updatedTraining : state.currentTraining,
        isLoading: false
      }));
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  deleteTraining: async (id: string) => {
    set({ isLoading: true, error: null });
    try {
      await trainingApi.deleteTraining(id);
      set(state => ({
        trainings: state.trainings.filter(t => t.id !== id),
        currentTraining: state.currentTraining?.id === id ? null : state.currentTraining,
        isLoading: false
      }));
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  }
}));
```

### 7.3 API Client

#### 7.3.1 Base API Client

```typescript
// lib/api/client.ts
class ApiClient {
  private baseURL: string;
  private token: string | null = null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  setToken(token: string) {
    this.token = token;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    
    const config: RequestInit = {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers
      }
    };

    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new ApiError(response.status, await response.text());
    }

    return response.json();
  }

  async get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  async post<T>(endpoint: string, data?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: data ? JSON.stringify(data) : undefined
    });
  }

  async put<T>(endpoint: string, data: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  async delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

export const apiClient = new ApiClient(process.env.NEXT_PUBLIC_API_URL!);
```

#### 7.3.2 Training API

```typescript
// lib/api/trainingApi.ts
export interface CreateTrainingRequest {
  trainerId: string;
  clientId: string;
  name: string;
  description: string;
  exercises: CreateExerciseRequest[];
}

export interface CreateExerciseRequest {
  name: string;
  sets: number;
  repetitions: number;
  weight: number;
  restTime: number;
  orderIndex: number;
}

export const trainingApi = {
  async getTrainings(params?: { trainerId?: string; clientId?: string }) {
    const queryParams = new URLSearchParams();
    if (params?.trainerId) queryParams.append('trainerId', params.trainerId);
    if (params?.clientId) queryParams.append('clientId', params.clientId);
    
    const endpoint = `/api/trainings${queryParams.toString() ? `?${queryParams}` : ''}`;
    return apiClient.get<Training[]>(endpoint);
  },

  async getTrainingById(id: string) {
    return apiClient.get<Training>(`/api/trainings/${id}`);
  },

  async createTraining(data: CreateTrainingRequest) {
    return apiClient.post<Training>('/api/trainings', data);
  },

  async updateTraining(id: string, data: Partial<CreateTrainingRequest>) {
    return apiClient.put<Training>(`/api/trainings/${id}`, data);
  },

  async deleteTraining(id: string) {
    return apiClient.delete<void>(`/api/trainings/${id}`);
  },

  async executeTraining(id: string, executionData: TrainingExecutionRequest) {
    return apiClient.post<TrainingExecution>(`/api/trainings/${id}/execute`, executionData);
  }
};
```

### 7.4 Componentes Principais

#### 7.4.1 Training Form Component

```typescript
// components/forms/TrainingForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const trainingSchema = z.object({
  name: z.string().min(1, 'Nome é obrigatório'),
  description: z.string().optional(),
  difficultyLevel: z.enum(['BEGINNER', 'INTERMEDIATE', 'ADVANCED']),
  estimatedDuration: z.number().min(1, 'Duração deve ser maior que 0'),
  exercises: z.array(z.object({
    name: z.string().min(1, 'Nome do exercício é obrigatório'),
    sets: z.number().min(1, 'Número de séries deve ser maior que 0'),
    repetitions: z.number().min(1, 'Número de repetições deve ser maior que 0'),
    weight: z.number().min(0, 'Peso deve ser maior ou igual a 0'),
    restTime: z.number().min(0, 'Tempo de descanso deve ser maior ou igual a 0'),
    orderIndex: z.number()
  })).min(1, 'Pelo menos um exercício é obrigatório')
});

type TrainingFormData = z.infer<typeof trainingSchema>;

interface TrainingFormProps {
  initialData?: Partial<TrainingFormData>;
  onSubmit: (data: TrainingFormData) => Promise<void>;
  isLoading?: boolean;
}

export function TrainingForm({ initialData, onSubmit, isLoading }: TrainingFormProps) {
  const { register, handleSubmit, control, formState: { errors } } = useForm<TrainingFormData>({
    resolver: zodResolver(trainingSchema),
    defaultValues: initialData
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'exercises'
  });

  const addExercise = () => {
    append({
      name: '',
      sets: 1,
      repetitions: 1,
      weight: 0,
      restTime: 60,
      orderIndex: fields.length
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div>
        <label htmlFor="name" className="block text-sm font-medium text-gray-700">
          Nome do Treino
        </label>
        <input
          {...register('name')}
          type="text"
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
        />
        {errors.name && (
          <p className="mt-1 text-sm text-red-600">{errors.name.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="description" className="block text-sm font-medium text-gray-700">
          Descrição
        </label>
        <textarea
          {...register('description')}
          rows={3}
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
        />
      </div>

      {/* Exercise Fields */}
      <div>
        <div className="flex justify-between items-center mb-4">
          <h3 className="text-lg font-medium text-gray-900">Exercícios</h3>
          <button
            type="button"
            onClick={addExercise}
            className="bg-indigo-600 text-white px-4 py-2 rounded-md hover:bg-indigo-700"
          >
            Adicionar Exercício
          </button>
        </div>

        {fields.map((field, index) => (
          <div key={field.id} className="border rounded-lg p-4 mb-4">
            <div className="flex justify-between items-center mb-2">
              <h4 className="font-medium">Exercício {index + 1}</h4>
              <button
                type="button"
                onClick={() => remove(index)}
                className="text-red-600 hover:text-red-800"
              >
                Remover
              </button>
            </div>

            <div className="grid grid-cols-2 gap-4">
              <div>
                <label className="block text-sm font-medium text-gray-700">
                  Nome
                </label>
                <input
                  {...register(`exercises.${index}.name`)}
                  className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700">
                  Séries
                </label>
                <input
                  {...register(`exercises.${index}.sets`, { valueAsNumber: true })}
                  type="number"
                  className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700">
                  Repetições
                </label>
                <input
                  {...register(`exercises.${index}.repetitions`, { valueAsNumber: true })}
                  type="number"
                  className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700">
                  Peso (kg)
                </label>
                <input
                  {...register(`exercises.${index}.weight`, { valueAsNumber: true })}
                  type="number"
                  step="0.1"
                  className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
              </div>
            </div>
          </div>
        ))}
      </div>

      <button
        type="submit"
        disabled={isLoading}
        className="w-full bg-indigo-600 text-white py-2 px-4 rounded-md hover:bg-indigo-700 disabled:opacity-50"
      >
        {isLoading ? 'Salvando...' : 'Salvar Treino'}
      </button>
    </form>
  );
}
```
