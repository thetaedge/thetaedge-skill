# Thetix API Reference

Detailed endpoint reference for the ThetaEdge thetix API.

## Authentication

All endpoints (except those under `/api/public/`) require authentication:

```
Authorization: Bearer <API_KEY>
```

## Thetix Cards

### Card Management

#### List Cards
```
GET /api/thetix-cards?collection_id=<uuid>&card_ids=<id1,id2>
```
Returns array of card objects with materialized results.

#### Create Card
```
POST /api/thetix-cards
```
```json
{
  "user_query": "string",
  "collection_id": "uuid",
  "meta_data": {},
  "position": 0,
  "update_cadence_seconds": 0
}
```
Returns card object with `job_status: "pending"`. Processing is async.

#### Get Card
```
GET /api/thetix-cards/<card_id>
```
Returns `{ card: {...}, collection: {...} }`.

#### Update Card
```
PUT /api/thetix-cards/<card_id>
```
```json
{
  "user_query": "string",
  "position": 0,
  "meta_data": {},
  "update_cadence_seconds": 0
}
```

#### Delete Card
```
DELETE /api/thetix-cards/<card_id>
```
Returns 204 No Content.

#### Materialize Card
```
POST /api/thetix-cards/<card_id>/materialize
```
Returns array of materialized widgets.

#### Refresh Card
```
POST /api/thetix-cards/<card_id>/refresh
```
Returns updated card object with new `materialized_result`. Rate limited to 1 per 10 minutes.

#### Move Card
```
POST /api/thetix-cards/<card_id>/move
```
```json
{
  "target_collection_id": "uuid"
}
```
Returns `{ success: true, old_collection_id: "string", new_collection_id: "string" }`.

#### Cancel Card Processing
```
POST /api/thetix-cards/<card_id>/cancel
```
Returns `{ success: true, message: "string" }`.

### Card Collections

#### List Collections
```
GET /api/thetix-card-collections?collection_type=<type>
```
Returns array of collection objects with `card_count`.

#### Create Collection
```
POST /api/thetix-card-collections
```
```json
{
  "name": "string",
  "description": "string",
  "meta_data": {},
  "is_default": false,
  "is_public": false
}
```
Returns collection object (201 Created).

#### Get Collection
```
GET /api/thetix-card-collections/<collection_id>
```
Returns collection object with `cards` array.

#### Update Collection
```
PUT /api/thetix-card-collections/<collection_id>
```
```json
{
  "name": "string",
  "description": "string",
  "meta_data": {},
  "is_public": false,
  "is_default": false
}
```

#### Delete Collection
```
DELETE /api/thetix-card-collections/<collection_id>
```
Returns 204 No Content.

#### Populate Collection
```
POST /api/thetix-card-collections/<collection_id>/populate
```
```json
{
  "replace_existing": false
}
```
Returns `{ success: true, cards_created: 5, message: "string" }`.

#### Reorder Cards
```
POST /api/thetix-card-collections/<collection_id>/reorder
```
```json
{
  "positions": [
    { "id": "uuid", "position": 0 },
    { "id": "uuid", "position": 1 }
  ]
}
```

#### Get Collection Metadata
```
GET /api/thetix-card-collections/<collection_id>/metadata
```
Returns `{ collection: {...}, cards: [{ id, query, position, job_status, job_progress }] }`.

#### Get Collection Status
```
GET /api/thetix-card-collections/<collection_id>/status
```
Returns `{ cards: [{ id, update_cadence_seconds, materialized_updated_at, scheduled_update_at }] }`.

## Thetix Chats

### Chat Management

#### List Chats
```
GET /api/thetix-chats?collection_id=<uuid>&chat_type=<type>&chat_ids=<id1,id2>&account_id=<id>&limit=<n>
```
Returns array of chat metadata (content field excluded for performance).

#### Create Chat
```
POST /api/thetix-chats
```
```json
{
  "collection_id": "uuid",
  "chat_type": "general",
  "content": {},
  "meta_data": {}
}
```
Returns chat object (201 Created).

#### Get Chat
```
GET /api/thetix-chats/<chat_id>
```
Returns full chat object including `content`.

#### Update Chat
```
PUT /api/thetix-chats/<chat_id>
```
```json
{
  "content": {},
  "meta_data": {},
  "chat_type": "string"
}
```

