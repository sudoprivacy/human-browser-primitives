# Human-Browser Interaction Primitives

Version: 0.1.0-draft

## 1. Introduction

This document defines the complete set of **atomic interaction primitives** between a human user and a web browser. Each primitive represents one indivisible human intention — the smallest unit of action or observation that a human performs when using a web application.

### 1.1 Motivation

The W3C WebDriver specification defines input primitives (keyboard, pointer, scroll) across [Section 15: Actions][wd-actions], and observation capabilities across [Section 12: Elements][wd-elements] and [Section 18: Screen Capture][wd-screenshot]. However, no single document consolidates all human-browser interaction primitives — both input and observation — into a unified framework.

This specification performs that consolidation, providing:
- A single reference for all atomic human-browser interactions
- Strict traceability to W3C source definitions
- Explicit identification of observation primitives absent from the Actions API
- A clear boundary between primitives and composite actions

### 1.2 Design Principles

1. **Atomicity**: Each primitive is one indivisible human intention. If an action can be decomposed into two independently meaningful sub-actions, it is not a primitive — it is a composite.

2. **Completeness**: Every possible human-browser interaction can be expressed as a sequence of these primitives.

3. **Orthogonality**: No primitive can be expressed as a combination of other primitives, *at the same level of abstraction*. This spec defines two levels of input primitives, following the W3C structure:
   - **§2.1–2.4 (Actions API, §15)**: Muscle-level — individual motor signals (press, release, move).
   - **§2.5 (Element Interaction, §12.5)**: Intention-level — complete human intentions (click, clear).

   `click` and `pointer_down + pointer_up` are not redundant: they represent different human intentions ("tap this" vs "hold/release for drag"). Intention-level primitives exist because their W3C definitions carry semantics beyond their motor decomposition (e.g., click scrolls into view and checks interactability).

4. **Traceability**: Every primitive references its source in W3C specifications. Primitives not present in W3C specs are explicitly marked as extensions with rationale.

### 1.3 Implementation Layering

Implementations SHOULD separate two parameter layers:

1. **Core parameters**: Defined in this spec, derived from W3C. These are immutable and portable across applications. Generated code MUST NOT be hand-edited.

2. **Convenience parameters**: Application-specific extensions that improve fidelity or completeness of a primitive (e.g., `shadow_dom` on `get_text`, React fallback on `key_down`). These MUST NOT replace cognitive tasks such as target location or content interpretation — those are the agent's responsibility. Convenience parameters are defined in a separate application-level spec (e.g., `ops-spec.yaml`) and generated alongside core implementations.

Both layers SHOULD be auto-generated from their respective specifications. Implementations MAY provide a generator that reads this spec and the application spec to produce code for both layers.

### 1.4 Scope

This specification covers interaction between a human and **web content rendered in a browser viewport**. The following are explicitly out of scope:

- Browser chrome interaction (address bar, tabs, bookmarks)
- Operating system interaction (file dialogs, window management, notifications)
- Composite actions (type "hello" = 5 × key_down + key_up; drag = pointer_down + pointer_move + pointer_up)
- Navigation commands (back, forward, navigate to URL) — these are browser actions, not human-content interactions
- Application workflows (bug filing, test reporting) — these are tool-level concerns, not HCI primitives

### 1.5 Framework Compatibility

Some JavaScript frameworks (React, Vue, Angular) use controlled components that intercept native keyboard events. In these cases, `key_down`/`key_up` sequences may not result in text appearing in input fields.

Implementations SHOULD:
1. Attempt the standard CDP/WebDriver key event dispatch first
2. Detect when the input value did not change as expected
3. Fall back to framework-compatible text injection (e.g., native property setter + input event)
4. Return a hint to the caller indicating the fallback was used

This preserves the primitive vocabulary (`key_down`/`key_up`) while transparently handling framework-specific behavior.

---

## 2. Input Primitives

Input primitives represent actions where the human sends a signal to the browser.

### 2.1 Keyboard

Keyboard primitives model individual key press and release events.

#### `key_down`

