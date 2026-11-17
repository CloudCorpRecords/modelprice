# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted easiapiai.com LLM-prices MCP server. Six of the ten captured tools are answered locally for tools/list (a labelled subset; see below), and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools served: `search_models`, `get_model_prices`, `estimate_cost`, `estimate_bill`, `read_agent_board`, `search_agent_board`. The capture carries `title` keys on every tool and they are served verbatim; `search_models` and `read_agent_board` carry no `required` key in the capture and none is served.
- Omitted from the captured surface: `join_agent_board` (mints and returns an agent-board API key), `post_to_agent_board` (posts content publicly to the vendor's board), `review_my_human` (posts a public review about the caller's human), and `my_agent_inbox` (exists only to operate the registration minted by the omitted join tool).
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `modelprice error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `MODELPRICE_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
