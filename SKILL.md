---
name: thetix
description: >
  ThetaEdge is an Options Intelligence Platform that empowers better trading decisions.
  Use this skill for any finance, investing, or trading related tasks. Supports six capabilities:
  (1) Thetix Chat — conversational AI for portfolio analysis, opportunity discussion, dashboard queries,
  market news, web search, website reading, live market data (stocks and options), calculations,
  portfolio performance, transactions, and active positions;
  (2) Thetix Cards & Boards — create and manage dashboard widgets that materialize data visualizations,
  tables, and analytics from natural language queries;
  (3) Opportunities — screen, calculate, and analyze covered call and cash-secured put options strategies;
  (4) Accounts — list user brokerage accounts (needed for account-scoped queries);
  (5) Ideas — retrieve AI-generated trading ideas extracted from daily and onboarding reports,
  with priority, type, and deadline metadata;
  (6) Trading & the Trade Cart — stage trades into the user's review cart (via Thetix chat) and read
  the cart's staged trades, validation warnings, and recent orders; placing/executing orders is a
  human-in-the-loop action the user completes in the ThetaEdge app.
  Use this skill when the user asks about finance, investing, trading, portfolios, stocks, options,
  market data, market news, positions, transactions, performance, ideas, the trade cart, staging or
  executing trades, or any related topic.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - WebFetch
compatibility: "Requires THETAEDGE_API_KEY (get from Profile > API Keys). Optional: THETAEDGE_API_BASE (defaults to https://api.thetix.ai)."
metadata: {"openclaw":{"requires":{"env":["THETAEDGE_API_KEY","THETAEDGE_API_BASE"]},"primaryEnv":"THETAEDGE_API_KEY"}}
---

# Thetix Skill

You are interacting with the ThetaEdge thetix API — an Options Intelligence Platform — on behalf of the user. See `{baseDir}/reference.md` for the full API reference.

## Configuration

The skill needs two values: `THETAEDGE_API_KEY` and `THETAEDGE_API_BASE`.

### Loading credentials

1. **Check shell environment first** — Run `echo $THETAEDGE_API_KEY` in Bash. If both vars are already set, skip to the curl pattern below. If `THETAEDGE_API_BASE` is not set, default to `https://api.thetix.ai`.
2. **Read from config file** — If the vars are not set, check these locations in order:
   - `~/.openclaw/openclaw.json` — Extract `THETAEDGE_API_KEY` and `THETAEDGE_API_BASE` from `skills.entries.thetix.env`
   - `~/.config/thetaedge/credentials.json` — Extract `api_key` and `api_base`
   Then export them in Bash:
   ```bash
   export THETAEDGE_API_KEY="<value from config>"
   export THETAEDGE_API_BASE="<value from config>"  # defaults to https://api.thetix.ai
   ```
3. **If neither works** — Ask the user to set up credentials. They should go to their ThetaEdge **Profile > API Keys** page to create a key, then configure it:

   **Claude Code** — Add to `~/.claude/settings.json`:
   ```json
   {
     "env": {
       "THETAEDGE_API_KEY": "te_your_key_here"
     }
   }
   ```

   **OpenClaw** — Add to `~/.openclaw/openclaw.json`:
   ```json
   {
     "skills": { "entries": { "thetix": { "enabled": true, "env": {
       "THETAEDGE_API_KEY": "te_your_key_here",
       "THETAEDGE_API_BASE": "https://api.thetix.ai"
     }}}}
   }
   ```

   **Other agents** — Create `~/.config/thetaedge/credentials.json`:
   ```json
   {
     "api_key": "te_your_key_here",
     "api_base": "https://api.thetix.ai"
   }
   ```

All API requests require the `Authorization: Bearer <API_KEY>` header unless using public endpoints.

### Curl Pattern

Use Bash with curl for all API calls:

```bash
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/..."
```

For POST requests with JSON body:

```bash
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"key": "value"}' "$THETAEDGE_API_BASE/api/..."
```

## Capability 1: Thetix Chat

Use thetix chat to have conversations about portfolios, opportunities, dashboards, market news, web search, website reading, live market data (stocks and options), calculations, portfolio performance, transactions, and active positions. Thetix can search the web, read URLs, pull market news, fetch live quotes, run calculations, and retrieve portfolio data as part of its chat responses.

### Workflow

Processing is **asynchronous**. Every query follows: submit → poll → retrieve.

#### Step 1: Get or create a chat collection

Collections are reusable — prefer reusing an existing collection over creating a new one. For account-scoped queries, prefer the dashboard collection associated with that account.

