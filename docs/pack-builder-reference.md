# Referencia de BloodScribe Creator: personajes y reglas de partida

Esta referencia enumera las opciones que ofrece el editor. Para aprender a construir un pack completo paso a paso, consulta el [`manual de autoría manual`](manual-autoria-packs.es.md). La referencia de bajo nivel del lenguaje de expresiones está en [`referencia-ast-declarativo.md`](referencia-ast-declarativo.md).

Las fuentes de verdad ejecutables dentro del código de BloodScribe son:

- `packages/core/src/types.ts`, para los contratos JSON.
- `packages/core/src/ruleSchema.ts`, para las opciones que muestra y valida el editor de mecánicas.
- `packages/core/src/votingRules.ts`, para las reglas de nominación y votación.

Los IDs son referencias estables. El runtime nunca decide un comportamiento por el nombre, el idioma, el ID del personaje o la colección del pack.

## 1. Qué se puede crear

### Personajes

Un personaje declara identidad, equipo, texto, presentación, fichas y cero o más programas en `rules.mechanics[]`. Cada programa usa las primitivas de la sección 4.

```json
{
  "id": "character:ejemplo:vigia",
  "copies": 3,
  "name": "Vigía",
  "teamId": "team:ejemplo:pueblo",
  "ability": "Cada noche descubres cuántos jugadores vivos son malos.",
  "rules": {
    "mechanics": [
      {
        "mechanicId": "mechanic:ejemplo:vigia:informacion:1",
        "tags": ["night-information"],
        "when": { "window": "night", "cadence": "each" },
        "input": { "kind": "none" },
        "effects": [
          {
            "type": "emitInformation",
            "value": { "type": "literal", "value": 2 },
            "presentation": { "kind": "number", "title": "Malos vivos" },
            "delivery": { "audience": { "type": "actor" } }
          }
        ],
        "usage": { "scope": "repeat", "consumeOn": "resolution" }
      }
    ]
  }
}
```

`copies` declara cuántas fichas físicas idénticas hay disponibles (entero positivo, `1` si se omite). Se conserva un único ID estable y el catálogo muestra `×N`; no se crean IDs progresivos como `aldeano-1`, `aldeano-2` ni se codifica la cantidad en el ID. `copies` es el máximo disponible, mientras que una obligación de usar varias fichas se declara aparte con `allowDuplicates` en el setup.

### Reglas generales

Una regla general vive en `gameRules[]`, no se reparte a un jugador y puede ejecutar los mismos programas declarativos que un personaje.

- `visibility: "base"`: infraestructura oculta que actúa en segundo plano.
- `visibility: "public"`: regla visible en el grimorio, el setup y el cuento.
- `gameRuleBindings[]`: enlaza una regla general al cuento con activación `automatic` o `setupChoice`.
- `requiredCharacterIds[]`: activa automáticamente la regla cuando todos esos personajes están en el cuento.

```json
{
  "entityKind": "gameRule",
  "id": "rule:ejemplo:anuncio-amanecer",
  "name": "Anuncio al amanecer",
  "categoryId": "rule-category:ejemplo:genericas",
  "ruleKind": "general",
  "visibility": "base",
  "mechanics": [
    {
      "mechanicId": "mechanic:ejemplo:anuncio-amanecer:1",
      "when": { "window": "dawn", "cadence": "each" },
      "input": { "kind": "none" },
      "effects": [{ "type": "recordAction", "actionId": "anuncio-amanecer" }],
      "usage": { "scope": "repeat", "consumeOn": "resolution" }
    }
  ]
}
```

### Sistema frente a Modificadores

`ruleKind` y `systemType` responden a preguntas distintas:

- `ruleKind` describe la mecánica ejecutable: `general`, `composition`, `voting`, `setup` o `gameEnd`.
- `systemType` describe dónde se presenta la regla. Si existe, la regla pertenece a Sistema; si se omite, pertenece a Modificadores.

Por ahora, solo Fabled, Loric, Maldiciones y Bendiciones se publican como Modificadores. Violinista, por ejemplo, conserva `ruleKind: "gameEnd"`, pero no declara `systemType`; por eso no se mezcla con los finales del Sistema.

El selector de Creator ofrece Modificador y los cinco tipos de Sistema. Al elegir un tipo de Sistema compone el ID jerárquico, oculta el campo de icono y muestra el SVG compartido:

| `systemType` | ID | SVG automático |
|---|---|---|
| `composition` | `rule:<propietario>:composition:<slug>` | Composición |
| `voting` | `rule:<propietario>:voting:<slug>` | Votación |
| `game-end` | `rule:<propietario>:game-end:<slug>` | Final de partida |
| `information` | `rule:<propietario>:information:<slug>` | Información |
| `other` | `rule:<propietario>:other:<slug>` | Genérico |

Un Modificador usa `rule:<propietario>:<slug>` y conserva el icono declarado por el contenido. Una regla de Sistema no incluye `icon` ni `category.icon`: la aplicación resuelve un único SVG autoritativo desde `systemType` y no admite overrides individuales.

