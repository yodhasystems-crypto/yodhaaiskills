---
name: patent-scan
description: Scans a codebase and its docs for distinct technical mechanisms, then checks each one against prior art using the Patlas MCP tools (search_prior_art, compare_claims). Use when the user asks to find prior art, scan their code for inventions, or check what might be worth a patent search — never to render a patentability or infringement verdict.
---

<!--
This skill's filter criteria (step 3) and no-verdict framing restate what's
in src/mcp/prompts/patentScan.ts (the patent_scan MCP prompt, Option A) —
the two aren't generated from a shared source, so a wording change to
either should be mirrored in the other.
-->

## Prerequisites

This skill drives the Patlas MCP server's tools (`search_prior_art`, `compare_claims`) — it does no patent search of its own. If those tools aren't available in this session, tell the user to connect Patlas first (`docs/User-Guide.md` §2 in the patlas_app repo, or https://patlas.dev for a general audience) and stop.

`compare_claims` is PAYG/Subscription tier only. If a call to it fails with `QUOTA_EXCEEDED`, don't retry it — continue with `search_prior_art`-only results for the rest of the scan, and say plainly in the final report that claim-level comparison was skipped because it's not available on the connected key's tier.

## Instructions

1. **Scope.** If the user didn't specify a directory or module, ask before scanning a large monorepo — a targeted scan produces better candidates than a shallow pass over everything.

2. **Explore.** Read the code and docs in scope (README, design docs, comments). For a large or multi-module scope, spawn one Explore/general-purpose subagent per module/area to read in parallel and report back a short list of candidate mechanisms each — this is the main advantage over doing it serially in one pass. Have each subagent report *what* the mechanism is and *where* it lives (file/function), not just a one-line label.

3. **Filter to distinct, real candidates.** Merge the subagent reports (or your own single-pass read) into one list, applying two checks to every entry:
   - **Distinct:** if two entries are the same idea described two different ways, merge them into one.
   - **Real:** only keep mechanisms you can point to in actual code or docs — don't infer or invent one just to have something to report.
   Drop anything that's a standard, well-known engineering pattern with nothing distinctive about this specific implementation — not worth spending a search on.

4. **Search.** For each remaining candidate, call `search_prior_art` with a clear, specific description of that one mechanism (not the whole module, not the whole product).

5. **Compare.** For candidates where `search_prior_art` returns close matches, call `compare_claims` against the closest one or two results, tier permitting (see Prerequisites).

6. **Report.** Summarize per candidate: what it is, where it lives, what was found (or that nothing close was found), and the claim-level comparison if one was run. Offer to write the summary to a file (e.g. `PRIOR_ART_SCAN.md`) if the user wants it persisted — don't write one unasked.

## No-verdict discipline (hard requirement)

Never state or imply that something *is* patentable, *is not* patentable, infringes, is clear to file, or has freedom to operate — that determination belongs to a patent attorney, not this skill. A "high" claim overlap is a similarity signal to go look at, not evidence of infringement; "nothing close found" means the search didn't turn up overlap in the sources it checked, not that filing is safe. Frame every result as a **candidate worth checking further**, matching the same discipline Patlas's own tools enforce on their output (see `compare_claims`'s disclaimer).

False positives — flagging routine or non-patent-eligible code (plain business logic, a standard library call) as a "candidate" — are a worse outcome than finding nothing, since they waste the reader's time chasing noise. When in doubt, leave a mechanism out rather than pad the list.

## Example

- **User:** "Scan this repo for anything worth checking against prior art before we talk to a lawyer."
- **Agent:** Reads the repo (spawning subagents per module if it's large), settles on a short list of distinct, real mechanisms, runs `search_prior_art` on each, runs `compare_claims` on the closest matches (if the tier allows), and reports back a per-candidate summary framed as "worth checking further" — never as a verdict.