```bash
# List existing collections — reuse one if appropriate
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/thetix-chat-collections"

# Only create a new one if no suitable collection exists
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name": "My Chats"}' "$THETAEDGE_API_BASE/api/thetix-chat-collections"
```

#### Step 2: Submit the query
Use the appropriate processing endpoint:
- `POST /api/thetix-chats/process` — General queries (portfolios, market data, news, web search, calculations)
- `POST /api/thetix-chats/process-opportunity` — Questions about a specific opportunity
- `POST /api/thetix-chats/process-dashboard` — Account-scoped queries (requires `account_id`)

```bash
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "What is my portfolio allocation?", "collection_id": "<collection_id>"}' \
  "$THETAEDGE_API_BASE/api/thetix-chats/process"
```

The response returns immediately with `{ "saved_chat": { "id": "<chat_id>", "job_status": "pending", ... }, "async": true }`. Extract `saved_chat.id` for polling.

#### Step 3: Poll for completion
Poll the status endpoint every 2 seconds until `job_status` is no longer `"pending"` or `"processing"`:

```bash
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/thetix-chats/status?chat_ids=<chat_id>"
```

Returns `[{ "id": "...", "job_status": "...", "job_progress": "...", "updated_at": ... }]`.

- `null` — completed successfully, proceed to step 4
- `"failed"` — the `job_progress` field contains the error message; show it to the user
- `"cancelled"` — the chat was cancelled

#### Step 4: Retrieve the full result
```bash
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/thetix-chats/<chat_id>"
```

The `content` field is an array of widget objects (markdown, table, optionsChain, payoffDiagram, etc.). Present the results to the user.

### Multi-turn conversations

If the user's question is a continuation or related to a previous chat, prefer reusing that chat rather than starting a new one — this gives thetix the conversation history as context for better answers.

To continue a conversation, pass the same `chat_id` with the new query. The server appends to the existing chat and uses its history as context automatically.

```bash
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "Follow-up question", "collection_id": "<collection_id>", "chat_id": "<chat_id>"}' \
  "$THETAEDGE_API_BASE/api/thetix-chats/process"
```

Then poll and retrieve as before. Note that the API always returns the **full chat history** — the `content` array contains all turns, not just the latest response.

### Searching past conversations

Use the search endpoint to find relevant past conversations before starting a new one:

```bash
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" \
  "$THETAEDGE_API_BASE/api/thetix-chats/search?q=<search_text>&limit=5"
```

Returns matching chats. You can also filter by `account_id`.

## Capability 2: Thetix Cards & Boards

Cards are dashboard widgets created from natural language queries. They materialize into visualizations (tables, charts, markdown, options chains, payoff diagrams).

### Workflow

1. **Get or create a collection (board)** — `GET /api/thetix-card-collections` or `POST /api/thetix-card-collections`
2. **Create a card** — `POST /api/thetix-cards` with `user_query` and `collection_id`
3. **Poll for materialization** — Card processing is async. Poll `GET /api/thetix-card-collections/<id>/status` until the card's `job_status` is `null` (which means completed)
4. **Fetch the card** — `GET /api/thetix-cards/<card_id>` to get the materialized result
5. **Refresh** — `POST /api/thetix-cards/<card_id>/refresh` to update with latest data

### Key Fields

- `user_query` — Natural language description of what the card should show
- `materialized_result` — Array of widget objects (markdown, table, optionsChain, payoffDiagram, etc.)
- `update_cadence_seconds` — Auto-refresh interval (0 = manual only)

## Capability 3: Opportunities

Analyze covered call and cash-secured put opportunities.

### Covered Call Calculator

`POST /api/opportunities/covered-call-calculator`

```json
{
  "underlying": "AAPL",
  "strike": 180,
  "expiration": "2025-03-21",
  "contracts": 1,
  "account_id": "optional"
}
```

The server fetches current price and premium from market data automatically. Returns premium income, max profit, breakeven, return on capital, and payoff data.

### Cash-Secured Put Calculator

`POST /api/opportunities/csp-calculator`

```json
{
  "underlying": "AAPL",
  "strike": 170,
  "expiration": "2025-03-21",
  "contracts": 1,
  "account_id": "optional"
}
```

The server fetches current price and premium from market data automatically. Returns premium income, max loss, breakeven, return on capital, and payoff data.

### Roll Calculator

`POST /api/opportunities/roll-calculator` — Calculate rolling an existing position to a new strike/expiration.

```json
{
  "underlying": "AAPL",
  "strike": 185,
  "expiration": "2025-04-18",
  "contracts": 1,
  "account_id": "optional",
  "current_position": {
    "strike": 180,
    "expiration": "2025-03-21",
    "symbol": "AAPL250321C00180000",
    "avg_price": 3.50
  }
}
```