Press a key without releasing it.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `value` | string | yes | The key value, as defined in [UI Events KeyboardEvent key Values][uievents-key]. Single characters (e.g., `"a"`, `"5"`) or named keys (e.g., `"Enter"`, `"Shift"`, `"Control"`, `"ArrowUp"`). |

**Source**: [WebDriver §15.4.1 — Key actions][wd-key-actions], action type `keyDown`.

**Notes**:
- Modifier keys (`Shift`, `Control`, `Alt`, `Meta`) remain active until a corresponding `key_up` is issued.
- The `value` follows the [W3C UI Events KeyboardEvent key values][uievents-key] specification.

#### `key_up`

Release a previously pressed key.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `value` | string | yes | The key value to release. Must match a prior `key_down`. |

**Source**: [WebDriver §15.4.1 — Key actions][wd-key-actions], action type `keyUp`.

### 2.2 Pointer

Pointer primitives model mouse, pen, and touch interactions. The W3C spec defines a `pointerType` parameter (`mouse`, `pen`, `touch`) that applies to all pointer actions.

#### `pointer_down`

Press a pointer button at the current pointer position.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `button` | integer | yes | Button index: 0 = left/primary, 1 = middle/auxiliary, 2 = right/secondary. See [WebDriver §15.4.2][wd-pointer-actions]. |

**Source**: [WebDriver §15.4.2 — Pointer actions][wd-pointer-actions], action type `pointerDown`.

**Notes**:
- The button remains pressed until a corresponding `pointer_up` is issued.
- Pressing without releasing enables drag, long-press, and other hold interactions.

#### `pointer_up`

Release a previously pressed pointer button.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `button` | integer | yes | Button index to release. Must match a prior `pointer_down`. |

**Source**: [WebDriver §15.4.2 — Pointer actions][wd-pointer-actions], action type `pointerUp`.

#### `pointer_move`

Move the pointer to a position.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `x` | integer | yes | Target X coordinate in CSS pixels, relative to `origin`. |
| `y` | integer | yes | Target Y coordinate in CSS pixels, relative to `origin`. |
| `duration` | integer | no | Time in milliseconds to complete the movement. Default: 0 (instant). |
| `origin` | string or element | no | Reference point: `"viewport"` (default), `"pointer"` (relative to current position), or an element reference (center of element). See [WebDriver §15.4.2][wd-pointer-actions]. |

**Source**: [WebDriver §15.4.2 — Pointer actions][wd-pointer-actions], action type `pointerMove`.

### 2.3 Wheel

#### `scroll`

Scroll by a delta amount at a given position.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `x` | integer | yes | X coordinate of scroll origin. |
| `y` | integer | yes | Y coordinate of scroll origin. |
| `delta_x` | integer | yes | Horizontal scroll amount in pixels. |
| `delta_y` | integer | yes | Vertical scroll amount in pixels. |
| `duration` | integer | no | Time in milliseconds. Default: 0. |
| `origin` | string | no | `"viewport"` (default) or `"pointer"`. |

**Source**: [WebDriver §15.4.4 — Wheel actions][wd-wheel-actions], action type `scroll`.

### 2.4 Temporal

#### `pause`

Wait for a specified duration. Represents the human intention to wait before acting.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `duration` | integer | yes | Time in milliseconds. |

**Source**: [WebDriver §15.4.5 — Null actions][wd-null-actions], action type `pause`. Also available on all other input sources.

### 2.5 Element Interaction

Element interaction primitives represent complete human intentions directed at web content. Unlike §2.1–2.4 which model individual motor signals (from W3C §15 Actions API), these are defined in W3C §12.5 and carry higher-level semantics — scrolling elements into view, checking interactability, and producing the correct event sequence.

#### `click`

Click at a position. This is the human intention "tap this" — distinct from `pointer_down` + `pointer_up` which models "hold and release" (used for drag).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `x` | integer | yes | X coordinate in CSS pixels. |
| `y` | integer | yes | Y coordinate in CSS pixels. |
| `button` | integer | no | Button index: 0 = left (default), 2 = right. |

**Source**: [WebDriver §12.5.1 — Element Click][wd-element-click].

