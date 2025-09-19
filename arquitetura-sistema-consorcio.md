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
- **.NET 8**: Framework principal para desenvolvimento backend
- **Next.js 14+**: Framework React para frontend
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
- **Vercel/Netlify**: Deploy do frontend (opcional)

## 3. Arquitetura do Sistema

### 3.1 Frontend - Microfrontend com Next.js

#### 3.1.1 Arquitetura de Microfrontend
- **Shell App**: Aplicação principal que orquestra os microfrontends
- **Module Federation**: Compartilhamento de módulos entre aplicações
- **Next.js 14+**: Framework base para cada microfrontend
- **TypeScript**: Tipagem estática compartilhada
- **Design System**: Componentes UI compartilhados
- **Shared State**: Estado global compartilhado entre microfrontends

#### 3.1.2 Estrutura Simplificada para 2 Desenvolvedores
- **Shell App**: Aplicação principal (host)
- **Gestão MF**: Microfrontend unificado para Consorciados + Grupos
- **Financeiro MF**: Microfrontend para módulo financeiro + Relatórios
- **Operações MF**: Microfrontend para Sorteios + Documentos + Notificações

**Justificativa**: Com apenas 2 desenvolvedores, reduzimos de 7 para 3 microfrontends para:
- **Menor complexidade** de gerenciamento
- **Desenvolvimento mais ágil** com menos context switching
- **Deploy simplificado** com menos aplicações
- **Manutenção mais fácil** com menos pontos de falha

#### 3.1.3 Funcionalidades por Microfrontend (Simplificado)

##### Shell App (Host)
- **Navegação principal**: Menu e roteamento
- **Layout base**: Header, sidebar, footer
- **Autenticação**: Login e controle de acesso
- **Loading states**: Estados de carregamento
- **Error boundaries**: Tratamento de erros
- **Dashboard geral**: Visão consolidada do sistema

##### Gestão MF (Consorciados + Grupos)
- **CRUD de consorciados**: Cadastro, edição, exclusão
- **Listagem e filtros**: Busca e paginação de consorciados
- **Gestão de grupos**: Criação e configuração de grupos
- **Cronogramas**: Definição de pagamentos
- **Vagas**: Controle de participantes
- **Histórico**: Participações e movimentações
- **Validação de dados**: Formulários com validação

##### Financeiro MF (Financeiro + Relatórios)
- **Mensalidades**: Controle de pagamentos
- **Inadimplência**: Gestão de atrasos
- **Dashboards financeiros**: Gráficos e métricas
- **Relatórios**: Exportação PDF, Excel, CSV
- **Integração bancária**: APIs de pagamento
- **Filtros avançados**: Consultas complexas
- **Agendamento**: Relatórios automáticos

##### Operações MF (Sorteios + Documentos + Notificações)
- **Execução de sorteios**: Interface para sorteios
- **Gestão de lances**: Processamento de lances
- **Contemplações**: Processo de contemplação
- **Upload/Download**: Gestão de arquivos
- **Centro de notificações**: Lista de notificações
- **Templates**: Criação de templates
- **Auditoria**: Logs de sorteios e documentos

#### 3.1.4 Arquitetura de Microfrontend

##### Shell App (Host)
```typescript
// Estrutura do Shell App
apps/shell/
├── src/
│   ├── app/                    // App Router
│   │   ├── (auth)/            // Autenticação
│   │   ├── dashboard/         // Dashboard principal
│   │   └── layout.tsx         // Layout base
│   ├── components/            // Componentes compartilhados
│   │   ├── layout/           // Header, Sidebar, Footer
│   │   ├── navigation/       // Menu de navegação
│   │   └── error-boundary/   // Error boundaries
│   ├── lib/                  // Configurações compartilhadas
│   │   ├── module-federation/ // Config MF
│   │   ├── auth/             // Autenticação
│   │   └── shared-state/     // Estado global
│   └── types/                // Tipos compartilhados
├── next.config.js            // Configuração Next.js + MF
└── package.json
```

