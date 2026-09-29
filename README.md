# Wellness Tracker — Demo

> **Demo only. Use fictional data only.**
> This tool is a learning prototype for wellness professionals.
> It stores nothing outside your own browser and offers no medical advice, diagnosis, or treatment guidance of any kind.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Summary](#2-architecture-summary)
3. [Opening and Previewing the App](#3-opening-and-previewing-the-app)
4. [How Demo Entries Are Stored and Erased](#4-how-demo-entries-are-stored-and-erased)
5. [Data Privacy and Safety Notice](#5-data-privacy-and-safety-notice)
6. [Maintenance Checklist](#6-maintenance-checklist)

---

## 1. Project Overview

Wellness Tracker is a **single-file, browser-only learning prototype** built for wellness professionals who want to explore a simple self-logging interface. It is not a clinical tool, not a medical device, and not suitable for recording real patient information.

Users log three personal metrics per day:

| Field | Range | Type |
|-------|-------|------|
| **Mood** | 1 (Poor) – 5 (Great) | Integer, select |
| **Sleep** | 0 – 24 hours | Decimal, 0.5 steps |
| **Water** | 0 – 20 glasses | Integer |

A summary card shows averages across the **seven most recent entries**. All data lives exclusively in the browser's `localStorage` under the key `wellness-tracker-demo-v1`. There is no backend, no network call, no account, and no data ever leaves the device.

---

## 2. Architecture Summary

```
index.html          ← the entire application: one file, no dependencies
│
├── <head>
│   ├── <meta charset> / <meta viewport>
│   ├── <meta http-equiv="Content-Security-Policy">   ← blocks all network
│   └── <style>                                        ← all CSS, inline
│
├── <body>
│   ├── #banner              Sticky "Demo only" notice (always visible)
│   ├── #form-section        Entry form — date, mood, sleep, water
│   │                        Save Entry / Update Entry / Cancel buttons
│   ├── #summary-section     Personal Demo Trends card (last 7 entries)
│   │   ├── stat-count       Number of entries in the window (max 7)
│   │   ├── avg-mood         Average mood score
│   │   ├── avg-sleep        Average sleep hours
│   │   └── avg-water        Average water glasses
│   ├── #log-section         Entry log, newest first
│   │   ├── .entry-row       Date · mood chip · sleep chip · water chip
│   │   └── Edit / Delete    Per-row actions; Delete requires confirmation
│   └── <footer>             localStorage privacy notice
│
└── <script>                 All JS, inline — no imports, no frameworks
    ├── getEntries()         Read + parse localStorage
    ├── saveEntries(arr)     Serialise + write localStorage
    ├── migrateOldKey()      One-time migration from legacy key name
    ├── validate()           Field-level validation; returns error map
    ├── showErrors(errors)   Inline error messages + aria-invalid
    ├── handleSave()         Validate → build entry → push → persist → render
    ├── handleUpdate()       Validate → mutate entry → persist → render
    ├── deleteEntry(id)      Confirm → filter → persist → render
    ├── eraseAll()           Confirm → clear array → persist → render
    ├── enterEditMode(id)    Populate form → scroll to form
    ├── exitEditMode()       Reset form → remove edit-mode class
    ├── renderSummary()      Compute 7-entry averages → update DOM
    ├── renderLog()          Build entry rows via createElement (no innerHTML)
    ├── renderAll()          renderSummary() + renderLog()
    └── seedDemoData()       Pre-load 3 fictional entries on first visit
```

### Key design decisions

- **No `innerHTML` for user-supplied values.** All entry data is written with `.textContent` or built via `createElement`. Only fixed static strings use HTML markup.
- **No external requests.** A `Content-Security-Policy` meta tag sets `default-src 'none'` and permits only `'self'` scripts and styles, preventing any accidental outbound call.
- **Keyboard accessible.** Every interactive element is a native `<button>` or form control with a visible `<label>`. Focus rings are preserved in both standard and Windows High Contrast modes.
- **320 px safe.** The summary grid uses a 2 × 2 tile layout at narrow widths; all cards are single-column, max 560 px centered.

---

## 3. Opening and Previewing the App

The app requires no build step, no package manager, and no server for local use.

### Option A — Open directly in a browser (simplest)

```bash
# macOS
open index.html

# Linux
xdg-open index.html

# Windows (PowerShell)
Start-Process index.html
```

Or drag `index.html` onto any browser tab. Because there are no network calls and no imports, the file runs fully offline from the filesystem.

### Option B — Bob desktop Live Preview

1. Open `index.html` in the editor pane.
2. Click the **Show Preview** icon (magnifier, top-right toolbar), or press:
   - macOS: `Shift ⌘ V`
   - Windows / Linux: `Shift Ctrl V`

### Option C — Local dev server (avoids any `file://` quirks)

```bash
# Node / npx
npx --yes serve . --listen 3000
# then open http://localhost:3000

# Python 3
python3 -m http.server 8080
# then open http://localhost:8080
```

### Option D — Static host (fictional data demo only)

You may host `index.html` on a static file host (GitHub Pages, Netlify, etc.) **for demonstration purposes with fictional data only**. Because localStorage is browser-local and unencrypted, hosting the file publicly does not expose stored data — each visitor's browser holds only their own entries. However, **do not instruct real users to log real personal or health information** on any hosted instance.

> 💡 No build step is needed for any option. The file has no dependencies.

---

## 4. How Demo Entries Are Stored and Erased

### Storage mechanism

All entries are stored as a JSON array in the current browser's `localStorage` under the key:

```
wellness-tracker-demo-v1
```

Each entry object has this shape:

```jsonc
{
  "id":    "uuid-v4-string",       // crypto.randomUUID()
  "date":  "YYYY-MM-DD",           // ISO date, no future dates
  "mood":  3,                      // integer 1–5
  "sleep": 6.5,                    // float 0–24, step 0.5
  "water": 8                       // integer 0–20
}
```

Data persists across page refreshes and browser restarts **within the same browser profile on the same device**. It does not sync across devices, browsers, or user accounts.

### Erasing entries

| Action | How |
|--------|-----|
| **Delete one entry** | Click **Delete** on any row → confirm the dialog |
| **Erase all entries** | Click **Erase all demo entries** (top of log) → confirm the dialog |
| **Browser-level wipe** | Open DevTools → Application → Local Storage → delete the key `wellness-tracker-demo-v1` |
| **Clear all site data** | Browser Settings → Privacy → Clear browsing data → check "Cookies and site data" |

After erasing all entries the summary card shows "No entries yet" and the three-entry seed data will **not** re-appear (seeding runs only once, on first load when the key is absent).

---

## 5. Data Privacy and Safety Notice

### localStorage is not encrypted

`localStorage` is plain-text storage held by the browser on the local filesystem. It is:

- ❌ **Not encrypted** — any process or user account with filesystem access can read it
- ❌ **Not access-controlled** — any JavaScript running on the same origin can read it
- ❌ **Not suitable for patient information** — it does not meet HIPAA, GDPR health-data, or any other clinical data-handling standard
- ❌ **Not backed up** — clearing browser data permanently destroys all entries
- ✅ **Browser-local only** — data never leaves the device; no server ever receives it

### No medical advice

This application does **not**:

- Diagnose any condition
- Recommend any treatment, medication, or clinical intervention
- Interpret results or flag health concerns
- Replace consultation with a qualified health professional

The summary figures (average mood, sleep, water) are **personal demo trends for learning purposes only**. They are explicitly labelled as such inside the application.

### Who this is for

This prototype is intended for **wellness professionals exploring interface patterns** for a potential future product. It must not be distributed to end users as a health-monitoring tool without appropriate clinical review, regulatory assessment, and proper data-security architecture.

---

## 6. Maintenance Checklist

Use this checklist whenever you change a field, its range, or its validation logic. Run every item in order — a failure in an early step can silently corrupt later steps.

### A. Changing a field name or adding a new field

- [ ] Add the input element to `#form-section` with a matching `<label for="…">` and `aria-describedby`
- [ ] Add the field ID to the `validate()` function with appropriate min/max/required rules
- [ ] Add the field to `showErrors()` and `clearErrors()` field arrays
- [ ] Add the field to `readForm()` return object
- [ ] Add the field to `resetForm()` clear block
- [ ] Add the field to the `handleSave()` entry object constructor
- [ ] Add the field to the `handleUpdate()` map callback
- [ ] Add the field to `enterEditMode()` population block
- [ ] Update `renderLog()` to display the new field in the chip row
- [ ] Update `renderSummary()` to include the new field's average (if applicable)
- [ ] Update the `seedDemoData()` entries to include valid values for the new field
- [ ] Update the data shape table in this README

### B. Changing an input range (e.g. mood 1–10, water 0–30)

- [ ] Update the `<input min max step>` or `<select>` option values in the HTML
- [ ] Update the matching guard in `validate()` (`< 0 || > N`)
- [ ] Update the hint text (`<div class="field-hint">`) to reflect the new range
- [ ] If the mood scale changes, update `MOOD_LABELS` map in the JS constants block
- [ ] Update `seedDemoData()` so seeded values fall within the new range
- [ ] Update the field table in Section 1 of this README

### C. Retest checklist (run after every change)

- [ ] **Save** — fill all fields with fictional values → Save Entry → entry appears in log → refresh page → entry persists
- [ ] **Validation** — submit with each field blank or out of range → inline error appears for each → no entry is added
- [ ] **Reload** — close and reopen the tab → all entries still present, summary recalculates correctly
- [ ] **Averages** — add 8 entries with known values → confirm summary shows averages of the 7 most recent only, count reads "7 of 7"
- [ ] **Edit** — click Edit on any entry → form pre-fills → change a value → Update Entry → log reflects new value → summary recalculates
- [ ] **Delete** — click Delete → confirm dialog appears → entry removed → summary count decrements → if last entry, empty state shown
- [ ] **Erase all** — click Erase all demo entries → confirm → log clears → summary shows empty state
- [ ] **Keyboard** — Tab through all form fields and buttons without using a mouse; every control must be reachable and operable with Enter/Space
- [ ] **320 px** — resize browser to 320 px wide → no horizontal scroll → all four summary tiles visible → chips wrap cleanly
- [ ] **No network** — open DevTools → Network tab → use the app fully → zero requests should appear

### D. Before sharing with any audience

- [ ] Confirm the sticky banner "Demo only — use fictional data" is visible on load
- [ ] Confirm the footer privacy notice is visible at the bottom of the page
- [ ] Confirm no real personal data is stored in the demo instance (use Erase all, or clear DevTools localStorage)
- [ ] Remind recipients verbally or in writing: **fictional data only, not for clinical use**

---

*Wellness Tracker — Demo · Single-file learning prototype · No medical advice · No patient data*
