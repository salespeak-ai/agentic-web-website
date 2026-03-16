# Under the Hood: The Agentic Web — MCP Server Profiles for Every Business

*By the Agentic Web team at [Salespeak](https://salespeak.ai)*

## 1. What is the Agentic Web?

When someone asks an AI assistant "Can I order a gluten-free cake from Rosa's Bakery for Saturday?" — today's AI scrapes a website, guesses at the answer, and hopes for the best. It can't check real-time availability, verify pricing, or actually place the order.

The Agentic Web changes this. It's an open specification for turning any business — a bakery, a SaaS company, an e-commerce store — into an **MCP server** that AI agents can connect to directly. Instead of scraping and guessing, the agent calls a verified API, gets an authoritative answer, and can take action on the user's behalf.

The core idea is simple: **companies don't just have websites — they have AI-native endpoints that answer questions and take actions.**

It's built on [MCP](https://modelcontextprotocol.io) (Anthropic's Model Context Protocol), the same protocol that Claude, ChatGPT, and other AI agents already speak. Any MCP-compatible client connects to your server out of the box — no custom integrations, no vendor lock-in.

### The problem it solves

Today, AI-powered research is broken. AI scrapes SEO-biased content and presents it as objective truth. Users get hallucinated pricing, outdated feature lists, and "contact sales" dead ends — with no way to verify accuracy or take action.

The Agentic Web inverts this. Instead of AI pulling data *from* your website, your business pushes authoritative data *to* AI through a standard protocol:

```
CURRENT:   AI Agent → Scrapes Website → Guesses Answer
AGENTIC:   AI Agent → Calls MCP Server → Gets Verified Answer + Takes Action
```

### How it relates to UCP, MCP, and A2A

The Agentic Web isn't competing with other protocols — it complements them:

- **MCP** (Anthropic) is the transport layer. The Agentic Web is an MCP Server Profile — conventions for building your endpoint as a standard MCP server.
- **UCP** (Google, Shopify, Walmart, Stripe) handles commerce — product discovery, cart, checkout, payments. If you sell products, UCP handles the transaction.
- **A2A** (Google) enables agent-to-agent communication. When a user's AI assistant needs to negotiate with your company's AI agent, A2A handles that handoff.

The Agentic Web handles everything that's *not* a purchase: answering questions, qualifying buyers, scheduling demos, opening tickets, providing verified company information. You might implement all three — or just one. They're independent.


## 2. Who benefits?

### Buyers

No more navigating websites, parsing marketing speak, or filling out the same form for the tenth time. Ask a question in plain language, get a verified answer, and book a demo — all in one conversation. Your context flows naturally between steps, and you control exactly what information you share.

### Businesses

You define what AI can say about you. No more hallucinated features, wrong pricing, or outdated information. Every interaction collects structured qualification data (company size, use case, budget) that flows into your CRM. Leads get routed to the right person automatically — not a round-robin queue.

### AI agents

Instead of guessing from stale training data, agents call a live API and get the real answer. Responses are cryptographically signed, so the agent can tell users "They are SOC2 Type II certified, verified March 2026" — not "I think they might be SOC2 compliant." And agents can go beyond answering questions: they can actually book the demo, start the trial, or open the ticket.


## 3. How it works — a walkthrough

Let's walk through a complete interaction. Rosa's Bakery has set up an Agentic Web MCP server. A customer asks their AI assistant about ordering a cake.

### Step 1: Connect and initialize

The AI agent connects to Rosa's MCP server and performs the standard handshake:

```
POST https://rosasbakery.com/mcp
Content-Type: application/json
```

```json
// Client → Server
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-03-26",
    "capabilities": {},
    "clientInfo": {
      "name": "claude-desktop",
      "version": "1.0.0"
    }
  }
}
```

```json
// Server → Client
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-03-26",
    "capabilities": {
      "tools": { "listChanged": true }
    },
    "serverInfo": {
      "name": "rosas-bakery-mcp",
      "version": "1.0.0"
    }
  }
}
```

The server returns an `Mcp-Session-Id` header that the client includes on all subsequent requests.

### Step 2: Discover available tools

The agent calls `tools/list` to see what Rosa's Bakery can do:

```json
// Server → Client (tools/list response)
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "ask_question",
        "description": "Ask about our menu, hours, pricing, dietary options, or custom orders",
        "inputSchema": {
          "type": "object",
          "properties": {
            "question": { "type": "string" }
          },
          "required": ["question"]
        }
      },
      {
        "name": "place_order",
        "description": "Place a cake or pastry order for pickup or delivery",
        "inputSchema": {
          "type": "object",
          "properties": {
            "items": { "type": "array" },
            "pickup_date": { "type": "string", "format": "date" },
            "name": { "type": "string" },
            "phone": { "type": "string" }
          },
          "required": ["items", "pickup_date", "name", "phone"]
        }
      },
      {
        "name": "qualify",
        "description": "Provide contact details to unlock custom order pricing and catering options"
      }
    ]
  }
}
```

Notice: the same `tools/list` pattern works for any business. An enterprise SaaS company would return `ask_question`, `schedule_demo`, `request_quote`, `open_ticket`. A bakery returns `ask_question`, `place_order`. The protocol is identical — only the tools change.

### Step 3: Ask a question

The user asked: "Can I order a gluten-free birthday cake for Saturday?"

```json
// Client → Server
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "ask_question",
    "arguments": {
      "question": "Can I order a gluten-free birthday cake for Saturday?"
    }
  }
}
```

