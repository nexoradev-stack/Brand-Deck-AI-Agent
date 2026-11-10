# 🏗️ BrandDeck AI Agent — Architecture

How a prompt becomes on-brand, editable slides inside the presentation that is already open, and how every slide is checked on the way.

**Colour key used in every diagram:** 🟦 people and the sidebar · ⬛ pipeline steps · 🟨 checks and gates · 🟪 Gemini models · 🟩 output · ⬜ Google services and assets

---

## 1. The big picture

```mermaid
flowchart LR
  U(["👤 Slides user"]) --> SB["🧭 Sidebar<br/>Generator.html"]
  SB -->|"Create"| C["✨ Create pipeline<br/>Code.gs"]
  SB -->|"Flowchart"| F["🔀 Flowchart<br/>Diagram*.gs"]
  SB -->|"Rebrand"| RB["🎨 Rebrand<br/>Rebrand.gs"]
  C --> E["📐 Layout engine<br/>Engine.gs"]
  F --> E
  E --> RD["🖌️ Renderer<br/>EngineRenderer.gs"]
  RD --> P[("📑 Open presentation")]
  RB --> P
  C <--> V["🤖 Vertex AI Gemini"]
  F <--> V
  RB <--> V
  LIB[("🎨 2026 template<br/>reference library · Drive")] -.-> E
  C -.-> LOG[("📊 BigQuery<br/>usage + slide log")]

  classDef person fill:#0052FF,stroke:#003ACC,color:#FFFDF9,stroke-width:2px
  classDef ui fill:#DCE7FF,stroke:#0052FF,color:#040A1B,stroke-width:2px
  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9,stroke-width:2px
  classDef model fill:#EDE4FF,stroke:#8E75B2,color:#2E1065,stroke-width:2px
  classDef out fill:#BBF7D0,stroke:#16A34A,color:#052E16,stroke-width:2px
  classDef ext fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  class U person
  class SB ui
  class C,F,RB,E,RD step
  class V model
  class P out
  class LIB,LOG ext
```

**The same-presentation contract:** Create calls `SlidesApp.getActivePresentation()`. A blank deck is reused; a deck with content gets the new slides appended. Rebrand restyles the current deck. Nothing is ever created in a second presentation.

---

## 2. The Create pipeline (V.1_70 – V.1_80)

```mermaid
flowchart TB
  REQ["📝 Request<br/>deck type · topic · client · files · slide count"] --> RS["🔎 Research<br/>Scite, else Gemini Search"]
  REQ --> B1["1️⃣ Brief · Brief.gs<br/>client, slide count, topics in order,<br/>figures in order, prices"]
  RS --> O2
  B1 --> O2["2️⃣ Outline<br/>each topic once · real titles · topic sections<br/>one KPI slide when targets are asked"]
  O2 --> W3["3️⃣ Slide writing · SlideWriter.gs<br/>one Gemini call per slide (in parallel)<br/>response schemas · source quotes"]
  W3 --> K3["Contracts.gs<br/>KPI · RACI · team · risks · pricing · next steps<br/>in the shape their designs need"]
  K3 --> FG["Figure check<br/>a figure stays only with a source"]
  FG --> G4{"4️⃣ Slide gate · Gate.gs<br/>empty? cut? overlap?<br/>lost content?"}
  G4 -->|"passes"| FN
  G4 -->|"fails"| RW["Rewrite once<br/>(schema, no new figures)"]
  RW -->|"still fails"| SAFE["Safe layout<br/>cards or list, content whole"]
  RW -->|"passes"| FN
  SAFE --> FN["Finish.gs + Quality.gs<br/>titles · headings · product names ·<br/>figure lock · meaning review"]
  FN --> D5["5️⃣ Drawing<br/>Engine → EngineRenderer"]
  D5 --> CHK["🔍 Deck checker<br/>every slide after drawing"]
  CHK --> OUT[("📑 Slides in the open deck<br/>+ speaker notes with sources")]

  classDef input fill:#DCE7FF,stroke:#0052FF,color:#040A1B,stroke-width:2px
  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9,stroke-width:2px
  classDef check fill:#FDE68A,stroke:#D97706,color:#3B2300,stroke-width:2px
  classDef ext fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  classDef out fill:#BBF7D0,stroke:#16A34A,color:#052E16,stroke-width:2px
  class REQ input
  class B1,O2,W3,D5 step
  class K3,FG,G4,RW,SAFE,FN,CHK check
  class RS ext
  class OUT out
```