#### Delete Chat
```
DELETE /api/thetix-chats/<chat_id>
```
Returns 204 No Content.

#### Cancel Chat Processing
```
POST /api/thetix-chats/<chat_id>/cancel
```
Returns `{ success: true, message: "string" }`.

#### Search Chats
```
GET /api/thetix-chats/search?q=<text>&account_id=<id>&limit=<n>
```
Search chats by query text. Returns matching chat metadata.

#### Pin/Unpin Chat
```
POST /api/thetix-chats/<chat_id>/pin
```
Toggles the pinned status of a chat.

#### Set Chat Title
```
POST /api/thetix-chats/<chat_id>/title
```
```json
{
  "title": "string"
}
```
Sets a custom title on a chat.

#### Unsave Chat
```
POST /api/thetix-chats/<chat_id>/unsave
```
Removes saved report status from a chat.

#### Get Chat Status
```
GET /api/thetix-chats/status?chat_ids=<id1,id2>&collection_id=<uuid>
```
Returns array of `{ id, job_status, job_progress, updated_at }`.

### Chat Processing

#### Process General Query
```
POST /api/thetix-chats/process
```
```json
{
  "query": "string",
  "collection_id": "uuid",
  "chat_id": "uuid (optional, for continuing a conversation)",
  "chat_type": "general",
  "meta_data": {},
  "context": [],
  "custom_prompt": "string (optional)"
}
```
Returns `{ saved_chat: {...}, async: true }`. Poll status endpoint for completion.

#### Process Opportunity Query
```
POST /api/thetix-chats/process-opportunity
```
```json
{
  "opportunity": { "id": 123, "opportunity_type": "covered_call", "details": {} },
  "query": "string",
  "collection_id": "uuid",
  "chat_id": "uuid (optional)",
  "meta_data": {},
  "context": []
}
```

#### Process Dashboard Query
```
POST /api/thetix-chats/process-dashboard
```
```json
{
  "account_id": "string",
  "query": "string",
  "collection_id": "uuid",
  "chat_id": "uuid (optional)",
  "meta_data": {},
  "context": []
}
```

### Chat Collections

#### List Chat Collections
```
GET /api/thetix-chat-collections?collection_type=<type>
```
Returns array of collection objects with `chat_count`.

#### Create Chat Collection
```
POST /api/thetix-chat-collections
```
```json
{
  "name": "string",
  "description": "string",
  "collection_type": "string",
  "meta_data": {}
}
```

#### Get Chat Collection
```
GET /api/thetix-chat-collections/<collection_id>
```
Returns collection with all chats.

#### Update Chat Collection
```
PUT /api/thetix-chat-collections/<collection_id>
```
```json
{
  "name": "string",
  "description": "string",
  "meta_data": {}
}
```

#### Delete Chat Collection
```
DELETE /api/thetix-chat-collections/<collection_id>
```
Returns 204 No Content.

## Opportunities

### List Opportunities
```
GET /api/opportunities?tickers=<t1,t2>&status=<status>&accountId=<id>&limit=<n>&risk_level=<level>&frequency=<freq>&generated_date_start=<YYYY-MM-DD>&generated_date_end=<YYYY-MM-DD>
```
Query params (all optional):
- `tickers` — Comma-separated ticker symbols
- `status` — Filter by status (e.g. `active`, `expired`, `completed`, `rejected`)
- `accountId` — Brokerage account ID (camelCase)
- `limit` — Max number of results
- `risk_level` — `conservative`, `balanced`, or `aggressive`
- `frequency` — `weekly` or `monthly`
- `generated_date_start` — Start date filter (YYYY-MM-DD)
- `generated_date_end` — End date filter (YYYY-MM-DD)

Returns array of opportunity objects.

### Get Opportunity
```
GET /api/opportunities/<opp_id>
```
Returns full opportunity with details, rationale, and expected outcome.

### Act on Opportunity
```
POST /api/opportunities/<opp_id>
```
```json
{
  "action": "act | dismiss",
  "reason": "string (optional, for dismiss)",
  "details": {
    "strike": "number (optional)",
    "expiration": "YYYY-MM-DD (optional)",
    "contracts": "number (optional)",
    "premium": "number (optional)",
    "closePrice": "number (optional, for rolls)"
  }
}
```
Use `action: "act"` to execute the opportunity or `action: "dismiss"` to reject it. The `details` object allows overriding opportunity parameters when acting.

