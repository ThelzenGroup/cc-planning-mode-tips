Contributions that improve or expand this playbook are very welcome.

## Quality bar

**A good tip:**
- Follows the 4-part template (Why & when / Example / How to apply / Pitfalls)
- Is specific to Claude Code's planning mode (not general Claude usage)
- Includes a real, concrete example (not a placeholder)
- Addresses a distinct insight not covered by existing tips
- Uses second-person voice and imperative headlines

**We won't accept:**
- General Claude tips that aren't planning-mode-specific
- Tips without concrete examples
- Near-duplicates of existing tips (different words, same insight)
- Tips aimed at beginners (this repo targets intermediate users)
- Placeholder content or incomplete submissions

## Style

Write in second person, imperative voice. Lead with the point.

❌ "It might be worth considering that Claude could potentially benefit from more context about why you're making the change."
✅ "Tell Claude why you're making the change, not just what you want."

❌ "In many cases, it can be helpful to..."
✅ "When X, do Y."

Keep "Why & when" to one sentence. "How to apply" must contain at least one action the reader can execute today.

## How to submit

**For new tips:** We recommend opening an issue first with your tip idea or draft. This takes 2 minutes and could save you rewriting a full tip if it overlaps with existing content or is out of scope. In the issue, include:
- The tip headline (imperative form)
- A one-sentence summary of what it teaches
- Which existing tips it's most similar to and how it differs

This lets us give you quick feedback before you invest time in a full write-up.

**For improvements to existing tips:** Open a PR directly with your changes. Improvements include: adding better examples, clarifying confusing language, fixing errors, or expanding the Pitfalls section with real failure modes you've encountered.

**All PRs must include a differentiation statement** (see PR template). This is required — PRs without it will not be reviewed. The differentiation statement should name the 1-3 most similar existing tips and explain the specific difference in insight or actionable guidance. Example:

> Most similar existing tips: "Give Claude the why behind the task"
> This tip differs because: it focuses on negative constraints (what NOT to do) rather than positive intent

PRs are reviewed against the quality bar above. Expect feedback on: whether the tip is truly distinct, whether the example is concrete enough, and whether the "How to apply" section gives actionable guidance. We aim to review PRs within 3-5 days.

## Checking for near-duplicates

Before writing your tip, answer this question: **What does this tip tell the reader to do that no existing tip tells them to do?**

If you can't answer this specifically, your tip may be a near-duplicate.

Near-duplicates are the most common reason for rejection. They happen when two tips use different words but give the reader the same mental model and the same actionable guidance. The test isn't "do these tips sound different?" — it's "would a reader who follows tip A do anything differently than a reader who follows tip B?"

**Example of a near-duplicate (would be rejected):**
- Existing tip: "Give Claude the why behind the task"
- Proposed tip: "Explain your reasoning to Claude"
- Problem: Different words, same insight, same action. Both tips tell the reader to provide intent/reasoning in their prompt. A reader who has internalized the first tip gains nothing from the second.

**Example of distinct tips (would be accepted):**
- Tip A: "Give Claude the why behind the task" (about intent)
- Tip B: "Name the exact files involved" (about scope)
- Why it works: Both improve plan prompts, but target completely different failure modes. A reader could apply both tips to the same prompt and get value from each. The actions are distinct: one adds reasoning, the other adds file paths.

When in doubt, read the existing tips in your target category carefully. If your tip feels like a variation or restatement of an existing one, it probably is.

## Copy-paste template

```markdown
### [Imperative headline — e.g. "Give Claude the why behind the task"]

**Why & when it matters:**
[1 sentence — the core mental model or condition]

**Example:**
[For prompting tips: ❌ weak version then ✅ strong version]
[For workflow tips: 3-5 sentence concrete scenario]

**How to apply it:**
[3-6 sentences of concrete, actionable guidance]

**Pitfalls:**
[2-4 sentences on edge cases and when NOT to follow this]
```

## Housekeeping

**Tip count:** When your PR adds a new tip, update the tip count in the README header (the "N battle-tested tips" line) in the same PR.

**License:** All contributions are licensed under MIT.
