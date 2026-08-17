---
name: create-bloodscribe-character
description: Create, refine, explain, or validate original BloodScribe Pack Builder characters with the user's own AI. Use when a user describes a character ability, provides character JSON, asks which mechanics represent an idea, or needs a proposal repaired against BloodScribe's public MCP.
---

# Create a BloodScribe character

Use the user's current AI to author the character. The public BloodScribe MCP is only a source of mechanics, recipes, documentation, and deterministic validation; it never provides access to BloodScribe's internal AI.

Reply in the user's language.

## Workflow

1. Confirm that `get_mechanic_catalog`, `search_mechanic_recipes`, and `validate_character_proposal` are available. Use the catalog version to detect an obsolete server.
2. If authentication is required, retry one public MCP tool once so the client starts OAuth. Tell the user to sign in and approve access in the BloodScribe browser page that opens. Never ask them to paste a token into the conversation. If the client cannot start OAuth, direct them to the README's **Manual setup** section; any fallback key belongs only in the client's secure credential settings, never in chat.
3. Get the pack context from an attached or exported `.bloodscribe.json`. If none is available, collect only the locale, collection, existing opaque IDs, and team IDs needed by the character.
4. Turn the idea into a mechanical brief. Ask only about choices that change behavior: timing, actor, target, identity mode, usage scope or `keyBy`, duration, visibility, and manual fallback.
5. Call `search_mechanic_recipes` with the user's intent. Use `get_mechanic_catalog` only when recipes do not cover a required primitive or field.
6. Build one complete character using opaque invented IDs. Mechanics must come from declared data, never from the character's ID, name, description, edition, or origin.
7. Call `validate_character_proposal`. On failure, change only the paths listed in `issues` and retry, at most three validation attempts.
8. Return the valid JSON plus a short automation summary based on `coverage`.

## Unsupported intentions

For every intent whose recipe reports `status: "unsupported"`, list its `missingPrimitive` and ask whether the user prefers to redesign it or handle it manually. Do not build or validate JSON until the user has chosen for each unsupported intent. If they accept manual handling, state the exact Storyteller operation and report it as manual coverage. Never invent an executable mechanic or silently weaken the requested behavior.

## Hard rules

- Never call or try to discover `create_character_proposal`; it is not a public MCP tool.
- Never call or imply access to BloodScribe's internal AI.
- Never assume installed Grimm characters, teams, translations, or IDs.
- Never infer mechanics from prose or names.
- Never silently choose identity semantics, targets, usage keys, durations, or hidden information.
- Never store, print, or request MCP tokens or other secrets.