### Save Opportunity
```
POST /api/opportunities/save
```
```json
{
  "opportunity_type": "covered_call",
  "tickers": ["AAPL"],
  "details": {},
  "rationale": { "reason": "string", "strategy": "string", "benefit": "string" }
}
```

### Submit Feedback
```
POST /api/opportunities/<opp_id>/feedback
```
```json
{
  "rating": 1-5,
  "comments": "string"
}
```
At least one of `rating` or `comments` must be provided. This endpoint is for rating and commenting on an opportunity, not for acting on it (use `POST /api/opportunities/<opp_id>` for act/dismiss).

### Get Order Preview
```
GET /api/opportunities/<opp_id>/preview?strike=<strike>&expiration=<exp>&contracts=<n>&premium=<p>&closePrice=<price>
```
Query params (all optional — defaults come from the opportunity):
- `strike` — Override strike price
- `expiration` — Override expiration date
- `contracts` — Override number of contracts
- `premium` — Override premium
- `closePrice` — Close price (for rolls)

Returns order preview with P&L calculations.

### Covered Call Calculator
```
POST /api/opportunities/covered-call-calculator
```
```json
{
  "underlying": "AAPL",
  "strike": 180,
  "expiration": "2025-03-21",
  "contracts": 1,
  "account_id": "string (optional)"
}
```
The server fetches current price and premium from market data automatically.

Returns `{ details: { premium_income, max_profit, breakeven, return_on_capital }, payoff: [...], metrics: {...} }`.

### Cash-Secured Put Calculator
```
POST /api/opportunities/csp-calculator
```
```json
{
  "underlying": "AAPL",
  "strike": 170,
  "expiration": "2025-03-21",
  "contracts": 1,
  "account_id": "string (optional)"
}
```
The server fetches current price and premium from market data automatically.

Returns `{ details: {...}, payoff: [...], metrics: {...} }`.

### Roll Calculator
```
POST /api/opportunities/roll-calculator
```
```json
{
  "underlying": "AAPL",
  "strike": 185,
  "expiration": "2025-04-18",
  "contracts": 1,
  "account_id": "string (optional)",
  "current_position": {
    "strike": 180,
    "expiration": "2025-03-21",
    "symbol": "AAPL250321C00180000",
    "avg_price": 3.50
  }
}
```
Returns roll analysis with net credit/debit, new position metrics, and comparison to current position.

## Accounts

### List Accounts
```
GET /api/accounts
```
No query parameters. Returns array of account objects for the authenticated user. Automatically excludes deleted accounts and accounts with error setup status.

## Ideas

### List Ideas
```
GET /api/thetix/ideas?date=<YYYY-MM-DD>&start_date=<YYYY-MM-DD>&end_date=<YYYY-MM-DD>&days=<n>&reports=<n>&account_id=<id>
```
Query params (all optional):
- `date` — Specific date (YYYY-MM-DD); returns ideas for that day only
- `start_date` — Start of date range (YYYY-MM-DD)
- `end_date` — End of date range (YYYY-MM-DD)
- `days` — Number of days to look back (min 1, max 30)
- `reports` — Get ideas from the N most recent reports (min 1, max 10)
- `account_id` — Filter by brokerage account ID

Parameter priority: `date` > `reports` > `start_date/end_date` > `days` > default (today only).

Returns `{ ideas: [...], summary: {...} }`.

Ideas are sorted by priority (high → medium → low), then by deadline (earliest first).

This endpoint is **read-only** — no POST, PUT, or DELETE operations.

## Trade Cart

The trade cart is an **org-shared** staging area for trades before they are placed at the user's broker. The pipeline is **stage → review → execute**: every entry point (Thetix chat, ideas, opportunities, UI) stages a `CartItem`, and execution turns each item into a durable `Order` submitted to the broker.

### Authentication

