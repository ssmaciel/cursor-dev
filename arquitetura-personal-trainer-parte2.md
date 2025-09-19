# Arquitetura Personal Trainer - Parte 2

## 8. Mobile (Flutter)

### 8.1 Arquitetura do App Mobile

#### 8.1.1 Estrutura de Projeto

```
personal_trainer_mobile/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── app.dart
│   │   └── routes/
│   │       ├── app_router.dart
│   │       └── route_names.dart
│   ├── core/
│   │   ├── constants/
│   │   ├── errors/
│   │   ├── network/
│   │   ├── utils/
│   │   └── theme/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   ├── training/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   ├── nutrition/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   └── profile/
│   │       ├── data/
│   │       ├── domain/
│   │       └── presentation/
│   ├── shared/
│   │   ├── widgets/
│   │   ├── providers/
│   │   └── services/
│   └── generated/
├── test/
├── integration_test/
└── pubspec.yaml
```

### 8.2 State Management (Riverpod)

```dart
// features/auth/presentation/providers/auth_provider.dart
final authProvider = StateNotifierProvider<AuthNotifier, AuthState>((ref) {
  return AuthNotifier(ref.read(authRepositoryProvider));
});

class AuthNotifier extends StateNotifier<AuthState> {
  final AuthRepository _authRepository;

  AuthNotifier(this._authRepository) : super(const AuthState.initial());

  Future<void> login(String email, String password) async {
    state = state.copyWith(isLoading: true, error: null);
    
    try {
      final result = await _authRepository.login(email, password);
      state = state.copyWith(
        isLoading: false,
        user: result.user,
        isAuthenticated: true,
      );
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        error: e.toString(),
      );
    }
  }
}
```

### 8.3 Offline Support

```dart
// core/database/database_helper.dart
class DatabaseHelper {
  static final DatabaseHelper _instance = DatabaseHelper._internal();
  factory DatabaseHelper() => _instance;
  DatabaseHelper._internal();

  Future<Database> _initDatabase() async {
    String path = join(await getDatabasesPath(), 'personal_trainer.db');
    
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE trainings (
        id TEXT PRIMARY KEY,
        trainer_id TEXT NOT NULL,
        client_id TEXT NOT NULL,
        name TEXT NOT NULL,
        description TEXT,
        is_synced INTEGER DEFAULT 0,
        created_at INTEGER NOT NULL
      )
    ''');
  }
}
```

---

## 9. Segurança

### 9.1 Autenticação JWT

```csharp
public class JwtService : IJwtService
{
    public string GenerateAccessToken(User user)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim("user_type", user.UserType.ToString()),
            new Claim("subscription_plan", user.SubscriptionPlan.ToString())
        };

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(60),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### 9.2 Criptografia de Dados

```csharp
public class EncryptionService : IEncryptionService
{
    public string Encrypt(string plainText)
    {
        using var aes = Aes.Create();
        aes.Key = _key;
        aes.IV = _iv;
        aes.Mode = CipherMode.CBC;
        aes.Padding = PaddingMode.PKCS7;

        using var encryptor = aes.CreateEncryptor();
        using var msEncrypt = new MemoryStream();
        using var csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write);
        using var swEncrypt = new StreamWriter(csEncrypt);

        swEncrypt.Write(plainText);
        return Convert.ToBase64String(msEncrypt.ToArray());
    }

    public string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password, BCrypt.Net.BCrypt.GenerateSalt(12));
    }
}
```

---

## 10. Multi-tenancy e Personalização

### 10.1 Estratégia de Multi-tenancy

```csharp
public class TenantService : ITenantService
{
    public async Task<Tenant> GetTenantBySubdomainAsync(string subdomain)
    {
        return await _tenantRepository.GetBySubdomainAsync(subdomain);
    }

    public async Task<Tenant> GetTenantByUserIdAsync(Guid userId)
    {
        var user = await _userRepository.GetByIdAsync(userId);
        return await _tenantRepository.GetByIdAsync(user.TenantId);
    }

    public async Task<CustomizationSettings> GetCustomizationAsync(Guid tenantId)
    {
        return await _customizationRepository.GetByTenantIdAsync(tenantId);
    }
}
```

### 10.2 App Personalizado

```csharp
public class AppCustomizationService : IAppCustomizationService
{
    public async Task<AppCustomization> GetAppCustomizationAsync(Guid trainerId)
    {
        var trainer = await _trainerRepository.GetByIdAsync(trainerId);
        return new AppCustomization
        {
            PrimaryColor = trainer.PrimaryColor,
            SecondaryColor = trainer.SecondaryColor,
            LogoUrl = trainer.LogoUrl,
            AppName = trainer.AppName,
            WelcomeMessage = trainer.WelcomeMessage
        };
    }
}
```

---

## 11. Sistema de Pagamentos

### 11.1 Integração Stripe

```csharp
public class PaymentService : IPaymentService
{
    private readonly StripeService _stripeService;

