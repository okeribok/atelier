---
title: Atelier Tutorial - AI-coding consistency machine
tags: tma0
---

# Learning Atelier: a hands-on tour

Atelier is a single HTML file. It has no server of its own — it talks to a
`llama.cpp` model running on your own machine, and it writes plain Markdown
and JavaScript files to a folder you pick. Nothing leaves your computer.

This tutorial is a set of small exercises, in order, each one turning on a
different part of the tool. Do them with the counter app that Atelier writes
into an empty folder the first time you open one. By the end you will have
touched every button in the header.

## Before you start

1. Start a local model: `llama-server -m <your-model.gguf> --jinja`
2. **Serve `atelier.html` over `http://`, don't just double-click it.**
   Chrome and Edge don't reliably support the file-system features this app
   needs when the page itself is opened as a `file://` URL — you may see the
   folder picker fail, or storage silently not work. From the folder
   containing `atelier.html`, run:

   ```
   python3 -m http.server 8000
   ```

   then open `http://localhost:8000/atelier.html`. (Node's `npx serve` works
   just as well if you have it.) This is a static file server with no logic
   of its own — it's not "the backend," it just exists because browsers
   treat `http://` pages and `file://` pages differently.
3. Click **Open folder**, choose an empty folder. Atelier writes the counter
   project into it and shows `story.md`.
4. Click **Ping**. You should see the model's name, its context size, and a
   round-trip time. If this fails, the rest of the tutorial will too — fix
   your `llama-server` URL first.

---

## Why a tool like this, and when *not* to use it

Cloud agents (Claude Code, Cursor, Codex, and similar tools) and a strong
on-prem model like Qwen3.8-27B are built around a different bet: give a
powerful model a long leash and a lot of autonomy, because the model is
good enough to be trusted with it. That bet is currently measured with a
benchmark called **SWE-bench Verified** — a set of real bug reports from
real open-source *Python* projects, where an agent is judged on whether its
patch makes a hidden test suite pass. It's a reasonable proxy for "can I
turn this loose on my actual codebase and trust the result," and it's why
you'll see it quoted everywhere in 2026 coding-tool comparisons, with
frontier agents now scoring in the high 80s (percent of issues resolved).

Worth knowing: the original SWE-bench is Python-only. There's a companion
benchmark, **SWE-bench Multimodal**, built specifically for front-end
JavaScript/HTML/CSS bugs — around 600 tasks from real JS libraries for
things like UI, charting, and mapping. The same tools that score in the
high 80s on the Python benchmark dropped to roughly 10% on this one when it
was introduced. That's not a knock on those tools — it just means "visual,
browser-side software" was, and largely still is, a harder, less-benchmarked
corner of the field. It's also exactly Atelier's home turf.

Atelier is answering a different question, on purpose. It assumes the model
is *not* good enough to trust with a long leash — because it's small,
because it's running on a laptop, because it might be a 3B or 4B model
with no SWE-bench score worth mentioning — and it builds a short, tight
loop around that weaker model instead: propose one small thing, check it
mechanically, test it, keep it only if it strictly helped, ask you when
it's unsure. None of that shows up on a SWE-bench leaderboard, because
SWE-bench measures model capability, not loop design.

**Use Atelier when:** you want a private, zero-cost, fully inspectable loop
for a small HTML/JS app or tool, you want to stay hands-on and see every
change before it lands, or you're teaching someone what "vibe coding"
actually does under the hood.
Atelier is not an "AI-coding tool", it is an AI-coding consistency machine.

**Reach for a cloud agent or a bigger on-prem model instead when:** the
codebase is large or multi-language, you need it to browse the web or call
real tools, or the task genuinely benefits from a model that can hold and
reason about far more context than a 3–8B model can — Atelier's layered
files and clipped context windows are a deliberate trade against that kind
of scale.

---


## Level 1 — Look around before you touch anything

Open every file in the left-hand tree. Notice:

- Files are numbered in reading order: `0` the project's own ground rules
  (`atelier.md`), `1` story, `2`/`2.1` spec and quality, `3` decisions,
  `4` architecture, `5` pseudocode, `6` tests, `7` snippets. This is the
  same order the model reads them in.
- Each row has a coloured dot. Green means consistent; grey means nothing
  depends on it yet.
- Click `snippets/ui.js`. Above the editor, the **blast radius line**
  shows what feeds it and what it feeds — this is the actual dependency
  graph, read straight off the files, not a guess.
- Click **Diagram** while `arch.md` is open. It renders the mermaid block
  from that file.
- Click **Run tests**. Watch the live preview iframe update, the console
  panel underneath it, and the pass/fail list at the bottom.

