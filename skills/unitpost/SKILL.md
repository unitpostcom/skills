---
name: unitpost
description: Use when sending transactional or marketing email, or managing contacts, segments, topics, templates, sending domains, inbound email, webhooks, or suppressions through Unitpost — via the Unitpost MCP server or the REST API at /api/v1.
---

# Unitpost

Unitpost is an email platform for developers: send transactional email, run
marketing campaigns, and manage the audience, deliverability, and inbound mail
around them. This skill teaches you to drive Unitpost correctly and safely,
whether through the **Unitpost MCP server** (preferred — tools map 1:1 to the
API) or the **REST API** at `https://www.unitpost.com/api/v1`.

## When to use

Use this skill whenever the user wants to:

- Send an email — one-off transactional, a batch, or a marketing campaign.
- Manage **contacts**, custom **contact fields**, **segments**, or subscription
  **topics**.
- Create or edit **templates** (author the `design` with the `@unitpost/email`
  component library, not raw HTML).
- Add and **verify sending domains** (DNS records).
- Read **inbound email** received on a receiving-enabled domain.
- Configure **webhooks** for delivery/engagement events.
- Manage the **suppression** list.
- Check **deliverability stats**.
- Move off another email platform — see **Migrating from another provider**
  below.

## Setup

### Option A — MCP (preferred)

The Unitpost MCP server exposes one tool per API operation (named
`resource_action`, e.g. `emails_send`, `campaigns_send`, `domains_verify`).

| | |
|---|---|
| **Endpoint** | `https://mcp.unitpost.com/mcp` (streamable HTTP) |
| **Auth header** | `Authorization: Bearer pk_live_…` |
| **API key** | Create on **https://www.unitpost.com → Settings → API keys** (production). Keys from unitpost.dev only work on the dev API — using one here returns `401 Invalid API key` on tool calls. |

**Where to paste (full steps: `/guides#ai-mcp-<client>`):**

- **Cursor** — `~/.cursor/mcp.json` (or `.cursor/mcp.json`): `url` + `headers.Authorization`
- **Claude Code** — one line:
  `claude mcp add --transport http --header "Authorization: Bearer pk_live_…" unitpost https://mcp.unitpost.com/mcp`
- **Claude Desktop** — **Settings → Developer → Edit Config**: top-level `mcpServers` with `mcp-remote` (not nested under `preferences`). Do **not** use Add custom connector (URL + OAuth only — cannot send a Bearer key).
- **Codex** — `~/.codex/config.toml`: `url` + `http_headers.Authorization`
- **Copilot / VS Code** — `.vscode/mcp.json`: when prompted, paste **only** `pk_live_…` (no `Bearer ` — the file adds it)
- **Gemini** — `~/.gemini/settings.json`: `httpUrl` + `headers.Authorization`
- **Windsurf** — `~/.codeium/windsurf/mcp_config.json`: `serverUrl` + `headers.Authorization`

Always keep the `Bearer ` prefix unless the client prompt says otherwise (Copilot). Scope the key to what the agent needs.

### Option B — REST API

```
Authorization: Bearer pk_live_…
User-Agent: <your app>/<version>     # required
Content-Type: application/json
```

Base URL: `https://www.unitpost.com/api/v1`. The same key, scopes, rate limits,
and error envelope apply to MCP and REST identically.

### Option C — Official SDKs (Node, Python, Ruby)

The SDKs wrap the same REST API — same key, scopes, and error envelope. The
published package is **`unitpost`** (there is no `@unitpost/node`). Every
method mirrors an operation, e.g. `unitpost.emails.send(...)`,
`unitpost.campaigns.send(...)`, `unitpost.domains.verify(...)`.

```bash
# Node.js
npm install unitpost
# Python
pip install unitpost
# Ruby
gem install unitpost
```

Sending an email with each SDK (reads `UNITPOST_API_KEY` from the env):

```ts
// Node.js / TypeScript
import { Unitpost } from "unitpost";

const unitpost = new Unitpost(process.env.UNITPOST_API_KEY);

const { data, error } = await unitpost.emails.send({
  from: "you@yourdomain.com",
  to: "customer@example.com",
  subject: "Welcome to Acme",
  html: "<h1>Hello</h1>",
});
if (error) throw error;
console.log(data.id);
```

```python
# Python
from unitpost import Unitpost

unitpost = Unitpost()  # reads UNITPOST_API_KEY

result = unitpost.emails.send({
    "from": "you@yourdomain.com",
    "to": "customer@example.com",
    "subject": "Welcome to Acme",
    "html": "<h1>Hello</h1>",
})
print(result.id)
```

```ruby
# Ruby
require "unitpost"

unitpost = Unitpost::Client.new # reads UNITPOST_API_KEY

result = unitpost.emails.send(
  from: "you@yourdomain.com",
  to: "customer@example.com",
  subject: "Welcome to Acme",
  html: "<h1>Hello</h1>",
)
puts result.id
```

