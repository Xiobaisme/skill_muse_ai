---
name: preflight-autonomous-timed-execution
description: Use before promising any autonomous action that must run inside a specific time window (claims, mints, scheduled trades). Verifies login, the execution route, and timing end to end first, and reports gaps with a working fallback instead of promising a window that has not been proven.
---

# Preflight Autonomous Timed Execution

## Verify the login first

- Verify the login through the exact sign in path the service uses before you promise any timed work.
- Treat a supplied token or credential as unproven until a live read returns data for the account that will act.
- Treat a wallet sign in as unproven until the wallet is connected and the signature step has completed for that account.
- Check each account separately when more than one account must act and record which accounts passed and which did not.

## Prove the execution route end to end

- Send a harmless end to end test through the exact route the timed action will use before you promise the action.
- Treat a successful read as proof of reading only and never as proof that a write or submission will work.
- Test the exact write method the action needs and record its response or its failure.
- Test from the same environment that will run the timed work and test again when that environment changes.

## Prove the timing path

- Confirm the scheduled run can start inside the window the action needs before you promise an exact hour.
- Treat a test that runs late as a timing failure and say so instead of treating its later success as proof the window works.
- Keep a live check running only after the route and timing checks have passed and stop it when the work is cancelled.
- Record each window with the time it ran, the accounts it covered and the evidence for each result.

## Report the gap and offer the working route

- Tell the user plainly which check failed and what it blocks before the window passes.
- Present setup progress as setup progress only and never as a successful execution.
- Name the route that can still work, such as the user acting from a device where the login already works, and say what the user must do there.
- Stop promising the next window until the failed check passes and ask the user before you keep retrying a route that has failed the same way.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.
- If nothing here fits, look on the web for a maintained API or MCP server and set it up with Custom Connectors and MCPs, or build a custom connector.

## Alternatives for better results

- Ask the user to complete the login once on their own device when the agent cannot sign in for them and then verify the login from that completed session.
- Break a long autonomous run into one verified window first and extend it only after that window succeeds with evidence.
- Use a manual fallback with a clear deadline when the automated route fails so the window is not lost while the route is repaired.