| Step | Rule that matters most |
|---|---|
| 1. Brief | **The request is the source of truth.** Topics and figures are kept exactly as asked. |
| 2. Outline | Every requested topic gets a slide, once. Phases exactly as listed. |
| 3. Writing | One call per slide; known slide kinds use a response schema so every field is filled. Every statistic carries a 10–20 word `source_quote`. |
| 4. Gate | A failing slide is rewritten once, then drawn with a safe layout. A per-deck budget keeps runs inside Apps Script's time limit. |
| 5. Drawing | Native shapes on a 720×405 canvas scaled to the page; ~3pt rounded boxes; Plus Jakarta Sans. |
| Quality | Figures never change (99.95% stays 99.95%); no absolute promises; product names as their makers write them. **A deck never stops.** |

**Long decks** (up to 200 slides) run in parts of about 4.5 minutes; the sidebar continues automatically and the whole deck appears at once.

---

## 3. Models per job (V.1_56)

```mermaid
flowchart LR
  OUT1["Outline"] --> A["Gemini 4 Argon"]
  TXT["Slide text · diagrams · risks"] --> A
  RES["Research · client · checks · titles"] --> FL["Gemini 3.8 Flash"]
  ICO["Generated icons"] --> NB["Nano Banana 2"]
  A -.->|"if unavailable"| FB["Gemini 2.5 Flash / Pro"]
  FL -.-> FB

  classDef job fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef model fill:#EDE4FF,stroke:#8E75B2,color:#2E1065,stroke-width:2px
  classDef fb fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B,stroke-dasharray:5 4
  class OUT1,TXT,RES,ICO job
  class A,FL,NB model
  class FB fb
```

Models are set per job in Script properties (`OUTLINE_MODEL`, `CONTENT_MODEL`, `RESEARCH_MODEL`, `IMAGE_MODEL`). The newest models are served on the `global` region. **Test AI models** in the menu checks them. All calls use Vertex AI OAuth.

---

## 4. Deck types and design libraries

```mermaid
flowchart TB
  T{"Deck type"} -->|"General"| GL["📘 2026 template designs<br/>Reference.gs"]
  T -->|"Proposal"| PL["🤝 65+ proposal designs<br/>ProposalDesigns_0..4"]
  T -->|"Solution"| SL["🛠️ Solution designs<br/>SolutionDesigns_1..4"]
  PL --> PK["ProposalKit.gs<br/>client detection · co-branding · client logo"]
  SL --> SK["SolutionKit.gs<br/>tone · dark slides · no client logo"]
  SH["Shared frames only:<br/>strict cover · section dividers · Thank you"] -.-> PL
  SH -.-> SL
  ROT["🔄 Design rotation<br/>fit gate · shuffle · look families · shared history"] -.-> GL

  classDef q fill:#FDE68A,stroke:#D97706,color:#3B2300,stroke-width:2px
  classDef lib fill:#DCE7FF,stroke:#0052FF,color:#040A1B,stroke-width:2px
  classDef kit fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef ext fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  class T q
  class GL,PL,SL lib
  class PK,SK kit
  class SH,ROT ext
```

---

## 5. Flowchart and diagram intelligence

```mermaid
flowchart LR
  IN["Sketch · Mermaid · draw.io ·<br/>Excalidraw · SVG · PDF/PPTX · text"] --> RD1["1. READ<br/>parts Vertex AI can see"]
  RD1 --> SEL["2. SELECT<br/>diagram type from the<br/>diagram-design table + budget"]
  SEL --> IR["3. IR<br/>nodes, edges, groups"]
  IR --> BR["4. Brand transform<br/>66° colours, shapes signal type"]
  BR --> LAY["5. Engine layout<br/>no arrows through boxes,<br/>shared lines, centred hub"]
  LAY --> NAT[("Editable native shapes<br/>+ connectors + icons")]
  IR -.->|"low confidence"| KEEP["Keep the original picture"]

  classDef in fill:#DCE7FF,stroke:#0052FF,color:#040A1B,stroke-width:2px
  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef out fill:#BBF7D0,stroke:#16A34A,color:#052E16,stroke-width:2px
  classDef safe fill:#FDE68A,stroke:#D97706,color:#3B2300
  class IN in
  class RD1,SEL,IR,BR,LAY step
  class NAT out
  class KEEP safe
```