Ejemplos válidos:

```json
[
  {
    "id": "rule:castronegro:game-end:final",
    "ruleKind": "gameEnd",
    "systemType": "game-end"
  },
  {
    "id": "rule:botc:fiddler",
    "ruleKind": "gameEnd",
    "icon": "<svg>...</svg>"
  },
  {
    "id": "rule:grimm:noche-sin-luna",
    "ruleKind": "general",
    "categoryId": "rule-category:grimm:curses",
    "icon": "<svg>...</svg>"
  }
]
```

BloodScribe instala siempre estas reglas, incluso con el catálogo de packs vacío:

- `rule:bloodscribe:composition:standard`
- `rule:bloodscribe:voting:standard`
- `rule:bloodscribe:game-end:standard`

Los packs las referencian desde `gameRuleBindings` o `votingRuleId`, pero no copian sus definiciones en `gameRules[]`. Pueden distribuir alternativas bajo su propio propietario, como `rule:grimm:composition:short`.

### Reglas de nominación y votación

Una regla de votación usa `ruleKind: "voting"` y guarda su contrato en `voting`. Cada cuento selecciona como máximo una mediante `votingRuleId`. Para crear una variante del mismo cuento con otras reglas se duplica el cuento y se cambia esa referencia.

Si el cuento no declara `votingRuleId`, usa la regla base del sistema. Si referencia una regla que ya no está instalada, la aplicación avisa y usa esa misma regla base. La partida guarda una copia completa de la regla elegida.

Una regla reutilizada por varios packs debe conservar exactamente el mismo ID canónico, aunque el namespace no coincida con el del pack que la incluye. Por ejemplo, varios packs pueden declarar y referenciar `rule:ejemplo:voting:simultanea`. Al cargarlos juntos, BloodScribe fusiona las copias por ID y muestra una sola regla asociada a todos sus libros. No generes un ID distinto por cuento o pack para una mecánica compartida. Este fragmento se centra en las referencias y omite los demás campos obligatorios de la regla y el cuento:

```json
{
  "gameRules": [
    {
      "entityKind": "gameRule",
      "id": "rule:ejemplo:voting:simultanea",
      "ruleKind": "voting",
      "systemType": "voting",
      "visibility": "public",
      "voting": { "version": 1 }
    }
  ],
  "tales": [
    {
      "id": "tale:otra-coleccion:mi-cuento",
      "votingRuleId": "rule:ejemplo:voting:simultanea"
    }
  ]
}
```

## 2. Opciones de nominación y votación

La definición siempre usa `version: 1` y separa nominación, candidatos, papeleta, resolución, momento y publicación. `pace`, `visibility` y `reveal` son ejes distintos: una votación puede ser simultánea y pública, pero mantenerse cerrada hasta que todos hayan respondido.

### 2.1 Nominación

| Campo | Opciones | Qué hace |
|---|---|---|
| `nomination.mode` | `none`, `public` | Desactiva la nominación o abre nominaciones públicas. |
| `nomineeLifeState` | `alive`, `dead`, `any` | Limita a quién se puede nominar. Solo existe con `mode: public`. |
| `allowSelfNomination` | `true`, `false` | Permite que alguien se nomine a sí mismo. |
| `maxPerNominatorPerDay` | entero positivo | Máximo de nominaciones iniciadas por persona y día. |
| `maxPerNomineePerDay` | entero positivo | Máximo de veces que una persona puede ser nominada por día. |

### 2.2 Candidatos

| Campo | Opciones | Qué hace |
|---|---|---|
| `candidates.source` | `nominated`, `eligiblePlayers` | Usa solo nominados o forma la lista directamente con jugadores elegibles. |
| `candidates.lifeState` | `alive`, `dead`, `any` | Limita quién puede aparecer en la papeleta. |

`source: nominated` exige nominaciones públicas. `approvalPerNominee` también exige una nominación y abre una votación separada para cada nominado.

### 2.3 Papeleta

`{"mode":"none"}` desactiva por completo la votación. En otro caso se declaran estos campos:

| Campo | Opciones | Qué hace |
|---|---|---|
| `shape` | `approvalPerNominee`, `singleChoice` | Aprobación sí/no de una nominación o elección de un candidato entre varios. |
| `enteredBy` | `storyteller`, `players` | Registra los votos el Narrador o cada jugador desde su móvil. |
| `pace` | `simultaneous`, `seatOrder` | Recoge todas las decisiones a la vez o sigue el orden de asientos. |
| `reveal` | `live`, `onClose` | Muestra cada voto al registrarse o solo al cerrar la ronda. |
| `visibility` | `public`, `secret` | Declara si las selecciones individuales son visibles o privadas. |
| `abstention` | `allowed`, `forbidden` | Permite o impide confirmar una papeleta sin candidato. |
| `order.start` | `afterNominee`, `nominee`, `nominator`, `storytellerSelected` | Punto de inicio cuando `pace` es `seatOrder`. |
| `order.direction` | `clockwise`, `counterclockwise` | Sentido del recorrido por asientos. |