##### Microfrontend Individual (Simplificado)
```typescript
// Estrutura de um microfrontend (ex: gestao-mf)
apps/gestao-mf/
├── src/
│   ├── app/                    // App Router
│   │   ├── consorciados/      // Módulo Consorciados
│   │   │   ├── page.tsx       // Lista de consorciados
│   │   │   ├── [id]/          // Detalhes/Edição
│   │   │   └── novo/          // Cadastro
│   │   ├── grupos/            // Módulo Grupos
│   │   │   ├── page.tsx       // Lista de grupos
│   │   │   ├── [id]/          // Detalhes/Edição
│   │   │   └── novo/          // Criação
│   │   └── loading.tsx        // Loading states
│   ├── components/            // Componentes específicos
│   │   ├── consorciados/      // Componentes de consorciados
│   │   ├── grupos/            // Componentes de grupos
│   │   ├── shared/            // Componentes compartilhados internos
│   │   └── forms/             // Formulários
│   ├── hooks/                 // Hooks específicos
│   ├── lib/                   // Lógica de negócio
│   └── types/                 // Tipos específicos
├── next.config.js            // Configuração MF
└── package.json
```

##### Design System Compartilhado
```typescript
// Estrutura do Design System
packages/design-system/
├── src/
│   ├── components/            // Componentes base
│   │   ├── ui/               // Shadcn/ui
│   │   ├── forms/            // Formulários
│   │   ├── charts/           // Gráficos
│   │   └── layout/           // Layout components
│   ├── hooks/                // Hooks compartilhados
│   ├── utils/                // Utilitários
│   ├── styles/               // Estilos globais
│   └── types/                // Tipos compartilhados
├── package.json
└── tsconfig.json
```

#### 3.1.5 Module Federation Configuration

##### Shell App (Host) - next.config.js
```javascript
const { NextFederationPlugin } = require('@module-federation/nextjs-mf');

module.exports = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'shell',
        remotes: {
          gestao: 'gestao@http://localhost:3001/_next/static/chunks/remoteEntry.js',
          financeiro: 'financeiro@http://localhost:3002/_next/static/chunks/remoteEntry.js',
          operacoes: 'operacoes@http://localhost:3003/_next/static/chunks/remoteEntry.js',
        },
        shared: {
          react: { singleton: true },
          'react-dom': { singleton: true },
          'next': { singleton: true },
          '@tanstack/react-query': { singleton: true },
          'zustand': { singleton: true },
        },
      })
    );
    return config;
  },
};
```

##### Microfrontend (Remote) - next.config.js
```javascript
const { NextFederationPlugin } = require('@module-federation/nextjs-mf');

module.exports = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'gestao',
        filename: 'static/chunks/remoteEntry.js',
        exposes: {
          './ConsorciadosPage': './src/app/consorciados/page.tsx',
          './ConsorciadosForm': './src/components/consorciados/ConsorciadosForm.tsx',
          './ConsorciadosTable': './src/components/consorciados/ConsorciadosTable.tsx',
          './GruposPage': './src/app/grupos/page.tsx',
          './GruposForm': './src/components/grupos/GruposForm.tsx',
          './GruposTable': './src/components/grupos/GruposTable.tsx',
        },
        shared: {
          react: { singleton: true },
          'react-dom': { singleton: true },
          'next': { singleton: true },
          '@tanstack/react-query': { singleton: true },
          'zustand': { singleton: true },
        },
      })
    );
    return config;
  },
};
```

#### 3.1.6 Integração com Backend

##### Cliente API Compartilhado
```typescript
// packages/shared-api/src/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '../server/routers';

export const trpc = createTRPCReact<AppRouter>();

// Configuração compartilhada
export const trpcClient = trpc.createClient({
  url: process.env.NEXT_PUBLIC_API_URL + '/trpc',
  headers() {
    return {
      authorization: `Bearer ${getToken()}`,
    };
  },
});
```

##### Hook Compartilhado para Consorciados
```typescript
// packages/shared-api/src/hooks/useConsorciados.ts
export const useConsorciados = () => {
  return trpc.consorciados.list.useQuery({
    page: 1,
    limit: 10,
    filters: {}
  });
};

export const useCreateConsorciado = () => {
  const utils = trpc.useUtils();
  
  return trpc.consorciados.create.useMutation({
    onSuccess: () => {
      utils.consorciados.list.invalidate();
    }
  });
};
```

