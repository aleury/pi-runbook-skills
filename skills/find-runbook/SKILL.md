---
name: find-runbook
description: Find the best matching runbook Markdown file for a stacktrace, exception, service error, incident description, or failure symptoms. Use codemode and TypeSafe Jev when available to rank candidate runbooks.
---

# Find Runbook

Use this skill when the user asks to find, choose, rank, or recommend a runbook for an error, stacktrace, log excerpt, alert, incident, or failure description.

## Workflow

1. Identify likely runbook Markdown files. Search only dedicated runbook locations by default:
   - `runbooks/**/*.md`
   - `docs/runbooks/**/*.md`
2. Do not include general documentation such as `docs/**/*.md`, plans, notes, or skill references unless the user explicitly asks to search broader docs or no dedicated runbook files exist.
3. Use filenames, headings, and keyword matches from the stacktrace or error description to narrow candidates.
4. Prefer codemode for this workflow so candidate discovery, snippet extraction, filtering, and classification can be batched in one script.
5. If TypeSafe Jev is available, use it to rank the narrowed candidate set.
6. If Jev is unavailable, rank candidates using direct evidence from filenames, headings, and matching snippets.
7. Do not send secrets, credentials, private tokens, or unnecessarily large logs to external classifier models. Minimize the state passed to Jev to the error summary and candidate metadata/snippets.

## Suggested codemode pattern

Use `find runbooks docs/runbooks -type f -name '*.md' 2>/dev/null` through `tools.bash` to discover Markdown files, then read only likely candidates. Keep snippets concise. Do not search general `docs/` by default.

When Jev is available, call it like this:

```js
const jev = await models.getModelOfType("classifier", "typesafe", "jev-latest");

const result = await models.classify(jev, {
  state: {
    error: "<stacktrace or incident description>",
    candidates: [
      {
        id: "candidate-1",
        path: "runbooks/example.md",
        title: "Example Runbook",
        excerpt: "Relevant headings and snippets only"
      }
    ]
  },
  questions: {
    bestRunbook: {
      type: "choice",
      instructions: "Choose the single runbook that best matches the error. Choose none if no candidate is relevant.",
      criteria: {
        "candidate-1": "This candidate is the best match",
        "none": "No candidate is relevant enough to recommend"
      }
    }
  }
});

return result.answers;
```

Use the selected choice's probability from `bestRunbook.probabilities` as the primary confidence signal. Do not add a separate Jev `score` question unless the user specifically asks for a classifier score; score outputs may not be calibrated to a 0-1 range.

If there are more candidates than fit comfortably, do a first-pass lexical filter and classify only the strongest 5-15 candidates.

## Response format

Return a concise result:

- **Best runbook:** path, or `none`
- **Confidence:** low/medium/high, preferably based on Jev's selected-choice probability when available
- **Why:** short rationale tied to the error and runbook evidence
- **Matching evidence:** bullets with relevant filename/heading/snippet matches
- **Next step:** suggest reading or following the selected runbook

If no good match exists, say so clearly and list the closest candidates separately.
