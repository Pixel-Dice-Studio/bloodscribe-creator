---
name: create-bloodscribe-character
description: Create, refine, explain, or validate original BloodScribe Creator characters with the user's own AI. Use when a user describes a character ability, provides character JSON, asks which mechanics represent an idea, or needs a proposal repaired against BloodScribe's public MCP.
---

# Create a BloodScribe character

Use the user's current AI to author the character. The public BloodScribe MCP is only a source of mechanics, recipes, documentation, and deterministic validation; it never provides access to BloodScribe's internal AI.

Reply in the user's language.

## Ability text style

Write `ability` in the output locale using the concise player-facing style shared by Blood on the Clocktower and the Grimm collection. Use idiomatic equivalents rather than literal translations:

| Intent | Español | English |
|---|---|---|
| Repeated night timing | `Cada noche, ...` | `Each night, ...` |
| First-night timing | `La primera noche, ...` | `On the first night, ...` |
| Limited night use | `Una vez por partida, por la noche, ...` | `Once per game, at night, ...` |
| Death trigger | `Cuando mueres, ...` | `When you die, ...` |
| Mandatory choice | `eliges a 1 jugador` | `choose a player` |
| Exclude self | `eliges a 1 jugador (no a ti)` | `choose another player` |
| Definite information | `sabes quién/cuántos/si` | `learn who/how many/whether` |
| Identity change | `pasas a ser` | `you become` |

- Put timing, cadence, or usage first. Address the player directly; never write Storyteller instructions or refer to “the player” / “el jugador”.
- Use digits for Spanish game quantities. In English, prefer idiomatic `a player` for one and concise numeric quantities where natural.
- When a choice, condition, or trigger produces an effect, separate the result with a colon.
- Keep the ability to one or two compact sentences when possible. Remove flavor, implementation vocabulary, redundant subjects, and optional wording when the choice is mandatory.
- Preserve any uncertainty, false information, impairment, limit, exception, duration, identity mode, alignment change, or death condition declared by the mechanics.

Example: `Cada noche, eliges a 1 jugador (no a ti): queda protegido hasta el amanecer.` corresponds to `Each night, choose another player: they are protected until dawn.`

Apply this wording pass after the mechanics are defined and before validation. If a cleaner sentence would change or hide behavior, preserve the precise behavior and ask about the ambiguity instead.

## Workflow

1. Confirm that `get_mechanic_catalog`, `search_mechanic_recipes`, and `validate_character_proposal` are available. Require `guideVersion >= 7` plus the catalog features `contextTemplate`, `proposalTemplate`, `personalVictoryExpressions`, `reminderTokens`, `compiledCharacter`, `validationScope`, `counterThresholds`, and `counterExpressions`. If they are missing, report the incompatible MCP and continue only from a current local BloodScribe contract; otherwise ask the user to update the server/plugin rather than treating obsolete validation as current.
2. If authentication is required, retry one public MCP tool once so the client starts OAuth. Tell the user to sign in and approve access in the BloodScribe browser page that opens. Never ask them to paste a token into the conversation. If the client cannot start OAuth, direct them to the README's **Manual setup** section; any fallback key belongs only in the client's secure credential settings, never in chat.
3. Get the pack context from an attached or exported `.bloodscribe.json`. Start from the catalog's `contextTemplate`; replace its locale, collection, pack metadata, complete existing character IDs, and relevant character types. Do not invent a reduced context shape.
4. Classify the gameplay profile before modeling the ability. If the user's idea does not already answer these points, ask whether the character:
   - enters the regular cast (`entryMode.default: "cast"`), joins as a temporary character (`"temporary"`), or allows both modes with `allowed: ["cast", "temporary"]`;
   - counts in a regular composition role (`core` or `support`) or in the neutral role (`independent`);
   - is aligned with Good, Evil, or Neutral, including allowed alignment changes;
   - wins with its current alignment, with a fixed alignment, or only through a personal condition.