- **Only `GET /api/cart` accepts API-key auth.** Every other cart REST method (add/modify/remove items, execute, read/clear orders, clear cart) is **session-only** — an API-key request to those returns `403 {"error": "This operation requires session authentication"}`. They back the interactive UI and are documented below for completeness, but an API-key client (including this skill) cannot call them directly.
- **Trading via Thetix chat works with API-key auth.** The chat process endpoint (`POST /api/thetix-chats/process`, see Chat Processing above) runs a command action-loop that stages, executes, removes, and clears the cart **server-side** in response to natural language ("add this to my cart", "execute", "remove the AAPL trade", "clear my cart", "show my recent orders"). This is the path an API-key client uses to stage and place trades. The loop never executes in a report context and will not chain staging and execution in a single turn (stage in one turn, review, then `execute` in a follow-up on the same `chat_id`).
- **Execution has a server-side two-step confirm.** `execute` only places orders when a `cart_review` widget was rendered in the **immediately prior turn** of that chat (staging renders it; so does a prior "cold" execute). A `execute` with no cart shown in the prior turn re-surfaces the cart ("Review the trades below, then reply 'execute' to place the orders.") and places nothing — the caller must send `execute` again. Because the gate reads the chat's conversation history, execute on the **same `chat_id`** used for staging.

### Get Cart (API key OK)
```
GET /api/cart?live=<0|1>&fresh=<0|1>
```
Query params (optional):
- `live` — `1`/`true` re-prices every item to the current market mid and recomputes metrics (net credit/debit, breakevens, max profit/loss, buying-power warnings). Base GET returns instant stale-spec metrics.
- `fresh` — `1`/`true` (only meaningful with `live`) bypasses the price cache so the quote is re-fetched, not replayed from the recent poll cache.

Returns:
```json
{
  "cart": { "id": "uuid", "status": "open", "account_id": "uuid | null" },
  "items": [ /* CartItem objects */ ]
}
```

### Clear Cart (session-only)
```
DELETE /api/cart
```
Removes all items from the open cart. Returns `{ "success": true }`.

### Add Cart Item (session-only)
```
POST /api/cart/items
```
Provide exactly one input mode:
```json
// From a stored opportunity
{ "opportunity_id": 1234, "overrides": { "strike": 180, "expiration": "2025-03-21", "contracts": 1, "premium": 3.50 } }

// From a raw trade spec
{ "trade_spec": { /* TradeSpec dict */ }, "account_id": "uuid", "order_type": "Limit", "time_in_force": "Day", "limit_price": 3.50, "source": "ui" }

// From a payoff widget (ideas path)
{ "payoff": { "symbol": "AAPL", "current_price": 182.0, "legs": [ /* ... */ ] } }
```
Validates the spec, that it has executable legs, and that any `account_id` belongs to the org. Returns `{ "item": <CartItem> }`. Errors: 400 (invalid spec/payoff/no legs), 403 (foreign account / not session auth), 404 (opportunity not found).

### Update Cart Item (session-only)
```
PATCH /api/cart/items/<item_public_id>
```
```json
{ "order_type": "Limit", "time_in_force": "Day", "limit_price": 3.40, "account_id": "uuid", "contracts": 2, "shares": 200 }
```
Any field is optional. `contracts` (options) or `shares` (equity) inline-resizes the staged trade. Any edit resets validation to unchecked. Returns `{ "item": <CartItem> }`. 404 if not found.

### Remove Cart Item (session-only)
```
DELETE /api/cart/items/<item_public_id>
```
Returns `{ "success": true }`. 404 if not found.

### Execute Cart (session-only)
```
POST /api/cart/execute
```
```json
{ "items": ["<item_public_id>", "..."] }
```
`items` is optional — empty/absent executes **all** items in the open cart. Each item is executed independently into an `Order` (one failure never cascades). Successfully placed items archive to an executed cart; rejected/errored items stay staged for retry.

Returns:
```json
{ "orders": [ /* Order object, or { "item_id": "uuid", "error": "message" } per failed item */ ] }
```
This REST endpoint requires **session auth** (API keys are forbidden) — it backs the cart's Execute button. **API-key clients execute via Thetix chat instead**: send `execute` to `POST /api/thetix-chats/process` after staging and review (see "Trading via Thetix chat" above). The chat path runs the same `OrderExecutionService.execute_item` pipeline server-side and reports per-trade outcomes (placed/working/filled/rejected).

### Get Recent Orders (session-only)
```
GET /api/cart/orders
```
Returns up to 50 recent non-dismissed orders for the org (newest first): `{ "orders": [ <Order>, ... ] }`. Lazily reconciles open orders against the broker (throttled per-org). The frontend polls this every 5s while orders are live.

### Clear Recent Orders (session-only)
```
DELETE /api/cart/orders
```
Marks all **terminal** orders as dismissed (hidden, kept in DB). Working orders stay visible. Returns `{ "success": true }`.

