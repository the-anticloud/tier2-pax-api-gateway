# Deploy Guide — PAX_API_GATEWAY
**Platform:** Anticloud PAX 27B harness | Air-gap capable

## Prerequisites
Python 3.11+, grpcio 1.62+, FastAPI 0.110+, python-jose (JWT), AIOSS_FORMAT

## Environment
16GB RAM. GPU on PAX_INFERENCE_CORE, not gateway. Gateway is CPU-only. Listens on :8443 (HTTPS) and :50051 (gRPC).

## AIOSS Integration
```bash
aioss init --module PAX_API_GATEWAY --output ./pax_api_gateway.aioss
aioss append --chain ./pax_api_gateway.aioss --payload ./output.bin --module PAX_API_GATEWAY
aioss verify --chain ./pax_api_gateway.aioss
```

## Air-Gap Deployment
```bash
pip download -r requirements.txt -d ./wheels/
# Transfer to air-gap host
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX 27B Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(model_path="./pax-27b-q4.gguf", module="PAX_API_GATEWAY",
                     aioss_chain="./pax_api_gateway.aioss",
                     classification="L5_NARROW_L2_GENERAL")
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./pax_api_gateway.aioss --verbose
python -m pax_api_gateway.tests.smoke
```