    public async Task<PaymentResult> ProcessSubscriptionAsync(SubscriptionRequest request)
    {
        var customer = await _stripeService.CreateCustomerAsync(new CustomerCreateOptions
        {
            Email = request.Email,
            Name = request.Name
        });

        var subscription = await _stripeService.CreateSubscriptionAsync(new SubscriptionCreateOptions
        {
            Customer = customer.Id,
            Items = new List<SubscriptionItemOptions>
            {
                new SubscriptionItemOptions
                {
                    Price = request.PriceId
                }
            }
        });

        return new PaymentResult
        {
            SubscriptionId = subscription.Id,
            Status = subscription.Status,
            CurrentPeriodEnd = subscription.CurrentPeriodEnd
        };
    }
}
```

### 11.2 Webhooks

```csharp
[ApiController]
[Route("api/webhooks")]
public class WebhookController : ControllerBase
{
    [HttpPost("stripe")]
    public async Task<IActionResult> StripeWebhook()
    {
        var json = await new StreamReader(HttpContext.Request.Body).ReadToEndAsync();
        
        try
        {
            var stripeEvent = EventUtility.ParseEvent(json);
            
            switch (stripeEvent.Type)
            {
                case Events.InvoicePaymentSucceeded:
                    await HandleInvoicePaymentSucceeded(stripeEvent);
                    break;
                case Events.CustomerSubscriptionDeleted:
                    await HandleSubscriptionDeleted(stripeEvent);
                    break;
            }

            return Ok();
        }
        catch (StripeException e)
        {
            return BadRequest(e.Message);
        }
    }
}
```

---

## 12. Monitoramento e Observabilidade

### 12.1 Logging Estruturado

```csharp
public class StructuredLogger
{
    private readonly ILogger _logger;

    public void LogTrainingCreated(Guid trainingId, Guid trainerId, string trainingName)
    {
        _logger.LogInformation("Training created: {TrainingId} by trainer {TrainerId} with name {TrainingName}",
            trainingId, trainerId, trainingName);
    }

    public void LogPaymentProcessed(string subscriptionId, decimal amount, string currency)
    {
        _logger.LogInformation("Payment processed: Subscription {SubscriptionId} for {Amount} {Currency}",
            subscriptionId, amount, currency);
    }
}
```

### 12.2 Health Checks

```csharp
public class DatabaseHealthCheck : IHealthCheck
{
    private readonly TrainingDbContext _context;

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            await _context.Database.ExecuteSqlRawAsync("SELECT 1", cancellationToken);
            return HealthCheckResult.Healthy("Database is accessible");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Database is not accessible", ex);
        }
    }
}
```

---

## 13. Deploy e DevOps

### 13.1 Docker Configuration

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["PersonalTrainer.Training.API/PersonalTrainer.Training.API.csproj", "PersonalTrainer.Training.API/"]
RUN dotnet restore "PersonalTrainer.Training.API/PersonalTrainer.Training.API.csproj"
COPY . .
WORKDIR "/src/PersonalTrainer.Training.API"
RUN dotnet build "PersonalTrainer.Training.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "PersonalTrainer.Training.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "PersonalTrainer.Training.API.dll"]
```

### 13.2 Kubernetes Deployment

```yaml
# training-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: training-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: training-service
  template:
    metadata:
      labels:
        app: training-service
    spec:
      containers:
      - name: training-service
        image: personal-trainer/training-service:latest
        ports:
        - containerPort: 80
        env:
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: training-service
spec:
  selector:
    app: training-service
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

## 14. Testes

### 14.1 Testes Unitários

```csharp
[TestClass]
public class TrainingServiceTests
{
    private Mock<ITrainingRepository> _mockRepository;
    private Mock<IMapper> _mockMapper;
    private TrainingService _service;

    [TestInitialize]
    public void Setup()
    {
        _mockRepository = new Mock<ITrainingRepository>();
        _mockMapper = new Mock<IMapper>();
        _service = new TrainingService(_mockRepository.Object, _mockMapper.Object);
    }

