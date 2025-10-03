# Praticas-extensionistas IV

# Sistema de Gerenciamento de Aplicações - Documentação

## 📋 Sobre o Projeto

Sistema completo de gerenciamento de aplicações com arquitetura moderna baseada em microserviços, containerização e infraestrutura cloud escalável.

### Tecnologias Principais

- **Backend**: Node.js, Python (FastAPI)
- **Frontend**: React.js
- **Banco de Dados**: PostgreSQL, Redis
- **Infraestrutura**: AWS (EKS, RDS, S3, CloudFront)
- **DevOps**: Docker, Kubernetes, Terraform, GitHub Actions
- **Monitoramento**: Prometheus, Grafana, CloudWatch

## 🏗️ Arquitetura

### Arquitetura de Microserviços

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                              │
│                  React.js Application                        │
│                 (CloudFront + S3)                            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                    API Gateway
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌─────▼──────┐ ┌───────▼────────┐
│  Auth Service  │ │   API      │ │  Notification  │
│   (Node.js)    │ │  Service   │ │    Service     │
│                │ │ (Node.js)  │ │   (Python)     │
└───────┬────────┘ └─────┬──────┘ └───────┬────────┘
        │                │                 │
        └────────────────┼─────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                 │
┌───────▼─────────┐            ┌─────────▼──────┐
│   PostgreSQL    │            │     Redis      │
│   (RDS Multi-AZ)│            │  (ElastiCache) │
└─────────────────┘            └────────────────┘
```

### Estrutura de Pacotes (UML)

```
┌──────────────────────────────────────────────────────────┐
│                    Application Layer                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │ Controllers│  │  Services  │  │   Models   │         │
│  └────────────┘  └────────────┘  └────────────┘         │
└──────────────────────────────────────────────────────────┘
                          │
┌──────────────────────────────────────────────────────────┐
│                    Business Layer                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │   Domain   │  │  Use Cases │  │ Validators │         │
│  └────────────┘  └────────────┘  └────────────┘         │
└──────────────────────────────────────────────────────────┘
                          │
┌──────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │ Repository │  │   Cache    │  │  External  │         │
│  │            │  │            │  │    APIs    │         │
│  └────────────┘  └────────────┘  └────────────┘         │
└──────────────────────────────────────────────────────────┘
```

## 🚀 Arquitetura de Implantação

### Ambiente AWS - Multi-AZ

```
┌─────────────────────────────────────────────────────────────┐
│                         AWS Cloud                            │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               Route 53 (DNS)                        │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        │                                     │
│  ┌─────────────────────▼───────────────────────────────┐    │
│  │           CloudFront (CDN)                          │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        │                                     │
│  ┌─────────────────────▼───────────────────────────────┐    │
│  │     Application Load Balancer (ALB)                 │    │
│  └───────┬──────────────────────────────────┬─────────┘    │
│          │                                   │               │
│  ┌───────▼──────────┐              ┌────────▼──────────┐   │
│  │   AZ us-east-1a  │              │   AZ us-east-1b   │   │
│  │                  │              │                   │   │
│  │  ┌────────────┐  │              │  ┌────────────┐  │   │
│  │  │ EKS Nodes  │  │              │  │ EKS Nodes  │  │   │
│  │  │ (Pods)     │  │              │  │ (Pods)     │  │   │
│  │  └─────┬──────┘  │              │  └─────┬──────┘  │   │
│  │        │         │              │        │         │   │
│  │  ┌─────▼──────┐  │              │  ┌─────▼──────┐  │   │
│  │  │ RDS Primary│  │              │  │RDS Standby │  │   │
│  │  │ PostgreSQL │  │              │  │ PostgreSQL │  │   │
│  │  └────────────┘  │              │  └────────────┘  │   │
│  │                  │              │                   │   │
│  │  ┌────────────┐  │              │  ┌────────────┐  │   │
│  │  │ElastiCache │  │              │  │ElastiCache │  │   │
│  │  │   Redis    │  │              │  │   Redis    │  │   │
│  │  └────────────┘  │              │  └────────────┘  │   │
│  └──────────────────┘              └───────────────────┘   │
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ S3 Buckets  │  │  CloudWatch  │  │   Lambda     │      │
│  │  (Storage)  │  │ (Monitoring) │  │ (Functions)  │      │
│  └─────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 🔄 Pipeline DevOps

### CI/CD Flow

```
┌──────────────┐
│   GitHub     │
│  Repository  │
└──────┬───────┘
       │
       │ git push
       ▼
┌──────────────┐
│GitHub Actions│
│   Trigger    │
└──────┬───────┘
       │
       ├─────────────────┐
       │                 │
       ▼                 ▼
┌──────────────┐  ┌─────────────┐
│  Build       │  │   Tests     │
│  - npm ci    │  │  - Unit     │
│  - Docker    │  │  - E2E      │
└──────┬───────┘  └─────┬───────┘
       │                │
       │                │
       ▼                ▼
┌──────────────────────────┐
│    Security Scan         │
│  - SonarQube            │
│  - Trivy (Container)    │
│  - OWASP Dependency     │
└───────────┬──────────────┘
            │
            ▼
    ┌───────────────┐
    │ Push to ECR   │
    │ (Container    │
    │  Registry)    │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │Deploy to EKS  │
    │  - Dev/Stg    │
    │  - Approval   │
    │  - Production │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │  Monitoring   │
    │  - Prometheus │
    │  - Grafana    │
    │  - CloudWatch │
    └───────────────┘
```

### Workflow de Deploy

