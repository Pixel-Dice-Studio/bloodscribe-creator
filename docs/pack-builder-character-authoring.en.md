# BloodScribe mechanic authoring guide

This reference is generated from the same contract used by the MCP. Examples use invented IDs.

## Authoring rules

- IDs are opaque references; they never select behavior.
- Classify participation, alignment, role, and victory first; never infer them from teamId, names, or prose.
- Declare participation exclusively with entryMode: cast, temporary, or both.
- A temporary entry automatically enables cast exclusion, evil night information, and expulsion; do not duplicate those rules as mechanics.
- Express every rule through when, input, usage, conditions, effects, and policies.
- Search recipes before composing complex primitives and always validate the final character.
- When several markers represent quantities of the same resource, use one adjustCounter and project it with copies or stages; creating separate Token 1, Token 2, and Token 3 markers is not recommended.
- Use keyBy when a limit combines day, night, actor, target, or triggering event.
- Use a typed duration, including untilEvent, instead of inferring duration from prose.
- A personal victory is evaluated when the game ends and may use any boolean ValueExpr; do not add resolveGameEnd merely to add that winner.
- Before delivery, verify that every rules claim in ability, howToPlay, howToRun, interactions, and cues is backed by gameplay, a declared mechanic, or explicit manual coverage; validation guarantees only contract and importability.
- manualInstruction and requiresManualModeling are last resorts and must name the missing primitive.
- Do not assume installed content; use invented IDs or IDs supplied by the context.

## Participation and victory profile

Before modeling the ability, classify how the character enters, counts, and wins. If the idea does not make this clear, the AI must ask. `teamId` only groups and presents content; it never replaces these fields. Every proposal uses `entryMode`.

### `regular-aligned` — Regular aligned character

Enters the bag, occupies a regular slot, and wins with its effective alignment.

- Counts among regular living players.
- Uses normal death flows and cannot be expelled as temporary.

```json
{
  "gameplay": {
    "role": "core",
    "allegiance": {
      "default": "good",
      "allowed": [
        "good"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast"
      ]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  }
}
```

### `temporary-character` — Temporary character

Enters outside the base cast, receives a secret alignment, and uses expulsion.

- Does not occupy a base-cast slot or count toward the standard two-alive ending.
- May join during play and is eligible for expulsion.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral",
        "good",
        "evil"
      ]
    },
    "entryMode": {
      "default": "temporary",
      "allowed": [
        "temporary"
      ]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  }
}
```

### `regular-neutral-personal` — Regular neutral with personal victory

Belongs to the regular cast and wins only when its declarative condition is met.

- Occupies the neutral composition axis and counts among regular living players.
- Does not automatically win with good or evil.
- storytellerDecision is only the profile's manual fallback; replace it with a boolean ValueExpr when the condition can be automated.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast"
      ]
    },
    "victory": {
      "type": "personal",
      "condition": {
        "type": "storytellerDecision",
        "decision": "resolveCharacterPersonalVictory"
      }
    }
  }
}
```

### `regular-neutral-fixed-side` — Regular neutral that wins with a fixed side

Remains neutral for identity and composition but shares one fixed side's victory.

- Counts among regular living players.
- Wins with the fixed side even while its alignment remains neutral.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast"
      ]
    },
    "victory": {
      "type": "withSide",
      "side": "good"
    }
  }
}
```

### `flexible-entry` — Flexible entry

The Storyteller decides whether the same definition enters the cast or as a temporary character.

- With cast it counts and dies as part of the regular cast.
- With temporary it stays outside the regular count and uses expulsion.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral",
        "good",
        "evil"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast",
        "temporary"
      ]
    },
    "victory": {
      "type": "personal",
      "condition": {
        "type": "storytellerDecision",
        "decision": "resolveCharacterPersonalVictory"
      }
    }
  }
}
```

## Complete gameplay examples

### `character:invented:night-envoy` — Evil temporary character joining during the day

1. It joins during the day through add-temporary-player. The immediate reveal shows character and alignment, but not main evil players.
2. On the next night, its first step separately shows every living main evil player. It excludes helpers and bluffs, then its own ability acts.
3. An approved temporaryExpulsion vote produces expulsion and death. Living and dead players may vote, no last breath is spent, and an execution may still happen that day.

```json
{
  "id": "character:invented:night-envoy",
  "name": "Enviado nocturno",
  "teamId": "team:invented:visitors",
  "gameplay": {
    "role": "support",
    "allegiance": {
      "default": "evil",
      "allowed": [
        "good",
        "evil"
      ]
    },
    "entryMode": {
      "default": "temporary",
      "allowed": [
        "temporary"
      ]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  },
  "summaryEs": "Cada noche, elige un jugador y aprende si está vivo."
}
```

### `character:invented:two-doors` — Dual-entry character

1. When entered as cast, it occupies a cast slot, counts among regular living players, uses ordinary death flows, and cannot be expelled as temporary.
2. When entered as temporary, it stays outside those counts, may join during play, and uses expulsion. Only this entry receives automatic evil information when evil.

```json
{
  "id": "character:invented:two-doors",
  "name": "Dos Puertas",
  "teamId": "team:invented:neutral",
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral",
        "good",
        "evil"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast",
        "temporary"
      ]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  },
  "summaryEs": "Una vez por partida, elige un jugador: aprende su alineamiento."
}
```

### `character:invented:common-mediator` — Neutral cast character that counts normally

1. Even with independent role and neutral allegiance, cast makes it occupy the cast, count for game end, and use normal death and execution.

```json
{
  "id": "character:invented:common-mediator",
  "name": "Mediador común",
  "teamId": "team:invented:neutral",
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast"
      ]
    },
    "victory": {
      "type": "withSide",
      "side": "good"
    }
  },
  "summaryEs": "Cada día, puede hacer una pregunta pública al Narrador."
}
```

### `character:invented:solitary-oath` — Independent character with personal victory

1. It counts as a regular player because it enters as cast, but it does not automatically share either side's victory: it wins only when its personal condition is satisfied.

```json
{
  "id": "character:invented:solitary-oath",
  "name": "Juramento solitario",
  "teamId": "team:invented:neutral",
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": [
        "neutral"
      ]
    },
    "entryMode": {
      "default": "cast",
      "allowed": [
        "cast"
      ]
    },
    "victory": {
      "type": "personal",
      "condition": {
        "type": "storytellerDecision",
        "decision": "inventedOathFulfilled"
      }
    }
  },
  "summaryEs": "Ganas solo si el Narrador confirma que cumpliste tu juramento público."
}
```

## Design questions

- Does the character enter the regular cast, as a temporary character, or through either mode?
- Does the character count as regular for composition and game end, or as an extra participant outside that count?
- Is its alignment good, evil, or neutral, and which alignments may it change to?
- Does it win with its current alignment, with a fixed side, or only through a personal condition?
- Is its composition role core, support, or independent?
- What observable result must the ability produce?
- When does it trigger and how often?
- Who acts, who may be targeted, and may dead players or the actor be selected?
- Should it inspect real, initial, shown, or registered identity?
- Which dimensions share the limit: day, night, actor, target, or triggering event?
- When does any persistent state, restriction, or marker end?
- What should happen if the ability fails, is blocked, redirected, or impaired?
- What happens if the source or target dies, changes identity, or becomes invalid, or if several triggers coincide?
- Which part, if any, requires an explicit Storyteller decision?

## Intent recipes

### `personal-victory-by-fatal-execution-before-night` — Personal victory by execution before a night

Win personally if the character dies by execution before a night deadline, regardless of the winning side.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.execution`, `eventFields.died`, `gameProperties.nightNumber`, `effects.applyMarker`, `valueNodes.query`

An execution trigger with died=true applies a permanent marker before the deadline; personal victory queries that marker when the game ends. It does not need resolveGameEnd.

**Ask**

- Which night is the exclusive deadline?
- Should an execution count when the character does not die?

**Limits**

- nightNumber starts at 1 and remains N during the day after night N; use nightNumber < 5 for “before the 5th night”.

```json
{
  "mechanicId": "mechanic:invented:personal-execution:rule:1",
  "tags": [
    "execution-tracking",
    "personal-victory"
  ],
  "when": {
    "window": "anyTime",
    "cadence": "each",
    "trigger": {
      "type": "event",
      "event": "execution",
      "bindings": {
        "eventSubject": "actor"
      },
      "where": {
        "type": "eventField",
        "field": "died",
        "value": true
      }
    }
  },
  "input": {
    "kind": "none"
  },
  "conditions": [
    {
      "type": "compare",
      "left": {
        "type": "game",
        "property": "nightNumber"
      },
      "operator": "lt",
      "right": 5
    }
  ],
  "effects": [
    {
      "type": "applyMarker",
      "kind": "reminder",
      "id": "personal-condition-met",
      "active": true,
      "targets": {
        "type": "binding",
        "binding": "actor"
      },
      "duration": {
        "type": "permanent"
      }
    }
  ],
  "policies": []
}
```

### `registered-evil-neighbours` — Check evil neighbours

Learn whether at least one of the two nearest living neighbours registers as evil.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.players`, `valueNodes.query`, `identityModes.registered`, `effects.emitInformation`

nearestMatching finds one living neighbour in each direction; the query projects registered allegiance and emitInformation delivers a boolean.

**Ask**

- Should dead neighbours be skipped?
- Should the ability inspect real or registered allegiance?

```json
{
  "mechanicId": "mechanic:invented:neighbour-reader:rule:1",
  "tags": [
    "night-information",
    "registration-information"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "compare",
        "left": {
          "type": "query",
          "from": {
            "entity": "players",
            "relation": {
              "type": "nearestMatching",
              "anchor": "actor",
              "directions": [
                "clockwise",
                "counterclockwise"
              ],
              "limitPerDirection": 1,
              "where": {
                "type": "alive",
                "value": true
              }
            }
          },
          "project": {
            "type": "identity",
            "identityMode": "registered",
            "property": "allegiance"
          },
          "aggregate": {
            "type": "collect"
          }
        },
        "operator": "includes",
        "right": "evil"
      },
      "presentation": {
        "kind": "boolean",
        "title": "Mal cercano"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `once-per-actor-each-day` — Once per actor each day

Allow one independent resolution for each actor on each day.

- Status: `supported`
- Automation: `automatic`
- Covers: `usageDimensions.day`, `usageDimensions.actor`, `effects.recordAction`

keyBy composes the day and actor dimensions into one usage key.

**Ask**

- Is usage consumed on attempt, resolution, or only on success?

```json
{
  "mechanicId": "mechanic:invented:daily-actor-action:rule:1",
  "tags": [
    "public-day-action"
  ],
  "when": {
    "window": "day",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "keyBy": [
      "day",
      "actor"
    ],
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "recordAction",
      "actionId": "invented-daily-action",
      "outcome": {
        "type": "literal",
        "value": "resolved"
      }
    }
  ],
  "policies": []
}
```

### `state-until-nomination` — State until a nomination

Apply a state that automatically ends when a nomination occurs.

- Status: `supported`
- Automation: `automatic`
- Covers: `durations.untilEvent`, `events.nomination`, `effects.setPlayerState`

The untilEvent duration stores the typed event pattern that removes the state.

**Ask**

- Should any nomination end it, or only one involving the target?

**Limits**

- This example ends on any nomination.

```json
{
  "mechanicId": "mechanic:invented:temporary-silence:rule:1",
  "tags": [
    "temporary-state"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "setPlayerState",
      "state": "invented-silenced",
      "active": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilEvent",
        "event": "nomination"
      }
    }
  ],
  "policies": []
}
```

### `learn-marker-source` — Learn a marker's source

Learn which player originated a particular reminder marker.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.markers`, `valueNodes.query`, `effects.applyMarker`, `effects.emitInformation`

Markers retain sourcePlayerId and sourceCharacterId, which a query can project.

**Ask**

- Does the marker remain source-owned or become independent?

**Limits**

- If several identical markers exist, add a condition that makes the intended one unambiguous.

```json
{
  "mechanicId": "mechanic:invented:marker-source-reader:rule:1",
  "tags": [
    "marker-information"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "markers"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "markerId",
              "values": [
                "invented-curse"
              ]
            },
            {
              "type": "markerActive",
              "value": true
            }
          ]
        },
        "project": {
          "type": "marker",
          "property": "sourcePlayerId"
        },
        "aggregate": {
          "type": "first"
        }
      },
      "presentation": {
        "kind": "player",
        "title": "Origen de la ficha"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `atomic-marker-transfer` — Atomically transfer a marker

Move a marker from the actor to a target while preserving identity, provenance, and ownership.

- Status: `supported`
- Automation: `automatic`
- Covers: `effects.moveMarker`, `events.markerChange`

moveMarker emits one change and preserves metadata from the existing marker.

**Ask**

- Who is the marker moved from, and may it be absent?

**Limits**

- The marker must exist exactly once on the source.

```json
{
  "mechanicId": "mechanic:invented:marker-transfer:rule:1",
  "tags": [
    "marker-transfer"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "moveMarker",
      "kind": "reminder",
      "id": "invented-object",
      "from": {
        "type": "binding",
        "binding": "actor"
      },
      "targets": {
        "type": "binding",
        "binding": "selected"
      }
    }
  ],
  "policies": []
}
```

### `learn-impaired-ability-user` — Find an impaired ability

Learn who used an ability that malfunctioned during the night.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.events`, `eventFields.actorAbilityMode`, `events.mechanicUse`, `effects.emitInformation`

The mechanicUse history captures actorAbilityMode and the actor participant at resolution time.

**Ask**

- Does a disabled ability count, or only a malfunctioning one?

```json
{
  "mechanicId": "mechanic:invented:impaired-user-reader:rule:1",
  "tags": [
    "activity-tracking"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "events"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "eventType",
              "values": [
                "mechanicUse"
              ]
            },
            {
              "type": "eventPeriod",
              "value": "currentNight"
            },
            {
              "type": "eventField",
              "field": "actorAbilityMode",
              "value": "malfunction"
            }
          ]
        },
        "project": {
          "type": "eventParticipant",
          "binding": "actor"
        },
        "aggregate": {
          "type": "first"
        }
      },
      "presentation": {
        "kind": "player",
        "title": "Habilidad perjudicada"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `learn-protected-target` — Learn who was protected

Identify a target whose resolution was prevented by protection.

- Status: `supported`
- Automation: `automatic`
- Covers: `eventFields.targetResultStatus`, `events.mechanicUse`, `effects.emitInformation`

targetResultStatus filters prevented resolutions and returns the captured selected target.

**Ask**

- Should protection, immunity, and invalid targets be distinguished?

```json
{
  "mechanicId": "mechanic:invented:protected-target-reader:rule:1",
  "tags": [
    "activity-tracking",
    "protection-information"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "events"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "eventType",
              "values": [
                "mechanicUse"
              ]
            },
            {
              "type": "eventPeriod",
              "value": "currentNight"
            },
            {
              "type": "eventField",
              "field": "targetResultStatus",
              "value": "prevented"
            }
          ]
        },
        "project": {
          "type": "eventParticipant",
          "binding": "selected"
        },
        "aggregate": {
          "type": "first"
        }
      },
      "presentation": {
        "kind": "player",
        "title": "Objetivo protegido"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `learn-copied-ability-user` — Find a copied ability

Learn who used a copied ability during the night.

- Status: `supported`
- Automation: `automatic`
- Covers: `eventFields.abilityProvenance`, `events.mechanicUse`, `effects.emitInformation`

abilityProvenance retains the provenance of the ability instance that was used.

**Ask**

- Do granted or borrowed abilities also count?

```json
{
  "mechanicId": "mechanic:invented:copied-user-reader:rule:1",
  "tags": [
    "activity-tracking",
    "ability-provenance"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "events"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "eventType",
              "values": [
                "mechanicUse"
              ]
            },
            {
              "type": "eventPeriod",
              "value": "currentNight"
            },
            {
              "type": "eventField",
              "field": "abilityProvenance",
              "value": "copied"
            }
          ]
        },
        "project": {
          "type": "eventParticipant",
          "binding": "actor"
        },
        "aggregate": {
          "type": "first"
        }
      },
      "presentation": {
        "kind": "player",
        "title": "Habilidad copiada"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `first-successful-ability-each-night` — First successful ability each night

React once to the first ability that succeeds each night.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.mechanicUse`, `eventFields.resolutionStatus`, `usageDimensions.night`

The trigger filters successful mechanicUse events and keyBy night limits the reaction to once per night.

**Ask**

- Should the reaction consume on resolution or only when its own effect succeeds?

```json
{
  "mechanicId": "mechanic:invented:first-night-success:rule:1",
  "tags": [
    "ability-reaction"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "trigger": {
      "type": "event",
      "event": "mechanicUse",
      "where": {
        "type": "eventField",
        "field": "resolutionStatus",
        "value": "succeeded"
      }
    }
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "keyBy": [
      "night"
    ],
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "binding",
        "binding": "eventSubject"
      },
      "presentation": {
        "kind": "player",
        "title": "Primera habilidad exitosa"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `react-to-redirected-ability` — React to a redirected ability

Trigger a reaction when an ability resolution was redirected.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.mechanicUse`, `eventFields.targetResultStatus`, `effects.recordAction`

The trigger filters mechanicUse events whose target result is redirected.

**Ask**

- Should it react to every redirection or only to a specific ability?

**Limits**

- If a resolution has mixed target results, narrow it by category or mechanic.

```json
{
  "mechanicId": "mechanic:invented:redirect-reaction:rule:1",
  "tags": [
    "ability-reaction"
  ],
  "when": {
    "window": "anyTime",
    "cadence": "each",
    "trigger": {
      "type": "event",
      "event": "mechanicUse",
      "where": {
        "type": "eventField",
        "field": "targetResultStatus",
        "value": "redirected"
      }
    }
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "recordAction",
      "actionId": "invented-redirect-reaction",
      "outcome": {
        "type": "literal",
        "value": "redirected"
      }
    }
  ],
  "policies": []
}
```

### `react-after-failed-ability` — React after a failed ability

Trigger a reaction after an ability finishes with failed status.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.mechanicUse`, `eventFields.resolutionStatus`, `effects.recordAction`

resolutionStatus distinguishes complete failure from partial results such as noEffect.

**Ask**

- Should a no-effect target count, or only a fully failed resolution?

```json
{
  "mechanicId": "mechanic:invented:failure-reaction:rule:1",
  "tags": [
    "ability-reaction"
  ],
  "when": {
    "window": "anyTime",
    "cadence": "each",
    "trigger": {
      "type": "event",
      "event": "mechanicUse",
      "where": {
        "type": "eventField",
        "field": "resolutionStatus",
        "value": "failed"
      }
    }
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "recordAction",
      "actionId": "invented-failure-reaction",
      "outcome": {
        "type": "literal",
        "value": "failed"
      }
    }
  ],
  "policies": []
}
```

### `count-effective-abilities-on-player` — Count abilities that affected a player

Count how many resolutions actually had a player as an effective target tonight.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.mechanicUse`, `bindings.effectTarget`, `aggregates.count`, `effects.emitInformation`

