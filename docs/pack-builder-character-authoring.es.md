# Guía de autoría mecánica de BloodScribe

Esta referencia se genera desde el mismo contrato que consume el MCP. Los ejemplos usan IDs inventados.

## Preguntas de diseño

- ¿Qué resultado observable debe producir la habilidad?
- ¿Cuándo se activa y con qué frecuencia?
- ¿Quién actúa, quién puede ser objetivo y pueden elegirse muertos o el propio actor?
- ¿Debe consultar identidad real, inicial, mostrada o registrada?
- ¿Qué comparte el límite: día, noche, actor, objetivo o evento?
- ¿Cuándo termina cualquier estado, restricción o ficha persistente?
- ¿Qué debe ocurrir si la habilidad falla, se bloquea, se redirige o está perjudicada?
- ¿Qué parte, si alguna, necesita una decisión explícita del Narrador?

## Recetas por intención

### `registered-evil-neighbours` — Comprobar vecinos malvados

Saber si al menos uno de los dos vecinos vivos más cercanos se registra como malvado.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `entities.players`, `relations.nearestMatching`, `identityModes.registered`, `effects.emitInformation`

nearestMatching obtiene un vecino vivo en cada dirección; la consulta proyecta su bando registrado y emitInformation entrega un sí/no.

**Preguntar**

- ¿Deben saltarse los vecinos muertos?
- ¿Debe consultarse el bando real o el registrado?

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

### `once-per-actor-each-day` — Una vez por actor cada día

Permitir una resolución independiente por actor y por día.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `usage.keyBy`, `usageDimensions.day`, `usageDimensions.actor`, `effects.recordAction`

keyBy compone las dimensiones day y actor en una única clave de uso.

**Preguntar**

- ¿El uso se consume al intentar, al resolver o solo al acertar?

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

### `state-until-nomination` — Estado hasta una nominación

Aplicar un estado que termina automáticamente cuando ocurre una nominación.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `durations.untilEvent`, `events.nomination`, `effects.setPlayerState`

La duración untilEvent conserva el patrón tipado que retira el estado.

**Preguntar**

- ¿Debe terminar con cualquier nominación o solo si participa el objetivo?

**Límites**

- Este ejemplo termina con cualquier nominación.

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

### `learn-marker-source` — Conocer el origen de una ficha

Conocer qué jugador originó una ficha recordatoria concreta.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `entities.markers`, `marker.project.sourcePlayerId`, `effects.applyMarker`, `effects.emitInformation`

Las fichas conservan sourcePlayerId y sourceCharacterId; una consulta puede proyectar esa procedencia.

**Preguntar**

- ¿La ficha conserva la propiedad de su fuente o debe ser independiente?

**Límites**

- Si hay varias fichas iguales, añade una condición que garantice cuál debe consultarse.

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

### `atomic-marker-transfer` — Transferir una ficha atómicamente

Mover una ficha del actor a un objetivo conservando identidad, procedencia y propiedad.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `effects.moveMarker`, `events.markerChange`, `marker.project.ownership`

moveMarker emite un único cambio y conserva los metadatos de la ficha existente.

**Preguntar**

- ¿De quién sale la ficha y puede faltar?

**Límites**

- La ficha debe existir exactamente una vez en el origen.

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

### `learn-impaired-ability-user` — Descubrir una habilidad perjudicada

Saber quién usó una habilidad que funcionó mal durante la noche.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `entities.events`, `eventFields.actorAbilityMode`, `events.mechanicUse`, `effects.emitInformation`

El historial mechanicUse captura actorAbilityMode y el participante actor en el momento de resolver.

**Preguntar**

- ¿Cuenta una habilidad totalmente deshabilitada o solo una que funcionó mal?

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

### `learn-protected-target` — Saber quién fue protegido

Identificar un objetivo cuya resolución fue impedida por una protección.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `eventFields.targetResultStatus`, `events.mechanicUse`, `effects.emitInformation`

targetResultStatus permite filtrar resoluciones prevented y recuperar el objetivo seleccionado capturado.

**Preguntar**

- ¿Debe distinguir protección, inmunidad y objetivo inválido?

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

### `learn-copied-ability-user` — Descubrir una habilidad copiada

Saber quién utilizó durante la noche una habilidad copiada.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `eventFields.abilityProvenance`, `events.mechanicUse`, `effects.emitInformation`

abilityProvenance conserva la procedencia de la instancia de habilidad usada.

**Preguntar**

- ¿También cuentan habilidades concedidas o prestadas?

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

### `first-successful-ability-each-night` — Primera habilidad exitosa de la noche

Reaccionar una sola vez a la primera habilidad que termina con éxito cada noche.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `events.mechanicUse`, `eventFields.resolutionStatus`, `usage.keyBy`, `usageDimensions.night`

El trigger filtra mechanicUse exitosos y keyBy night limita la reacción a una por noche.

**Preguntar**

- ¿La reacción debe consumir uso al resolver o solo si su propio efecto tiene éxito?

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

### `react-to-redirected-ability` — Reaccionar a una habilidad redirigida

Activar una reacción cuando la resolución de una habilidad fue redirigida.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `events.mechanicUse`, `eventFields.targetResultStatus`, `effects.recordAction`

El trigger filtra eventos mechanicUse cuyo resultado de objetivo es redirected.

**Preguntar**

- ¿Debe reaccionar a cualquier redirección o solo a una habilidad concreta?

**Límites**

- Si una resolución tiene varios resultados distintos, filtra por una categoría o mecánica más concreta.

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

### `react-after-failed-ability` — Reaccionar después de un fallo

Activar una reacción después de que una habilidad termine con estado failed.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `events.mechanicUse`, `eventFields.resolutionStatus`, `effects.recordAction`

resolutionStatus distingue el fallo completo de resultados parciales como noEffect.

**Preguntar**

- ¿Debe contar un objetivo sin efecto o solo un fallo completo de resolución?

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

### `count-effective-abilities-on-player` — Contar habilidades que afectaron a un jugador

Contar cuántas resoluciones tuvieron realmente a un jugador como objetivo efectivo esta noche.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `events.mechanicUse`, `bindings.effectTarget`, `aggregates.count`, `effects.emitInformation`

effectTarget conserva los objetivos efectivos después de protecciones y redirecciones; count cuenta las resoluciones coincidentes.

**Preguntar**

- ¿Debe contar resoluciones o habilidades únicas?

**Límites**

- Este ejemplo cuenta resoluciones, no IDs de habilidad únicos.

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

### `registered-evil-used-ability-on-good` — Malvado registrado actuó sobre bueno

Descubrir si un actor registrado como malvado usó una habilidad sobre alguien registrado como bueno esta noche.

- Estado: `supported`
- Automatización: `automatic`
- Cubre: `events.mechanicUse`, `predicates.eventParticipantIdentity`, `identityModes.registered`, `effects.emitInformation`

El evento conserva las identidades registradas del actor y del objetivo seleccionado en el momento de la resolución.

**Preguntar**

- ¿Debe usarse identidad real o registrada para cada participante?

**Límites**

- Este ejemplo comprueba el objetivo seleccionado, no el objetivo efectivo redirigido.

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

### `repeat-ability-after-no-effect` — Repetir una habilidad sin efecto

Volver a ejecutar de forma genérica la misma instancia de habilidad cuando no produjo efecto.

- Estado: `unsupported`
- Automatización: `manual`
- Cubre: `events.mechanicUse`, `eventFields.targetResultStatus`

El historial puede detectar noEffect, pero no existe una primitiva genérica para volver a invocar una instancia arbitraria.

**Límites**

- No simules la repetición con texto ni IDs.

**Primitiva ausente:** Una operación tipada que vuelva a invocar la resolución identificada conservando actor, selección y procedencia.

### `learn-altered-information-recipient` — Saber quién recibió información alterada

Consultar posteriormente el destinatario exacto de una información transformada.

- Estado: `unsupported`
- Automatización: `manual`
- Cubre: `effects.modifyInformation`, `events.mechanicUse`

modifyInformation puede alterar la entrega, pero el ledger no expone todavía un atributo informationAltered consultable.

**Límites**

- No infieras este dato desde el texto mostrado.

**Primitiva ausente:** Un atributo de evento que registre transformación de información y sus destinatarios efectivos.

## Matriz exhaustiva

