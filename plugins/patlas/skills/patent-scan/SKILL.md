---
name: patent-scan
description: Scans a codebase and its docs for distinct technical mechanisms, then checks each one against prior art using the Patlas MCP tools (search_prior_art, compare_claims). Use when the user asks to find prior art, scan their code for inventions, or check what might be worth a patent search — never to render a patentability or infringement verdict.
---

<!--
Copyright 2026 Yodha Systems LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

This license covers this skill file only. It does not grant any rights to
the Patlas MCP server, its API, or the underlying service this skill calls
into — use of Patlas is governed separately by its Terms of Service at
https://patlas.dev/terms.
-->

## Prerequisites

This skill drives the Patlas MCP server's tools — it does no patent search of its own. **Resolve the actual tool names before calling anything**: look through the tools available in this session for names matching `*search_prior_art` and `*compare_claims`. A connected remote MCP server's tools are commonly exposed with a `mcp__<server-name>__<tool-name>` prefix (e.g. `mcp__patlas__search_prior_art`), not the bare name — use whichever matching name is actually present in this session's tool list. If neither a bare name nor a `*search_prior_art`-suffixed name is found, Patlas isn't connected: tell the user to connect it first (sign in with GitHub — no API key needed — by pointing an MCP client at `https://patlas.dev/mcp`; see https://patlas.dev for details) and stop. Don't assume the user has any particular codebase available beyond the one they're asking you to scan — this skill runs against *their* project, not Patlas's own.

`compare_claims` is PAYG/Subscription tier only. If a call to it fails with `QUOTA_EXCEEDED`, don't retry it — continue with `search_prior_art`-only results for the rest of the scan, and say plainly in the final report that claim-level comparison was skipped because it's not available on the connected key's tier.

## Instructions

1. **Scope.** If the user didn't specify a directory or module, ask before scanning a large monorepo — a targeted scan produces better candidates than a shallow pass over everything.

2. **Explore.** Read the code and docs in scope (README, design docs, comments). For a large or multi-module scope, spawn one Explore/general-purpose subagent per module/area to read in parallel and report back a short list of candidate mechanisms each — this is the main advantage over doing it serially in one pass. Have each subagent report *what* the mechanism is and *where* it lives (file/function), not just a one-line label.

3. **Filter to distinct, real candidates.** Merge the subagent reports (or your own single-pass read) into one list, applying two checks to every entry:
   - **Distinct:** if two entries are the same idea described two different ways, merge them into one.
   - **Real:** only keep mechanisms you can point to in actual code or docs — don't infer or invent one just to have something to report.
   Drop anything that's a standard, well-known engineering pattern with nothing distinctive about this specific implementation — not worth spending a search on.

4. **Search.** For each remaining candidate, call the resolved `search_prior_art` tool (see Prerequisites) with a clear, specific description of that one mechanism (not the whole module, not the whole product).

5. **Compare.** For candidates where `search_prior_art` returns close matches, call the resolved `compare_claims` tool against the closest one or two results, tier permitting (see Prerequisites).

6. **Report.** Summarize per candidate: what it is, where it lives, what was found (or that nothing close was found), and the claim-level comparison if one was run. Offer to write the summary to a file (e.g. `PRIOR_ART_SCAN.md`) if the user wants it persisted — don't write one unasked.

## No-verdict discipline (hard requirement)

Never state or imply that something *is* patentable, *is not* patentable, infringes, is clear to file, or has freedom to operate — that determination belongs to a patent attorney, not this skill. A "high" claim overlap is a similarity signal to go look at, not evidence of infringement; "nothing close found" means the search didn't turn up overlap in the sources it checked, not that filing is safe. Frame every result as a **candidate worth checking further**, matching the same discipline Patlas's own tools enforce on their output (see `compare_claims`'s disclaimer).

False positives — flagging routine or non-patent-eligible code (plain business logic, a standard library call) as a "candidate" — are a worse outcome than finding nothing, since they waste the reader's time chasing noise. When in doubt, leave a mechanism out rather than pad the list.

## Example

- **User:** "Scan this repo for anything worth checking against prior art before we talk to a lawyer."
- **Agent:** Reads the repo (spawning subagents per module if it's large), settles on a short list of distinct, real mechanisms, resolves the actual `search_prior_art`/`compare_claims` tool names for this session (see Prerequisites), runs `search_prior_art` on each candidate, runs `compare_claims` on the closest matches (if the tier allows), and reports back a per-candidate summary framed as "worth checking further" — never as a verdict.
