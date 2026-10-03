# L5 Narrow / L2 General Classification — PAX_API_GATEWAY
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE

## L5 Narrow
PAX_API_GATEWAY enforces L5 Narrow by restricting all external API access to a well-defined PAX 27B request schema. No arbitrary code execution, no unauthenticated endpoints, no data exfiltration paths. Every API call is JWT-authenticated and AIOSS-chained.

## L2 General
L2 General means PAX_API_GATEWAY is the universal external interface for PAX across all deployment contexts — hospital web portal, defense operator console, robotics fleet manager.

## PAX Integration
API Gateway is the outermost PAX harness layer. All external requests to PAX 27B flow through it: auth validation, rate limiting, request routing to PAX_INFERENCE_CORE, response streaming back to caller.

## AIOSS Audit Relevance
Every API request/response pair (request hash + response hash + caller identity) produced by PAX_API_GATEWAY is appended to the AIOSS chain.
Chain formula: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n)
Tamper-evident, air-gap verifiable, no cloud dependency.

## Regulatory
NIST SP 800-95 (web services security), OAuth 2.0, OWASP API Security Top 10
