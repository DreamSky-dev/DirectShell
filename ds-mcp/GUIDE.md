# DirectShell — Quick Start Guide

## First Thing: Read the Screen

Call `ds_update_view()` — this is your PRIMARY tool. It returns:
1. **Visible text** — only what a human can see in the viewport right now
2. **Tools** — every actionable element currently rendered on screen

Always start here. Always come back here after major actions.

## Tool Reference

| Tool | What it does |
|---|---|
| **Perception** | |
| `ds_update_view()` | **START HERE.** Viewport-visible text + actionable elements. |
| `ds_screen()` | Viewport-visible text only (no tools). |
| `ds_print()` | **Full page** text from top to bottom (not just viewport). |
| `ds_state()` | Compact numbered list of operable elements with type prefixes. |
| `ds_elements()` | All interactive elements with input type classification. |
| `ds_query(sql)` | SQL query against the element database. |
| `ds_find(pattern)` | Find elements by name pattern (% as wildcard). |
| `ds_events()` | Delta perception — only what changed since last check. |
| **Navigation** | |
| `ds_apps()` | Lists all open applications and which one is focused. |
| `ds_focus(app)` | Switches to a different native application (UIA snap). |
| `ds_tabs()` | Lists all open browser tabs (CDP). |
| `ds_tab(identifier)` | Switches to a browser tab by number or name (CDP). |
| `ds_navigate(url)` | Navigates the active browser tab to a URL (CDP). |
| **Actions** | |
| `ds_act(tool_number)` | Executes an action from `ds_update_view()` by its number. |
| `ds_click(name)` | Clicks a named UI element. |
| `ds_text(value, target)` | Sets text in an input field instantly (preferred over ds_type). |
| `ds_type(text)` | Types text character-by-character into whatever has focus. |
| `ds_key(combo)` | Presses a key combo (e.g., "ctrl+s", "enter", "pagedown"). |
| `ds_scroll(direction)` | Scrolls up/down/left/right by mouse wheel notches. |
| `ds_batch(actions)` | Executes multiple actions in sequence. |
| **Memory** | |
| `ds_status()` | Check if snapped to an app. |
| `ds_learn(app, context)` | Read/write persistent learnings for an app. |
| `ds_profile_list()` | List known app profiles. |
| `ds_profile_save(app, ...)` | Save semantic element mappings for an app. |
| `ds_profile_get(app)` | Get saved profile for an app. |

## Workflow

### Browser (CDP — port 9222)
```
ds_tabs()              → see all browser tabs
ds_tab("computerbase") → switch to tab
ds_update_view()       → viewport text + tools
ds_act(3)              → execute tool #3
ds_navigate("url")     → go to URL
ds_scroll("down", 3)   → scroll down
ds_update_view()       → verify result
```

### Native App (UIA)
```
ds_apps()              → see what's open
ds_focus("discord")    → snap to it
ds_update_view()       → read screen + get tools
ds_act(3)              → execute tool #3
ds_update_view()       → verify result
```

## Key Rules

- **Viewport by default**: All tools show ONLY what is visible on screen. Exception: `ds_print()` reads the full page.
- **Browser = CDP, Native = UIA**: Browser actions go through CDP (port 9222), native app actions through UIA. No mix.
- **ds_text > ds_type**: Always prefer ds_text (instant). Only use ds_type for apps that reject ValuePattern (Discord chat, terminals).
