---
name: simulang
description: Automate desktop tasks using Simulang, a JavaScript-based DSL for Simular Pro. Use when the user asks about desktop automation, UI scripting, controlling applications, keyboard/mouse automation, or writing Simular Pro scripts.
---

# Simulang (Simular Pro Action Language)

Simulang is a JavaScript-based DSL for **Simular Pro** that controls desktop environments through natural language-like functions for keyboard, mouse, perception, and application control.

## Quick Reference

| Category | Key Functions |
|----------|---------------|
| **Application** | `open({app, url})` |
| **Keyboard** | `type({text, withReturn})`, `press({key, cmd, ctrl, shift})` |
| **Mouse** | `click({at, mode, spatialRelation})`, `scroll({direction})` |
| **Perception** | `pageContent()`, `ask({prompt, context})`, `conceptsExist({concepts})` |
| **Wait** | `wait({waitTime})`, `waitForConcepts({concepts})` |
| **User** | `respond({message, requireConfirm})` |
| **Files** | `readFile({path})`, `writeToFile({text, path})` |
| **Google Sheets** | `getGoogleSheetCellValue({cell})`, `setGoogleSheetCellValue({cell, value})` |

## Click Modes

```javascript
// Default: text-based grounding
click({at: "sign in button"})

// Vision: for images/icons without text labels
click({at: "logo icon", mode: "vision"})

// Text + Screenshot: combines both methods
click({at: "Introduction in Simular Browser section", mode: "textAndScreenshot"})

// Spatial relation: disambiguate similar elements
click({at: "close button", spatialRelation: "containedIn", anchorConcept: "dialog"})
```

**Spatial options:** `closest`, `furthest`, `above`, `below`, `left`, `right`, `contains`, `containedIn`

## Common Pattern: Extract & Process

```javascript
function main() {
    open({url: "https://example.com"});
    wait({waitTime: 3});
    
    var content = pageContent();
    var result = ask({
        prompt: "Extract all items. Return as JSON array.",
        context: content
    });
    
    return JSON.parse(result);
}
```

## Best Practices

1. **Wait for loads:** Use `wait({waitTime: 2-3})` after navigation
2. **Be specific:** Use role+value in descriptions ("sign in button" not "button")
3. **Use `ask()` for extraction:** Pair with `pageContent()` for structured data
4. **Handle errors:** Use `respond({message, requireConfirm: true})` for confirmations

## Resources

- [reference.md](reference.md) - Complete API documentation
- [examples.md](examples.md) - Practical workflow examples
- [Official docs](https://docs.simular.ai/simular-pro/simulang)
