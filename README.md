# BloodScribe Pack Builder plugin

## Español

Este repositorio distribuye el plugin `bloodscribe-pack-builder`. El plugin guía a tu propia IA para crear personajes con las recetas y la validación determinista del MCP público. Nunca llama ni permite acceder a la IA interna de BloodScribe.

El endpoint es `https://bloodscribe.app/mcp`. La primera vez que la IA lo use, BloodScribe se abrirá en el navegador para que inicies sesión y autorices la conexión; no necesitas copiar un bearer token.

### Instalar en ChatGPT / Codex

```sh
codex plugin marketplace add Pixel-Dice-Studio/bloodscribe-pack-builder --ref main
codex plugin add bloodscribe-pack-builder@bloodscribe-pack-builder
```

Abre una tarea nueva y pide crear un personaje. Codex iniciará la autorización OAuth cuando necesite el MCP.

### Instalar en Claude Code

```sh
claude plugin marketplace add Pixel-Dice-Studio/bloodscribe-pack-builder
claude plugin install bloodscribe-pack-builder@bloodscribe-pack-builder
```

Abre una conversación nueva y pide crear un personaje. Claude Code iniciará la misma autorización OAuth.

### Configuración manual

Si tu cliente admite MCP pero no plugins, instala o copia el skill [`create-bloodscribe-character`](./plugins/bloodscribe-pack-builder/skills/create-bloodscribe-character/SKILL.md) y configura `https://bloodscribe.app/mcp`. OAuth es la opción recomendada. Solo los clientes sin OAuth necesitan generar una clave en **Perfil → Pack Builder → Clave MCP manual**; configúrala únicamente en el almacén seguro de credenciales del cliente, nunca en un chat ni en el repositorio.

### Ejemplos reales

Los [packs oficiales de Grimm](https://github.com/Pixel-Dice-Studio/bloodscribe-official-packs) son referencias editoriales. No son dependencias del plugin ni se cargan automáticamente.

## English

This repository distributes the `bloodscribe-pack-builder` plugin. The plugin guides your own AI through public MCP recipes and deterministic validation. It never calls or exposes BloodScribe's internal AI.

The endpoint is `https://bloodscribe.app/mcp`. The first time your AI uses it, BloodScribe opens in the browser so you can sign in and authorize the connection; no bearer token needs to be copied.

### Install in ChatGPT / Codex

```sh
codex plugin marketplace add Pixel-Dice-Studio/bloodscribe-pack-builder --ref main
codex plugin add bloodscribe-pack-builder@bloodscribe-pack-builder
```

Start a new task and ask it to create a character. Codex starts OAuth when it first needs the MCP.

### Install in Claude Code

```sh
claude plugin marketplace add Pixel-Dice-Studio/bloodscribe-pack-builder
claude plugin install bloodscribe-pack-builder@bloodscribe-pack-builder
```

Start a new conversation and ask it to create a character. Claude Code starts the same OAuth flow.

### Manual setup

If your client supports MCP but not plugins, install or copy the [`create-bloodscribe-character`](./plugins/bloodscribe-pack-builder/skills/create-bloodscribe-character/SKILL.md) skill and configure `https://bloodscribe.app/mcp`. OAuth is recommended. Only clients without OAuth need a key from **Profile → Pack Builder → Manual MCP key**; configure it only in the client's secure credential store, never in chat or in the repository.

### Real examples

The [official Grimm packs](https://github.com/Pixel-Dice-Studio/bloodscribe-official-packs) are editorial references. They are not plugin dependencies and are never loaded automatically.
