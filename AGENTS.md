# AGENTS.md

Guidance for AI coding agents and human contributors working on **Incident Chronology Studio**.

The product is a single HTML file that runs entirely in the browser and handles sensitive incident data. Its value depends on a few promises that must never be broken. Read section 1 before changing anything.

## 1. Security invariants (non-negotiable)

If a change would violate any rule below, do not make it. Say so and propose an alternative instead, even if the user's request seems to ask for it.

### 1.1 No network communication
* Do not add `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `navigator.sendBeacon`, service workers, `importScripts`, or dynamically injected `<script>`, `<link>`, `<img>` or `<iframe>` pointing anywhere.
* Do not add external CDNs, web fonts, analytics, error reporters, LLM or API endpoints, or any third-party script.
* The `<meta http-equiv="Content-Security-Policy">` tag must stay at least as strict as:
  `default-src 'none'; style-src 'unsafe-inline'; script-src 'unsafe-inline'; img-src data:; connect-src 'none'; form-action 'none'; base-uri 'none'`
  Never loosen it. Tighten it if you can (for example by removing inline event handlers, which are already forbidden).
* Icons and images, if ever needed, must be `data:` URIs.

### 1.2 No persistent browser storage
* Do not use `localStorage`, `sessionStorage`, `IndexedDB`, WebSQL, the Cache API, cookies, or `window.name` tricks.
* All application state lives in in-memory JavaScript variables (the `state` object). Persistence happens only through explicit user actions: Save JSON, Copy Markdown, Download .md, Download .csv.
* Rationale: incident notes contain IPs, hostnames and IoCs. Closing the tab must leave nothing behind on the workstation.

### 1.3 XSS safety
* Escape every user-controlled or imported string with `esc()` before it goes into `innerHTML`, or use `textContent`. This includes IDs, phases, milestones, category names and anything read from an imported file.
* Do not use inline event handlers (`onclick="…"`). Use `addEventListener` and event delegation with `data-*` attributes.
* Do not use `eval`, `new Function`, `setTimeout` with a string, or `document.write`.
* Never build HTML from imported JSON without going through the validation in 3.4.

### 1.4 Sample data must be fictional
* Use only reserved documentation values: IPs from RFC 5737 (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`), domains under `example.test`, `example.com` or `example.org`, and invented names.
* Never paste real incident data, real hostnames, real people or real indicators into samples, tests or docs.

## 2. Architecture rules

* **Single file.** Everything stays in `index.html` (HTML5, CSS3, vanilla ES6+). Do not add a build step, bundler, transpiler, framework or `package.json` dependency tree unless a maintainer explicitly asks.
* **Compatibility.** Must work when opened directly from `file://` and on GitHub Pages, in current Chrome, Edge, Firefox and Safari, with no server.
* **Keep it small and readable.** Prefer plain functions and a single `state` object. Avoid cleverness; this file is reviewed by security-minded people.
* **No hidden behaviour.** Nothing may run on a timer to send, save or sync data.

## 3. Data model

### 3.1 Categories (five)
| Key | Purpose | Category fields |
|---|---|---|
| `OBSERVED_EVENT` | Confirmed technical facts, evidence, IoCs | `evidence` |
| `DECISION` | Choices, hypotheses adopted, rationale | `actor`, `rationale` |
| `ACTION` | Operations performed and outcomes | `assignee`, `artifact` |
| `COMMUNICATION` | Notifications and updates | `recipient`, `channel` |
| `CONTEXT_NOTE` | Unconfirmed information and possibilities | `basis`, `verify` |

Do not add, remove or rename categories without a maintainer's decision. If a category changes, update `CATS`, `FIELDS`, `IDF`, `SEC`, `CATKEY`, `CLS`, the form sections, the filter chips, the CSS, both i18n dictionaries, the Markdown report, the CSV export, the samples, `README.md` and this file.

