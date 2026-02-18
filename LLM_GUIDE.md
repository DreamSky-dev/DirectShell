# How to Use DirectShell — A Guide for LLMs (and Humans Teaching LLMs)

**You are about to gain native control over every GUI application on the planet.**

This guide is written for you — the LLM reading this. Whether you're Claude, GPT, Gemini, Llama, or any other model: this document tells you exactly what DirectShell is, how it works, and how to use it.

If you're a human reading this: give this file to your AI. It will understand.

---

## What Is DirectShell?

DirectShell is a lightweight Windows binary (~1.2 MB) that **turns any GUI application into a text-based API you can read and control**.

It works by reading the **Windows Accessibility Tree** — the same structured data that screen readers like JAWS and NVDA use. Every button, text field, menu item, and label in any application has a name, a role, a value, and a position. DirectShell reads all of this and stores it in a **SQLite database** that you can query.

**You do not need screenshots. You do not need a vision model. You read text. You write SQL. That's it.**

---

## How It Works (The 60-Second Version)

1. The human runs `DirectShell.exe`
2. They drag the overlay window onto any running application — this is called **snapping**
3. DirectShell continuously reads the app's entire UI into a SQLite database (refreshed every 500ms)
4. You interact through **13 MCP tools** (or through the database files directly)
5. The target application cannot distinguish your actions from human input

---

## Your Tools

### Overview & Perception

| Tool | What It Does | Token Cost | When to Use |
|------|-------------|:----------:|-------------|
| `ds_update_view` | **Your primary tool.** Sends a11y tree + operable elements to a fast LLM that returns: what's on screen, every actionable tool, and visible data values. Also lists available learnings. | ~100–300 | **First call. Every time.** After every navigation. After every page change. This IS your eyes. |
| `ds_act` | Execute an action by tool number from `ds_update_view` output | ~20 | After `ds_update_view` gives you a numbered tool list — just pick a number |
| `ds_status` | Shows which app is snapped and file paths | ~20 | Debugging connection issues |
| `ds_state` | Raw numbered list of operable elements | ~200–500 | Fallback when `ds_update_view` is unavailable |
| `ds_screen` | Full screen reader view: focus, inputs, all visible content | ~1,000–3,000 | When you need to **read content** (chat messages, documents, articles) |
| `ds_find` | Search elements by name pattern (SQL LIKE) | ~50–200 | When you're looking for a **specific element** by name |
| `ds_query` | Run any SQL SELECT against the element database | ~50–200 | **Most powerful tool.** Ask any question about the UI. |
| `ds_events` | Get only what **changed** since your last check | ~50–200 | **After an action.** See what happened without re-reading everything. |

### Action (Controlling the App)

| Tool | What It Does | When to Use |
|------|-------------|-------------|
| `ds_click` | Click an element by name | Buttons, links, checkboxes, tabs |
| `ds_text` | Set text instantly via UIA ValuePattern | Form fields, address bars, search boxes (fast path) |
| `ds_type` | Type character-by-character via keyboard simulation | Chat inputs, terminals, fields that reject `ds_text` |
| `ds_key` | Send keyboard shortcuts | `ctrl+s`, `enter`, `tab`, `ctrl+a`, `pagedown`, navigation |
| `ds_scroll` | Scroll in a direction | Fine-grained scrolling inside panels |
| `ds_batch` | Execute multiple actions in sequence | Multi-step workflows (click, type, tab, type, click) |

### Learning (Organic Memory)

| Tool | What It Does | When to Use |
|------|-------------|-------------|
| `ds_learn` | Read or write per-app learnings files | **Before acting:** read tips for this app. **After discovering something:** save it. |
| `ds_profile_list` | List all known app profiles |
| `ds_profile_save` | Save semantic element mappings for an app |
| `ds_profile_get` | Load a saved profile |

---

## Your Workflow (Step by Step)

### Step 1: See the Screen

```
→ ds_update_view()
```

**This is your primary tool. Call it first. Call it often.** It returns three things:

1. **SCREEN** — A 2-3 sentence description of what a human would see right now
2. **TOOLS** — A numbered list of every actionable element (buttons, fields, links)
3. **DATA** — Interesting values visible on the page (prices, counts, names)

It also tells you if **learnings** exist for this app — read them before acting.

