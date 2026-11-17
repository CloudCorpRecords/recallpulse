# recallpulse

How fast are complaints about a product piling up, and what does that velocity say about its recall risk? recallpulse is a local forwarding client for the hosted recall-risk MCP service: it answers `tools/list` locally from the captured tool definition and forwards every call to the service, returning the answers exactly as the service sends them back.

The single tool, `recall_risk`, returns a 0–100 risk score and band for a named product, brand or model, together with the signal inputs behind it — complaint velocity and acceleration, and the severe-hazard share. The score is a signal computed from public complaint data, shown with its inputs; it is not an official recall, not a claim that a product is defective, and not a verdict about any person. This client treats it that way: it forwards the question and passes the answer through.

The 1 captured tool is built; there are no omissions.

## The tool

- **`recall_risk`** — The complaint-velocity recall-risk signal for one tracked product, brand or model: a 0–100 risk score, its band, complaint velocity and acceleration, and the severe-hazard share. `entity` is required: a product/brand/model name or a tracked id (for example `Ford Mustang Mach-E`, `Dexcom G7`, or `veh-tesla-modely-2023`).

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/recallpulse mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "recallpulse": {
      "command": "npx",
      "args": ["-y", "@reneturcios/recallpulse", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/recallpulse
docker run --rm ghcr.io/cloudcorprecords/recallpulse recall_risk '{"entity":"Dexcom G7"}'
```

## CLI

One binary serves both the command line and the MCP server. The tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
recallpulse recall_risk '{"entity":"Ford Mustang Mach-E"}'
recallpulse recall_risk '{"entity":"veh-tesla-modely-2023"}'
recallpulse --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `RECALLRISK_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://cassandra.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `recallpulse error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 1 tool, deep-equal, with no title invented and `required: [entity]` exactly as captured), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
