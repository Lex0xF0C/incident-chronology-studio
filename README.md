<p align="center">
  <img src="assets/screen.png" alt="Secriptorium Logo" width="500" height="500">
</p>

# 🛡️ Incident Chronology Studio

> **Browser-Only, Zero-Server Incident Chronology & Markdown Report Generator for Security Responders**

[Live demo](https://lex0xf0c.github.io/incident-chronology-studio/)

Incident Chronology Studio is a lightweight, privacy-first web application designed for Security Operations Center (SOC) analysts, CSIRT members, and emergency responders to record timeline events, track decision rationales, and generate structured Markdown incident reports during cyber security incident handling.

Built specifically for high-security environments, the application operates **100% locally within the browser** with **zero external server communication, zero backend infrastructure, and zero tracking analytics**.

---
## What is an Incident Chronology and Why is it Essential?

### 1. What is an Incident Chronology?
In cyber incident response, a chronology is a structured, multidimensional timeline that records the entire lifecycle of an incident across different operational layers:
* **Event Time (System Occurrence):** The exact time when an event physically or logically happened within the target system or network.
* **Detection Time (Observation):** The moment when security monitoring tools or human analysts first detect the anomaly.
* **Attribution & Decision Time:** The timestamp and rationale of when responders reach a critical conclusion or make a strategic choice.
* **Action & Communication Milestones:** The precise completion time of containment procedures, forensic evidence collection, or stakeholder notifications.

### 2. Why is it Vital for Cyber Incident Response?
Cybersecurity incidents are often complex and require weeks or months to fully remediate. Maintaining a robust chronology directly supports effective incident management by providing:
* **Unambiguous Accountability & Governance:** Recording who made specific decisions, when, and based on what evidence ensures complete transparency and auditability during high-pressure crises.
* **Dwell Time Analysis:** Separating pre-detection occurrences from detection triggers helps measure the duration of attacker stealth and improves future threat-hunting strategies.
* **Continuous Improvement & Lessons Learned:** Clearly separating confirmed facts from working hypotheses accelerates root-cause analysis, enabling teams to share critical lessons and strengthen overall organizational resilience.

## 🔒 Security & Privacy Architecture

The core design philosophy of Incident Chronology Studio is **absolute data isolation and zero external exposure**.

```
[ User Input / Logs ]  ───►  [ Browser Memory (JavaScript State) ]
                                      │
                         ┌────────────┴────────────┐
                         ▼                         ▼
              [ Markdown Report Export ]   [ JSON State Export ]
                         │                         │
                         └────────────┬────────────┘
                                      ▼
                        (Saved strictly to Local Disk)
```

### Core Principles

* **Local-First & Client-Side Processing**: All processing occurs strictly within the user's local web browser execution context. No data is ever transmitted to external endpoints or cloud services.
* **No Backend Services & No User Accounts**: Operates without server-side APIs, authentication systems, database persistence, or cloud sync.
* **No Telemetry or Tracking**: Contains zero analytics scripts, tracking pixels, telemetry probes, or third-party monitoring SDKs.
* **No Persistent Browser Storage**: The application intentionally **does not use** `LocalStorage`, `SessionStorage`, `IndexedDB`, or `Cookies`. Closing or refreshing the page instantly purges all unsaved state from browser memory. This deliberate design prevents residual sensitive incident data (such as internal IP addresses, hostnames, IoCs, or analyst notes) from remaining on workstation storage after incident closure.
* **Explicit User-Controlled Export**: Data retention occurs exclusively when the user manually triggers an export as a JSON state backup or copies/saves the generated Markdown report.

### Security Controls

* **XSS Prevention**: Strict HTML escaping on all user-supplied text before rendering elements in the DOM.
* **Content Security Policy (CSP)**: Designed to run under restrictive CSP headers with no external script or style sources.
* **Safe Client-Side Import**: Defensive JSON schema validation during data import without using dynamic code evaluation (`eval`).
* **Zero External Dependencies**: Self-contained single-file architecture (`index.html`) using Vanilla HTML5, CSS3, and JavaScript (ES6+) with no external NPM packages or CDN resources.

### Threat Model & Boundaries

* **Designed to Prevent**: Accidental data leakage to cloud services, third-party analytics exposure, compliance violations under strict data protection frameworks (e.g., GDPR, NIS2, HIPAA), and unencrypted data lingering in browser storage.
* **Out of Scope (User Responsibility)**:
  * Compromised local endpoints or operating systems.
  * Malicious browser extensions or keyloggers active on the user's machine.
  * Security of manually exported JSON and Markdown files saved to the local filesystem.

---

## ✨ Key Features

1. **4-Category Incident Event Model**:
   * 💻 **Observed Event**: Objective technical facts, log events, telemetry, and Indicators of Compromise (IoCs).
   * 🧠 **Decision**: Human analyst decisions, working hypotheses, attribution notes, and rationale.
   * 🛠️ **Response Action**: Tactical mitigation steps, endpoint isolations, forensic acquisitions, and completion milestones.
   * 📢 **Communication**: External and internal stakeholder notifications, legal/PR briefs, and status updates.

2. **NIST SP 800-61 Rev 3 Lifecycle Alignment**:
   * Supports structured tagging across full incident phases:
     * `0. Occurrence / Latency` (Pre-detection attack activity & dwell time tracking)
     * `1. Detection / Alert`
     * `2. Triage / Attribution`
     * `3. Containment`
     * `4. Forensics / Acquisition`
     * `5. Eradication / Recovery`
     * `6. Communication`

3. **Flexible Time Frame Inputs**:
   * **Exact Timestamp**: Minute-precise ISO timestamps.
   * **Range**: Duration tracking (`Start Time ➔ End Time`).
   * **Flexible Text**: Natural language time estimates (e.g., `Nov 5 Night ~ Nov 6 Morning`) for retroactive forensic findings.

4. **Automated Markdown Report Generation**:
   * Instant 1-click generation of structured Markdown post-mortem reports categorized into 4 executive sections.

5. **Multi-Language (i18n) UI**:
   * English default interface with seamless runtime toggle to Japanese (`日本語`).

---

## 🚀 Quick Start

Because the studio is built as a single, self-contained `index.html` file, deployment requires no build tools, Node.js, or server setup.

### Option 1: Direct File Launch (Air-Gapped / Offline)
1. Download or clone the repository.
2. Open `index.html` directly in any modern web browser (Chrome, Firefox, Edge, Safari).
3. Begin recording incident events immediately.

### Option 2: GitHub Pages / Internal Static Server
* Host `index.html` on GitHub Pages, an internal Nginx/Apache web server, or an air-gapped intranet web share.

---

## 📂 Data Export & Backup

To preserve your work before closing the browser tab:

* **Save State**: Click **Save JSON** (or **Export JSON**) to download the raw state file (`incident-chronology-<timestamp>.json`).
* **Restore State**: Click **Load JSON** (or **Import JSON**) to restore a previously saved JSON session.
* **Copy Report**: Click **📋 Copy Markdown** to copy the formatted report directly to your clipboard.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.