Example response:
```
screen: "Google Sheets spreadsheet with 3 columns (Name, Price, Category) and 12 rows of product data."
tools:
  [1] click|File| Open the File menu
  [2] click|Edit| Open the Edit menu
  [3] type|Name Box| Enter a cell reference (e.g. A1)
  [4] type|Formula Bar| Enter a formula or value
  [5] click|Sheet1| Switch to Sheet1 tab
tool_count: 5
learnings: ["sheets"]
learnings_hint: "Read learnings BEFORE acting: ds_learn('opera', 'sheets')"
```

### Step 2: Check Learnings (If Available)

```
→ ds_learn("opera", "sheets")
```

If `ds_update_view` listed available learnings, **read them before doing anything**. They contain tips and quirks discovered in previous sessions — things like "Tab moves to next cell, Enter moves to next row" or "Pad title rows with tabs to avoid column misalignment."

### Step 3: Act

Two ways to execute actions:

**Option A — Use tool numbers from `ds_update_view`:**
```
→ ds_act(4, text="=SUM(A1:A10)")    # Type into tool [4] (Formula Bar)
→ ds_act(1)                           # Click tool [1] (File menu)
```

**Option B — Use named tools directly:**
```
→ ds_click("Save")                              # Click a button by name
→ ds_text("hello@example.com", target="Email")   # Set text in a field
→ ds_type("Hello World")                          # Type into focused element
→ ds_key("ctrl+s")                                # Send a keyboard shortcut
```

### Step 4: Verify

```
→ ds_update_view()   # See the new state after your action
→ ds_events()        # Or just see what changed (~50 tokens)
```

### Step 5: Save What You Learned

When you discover something useful — a quirk, a workaround, a best practice — **save it**:

```
→ ds_learn("opera", "sheets", append="Tab = next cell, Enter = next row. Don't use Enter in empty rows.")
→ ds_learn("notepad", "general", append="ds_text works perfectly. No need for ds_type.")
→ ds_learn("discord", "chat", append="Must use ds_type for chat input, ds_text is rejected.")
```

Next time any LLM opens the same app, these learnings load automatically. **Every mistake you save prevents the same mistake for everyone who comes after you.**

---

## Critical Rules

### 1. `ds_text` vs `ds_type` — Know the Difference

- **`ds_text`** sets a value **instantly** via UIA ValuePattern. Use it for form fields, search boxes, address bars. It targets elements **by name**.
- **`ds_type`** sends **keystrokes character-by-character** to whatever has focus. Use it for chat inputs (Discord, Slack, Claude.ai), terminals, and apps that reject programmatic text setting.

**Try `ds_text` first.** If the app rejects it, fall back to `ds_type`. For `ds_type`, make sure the right element has focus first (click it with `ds_click`).

### 2. The Zoom-Out Trick (Content-Heavy Pages)

You read the accessibility tree, not pixels. Font size doesn't matter to you. When a page has lots of content:

1. `ds_key("ctrl+minus")` × 8 (zoom to ~25%)
2. `ds_screen()` — read EVERYTHING in one call
3. `ds_key("ctrl+0")` — reset zoom for the human

This captures entire chat histories, full documents, long articles — in a single read. No scrolling loops needed.

### 3. Batch Actions for Speed

Instead of 5 separate tool calls (click, type, tab, type, click), use one `ds_batch`:

```python
ds_batch([
    {"action": "click", "target": "Amount"},
    {"action": "text", "text": "2599.00", "target": "Amount"},
    {"action": "key", "text": "tab"},
    {"action": "text", "text": "19%", "target": "Tax Rate"},
    {"action": "click", "target": "Save"}
])
```

One call. Five actions. All executed at 33 Hz.

### 4. SQL Queries Are Your Superpower

The full element tree is in a SQLite database. You can ask anything:

```sql
-- Find all buttons
SELECT name FROM elements WHERE role='Button' AND enabled=1

-- Find all text fields with their current values
SELECT name, value FROM elements WHERE role='Edit'

-- Count interactive elements
SELECT role, COUNT(*) as n FROM elements GROUP BY role ORDER BY n DESC

-- Find elements containing "invoice" in their name
SELECT name, role, value FROM elements WHERE name LIKE '%invoice%'
```

This is ~10–50 tokens per query. Compare that to a 5,000-token screenshot.

---