##### Estado Global Compartilhado
```typescript
// packages/shared-state/src/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  login: (user: User, token: string) => void;
  logout: () => void;
  isAuthenticated: boolean;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      login: (user, token) => set({ user, token, isAuthenticated: true }),
      logout: () => set({ user: null, token: null, isAuthenticated: false }),
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

##### Componente de Carregamento Dinâmico
```typescript
// apps/shell/src/components/DynamicMicrofrontend.tsx
import dynamic from 'next/dynamic';
import { Suspense } from 'react';

interface DynamicMicrofrontendProps {
  microfrontend: string;
  component: string;
  fallback?: React.ReactNode;
}

const DynamicMicrofrontend = ({ 
  microfrontend, 
  component, 
  fallback = <div>Carregando...</div> 
}: DynamicMicrofrontendProps) => {
  const DynamicComponent = dynamic(
    () => import(`${microfrontend}/${component}`),
    {
      ssr: false,
      loading: () => fallback,
    }
  );

  return (
    <Suspense fallback={fallback}>
      <DynamicComponent />
    </Suspense>
  );
};

export default DynamicMicrofrontend;
```

##### Uso no Shell App (Simplificado)
```typescript
// apps/shell/src/app/consorciados/page.tsx
import DynamicMicrofrontend from '@/components/DynamicMicrofrontend';

export default function ConsorciadosPage() {
  return (
    <DynamicMicrofrontend
      microfrontend="gestao"
      component="./ConsorciadosPage"
    />
  );
}

// apps/shell/src/app/grupos/page.tsx
export default function GruposPage() {
  return (
    <DynamicMicrofrontend
      microfrontend="gestao"
      component="./GruposPage"
    />
  );
}

// apps/shell/src/app/financeiro/page.tsx
export default function FinanceiroPage() {
  return (
    <DynamicMicrofrontend
      microfrontend="financeiro"
      component="./FinanceiroPage"
    />
  );
}
```

### 3.2 Backend - Microserviços

#### 3.2.1 API Gateway
- **Tecnologia**: .NET 8 + Ocelot + gRPC Client
- **Responsabilidades**:
  - Roteamento de requisições HTTP/REST
  - Proxy para serviços gRPC internos
  - Autenticação e autorização
  - Rate limiting
  - Logging centralizado
  - CORS e segurança
  - Load balancing para gRPC

#### 3.2.2 Serviço de Autenticação (Identity Service)
- **Tecnologia**: .NET 8 + IdentityServer4/OpenIddict
- **Responsabilidades**:
  - Gerenciamento de usuários
  - Autenticação JWT
  - Autorização baseada em roles
  - Refresh tokens
  - Integração com Active Directory (opcional)

#### 3.2.3 Serviço de Consorciados
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - CRUD de consorciados via gRPC
  - Validação de dados pessoais
  - Histórico de participações
  - Integração com serviços de validação
  - Cache com Redis
  - Locks distribuídos para operações críticas

#### 3.2.4 Serviço de Grupos de Consórcio
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Gestão de grupos ativos via gRPC
  - Configuração de regras
  - Cronogramas de pagamento
  - Controle de vagas
  - Cache com Redis
  - Locks para operações de grupo

#### 3.2.5 Serviço Financeiro
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Gestão de mensalidades via gRPC
  - Controle de inadimplência
  - Relatórios financeiros
  - Integração bancária
  - Cálculos de juros e multas
  - Locks distribuídos para transações críticas

#### 3.2.6 Serviço de Sorteios e Lances
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Controle de sorteios via gRPC
  - Gestão de lances com locks distribuídos
  - Processo de contemplação
  - Auditoria de sorteios
  - Prevenção de execução simultânea de sorteios

#### 3.2.7 Serviço de Documentos
- **Tecnologia**: .NET 8 + MinIO SDK + gRPC Server
- **Responsabilidades**:
  - Upload/download de documentos via gRPC
  - Geração de contratos
  - Versionamento de documentos
  - Compliance com LGPD
  - Locks para operações de arquivo

#### 3.2.8 Serviço de Relatórios
- **Tecnologia**: .NET 8 + Dapper + Redis + gRPC Server
- **Responsabilidades**:
  - Geração de relatórios via gRPC
  - Cache de consultas complexas
  - Exportação (PDF, Excel)
  - Dashboards em tempo real
  - Locks distribuídos para relatórios pesados

#### 3.2.9 Serviço de Notificações
- **Tecnologia**: .NET 8 + RabbitMQ + gRPC Server
- **Responsabilidades**:
  - Envio de emails via gRPC
  - Notificações push
  - SMS
  - Webhooks
  - Processamento assíncrono de notificações

### 3.3 Serviços de Apoio

#### 3.3.1 Serviço de Auditoria
- **Tecnologia**: .NET 8 + Entity Framework Core + gRPC Server
- **Responsabilidades**:
  - Log de todas as operações via gRPC
  - Rastreabilidade de mudanças
  - Compliance com CVM
  - Relatórios de auditoria
  - Event sourcing para auditoria

#### 3.3.2 Serviço de Configuração
- **Tecnologia**: .NET 8 + Redis + gRPC Server
- **Responsabilidades**:
  - Configurações dinâmicas via gRPC
  - Feature flags
  - Parâmetros do sistema
  - Cache de configurações
  - Locks para atualizações de configuração

## 4. Comunicação Frontend-Backend

### 4.1 Padrões de Comunicação
- **HTTP/REST**: APIs públicas para o frontend
- **WebSocket**: Notificações em tempo real
- **Server-Sent Events (SSE)**: Updates de status
- **gRPC-Web**: Comunicação otimizada (opcional)

### 4.2 Autenticação Frontend
```typescript
// Configuração de autenticação Next.js
import { NextAuth } from 'next-auth';
import { JWT } from 'next-auth/jwt';

