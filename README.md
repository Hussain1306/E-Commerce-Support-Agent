# n8n MCP Support Agent

An n8n demo that shows **MCP (Model Context Protocol) Server and Client nodes working together** to solve a real-world problem: automating e-commerce customer support (order lookup, refund eligibility, and human escalation).

The tools live on an MCP server, and the AI agent discovers and calls them at runtime. No order data or policy is hardcoded into the agent.

## Architecture

```
┌───────────────────────────┐        MCP         ┌──────────────────────────────┐
│ Workflow 2: MCP Client    │  ───────────────►  │ Workflow 1: MCP Server       │
│                           │   (tool discovery  │                              │
│ Chat Trigger              │    + tool calls)   │ MCP Server Trigger           │
│   └─► AI Agent            │                    │   ├─ lookup_order            │
│        ├─ OpenAI Model    │                    │   ├─ get_refund_policy       │
│        ├─ Memory          │                    │   └─ create_escalation_ticket│
│        └─ MCP Client Tool │                    │                              │
└───────────────────────────┘                    └──────────────────────────────┘
```

## Files

| File | Description |
|------|-------------|
| `1_mcp_server_support_tools.json` | n8n workflow exposing three support tools via the MCP Server Trigger |
| `2_mcp_client_support_agent.json` | n8n workflow with a chat-driven AI agent that consumes those tools via the MCP Client Tool |

## The workflow the agent follows

1. Look up the order.
2. Fetch the refund policy.
3. Decide eligibility.
4. Escalate to a human if the order is ineligible, the customer disputes the decision, or the refund is over $100.

## Setup

**Requirements:** n8n (cloud or self-hosted, a recent version with MCP nodes) and an OpenAI API key.

1. Import `1_mcp_server_support_tools.json` into n8n and **activate** it.
2. Open the **MCP Server Trigger** node and copy its **Production URL**.
3. Import `2_mcp_client_support_agent.json`.
4. Add your OpenAI credential to the **OpenAI Chat Model** node.
5. Paste the Production URL into the **MCP Client** node's `endpointUrl` field. The transport is set to Streamable HTTP, so change it if your server node shows a different transport.
6. Open the chat (**Open chat** button) and try the prompts below.

## Demo prompts

The server includes mock orders so the demo works without any external accounts.

| Prompt | Expected behavior |
|--------|-------------------|
| `Where is order ORD-1002?` | Returns the in-transit status, carrier and ETA |
| `I want a refund for ORD-1001` | Delivered 7 days ago, so the refund is eligible |
| `Refund ORD-1003 please` | Delivered 53 days ago, so the refund is denied and a ticket is created |
| `Cancel ORD-1004` | Order is still processing, so it can be cancelled for a full refund |

## Why MCP?

- **Decoupled tools:** the team that owns the tools maintains the server, and agents pick up changes automatically.
- **Reusable:** any MCP-compatible client (Claude Desktop, Cursor, other n8n workflows) can connect to the same server.
- **Grounded answers:** the agent has no built-in knowledge of orders or policy, so it has to call tools instead of guessing.

## Taking it to production

- Replace the Code tool nodes with real integrations (Shopify, Jira, Slack, a database).
- Enable authentication (e.g. bearer token) on the MCP Server Trigger.
- Persist chat memory in Postgres or Redis instead of the in-memory buffer.
- Add logging and error handling around tool calls.

## Tech stack

n8n, MCP (Model Context Protocol), LangChain-based AI Agent node, OpenAI GPT-4o-mini

## Author

Hussain, AI/ML & Automation Engineer