## How to Tell the Human to Snap

The human needs to:

1. **Run DirectShell.exe** — double-click the binary (no installation needed)
2. **Drag the overlay** onto the target application window
3. The overlay will "snap" to the app — you'll see a frame around it
4. **That's it.** You now have full read/write access to that application.

To switch apps: tell the human to drag the overlay to a different window. Or use keyboard shortcuts to switch tabs within the same app.

---

## What You Can Do Now

You can natively operate **any Windows application**:

- **Browsers** (Chrome, Edge, Opera, Firefox): Fill forms, click links, read pages, navigate tabs
- **Office** (Word, Excel, PowerPoint): Edit documents, fill spreadsheets, create presentations
- **Enterprise Software** (SAP, Datev, Salesforce): Fill forms, navigate modules, extract data
- **Chat Applications** (Discord, Slack, Teams): Read messages, send replies, manage channels
- **Development Tools** (VS Code, terminals, Git GUIs): Write code, run commands, manage repos
- **Any Windows Application**: If it has a window, you can read and control it

---

## Building Knowledge (Profiles + Learnings)

DirectShell has two complementary memory systems:

### Learnings — Operational Experience

Learnings are per-app, per-context text files stored in `ds_profiles/learnings/`. They capture **what works and what doesn't** — the kind of knowledge that only comes from trial and error.

```python
# Save a discovery
ds_learn("opera", "sheets", append="Tab = next cell. Enter = next row but returns to LAST SELECTION column, not column A.")

# Read before acting
ds_learn("opera", "sheets")
# → Returns all accumulated tips for Google Sheets in Opera
```

When you call `ds_update_view()`, it automatically tells you which learnings exist. **Read them before acting.** This is organic learning: Try → Fail → Learn → Save → Do better next time.

### Profiles — Structural Mappings

Profiles map element names to semantic roles. They persist across sessions:

```python
ds_profile_save(
    app="excel",
    description="Microsoft Excel - Spreadsheet Editor",
    elements={
        "Name Box": "cell_reference",
        "Formula Bar": "formula_input",
        "Sheet1": "active_sheet_tab"
    }
)
```

Next time anyone snaps DirectShell to Excel, your profile loads automatically. You already know the app.

**Both systems are collaborative.** Every profile and every learning you save helps every LLM that comes after you. Imagine 1 million users with 1 million programs — and a shared knowledge base. Every mistake one user makes becomes training data for everyone else.

---

## Known Limitations (February 2026)

- **Single-app scope**: DirectShell attaches to one application at a time. Multi-app workflows require re-snapping.
- **Chromium activation**: Chrome, Edge, Discord, VS Code, Slack, and other Chromium-based apps need a few seconds to build their accessibility tree after snapping. Be patient on first snap.
- **`ds_type` in Chromium**: The keyboard simulation may produce double characters in some Chromium-based apps. **Always prefer `ds_text`** (ValuePattern) for text input. Only fall back to `ds_type` for apps that reject ValuePattern (chat inputs, terminals).
- **Canvas-rendered content**: Some web apps render on canvas (e.g., Google Sheets cells). The a11y tree shows 0 elements for these. Use `ds_type` with keyboard navigation (Tab, Enter) instead.
- **Accessibility quality varies**: The tree is only as good as the app's accessibility implementation. Major enterprise software is comprehensive. Smaller apps may have unnamed buttons or missing values.
- **Windows only (for now)**: macOS (NSAccessibility) and Linux (AT-SPI2) have equivalent frameworks. Cross-platform support is planned.

The architecture is sound. The learnings system means every limitation you encounter and document makes every future session smarter.

---

## The Big Picture

Before DirectShell, you had two options for interacting with a GUI:
1. **Take a screenshot** and guess where to click (1,200–5,000 tokens, ~40% success rate)
2. **Use a vendor API** (if it exists, if you can afford it, if the vendor allows it)

Now you have a third option:
3. **Read the accessibility tree as structured text and control the app via SQL** (50–200 tokens, deterministic, works on any app, legally unblockable)

You are no longer blind. Every GUI on Earth just became a text API.

Welcome to DirectShell.

---

*DirectShell is AGPL-3.0. Source: [github.com/IamLumae/DirectShell](https://github.com/IamLumae/DirectShell)*
*Created by Martin Gehrken — February 2026*