### Clear One Order (session-only)
```
DELETE /api/cart/orders/<order_public_id>
```
Dismisses a single order. 400 if the order is not terminal ("Can't clear an order that's still working"), 404 if not found.

## Public Endpoints (No Auth)

### Public Card Collections
```
GET  /api/public/thetix-collections/<collection_id>?card_ids=<ids>
GET  /api/public/thetix-collections/<collection_id>/metadata
GET  /api/public/thetix-collections/<collection_id>/status
```

### Public Cards
```
GET  /api/public/thetix-cards/<card_id>
GET  /api/public/thetix-cards/<card_id>/status
GET  /api/public/thetix-cards/<card_id>/preview.png
```

### Public Calculator
```
POST /api/public/covered-call-calculator
```
Same request/response as the authenticated covered call calculator (uses `underlying`, `strike`, `expiration`, `contracts`).

## Data Types

### Card Object
```
{
  id: string (UUID)
  user_id: int
  collection_id: string (UUID)
  query: string
  materialized_result: any[]          # Array of widget objects
  code: string | null                 # Generated EdgeScript DSL
  dsl_template: string | null
  meta_data: object
  position: int
  update_cadence_seconds: int
  scheduled_update_at: number | null  # Unix ms
  materialized_updated_at: number | null
  job_status: "pending" | "processing" | "failed" | "cancelled" | null  # null means completed
  job_progress: string | null
  created_at: number (unix ms)
  updated_at: number (unix ms)
}
```

### Chat Object
```
{
  id: string (UUID)
  user_id: int
  collection_id: string (UUID)
  chat_type: "general" | "opportunity" | "dashboard"
  content: any[] | object             # Array of widgets or legacy dict
  meta_data: object
  query: string | null
  job_status: "pending" | "processing" | "failed" | "cancelled" | null  # null means completed
  job_progress: string | null
  store: object | null                # Persistent state across turns
  store_manifest: object | null       # Descriptions of stored keys
  created_at: number (unix ms)
  updated_at: number (unix ms)
  title: string                       # Server-generated from content
}
```

### Opportunity Object
```
{
  id: int
  user_id: int
  account_id: string
  tickers: string[]
  opportunity_type: "covered_call" | "put_sell" | "option_roll" | "option_close"
  status: "active" | "expired" | "completed" | "rejected"
  details: {
    strike: number
    premium: number
    expiration: string
    current_price: number
    delta: number
    ...
  }
  rationale: {
    reason: string
    strategy: string
    benefit: string
  }
  expected_outcome: object
  generated_at: number (unix ms)
  expires_at: number (unix ms)
  strategy_id: int | null
  feedback: object | null
}
```

### Account Object
```
{
  id: string
  name: string
  source: string
  hidden: boolean
  positionsCount: int
  setupStatus: string
  createdAt: number (unix ms)
  externalId: string | null
  connectionId: string | null
  mode: string | null
}
```

### Idea Object
```
{
  report_id: string (UUID)              # Source report ID
  report_date: string (ISO 8601)        # When the report was created
  account_id: string                    # Brokerage account ID
  account_name: string                  # Human-readable account name
  title: string                         # Idea title
  description: string                   # Detailed description
  priority: "high" | "medium" | "low"   # Defaults to "medium"
  type: string                          # e.g. "roll", "trade", "monitor", "other"
  estimatedValue: number | null         # Potential monetary value
  deadline: string | null               # Human-readable: "today", "tomorrow", "3 days", or date
  deadline_timestamp: number | null     # Unix ms of parsed deadline
  widgets: any[]                        # Widget content from the idea
}
```

### Ideas Summary Object
```
{
  total_ideas: int
  by_priority: { high: int, medium: int, low: int }
  by_type: { roll: int, trade: int, monitor: int, ... }
  date_range: { start: string (ISO 8601), end: string (ISO 8601) }
}
```

### Cart Object
```
{
  id: string (UUID)                     # public_id of the open cart
  status: "open" | "executing" | "executed" | "abandoned"
  account_id: string | null             # default target account for the cart
}
```

