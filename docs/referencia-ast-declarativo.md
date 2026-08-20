# Referencia del AST declarativo de mecánicas

Esta es la referencia operativa del lenguaje declarativo de BloodScribe tras la auditoría de abstracción. Describe el contrato `mechanics-1.0` que aceptan los packs, no una API para codificar comportamientos por ID de personaje. Los packs canónicos usan `schemaVersion: "bloodscribe-v3"`; el campo `version` identifica la versión publicable del contenido.

Esta referencia describe el lenguaje declarativo que usa BloodScribe. Sus fuentes de verdad ejecutables dentro del código son:

- `packages/core/src/ruleLanguage.ts`: expresiones, consultas, predicados, relaciones y evaluación.
- `packages/core/src/types.ts`: mecánicas, eventos, entradas, efectos y presentación.
- `packages/core/src/ruleSchema.ts`: catálogo que usa el Pack Builder, campos admitidos y validación.

Los ejemplos usan IDs inventados deliberadamente. Un cambio de ID no debe cambiar el resultado mecánico.

## 1. Forma de una mecánica

Un pack escribe `AuthoredMechanic`. El importador normaliza `when` como `schedule`, `input` como `selection` y completa los valores por defecto del contrato de runtime.

```json
{
  "mechanicId": "vigia-cobalto:night-info:1",
  "tags": ["night-information"],
  "when": {
    "window": "night",
    "cadence": "each"
  },
  "input": { "kind": "none" },
  "usage": {
    "scope": "repeat",
    "limit": { "type": "literal", "value": 1 },
    "consumeOn": "resolution"
  },
  "conditions": [],
  "effects": [
    {
      "type": "emitInformation",
      "value": {
        "type": "query",
        "from": { "entity": "players" },
        "where": { "type": "alive", "value": true },
        "aggregate": { "type": "count" }
      },
      "presentation": {
        "kind": "number",
        "title": "Jugadores vivos"
      },
      "delivery": {
        "audience": { "type": "actor" },
        "timing": "immediate"
      }
    }
  ]
}
```

| Campo | Uso |
|---|---|
| `mechanicId` | Identificador opaco y estable dentro del contenido. No selecciona comportamiento. |
| `tags` | Capacidades semánticas para descubrimiento y composición. |
| `when` | Ventana, etapa, cadencia, evento disparador y demora. |
| `input` | Datos que debe aportar el Narrador o la UI. |
| `usage` | Límite, ámbito del límite y momento de consumo. |
| `conditions` | Lista de `ValueExpr`; todas deben ser verdaderas. |
| `effects` | Efectos declarativos que se resuelven en orden. |
| `policies` | Excepciones tipadas de interacción con los asistentes. |
| `presentation` | Textos, resultados y controles declarados por el pack. |
| `requiresManualModeling` | Indica que la mecánica conserva una parte no representada por el contrato. No sustituye a `manualInstruction`. |

## 2. Vocabulario compartido

### 2.1 Bindings

Un binding apunta a un participante ya conocido por la resolución.

| Valor | Significado | Ejemplo |
|---|---|---|
| `actor` | Jugador que ejecuta la regla. | `{"type":"binding","binding":"actor"}` |
| `selected` | Jugador o jugadores elegidos. | `{"type":"binding","binding":"selected"}` |
| `source` | Jugador que origina el efecto. | `{"type":"binding","binding":"source"}` |
| `nominator` | Jugador que nomina. | `{"type":"binding","binding":"nominator"}` |
| `nominee` | Jugador nominado. | `{"type":"binding","binding":"nominee"}` |
| `eventSubject` | Participante principal del evento disparador. | `{"type":"binding","binding":"eventSubject"}` |
| `effectTarget` | Objetivo que se está resolviendo dentro de un efecto compuesto. | `{"type":"binding","binding":"effectTarget"}` |

`effectTarget` no puede declararse como binding de entrada de un `EventPattern`; solo existe durante la resolución de efectos.

### 2.2 Entidades e identidad

| Grupo | Valores |
|---|---|
| Entidades consultables | `players`, `characters`, `events`, `markers` |
| Identidad de jugador | `real`, `shown`, `registered` |
| Facetas | `teamId`, `allegiance`, `role`, `character`, `mechanicTags` |
| Alineamientos | `good`, `evil`, `neutral` |
| Funciones | `core`, `support`, `independent` |
| Modos de entrada | `cast`, `temporary`; las reglas públicas viven en `gameRules[]` |

- `real`: identidad mecánica real.
- `shown`: identidad entregada o mostrada al jugador.
- `registered`: identidad después de aplicar decisiones de registro.
- Las consultas de `characters` usan la identidad semántica del personaje y por eso no reciben `identityMode`.
- `teamId` y los IDs de personaje son datos del pack. El motor compara valores; nunca ramifica por un ID concreto.

### 2.3 Datos del contexto

| Nodo | Opciones | Ejemplo |
|---|---|---|
| `game` | `phase`, `dayNumber`, `nightNumber` | `{"type":"game","property":"dayNumber"}` |
| `fact` | `outcome`, `guessResult`, `targetMechanicTags` | `{"type":"fact","fact":"outcome"}` |
| `inputValue` | `string`, `number`, `boolean`, `null`, `array`, `object`, `entityId`, `playerId`, `characterId` | `{"type":"inputValue","valueType":"playerId"}` |

## 3. `ValueExpr`: todos los nodos

Toda condición, objetivo, límite, valor informativo y parte calculada se expresa con uno de estos nodos cerrados.

| `type` | Explicación | Ejemplo mínimo |
|---|---|---|
| `literal` | Valor JSON fijo. | `{"type":"literal","value":2}` |
| `binding` | Participante vinculado. | `{"type":"binding","binding":"actor"}` |
| `game` | Dato actual de la partida. | `{"type":"game","property":"nightNumber"}` |
| `setup` | Datos calculados de preparación. | `{"type":"setup","property":"mainEvilBluffCharacterIds"}` |
| `fact` | Dato semántico de la resolución. | `{"type":"fact","fact":"guessResult"}` |
| `storytellerDecision` | Booleano decidido por el Narrador, con parámetros opcionales. | `{"type":"storytellerDecision","decision":"isFair"}` |
| `decisionValue` | Valor tipado de una decisión normalizada ya respondida; si falta, la resolución queda pendiente. | `{"type":"decisionValue","decision":"targetFate"}` |
| `all` | `true` si todas las expresiones son verdaderas. | `{"type":"all","conditions":[{"type":"literal","value":true}]}` |
| `any` | `true` si alguna expresión es verdadera. | `{"type":"any","conditions":[{"type":"literal","value":true}]}` |
| `not` | Niega una expresión booleana. | `{"type":"not","condition":{"type":"literal","value":false}}` |
| `query` | Consulta entidades, filtra, proyecta y agrega. | Véase la sección 4. |
| `compare` | Compara dos valores. | `{"type":"compare","left":{"type":"game","property":"dayNumber"},"operator":"gte","right":3}` |
| `array` | Construye una lista de expresiones. | `{"type":"array","items":[{"type":"literal","value":"a"}]}` |
| `object` | Construye un objeto con campos calculados. | `{"type":"object","entries":{"alive":{"type":"literal","value":true}}}` |
| `concat` | Concatena listas resueltas. | `{"type":"concat","values":[{"type":"literal","value":["a"]},{"type":"literal","value":["b"]}]}` |
| `set` | Aplica `union`, `intersection` o `difference` sin duplicados. | `{"type":"set","operation":"union","values":[{"type":"literal","value":["a"]},{"type":"literal","value":["b"]}]}` |
| `math` | Aplica `add`, `subtract`, `min` o `max` a valores numéricos. | `{"type":"math","operation":"add","values":[{"type":"literal","value":1},{"type":"literal","value":2}]}` |
| `unique` | Elimina duplicados de una lista. | `{"type":"unique","value":{"type":"literal","value":["a","a"]}}` |
| `length` | Devuelve longitud de lista o texto. | `{"type":"length","value":{"type":"literal","value":[1,2]}}` |
| `take` | Conserva los primeros `count` elementos; `count` es entero no negativo. | `{"type":"take","value":{"type":"literal","value":[1,2]},"count":1}` |
| `at` | Obtiene el elemento de índice no negativo. | `{"type":"at","value":{"type":"literal","value":["a"]},"index":0}` |
| `if` | Elige entre `then` y `else`. | `{"type":"if","condition":{"type":"literal","value":true},"then":{"type":"literal","value":"sí"},"else":{"type":"literal","value":"no"}}` |
| `inputValue` | Lee y valida la entrada tipada de la resolución. | `{"type":"inputValue","valueType":"number"}` |