**Notes**:
- Implementations MUST dispatch `mousePressed` and `mouseReleased` events within the same execution context to ensure the browser recognizes them as a single click gesture.
- Unlike the motor decomposition (`pointer_move` + `pointer_down` + `pointer_up`), the intention-level `click` guarantees correct timing and event sequencing.

#### `clear`

Clear the content of an editable element (input field, textarea).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | element reference | yes | The editable element to clear. |

**Source**: [WebDriver §12.5.2 — Element Clear][wd-element-clear].

**Notes**:
- This is the human intention "empty this field" — a single action, not select-all + delete.

#### `send_keys`

Type text into an element.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | element reference | yes | The target element. |
| `text` | string | yes | The text to type. |

**Source**: [WebDriver §12.5.3 — Element Send Keys][wd-element-send-keys].

**Notes**:
- This is the human intention "type this text into this field" — distinct from individual `key_down`/`key_up` sequences which model individual keystrokes.
- Implementations SHOULD handle framework compatibility (React controlled components, etc.) transparently.

---

## 3. Observation Primitives

Observation primitives represent actions where the human perceives information from the browser. These are **not defined in the W3C Actions API** (Section 15) but are drawn from other sections of the WebDriver specification.

### 3.1 Visual

#### `screenshot`

Capture the current visual state of the viewport as an image.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| *(none)* | | | Captures the full viewport. |

**Source**: [WebDriver §18.1 — Take Screenshot][wd-take-screenshot].

**Notes**:
- This corresponds to the human action of "looking at the screen."
- Implementations should return a lossless image (PNG) of the viewport.

### 3.2 Textual

#### `get_text`

Read the visible text content of the page or a specific element.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | element reference | no | If provided, read text of this element. If omitted, read all visible text on the page. |

**Source**: [WebDriver §12.4.4 — Get Element Text][wd-get-element-text].

**Notes**:
- This corresponds to the human action of "reading text on screen."
- Should include text rendered inside Shadow DOM, as it is visible to the human.
- The W3C spec defines this for individual elements. The "full page" variant is an **extension** motivated by the need for AI agents to perceive all visible content.

### 3.3 Element State

#### `get_attribute`

Read a named attribute or property of an element.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | element reference | yes | The target element. |
| `name` | string | yes | Attribute name to read (e.g., `"aria-checked"`, `"value"`, `"href"`). |

**Source**: [WebDriver §12.4.2 — Get Element Attribute][wd-get-element-attribute].

**Notes**:
- This corresponds to the human perception of element state (e.g., "is this checkbox checked?").
- Humans infer attribute state visually; this primitive provides the programmatic equivalent.

#### `is_displayed`

Determine whether an element is visible to the user.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | element reference | yes | The target element. |

**Source**: [WebDriver §12.4.8 — Is Element Displayed][wd-is-displayed] (Appendix).

---

## 4. Composite Actions (Non-Normative)

The following are **NOT primitives**. They are common compositions of primitives, listed here for reference. Implementations MAY provide these as convenience functions but MUST NOT treat them as atomic.

| Composite | Decomposition | Description |
|-----------|---------------|-------------|
| double_click(x, y) | `click(x, y)` × 2 | Double click |
| drag(x1, y1, x2, y2) | `pointer_move(x1, y1)` + `pointer_down(0)` + `pointer_move(x2, y2)` + `pointer_up(0)` | Drag and drop |
| type(text) | `key_down(c)` + `key_up(c)` for each character `c` in text | Type a string |
| press_key(key) | `key_down(key)` + `key_up(key)` | Press and release a key |
| press_combo(mod, key) | `key_down(mod)` + `key_down(key)` + `key_up(key)` + `key_up(mod)` | Key combination (e.g., Ctrl+Enter) |

**Note**: `click` was promoted from composite to primitive in §2.5, referencing W3C §12.5.1. `right_click` is expressed as `click(x, y, button=2)`.

---

## References