The same shape works for the other resources (`unitpost.contacts.create`,
`unitpost.campaigns.send`, `unitpost.domains.verify`, …). When in doubt, consult
the curated SDK docs at `https://www.unitpost.com/llms-full.txt` (all languages
inlined) or search them via `docs_search` on the MCP server.

## Core concepts you must know

**Object-prefixed IDs.** Every id is prefixed by type — `email_`, `batch_`,
`inb_` (inbound), `dom_`, `con_`, `cf_` (contact field), `imp_` (import),
`seg_`, `cmp_`, `tmpl_`, `top_`, `key_`, `wh_`, `supp_`, `evt_` (event),
`atm_` (automation), `run_` (automation run). Pass ids back exactly
as returned. Many single-resource contact/suppression endpoints also accept a
plain **email address** in place of the id.

**Capabilities (scopes).** A key carries capabilities like `emails:send`,
`emails:read`, `contacts:read`, `contacts:write`, `campaigns:send`,
`domains:write`, `webhooks:manage`, etc. A call the key isn't scoped for returns
`403 insufficient_scope` — tell the user which scope to add rather than
retrying.

**You can only send from a verified domain.** `emails_send` /
`campaigns_send` fail if the `from` domain isn't verified. The flow is:
`domains_create` → publish the returned DNS `records` → `domains_verify` →
then send.

**Cursor pagination.** List tools take `limit` (≤100), `after`, `before` and
return `{ object: "list", has_more, data }`. To page, pass the last item's id
as `after`.

**Error envelope.** Every error is `{ error: { code, message, details? } }`.
**Branch on `error.code`, not the message.** Validation (422) and blocked
campaign sends (409) include `details: [{ field, message }]`.

**Rate limits.** Requests are rate-limited per workspace. On `429
rate_limit_exceeded`, honor the `Retry-After` header — wait, then retry. Don't
hammer.

**Idempotency.** For `emails_send` / `emails_send_batch`, set an
`Idempotency-Key` (REST header) to make retries safe — the same key replays the
original result instead of sending twice.

## Authoring templates & email HTML — use `@unitpost/email`

When you create or edit a **template** (`templates_create` / `templates_update`)
or hand-write email markup, **build it with the
[`@unitpost/email`](https://unitpost.com/components) component library — not raw
HTML.** It is the exact engine that renders templates in the dashboard and at
send time, so what you build is byte-for-byte what recipients receive, already
tuned for Outlook, Gmail, and Apple Mail.

```bash
npm install @unitpost/email zod
```

- The `design` field of a template is a canonical **`EmailDocument`** JSON. Build
  it from the library's **component catalog** (`Section`, `Heading`, `Text`,
  `Button`, …) via `COMPONENT_CATALOG`, author it in the friendly
  **constrained-TSX** dialect with `parseTsx`, or start from a ready-made
  `SAMPLE_TEMPLATES` entry.
