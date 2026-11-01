# APE MCP plus Sigil demo (Meta hackathon, standalone copy)

This copy is a standalone demo of APE with a Sigil trust gate. It is not
the APE main repo, and nothing here has been pushed or merged anywhere.

## What the gate does

A request arrives, APE proposes a tool call, and before that call runs,
`src/agent/sigil-gate.js` checks the tool's Sigil badge:

1. Load the exact badge (server plus version) from a Sigil index directory.
2. Verify its Ed25519 signature and keyId.
3. Check revocation.
4. Apply the policy: block high and critical risk, block unknown badges,
   block bad signatures, block revoked badges. Low and medium are allowed.
5. Return a decision plus an audit entry (tool, server, version, badge
   type, risk score and level, keyId, signature validity, revoked flag,
   decision, reason, timestamp).

The gate is wired into `invokeTool` in `src/agent/registry.js`, which is
the choke point for both the sequential loop and the parallel fan-out,
and, since the Oct 31 Wonder-loop fix, for the loop's terminal finish
path too. Before that fix, finish bypassed `invokeTool` entirely, so a
gated finish tool was never checked in a live loop. Now the loop checks
a gated finish through `invokeTool` first; a denial is returned to the
model and the loop continues, instead of ending as if the finish ran.
A tool only gets gated when its profile entry carries `sigilServer` and
`sigilVersion`; ungated tools behave exactly as before. A denied call
returns `{ error: "sigil_blocked", sigil: <audit entry> }` and never
reaches dispatch. `sigil_blocked` is not in the loop's retryable set,
so a denial records once and the loop moves on.

## Run it

```
node demo/sigil-gate-demo.mjs
node --test tests/sigil-gate.test.js tests/sigil-gate-invoke.test.js
npm test
```

Wonder-in-loop, verified Oct 31 in this sandbox (demo/wonder-loop.mjs,
run with a connector surrogate key, never a raw key in a file):

- Allow: Muse Wonder 1.0 called finish, provider meta, model
  muse-wonder-1.0, stop explicit_final_answer, outcome
  "wonder loop test".
- Block: the same Wonder run, with finish gated to the revoked
  fixture, recorded two `sigil_blocked:revoked` steps, and Wonder
  itself stopped retrying the blocked call, ending with
  no_tool_call_in_step and an outcome explaining the block.
  The blocked tool never ran.

Tests after the fix: 13 gate and provider tests pass, 184 original
suite tests pass, in two runs.

The two demo cases use pinned fixture badges under
`demo/fixtures/sigil-index` (see that directory's README):

- Allow: @apify/actors-mcp-server 0.17.1, score 100, low risk.
- Block: fixture-server 0.1.0, score 0, critical risk, revoked.
  This is a Sigil test fixture, not a real MCP server.

A third fixture, @modelcontextprotocol/server-github 2025.4.8 (score 65,
high risk), proves the policy blocks a verified high-risk badge too.

## 3-minute video outline

- 0:00 Problem: an agent that trusts a tool's README before it calls it.
- 0:30 Gate code: badge load, signature verify, revocation check, policy.
- 1:00 Run `node demo/sigil-gate-demo.mjs`: ALLOW with the audit entry,
  then DENY with the revocation reason in the audit entry.
- 1:30 Run the Sigil tests: allow, revoked block, unknown block,
  tampered block, high-risk block, invokeTool allow and block.
- 2:00 Show the invokeTool wiring: the gate sits before dispatch, and
  ungated tools are untouched. Then show the Wonder loop: allow, then
  block, with the blocked tool never running.
- 2:30 Limits, stated plainly: the demo runs with pinned fixtures;
  the blocked case is a labelled fixture, not a real server; the
  Wonder loop was run in this sandbox through the connector
  credential, and live mode still needs Inan's account and spend
  approvals for Stripe, AWS, and SMS.

## Honest limits for the submission

- A Meta model call is now wired and verified in the loop, in this
  sandbox, through the connector credential. It is not yet a public
  demo repo, and the demo copy has not been pushed anywhere.
- The blocked demo case is a fixture. The submission must say so.
- Badge versions are pinned to the fixtures above. If the live index
  moves, the demo still shows exactly these versions.