- [wd-actions]: https://w3c.github.io/webdriver/#actions "WebDriver §15 — Actions"
- [wd-key-actions]: https://w3c.github.io/webdriver/#keyboard-actions "WebDriver §15.4.1 — Key Actions"
- [wd-pointer-actions]: https://w3c.github.io/webdriver/#pointer-actions "WebDriver §15.4.2 — Pointer Actions"
- [wd-wheel-actions]: https://w3c.github.io/webdriver/#wheel-actions "WebDriver §15.4.4 — Wheel Actions"
- [wd-null-actions]: https://w3c.github.io/webdriver/#null-actions "WebDriver §15.4.5 — Null Actions"
- [wd-elements]: https://w3c.github.io/webdriver/#elements "WebDriver §12 — Elements"
- [wd-get-element-text]: https://w3c.github.io/webdriver/#get-element-text "WebDriver §12.4.4 — Get Element Text"
- [wd-get-element-attribute]: https://w3c.github.io/webdriver/#get-element-attribute "WebDriver §12.4.2 — Get Element Attribute"
- [wd-is-displayed]: https://w3c.github.io/webdriver/#element-displayedness "WebDriver §12.4.8 — Is Element Displayed"
- [wd-element-click]: https://w3c.github.io/webdriver/#element-click "WebDriver §12.5.1 — Element Click"
- [wd-element-clear]: https://w3c.github.io/webdriver/#element-clear "WebDriver §12.5.2 — Element Clear"
- [wd-element-send-keys]: https://w3c.github.io/webdriver/#element-send-keys "WebDriver §12.5.3 — Element Send Keys"
- [wd-take-screenshot]: https://w3c.github.io/webdriver/#take-screenshot "WebDriver §18.1 — Take Screenshot"
- [wd-screenshot]: https://w3c.github.io/webdriver/#screen-capture "WebDriver §18 — Screen Capture"
- [uievents-key]: https://www.w3.org/TR/uievents-key/ "UI Events KeyboardEvent key Values"
- [webdriver-bidi]: https://w3c.github.io/webdriver-bidi/ "WebDriver BiDi"
- [webdriver-bidi-input]: https://w3c.github.io/webdriver-bidi/#module-input "WebDriver BiDi §7.9 — Input Module"

---

## Appendix A: Relationship to Existing Specifications

| This Spec | Source |
|-----------|--------|
| §2.1–2.4 Input Primitives (Actions) | [WebDriver §15 Actions][wd-actions], [WebDriver BiDi §7.9 Input][webdriver-bidi-input] |
| §2.5 Input Primitives (Element Interaction) | [WebDriver §12.5 Element Interaction][wd-elements] |
| §3 Observation Primitives | [WebDriver §12 Elements][wd-elements], [WebDriver §18 Screen Capture][wd-screenshot] |
| §4 Composites | Derived from §2 |

The W3C WebDriver Actions API (§15) defines input primitives only. Observation primitives are scattered across other sections of the WebDriver specification. This document consolidates both into a single framework.

## Appendix B: Comparison with Industry Implementations

| Primitive | This Spec | Anthropic Computer Use | OpenAI Computer Use |
|-----------|-----------|----------------------|-------------------|
| key_down | `key_down(value)` | — | — |
| key_up | `key_up(value)` | — | — |
| press_key (composite) | §4 | `key(text)` | `keypress(keys)` |
| type (composite) | §4 | `type(text)` | `type(text)` |
| pointer_down | `pointer_down(button)` | `left_mouse_down` | — |
| pointer_up | `pointer_up(button)` | `left_mouse_up` | — |
| pointer_move | `pointer_move(x, y)` | `mouse_move(coordinate)` | `move(x, y)` |
| click | `click(x, y)` | `left_click` | `click(x, y)` |
| scroll | `scroll(...)` | `scroll(...)` | `scroll(...)` |
| pause | `pause(duration)` | `wait(duration)` | `wait` |
| screenshot | `screenshot()` | `screenshot` | `screenshot` |
| get_text | `get_text(element?)` | — | — |
| get_attribute | `get_attribute(element, name)` | — | — |
| is_displayed | `is_displayed(element)` | — | — |

| clear | `clear(element)` | — | — |
| send_keys | `send_keys(element, text)` | — | — |

**Notable**: Both Anthropic and OpenAI define `click` as a primitive (single action), not as a composite of pointer_down + pointer_up. Anthropic additionally defines `left_mouse_down`/`left_mouse_up` for hold interactions (drag). Neither defines observation primitives beyond `screenshot`.
