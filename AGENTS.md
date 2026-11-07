# 🤖 AGENTS.md — AI Agent Guidance & Coding Standards

> **Mandatory Guidelines for AI Agents, Copilots, and Automated Contributors Working on Incident Chronology Studio**

This document specifies the strict architectural constraints, security invariants, and coding standards that **every AI Agent or automated tool MUST follow** when inspecting, modifying, or extending this repository.

---

## ⛔ 1. Core Security Invariants (NON-NEGOTIABLE)

The primary requirement of this repository, as specified in `securitydesigin.txt`, is **absolute privacy, local-first browser execution, and zero external data transmission**.

When modifying or adding code to this project, Agents **MUST NOT**:

1. **NEVER Add Network Communication or External Calls**:
   * Do NOT add any `fetch()`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `navigator.sendBeacon()`, or dynamic script injections.
   * Do NOT integrate third-party APIs, LLM server endpoints, external CDNs (e.g., Bootstrap, Tailwind, FontAwesome, Google Fonts, i18next), or analytics/tracking tools (e.g., Google Analytics, Sentry, Mixpanel).
   * All CSS, JS, fonts, and assets MUST remain inline or strictly local.

2. **NEVER Introduce Persistent Browser Storage**:
   * Do NOT use `localStorage`, `sessionStorage`, `indexedDB`, `WebSQL`, or `Cookies`.
   * All application data MUST reside exclusively in **in-memory JavaScript variables** during an active session.
   * Data persistence is strictly manual and user-driven via JSON file downloads and uploads.
   * *Rationale*: In-memory execution ensures all sensitive incident notes (IoCs, hostnames, IP addresses) vanish immediately when the tab is closed or refreshed, preventing residual sensitive artifacts on local workstation disks.

3. **NEVER Compromise Client-Side XSS Protection**:
   * ALWAYS sanitize and HTML-escape all user-supplied content before inserting it into the DOM.
   * Use `escapeHtml()` helpers or `textContent` / `innerText` bindings.
   * Do NOT use `innerHTML` with unsanitized user inputs.
   * Do NOT use `eval()`, `new Function()`, or dynamic script execution when parsing imported JSON files.

---

## 🏗️ 2. Architectural & Codebase Conventions

1. **Single-File Pure Frontend Stack**:
   * The application is built as a self-contained `index.html` file using standard **HTML5, CSS3, and Vanilla JavaScript (ES6+)**.
   * Do NOT introduce build steps, bundlers (Webpack, Vite, Rollup), transpilers (Babel, TypeScript), or NPM dependency trees unless explicitly instructed by a core maintainer.

2. **i18n (Multi-Language) Standard**:
   * **Default Language**: English (`en`).
   * **Secondary Language**: Japanese (`ja`).
   * All UI text elements MUST use data attributes (`data-i18n="key"` for text, `data-i18n-ph="key"` for placeholders).
   * When adding new UI elements, update BOTH `i18n.en` and `i18n.ja` key-value pairs in the dictionary object inside `index.html`.

3. **Data Model & Schema Integrity**:
   * Maintain the **4-Category Schema**:
     * `OBSERVED_EVENT`: Technical facts, evidence, log references, IoCs.
     * `DECISION`: Analyst choices, hypotheses, rationale, decision-maker attribution.
     * `ACTION`: Operations, isolation steps, forensic collection, operator attribution, outcomes.
     * `COMMUNICATION`: Stakeholder updates, legal/PR reports, channels, recipients.
   * Maintain alignment with **NIST SP 800-61 Rev 3** incident lifecycle phases (Phases 0 through 6).

4. **Flexible Timestamp Handling**:
   * Preserve support for Exact timestamps (`ISO 8601`), Range durations (`Start ➔ End`), and Flexible Text representations (`Nov 5 Night`).

---

## 🧪 3. Agent Verification Checklist

Before finalizing any pull request or code change, AI Agents MUST verify the following checklist:

- [ ] **Zero External Requests**: Confirmed that `rg -i "fetch|xmlhttprequest|websocket|http://|https://" index.html` returns NO network telemetry or external resources.
- [ ] **Zero Persistence APIs**: Confirmed that `localStorage`, `sessionStorage`, `indexedDB`, and `cookie` are NOT referenced anywhere in the script.
- [ ] **Complete i18n Mapping**: All newly added DOM labels have corresponding entries in both `i18n.en` and `i18n.ja`.
- [ ] **Strict HTML Escaping**: All user-controlled fields (`summary`, `evidence`, `actor`, `rationale`, `artifact`, `recipient`) pass through HTML escaping before DOM insertion.
- [ ] **JSON Import Safety**: Importing custom JSON files validates array structures and does not execute arbitrary script content.
- [ ] **Markdown Output**: Generated Markdown correctly reflects all 4 structured sections without syntax errors.

---

## 📌 Summary for AI Prompting

If you are an AI assistant tasked with generating code for this repository, adopt this persona:

> "You are an expert security software architect focused on air-gapped, privacy-first client-side web tools. You write clean, robust Vanilla JS and HTML5 code with zero external dependencies, zero persistent storage, and strict XSS safety."
