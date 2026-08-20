# BloodScribe mechanic authoring guide

This reference is generated from the same contract used by the MCP. Examples use invented IDs.

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
- Which part, if any, requires an explicit Storyteller decision?

## Intent recipes

### `registered-evil-neighbours` — Check evil neighbours

Learn whether at least one of the two nearest living neighbours registers as evil.

- Status: `supported`
- Automation: `automatic`
- Covers: `entities.players`, `relations.nearestMatching`, `identityModes.registered`, `effects.emitInformation`

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
- Covers: `usage.keyBy`, `usageDimensions.day`, `usageDimensions.actor`, `effects.recordAction`

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
- Covers: `entities.markers`, `marker.project.sourcePlayerId`, `effects.applyMarker`, `effects.emitInformation`

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
- Covers: `effects.moveMarker`, `events.markerChange`, `marker.project.ownership`

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
- Covers: `events.mechanicUse`, `eventFields.resolutionStatus`, `usage.keyBy`, `usageDimensions.night`

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
- Covers: `events.mechanicUse`, `predicates.eventParticipantIdentity`, `identityModes.registered`, `effects.emitInformation`

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
| `eventFields.died` | Supported event fields option identified by died. | `{"type":"eventField","field":"died","value":"<value>"}` |
| `eventFields.attribution` | Supported event fields option identified by attribution. | `{"type":"eventField","field":"attribution","value":"<value>"}` |
| `eventFields.resolution` | Supported event fields option identified by resolution. | `{"type":"eventField","field":"resolution","value":"<value>"}` |
| `eventFields.known` | Supported event fields option identified by known. | `{"type":"eventField","field":"known","value":"<value>"}` |
| `eventFields.occurrence` | Supported event fields option identified by occurrence. | `{"type":"eventField","field":"occurrence","value":"<value>"}` |
| `eventFields.signal` | Supported event fields option identified by signal. | `{"type":"eventField","field":"signal","value":"<value>"}` |
| `eventFields.characterId` | Supported event fields option identified by characterId. | `{"type":"eventField","field":"characterId","value":"<value>"}` |
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
| `effects.moveMarker` | Atomically transfers an existing marker between players. | `{"type":"moveMarker","kind":"reminder","id":"marker","from":{"type":"binding","binding":"actor"},"targets":{"type":"binding","binding":"selected"}}` |
| `effects.moveMarker.fields.type` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"type":"<type>"}` |
| `effects.moveMarker.fields.when` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"when":"<when>"}` |
| `effects.moveMarker.fields.delay` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"delay":"<delay>"}` |
| `effects.moveMarker.fields.targets` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"targets":"<targets>"}` |
| `effects.moveMarker.fields.affectedBy` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"affectedBy":"<affectedBy>"}` |
| `effects.moveMarker.fields.kind` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"kind":"<kind>"}` |
| `effects.moveMarker.fields.id` | Field accepted by moveMarker; its value must satisfy the typed contract. | `{"id":"<id>"}` |
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
| `effects.modifyVote.fields.tallyValidity` | Field accepted by modifyVote; its value must satisfy the typed contract. | `{"tallyValidity":"<tallyValidity>"}` |
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
