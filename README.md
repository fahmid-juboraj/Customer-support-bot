# NovaMart Multi-Agent E-commerce RAG (Udacity AWS Agent Engineer, Project 3)

A multi-agent customer-support system on Amazon Bedrock AgentCore, built with the Strands Agents SDK.

## Architecture
- **Orchestrator** (Claude Haiku 4.5, temp 0.0) routes each request with six routing rules to workers:
  - **InventoryAgent** (temp 0.1): order status, customer tier, order history (DynamoDB).
  - **PolicyAgent** (coordinator temp 0.2): three retriever agents (returns / shipping / warranty, temp 0.0) run in parallel via `ThreadPoolExecutor(max_workers=3)` against Bedrock Knowledge Bases (Titan Embed v2, S3 Vectors).
  - **RefundAgent** (temp 0.1): checks delivered status and the 60/30-day tier window, then initiates the return.
  - **CommunicationAgent** (temp 0.3): composes the final customer reply and does arithmetic.
- **WorkflowState** (DynamoDB) shares worker outputs through version-based optimistic locking.
- **Guardrail** (STANDARD tier, cross-region `us.guardrail.v1:0`): denies competitor, pricing-negotiation, and legal-threat topics; blocks card/SSN, anonymizes email/phone.
- **AgentCore Memory** (SESSION_SUMMARY, 7 days), **Runtime** deployed with the AgentCore CLI, **Observability** (CloudWatch + X-Ray, 100% sampling).

## What was done
1. Completed every TODO in `src/agent_orchestrator.py`.
2. Deployed infrastructure (`infrastructure/starter_stack.yaml`, `seed_data.py`), created three Knowledge Bases and synced policy docs.
3. Deployed runtime, guardrail, memory and observability; verified routing for three scenarios.
4. `python tests/test_agent.py all` -> **120/120**.

## Screenshots
- `screenshots/01_test_120_of_120.png` test score
- `screenshots/02_xray_service_map.png` X-Ray map: Orchestrator -> workers -> Knowledge Bases
- `screenshots/03_routing_summary.png`, `04_guardrail_test.png`

## Setup
Copy `.env.example` to `.env`, fill in KB IDs, runtime ARN and guardrail ID, then see `README_STARTER.md`.
Never commit `.env` or AWS credentials. Tear down with `python infrastructure/cleanup.py --yes` when finished.