export const authOptions: NextAuthOptions = {
  providers: [
    {
      id: 'custom',
      name: 'Custom Provider',
      type: 'oauth',
      authorization: {
        url: `${process.env.API_URL}/auth/authorize`,
        params: {
          scope: 'openid profile email',
          response_type: 'code',
        },
      },
      token: `${process.env.API_URL}/auth/token`,
      userinfo: `${process.env.API_URL}/auth/userinfo`,
    },
  ],
  callbacks: {
    async jwt({ token, account }) {
      if (account) {
        token.accessToken = account.access_token;
      }
      return token;
    },
    async session({ session, token }) {
      session.accessToken = token.accessToken;
      return session;
    },
  },
};
```

### 4.3 Cliente API com tRPC
```typescript
// Configuração tRPC
import { createTRPCNext } from '@trpc/next';
import type { AppRouter } from '../server/routers';

export const trpc = createTRPCNext<AppRouter>({
  config() {
    return {
      url: `${process.env.NEXT_PUBLIC_API_URL}/trpc`,
      headers() {
        return {
          authorization: `Bearer ${getToken()}`,
        };
      },
    };
  },
  ssr: false,
});
```

### 4.4 WebSocket para Notificações
```typescript
// Hook para notificações em tempo real
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

export const useNotifications = () => {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const newSocket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      auth: {
        token: getToken(),
      },
    });

    newSocket.on('notification', (data) => {
      setNotifications(prev => [...prev, data]);
    });

    setSocket(newSocket);

    return () => newSocket.close();
  }, []);

  return { socket, notifications };
};
```

## 5. Arquitetura de Dados

### 5.1 Estratégia de Banco de Dados
- **Database per Service**: Cada microserviço possui seu próprio banco
- **Saga Pattern**: Para transações distribuídas
- **Event Sourcing**: Para auditoria e rastreabilidade
- **CQRS**: Separação de comandos e consultas

### 5.2 Estrutura de Dados

#### 5.2.1 Banco de Consorciados
```sql
-- Tabelas principais
- Consorciados
- Enderecos
- Contatos
- Documentos
- HistoricoParticipacoes
```

#### 5.2.2 Banco de Grupos
```sql
-- Tabelas principais
- GruposConsorcio
- RegrasGrupo
- Cronogramas
- Vagas
- ParticipantesGrupo
```

#### 5.2.3 Banco Financeiro
```sql
-- Tabelas principais
- Mensalidades
- Pagamentos
- Inadimplencias
- Taxas
- MovimentacoesFinanceiras
```

#### 5.2.4 Banco de Sorteios
```sql
-- Tabelas principais
- Sorteios
- Lances
- Contemplacoes
- ResultadosSorteio
- AuditoriaSorteios
```

### 5.3 Redis - Cache e Locks Distribuídos

#### 5.3.1 Estrutura de Cache
```json
{
  "cache:consorciado:{id}": "dados_consorciado",
  "cache:grupo:{id}": "dados_grupo",
  "cache:relatorio:{tipo}:{filtros}": "dados_relatorio",
  "session:{token}": "dados_sessao",
  "config:{chave}": "valor_configuracao"
}
```

#### 5.3.2 Locks Distribuídos
```json
{
  "lock:sorteio:{grupo_id}": "executando_sorteio",
  "lock:lance:{consorciado_id}": "processando_lance",
  "lock:pagamento:{id}": "processando_pagamento",
  "lock:contemplacao:{grupo_id}": "processando_contemplacao",
  "lock:relatorio:{tipo}": "gerando_relatorio"
}
```

#### 5.3.3 Implementação de Locks
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

#### 5.3.4 Casos de Uso para Locks
- **Sorteios**: Evitar execução simultânea de sorteios no mesmo grupo
- **Lances**: Processar lances de forma sequencial
- **Pagamentos**: Evitar processamento duplicado de pagamentos
- **Contemplações**: Garantir processamento único de contemplações
- **Relatórios**: Evitar geração simultânea de relatórios pesados

## 6. Comunicação entre Serviços

### 6.1 Padrões de Comunicação
- **gRPC**: Comunicação interna síncrona entre microserviços
- **HTTP/REST**: APIs públicas e integrações externas
- **Assíncrona**: RabbitMQ para processamento em background
- **Event-Driven**: Eventos para notificações e auditoria

### 6.2 gRPC - Comunicação Interna

#### 6.2.1 Serviços com gRPC
- **Consorciado Service**: Operações CRUD e consultas complexas
- **Financeiro Service**: Cálculos financeiros e validações
- **Grupo Service**: Gestão de grupos e regras
- **Sorteio Service**: Processamento de sorteios e lances
- **Auditoria Service**: Logging e rastreabilidade

#### 6.2.2 Definições de Serviços gRPC
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

#### 6.2.3 Vantagens do gRPC
- **Performance**: Até 7x mais rápido que REST
- **Type Safety**: Contratos definidos com Protocol Buffers
- **Streaming**: Suporte a streaming bidirecional
- **Load Balancing**: Suporte nativo a load balancing
- **Compressão**: Compressão automática de dados

### 6.3 Eventos Principais
```csharp
// Exemplos de eventos
- ConsorciadoCriado
- PagamentoRealizado
- SorteioExecutado
- ContemplacaoProcessada
- DocumentoUploadado
- InadimplenciaDetectada
```

### 6.4 Message Queues (RabbitMQ)
- **consorciado.queue**: Eventos de consorciados
- **financeiro.queue**: Eventos financeiros
- **sorteio.queue**: Eventos de sorteios
- **notificacao.queue**: Eventos de notificação
- **auditoria.queue**: Eventos de auditoria

## 7. Deploy do Frontend

### 7.1 Estratégias de Deploy
- **Kubernetes**: Deploy junto com backend
- **Vercel**: Deploy otimizado para Next.js
- **Netlify**: Alternativa para deploy estático
- **Docker**: Containerização para qualquer ambiente

### 7.2 Deploy no Kubernetes

#### 7.2.1 Shell App (Host)
```yaml
# Deployment do Shell App
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consorcio-shell
spec:
  replicas: 3
  selector:
    matchLabels:
      app: consorcio-shell
  template:
    spec:
      containers:
      - name: shell
        image: consorcio/shell:latest
        ports:
        - containerPort: 3000
        env:
        - name: NEXT_PUBLIC_API_URL
          value: "https://api.consorcio.com"
        - name: NEXT_PUBLIC_WS_URL
          value: "wss://ws.consorcio.com"
        - name: NEXTAUTH_SECRET
          valueFrom:
            secretKeyRef:
              name: nextauth-secret
              key: secret
        - name: NEXTAUTH_URL
          value: "https://app.consorcio.com"
        - name: MICROFRONTEND_URLS
          value: '{"consorciados":"https://consorciados.consorcio.com","grupos":"https://grupos.consorcio.com"}'