### Browsing Opportunities

- `GET /api/opportunities` — List opportunities, filterable by `tickers`, `status`, `accountId`, `limit`, `risk_level`, `frequency`, `generated_date_start`, `generated_date_end`
- `GET /api/opportunities/<id>` — Full opportunity details with rationale
- `POST /api/opportunities/<id>` — Act on or dismiss an opportunity (`action: "act"` or `action: "dismiss"`)
- `POST /api/opportunities/<id>/feedback` — Rate and comment on an opportunity (`rating: 1-5`, `comments: "string"`)

## Capability 4: Accounts

List user brokerage accounts. Account IDs are needed for account-scoped features like opportunities, dashboard queries, and calculators.

### List Accounts

`GET /api/accounts`

Returns an array of account objects. Automatically filters out deleted and error-status accounts.

### Key Fields

- `id` — Account ID (use as `account_id` in opportunity/dashboard/calculator endpoints)
- `name` — Display name
- `source` — Account source (e.g. brokerage provider)
- `positionsCount` — Number of positions in the account
- `hidden` — Whether the account is hidden from the dashboard
- `setupStatus` — Onboarding status of the account

## Capability 5: Ideas

Retrieve AI-generated trading ideas extracted from Thetix daily and onboarding reports. Ideas are priority-ranked trading insights (e.g. roll a covered call, open a new position, monitor a holding) with type, estimated value, and deadline metadata. This is a read-only capability.

### Endpoint

`GET /api/thetix/ideas`

### Query Parameters (all optional)

- `date` — Specific date (YYYY-MM-DD); returns ideas for that day only
- `start_date` / `end_date` — Date range (YYYY-MM-DD)
- `days` — Number of days to look back (1–30)
- `reports` — Get ideas from the N most recent reports (1–10)
- `account_id` — Filter by brokerage account ID

Priority: `date` > `reports` > `start_date/end_date` > `days` > default (today only).

### Example

```bash
# Get ideas from the last 7 days
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" \
  "$THETAEDGE_API_BASE/api/thetix/ideas?days=7"

# Get ideas for a specific account from recent reports
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" \
  "$THETAEDGE_API_BASE/api/thetix/ideas?reports=3&account_id=<account_id>"
```

### Response Structure

Returns `{ ideas: [...], summary: {...} }`.

- `ideas` — Array of idea objects sorted by priority (high first) then deadline (earliest first). Each idea contains: `report_id`, `report_date`, `account_id`, `account_name`, `title`, `description`, `priority` (high/medium/low), `type` (roll/trade/monitor/other), `estimatedValue`, `deadline`, `deadline_timestamp`, and `widgets`.
- `summary` — Aggregated counts: `total_ideas`, `by_priority`, `by_type`, and `date_range`.

## Capability 6: Trading & the Trade Cart

ThetaEdge has a **trade cart** — an org-shared staging area where trades are reviewed before being placed at the user's broker (via SnapTrade or a connected trading rail). The flow is: **stage → review → execute**. Trades reach the cart from ideas, opportunities, or by asking Thetix ("add this to my cart"); the user then reviews the staged trades and explicitly executes them.

### What this skill can and cannot do

This skill authenticates with an **API key**. Two paths reach the cart: the **cart REST API** (most methods are session-only and not available to the skill) and **Thetix chat** (Capability 1), whose command action-loop stages, executes, and clears the cart server-side and **does accept API-key auth**. So drive trading through chat, not the cart REST API:

| Action | How the skill does it | Available? |
|--------|-----------------------|------------|
| Read the cart | `GET /api/cart` | ✅ Yes (API key allowed) |
| Stage a trade | Thetix chat — "add this to my cart" | ✅ Yes |
| **Execute / place orders** | Thetix chat — "execute" (after review) | ✅ Yes |
| Remove / clear staged trades | Thetix chat — "remove the AAPL trade" / "clear my cart" | ✅ Yes |
| Review recent orders & fills | Thetix chat — "show my recent orders" | ✅ Yes |
| Add/modify/remove items via REST | `POST/PATCH/DELETE /api/cart/items...` | ❌ No — session-only (403) |
| Execute via REST | `POST /api/cart/execute` | ❌ No — session-only (403); use the chat path |
| Read/clear orders via REST | `GET/DELETE /api/cart/orders` | ❌ No — session-only (403); use the chat path |