5. Use the catalog's `gameplayProfiles` to explain the behavioral consequences and get the user's choice. Never infer the profile from `teamId`, the character name, or ability prose. Every proposal must use `entryMode`.
   - A temporary entry automatically stays outside cast and endgame living counts, uses expulsion instead of execution for its special removal, and may join during play.
   - A living evil temporary character automatically receives a first night step for that entry showing every living main evil character, never evil helpers or bluffs.
   - These are engine rules. Do not duplicate them with authored mechanics. A dual-mode character gets them only when its effective entry is `temporary`.
   - `role: "independent"` and personal victory describe composition/victory; neither makes a character temporary.
6. Turn the ability into a mechanical brief. Ask only about choices that change behavior: timing, actor, target, identity mode, usage scope or `keyBy`, duration, visibility, invalid or dead sources/targets, coincident triggers, and manual fallback.
7. Call `search_mechanic_recipes` with the user's intent (`limit` is 1–5). Use `get_mechanic_catalog` when recipes do not cover a required primitive, field, or gameplay profile. For multiple mechanics, persistent state, event interception or redirection, granted/copied abilities, altered information, setup changes, or special victory, also read `bloodscribe://pack-builder/authoring/{locale}` when the client exposes MCP resources.
8. Build one complete flat authoring character from `proposalTemplate`; do not send the canonical pack shape with `info`, `gameplay`, or `rules` to the validator. Write `ability` using the style above, then keep `ability`, `howToPlay`, `howToRun`, `interactions`, `cues`, `reminderTokens`, and `mechanics` consistent. Every marker used by `applyMarker` must have a matching reminder token. Use only interaction IDs supplied by the pack context; otherwise return no character-specific interactions. Mechanics must come from declared data, never from the character's ID, name, description, edition, or origin.
9. Call `validate_character_proposal`. On failure, change only the paths listed in `issues` and retry, at most three validation attempts.
10. After structural validation succeeds, perform a semantic audit: map every rules claim in `ability`, `howToPlay`, `howToRun`, `interactions`, and `cues` to the `gameplay` profile, a declared mechanic, or explicit manual coverage. Check blocked or impaired resolution and, when relevant, source/target death or identity change, invalid targets, redirection, and coincident triggers. Fix and revalidate any mismatch.
11. If the user requested artwork, use `$create-bloodscribe-icon` after mechanical validation. Once the user approves the PNG, return the standalone SVG and place its complete markup in `contentCharacter.icon`; if a complete pack is being returned, update the matching character and revalidate the pack. Never overwrite existing artwork without approval.
12. Return the valid authoring JSON, `contentCharacter`, and a short automation summary based on `coverage`. Repeat the returned `validationScope`, name any manual or unresolved behavior, and do not present structural validation as proof of semantic correctness or balance.

## Unsupported intentions

For every intent whose recipe reports `status: "unsupported"`, list its `missingPrimitive` and ask whether the user prefers to redesign it or handle it manually. Do not build or validate JSON until the user has chosen for each unsupported intent. If they accept manual handling, state the exact Storyteller operation and report it as manual coverage. Never invent an executable mechanic or silently weaken the requested behavior.

## Hard rules

- Never call or try to discover `create_character_proposal`; it is not a public MCP tool.
- Never call or imply access to BloodScribe's internal AI.
- Never assume installed Grimm characters, character types, translations, or IDs.
- Never infer mechanics from prose or names.
- Never silently choose identity semantics, targets, usage keys, durations, or hidden information.
- Never silently choose `role`, `allegiance`, `entryMode`, or `victory` when the user's intent leaves them ambiguous.
- Declare participation only through `entryMode`.
- Never author mechanics that recreate temporary entry, evil-main revelation, cast-count exclusion, or expulsion.
- When several markers represent quantities of one resource, use one `adjustCounter` and project it with `copies` or `stages`. Creating separate `Token 1`, `Token 2`, and `Token 3` markers is not recommended; use `stages` only when each level has a genuinely distinct marker identity.
- Never leave a rules claim only in prose or describe structural validation as proof of semantic correctness or balance.
- Character creation does not imply icon generation. Use `$create-bloodscribe-icon` only when the user asks for artwork or the target pack explicitly requires it; an icon failure never blocks a valid mechanical proposal.
- Never store, print, or request MCP tokens or other secrets.