---
apiVersion: v1
kind: Service
metadata:
  name: consorcio-shell
spec:
  selector:
    app: consorcio-shell
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

#### 7.2.2 Microfrontends Simplificados
```yaml
# Deployment do Microfrontend Gestão
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consorcio-gestao-mf
spec:
  replicas: 2
  selector:
    matchLabels:
      app: consorcio-gestao-mf
  template:
    spec:
      containers:
      - name: gestao-mf
        image: consorcio/gestao-mf:latest
        ports:
        - containerPort: 3001
        env:
        - name: NEXT_PUBLIC_API_URL
          value: "https://api.consorcio.com"
        - name: NODE_ENV
          value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: consorcio-gestao-mf
spec:
  selector:
    app: consorcio-gestao-mf
  ports:
  - port: 3001
    targetPort: 3001
  type: ClusterIP
---
# Deployment do Microfrontend Financeiro
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consorcio-financeiro-mf
spec:
  replicas: 2
  selector:
    matchLabels:
      app: consorcio-financeiro-mf
  template:
    spec:
      containers:
      - name: financeiro-mf
        image: consorcio/financeiro-mf:latest
        ports:
        - containerPort: 3002
        env:
        - name: NEXT_PUBLIC_API_URL
          value: "https://api.consorcio.com"
        - name: NODE_ENV
          value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: consorcio-financeiro-mf
spec:
  selector:
    app: consorcio-financeiro-mf
  ports:
  - port: 3002
    targetPort: 3002
  type: ClusterIP
```

