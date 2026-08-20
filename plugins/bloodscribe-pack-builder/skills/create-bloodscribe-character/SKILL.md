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
3. Get the pack context from an attached or exported `.bloodscribe.json`. If none is available, collect only the locale, collection, existing opaque IDs, and each relevant team's ID plus its `role`, `allegiance`, `entryMode`, and `victory` defaults.
4. Classify the gameplay profile before modeling the ability. If the user's idea does not already answer these points, ask whether the character:
   - enters the regular cast (`entryMode.default: "cast"`), joins as a temporary character (`"temporary"`), or allows both modes with `allowed: ["cast", "temporary"]`;
   - counts in a regular composition role (`core` or `support`) or in the neutral role (`independent`);
   - is good, evil, or neutral, including allowed alignment changes;
   - wins with its current alignment, with a fixed side, or only through a personal condition.
5. Use the catalog's `gameplayProfiles` to explain the behavioral consequences and get the user's choice. Never infer the profile from `teamId`, the character name, or ability prose. Every proposal must use `entryMode`.
   - A temporary entry automatically stays outside cast and endgame living counts, uses expulsion instead of execution for its special removal, and may join during play.
   - A living evil temporary character automatically receives a first night step for that entry showing every living main evil character, never evil helpers or bluffs.
   - These are engine rules. Do not duplicate them with authored mechanics. A dual-mode character gets them only when its effective entry is `temporary`.
   - `role: "independent"` and personal victory describe composition/victory; neither makes a character temporary.
6. Turn the ability into a mechanical brief. Ask only about choices that change behavior: timing, actor, target, identity mode, usage scope or `keyBy`, duration, visibility, and manual fallback.
7. Call `search_mechanic_recipes` with the user's intent. Use `get_mechanic_catalog` when recipes do not cover a required primitive, field, or gameplay profile.
8. Build one complete character using opaque invented IDs and a complete `gameplay` profile. Mechanics must come from declared data, never from the character's ID, name, description, edition, or origin.
9. Call `validate_character_proposal`. On failure, change only the paths listed in `issues` and retry, at most three validation attempts.
10. Return the valid JSON plus a short automation summary based on `coverage`.

## Unsupported intentions

For every intent whose recipe reports `status: "unsupported"`, list its `missingPrimitive` and ask whether the user prefers to redesign it or handle it manually. Do not build or validate JSON until the user has chosen for each unsupported intent. If they accept manual handling, state the exact Storyteller operation and report it as manual coverage. Never invent an executable mechanic or silently weaken the requested behavior.

## Hard rules

- Never call or try to discover `create_character_proposal`; it is not a public MCP tool.
- Never call or imply access to BloodScribe's internal AI.
- Never assume installed Grimm characters, teams, translations, or IDs.
- Never infer mechanics from prose or names.
- Never silently choose identity semantics, targets, usage keys, durations, or hidden information.
- Never silently choose `role`, `allegiance`, `entryMode`, or `victory` when the user's intent leaves them ambiguous.
- Declare participation only through `entryMode`.
- Never author mechanics that recreate temporary entry, evil-main revelation, cast-count exclusion, or expulsion.
- Never store, print, or request MCP tokens or other secrets.