```json
// Server → Client
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Yes! We have gluten-free Vanilla Bean cakes in 6-inch ($30) and 8-inch ($42). We need 48 hours notice, so Saturday pickup is available if you order by Thursday."
      }
    ],
    "structuredContent": {
      "answer": "Gluten-free Vanilla Bean cakes available",
      "options": [
        { "size": "6-inch", "price": 30 },
        { "size": "8-inch", "price": 42 }
      ],
      "leadTime": "48 hours",
      "saturdayAvailable": true,
      "orderDeadline": "Thursday",
      "suggestedActions": ["place_order"],
      "verification": {
        "algorithm": "Ed25519",
        "keyId": "rosas-2025-06",
        "signature": "base64url-encoded-signature..."
      }
    }
  }
}
```

The response has two parts:
- **`content`**: Human-readable text the AI shows to the user.
- **`structuredContent`**: Machine-readable data with pricing, availability, and a cryptographic signature proving this came directly from Rosa's Bakery — not from a hallucination.

### Step 4: Place the order

The user says "Great, I'll take the 8-inch for Saturday." The agent calls `place_order`:

```json
// Client → Server
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "place_order",
    "arguments": {
      "items": [{ "cake": "Vanilla Bean GF", "size": "8-inch" }],
      "pickup_date": "2026-03-21",
      "name": "Jamie Chen",
      "phone": "555-0123"
    }
  }
}
```

```json
// Server → Client
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Order confirmed! 8-inch Gluten-Free Vanilla Bean cake for Saturday March 21. Pickup anytime after 10am. Order #RB-4521."
      }
    ],
    "structuredContent": {
      "orderId": "RB-4521",
      "status": "confirmed",
      "total": 42,
      "pickupDate": "2026-03-21",
      "pickupTime": "after 10:00 AM",
      "verification": {
        "algorithm": "Ed25519",
        "keyId": "rosas-2025-06",
        "signature": "base64url-encoded-signature..."
      }
    }
  }
}
```

From question to confirmed order in four JSON-RPC calls. The user never left their AI assistant. Rosa's Bakery never built a custom integration for any specific AI platform.

### The same pattern at scale

Replace Rosa's Bakery with an enterprise SaaS company and the flow is identical:

| Step | Bakery | SaaS Company |
|------|--------|--------------|
| initialize | Connect to rosasbakery.com/mcp | Connect to acme.com/mcp |
| tools/list | ask_question, place_order | ask_question, schedule_demo, request_quote, qualify |
| ask_question | "Got gluten-free cakes?" | "Support Salesforce integration?" |
| Action | place_order | schedule_demo (after qualify) |

One protocol. Any business.


## 4. Verified responses

A critical problem with AI today: when it tells you "Acme is SOC2 certified," you have no idea if that's true or hallucinated. The Agentic Web solves this with **Ed25519 signatures**.

Every `structuredContent` response can carry a cryptographic signature. The verification flow:

1. **Extract** the `structuredContent` from the tool result.
2. **Canonicalize** using [JCS (RFC 8785)](https://www.rfc-editor.org/rfc/rfc8785) to get a deterministic byte string.
3. **Verify** the Ed25519 signature against the vendor's public key, published at `/.well-known/jwks.json` ([RFC 7517](https://www.rfc-editor.org/rfc/rfc7517)).

```json
// GET https://rosasbakery.com/.well-known/jwks.json
{
  "keys": [
    {
      "kty": "OKP",
      "crv": "Ed25519",
      "kid": "rosas-2025-06",
      "x": "base64url-encoded-public-key",
      "use": "sig"
    }
  ]
}
```

The agent can now tell the user with certainty: *"This pricing is verified directly by Rosa's Bakery, signed 2 minutes ago"* — not *"Based on what I found online..."*


## 5. Qualification — the conversational form

Some information shouldn't be free. Enterprise pricing, custom quotes, and security documentation often require context about the buyer. Traditional web forms are friction-heavy and context-destroying.

The Agentic Web replaces forms with the `qualify` tool — progressive, conversational data collection within the same MCP session.

```json
// Agent asks for enterprise pricing → server responds:
{
  "structuredContent": {
    "qualificationRequired": true,
    "reason": "Pricing details require company information",
    "requiredFields": ["company_name", "company_size", "use_case", "email"]
  }
}
```

The agent collects this naturally through conversation — "What company are you with? How big is your team?" — then submits via the `qualify` tool. The session transitions from `unqualified` → `qualifying` → `qualified`, unlocking gated tools like `schedule_demo` and `request_quote`.

The vendor controls all qualification logic. The AI can't skip it or bypass it. But unlike a web form, the buyer's full conversation context is preserved — when the demo happens, the sales rep knows exactly what was discussed.


## 6. Get started

The Agentic Web specification is live at [agentic-web.ai/specification.html](https://agentic-web.ai/specification.html). It covers transport, tools, qualification, verification, and authorization in detail — with copy-paste JSON-RPC 2.0 examples throughout.

The spec is open and protocol-level. Any MCP-compatible client — Claude, ChatGPT, Gemini, or a custom agent — connects without special adapters.

**What's next:**
- Reference implementation (coming soon)
- SDK quickstart guides
- Community contributions via [GitHub](https://github.com/salespeak-ai/agentic-web-website)

We're early. The protocols are emerging, the ecosystem is forming, and the best implementations haven't been built yet. If you're thinking about how AI agents should interact with businesses, we'd love to hear from you.

---

*The Agentic Web is powered by [Salespeak](https://salespeak.ai).*
