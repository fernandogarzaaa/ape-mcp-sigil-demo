# Sigil fixture badges

Verbatim copies of Sigil index files at pinned versions, copied Oct 31, 2026
from the local sigil-index checkout. Signatures are intact, so the gate
verifies them exactly as it would verify the live index files.

- badges/apify-actors-mcp-server/0.17.1.json: @apify/actors-mcp-server,
  riskScore 100, riskLevel low. The demo allow case.
- badges/modelcontextprotocol-server-github/2025.4.8.json:
  @modelcontextprotocol/server-github, riskScore 65, riskLevel high.
  The demo policy case: verified, but denied under the default policy,
  which blocks high and critical.
- badges/fixture-server/0.1.0.json plus
  revocations/fixture-server/0.1.0.json: fixture-server, riskScore 0,
  riskLevel critical, and revoked. The demo block case. This is a Sigil
  test fixture, not a real MCP server, and the demo labels it as such.

The local sigil-index checkout at copy time held 26 badges and predates
the remote index after PR 47. These fixtures pin the demo to exactly the
badge versions shown above, instead of depending on whatever the index
holds at demo time.