effectTarget retains effective targets after protection and redirection; count counts matching resolutions.

**Ask**

- Should it count resolutions or unique abilities?

**Limits**

- This example counts resolutions, not unique ability IDs.

```json
{
  "mechanicId": "mechanic:invented:effective-target-counter:rule:1",
  "tags": [
    "activity-tracking"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "events"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "eventType",
              "values": [
                "mechanicUse"
              ]
            },
            {
              "type": "eventPeriod",
              "value": "currentNight"
            },
            {
              "type": "eventParticipant",
              "binding": "effectTarget",
              "participant": "selected"
            }
          ]
        },
        "aggregate": {
          "type": "count"
        }
      },
      "presentation": {
        "kind": "number",
        "title": "Habilidades efectivas"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `registered-evil-used-ability-on-good` — Registered evil acted on good

Learn whether an actor registered as evil used an ability on someone registered as good tonight.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.mechanicUse`, `predicateTypes.events.eventParticipantIdentity`, `identityModes.registered`, `effects.emitInformation`

The event retains the actor's and selected target's registered identities at resolution time.

**Ask**

- Should each participant use real or registered identity?

**Limits**

- This example checks the selected target, not a redirected effective target.

```json
{
  "mechanicId": "mechanic:invented:registered-evil-on-good:rule:1",
  "tags": [
    "activity-tracking",
    "registration-information"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": {
          "entity": "events"
        },
        "where": {
          "type": "all",
          "conditions": [
            {
              "type": "eventType",
              "values": [
                "mechanicUse"
              ]
            },
            {
              "type": "eventPeriod",
              "value": "currentNight"
            },
            {
              "type": "eventParticipantIdentity",
              "participant": "actor",
              "identityMode": "registered",
              "facet": "allegiance",
              "values": [
                "evil"
              ]
            },
            {
              "type": "eventParticipantIdentity",
              "participant": "selected",
              "identityMode": "registered",
              "facet": "allegiance",
              "values": [
                "good"
              ]
            }
          ]
        },
        "aggregate": {
          "type": "exists"
        }
      },
      "presentation": {
        "kind": "boolean",
        "title": "Malvado sobre bueno"
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `protect-selected-from-death-until-dawn` — Protect from death until dawn

Choose a player at night and prevent them from dying until dawn.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.death`, `effects.applyMarker`, `effects.interceptEvent`, `durations.untilWindow`

The marker makes the protection visible and the interceptor cancels death events targeting that player for the same duration.

**Ask**

- Should it prevent every death or only deaths caused by one attack type?
- Is the protection consumed when it prevents a death?

**Limits**

- Add match when the protection covers only one cause of death; never infer that cause from ability prose.

```json
{
  "mechanicId": "mechanic:invented:dawn-protection:rule:1",
  "tags": [
    "night-protection"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "applyMarker",
      "kind": "reminder",
      "id": "invented-protected",
      "active": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dawn"
      }
    },
    {
      "type": "interceptEvent",
      "event": "death",
      "reaction": {
        "type": "cancel"
      },
      "reminder": "invented-protected",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dawn"
      }
    }
  ],
  "policies": []
}
```

### `counter-threshold-triggers-effect` — Trigger an effect at a counter threshold

Accumulate progress on a player and execute an effect once when a threshold is crossed.

- Status: `supported`
- Automation: `automatic`
- Covers: `bindings.effectTarget`, `effects.adjustCounter`, `effects.death`

adjustCounter stores the value and its projected markers; each thresholds entry runs its effects with the updated player available through effectTarget.

**Ask**

- Which value triggers the effect?
- Does the counter reset or remain at its limit?
- Is there more than one consequence at different heights?

**Limits**

- trigger=crossing prevents later resolutions from repeating the effect while the counter remains at the threshold.
- If the resource must strike again whenever a unit is added with the counter already full, use trigger=reaching and the accumulable-resource-shared-cap recipe.

```json
{
  "mechanicId": "mechanic:invented:pressure-threshold:rule:1",
  "tags": [
    "persistent-counter",
    "threshold-effect"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "adjustCounter",
      "counter": "invented-pressure-count",
      "delta": 1,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "bounds": {
        "min": 0,
        "max": 3
      },
      "projection": {
        "kind": "reminder",
        "mode": "copies",
        "id": "invented-pressure"
      },
      "duration": {
        "type": "permanent"
      },
      "thresholds": [
        {
          "operator": "gte",
          "value": 3,
          "trigger": "crossing",
          "effects": [
            {
              "type": "death",
              "targets": {
                "type": "binding",
                "binding": "effectTarget"
              }
            }
          ]
        }
      ]
    }
  ],
  "policies": []
}
```

### `accumulable-resource-shared-cap` — Accumulable resource with a cap, a state and a consequence

Model markers that accumulate on a player —one marker per unit—, flag a state while any remains, and trigger a consequence at the cap, even when several different sources add to or remove from the same resource.

- Status: `supported`
- Automation: `automatic`
- Covers: `bindings.effectTarget`, `effects.adjustCounter`, `effects.death`, `predicateTypes.players.counter`, `valueNodes.counterValue`, `aggregates.sum`

A single counter declares the resource and every source shares it with scope=shared; each source declares its own adjustCounter with the same counter, bounds and projection. bounds caps the stored reserve, projection.mode=copies draws one marker per unit —stages names levels when each step has its own marker— and stateProjection derives the state while the value satisfies activeWhen. thresholds chains the consequences of the same counter: each entry declares its value, its trigger and its own effects, so the second and third unit need no extra counter. trigger=reaching keeps the consequence alive once the counter sits at bounds.max: the cap limits the reserve, not the hit. resetTo consumes the resource so it can accumulate again from below. Removing units is the same effect with a negative delta and no thresholds. To read the resource from another rule, ask the counter —the counter condition and the counterValue value—, never which marker happens to be on the table. To sum that counter across all players, use a players query with project.type=counter and aggregate.type=sum.

**Ask**

- Do several sources feed the same resource, or does each source keep its own?
- What happens when a unit is added with the resource already at its cap?
- Is the resource consumed when the consequence fires, or does it stay accumulated?
- Should whoever sees the markers know the exact quantity?

**Limits**

- Creating separate Token 1, Token 2, and Token 3 markers to represent values 1, 2, and 3 is not recommended. Use one counter with projection.mode=copies; use stages only when every level needs its own visual marker identity.
- Do not declare one counter per consequence or one marker per rung: two counters raised together drift apart as soon as someone removes units, and a ladder of markers forces every source to repeat the resource logic.
- Do not declare count unless the rule imposes physical scarcity of the marker: without count the inventory never runs out of copies. The per-player cap lives in bounds and the table-wide total in the setup rule.
- countVisibility=hidden hides the quantity from whoever sees the marker without changing the mechanical value.

```json
{
  "mechanicId": "mechanic:invented:shared-resource-cap:rule:1",
  "tags": [
    "cumulative-marker",
    "shared-counter",
    "threshold-effect"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "adjustCounter",
      "counter": "invented-resource",
      "scope": "shared",
      "delta": 1,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "bounds": {
        "min": 0,
        "max": 3
      },
      "projection": {
        "kind": "reminder",
        "mode": "copies",
        "id": "invented-resource-unit"
      },
      "stateProjection": {
        "state": "invented-burdened",
        "activeWhen": {
          "operator": "gte",
          "value": 1
        }
      },
      "duration": {
        "type": "permanent"
      },
      "thresholds": [
        {
          "operator": "gte",
          "value": 3,
          "trigger": "reaching",
          "effects": [
            {
              "type": "death",
              "targets": {
                "type": "binding",
                "binding": "effectTarget"
              }
            }
          ]
        }
      ]
    }
  ],
  "policies": []
}
```

### `restrict-selected-ability-until-dusk` — Block an ability until the next dusk

Choose a player and temporarily block their ability, allowing false information while the block lasts.

- Status: `supported`
- Automation: `assisted`
- Covers: `effects.applyMarker`, `effects.restrict`, `durations.untilWindow`

restrict declares the ability restriction and its duration; the parallel marker lets the Storyteller see who is affected.

**Ask**

- May the target still wake?
- Must received information be false, or may it merely be false?

**Limits**

- Restricting ability does not itself prevent wake; add that restriction only when the ability requires it.

```json
{
  "mechanicId": "mechanic:invented:ability-block:rule:1",
  "tags": [
    "temporary-ability-restriction"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "applyMarker",
      "kind": "reminder",
      "id": "invented-ability-blocked",
      "active": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dusk",
        "count": 1
      }
    },
    {
      "type": "restrict",
      "restrictions": [
        "ability"
      ],
      "informationMayBeFalse": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dusk",
        "count": 1
      }
    }
  ],
  "policies": []
}
```

### `gain-chosen-character-ability-tonight` — Gain a chosen character's ability

Choose a character and temporarily grant its ability to the actor.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.characters`, `valueNodes.inputValue`, `inputKinds.character`, `effects.grantAbility`

The character input yields a typed characterId; grantAbility assigns it to the actor as owner and controller until dawn.

**Ask**

- May an in-play character be chosen?
- Who controls the granted ability, and when does it expire?

**Limits**

- Filter candidates explicitly; never infer allowed choices from prose or a visible team label.

```json
{
  "mechanicId": "mechanic:invented:borrowed-ability:rule:1",
  "tags": [
    "granted-ability"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 1
  },
  "input": {
    "kind": "character",
    "candidates": {
      "type": "query",
      "from": {
        "entity": "characters"
      },
      "where": {
        "type": "inPlay",
        "value": false
      },
      "aggregate": {
        "type": "collect"
      }
    }
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "grantAbility",
      "active": true,
      "abilityCharacterId": {
        "type": "inputValue",
        "valueType": "characterId"
      },
      "owner": "targets",
      "controller": "owner",
      "ownership": "sourceAbility",
      "targets": {
        "type": "binding",
        "binding": "actor"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dawn"
      }
    }
  ],
  "policies": []
}
```

### `replace-one-setup-bucket-with-another` — Replace one setup category with another

Add one slot to a setup category and remove it from another without changing the total player count.

- Status: `supported`
- Automation: `assisted`
- Covers: `windows.setup`, `effects.modifySetup`

Two opposing adjustBucket operations preserve game size and keep the category identifiers in pack data.

**Ask**

- Which category gains the slot, and which one yields it?
- What happens when the source category is already empty?

**Limits**

- Buckets are pack references; replace the invented IDs with categories declared by the context.

```json
{
  "mechanicId": "mechanic:invented:setup-swap:rule:1",
  "tags": [
    "setup-adjustment"
  ],
  "when": {
    "window": "setup",
    "cadence": "once"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "modifySetup",
      "operations": [
        {
          "type": "adjustBucket",
          "bucket": "invented-added-bucket",
          "delta": 1
        },
        {
          "type": "adjustBucket",
          "bucket": "invented-removed-bucket",
          "delta": -1
        }
      ]
    }
  ],
  "policies": []
}
```

### `may-register-as-another-allegiance` — May register as another alignment

Allow the actor to register as evil and support without changing their real identity.

- Status: `supported`
- Automation: `assisted`
- Covers: `alignments.evil`, `roles.support`, `effects.registerAs`

registerAs changes only registered identity; mode=mayRegisterAs leaves the choice to the Storyteller and does not alter real alignment.

**Ask**

- Is the alternate registration optional or mandatory?
- Does it still work after death?

**Limits**

- Do not use changeAlignment to model registration: that operation changes the player's real state.

```json
{
  "mechanicId": "mechanic:invented:alternate-registration:rule:1",
  "tags": [
    "registration"
  ],
  "when": {
    "window": "anyTime",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "registerAs",
      "mode": "mayRegisterAs",
      "alignment": [
        "evil"
      ],
      "roles": [
        "support"
      ],
      "targets": {
        "type": "binding",
        "binding": "actor"
      },
      "worksWhenDead": true
    }
  ],
  "policies": []
}
```

### `redirect-execution-to-selected-once` — Redirect an execution once

During an execution, choose a living replacement and redirect the event to them once per game.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.execution`, `eventBindings.nominee`, `effects.interceptEvent`

interceptEvent preserves the execution event while replacing its target; the usage limit prevents later redirects.

**Ask**

- Is the redirection mandatory or optional?
- Which participants may replace the nominee?

**Limits**

- Redirecting does not create a second execution; the reaction changes the existing event.

```json
{
  "mechanicId": "mechanic:invented:execution-redirect:rule:1",
  "tags": [
    "execution-interception",
    "target-redirection"
  ],
  "when": {
    "window": "execution",
    "cadence": "each"
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1,
    "candidates": {
      "type": "query",
      "from": {
        "entity": "players"
      },
      "where": {
        "type": "all",
        "conditions": [
          {
            "type": "alive",
            "value": true
          },
          {
            "type": "not",
            "condition": {
              "type": "isBinding",
              "binding": "nominee"
            }
          }
        ]
      },
      "aggregate": {
        "type": "collect"
      }
    }
  },
  "usage": {
    "scope": "game",
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "interceptEvent",
      "event": "execution",
      "targets": {
        "type": "binding",
        "binding": "nominee"
      },
      "reaction": {
        "type": "redirect",
        "targets": {
          "type": "binding",
          "binding": "selected"
        }
      },
      "optional": true
    }
  ],
  "policies": []
}
```

### `storyteller-decision-branches-effect` — Branch effects on a Storyteller decision

Ask the Storyteller for an enumerated mechanical decision and run an effect only for one response.

- Status: `supported`
- Automation: `assisted`
- Covers: `valueNodes.decisionValue`, `effects.storytellerDecision`, `effects.death`

storytellerDecision stores a typed value and decisionValue conditions any later effect in the same step.

**Ask**

- What are all valid responses?
- Which effects belong to each response?

**Limits**

- Options are mechanical data; never parse free-form prose to select the branch.

```json
{
  "mechanicId": "mechanic:invented:storyteller-branch:rule:1",
  "tags": [
    "storyteller-decision",
    "conditional-effect"
  ],
  "when": {
    "window": "day",
    "cadence": "once"
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "game",
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "storytellerDecision",
      "decision": "invented-resolution",
      "options": [
        "apply",
        "skip"
      ]
    },
    {
      "type": "death",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "when": [
        {
          "type": "compare",
          "left": {
            "type": "decisionValue",
            "decision": "invented-resolution"
          },
          "operator": "eq",
          "right": "apply"
        }
      ]
    }
  ],
  "presentation": {
    "decisionPrompts": {
      "invented-resolution": {
        "question": "¿Se aplica la consecuencia inventada?",
        "options": [
          {
            "value": "apply",
            "label": "Aplicar"
          },
          {
            "value": "skip",
            "label": "Omitir"
          }
        ]
      }
    }
  },
  "policies": []
}
```

