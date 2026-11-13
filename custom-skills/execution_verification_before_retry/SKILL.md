---
name: execution-verification-before-retry
description: Verify an uncertain execution from its authoritative record before retrying it, so a completed action is never issued twice.
---

# Verify an execution before retrying it

## Treat unknown as unverified

- Treat an unknown, pending, or timed out submission as unverified rather than failed.
- Hold the next submission for the same economic action until verification finishes or its retry budget ends.
- Keep the submission identifier and the requested amount or quantity with the unverified record so verification can find it later.

## Verify from the authoritative record

- Check the authoritative history by submission identifier first and use the recorded fill quantity and amount when history confirms a fill.
- Check the balance change next when history lookup fails or returns no identifier and use a matching balance change as confirmation only when it matches the requested action in asset and size.
- Repeat the history and balance checks with a short wait between attempts instead of issuing a second submission after one failed lookup.

## Retry verification before retrying the action

- Retry verification several times across transient errors before deciding that a submission failed.
- Stop retrying the same action when repeated attempts return a shortage error after a fill was already confirmed or a balance change already shows the action completed.
- Shrink or pause the next action when the current balance cannot support it instead of repeating the full original action until a stop limit ends the run.
- Record a submission as filled from history even when the local log first counted it as failed and correct the running total from that history record.

## Reconcile and report

- Reconcile the local log against history and balances before reporting a total, a stop reason, or a remaining amount.
- Report the verified total, the unverified items still open, and the evidence used for each filled item instead of reporting only the local log total.
- Name the next safe step when verification remains inconclusive and ask the user before issuing a duplicate action that could spend twice.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.
- If nothing here fits, look on the web for a maintained API or MCP server and set it up with Custom Connectors and MCPs, or build a custom connector.

## Alternatives for better results

- Use a unique client identifier for each submission when the service supports one so a later history lookup can find the exact submission.
- Use a smaller test submission first when a new execution path is untested and verify its fill before running a larger sequence.
- Pause the sequence and verify all open submissions when transient errors cluster instead of letting each failure add another duplicate attempt.
