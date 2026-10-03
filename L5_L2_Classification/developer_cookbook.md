# Developer Cookbook — PAX_API_GATEWAY
**Stack:** Python 3.11, gRPC, FastAPI, JWT, rate limiting, AIOSS_FORMAT

## Core Usage Patterns

## Route a PAX inference request
```python
import httpx
resp = httpx.post("https://localhost:8443/v1/inference",
    headers={"Authorization": f"Bearer {jwt_token}"},
    json={"prompt": "Analyze biosignal data", "max_tokens": 512})
print(resp.json()["text"], resp.json()["chain_hash"])
```

## gRPC client
```python
import grpc
from pax_api_gateway.proto import gateway_pb2_grpc, gateway_pb2
channel = grpc.secure_channel("localhost:50051", creds)
stub = gateway_pb2_grpc.PAXGatewayStub(channel)
resp = stub.Infer(gateway_pb2.InferRequest(prompt=prompt_bytes, max_tokens=512))
```

## Issue JWT token
```python
from pax_api_gateway import TokenIssuer
issuer = TokenIssuer(secret="./jwt_secret.key")
token = issuer.issue(subject="clinical_team_001", scopes=["infer", "audit"])
```

## Rate limit configuration
```python
from pax_api_gateway import RateLimitConfig
config = RateLimitConfig(requests_per_minute=60, burst=10, per_subject=True)
```

## AIOSS Append
```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()
```

## Performance
Connection pooling for gRPC: `options=[('grpc.max_connection_idle_ms', 30000)]`. JWT validation cached per token (TTL = token expiry). Rate limiting via token bucket, not sliding window.

## Integration
Wraps PAX_INFERENCE_CORE. Used by INTE11ECT_APP, MIIRAI_CHAT, LIBERN_PLATFORM. Auth keys stored in MF_SO_PASSWORD_MANAGER.
