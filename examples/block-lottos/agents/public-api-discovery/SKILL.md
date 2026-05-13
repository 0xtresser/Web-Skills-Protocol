---
name: block-lottos-public-api-discovery
description: Discover Block Lottos public API and agent-readable documents.
version: 1.0.0
---

# Block Lottos Public API Discovery

Use this skill when an agent needs machine-readable context for Block Lottos without scraping or guessing.

## Public documents

- llms.txt: https://blocklottos.com/llms.txt
- agents.txt: https://blocklottos.com/agents.txt
- OpenAPI: https://blocklottos.com/openapi.json
- AI plugin manifest: https://blocklottos.com/.well-known/ai-plugin.json
- API docs: https://blocklottos.com/api-docs

## Public read endpoints

- Current jackpot: `GET https://blocklottos.com/api/jackpot.php`
- Site stats and next draw: `GET https://blocklottos.com/api/stats.php`

## Guidance

Prefer the OpenAPI spec and agent docs over scraping HTML. Read-only API calls require no API key. Any payment, ticket, or ad-payment action must be user-controlled and explicitly signed by the user's wallet.