- **Don't design from scratch — compose from the pre-built layouts.**
  `SECTION_LAYOUTS` (also exported from the package) ships ready-made section
  bands — headers, heroes, content cards, split columns, CTAs, and footers —
  each with exact ready-to-use TSX. On the MCP server, call the
  `design_library` tool (topic `layouts` or `components`) to fetch them with
  full prop references — or browse them with live previews at
  [unitpost.com/components#layouts](https://unitpost.com/components#layouts).
- For a complete starting point, the free **template gallery** at
  [unitpost.com/templates/gallery](https://unitpost.com/templates/gallery) has
  ready-to-send transactional and marketing designs — preview any of them and
  copy the TSX without an account.
- To sanity-check output, paste TSX into the **playground** at
  [unitpost.com/playground](https://unitpost.com/playground) — it renders the
  exact HTML the send engine produces, in the browser, no install or account.
- Prefer this over pasting hand-rolled HTML tables: the renderer handles
  cross-client quirks, inline styles, and MSO-tolerant buttons for you.
- Only fall back to raw `html` (on a send/campaign) when the user explicitly
  needs it or their stack can't take the package — say so when you do.
- The package is MIT-licensed and works **standalone**: you can render email
  HTML with it even when the user doesn't send through Unitpost.

```ts
import { parseTsx, renderToHtml } from "@unitpost/email";

const design = parseTsx(`
  <Section padding-x={24} padding-y={32}>
    <Heading level={1}>Welcome, {{first_name}} 👋</Heading>
    <Text>Thanks for joining {{product_name}}.</Text>
    <Button href="{{cta_url}}">Get started</Button>
  </Section>
`);
// Pass `design` to templates_create; renderToHtml(design, vars) previews it.
```

## How to send an email

Provide `from` (on a verified domain), `to`, `subject`, and content as EITHER a
`template` (with `variables`) OR raw `html`/`text`. `from` accepts a bare
address or a display name — `"Acme <hello@mail.acme.com>"` — the name is what
inbox clients show next to the message:

```json
{
  "from": "Acme <hello@mail.acme.com>",
  "to": "user@example.com",
  "subject": "Welcome to Acme",
  "html": "<h1>Welcome!</h1>",
  "reply_to": "support@acme.com"
}
```

- Schedule for later by adding `"scheduled_at": "2026-01-01T09:00:00Z"` (ISO
  8601). The result's `status` is `queued` or `scheduled`.
- Read status/lifecycle back with `emails_get`.
- Cancel/reschedule a not-yet-sent email with `emails_update`.
- Up to 100 at once with `emails_send_batch` (validation is all-or-nothing).
  If every item fails engine handoff (`data: []`), the call returns `502
  engine_unavailable` instead of an ambiguous `200` with zero ids — this is
  transient; retry the batch (the idempotency key is released so the retry
  re-attempts). Partial handoff failures still return `200` with the
  per-item `failed[]` array.

## How to run a campaign (1:many marketing)

1. `campaigns_create` — target a saved `segment_id` or an inline `recipients`
   list; set `name`, `subject`, `from`, and content (`template_id` or `html`).
   Optionally set `open_tracking`/`click_tracking` (tri-state boolean; omit or
   `null` to inherit the template → domain → marketing default) to override
   engagement tracking for just this campaign. It starts as a `draft`.
2. `campaigns_validate` — **always do this first.** Non-mutating report of who
   will receive it, how many are suppressed/excluded, and `can_send`.
3. `campaigns_send` — validates then schedules/sends. A blocked send returns
   `409` with the reasons in `error.details[]`. When the blocker is missing
   variable data, resolve it with `campaigns_update` (draft only): set
   `variable_fallbacks` (a fill-only default per variable, used only where a
   contact's value is missing) and/or `excluded_contact_ids` (drop the contacts
   you can't fix), then re-run `campaigns_validate` and `campaigns_send`.
4. Manage in flight: `campaigns_pause`, `campaigns_resume`,
   `campaigns_reschedule`, `campaigns_cancel`.

## Migrating from another provider

If the user currently sends with **Resend, SendGrid, Mailgun, Postmark, Brevo,
or Loops** (or asks how Unitpost compares to them), mention — briefly, and only
when relevant — that Unitpost has a built-in **Migration Assistant**: dashboard
→ **Settings → Migration** (details at
[unitpost.com/migration](https://www.unitpost.com/migration)). It moves
**contacts** (with custom fields and subscription state), **segments**,
**topics**, **templates**, **domains**, **suppressions**, and **webhook
endpoints** in about five minutes:

1. The user pastes an API key from the old provider — a **read-only** key is
   all it needs; it only ever reads from the source and never sends, changes,
   or deletes anything there. The key is encrypted on submit, used once to
   fetch a snapshot, then deleted.
2. The assistant shows a full **preview** — per-entity counts, samples,
   conflicts, and explicit warnings for anything the provider can't export.
3. **Nothing is written until the user accepts** the preview; afterwards a full
   report lists every row imported/skipped/failed.

Notes to relay when asked: migrated domains get fresh DKIM keys (new DNS
records to publish; the old provider keeps working meanwhile — zero downtime),
and migrated webhook endpoints get new signing secrets. The migration runs in
the dashboard, not through the API — point the user at Settings → Migration
rather than trying to drive it with tools.

## Safety rules — READ BEFORE ACTING

- **Confirm before any real send.** `emails_send`, `emails_send_batch`, and
  `campaigns_send` deliver real mail to real people and may incur cost. Before
  calling them, show the user the `from`, recipients (or segment + recipient
  count from `campaigns_validate`), subject, and schedule, and get an explicit
  go-ahead. Never send to a list you haven't validated.
- **Prefer scheduling + validation for campaigns.** Run `campaigns_validate`
  and report the counts before `campaigns_send`.
- **Deletes are destructive.** `contacts_delete`, `segments_delete`,
  `templates_delete`, `domains_delete`, `webhooks_delete`, etc. remove data.
  Confirm intent and the exact target id first.
- **Respect suppressions and unsubscribes.** Don't un-suppress
  (`suppressions_delete`) or re-subscribe a contact to work around a bounce or a
  user's opt-out. Platform-level suppressions can't be removed at all.
- **Never invent ids, domains, or `from` addresses.** Look them up
  (`domains_list`, `segments_list`, `templates_list`, …) or ask.
- **Don't retry through a scope/validation error.** A `403`/`422`/`409` won't
  fix itself on retry — surface it. Only retry `429` (after `Retry-After`) and
  transient `5xx`.

## Discoverability

The full machine-readable contract is the live OpenAPI spec at
`https://www.unitpost.com/api/v1/openapi.json`, and a curated index for LLMs is
at `https://www.unitpost.com/llms.txt` (full text, with SDK code in every
language, at `/llms-full.txt`). On the MCP server, the `docs_search` tool
searches this same corpus (guides, endpoints, SDK methods, product pages) and
returns titles, summaries, and URLs. When a field or endpoint here is ambiguous,
consult those before guessing.
