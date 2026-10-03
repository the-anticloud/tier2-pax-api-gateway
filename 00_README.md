# PAX API Gateway — OpenAI-Compatible API Server

**Status:** Production | **Version:** 1.0.0 | **Author:** PAX API Team  
**Domain:** 0-1.gg/pax/api-gateway

---

## What Is PAX API Gateway?

PAX API Gateway is an OpenAI-compatible API server that wraps all PAX inference and reasoning engines, providing a standard interface for any client expecting OpenAI's chat completions, embeddings, or completion APIs.

```
Client (OpenAI SDK, curl, custom)
    ↓
PAX API Gateway (OpenAI-compatible interface)
    ├─→ /v1/chat/completions (streaming + non-streaming)
    ├─→ /v1/embeddings
    ├─→ /v1/models (list available models)
    └─→ /v1/completions (legacy)
    ↓
PAX System (inference, reasoning, caching, routing)
    ↓
Response (standard OpenAI format)
```

---

## Key Specifications

| Aspect | Details |
|--------|---------|
| **Protocols** | HTTP/1.1, HTTP/2, WebSocket (SSE) |
| **API Compatibility** | OpenAI Chat Completions v1 (100% compatible) |
| **Models Supported** | pax-one-l5-narrow-27b, custom models via adapter registry |
| **Throughput** | 1000 req/sec (single instance) |
| **Latency (P95)** | <500ms first-token, <50ms/token streaming |
| **Authentication** | API keys, JWT tokens |
| **Rate Limiting** | Per-key, per-IP, per-tenant quotas |
| **Response Format** | JSON (streaming and buffered) |

---

## Architecture

### Layer 1: Request Handler
- Protocol translation (HTTP → internal PAX format)
- OpenAI API validation (schema checking)
- Authentication (API key verification)
- Rate limiting (quota enforcement)

### Layer 2: Model Router
- Model selection (default: pax-one-l5-narrow-27b)
- Temperature/top_p validation
- Max tokens enforcement
- Tool-calling preprocessing

### Layer 3: Backend Dispatch
- Routing via PAX_ROUTER
- Cache lookup (PAX_CACHE)
- Priority assignment (PAX_SCHEDULER)
- Security validation (PAX_SECURITY)

### Layer 4: Response Formatting
- OpenAI format compliance
- Streaming SSE format
- Error handling (standard OpenAI error codes)
- Usage tracking (tokens, cost)

---

## Performance Characteristics

### Throughput (Verified from PAX_RESULTS.md)
- **Throughput:** 100 tok/sec on H100, 30 tok/sec on RTX 3090 dual
- **Gateway overhead:** <5ms (3% of total latency)
- **Scaling:** Horizontal via Kubernetes

### Latency
- **First-token:** ~500ms P95 (dominated by inference)
- **Streaming:** ~50ms/token P95
- **Non-streaming:** ~5-10s for 100-token response

---

## Quick Start

### Installation
```bash
pip install pax-api-gateway

# Or docker
docker run -p 8000:8000 pax-api-gateway:latest
```

### Configuration (YAML)
```yaml
gateway:
  port: 8000
  host: 0.0.0.0
  workers: 8
  
  models:
    - name: "pax-one-l5-narrow-27b"
      default: true
      context_window: 8192
      quantization: "fp8"
  
  authentication:
    api_keys_enabled: true
    jwt_enabled: false
  
  rate_limiting:
    global_rps: 1000
    per_key_rps: 100
    burst_size: 200
  
  features:
    streaming: true
    function_calling: true
    vision: true
```

### Python Example (OpenAI SDK)
```python
from openai import OpenAI

# Point to PAX Gateway
client = OpenAI(
    api_key="pax-key-xxxxx",
    base_url="http://localhost:8000/v1"
)

# Standard OpenAI API call
response = client.chat.completions.create(
    model="pax-one-l5-narrow-27b",
    messages=[
        {"role": "user", "content": "What is 42 * 37?"}
    ],
    temperature=0.7
)
print(response.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="pax-one-l5-narrow-27b",
    messages=[{"role": "user", "content": "..."}],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")
```

### Curl Example
```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Authorization: Bearer pax-key-xxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "pax-one-l5-narrow-27b",
    "messages": [{"role": "user", "content": "What is 42 * 37?"}],
    "temperature": 0.7
  }'
```

### Docker Deployment
```bash
docker run -d \
  -p 8000:8000 \
  -e PAX_ROUTER_URL="http://pax-router:8001" \
  -e API_KEY_SECRET="pax-key-xxxxx" \
  pax-api-gateway:latest
```

---

## Integration Points

### Primary Consumers
- **PAX_ROUTER** — Gateway routes requests via router
- **PAX_CACHE** — Gateway checks cache before dispatch
- **PAX_SCHEDULER** — Gateway assigns priority
- **PAX_SECURITY** — Gateway enforces authentication
- **PAX_MONITOR_SYSTEM** — Gateway publishes API metrics

### Complementary
- **ANTICODE_AGENT** (Tier 1) — Uses Gateway for code generation
- **api-oss-gateway** (Tier 3) — Enterprise wrapper around Gateway
- **All PAX engines** — Accessed via Gateway

### Deployment
- **Kubernetes** — Pod deployment + HPA scaling
- **api-oss-monitor** (Tier 3) — Live API health tracking
- **OpenAI SDK** — Client compatibility

---

## Configuration Reference

### Endpoints

**Chat Completions (Primary)**
```
POST /v1/chat/completions
Content-Type: application/json
Authorization: Bearer $API_KEY

{
  "model": "pax-one-l5-narrow-27b",
  "messages": [...],
  "temperature": 0.7,
  "stream": false
}
```

**Embeddings**
```
POST /v1/embeddings

{
  "model": "pax-one-l5-narrow-27b",
  "input": "Text to embed"
}
```

**Models List**
```
GET /v1/models
```

---

## Security & Compliance

- **API key validation:** All requests require valid key
- **Rate limiting:** Per-client quotas prevent abuse
- **Input validation:** OpenAI schema compliance
- **Audit logging:** All requests logged via AIOSS
- **TLS:** All communication encrypted
- **Multi-tenant isolation:** Tenant segregation via API key

---

## Roadmap

- **Q4 2026:** Vision API support (images + video)
- **Q1 2027:** Fine-tuning API
- **Q2 2027:** Batch processing API
- **Q3 2027:** Audio API support

---

## References

- **OpenAI API Docs:** https://platform.openai.com/docs/api-reference
- **Source:** 0-1.gg/pax/api-gateway
- **GitHub:** github.com/0-1-gg/pax-api-gateway
- **Benchmarks:** PAX_RESULTS.md

---

**Next:** See APPENDIX/ for API patterns and integration
