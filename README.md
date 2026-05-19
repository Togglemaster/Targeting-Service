# Targeting Service 🎯

Serviço de gerenciamento de regras de segmentação (targeting rules) do **ToggleMaster**. Este serviço define regras complexas para quando uma feature flag deve estar ativa.

## 🎯 Descrição do Serviço

O Targeting Service gerencia as regras de segmentação para cada feature flag. Ele:

1. Define regras complexas para ativação de flags (ex: "50% dos usuários", "usuários do país X")
2. Armazena regras em PostgreSQL como JSON
3. Requer autenticação via chaves de API (valida com Auth Service)
4. Expõe endpoints para gerenciar regras por flag
5. Fornece endpoint `/health` para monitoramento

**Função crítica:** Sem regras de targeting, uma flag seria ativada para 100% dos usuários. Este serviço controla o escopo exato da ativação.

## 📦 Stack Técnico

- **Linguagem:** Python 3.9+
- **Framework:** Flask
- **Banco de Dados:** PostgreSQL (com suporte a JSON)
- **Autenticação:** Bearer Token (integração com Auth Service)
- **Dependências principais:** psycopg2, flask, python-dotenv

## 🚀 Como Usar

### Pré-requisitos Locais

- Python 3.9 ou superior
- PostgreSQL 12+ (instalado ou via Docker)
- Auth Service rodando (porta 8001)

### Setup Local

#### 1. Clone e Navegue para o Diretório
```bash
cd Targeting-Service
```

#### 2. Prepare o Banco de Dados

Crie um banco de dados PostgreSQL:
```bash
createdb targeting_db
```

Execute o script de inicialização:
```bash
psql -U seu_usuario -d targeting_db -f db/init.sql
```

Este script cria a tabela `targeting_rules` com a seguinte estrutura:
```sql
CREATE TABLE targeting_rules (
    id SERIAL PRIMARY KEY,
    flag_name VARCHAR(255) NOT NULL,
    rules JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(flag_name)
);
```

#### 3. Configure as Variáveis de Ambiente
Crie um arquivo `.env` na raiz do serviço:

```env
# Banco de Dados PostgreSQL
DATABASE_URL=postgres://usuario:senha@localhost:5432/targeting_db

# Ou configure individualmente:
POSTGRES_USER=togglemaster
POSTGRES_PASSWORD=seu_password_seguro
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=targeting_db

# Serviço
PORT=8003

# Auth Service (para validação de chaves)
AUTH_SERVICE_URL=http://localhost:8001

# Ambiente
ENVIRONMENT=development
```

#### 4. Instale as Dependências
```bash
pip install -r requirements.txt
```

#### 5. Inicie o Serviço
```bash
gunicorn --bind 0.0.0.0:8003 app:app
```

O servidor estará disponível em `http://localhost:8003`.

### Testando Localmente

#### 1. Crie uma Chave de API
Primeiro, gere uma chave via Auth Service:

```bash
curl -X POST http://localhost:8001/admin/keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer admin-secreto-123" \
  -d '{"name": "targeting-service-client"}'

# Salve a chave retornada como SUA_CHAVE_API
```

#### 2. Health Check
```bash
curl http://localhost:8003/health
# Resposta esperada: {"status":"ok"}
```

#### 3. Criar Regras de Targeting para uma Flag

```bash
curl -X POST http://localhost:8003/targeting/enable-new-dashboard \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_CHAVE_API" \
  -d '{
    "rules": {
      "percentage": 50,
      "countries": ["BR", "US"],
      "user_segments": ["premium"],
      "blacklist": ["user-123", "user-456"]
    }
  }'

# Resposta esperada:
# {
#   "flag_name": "enable-new-dashboard",
#   "rules": {
#     "percentage": 50,
#     "countries": ["BR", "US"],
#     "user_segments": ["premium"],
#     "blacklist": ["user-123", "user-456"]
#   },
#   "created_at": "2025-05-17T10:30:00"
# }
```

#### 4. Obter Regras de uma Flag
```bash
curl http://localhost:8003/targeting/enable-new-dashboard \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: JSON com as regras
```

#### 5. Listar Todas as Regras
```bash
curl http://localhost:8003/targeting \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: lista de todas as regras
```

#### 6. Atualizar Regras
```bash
curl -X PUT http://localhost:8003/targeting/enable-new-dashboard \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_CHAVE_API" \
  -d '{
    "rules": {
      "percentage": 75,
      "countries": ["BR", "US", "MX"],
      "user_segments": ["premium", "beta-testers"]
    }
  }'
```

#### 7. Deletar Regras de uma Flag
```bash
curl -X DELETE http://localhost:8003/targeting/enable-new-dashboard \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: {"message":"Regras de targeting deletadas"}
```

## 🔧 Variáveis de Ambiente

### Obrigatórias
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `POSTGRES_USER` | Usuário PostgreSQL | `togglemaster` |
| `POSTGRES_PASSWORD` | Senha PostgreSQL | `senha_forte_123` |
| `POSTGRES_HOST` | Host PostgreSQL | `localhost` |
| `POSTGRES_PORT` | Porta PostgreSQL | `5432` |
| `POSTGRES_DB` | Nome do banco de dados | `targeting_db` |
| `AUTH_SERVICE_URL` | URL do Auth Service | `http://localhost:8001` |

**Nota:** Alternativamente, use `DATABASE_URL` ao invés das variáveis individuais.