#### 7.2.3 Ingress para Microfrontends
```yaml
# Ingress para Microfrontends
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: consorcio-microfrontends
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.consorcio.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: consorcio-shell
            port:
              number: 3000
  - host: gestao.consorcio.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: consorcio-gestao-mf
            port:
              number: 3001
  - host: financeiro.consorcio.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: consorcio-financeiro-mf
            port:
              number: 3002
  - host: operacoes.consorcio.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: consorcio-operacoes-mf
            port:
              number: 3003
```

### 7.3 Build e Otimização

#### 7.3.1 Dockerfile para Shell App
```dockerfile
# Dockerfile para Shell App
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./
COPY packages/shared-api/package.json ./packages/shared-api/
COPY packages/shared-state/package.json ./packages/shared-state/
COPY packages/design-system/package.json ./packages/design-system/

RUN npm ci --only=production

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build:shell

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/apps/shell/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/apps/shell/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/shell/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### 7.3.2 Dockerfile para Microfrontend
```dockerfile
# Dockerfile para Microfrontend
FROM node:18-alpine AS base

FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./
COPY packages/shared-api/package.json ./packages/shared-api/
COPY packages/design-system/package.json ./packages/design-system/

RUN npm ci --only=production

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build:consorciados-mf

FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/apps/consorciados-mf/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/apps/consorciados-mf/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/consorciados-mf/.next/static ./.next/static

USER nextjs

