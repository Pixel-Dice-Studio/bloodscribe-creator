---
name: create-bloodscribe-rule
description: Create, refine, explain, or validate BloodScribe game rules and link them to a tale. Use for composition, voting, setup, game-end, general, Blessing, Curse, Fabled, Loric, or other modifier rules; return a complete validated pack when one is provided.
---

# Create a BloodScribe rule

Use the user's current AI to author the rule. The public BloodScribe MCP only supplies deterministic knowledge and validation; it never calls BloodScribe's internal AI.

Reply in the user's language.

## Workflow

1. Confirm `get_mechanic_catalog`, `search_rule_recipes`, and `validate_content_pack_proposal` are available. Require MCP `>= 2.2.0`, mechanic `guideVersion >= 7`, `ruleAuthoring.guideVersion >= 2`, and the catalog features `contentPackTemplate`, `contentPackValidation`, `mechanicGapSpecification`, `counterThresholds`, and `counterExpressions`.
2. If authentication is required, retry one public tool once to start OAuth. Ask the user to approve the browser flow; never request or print a token.
3. Start from the complete attached/exported `.bloodscribe.json`. If none exists, clone `contentPackTemplate`; never invent a reduced pack shape.
4. Classify before modeling:
   - System rule or Modifier;
   - `ruleKind`: `general`, `composition`, `voting`, `setup`, or `gameEnd`;
   - mandatory `automatic` or optional `setupChoice` activation;
   - affected tale, persistent state, timing, visibility, and interaction with an existing exclusive rule.
5. Call `search_rule_recipes`. Use `get_mechanic_catalog` and `bloodscribe://pack-builder/rules/{locale}` for fields and primitives.
6. Add the rule to `gameRules[]` with opaque IDs and declarative mechanics. Never select behavior from its ID, name, text, collection, or origin.
7. Link it correctly:
   - composition replaces the previous composition and is the only automatic unconditional composition binding;
   - voting replaces `votingRuleId` and is never added to `gameRuleBindings`;
   - a game-end rule replaces another rule in its exclusive category;
   - other mandatory or optional rules use `gameRuleBindings` with `automatic` or `setupChoice`.
8. Keep the owning Book's `gameRuleIds` aligned, then call `validate_content_pack_proposal`. Fix only reported paths and retry at most three times.
9. Audit every claim in `ability`, `howToPlay`, and `howToRun` against structural fields, mechanics, or explicit manual coverage. Validation proves schema, importability, references, and rule selection; it does not prove semantic equivalence or balance.
10. Return the complete validated `.bloodscribe.json` plus a short automation summary.

## Missing primitives

If no supported recipe or catalog primitive represents the intention, do not add executable rule data. Return a mechanic gap containing exactly: `intent`, `missingPrimitive`, `requiredState`, `trigger`, `inputs`, `outputs`, `edgeCases`, `manualFallback`, and `acceptanceScenarios`. Ask whether the user wants to redesign or keep the explicit manual fallback.

## Hard rules

- Never call or imply access to BloodScribe's internal AI.
- Never assume installed Grimm, BOTC, character-type, character, rule, translation, or icon data.
- Never leave behavior only in prose; explicit `source.modeling.kind: "narrator-text"` is required for intentionally manual rules.
- Never bind voting through `gameRuleBindings` or keep two composition bindings.
- Never add a missing primitive to the core or fake it with content-specific IDs.
- When several markers represent quantities of one resource, use one `adjustCounter` and project it with `copies` or `stages`. Creating separate `Token 1`, `Token 2`, and `Token 3` markers is not recommended; use `stages` only when each level has a genuinely distinct marker identity.
- Rule creation does not imply icon generation.
- Never store, print, or request secrets.