1. **Desenvolvimento** → Push no branch `develop`
2. **Build & Test** → GitHub Actions executa testes automatizados
3. **Security Scan** → Análise de vulnerabilidades
4. **Deploy Dev** → Automático em ambiente de desenvolvimento
5. **Deploy Staging** → Após aprovação, deploy em staging
6. **Deploy Production** → Após validação, deploy em produção com blue-green

## 📁 Estrutura do Repositório

```
project-root/
├── .github/
│   └── workflows/
│       ├── ci.yml              # Pipeline de CI
│       ├── cd.yml              # Pipeline de CD
│       └── security.yml        # Security checks
│
├── services/
│   ├── auth-service/
│   │   ├── src/
│   │   ├── tests/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── api-service/
│   │   ├── src/
│   │   ├── tests/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   └── notification-service/
│       ├── src/
│       ├── tests/
│       ├── Dockerfile
│       └── requirements.txt
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   └── utils/
│   ├── Dockerfile
│   └── package.json
│
├── infrastructure/
│   ├── terraform/
│   │   ├── modules/
│   │   │   ├── vpc/
│   │   │   ├── eks/
│   │   │   ├── rds/
│   │   │   └── s3/
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   └── main.tf
│   │
│   └── kubernetes/
│       ├── base/
│       │   ├── deployments/
│       │   ├── services/
│       │   └── configmaps/
│       └── overlays/
│           ├── dev/
│           ├── staging/
│           └── production/
│
├── docs/
│   ├── architecture/
│   │   ├── diagrams/
│   │   └── decisions/
│   ├── api/
│   │   └── openapi.yaml
│   └── guides/
│       ├── development.md
│       └── deployment.md
│
├── scripts/
│   ├── deploy.sh
│   ├── setup-dev.sh
│   └── db-migrate.sh
│
├── docker-compose.yml         # Desenvolvimento local
├── README.md
├── .gitignore
└── LICENSE
```

## 🛠️ Setup do Ambiente de Desenvolvimento

### Pré-requisitos

- Node.js 18+
- Docker & Docker Compose
- kubectl
- AWS CLI
- Terraform

### Instalação Local

```bash
# 1. Clonar o repositório
git clone https://github.com/seu-usuario/seu-projeto.git
cd seu-projeto

# 2. Instalar dependências
npm install

# 3. Configurar variáveis de ambiente
cp .env.example .env
# Editar .env com suas configurações

# 4. Subir ambiente local com Docker Compose
docker-compose up -d

# 5. Rodar migrações do banco
npm run db:migrate

# 6. Iniciar aplicação
npm run dev
```

### Acessos Locais

- **Frontend**: http://localhost:3000
- **API Gateway**: http://localhost:8080
- **Auth Service**: http://localhost:3001
- **API Service**: http://localhost:3002
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379

## 🧪 Testes

```bash
# Testes unitários
npm run test

# Testes E2E
npm run test:e2e

# Coverage
npm run test:coverage

# Lint
npm run lint
```

## 📦 Deploy

### Deploy Manual

```bash
# Build das imagens
docker build -t auth-service:latest ./services/auth-service
docker build -t api-service:latest ./services/api-service

# Push para ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/auth-service:latest

# Deploy no Kubernetes
kubectl apply -f infrastructure/kubernetes/production/
```

### Deploy Automático (CI/CD)

O deploy automático é realizado via GitHub Actions:

- **Branch develop** → Deploy automático em DEV
- **Branch staging** → Deploy automático em STAGING
- **Tag vX.X.X** → Deploy em PRODUCTION (com aprovação manual)

## 📊 Monitoramento

### Dashboards

- **Grafana**: https://grafana.seu-dominio.com
- **Prometheus**: https://prometheus.seu-dominio.com
- **CloudWatch**: AWS Console

### Métricas Principais

- Taxa de requisições (RPS)
- Latência média (p50, p95, p99)
- Taxa de erro (4xx, 5xx)
- Uso de CPU e memória
- Conexões de banco de dados
- Cache hit rate

### Alertas Configurados

- CPU > 80% por 5 minutos
- Memória > 85% por 5 minutos
- Taxa de erro > 5% por 2 minutos
- Latência p95 > 1000ms por 5 minutos
- Disco > 90%

## 🔒 Segurança

### Práticas Implementadas

- ✅ Autenticação JWT
- ✅ HTTPS obrigatório
- ✅ Rate limiting
- ✅ CORS configurado
- ✅ Secrets no AWS Secrets Manager
- ✅ Scan de vulnerabilidades (Trivy)
- ✅ Dependências atualizadas
- ✅ WAF na CloudFront
- ✅ Backup automático (RDS)
- ✅ Logs centralizados

### Compliance

- ISO 27001
- SOC 2
- LGPD
- GDPR Ready

## 👥 Contribuindo

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

### Padrões de Commit

```
feat: nova funcionalidade
fix: correção de bug
docs: documentação
style: formatação
refactor: refatoração
test: testes
chore: tarefas gerais
```

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 📧 Contato

github.com/cauaessing-png

## 🗺️ Roadmap

- [x] Setup inicial do projeto
- [x] Implementação de autenticação
- [x] API REST completa
- [x] Deploy na AWS
- [x] Pipeline CI/CD
- [ ] Implementação de GraphQL
- [ ] Aplicativo mobile
- [ ] Integração com IA
- [ ] Multi-tenancy

---

**Versão**: 1.0.0  
**Última atualização**: Outubro 2025  
**Mantido por**: caua