Nothing here calls the model. This is the deterministic half of Atelier.

## Level 2 — Make one small edit by hand, manually

Open `story.md` and change one word — for example, "instant" to
"immediate." Save (click away from the box, or Ctrl/Cmd+S).

- The tree now marks `story.md`'s neighbour (`spec.md`) with `!` — it
  hasn't been reviewed against the new story yet. No model call happened;
  this is a hash comparison.
- Click `spec.md`, then **Ask model**. This is a *review*, not an
  automatic rewrite: the model either says nothing needs to change, or you
  get a diff card with **Accept**, **Reject**, and **Why?**.
- Click **Why?** on the diff (if you got one). This is a second, separate
  model call that explains the change in plain language — useful for
  spotting a change you don't actually agree with before you accept it.

This is propagation *by hand*: one hop, one review, one decision, every
time. Nothing here is automatic outside of the staleness marks.

## Level 3 — A real change, still by hand

Edit `story.md` to add: *"It also has a Reset button that sets the
counter back to zero."* Save it.

Now walk the chain yourself: `spec.md` (Ask model, review, Accept),
`decisions.md`, `arch.md`, `pseudo.md`, and finally the snippet Atelier
proposes for reset. Each step is its own diff. If several files pile up
waiting for you, click **Accept all** once instead of clicking through
each Accept individually.

Notice this only ever moved *downhill* — story toward code — because
that's the direction you chose to walk it in. Nothing stopped you from
starting at `decisions.md` instead and letting a change ripple back up
toward `story.md`; try that on a throwaway change and see what the model
proposes for the narrative.

## Level 4 — Let YOLO do Level 3 for you

First, undo Level 3: use the **Rollback** dropdown and pick the entry from
before you started (or just re-open the folder fresh). Make the same
story edit again, but this time click **YOLO**.

Watch:
- The status lamp turns blue while a file is being reconciled, and the
  tree shows a dashed border on whatever is currently queued.
- It walks story → spec → quality → decisions → arch → pseudo → tests →
  snippets, automatically, then runs the tests.
- When it finishes, check the **Rollback** dropdown — there are now two
  new entries, `[YOLO before]` and `[YOLO after]`.
- Click **Yeet**. This restores the `[YOLO before]` state in one click —
  the "I don't like where that went" button.

Do it again, and this time click **YOLO** a second time mid-run to stop
it, and see how it leaves a clean partial state rather than a half-applied
mess.

## Level 5 — Break something on purpose

Open `snippets/step.js` in the editor and introduce a bug — change
`n + 1` to `n + 2`. Save it.

- The blast-radius line under the editor now lists what needs review.
- Click **Run tests**. You'll see a failing test with a **Fix code**
  button next to it — click it for a single, targeted repair.
- Or, revert your bug and instead ask the model (via the ask box) to
  "make the increment button add two instead of one" — a real change —
  then run **YOLO**. Because failing-test fixes are only kept when they
  strictly reduce the number of failures, you can watch the ledger
  (`.atelier/ledger.md`, or the on-screen log) show a fix being tried,
  checked, and kept.

To see the safety net rather than just trust it: hand-edit
`snippets/step.js` into something that clearly won't fix the test (add an
unrelated console.log, say) and run **YOLO** again. Watch it try the
change, see the failure count didn't drop, and revert it — right there in
the log.

## Level 6 — Edit outside Atelier entirely

With Atelier still open, use a plain text editor to open `app/index.html`
in the project folder and change something directly in the code — say, the
text on a button. Save the file from your text editor.

Go back to Atelier and click **Check sync**. It notices the file on disk no
longer matches what it last wrote, and offers to **import the edit** into
the matching snippet (or discard it). This is the "you can always break
the abstraction, and Atelier helps you reconcile it" escape hatch — it
never silently overwrites work you did by hand.

## Level 7 — Non-functional requirements and decisions

Open `quality.md` and add a line: `Q4 [2] accessibility: buttons are
reachable by keyboard alone.` Save it, then run **Checks** — the
deterministic scoreboard (format, missing files, unit tests, app tests,
quality fitness tests, console, libraries) without touching the model.

Now open `decisions.md` and change D1's `chosen:` from `pure` to `oo`.
Save, then **Ask model** on the affected snippet. Atelier looks for
`snippets/step~oo.js` — a variant with the same public interface as
`step.js`, so the same tests still apply — and offers to generate it if
it's missing. This is how one architectural choice can change an
implementation style everywhere it applies, without touching the tests
that verify behaviour.

Try adding `avoid: Math.max` to that decision block and ask for the
variant again — you should see the model's own attempt get rejected if it
uses the forbidden pattern, before you ever see it.

