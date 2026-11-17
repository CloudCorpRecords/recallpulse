# BUILD REPORT — recallpulse

Clean-room rebuild of `com.dropwatchhq/recall-risk`
v1.0.0 (remote-only MCP server at `https://cassandra.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/recallpulse` 0.1.0, registry name
`io.github.CloudCorpRecords/recallpulse`.

## Surface built

Full surface: **the 1 captured tool built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `recall_risk` | `entity` |

The served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
key, so none is served. The tool literal was generated
programmatically from
`harness/fixtures/recallrisk-tools-original.json` by the script at
`/tmp/b47-recallpulse-scratch/gen_forward.py` (parse + re-serialise
into `src/forward.ts`) — never retyped — and post-generation checks
(the generator's own round-trip assertion, the registration-integrity
test suite, plus the parity harness) confirmed the served literal
deep-equals the fixture's 1 tool object. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool's name,
description, required field (`entity`) and property shape. The
spec's note that the capture carries no `title` key matches the
fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://cassandra.dropwatchhq.com/mcp`.
- Env override `RECALLRISK_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`RECALLRISK_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `recallpulse error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Recall-score judgment and RE-TRIAGE FLAG (from the spec, recorded
  by the coordinator on the firsthand capture read): `recall_risk`
  returns a 0–100 recall-risk score + band for a caller-named
  PRODUCT / BRAND / MODEL (captured examples: 'Ford Mustang
  Mach-E', 'Dexcom G7', a tracked product id), computed from
  PUBLIC complaint data, with its signal inputs shown (complaint
  velocity & acceleration, severe-hazard share) and an express
  captured disclaimer that it is "a RISK SIGNAL from public
  complaint data, NOT an official recall or a claim the product
  is defective". The subject is a product/brand, never a person,
  provider or counterparty being onboarded; no exclusion list is
  checked, no eligibility or compliance decision is gated, and
  the inputs are shown rather than hidden. That places it with
  the awardsignal distinction (score over public data about a
  non-person subject, gating nothing) rather than the screening
  SAFETY family (entity-screen's named-entity exclusion risk
  score; docketpeek's omitted `quick_scan` go/caution/stop
  verdict on a caller-named BUSINESS for diligence). Because a
  0–100 risk score for a named subject sits near that family
  boundary, this build carries the gap-392 flag: **the 0–100
  product/brand risk score was kept on the awardsignal
  distinction, and a reader who weighs the distinction
  differently should re-triage recallpulse before its publish
  wave — recallpulse is FLAGGED FOR RE-TRIAGE BEFORE ITS PUBLISH
  WAVE.** No payment, metering, contact, signup or send tool
  exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/recallpulse`.
- Package entries: **128**.
- sha256: `b2f84697e45c142b23c78a77ab3ce7b5d24932d0c3cd3ee0248ee2300edd99e3`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Recall-risk signal for products and brands
  from public complaint velocity data." — **79 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/recallpulse:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/recallpulse"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/recallpulse`
  and its smoke test asserts the server name `recallpulse`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **52 tests, 52 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 1
   tool, including the no-title and `required: [entity]` exactness
   assertions, plus session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, `recallpulse error:` rendering,
   MCP-layer validation, and CLI behaviour — `--version`, a
   per-tool call against a local stub upstream, and usage exit 2).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/recallrisk-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/recallpulse/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/recallpulse/parity-out` → **verdict PASS, parity 100.0**
   (1/1 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