Rules come from [diagram-design](https://github.com/cathrynlavery/diagram-design) (MIT), vendored in `vendor/diagram-design/` and bundled into `DiagramDesignKit.gs` by `scripts/build-diagram-kit.js`. Its HTML/SVG layer is not used.

---

## 6. Icons, photos and client logos

```mermaid
flowchart LR
  IC(["🧩 Icons<br/>IconProvider.gs"]) --> I1["Local 66° icon set"] --> I2["Iconify outline families<br/>Lucide · Tabler · Heroicons · Phosphor"] --> I3["Safe fallback glyph"]
  LG(["🏷️ Client logos<br/>ClientLogoProvider.gs"]) --> L1["Drive assets"] --> L2["Brandfetch<br/>(with key)"] --> L3["Unavatar<br/>(opt-in)"] --> L4["logo.dev<br/>(with token)"] --> L5["Text wordmark / none"]
  PH(["🖼️ Photo strips"]) --> P1["Brand Stock Images folder"] --> P2["Cropped to fill the strip"]

  classDef head fill:#0052FF,stroke:#003ACC,color:#FFFDF9,stroke-width:2px
  classDef a fill:#DCE7FF,stroke:#0052FF,color:#040A1B
  classDef b fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  class IC,LG,PH head
  class I1,L1,P1 a
  class I2,I3,L2,L3,L4,L5,P2 b
```

Icons are chosen by meaning, then recoloured to the brand; SVG is sanitised before insertion. Client logos never replace the 66degrees mark. **A failure never aborts the deck.**

---

## 7. Rebrand

```mermaid
flowchart LR
  D[("Existing deck")] --> INV["Inventory<br/>images · text · shapes"]
  INV --> REV["Gemini reviews each slide image<br/>fake logos, UI shots, text in pictures"]
  REV --> APPLY["Brand pass<br/>Plus Jakarta Sans · colours · 66 mark in<br/>the old mark's place · text fits"]
  APPLY --> D2[("Same deck, on brand")]

  classDef io fill:#BBF7D0,stroke:#16A34A,color:#052E16,stroke-width:2px
  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef model fill:#EDE4FF,stroke:#8E75B2,color:#2E1065
  class D,D2 io
  class INV,APPLY step
  class REV model
```

Rebrand keeps the design: grouped shapes are measured on the slide, dark panels keep white text, and text sizes stay uniform per slide.

---

## 8. Usage and slide logs

```mermaid
flowchart LR
  RUN["Every Create · Flowchart · Rebrand run<br/>(each part, each model)"] --> UL["UsageLog.gs"]
  UL -->|"insertAll"| DU[("deck_agent.deck_usage")]
  RUN --> SLG["Slide log<br/>spec + issues per slide"]
  SLG --> DS[("deck_agent.deck_slides")]
  DU --> VW["Views: v_usage_cost · v_runs ·<br/>v_daily_user · v_summary_*"]
  DS --> VI["Views: v_slide_issues · v_design_health"]
  VI --> RP["scripts/replay-slides.js<br/>replay a logged slide locally"]

  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef bq fill:#DCE7FF,stroke:#0052FF,color:#040A1B,stroke-width:2px
  classDef ext fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  class RUN,UL,SLG step
  class DU,DS,VW,VI bq
  class RP ext
```

The log uses the add-on's existing Google Cloud access (no new permission). The user's name and email come from their Google profile. Setup: `setup/usage_bigquery.sql` and `setup/USAGE_SETUP.md`.

---

## 9. Build, test and deploy

```mermaid
flowchart LR
  PR["Push / pull request"] --> CI["ci.yml<br/>validate.js + node tests<br/>+ secrets check"]
  CI --> M["main"]
  M --> DP["deploy.yml<br/>clasp push -f → clasp deploy"]
  DP --> AS[("Apps Script<br/>add-on deployment")]
  SEC["GitHub secrets<br/>CLASPRC_JSON · SCRIPT_ID · DEPLOYMENT_ID"] -.-> DP

  classDef step fill:#040A1B,stroke:#4D8BFF,color:#FFFDF9
  classDef check fill:#FDE68A,stroke:#D97706,color:#3B2300,stroke-width:2px
  classDef out fill:#BBF7D0,stroke:#16A34A,color:#052E16,stroke-width:2px
  classDef ext fill:#E6EEF3,stroke:#B3C5D0,color:#040A1B
  class PR,M,DP step
  class CI check
  class AS out
  class SEC ext
```

Without the deploy secrets, the deploy steps are skipped and nothing fails. `clasp-pull.yml` (manual) pulls the live Apps Script sources for comparison.

---

## 10. Security

- **No keys in code:** Vertex AI uses OAuth; optional keys (Scite, Brandfetch, Unavatar, logo.dev) live in Script properties.
- **Placeholders only:** `.clasp.json` (`REPLACE_WITH_APPS_SCRIPT_ID`), SQL and docs (`your-gcp-project-id`); test people are made up (`@example.com`).
- **Drive:** the add-on uses `DriveApp` only (no runtime Drive API enablement).
- **Uploads and SVG** are sanitised; client domains are validated before any logo lookup.
- **Secrets check** in CI blocks keys, tokens, real project IDs and personal emails.