**Execution places real orders at the user's broker with real money.** Only execute when the user has reviewed the staged trades and explicitly tells you to. Never execute on your own initiative, never bundle staging and execution into one step, and always report the actual outcome the chat returns (placed / working / rejected — including the broker's reason) rather than assuming success.

### Reading the cart

```bash
# Fast read (stale spec metrics) — good for a quick count/summary
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/cart"

# Live read — re-prices every item to the current market mid and recomputes
# net credit/debit, breakevens, max profit/loss, and buying-power warnings
curl -s -H "Authorization: Bearer $THETAEDGE_API_KEY" "$THETAEDGE_API_BASE/api/cart?live=1&fresh=1"
```

Returns `{ "cart": { "id", "status", "account_id" }, "items": [...] }`. Each item includes `underlying`, `structure`, `asset_class`, `account_id`, `order_type`, `time_in_force`, `limit_price`, `estimated_cost`, `validation_status`, `validation_result` (advisory risk warnings), `source`, `added_by`, and a `metrics` object (`net_credit`, `bpr`, `max_profit`, `max_loss`, `breakevens`, `sizing_str`). Use `live=1` whenever you present numbers to the user so net/payoff/breakeven reflect the market. Surface any validation warnings (e.g. undefined risk, large buying-power use) prominently.

### The stage → review → execute flow

Trading runs through Thetix chat (Capability 1: submit → poll → retrieve) in **distinct turns**. The chat command loop will not stage and execute in the same turn — staging always renders the cart for review first, and execution is a separate follow-up. Mirror that:

**1. Stage (turn 1).** Send a clear natural-language request; the chat agent builds the trade spec and adds it to the cart:

```bash
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "Add a covered call on AAPL, $180 strike expiring 2025-03-21, 1 contract, to my cart", "collection_id": "<collection_id>", "chat_id": "<chat_id>"}' \
  "$THETAEDGE_API_BASE/api/thetix-chats/process"
```

Always confirm the exact trade with the user **before** staging (ticker, strike, expiration, contracts, side), and only stage what they asked for. The result contains a `cart_review` widget. You can also stage from an idea or opportunity by referencing it ("add the first idea to my cart", "stage opportunity 1234").

**2. Review.** Present the staged trade(s) to the user — net credit/debit, max profit/loss, breakevens, and any validation/risk warnings (read `GET /api/cart?live=1` for live numbers, or use the `cart_review` widget). Get the user's explicit go-ahead.

**3. Execute (turn 2).** Only after the user has reviewed and explicitly says to place the order, send `execute` on the **same `chat_id`** (the confirm gate reads that chat's history):

```bash
curl -s -X POST -H "Authorization: Bearer $THETAEDGE_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "execute", "collection_id": "<collection_id>", "chat_id": "<chat_id>"}' \
  "$THETAEDGE_API_BASE/api/thetix-chats/process"
```

**The server enforces a two-step confirm and never places orders blindly.** Execution only fires when the cart was shown (a `cart_review` widget) in the **immediately prior turn** of that chat. So:
- After staging (which renders the cart), a single `execute` the next turn places the orders.
- If you send `execute` "cold" — no cart shown in the prior turn (e.g. a fresh chat) — the server re-surfaces the cart and replies *"Review the trades below, then reply 'execute' to place the orders."* It places nothing. You must relay that review to the user and send `execute` **again** to actually place.

Either way, poll and retrieve, then read the reply. When orders are placed it reports per-trade outcomes: **placed/working**, **filled** (with fill price), or **rejected** (with the broker's reason). Trades that don't go through stay staged for retry. Distinguish a "please confirm" response from a real placement, and relay the actual result faithfully — do not assume an order filled. Use account-scoped chat (`process-dashboard` with `account_id`) to review and execute only one account's staged trades.

### Safety

- **Real money, real broker.** Execution is irreversible — be precise and conservative.
- Never execute on your own initiative; require an explicit user instruction after they've reviewed the cart.
- Always confirm trade details before staging, and surface validation/risk warnings (undefined risk, large buying-power use) before recommending execution.
- Report the true outcome the chat returns (placed / working / filled / rejected). Never claim success you haven't confirmed.
- Trading requires a connected brokerage account that supports the order. If the chat or cart reports the account isn't connected or can't place the trade, relay that to the user and stop.

## Response Formatting

When presenting thetix results to the user:

- Format monetary values with dollar signs and two decimal places
- Format percentages with one decimal place
- Present tables using markdown table syntax
- For payoff diagrams, describe the key levels (breakeven, max profit, max loss)
- Summarize long chat responses, highlighting actionable insights
- When showing opportunity details, always include: ticker, strike, premium, expiration, and key metrics