### `relation-and-restriction-while-source-alive` — Keep a relation and restriction while the source lives

Link a player to the actor and block their ability only while the actor remains alive.

- Status: `supported`
- Automation: `assisted`
- Covers: `valueNodes.query`, `durations.whileCondition`, `effects.setPlayerRelation`, `effects.restrict`

The relation and restriction share a whileCondition duration that queries the actor's life, so they end together.

**Ask**

- Does the relation survive its source?
- Should it also end when the source loses its ability?

**Limits**

- ownership=independent keeps the link from implicitly depending on the source character ID; duration declares its lifecycle.

```json
{
  "mechanicId": "mechanic:invented:living-source-link:rule:1",
  "tags": [
    "persistent-relation",
    "source-lifetime"
  ],
  "when": {
    "window": "firstNight",
    "cadence": "once",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "conditions": [],
  "effects": [
    {
      "type": "setPlayerRelation",
      "kind": "invented-bound",
      "markerId": "invented-bound",
      "active": true,
      "ownership": "independent",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "whileCondition",
        "condition": {
          "type": "query",
          "from": {
            "entity": "players"
          },
          "where": {
            "type": "all",
            "conditions": [
              {
                "type": "isBinding",
                "binding": "actor"
              },
              {
                "type": "alive",
                "value": true
              }
            ]
          },
          "aggregate": {
            "type": "exists"
          }
        }
      }
    },
    {
      "type": "restrict",
      "restrictions": [
        "ability"
      ],
      "informationMayBeFalse": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "whileCondition",
        "condition": {
          "type": "query",
          "from": {
            "entity": "players"
          },
          "where": {
            "type": "all",
            "conditions": [
              {
                "type": "isBinding",
                "binding": "actor"
              },
              {
                "type": "alive",
                "value": true
              }
            ]
          },
          "aggregate": {
            "type": "exists"
          }
        }
      }
    }
  ],
  "policies": []
}
```

### `change-selected-character-preserving-alignment` — Change character while preserving alignment