`seatOrder` exige `order`. `approvalPerNominee` exige registro del Narrador, visibilidad pública, electorado estándar, apertura por nominación y resolución al anochecer.

### 2.4 Electorado y Últimos suspiros

Electorado estándar:

```json
{ "kind": "standard", "lastBreathsPerPlayer": 1 }
```

`lastBreathsPerPlayer` acepta cualquier entero seguro mayor o igual que cero:

- `0`: un jugador muerto no puede votar.
- `1`: dispone de un voto confirmado después de morir.
- `2` o más: dispone de esa reserva total durante toda la partida.

Un Último suspiro se consume al cerrar un voto válido con candidato mientras el jugador está muerto. Abstenerse, corregir antes del cierre, cancelar o emitir un voto inválido no consume. La reserva no se recarga al resucitar ni al morir de nuevo.

Electorado designado:

```json
{
  "kind": "designated",
  "role": "judge",
  "countsByPlayerCount": { "5": 2, "6": 2, "7": 3 },
  "identities": "secret",
  "peers": "hidden",
  "requireAliveAtResolution": true,
  "onAllDead": { "winner": "evil" }
}
```

El Narrador asigna exactamente el número de jueces indicado para el tamaño de mesa. Sus identidades y las de sus compañeros permanecen ocultas; un juez muerto no vota y, si todos mueren, gana el mal. Los jueces no usan Últimos suspiros.

### 2.5 Umbral, ganador y empate

| Campo | Opciones | Qué hace |
|---|---|---|
| `threshold.kind` | `none` | No exige mínimo; gana la mayor puntuación positiva. |
| `threshold.kind` | `fixed` + `votes` | Exige un número fijo de votos. |
| `threshold.kind` | `fraction` | Calcula una fracción sobre `livingPlayers`, `eligibleVoters` o `validBallots`. |
| `fraction.comparison` | `atLeast`, `moreThan` | Redondea al mínimo que alcanza o supera estrictamente la fracción. |
| `winner` | `highest` | Elige la mayor puntuación que alcanza el umbral. |
| `onTie.kind` | `spareAll`, `executeAll`, `storytellerChoice`, `runoff` | Indulta, ejecuta a todos los líderes, deja decidir al Narrador o abre segunda vuelta. |
| `onNoQualifier.kind` | `spareAll`, `storytellerChoice`, `runoff` | Resuelve el caso en que nadie alcanza el umbral. |
| `runoff.maxRounds` | entero positivo | Número máximo de rondas. |
| `runoff.exhausted` | `spareAll`, `storytellerChoice` | Resultado al agotar las rondas. |

Ejemplo de mayoría de vivos:

```json
{
  "kind": "fraction",
  "base": "livingPlayers",
  "numerator": 1,
  "denominator": 2,
  "comparison": "atLeast"
}
```

### 2.6 Momento y publicación

| Campo | Opciones | Qué hace |
|---|---|---|
| `timing.open` | `perNomination`, `day`, `preDawn` | Abre por nominación, durante el día o al final de la noche. |
| `timing.resolve` | `perNomination`, `dusk`, `preDawn` | Resuelve por nominación, al anochecer o antes del amanecer. |
| `disclosure.publicResult` | `resultOnly`, `totals` | Publica solo el resultado o también los totales por candidato. |

`enteredBy` también define el cierre. Con `storyteller`, cerrar resuelve y aplica el resultado inmediatamente. Con `players`, cerrar bloquea las papeletas y calcula el resultado solo para el Narrador: los móviles muestran que la votación ha terminado hasta que pulse **Comunicar resultados**. Entonces se aplican las ejecuciones, se sincroniza la ruleta de 12 segundos si alguien muere y se publica el desenlace según `disclosure.publicResult`.

`reveal` controla las papeletas, no el momento de publicar el desenlace. `live` permite ver una papeleta pública mientras la ronda sigue abierta; `onClose` la retiene hasta la comunicación. `visibility: "secret"` nunca publica quién votó a quién, tampoco después de comunicar. Las identidades de un electorado designado continúan siendo privadas.

Los callbacks de automatización siguen el desenlace real, no el mero cierre de la papeleta:

| Callback | Cuándo se emite |
|---|---|
| `execution` | Una vez por cada ejecución aplicada al comunicar. Incluye si la persona murió. |
| `death` | Empieza la ruleta porque al menos una ejecución causó una muerte. |
| `deathReveal` | Termina la ruleta y se publican las personas que realmente murieron. |
| `votingTie` | El empate queda sin resolver o requiere otra ronda. No emite también `noDeath`. |
| `noDeath` | Nadie alcanza el umbral o todas las personas elegidas sobreviven. |

Los payloads de votación incluyen ronda, causa y jugadores públicos, pero nunca papeletas secretas ni identidades del jurado.

### 2.7 Ejemplos completos

Votación pública simultánea, registrada por el Narrador:

