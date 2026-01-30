# Scheduler API

> Send messages to the future - with natural language support! 🇧🇷

A webhook scheduling service built by DinastIA Community - the largest AI Agents community in Brazil.

## ✨ Features

- **Natural Language Dates**: Schedule using "amanhã às 14:00", "segunda-feira às 18h", etc.
- **Multiple Date Formats**: Supports Brazilian format, ISO, and relative times
- **One-time Execution**: Jobs run only once at the specified timestamp
- **Persistence**: Messages survive server restarts (stored in Redis)
- **Auto Cleanup**: Messages are removed after webhook execution
- **Past Date Validation**: Prevents scheduling for dates that already passed

## � Accepted Date Formats

The `scheduleTo` field accepts many formats:

### 🇧🇷 Linguagem Natural (Português)

| Entrada | Interpretação |
|---------|---------------|
| `"amanhã às 14:00"` | Amanhã, 14:00 |
| `"hoje às 18h"` | Hoje, 18:00 |
| `"hoje às 18:30"` | Hoje, 18:30 |
| `"segunda-feira às 9:00"` | Próxima segunda, 09:00 |
| `"próximo sábado às 10:00"` | Sábado que vem, 10:00 |
| `"daqui 2 horas"` | Hora atual + 2h |
| `"em 30 minutos"` | Hora atual + 30min |
| `"depois de amanhã às 15:00"` | Depois de amanhã, 15:00 |

### 📆 Formato Brasileiro

| Entrada | Interpretação |
|---------|---------------|
| `"30-01-2026 14:00"` | 30/Jan/2026, 14:00 |
| `"30/01/2026 14:00"` | 30/Jan/2026, 14:00 |
| `"30-01-2026 14:00:30"` | 30/Jan/2026, 14:00:30 |

### 🌐 Formato ISO

| Entrada | Interpretação |
|---------|---------------|
| `"2026-01-30T14:00:00"` | Horário local |
| `"2026-01-30T14:00:00Z"` | UTC |
| `"2026-01-30T14:00:00-03:00"` | Com offset |

### ⏱️ Tempo Relativo

| Entrada | Interpretação |
|---------|---------------|
| `"in 2 hours"` | +2 horas |
| `"in 30 minutes"` | +30 minutos |
| `"next monday at 9am"` | Próxima segunda, 09:00 |

## 📦 Payload Structure

### Request Body

```json
{
  "id": "unique-message-id",
  "scheduleTo": "amanhã às 14:00",
  "payload": {
    "any": "data",
    "you": "want",
    "to": "send"
  },
  "webhookUrl": "https://your-webhook.com/endpoint",
  "timezone": "America/Sao_Paulo"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `id` | string | ✅ | ID único da mensagem |
| `scheduleTo` | string | ✅ | Data/hora (qualquer formato aceito) |
| `payload` | object | ✅ | Dados enviados ao webhook |
| `webhookUrl` | string | ✅ | URL que receberá o POST |
| `timezone` | string | ❌ | Timezone (default: `America/Sao_Paulo`) |

### Response (Sucesso)

```json
{
  "status": "scheduled",
  "messageId": "unique-message-id",
  "scheduledFor": "2026-01-30T17:00:00+00:00",
  "parsedTime": "30-01-2026 14:00",
  "inputTime": "amanhã às 14:00",
  "timezone": "America/Sao_Paulo"
}
```

| Campo | Descrição |
|-------|-----------|
| `scheduledFor` | Horário em UTC (quando vai executar) |
| `parsedTime` | Como o sistema interpretou sua entrada |
| `inputTime` | O que você enviou |

### Response (Erro - Data Passada)

```json
{
  "detail": "Data/hora já passou! Horário atual: 30-01-2026 08:15. Solicitado: 29-01-2026 14:00"
}
```

## 🔧 Requirements

- Python 3.9+
- Redis server
- Dependencies: `pip install -r requirements.txt`

## 📦 Installation

```bash
git clone https://github.com/Alan-silva01/scheduler.git
cd scheduler
pip install -r requirements.txt
```

Create `.env` file:
```env
REDIS_URL=redis://localhost:6379
API_TOKEN=your-secret-token-here
```

## 🚀 Running

```bash
python scheduler_api.py
```

Server starts on `http://localhost:8000`

## 🔐 Authentication

All endpoints (except `/health`) require Bearer token:

```
Authorization: Bearer your-secret-token-here
```

## 📡 API Endpoints

### POST /messages - Create Scheduled Message

```bash
curl -X POST http://localhost:8000/messages \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "msg-123",
    "scheduleTo": "amanhã às 14:00",
    "payload": {"mensagem": "Olá!"},
    "webhookUrl": "https://your-webhook.com"
  }'
```

### GET /messages - List Scheduled Messages

```bash
curl http://localhost:8000/messages \
  -H "Authorization: Bearer your-token"
```

### DELETE /messages/{id} - Cancel Scheduled Message

```bash
curl -X DELETE http://localhost:8000/messages/msg-123 \
  -H "Authorization: Bearer your-token"
```

### GET /health - Health Check (No auth)

```bash
curl http://localhost:8000/health
```

## ☁️ Deploy (Render + Upstash)

1. Create Redis on [Upstash](https://upstash.com) (free tier)
2. Create Web Service on [Render](https://render.com) (free tier)
3. Set environment variables:
   - `REDIS_URL`: From Upstash
   - `API_TOKEN`: Your secret
4. Use [cron-job.org](https://cron-job.org) to ping `/health` every 5 min (keeps alive)

## 📚 Dependencies

- `fastapi` - Web framework
- `uvicorn` - ASGI server
- `redis` - Redis client
- `apscheduler` - Job scheduling
- `dateparser` - Natural language date parsing
- `pytz` - Timezone handling
- `requests` - HTTP client
- `pydantic` - Data validation
- `python-dotenv` - Environment variables

## ⚠️ Error Codes

| Code | Description |
|------|-------------|
| `400` | Invalid date format or date in the past |
| `401` | Missing or invalid token |
| `404` | Message not found |
| `409` | Message ID already exists |
| `500` | Internal error |

## 👥 About

Developed by [DinastIA Community](https://github.com/DinastIA-UK) - the largest AI Agents community in Brazil.

## 📄 License

MIT License