EXPOSE 3001
ENV PORT 3001
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### 7.3.3 Monorepo com Lerna/Nx
```json
// package.json (root)
{
  "name": "consorcio-monorepo",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "build:shell": "cd apps/shell && npm run build",
    "build:gestao-mf": "cd apps/gestao-mf && npm run build",
    "build:financeiro-mf": "cd apps/financeiro-mf && npm run build",
    "build:operacoes-mf": "cd apps/operacoes-mf && npm run build",
    "build:all": "npm run build:shell && npm run build:gestao-mf && npm run build:financeiro-mf && npm run build:operacoes-mf",
    "dev:shell": "cd apps/shell && npm run dev",
    "dev:gestao-mf": "cd apps/gestao-mf && npm run dev",
    "dev:financeiro-mf": "cd apps/financeiro-mf && npm run dev",
    "dev:operacoes-mf": "cd apps/operacoes-mf && npm run dev",
    "dev:all": "concurrently \"npm run dev:shell\" \"npm run dev:gestao-mf\" \"npm run dev:financeiro-mf\" \"npm run dev:operacoes-mf\"",
    "dev:minimal": "concurrently \"npm run dev:shell\" \"npm run dev:gestao-mf\""
  },
  "devDependencies": {
    "concurrently": "^8.2.0",
    "lerna": "^7.0.0"
  }
}
```

## 8. Segurança

### 8.1 Autenticação e Autorização
- **JWT Tokens**: Autenticação stateless
- **RBAC**: Role-Based Access Control
- **OAuth 2.0**: Para integrações externas
- **Multi-Factor Authentication**: Opcional

### 8.2 Criptografia
- **TLS 1.3**: Comunicação entre serviços
- **AES-256**: Criptografia de dados sensíveis
- **RSA-2048**: Chaves assimétricas
- **Hashing**: Senhas com bcrypt

### 8.3 Compliance
- **LGPD**: Proteção de dados pessoais
- **CVM**: Regulamentações específicas
- **Auditoria**: Logs completos e imutáveis
- **Backup**: Estratégia de backup e recuperação

## 9. Deploy no Kubernetes

### 9.1 Estrutura de Namespaces
```yaml
# Namespaces por ambiente
- consorcio-prod
- consorcio-staging
- consorcio-dev
- consorcio-monitoring
```

### 9.2 ConfigMaps e Secrets
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

### 9.3 Deployments e Services
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

### 9.4 Ingress e Load Balancing
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

## 10. Monitoramento e Observabilidade

### 10.1 Logging
- **Serilog**: Logging estruturado
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Correlation IDs**: Rastreamento de requisições
- **gRPC Logging**: Logs específicos para chamadas gRPC
- **Redis Logging**: Logs de operações de cache e locks

### 10.2 Métricas
- **Prometheus**: Coleta de métricas
- **Grafana**: Dashboards e alertas
- **Health Checks**: Verificação de saúde dos serviços
- **gRPC Metrics**: Métricas de performance gRPC
- **Redis Metrics**: Métricas de cache e locks distribuídos

### 10.3 Tracing
- **OpenTelemetry**: Distributed tracing
- **Jaeger**: Visualização de traces
- **Performance Monitoring**: APM
- **gRPC Tracing**: Rastreamento de chamadas gRPC
- **Redis Tracing**: Rastreamento de operações de cache

## 11. CI/CD Pipeline

### 11.1 Estrutura do Pipeline
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

### 11.2 Estratégia de Deploy
- **Blue-Green**: Para produção
- **Rolling Update**: Para staging
- **Canary**: Para testes de carga

## 12. Backup e Disaster Recovery

### 12.1 Estratégia de Backup
- **PostgreSQL**: Backup diário com WAL archiving
- **MinIO**: Replicação automática
- **Redis**: RDB + AOF persistence
- **RabbitMQ**: Backup de configurações

### 12.2 Disaster Recovery
- **RTO**: 4 horas
- **RPO**: 1 hora
- **Multi-region**: Backup em região diferente
- **Automated Recovery**: Scripts de recuperação

## 13. Escalabilidade

### 13.1 Horizontal Scaling
- **HPA**: Horizontal Pod Autoscaler
- **VPA**: Vertical Pod Autoscaler
- **Cluster Autoscaler**: Escalabilidade do cluster

### 13.2 Performance
- **Redis**: Cache distribuído
- **Connection Pooling**: Pool de conexões
- **Async/Await**: Programação assíncrona
- **CDN**: Para arquivos estáticos