```json
{
  "version": 1,
  "nomination": { "mode": "public", "nomineeLifeState": "any", "allowSelfNomination": false, "maxPerNominatorPerDay": 1, "maxPerNomineePerDay": 1 },
  "candidates": { "source": "nominated", "lifeState": "any" },
  "ballot": { "shape": "approvalPerNominee", "enteredBy": "storyteller", "pace": "simultaneous", "reveal": "live", "visibility": "public", "electorate": { "kind": "standard", "lastBreathsPerPlayer": 1 }, "abstention": "allowed" },
  "resolution": { "threshold": { "kind": "fraction", "base": "livingPlayers", "numerator": 1, "denominator": 2, "comparison": "atLeast" }, "winner": "highest", "onTie": { "kind": "spareAll" }, "onNoQualifier": { "kind": "spareAll" } },
  "timing": { "open": "perNomination", "resolve": "dusk" },
  "disclosure": { "publicResult": "totals" }
}
```

Elección secreta desde los móviles entre nominados:

```json
{
  "version": 1,
  "nomination": { "mode": "public", "nomineeLifeState": "alive", "allowSelfNomination": false, "maxPerNominatorPerDay": 1, "maxPerNomineePerDay": 1 },
  "candidates": { "source": "nominated", "lifeState": "alive" },
  "ballot": { "shape": "singleChoice", "enteredBy": "players", "pace": "simultaneous", "reveal": "onClose", "visibility": "secret", "electorate": { "kind": "standard", "lastBreathsPerPlayer": 0 }, "abstention": "allowed" },
  "resolution": { "threshold": { "kind": "none" }, "winner": "highest", "onTie": { "kind": "runoff", "maxRounds": 2, "exhausted": "spareAll" }, "onNoQualifier": { "kind": "spareAll" } },
  "timing": { "open": "day", "resolve": "dusk" },
  "disclosure": { "publicResult": "totals" }
}
```

Sin nominaciones ni votación:

```json
{
  "version": 1,
  "nomination": { "mode": "none" },
  "candidates": { "source": "eligiblePlayers", "lifeState": "alive" },
  "ballot": { "mode": "none" },
  "resolution": { "threshold": { "kind": "none" }, "winner": "highest", "onTie": { "kind": "spareAll" }, "onNoQualifier": { "kind": "spareAll" } },
  "timing": { "open": "day", "resolve": "dusk" },
  "disclosure": { "publicResult": "resultOnly" }
}
```

## 3. Forma de un programa mecánico

Personajes y reglas generales usan la misma estructura:

```json
{
  "mechanicId": "mechanic:ejemplo:regla:1",
  "tags": ["capacidad-semantica"],
  "when": { "window": "day", "cadence": "each" },
  "input": { "kind": "players", "min": 1, "max": 1 },
  "conditions": [],
  "effects": [{ "type": "recordAction", "actionId": "accion-ejemplo" }],
  "usage": { "scope": "game", "limit": { "type": "literal", "value": 1 }, "consumeOn": "resolution" },
  "policies": [],
  "presentation": { "label": "Acción de ejemplo" }
}
```

| Campo | Uso |
|---|---|
| `mechanicId` | ID estable para persistencia y diagnóstico. No selecciona comportamiento. |
| `tags` | Capacidades semánticas que otras reglas pueden consultar. |
| `when` | Momento, cadencia, evento disparador y demora. |
| `input` | Entrada que presenta Creator. |
| `conditions` | Expresiones que deben cumplirse. |
| `effects` | Operaciones ejecutadas en orden. |
| `usage` | Límite y momento de consumo. |
| `policies` | Excepciones tipadas de interacción con los asistentes. |
| `presentation` | Textos y controles; nunca se interpreta como lógica. |
| `requiresManualModeling` | Reconoce una parte aún no representada. No sustituye un efecto. |

### 3.1 Ventanas y eventos

Ventanas de `when.window`:

| Valor | Momento |
|---|---|
| `setup` | Preparación de la partida. |
| `firstNight` | Primera noche. |
| `night` | Noches posteriores o cualquier noche según `startsAt`. |
| `dawn` | Resolución previa al anuncio del amanecer. |
| `day` | Día abierto. |
| `voting` | Durante una votación. |
| `speech` | Conversación pública o privada. |
| `nomination` | Nominación. |
| `execution` | Ejecución. |
| `expulsion` | Expulsión de un personaje temporal. |
| `dusk` | Transición a la noche. |
| `mainEvilInfo` | Información inicial del malvado principal. |
| `gameEnd` | Comprobación del final. |
| `anyTime` | Sin ventana concreta. |

`cadence` puede ser `once` o `each`; `startsAt` fija el primer día o noche. `delay` usa `night`, `day` o `phase` y un `count` positivo.

Eventos de `when.trigger.event`: `characterEntry`, `abilityGrant`, `death`, `execution`, `expulsion`, `nomination`, `publicAction`, `mechanicUse`, `mechanicTargeted`, `tableAction`, `stateChange`, `restrictionChange`, `markerChange`, `storytellerSignal` y `storytellerExecution`. Este último procede de una nominación y ejecución tipadas del Narrador, no de una señal de texto.

