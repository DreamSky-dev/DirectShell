# Day 2: When an AI Learns to Use a Browser for the First Time

*February 18, 2026*

It's been a day of highs and lows. Day 1 proved the concept behind DirectShell works. First tests. First programs. First successes. 360 cells in Google Sheets, cross-app communication, Notepad in milliseconds. All without a single screenshot.

Day 2 is teaching me something different — something I didn't see coming.

Giving an AI data and tools is one thing. Knowing HOW it uses those tools and data is another.

AI models are built on pattern matching. They complete and replicate what they know from their training data. The problem: DirectShell and its concept are new. No training data. No prior knowledge. No "here's how to do it right." It is, in the truest sense, an AI learning to operate a browser like a human for the first time. With a keyboard. With a mouse. And with "shit, pressed Enter one too many times."

I know what you're thinking — an AI knows browsers and keys. And yes, it knows the basic HOW. But it knows APIs. It knows images. What it doesn't know: How to fill a table in Google Sheets using a keyboard and mouse. That's a first.

## What happened today

I asked it to create a benchmark table in Google Sheets. 3 sections, 8 columns, 25+ rows. Here's what followed:

- **First attempt:** Everything landed in a single cell. Tab and Enter were written as text characters instead of executed as keyboard commands. The translator between intent and keystrokes was missing.

- **Second attempt:** Tabs and Enter worked — but the title "bled" into the data rows. Sheets jumps back to the start column of the last selection after Enter, not to column A. The AI didn't know that.

- **Third attempt:** Half the characters landed in Sheets, the other half in my terminal. Focus kept switching between windows.

- **Fourth attempt:** Rebuilt to be completely non-blocking — PostMessage directly to the window handle. No more focus stealing. But: Chromium ignores simple WM_CHAR messages. Nothing arrived.

Classic trial and error. Frustrating and educational in equal measure.

But this is exactly what brought me closer to a solution. Because I realized: "I need to create this training data myself."

## The Solution: Organic Learning Through Learnings Files

I built the following mechanism into the MCP pipeline: For each application and context (e.g., "Opera + Google Sheets"), a learnings file is created. The AI stores its discoveries there — what works, what doesn't, what quirks a program has.

Next time it calls `ds_update_view`, it doesn't just get Gemini's screen analysis back (more on that in a moment) — it also gets a list of all available learnings for that app. It reads the relevant file before taking action.

This is organic learning: **Try → Fail → Learn → Save → Do better next time.**

Today, this solved the Sheets problem. It now knows: Tab = next cell, Enter = next row, no tabs in empty lines, pad title rows with tabs. Next time, it'll get it right on the first try.

Now imagine: 1 million users with 1 million programs. And a shared database. Then it'll work fantastically. Every mistake one user makes becomes training data for everyone else.

## Second Update: CDP — When the Accessibility Tree Isn't Enough

We discovered that some web pages don't expose all interactive elements in the accessibility tree. Google Sheets, for example, renders on canvas — the a11y tree shows exactly 0 operable elements for the spreadsheet itself.

The insight: Programs vs. browsers are fundamentally different in some aspects. The accessibility tree is perfect for native applications — there we see every button, text field, menu. But browsers have a second data source: the DOM.

The solution: We built Chrome DevTools Protocol (CDP) directly into the DirectShell executable. When snapping to a browser, DirectShell automatically checks if CDP is active on port 9222. If not, it shows a popup, closes the browser, and relaunches it with the right flags (`--remote-debugging-port=9222 --remote-allow-origins=*`). Fully automatic, no manual fiddling.

The result: Where the a11y tree showed 0 elements, CDP now delivers 65+ actionable tools — menus, formatting buttons, font sizes, everything. And the best part: it integrates seamlessly. `ds_update_view` now sends both a11y data AND CDP data to the Gemini translator, which builds a unified tool list from both sources.

At the same time, I built `ds_update_view` itself. It now sends the a11y and a11y.snap files along with the DOM tree handle (when available) to a cheap, fast Gemini Flash 2.5 Lite — which produces a single response: "what am I looking at and what input fields exist." I parse this response live, creating an up-to-date tool list for the MCP on the fly. This is a minimal increase in "cost" but still orders of magnitude smaller than feeding 10-50 screenshots into a vision model.

## Two extremely valuable learnings

1. **AI doesn't just need tools — it needs experience.** And if we systematically store and share that experience, something powerful emerges.

2. **One data source isn't enough.** Programs need a11y. Browsers need a11y + DOM. The architecture must be flexible enough to fuse multiple sources.

Until the next update.
— Martin