### Opcionais
| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `PORT` | Porta do servidor | `8003` |
| `ENVIRONMENT` | Ambiente (development/production) | `development` |
| `LOG_LEVEL` | Nível de log | `INFO` |
| `MAX_CONNECTIONS` | Conexões máximas ao BD | `10` |

## 🔐 GitHub Secrets Necessários

Configure os seguintes secrets no GitHub para CI/CD:

```yaml
POSTGRES_USER
  Descrição: Usuário PostgreSQL
  Valor: togglemaster

POSTGRES_PASSWORD
  Descrição: Senha PostgreSQL
  Valor: <sua-senha-forte>

POSTGRES_HOST
  Descrição: Host PostgreSQL
  Valor: db.example.com

POSTGRES_PORT
  Descrição: Porta PostgreSQL
  Valor: 5432

POSTGRES_DB
  Descrição: Nome do banco de dados
  Valor: targeting_db

DATABASE_URL
  Descrição: String de conexão completa
  Valor: postgres://user:password@host:5432/targeting_db

AUTH_SERVICE_URL
  Descrição: URL do Auth Service
  Valor: http://auth-service:8001

DOCKERHUB_USERNAME
  Descrição: Docker Hub username
  Valor: <seu-username>

DOCKERHUB_TOKEN
  Descrição: Docker Hub personal access token
  Valor: <seu-token>

REGISTRY_URL
  Descrição: URL do registry de container
  Valor: docker.io

SONAR_TOKEN
  Descrição: Token SonarQube
  Valor: <seu-token>
```

## 📊 Endpoints da API

### Públicos (sem autenticação)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/health` | Verifica saúde do serviço |

### Protegidos (requer Bearer Token)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/targeting` | Lista todas as regras |
| POST | `/targeting/{flag_name}` | Cria regras para uma flag |
| GET | `/targeting/{flag_name}` | Obtém regras de uma flag |
| PUT | `/targeting/{flag_name}` | Atualiza regras de uma flag |
| DELETE | `/targeting/{flag_name}` | Deleta regras de uma flag |

## 📋 Formato das Regras (JSON)

As regras são armazenadas como JSON. Exemplo completo:

```json
{
  "percentage": 50,
  "countries": ["BR", "US", "MX"],
  "user_segments": ["premium", "beta-testers"],
  "blacklist": ["user-123", "user-456"],
  "whitelist": ["user-789"],
  "start_date": "2025-05-17T00:00:00Z",
  "end_date": "2025-06-17T23:59:59Z"
}
```

### Campos das Regras

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `percentage` | número (0-100) | Percentual de usuários que recebem a flag |
| `countries` | array | Códigos de país (ISO 3166-1 alpha-2) |
| `user_segments` | array | Segmentos de usuários (ex: premium, beta) |
| `blacklist` | array | IDs de usuários que NÃO recebem a flag |
| `whitelist` | array | IDs de usuários que SEMPRE recebem a flag |
| `start_date` | ISO 8601 | Data/hora de início |
| `end_date` | ISO 8601 | Data/hora de fim |

## 🎯 Exemplos de Regras

### Exemplo 1: Rollout progressivo
```json
{
  "percentage": 10,
  "start_date": "2025-05-17T00:00:00Z",
  "end_date": "2025-05-24T00:00:00Z"
}
```

### Exemplo 2: Apenas usuários premium em certos países
```json
{
  "user_segments": ["premium"],
  "countries": ["BR", "US"]
}
```

### Exemplo 3: Whitelist + Blacklist
```json
{
  "percentage": 50,
  "whitelist": ["user-vip-1", "user-vip-2"],
  "blacklist": ["user-problematic-1"]
}
```

## 🏗️ Arquitetura

```
Request → Middleware Auth → Flask Route → Validator → Database (JSONB) → Response
                ↓
         Validação na Auth Service
```

## 🐛 Troubleshooting

### Problema: "psycopg2.OperationalError: could not connect"
**Solução:** Verifique credenciais do PostgreSQL
```bash
psql -U seu_usuario -d targeting_db -c "SELECT 1"
```

### Problema: "Authorization header obrigatório"
**Solução:** Adicione o header Authorization
```bash
curl -H "Authorization: Bearer sua_chave" ...
```

### Problema: "Invalid JSON in rules"
**Solução:** Valide a estrutura JSON das regras

### Problema: "Flag_name already exists"
**Solução:** Use PUT para atualizar ao invés de POST

## 📊 Monitoramento

### Logs
```bash
docker logs targeting-service
# ou localmente
tail -f logs/targeting-service.log
```

### Métricas
- Número total de regras
- Taxa de atualizações
- Tempo de resposta

## 📈 Boas Práticas

1. **Sempre use whitelist para usuários especiais** (VIP, testers)
2. **Valide percentuais** (devem estar entre 0-100)
3. **Teste regras antes de aplicar** em produção
4. **Documenta o propósito** de cada regra
5. **Use datas de fim** para rollouts temporários

## 📚 Recursos Adicionais

- [Flask Documentation](https://flask.palletsprojects.com/)
- [PostgreSQL JSON Documentation](https://www.postgresql.org/docs/current/datatype-json.html)
- [ToggleMaster Architecture](../README.md)

## 👥 Suporte

Para dúvidas ou problemas, abra uma issue no repositório principal ou entre em contato com o time DevOps.
