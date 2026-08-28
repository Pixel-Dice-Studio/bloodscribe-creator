---
name: create-bloodscribe-tale
description: Create, refine, or modify a BloodScribe tale using characters and rules from a complete pack. Use when composing a new tale, changing its cast, or adding, replacing, or removing composition, voting, setup, game-end, general, or modifier rules.
---

# Create or modify a BloodScribe tale

Use the user's current AI to compose the tale. The public BloodScribe MCP supplies deterministic contracts and validation only; it never calls BloodScribe's internal AI.

Reply in the user's language.

## Workflow

1. Confirm `get_mechanic_catalog`, `search_rule_recipes`, and `validate_content_pack_proposal` are available. Require MCP `>= 2.2.0`, mechanic `guideVersion >= 7`, `ruleAuthoring.guideVersion >= 2`, and the catalog features `contentPackTemplate`, `taleAuthoring`, `contentPackValidation`, `counterExpressions`, and `tallyElements`.
2. If authentication is required, retry one public tool once to start OAuth. Ask the user to approve the browser flow; never request or print a token.
3. Start from the complete attached/exported `.bloodscribe.json`; otherwise clone `contentPackTemplate`.
4. Decide whether to create a new tale or modify an existing one. Use only characters, teams, and rules present in that pack. Preserve opaque IDs and existing references unless the user explicitly replaces them.
5. Read `bloodscribe://pack-builder/tales/{locale}` and apply these invariants:
   - exactly one composition binding, `automatic`, with no `activeWhen`;
   - at most one voting rule through `votingRuleId`;
   - replace rules in categories whose `selection.maxActive` would be exceeded;
   - mandatory rules use `automatic`; optional modifiers use `setupChoice`.
6. Keep `books[].taleIds`, `characterIds`, `gameRuleIds`, and `teamIds` aligned with the tale. If a new rule is required, follow the `create-bloodscribe-rule` contract and call `search_rule_recipes` before adding it.
7. If a required mechanic is unsupported, return the complete mechanic-gap specification and do not add fictional executable behavior.
8. Call `validate_content_pack_proposal`, fix only reported paths, and retry at most three times.
9. Reconcile every rules claim in the tale and its linked rules with declared mechanics or explicit manual coverage. Do not present validation as proof of semantic equivalence or balance.
10. If the user requested artwork for characters in the tale, use `$create-bloodscribe-icon` after rules validation. Once each PNG is approved, return the standalone SVGs, place their complete markup in the matching `characters[].icon`, and revalidate the complete pack. Never overwrite existing artwork without approval.
11. Return the complete validated `.bloodscribe.json` and a concise summary of created, retained, replaced, and optional rules.

## Hard rules

- Never call or imply access to BloodScribe's internal AI.
- Never assume preinstalled characters, rules, teams, translations, or IDs.
- Never create behavior branches from a content ID, name, text, edition, or origin.
- Never duplicate system default rule definitions inside the pack; reference their canonical IDs.
- To publish a shared counter total, add one tale element with `kind: "tally"`, `counter`, and optional `remainderCounter`. Do not create separate `Token 1`, `Token 2`, and `Token 3` elements.
- Never leave unresolved character or rule references.
- Never add a missing primitive to the core from this workflow.
- Tale creation does not imply icon generation; only generate character SVGs when the user requests artwork or the target pack requires it.
- Never store, print, or request secrets.
