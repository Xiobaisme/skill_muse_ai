---
name: multi-model-browser-pipeline
description: Use when stuck on a bug or problem and a second opinion is needed. Consults ChatGPT, Claude and Gemini in order through their browser chats (analysis, review, alternatives), with the bug already found and evidenced locally, then combines the findings and executes the plan.
---

# Multi Model Browser Pipeline

## Consult the models in order

- Start with the first model for analysis when you are stuck or cannot find a cause.
- Ask the second model to review that analysis for errors and gaps.
- Ask the third model for alternative solutions the first two did not cover.
- Combine the three findings yourself and execute the combined plan.
- Do not ask the user for approval for each prompt when the user has already approved the pipeline for the task.

## Validate found bugs, don't prompt basic discovery

- Find the bug first yourself (code + deterministic test), then prompt the models to VALIDATE it.
- Carry the full evidence into each prompt: root cause, exact code excerpts, the failing test, and its output.
- Never send a generic/basic "find bugs" ideation prompt; ungrounded hypotheses waste the round.
- A model's validation never replaces local verification: re-run the test and re-check duplicates yourself before reporting.

## Use browser chat only

- Prompt each model in its own browser chat app as a regular chat user.
- Do not use a command line tool for the model when the browser chat route is available.
- Keep the command line route parked unless the user explicitly orders it.
- Carry the exact problem, the evidence you already have and the question for that model into each browser task.
- Record what each model answered before you move to the next model.

## Check the account before signing in

- Check which account each model chat is signed in as before you prompt it.
- Check the account rule recorded in memory for that model before you sign in anywhere.
- Never sign a model in with an account reserved for a different model.
- Stop and correct the sign in at once when you find a model on the wrong account.
- Do not complete a sign in that would mix accounts even when the sign in screen offers it.

## Handle a blocked sign in plainly

- Treat a captcha that never clears as a blocked sign in and not as a prompt to keep trying.
- Stop after the captcha stalls, report the block plainly and continue with the models that are signed in.
- Skip the blocked model for that round when the user orders the skip.
- Do not ask the user for a login or a code that the login tools were given to handle.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.
- If nothing here fits, look on the web for a maintained API or MCP server and set it up with Custom Connectors and MCPs, or build a custom connector.

## Websites

- [ChatGPT](https://chatgpt.com/) for the first analysis. Carry the problem and the evidence into the browser task and verify the signed in account before prompting.
- [Claude](https://claude.ai/) for the review of that analysis. Carry the analysis into the browser task and verify the signed in account before prompting.
- [Gemini](https://gemini.google.com/) for alternative solutions. Carry the problem and the earlier findings into the browser task and verify the signed in account before prompting.

## Alternatives for better results

- Use one model alone when the question is narrow and a full pipeline would add no new view.
- Ask the user which job a new model would do before designing a bridge to it.
- Save the combined finding and the source for each model answer so a later session can check the reasoning.
