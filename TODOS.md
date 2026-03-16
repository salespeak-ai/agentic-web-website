# TODOs

## P1 — High Priority

### Diversify examples site-wide
- **What:** Update a2a-chat.html, a2a-sequence.html, and why.html to remove enterprise-only framing and add diverse business examples
- **Why:** The "any business" vision is only credible if examples span B2C, B2B, and small business across all pages — not just the spec
- **Effort:** M (2-3 hours)
- **Depends on:** spec + index + sequence updates shipping first

## P2 — Medium Priority

### Copy-to-clipboard on code blocks
- **What:** Add a small clipboard icon button to all `<pre>` code blocks on specification.html
- **Why:** Technical readers expect this on spec/docs pages. Reduces friction for developers implementing the protocol
- **Effort:** S (20 min JS)

### Extract shared CSS into styles.css
- **What:** Pull duplicated CSS (nav, footer, variables, typography, layout — ~650 lines per page) into a shared `styles.css` and link from all 7 HTML pages
- **Why:** Every page change currently requires updating CSS in 7 places. Maintenance burden grows with each new page
- **Effort:** M (2 hours, touches all 7 pages, risk of visual regressions)

### Quick Start callout + guide
- **What:** Add a highlighted callout at the top of the spec page: "Get started in 5 minutes" linking to a quickstart guide or reference implementation
- **Why:** The spec is comprehensive but intimidating. A quickstart signals approachability and lowers the barrier to first implementation
- **Effort:** M (depends on reference implementation existing)
- **Depends on:** Reference implementation / SDK

## P3 — Vision Items

### Persona toggle tabs on spec code examples
- **What:** Add toggleable tabs (SaaS / E-commerce / Local Business) on spec code blocks so readers can switch example context
- **Why:** The cathedral version of "any business" — proves universality interactively rather than through static examples
- **Effort:** L (4 hours)
- **Depends on:** Core spec rewrite shipped and stable

### Anchor links on h3 sub-headings
- **What:** Add hoverable `#` anchor links on all h3 headings in the spec for easy deep-linking and sharing
- **Why:** Standard developer docs pattern. Lets people share links to specific sub-sections
- **Effort:** S (15 min CSS + JS)