| ID | Qué hace | Fragmento |
|---|---|---|
| `valueNodes.literal` | Usa un valor declarado. | `{"type":"literal"}` |
| `valueNodes.binding` | Usa un participante vinculado por la regla. | `{"type":"binding"}` |
| `valueNodes.game` | Usa fase, día o noche actuales. | `{"type":"game"}` |
| `valueNodes.setup` | Usa un valor preparado durante el setup. | `{"type":"setup"}` |
| `valueNodes.fact` | Usa un resultado tipado de la resolución actual. | `{"type":"fact"}` |
| `valueNodes.storytellerDecision` | Usa una decisión humana declarada por la regla. | `{"type":"storytellerDecision"}` |
| `valueNodes.decisionValue` | Usa el valor de una decisión humana ya resuelta. | `{"type":"decisionValue"}` |
| `valueNodes.all` | Exige que todas las condiciones sean verdaderas. | `{"type":"all"}` |
| `valueNodes.any` | Exige que al menos una condición sea verdadera. | `{"type":"any"}` |
| `valueNodes.not` | Invierte una condición booleana. | `{"type":"not"}` |
| `valueNodes.query` | Busca, filtra y obtiene entidades de la partida. | `{"type":"query"}` |
| `valueNodes.compare` | Compara dos valores ya resueltos. | `{"type":"compare"}` |
| `valueNodes.array` | Agrupa varios valores en orden. | `{"type":"array"}` |
| `valueNodes.object` | Agrupa valores con nombres declarados. | `{"type":"object"}` |
| `valueNodes.concat` | Une varias listas ya resueltas. | `{"type":"concat"}` |
| `valueNodes.set` | Une, intersecta o resta listas sin duplicados. | `{"type":"set"}` |
| `valueNodes.math` | Combina valores numéricos con una operación aritmética. | `{"type":"math"}` |
| `valueNodes.unique` | Elimina valores repetidos de una lista. | `{"type":"unique"}` |
| `valueNodes.length` | Obtiene la longitud de una lista o texto. | `{"type":"length"}` |
| `valueNodes.take` | Conserva los primeros resultados de una lista. | `{"type":"take"}` |
| `valueNodes.at` | Obtiene un elemento de una lista por su posición. | `{"type":"at"}` |
| `valueNodes.if` | Elige uno de dos valores mediante una condición booleana. | `{"type":"if"}` |
| `valueNodes.inputValue` | Usa un valor tipado introducido durante la resolución. | `{"type":"inputValue"}` |
| `terminology.teamId` | Categoría visible declarada por el pack; no selecciona comportamiento. | `{"value":"teamId"}` |
| `events.characterEntry` | Un personaje entra en una plaza del reparto o independiente. | `{"type":"event","event":"characterEntry"}` |
| `events.death` | Un jugador muere o evita morir. | `{"type":"event","event":"death"}` |
| `events.execution` | Se resuelve una ejecución. | `{"type":"event","event":"execution"}` |
| `events.exile` | Se resuelve el exilio de un Viajero. | `{"type":"event","event":"exile"}` |
| `events.nomination` | Se registra una nominación y su votación. | `{"type":"event","event":"nomination"}` |
| `events.publicAction` | Se resuelve una acción pública declarada. | `{"type":"event","event":"publicAction"}` |
| `events.mechanicUse` | Se registra el intento o la resolución de una mecánica declarada. | `{"type":"event","event":"mechanicUse"}` |
| `events.mechanicTargeted` | Una mecánica selecciona un objetivo. | `{"type":"event","event":"mechanicTargeted"}` |
| `events.stateChange` | Cambia un estado tipado de un jugador. | `{"type":"event","event":"stateChange"}` |
| `events.restrictionChange` | Añade o retira restricciones declarativas de un jugador. | `{"type":"event","event":"restrictionChange"}` |
| `events.markerChange` | Añade, retira o mueve una ficha tipada. | `{"type":"event","event":"markerChange"}` |
| `events.abilityGrant` | Concede o retira una habilidad a un jugador o a la partida. | `{"type":"event","event":"abilityGrant"}` |
| `events.storytellerSignal` | El Narrador emite una señal semántica. | `{"type":"event","event":"storytellerSignal"}` |
| `events.storytellerExecution` | Se ejecuta al Narrador como objetivo no jugador. | `{"type":"event","event":"storytellerExecution"}` |
| `eventBindings.actor` | Jugador que ejecuta la regla. | `{"value":"actor"}` |
| `eventBindings.selected` | Jugador o jugadores elegidos. | `{"value":"selected"}` |
| `eventBindings.source` | Jugador que origina el efecto. | `{"value":"source"}` |
| `eventBindings.nominator` | Jugador que nomina. | `{"value":"nominator"}` |
| `eventBindings.nominee` | Jugador nominado. | `{"value":"nominee"}` |
| `eventBindings.voter` | Jugador o jugadores que votan. | `{"value":"voter"}` |
| `eventBindings.eventSubject` | Participante principal del evento. | `{"value":"eventSubject"}` |
| `bindings.actor` | Jugador que ejecuta la regla. | `{"value":"actor"}` |
| `bindings.selected` | Jugador o jugadores elegidos. | `{"value":"selected"}` |
| `bindings.source` | Jugador que origina el efecto. | `{"value":"source"}` |
| `bindings.nominator` | Jugador que nomina. | `{"value":"nominator"}` |
| `bindings.nominee` | Jugador nominado. | `{"value":"nominee"}` |
| `bindings.voter` | Jugador o jugadores que votan. | `{"value":"voter"}` |
| `bindings.eventSubject` | Participante principal del evento. | `{"value":"eventSubject"}` |
| `bindings.effectTarget` | Objetivo actual de un efecto compuesto. | `{"value":"effectTarget"}` |
| `entities.players` | Jugadores sentados en la mesa. | `{"value":"players"}` |
| `entities.characters` | Identidades disponibles o en juego. | `{"value":"characters"}` |
| `entities.events` | Historial semántico de la partida. | `{"value":"events"}` |
| `entities.markers` | Fichas y estados proyectados. | `{"value":"markers"}` |
| `predicateTypes.players.alive` | Comprueba si el jugador está vivo. | `{"value":"alive"}` |
| `predicateTypes.players.identity` | Compara una faceta de la identidad del jugador. | `{"value":"identity"}` |
| `predicateTypes.players.assignment` | Compara el tipo de asiento del jugador. | `{"value":"assignment"}` |
| `predicateTypes.players.identityMatchesBinding` | Compara una faceta con otro participante vinculado. | `{"value":"identityMatchesBinding"}` |
| `predicateTypes.players.identityMatchesInput` | Compara una faceta con el valor introducido durante la resolución. | `{"value":"identityMatchesInput"}` |
| `predicateTypes.players.state` | Comprueba un estado persistente del jugador. | `{"value":"state"}` |
| `predicateTypes.players.marker` | Comprueba una ficha concreta del jugador. | `{"value":"marker"}` |
| `predicateTypes.players.isBinding` | Comprueba si es el participante indicado. | `{"value":"isBinding"}` |
| `predicateTypes.players.all` | Exige que se cumplan todas las condiciones. | `{"value":"all"}` |
| `predicateTypes.players.any` | Exige que se cumpla al menos una condición. | `{"value":"any"}` |
| `predicateTypes.players.not` | Invierte el resultado de otra condición. | `{"value":"not"}` |
| `predicateTypes.characters.inPlay` | Comprueba si el personaje está en juego. | `{"value":"inPlay"}` |
| `predicateTypes.characters.identity` | Compara una faceta de la identidad del personaje. | `{"value":"identity"}` |
| `predicateTypes.characters.identityMatchesBinding` | Compara una faceta del personaje con un participante vinculado. | `{"value":"identityMatchesBinding"}` |
| `predicateTypes.characters.identityMatchesInput` | Compara una faceta del personaje con el valor introducido durante la resolución. | `{"value":"identityMatchesInput"}` |
| `predicateTypes.characters.all` | Exige que se cumplan todas las condiciones. | `{"value":"all"}` |
| `predicateTypes.characters.any` | Exige que se cumpla al menos una condición. | `{"value":"any"}` |
| `predicateTypes.characters.not` | Invierte el resultado de otra condición. | `{"value":"not"}` |
| `predicateTypes.events.eventType` | Compara el tipo semántico del evento. | `{"value":"eventType"}` |
| `predicateTypes.events.eventCategory` | Compara la categoría semántica del evento. | `{"value":"eventCategory"}` |
| `predicateTypes.events.eventPeriod` | Limita el evento a un periodo de la partida. | `{"value":"eventPeriod"}` |
| `predicateTypes.events.eventField` | Compara un campo tipado del evento. | `{"value":"eventField"}` |
| `predicateTypes.events.eventRestriction` | Compara las restricciones declaradas por el efecto. | `{"value":"eventRestriction"}` |
| `predicateTypes.events.eventParticipant` | Comprueba un participante del evento. | `{"value":"eventParticipant"}` |
| `predicateTypes.events.eventParticipantIdentity` | Compara la identidad de un participante del evento. | `{"value":"eventParticipantIdentity"}` |
| `predicateTypes.events.all` | Exige que se cumplan todas las condiciones. | `{"value":"all"}` |
| `predicateTypes.events.any` | Exige que se cumpla al menos una condición. | `{"value":"any"}` |
| `predicateTypes.events.not` | Invierte el resultado de otra condición. | `{"value":"not"}` |
| `predicateTypes.markers.markerKind` | Compara el tipo semántico de la ficha. | `{"value":"markerKind"}` |
| `predicateTypes.markers.markerId` | Compara el identificador estable de la ficha. | `{"value":"markerId"}` |
| `predicateTypes.markers.markerActive` | Comprueba si la ficha está activa. | `{"value":"markerActive"}` |
| `predicateTypes.markers.all` | Exige que se cumplan todas las condiciones. | `{"value":"all"}` |
| `predicateTypes.markers.any` | Exige que se cumpla al menos una condición. | `{"value":"any"}` |
| `predicateTypes.markers.not` | Invierte el resultado de otra condición. | `{"value":"not"}` |
| `aggregates.collect` | Devuelve todos los resultados. | `{"value":"collect"}` |
| `aggregates.count` | Cuenta los resultados. | `{"value":"count"}` |
| `aggregates.exists` | Indica si hay algún resultado. | `{"value":"exists"}` |
| `aggregates.all` | Indica si todos cumplen. | `{"value":"all"}` |
| `aggregates.exactly` | Comprueba una cantidad exacta. | `{"value":"exactly"}` |
| `aggregates.first` | Devuelve el primer resultado. | `{"value":"first"}` |
| `aggregates.last` | Devuelve el último resultado. | `{"value":"last"}` |
| `aggregates.nearest` | Devuelve el resultado más cercano. | `{"value":"nearest"}` |
| `aggregates.distance` | Devuelve la distancia de mesa. | `{"value":"distance"}` |
| `aggregates.direction` | Devuelve la dirección de mesa. | `{"value":"direction"}` |
| `aggregates.adjacentPairCount` | Cuenta pares adyacentes que cumplen. | `{"value":"adjacentPairCount"}` |
| `comparisons.eq` | Ambos valores son iguales. | `{"value":"eq"}` |
| `comparisons.neq` | Los valores son distintos. | `{"value":"neq"}` |
| `comparisons.gt` | El valor izquierdo es mayor. | `{"value":"gt"}` |
| `comparisons.gte` | El valor izquierdo es mayor o igual. | `{"value":"gte"}` |
| `comparisons.lt` | El valor izquierdo es menor. | `{"value":"lt"}` |
| `comparisons.lte` | El valor izquierdo es menor o igual. | `{"value":"lte"}` |
| `comparisons.includes` | La colección o texto incluye el valor. | `{"value":"includes"}` |
| `directions.clockwise` | Recorre la mesa en sentido horario. | `{"value":"clockwise"}` |
| `directions.counterclockwise` | Recorre la mesa en sentido antihorario. | `{"value":"counterclockwise"}` |
| `inputValueTypes.string` | Un valor de texto. | `{"value":"string"}` |
| `inputValueTypes.number` | Un valor numérico. | `{"value":"number"}` |
| `inputValueTypes.boolean` | Un valor booleano. | `{"value":"boolean"}` |
| `inputValueTypes.null` | Ausencia explícita de valor. | `{"value":"null"}` |
| `inputValueTypes.array` | Una lista de valores. | `{"value":"array"}` |
| `inputValueTypes.object` | Un resultado estructurado. | `{"value":"object"}` |
| `inputValueTypes.entityId` | ID opaco de una entidad. | `{"value":"entityId"}` |
| `inputValueTypes.playerId` | ID opaco de un jugador. | `{"value":"playerId"}` |
| `inputValueTypes.characterId` | ID opaco de un personaje. | `{"value":"characterId"}` |
| `gameProperties.phase` | Fase actual de la partida. | `{"value":"phase"}` |
| `gameProperties.dayNumber` | Día actual. | `{"value":"dayNumber"}` |
| `gameProperties.nightNumber` | Noche actual. | `{"value":"nightNumber"}` |
| `facts.outcome` | Resultado semántico de la resolución. | `{"value":"outcome"}` |
| `facts.guessResult` | Resultado de una apuesta declarada. | `{"value":"guessResult"}` |
| `facts.targetMechanicTags` | Tags mecánicos del objetivo actual. | `{"value":"targetMechanicTags"}` |
| `facts.operationWouldEndGame` | Indica si la operación pendiente podría activar un final estándar. | `{"value":"operationWouldEndGame"}` |
| `setOperations.union` | Combina valores sin duplicados. | `{"value":"union"}` |
| `setOperations.intersection` | Conserva valores comunes. | `{"value":"intersection"}` |
| `setOperations.difference` | Retira valores presentes en otras listas. | `{"value":"difference"}` |
| `mathOperations.add` | Suma todos los valores numéricos. | `{"value":"add"}` |
| `mathOperations.subtract` | Resta los valores siguientes al primero. | `{"value":"subtract"}` |
| `mathOperations.min` | Devuelve el menor valor numérico. | `{"value":"min"}` |
| `mathOperations.max` | Devuelve el mayor valor numérico. | `{"value":"max"}` |
| `identityModes.real` | Consulta la identidad mecánica real. | `{"identityMode":"real"}` |
| `identityModes.initial` | Consulta la identidad asignada al comienzo de la partida. | `{"identityMode":"initial"}` |
| `identityModes.base` | Consulta la identidad declarada por el personaje antes de cambios de bando. | `{"identityMode":"base"}` |
| `identityModes.shown` | Consulta la identidad mostrada al jugador. | `{"identityMode":"shown"}` |
| `identityModes.registered` | Aplica decisiones de registro antes de consultar. | `{"identityMode":"registered"}` |
| `identityFacets.teamId` | Categoría visible estable del pack. | `{"value":"teamId"}` |
| `identityFacets.allegiance` | Bando bueno, malo o neutral. | `{"value":"allegiance"}` |
| `identityFacets.role` | Función mecánica dentro del bando. | `{"value":"role"}` |
| `identityFacets.character` | Identidad concreta declarada por el pack. | `{"value":"character"}` |
| `identityFacets.mechanicTags` | Tags mecánicos declarados. | `{"value":"mechanicTags"}` |
| `comparableIdentityFacets.teamId` | Categoría visible estable del pack. | `{"value":"teamId"}` |
| `comparableIdentityFacets.allegiance` | Bando bueno, malo o neutral. | `{"value":"allegiance"}` |
| `comparableIdentityFacets.role` | Función mecánica dentro del bando. | `{"value":"role"}` |
| `comparableIdentityFacets.character` | Identidad concreta declarada por el pack. | `{"value":"character"}` |
| `alignments.good` | Bando bueno. | `{"value":"good"}` |
| `alignments.evil` | Bando malo. | `{"value":"evil"}` |
| `alignments.neutral` | Bando neutral. | `{"value":"neutral"}` |
| `roles.core` | Pieza principal de su bando. | `{"value":"core"}` |
| `roles.support` | Pieza de apoyo de su bando. | `{"value":"support"}` |
| `roles.independent` | Participa con una política propia. | `{"value":"independent"}` |
| `assignments.player` | Ocupa un asiento normal. | `{"value":"player"}` |
| `assignments.temporalPlayer` | Se incorpora temporalmente a la partida. | `{"value":"temporalPlayer"}` |
| `eventPeriods.currentDay` | Eventos del día actual. | `{"value":"currentDay"}` |
| `eventPeriods.currentNight` | Eventos de la noche actual. | `{"value":"currentNight"}` |
| `eventPeriods.previousDay` | Eventos del día anterior. | `{"value":"previousDay"}` |
| `eventPeriods.previousNight` | Eventos de la noche anterior. | `{"value":"previousNight"}` |
| `eventPeriods.game` | Todos los eventos de la partida. | `{"value":"game"}` |
| `eventRestrictionMatches.all` | Exige que el evento contenga todas las restricciones indicadas. | `{"value":"all"}` |
| `eventRestrictionMatches.any` | Exige que el evento contenga al menos una restricción indicada. | `{"value":"any"}` |
| `eventRestrictionMatches.exact` | Exige exactamente el mismo conjunto de restricciones. | `{"value":"exact"}` |
| `eventFields.actionId` | ID semántico de la acción. | `{"type":"eventField","field":"actionId","value":"<value>"}` |
| `eventFields.outcome` | Resultado semántico del evento. | `{"type":"eventField","field":"outcome","value":"<value>"}` |
| `eventFields.died` | Indica si la ejecución produjo muerte. | `{"type":"eventField","field":"died","value":"<value>"}` |
| `eventFields.attribution` | Bando mecánico al que se atribuye la muerte. | `{"type":"eventField","field":"attribution","value":"<value>"}` |
| `eventFields.resolution` | Flujo mecánico que está resolviendo el evento candidato. | `{"type":"eventField","field":"resolution","value":"<value>"}` |
| `eventFields.known` | Indica si el evento era conocido. | `{"type":"eventField","field":"known","value":"<value>"}` |
| `eventFields.occurrence` | Ocurrencia semántica registrada. | `{"type":"eventField","field":"occurrence","value":"<value>"}` |
| `eventFields.signal` | Señal semántica del Narrador. | `{"type":"eventField","field":"signal","value":"<value>"}` |
| `eventFields.characterId` | ID del personaje relacionado. | `{"type":"eventField","field":"characterId","value":"<value>"}` |
| `eventFields.targetCount` | Cantidad exacta de jugadores elegidos por la mecánica. | `{"type":"eventField","field":"targetCount","value":"<value>"}` |
| `eventFields.targetAlignment` | Bando capturado al elegir. | `{"type":"eventField","field":"targetAlignment","value":"<value>"}` |
| `eventFields.targetAlive` | Estado vital capturado. | `{"type":"eventField","field":"targetAlive","value":"<value>"}` |
| `eventFields.state` | Estado que cambió. | `{"type":"eventField","field":"state","value":"<value>"}` |
| `eventFields.active` | Valor activo del cambio. | `{"type":"eventField","field":"active","value":"<value>"}` |
| `eventFields.markerId` | ID estable de la ficha que cambió. | `{"type":"eventField","field":"markerId","value":"<value>"}` |
| `eventFields.markerKind` | Tipo semántico de la ficha que cambió. | `{"type":"eventField","field":"markerKind","value":"<value>"}` |
| `eventFields.resolutionId` | ID estable de la resolución mecánica. | `{"type":"eventField","field":"resolutionId","value":"<value>"}` |
| `eventFields.resolutionStatus` | Estado semántico final de la resolución. | `{"type":"eventField","field":"resolutionStatus","value":"<value>"}` |
| `eventFields.resolutionPhase` | Fase capturada cuando se resolvió la mecánica. | `{"type":"eventField","field":"resolutionPhase","value":"<value>"}` |
| `eventFields.actorAbilityMode` | Indica si la habilidad del actor funcionó normalmente, mal o quedó deshabilitada. | `{"type":"eventField","field":"actorAbilityMode","value":"<value>"}` |
| `eventFields.abilityProvenance` | Indica si la habilidad era propia, concedida, copiada u otra procedencia. | `{"type":"eventField","field":"abilityProvenance","value":"<value>"}` |
| `eventFields.effectiveTargetCount` | Cantidad de jugadores sobre los que terminó actuando la mecánica. | `{"type":"eventField","field":"effectiveTargetCount","value":"<value>"}` |
| `eventFields.targetResultStatus` | Resultado común de los objetivos cuando todos comparten el mismo estado. | `{"type":"eventField","field":"targetResultStatus","value":"<value>"}` |
| `eventFields.causedByEventId` | ID del evento que originó este evento automático. | `{"type":"eventField","field":"causedByEventId","value":"<value>"}` |
| `information.kinds.boolean` | Información booleana. | `{"value":"boolean"}` |
| `information.kinds.number` | Información numérica. | `{"value":"number"}` |
| `information.kinds.text` | Información textual. | `{"value":"text"}` |
| `information.kinds.player` | Un jugador. | `{"value":"player"}` |
| `information.kinds.players` | Varios jugadores. | `{"value":"players"}` |
| `information.kinds.character` | Un personaje. | `{"value":"character"}` |
| `information.kinds.characters` | Varios personajes. | `{"value":"characters"}` |
| `information.kinds.identity` | Una faceta de identidad. | `{"value":"identity"}` |
| `information.kinds.direction` | Dirección de mesa. | `{"value":"direction"}` |
| `information.kinds.distance` | Distancia de mesa. | `{"value":"distance"}` |
| `information.kinds.grimoire` | Vista estructurada del grimorio. | `{"value":"grimoire"}` |
| `information.kinds.structured` | Resultado con campos declarados. | `{"value":"structured"}` |
| `information.audiences.actor` | Entrega al actor. | `{"value":"actor"}` |
| `information.audiences.selected` | Entrega al objetivo seleccionado. | `{"value":"selected"}` |
| `information.audiences.eventSubject` | Entrega al sujeto del evento. | `{"value":"eventSubject"}` |
| `information.audiences.public` | Entrega públicamente. | `{"value":"public"}` |
| `information.audiences.storyteller` | Entrega solo al Narrador. | `{"value":"storyteller"}` |
| `information.audiences.players` | Entrega a jugadores resueltos por una expresión. | `{"value":"players"}` |
| `information.timings.immediate` | Entrega al resolver. | `{"value":"immediate"}` |
| `information.timings.dawn` | Entrega al amanecer. | `{"value":"dawn"}` |
| `information.timings.privateDay` | Entrega privada durante el día. | `{"value":"privateDay"}` |
| `information.timings.nextRecipientWake` | Entrega en el siguiente despertar nocturno de cada destinatario. | `{"value":"nextRecipientWake"}` |
| `information.modes.shared` | Todos reciben el mismo valor. | `{"value":"shared"}` |
| `information.modes.perRecipient` | Calcula o presenta por destinatario. | `{"value":"perRecipient"}` |
| `information.consentSubjects.selected` | Solicita consentimiento al seleccionado. | `{"value":"selected"}` |
| `information.consentSubjects.eventSubject` | Solicita consentimiento al sujeto del evento. | `{"value":"eventSubject"}` |
| `information.transforms.whenSourceAffected` | Transforma si el origen está afectado. | `{"value":"whenSourceAffected"}` |
| `information.transformReactions.allowArbitraryValue` | Permite al Narrador sustituir el valor. | `{"value":"allowArbitraryValue"}` |
| `themes.purple` | Tema púrpura. | `{"value":"purple"}` |
| `themes.ink` | Tema tinta. | `{"value":"ink"}` |
| `themes.oxblood` | Tema sangre de buey. | `{"value":"oxblood"}` |
| `themes.verdigris` | Tema verdigrís. | `{"value":"verdigris"}` |
| `themes.midnight` | Tema medianoche. | `{"value":"midnight"}` |
| `themes.leather` | Tema cuero. | `{"value":"leather"}` |
| `backdrops.none` | Sin fondo decorativo. | `{"value":"none"}` |
| `backdrops.astrolabe` | Fondo de astrolabio. | `{"value":"astrolabe"}` |
| `backdrops.orrery` | Fondo de planetario. | `{"value":"orrery"}` |
| `backdrops.rose` | Fondo de rosa. | `{"value":"rose"}` |
| `backdrops.filigree` | Fondo de filigrana. | `{"value":"filigree"}` |
| `backdrops.plain` | Fondo liso. | `{"value":"plain"}` |
| `windows.setup` | Durante la preparación de la partida. | `{"window":"setup"}` |
| `windows.firstNight` | Solo durante la primera noche. | `{"window":"firstNight"}` |
| `windows.night` | Durante una fase nocturna. | `{"window":"night"}` |
| `windows.dawn` | Al cerrar la noche y comunicar sus cambios. | `{"window":"dawn"}` |
| `windows.day` | Durante una fase diurna. | `{"window":"day"}` |
| `windows.voting` | Mientras se resuelve una votación. | `{"window":"voting"}` |
| `windows.speech` | Durante la conversación pública o privada. | `{"window":"speech"}` |
| `windows.nomination` | Mientras se propone y resuelve una nominación. | `{"window":"nomination"}` |
| `windows.execution` | Durante la resolución de una ejecución. | `{"window":"execution"}` |
| `windows.exile` | Durante la resolución de un exilio. | `{"window":"exile"}` |
| `windows.dusk` | En la transición del día a la noche. | `{"window":"dusk"}` |
| `windows.mainEvilInfo` | Durante el paso de información inicial del malvado principal. | `{"window":"mainEvilInfo"}` |
| `windows.gameEnd` | Cuando se comprueban condiciones de victoria. | `{"window":"gameEnd"}` |
| `windows.anyTime` | Sin restringir la regla a una fase concreta. | `{"window":"anyTime"}` |
| `cadences.once` | La agenda se activa una única vez. | `{"cadence":"once"}` |
| `cadences.each` | La agenda se activa en cada coincidencia. | `{"cadence":"each"}` |
| `inputKinds.none` | No solicita una decisión adicional. | `{"kind":"none"}` |
| `inputKinds.players` | Selecciona uno o varios jugadores. | `{"kind":"players"}` |
| `inputKinds.character` | Selecciona una identidad de personaje. | `{"kind":"character"}` |
| `inputKinds.playerAndCharacter` | Combina ambas selecciones. | `{"kind":"playerAndCharacter"}` |
| `inputKinds.participantResponses` | Recoge una respuesta independiente de varios participantes. | `{"kind":"participantResponses"}` |
| `inputKinds.text` | Solicita texto tipado al Narrador. | `{"kind":"text"}` |
| `inputKinds.playerCharacterGuesses` | Registra pares de jugador y personaje. | `{"kind":"playerCharacterGuesses"}` |
| `inputKinds.seatSwaps` | Selecciona pares de asientos que se intercambian. | `{"kind":"seatSwaps"}` |
| `inputKinds.vote` | Recoge una decisión de voto. | `{"kind":"vote"}` |
| `inputKinds.contest` | Recoge votos enfrentados. | `{"kind":"contest"}` |
| `usageScopes.repeat` | Límite independiente por activación. | `{"scope":"repeat"}` |
| `usageScopes.day` | Límite compartido durante el día actual. | `{"scope":"day"}` |
| `usageScopes.night` | Límite compartido durante la noche actual. | `{"scope":"night"}` |
| `usageScopes.game` | Límite para toda la partida. | `{"scope":"game"}` |
| `usageScopes.actor` | Límite independiente por actor. | `{"scope":"actor"}` |
| `usageScopes.target` | Límite independiente por objetivo. | `{"scope":"target"}` |
| `usageScopes.trigger` | Límite independiente por evento activador. | `{"scope":"trigger"}` |
| `usageDimensions.day` | Separa usos por número de día. | `{"keyBy":["day"]}` |
| `usageDimensions.night` | Separa usos por número de noche. | `{"keyBy":["night"]}` |
| `usageDimensions.actor` | Separa usos por jugador que actúa. | `{"keyBy":["actor"]}` |
| `usageDimensions.target` | Separa usos por jugador objetivo. | `{"keyBy":["target"]}` |
| `usageDimensions.trigger` | Separa usos por evento activador. | `{"keyBy":["trigger"]}` |
| `consumeOn.attempt` | Consume el uso al iniciar el intento. | `{"consumeOn":"attempt"}` |
| `consumeOn.resolution` | Consume el uso cuando se completa la resolución. | `{"consumeOn":"resolution"}` |
| `consumeOn.success` | Consume el uso solo si el resultado tiene éxito. | `{"consumeOn":"success"}` |
| `durations.permanent` | Permanece durante toda la partida. | `{"type":"permanent"}` |
| `durations.untilWindow` | Termina al alcanzar una ventana declarada. | `{"type":"untilWindow"}` |
| `durations.whileTargetAlive` | Termina cuando muere el objetivo. | `{"type":"whileTargetAlive"}` |
| `durations.whileCondition` | Permanece mientras una condición declarativa sea verdadera. | `{"type":"whileCondition"}` |
| `durations.untilEvent` | Termina cuando coincide un patrón de evento tipado. | `{"type":"untilEvent","event":"death"}` |
| `effects.death` | Mata los objetivos resueltos. | `{"type":"death"}` |
| `effects.death.fields.type` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.death.fields.when` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.death.fields.delay` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.death.fields.targets` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.death.fields.affectedBy` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.death.fields.duration` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.death.fields.optional` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.death.fields.reminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.death.fields.reminderTokens` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.death.fields.spentReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.death.fields.bypassesDeathProtection` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"bypassesDeathProtection":"<bypassesDeathProtection>"}` |
| `effects.death.fields.bypassesProtection` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"bypassesProtection":"<bypassesProtection>"}` |
| `effects.death.fields.chargeReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"chargeReminder":"<chargeReminder>"}` |
| `effects.death.fields.conditionTarget` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"conditionTarget":"<conditionTarget>"}` |
| `effects.death.fields.consumeTargetReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"consumeTargetReminder":"<consumeTargetReminder>"}` |
| `effects.death.fields.continuesNomination` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"continuesNomination":"<continuesNomination>"}` |
| `effects.death.fields.createsReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"createsReminder":"<createsReminder>"}` |
| `effects.death.fields.jumpReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"jumpReminder":"<jumpReminder>"}` |
| `effects.death.fields.mustNominateOnDay3` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"mustNominateOnDay3":"<mustNominateOnDay3>"}` |
| `effects.death.fields.protection` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"protection":"<protection>"}` |
| `effects.death.fields.publicAnnouncement` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"publicAnnouncement":"<publicAnnouncement>"}` |
| `effects.death.fields.registration` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"registration":"<registration>"}` |
| `effects.death.fields.requiredActionOutcome` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"requiredActionOutcome":"<requiredActionOutcome>"}` |
| `effects.death.fields.requiresSourceAlive` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.death.fields.resolution` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"resolution":"<resolution>"}` |
| `effects.death.fields.resultOutcome` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"resultOutcome":"<resultOutcome>"}` |
| `effects.death.fields.secondaryReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"secondaryReminder":"<secondaryReminder>"}` |
| `effects.death.fields.targetAlignment` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"targetAlignment":"<targetAlignment>"}` |
| `effects.death.fields.targetNotTeam` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"targetNotTeam":"<targetNotTeam>"}` |
| `effects.death.fields.targetNotProfile` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"targetNotProfile":"<targetNotProfile>"}` |
| `effects.death.fields.targetReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"targetReminder":"<targetReminder>"}` |
| `effects.death.fields.triggerActionDayScope` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"triggerActionDayScope":"<triggerActionDayScope>"}` |
| `effects.death.fields.triggerActionId` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"triggerActionId":"<triggerActionId>"}` |
| `effects.death.fields.triggerReminder` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"triggerReminder":"<triggerReminder>"}` |
| `effects.death.fields.useRegisteredIdentity` | Campo admitido por death; su valor debe cumplir el contrato tipado. | `{"useRegisteredIdentity":"<useRegisteredIdentity>"}` |
| `effects.resurrect` | Devuelve a la vida los objetivos resueltos. | `{"type":"resurrect"}` |
| `effects.resurrect.fields.type` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.resurrect.fields.when` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.resurrect.fields.delay` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.resurrect.fields.targets` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.resurrect.fields.affectedBy` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.resurrect.fields.duration` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.resurrect.fields.optional` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.resurrect.fields.reminder` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.resurrect.fields.reminderTokens` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.resurrect.fields.spentReminder` | Campo admitido por resurrect; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.execute` | Ejecuta los objetivos resueltos. | `{"type":"execute"}` |
| `effects.execute.fields.type` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.execute.fields.when` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.execute.fields.delay` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.execute.fields.targets` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.execute.fields.affectedBy` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.execute.fields.duration` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.execute.fields.optional` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.execute.fields.reminder` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.execute.fields.reminderTokens` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.execute.fields.spentReminder` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.execute.fields.dies` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"dies":"<dies>"}` |
| `effects.execute.fields.requiresSourceAlive` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.execute.fields.targetReminder` | Campo admitido por execute; su valor debe cumplir el contrato tipado. | `{"targetReminder":"<targetReminder>"}` |
| `effects.setPlayerState` | Crea o actualiza un estado explícito. | `{"type":"setPlayerState","state":"state","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.setPlayerState.fields.type` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.setPlayerState.fields.when` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.setPlayerState.fields.delay` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.setPlayerState.fields.targets` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.setPlayerState.fields.affectedBy` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.setPlayerState.fields.duration` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.setPlayerState.fields.state` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"state":"<state>"}` |
| `effects.setPlayerState.fields.active` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"active":"<active>"}` |
| `effects.setPlayerState.fields.exclusive` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"exclusive":"<exclusive>"}` |
| `effects.setPlayerState.fields.excludeInitialTargets` | Campo admitido por setPlayerState; su valor debe cumplir el contrato tipado. | `{"excludeInitialTargets":"<excludeInitialTargets>"}` |
| `effects.applyMarker` | Añade o retira una ficha recordatoria. | `{"type":"applyMarker","kind":"reminder","id":"marker","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.applyMarker.fields.type` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.applyMarker.fields.when` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.applyMarker.fields.delay` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.applyMarker.fields.targets` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.applyMarker.fields.affectedBy` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.applyMarker.fields.duration` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.applyMarker.fields.kind` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"kind":"<kind>"}` |
| `effects.applyMarker.fields.id` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"id":"<id>"}` |
| `effects.applyMarker.fields.active` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"active":"<active>"}` |
| `effects.applyMarker.fields.exclusive` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"exclusive":"<exclusive>"}` |
| `effects.applyMarker.fields.ownership` | Campo admitido por applyMarker; su valor debe cumplir el contrato tipado. | `{"ownership":"<ownership>"}` |
| `effects.moveMarker` | Transfiere atómicamente una ficha existente entre jugadores. | `{"type":"moveMarker","kind":"reminder","id":"marker","from":{"type":"binding","binding":"actor"},"targets":{"type":"binding","binding":"selected"}}` |
| `effects.moveMarker.fields.type` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.moveMarker.fields.when` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.moveMarker.fields.delay` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.moveMarker.fields.targets` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.moveMarker.fields.affectedBy` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.moveMarker.fields.kind` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"kind":"<kind>"}` |
| `effects.moveMarker.fields.id` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"id":"<id>"}` |
| `effects.moveMarker.fields.from` | Campo admitido por moveMarker; su valor debe cumplir el contrato tipado. | `{"from":"<from>"}` |
| `effects.adjustCounter` | Ajusta un contador persistente del objetivo. | `{"type":"adjustCounter","counter":"counter","delta":1,"targets":{"type":"binding","binding":"selected"}}` |
| `effects.adjustCounter.fields.type` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.adjustCounter.fields.when` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.adjustCounter.fields.delay` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.adjustCounter.fields.targets` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.adjustCounter.fields.affectedBy` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.adjustCounter.fields.duration` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.adjustCounter.fields.counter` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"counter":"<counter>"}` |
| `effects.adjustCounter.fields.delta` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"delta":"<delta>"}` |
| `effects.adjustCounter.fields.scope` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"scope":"<scope>"}` |
| `effects.adjustCounter.fields.bounds` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"bounds":"<bounds>"}` |
| `effects.adjustCounter.fields.projection` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"projection":"<projection>"}` |
| `effects.adjustCounter.fields.stateProjection` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"stateProjection":"<stateProjection>"}` |
| `effects.adjustCounter.fields.threshold` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"threshold":"<threshold>"}` |
| `effects.adjustCounter.fields.onThreshold` | Campo admitido por adjustCounter; su valor debe cumplir el contrato tipado. | `{"onThreshold":"<onThreshold>"}` |
| `effects.changeAlignment` | Cambia el alineamiento de los objetivos. | `{"type":"changeAlignment"}` |
| `effects.changeAlignment.fields.type` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.changeAlignment.fields.when` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.changeAlignment.fields.delay` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.changeAlignment.fields.targets` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.changeAlignment.fields.affectedBy` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.changeAlignment.fields.duration` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.changeAlignment.fields.optional` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.changeAlignment.fields.reminder` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.changeAlignment.fields.reminderTokens` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.changeAlignment.fields.spentReminder` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.changeAlignment.fields.alignment` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"alignment":"<alignment>"}` |
| `effects.changeAlignment.fields.allowsSelfTarget` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"allowsSelfTarget":"<allowsSelfTarget>"}` |
| `effects.changeAlignment.fields.notifyPlayer` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"notifyPlayer":"<notifyPlayer>"}` |
| `effects.changeAlignment.fields.setupEffect` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"setupEffect":"<setupEffect>"}` |
| `effects.changeAlignment.fields.targetAlignment` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"targetAlignment":"<targetAlignment>"}` |
| `effects.changeAlignment.fields.targetTeam` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"targetTeam":"<targetTeam>"}` |
| `effects.changeAlignment.fields.targetProfile` | Campo admitido por changeAlignment; su valor debe cumplir el contrato tipado. | `{"targetProfile":"<targetProfile>"}` |
| `effects.changeCharacter` | Sustituye una identidad de personaje. | `{"type":"changeCharacter"}` |
| `effects.changeCharacter.fields.type` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.changeCharacter.fields.when` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.changeCharacter.fields.delay` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.changeCharacter.fields.targets` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.changeCharacter.fields.affectedBy` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.changeCharacter.fields.duration` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.changeCharacter.fields.optional` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.changeCharacter.fields.reminder` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.changeCharacter.fields.reminderTokens` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.changeCharacter.fields.spentReminder` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.changeCharacter.fields.allowedBuckets` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"allowedBuckets":"<allowedBuckets>"}` |
| `effects.changeCharacter.fields.arbitraryDeathsIfMainEvilCreated` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"arbitraryDeathsIfMainEvilCreated":"<arbitraryDeathsIfMainEvilCreated>"}` |
| `effects.changeCharacter.fields.characterType` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"characterType":"<characterType>"}` |
| `effects.changeCharacter.fields.characterProfile` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"characterProfile":"<characterProfile>"}` |
| `effects.changeCharacter.fields.countTeams` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"countTeams":"<countTeams>"}` |
| `effects.changeCharacter.fields.countProfiles` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"countProfiles":"<countProfiles>"}` |
| `effects.changeCharacter.fields.countTiming` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"countTiming":"<countTiming>"}` |
| `effects.changeCharacter.fields.createsReminder` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"createsReminder":"<createsReminder>"}` |
| `effects.changeCharacter.fields.markOriginalAbilityHolderIfInPlay` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"markOriginalAbilityHolderIfInPlay":"<markOriginalAbilityHolderIfInPlay>"}` |
| `effects.changeCharacter.fields.gainsAbilityOfChosenCharacter` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"gainsAbilityOfChosenCharacter":"<gainsAbilityOfChosenCharacter>"}` |
| `effects.changeCharacter.fields.minimumAlive` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"minimumAlive":"<minimumAlive>"}` |
| `effects.changeCharacter.fields.mustNeighborTeam` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"mustNeighborTeam":"<mustNeighborTeam>"}` |
| `effects.changeCharacter.fields.mustNeighborProfile` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"mustNeighborProfile":"<mustNeighborProfile>"}` |
| `effects.changeCharacter.fields.newCharacter` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"newCharacter":"<newCharacter>"}` |
| `effects.changeCharacter.fields.newAlignment` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"newAlignment":"<newAlignment>"}` |
| `effects.changeCharacter.fields.newTeam` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"newTeam":"<newTeam>"}` |
| `effects.changeCharacter.fields.newProfile` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"newProfile":"<newProfile>"}` |
| `effects.changeCharacter.fields.oldMainEvilDies` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"oldMainEvilDies":"<oldMainEvilDies>"}` |
| `effects.changeCharacter.fields.preserveAlignment` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"preserveAlignment":"<preserveAlignment>"}` |
| `effects.changeCharacter.fields.realCharacter` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"realCharacter":"<realCharacter>"}` |
| `effects.changeCharacter.fields.realTeam` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"realTeam":"<realTeam>"}` |
| `effects.changeCharacter.fields.realProfile` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"realProfile":"<realProfile>"}` |
| `effects.changeCharacter.fields.result` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"result":"<result>"}` |
| `effects.changeCharacter.fields.shownAs` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"shownAs":"<shownAs>"}` |
| `effects.changeCharacter.fields.targetCharacter` | Campo admitido por changeCharacter; su valor debe cumplir el contrato tipado. | `{"targetCharacter":"<targetCharacter>"}` |
| `effects.grantAbility` | Concede una habilidad declarada. | `{"type":"grantAbility"}` |
| `effects.grantAbility.fields.type` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.grantAbility.fields.when` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.grantAbility.fields.delay` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.grantAbility.fields.targets` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.grantAbility.fields.affectedBy` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.grantAbility.fields.duration` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.grantAbility.fields.optional` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.grantAbility.fields.reminder` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.grantAbility.fields.reminderTokens` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.grantAbility.fields.spentReminder` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.grantAbility.fields.active` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"active":"<active>"}` |
| `effects.grantAbility.fields.abilityCharacterId` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"abilityCharacterId":"<abilityCharacterId>"}` |
| `effects.grantAbility.fields.owner` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"owner":"<owner>"}` |
| `effects.grantAbility.fields.controller` | Campo admitido por grantAbility; su valor debe cumplir el contrato tipado. | `{"controller":"<controller>"}` |
| `effects.swapSeats` | Intercambia posiciones en el círculo. | `{"type":"swapSeats"}` |
| `effects.swapSeats.fields.type` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.swapSeats.fields.when` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.swapSeats.fields.delay` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.swapSeats.fields.targets` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.swapSeats.fields.affectedBy` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapSeats.fields.duration` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.swapSeats.fields.optional` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.swapSeats.fields.reminder` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.swapSeats.fields.reminderTokens` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapSeats.fields.spentReminder` | Campo admitido por swapSeats; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.swapCharacters` | Intercambia identidades entre participantes. | `{"type":"swapCharacters"}` |
| `effects.swapCharacters.fields.type` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.swapCharacters.fields.when` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.swapCharacters.fields.delay` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.swapCharacters.fields.targets` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.swapCharacters.fields.affectedBy` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapCharacters.fields.duration` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.swapCharacters.fields.optional` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.swapCharacters.fields.reminder` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.swapCharacters.fields.reminderTokens` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapCharacters.fields.spentReminder` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.swapCharacters.fields.actor` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"actor":"<actor>"}` |
| `effects.swapCharacters.fields.resultingState` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"resultingState":"<resultingState>"}` |
| `effects.swapCharacters.fields.resultingStateDuration` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"resultingStateDuration":"<resultingStateDuration>"}` |
| `effects.swapCharacters.fields.swapsCharactersAndAlignments` | Campo admitido por swapCharacters; su valor debe cumplir el contrato tipado. | `{"swapsCharactersAndAlignments":"<swapsCharactersAndAlignments>"}` |
| `effects.swapTargets` | Intercambia objetivos ya resueltos. | `{"type":"swapTargets"}` |
| `effects.swapTargets.fields.type` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.swapTargets.fields.when` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.swapTargets.fields.delay` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.swapTargets.fields.targets` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.swapTargets.fields.affectedBy` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.swapTargets.fields.duration` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.swapTargets.fields.optional` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.swapTargets.fields.reminder` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.swapTargets.fields.reminderTokens` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.swapTargets.fields.spentReminder` | Campo admitido por swapTargets; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.emitInformation` | Calcula y entrega información tipada. | `{"type":"emitInformation","value":{"type":"literal","value":""},"presentation":{"kind":"text","title":"Información"},"delivery":{"audience":{"type":"storyteller"}}}` |
| `effects.emitInformation.fields.type` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.emitInformation.fields.when` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.emitInformation.fields.delay` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.emitInformation.fields.targets` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.emitInformation.fields.affectedBy` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.emitInformation.fields.duration` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.emitInformation.fields.optional` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.emitInformation.fields.reminder` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.emitInformation.fields.reminderTokens` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.emitInformation.fields.spentReminder` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.emitInformation.fields.value` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"value":"<value>"}` |
| `effects.emitInformation.fields.presentation` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"presentation":"<presentation>"}` |
| `effects.emitInformation.fields.delivery` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"delivery":"<delivery>"}` |
| `effects.emitInformation.fields.transform` | Campo admitido por emitInformation; su valor debe cumplir el contrato tipado. | `{"transform":"<transform>"}` |
| `effects.prepareInformation` | Prepara información verdadera y alternativa para una entrega posterior. | `{"type":"prepareInformation","candidates":{"type":"array","items":[]},"modes":["pair"],"characterChoice":{"source":"truthfulPlayer","identityMode":"real"},"reminders":{"truthful":"Verdadero","alternative":"Alternativa"}}` |
| `effects.prepareInformation.fields.type` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.prepareInformation.fields.when` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.prepareInformation.fields.delay` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.prepareInformation.fields.targets` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.prepareInformation.fields.affectedBy` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.prepareInformation.fields.duration` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.prepareInformation.fields.optional` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.prepareInformation.fields.reminder` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.prepareInformation.fields.reminderTokens` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.prepareInformation.fields.spentReminder` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.prepareInformation.fields.candidates` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"candidates":"<candidates>"}` |
| `effects.prepareInformation.fields.modes` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"modes":"<modes>"}` |
| `effects.prepareInformation.fields.zeroWhen` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"zeroWhen":"<zeroWhen>"}` |
| `effects.prepareInformation.fields.characterChoice` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"characterChoice":"<characterChoice>"}` |
| `effects.prepareInformation.fields.reminders` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"reminders":"<reminders>"}` |
| `effects.prepareInformation.fields.shownIdentityOverride` | Campo admitido por prepareInformation; su valor debe cumplir el contrato tipado. | `{"shownIdentityOverride":"<shownIdentityOverride>"}` |
| `effects.resolveGameEnd` | Declara un ganador inmediato o programa una resolución declarativa. | `{"type":"resolveGameEnd","mode":"immediate","winner":{"type":"fixed","team":"good"},"reason":"Describe por qué termina la partida."}` |
| `effects.resolveGameEnd.fields.type` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.resolveGameEnd.fields.when` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.resolveGameEnd.fields.delay` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.resolveGameEnd.fields.targets` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.resolveGameEnd.fields.affectedBy` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.resolveGameEnd.fields.duration` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.resolveGameEnd.fields.optional` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.resolveGameEnd.fields.reminder` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.resolveGameEnd.fields.reminderTokens` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.resolveGameEnd.fields.spentReminder` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.resolveGameEnd.fields.mode` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"mode":"<mode>"}` |
| `effects.resolveGameEnd.fields.winner` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"winner":"<winner>"}` |
| `effects.resolveGameEnd.fields.reason` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"reason":"<reason>"}` |
| `effects.resolveGameEnd.fields.precedence` | Campo admitido por resolveGameEnd; su valor debe cumplir el contrato tipado. | `{"precedence":"<precedence>"}` |
| `effects.blockGameEnd` | Impide cerrar un resultado mientras esta política siga activa. | `{"type":"blockGameEnd","winner":"good","reason":"La victoria está bloqueada."}` |
| `effects.blockGameEnd.fields.type` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.blockGameEnd.fields.when` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.blockGameEnd.fields.delay` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.blockGameEnd.fields.targets` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.blockGameEnd.fields.affectedBy` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.blockGameEnd.fields.duration` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.blockGameEnd.fields.optional` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.blockGameEnd.fields.reminder` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.blockGameEnd.fields.reminderTokens` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.blockGameEnd.fields.spentReminder` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.blockGameEnd.fields.winner` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"winner":"<winner>"}` |
| `effects.blockGameEnd.fields.reason` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"reason":"<reason>"}` |
| `effects.blockGameEnd.fields.precedence` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"precedence":"<precedence>"}` |
| `effects.blockGameEnd.fields.activation` | Campo admitido por blockGameEnd; su valor debe cumplir el contrato tipado. | `{"activation":"<activation>"}` |
| `effects.transformGameEnd` | Transforma el conjunto definitivo de ganadores. | `{"type":"transformGameEnd","operation":"invertWinners","reason":"Se invierten ganadores y perdedores."}` |
| `effects.transformGameEnd.fields.type` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.transformGameEnd.fields.when` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.transformGameEnd.fields.delay` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.transformGameEnd.fields.targets` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.transformGameEnd.fields.affectedBy` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.transformGameEnd.fields.duration` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.transformGameEnd.fields.optional` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.transformGameEnd.fields.reminder` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.transformGameEnd.fields.reminderTokens` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.transformGameEnd.fields.spentReminder` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.transformGameEnd.fields.operation` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"operation":"<operation>"}` |
| `effects.transformGameEnd.fields.reason` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"reason":"<reason>"}` |
| `effects.transformGameEnd.fields.precedence` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"precedence":"<precedence>"}` |
| `effects.transformGameEnd.fields.activation` | Campo admitido por transformGameEnd; su valor debe cumplir el contrato tipado. | `{"activation":"<activation>"}` |
| `effects.startActionSequence` | Mantiene una cadena de acciones abierta hasta cumplir su condición de salida. | `{"type":"startActionSequence","action":"nomination","onAction":"killNominee","nextActor":"nominee","fallbackActor":"storyteller","repeatUntil":{"type":"literal","value":false}}` |
| `effects.startActionSequence.fields.type` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.startActionSequence.fields.when` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.startActionSequence.fields.delay` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.startActionSequence.fields.targets` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.startActionSequence.fields.affectedBy` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.startActionSequence.fields.duration` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.startActionSequence.fields.optional` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.startActionSequence.fields.reminder` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.startActionSequence.fields.reminderTokens` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.startActionSequence.fields.spentReminder` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.startActionSequence.fields.action` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"action":"<action>"}` |
| `effects.startActionSequence.fields.startsAtDay` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"startsAtDay":"<startsAtDay>"}` |
| `effects.startActionSequence.fields.onAction` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"onAction":"<onAction>"}` |
| `effects.startActionSequence.fields.nextActor` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"nextActor":"<nextActor>"}` |
| `effects.startActionSequence.fields.fallbackActor` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"fallbackActor":"<fallbackActor>"}` |
| `effects.startActionSequence.fields.repeatUntil` | Campo admitido por startActionSequence; su valor debe cumplir el contrato tipado. | `{"repeatUntil":"<repeatUntil>"}` |
| `effects.interceptEvent` | Cancela, redirige o sustituye un evento que coincide con un patrón semántico. | `{"type":"interceptEvent","event":"death","reaction":{"type":"cancel"}}` |
| `effects.interceptEvent.fields.type` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.interceptEvent.fields.when` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.interceptEvent.fields.delay` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.interceptEvent.fields.targets` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.interceptEvent.fields.affectedBy` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.interceptEvent.fields.duration` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.interceptEvent.fields.optional` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.interceptEvent.fields.reminder` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.interceptEvent.fields.reminderTokens` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.interceptEvent.fields.event` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"event":"<event>"}` |
| `effects.interceptEvent.fields.bindings` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"bindings":"<bindings>"}` |
| `effects.interceptEvent.fields.match` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"match":"<match>"}` |
| `effects.interceptEvent.fields.reaction` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"reaction":"<reaction>"}` |
| `effects.interceptEvent.fields.priority` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"priority":"<priority>"}` |
| `effects.interceptEvent.fields.consumption` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"consumption":"<consumption>"}` |
| `effects.interceptEvent.fields.appliesWhenProtectionBypassed` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"appliesWhenProtectionBypassed":"<appliesWhenProtectionBypassed>"}` |
| `effects.interceptEvent.fields.scope` | Campo admitido por interceptEvent; su valor debe cumplir el contrato tipado. | `{"scope":"<scope>"}` |
| `effects.disableAbility` | Desactiva una habilidad según el contrato. | `{"type":"disableAbility"}` |
| `effects.disableAbility.fields.type` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.disableAbility.fields.when` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.disableAbility.fields.delay` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.disableAbility.fields.targets` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.disableAbility.fields.affectedBy` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.disableAbility.fields.duration` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.disableAbility.fields.optional` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.disableAbility.fields.reminder` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.disableAbility.fields.reminderTokens` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.disableAbility.fields.spentReminder` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.disableAbility.fields.blocks` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"blocks":"<blocks>"}` |
| `effects.disableAbility.fields.consumeOnPass` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"consumeOnPass":"<consumeOnPass>"}` |
| `effects.disableAbility.fields.informationMayBeFalse` | Campo admitido por disableAbility; su valor debe cumplir el contrato tipado. | `{"informationMayBeFalse":"<informationMayBeFalse>"}` |
| `effects.restrict` | Limita acciones disponibles para los objetivos. | `{"type":"restrict"}` |
| `effects.restrict.fields.type` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.restrict.fields.when` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.restrict.fields.delay` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.restrict.fields.targets` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.restrict.fields.affectedBy` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.restrict.fields.duration` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.restrict.fields.optional` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.restrict.fields.reminder` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.restrict.fields.reminderTokens` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.restrict.fields.spentReminder` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.restrict.fields.actions` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"actions":"<actions>"}` |
| `effects.restrict.fields.informationMayBeFalse` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"informationMayBeFalse":"<informationMayBeFalse>"}` |
| `effects.restrict.fields.requiresSourceAlive` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"requiresSourceAlive":"<requiresSourceAlive>"}` |
| `effects.restrict.fields.restrictions` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"restrictions":"<restrictions>"}` |
| `effects.restrict.fields.relation` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"relation":"<relation>"}` |
| `effects.restrict.fields.exception` | Campo admitido por restrict; su valor debe cumplir el contrato tipado. | `{"exception":"<exception>"}` |
| `effects.registerAs` | Cambia cómo se registra una identidad. | `{"type":"registerAs"}` |
| `effects.registerAs.fields.type` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.registerAs.fields.when` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.registerAs.fields.delay` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.registerAs.fields.targets` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.registerAs.fields.affectedBy` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.registerAs.fields.duration` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.registerAs.fields.optional` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.registerAs.fields.reminder` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.registerAs.fields.reminderTokens` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.registerAs.fields.spentReminder` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.registerAs.fields.affects` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"affects":"<affects>"}` |
| `effects.registerAs.fields.alignment` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"alignment":"<alignment>"}` |
| `effects.registerAs.fields.roles` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"roles":"<roles>"}` |
| `effects.registerAs.fields.characterIds` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"characterIds":"<characterIds>"}` |
| `effects.registerAs.fields.characterId` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"characterId":"<characterId>"}` |
| `effects.registerAs.fields.lifeState` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"lifeState":"<lifeState>"}` |
| `effects.registerAs.fields.mode` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"mode":"<mode>"}` |
| `effects.registerAs.fields.registersAs` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"registersAs":"<registersAs>"}` |
| `effects.registerAs.fields.teamIds` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"teamIds":"<teamIds>"}` |
| `effects.registerAs.fields.triggerReminder` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"triggerReminder":"<triggerReminder>"}` |
| `effects.registerAs.fields.worksWhenDead` | Campo admitido por registerAs; su valor debe cumplir el contrato tipado. | `{"worksWhenDead":"<worksWhenDead>"}` |
| `effects.modifyTargets` | Amplía o reduce el máximo de objetivos de habilidades que cumplan el perfil declarado. | `{"type":"modifyTargets","delta":1}` |
| `effects.modifyTargets.fields.type` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifyTargets.fields.when` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifyTargets.fields.delay` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifyTargets.fields.targets` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifyTargets.fields.affectedBy` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyTargets.fields.duration` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifyTargets.fields.optional` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifyTargets.fields.reminder` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifyTargets.fields.reminderTokens` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyTargets.fields.spentReminder` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyTargets.fields.delta` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"delta":"<delta>"}` |
| `effects.modifyTargets.fields.sourceProfile` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"sourceProfile":"<sourceProfile>"}` |
| `effects.modifyTargets.fields.targetMechanicTags` | Campo admitido por modifyTargets; su valor debe cumplir el contrato tipado. | `{"targetMechanicTags":"<targetMechanicTags>"}` |
| `effects.modifyVote` | Cambia el peso de votantes calculados. | `{"type":"modifyVote","targets":{"type":"binding","binding":"selected"},"weight":2}` |
| `effects.modifyVote.fields.type` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifyVote.fields.when` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifyVote.fields.delay` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifyVote.fields.targets` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifyVote.fields.affectedBy` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyVote.fields.duration` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifyVote.fields.optional` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifyVote.fields.reminder` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifyVote.fields.reminderTokens` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyVote.fields.spentReminder` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyVote.fields.weight` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"weight":"<weight>"}` |
| `effects.modifyVote.fields.pairedTargets` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"pairedTargets":"<pairedTargets>"}` |
| `effects.modifyVote.fields.pairedWeight` | Campo admitido por modifyVote; su valor debe cumplir el contrato tipado. | `{"pairedWeight":"<pairedWeight>"}` |
| `effects.modifySetup` | Aplica operaciones tipadas a cantidades o asignaciones del setup. | `{"type":"modifySetup","operations":[{"type":"adjustBucket","bucket":"setupBucket","delta":0}]}` |
| `effects.modifySetup.fields.type` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifySetup.fields.when` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifySetup.fields.delay` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifySetup.fields.targets` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifySetup.fields.affectedBy` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifySetup.fields.duration` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifySetup.fields.optional` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifySetup.fields.reminder` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifySetup.fields.reminderTokens` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifySetup.fields.spentReminder` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifySetup.fields.operations` | Campo admitido por modifySetup; su valor debe cumplir el contrato tipado. | `{"operations":"<operations>"}` |
| `effects.restrictSetupCombination` | Limita cuántas identidades declaradas pueden coincidir en la preparación. | `{"type":"restrictSetupCombination","characterIds":["character:invented:first","character:invented:second"],"maximum":1}` |
| `effects.restrictSetupCombination.fields.type` | Campo admitido por restrictSetupCombination; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.restrictSetupCombination.fields.when` | Campo admitido por restrictSetupCombination; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.restrictSetupCombination.fields.characterIds` | Campo admitido por restrictSetupCombination; su valor debe cumplir el contrato tipado. | `{"characterIds":"<characterIds>"}` |
| `effects.restrictSetupCombination.fields.maximum` | Campo admitido por restrictSetupCombination; su valor debe cumplir el contrato tipado. | `{"maximum":"<maximum>"}` |
| `effects.modifyInformation` | Transforma información antes de entregarla. | `{"type":"modifyInformation"}` |
| `effects.modifyInformation.fields.type` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifyInformation.fields.when` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifyInformation.fields.delay` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifyInformation.fields.targets` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifyInformation.fields.affectedBy` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyInformation.fields.duration` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifyInformation.fields.optional` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifyInformation.fields.reminder` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifyInformation.fields.reminderTokens` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyInformation.fields.spentReminder` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyInformation.fields.audience` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"audience":"<audience>"}` |
| `effects.modifyInformation.fields.mustBeFalse` | Campo admitido por modifyInformation; su valor debe cumplir el contrato tipado. | `{"mustBeFalse":"<mustBeFalse>"}` |
| `effects.modifyStartingKnowledge` | Activa o desactiva pasos tipados de conocimiento inicial. | `{"type":"modifyStartingKnowledge","steps":["evilTeamRecognition"],"active":false}` |
| `effects.modifyStartingKnowledge.fields.type` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifyStartingKnowledge.fields.when` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifyStartingKnowledge.fields.delay` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifyStartingKnowledge.fields.targets` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifyStartingKnowledge.fields.affectedBy` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyStartingKnowledge.fields.duration` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifyStartingKnowledge.fields.optional` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifyStartingKnowledge.fields.reminder` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifyStartingKnowledge.fields.reminderTokens` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyStartingKnowledge.fields.spentReminder` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyStartingKnowledge.fields.steps` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"steps":"<steps>"}` |
| `effects.modifyStartingKnowledge.fields.active` | Campo admitido por modifyStartingKnowledge; su valor debe cumplir el contrato tipado. | `{"active":"<active>"}` |
| `effects.modifyNomination` | Cambia la resolución de una nominación. | `{"type":"modifyNomination"}` |
| `effects.modifyNomination.fields.type` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.modifyNomination.fields.when` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.modifyNomination.fields.delay` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.modifyNomination.fields.targets` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.modifyNomination.fields.affectedBy` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.modifyNomination.fields.duration` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.modifyNomination.fields.optional` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.modifyNomination.fields.reminder` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.modifyNomination.fields.reminderTokens` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.modifyNomination.fields.spentReminder` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.modifyNomination.fields.allowsStorytellerNominee` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"allowsStorytellerNominee":"<allowsStorytellerNominee>"}` |
| `effects.modifyNomination.fields.abilitySpentEvenIfNoExecution` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"abilitySpentEvenIfNoExecution":"<abilitySpentEvenIfNoExecution>"}` |
| `effects.modifyNomination.fields.countsAsExecution` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"countsAsExecution":"<countsAsExecution>"}` |
| `effects.modifyNomination.fields.createsReminder` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"createsReminder":"<createsReminder>"}` |
| `effects.modifyNomination.fields.voteDelta` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"voteDelta":"<voteDelta>"}` |
| `effects.modifyNomination.fields.requiresActorAbstention` | Campo admitido por modifyNomination; su valor debe cumplir el contrato tipado. | `{"requiresActorAbstention":"<requiresActorAbstention>"}` |
| `effects.recordAction` | Registra una acción y su resultado. | `{"type":"recordAction"}` |
| `effects.recordAction.fields.type` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.recordAction.fields.when` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.recordAction.fields.delay` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.recordAction.fields.targets` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.recordAction.fields.affectedBy` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.recordAction.fields.duration` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.recordAction.fields.optional` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.recordAction.fields.reminder` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.recordAction.fields.reminderTokens` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.recordAction.fields.spentReminder` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.recordAction.fields.action` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"action":"<action>"}` |
| `effects.recordAction.fields.actionId` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"actionId":"<actionId>"}` |
| `effects.recordAction.fields.outcome` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"outcome":"<outcome>"}` |
| `effects.recordAction.fields.creates` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"creates":"<creates>"}` |
| `effects.recordAction.fields.effect` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"effect":"<effect>"}` |
| `effects.recordAction.fields.failureEffect` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"failureEffect":"<failureEffect>"}` |
| `effects.recordAction.fields.failureOutcome` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"failureOutcome":"<failureOutcome>"}` |
| `effects.recordAction.fields.maxGuesses` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"maxGuesses":"<maxGuesses>"}` |
| `effects.recordAction.fields.resolvesExecution` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"resolvesExecution":"<resolvesExecution>"}` |
| `effects.recordAction.fields.restrictions` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"restrictions":"<restrictions>"}` |
| `effects.recordAction.fields.successOutcome` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"successOutcome":"<successOutcome>"}` |
| `effects.recordAction.fields.targetMechanicTags` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"targetMechanicTags":"<targetMechanicTags>"}` |
| `effects.recordAction.fields.targetRegistrationTeams` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"targetRegistrationTeams":"<targetRegistrationTeams>"}` |
| `effects.recordAction.fields.recordAs` | Campo admitido por recordAction; su valor debe cumplir el contrato tipado. | `{"recordAs":"<recordAs>"}` |
| `effects.storytellerDecision` | Solicita una decisión humana declarada. | `{"type":"storytellerDecision","decision":"decision"}` |
| `effects.storytellerDecision.fields.type` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.storytellerDecision.fields.when` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.storytellerDecision.fields.delay` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.storytellerDecision.fields.targets` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.storytellerDecision.fields.affectedBy` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.storytellerDecision.fields.duration` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.storytellerDecision.fields.optional` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.storytellerDecision.fields.reminder` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.storytellerDecision.fields.reminderTokens` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.storytellerDecision.fields.spentReminder` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.storytellerDecision.fields.decision` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"decision":"<decision>"}` |
| `effects.storytellerDecision.fields.options` | Campo admitido por storytellerDecision; su valor debe cumplir el contrato tipado. | `{"options":"<options>"}` |
| `effects.manualInstruction` | Muestra una resolución guiada no automatizada. | `{"type":"manualInstruction","instruction":"Describe cómo resolver esta regla."}` |
| `effects.manualInstruction.fields.type` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `effects.manualInstruction.fields.when` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"when":"<when>"}` |
| `effects.manualInstruction.fields.delay` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"delay":"<delay>"}` |
| `effects.manualInstruction.fields.targets` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"targets":"<targets>"}` |
| `effects.manualInstruction.fields.affectedBy` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"affectedBy":"<affectedBy>"}` |
| `effects.manualInstruction.fields.duration` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `effects.manualInstruction.fields.optional` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"optional":"<optional>"}` |
| `effects.manualInstruction.fields.reminder` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"reminder":"<reminder>"}` |
| `effects.manualInstruction.fields.reminderTokens` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"reminderTokens":"<reminderTokens>"}` |
| `effects.manualInstruction.fields.spentReminder` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"spentReminder":"<spentReminder>"}` |
| `effects.manualInstruction.fields.affects` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"affects":"<affects>"}` |
| `effects.manualInstruction.fields.ruleStepActivation` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"ruleStepActivation":"<ruleStepActivation>"}` |
| `effects.manualInstruction.fields.aliveThreshold` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"aliveThreshold":"<aliveThreshold>"}` |
| `effects.manualInstruction.fields.bluffCountOptions` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"bluffCountOptions":"<bluffCountOptions>"}` |
| `effects.manualInstruction.fields.durationMinutes` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"durationMinutes":"<durationMinutes>"}` |
| `effects.manualInstruction.fields.instruction` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"instruction":"<instruction>"}` |
| `effects.manualInstruction.fields.publicKnown` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"publicKnown":"<publicKnown>"}` |
| `effects.manualInstruction.fields.reason` | Campo admitido por manualInstruction; su valor debe cumplir el contrato tipado. | `{"reason":"<reason>"}` |
| `policies.recordTargetSelectionWhenDisabled` | Conserva la selección aunque el actor esté deshabilitado. | `{"type":"recordTargetSelectionWhenDisabled"}` |
| `policies.recordTargetSelectionWhenDisabled.fields.type` | Campo admitido por recordTargetSelectionWhenDisabled; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.presentStartingInformationAsShownIdentity` | Presenta de forma transitoria la información inicial que recibiría la identidad mostrada. | `{"type":"presentStartingInformationAsShownIdentity"}` |
| `policies.presentStartingInformationAsShownIdentity.fields.type` | Campo admitido por presentStartingInformationAsShownIdentity; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.relayShownSelection` | Registra y transmite la selección hecha mediante la identidad mostrada. | `{"type":"relayShownSelection"}` |
| `policies.relayShownSelection.fields.type` | Campo admitido por relayShownSelection; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.assignSelectedCharacter` | Asigna el personaje seleccionado. | `{"type":"assignSelectedCharacter"}` |
| `policies.assignSelectedCharacter.fields.type` | Campo admitido por assignSelectedCharacter; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.grantSelectedAbility` | Concede la habilidad seleccionada. | `{"type":"grantSelectedAbility"}` |
| `policies.grantSelectedAbility.fields.type` | Campo admitido por grantSelectedAbility; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.grantSelectedAbility.fields.target` | Campo admitido por grantSelectedAbility; su valor debe cumplir el contrato tipado. | `{"target":"<target>"}` |
| `policies.grantSelectedAbility.fields.abilityAlignment` | Campo admitido por grantSelectedAbility; su valor debe cumplir el contrato tipado. | `{"abilityAlignment":"<abilityAlignment>"}` |
| `policies.grantSelectedAbility.fields.duration` | Campo admitido por grantSelectedAbility; su valor debe cumplir el contrato tipado. | `{"duration":"<duration>"}` |
| `policies.grantSelectedAbility.fields.requireInPlay` | Campo admitido por grantSelectedAbility; su valor debe cumplir el contrato tipado. | `{"requireInPlay":"<requireInPlay>"}` |
| `policies.overrideChooserAlignment` | Fija el bando usado por el selector. | `{"type":"overrideChooserAlignment"}` |
| `policies.overrideChooserAlignment.fields.type` | Campo admitido por overrideChooserAlignment; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.overrideChooserAlignment.fields.alignment` | Campo admitido por overrideChooserAlignment; su valor debe cumplir el contrato tipado. | `{"alignment":"<alignment>"}` |
| `policies.continueAfterTargetReaction` | Continúa después de la reacción del objetivo. | `{"type":"continueAfterTargetReaction"}` |
| `policies.continueAfterTargetReaction.fields.type` | Campo admitido por continueAfterTargetReaction; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.suppressStandaloneNightStep` | Evita un paso nocturno independiente. | `{"type":"suppressStandaloneNightStep"}` |
| `policies.suppressStandaloneNightStep.fields.type` | Campo admitido por suppressStandaloneNightStep; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.ignoreActorAbilityRestriction` | Resuelve una mecánica externa aunque la habilidad del actor esté anulada. | `{"type":"ignoreActorAbilityRestriction"}` |
| `policies.ignoreActorAbilityRestriction.fields.type` | Campo admitido por ignoreActorAbilityRestriction; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.allowDeadActorWithPendingAction` | Mantiene una acción pendiente tras la muerte. | `{"type":"allowDeadActorWithPendingAction"}` |
| `policies.allowDeadActorWithPendingAction.fields.type` | Campo admitido por allowDeadActorWithPendingAction; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.allowDeadActorWithPendingAction.fields.actionId` | Campo admitido por allowDeadActorWithPendingAction; su valor debe cumplir el contrato tipado. | `{"actionId":"<actionId>"}` |
| `policies.requireRecordedTargetAlignment` | Exige el bando capturado en la acción. | `{"type":"requireRecordedTargetAlignment"}` |
| `policies.requireRecordedTargetAlignment.fields.type` | Campo admitido por requireRecordedTargetAlignment; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.requireRecordedTargetAlignment.fields.actionId` | Campo admitido por requireRecordedTargetAlignment; su valor debe cumplir el contrato tipado. | `{"actionId":"<actionId>"}` |
| `policies.requireRecordedTargetAlignment.fields.alignment` | Campo admitido por requireRecordedTargetAlignment; su valor debe cumplir el contrato tipado. | `{"alignment":"<alignment>"}` |
| `policies.limitSelectionByRecordedActionCount` | Limita la selección por acciones registradas. | `{"type":"limitSelectionByRecordedActionCount"}` |
| `policies.limitSelectionByRecordedActionCount.fields.type` | Campo admitido por limitSelectionByRecordedActionCount; su valor debe cumplir el contrato tipado. | `{"type":"<type>"}` |
| `policies.limitSelectionByRecordedActionCount.fields.actionId` | Campo admitido por limitSelectionByRecordedActionCount; su valor debe cumplir el contrato tipado. | `{"actionId":"<actionId>"}` |
| `policies.limitSelectionByRecordedActionCount.fields.period` | Campo admitido por limitSelectionByRecordedActionCount; su valor debe cumplir el contrato tipado. | `{"period":"<period>"}` |
