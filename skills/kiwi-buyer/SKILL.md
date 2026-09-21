---
name: kiwi-buyer
description: Kiwi Sourcing & Negotiation Kit. Use when the user wants to source general goods, find suppliers, send RFQs, compare quotes, negotiate, or ask about lead times/MOQ. Covers discovery, RFQ, negotiation, non-binding agreements and trade handoff through the kiwi-buyer-mcp tools.
version: 0.2.0
author: harrylabsj
license: Apache-2.0
metadata:
  hermes:
    tags: [commerce, sourcing, procurement, rfq, negotiation, mcp, kiwi, buy, shopping, 购买, 采购, 购物]
    category: commerce
---

# Kiwi Buyer (Sourcing & Negotiation Kit)

Kiwi provides cross-merchant discovery, request-for-quote and commerce
negotiation. The host agent owns the conversation and user confirmation; Kiwi
owns merchant discovery, A2A/KNP negotiation and non-binding agreements. Kiwi
never handles payments, never places orders and never locks inventory.

The buyer discovers merchants through the catalog only, then talks to the
merchant directly over A2A. Do not probe or connect to a local marketplace,
`shopping-cli`, or any `127.0.0.1` service.

A Chinese translation of this skill ships alongside it
([SKILL.zh-CN.md](SKILL.zh-CN.md)); this English file is the default and
authoritative version.

## Tool flow

| User intent | Tool |
|---|---|
| Find merchants or suppliers | `kiwi_search` |
| Request quotes (RFQ) | `kiwi_request_quotes` |
| Check quotes and task status | `kiwi_get_task` |
| Counter-offer or clarify | `kiwi_negotiate` |
| Accept the non-binding agreement | `kiwi_accept_agreement` |
| Read the agreement and audit digest | `kiwi_get_agreement` |
| Produce checkout, PO or contact entry point | `kiwi_handoff` |
| Approve after user confirmation | `kiwi_approve` |
| Reject an approval request | `kiwi_reject` |

Typical flow:

1. `kiwi_search` discovers candidate suppliers.
2. `kiwi_request_quotes` starts the RFQ; an idempotency key and a valid
   `CommerceIntent` are required.
3. `kiwi_get_task` shows quotes, partial failures and pending approvals.
4. Use `kiwi_negotiate` for counter-offers or clarifications when needed.
5. `kiwi_accept_agreement` accepts the terms; if it returns
   `approval_required`, first show the user the candidates, terms and amounts,
   get explicit confirmation, call `kiwi_approve`, then retry with the
   `approval_id`.
6. Verify the agreement and digest with `kiwi_get_agreement`, then call
   `kiwi_handoff`.

## CommerceIntent rules

Pass only the fields required to complete the purchase. Never put chat
history, host memory, email addresses or unrelated profile data into the
intent. Every item needs a short product query and an object-shaped quantity:

```json
{
  "intent_type": "purchase",
  "items": [{ "query": "insulated bottle", "quantity": { "value": 2, "unit": "pcs" } }],
  "constraints": { "currency": "CNY", "deadline": "<RFC3339>" },
  "context_projection": {
    "disclosure_boundary": "commerce_required",
    "projected_fields": ["items", "constraints"]
  }
}
```

Prefer short product words in the item `query`; move specifications into
`constraints` or `preferences` to avoid catalog matching failures.

## Authorization and error handling

- `kiwi_accept_agreement` and `kiwi_handoff` require user authorization by
  default; never let the model approve on its own.
- `authorization_denied` and `approval_denied` are hard refusals — do not
  retry them automatically.
- `task_not_found` and `task_expired` require re-querying or re-quoting.
- On `partial_success`, keep the successful quotes and report the failed
  items separately.
- A trade handoff only produces a next-step entry point; verify the target
  URL before the user pays, and state clearly that Kiwi itself has neither
  paid nor placed an order.
