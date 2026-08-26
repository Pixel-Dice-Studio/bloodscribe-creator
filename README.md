# BloodScribe Creator plugin

[Español](#español) · [English](#english)

## Español

Este repositorio distribuye el plugin `bloodscribe-creator`. Creator reúne el editor web, las guías y las herramientas MCP que permiten a tu propia IA crear personajes con recetas y validación determinista. Nunca llama ni permite acceder a la IA interna de BloodScribe.

El endpoint es `https://bloodscribe.app/mcp`. Durante la instalación o el primer uso, según el cliente, BloodScribe se abrirá en el navegador para que inicies sesión y autorices la conexión; no necesitas copiar un bearer token.

### Documentación

- [Configurar el MCP en ChatGPT / Codex](#instalar-en-chatgpt--codex)
- [Configurar el MCP en Claude Code](#instalar-en-claude-code)
- [Configurar manualmente el MCP y el skill](#configuración-manual)
- [Crear manualmente un pack completo](./docs/manual-autoria-packs.es.md)
- [Descargar un pack completo de demostración](./docs/ejemplo-pack-manual.es.bloodscribe.json)
- [Crear personajes: matriz de mecánicas, recetas y ejemplos](./docs/pack-builder-character-authoring.es.md)
- [Usar BloodScribe Creator y componer mecánicas](./docs/pack-builder-reference.md)
- [Consultar expresiones, relaciones, filtros y agregados](./docs/referencia-ast-declarativo.md)
- [Ver personajes reales de ejemplo](#ejemplos-reales)

### Instalar en ChatGPT / Codex

```sh
codex plugin marketplace add Pixel-Dice-Studio/bloodscribe-creator --ref main
codex plugin add bloodscribe-creator@bloodscribe-creator
```

Abre una tarea nueva y pide crear un personaje. Codex iniciará la autorización OAuth cuando necesite el MCP.

### Instalar en Claude Code

```sh
claude plugin marketplace add Pixel-Dice-Studio/bloodscribe-creator
claude plugin install bloodscribe-creator@bloodscribe-creator
```

Abre una conversación nueva y pide crear un personaje. Claude Code iniciará la misma autorización OAuth.

### Configuración manual

Si tu cliente admite MCP pero no plugins, instala o copia el skill [`create-bloodscribe-character`](./plugins/bloodscribe-creator/skills/create-bloodscribe-character/SKILL.md) y configura `https://bloodscribe.app/mcp`. OAuth es la opción recomendada. Solo los clientes sin OAuth necesitan generar una clave en **BloodScribe Creator → Perfil → MCP de Creator**; configúrala únicamente en el almacén seguro de credenciales del cliente, nunca en un chat ni en el repositorio.

### Ejemplos reales

Los [packs oficiales de Grimm](https://github.com/Pixel-Dice-Studio/bloodscribe-official-packs) son referencias editoriales. No son dependencias del plugin ni se cargan automáticamente.

## English

This repository distributes the `bloodscribe-creator` plugin. Creator brings together the web editor, authoring guides, and MCP tools that guide your own AI through public recipes and deterministic validation. It never calls or exposes BloodScribe's internal AI.

The endpoint is `https://bloodscribe.app/mcp`. During installation or first use, depending on the client, BloodScribe opens in the browser so you can sign in and authorize the connection; no bearer token needs to be copied.

### Documentation

- [Configure MCP in ChatGPT / Codex](#install-in-chatgpt--codex)
- [Configure MCP in Claude Code](#install-in-claude-code)
- [Configure MCP and the skill manually](#manual-setup)
- [Author a complete pack manually (Spanish)](./docs/manual-autoria-packs.es.md)
- [Download the complete demonstration pack](./docs/ejemplo-pack-manual.es.bloodscribe.json)
- [Create characters: mechanic matrix, recipes, and examples](./docs/pack-builder-character-authoring.en.md)
- [Use BloodScribe Creator and compose mechanics (Spanish)](./docs/pack-builder-reference.md)
- [Reference expressions, relations, filters, and aggregates (Spanish)](./docs/referencia-ast-declarativo.md)
- [Browse real character examples](#real-examples)

### Install in ChatGPT / Codex

```sh
codex plugin marketplace add Pixel-Dice-Studio/bloodscribe-creator --ref main
codex plugin add bloodscribe-creator@bloodscribe-creator
```

Start a new task and ask it to create a character. Codex starts OAuth when it first needs the MCP.

### Install in Claude Code

```sh
claude plugin marketplace add Pixel-Dice-Studio/bloodscribe-creator
claude plugin install bloodscribe-creator@bloodscribe-creator
```

Start a new conversation and ask it to create a character. Claude Code starts the same OAuth flow.

### Manual setup

If your client supports MCP but not plugins, install or copy the [`create-bloodscribe-character`](./plugins/bloodscribe-creator/skills/create-bloodscribe-character/SKILL.md) skill and configure `https://bloodscribe.app/mcp`. OAuth is recommended. Only clients without OAuth need a key from **BloodScribe Creator → Profile → Creator MCP**; configure it only in the client's secure credential store, never in chat or in the repository.

### Real examples

The [official Grimm packs](https://github.com/Pixel-Dice-Studio/bloodscribe-official-packs) are editorial references. They are not plugin dependencies and are never loaded automatically.
