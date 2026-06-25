# RAG Agent — AIO Strategy Knowledge Base
> Bootcamp agai-04 | PALO IT | Proyecto Final S26

Sistema de Recuperación Aumentada por Generación (RAG) construido sobre n8n Cloud, Pinecone y Google Gemini. Permite consultar la base de conocimiento de AIO Strategy mediante lenguaje natural con respuestas fundamentadas en documentos indexados.

---

## 🏗️ Arquitectura

```
Usuario
  │
  ▼
Webhook POST /aio-rag-chat
  │  { message, sessionId, history }
  ▼
Parse & Validate Input
  │  Valida payload, construye geminiBodyString
  ▼
Embed User Query
  │  HTTP POST → Gemini Embedding API
  │  Modelo: gemini-embedding-001 (3072 dims)
  ▼
Build Pinecone Body
  │  Construye pineconeBodyString con vector
  ▼
Query Pinecone (Top-5 Retrieval)
  │  HTTP POST → Pinecone /query
  │  Namespace: aio-strategy-docs | Metric: cosine
  ▼
Build RAG Context
  │  Filtra matches > 0.35 score
  │  Construye bloques de contexto con fuentes
  ▼
Build LLM Prompt
  │  System prompt AIO Strategy + contexto + historial
  │  Construye geminiBodyString para LLM
  ▼
LLM Generate Answer (Gemini 2.5 Flash)
  │  HTTP POST → Gemini generateContent API
  ▼
Format Final Response
  │  Extrae texto, actualiza historial (últimos 20 turnos)
  ▼
Respond to Webhook
     { response, sources, matchCount, sessionId, history }
```

---

## 🛠️ Stack Tecnológico

| Componente | Tecnología |
|---|---|
| Orquestación | n8n Cloud |
| Vector Store | Pinecone (cosine, 3072 dims) |
| Embeddings | Google Gemini `gemini-embedding-001` |
| LLM | Google Gemini `gemini-2.5-flash` |
| Trigger | Webhook HTTP POST |

---

## 📂 Workflows

### Workflow 1 — Vector Store Builder ✅
Indexa documentos de Google Drive en Pinecone automáticamente.

```
Google Drive Trigger
  → Extract Text (Code)
  → Text Chunker
  → Build Gemini Embed Body (Code)
  → HTTP Request: Gemini Embeddings
  → Build Pinecone Upsert Payload (Code)
  → HTTP Request: Pinecone Upsert
  → Log Success
```

### Workflow 2 — RAG Agent ✅
Responde consultas en lenguaje natural usando los documentos indexados.

```
Webhook → Parse → Embed → Retrieve → Context → Prompt → LLM → Format → Respond
```

---

## 🔌 API Reference

### Endpoint
```
POST /webhook/aio-rag-chat
```

### Request
```json
{
  "message": "¿Qué servicios ofrece AIO Strategy?",
  "sessionId": "session-001",
  "history": []
}
```

### Response
```json
{
  "response": "AIO Strategy ofrece los siguientes servicios:\n* Diagnóstico de madurez de datos\n* Implementación de pipelines de ML\n* Automatización de procesos con AI\n* Dashboards de inteligencia de negocio\n\nFuente: [aio_strategy_sample]",
  "sources": ["aio_strategy_sample"],
  "matchCount": 1,
  "sessionId": "session-001",
  "history": [
    { "role": "user", "content": "¿Qué servicios ofrece AIO Strategy?" },
    { "role": "assistant", "content": "AIO Strategy ofrece..." }
  ],
  "metadata": {
    "model": "gemini-2.5-flash",
    "embeddingModel": "gemini-embedding-001",
    "hasContext": true,
    "timestamp": "2026-06-25T17:36:48.566Z"
  }
}
```

### Test con curl
```bash
curl -X POST https://<tu-instancia>.app.n8n.cloud/webhook/aio-rag-chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "¿Qué servicios ofrece AIO Strategy?",
    "sessionId": "test-001",
    "history": []
  }'
```

---

## ⚙️ Configuración

### Variables requeridas

| Variable | Descripción |
|---|---|
| `GEMINI_API_KEY` | Google AI Studio API Key |
| `PINECONE_API_KEY` | Pinecone API Key |

### Pinecone Index
```
Index:      aio-strategy-kb
Host:       aio-strategy-kb-e77nyao.svc.aped-4627-b74a.pinecone.io
Namespace:  aio-strategy-docs
Dimensions: 3072
Metric:     cosine
```

### Patrón crítico — HTTP Requests en n8n
Todos los HTTP Requests hacia APIs externas usan el patrón **Raw body**:
- Body Content Type: `Raw`
- Content Type: `text/plain`
- El JSON se pre-construye como string en el nodo Code anterior

Este patrón resuelve el problema de expresiones dinámicas dentro de JSON en n8n Cloud.

---

## 🎯 Dominio

**AIO Strategy** — Consultora B2B de AI y Datos  
Sector: Manufactura, Agroindustria y Logística  
Región: Bajío, México (Querétaro, Guanajuato, SLP, Aguascalientes)  
Web: [aiostrategy.tech](https://aiostrategy.tech)

---

## 👤 Autor

**Juan Miguel Ramírez de Jesús**  
AI Engineer & Data Scientist  
GitHub: [github.com/jmiguel-r](https://github.com/jmiguel-r)  
Bootcamp: agai-04 Agentic AI — PALO IT