## Level 8 — Bring in a real library

By default, Atelier's `atelier.md` rules forbid network calls and
`import`/`require` in your snippets — that's a deliberate default, not a
missing feature, meant to stop a small model from quietly reaching for
frameworks you never asked for. But sometimes you *do* want a library.
Here's the sanctioned way in, using **marked.js** (a small Markdown-to-HTML
parser) as the example — you'd do the same for any UMD-style library that
exposes itself as a plain global variable, such as pdf.js or a QR-code
generator.

1. Open `atelier.md` and add a line: *"May load the marked.js library via
   a script tag, for rendering notes as Markdown (allow network)."* The
   exact phrase **"allow network"** is what turns the mechanism on — leave
   it out and any library URL you add below is simply ignored (Checks
   will tell you so).
2. Look up the current version at <https://cdnjs.com/libraries/marked> and
   copy its pinned URL — don't use an unversioned link; you want a specific
   version, not "whatever is newest today."
3. Open `decisions.md` and add a new block:

   ```
   ## D3: Notes rendering
   drivers: S1
   options:
   - none: plain text notes
   - markdown: render notes as Markdown using marked, https://cdnjs.cloudflare.com/ajax/libs/marked/<version>/marked.min.js
   chosen: markdown
   stage: works
   why: nicer notes
   affects: ui
   ```

   The URL lives right inside the chosen option's own description — that's
   the entire syntax. Save the file.
4. Click **Checks**. The new **Libraries** row should now list that URL —
   confirmation that the mechanism picked it up and network is allowed.
5. Ask the model to add a notes feature to `snippets/ui.js` that calls the
   global `marked.parse(text)` to render Markdown. Because the library
   loads as a plain `<script>` tag before your code runs, your snippet can
   just use the global `marked` — no import needed, and the existing
   "no import" rule stays intact.

One thing worth knowing before you rely on this for anything real:
`marked.parse()` does **not** sanitize its output. Inserting arbitrary
Markdown into the page via `innerHTML` is fine for your own notes on your
own machine, but it's a real risk the moment the text could come from
someone else — you'd want something like DOMPurify on top before doing
that. This is, in miniature, exactly the kind of judgment call that "no
libraries unless the story asks" was quietly protecting you from.

## Level 9 — The reasoning dial

Open the **think:** dropdown. `auto` uses low effort for quick reviews and
high effort for anything that writes a file; `off` disables thinking
everywhere for speed; `high` forces it always. If your model is Granite
4.2 or Qwen3.8, try the same request (Level 3's reset button, say) under
`off` and then `high`, and compare both the wait time and the quality of
what comes back. There's no universally right setting — it's a genuine
trade you're making per project, and per how patient you are.

## Level 10 — Export and workflows

- Click **Packet**. It copies every file, in reading order, as one
  Markdown document — useful for pasting into a bigger model for a second
  opinion, or just for reading the whole project end to end.
- Open the **Workflows** menu and try **Tests for every claim**: it checks
  that every `S`-line in `spec.md` has a matching test, and proposes the
  missing ones.
- Click **Mark synced**. This snapshots the project, and writes a
  plain-language changelog entry — the one artifact in Atelier meant for a
  non-technical reader who wants to know what changed and why, without
  reading a diff.
  
## Mental model
By now you may have discovered it already, but here are the three things to keep in mind when trying to wrap your head around what Atelier does

### 1. Consistency

```text
story ↔ spec ↔ decisions ↔ architecture ↔ pseudo+snippets ↔ code
```

### 2. Evidence

```text
tests
checks
browser runtime
quality constraints
dependency graph
```

### 3. Control

```text
human review
accept/reject
YOLO
rollback
yeet
```


## What you've now touched

Open folder, Resume, Ping, Run tests, Check sync, Checks, Ask model,
Diagram, Accept / Reject / Why?, Accept all, YOLO, Yeet, Rollback,
Snapshot, Mark synced, the think dial, Workflows, Packet, and bringing in
your first external library — plus the tree's status dots, the
blast-radius line, and direct file edits both inside and outside the app.

The thing worth noticing across all ten levels: at no point did the tool
do anything to a file you couldn't see, undo, or explain. That's the
actual bet Atelier is making — not that the model is smart, but that the
loop around it is honest.

## Link to hardware
![](https://www.cnet.com/wp-content/uploads/sites/2/Codex-Micro-3.jpg?resize=1536,1157)
The button row in Atelier is inspired by the Codex Micro keypad and can be mapped to a macro keypad if you wish; each button uses its order on the display to map to a number key as access key, starting from YOLO. Press ALT + 1 to YOLO. Hovering shows the number.
