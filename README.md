# 🛡️ Incident Chronology Studio

**A browser-only incident chronology and Markdown report generator for security responders. No server, no storage, no network.**

[Live demo](https://lex0xf0c.github.io/incident-chronology-studio/) · English / 日本語 · MIT License

<p align="center">
  <img src="assets/screen.png" alt="Screenshot of Incident Chronology Studio" width="720">
</p>

Incident Chronology Studio is a single HTML file that helps SOC analysts, CSIRT members and incident commanders keep a clean timeline while an incident is still unfolding. It separates **facts**, **decisions**, **actions**, **communications** and **unconfirmed possibilities**, and turns them into a structured Markdown report when you are done.

Everything runs inside your browser tab. Nothing is sent anywhere, and nothing is written to the browser's storage.

---

## Why a chronology?

During an incident, memory fails and chat history scatters. A good chronology answers, afterwards and under pressure alike:

* **What happened, and when?** Including activity that began *before* anyone noticed.
* **Who decided what, and on what grounds?** Accountability and auditability.
* **What is fact and what is a guess?** Teams act better when the two are never mixed.
* **What was done, by whom, with what result?** The basis for the after-action report and lessons learned.

This tool keeps those answers in one place, with as little friction as a whiteboard.

## Features

* **Five entry categories**
  | Category | Use it for |
  |---|---|
  | 💻 Observed Event | Confirmed technical facts: log lines, alerts, IoCs, hashes |
  | 🧠 Decision | Analyst or commander choices, with the decision maker and rationale |
  | 🛠️ Response Action | Isolation, collection, eradication, recovery: who did what, and the outcome |
  | 📢 Communication | Notifications and updates: recipient and channel |
  | ❓ Context & Potential Note | Unconfirmed information. Park it here, then **Confirm → Observed Event** once verified |
* **Flexible time input**: exact timestamp, a start → end range, or free text such as `Nov 5 night` for retroactive findings. A timestamp is always kept as the sort anchor. A timezone field is recorded in the report.
* **Incident phases (0–6)**: Occurrence / Latency (pre-detection), Detection, Triage, Containment, Forensics, Eradication / Recovery, Communication.
* **Edit, delete, sort and filter** timeline cards at any time.
* **Markdown report** with a chronological timeline table plus one section per category. Unconfirmed notes get their own clearly labelled section. Copy it or download it as `.md`.
* **JSON save / load** to continue later or hand over to the next shift.
* **Built-in sample incidents** (English and Japanese) so you can see how it works in one click.
* **English and Japanese UI**, switchable at runtime.
* **Unsaved-work warning** before closing or reloading the tab.

### The note → fact workflow

Early in an incident most information is shaky. Record it as a **Context & Potential Note** with its basis and what would confirm or rule it out. When it is verified, press **Confirm → Observed Event**: the entry opens in edit mode as an Observed Event, with the basis pre-filled as evidence. Add the confirming evidence and save. The original timestamp is kept, and the report never presents an unconfirmed note as a fact.

## Quick start

No build tools, no dependencies, no server.

1. Download `index.html` (or clone the repository).
2. Open it in any modern browser: Chrome, Edge, Firefox or Safari.
3. Click **Load sample** to explore, or start adding entries.
4. Before closing the tab, click **Save JSON**. Your data lives only in memory.

You can also host `index.html` on GitHub Pages or any static web server. For sensitive incidents, download the file and open it locally instead: you then depend on no hosting provider at all, and it works fully offline and on air-gapped machines.

## Privacy and security design

The core promise: **incident data never leaves your machine, and never lingers on it unless you export it.**

| Principle | How it is enforced |
|---|---|
| No network communication | A `Content-Security-Policy` meta tag sets `default-src 'none'` and `connect-src 'none'`. The browser itself blocks requests to other origins, even if code were changed by mistake. |
| No external resources | All HTML, CSS and JavaScript live in one file. No CDN, fonts, analytics or third-party scripts. |
| No persistent browser storage | No `localStorage`, `sessionStorage`, `IndexedDB` or cookies. Closing or reloading the tab discards all unsaved data, so hostnames, IPs and IoCs do not remain on the workstation. |
| User-controlled export | Data leaves the page only when you click Save JSON, Copy Markdown or Download .md. |
| XSS defence | Every user-supplied value is HTML-escaped before it reaches the DOM. No `eval`, no `new Function`, no inline event handlers. |
| Safe JSON import | Imported files are parsed with `JSON.parse` only. Categories, phases and timestamps are validated against allow-lists, strings are length-limited, unknown entries are dropped and IDs are regenerated. |

### Limits of this design

This tool protects the page, not the machine around it. Out of scope:

* A compromised workstation, operating system or browser.
* Malicious browser extensions or keyloggers.
* Exported JSON and Markdown files. They are plain text, so protect them as you would any incident record (encrypted storage, appropriate TLP handling).
* The integrity of the file you download. If you use the hosted demo, verify the file against the repository, or build your trust once and keep your own copy.

A `<meta>` CSP cannot set `frame-ancestors`. If you host the page yourself and need clickjacking protection, add that header at the web server.

## Data format

**JSON (schema version 2)**

```json
{
  "app": "incident-chronology-studio",
  "schemaVersion": 2,
  "meta": { "title": "…", "commander": "…", "tz": "UTC+01:00" },
  "items": [
    {
      "id": "…",
      "category": "OBSERVED_EVENT",
      "timeMode": "exact",
      "time": "2026-11-05T09:12",
      "timeEnd": "",
      "timeText": "",
      "phase": "p1",
      "summary": "EDR alert: suspicious beaconing from FIN-PC-014.",
      "evidence": "EDR alert #48213"
    }
  ]
}
```

* `category`: `OBSERVED_EVENT`, `DECISION`, `ACTION`, `COMMUNICATION` or `CONTEXT_NOTE`.
* `timeMode`: `exact`, `range` or `text`. `time` (format `YYYY-MM-DDTHH:mm`) is always the sort anchor.
* `phase`: `p0` to `p6`.
* Category-specific fields: `evidence` · `actor`, `rationale` · `assignee`, `artifact` · `recipient`, `channel` · `basis`, `verify`.
* Files from earlier versions (`eventTime`, English phase names) are still accepted.

**Markdown report**: header (title, commander, timezone, generation time in UTC), then `1. Chronological Timeline`, `2. Confirmed Observed Events & IoCs`, `3. Context & Potential Notes (Unconfirmed)`, `4. Decisions`, `5. Response Actions`, `6. Stakeholder Communications`. The report text is currently in English regardless of the UI language.

## Scope and status

This is a note-taking and reporting aid. It is **not** a case management system, a SIEM, or a replacement for your ticketing and evidence-handling procedures. It is a young project: expect rough edges, and review the generated report before you rely on it.

The phase list follows the general shape of common incident lifecycles (see the NIST SP 800-61 and ENISA incident management guidance) but is **not** a formal implementation of either. Ideas under consideration include a Post-incident / lessons-learned phase, NIST CSF 2.0 function tags, a Traffic Light Protocol marking for reports, and a translated report.

## Development

* One file: `index.html`. Vanilla HTML5, CSS3 and ES6+. No build step, bundler or package manager.
* Contributors and AI coding agents: read [`AGENTS.md`](AGENTS.md) first. It defines the security invariants that every change must keep.
* All sample data is fictional and uses reserved documentation addresses and names only.

## License

MIT. See [`LICENSE`](LICENSE).
