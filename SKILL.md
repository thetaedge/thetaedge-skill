---
name: thetix
description: >
  ThetaEdge is an Options Intelligence Platform that empowers better trading decisions.
  Use this skill for any finance, investing, or trading related tasks. Supports five capabilities:
  (1) Thetix Chat — conversational AI for portfolio analysis, opportunity discussion, dashboard queries,
  market news, web search, website reading, live market data (stocks and options), calculations,
  portfolio performance, transactions, and active positions;
  (2) Thetix Cards & Boards — create and manage dashboard widgets that materialize data visualizations,
  tables, and analytics from natural language queries;
  (3) Opportunities — screen, calculate, and analyze covered call and cash-secured put options strategies;
  (4) Accounts — list and create user brokerage accounts (needed for account-scoped queries);
  (5) Ideas — retrieve AI-generated trading ideas extracted from daily and onboarding reports,
  with priority, type, and deadline metadata.
  Use this skill when the user asks about finance, investing, trading, portfolios, stocks, options,
  market data, market news, positions, transactions, performance, ideas, or any related topic.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - WebFetch
metadata: {}
---

# Thetix Skill

You are interacting with the ThetaEdge thetix API — an Options Intelligence Platform — on behalf of the user. See `{baseDir}/reference.md` for the full API reference.

## Configuration

The skill needs two values: `THETAEDGE_API_KEY` and `THETAEDGE_API_BASE`.

### Loading credentials

1. **Check shell environment first** — Run `echo $THETAEDGE_API_KEY` in Bash. If both vars are already set, skip to the curl pattern below. If `THETAEDGE_API_BASE` is not set, default to `https://api.thetaedge.ai`.
2. **Read from config file** — If the vars are not set, check these locations in order:
   - `~/.openclaw/openclaw.json` — Extract `THETAEDGE_API_KEY` and `THETAEDGE_API_BASE` from `skills.entries.thetix.env`
   - `~/.config/thetaedge/credentials.json` — Extract `api_key` and `api_base`
   Then export them in Bash:
   ```bash
   export THETAEDGE_API_KEY="<value from config>"
   export THETAEDGE_API_BASE="<value from config>"  # defaults to https://api.thetaedge.ai
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
       "THETAEDGE_API_BASE": "https://api.thetaedge.ai"
     }}}}
   }
   ```

   **Other agents** — Create `~/.config/thetaedge/credentials.json`:
   ```json
   {
     "api_key": "te_your_key_here",
     "api_base": "https://api.thetaedge.ai"
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

List and create user brokerage accounts. Account IDs are needed for account-scoped features like opportunities, dashboard queries, and calculators.

### List Accounts

`GET /api/accounts`

Returns an array of account objects. Automatically filters out deleted and error-status accounts.

### Create Manual Account

`POST /api/accounts`

```json
{
  "name": "My Account"
}
```

Returns 201 with the new account object. Triggers background onboarding.

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

## Response Formatting

When presenting thetix results to the user:

- Format monetary values with dollar signs and two decimal places
- Format percentages with one decimal place
- Present tables using markdown table syntax
- For payoff diagrams, describe the key levels (breakeven, max profit, max loss)
- Summarize long chat responses, highlighting actionable insights
- When showing opportunity details, always include: ticker, strike, premium, expiration, and key metrics