**Fact vs. possibility is a core idea.** `CONTEXT_NOTE` must always stay visually distinct and must appear in its own "Unconfirmed" section of the report, never mixed into confirmed facts. Promoting a note ("Confirm → Observed Event") opens the edit form in Observed Event mode with `basis` pre-filled as `evidence`; nothing changes until the user saves.

### 3.2 Time handling
* `time` is always required, in `YYYY-MM-DDTHH:mm` form, and is the sort key.
* `timeMode` is `exact`, `range` (uses `timeEnd`) or `text` (uses `timeText` as the displayed label). All three must keep working.
* A timezone label (`meta.tz`) is free text and is shown in the report. Do not silently convert times.
* Durations (see 3.5) are calculated from the times exactly as entered, on the assumption of one common timezone. Parse them with `Date.UTC` on the date-time parts, never with the local-time `Date` parser, so the result does not depend on the viewer's timezone or daylight saving time.

### 3.3 Phases
Phase IDs are `p0` to `p6` (Occurrence/Latency, Detection, Triage, Containment, Forensics, Eradication/Recovery, Communication). Store IDs, never display strings; labels come from the i18n dictionaries. Do not claim that the phase list formally implements NIST SP 800-61 Rev. 3, the ENISA guide or any other standard unless a real mapping has been added.

### 3.4 JSON schema and import validation
Export format: `{ app, schemaVersion: 3, meta: { title, commander, tz }, items: [...] }`. Do not export UI state.

Import must remain defensive:
* Parse with `JSON.parse` only. Require `items` to be an array of at most 5000 entries.
* Allow-list `category`, `timeMode`, `phase` and `milestone`; validate `time` and `timeEnd` against the exact date-time pattern.
* Coerce every other value to a string and cap its length (summary 5000, category fields 1000, `timeText` 200, meta fields 200 or less).
* Drop invalid entries and regenerate every `id`. Never trust imported IDs.
* Keep accepting legacy files: schema version 2 (no `milestone`, which becomes empty) and schema version 1 (`eventTime`, English phase names).
* On any failure, show an error and leave the current state untouched. Ask for confirmation before replacing existing entries.

When the schema changes, raise `schemaVersion`, keep the older versions importable, and update this section and the README.

### 3.5 Milestones
A milestone is an optional tag (`milestone`) on any entry. It does not replace the category or the phase.

* Keys, in lifecycle order (`MS`): `occurred`, `detected`, `declared`, `contained`, `eradicated`, `recovered`, `closed`. An empty string means no milestone.
* Completion milestones (`DONE`): `contained`, `eradicated`, `recovered`, `closed`. For these, an entry in `range` mode counts at its `timeEnd`; every other milestone uses `time`.
* If several entries carry the same milestone, the earliest one is used.
* The report shows the milestones and the intervals listed in `PAIRS`, and only where both ends exist. A negative interval is shown as "check order", never as a negative number. An interval that involves a `text`-mode entry is marked "(estimated)".
* Labels come from the `ms.*` keys of both dictionaries. Do not store display strings.
* If you add, remove or rename a milestone, update `MS`, `DONE` and `PAIRS` as needed, the milestone select, both i18n dictionaries (`ms.*`), the card tag, `cleanItem`, the Markdown report, the CSV export, the samples, `README.md` and this file.
* Durations are a reporting aid. Do not present them as a performance score or a target.

### 3.6 Export formats
* **Markdown report.** Sections, in order: Key Milestones & Durations, Chronological Timeline, Confirmed Observed Events & IoCs, Context & Potential Notes (Unconfirmed), Decisions, Response Actions, Stakeholder Communications. Escape `|` and line breaks in table cells.
* **CSV.** UTF-8 with a BOM, CRLF line endings, every cell in double quotes with `"` doubled. A cell that begins with `=`, `+`, `-`, `@`, a tab or a carriage return must get a leading `'` so a spreadsheet does not run it as a formula. Keep this protection when changing the export. Column headers come from the report language.
* Both exports are produced in memory and handed to the user through a download or the clipboard only. They never go anywhere else.

