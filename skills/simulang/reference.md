# Simulang API Reference

Complete API documentation for Simulang. See [official docs](https://docs.simular.ai/simular-pro/simulang).

---

## General

### act

Completes a task specified in natural language.

```javascript
act({task: "Find all events of 100 museums in San Francisco", maxSteps: 50})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `task` | String | An arbitrary task |
| `maxSteps` | Int | Stop when this number of primitive actions have been executed |

---

## Application

### open

Open or switch to an application or URL.

```javascript
open({app: "Google Chrome"})
open({url: "https://simular.ai"})
open({app: "Zoom"})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `app` | String | Application name (e.g., "Google Chrome", "iMessage") |
| `url` | String | URL address of a web page |
| `waitForLoadComplete` | Bool | Wait for app/URL to finish loading. Default: true |
| `waitTime` | Int | Seconds to wait after opening, in addition to load wait |

---

## Keyboard Control

### type

Type text using the keyboard.

```javascript
type({text: "John Smith"})
type({text: "https://google.com", withReturn: true})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | String | Text to type |
| `withReturn` | Bool | Press return/enter after typing |
| `waitTime` | Int | Seconds to wait after typing |
| `waitForLoadComplete` | Bool | Wait for page to load after typing |

### press / shortcut

Perform keyboard shortcuts.

```javascript
press({key: "a", cmd: true})           // Cmd+A (select all)
press({key: "c", cmd: true})           // Cmd+C (copy)
press({key: "t", cmd: true, shift: true})  // Cmd+Shift+T
press({key: "enter"})
press({key: "tab"})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | String | Key to press |
| `cmd` | Bool | Hold Command modifier |
| `ctrl` | Bool | Hold Control modifier |
| `option` | Bool | Hold Option modifier |
| `shift` | Bool | Hold Shift modifier |
| `waitTime` | Int | Seconds to wait after action |

---

## Mouse Control

### click

Click on a UI element with various targeting modes.

```javascript
// Basic click
click({at: "sign in button"})

// With spatial relation
click({at: "link Introduction", spatialRelation: "closest", anchorConcept: "heading Simular Pro"})

// Vision mode (for images/icons without text)
click({at: "simular logo", mode: "vision"})

// Text and screenshot mode
click({at: "Introduction link in Simular Browser section", mode: "textAndScreenshot"})

// Double click
click({at: "Styles.zip", clickType: "doubleClick"})

// Right click
click({at: "file.txt", clickType: "right"})

// Cmd+click
click({at: "verify insurance button", withCommand: true})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `at` | String | Description of target (use role+value: "sign in button") |
| `mode` | String | "vision", "textAndScreenshot", or empty for text-based |
| `clickType` | String | "left" (default), "right", "doubleClick" |
| `withCommand` | Bool | Hold Command during click |
| `element` | UIElement | Direct element reference (ignores `at`) |
| `spatialRelation` | String | Comma-separated: "closest", "furthest", "above", "below", "left", "right", "contains", "containedIn" |
| `anchorConcept` | String | Description of anchor element for spatial relation |
| `prior` | String | Global position hint: "left", "right", "top", "bottom" |
| `position` | String | Position within element: "center", "topleft", "bottomright", etc. |
| `includeInvisible` | Bool | Search elements not currently visible (needs scrolling) |
| `waitForLoadComplete` | Bool | Wait for page load after click |
| `waitTime` | Int | Seconds to wait after click |

### move

Move cursor to an element (same parameters as `click` but uses `to` instead of `at`).

```javascript
move({to: "textarea below verify insurance"})
```

### drag

Drag from current cursor position to a destination.

```javascript
drag({to: "drop zone"})
drag({to: "target area", destinationApp: "Google Chrome"})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `to` | String | Destination description |
| `element` | UIElement | Destination element |
| `destinationApp` | String | Target app if different from origin |

### scroll

Scroll in a direction.

```javascript
scroll({direction: "down"})
scroll({direction: "up", distance: 500})
scroll({direction: "left", distance: 200})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `direction` | String | "up", "down", "left", "right". Default: "down" |
| `distance` | Int | Pixels to scroll. Default: 200 |

---

## Perception

### stateSatisfies

Check if screen meets a natural language condition (slower, uses vision model).

```javascript
if (stateSatisfies({condition: "the page shows a login form"})) {
    respond({message: "Please log in first"});
}
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `condition` | String | Condition to check on current screen |

**Returns:** Boolean

### conceptsExist

Check if all text concepts are visible on screen (faster, text-based).

```javascript
if (conceptsExist({concepts: ["play button", "pause button"]})) {
    click({at: "play button"});
}
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `concepts` | [String] | Array of target concepts to find |

**Returns:** Boolean

### pageContent

Get structured content and screenshot of current screen.

```javascript
var content = pageContent();
// Returns: {imageFilePath: "...", text: "..."}
```

**Returns:** JSON object with `imageFilePath` and `text` fields

---

## Text Generation

### ask

Query a vision-language model about the screen.

```javascript
var content = pageContent();
var result = ask({
    prompt: "Extract all headlines. Return as JSON array.",
    context: content
});
var headlines = JSON.parse(result);
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `prompt` | String | Query for the vision language model |
| `context` | Object/Array | JSON from `pageContent()` or array of such objects |

**Returns:** String response from the model

---

## Wait

### wait

Pause execution for a duration.

```javascript
wait({waitTime: 3})              // 3 seconds
wait({waitTime: 500, unit: "ms"}) // 500 milliseconds
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `waitTime` | Int | Duration to wait |
| `unit` | String | "s" (seconds, default) or "ms" (milliseconds) |

### waitForConcepts

Wait until all concepts appear on screen (timeout: 10 seconds).

```javascript
waitForConcepts({concepts: ["log in button", "sign up button"]})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `concepts` | [String] | Array of concepts to wait for |

---

## User Interaction

### respond

Display a message to the user, optionally requiring confirmation or input.

```javascript
// Simple message
respond({message: "Task complete!"})

// Require yes/no confirmation
var confirmed = respond({message: "Proceed with deletion?", requireConfirm: true});

// Require text input
var userInput = respond({message: "Enter your name:", requireTextInput: true});
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `message` | String | Message to show |
| `requireConfirm` | Bool | Require yes/no answer |
| `requireTextInput` | Bool | Require text input |

**Returns:** Boolean (for confirm) or String (for text input)

---

## System IO

### copyToClipboard / getFromClipboard

```javascript
copyToClipboard({text: "Hello World"})
var content = getFromClipboard()
```

### saveScreenshot / screenshotToClipboard

```javascript
saveScreenshot({fileName: "capture.png", directory: "/Users/me/Desktop"})
screenshotToClipboard()
screenshotToClipboard({element: tableElement})  // Screenshot specific element
```

### readFile / writeToFile

```javascript
var content = readFile({path: "/Users/me/data.json"})

writeToFile({text: jsonResult, path: "/Users/me/output.json"})
writeToFile({text: data, path: "result.json", overwrite: true})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | String | Content to write |
| `path` | String | File path (absolute or filename for default cache dir) |
| `overwrite` | Bool | Overwrite existing file |

---

## Google Sheets

### getGoogleSheetCellValue / setGoogleSheetCellValue

```javascript
var value = getGoogleSheetCellValue({cell: "B42"})
setGoogleSheetCellValue({cell: "A1", value: "Header"})
```

### getGoogleSheetColumns

Get column IDs for headers.

```javascript
var cols = getGoogleSheetColumns({headers: ["Name", "Email", "Date"]})
// Returns: ["A", "B", "C"]
```

---

## Advanced GUI Functions

### getElements

Get UI elements matching criteria.

```javascript
// Get all links
var links = getElements({elementRoles: ["link"]})

// Get elements with spatial relation
var cells = getElements({
    elementRoles: ["cell"],
    spatialRelation: "right",
    anchorOverallDescription: "Name column"
})

// Return as string descriptions
var descriptions = getElements({
    elementRoles: ["button"],
    returnType: "stringArray"
})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `elementRoles` | [String] | Filter by role: "link", "button", "cell", etc. |
| `elementOverallDescription` | String | Description of elements to find |
| `root` | UIElement | Limit search to within this element |
| `spatialRelation` | String | "closest", "above", "below", "left", "right", "contains", "containedIn", "sameRow", "sameColumn" |
| `anchorOverallDescription` | String | Anchor element description |
| `sortBy` | String | Sort by "x" (left-right) or "y" (top-bottom) |
| `returnType` | String | "elementArray", "string", "stringArray", "strToElemDict" |

### getContent

Extract text content from screen or element.

```javascript
var content = getContent({format: "json"})
var tableContent = getContent({inElement: tableElement, format: "xml"})
var sectionContent = getContent({inConcept: "job post details", format: "flat"})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `inConcept` | String | Get content matching this concept |
| `inElement` | UIElement | Get content from this element |
| `format` | String | "flat", "json", "xml", "xmlSlim" |

### getAttributeOfElement

Get an attribute value from an element.

```javascript
var value = getAttributeOfElement({
    elementRole: "textfield",
    elementOverallDescription: "email input",
    attribute: "value"
})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `elementRole` | String | Role of target element |
| `elementOverallDescription` | String | Description of element |
| `attribute` | String | "role", "description", "title", "value" |

### Table Functions

```javascript
// Get cells from a row/column
var cells = getCells({row: rowElement})

// Get cell value
var value = getCellValue({cell: cellElement})

// Get cell label (Excel)
var label = getCellLabel({cell: cellElement})  // Returns "A1"

// Get cell indices by value
var indices = getCellIndices({cellValues: ["value1", "value2"]})

// Get entire column
var column = getTableColumn({header: "Website"})
```
