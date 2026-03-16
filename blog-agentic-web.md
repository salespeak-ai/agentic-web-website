# Under the Hood: The Agentic Web

*By the Agentic Web team at [Salespeak](https://salespeak.ai)*

## 1. What is the Agentic Web?

Someone asks their AI assistant: "Can I order a gluten-free cake from Rosa's Bakery for Saturday?" The AI scrapes a website, guesses at the answer, and hopes for the best. It can't check availability, verify pricing, or place the order.

We're building a fix for this. The Agentic Web is an open spec for turning any business into an [MCP](https://modelcontextprotocol.io) server that AI agents connect to directly. Instead of scraping and guessing, the agent calls a real API, gets a signed answer, and takes action.

**Companies don't just have websites. They have AI-native endpoints that answer questions and do things.**

It's built on MCP (Anthropic's Model Context Protocol), the same protocol that Claude, ChatGPT, and other AI agents already speak. Any MCP client connects to your server out of the box. No custom integrations, no lock-in.

### The problem

AI-powered research is broken. AI scrapes SEO-biased content and presents it as truth. Users get hallucinated pricing, outdated feature lists, and "contact sales" dead ends. No way to verify anything. No way to act on it.

The Agentic Web flips this. Your business pushes first-party data to AI through a standard protocol:

```
TODAY:     AI Agent → Scrapes Website → Guesses Answer
AGENTIC:   AI Agent → Calls MCP Server → Verified Answer + Action
```

### Where it sits relative to UCP, MCP, and A2A

These protocols don't compete. They cover different ground.

**MCP** (Anthropic) is the transport layer. The Agentic Web is an MCP Server Profile: conventions for building your endpoint as a standard MCP server.

**UCP** (Google, Shopify, Walmart, Stripe) handles commerce: product discovery, cart, checkout, payments. If you sell products, UCP handles the transaction. The Agentic Web handles everything else: answering questions, qualifying buyers, scheduling demos, opening tickets.

**A2A** (Google) is for agent-to-agent communication. When a user's AI needs to negotiate with your company's AI, A2A handles that.

You might implement all three, or just one. They're independent.


## 2. Who benefits?

### Buyers

No more navigating websites, parsing marketing copy, or filling out the same form for the tenth time. Ask in plain language, get a verified answer, book a demo. All in one conversation. You control what you share.

### Businesses

You define what AI says about you. No hallucinated features, no wrong pricing. Every interaction collects structured qualification data (company size, use case, budget) that goes into your CRM. Leads route to the right person automatically.

### AI agents

Agents call a live API instead of guessing from stale training data. Responses are cryptographically signed, so the agent can say "They are SOC2 Type II certified, verified March 2026" with actual proof behind it. And agents don't just answer questions. They book the demo. Start the trial. Open the ticket.


## 3. How it works

We'll walk through a real interaction. Rosa's Bakery has an Agentic Web MCP server. A customer asks their AI about ordering a cake.

### Step 1: Connect

The agent connects and does the standard MCP handshake:

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

The server returns an `Mcp-Session-Id` header. The client includes it on every request after this.

### Step 2: Discover tools

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

The same `tools/list` pattern works for any business. A SaaS company would return `ask_question`, `schedule_demo`, `request_quote`, `open_ticket`. A bakery returns `ask_question`, `place_order`. Same protocol. Different tools.

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

Two things in the response. `content` is the human-readable text the AI shows. `structuredContent` is machine-readable data with pricing, availability, and a cryptographic signature proving this came from Rosa's Bakery, not from a hallucination.

### Step 4: Place the order

The user says "Great, I'll take the 8-inch for Saturday."

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

Four JSON-RPC calls. Question to confirmed order. The user never left their AI assistant. Rosa's Bakery never built a custom integration for any AI platform.

### Same pattern, different business

| Step | Bakery | SaaS Company |
|------|--------|--------------|
| initialize | rosasbakery.com/mcp | acme.com/mcp |
| tools/list | ask_question, place_order | ask_question, schedule_demo, request_quote, qualify |
| ask_question | "Got gluten-free cakes?" | "Support Salesforce integration?" |
| Action | place_order | schedule_demo (after qualify) |

One protocol. Any business.


## 4. Verified responses

When AI tells you "Acme is SOC2 certified," you have no idea if that's real or hallucinated. The Agentic Web fixes this with Ed25519 signatures.

Every `structuredContent` response can carry a cryptographic signature. Verification is three steps:

1. Extract `structuredContent` from the tool result.
2. Canonicalize it using [JCS (RFC 8785)](https://www.rfc-editor.org/rfc/rfc8785) for a deterministic byte string.
3. Verify the Ed25519 signature against the vendor's public key at `/.well-known/jwks.json` ([RFC 7517](https://www.rfc-editor.org/rfc/rfc7517)).

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

Now the agent can say: *"This pricing is verified by Rosa's Bakery, signed 2 minutes ago."* Not *"Based on what I found online..."*


## 5. Qualification: the conversational form

Some information shouldn't be free. Custom quotes and security docs need context about the buyer. Web forms are friction machines that destroy context.

The `qualify` tool replaces them. It's progressive, conversational data collection within a single MCP session.

```json
// Agent asks for pricing → server responds:
{
  "structuredContent": {
    "qualificationRequired": true,
    "reason": "Pricing details require company information",
    "requiredFields": ["company_name", "company_size", "use_case", "email"]
  }
}
```

The agent collects this through conversation: "What company are you with? How big is your team?" Then submits via `qualify`. The session transitions from `unqualified` → `qualifying` → `qualified`, unlocking gated tools like `schedule_demo` and `request_quote`.

The vendor controls the logic. The AI can't skip or bypass qualification. But unlike a form, the buyer's full conversation context travels with them. When the demo happens, the sales rep already knows what was discussed.


## 6. Get started

The spec is at [agentic-web.ai/specification.html](https://agentic-web.ai/specification.html). Transport, tools, qualification, verification, authorization. Copy-paste JSON-RPC 2.0 examples throughout.

Any MCP client connects without adapters. Claude, ChatGPT, Gemini, or your own agent.

We're working on a reference implementation and SDK quickstart guides. The code will live on [GitHub](https://github.com/salespeak-ai/agentic-web-website).

We don't have 20 launch partners or a finished SDK. What we have is a spec that works, a protocol that's real, and a bet that every business will need an AI-native endpoint within the next two years. If you're thinking about the same problem, [come build with us](https://github.com/salespeak-ai/agentic-web-website).

---

*The Agentic Web is powered by [Salespeak](https://salespeak.ai).*