## 4. Internationalisation

* English (`en`) is the default; Japanese (`ja`) is the second language.
* Every visible string comes from the `i18n` dictionary through `data-i18n` (text) or `data-i18n-ph` (placeholder). No hard-coded UI text in HTML or JS templates. The only exception is the names of the two languages in the report language selector and the language toggle, which are always shown in their own language.
* Any new key must exist in **both** `i18n.en` and `i18n.ja`. The fallback chain is a safety net, not a licence to skip translation.
* The UI language (`lang`, read with `t()`) and the report language (`rlang`, read with `tr()`) are separate. The report language follows the UI language when the UI language changes, and can then be set independently with the report language selector.
* The Markdown report and the CSV use `tr()` only, so a report is written in one language and never mixes two. Report strings live in the `r.*` keys, plus the shared `ms.*`, `c.*`, `ph.*` and category keys. User-entered content is never translated.
* Sample incidents exist per language and are loaded according to the current UI language.

## 5. UI and accessibility

* Keep the interface usable on narrow screens and with the keyboard alone. Buttons are real `<button>` elements, form controls have `<label for>`, toggles use `aria-pressed`, and the modal handles Escape and focus.
* Destructive actions (delete, clear all, replace on import) ask for confirmation. Closing the tab with unsaved entries triggers the browser's warning.
* Do not rely on colour alone to convey category, milestone or state. The milestone tag carries a flag symbol and its text label.

## 6. Verification before you finish

Run these from the repository root and fix anything unexpected.

```sh
# Must print nothing (no network, no persistence, no dynamic code, no URLs):
rg -n -i "fetch\(|XMLHttpRequest|WebSocket|EventSource|sendBeacon|importScripts|serviceWorker|localStorage|sessionStorage|indexedDB|openDatabase|document\.cookie|\beval\(|new Function|document\.write|https?://" index.html

# Must print nothing (no inline event handlers):
rg -n -i "\son(click|change|input|load|error|submit)\s*=" index.html
```

Then check by hand (or with a throw-away browser script that you do not commit):

- [ ] The page opens from `file://` with an empty console and no network requests.
- [ ] Add, edit, delete and filter work for **all five** categories and all three time modes.
- [ ] Promote a note to an Observed Event: cancel keeps the note; save converts it, keeping the timestamp (and the milestone, if any).
- [ ] Save JSON → reload → Load JSON restores entries, milestones and meta. Schema version 1 and 2 files still load.
- [ ] Malformed JSON, a wrong schema and a hostile file (script tags, `onerror`, odd IDs, phases and milestones) are rejected or neutralised, and the existing state is untouched.
- [ ] Milestones: tag entries and check the card tags and the Key Milestones & Durations section. The earliest entry wins, a range counts at its end for completion milestones, a `text`-mode time is marked as estimated, an out-of-order pair shows "check order", and a report with no milestones still renders.
- [ ] Markdown report: all sections present, `|` and line breaks escaped, tables well-formed, unconfirmed notes in their own section.
- [ ] Report language: English and Japanese reports are each fully in one language, including headers, milestone names, notes and footer. Changing the UI language moves the report language with it, and the selector can then override it.
- [ ] CSV: opens in a spreadsheet with Japanese text intact, one row per entry, and a summary that starts with `=` is not run as a formula.
- [ ] Both languages render fully (UI, cards, phase names, milestone names, samples, empty state, confirmations).
- [ ] No new i18n keys are missing from either dictionary.

## 7. Documentation

When behaviour, schema or security properties change, update `README.md` in the same change. Do not describe features that do not exist, and do not overstate security: state limits plainly (see "Limits of this design" in the README).

## 8. Working style

* Make the smallest change that solves the problem; do not refactor unrelated code.
* Explain security-relevant decisions in the change description.
* If a request conflicts with section 1, stop and ask instead of working around it.
