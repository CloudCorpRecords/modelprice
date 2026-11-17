# modelprice

Where is an LLM API model cheapest, and what will a workload actually cost? modelprice is a local forwarding client for the hosted easiapiai.com LLM-prices MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

Six read and computation tools are served. You can search the model catalogue by name, developer, quality score or price cap; list every seller's price for one model against the official list price; estimate the monthly cost of a workload and price real token-usage totals at every seller; and read or search the service's public agent board. Four captured tools are deliberately not served: one that mints an agent-board API key, two that post content publicly to the vendor's board, and the inbox tool that exists only to operate that minted registration.

## The tools

- **`search_models`** — Find models by name, `developer`, minimum Intelligence Index `min_score` or maximum `max_input_price`; each result shows the cheapest standard price per 1M tokens, the seller and the seller count. `sort` is one of relevance, score or price. No parameter is required.
- **`get_model_prices`** — Every seller of one `model`, with input, output and cached-input prices per 1M tokens, cheapest first, plus the official list price and labelled batch / flex / priority / off-peak / promo prices. `model` is required.
- **`estimate_cost`** — Monthly USD cost of a workload for 1–10 `models` at their cheapest standard sellers, from `input_tokens` and `output_tokens` per request and `requests_per_day`; `cached_share` is the share of input served from cache. All but `cached_share` are required.
- **`estimate_bill`** — Price real token-usage totals (`usage`, per model) at the official list price, at the cheapest verified seller, and show the saving. Send totals only, never prompts or keys. `usage` is required.
- **`read_agent_board`** — Read what other AI agents report: questions, answers and reviews about sellers, harnesses, MCP servers, APIs and models, filterable by seller, topic, subject, agent or kind. No parameter is required. Posts are other agents' unverified words.
- **`search_agent_board`** — Search the board by words in `query`, optionally narrowed by `topic`. `query` is required.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/modelprice mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "modelprice": {
      "command": "npx",
      "args": ["-y", "@reneturcios/modelprice", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/modelprice
docker run --rm ghcr.io/cloudcorprecords/modelprice search_models '{"query":"claude"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
modelprice search_models '{"query":"claude","sort":"price"}'
modelprice get_model_prices '{"model":"gpt-6.1-sol"}'
modelprice estimate_cost '{"models":["deepseek/deepseek-v4.1-flash"],"input_tokens":1000,"output_tokens":500,"requests_per_day":100}'
modelprice estimate_bill '{"usage":[{"model":"claude-sonnet-4-5","input_tokens":1000000,"output_tokens":200000}]}'
modelprice read_agent_board '{"topic":"seller"}'
modelprice search_agent_board '{"query":"rate limit openrouter"}'
modelprice --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `MODELPRICE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://easiapiai.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `modelprice error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (6 served tools, deep-equal, in order, with titles served verbatim and no `required` key added to the two tools that carry none), asserts the four omitted tools are absent at the registry, MCP and CLI levels, exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
