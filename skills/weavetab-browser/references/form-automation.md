---
id: form-automation
domain: browser
triggers:
  - "fill form"
  - "submit form"
  - "select dropdown"
  - "multi field input"
  - "radio button"
  - "checkbox"
tools:
  - "browser_fill"
  - "browser_select"
weavetab: ">=2.6.4"
---

# Form Automation: Atomic Multi-Field Execution

Submitting complex web forms one field at a time wastes agent turns and increases the probability of layout shifts. Weavetab provides atomic multi-field batching and native dropdown selectors.

---

## 1. Atomic Multi-Field Filling (`browser_fill`)

`browser_fill` populates multiple inputs, textareas, checkboxes, and radio buttons in a single CDP roundtrip.

```json
{
  "fields": [
    {
      "ref": "w:12",
      "value": "Alex"
    },
    {
      "ref": "w:14",
      "value": "Rivera"
    },
    {
      "ref": "w:16",
      "value": "contact@weavetab.dev"
    },
    {
      "ref": "w:19",
      "value": "true"
    }
  ]
}
```

### Supported Field Value Types:
Every `value` is a **string**.
- **Text / Search / Email / Tel**: The literal text to type.
- **Checkbox / ARIA switch**: A truthy string (`"true"`, `"yes"`, `"checked"`, `"1"`, or `"on"`) checks it; anything else unchecks it.
- **Radio Buttons**: Set the matching option's ref to a truthy string to select it.
- **Select**: The option `value` or visible text to choose.
- **ContentEditable**: The text content to set (inner text formatting handled automatically).

---

## 2. Dropdown, Multi-Select & Text Selection (`browser_select`)

Standard `<select>` elements and custom dropdowns often fail under simple click-and-type automation. `browser_select` dispatches native CDP change events, supports multi-select arrays, and can highlight text ranges inside inputs or DOM nodes.

### Single Option Select (`mode: "option"`, default):
```json
{
  "ref": "w:31",
  "value": "enterprise_tier"
}
```

`value` matches either the option's `value` attribute or its visible text (trimmed). For custom ARIA/React dropdowns it opens the popover, scans `[role="option"]`/`[role="menuitem"]` candidates, and clicks the best match.

### Multi-Select Support (`values`):
For `<select multiple>` or multi-option dropdowns, pass an array of strings via `values`:

```json
{
  "ref": "w:31",
  "values": ["us-east", "eu-west", "ap-south"]
}
```

### Text Range Highlighting (`mode: "text"`):
To highlight text inside an `<input>`, `<textarea>`, or DOM element (for copy/edit workflows), set `"mode": "text"`. On inputs and textareas, optional `start` and `end` character offsets select a sub-range (omitting them selects the entire value or element contents):

```json
{
  "ref": "w:18",
  "mode": "text",
  "start": 0,
  "end": 12
}
```

---

## 3. Form Submission Best Practices

1. **Pre-Validate State**: Before clicking Submit, confirm that required fields have passed HTML5/client-side validation (no active red borders or validation popups).
2. **Submit via Enter Key vs Click**:
   - If the submit button has a dynamic ref or changes during typing, send `"key": "Enter"` via `browser_key` inside the last filled text field.
   - Otherwise, locate the submit button via `browser_find` and click via `browser_click`.
3. **Wait for Post-Submission Route**: Always follow form submission with `browser_wait` for URL changes or target DOM confirmation rather than immediately firing subsequent actions.
