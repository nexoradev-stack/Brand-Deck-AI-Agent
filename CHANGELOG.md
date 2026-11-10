# 📜 Changelog

All notable changes to the 66° Deck Agent. Versions follow the project's own numbering (`V.1_xx`); each release below groups the versions of one stage.

---

## v1.80.1 — Repository docs and privacy · 10 Nov 2026

- 🎨 New README with banner, badges, colour-coded diagram and up-to-date pipeline (V.1_80)
- 🏗️ **ARCHITECTURE.md** with 9 colour-coded diagrams: big picture, Create pipeline, models, deck types, Flowchart, icons and logos, Rebrand, logs, deploy
- 📜 This changelog
- 🛡️ `scripts/check-secrets.sh` + `ci.yml`: tests and a secrets check on every push and pull request
- 🔒 Placeholders and made-up test people only: no real project IDs, Apps Script IDs or personal emails anywhere in the history

## v1.80 — The five-step pipeline · 29 Oct – 2 Nov 2026 (V.1_70 – V.1_80)

- 1️⃣ **The brief** (V.1_70): the request is the source of truth — client, slide count, topics and figures kept exactly as asked; figure check
- 2️⃣ **The outline** (V.1_71): each topic once, real titles, topic sections
- 3️⃣ **Slide shapes** (V.1_72): contracts for KPI, RACI, team, risks, pricing and next steps, with response schemas
- 4️⃣ **The slide gate** (V.1_73): a failing slide is rewritten once, then drawn with a safe layout
- 5️⃣ **Drawing** (V.1_74): no empty space under titles; final text checks (`Finish.gs`)
- 🧪 **Quality layer** (V.1_78): figure lock, rules per slide kind, product-name dictionaries, meaning review
- 🤖 **Gemini's way** (V.1_79): one Gemini call per slide in parallel, schemas, `source_quote` for every statistic
- 🔒 **KPI lock and last word** (V.1_80): every requested topic present; KPIs keep their figures
- 🐛 Fix rounds from test decks 42–51 for General, Proposal and Solution decks

## v1.69 — Models, checks and logs · 26 – 29 Oct 2026 (V.1_56 – V.1_69)

- 🤖 **Models per job** (V.1_56): Gemini 4 Argon writes, Gemini 3.8 Flash researches and checks, Nano Banana 2 draws icons; **Test AI models** menu item
- 🧱 **Quality gate** (V.1_58): slides filled, no empty panels or cut sentences; new layer, region, Gantt and runbook designs
- 🎨 **Rebrand** (V.1_59 – V.1_61): grouped shapes measured on the slide, Plus Jakarta Sans only, design kept, 66 mark in the old mark's place, uniform text sizes
- 📊 **Usage log** to BigQuery (V.1_62) and **slide log + replay** (V.1_64)
- 🔍 **The deck checker** (V.1_68): every slide checked before and after drawing
- ✨ Spiral agenda, Gantt as table, figures as KPI tiles, RACI role columns, notes that match the slides (V.1_63 – V.1_69)

## v1.55 — Solution Deck · 15 – 17 Oct 2026 (V.1_52 – V.1_55)

- 🛠️ Solution Deck design library and content tone (V.1_52)
- 🌓 Batch 2 designs, dark slides, no client logo (V.1_53)
- 🐛 Fix rounds from test decks 24–26

## v1.51 — Proposal Deck · 9 – 15 Oct 2026 (V.1_37 – V.1_51)

- 🤝 Proposal library switch and strict proposal cover with the client logo (V.1_37)
- 🏷️ Client logos: `ClientLogoProvider` (Brandfetch, Unavatar), logo.dev, Wikidata domains (V.1_37 – V.1_38)
- 🗂️ **65 proposal designs**, section dividers, proposal agendas (V.1_42)
- 👥 Team structure drawn, workstreams always have slides, risks with mitigations, KPIs keep their numbers (V.1_43 – V.1_50)
- 🖼️ Background photos and photo strips from the brand's Stock Images folder (V.1_44 – V.1_49)
- 🧭 Simpler Create tab: deck type first, client fields only for Proposal (V.1_41)
- 📖 Gemini plans the story first; the whole deck appears at once (V.1_51)

## v1.36 — Bigger decks, diagrams and Flowchart · 7 – 9 Oct 2026 (V.1_24 – V.1_36)

- 📁 Moved to the 66degrees Drive: brand folder, reference library and template (V.1_24)
- 🧭 New Generator panel: Create/Rebrand modes, multiple files, cancel, readable steps (V.1_26)
- 🔄 **Design rotation** across decks and users (V.1_27)
- 📏 **Up to 200 slides**, built in parts, panel continues automatically (V.1_28 – V.1_30)
- 🧩 **Diagram reconstruction and branded icon intelligence** (PR #30)
- 🔀 **Flowchart tab**: describe steps or upload a sketch (V.1_31); diagrams follow diagram-design (V.1_32 – V.1_36)

## v1.17 — Draw into the open deck · 1 – 6 Oct 2026 (to V.1_17)

- 📑 Aligned with the V.1_17 spec: slides are drawn **into the presentation that is already open**
- 🛡️ Renderer safety: valid shape types, shapes without text, zero-height inserts
- 🔐 **Vertex AI OAuth** only; no Gemini API key
- 📂 Drive via `DriveApp` only (no runtime Drive API enablement)
- ⚙️ Admin setup at first authorisation; clean user menu
- 🚢 CI deploy with clasp; clasp pull helper
- 🗺️ Architecture flowchart in the README

## v1.0 — First version · 30 Sep – 1 Oct 2026

- 🎉 Initial commit and the complete 66degrees deck generation pipeline (PR #2)
- 🤖 Gemini planner moved to Vertex AI
- ✅ Cursor connected to GitHub (PR #1)

---

🙌 nexoradev-stack · eneelkant · Claude · Cursor