    [TestMethod]
    public async Task CreateTraining_ValidRequest_ReturnsTrainingDto()
    {
        // Arrange
        var request = new CreateTrainingCommand
        {
            Name = "Test Training",
            TrainerId = Guid.NewGuid(),
            ClientId = Guid.NewGuid()
        };

        var training = new Training(request.TrainerId, request.ClientId, request.Name, "");
        var expectedDto = new TrainingDto { Id = training.Id, Name = training.Name };

        _mockRepository.Setup(r => r.AddAsync(It.IsAny<Training>()))
                      .ReturnsAsync(training);
        _mockMapper.Setup(m => m.Map<TrainingDto>(training))
                  .Returns(expectedDto);

        // Act
        var result = await _service.CreateTrainingAsync(request);

        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(request.Name, result.Name);
        _mockRepository.Verify(r => r.AddAsync(It.IsAny<Training>()), Times.Once);
    }
}
```

### 14.2 Testes de Integração

```csharp
[TestClass]
public class TrainingControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public TrainingControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [TestMethod]
    public async Task GetTrainings_ReturnsSuccessStatusCode()
    {
        // Arrange
        var trainerId = Guid.NewGuid().ToString();

        // Act
        var response = await _client.GetAsync($"/api/trainings?trainerId={trainerId}");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.AreEqual("application/json; charset=utf-8", 
                       response.Content.Headers.ContentType?.ToString());
    }
}
```

---

## 15. Checklist de Implementação

### 15.1 Fase 1 - Infraestrutura Base (4 semanas)
- [ ] Configuração do ambiente de desenvolvimento
- [ ] Setup do banco PostgreSQL
- [ ] Configuração do Redis
- [ ] Setup do RabbitMQ
- [ ] Configuração do Docker
- [ ] CI/CD pipeline básico

### 15.2 Fase 2 - Backend Core (6 semanas)
- [ ] User Service com autenticação
- [ ] Training Service básico
- [ ] Nutrition Service básico
- [ ] API Gateway
- [ ] Testes unitários
- [ ] Documentação Swagger

### 15.3 Fase 3 - Frontend Web (4 semanas)
- [ ] Setup Next.js
- [ ] Autenticação frontend
- [ ] Dashboard básico
- [ ] CRUD de treinos
- [ ] CRUD de planos alimentares

### 15.4 Fase 4 - Mobile (6 semanas)
- [ ] Setup Flutter
- [ ] Autenticação mobile
- [ ] Sincronização offline
- [ ] Push notifications
- [ ] App store deployment

### 15.5 Fase 5 - Funcionalidades Avançadas (4 semanas)
- [ ] Sistema de pagamentos
- [ ] Multi-tenancy
- [ ] Analytics
- [ ] Relatórios
- [ ] Notificações avançadas

### 15.6 Fase 6 - Produção (2 semanas)
- [ ] Deploy em produção
- [ ] Monitoramento
- [ ] Backup e recovery
- [ ] Performance tuning
- [ ] Security audit

---

## 16. Estimativas de Esforço

### 16.1 Desenvolvimento Backend
- **User Service**: 3 semanas
- **Training Service**: 4 semanas
- **Nutrition Service**: 4 semanas
- **Communication Service**: 3 semanas
- **Payment Service**: 3 semanas
- **Notification Service**: 2 semanas

**Total Backend**: 19 semanas

### 16.2 Desenvolvimento Frontend
- **Web Application**: 6 semanas
- **Mobile Application**: 8 semanas
- **Shared Components**: 2 semanas

**Total Frontend**: 16 semanas

### 16.3 Infraestrutura e DevOps
- **Setup inicial**: 2 semanas
- **CI/CD**: 2 semanas
- **Monitoramento**: 1 semana
- **Security**: 2 semanas

**Total Infraestrutura**: 7 semanas

### 16.4 Testes e Qualidade
- **Testes unitários**: 4 semanas
- **Testes de integração**: 3 semanas
- **Testes E2E**: 2 semanas
- **Code review e refactoring**: 2 semanas

**Total Testes**: 11 semanas

**Estimativa Total**: 53 semanas (13 meses)

---

## 17. Roadmap de Desenvolvimento

### Q1 2024 - MVP Backend
- Infraestrutura base
- User Service
- Training Service básico
- Autenticação e autorização

### Q2 2024 - Frontend e Mobile
- Web application MVP
- Mobile app básico
- Sincronização offline
- Notificações push

### Q3 2024 - Funcionalidades Completas
- Nutrition Service
- Sistema de pagamentos
- Multi-tenancy
- Analytics básico

### Q4 2024 - Produção e Otimização
- Deploy em produção
- Performance tuning
- Security audit
- Monitoramento avançado

---

## 18. Considerações de Escalabilidade

### 18.1 Horizontal Scaling
- Microserviços independentes
- Load balancing
- Auto-scaling baseado em métricas
- Database sharding por tenant

### 18.2 Performance
- Cache distribuído (Redis)
- CDN para assets estáticos
- Database indexing otimizado
- Connection pooling

### 18.3 Disponibilidade
- Multi-region deployment
- Database replication
- Circuit breakers
- Health checks

---

## 19. Compliance e Regulamentações

### 19.1 LGPD (Brasil)
- Consentimento explícito
- Direito ao esquecimento
- Portabilidade de dados
- Relatórios de violação

### 19.2 GDPR (Europa)
- Privacy by design
- Data protection impact assessment
- Data protection officer
- Cross-border transfers

### 19.3 PCI-DSS (Pagamentos)
- Criptografia de dados
- Network security
- Access control
- Regular security testing

---

Este documento fornece uma base sólida para o desenvolvimento completo do sistema Personal Trainer, cobrindo todos os aspectos técnicos, de negócio e operacionais necessários para o sucesso do projeto.