Choose a player and an out-of-play character, replace their real identity, and preserve their current alignment.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.characters`, `valueNodes.inputValue`, `inputKinds.playerAndCharacter`, `effects.changeCharacter`

playerAndCharacter yields a player and characterId in one input; changeCharacter uses that value and preserveAlignment avoids an accidental conversion.

**Ask**

- May an in-play character be chosen?
- Is alignment, role, or both preserved?

**Limits**

- Never derive the new character from its name; use the typed characterId supplied by the input.

```json
{
  "mechanicId": "mechanic:invented:identity-replacement:rule:1",
  "tags": [
    "character-change",
    "preserve-alignment"
  ],
  "when": {
    "window": "night",
    "cadence": "each",
    "startsAt": 2
  },
  "input": {
    "kind": "playerAndCharacter",
    "player": {
      "kind": "players",
      "min": 1,
      "max": 1
    },
    "characterCandidates": {
      "type": "query",
      "from": {
        "entity": "characters"
      },
      "where": {
        "type": "inPlay",
        "value": false
      },
      "aggregate": {
        "type": "collect"
      }
    }
  },
  "usage": {
    "scope": "repeat"
  },
  "conditions": [],
  "effects": [
    {
      "type": "changeCharacter",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "newCharacter": {
        "type": "inputValue",
        "valueType": "characterId"
      },
      "preserveAlignment": true
    }
  ],
  "policies": []
}
```

### `change-alignment-and-record-source-relation` — Change alignment and retain the source relation

Convert a player and persistently record who caused the change.

- Status: `supported`
- Automation: `automatic`
- Covers: `alignments.evil`, `effects.changeAlignment`, `effects.setPlayerRelation`

changeAlignment changes real state; setPlayerRelation separately preserves provenance for later queries or effects.

**Ask**

- Which alignment do they change to?
- Can the conversion or relation later be reversed?

**Limits**

- The relation does not replace the alignment change, and the alignment change alone does not record who caused it.

```json
{
  "mechanicId": "mechanic:invented:alignment-conversion:rule:1",
  "tags": [
    "alignment-change",
    "conversion-provenance"
  ],
  "when": {
    "window": "night",
    "cadence": "once",
    "startsAt": 1
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1
  },
  "usage": {
    "scope": "game",
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "changeAlignment",
      "alignment": "evil",
      "notifyPlayer": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      }
    },
    {
      "type": "setPlayerRelation",
      "kind": "invented-converted-by",
      "markerId": "invented-converted",
      "active": true,
      "ownership": "independent",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "permanent"
      }
    }
  ],
  "policies": []
}
```

### `invalidate-vote-when-only-one-group-votes` — Invalidate a vote when only one group votes

Make a tally invalid when every voter belongs to the same declared alignment.

- Status: `supported`
- Automation: `automatic`
- Covers: `windows.voting`, `identityModes.real`, `effects.modifyVote`

tallyValidity evaluates the voter set and declares a mechanical reason when every voter matches the predicate.

**Ask**

- Which group invalidates the tally when voting alone?
- Does the rule affect every voting purpose?

**Limits**

- invalidWhen=allMatch changes neither weights nor threshold: it invalidates the entire tally.

```json
{
  "mechanicId": "mechanic:invented:voter-validity:rule:1",
  "tags": [
    "voting-validity"
  ],
  "when": {
    "window": "voting",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "modifyVote",
      "purposes": [
        "standard"
      ],
      "tallyValidity": {
        "voters": {
          "type": "query",
          "from": {
            "entity": "players"
          },
          "where": {
            "type": "identity",
            "identityMode": "real",
            "facet": "allegiance",
            "values": [
              "evil"
            ]
          },
          "aggregate": {
            "type": "collect"
          }
        },
        "invalidWhen": "allMatch",
        "reason": "Solo votó el grupo declarado."
      }
    }
  ],
  "policies": []
}
```

### `prepare-truthful-or-zero-structured-information` — Prepare truthful, alternative, or zero information

Prepare structured information with one truthful player and an alternative, or report that none exists.

- Status: `supported`
- Automation: `assisted`
- Covers: `inputValueTypes.object`, `valueNodes.inputValue`, `effects.prepareInformation`, `effects.emitInformation`

prepareInformation builds the value and its helper markers; emitInformation then delivers the typed object without reconstructing it from prose.

**Ask**

- Which identity determines truth: real, registered, or shown?
- When should the zero variant be used?

**Limits**

- zeroWhen must inspect the same universe as candidates so it cannot contradict the prepared clue.

```json
{
  "mechanicId": "mechanic:invented:prepared-clue:rule:1",
  "tags": [
    "prepared-information",
    "structured-information"
  ],
  "when": {
    "window": "firstNight",
    "cadence": "once",
    "startsAt": 1
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "prepareInformation",
      "candidates": {
        "type": "query",
        "from": {
          "entity": "players"
        },
        "where": {
          "type": "identity",
          "identityMode": "registered",
          "facet": "allegiance",
          "values": [
            "good"
          ]
        },
        "aggregate": {
          "type": "collect"
        },
        "project": {
          "type": "entityId"
        }
      },
      "modes": [
        "pair",
        "zero"
      ],
      "zeroWhen": {
        "type": "compare",
        "left": {
          "type": "query",
          "from": {
            "entity": "players"
          },
          "where": {
            "type": "identity",
            "identityMode": "registered",
            "facet": "allegiance",
            "values": [
              "good"
            ]
          },
          "aggregate": {
            "type": "exists"
          }
        },
        "operator": "eq",
        "right": false
      },
      "characterChoice": {
        "source": "truthfulPlayer",
        "identityMode": "registered"
      },
      "reminders": {
        "truthful": "invented-truthful",
        "alternative": "invented-alternative"
      }
    },
    {
      "type": "emitInformation",
      "value": {
        "type": "inputValue",
        "valueType": "object"
      },
      "presentation": {
        "kind": "structured",
        "title": "Pista preparada",
        "fields": [
          {
            "key": "players",
            "label": "Jugadores",
            "kind": "players"
          },
          {
            "key": "character",
            "label": "Personaje",
            "kind": "character"
          },
          {
            "key": "none",
            "label": "Ninguno",
            "kind": "boolean"
          }
        ]
      },
      "delivery": {
        "audience": {
          "type": "actor"
        }
      }
    }
  ],
  "policies": []
}
```

### `public-guess-conditionally-ends-game` — End the game after a correct guess

Publicly choose a player and character, record whether they match, and end the game only when the guess is correct.

- Status: `supported`
- Automation: `assisted`
- Covers: `facts.outcome`, `valueNodes.if`, `inputKinds.playerAndCharacter`, `effects.recordAction`, `effects.resolveGameEnd`

recordAction computes a typed outcome; resolveGameEnd then reads facts.outcome instead of duplicating the success logic.

**Ask**

- Does the comparison use real, registered, or shown identity?
- Who wins when the result is correct?

**Limits**

- Declare precedence=override only when this victory must beat simultaneous standard endings.

```json
{
  "mechanicId": "mechanic:invented:winning-guess:rule:1",
  "tags": [
    "public-guess",
    "conditional-game-end"
  ],
  "when": {
    "window": "day",
    "cadence": "once"
  },
  "input": {
    "kind": "playerAndCharacter",
    "player": {
      "kind": "players",
      "min": 1,
      "max": 1
    },
    "characterCandidates": {
      "type": "query",
      "from": {
        "entity": "characters"
      },
      "aggregate": {
        "type": "collect"
      }
    }
  },
  "usage": {
    "scope": "game",
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "recordAction",
      "actionId": "invented-public-guess",
      "recordAs": "publicDeclaration",
      "outcome": {
        "type": "if",
        "condition": {
          "type": "query",
          "from": {
            "entity": "players"
          },
          "where": {
            "type": "all",
            "conditions": [
              {
                "type": "isBinding",
                "binding": "selected"
              },
              {
                "type": "identityMatchesInput",
                "identityMode": "real",
                "facet": "character"
              }
            ]
          },
          "aggregate": {
            "type": "exists"
          }
        },
        "then": {
          "type": "literal",
          "value": "correct"
        },
        "else": {
          "type": "literal",
          "value": "incorrect"
        }
      }
    },
    {
      "type": "resolveGameEnd",
      "mode": "immediate",
      "winner": {
        "type": "fixed",
        "team": "good"
      },
      "reason": "La adivinación pública inventada fue correcta.",
      "precedence": "override",
      "when": [
        {
          "type": "compare",
          "left": {
            "type": "fact",
            "fact": "outcome"
          },
          "operator": "eq",
          "right": "correct"
        }
      ]
    }
  ],
  "policies": []
}
```

### `block-game-end-before-round-threshold` — Block an ending before a round threshold

Prevent one side from winning before a specific night even when its normal condition is met.

- Status: `supported`
- Automation: `automatic`
- Covers: `windows.gameEnd`, `gameProperties.nightNumber`, `effects.blockGameEnd`

The mechanic runs at gameEnd and blockGameEnd vetoes only the declared winner while nightNumber remains below the threshold.

**Ask**

- Which result is blocked?
- Is the night threshold inclusive or exclusive?

**Limits**

- nightNumber starts at 1; state precisely whether the ending becomes available during or after the threshold night.

```json
{
  "mechanicId": "mechanic:invented:early-ending-block:rule:1",
  "tags": [
    "game-end-block",
    "round-threshold"
  ],
  "when": {
    "window": "gameEnd",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [
    {
      "type": "compare",
      "left": {
        "type": "game",
        "property": "nightNumber"
      },
      "operator": "lt",
      "right": 3
    }
  ],
  "effects": [
    {
      "type": "blockGameEnd",
      "winner": "good",
      "reason": "La tercera noche inventada todavía no ha comenzado.",
      "activation": {
        "lifeState": "any",
        "abilityState": "any"
      }
    }
  ],
  "policies": []
}
```

### `delayed-consequence-after-death` — Schedule a consequence after a death

When the actor dies, choose a living player, mark them, and resolve their death one night later.

- Status: `supported`
- Automation: `automatic`
- Covers: `events.death`, `eventBindings.eventSubject`, `effects.applyMarker`, `effects.death`

The trigger preserves the dead actor, the marker exposes the pending consequence, and delay schedules the effect without ad hoc timers.

**Ask**

- Is the delay measured in phases, days, or nights?
- Does the consequence remain pending if the target changes identity?

**Limits**

- The marker and delayed effect must describe the same deadline so visible state cannot drift.

```json
{
  "mechanicId": "mechanic:invented:death-delay:rule:1",
  "tags": [
    "event-trigger",
    "delayed-consequence"
  ],
  "when": {
    "trigger": {
      "type": "event",
      "event": "death",
      "bindings": {
        "eventSubject": "actor"
      }
    },
    "delay": {
      "unit": "phase",
      "count": 0
    }
  },
  "input": {
    "kind": "players",
    "min": 1,
    "max": 1,
    "candidates": {
      "type": "query",
      "from": {
        "entity": "players"
      },
      "where": {
        "type": "alive",
        "value": true
      },
      "aggregate": {
        "type": "collect"
      }
    }
  },
  "usage": {
    "scope": "trigger",
    "limit": {
      "type": "literal",
      "value": 1
    },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "applyMarker",
      "kind": "reminder",
      "id": "invented-delayed-consequence",
      "active": true,
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "duration": {
        "type": "untilWindow",
        "window": "dawn",
        "count": 2
      }
    },
    {
      "type": "death",
      "targets": {
        "type": "binding",
        "binding": "selected"
      },
      "delay": {
        "unit": "night",
        "count": 1
      }
    }
  ],
  "policies": []
}
```

### `optional-setup-choice-with-remainder` — Choose a setup adjustment and recalculate the remainder

Offer the Storyteller several variations for one category and recalculate another to preserve total player count.

- Status: `supported`
- Automation: `assisted`
- Covers: `windows.setup`, `effects.modifySetup`

chooseAdjustments declares the alternatives and setBucketCount with remainder keeps final cast size stable.

**Ask**

- Which adjustments may the Storyteller choose?
- Which category absorbs the remaining cast slots?

**Limits**

- Every option must leave a valid composition; optional does not repair impossible combinations.

```json
{
  "mechanicId": "mechanic:invented:setup-choice:rule:1",
  "tags": [
    "setup-choice",
    "stable-player-count"
  ],
  "when": {
    "window": "setup",
    "cadence": "once"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "modifySetup",
      "optional": true,
      "operations": [
        {
          "type": "chooseAdjustments",
          "options": [
            {
              "adjustments": [
                {
                  "bucket": "invented-variable-bucket",
                  "delta": 0
                }
              ]
            },
            {
              "adjustments": [
                {
                  "bucket": "invented-variable-bucket",
                  "delta": 1
                }
              ]
            }
          ]
        },
        {
          "type": "setBucketCount",
          "bucket": "invented-remainder-bucket",
          "count": {
            "type": "remainder"
          }
        }
      ]
    }
  ],
  "policies": []
}
```

### `coordinate-nomination-limit-and-vote-weight` — Coordinate nomination limit and vote weight

Allow the actor to nominate more often and give their vote a different weight under the same rule.

- Status: `supported`
- Automation: `assisted`
- Covers: `windows.nomination`, `effects.modifyNomination`, `effects.modifyVote`

modifyNomination and modifyVote declare the two changes separately while activating from the same nomination surface.

**Ask**

- How many nominations may they make?
- Does the special weight work after death?

**Limits**

- Do not use voteDelta as a persistent weight substitute: nomination limit and vote weight are separate contracts.

```json
{
  "mechanicId": "mechanic:invented:nomination-and-vote:rule:1",
  "tags": [
    "nomination-modifier",
    "vote-modifier"
  ],
  "when": {
    "window": "nomination",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "modifyNomination",
      "maxNominationsPerDay": 2,
      "worksWhenDead": true
    },
    {
      "type": "modifyVote",
      "targets": {
        "type": "binding",
        "binding": "actor"
      },
      "weight": 2,
      "worksWhenDead": true
    }
  ],
  "policies": []
}
```

### `repeat-ability-after-no-effect` — Repeat an ability after no effect

Generically execute the same ability instance again when it produced no effect.

- Status: `unsupported`
- Automation: `manual`
- Covers: `events.mechanicUse`, `eventFields.targetResultStatus`

History can detect noEffect, but there is no generic primitive that invokes an arbitrary ability instance again.

**Limits**

- Do not simulate the repeat through prose or IDs.

**Missing primitive:** A typed operation that re-invokes an identified resolution while preserving actor, selection, and provenance.

### `learn-altered-information-recipient` — Learn who received altered information

Later query the exact recipient of transformed information.

- Status: `unsupported`
- Automation: `manual`
- Covers: `effects.modifyInformation`, `events.mechanicUse`

modifyInformation can change delivery, but the ledger does not yet expose a queryable informationAltered attribute.

**Limits**

- Do not infer this fact from displayed text.

**Missing primitive:** An event attribute that records information transformation and its effective recipients.

## Exhaustive matrix

| ID | Purpose | Fragment |
|---|---|---|
| `valueNodes.literal` | Builds a typed literal value expression. | `{"type":"literal"}` |
| `valueNodes.binding` | Builds a typed binding value expression. | `{"type":"binding"}` |
| `valueNodes.game` | Builds a typed game value expression. | `{"type":"game"}` |
| `valueNodes.counterValue` | Builds a typed counter value value expression. | `{"type":"counterValue"}` |
| `valueNodes.setup` | Builds a typed setup value expression. | `{"type":"setup"}` |
| `valueNodes.fact` | Builds a typed fact value expression. | `{"type":"fact"}` |
| `valueNodes.storytellerDecision` | Builds a typed storyteller decision value expression. | `{"type":"storytellerDecision"}` |
| `valueNodes.decisionValue` | Builds a typed decision value value expression. | `{"type":"decisionValue"}` |
| `valueNodes.all` | Builds a typed all value expression. | `{"type":"all"}` |
| `valueNodes.any` | Builds a typed any value expression. | `{"type":"any"}` |
| `valueNodes.not` | Builds a typed not value expression. | `{"type":"not"}` |
| `valueNodes.query` | Builds a typed query value expression. | `{"type":"query"}` |
| `valueNodes.compare` | Builds a typed compare value expression. | `{"type":"compare"}` |
| `valueNodes.array` | Builds a typed array value expression. | `{"type":"array"}` |
| `valueNodes.object` | Builds a typed object value expression. | `{"type":"object"}` |
| `valueNodes.concat` | Builds a typed concat value expression. | `{"type":"concat"}` |
| `valueNodes.set` | Builds a typed set value expression. | `{"type":"set"}` |
| `valueNodes.math` | Builds a typed math value expression. | `{"type":"math"}` |
| `valueNodes.unique` | Builds a typed unique value expression. | `{"type":"unique"}` |
| `valueNodes.length` | Builds a typed length value expression. | `{"type":"length"}` |
| `valueNodes.take` | Builds a typed take value expression. | `{"type":"take"}` |
| `valueNodes.at` | Builds a typed at value expression. | `{"type":"at"}` |
| `valueNodes.if` | Builds a typed if value expression. | `{"type":"if"}` |
| `valueNodes.inputValue` | Builds a typed input value value expression. | `{"type":"inputValue"}` |
| `terminology.teamId` | Supported terminology option identified by teamId. | `{"value":"teamId"}` |
| `events.characterEntry` | Supported events option identified by characterEntry. | `{"type":"event","event":"characterEntry"}` |
| `events.death` | Supported events option identified by death. | `{"type":"event","event":"death"}` |
| `events.execution` | Supported events option identified by execution. | `{"type":"event","event":"execution"}` |
| `events.expulsion` | Supported events option identified by expulsion. | `{"type":"event","event":"expulsion"}` |
| `events.nomination` | Supported events option identified by nomination. | `{"type":"event","event":"nomination"}` |
| `events.publicAction` | Supported events option identified by publicAction. | `{"type":"event","event":"publicAction"}` |
| `events.mechanicUse` | Supported events option identified by mechanicUse. | `{"type":"event","event":"mechanicUse"}` |
| `events.mechanicTargeted` | Supported events option identified by mechanicTargeted. | `{"type":"event","event":"mechanicTargeted"}` |
| `events.tableAction` | Supported events option identified by tableAction. | `{"type":"event","event":"tableAction"}` |
| `events.stateChange` | Supported events option identified by stateChange. | `{"type":"event","event":"stateChange"}` |
| `events.restrictionChange` | Supported events option identified by restrictionChange. | `{"type":"event","event":"restrictionChange"}` |
| `events.markerChange` | Supported events option identified by markerChange. | `{"type":"event","event":"markerChange"}` |
| `events.abilityGrant` | Supported events option identified by abilityGrant. | `{"type":"event","event":"abilityGrant"}` |
| `events.storytellerSignal` | Supported events option identified by storytellerSignal. | `{"type":"event","event":"storytellerSignal"}` |
| `events.storytellerExecution` | Supported events option identified by storytellerExecution. | `{"type":"event","event":"storytellerExecution"}` |
| `eventBindings.actor` | Supported event bindings option identified by actor. | `{"value":"actor"}` |
| `eventBindings.selected` | Supported event bindings option identified by selected. | `{"value":"selected"}` |
| `eventBindings.source` | Supported event bindings option identified by source. | `{"value":"source"}` |
| `eventBindings.nominator` | Supported event bindings option identified by nominator. | `{"value":"nominator"}` |
| `eventBindings.nominee` | Supported event bindings option identified by nominee. | `{"value":"nominee"}` |
| `eventBindings.voter` | Supported event bindings option identified by voter. | `{"value":"voter"}` |
| `eventBindings.eventSubject` | Supported event bindings option identified by eventSubject. | `{"value":"eventSubject"}` |
| `bindings.actor` | Supported bindings option identified by actor. | `{"value":"actor"}` |
| `bindings.selected` | Supported bindings option identified by selected. | `{"value":"selected"}` |
| `bindings.source` | Supported bindings option identified by source. | `{"value":"source"}` |
| `bindings.nominator` | Supported bindings option identified by nominator. | `{"value":"nominator"}` |
| `bindings.nominee` | Supported bindings option identified by nominee. | `{"value":"nominee"}` |
| `bindings.voter` | Supported bindings option identified by voter. | `{"value":"voter"}` |
| `bindings.eventSubject` | Supported bindings option identified by eventSubject. | `{"value":"eventSubject"}` |
| `bindings.effectTarget` | Supported bindings option identified by effectTarget. | `{"value":"effectTarget"}` |
| `entities.players` | Supported entities option identified by players. | `{"value":"players"}` |
| `entities.characters` | Supported entities option identified by characters. | `{"value":"characters"}` |
| `entities.events` | Supported entities option identified by events. | `{"value":"events"}` |
| `entities.markers` | Supported entities option identified by markers. | `{"value":"markers"}` |
| `predicateTypes.players.alive` | Supported players option identified by alive. | `{"value":"alive"}` |
| `predicateTypes.players.identity` | Supported players option identified by identity. | `{"value":"identity"}` |
| `predicateTypes.players.entryMode` | Supported players option identified by entryMode. | `{"value":"entryMode"}` |
| `predicateTypes.players.identityMatchesBinding` | Supported players option identified by identityMatchesBinding. | `{"value":"identityMatchesBinding"}` |
| `predicateTypes.players.identityMatchesInput` | Supported players option identified by identityMatchesInput. | `{"value":"identityMatchesInput"}` |
| `predicateTypes.players.state` | Supported players option identified by state. | `{"value":"state"}` |
| `predicateTypes.players.marker` | Supported players option identified by marker. | `{"value":"marker"}` |
| `predicateTypes.players.counter` | Supported players option identified by counter. | `{"value":"counter"}` |
| `predicateTypes.players.membership` | Supported players option identified by membership. | `{"value":"membership"}` |
| `predicateTypes.players.isBinding` | Supported players option identified by isBinding. | `{"value":"isBinding"}` |
| `predicateTypes.players.all` | Supported players option identified by all. | `{"value":"all"}` |
| `predicateTypes.players.any` | Supported players option identified by any. | `{"value":"any"}` |
| `predicateTypes.players.not` | Supported players option identified by not. | `{"value":"not"}` |
| `predicateTypes.characters.inPlay` | Supported characters option identified by inPlay. | `{"value":"inPlay"}` |
| `predicateTypes.characters.identity` | Supported characters option identified by identity. | `{"value":"identity"}` |
| `predicateTypes.characters.identityMatchesBinding` | Supported characters option identified by identityMatchesBinding. | `{"value":"identityMatchesBinding"}` |
| `predicateTypes.characters.identityMatchesInput` | Supported characters option identified by identityMatchesInput. | `{"value":"identityMatchesInput"}` |
| `predicateTypes.characters.all` | Supported characters option identified by all. | `{"value":"all"}` |
| `predicateTypes.characters.any` | Supported characters option identified by any. | `{"value":"any"}` |
| `predicateTypes.characters.not` | Supported characters option identified by not. | `{"value":"not"}` |
| `predicateTypes.events.eventType` | Supported events option identified by eventType. | `{"value":"eventType"}` |
| `predicateTypes.events.eventCategory` | Supported events option identified by eventCategory. | `{"value":"eventCategory"}` |
| `predicateTypes.events.eventPeriod` | Supported events option identified by eventPeriod. | `{"value":"eventPeriod"}` |
| `predicateTypes.events.eventField` | Supported events option identified by eventField. | `{"value":"eventField"}` |
| `predicateTypes.events.eventRestriction` | Supported events option identified by eventRestriction. | `{"value":"eventRestriction"}` |
| `predicateTypes.events.eventParticipant` | Supported events option identified by eventParticipant. | `{"value":"eventParticipant"}` |
| `predicateTypes.events.eventParticipantIdentity` | Supported events option identified by eventParticipantIdentity. | `{"value":"eventParticipantIdentity"}` |
| `predicateTypes.events.all` | Supported events option identified by all. | `{"value":"all"}` |
| `predicateTypes.events.any` | Supported events option identified by any. | `{"value":"any"}` |
| `predicateTypes.events.not` | Supported events option identified by not. | `{"value":"not"}` |
| `predicateTypes.markers.markerKind` | Supported markers option identified by markerKind. | `{"value":"markerKind"}` |
| `predicateTypes.markers.markerId` | Supported markers option identified by markerId. | `{"value":"markerId"}` |
| `predicateTypes.markers.markerActive` | Supported markers option identified by markerActive. | `{"value":"markerActive"}` |
| `predicateTypes.markers.markerEntity` | Supported markers option identified by markerEntity. | `{"value":"markerEntity"}` |
| `predicateTypes.markers.markerSourceCharacter` | Supported markers option identified by markerSourceCharacter. | `{"value":"markerSourceCharacter"}` |
| `predicateTypes.markers.all` | Supported markers option identified by all. | `{"value":"all"}` |
| `predicateTypes.markers.any` | Supported markers option identified by any. | `{"value":"any"}` |
| `predicateTypes.markers.not` | Supported markers option identified by not. | `{"value":"not"}` |
| `aggregates.collect` | Supported aggregates option identified by collect. | `{"value":"collect"}` |
| `aggregates.sum` | Supported aggregates option identified by sum. | `{"value":"sum"}` |
| `aggregates.count` | Supported aggregates option identified by count. | `{"value":"count"}` |
| `aggregates.exists` | Supported aggregates option identified by exists. | `{"value":"exists"}` |
| `aggregates.all` | Supported aggregates option identified by all. | `{"value":"all"}` |
| `aggregates.exactly` | Supported aggregates option identified by exactly. | `{"value":"exactly"}` |
| `aggregates.first` | Supported aggregates option identified by first. | `{"value":"first"}` |
| `aggregates.last` | Supported aggregates option identified by last. | `{"value":"last"}` |
| `aggregates.nearest` | Supported aggregates option identified by nearest. | `{"value":"nearest"}` |
| `aggregates.distance` | Supported aggregates option identified by distance. | `{"value":"distance"}` |
| `aggregates.direction` | Supported aggregates option identified by direction. | `{"value":"direction"}` |
| `aggregates.adjacentPairCount` | Supported aggregates option identified by adjacentPairCount. | `{"value":"adjacentPairCount"}` |
| `comparisons.eq` | Supported comparisons option identified by eq. | `{"value":"eq"}` |
| `comparisons.neq` | Supported comparisons option identified by neq. | `{"value":"neq"}` |
| `comparisons.gt` | Supported comparisons option identified by gt. | `{"value":"gt"}` |
| `comparisons.gte` | Supported comparisons option identified by gte. | `{"value":"gte"}` |
| `comparisons.lt` | Supported comparisons option identified by lt. | `{"value":"lt"}` |
| `comparisons.lte` | Supported comparisons option identified by lte. | `{"value":"lte"}` |
| `comparisons.includes` | Supported comparisons option identified by includes. | `{"value":"includes"}` |
| `directions.clockwise` | Supported directions option identified by clockwise. | `{"value":"clockwise"}` |
| `directions.counterclockwise` | Supported directions option identified by counterclockwise. | `{"value":"counterclockwise"}` |
| `inputValueTypes.string` | Supported input value types option identified by string. | `{"value":"string"}` |
| `inputValueTypes.number` | Supported input value types option identified by number. | `{"value":"number"}` |
| `inputValueTypes.boolean` | Supported input value types option identified by boolean. | `{"value":"boolean"}` |
| `inputValueTypes.null` | Supported input value types option identified by null. | `{"value":"null"}` |
| `inputValueTypes.array` | Supported input value types option identified by array. | `{"value":"array"}` |
| `inputValueTypes.object` | Supported input value types option identified by object. | `{"value":"object"}` |
| `inputValueTypes.entityId` | Supported input value types option identified by entityId. | `{"value":"entityId"}` |
| `inputValueTypes.playerId` | Supported input value types option identified by playerId. | `{"value":"playerId"}` |
| `inputValueTypes.characterId` | Supported input value types option identified by characterId. | `{"value":"characterId"}` |
| `gameProperties.phase` | Supported game properties option identified by phase. | `{"value":"phase"}` |
| `gameProperties.dayNumber` | Supported game properties option identified by dayNumber. | `{"value":"dayNumber"}` |
| `gameProperties.nightNumber` | Supported game properties option identified by nightNumber. | `{"value":"nightNumber"}` |
| `facts.outcome` | Supported facts option identified by outcome. | `{"value":"outcome"}` |
| `facts.guessResult` | Supported facts option identified by guessResult. | `{"value":"guessResult"}` |
| `facts.targetMechanicTags` | Supported facts option identified by targetMechanicTags. | `{"value":"targetMechanicTags"}` |
| `facts.operationWouldEndGame` | Supported facts option identified by operationWouldEndGame. | `{"value":"operationWouldEndGame"}` |
| `setOperations.union` | Supported set operations option identified by union. | `{"value":"union"}` |
| `setOperations.intersection` | Supported set operations option identified by intersection. | `{"value":"intersection"}` |
| `setOperations.difference` | Supported set operations option identified by difference. | `{"value":"difference"}` |
| `mathOperations.add` | Supported math operations option identified by add. | `{"value":"add"}` |
| `mathOperations.subtract` | Supported math operations option identified by subtract. | `{"value":"subtract"}` |
| `mathOperations.min` | Supported math operations option identified by min. | `{"value":"min"}` |
| `mathOperations.max` | Supported math operations option identified by max. | `{"value":"max"}` |
| `identityModes.real` | Supported identity modes option identified by real. | `{"identityMode":"real"}` |
| `identityModes.initial` | Supported identity modes option identified by initial. | `{"identityMode":"initial"}` |
| `identityModes.base` | Supported identity modes option identified by base. | `{"identityMode":"base"}` |
| `identityModes.shown` | Supported identity modes option identified by shown. | `{"identityMode":"shown"}` |
| `identityModes.apparent` | Supported identity modes option identified by apparent. | `{"identityMode":"apparent"}` |
| `identityModes.registered` | Applies registration decisions before reading identity. | `{"identityMode":"registered"}` |
| `identityFacets.teamId` | Supported identity facets option identified by teamId. | `{"value":"teamId"}` |
| `identityFacets.allegiance` | Supported identity facets option identified by allegiance. | `{"value":"allegiance"}` |
| `identityFacets.role` | Supported identity facets option identified by role. | `{"value":"role"}` |
| `identityFacets.character` | Supported identity facets option identified by character. | `{"value":"character"}` |
| `identityFacets.mechanicTags` | Supported identity facets option identified by mechanicTags. | `{"value":"mechanicTags"}` |
| `comparableIdentityFacets.teamId` | Supported comparable identity facets option identified by teamId. | `{"value":"teamId"}` |
| `comparableIdentityFacets.allegiance` | Supported comparable identity facets option identified by allegiance. | `{"value":"allegiance"}` |
| `comparableIdentityFacets.role` | Supported comparable identity facets option identified by role. | `{"value":"role"}` |
| `comparableIdentityFacets.character` | Supported comparable identity facets option identified by character. | `{"value":"character"}` |
| `alignments.good` | Supported alignments option identified by good. | `{"value":"good"}` |
| `alignments.evil` | Supported alignments option identified by evil. | `{"value":"evil"}` |
| `alignments.neutral` | Supported alignments option identified by neutral. | `{"value":"neutral"}` |
| `roles.core` | Supported roles option identified by core. | `{"value":"core"}` |
| `roles.support` | Supported roles option identified by support. | `{"value":"support"}` |
| `roles.independent` | Supported roles option identified by independent. | `{"value":"independent"}` |
| `entryModes.cast` | Supported entry modes option identified by cast. | `{"value":"cast"}` |
| `entryModes.temporary` | Supported entry modes option identified by temporary. | `{"value":"temporary"}` |
| `eventPeriods.currentBatch` | Supported event periods option identified by currentBatch. | `{"value":"currentBatch"}` |
| `eventPeriods.currentDay` | Supported event periods option identified by currentDay. | `{"value":"currentDay"}` |
| `eventPeriods.currentNight` | Supported event periods option identified by currentNight. | `{"value":"currentNight"}` |
| `eventPeriods.previousDay` | Supported event periods option identified by previousDay. | `{"value":"previousDay"}` |
| `eventPeriods.previousNight` | Supported event periods option identified by previousNight. | `{"value":"previousNight"}` |
| `eventPeriods.game` | Supported event periods option identified by game. | `{"value":"game"}` |
| `eventRestrictionMatches.all` | Supported event restriction matches option identified by all. | `{"value":"all"}` |
| `eventRestrictionMatches.any` | Supported event restriction matches option identified by any. | `{"value":"any"}` |
| `eventRestrictionMatches.exact` | Supported event restriction matches option identified by exact. | `{"value":"exact"}` |
| `eventFields.actionId` | Supported event fields option identified by actionId. | `{"type":"eventField","field":"actionId","value":"<value>"}` |
| `eventFields.outcome` | Supported event fields option identified by outcome. | `{"type":"eventField","field":"outcome","value":"<value>"}` |
| `eventFields.guessResult` | Supported event fields option identified by guessResult. | `{"type":"eventField","field":"guessResult","value":"<value>"}` |
| `eventFields.died` | Supported event fields option identified by died. | `{"type":"eventField","field":"died","value":"<value>"}` |
| `eventFields.attribution` | Supported event fields option identified by attribution. | `{"type":"eventField","field":"attribution","value":"<value>"}` |
| `eventFields.resolution` | Supported event fields option identified by resolution. | `{"type":"eventField","field":"resolution","value":"<value>"}` |
| `eventFields.known` | Supported event fields option identified by known. | `{"type":"eventField","field":"known","value":"<value>"}` |
| `eventFields.occurrence` | Supported event fields option identified by occurrence. | `{"type":"eventField","field":"occurrence","value":"<value>"}` |
| `eventFields.signal` | Supported event fields option identified by signal. | `{"type":"eventField","field":"signal","value":"<value>"}` |
| `eventFields.characterId` | Supported event fields option identified by characterId. | `{"type":"eventField","field":"characterId","value":"<value>"}` |
| `eventFields.sourceCharacterId` | Supported event fields option identified by sourceCharacterId. | `{"type":"eventField","field":"sourceCharacterId","value":"<value>"}` |
| `eventFields.targetCount` | Supported event fields option identified by targetCount. | `{"type":"eventField","field":"targetCount","value":"<value>"}` |
| `eventFields.targetAlignment` | Supported event fields option identified by targetAlignment. | `{"type":"eventField","field":"targetAlignment","value":"<value>"}` |
| `eventFields.targetAlive` | Supported event fields option identified by targetAlive. | `{"type":"eventField","field":"targetAlive","value":"<value>"}` |
| `eventFields.state` | Supported event fields option identified by state. | `{"type":"eventField","field":"state","value":"<value>"}` |
| `eventFields.active` | Supported event fields option identified by active. | `{"type":"eventField","field":"active","value":"<value>"}` |
| `eventFields.markerId` | Supported event fields option identified by markerId. | `{"type":"eventField","field":"markerId","value":"<value>"}` |
| `eventFields.markerKind` | Supported event fields option identified by markerKind. | `{"type":"eventField","field":"markerKind","value":"<value>"}` |
| `eventFields.resolutionId` | Supported event fields option identified by resolutionId. | `{"type":"eventField","field":"resolutionId","value":"<value>"}` |
| `eventFields.resolutionStatus` | Reports the typed final status of a mechanic resolution. | `{"type":"eventField","field":"resolutionStatus","value":"<value>"}` |
| `eventFields.resolutionPhase` | Supported event fields option identified by resolutionPhase. | `{"type":"eventField","field":"resolutionPhase","value":"<value>"}` |
| `eventFields.actorAbilityMode` | Reports whether the actor worked normally, malfunctioned, or was disabled. | `{"type":"eventField","field":"actorAbilityMode","value":"<value>"}` |
| `eventFields.abilityProvenance` | Reports whether an ability was native, granted, copied, or otherwise sourced. | `{"type":"eventField","field":"abilityProvenance","value":"<value>"}` |
| `eventFields.effectiveTargetCount` | Counts players the mechanic ultimately affected. | `{"type":"eventField","field":"effectiveTargetCount","value":"<value>"}` |
| `eventFields.targetResultStatus` | Reports the common result applied to a mechanic's targets. | `{"type":"eventField","field":"targetResultStatus","value":"<value>"}` |
| `eventFields.tableAction` | Supported event fields option identified by tableAction. | `{"type":"eventField","field":"tableAction","value":"<value>"}` |
| `eventFields.causalOutcome` | Supported event fields option identified by causalOutcome. | `{"type":"eventField","field":"causalOutcome","value":"<value>"}` |
| `eventFields.originalTargets` | Supported event fields option identified by originalTargets. | `{"type":"eventField","field":"originalTargets","value":"<value>"}` |
| `eventFields.effectiveTargets` | Supported event fields option identified by effectiveTargets. | `{"type":"eventField","field":"effectiveTargets","value":"<value>"}` |
| `eventFields.sourceMechanicId` | Supported event fields option identified by sourceMechanicId. | `{"type":"eventField","field":"sourceMechanicId","value":"<value>"}` |
| `eventFields.reactionSources` | Supported event fields option identified by reactionSources. | `{"type":"eventField","field":"reactionSources","value":"<value>"}` |
| `eventFields.causedByEventId` | Supported event fields option identified by causedByEventId. | `{"type":"eventField","field":"causedByEventId","value":"<value>"}` |
| `information.kinds.boolean` | Supported kinds option identified by boolean. | `{"value":"boolean"}` |
| `information.kinds.number` | Supported kinds option identified by number. | `{"value":"number"}` |
| `information.kinds.text` | Supported kinds option identified by text. | `{"value":"text"}` |
| `information.kinds.player` | Supported kinds option identified by player. | `{"value":"player"}` |
| `information.kinds.players` | Supported kinds option identified by players. | `{"value":"players"}` |
| `information.kinds.character` | Supported kinds option identified by character. | `{"value":"character"}` |
| `information.kinds.characters` | Supported kinds option identified by characters. | `{"value":"characters"}` |
| `information.kinds.identity` | Supported kinds option identified by identity. | `{"value":"identity"}` |
| `information.kinds.direction` | Supported kinds option identified by direction. | `{"value":"direction"}` |
| `information.kinds.distance` | Supported kinds option identified by distance. | `{"value":"distance"}` |
| `information.kinds.grimoire` | Supported kinds option identified by grimoire. | `{"value":"grimoire"}` |
| `information.kinds.structured` | Supported kinds option identified by structured. | `{"value":"structured"}` |
| `information.audiences.actor` | Supported audiences option identified by actor. | `{"value":"actor"}` |
| `information.audiences.selected` | Supported audiences option identified by selected. | `{"value":"selected"}` |
| `information.audiences.eventSubject` | Supported audiences option identified by eventSubject. | `{"value":"eventSubject"}` |
| `information.audiences.public` | Supported audiences option identified by public. | `{"value":"public"}` |
| `information.audiences.storyteller` | Supported audiences option identified by storyteller. | `{"value":"storyteller"}` |
| `information.audiences.players` | Supported audiences option identified by players. | `{"value":"players"}` |
| `information.timings.immediate` | Supported timings option identified by immediate. | `{"value":"immediate"}` |
| `information.timings.dawn` | Supported timings option identified by dawn. | `{"value":"dawn"}` |
| `information.timings.privateDay` | Supported timings option identified by privateDay. | `{"value":"privateDay"}` |
| `information.timings.nextRecipientWake` | Supported timings option identified by nextRecipientWake. | `{"value":"nextRecipientWake"}` |
| `information.modes.shared` | Supported modes option identified by shared. | `{"value":"shared"}` |
| `information.modes.perRecipient` | Supported modes option identified by perRecipient. | `{"value":"perRecipient"}` |
| `information.consentSubjects.selected` | Supported consent subjects option identified by selected. | `{"value":"selected"}` |
| `information.consentSubjects.eventSubject` | Supported consent subjects option identified by eventSubject. | `{"value":"eventSubject"}` |
| `information.transforms.whenSourceAffected` | Supported transforms option identified by whenSourceAffected. | `{"value":"whenSourceAffected"}` |
| `information.transformReactions.allowArbitraryValue` | Supported transform reactions option identified by allowArbitraryValue. | `{"value":"allowArbitraryValue"}` |
| `themes.purple` | Supported themes option identified by purple. | `{"value":"purple"}` |
| `themes.ink` | Supported themes option identified by ink. | `{"value":"ink"}` |
| `themes.oxblood` | Supported themes option identified by oxblood. | `{"value":"oxblood"}` |
| `themes.verdigris` | Supported themes option identified by verdigris. | `{"value":"verdigris"}` |
| `themes.midnight` | Supported themes option identified by midnight. | `{"value":"midnight"}` |
| `themes.leather` | Supported themes option identified by leather. | `{"value":"leather"}` |
| `themes.absinthe` | Supported themes option identified by absinthe. | `{"value":"absinthe"}` |
| `themes.soot` | Supported themes option identified by soot. | `{"value":"soot"}` |
| `themes.rosewood` | Supported themes option identified by rosewood. | `{"value":"rosewood"}` |
| `themes.pallor` | Supported themes option identified by pallor. | `{"value":"pallor"}` |
| `themes.fog` | Supported themes option identified by fog. | `{"value":"fog"}` |
| `themes.heather` | Supported themes option identified by heather. | `{"value":"heather"}` |
| `backdrops.none` | Supported backdrops option identified by none. | `{"value":"none"}` |
| `backdrops.astrolabe` | Supported backdrops option identified by astrolabe. | `{"value":"astrolabe"}` |
| `backdrops.orrery` | Supported backdrops option identified by orrery. | `{"value":"orrery"}` |
| `backdrops.rose` | Supported backdrops option identified by rose. | `{"value":"rose"}` |
| `backdrops.filigree` | Supported backdrops option identified by filigree. | `{"value":"filigree"}` |
| `backdrops.baroque` | Supported backdrops option identified by baroque. | `{"value":"baroque"}` |
| `backdrops.thorns` | Supported backdrops option identified by thorns. | `{"value":"thorns"}` |
| `backdrops.cobweb` | Supported backdrops option identified by cobweb. | `{"value":"cobweb"}` |
| `backdrops.ribcage` | Supported backdrops option identified by ribcage. | `{"value":"ribcage"}` |
| `backdrops.candelabra` | Supported backdrops option identified by candelabra. | `{"value":"candelabra"}` |
| `backdrops.bramble` | Supported backdrops option identified by bramble. | `{"value":"bramble"}` |
| `backdrops.sigil` | Supported backdrops option identified by sigil. | `{"value":"sigil"}` |
| `backdrops.runes` | Supported backdrops option identified by runes. | `{"value":"runes"}` |
| `backdrops.blood` | Supported backdrops option identified by blood. | `{"value":"blood"}` |
| `backdrops.plain` | Supported backdrops option identified by plain. | `{"value":"plain"}` |
| `windows.setup` | Supported windows option identified by setup. | `{"window":"setup"}` |
| `windows.firstNight` | Supported windows option identified by firstNight. | `{"window":"firstNight"}` |
| `windows.night` | Supported windows option identified by night. | `{"window":"night"}` |
| `windows.dawn` | Supported windows option identified by dawn. | `{"window":"dawn"}` |
| `windows.day` | Supported windows option identified by day. | `{"window":"day"}` |
| `windows.voting` | Supported windows option identified by voting. | `{"window":"voting"}` |
| `windows.speech` | Supported windows option identified by speech. | `{"window":"speech"}` |
| `windows.nomination` | Supported windows option identified by nomination. | `{"window":"nomination"}` |
| `windows.execution` | Supported windows option identified by execution. | `{"window":"execution"}` |
| `windows.expulsion` | Supported windows option identified by expulsion. | `{"window":"expulsion"}` |
| `windows.dusk` | Supported windows option identified by dusk. | `{"window":"dusk"}` |
| `windows.mainEvilInfo` | Supported windows option identified by mainEvilInfo. | `{"window":"mainEvilInfo"}` |
| `windows.gameEnd` | Supported windows option identified by gameEnd. | `{"window":"gameEnd"}` |
| `windows.anyTime` | Supported windows option identified by anyTime. | `{"window":"anyTime"}` |
| `cadences.once` | Supported cadences option identified by once. | `{"cadence":"once"}` |
| `cadences.each` | Supported cadences option identified by each. | `{"cadence":"each"}` |
| `inputKinds.none` | Supported input kinds option identified by none. | `{"kind":"none"}` |
| `inputKinds.players` | Supported input kinds option identified by players. | `{"kind":"players"}` |
| `inputKinds.character` | Supported input kinds option identified by character. | `{"kind":"character"}` |
| `inputKinds.playerAndCharacter` | Supported input kinds option identified by playerAndCharacter. | `{"kind":"playerAndCharacter"}` |
| `inputKinds.participantResponses` | Supported input kinds option identified by participantResponses. | `{"kind":"participantResponses"}` |
| `inputKinds.text` | Supported input kinds option identified by text. | `{"kind":"text"}` |
| `inputKinds.playerCharacterGuesses` | Supported input kinds option identified by playerCharacterGuesses. | `{"kind":"playerCharacterGuesses"}` |
| `inputKinds.seatSwaps` | Supported input kinds option identified by seatSwaps. | `{"kind":"seatSwaps"}` |
| `inputKinds.vote` | Supported input kinds option identified by vote. | `{"kind":"vote"}` |
| `inputKinds.contest` | Supported input kinds option identified by contest. | `{"kind":"contest"}` |
| `usageScopes.repeat` | Supported usage scopes option identified by repeat. | `{"scope":"repeat"}` |
| `usageScopes.day` | Supported usage scopes option identified by day. | `{"scope":"day"}` |
| `usageScopes.night` | Supported usage scopes option identified by night. | `{"scope":"night"}` |
| `usageScopes.game` | Supported usage scopes option identified by game. | `{"scope":"game"}` |
| `usageScopes.actor` | Supported usage scopes option identified by actor. | `{"scope":"actor"}` |
| `usageScopes.target` | Supported usage scopes option identified by target. | `{"scope":"target"}` |
| `usageScopes.trigger` | Supported usage scopes option identified by trigger. | `{"scope":"trigger"}` |
| `usageDimensions.day` | Separates usage buckets by day number. | `{"keyBy":["day"]}` |
| `usageDimensions.night` | Separates usage buckets by night number. | `{"keyBy":["night"]}` |
| `usageDimensions.actor` | Separates usage buckets by acting player. | `{"keyBy":["actor"]}` |
| `usageDimensions.target` | Separates usage buckets by target player. | `{"keyBy":["target"]}` |
| `usageDimensions.trigger` | Separates usage buckets by triggering event. | `{"keyBy":["trigger"]}` |
| `consumeOn.attempt` | Supported consume on option identified by attempt. | `{"consumeOn":"attempt"}` |
| `consumeOn.resolution` | Supported consume on option identified by resolution. | `{"consumeOn":"resolution"}` |
| `consumeOn.success` | Supported consume on option identified by success. | `{"consumeOn":"success"}` |
| `durations.permanent` | Supported durations option identified by permanent. | `{"type":"permanent"}` |
| `durations.untilWindow` | Supported durations option identified by untilWindow. | `{"type":"untilWindow"}` |
| `durations.whileTargetAlive` | Supported durations option identified by whileTargetAlive. | `{"type":"whileTargetAlive"}` |
| `durations.whileCondition` | Supported durations option identified by whileCondition. | `{"type":"whileCondition"}` |
| `durations.untilEvent` | Ends when a typed event pattern matches. | `{"type":"untilEvent","event":"death"}` |
| `effects.death` | Supported effects option identified by death. | `{"type":"death"}` |
| `effects.death.fields.type` | Field accepted by death; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.death.fields.when` | Field accepted by death; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.death.fields.delay` | Field accepted by death; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.death.fields.targets` | Field accepted by death; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.death.fields.affectedBy` | Field accepted by death; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.death.fields.duration` | Field accepted by death; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.death.fields.optional` | Field accepted by death; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.death.fields.reminder` | Field accepted by death; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.death.fields.reminderTokens` | Field accepted by death; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.death.fields.spentReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.death.fields.attribution` | Field accepted by death; its value must satisfy the typed contract. | `{"attribution":"<attribution>"}` |
| `effects.death.fields.bypassesDeathProtection` | Field accepted by death; its value must satisfy the typed contract. | `{"bypassesDeathProtection":"<bypassesDeathProtection>"}` |
| `effects.death.fields.bypassesProtection` | Field accepted by death; its value must satisfy the typed contract. | `{"bypassesProtection":"<bypassesProtection>"}` |
| `effects.death.fields.chargeReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"chargeReminder":"<chargeReminder>"}` |
| `effects.death.fields.conditionTarget` | Field accepted by death; its value must satisfy the typed contract. | `{"conditionTarget":"<conditionTarget>"}` |
| `effects.death.fields.consumeTargetReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"consumeTargetReminder":"<consumeTargetReminder>"}` |
| `effects.death.fields.continuesNomination` | Field accepted by death; its value must satisfy the typed contract. | `{"continuesNomination":"<continuesNomination>"}` |
| `effects.death.fields.createsReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"createsReminder":"<createsReminder>"}` |
| `effects.death.fields.jumpReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"jumpReminder":"<jumpReminder>"}` |
| `effects.death.fields.mustNominateOnDay3` | Field accepted by death; its value must satisfy the typed contract. | `{"mustNominateOnDay3":"<mustNominateOnDay3>"}` |
| `effects.death.fields.protection` | Field accepted by death; its value must satisfy the typed contract. | `{"protection":"<protection>"}` |
| `effects.death.fields.publicAnnouncement` | Field accepted by death; its value must satisfy the typed contract. | `{"publicAnnouncement":"<publicAnnouncement>"}` |
| `effects.death.fields.registration` | Field accepted by death; its value must satisfy the typed contract. | `{"registration":"<registration>"}` |
| `effects.death.fields.requiredActionOutcome` | Field accepted by death; its value must satisfy the typed contract. | `{"requiredActionOutcome":"<requiredActionOutcome>"}` |
| `effects.death.fields.requiresSourceAlive` | Field accepted by death; its value must satisfy the typed contract. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.death.fields.resolution` | Field accepted by death; its value must satisfy the typed contract. | `{"resolution":"<resolution>"}` |
| `effects.death.fields.resultOutcome` | Field accepted by death; its value must satisfy the typed contract. | `{"resultOutcome":"<resultOutcome>"}` |
| `effects.death.fields.secondaryReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"secondaryReminder":"<secondaryReminder>"}` |
| `effects.death.fields.targetAlignment` | Field accepted by death; its value must satisfy the typed contract. | `{"targetAlignment":"<targetAlignment>"}` |
| `effects.death.fields.targetNotTeam` | Field accepted by death; its value must satisfy the typed contract. | `{"targetNotTeam":"<targetNotTeam>"}` |
| `effects.death.fields.targetNotProfile` | Field accepted by death; its value must satisfy the typed contract. | `{"targetNotProfile":"<targetNotProfile>"}` |
| `effects.death.fields.targetReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"targetReminder":"<targetReminder>"}` |
| `effects.death.fields.triggerActionDayScope` | Field accepted by death; its value must satisfy the typed contract. | `{"triggerActionDayScope":"<triggerActionDayScope>"}` |
| `effects.death.fields.triggerActionId` | Field accepted by death; its value must satisfy the typed contract. | `{"triggerActionId":"<triggerActionId>"}` |
| `effects.death.fields.triggerReminder` | Field accepted by death; its value must satisfy the typed contract. | `{"triggerReminder":"<triggerReminder>"}` |
| `effects.death.fields.useRegisteredIdentity` | Field accepted by death; its value must satisfy the typed contract. | `{"useRegisteredIdentity":"<useRegisteredIdentity>"}` |
| `effects.resurrect` | Supported effects option identified by resurrect. | `{"type":"resurrect"}` |
| `effects.resurrect.fields.type` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.resurrect.fields.when` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.resurrect.fields.delay` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.resurrect.fields.targets` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.resurrect.fields.affectedBy` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.resurrect.fields.duration` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.resurrect.fields.optional` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.resurrect.fields.reminder` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.resurrect.fields.reminderTokens` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.resurrect.fields.spentReminder` | Field accepted by resurrect; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.execute` | Supported effects option identified by execute. | `{"type":"execute"}` |
| `effects.execute.fields.type` | Field accepted by execute; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.execute.fields.when` | Field accepted by execute; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.execute.fields.delay` | Field accepted by execute; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.execute.fields.targets` | Field accepted by execute; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.execute.fields.affectedBy` | Field accepted by execute; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.execute.fields.duration` | Field accepted by execute; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.execute.fields.optional` | Field accepted by execute; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.execute.fields.reminder` | Field accepted by execute; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.execute.fields.reminderTokens` | Field accepted by execute; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.execute.fields.spentReminder` | Field accepted by execute; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.execute.fields.dies` | Field accepted by execute; its value must satisfy the typed contract. | `{"dies":"<dies>"}` |
| `effects.execute.fields.requiresSourceAlive` | Field accepted by execute; its value must satisfy the typed contract. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.execute.fields.targetReminder` | Field accepted by execute; its value must satisfy the typed contract. | `{"targetReminder":"<targetReminder>"}` |
| `effects.setPlayerState` | Supported effects option identified by setPlayerState. | `{"type":"setPlayerState","state":"state","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.setPlayerState.fields.type` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.setPlayerState.fields.when` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.setPlayerState.fields.delay` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.setPlayerState.fields.targets` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.setPlayerState.fields.affectedBy` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.setPlayerState.fields.duration` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.setPlayerState.fields.state` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"state":"<state>"}` |
| `effects.setPlayerState.fields.active` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"active":"<active>"}` |
| `effects.setPlayerState.fields.ownership` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"ownership":"<ownership>"}` |
| `effects.setPlayerState.fields.exclusive` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"exclusive":"<exclusive>"}` |
| `effects.setPlayerState.fields.excludeInitialTargets` | Field accepted by setPlayerState; its value must satisfy the typed contract. | `{"excludeInitialTargets":"<excludeInitialTargets>"}` |
| `effects.setPlayerRelation` | Supported effects option identified by setPlayerRelation. | `{"type":"setPlayerRelation","kind":"linked","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.setPlayerRelation.fields.type` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.setPlayerRelation.fields.when` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.setPlayerRelation.fields.delay` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.setPlayerRelation.fields.targets` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.setPlayerRelation.fields.affectedBy` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.setPlayerRelation.fields.duration` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.setPlayerRelation.fields.kind` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"kind":"<kind>"}` |
| `effects.setPlayerRelation.fields.active` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"active":"<active>"}` |
| `effects.setPlayerRelation.fields.markerId` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"markerId":"<markerId>"}` |
| `effects.setPlayerRelation.fields.ownership` | Field accepted by setPlayerRelation; its value must satisfy the typed contract. | `{"ownership":"<ownership>"}` |
| `effects.applyMarker` | Adds or removes a reminder marker while retaining source metadata. | `{"type":"applyMarker","kind":"reminder","id":"marker","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.applyMarker.fields.type` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.applyMarker.fields.when` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.applyMarker.fields.delay` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.applyMarker.fields.targets` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.applyMarker.fields.affectedBy` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.applyMarker.fields.duration` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.applyMarker.fields.kind` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"kind":"<kind>"}` |
| `effects.applyMarker.fields.id` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"id":"<id>"}` |
| `effects.applyMarker.fields.active` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"active":"<active>"}` |
| `effects.applyMarker.fields.exclusive` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"exclusive":"<exclusive>"}` |
| `effects.applyMarker.fields.ownership` | Field accepted by applyMarker; its value must satisfy the typed contract. | `{"ownership":"<ownership>"}` |
| `effects.moveMarker` | Atomically transfers an existing marker or all marker-backed cancellation protections between players. | `{"type":"moveMarker","kind":"reminder","id":"marker","from":{"type":"binding","binding":"actor"},"targets":{"type":"binding","binding":"selected"}}` |
| `effects.moveMarker.fields.type` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.moveMarker.fields.when` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.moveMarker.fields.delay` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.moveMarker.fields.targets` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.moveMarker.fields.affectedBy` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.moveMarker.fields.kind` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"kind":"<kind>"}` |
| `effects.moveMarker.fields.id` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"id":"<id>"}` |
| `effects.moveMarker.fields.allProtections` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"allProtections":"<allProtections>"}` |
| `effects.moveMarker.fields.from` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"from":"<from>"}` |
| `effects.adjustCounter` | Supported effects option identified by adjustCounter. | `{"type":"adjustCounter","counter":"counter","delta":1,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.adjustCounter.fields.type` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.adjustCounter.fields.when` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.adjustCounter.fields.delay` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.adjustCounter.fields.targets` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.adjustCounter.fields.affectedBy` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.adjustCounter.fields.duration` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.adjustCounter.fields.counter` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"counter":"<counter>"}` |
| `effects.adjustCounter.fields.delta` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"delta":"<delta>"}` |
| `effects.adjustCounter.fields.scope` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"scope":"<scope>"}` |
| `effects.adjustCounter.fields.bounds` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"bounds":"<bounds>"}` |
| `effects.adjustCounter.fields.projection` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"projection":"<projection>"}` |
| `effects.adjustCounter.fields.stateProjection` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"stateProjection":"<stateProjection>"}` |
| `effects.adjustCounter.fields.thresholds` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"thresholds":"<thresholds>"}` |
| `effects.adjustCounter.fields.threshold` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"threshold":"<threshold>"}` |
| `effects.adjustCounter.fields.onThreshold` | Field accepted by adjustCounter; its value must satisfy the typed contract. | `{"onThreshold":"<onThreshold>"}` |
| `effects.changeAlignment` | Supported effects option identified by changeAlignment. | `{"type":"changeAlignment"}` |
| `effects.changeAlignment.fields.type` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.changeAlignment.fields.when` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.changeAlignment.fields.delay` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.changeAlignment.fields.targets` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.changeAlignment.fields.affectedBy` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.changeAlignment.fields.duration` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.changeAlignment.fields.optional` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.changeAlignment.fields.reminder` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.changeAlignment.fields.reminderTokens` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.changeAlignment.fields.spentReminder` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.changeAlignment.fields.alignment` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"alignment":"<alignment>"}` |
| `effects.changeAlignment.fields.allowsSelfTarget` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"allowsSelfTarget":"<allowsSelfTarget>"}` |
| `effects.changeAlignment.fields.notifyPlayer` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"notifyPlayer":"<notifyPlayer>"}` |
| `effects.changeAlignment.fields.setupEffect` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"setupEffect":"<setupEffect>"}` |
| `effects.changeAlignment.fields.targetAlignment` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"targetAlignment":"<targetAlignment>"}` |
| `effects.changeAlignment.fields.targetTeam` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"targetTeam":"<targetTeam>"}` |
| `effects.changeAlignment.fields.targetProfile` | Field accepted by changeAlignment; its value must satisfy the typed contract. | `{"targetProfile":"<targetProfile>"}` |
| `effects.changeCharacter` | Supported effects option identified by changeCharacter. | `{"type":"changeCharacter"}` |
| `effects.changeCharacter.fields.type` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.changeCharacter.fields.when` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.changeCharacter.fields.delay` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.changeCharacter.fields.targets` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.changeCharacter.fields.affectedBy` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.changeCharacter.fields.duration` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.changeCharacter.fields.optional` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.changeCharacter.fields.reminder` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.changeCharacter.fields.reminderTokens` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.changeCharacter.fields.spentReminder` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.changeCharacter.fields.allowedBuckets` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"allowedBuckets":"<allowedBuckets>"}` |
| `effects.changeCharacter.fields.allowShownCharacterInPlay` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"allowShownCharacterInPlay":"<allowShownCharacterInPlay>"}` |
| `effects.changeCharacter.fields.arbitraryDeathsIfMainEvilCreated` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"arbitraryDeathsIfMainEvilCreated":"<arbitraryDeathsIfMainEvilCreated>"}` |
| `effects.changeCharacter.fields.characterType` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"characterType":"<characterType>"}` |
| `effects.changeCharacter.fields.characterProfile` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"characterProfile":"<characterProfile>"}` |
| `effects.changeCharacter.fields.countTeams` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"countTeams":"<countTeams>"}` |
| `effects.changeCharacter.fields.countProfiles` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"countProfiles":"<countProfiles>"}` |
| `effects.changeCharacter.fields.countTiming` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"countTiming":"<countTiming>"}` |
| `effects.changeCharacter.fields.createsReminder` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"createsReminder":"<createsReminder>"}` |
| `effects.changeCharacter.fields.markOriginalAbilityHolderIfInPlay` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"markOriginalAbilityHolderIfInPlay":"<markOriginalAbilityHolderIfInPlay>"}` |
| `effects.changeCharacter.fields.gainsAbilityOfChosenCharacter` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"gainsAbilityOfChosenCharacter":"<gainsAbilityOfChosenCharacter>"}` |
| `effects.changeCharacter.fields.minimumAlive` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"minimumAlive":"<minimumAlive>"}` |
| `effects.changeCharacter.fields.mustNeighborTeam` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"mustNeighborTeam":"<mustNeighborTeam>"}` |
| `effects.changeCharacter.fields.mustNeighborProfile` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"mustNeighborProfile":"<mustNeighborProfile>"}` |
| `effects.changeCharacter.fields.newCharacter` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"newCharacter":"<newCharacter>"}` |
| `effects.changeCharacter.fields.newAlignment` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"newAlignment":"<newAlignment>"}` |
| `effects.changeCharacter.fields.newTeam` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"newTeam":"<newTeam>"}` |
| `effects.changeCharacter.fields.newProfile` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"newProfile":"<newProfile>"}` |
| `effects.changeCharacter.fields.oldMainEvilDies` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"oldMainEvilDies":"<oldMainEvilDies>"}` |
| `effects.changeCharacter.fields.preserveAlignment` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"preserveAlignment":"<preserveAlignment>"}` |
| `effects.changeCharacter.fields.realCharacter` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"realCharacter":"<realCharacter>"}` |
| `effects.changeCharacter.fields.realTeam` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"realTeam":"<realTeam>"}` |
| `effects.changeCharacter.fields.realProfile` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"realProfile":"<realProfile>"}` |
| `effects.changeCharacter.fields.result` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"result":"<result>"}` |
| `effects.changeCharacter.fields.shownAs` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"shownAs":"<shownAs>"}` |
| `effects.changeCharacter.fields.targetCharacter` | Field accepted by changeCharacter; its value must satisfy the typed contract. | `{"targetCharacter":"<targetCharacter>"}` |
| `effects.grantAbility` | Supported effects option identified by grantAbility. | `{"type":"grantAbility"}` |
| `effects.grantAbility.fields.type` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.grantAbility.fields.when` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.grantAbility.fields.delay` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.grantAbility.fields.targets` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.grantAbility.fields.affectedBy` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.grantAbility.fields.duration` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.grantAbility.fields.optional` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.grantAbility.fields.reminder` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.grantAbility.fields.reminderTokens` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.grantAbility.fields.spentReminder` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.grantAbility.fields.active` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"active":"<active>"}` |
| `effects.grantAbility.fields.abilityCharacterId` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"abilityCharacterId":"<abilityCharacterId>"}` |
| `effects.grantAbility.fields.owner` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"owner":"<owner>"}` |
| `effects.grantAbility.fields.controller` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"controller":"<controller>"}` |
| `effects.grantAbility.fields.ownership` | Field accepted by grantAbility; its value must satisfy the typed contract. | `{"ownership":"<ownership>"}` |
| `effects.triggerAbility` | Supported effects option identified by triggerAbility. | `{"type":"triggerAbility","mechanicTag":"ability-tag"}` |
| `effects.triggerAbility.fields.type` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.triggerAbility.fields.when` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.triggerAbility.fields.delay` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.triggerAbility.fields.targets` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.triggerAbility.fields.affectedBy` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.triggerAbility.fields.duration` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.triggerAbility.fields.optional` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.triggerAbility.fields.reminder` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.triggerAbility.fields.reminderTokens` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.triggerAbility.fields.spentReminder` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.triggerAbility.fields.mechanicTag` | Field accepted by triggerAbility; its value must satisfy the typed contract. | `{"mechanicTag":"<mechanicTag>"}` |
| `effects.swapSeats` | Supported effects option identified by swapSeats. | `{"type":"swapSeats"}` |
| `effects.swapSeats.fields.type` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.swapSeats.fields.when` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.swapSeats.fields.delay` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.swapSeats.fields.targets` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.swapSeats.fields.affectedBy` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapSeats.fields.duration` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.swapSeats.fields.optional` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.swapSeats.fields.reminder` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.swapSeats.fields.reminderTokens` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapSeats.fields.spentReminder` | Field accepted by swapSeats; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.swapCharacters` | Supported effects option identified by swapCharacters. | `{"type":"swapCharacters"}` |
| `effects.swapCharacters.fields.type` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.swapCharacters.fields.when` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.swapCharacters.fields.delay` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.swapCharacters.fields.targets` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.swapCharacters.fields.affectedBy` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapCharacters.fields.duration` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.swapCharacters.fields.optional` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.swapCharacters.fields.reminder` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.swapCharacters.fields.reminderTokens` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapCharacters.fields.spentReminder` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.swapCharacters.fields.actor` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"actor":"<actor>"}` |
| `effects.swapCharacters.fields.resultingState` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"resultingState":"<resultingState>"}` |
| `effects.swapCharacters.fields.resultingStateDuration` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"resultingStateDuration":"<resultingStateDuration>"}` |
| `effects.swapCharacters.fields.swapsCharactersAndAlignments` | Field accepted by swapCharacters; its value must satisfy the typed contract. | `{"swapsCharactersAndAlignments":"<swapsCharactersAndAlignments>"}` |
| `effects.swapTargets` | Supported effects option identified by swapTargets. | `{"type":"swapTargets"}` |
| `effects.swapTargets.fields.type` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.swapTargets.fields.when` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.swapTargets.fields.delay` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.swapTargets.fields.targets` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.swapTargets.fields.affectedBy` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapTargets.fields.duration` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.swapTargets.fields.optional` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.swapTargets.fields.reminder` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.swapTargets.fields.reminderTokens` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapTargets.fields.spentReminder` | Field accepted by swapTargets; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.emitInformation` | Calculates and delivers typed information. | `{"type":"emitInformation","value":{"type":"literal","value":""},"presentation":{"kind":"text","title":"Información"},"delivery":{"audience":{"type":"storyteller"}}}` |
| `effects.emitInformation.fields.type` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.emitInformation.fields.when` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.emitInformation.fields.delay` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.emitInformation.fields.targets` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.emitInformation.fields.affectedBy` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.emitInformation.fields.duration` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.emitInformation.fields.optional` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.emitInformation.fields.reminder` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.emitInformation.fields.reminderTokens` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.emitInformation.fields.spentReminder` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.emitInformation.fields.value` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"value":"<value>"}` |
| `effects.emitInformation.fields.presentation` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"presentation":"<presentation>"}` |
| `effects.emitInformation.fields.delivery` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"delivery":"<delivery>"}` |
| `effects.emitInformation.fields.transform` | Field accepted by emitInformation; its value must satisfy the typed contract. | `{"transform":"<transform>"}` |
| `effects.prepareInformation` | Supported effects option identified by prepareInformation. | `{"type":"prepareInformation","candidates":{"type":"array","items":[]},"modes":["pair"],"characterChoice":{"source":"truthfulPlayer","identityMode":"real"},"reminders":{"truthful":"Verdadero","alternative":"Alternativa"}}` |
| `effects.prepareInformation.fields.type` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.prepareInformation.fields.when` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.prepareInformation.fields.delay` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.prepareInformation.fields.targets` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.prepareInformation.fields.affectedBy` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.prepareInformation.fields.duration` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.prepareInformation.fields.optional` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.prepareInformation.fields.reminder` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.prepareInformation.fields.reminderTokens` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.prepareInformation.fields.spentReminder` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.prepareInformation.fields.candidates` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"candidates":"<candidates>"}` |
| `effects.prepareInformation.fields.modes` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"modes":"<modes>"}` |
| `effects.prepareInformation.fields.zeroWhen` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"zeroWhen":"<zeroWhen>"}` |
| `effects.prepareInformation.fields.characterChoice` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"characterChoice":"<characterChoice>"}` |
| `effects.prepareInformation.fields.reminders` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"reminders":"<reminders>"}` |
| `effects.prepareInformation.fields.shownIdentityOverride` | Field accepted by prepareInformation; its value must satisfy the typed contract. | `{"shownIdentityOverride":"<shownIdentityOverride>"}` |
| `effects.resolveGameEnd` | Supported effects option identified by resolveGameEnd. | `{"type":"resolveGameEnd","mode":"immediate","winner":{"type":"fixed","team":"good"},"reason":"Describe por qué termina la partida."}` |
| `effects.resolveGameEnd.fields.type` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.resolveGameEnd.fields.when` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.resolveGameEnd.fields.delay` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.resolveGameEnd.fields.targets` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.resolveGameEnd.fields.affectedBy` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.resolveGameEnd.fields.duration` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.resolveGameEnd.fields.optional` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.resolveGameEnd.fields.reminder` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.resolveGameEnd.fields.reminderTokens` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.resolveGameEnd.fields.spentReminder` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.resolveGameEnd.fields.mode` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"mode":"<mode>"}` |
| `effects.resolveGameEnd.fields.winner` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"winner":"<winner>"}` |
| `effects.resolveGameEnd.fields.reason` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"reason":"<reason>"}` |
| `effects.resolveGameEnd.fields.precedence` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"precedence":"<precedence>"}` |
| `effects.resolveGameEnd.fields.activation` | Field accepted by resolveGameEnd; its value must satisfy the typed contract. | `{"activation":"<activation>"}` |
| `effects.blockGameEnd` | Supported effects option identified by blockGameEnd. | `{"type":"blockGameEnd","winner":"good","reason":"La victoria está bloqueada."}` |
| `effects.blockGameEnd.fields.type` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.blockGameEnd.fields.when` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.blockGameEnd.fields.delay` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.blockGameEnd.fields.targets` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.blockGameEnd.fields.affectedBy` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.blockGameEnd.fields.duration` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.blockGameEnd.fields.optional` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.blockGameEnd.fields.reminder` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.blockGameEnd.fields.reminderTokens` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.blockGameEnd.fields.spentReminder` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.blockGameEnd.fields.winner` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"winner":"<winner>"}` |
| `effects.blockGameEnd.fields.reason` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"reason":"<reason>"}` |
| `effects.blockGameEnd.fields.precedence` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"precedence":"<precedence>"}` |
| `effects.blockGameEnd.fields.activation` | Field accepted by blockGameEnd; its value must satisfy the typed contract. | `{"activation":"<activation>"}` |
| `effects.transformGameEnd` | Supported effects option identified by transformGameEnd. | `{"type":"transformGameEnd","operation":"invertWinners","reason":"Se invierten ganadores y perdedores."}` |
| `effects.transformGameEnd.fields.type` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.transformGameEnd.fields.when` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.transformGameEnd.fields.delay` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.transformGameEnd.fields.targets` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.transformGameEnd.fields.affectedBy` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.transformGameEnd.fields.duration` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.transformGameEnd.fields.optional` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.transformGameEnd.fields.reminder` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.transformGameEnd.fields.reminderTokens` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.transformGameEnd.fields.spentReminder` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.transformGameEnd.fields.operation` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"operation":"<operation>"}` |
| `effects.transformGameEnd.fields.reason` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"reason":"<reason>"}` |
| `effects.transformGameEnd.fields.precedence` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"precedence":"<precedence>"}` |
| `effects.transformGameEnd.fields.activation` | Field accepted by transformGameEnd; its value must satisfy the typed contract. | `{"activation":"<activation>"}` |
| `effects.startActionSequence` | Supported effects option identified by startActionSequence. | `{"type":"startActionSequence","action":"nomination","onAction":"killNominee","nextActor":"nominee","fallbackActor":"storyteller","repeatUntil":{"type":"literal","value":false}}` |
| `effects.startActionSequence.fields.type` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.startActionSequence.fields.when` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.startActionSequence.fields.delay` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.startActionSequence.fields.targets` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.startActionSequence.fields.affectedBy` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.startActionSequence.fields.duration` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.startActionSequence.fields.optional` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.startActionSequence.fields.reminder` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.startActionSequence.fields.reminderTokens` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.startActionSequence.fields.spentReminder` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.startActionSequence.fields.action` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"action":"<action>"}` |
| `effects.startActionSequence.fields.startsAtDay` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"startsAtDay":"<startsAtDay>"}` |
| `effects.startActionSequence.fields.onAction` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"onAction":"<onAction>"}` |
| `effects.startActionSequence.fields.nextActor` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"nextActor":"<nextActor>"}` |
| `effects.startActionSequence.fields.fallbackActor` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"fallbackActor":"<fallbackActor>"}` |
| `effects.startActionSequence.fields.repeatUntil` | Field accepted by startActionSequence; its value must satisfy the typed contract. | `{"repeatUntil":"<repeatUntil>"}` |
| `effects.interceptEvent` | Cancels, redirects, or replaces a matching typed event. | `{"type":"interceptEvent","event":"death","reaction":{"type":"cancel"}}` |
| `effects.interceptEvent.fields.type` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.interceptEvent.fields.when` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.interceptEvent.fields.delay` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.interceptEvent.fields.targets` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.interceptEvent.fields.affectedBy` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.interceptEvent.fields.duration` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.interceptEvent.fields.optional` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.interceptEvent.fields.reminder` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.interceptEvent.fields.reminderTokens` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.interceptEvent.fields.event` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"event":"<event>"}` |
| `effects.interceptEvent.fields.bindings` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"bindings":"<bindings>"}` |
| `effects.interceptEvent.fields.match` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"match":"<match>"}` |
| `effects.interceptEvent.fields.reaction` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"reaction":"<reaction>"}` |
| `effects.interceptEvent.fields.priority` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"priority":"<priority>"}` |
| `effects.interceptEvent.fields.consumption` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"consumption":"<consumption>"}` |
| `effects.interceptEvent.fields.appliesWhenProtectionBypassed` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"appliesWhenProtectionBypassed":"<appliesWhenProtectionBypassed>"}` |
| `effects.interceptEvent.fields.scope` | Field accepted by interceptEvent; its value must satisfy the typed contract. | `{"scope":"<scope>"}` |
| `effects.disableAbility` | Supported effects option identified by disableAbility. | `{"type":"disableAbility"}` |
| `effects.disableAbility.fields.type` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.disableAbility.fields.when` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.disableAbility.fields.delay` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.disableAbility.fields.targets` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.disableAbility.fields.affectedBy` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.disableAbility.fields.duration` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.disableAbility.fields.optional` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.disableAbility.fields.reminder` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.disableAbility.fields.reminderTokens` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.disableAbility.fields.spentReminder` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.disableAbility.fields.blocks` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"blocks":"<blocks>"}` |
| `effects.disableAbility.fields.consumeOnPass` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"consumeOnPass":"<consumeOnPass>"}` |
| `effects.disableAbility.fields.informationMayBeFalse` | Field accepted by disableAbility; its value must satisfy the typed contract. | `{"informationMayBeFalse":"<informationMayBeFalse>"}` |
| `effects.restrict` | Supported effects option identified by restrict. | `{"type":"restrict"}` |
| `effects.restrict.fields.type` | Field accepted by restrict; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.restrict.fields.when` | Field accepted by restrict; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.restrict.fields.delay` | Field accepted by restrict; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.restrict.fields.targets` | Field accepted by restrict; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.restrict.fields.affectedBy` | Field accepted by restrict; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.restrict.fields.duration` | Field accepted by restrict; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.restrict.fields.optional` | Field accepted by restrict; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.restrict.fields.reminder` | Field accepted by restrict; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.restrict.fields.reminderTokens` | Field accepted by restrict; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.restrict.fields.spentReminder` | Field accepted by restrict; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.restrict.fields.actions` | Field accepted by restrict; its value must satisfy the typed contract. | `{"actions":"<actions>"}` |
| `effects.restrict.fields.informationMayBeFalse` | Field accepted by restrict; its value must satisfy the typed contract. | `{"informationMayBeFalse":"<informationMayBeFalse>"}` |
| `effects.restrict.fields.requiresSourceAlive` | Field accepted by restrict; its value must satisfy the typed contract. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.restrict.fields.restrictions` | Field accepted by restrict; its value must satisfy the typed contract. | `{"restrictions":"<restrictions>"}` |
| `effects.restrict.fields.relation` | Field accepted by restrict; its value must satisfy the typed contract. | `{"relation":"<relation>"}` |
| `effects.restrict.fields.exception` | Field accepted by restrict; its value must satisfy the typed contract. | `{"exception":"<exception>"}` |
| `effects.registerAs` | Supported effects option identified by registerAs. | `{"type":"registerAs"}` |
| `effects.registerAs.fields.type` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.registerAs.fields.when` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.registerAs.fields.delay` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.registerAs.fields.targets` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.registerAs.fields.affectedBy` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.registerAs.fields.duration` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.registerAs.fields.optional` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.registerAs.fields.reminder` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.registerAs.fields.reminderTokens` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.registerAs.fields.spentReminder` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.registerAs.fields.affects` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"affects":"<affects>"}` |
| `effects.registerAs.fields.alignment` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"alignment":"<alignment>"}` |
| `effects.registerAs.fields.roles` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"roles":"<roles>"}` |
| `effects.registerAs.fields.characterIds` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"characterIds":"<characterIds>"}` |
| `effects.registerAs.fields.characterId` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"characterId":"<characterId>"}` |
| `effects.registerAs.fields.lifeState` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"lifeState":"<lifeState>"}` |
| `effects.registerAs.fields.mode` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"mode":"<mode>"}` |
| `effects.registerAs.fields.registersAs` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"registersAs":"<registersAs>"}` |
| `effects.registerAs.fields.teamIds` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"teamIds":"<teamIds>"}` |
| `effects.registerAs.fields.triggerReminder` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"triggerReminder":"<triggerReminder>"}` |
| `effects.registerAs.fields.worksWhenDead` | Field accepted by registerAs; its value must satisfy the typed contract. | `{"worksWhenDead":"<worksWhenDead>"}` |
| `effects.modifyTargets` | Supported effects option identified by modifyTargets. | `{"type":"modifyTargets","delta":1}` |
| `effects.modifyTargets.fields.type` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifyTargets.fields.when` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifyTargets.fields.delay` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifyTargets.fields.targets` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifyTargets.fields.affectedBy` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyTargets.fields.duration` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifyTargets.fields.optional` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifyTargets.fields.reminder` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifyTargets.fields.reminderTokens` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyTargets.fields.spentReminder` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyTargets.fields.delta` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"delta":"<delta>"}` |
| `effects.modifyTargets.fields.candidates` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"candidates":"<candidates>"}` |
| `effects.modifyTargets.fields.sourceProfile` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"sourceProfile":"<sourceProfile>"}` |
| `effects.modifyTargets.fields.targetMechanicTags` | Field accepted by modifyTargets; its value must satisfy the typed contract. | `{"targetMechanicTags":"<targetMechanicTags>"}` |
| `effects.modifyVote` | Supported effects option identified by modifyVote. | `{"type":"modifyVote","targets":{"type":"binding","binding":"selected"},"weight":2}` |
| `effects.modifyVote.fields.type` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifyVote.fields.when` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifyVote.fields.delay` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifyVote.fields.targets` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifyVote.fields.affectedBy` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyVote.fields.duration` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifyVote.fields.optional` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifyVote.fields.reminder` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifyVote.fields.reminderTokens` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyVote.fields.spentReminder` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyVote.fields.weight` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"weight":"<weight>"}` |
| `effects.modifyVote.fields.pairedTargets` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"pairedTargets":"<pairedTargets>"}` |
| `effects.modifyVote.fields.pairedWeight` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"pairedWeight":"<pairedWeight>"}` |
| `effects.modifyVote.fields.threshold` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"threshold":"<threshold>"}` |
| `effects.modifyVote.fields.purposes` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"purposes":"<purposes>"}` |
| `effects.modifyVote.fields.electorate` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"electorate":"<electorate>"}` |
| `effects.modifyVote.fields.creditRequired` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"creditRequired":"<creditRequired>"}` |
| `effects.modifyVote.fields.requiredVoters` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"requiredVoters":"<requiredVoters>"}` |
| `effects.modifyVote.fields.tallyValidity` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"tallyValidity":"<tallyValidity>"}` |
| `effects.modifyVote.fields.worksWhenDead` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"worksWhenDead":"<worksWhenDead>"}` |
| `effects.modifySetup` | Supported effects option identified by modifySetup. | `{"type":"modifySetup","operations":[{"type":"adjustBucket","bucket":"setupBucket","delta":0}]}` |
| `effects.modifySetup.fields.type` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifySetup.fields.when` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifySetup.fields.delay` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifySetup.fields.targets` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifySetup.fields.affectedBy` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifySetup.fields.duration` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifySetup.fields.optional` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifySetup.fields.reminder` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifySetup.fields.reminderTokens` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifySetup.fields.spentReminder` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifySetup.fields.operations` | Field accepted by modifySetup; its value must satisfy the typed contract. | `{"operations":"<operations>"}` |
| `effects.restrictSetupCombination` | Supported effects option identified by restrictSetupCombination. | `{"type":"restrictSetupCombination","characterIds":["character:invented:first","character:invented:second"],"maximum":1}` |
| `effects.restrictSetupCombination.fields.type` | Field accepted by restrictSetupCombination; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.restrictSetupCombination.fields.when` | Field accepted by restrictSetupCombination; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.restrictSetupCombination.fields.characterIds` | Field accepted by restrictSetupCombination; its value must satisfy the typed contract. | `{"characterIds":"<characterIds>"}` |
| `effects.restrictSetupCombination.fields.maximum` | Field accepted by restrictSetupCombination; its value must satisfy the typed contract. | `{"maximum":"<maximum>"}` |
| `effects.modifyInformation` | Supported effects option identified by modifyInformation. | `{"type":"modifyInformation"}` |
| `effects.modifyInformation.fields.type` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifyInformation.fields.when` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifyInformation.fields.delay` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifyInformation.fields.targets` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifyInformation.fields.affectedBy` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyInformation.fields.duration` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifyInformation.fields.optional` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifyInformation.fields.reminder` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifyInformation.fields.reminderTokens` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyInformation.fields.spentReminder` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyInformation.fields.audience` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"audience":"<audience>"}` |
| `effects.modifyInformation.fields.mustBeFalse` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"mustBeFalse":"<mustBeFalse>"}` |
| `effects.modifyInformation.fields.redact` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"redact":"<redact>"}` |
| `effects.modifyInformation.fields.redactValues` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"redactValues":"<redactValues>"}` |
| `effects.modifyInformation.fields.redactCharacterTokens` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"redactCharacterTokens":"<redactCharacterTokens>"}` |
| `effects.modifyInformation.fields.sourceCharacterIds` | Field accepted by modifyInformation; its value must satisfy the typed contract. | `{"sourceCharacterIds":"<sourceCharacterIds>"}` |
| `effects.modifyStartingKnowledge` | Supported effects option identified by modifyStartingKnowledge. | `{"type":"modifyStartingKnowledge","steps":["evilTeamRecognition"],"active":false}` |
| `effects.modifyStartingKnowledge.fields.type` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifyStartingKnowledge.fields.when` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifyStartingKnowledge.fields.delay` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifyStartingKnowledge.fields.targets` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifyStartingKnowledge.fields.affectedBy` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyStartingKnowledge.fields.duration` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifyStartingKnowledge.fields.optional` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifyStartingKnowledge.fields.reminder` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifyStartingKnowledge.fields.reminderTokens` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyStartingKnowledge.fields.spentReminder` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyStartingKnowledge.fields.steps` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"steps":"<steps>"}` |
| `effects.modifyStartingKnowledge.fields.active` | Field accepted by modifyStartingKnowledge; its value must satisfy the typed contract. | `{"active":"<active>"}` |
| `effects.modifyNomination` | Supported effects option identified by modifyNomination. | `{"type":"modifyNomination"}` |
| `effects.modifyNomination.fields.type` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.modifyNomination.fields.when` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.modifyNomination.fields.delay` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.modifyNomination.fields.targets` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.modifyNomination.fields.affectedBy` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyNomination.fields.duration` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.modifyNomination.fields.optional` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.modifyNomination.fields.reminder` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.modifyNomination.fields.reminderTokens` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyNomination.fields.spentReminder` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyNomination.fields.allowsStorytellerNominee` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"allowsStorytellerNominee":"<allowsStorytellerNominee>"}` |
| `effects.modifyNomination.fields.abilitySpentEvenIfNoExecution` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"abilitySpentEvenIfNoExecution":"<abilitySpentEvenIfNoExecution>"}` |
| `effects.modifyNomination.fields.countsAsExecution` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"countsAsExecution":"<countsAsExecution>"}` |
| `effects.modifyNomination.fields.createsReminder` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"createsReminder":"<createsReminder>"}` |
| `effects.modifyNomination.fields.voteDelta` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"voteDelta":"<voteDelta>"}` |
| `effects.modifyNomination.fields.requiresActorAbstention` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"requiresActorAbstention":"<requiresActorAbstention>"}` |
| `effects.modifyNomination.fields.maxNominationsPerDay` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"maxNominationsPerDay":"<maxNominationsPerDay>"}` |
| `effects.modifyNomination.fields.worksWhenDead` | Field accepted by modifyNomination; its value must satisfy the typed contract. | `{"worksWhenDead":"<worksWhenDead>"}` |
| `effects.performTableAction` | Supported effects option identified by performTableAction. | `{"type":"performTableAction","action":"devour","targets":{"type":"binding","binding":"selected"},"consequences":[]}` |
| `effects.performTableAction.fields.type` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.performTableAction.fields.when` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.performTableAction.fields.delay` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.performTableAction.fields.targets` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.performTableAction.fields.affectedBy` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.performTableAction.fields.duration` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.performTableAction.fields.optional` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.performTableAction.fields.reminder` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.performTableAction.fields.reminderTokens` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.performTableAction.fields.spentReminder` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.performTableAction.fields.action` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"action":"<action>"}` |
| `effects.performTableAction.fields.consequences` | Field accepted by performTableAction; its value must satisfy the typed contract. | `{"consequences":"<consequences>"}` |
| `effects.recordAction` | Supported effects option identified by recordAction. | `{"type":"recordAction"}` |
| `effects.recordAction.fields.type` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.recordAction.fields.when` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.recordAction.fields.delay` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.recordAction.fields.targets` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.recordAction.fields.affectedBy` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.recordAction.fields.duration` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.recordAction.fields.optional` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.recordAction.fields.reminder` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.recordAction.fields.reminderTokens` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.recordAction.fields.spentReminder` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.recordAction.fields.action` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"action":"<action>"}` |
| `effects.recordAction.fields.actionId` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"actionId":"<actionId>"}` |
| `effects.recordAction.fields.outcome` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"outcome":"<outcome>"}` |
| `effects.recordAction.fields.creates` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"creates":"<creates>"}` |
| `effects.recordAction.fields.effect` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"effect":"<effect>"}` |
| `effects.recordAction.fields.failureEffect` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"failureEffect":"<failureEffect>"}` |
| `effects.recordAction.fields.failureOutcome` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"failureOutcome":"<failureOutcome>"}` |
| `effects.recordAction.fields.maxGuesses` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"maxGuesses":"<maxGuesses>"}` |
| `effects.recordAction.fields.resolvesExecution` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"resolvesExecution":"<resolvesExecution>"}` |
| `effects.recordAction.fields.restrictions` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"restrictions":"<restrictions>"}` |
| `effects.recordAction.fields.successOutcome` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"successOutcome":"<successOutcome>"}` |
| `effects.recordAction.fields.targetMechanicTags` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"targetMechanicTags":"<targetMechanicTags>"}` |
| `effects.recordAction.fields.targetRegistrationTeams` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"targetRegistrationTeams":"<targetRegistrationTeams>"}` |
| `effects.recordAction.fields.recordAs` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"recordAs":"<recordAs>"}` |
| `effects.recordAction.fields.recordCorrectGuesses` | Field accepted by recordAction; its value must satisfy the typed contract. | `{"recordCorrectGuesses":"<recordCorrectGuesses>"}` |
| `effects.storytellerDecision` | Supported effects option identified by storytellerDecision. | `{"type":"storytellerDecision","decision":"decision"}` |
| `effects.storytellerDecision.fields.type` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.storytellerDecision.fields.when` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.storytellerDecision.fields.delay` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.storytellerDecision.fields.targets` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.storytellerDecision.fields.affectedBy` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.storytellerDecision.fields.duration` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.storytellerDecision.fields.optional` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.storytellerDecision.fields.reminder` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.storytellerDecision.fields.reminderTokens` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.storytellerDecision.fields.spentReminder` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.storytellerDecision.fields.decision` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"decision":"<decision>"}` |
| `effects.storytellerDecision.fields.options` | Field accepted by storytellerDecision; its value must satisfy the typed contract. | `{"options":"<options>"}` |
| `effects.manualCheckpoint` | Supported effects option identified by manualCheckpoint. | `{"type":"manualCheckpoint","reason":"storytellerJudgment","prompt":"Confirma el resultado.","outcomes":[{"id":"confirmed","label":"Confirmado","effects":[]}],"blocking":true}` |
| `effects.manualCheckpoint.fields.type` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.manualCheckpoint.fields.when` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.manualCheckpoint.fields.delay` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.manualCheckpoint.fields.targets` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.manualCheckpoint.fields.affectedBy` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.manualCheckpoint.fields.duration` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.manualCheckpoint.fields.optional` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.manualCheckpoint.fields.reminder` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.manualCheckpoint.fields.reminderTokens` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.manualCheckpoint.fields.spentReminder` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.manualCheckpoint.fields.reason` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"reason":"<reason>"}` |
| `effects.manualCheckpoint.fields.prompt` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"prompt":"<prompt>"}` |
| `effects.manualCheckpoint.fields.outcomes` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"outcomes":"<outcomes>"}` |
| `effects.manualCheckpoint.fields.blocking` | Field accepted by manualCheckpoint; its value must satisfy the typed contract. | `{"blocking":"<blocking>"}` |
| `effects.manualInstruction` | Supported effects option identified by manualInstruction. | `{"type":"manualInstruction","instruction":"Describe cómo resolver esta regla."}` |
| `effects.manualInstruction.fields.type` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.manualInstruction.fields.when` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.manualInstruction.fields.delay` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.manualInstruction.fields.targets` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.manualInstruction.fields.affectedBy` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.manualInstruction.fields.duration` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `effects.manualInstruction.fields.optional` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"optional":"<optional>"}` |
| `effects.manualInstruction.fields.reminder` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"reminder":"<reminder>"}` |
| `effects.manualInstruction.fields.reminderTokens` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.manualInstruction.fields.spentReminder` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"spentReminder":"<spentReminder>"}` |
| `effects.manualInstruction.fields.affects` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"affects":"<affects>"}` |
| `effects.manualInstruction.fields.ruleStepActivation` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"ruleStepActivation":"<ruleStepActivation>"}` |
| `effects.manualInstruction.fields.aliveThreshold` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"aliveThreshold":"<aliveThreshold>"}` |
| `effects.manualInstruction.fields.bluffCountOptions` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"bluffCountOptions":"<bluffCountOptions>"}` |
| `effects.manualInstruction.fields.durationMinutes` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"durationMinutes":"<durationMinutes>"}` |
| `effects.manualInstruction.fields.instruction` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"instruction":"<instruction>"}` |
| `effects.manualInstruction.fields.publicKnown` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"publicKnown":"<publicKnown>"}` |
| `effects.manualInstruction.fields.reason` | Field accepted by manualInstruction; its value must satisfy the typed contract. | `{"reason":"<reason>"}` |
| `policies.recordTargetSelectionWhenDisabled` | Supported policies option identified by recordTargetSelectionWhenDisabled. | `{"type":"recordTargetSelectionWhenDisabled"}` |
| `policies.recordTargetSelectionWhenDisabled.fields.type` | Field accepted by recordTargetSelectionWhenDisabled; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.presentStartingInformationAsShownIdentity` | Supported policies option identified by presentStartingInformationAsShownIdentity. | `{"type":"presentStartingInformationAsShownIdentity"}` |
| `policies.presentStartingInformationAsShownIdentity.fields.type` | Field accepted by presentStartingInformationAsShownIdentity; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.relayShownSelection` | Supported policies option identified by relayShownSelection. | `{"type":"relayShownSelection"}` |
| `policies.relayShownSelection.fields.type` | Field accepted by relayShownSelection; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.assignSelectedCharacter` | Supported policies option identified by assignSelectedCharacter. | `{"type":"assignSelectedCharacter"}` |
| `policies.assignSelectedCharacter.fields.type` | Field accepted by assignSelectedCharacter; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.grantSelectedAbility` | Supported policies option identified by grantSelectedAbility. | `{"type":"grantSelectedAbility"}` |
| `policies.grantSelectedAbility.fields.type` | Field accepted by grantSelectedAbility; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.grantSelectedAbility.fields.target` | Field accepted by grantSelectedAbility; its value must satisfy the typed contract. | `{"target":"<target>"}` |
| `policies.grantSelectedAbility.fields.abilityAlignment` | Field accepted by grantSelectedAbility; its value must satisfy the typed contract. | `{"abilityAlignment":"<abilityAlignment>"}` |
| `policies.grantSelectedAbility.fields.duration` | Field accepted by grantSelectedAbility; its value must satisfy the typed contract. | `{"duration":"<duration>"}` |
| `policies.grantSelectedAbility.fields.requireInPlay` | Field accepted by grantSelectedAbility; its value must satisfy the typed contract. | `{"requireInPlay":"<requireInPlay>"}` |
| `policies.overrideChooserAlignment` | Supported policies option identified by overrideChooserAlignment. | `{"type":"overrideChooserAlignment"}` |
| `policies.overrideChooserAlignment.fields.type` | Field accepted by overrideChooserAlignment; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.overrideChooserAlignment.fields.alignment` | Field accepted by overrideChooserAlignment; its value must satisfy the typed contract. | `{"alignment":"<alignment>"}` |
| `policies.continueAfterTargetReaction` | Supported policies option identified by continueAfterTargetReaction. | `{"type":"continueAfterTargetReaction"}` |
| `policies.continueAfterTargetReaction.fields.type` | Field accepted by continueAfterTargetReaction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.suppressStandaloneNightStep` | Supported policies option identified by suppressStandaloneNightStep. | `{"type":"suppressStandaloneNightStep"}` |
| `policies.suppressStandaloneNightStep.fields.type` | Field accepted by suppressStandaloneNightStep; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.ignoreActorAbilityRestriction` | Supported policies option identified by ignoreActorAbilityRestriction. | `{"type":"ignoreActorAbilityRestriction"}` |
| `policies.ignoreActorAbilityRestriction.fields.type` | Field accepted by ignoreActorAbilityRestriction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.allowDeadActorWithPendingAction` | Supported policies option identified by allowDeadActorWithPendingAction. | `{"type":"allowDeadActorWithPendingAction"}` |
| `policies.allowDeadActorWithPendingAction.fields.type` | Field accepted by allowDeadActorWithPendingAction; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.allowDeadActorWithPendingAction.fields.actionId` | Field accepted by allowDeadActorWithPendingAction; its value must satisfy the typed contract. | `{"actionId":"<actionId>"}` |
| `policies.requireRecordedTargetAlignment` | Supported policies option identified by requireRecordedTargetAlignment. | `{"type":"requireRecordedTargetAlignment"}` |
| `policies.requireRecordedTargetAlignment.fields.type` | Field accepted by requireRecordedTargetAlignment; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.requireRecordedTargetAlignment.fields.actionId` | Field accepted by requireRecordedTargetAlignment; its value must satisfy the typed contract. | `{"actionId":"<actionId>"}` |
| `policies.requireRecordedTargetAlignment.fields.alignment` | Field accepted by requireRecordedTargetAlignment; its value must satisfy the typed contract. | `{"alignment":"<alignment>"}` |
| `policies.limitSelectionByRecordedActionCount` | Supported policies option identified by limitSelectionByRecordedActionCount. | `{"type":"limitSelectionByRecordedActionCount"}` |
| `policies.limitSelectionByRecordedActionCount.fields.type` | Field accepted by limitSelectionByRecordedActionCount; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `policies.limitSelectionByRecordedActionCount.fields.actionId` | Field accepted by limitSelectionByRecordedActionCount; its value must satisfy the typed contract. | `{"actionId":"<actionId>"}` |
| `policies.limitSelectionByRecordedActionCount.fields.period` | Field accepted by limitSelectionByRecordedActionCount; its value must satisfy the typed contract. | `{"period":"<period>"}` |
