# Manual de autoría manual de packs de BloodScribe

Esta guía explica cómo escribir a mano un pack `bloodscribe-3.1`: desde la raíz del JSON hasta equipos, personajes, reglas, composiciones, votación, cuentos y programas mecánicos. Está pensada para entender cómo se combinan las primitivas, no solo para copiar campos aislados.

Puedes consultar o descargar el [pack completo de demostración](ejemplo-pack-manual.es.bloodscribe.json). Ese archivo usa únicamente IDs y contenido inventados y se valida automáticamente con el mismo importador que usa BloodScribe.

## Índice

1. [Modelo mental](#1-modelo-mental)
2. [Orden recomendado de construcción](#2-orden-recomendado-de-construcción)
3. [Raíz del pack](#3-raíz-del-pack)
4. [IDs, claves y referencias](#4-ids-claves-y-referencias)
5. [Colecciones y Libros](#5-colecciones-y-libros)
6. [Equipos](#6-equipos)
7. [Personajes](#7-personajes)
8. [Cómo componer una mecánica](#8-cómo-componer-una-mecánica)
9. [Expresiones, consultas y relaciones](#9-expresiones-consultas-y-relaciones)
10. [Fichas, estados, duración y procedencia](#10-fichas-estados-duración-y-procedencia)
11. [Reglas de partida](#11-reglas-de-partida)
12. [Composición del reparto](#12-composición-del-reparto)
13. [Nominación y votación](#13-nominación-y-votación)
14. [Cuentos y activación de reglas](#14-cuentos-y-activación-de-reglas)
15. [Ejemplo razonado: vecinos registrados como malvados](#15-ejemplo-razonado-vecinos-registrados-como-malvados)
16. [Validar, probar y publicar](#16-validar-probar-y-publicar)
17. [Errores habituales](#17-errores-habituales)
18. [Referencias exhaustivas](#18-referencias-exhaustivas)

## 1. Modelo mental

Un pack no es código. Es un documento JSON declarativo que contiene entidades y programas mecánicos.

```text
Pack
├── Libros
│   └── Colección embebida
├── Equipos
├── Personajes
│   └── rules.mechanics[]
├── Reglas de partida
│   ├── composición
│   ├── votación
│   ├── setup / final
│   └── reglas generales
│       └── rules.mechanics[]
└── Cuentos
    ├── personajes incluidos
    ├── copia de sus equipos
    └── bindings de reglas
```

Las responsabilidades están separadas:

| Pieza | Decide |
|---|---|
| Pack | Transporte, idioma, versión y autoría. |
| Colección | Universo editorial compartido. |
| Libro | Agrupación estable de cuentos y entidades. |
| Equipo | Vocabulario, color y perfil mecánico predeterminado. |
| Personaje | Identidad, texto, fichas y habilidades. |
| Regla | Comportamiento global no asignado a un jugador. |
| Composición | Cuántos personajes de cada perfil forman la partida. |
| Votación | Cómo se nomina, vota, resuelve y comunica. |
| Cuento | Qué personajes y reglas se usan juntos. |
| Mecánica | Cuándo ocurre algo, sobre quién y con qué resultado. |

Cuatro reglas protegen el contrato:

1. Los nombres, textos e IDs nunca seleccionan comportamiento.
2. Toda lógica ejecutable vive en campos tipados: `when`, `input`, `conditions`, `effects`, `usage` y `policies`.
3. `presentation`, `info`, `cues` e `interactions` explican o muestran; no ejecutan.
4. Si falta una primitiva, se declara `requiresManualModeling: true` o se usa una operación humana tipada. No se inventa un `type` opaco.

## 2. Orden recomendado de construcción

Construir en este orden evita referencias rotas:

1. Elige un namespace estable, por ejemplo `mi-juego`.
2. Declara los metadatos raíz y la versión.
3. Declara los equipos y sus perfiles predeterminados.
4. Crea los personajes que usan esos equipos.
5. Añade exactamente una regla de composición para cada Cuento.
6. Añade una regla de votación si no quieres usar el fallback estándar.
7. Añade reglas globales, de setup o de final.
8. Crea el Cuento y enlaza personajes y reglas.
9. Crea el Libro y enumera todas sus entidades.
10. Importa el JSON, corrige errores y prueba una partida antes de publicarlo.

El [pack de demostración](ejemplo-pack-manual.es.bloodscribe.json) sigue este orden aunque JSON muestre el Libro antes que las entidades.

## 3. Raíz del pack

La forma canónica es:

```json
{
  "schemaVersion": "bloodscribe-3.1",
  "id": "mi-juego:pack-principal",
  "key": "pack-principal",
  "version": "1.0.0",
  "locale": "es",
  "name": "Mi pack",
  "description": "Descripción pública.",
  "author": "Nombre del autor",
  "sourceUrl": "https://github.com/organizacion/mi-pack",
  "legal": {
    "license": "Licencia del contenido",
    "disclaimer": "Aviso necesario"
  },
  "links": [],
  "books": [],
  "teams": [],
  "characters": [],
  "gameRules": [],
  "tales": []
}
```

`schemaVersion` es el único dato de compatibilidad con el motor; no se añade un campo `engineVersion`. Su minor aumenta cuando se incorporan primitivas, sintaxis o campos opcionales compatibles, y su major cuando se retira o cambia el significado de una declaración o hace falta transformar la estructura existente. Las correcciones internas que no cambian el JSON no alteran esta versión. BloodScribe migra formatos históricos compatibles al importar, pero todo pack nuevo o guardado usa siempre la versión canónica actual.

El campo `version` es independiente: identifica la publicación del contenido del pack, no la compatibilidad del motor.

| Campo | Regla |
|---|---|
| `schemaVersion` | Siempre `bloodscribe-3.1`. |
| `id` | Identidad estable del pack. No se traduce. |
| `key` | Minúsculas, números y guiones simples. |
| `version` | Increméntala al cambiar comportamiento o contenido publicado. |
| `locale` | Un idioma por archivo. Todos los textos visibles usan ese idioma. |
| `legal` | Licencia y aviso del contenido. |
| `books` | Puede estar vacío en packs auxiliares sin Cuentos. |
| `teams` | Fuente raíz para personajes sueltos; los Cuentos copian los que usan. |
| `characters`, `gameRules`, `tales` | Entidades declarativas del pack. |

Un pack parcial puede contener solo personajes, solo reglas o cualquier combinación. Si contiene un Cuento, debe incluir también su Libro.

## 4. IDs, claves y referencias

Los IDs son referencias persistentes. No contienen lógica y no cambian al traducir el pack.

| Entidad | Forma recomendada |
|---|---|
| Colección | `collection:<namespace>` |
| Libro | `book:<namespace>:<slug>` |
| Cuento | `tale:<namespace>:<slug>` |
| Equipo | `team:<namespace>:<slug>` |
| Personaje | `character:<namespace>:<slug>` |
| Regla modificadora | `rule:<namespace>:<slug>` |
| Regla de Sistema | `rule:<namespace>:<systemType>:<slug>` |
| Mecánica | `mechanic:<namespace>:<entidad>:<capacidad>:<número>` |

Si una regla declara `systemType`, su ID debe incluir ese segmento:

```json
{
  "id": "rule:mi-juego:composition:reparto-base",
  "systemType": "composition"
}
```

Los valores actuales de `systemType` son `composition`, `voting`, `game-end`, `information` y `other`. Una regla modificadora omite `systemType` y conserva un ID corto como `rule:mi-juego:bendicion-del-rio`.

Al renombrar un ID hay que actualizar todas sus referencias:

- listas del Libro;
- `characterIds` y `gameRuleBindings` del Cuento;
- `votingRuleId`;
- `requiredGameRuleIds`;
- `interactions[].characterId`;
- expresiones o efectos que apunten expresamente a una entidad.

## 5. Colecciones y Libros

La Colección se incluye dentro de cada Libro. El Libro enumera sus Cuentos, personajes, reglas y equipos.

```json
{
  "id": "book:mi-juego:libro-uno",
  "name": "Libro uno",
  "description": "Primer conjunto de cuentos.",
  "collection": {
    "id": "collection:mi-juego",
    "name": "Mi juego",
    "description": "Universo compartido."
  },
  "origin": "custom",
  "author": "Nombre del autor",
  "taleIds": ["tale:mi-juego:primer-cuento"],
  "characterIds": ["character:mi-juego:vigia"],
  "gameRuleIds": ["rule:mi-juego:composition:reparto-base"],
  "teamIds": ["team:mi-juego:habitantes"]
}
```

`ContentPack` es el archivo de transporte; el Libro es la agrupación editorial. No uses `edition` para sustituir al Libro.

## 6. Equipos

Un equipo proporciona términos, presentación y valores mecánicos predeterminados.

```json
{
  "teamId": "team:mi-juego:habitantes",
  "terms": {
    "singular": "Habitante",
    "plural": "Habitantes"
  },
  "description": "Núcleo del bando bueno.",
  "defaults": {
    "role": "core",
    "allegiance": {
      "default": "good",
      "allowed": ["good"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast"]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  },
  "colors": {
    "text": {
      "onLight": "#155E75",
      "onDark": "#A5F3FC"
    },
    "surface": "#2563EB",
    "border": "#1D4ED8",
    "reminderSurface": "#1E3A8A",
    "gradient": {
      "from": "#2563EB",
      "to": "#4F46E5"
    }
  }
}
```

### Perfil mecánico

| Campo | Valores |
|---|---|
| `role` | `core`, `support`, `independent` |
| `allegiance.default` | `good`, `evil`, `neutral` |
| `allegiance.allowed` | Alineamientos a los que puede cambiar legítimamente. |
| `entryMode.default` | `cast` o `temporary` |
| `entryMode.allowed` | Modos de entrada permitidos. |
| `victory.type` | `withCurrentAllegiance`, `withSide` o `personal` |

Antes de escribir la habilidad, decide estos ejes en este orden:

1. Si entra en el reparto (`cast`), como personaje temporal (`temporary`) o de ambas formas.
2. Si ocupa un rol `core`, `support` o `independent` en la composición.
3. Si su alineamiento es `good`, `evil` o `neutral` y a cuáles puede cambiar.
4. Si gana con su alineamiento actual, con un bando fijo o mediante una condición personal.

`teamId`, el nombre y el texto de habilidad no resuelven ninguna de esas decisiones. La participación se declara exclusivamente mediante `entryMode`.

### Perfiles completos habituales

Personaje regular de un bando:

```json
{
  "gameplay": {
    "role": "core",
    "allegiance": {
      "default": "good",
      "allowed": ["good"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast"]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  }
}
```

Personaje temporal: queda fuera del reparto base y del conteo estándar de vivos, puede incorporarse durante la partida y es elegible para expulsión. Su alineamiento se asigna de forma secreta al entrar. Si es malvado, el motor crea como primer paso de su primera noche aplicable una revelación de todos los malvados principales vivos; no incluye ayudantes ni faroles. Estas reglas son automáticas y no se repiten en `mechanics`.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": ["neutral", "good", "evil"]
    },
    "entryMode": {
      "default": "temporary",
      "allowed": ["temporary"]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  }
}
```

Neutral dentro del reparto con victoria personal: ocupa una plaza neutral, cuenta entre los vivos regulares, muere normalmente y no gana automáticamente con bueno ni malvado.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": ["neutral"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast"]
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

La condición `personal` se comprueba al resolver un final. Si cumplirla debe terminar la partida por sí mismo, declara además una mecánica `resolveGameEnd`.

Neutral dentro del reparto que gana siempre con un bando fijo:

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": ["neutral"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast"]
    },
    "victory": {
      "type": "withSide",
      "side": "good"
    }
  }
}
```

Entrada flexible: el mismo personaje puede comportarse como parte normal del reparto o como personaje temporal. El comportamiento efectivo depende del modo elegido para ese jugador.

```json
{
  "gameplay": {
    "role": "independent",
    "allegiance": {
      "default": "neutral",
      "allowed": ["neutral", "good", "evil"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast", "temporary"]
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

Si “Universal” es solo una categoría editorial, usa un `teamId` universal pero conserva `role`, `allegiance`, `entryMode` y `victory` del comportamiento real. No conviertas la etiqueta universal en una rama mecánica.

La composición estándar se deriva de `role` y `allegiance`, no del nombre ni del ID del equipo:

| Alineamiento y rol | Eje de composición |
|---|---|
| `good + core` | `good` |
| `good + support` | `goodSupport` |
| `evil + support` | `evilSupport` |
| `evil + core` | `evil` |
| `neutral` o `independent` | `neutral` o entrada independiente, según el perfil |

Un personaje puede sobrescribir parte de `gameplay`, pero los valores comunes deben vivir en el equipo para evitar repetición.

Los colores son `#RRGGBB` opacos. No uses variables CSS, nombres de color ni alfa.

En textos visibles puedes resolver el vocabulario del equipo con `{{team:mi-juego:habitantes}}` y su plural con `{{team:mi-juego:habitantes|plural}}`.

## 7. Personajes

Un personaje completo separa identidad, perfil, documentación y ejecución:

```json
{
  "id": "character:mi-juego:vigia",
  "copies": 1,
  "teamId": "team:mi-juego:habitantes",
  "gameplay": {
    "role": "core",
    "allegiance": {
      "default": "good",
      "allowed": ["good"]
    },
    "entryMode": {
      "default": "cast",
      "allowed": ["cast"]
    },
    "victory": {
      "type": "withCurrentAllegiance"
    }
  },
  "info": {
    "name": "Vigía",
    "ability": "Cada noche descubres un dato.",
    "howToPlay": "Explica cómo tomar decisiones con la habilidad.",
    "howToRun": "Explica cómo dirigirla.",
    "interactions": []
  },
  "rules": {
    "setup": false,
    "wake": {
      "first": true,
      "other": true
    },
    "reminderTokens": [],
    "cues": [],
    "mechanics": []
  }
}
```

### Identidad y copias

- `copies` es el número de fichas físicas idénticas disponibles. Si falta, vale `1`.
- No crees IDs como `vigia-1`, `vigia-2` para representar copias.
- `teamId` debe existir en `teams` o en otro pack instalado del que dependa deliberadamente el contenido.
- `requiredGameRuleIds` enlaza reglas que siempre deben acompañar al personaje.

### Texto

- `info.ability` es la frase breve visible en la ficha.
- `howToPlay` explica decisiones del jugador.
- `howToRun` explica pasos del Narrador.
- `interactions[]` documenta relaciones, pero no ejecuta jinxes ni efectos.
- Los textos admiten títulos, negrita, cursiva, listas y citas Markdown limitadas.

### Noche y cues

`rules.wake` ayuda a presentar el orden nocturno. `cues[]` muestra instrucciones contextuales. Ninguno sustituye a `rules.mechanics[]`.

```json
{
  "when": {
    "window": "night",
    "occurrence": {
      "type": "relative",
      "value": "first"
    }
  },
  "audience": "storyteller",
  "body": "Despierta al Vigía y entrega su información."
}
```

### Modelado manual

`manualDirection: true` indica que la dirección requiere intervención. Dentro de una mecánica, `requiresManualModeling: true` reconoce una parte no representada. Para pasos humanos que sí están tipados, usa efectos como `storytellerDecision` o `manualInstruction`.

## 8. Cómo componer una mecánica

Personajes y reglas generales usan exactamente la misma forma:

```json
{
  "mechanicId": "mechanic:mi-juego:vigia:capacidad:1",
  "tags": ["night-information"],
  "when": {
    "window": "night",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [],
  "usage": {
    "scope": "repeat",
    "consumeOn": "resolution"
  },
  "policies": [],
  "presentation": {}
}
```

Piensa en una mecánica como una frase estructurada:

> **Cuándo** se ofrece, **qué entrada** necesita, **si se cumplen** sus condiciones, **ejecuta** estos efectos, **con este límite**, **estas políticas** y **esta presentación**.

### 8.1 `mechanicId` y `tags`

`mechanicId` sirve para persistencia y diagnóstico. `tags` describe capacidades que otras reglas pueden descubrir. Ninguno debe usarse como un nombre secreto de función.

### 8.2 `when`: momento y disparador

`when.window` sitúa la capacidad en `setup`, `firstNight`, `night`, `dawn`, `day`, `voting`, `speech`, `nomination`, `execution`, `expulsion`, `dusk`, `mainEvilInfo`, `gameEnd` o `anyTime`.

`cadence` es `once` o `each`. `startsAt` fija el primer día o noche. `trigger` convierte la mecánica en una reacción a un evento:

```json
{
  "window": "nomination",
  "cadence": "each",
  "trigger": {
    "type": "event",
    "event": "nomination"
  }
}
```

Usa `activeWhen` para decidir si el paso debe existir. Usa `conditions` cuando el paso existe pero puede resolverse sin efecto.

### 8.3 `input`: selección o respuesta

| `kind` | Uso habitual |
|---|---|
| `none` | No hay elección. |
| `players` | Uno o varios jugadores con filtros y cardinalidad. |
| `character` | Elegir una identidad del catálogo. |
| `playerAndCharacter` | Declaración que combina jugador y personaje. |
| `participantResponses` | Varias personas responden entre opciones. |
| `text` | Declaración, consejo, deseo o sí/no. |
| `playerCharacterGuesses` | Conjeturas tipadas. |
| `seatSwaps` | Parejas de asientos. |
| `vote` | Voto especial tipado. |
| `contest` | Resolución de contienda. |

Ejemplo de un objetivo vivo que no sea el actor:

```json
{
  "kind": "players",
  "min": 1,
  "max": 1
}
```

Los filtros de vivos, el propio actor y otras restricciones se expresan mediante `candidates` o `constraints` tipados. No confíes únicamente en el selector visual: cualquier restricción importante debe quedar en el `input` o en una condición evaluable.

### 8.4 `conditions`: permiso para resolver

Todas las condiciones deben cumplirse. Para alternativas, negación o grupos anidados usa `all`, `any` y `not`.

```json
{
  "type": "sourceAffected",
  "states": ["poisoned", "drunk", "registration"],
  "value": false
}
```

Las condiciones pueden leer al actor, la selección, participantes del evento, identidades reales o registradas, estados, recordatorios, acciones grabadas y consultas históricas.

### 8.5 `effects`: operaciones en orden

Los efectos se ejecutan en el orden del array. Grupos principales:

| Resultado | Primitivas principales |
|---|---|
| Vida y ejecución | `death`, `resurrect`, `execute` |
| Estado y fichas | `setPlayerState`, `applyMarker`, `moveMarker`, `adjustCounter` |
| Identidad y posición | `changeAlignment`, `changeCharacter`, `grantAbility`, `swapSeats`, `swapCharacters`, `swapTargets` |
| Información | `prepareInformation`, `emitInformation`, `modifyInformation` |
| Intercepción | `interceptEvent`, `disableAbility`, `restrict`, `modifyTargets`, `modifyVote`, `modifySetup`, `modifyNomination` |
| Registro y final | `recordAction`, `resolveGameEnd`, `registerAs` |
| Intervención humana | `storytellerDecision`, `manualInstruction` |

Un efecto puede tener su propio `when`, `targets`, `duration`, `delay` o `optional`. Esto permite que una misma mecánica produzca varios resultados relacionados sin duplicar la selección.

### 8.6 `usage`: quién comparte el límite

Un ámbito simple usa `scope`:

```json
{
  "scope": "game",
  "limit": {
    "type": "literal",
    "value": 1
  },
  "consumeOn": "success"
}
```

Los ámbitos disponibles son `repeat`, `day`, `night`, `game`, `actor`, `target` y `trigger`.

Un límite compuesto usa `keyBy` y omite `scope`:

```json
{
  "keyBy": ["day", "actor"],
  "limit": {
    "type": "literal",
    "value": 1
  },
  "consumeOn": "resolution"
}
```

`consumeOn` distingue:

- `attempt`: se gasta aunque no se resuelva;
- `resolution`: se gasta al completar la resolución;
- `success`: solo se gasta si produce éxito.

### 8.7 `policies`: variaciones tipadas

Las políticas modifican cómo interactúa la mecánica con los asistentes sin inventar otro efecto. Ejemplos: permitir un actor muerto con una acción pendiente, exigir una alineación grabada, presentar información inicial como identidad mostrada o suprimir un paso nocturno independiente.

Consulta la [matriz generada](pack-builder-character-authoring.es.md#matriz-exhaustiva) antes de añadir una política: el ID y los campos permitidos deben existir allí.

### 8.8 `presentation`: interfaz, no lógica

`presentation` contiene labels, títulos, descripciones, controles y resultados visibles. Puede declarar un `actionId` estable para una acción pública, pero el comportamiento continúa en `effects`.

```json
{
  "label": "Usar la habilidad",
  "title": "Elige un jugador",
  "description": "La selección será pública.",
  "action": {
    "actionId": "mi-juego-accion-publica"
  }
}
```

Traducir un pack cambia esos textos, no `actionId`, `mechanicId` ni la estructura de efectos.

## 9. Expresiones, consultas y relaciones

`ValueExpr` es el lenguaje que compone valores, objetivos, límites y condiciones. Los nodos más usados son:

| Nodo | Pregunta que responde |
|---|---|
| `literal` | ¿Cuál es el valor fijo? |
| `binding` | ¿Quién es actor, seleccionado, nominador, nominado o sujeto del evento? |
| `game` | ¿Qué día, noche o fase es? |
| `inputValue` | ¿Qué eligió o respondió el usuario? |
| `query` | ¿Qué jugadores, personajes, eventos o fichas cumplen un filtro? |
| `compare` | ¿Cómo se compara un valor con otro? |
| `all`, `any`, `not` | ¿Cómo se combinan condiciones? |
| `math`, `length`, `unique`, `take`, `at` | ¿Cómo se transforma el resultado? |
| `if` | ¿Qué valor corresponde a cada rama? |

Una consulta sigue este recorrido:

```text
from → relation/where → project → aggregate
```

Ejemplo: contar jugadores vivos con identidad registrada malvada:

```json
{
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
        "type": "identity",
        "identityMode": "registered",
        "facet": "allegiance",
        "values": ["evil"]
      }
    ]
  },
  "aggregate": {
    "type": "count"
  }
}
```

Elige conscientemente el modo de identidad:

- `real`: identidad actual real;
- `initial`: identidad inicial;
- `shown`: lo mostrado al jugador;
- `registered`: aplica alteraciones de registro antes de filtrar o proyectar.

La [referencia del AST](referencia-ast-declarativo.md) enumera relaciones de mesa, filtros, proyecciones, agregados, campos de eventos y bindings.

## 10. Fichas, estados, duración y procedencia

Declara primero el stock visible en `rules.reminderTokens`:

```json
{
  "id": "watched",
  "kind": "reminder",
  "value": "Watched",
  "label": "Vigilado",
  "duration": "manual"
}
```

`id`, `kind`, `value` y `label` son obligatorios. `kind` es `state` o `reminder`.

`count` es opcional y significa **escasez**: declara cuántos ejemplares físicos existen, y cuando se agotan no se puede colocar otro. Sin `count` la ficha no tiene reserva: el inventario ofrece siempre una copia libre y el panel manual la muestra como «sin límite». Declárala solo cuando la regla diga «como máximo N sobre la mesa»; un tope por jugador se declara en `bounds` del contador, no aquí.

Después crea, retira o mueve la ficha mediante efectos:

```json
{
  "type": "applyMarker",
  "kind": "reminder",
  "id": "watched",
  "active": true,
  "targets": {
    "type": "binding",
    "binding": "selected"
  },
  "duration": {
    "type": "untilEvent",
    "event": "nomination",
    "bindings": {
      "nominee": "effectTarget"
    }
  }
}
```

Una duración tipada puede ser `permanent`, `untilWindow`, `whileTargetAlive`, `whileCondition` o `untilEvent`. No uses una frase libre como sustituto.

`moveMarker` mueve atómicamente una ficha existente y conserva su procedencia, propiedad y duración:

```json
{
  "type": "moveMarker",
  "kind": "reminder",
  "id": "watched",
  "from": {
    "type": "binding",
    "binding": "actor"
  },
  "targets": {
    "type": "binding",
    "binding": "selected"
  }
}
```

No simules una transferencia con “retirar y volver a crear” si necesitas conservar la procedencia. Las consultas de fichas pueden leer quién la colocó, quién es su propietario y qué evento la originó cuando esos datos forman parte del contrato.

Para contadores, `adjustCounter` conserva el número como fuente de verdad y puede proyectarlo como estado o fichas visibles.

## 11. Reglas de partida

Una Regla usa el mismo programa mecánico que un personaje, pero no tiene portador jugador.

```json
{
  "entityKind": "gameRule",
  "id": "rule:mi-juego:other:anuncio",
  "visibility": "public",
  "ruleKind": "general",
  "systemType": "other",
  "name": "Anuncio al amanecer",
  "info": {
    "name": "Anuncio al amanecer",
    "ability": "El Narrador realiza un anuncio al amanecer."
  },
  "categoryId": "rule-category:common:generic",
  "category": {
    "id": "rule-category:common:generic",
    "terms": {
      "singular": "Regla genérica",
      "plural": "Reglas genéricas"
    }
  },
  "bookIds": ["book:mi-juego:libro-uno"],
  "rules": {
    "setup": false,
    "wake": {
      "first": false,
      "other": false
    },
    "cues": [],
    "mechanics": []
  }
}
```

### `ruleKind` frente a `systemType`

| Campo | Significado |
|---|---|
| `ruleKind` | Contrato ejecutable: `general`, `composition`, `voting`, `setup` o `gameEnd`. |
| `systemType` | Ubicación visual en Sistema: `composition`, `voting`, `game-end`, `information` u `other`. |
| sin `systemType` | La regla se presenta como Modificador. |

`visibility: "base"` oculta la regla en superficies públicas pero conserva su ejecución. `visibility: "public"` la muestra y permite activación `automatic` o `setupChoice` desde el Cuento.

`categoryId` agrupa reglas para selección y presentación. No es un selector de comportamiento. Una categoría puede declarar `selection.maxActive` para limitar cuántas reglas de ese grupo están activas.

Las reglas `setup` y `gameEnd` siguen siendo reglas tipadas; no escribas su lógica en el texto `ability`.

### Reglas de preparación

Una regla `setup` usa mecánicas con `when.window: "setup"` y efectos como `modifySetup`, `changeCharacter`, `changeAlignment`, `applyMarker` o una decisión tipada del Narrador. El Cuento la enlaza como cualquier otra regla global.

### Reglas de final

Una regla `gameEnd` evalúa condiciones en `when.window: "gameEnd"` y declara el resultado con `resolveGameEnd`:

```json
{
  "mechanicId": "mechanic:mi-juego:final:condicion:1",
  "tags": ["game-end"],
  "when": {
    "window": "gameEnd",
    "cadence": "each"
  },
  "input": {
    "kind": "none"
  },
  "conditions": [],
  "effects": [
    {
      "type": "resolveGameEnd",
      "mode": "immediate",
      "winner": {
        "type": "fixed",
        "team": "good"
      },
      "reason": "Se cumplió la condición declarada."
    }
  ],
  "usage": {
    "scope": "repeat"
  },
  "policies": []
}
```

La condición real debe estar en `conditions` o en expresiones del efecto. `reason` solo explica el resultado. Usa `systemType: "game-end"` para un final del Sistema; omítelo si la regla debe presentarse como Modificador.

## 12. Composición del reparto

Cada Cuento necesita exactamente una regla `composition`, automática e incondicional.

```json
{
  "entityKind": "gameRule",
  "id": "rule:mi-juego:composition:reparto-base",
  "visibility": "base",
  "ruleKind": "composition",
  "systemType": "composition",
  "name": "Reparto base",
  "categoryId": "rule-category:mi-juego:composicion",
  "category": {
    "id": "rule-category:mi-juego:composicion",
    "terms": {
      "singular": "Composición",
      "plural": "Composiciones"
    }
  },
  "composition": {
    "enforcement": "exact",
    "playerCounts": {
      "5": {
        "good": 2,
        "goodSupport": 1,
        "neutral": 0,
        "evilSupport": 1,
        "evil": 1
      }
    },
    "rosterMinimums": {
      "good": 2,
      "goodSupport": 1,
      "neutral": 0,
      "evilSupport": 1,
      "evil": 1
    },
    "maxExtraPlayers": 0,
    "inPlayMinimums": {
      "evil": 1
    },
    "inPlayMaximums": {
      "evil": 1
    }
  }
}
```

Reglas de validación:

- cada clave de `playerCounts` es un entero positivo;
- los cinco ejes son obligatorios y usan enteros no negativos;
- cada fila suma exactamente el número de jugadores de su clave;
- `rosterMinimums` declara los cinco ejes;
- `maxExtraPlayers` es un entero no negativo;
- un mínimo nunca supera su máximo;
- `exact` bloquea diferencias y `recommended` las convierte en avisos.

`rosterMinimums` comprueba que el catálogo del Cuento tenga suficientes copias. `inPlayMinimums` e `inPlayMaximums` limitan el reparto de una partida concreta.

## 13. Nominación y votación

Una regla `voting` controla seis capas:

1. `nomination`: quién puede nominar y ser nominado.
2. `candidates`: de dónde sale la papeleta.
3. `ballot`: forma, entrada, ritmo, visibilidad y electorado.
4. `resolution.threshold`: qué cantidad permite optar al resultado.
5. `resolution.winner`: cómo se elige y resuelven empates.
6. `timing` y `disclosure`: cuándo termina y qué se publica.

El [pack completo de demostración](ejemplo-pack-manual.es.bloodscribe.json) incluye la regla estándar íntegra. Su forma resumida es:

```json
{
  "entityKind": "gameRule",
  "id": "rule:mi-juego:voting:votacion-publica",
  "visibility": "public",
  "ruleKind": "voting",
  "systemType": "voting",
  "categoryId": "rule-category:common:voting-system",
  "voting": {
    "version": 1,
    "nomination": {},
    "candidates": {},
    "ballot": {},
    "resolution": {},
    "timing": {},
    "disclosure": {}
  }
}
```

El Cuento la selecciona con `votingRuleId`, no mediante un binding. Si falta o la regla instalada desaparece, BloodScribe usa la votación pública estándar y muestra un aviso.

La [referencia del BloodScribe Creator](pack-builder-reference.md#2-opciones-de-nominación-y-votación) enumera todos los valores y compatibilidades de votación.

## 14. Cuentos y activación de reglas

El Cuento selecciona el contenido jugable:

```json
{
  "id": "tale:mi-juego:primer-cuento",
  "bookId": "book:mi-juego:libro-uno",
  "category": "tale",
  "author": "Nombre del autor",
  "text": {
    "name": "Primer cuento",
    "tagline": "Una frase breve.",
    "description": "Descripción pública."
  },
  "teams": [],
  "characterIds": [
    "character:mi-juego:vigia"
  ],
  "gameRuleBindings": [
    {
      "ruleId": "rule:mi-juego:composition:reparto-base",
      "activation": "automatic"
    },
    {
      "ruleId": "rule:mi-juego:other:anuncio",
      "activation": "setupChoice"
    }
  ],
  "votingRuleId": "rule:mi-juego:voting:votacion-publica",
  "presentation": {
    "theme": "verdigris",
    "backdrop": "astrolabe"
  }
}
```

Reglas de enlace:

- debe existir exactamente un binding de composición;
- la composición es `automatic` y no declara `activeWhen`;
- una regla `base` solo puede ser automática;
- una regla pública puede ser `automatic` o `setupChoice`;
- `activeWhen` habilita una regla según una expresión declarativa;
- `votingRuleId` apunta a una regla `voting`;
- `characterIds` solo contiene personajes jugables del Cuento;
- `teams` copia las definiciones necesarias para que el snapshot de partida sea autocontenido.

Temas disponibles: `ink`, `purple`, `oxblood`, `verdigris`, `midnight` y `leather`. Fondos: `none`, `astrolabe`, `orrery`, `rose`, `filigree`, `baroque`, `thorns`, `cobweb`, `ribcage`, `candelabra`, `bramble`, `sigil`, `runes`, `blood` y `plain`.

## 15. Ejemplo razonado: vecinos registrados como malvados

Petición:

> Cada noche, descubre si al menos uno de tus dos vecinos vivos más cercanos se registra como malvado.

Descomposición:

| Pregunta | Decisión |
|---|---|
| ¿Cuándo? | `when.window: night`, cada noche. |
| ¿Necesita elección? | No: `input.kind: none`. |
| ¿Desde dónde parte? | De `players`. |
| ¿Qué relación de mesa? | `nearestMatching`, una persona por dirección. |
| ¿Se saltan muertos? | Sí, mediante `where: alive`. |
| ¿Qué identidad consulta? | `registered`, faceta `allegiance`. |
| ¿Qué resultado necesita? | Colección de alineamientos que incluya `evil`. |
| ¿Cómo se entrega? | Booleano al actor mediante `emitInformation`. |

```json
{
  "mechanicId": "mechanic:mi-juego:lector-vecinos:informacion:1",
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
  "usage": {
    "scope": "repeat"
  },
  "policies": []
}
```

La mecánica no contiene el nombre del personaje, del equipo malvado ni una rama especial del motor. Otro personaje puede reutilizarla cambiando textos e IDs sin cambiar su comportamiento.

## 16. Validar, probar y publicar

### Validación local en BloodScribe

1. Abre el BloodScribe Creator.
2. Ve a **Publicación**.
3. Abre el validador e importa o pega el JSON.
4. Corrige todos los errores bloqueantes.
5. Revisa los avisos de mecánicas manuales, catálogo insuficiente y referencias externas.
6. Instala el pack localmente.
7. Crea una partida con cada Cuento y recorre setup, noche, día, nominación, votación y final cuando corresponda.
8. Descarga el `.bloodscribe.json` final.

### Publicación por autodiscovery

1. Crea un repositorio público de GitHub que no esté archivado ni sea un fork.
2. Añade el topic exacto `bloodscribe`.
3. Sube cada archivo terminado en `.bloodscribe.json` a la raíz de la rama predeterminada.
4. Incrementa `version` antes de sustituir una versión publicada cuyo comportamiento o contenido haya cambiado.

No hacen falta scripts remotos ni código ejecutable. BloodScribe descarga y valida el JSON.

## 17. Errores habituales

| Error | Corrección |
|---|---|
| Enlazar documentación o código privado | Publica la guía y los ejemplos junto al pack. |
| Usar texto o ID como selector | Mueve la lógica a una primitiva tipada. |
| Inventar `effects[].type` | Usa la matriz admitida o declara modelado manual. |
| Poner `systemType` en un ID corto | Usa `rule:<namespace>:<systemType>:<slug>`. |
| Traducir IDs entre archivos ES/EN | Conserva IDs y traduce solo textos. |
| Declarar equipo solo dentro del Cuento | Publícalo también en `teams` raíz. |
| Omitir la copia de equipos del Cuento | Incluye las definiciones necesarias para su snapshot. |
| Crear una composición dentro del Cuento | Declárala como Regla y enlázala automáticamente. |
| Enlazar dos composiciones | Cada Cuento admite exactamente una. |
| Usar un binding para votación | Usa `votingRuleId`. |
| Confundir `wake` o `cues` con ejecución | Añade la mecánica en `rules.mechanics[]`. |
| Usar una duración textual | Declara un objeto de duración tipado. |
| Recrear una ficha al transferirla | Usa `moveMarker` para conservar procedencia y propiedad. |
| Limitar “una vez por actor al día” con un solo scope | Usa `usage.keyBy: ["day", "actor"]`. |
| Ocultar una carencia tras `manualInstruction` | Explica qué paso realiza el Narrador y marca la cobertura manual. |
| Publicar sin probar el JSON descargado | Instala exactamente el mismo archivo antes de publicarlo. |

## 18. Referencias exhaustivas

Este manual enseña el proceso y la composición. Para consultar cada opción admitida:

- [Pack completo de demostración](ejemplo-pack-manual.es.bloodscribe.json): archivo importable y validado.
- [Referencia del BloodScribe Creator](pack-builder-reference.md): campos de personajes, reglas, votación, efectos y políticas.
- [Matriz y recetas mecánicas](pack-builder-character-authoring.es.md): cada primitiva, subopción, campo, estado de soporte y ejemplo.
- [Referencia del AST declarativo](referencia-ast-declarativo.md): consultas, relaciones, filtros, proyecciones, agregados y expresiones.
- [English mechanic authoring guide](pack-builder-character-authoring.en.md): matriz y recetas en inglés.

Cuando una intención no aparece como soportada en la matriz, no intentes aproximarla con texto, labels o IDs. Rediseña la habilidad o documenta un paso manual explícito.