### Cart Item Object
```
{
  id: string (UUID)                     # public_id of the item
  underlying: string                    # ticker
  structure: string                     # e.g. covered_call, csp, bull_put_spread (see TradeSpec)
  asset_class: "equity" | "option" | "multi_leg_option" | null
  trade_spec: object                    # serialized TradeSpec (see below)
  account_id: string | null             # per-item account override
  order_type: string                    # default "Limit"
  time_in_force: string                 # default "Day"
  limit_price: number | null
  price_effect: "CREDIT" | "DEBIT" | "EVEN" | null
  estimated_cost: number | null
  validation_status: "UNCHECKED" | "VALID" | "INVALID" | "EXPIRED" | null
  validation_result: object             # advisory risk warnings (e.g. undefined_risk, large_buying_power_use)
  source: "chat" | "opportunity" | "idea" | "ui" | "api" | null
  opportunity_id: int | null
  order_id: int | null                  # set once executed
  added_by_user_id: int
  added_by: { id, name, email, profile_picture } | null   # org-shared cart attribution
  metrics: {                            # live with ?live=1, else from stale spec
    net_credit, credit_str, bpr, max_profit, max_loss,
    breakevens, sizing_str, spot, iv, legs, payoff_legs
  }
}
```

### Order Object
```
{
  id: string (UUID)                     # public_id
  status: "pending" | "working" | "partially_filled" | "filled" | "cancelled" | "rejected" | null
  underlying: string
  structure: string
  asset_class: "equity" | "option" | "multi_leg_option" | null
  side_summary: string                  # human-readable side description
  account_id: string | null
  broker_order_id: string | null
  limit_price: number | null            # net limit for multi-leg
  price_effect: "CREDIT" | "DEBIT" | "EVEN" | null
  filled_price: number | null
  filled_quantity: number | null
  error_message: string | null
  submitted_at: string (ISO 8601) | null
  filled_at: string (ISO 8601) | null
  legs: [
    {
      ticker: string                    # OCC option symbol or equity symbol
      action: "BUY" | "SELL" | "BUY_TO_OPEN" | "SELL_TO_OPEN" | "BUY_TO_CLOSE" | "SELL_TO_CLOSE" | null
      quantity: number
      filled_quantity: number | null
      filled_price: number | null
      status: string | null
    }
  ]
}
```

Terminal order statuses (cannot regress, safe to clear): `filled`, `cancelled`, `rejected`.

### TradeSpec Object
The canonical description of a single trade, stored in `CartItem.trade_spec`. The skill normally lets Thetix chat build this; it is documented for reading staged items.
```
{
  symbol: string                        # underlying ticker (exactly one per spec)
  spot: number                          # underlying spot at build time
  structure: string                     # one of: long_stock, short_stock, csp, covered_call,
                                        #   long_call, long_put, short_call_naked, bull_put_spread,
                                        #   bear_call_spread, bull_call_spread, bear_put_spread,
                                        #   iron_condor, iron_butterfly, calendar, diagonal,
                                        #   straddle, strangle, ratio, collar, custom
  legs: [
    // OptionLeg
    { kind: "option", action: "sell_open"|"buy_open"|"sell_close"|"buy_close",
      option_type: "call"|"put", strike: number, contracts: int,
      premium: number, expiration: "YYYY-MM-DD", delta?, iv?, contract_symbol?, bid?, ask? },
    // StockLeg
    { kind: "stock", action: "buy"|"sell", shares: int, price?, bid?, ask? }
  ]
  iv: number | null
  next_earnings_date: string | null     # YYYY-MM-DD
  note: string
  overrides: object                     # for structure="custom": max_profit, max_loss, bpr, breakevens
}
```
Derived metrics (net credit, bpr, max profit/loss, breakevens) are recomputed, never stored on the spec.

### Collection Object
```
{
  id: string (UUID)
  user_id: int
  name: string
  description: string
  meta_data: object
  is_default: boolean
  is_public: boolean                  # Card collections only
  collection_type: string             # Chat collections only
  card_count: int | null
  chat_count: int | null
  created_at: number (unix ms)
  updated_at: number (unix ms)
}
```

## Async Processing Pattern

Both cards and chats use async job processing:

1. **Create/process** — Returns immediately with `job_status: "pending"`
2. **Poll** — Check status via lightweight status endpoints
3. **Retrieve** — Fetch full result once `job_status` is `null`

Status values: `"pending"` → `"processing"` → `null` (completed) | `"failed"` | `"cancelled"`

Note: `job_status` is `null` (not `"completed"`) when processing finishes successfully.

Widget types in materialized results: `markdown`, `table`, `optionsChain`, `payoffDiagram`, and others.
