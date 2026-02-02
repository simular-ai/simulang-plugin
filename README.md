# Simulang Plugin

A Claude Code plugin for automating desktop tasks using Simulang, a JavaScript-based DSL for [Simular Pro](https://simular.ai).

## Features

- Control keyboard and mouse actions
- Interact with desktop applications
- Extract content from screens using perception
- Automate web browsing and form filling
- Read/write files and Google Sheets

## Installation

1. Add this marketplace to your Claude Code configuration:

```bash
claude plugin marketplace add https://github.com/simular-ai/simulang-plugin.git
```

2. Open Claude and install this skill with the command:

```bash
/plugin install simulang
```

3. Depending on the scope of installation you choose (e.g. user scope), you may need to restart Claude

## Usage

Invoke the simulang skill by typing `/simulang` followed by your task description:

```
/simulang book a flight from SF to NYC tomorrow
```

Or simply describe a desktop automation task and Claude will use Simulang to generate the appropriate script.

## Quick Reference

| Category | Key Functions |
|----------|---------------|
| **Application** | `open({app, url})` |
| **Keyboard** | `type({text, withReturn})`, `press({key, cmd, ctrl, shift})` |
| **Mouse** | `click({at, mode, spatialRelation})`, `scroll({direction})` |
| **Perception** | `pageContent()`, `ask({prompt, context})` |
| **Wait** | `wait({waitTime})`, `waitForConcepts({concepts})` |

## Resources

- [Simulang Reference](skills/simulang/reference.md) - Complete API documentation
- [Examples](skills/simulang/examples.md) - Practical workflow examples
- [Official Docs](https://docs.simular.ai/simular-pro/simulang)
