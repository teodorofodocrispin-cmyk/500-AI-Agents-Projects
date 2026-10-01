# SENTINEL — Transaction Safety Oracle Agent

A pre-execution safety oracle for autonomous AI agents. Call SENTINEL **before**
an agent signs any on-chain transaction and receive a signed **SAFE / UNSAFE /
UNKNOWN** verdict.

This agent is a thin client for the [SENTINEL](https://sentinel-agent.dev) API.
The oracle logic — contract security scanning (GoPlus), call simulation
(Alchemy), and ed25519 receipt signing — lives behind the API, so no
security-critical code ships in this repo.

> **Author affiliation:** This agent wraps the SENTINEL API, a project by the same author (teodorofodocrispin-cmyk). It is an official client, not a third-party wrapper.

## Why it matters

Autonomous agents that move funds will, at some point, be asked to sign a
malicious or buggy transaction. SENTINEL is the guardrail: verify first, sign
second. It returns a verdict plus a cryptographically signed receipt that can be
audited after the fact.

- SAFE / UNSAFE / UNKNOWN verdict
- Signed receipt (ed25519) for audit trails
- Checks contract security (GoPlus) and simulates the call (Alchemy)
- Fail-closed: if the oracle is unreachable, the verdict is UNKNOWN and the
  agent must NOT sign

## Quick start

```bash
pip install -r requirements.txt
python agent.py --tx '{"chain":"base","to":"0x0000...","data":"0x","value":"0"}'
```

SENTINEL has no free tier: each verdict is paid per call via
[x402](https://x402.org) (USDC on Base, about $0.005 for a typical transaction).
Without a payment the agent **fails closed** (`verdict: UNKNOWN`,
`safe_to_sign: false`).

The easiest way to pay is to let an x402-capable client sign and retry the call
for you (for example `npx agentcash fetch https://sentinel-agent.dev/v1/guard`).
If you already have a signed x402 payment payload (the base64 value an x402
client sends in `PAYMENT-SIGNATURE`), pass it to this agent:

```bash
python agent.py --tx '<tx json>' --payment "<base64_payment_payload>"
# or: SENTINEL_PAYMENT=<base64_payment_payload> python agent.py --tx '<tx json>'
```

## Sample output

```json
{
  "verdict": "SAFE",
  "safe_to_sign": true,
  "receipt": "signed-ed25519-receipt...",
  "checks": {"contract_security": "pass", "simulation": "no-revert"}
}
```

(Runtime: ~2–5 s per call. End-to-end demo under 10 min.)

## Ethical considerations

- Fail-closed by design: an unreachable oracle blocks the transaction, it never
  silently approves it.
- The signed receipt is the agent's audit trail — keep it for compliance.
- SENTINEL reduces (does not eliminate) risk. High-value transactions should
  still have human oversight.

## Compliance context

Aligned with agent-safety standards for autonomous on-chain agents. Use wherever
an AI agent holds a wallet and signs transactions.

## License

MIT (repository root). The SENTINEL API is a separate service; see its terms.
