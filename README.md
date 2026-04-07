# targeting-service (Python)

Este é o serviço de regras de segmentação (targeting) do projeto ToggleMaster. Ele é responsável por gerenciar regras complexas (ex: "50% dos usuários", "usuários do país X") para uma feature flag específica.

**IMPORTANTE:** Este serviço também é protegido e depende que o `auth-service` esteja rodando (ex: em `http://localhost:8001`).

## 📦 Pré-requisitos (Local)

* [Python](https://www.python.org/) (versão 3.9 ou superior)
* [PostgreSQL](https://www.postgresql.org/download/)
* O `auth-service` deve estar rodando.

## 🚀 Rodando Localmente

1.  **Clone o repositório** e entre na pasta `targeting-service`.

2.  **Prepare o Banco de Dados:**
    * Crie um banco de dados no seu PostgreSQL (ex: `targeting_db`).
    * Execute o script `db/init.sql` para criar a tabela `targeting_rules`:
        ```bash
        psql -U seu_usuario -d targeting_db -f db/init.sql
        ```

3.  **Configure as Variáveis de Ambiente:**
    Crie um arquivo chamado `.env` na raiz desta pasta (`targeting-service/`) com o seguinte conteúdo:
    ```.env
    # String de conexão do seu banco de dados PostgreSQL
    DATABASE_URL="postgres://SEU_USUARIO:SUA_SENHA@localhost:5432/targeting_db"
    
    # Porta que este serviço (targeting-service) irá rodar
    PORT="8003"
    
    # URL do auth-service (que deve estar rodando na porta 8001)
    AUTH_SERVICE_URL="http://localhost:8001"
    ```

4.  **Instale as Dependências:**
    ```bash
    pip install -r requirements.txt
    ```

5.  **Inicie o Serviço:**
    ```bash
    gunicorn --bind 0.0.0.0:8003 app:app
    ```
    O servidor estará rodando em `http://localhost:8003`.

## 🔧 Variáveis de Ambiente

O serviço requer as seguintes variáveis de ambiente para funcionar:

### Obrigatórias
- **`DATABASE_URL`** - String de conexão completa com o PostgreSQL
  - Exemplo: `postgres://usuario:senha@localhost:5432/targeting_db`
  - Alternativamente, pode ser montada a partir das variáveis individuais abaixo
- **`AUTH_SERVICE_URL`** - URL do Auth Service para validação de chaves
  - Exemplo: `http://localhost:8001`

### Variáveis Individuais (se DATABASE_URL não for definida)
- **`POSTGRES_USER`** - Usuário do PostgreSQL
- **`POSTGRES_PASSWORD`** - Senha do PostgreSQL  
- **`POSTGRES_HOST`** - Host do PostgreSQL (ex: `localhost`)
- **`POSTGRES_PORT`** - Porta do PostgreSQL (ex: `5432`)
- **`POSTGRES_DB`** - Nome do banco de dados

### Outras Variáveis
- **`PORT`** - Porta onde o serviço irá rodar (padrão: `8003`)

### Exemplo de arquivo .env
```env
# Opção 1: URL completa
DATABASE_URL="postgres://togglemaster:senha123@localhost:5432/targeting_db"

# Opção 2: Variáveis individuais
POSTGRES_USER="togglemaster"
POSTGRES_PASSWORD="senha123"
POSTGRES_HOST="localhost"
POSTGRES_PORT="5432"
POSTGRES_DB="targeting_db"

# Configurações do serviço
PORT="8003"
AUTH_SERVICE_URL="http://localhost:8001"
```

### Notas Importantes
- O **Auth Service** deve estar acessível na URL configurada
- O banco de dados PostgreSQL deve ter a tabela `targeting_rules` criada (use `db/init.sql`)
- Todas as requisições (exceto `/health`) exigem header `Authorization: Bearer <chave>`
- As regras são armazenadas como JSON no campo `rules` da tabela

## 🧪 Testando os Endpoints

Lembre-se de obter sua `SUA_CHAVE_API` no `auth-service` (veja o README do `flag-service`).

**1. Verifique a Saúde (Health Check):**
```bash
curl http://localhost:8003/health
```
Saída esperada: `{"status":"ok"}`

**2. Crie uma nova Regra de Segmentação:** Vamos criar uma regra para a flag enable-new-dashboard (que você criou no flag-service). Esta regra fará a flag aparecer para 50% dos usuários.
```bash
curl -X POST http://localhost:8003/rules \
-H "Content-Type: application/json" \
-H "Authorization: Bearer SUA_CHAVE_API" \
-d '{
    "flag_name": "enable-new-dashboard",
    "is_enabled": true,
    "rules": {
        "type": "PERCENTAGE",
        "value": 50
    }
}'
```
Saída esperada: (Um JSON com os dados da regra criada).

**3. Busque a Regra que você criou:**
```bash
curl http://localhost:8003/rules/enable-new-dashboard \
-H "Authorization: Bearer SUA_CHAVE_API"
```
Saída esperada: (O JSON da regra que você acabou de criar).

**4. Atualize a Regra (mude para 75%):**
```bash
curl -X PUT http://localhost:8003/rules/enable-new-dashboard \
-H "Content-Type: application/json" \
-H "Authorization: Bearer SUA_CHAVE_API" \
-d '{
    "rules": {
        "type": "PERCENTAGE",
        "value": 75
    }
}'
```
Saída esperada: (O JSON da regra atualizada, com `"value": 75`).
