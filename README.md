# Finia Backend

Backend em Python (FastAPI) para o dashboard financeiro pessoal.
Serve como proxy seguro para a API do Claude (Anthropic) e para a Pluggy (Open Finance Brasil).

## Estrutura

```
finia-backend/
├── app/
│   ├── main.py              # Entrada da aplicação, CORS, routers
│   ├── config.py            # Leitura do .env
│   ├── routers/
│   │   ├── claude.py        # POST /api/claude/chat
│   │   └── pluggy.py        # GET  /api/pluggy/...
│   └── services/
│       ├── claude_service.py   # Chamada à API Anthropic
│       └── pluggy_service.py   # Chamada à API Pluggy
├── requirements.txt
├── Procfile                 # Para Railway / Heroku
├── .env.example             # Modelo das variáveis de ambiente
└── .gitignore
```

## Pré-requisitos

- Python 3.11+
- Conta na [Anthropic](https://console.anthropic.com) (chave da API)
- Conta na [Pluggy](https://dashboard.pluggy.ai) (clientId + clientSecret)

---

## Rodando localmente

### 1. Clone e entre na pasta
```bash
git clone <seu-repo>
cd finia-backend
```

### 2. Crie e ative o ambiente virtual
```bash
python -m venv venv
source venv/bin/activate      # Mac/Linux
# venv\Scripts\activate       # Windows
```

### 3. Instale as dependências
```bash
pip install -r requirements.txt
```

### 4. Configure as variáveis de ambiente
```bash
cp .env.example .env
# Abra .env e preencha as chaves
```

### 5. Inicie o servidor
```bash
uvicorn app.main:app --reload
```

O servidor sobe em `http://localhost:8000`.
Documentação automática em `http://localhost:8000/docs`.

---

## Endpoints

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/health` | Verifica se o servidor está no ar |
| POST | `/api/claude/chat` | Envia mensagem para o Claude |
| GET | `/api/pluggy/connect-token` | Gera token para o Pluggy Widget |
| GET | `/api/pluggy/accounts/{item_id}` | Lista contas bancárias |
| GET | `/api/pluggy/transactions/{account_id}` | Lista transações |
| GET | `/api/pluggy/status/{item_id}` | Status da conexão bancária |

---

## Deploy

### Railway (recomendado — mais simples)
1. Crie conta em [railway.app](https://railway.app)
2. "New Project" → "Deploy from GitHub repo"
3. Adicione as variáveis de ambiente no painel do Railway (Settings → Variables)
4. O Railway detecta o `Procfile` automaticamente e faz o deploy

### Vercel
1. Instale: `npm i -g vercel`
2. Crie `vercel.json` na raiz:
   ```json
   {
     "builds": [{ "src": "app/main.py", "use": "@vercel/python" }],
     "routes": [{ "src": "/(.*)", "dest": "app/main.py" }]
   }
   ```
3. `vercel --prod`
4. Adicione as variáveis em Settings → Environment Variables

### VPS (Ubuntu)
```bash
# Instalar dependências
sudo apt update && sudo apt install python3.11 python3-pip -y

# Clonar e configurar
git clone <seu-repo> && cd finia-backend
pip install -r requirements.txt
cp .env.example .env && nano .env   # preencha as chaves

# Rodar com systemd ou supervisor
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

---

## Adaptando o frontend

No `dashboard-financeiro.html`, substitua as chamadas diretas às APIs por chamadas ao backend:

```javascript
// ANTES (inseguro — chave exposta)
fetch('https://api.anthropic.com/v1/messages', {
  headers: { 'x-api-key': 'sk-ant-...' }
})

// DEPOIS (seguro — chave no servidor)
fetch('https://seubackend.railway.app/api/claude/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ messages, financial_context })
})
```

```javascript
// Pluggy: gerar connect token antes de abrir o widget
const { token } = await fetch('/api/pluggy/connect-token').then(r => r.json())

// Pluggy: buscar contas após autenticação
const { accounts } = await fetch(`/api/pluggy/accounts/${itemId}`).then(r => r.json())

// Pluggy: buscar transações
const { transactions } = await fetch(`/api/pluggy/transactions/${accountId}`).then(r => r.json())
```