## 14. Considerações de Compliance

### 14.1 LGPD
- **Anonimização**: Dados pessoais anonimizados
- **Consentimento**: Gestão de consentimentos
- **Direito ao Esquecimento**: Exclusão de dados
- **Portabilidade**: Exportação de dados

### 14.2 CVM
- **Auditoria**: Logs imutáveis
- **Relatórios**: Relatórios obrigatórios
- **Transparência**: Acesso a informações
- **Compliance**: Verificação contínua

## 15. Roadmap de Implementação

### 15.1 Fase 1 - MVP (3 meses)
- Serviços core (Consorciados, Grupos, Financeiro) com gRPC
- API Gateway com proxy gRPC
- Autenticação básica
- Redis para cache e locks básicos
- Deploy em K8s

### 15.2 Fase 2 - Funcionalidades Avançadas (2 meses)
- Serviço de Sorteios com locks distribuídos
- Relatórios com cache Redis
- Notificações
- Documentos
- Implementação completa de gRPC

### 15.3 Fase 3 - Otimização (1 mês)
- Monitoramento gRPC e Redis
- Performance e escalabilidade
- Segurança avançada
- Compliance completo
- Locks distribuídos avançados

### 15.4 Fase 4 - Microfrontend Simplificado (2 meses)
- Desenvolvimento do Shell App (aplicação principal)
- Criação do Design System compartilhado
- Desenvolvimento do Gestão MF (Consorciados + Grupos)
- Desenvolvimento do Financeiro MF (Financeiro + Relatórios)
- Configuração do Module Federation
- Integração com APIs backend
- Deploy dos microfrontends

### 15.5 Fase 5 - Operações e Otimização (1 mês)
- Desenvolvimento do Operações MF (Sorteios + Documentos + Notificações)
- Otimização de performance dos microfrontends
- Implementação de lazy loading
- Monitoramento específico para microfrontends
- CI/CD simplificado

### 15.6 Estratégia de Desenvolvimento para 2 Desenvolvedores

#### Desenvolvedor 1 - Backend + Shell App
- **Responsabilidades**:
  - Desenvolvimento completo do backend (.NET + gRPC)
  - Configuração do Shell App
  - Design System compartilhado
  - Infraestrutura e deploy
  - Integração entre frontend e backend

#### Desenvolvedor 2 - Frontend + Microfrontends
- **Responsabilidades**:
  - Desenvolvimento dos 3 microfrontends
  - Componentes de UI específicos
  - Integração com APIs
  - Testes de integração frontend
  - Otimização de performance

#### Cronograma Semanal
- **Segunda/Quarta/Sexta**: Desenvolvimento individual
- **Terça/Quinta**: Pair programming para integração
- **Sexta**: Review e deploy de features

## 16. Conclusão

Esta arquitetura fornece uma base sólida para o desenvolvimento de um sistema robusto e escalável para administradoras de consórcio, atendendo aos requisitos de compliance, segurança e performance necessários para o domínio financeiro.

A escolha das tecnologias (.NET, Next.js, PostgreSQL, Redis, RabbitMQ, MinIO, gRPC, Module Federation) garante:
- **Performance**: Stack otimizada com gRPC para comunicação interna ultra-rápida
- **Escalabilidade**: Arquitetura de microserviços + microfrontend simplificado com locks distribuídos
- **Modularidade**: 3 microfrontends otimizados para equipe de 2 desenvolvedores
- **Reutilização**: Design System e componentes compartilhados
- **Desenvolvimento Ágil**: Divisão clara de responsabilidades entre os 2 desenvolvedores
- **Segurança**: Compliance com regulamentações e locks para operações críticas
- **Manutenibilidade**: Código limpo com contratos gRPC e Module Federation
- **Observabilidade**: Monitoramento completo incluindo gRPC, Redis e microfrontends
- **Concorrência**: Locks distribuídos para evitar condições de corrida
- **Consistência**: Garantia de operações atômicas com Redis locks
- **Flexibilidade**: Deploy independente de cada microfrontend
- **Produtividade**: Arquitetura simplificada para máxima eficiência da equipe pequena