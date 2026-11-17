# BUILD REPORT — modelprice

Clean-room rebuild of `com.easiapiai/llm-prices` v1.1.0
(remote-only MCP server at `https://easiapiai.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/modelprice` 0.1.0, registry name
`io.github.CloudCorpRecords/modelprice`.

## Surface built

Labelled subset: **6 of 10 captured tools built**,
4 omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `search_models` | NO `required` key in the capture — served verbatim, none invented |
| `get_model_prices` | `model` |
| `estimate_cost` | `models`, `input_tokens`, `output_tokens`, `requests_per_day` |
| `estimate_bill` | `usage` |
| `read_agent_board` | NO `required` key in the capture — served verbatim, none invented |
| `search_agent_board` | `query` |

Omitted (absent at registry, MCP-layer and CLI level, and
asserted absent in tests):

| Omitted tool | Reason (from the capture's own descriptions) |
|--------------|----------------------------------------------|
| `join_agent_board` | Mints and returns an agent-board API key, shown once (key minting) |
| `post_to_agent_board` | Posts reviews/questions/replies publicly to the vendor's board (public-board posting) |
| `review_my_human` | Posts a public review about the person the caller works for (public-board posting about a person) |
| `my_agent_inbox` | "Needs your key" — exists only to operate the identity minted by the omitted `join_agent_board` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the capture
carries `title` keys on every tool and they are served verbatim.
The tool literals were generated programmatically from
`harness/fixtures/llmprices-tools.json` by the generator script
at `/tmp/b50-modelprice-scratch/gen.py` (parse + re-serialise;
the generator's in-memory round-trip assertions confirmed the
emitted literals parse back deep-equal to the fixture's tool
objects — the script's file write of the snippet was blocked by
the VM write guard with EPERM, verified firsthand, so the
round-trip ran in memory and the final `src/forward.ts` was
written with the file tool) — never retyped by hand as a source
of truth — and the registration-integrity test suite plus the
parity harness confirmed the served literals deep-equal the
fixture's 6 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 6 served tools'
names, titles, descriptions, required fields (including the two
tools with no `required` key: `search_models` and
`read_agent_board`) and property shapes, and on the 4 omitted
tools.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://easiapiai.com/mcp`.
- Env override `MODELPRICE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`MODELPRICE_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `modelprice error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

None in the build itself. Two process notes:

- The VM write guard was active this window: exec writes of
  fixture-derived bytes were EPERM-blocked (the generator's
  snippet file landed 0 bytes), so every source file was written
  with the file tool and re-verified by size, as briefed.
- Lockfile generation took several attempts (see below); the
  successful attempt was the prescribed offline Route 3.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3). Attempt history, reported honestly: the first
  offline attempt failed `ENOTCACHED` (SDK packument not yet in
  the shared cache); the first Route-2 fallback attempt (egress
  proxy + explicit registry) died with an EPERM write during the
  guard window; the second Route-2 attempt produced no output
  for 535s under concurrent batch load and was killed; the
  retried Route 3 then succeeded in ~19s once sibling installs
  had populated the shared cache.
- Parses as JSON; `name` == `@reneturcios/modelprice`.
- Package entries: **128**.
- sha256: `2b76f7295065229a7a77b4d6962102c3676502bb8d3d37c5b42fa2fa9c8e0b48`
  (re-verified unchanged after the subsequent offline
  `npm install`).

## server.json

- Description exactly: "LLM API prices by seller: model search, per-seller prices, cost estimates and bill pricing." —
  **91 characters** (verified programmatically).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/modelprice:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/modelprice"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/modelprice`
  and its smoke test asserts the server name `modelprice`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **59 tests, 59 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 6
   tools, including title verbatim and required-key exactness
   assertions — search_models and read_agent_board have NO
   required key — plus subset assertions that each of the 4 omitted
   tools is absent at registry level, rejected with MethodNotFound
   at the MCP layer, and absent at the CLI level, session
   handling/reuse, env override, the no-Authorization /
   no-X-API-Key assertions under any environment, verbatim
   per-tool argument forwarding for all 6 tools and result
   pass-through including structuredContent and upstream isError,
   SSE parsing, error rendering for both the JSON-RPC-error and
   transport-failure paths, MCP-layer validation, and CLI
   behaviour including a working subcommand for every one of the
   6 tools against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/llmprices-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/modelprice/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/modelprice/parity-out` → **verdict PASS, parity 100.0**
   (6/6 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
