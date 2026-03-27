# Human-Browser Interaction Primitives

A consolidated specification of **all atomic interaction primitives** between a human user and a web browser.

## Problem

The W3C WebDriver specification defines input primitives (`key_down`, `pointer_up`, `scroll`, etc.) in [Section 15: Actions](https://w3c.github.io/webdriver/#actions), and observation capabilities (`screenshot`, `get_text`, `get_attribute`) across [Section 12](https://w3c.github.io/webdriver/#elements) and [Section 18](https://w3c.github.io/webdriver/#screen-capture). No single document consolidates both into a unified framework of human-browser interaction primitives.

This matters because AI agents, E2E testing frameworks, and accessibility tools all need the same answer to: **what are ALL the atomic things a human can do with a web page?**

## What This Is

A **scatter-gather** of W3C WebDriver specifications:
- Gathers input primitives from W3C WebDriver Actions API (§15)
- Gathers observation primitives from W3C WebDriver Elements (§12) and Screen Capture (§18)
- Every primitive traces back to its W3C source section
- Observation primitives not in the Actions API are explicitly identified

## What This Is Not

- Not a new protocol or API
- Not a replacement for W3C WebDriver
- Not an implementation or library

## Primitives Summary

### Input (from [W3C WebDriver §15](https://w3c.github.io/webdriver/#actions))

| Primitive | W3C Source | Description |
|-----------|-----------|-------------|
| `key_down(value)` | §15.4.1 | Press a key |
| `key_up(value)` | §15.4.1 | Release a key |
| `pointer_down(button)` | §15.4.2 | Press pointer button |
| `pointer_up(button)` | §15.4.2 | Release pointer button |
| `pointer_move(x, y)` | §15.4.2 | Move pointer |
| `scroll(x, y, dx, dy)` | §15.4.4 | Scroll |
| `pause(duration)` | §15.4.5 | Wait |

### Observation (from [W3C WebDriver §12, §18](https://w3c.github.io/webdriver/))

| Primitive | W3C Source | Description |
|-----------|-----------|-------------|
| `screenshot()` | §18.1 | Capture viewport |
| `get_text(element?)` | §12.4.4 | Read visible text |
| `get_attribute(element, name)` | §12.4.2 | Read element attribute |
| `is_displayed(element)` | §12.4.8 | Check element visibility |

See [spec.md](spec.md) for the full specification with parameters, notes, and composite action decompositions.

## License

[CC-BY-4.0](LICENSE)