Cada evento conserva `causedByEventId` cuando nace de otro y una copia de las identidades de sus participantes en ese instante. Las consultas históricas usan esa copia, por lo que un cambio posterior de personaje o alineamiento no reescribe lo sucedido.

### 3.2 Entradas

| `input.kind` | Opciones principales | Ejemplo |
|---|---|---|
| `none` | Sin entrada. | `{"kind":"none"}` |
| `players` | `min`, `max`, `allowRepeated`, `candidates`, `constraints`, `cardinality`. | `{"kind":"players","min":1,"max":2}` |
| `character` | `candidates`, `optional`. | `{"kind":"character","optional":true}` |
| `playerAndCharacter` | `player`, `characterCandidates`, `declarerCandidates`; con `recordAction.recordAs: "publicDeclaration"` registra declarante, objetivo y personaje declarado. | `{"kind":"playerAndCharacter","player":{"kind":"players","min":1,"max":1}}` |
| `participantResponses` | `participants`, `options`; cada opción declara valor, texto y efectos, con `allSameEffects` opcional. | `{"kind":"participantResponses","participants":{"kind":"players","min":1,"max":"all"},"options":[]}` |
| `text` | `format`: `statement`, `advice`, `wish`, `yesNo`. | `{"kind":"text","format":"wish"}` |
| `playerCharacterGuesses` | `max`. | `{"kind":"playerCharacterGuesses","max":3}` |
| `seatSwaps` | `minPairs`, `maxPairs`. | `{"kind":"seatSwaps","minPairs":1,"maxPairs":2}` |
| `vote` | `mode: cult`. | `{"kind":"vote","mode":"cult"}` |
| `contest` | Sin opciones adicionales. | `{"kind":"contest"}` |

### 3.3 Uso

| Campo | Opciones | Uso |
|---|---|---|
| `scope` | `repeat`, `day`, `night`, `game`, `actor`, `target`, `trigger` | Ámbito que comparte el límite. |
| `limit` | Cualquier `ValueExpr` numérico no negativo | Número de usos; si falta, no hay contador. |
| `consumeOn` | `attempt`, `resolution`, `success` | Consume al intentar, resolver o acertar. |
| `spentReminder` | ID de ficha | Materializa visualmente el uso gastado. |

Para ámbitos compuestos se omite `scope` y se usa `keyBy`, por ejemplo `{"keyBy":["day","actor"],"limit":{"type":"literal","value":1}}` para una vez por actor cada día. Las dimensiones admitidas son `day`, `night`, `actor`, `target` y `trigger`.

Las duraciones tipadas admiten `permanent`, `untilWindow`, `whileTargetAlive`, `whileCondition` y `untilEvent`. Esta última conserva un patrón de evento y sus bindings, por ejemplo `{"type":"untilEvent","event":"nomination"}`.

### 3.4 Expresiones y condiciones

Las condiciones, objetivos, límites y valores usan `ValueExpr`. Nodos disponibles:

| `type` | Uso | Ejemplo mínimo |
|---|---|---|
| `literal` | Valor JSON fijo. | `{"type":"literal","value":1}` |
| `binding` | `actor`, `selected`, `source`, `nominator`, `nominee`, `eventSubject`, `effectTarget`. | `{"type":"binding","binding":"actor"}` |
| `game` | `phase`, `dayNumber`, `nightNumber`. | `{"type":"game","property":"dayNumber"}` |
| `setup` | Dato calculado de la preparación. | `{"type":"setup","property":"mainEvilBluffCharacterIds"}` |
| `fact` | Hecho de la resolución actual. | `{"type":"fact","fact":"outcome"}` |
| `storytellerDecision` | Booleano decidido por el Narrador. | `{"type":"storytellerDecision","decision":"isFair"}` |
| `decisionValue` | Valor de una decisión tipada ya contestada. | `{"type":"decisionValue","decision":"fate"}` |
| `all`, `any`, `not` | Composición booleana. | `{"type":"not","condition":{"type":"literal","value":false}}` |
| `query` | Consulta jugadores, personajes, eventos o fichas. | `{"type":"query","from":{"entity":"players"},"aggregate":{"type":"count"}}` |
| `compare` | `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `includes`. | `{"type":"compare","left":{"type":"literal","value":2},"operator":"gte","right":1}` |
| `array`, `object` | Construye valores compuestos. | `{"type":"array","items":[]}` |
| `concat` | Concatena listas. | `{"type":"concat","values":[]}` |
| `set` | `union`, `intersection`, `difference`. | `{"type":"set","operation":"union","values":[]}` |
| `math` | `add`, `subtract`, `min`, `max`. | `{"type":"math","operation":"add","values":[{"type":"literal","value":1},{"type":"literal","value":2}]}` |
| `unique` | Elimina duplicados. | `{"type":"unique","value":{"type":"literal","value":["a","a"]}}` |
| `length` | Longitud de lista o texto. | `{"type":"length","value":{"type":"literal","value":[1,2]}}` |
| `take` | Primeros `count` elementos. | `{"type":"take","value":{"type":"literal","value":[1,2]},"count":1}` |
| `at` | Elemento por índice. | `{"type":"at","value":{"type":"literal","value":["a"]},"index":0}` |
| `if` | Alternativa condicional. | `{"type":"if","condition":{"type":"literal","value":true},"then":{"type":"literal","value":1},"else":{"type":"literal","value":0}}` |
| `inputValue` | Lee la entrada tipada. | `{"type":"inputValue","valueType":"playerId"}` |

Para consultas, predicados, relaciones de mesa, agregados y proyecciones, consulta la [referencia AST](referencia-ast-declarativo.md#4-consultas).

## 4. Primitivas de efecto

Los efectos comparten, cuando corresponde, `when`, `delay`, `targets`, `affectedBy`, `duration`, `optional`, `reminder`, `reminderTokens` y `spentReminder`. La tabla enumera las opciones propias; Creator conserva campos avanzados admitidos por el esquema.

| `effects[].type` | Qué hace | Opciones propias principales | Ejemplo mínimo |
|---|---|---|---|
| `death` | Mata objetivos y registra la atribución. | `protection`, bypass, resolución especial, requisitos y fichas. | `{"type":"death","targets":{"type":"binding","binding":"selected"}}` |
| `resurrect` | Devuelve objetivos a la vida. | Sin opciones propias. | `{"type":"resurrect","targets":{"type":"binding","binding":"selected"}}` |
| `execute` | Ejecuta y registra si el objetivo muere. | `dies`, `requiresSourceAlive`, `targetReminder`. | `{"type":"execute","targets":{"type":"binding","binding":"selected"},"dies":true}` |
| `setPlayerState` | Activa o desactiva un estado. | `state`, `active`, `exclusive`. | `{"type":"setPlayerState","state":"silenced","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `setPlayerRelation` | Activa o desactiva una relación persistente entre la fuente y cada objetivo. | `kind`, `active`, `duration`, `ownership`, `markerId`. | `{"type":"setPlayerRelation","kind":"protected","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `applyMarker` | Añade o retira una ficha. | `kind`, `id`, `active`, `exclusive`, `ownership`. | `{"type":"applyMarker","kind":"reminder","id":"watched","active":true,"targets":{"type":"binding","binding":"selected"}}` |
| `moveMarker` | Mueve atómicamente una ficha existente. | `kind`, `id`, `from`; `targets` es el destino. | `{"type":"moveMarker","kind":"reminder","id":"crown","from":{"type":"binding","binding":"actor"},"targets":{"type":"binding","binding":"selected"}}` |
| `adjustCounter` | Ajusta un contador y puede proyectar estado o disparar un umbral. | `counter`, `delta`, `scope`, `bounds`, `projection`, `stateProjection`, `threshold`, `onThreshold`. | `{"type":"adjustCounter","counter":"charges","delta":1,"targets":{"type":"binding","binding":"actor"}}` |
| `changeAlignment` | Cambia el alineamiento. | `alignment`, `notifyPlayer`, perfiles y límites de objetivo. | `{"type":"changeAlignment","alignment":"evil","targets":{"type":"binding","binding":"selected"}}` |
| `changeCharacter` | Sustituye identidad real o mostrada. | Personaje/equipo nuevo, preservación de alineamiento, límites y consecuencias. | `{"type":"changeCharacter","newCharacter":"character:ejemplo:nuevo","targets":{"type":"binding","binding":"selected"}}` |
| `grantAbility` | Concede o retira una habilidad declarada. | `abilityCharacterId`, `active`, `owner`, `controller`, `ownership`. | `{"type":"grantAbility","abilityCharacterId":"character:ejemplo:fuente","active":true,"ownership":"sourceAbility","targets":{"type":"binding","binding":"selected"}}` |
| `triggerAbility` | Activa una mecánica declarada del objetivo. | `mechanicTag`. | `{"type":"triggerAbility","mechanicTag":"ability-tag","targets":{"type":"binding","binding":"selected"}}` |
| `swapSeats` | Intercambia asientos seleccionados. | Sin opciones propias. | `{"type":"swapSeats","targets":{"type":"binding","binding":"selected"}}` |
| `swapCharacters` | Intercambia identidades entre jugadores. | `actor`, `resultingState`, `resultingStateDuration`, `swapsCharactersAndAlignments`. | `{"type":"swapCharacters","targets":{"type":"binding","binding":"selected"}}` |
| `swapTargets` | Intercambia objetivos ya resueltos. | Sin opciones propias. | `{"type":"swapTargets","targets":{"type":"binding","binding":"selected"}}` |
| `emitInformation` | Calcula y entrega información tipada. | `value`, `presentation`, `delivery`, `transform`. | `{"type":"emitInformation","value":{"type":"literal","value":true},"presentation":{"kind":"boolean","title":"Resultado"},"delivery":{"audience":{"type":"actor"}}}` |
| `prepareInformation` | Prepara información verdadera y alternativa. | `candidates`, `modes`, `zeroWhen`, `characterChoice`, `reminders`, `shownIdentityOverride`. | `{"type":"prepareInformation","candidates":{"type":"array","items":[]},"modes":["pair"],"characterChoice":{"source":"truthfulPlayer","identityMode":"real"},"reminders":{"truthful":"Verdadero","alternative":"Alternativa"}}` |
| `resolveGameEnd` | Declara o programa ganadores. | `mode`, `winner`, `reason`, `precedence`; `override` desplaza resultados `standard`, y `delay` se usa si es `queued`. | `{"type":"resolveGameEnd","mode":"immediate","winner":{"type":"fixed","team":"good"},"reason":"Condición cumplida"}` |
| `blockGameEnd` | Impide cerrar una victoria mientras la fuente esté activa. | `winner`: `good`, `evil` o `any`; `reason`, `activation.lifeState`. | `{"type":"blockGameEnd","winner":"good","reason":"La victoria está bloqueada."}` |
| `transformGameEnd` | Transforma el conjunto final de ganadores después de resolver precedencia y bloqueos. | `operation: "invertWinners"`, `reason`, `activation.lifeState`. | `{"type":"transformGameEnd","operation":"invertWinners","reason":"Se invierten ganadores y perdedores."}` |
| `startActionSequence` | Mantiene una secuencia obligatoria hasta que una expresión indique que terminó. | Actualmente: nominación, muerte del nominado, siguiente actor nominado y respaldo del Narrador. | `{"type":"startActionSequence","action":"nomination","onAction":"killNominee","nextActor":"nominee","repeatUntil":{"type":"literal","value":false}}` |
| `performTableAction` | Registra una acción física tipada antes de resolver sus consecuencias. | `action`: `devour`, `feed`, `abandon`, `occupy` o `building`; `targets`, `consequences`. | `{"type":"performTableAction","action":"devour","targets":{"type":"binding","binding":"selected"},"consequences":[]}` |
| `interceptEvent` | Cancela, redirige o sustituye un evento coincidente. | `event`, `bindings`, `match`, `reaction`, `priority`, `consumption`, `scope`, `appliesWhenProtectionBypassed`. | `{"type":"interceptEvent","event":"death","reaction":{"type":"cancel"}}` |
| `disableAbility` | Anula total o parcialmente una habilidad. | `blocks`, `consumeOnPass`, `informationMayBeFalse`. | `{"type":"disableAbility","blocks":"ability","targets":{"type":"binding","binding":"selected"}}` |
| `restrict` | Limita acciones disponibles. | `actions`, `restrictions`, `relation`, `requiresSourceAlive`, `informationMayBeFalse`. | `{"type":"restrict","actions":["vote"],"targets":{"type":"binding","binding":"selected"}}` |
| `registerAs` | Cambia cómo se registra una identidad. | `affects`, `alignment`, `roles`, `characterIds`, `lifeState`, `mode`, `registersAs`, `teamIds`, `worksWhenDead`. | `{"type":"registerAs","alignment":"evil","targets":{"type":"binding","binding":"actor"}}` |
| `modifyTargets` | Aumenta o reduce el máximo de objetivos. | `delta`, `sourceProfile`, `targetMechanicTags`. | `{"type":"modifyTargets","delta":1,"targetMechanicTags":["night-information"]}` |
| `modifyVote` | Cambia la política de una votación. | `purposes`, `threshold`, pesos, electorado, créditos y validez del recuento. | `{"type":"modifyVote","purposes":["standard"],"threshold":0}` |
| `modifySetup` | Modifica cantidades o asignaciones de preparación. | `operations`: ajustar, elegir, reemplazar, fijar, duplicar, exigir, retirar o configurar reparto. | `{"type":"modifySetup","operations":[{"type":"adjustBucket","bucket":"outsider","delta":1}]}` |
| `restrictSetupCombination` | Limita cuántos personajes de una combinación pueden estar en juego. | `characterIds`, `maximum`. | `{"type":"restrictSetupCombination","characterIds":["character:ejemplo:uno","character:ejemplo:dos"],"maximum":1}` |
| `modifyInformation` | Transforma información antes de entregarla. | `audience`, `mustBeFalse`. | `{"type":"modifyInformation","mustBeFalse":true}` |
| `modifyStartingKnowledge` | Activa o desactiva pasos tipados de información inicial. | `steps`, `active`. | `{"type":"modifyStartingKnowledge","steps":["evilTeamRecognition"],"active":false}` |
| `modifyNomination` | Cambia el resultado de una nominación. | `countsAsExecution`, `voteDelta`, `requiresActorAbstention`, consumo y fichas. | `{"type":"modifyNomination","voteDelta":1}` |
| `recordAction` | Registra una acción y su resultado para consultas posteriores. | `actionId`, `outcome`, éxito/fallo, restricciones y metadatos del objetivo. | `{"type":"recordAction","actionId":"claim-example"}` |
| `storytellerDecision` | Solicita una decisión humana cerrada. | `decision`, `options`; los textos viven en `presentation.decisionPrompts`. | `{"type":"storytellerDecision","decision":"fate","options":["dies","survives"]}` |
| `manualCheckpoint` | Pausa la resolución hasta que el Narrador elige un resultado cerrado y persistido. | `reason`, `prompt`, `outcomes`, `blocking`, `optional`; cada resultado declara sus efectos. | `{"type":"manualCheckpoint","reason":"storytellerJudgment","prompt":"¿Se cumplió?","outcomes":[{"id":"yes","label":"Sí","effects":[]}],"blocking":true}` |
| `manualInstruction` | Muestra un paso que el motor no puede ejecutar. | `instruction`, `reason`, `publicKnown`, `ruleStepActivation` y ayudas. | `{"type":"manualInstruction","instruction":"Resuelve el acuerdo de la mesa.","reason":"Decisión social"}` |

### Información

`emitInformation.presentation.kind` admite `boolean`, `number`, `text`, `player`, `players`, `character`, `characters`, `identity`, `direction`, `distance`, `grimoire` y `structured`.

La audiencia puede ser `actor`, `selected`, `eventSubject`, `public`, `storyteller` o `players` con una expresión. `delivery.timing` admite `immediate`, `dawn`, `privateDay` y `nextRecipientWake`; `mode` puede ser `shared` o `perRecipient`.

### Preparación

Las operaciones de `modifySetup` son `adjustBucket`, `chooseAdjustments`, `replaceBucket`, `setBucketCount`, `allowDuplicates`, `requireCharacter`, `requireBucket`, `removeCharacters` y `configureDeal`. Los recuentos pueden ser literales, una elección, el número de jugadores, una mayoría, la suma de categorías, copias de la fuente o el resto disponible.

## 5. Políticas

Las políticas conectan una mecánica con los asistentes sin crear comportamiento por personaje.

| `policies[].type` | Qué hace | Opciones |
|---|---|---|
| `recordTargetSelectionWhenDisabled` | Conserva la selección aunque el actor esté deshabilitado. | Ninguna. |
| `presentStartingInformationAsShownIdentity` | Presenta transitoriamente la información inicial de la identidad mostrada. | Ninguna. |
| `relayShownSelection` | Registra y transmite la selección hecha mediante la identidad mostrada. | Ninguna. |
| `assignSelectedCharacter` | Asigna el personaje seleccionado. | Ninguna. |
| `grantSelectedAbility` | Concede la habilidad seleccionada. | `target`, `abilityAlignment`, `duration`, `requireInPlay`. |
| `overrideChooserAlignment` | Fuerza el alineamiento usado por el selector. | `alignment`. |
| `continueAfterTargetReaction` | Continúa tras la reacción del objetivo. | Ninguna. |
| `suppressStandaloneNightStep` | Evita un paso nocturno independiente. | Ninguna. |
| `ignoreActorAbilityRestriction` | Resuelve una mecánica externa aunque la habilidad del actor esté anulada. | Ninguna. |
| `allowDeadActorWithPendingAction` | Mantiene una acción pendiente tras la muerte. | `actionId`. |
| `requireRecordedTargetAlignment` | Exige el alineamiento capturado por una acción previa. | `actionId`, `alignment`. |
| `limitSelectionByRecordedActionCount` | Limita la selección por acciones registradas. | `actionId`, `period`: `currentDay` o `game`. |

## 6. Presentación y modelado manual

`presentation` configura labels, títulos, descripciones, guías nocturnas, controles, resultados y acciones públicas. Ningún texto decide mecánicas.

- Usa `storytellerDecision` cuando el Narrador elige entre opciones cerradas que el runtime puede resolver.
- Usa `manualCheckpoint` cuando esa elección debe quedar pendiente, bloquear el flujo y persistir su resultado antes de aplicar efectos.
- Usa `manualInstruction` cuando la propia operación todavía no existe o depende de una interacción física/social.
- Usa `requiresManualModeling: true` solo para reconocer que el programa conserva una parte sin modelar; no convierte una descripción en lógica.

## 7. Comprobación antes de exportar

La matriz exhaustiva y las recetas completas por intención están disponibles en la [guía de autoría en español](pack-builder-character-authoring.es.md) y la [English authoring guide](pack-builder-character-authoring.en.md).

- Cada comportamiento está en `voting`, `when`, `input`, `conditions`, `effects`, `usage` o `policies`, nunca solo en prosa.
- Los IDs permanecen estables entre idiomas y no seleccionan handlers.
- Las reglas de Sistema usan cuatro segmentos y un `systemType` coherente; los Modificadores usan tres segmentos y omiten `systemType`.
- Las tres reglas estándar solo se referencian: el pack exportado no duplica sus datos ni sus iconos.
- Al importar packs anteriores, BloodScribe migra los alias conocidos y sus referencias. Un pack nuevo con un ID incoherente se rechaza.
- Toda decisión asistida tiene opciones y textos de presentación cerrados.
- Las reglas generales se enlazan mediante `gameRuleBindings`; la regla de votación mediante `votingRuleId`.
- El cuento duplicado conserva sus referencias hasta que el autor las cambie.
- Creator valida el JSON con el mismo importador que usa la instalación.