Operadores de `compare`: `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `includes`.

El valor literal permitido es JSON recursivo: `string`, `number`, `boolean`, `null`, listas u objetos compuestos de esos valores.

## 4. Consultas

Forma general:

```json
{
  "type": "query",
  "from": { "entity": "players" },
  "where": { "type": "alive", "value": true },
  "project": {
    "type": "identity",
    "identityMode": "registered",
    "property": "teamId"
  },
  "aggregate": { "type": "collect" }
}
```

`where` y `project` son opcionales. Sin proyección, la consulta devuelve IDs para agregados de colección o entidad. El predicado y la proyección deben corresponder al tipo indicado en `from`.

### 4.1 Conjuntos de origen

| Entidad | Forma | Particularidad |
|---|---|---|
| Jugadores | `{"entity":"players"}` | Puede incorporar una `relation`. |
| Personajes | `{"entity":"characters"}` | Consulta identidades disponibles o en juego. |
| Eventos | `{"entity":"events"}` | Consulta el historial semántico. |
| Fichas | `{"entity":"markers"}` | Consulta fichas y estados proyectados. |

### 4.2 Relaciones de mesa

Solo se aplican a `players`.

`nearestMatching` busca los primeros candidatos que cumplen `where` desde un `anchor`.

```json
{
  "entity": "players",
  "relation": {
    "type": "nearestMatching",
    "anchor": "actor",
    "directions": ["clockwise", "counterclockwise"],
    "limitPerDirection": 1,
    "where": { "type": "alive", "value": true }
  }
}
```

`traverseTable` recorre hasta el primer jugador que cumple el predicado de parada.

```json
{
  "entity": "players",
  "relation": {
    "type": "traverseTable",
    "from": { "type": "binding", "binding": "source" },
    "directions": ["clockwise"],
    "limitPerDirection": 4,
    "stop": {
      "type": "firstMatching",
      "where": { "type": "alive", "value": true }
    }
  }
}
```

`anchor`/`from` admite un binding directo o cualquier `ValueExpr` que resuelva un ID. Las direcciones son `clockwise` y `counterclockwise`; `limitPerDirection` debe ser un entero positivo.

### 4.3 Predicados de jugador

| `type` | Campos y significado | Ejemplo |
|---|---|---|
| `alive` | `value`; `subject?` limita la comprobación al binding. | `{"type":"alive","value":true}` |
| `identity` | `identityMode`, `facet`, `values`, `subject?`; para `mechanicTags`, `match?` puede ser `all` o `any`. | `{"type":"identity","identityMode":"registered","facet":"allegiance","values":["evil"]}` |
| `entryMode` | `values`: `cast`, `temporary`; las reglas públicas no son personajes seleccionables. | `{"type":"entryMode","values":["cast"]}` |
| `identityMatchesBinding` | Compara `teamId`, `allegiance`, `role` o `character` con otro binding bajo un `identityMode`. | `{"type":"identityMatchesBinding","identityMode":"real","facet":"teamId","binding":"actor"}` |
| `state` | `values`, `active?`, `subject?`. | `{"type":"state","values":["drunk"],"active":true}` |
| `marker` | `markerId`, `active?`, `subject?`. | `{"type":"marker","markerId":"protected","active":true}` |
| `isBinding` | `binding`, `value?`; comprueba si la entidad es ese participante. | `{"type":"isBinding","binding":"selected","value":false}` |
| `all` | Todos los predicados de `conditions`. | `{"type":"all","conditions":[{"type":"alive","value":true}]}` |
| `any` | Algún predicado de `conditions`. | `{"type":"any","conditions":[{"type":"alive","value":true}]}` |
| `not` | Niega `condition`. | `{"type":"not","condition":{"type":"alive","value":false}}` |

### 4.4 Predicados de personaje

| `type` | Campos y significado | Ejemplo |
|---|---|---|
| `inPlay` | `value`. | `{"type":"inPlay","value":false}` |
| `identity` | `facet`, `values`; `mechanicTags` admite `match`. No usa `identityMode`. | `{"type":"identity","facet":"mechanicTags","values":["poisoner"],"match":"any"}` |
| `all` | Todos los predicados. | `{"type":"all","conditions":[{"type":"inPlay","value":false}]}` |
| `any` | Algún predicado. | `{"type":"any","conditions":[{"type":"inPlay","value":true}]}` |
| `not` | Niega un predicado. | `{"type":"not","condition":{"type":"inPlay","value":true}}` |

### 4.5 Predicados de evento

| `type` | Campos y significado | Ejemplo |
|---|---|---|
| `eventType` | `values`: tipos semánticos. | `{"type":"eventType","values":["death"]}` |
| `eventCategory` | `values`: categorías declaradas. | `{"type":"eventCategory","values":["night"]}` |
| `eventPeriod` | `value`: `currentDay`, `currentNight`, `previousDay`, `previousNight`, `game`. | `{"type":"eventPeriod","value":"currentDay"}` |
| `eventField` | `field` y literal `value`. | `{"type":"eventField","field":"died","value":true}` |
| `eventRestriction` | `values` y `match?`: `all`, `any` o `exact`; compara el conjunto de restricciones declarado por el efecto. | `{"type":"eventRestriction","values":["ability","vote"],"match":"all"}` |
| `eventParticipant` | `binding` del evento, más `participant?` y `value?` para contrastarlo con un binding actual. | `{"type":"eventParticipant","binding":"nominee","participant":"actor"}` |
| `eventParticipantIdentity` | `participant`, `identityMode`, `facet`, `values`. | `{"type":"eventParticipantIdentity","participant":"eventSubject","identityMode":"registered","facet":"allegiance","values":["good"]}` |
| `all` / `any` / `not` | Composición booleana de predicados de evento. | `{"type":"not","condition":{"type":"eventType","values":["death"]}}` |

Campos de `eventField`: `actionId`, `outcome`, `died`, `attribution`, `resolution`, `known`, `occurrence`, `signal`, `characterId`, `targetCount`, `targetAlignment`, `targetAlive`, `state`, `active`. En muertes, `attribution` identifica el bando de la fuente y `resolution` distingue flujos especiales como `mainEvilAttack`.

### 4.6 Predicados de ficha

| `type` | Campos | Ejemplo |
|---|---|---|
| `markerKind` | `values` | `{"type":"markerKind","values":["reminder"]}` |
| `markerId` | `values` | `{"type":"markerId","values":["protected"]}` |
| `markerActive` | `value` | `{"type":"markerActive","value":true}` |
| `all` / `any` / `not` | Composición booleana. | `{"type":"not","condition":{"type":"markerActive","value":false}}` |

### 4.7 Agregados

| `type` | Resultado | Ejemplo |
|---|---|---|
| `collect` | Todos los resultados. | `{"type":"collect"}` |
| `count` | Cantidad de resultados. | `{"type":"count"}` |
| `exists` | Si existe algún resultado. | `{"type":"exists"}` |
| `all` | Si todos los elementos de origen cumplen el filtro. | `{"type":"all"}` |
| `exactly` | Si hay exactamente `value` resultados. | `{"type":"exactly","value":2}` |
| `first` | Primer resultado. | `{"type":"first"}` |
| `nearest` | Resultado con menor distancia de mesa. | `{"type":"nearest"}` |
| `distance` | Distancia de mesa del resultado. | `{"type":"distance"}` |
| `direction` | `clockwise`, `counterclockwise` o `both`. | `{"type":"direction"}` |
| `adjacentPairCount` | Pares adyacentes; `circular?` incluye el cierre del círculo. | `{"type":"adjacentPairCount","circular":true}` |

### 4.8 Proyecciones

| `type` | Entidad compatible | Opciones | Ejemplo |
|---|---|---|---|
| `entityId` | Todas | — | `{"type":"entityId"}` |
| `identity` | `players` | `identityMode`; `property`: `characterId`, `teamId`, `allegiance`, `role` | `{"type":"identity","identityMode":"shown","property":"characterId"}` |
| `character` | `characters` | `property`: `characterId`, `teamId`, `allegiance`, `role`, `inPlay` | `{"type":"character","property":"teamId"}` |
| `event` | `events` | `property`: `eventId`, `eventType`, `period`, `participantIds` | `{"type":"event","property":"eventType"}` |
| `eventParticipant` | `events` | `binding` | `{"type":"eventParticipant","binding":"nominee"}` |
| `marker` | `markers` | `property`: `markerId`, `kind`, `active`, `entityId` | `{"type":"marker","property":"entityId"}` |

## 5. Eventos, agenda, entrada, uso y duración

### 5.1 `EventPattern`

Tipos de evento: `characterEntry`, `death`, `execution`, `expulsion`, `nomination`, `publicAction`, `mechanicUse`, `mechanicTargeted`, `stateChange`, `restrictionChange`, `markerChange`, `storytellerSignal`.

```json
{
  "type": "event",
  "event": "execution",
  "bindings": {
    "eventSubject": "nominee"
  },
  "where": {
    "type": "eventField",
    "field": "died",
    "value": true
  }
}
```

`bindings` remapea bindings del evento a los de la mecánica. `where` usa `EventPredicateExpr` y filtra por estructura semántica, nunca por texto mostrado.

### 5.2 Agenda `when`

| Campo | Opciones |
|---|---|
| `window` | `setup`, `firstNight`, `night`, `dawn`, `day`, `voting`, `speech`, `nomination`, `execution`, `expulsion`, `dusk`, `mainEvilInfo`, `gameEnd`, `anyTime` |
| `stage` | ID semántico de etapa declarado por el pack. |
| `cadence` | `once`, `each` |
| `startsAt` | Número de día/noche desde el que se activa. |
| `trigger` | Un `EventPattern`. |
| `delay` | `{ "unit": "night", "count": number }`, usando como unidad `night`, `day` o `phase`. |

Ejemplo:

```json
{
  "window": "night",
  "stage": "after-deaths",
  "cadence": "each",
  "startsAt": 2,
  "trigger": { "type": "event", "event": "death" },
  "delay": { "unit": "night", "count": 1 }
}
```

### 5.3 Entrada `input`

En packs se usa `kind`; en runtime se normaliza a `type`.

| `kind` | Campos | Ejemplo |
|---|---|---|
| `none` | — | `{"kind":"none"}` |
| `players` | `min`, `max` (`number` o `all`), `allowRepeated?`, `candidates?`, `constraints?` | `{"kind":"players","min":1,"max":2}` |
| `character` | `candidates?`, `optional?` | `{"kind":"character","optional":true}` |
| `playerAndCharacter` | `player`, `characterCandidates?` | `{"kind":"playerAndCharacter","player":{"kind":"players","min":1,"max":1}}` |
| `text` | `format`: `statement`, `advice`, `wish`, `yesNo` | `{"kind":"text","format":"wish"}` |
| `playerCharacterGuesses` | `max` | `{"kind":"playerCharacterGuesses","max":3}` |
| `seatSwaps` | `minPairs`, `maxPairs` | `{"kind":"seatSwaps","minPairs":1,"maxPairs":2}` |
| `vote` | `mode: "cult"` | `{"kind":"vote","mode":"cult"}` |
| `contest` | — | `{"kind":"contest"}` |

`candidates` es un `ValueExpr`; cada elemento de `constraints` también lo es.

### 5.4 Uso

```json
{
  "scope": "actor",
  "limit": { "type": "literal", "value": 1 },
  "consumeOn": "success",
  "spentReminder": "No Ability"
}
```

- `scope`: `repeat`, `day`, `night`, `game`, `actor`, `target`, `trigger`.
- `limit`: `ValueExpr` que debe resolver un entero no negativo; si se omite, el uso no queda limitado por contador.
- `consumeOn`: `attempt`, `resolution`, `success`.
- `spentReminder`: ficha persistente que materializa el consumo cuando la mecánica necesita impedir usos posteriores.

Para combinar ámbitos se omite `scope` y se declara `keyBy` con una o más dimensiones entre `day`, `night`, `actor`, `target` y `trigger`. `{"keyBy":["day","actor"],"limit":{"type":"literal","value":1}}` significa una vez por actor cada día.

### 5.5 Duración

```json
{ "type": "permanent" }
```

```json
{ "type": "untilWindow", "window": "dusk", "count": 1 }
```

```json
{
  "type": "whileCondition",
  "condition": {
    "type": "query",
    "from": { "entity": "players" },
    "where": { "type": "isBinding", "binding": "source" },
    "aggregate": { "type": "exists" }
  }
}
```

```json
{
  "type": "untilEvent",
  "event": "nomination"
}
```

`untilEvent` conserva los bindings de contexto del efecto y termina cuando coincide el patrón tipado. Permite estados hasta una nominación, una votación, una muerte o el siguiente uso de una mecánica sin interpretar texto visible.

## 6. Efectos declarativos

Los campos comunes de la mayoría de efectos son:

| Campo | Significado |
|---|---|
| `when` | Lista de `ValueExpr`; condiciones locales del efecto. |
| `delay` | Demora en noches, días o fases. |
| `targets` | `ValueExpr` que resuelve uno o varios IDs. |
| `affectedBy` | Estados semánticos que alteran la resolución. |
| `duration` | Duración tipada; algunos efectos históricos aún aceptan una cadena. |
| `optional` | El efecto puede omitirse. |
| `reminder`, `reminderTokens`, `spentReminder` | Presentación y seguimiento de fichas. |

`setPlayerState`, `applyMarker` y `adjustCounter` usan una base cerrada: exigen `targets` y solo aceptan `when`, `delay`, `affectedBy`, duración tipada y sus campos específicos.

### 6.1 Catálogo completo

| `type` | Qué hace | Campos específicos principales | Ejemplo |
|---|---|---|---|
| `death` | Mata los objetivos resueltos. | Resolución especial, bypass de protecciones y metadatos de seguimiento. La atribución se deriva del personaje fuente. | `{"type":"death","targets":{"type":"binding","binding":"selected"}}` |
| `resurrect` | Devuelve objetivos a la vida. | Solo campos comunes. | `{"type":"resurrect","targets":{"type":"binding","binding":"selected"}}` |
| `execute` | Ejecuta objetivos y registra la resolución. | `dies`, `requiresSourceAlive`, `targetReminder`. | `{"type":"execute","targets":{"type":"binding","binding":"selected"},"dies":true}` |
| `setPlayerState` | Activa o desactiva un estado explícito y su ficha física asociada. | `state`, `active`; `targets` obligatorio. | `{"type":"setPlayerState","targets":{"type":"binding","binding":"selected"},"state":"estado-nublado","active":true}` |
| `setPlayerRelation` | Activa o desactiva una relación persistente entre la fuente y cada objetivo. | `kind`, `active`, `duration?`, `ownership?`, `markerId?`; `targets` obligatorio. | `{"type":"setPlayerRelation","targets":{"type":"binding","binding":"selected"},"kind":"protegido","active":true}` |
| `applyMarker` | Añade o retira una ficha recordatoria. | `kind:"reminder"`, `id`, `active`, `exclusive?`, `ownership?`. | `{"type":"applyMarker","targets":{"type":"binding","binding":"selected"},"kind":"reminder","id":"watched","active":true}` |
| `moveMarker` | Transfiere atómicamente una ficha entre jugadores. | `kind:"reminder"`, `id`, `from`; `targets` indica el destino. | `{"type":"moveMarker","kind":"reminder","id":"crown","from":{"type":"binding","binding":"actor"},"targets":{"type":"binding","binding":"selected"}}` |
| `adjustCounter` | Ajusta un contador persistente y puede proyectar fichas/estados o disparar efectos al cruzar un umbral. | `counter`, `delta`, `scope?`, `bounds?`, `projection?`, `stateProjection?`, `threshold?`, `onThreshold?`. | `{"type":"adjustCounter","targets":{"type":"binding","binding":"actor"},"counter":"charges","delta":1,"bounds":{"max":3}}` |
| `changeAlignment` | Cambia el alineamiento. | `alignment`, `allowsSelfTarget`, `notifyPlayer`, `setupEffect`, `targetAlignment`, `targetTeam`. | `{"type":"changeAlignment","targets":{"type":"binding","binding":"selected"},"alignment":"evil"}` |
| `changeCharacter` | Sustituye una identidad. | Identidad real/mostrada, equipo, límites de elección y consecuencias de la sustitución. | `{"type":"changeCharacter","targets":{"type":"binding","binding":"selected"},"newCharacter":"personaje-zafiro","preserveAlignment":true}` |
| `grantAbility` | Concede o retira una habilidad declarada. | `active`, `abilityCharacterId`, `owner`, `controller`, `ownership`. | `{"type":"grantAbility","targets":{"type":"binding","binding":"selected"},"abilityCharacterId":"habilidad-ambar","active":true,"ownership":"sourceAbility"}` |
| `swapSeats` | Intercambia posiciones en el círculo. | Solo campos comunes. | `{"type":"swapSeats","targets":{"type":"binding","binding":"selected"}}` |
| `swapCharacters` | Intercambia identidades entre participantes. | `actor`, `resultingState`, `resultingStateDuration`, `swapsCharactersAndAlignments`. | `{"type":"swapCharacters","targets":{"type":"binding","binding":"selected"},"swapsCharactersAndAlignments":true,"resultingState":"estado-nublado"}` |
| `swapTargets` | Intercambia objetivos ya resueltos. | Solo campos comunes. | `{"type":"swapTargets","targets":{"type":"binding","binding":"selected"}}` |
| `emitInformation` | Calcula y entrega información tipada. | `value`, `presentation`, `delivery`, `transform?`; los tres primeros son obligatorios. | Véase 6.2. |
| `prepareInformation` | Prepara información verdadera/alternativa para entrega posterior. | `candidates`, `modes`, `zeroWhen?`, `characterChoice`, `reminders`, `shownIdentityOverride?`. | Véase 6.3. |
| `resolveGameEnd` | Declara o programa ganador. | `mode`, `winner`, `reason`; `delay` obligatorio solo en modo `queued`. | Véase 6.4. |
| `interceptEvent` | Cancela o redirige un evento semántico. | `event`, `match?`, `reaction`, `priority?`, `consumption?`, `appliesWhenProtectionBypassed?`. | Véase 6.5. |
| `disableAbility` | Desactiva una habilidad según el contrato. | `blocks`, `consumeOnPass`, `informationMayBeFalse`. | `{"type":"disableAbility","targets":{"type":"binding","binding":"selected"},"blocks":"ability"}` |
| `restrict` | Limita acciones disponibles. | `actions`, `requiresSourceAlive`, `restrictions`, `relation`, `informationMayBeFalse`. | `{"type":"restrict","targets":{"type":"binding","binding":"selected"},"actions":["vote"]}` |
| `registerAs` | Cambia cómo se registra una identidad. | `affects`, `alignment`, `roles`, `characterIds`, `lifeState`, `mode`, `registersAs`, `teamIds`, `triggerReminder`, `worksWhenDead`. | `{"type":"registerAs","targets":{"type":"binding","binding":"actor"},"registersAs":["equipo-carmesi"],"worksWhenDead":true}` |
| `modifyTargets` | Amplía o reduce los objetivos de habilidades que cumplan un perfil. | `delta` obligatorio; `sourceProfile?`, `targetMechanicTags?`. | `{"type":"modifyTargets","delta":-1,"sourceProfile":{"allegiances":["evil"]}}` |
| `modifyVote` | Cambia el peso de los votantes resueltos. | `weight`, `pairedTargets?`, `pairedWeight?`; los pesos aceptan número o `ValueExpr`. | `{"type":"modifyVote","targets":{"type":"binding","binding":"actor"},"weight":2}` |
| `modifySetup` | Aplica operaciones tipadas al setup. | `operations` obligatorio. | Véase 6.6. |
| `modifyInformation` | Cambia reglas de información antes de entregarla. | `audience`, `mustBeFalse`. | `{"type":"modifyInformation","mustBeFalse":true,"audience":{"type":"actor"}}` |
| `modifyNomination` | Cambia la resolución de nominación. | `abilitySpentEvenIfNoExecution`, `countsAsExecution`, `createsReminder`, `voteDelta`, `requiresActorAbstention`. | `{"type":"modifyNomination","voteDelta":1}` |
| `recordAction` | Registra una acción pública y un resultado semántico. | `actionId`, `outcome` y campos históricos de compatibilidad. | `{"type":"recordAction","actionId":"declaracion-rubi","outcome":{"type":"inputValue","valueType":"string"}}` |
| `storytellerDecision` | Solicita una decisión humana nombrada. | `decision`; `options` activa el contrato normalizado. | `{"type":"storytellerDecision","decision":"targetFate","options":["dies","survives"]}` |
| `manualInstruction` | Muestra una resolución guiada que el motor no ejecuta. | `instruction`, `reason`, `ruleStepActivation` y metadatos de ayuda. | `{"type":"manualInstruction","instruction":"Comprueba el acuerdo de la mesa.","reason":"El consenso no es un hecho mecánico tipado."}` |

Campos admitidos adicionales de los efectos heredados más amplios:

- `death`: `bypassesDeathProtection`, `bypassesProtection`, `chargeReminder`, `conditionTarget`, `consumeTargetReminder`, `continuesNomination`, `createsReminder`, `jumpReminder`, `mustNominateOnDay3`, `protection`, `publicAnnouncement`, `registration`, `requiredActionOutcome`, `requiresSourceAlive`, `resolution`, `resultOutcome`, `secondaryReminder`, `targetAlignment`, `targetNotTeam`, `targetNotProfile`, `targetReminder`, `triggerActionDayScope`, `triggerActionId`, `triggerReminder`, `useRegisteredIdentity`.
- `changeCharacter`: `allowedBuckets`, `arbitraryDeathsIfMainEvilCreated`, `characterType`, `characterProfile`, `countTeams`, `countProfiles`, `countTiming`, `createsReminder`, `markOriginalAbilityHolderIfInPlay`, `gainsAbilityOfChosenCharacter`, `minimumAlive`, `mustNeighborTeam`, `mustNeighborProfile`, `newCharacter`, `newTeam`, `newProfile`, `oldMainEvilDies`, `preserveAlignment`, `realCharacter`, `realTeam`, `realProfile`, `result`, `shownAs`, `targetCharacter`.
- `recordAction`: `action`, `actionId`, `outcome`, `creates`, `effect`, `failureEffect`, `failureOutcome`, `maxGuesses`, `resolvesExecution`, `restrictions`, `successOutcome`, `targetMechanicTags`, `targetRegistrationTeams`.
- `storytellerDecision`: `decision`, `options`; la pregunta y los labels viven en `presentation.decisionPrompts[decision]`.
- `manualInstruction`: `affects`, `aliveThreshold`, `bluffCountOptions`, `durationMinutes`, `instruction`, `publicKnown`, `reason`.

Un nombre de estado nunca decide comportamiento. El pack compone `setPlayerState` con `restrict`, declara su ficha y decide su label. `informationMayBeFalse` solo altera la información producida por la habilidad restringida: una notificación real del propio estado conserva su verdad. Por ejemplo, un estado que incluya `wake` no ejecuta el despertar normal y puede entregar su aviso privado al final de la noche con `delivery.timing: "nextRecipientWake"` y `delivery.separateWake: true`, antes de pasar al amanecer público.

### 6.2 Información emitida

```json
{
  "type": "emitInformation",
  "value": {
    "type": "query",
    "from": { "entity": "players" },
    "where": {
      "type": "identity",
      "identityMode": "registered",
      "facet": "allegiance",
      "values": ["evil"]
    },
    "aggregate": { "type": "count" }
  },
  "presentation": {
    "kind": "number",
    "title": "Cantidad detectada",
    "emptyLabel": "Ninguno"
  },
  "delivery": {
    "audience": { "type": "actor" },
    "timing": "immediate",
    "mode": "shared"
  },
  "transform": {
    "type": "whenSourceAffected",
    "affectedBy": ["drunk", "poisoned"],
    "reaction": { "type": "allowArbitraryValue" }
  }
}
```

`presentation.kind`: `boolean`, `number`, `text`, `player`, `players`, `character`, `characters`, `identity`, `direction`, `distance`, `grimoire`, `structured`.

La presentación admite `title`, `description`, `fields`, `options`, `trueLabel`, `falseLabel`, `emptyLabel`, `showHandIcon`. `options` declara el dominio cerrado de una salida escalar como pares `{ value, label }`; el motor conserva `value` y la interfaz muestra `label`, por lo que dirección, identidad y otras elecciones no dependen del ID del personaje. Cada field declara `key`, `label`, `kind` y opcionalmente `required`, `min`, `max`, `options`.

Audiencias: `actor`, `selected`, `eventSubject`, `public`, `storyteller` o `players` con un `ValueExpr`. La entrega admite `timing: immediate | dawn | privateDay | nextRecipientWake`, `mode: shared | perRecipient`, `separateWake: true` para un despertar privado terminal y consentimiento de `selected` o `eventSubject`.

### 6.3 Información preparada

```json
{
  "type": "prepareInformation",
  "candidates": {
    "type": "query",
    "from": { "entity": "players" },
    "where": { "type": "alive", "value": true },
    "aggregate": { "type": "collect" }
  },
  "modes": ["pair", "zero"],
  "zeroWhen": { "type": "storytellerDecision", "decision": "showZero" },
  "characterChoice": {
    "source": "truthfulPlayer",
    "identityMode": "registered"
  },
  "reminders": {
    "truthful": "Verdadero",
    "alternative": "Alternativa"
  }
}
```

`characterChoice` puede usar el jugador verdadero (`real` o `registered`) o una expresión de candidatos: `{"source":"candidates","candidates": ValueExpr}`. `shownIdentityOverride` repite el mismo plan para identidad mostrada.

### 6.4 Final de partida

Ganador inmediato fijo:

```json
{
  "type": "resolveGameEnd",
  "mode": "immediate",
  "winner": { "type": "fixed", "team": "good" },
  "reason": "Se cumplió la condición declarada."
}
```

Ganador programado y opuesto al equipo del origen:

```json
{
  "type": "resolveGameEnd",
  "mode": "queued",
  "winner": {
    "type": "opposite",
    "of": {
      "type": "source",
      "value": {
        "type": "query",
        "from": { "entity": "players" },
        "where": { "type": "isBinding", "binding": "source" },
        "project": {
          "type": "identity",
          "identityMode": "real",
          "property": "allegiance"
        },
        "aggregate": { "type": "first" }
      }
    }
  },
  "delay": { "unit": "phase", "count": 1 },
  "reason": "Final aplazado."
}
```

Formas de ganador:

- `fixed`: equipo ganador fijo.
- `source`: el equipo se obtiene de un `ValueExpr`.
- `opposite`: opuesto de `fixed` o `source`.
- `condition`: lista de `{when, winner}` y un `otherwise`.
- `personal`: `players` es un `ValueExpr`; solo está permitido en modo inmediato.

### 6.5 Intercepción de eventos

```json
{
  "type": "interceptEvent",
  "event": "death",
  "match": {
    "type": "eventParticipant",
    "binding": "eventSubject",
    "participant": "selected"
  },
  "reaction": { "type": "cancel" },
  "priority": 50,
  "consumption": {
    "type": "once",
    "reminder": "proteccion-usada"
  }
}
```

La reacción puede ser `cancel` o `redirect`; esta última exige `targets: ValueExpr`. El consumo opcional es de una sola vez y admite dos formas excluyentes:

- `{"type":"once","reminder":"proteccion-usada"}` consume una ficha recordatoria de una fuente con jugador.
- `{"type":"once","recordMechanicUse":true}` persiste el uso sobre la propia mecánica, también para reglas de partida sin jugador fuente.

`appliesWhenProtectionBypassed: true` declara que el interceptor también puede actuar cuando el evento fuente ignora la protección normal. Si se omite, una muerte con `bypassesProtection` no queda interceptada.

### 6.6 Modificación de setup

Operaciones disponibles:

| `type` | Campos | Ejemplo |
|---|---|---|
| `adjustBucket` | `bucket`, `delta` | `{"type":"adjustBucket","bucket":"outsider","delta":1}` |
| `chooseAdjustments` | `options`, cada una con `adjustments` | `{"type":"chooseAdjustments","options":[{"adjustments":[{"bucket":"outsider","delta":1}]}]}` |
| `replaceBucket` | `from`, `to`, `count` (`number` o `all`) | `{"type":"replaceBucket","from":"minion","to":"outsider","count":1}` |
| `setBucketCount` | `bucket`, `count: SetupCountSpec` | `{"type":"setBucketCount","bucket":"demon","count":{"type":"literal","value":1}}` |
| `allowDuplicates` | `targets: ValueExpr`, `copies: SetupCountSpec` | `{"type":"allowDuplicates","targets":{"type":"binding","binding":"selected"},"copies":{"type":"sourceCopies"}}` |
| `requireCharacter` | `characterId`; `onMissing` puede ser `add` o `error` | `{"type":"requireCharacter","characterId":"personaje-ambar","onMissing":"add"}` |
| `requireBucket` | `bucket`, `minimum`, `replaceFrom`; garantiza el mínimo sustituyendo otra categoría solo cuando haga falta | `{"type":"requireBucket","bucket":"minion","minimum":1,"replaceFrom":"townsfolk"}` |
| `removeCharacters` | `targets: ValueExpr` | `{"type":"removeCharacters","targets":{"type":"binding","binding":"selected"}}` |
| `configureDeal` | `assignment`, `tokenDraw`, `playerKnowledge` | `{"type":"configureDeal","assignment":"players","tokenDraw":"bag","playerKnowledge":"shown"}` |

`SetupCountSpec` admite:

- `{"type":"literal","value":2}`
- `{"type":"choice","default":"base","values":[0,1,2]}`
- `{"type":"playerCount"}`
- `{"type":"majorityOfPlayers","leaveAtLeast":1}`
- `{"type":"sumBuckets","buckets":["minion","demon"]}`
- `{"type":"sourceCopies"}`
- `{"type":"remainder"}`

## 7. Políticas de interacción

Las políticas son átomos cerrados que conectan una mecánica con los asistentes sin introducir registros por personaje.

| `type` | Explicación | Campos adicionales |
|---|---|---|
| `recordTargetSelectionWhenDisabled` | Conserva la selección aunque el actor esté deshabilitado. | — |
| `presentStartingInformationAsShownIdentity` | Presenta transitoriamente la información inicial de la identidad mostrada. | — |
| `relayShownSelection` | Registra y transmite la selección hecha mediante la identidad mostrada. | — |
| `assignSelectedCharacter` | Asigna el personaje seleccionado. | — |
| `grantSelectedAbility` | Concede la habilidad elegida. | `target`, `abilityAlignment`, `duration`, `requireInPlay` |
| `overrideChooserAlignment` | Fuerza el alineamiento usado por el selector. | `alignment` |
| `continueAfterTargetReaction` | Continúa después de la reacción del objetivo. | — |
| `suppressStandaloneNightStep` | Evita un paso nocturno independiente. | — |
| `ignoreActorAbilityRestriction` | Resuelve una mecánica externa aunque la habilidad del actor esté anulada. | — |
| `allowDeadActorWithPendingAction` | Mantiene una acción pendiente tras la muerte. | `actionId` |
| `requireRecordedTargetAlignment` | Exige el alineamiento capturado al registrar una acción. | `actionId`, `alignment` |
| `limitSelectionByRecordedActionCount` | Limita la selección por recuento de acciones registradas. | `actionId`; `period` puede ser `currentDay` o `game` |

Ejemplo:

```json
{
  "policies": [
    {
      "type": "requireRecordedTargetAlignment",
      "actionId": "marca-cobalto",
      "alignment": "good"
    }
  ]
}
```

## 8. Presentación declarativa

`presentation` no ejecuta reglas: explica y configura cómo se muestran. Puede declarar `label`, `title`, `description`, `affectedMessage`, `outcomeLabel`, `outcomes`, `actionLabel`, `deathLabel`, `survivalLabel`, `nightGuide`, `nightControl` y `localizations`.

`presentation.action` configura una acción pública o de nominación con:

- `actionId`, `surface`, `resolutionKind`, `defaultOutcome`.
- `allowDeadActor`, `captureTargetAlignment`, `exclusiveWhilePending`.
- instrucciones y labels de objetivo/texto público o privado.
- consumo de habilidad y resultados que no la consumen.
- `contestVotes` para una contienda.

Ningún texto de presentación se interpreta para decidir comportamiento.

## 9. Cuándo usar `storytellerDecision` y cuándo `manualInstruction`

Hay dos conceptos distintos:

- `storytellerDecision` es una entrada humana **tipada dentro de una regla que el motor sí puede continuar evaluando**. Puede aparecer como `ValueExpr` o como efecto asistido.
- `manualInstruction` es un efecto terminal o parcial que **el motor no ejecuta**. Solo guía al Narrador y debe explicar por qué falta modelado.

Una decisión booleana histórica puede seguir apareciendo como `ValueExpr`. Para contenido nuevo con botones y consecuencias debe declararse un efecto normalizado, su presentación y efectos condicionados por `decisionValue`:

```json
{
  "effects": [
    {
      "type": "storytellerDecision",
      "decision": "targetFate",
      "options": ["dies", "survives"]
    },
    {
      "type": "death",
      "targets": { "type": "binding", "binding": "selected" },
      "when": [{
        "type": "compare",
        "left": { "type": "decisionValue", "decision": "targetFate" },
        "operator": "eq",
        "right": "dies"
      }]
    }
  ],
  "presentation": {
    "decisionPrompts": {
      "targetFate": {
        "question": "¿Qué ocurre con {targetName}?",
        "options": [
          { "value": "dies", "label": "Muere" },
          { "value": "survives", "label": "Sobrevive" }
        ]
      }
    }
  }
}
```

Las opciones mecánicas y las opciones de presentación deben coincidir. Una respuesta ausente no se sustituye por un valor: `decisionValue` produce una resolución pendiente. Al confirmar, el runtime registra `record-mechanic-decision` y aplica los efectos derivados en el mismo lote.

Ejemplo realmente manual:

```json
{
  "type": "manualInstruction",
  "instruction": "Decide qué equipo fue responsable del acuerdo social.",
  "reason": "La responsabilidad social es una valoración subjetiva."
}
```

Regla práctica: una elección humana no obliga por sí sola a salir del AST. Si la elección produce un valor tipable y después se aplican efectos conocidos, debe ser una `storytellerDecision` normalizada y sus consecuencias deben consultar `decisionValue`. `inputValue` queda para datos introducidos por el control principal; `manualInstruction`, para acciones, eventos, estados históricos o juicios que el contrato todavía no puede representar.

## 10. Censo actual de `manualInstruction`

Censo realizado sobre los ocho seeds mantenidos de `docs/content/botc/source/scripts/` el 27 de julio de 2026. Las cifras cuentan apariciones por pack; una misma entidad incluida en dos packs cuenta dos veces.

| Métrica | Cantidad |
|---|---:|
| Mecánicas que contienen `manualInstruction` | 95 |
| Mecánicas cuyo único efecto es manual | 80 |
| Mecánicas mixtas: efecto tipado + tramo manual | 15 |
| Mecánicas con `requiresManualModeling: true` | 7 |
| `requiresManualModeling` sin `manualInstruction` | 0 |
| Motivo genérico `storytellerJudgement` | 76 |
| Sin campo `reason` | 8 |
| Con una carencia concreta documentada | 11 |

Distribución: Bad Moon Rising 5, Fabled 17, Grimm Hansel & Gretel 4, Loric 13, The Carousel 42, contenido temporal BOTC 14, Sects & Violets 0 y Trouble Brewing 0.

### 10.1 Las 7 marcadas explícitamente como modelado manual

Estas son el núcleo que el corpus declara como no representable de extremo a extremo con el contrato actual:

| Pack | Entidad | `mechanicId` | Carencia declarada |
|---|---|---|---|
| Grimm Hansel & Gretel | El Pájaro Blanco | `pajaro-blanco:win-condition:1` | La atribución histórica de observaciones exige confirmación humana. |
| Loric | Hindu | `hindu:win-condition:1` | Transformación y disponibilidad de reemplazo legal. |
| The Carousel | Atheist | `atheist:win-condition:1` | La ejecución del Narrador no es un evento tipado. |
| The Carousel | Atheist | `atheist:loss-condition:1` | La ejecución del Narrador no es un evento tipado. |
| The Carousel | Politician | `politician:win-condition:1` | La responsabilidad decisiva es subjetiva. |
| The Carousel | Damsel | `damsel:loss-condition:1` | Falta una acción tipada para la declaración pública. |
| The Carousel | Summoner | `summoner:loss-condition:1` | La disponibilidad de personajes exige confirmación humana. |

`requiresManualModeling` no significa que toda la mecánica deba quedarse manual para siempre. Señala una frontera conocida y auditable. Si se añade una primitiva general que cubre esa frontera, debe retirarse tanto el flag como el `manualInstruction` correspondiente.

### 10.2 Otras carencias concretas

La aplicación declarativa de restricciones ya es interceptable mediante `restrictionChange`: una protección puede comparar el conjunto mecánico (`ability`, `vote`, `speak`, etc.) sin depender del nombre del estado ni del personaje que lo origina. La expulsión también es interceptable mediante `expulsion`; las protecciones declarativas específicas pueden cancelarla sin convertirla en ejecución.

### 10.3 Mecánicas mixtas restantes

Estas ya tienen una resolución mecánica parcial y no deberían describirse como «totalmente manuales».

| Pack | Entidad | `mechanicId` | Efectos tipados que acompañan al manual |
|---|---|---|---|
| Bad Moon Rising | Minstrel | `minstrel:global-rule:1` | `setPlayerState` |
| Bad Moon Rising | Goon | `goon:status-retaliation:1` | `setPlayerState`, `applyMarker` |
| Fabled | Deus ex Fiasco | `deusexfiasco:public-day-action:1` | `recordAction` |
| Fabled | Revolutionary | `revolutionary:setup-reminder:1` | `applyMarker` |
| Loric | Knaves | `knaves:public-day-action:1` | `recordAction` |
| The Carousel | Fearmonger | `fearmonger:reminder-choice:1` | `applyMarker` |
| The Carousel | Xaan | `xaan:setup-modifier:1` | `modifySetup` |
| The Carousel | Marionette | `marionette:setup-modifier:1` | `modifySetup` |
| The Carousel | Boomdandy | `boomdandy:public-day-action:2` | `recordAction`, `death` |
| The Carousel | Riot | `riot:setup-modifier:1` | `modifySetup` |
| The Carousel | Deus ex Fiasco | `deusexfiasco:public-day-action:1` | `recordAction` |
| Contenido temporal BOTC | Judge | `judge:public-day-action:1` | `recordAction` |

### 10.4 Inventario completo por pack

Leyenda de cobertura:

- `manual`: el único efecto de la mecánica es `manualInstruction`.
- `mixta (...)`: conserva los efectos tipados indicados y añade una instrucción manual.
- `manual !`: además declara `requiresManualModeling: true`.

#### Bad Moon Rising

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| Sailor | `sailor:status-choice:1` | manual | `chooseStateRecipient` |
| Innkeeper | `innkeeper:status-assignment:1` | manual | `chooseStateRecipient` |
| Courtier | `courtier:status-choice:1` | manual | `manageTimedStateSequence` |
| Minstrel | `minstrel:global-rule:1` | mixta (`setPlayerState`) | `globalRule` |
| Goon | `goon:status-retaliation:1` | mixta (`setPlayerState`, `applyMarker`) | `resolveTargetReactionInteractionExceptions` |

#### Fabled

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| Angel | `angel:global-rule:1` | manual | `removeAngelOnFinalDay` |
| Buddhist | `buddhist:global-rule:1` | manual | `announceBuddhistPlayers` |
| Buddhist | `buddhist:global-rule:2` | manual | `veteranPlayersSilentFirstTwoMinutes` |
| Deus ex Fiasco | `deusexfiasco:global-rule:1` | manual | `announceDeusExFiascoInPlay` |
| Deus ex Fiasco | `deusexfiasco:public-day-action:1` | mixta (`recordAction`) | `correctAndAdmitStorytellerMistake` |
| Deus ex Fiasco | `deusexfiasco:global-rule:2` | manual | `storytellerMustMakeAtLeastOneCorrectedMistake` |
| Djinn | `djinn:global-rule:1` | manual | `announceAllDjinnJinxRules` |
| Djinn | `djinn:global-rule:2` | manual | `applyScriptToolJinxRules` |
| Ferryman | `ferryman:global-rule:1` | manual | `restoreDeadVoteTokensOnFinalDay` |
| Fiddler | `fiddler:global-rule:1` | manual | `allPlayersVoteForFiddlerContestWinner` |
| Hell's Librarian | `hellslibrarian:global-rule:1` | manual | `storytellerMayRequestSilence` |
| Revolutionary | `revolutionary:global-rule:1` | manual | `twoConsentingNeighborPlayersShareAlignment` |
| Revolutionary | `revolutionary:setup-reminder:1` | mixta (`applyMarker`) | `selectedPlayersMustBeAdjacent` |
| Revolutionary | `revolutionary:global-rule:2` | manual | `revolutionaryAlignmentChangeAffectsBothOrNeither` |
| Spirit of Ivory | `spiritofivory:global-rule:1` | manual | `atMostOneExtraEvilPlayer` |
| Toymaker | `toymaker:global-rule:1` | manual | `evilPlayersGetNormalStartingInfoInSmallGames` |
| Toymaker | `toymaker:global-rule:2` | manual | `demonMustPassAttackOnceBeforeGameEndingAttack` |

#### Grimm Hansel & Gretel

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| El Pájaro Blanco | `pajaro-blanco:win-condition:1` | manual ! | Comprobar vida y observaciones de los equipos requeridos. |

#### Loric

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| Bootlegger | `bootlegger:global-rule:1` | manual | `announceHomebrewCharactersAndRules` |
| Bootlegger | `bootlegger:global-rule:2` | manual | `applyDeclaredHomebrewRules` |
| Big Wig | `bigwig:global-rule:1` | manual | `onlyChosenSpeakerMaySpeakUntilVote` |
| God of Ug | `godofug:global-rule:1` | manual | `hatWearerVoteCountsDouble` |
| God of Ug | `godofug:global-rule:2` | manual | `hatWearerMustSpeakOneSyllableWords` |
| Hindu | `hindu:win-condition:1` | manual ! | Comprobar transformación del Demonio y reemplazo legal. |
| Knaves | `knaves:global-rule:1` | manual | `twoStorytellersOneTruthOneLie` |
| Knaves | `knaves:public-day-action:1` | mixta (`recordAction`) | `swapTruthAndLieStorytellers` |
| Pope | `pope:global-rule:1` | manual | `mainEvilBluffsMayIncludeDuplicateGoodCharacters` |
| Storm Catcher | `stormcatcher:global-rule:1` | manual | `announceFavouredGoodCharacter` |
| Tor | `tor:global-rule:1` | manual | `evilPlayersDoNotLearnEachOther` |
| Ventriloquist | `ventriloquist:global-rule:1` | manual | `clearVentriloquistMadReminder` |
| Zenomancer | `zenomancer:global-rule:1` | manual | `zenomancerInfoIsTrueEvenIfDrunkOrPoisoned` |

#### The Carousel

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| King | `king:global-rule:1` | manual | `globalRule` |
| Lycanthrope | `lycanthrope:global-rule:1` | manual | `demonCannotKillAfterLycanthropeGoodKill` |
| Alchemist | `alchemist:global-rule:1` | manual | `hasMinionAbilityWithoutEvilInfo` |
| Cannibal | `cannibal:global-rule:1` | manual | `gainRecentlyKilledExecuteeAbility` |
| Amnesiac | `amnesiac:global-rule:1` | manual | `storytellerDefinesAmnesiacAbility` |
| Banshee | `banshee:global-rule:1` | manual | `bansheeDoubleNominationAndVote` |
| Poppy Grower | `poppygrower:global-rule:1` | manual | `evilPlayersDoNotLearnEachOther` |
| Atheist | `atheist:global-rule:1` | manual | `storytellerMayBreakRules` |
| Atheist | `atheist:win-condition:1` | manual ! | Resolver ejecución del Narrador con Atheist en juego. |
| Atheist | `atheist:loss-condition:1` | manual ! | Resolver ejecución del Narrador sin Atheist en juego. |
| Hermit | `hermit:global-rule:1` | manual | `hasAllOutsiderAbilitiesOnScript` |
| Golem | `golem:global-rule:1` | manual | `golemMayNominateOnce` |
| Plague Doctor | `plaguedoctor:global-rule:1` | manual | `storytellerGainsMinionAbility` |
| Politician | `politician:win-condition:1` | manual ! | Decidir si fue el principal responsable de la victoria. |
| Zealot | `zealot:global-rule:1` | manual | `mustVoteWhenFiveOrMoreAlive` |
| Damsel | `damsel:loss-condition:1` | manual ! | Resolver la declaración pública que identifica a Damsel. |
| Heretic | `heretic:global-rule:1` | manual | `invertWinLoss` |
| Fearmonger | `fearmonger:reminder-choice:1` | mixta (`applyMarker`) | `announceMarkerHolderChange` |
| Wizard | `wizard:global-rule:1` | manual | `storytellerMayGrantWishWithPriceAndClue` |
| Xaan | `xaan:setup-modifier:1` | mixta (`modifySetup`) | La cantidad de outsiders elegida determina cuándo actúa. |
| Marionette | `marionette:setup-modifier:1` | mixta (`modifySetup`) | Retirar la identidad real de la bolsa y entregar una identidad mostrada. |
| Wraith | `wraith:global-rule:1` | manual | `mayOpenEyesWithEvilPlayers` |
| Summoner | `summoner:global-rule:1` | manual | `noDemonDoesNotEndGameBeforeSummonerActs` |
| Summoner | `summoner:loss-condition:1` | manual ! | Comprobar si no existe ningún Demonio legal. |
| Boomdandy | `boomdandy:public-day-action:2` | mixta (`recordAction`, `death`) | `advanceToDuskAfterBoomdandyCountdown` |
| Vizier | `vizier:global-rule:1` | manual | `publicCharacterAnnouncement` |
| Organ Grinder | `organgrinder:global-rule:1` | manual | `secretVotingAndTally` |
| Boffin | `boffin:global-rule:1` | manual | `demonHasNotInPlayGoodAbility` |
| Legion | `legion:global-rule:1` | manual | `executionsFailIfOnlyEvilVoted` |
| Lord of Typhon | `lordoftyphon:global-rule:1` | manual | `evilCharactersMustBeContiguousWithDemonInMiddle` |
| Al-Hadikhia | `alhadikhia:global-rule:1` | manual | `chosenPlayersSilentlyChooseLiveOrDie` |
| Riot | `riot:global-rule:1` | manual | `riotDayCounter` |
| Riot | `riot:setup-modifier:1` | mixta (`modifySetup`) | Informar que los slots sustituidos comparten identidad. |
| Riot | `riot:global-rule:2` | manual | `storytellerMustNominateIfChainStops` |
| Leviathan | `leviathan:global-rule:1` | manual | `advanceLeviathanDayMarker` |
| Leviathan | `leviathan:global-rule:2` | manual | `leviathanDayCounter` |
| Gnome | `gnome:global-rule:1` | manual | `allPlayersLearnAmigoOfGnome` |
| Deus ex Fiasco | `deusexfiasco:global-rule:1` | manual | `announceDeusExFiascoInPlay` |
| Deus ex Fiasco | `deusexfiasco:public-day-action:1` | mixta (`recordAction`) | `correctAndAdmitStorytellerMistake` |
| Deus ex Fiasco | `deusexfiasco:global-rule:2` | manual | `storytellerMustMakeAtLeastOneCorrectedMistake` |
| Ferryman | `ferryman:global-rule:1` | manual | `restoreDeadVoteTokensOnFinalDay` |
| Storm Catcher | `stormcatcher:global-rule:1` | manual | `announceFavouredGoodCharacter` |

#### Personajes temporales del pack BOTC

| Entidad | `mechanicId` | Cobertura | `instruction` |
|---|---|---|---|
| Barista | `barista:reminder-choice:1` | manual | `chooseAndApplyDeclaredMarkerMode` |
| Barista | `barista:global-rule:1` | manual | `baristaChosenModeAppliesUntilDusk` |
| Butcher | `butcher:global-rule:1` | manual | `allowSecondNominationAfterFirstExecution` |
| Matron | `matron:global-rule:1` | manual | `playersMayOnlyPrivateTalkFromSeats` |
| Judge | `judge:public-day-action:1` | mixta (`recordAction`) | `forceExecutionPassOrFail` |
| Apprentice | `apprentice:global-rule:1` | manual | `temporaryCharacterHasChosenAbilityUntilDeath` |
| Beggar | `beggar:global-rule:1` | manual | `beggarMustSpendDonatedVoteTokenToVote` |
| Scapegoat | `scapegoat:global-rule:1` | manual | `scapegoatExecutionCountsAsExecution` |
| Gnome | `gnome:global-rule:1` | manual | `allPlayersLearnAmigoOfGnome` |
| Bishop | `bishop:global-rule:1` | manual | `onlyStorytellerMayNominate` |
| Voudon | `voudon:global-rule:1` | manual | `onlyVoudonAndDeadMayVote` |
| Voudon | `voudon:global-rule:2` | manual | `deadVotesDoNotSpendGhostVote` |
| Voudon | `voudon:global-rule:3` | manual | `highestVoteTotalExecutesWithoutMajorityThreshold` |

Sects & Violets y Trouble Brewing no contienen ningún `manualInstruction` en sus seeds actuales.

### 10.5 Interpretación del censo

Los casos del censo no son carencias inevitables del AST:

- Los 7 marcados con `requiresManualModeling` son la frontera manual explícita del contenido actual.
- Los casos adicionales con motivo concreto señalan eventos aún ausentes.
- Los mixtos ya tienen una base tipada; su siguiente paso es sustituir solo la instrucción residual.
- Los 76 `storytellerJudgement` son un backlog heterogéneo. Algunos son decisiones humanas legítimas; otros son reglas globales, anuncios, votación, información inicial o seguimiento que podrían convertirse en primitivas generales.
- Los 8 casos sin `reason` deberían recibir un motivo concreto o perder el `manualInstruction` si el efecto tipado ya basta.

Una migración futura debe priorizar patrones repetidos, no personajes: anuncios públicos, reglas de votación/nominación, información inicial, concesión temporal de habilidades y eventos faltantes. Cada nueva primitiva se implementa una vez y se prueba con IDs inventados.
