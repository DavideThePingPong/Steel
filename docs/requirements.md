# Project Requirements

<!--
This is a living requirements document meant for iterative, AI-assisted (“vibe”) coding.
Use the ✅ checkboxes to mark items done. Replace examples with your project’s specifics.
-->

## 0. Document Meta
- **Project name:** _TBD_
- **Repo URL:** _TBD_
- **Doc owner:** _TBD_
- **Last updated:** 2025-08-20
- **Status:** Draft ✅ ☐ In review ☐ Approved ☐

---

## 1. Problem Statement & Vision
**Problem:** Field teams assembling steel frames often lack quick, standards-aligned guidance on plates, fixings, maximum loads, and connection details for a given steel member. This causes delays, rework, and risk.

**Vision (elevator pitch):** A standalone web app that, given a steel member type and key measurements, returns succinct, accurate engineering information—plates, fixings, maximum loads, and connection options—following Australian Standards, enabling employees to construct steel frames efficiently and safely.

**Success criteria / North star:**
- [ ] MVP computes and displays required fixings/plates and maximum permissible loads for a set of common members and connections
- [ ] Output clearly cites the governing Standard and version used in each calculation
- [ ] Field users can complete the core flow in < 60 seconds on a laptop/tablet

---

## 2. Scope
**In scope (MVP):**
- [ ] Standalone web app (no sign-in required) that runs locally or from static hosting
- [ ] Input: steel member type (e.g., UB/UC/PFC/SHS/RHS/EA/GA/Lip-C) + size/grade, connection scenario (e.g., shear plate, end plate, angle cleat, base plate), and basic load parameters
- [ ] Compute: indicative plate sizes/thickness, fastener type/grade/count, edge/end distances, hole/slot types, weld requirements where applicable, and maximum allowable loads per selected standard methodology
- [ ] Standards catalogue with version tagging (e.g., AS 4100:2020, etc.—**MVP fixed to AS 4100:2020**)
- [ ] Results screen with a compact calculation summary and printable view

**Out of scope (for now):**
- [ ] Producing certified engineering sign-off (app is an aid, not a substitute for a structural engineer)
- [ ] Full environment action derivation (wind/snow/earthquake site modelling)—MVP expects user-provided design actions or simplified presets
- [ ] Finite element analysis or 3D modelling
- [ ] Non-Australian standards

**Assumptions:**
- Users can supply design actions or accept app presets for typical cases
- App will store only parameters and formulae, not reproduce standard text/tables verbatim

**Constraints:**
- **Only structural steel members** (e.g., grades 300/350/450 MPa within AS 4100 / AS/NZS 4600 scope). Connections may attach to other substrates (steel, concrete, timber) **but non-steel substrate design is post-MVP** unless explicitly stated.
- Respect standards copyright; expose methodology and references, not raw tables
- Prefer offline-capable PWA; minimal stack and minimal hosting friction

---

## 3. Stakeholders & Users
**Stakeholders:** Product owner, field supervisors, structural engineer (review), developers.

**Primary user personas:**
- **Site Assembler (Field Tech):** Needs fast, unambiguous connection details. Success = correct install on first go.
- **Site Supervisor:** Needs print/PDF summary attached to work pack. Success = schedule adherence, fewer RFIs.
- **Structural Engineer (Reviewer):** Needs to verify method, inputs, and safety factors. Success = traceable calcs and standard/version clarity.

---

## 4. User Stories (MVP backlog)
> Format: *As a* [persona], *I want* [capability], *so that* [outcome]. Include acceptance criteria.

- **US-01:** As a field tech, I want to select a member type and size, so that I get connection options.  
  **Acceptance:**
  - [ ] Member libraries include common UB/UC/PFC/SHS/RHS/EA/GA/Lip-C
  - [ ] Material grade selection (e.g., 300/350/450 MPa) affects results

- **US-02:** As a field tech, I want to choose a connection scenario (e.g., single shear plate), so that I can enter design actions and constraints.  
  **Acceptance:**
  - [ ] Inputs for shear/axial/moment as applicable
  - [ ] Edge conditions (min distances, plate bounds) validated

- **US-03:** As a field tech, I want the app to compute required plate dimensions, fastener grade/count/spacing, and weld specs, so that I can assemble correctly.  
  **Acceptance:**
  - [ ] Output lists governing limit states (bearing, net section, bolt shear, block shear, tear-out, weld strength)
  - [ ] Shows controlling check and utilisation ratios
  - [ ] Cites standard(s) and clause references by version

- **US-04:** As a supervisor, I want a printable summary/PDF, so that I can include it in the work pack.  
  **Acceptance:**
  - [ ] One-page summary with inputs, results, at-a-glance diagram, standards

- **US-05:** As an engineer, I want a method statement view, so that I can verify assumptions and safety factors.  
  **Acceptance:**
  - [ ] Toggle to show calc steps and factors (γ, φ) without reproducing tables

**Non-goals:** mobile native app; multi-tenant accounts; certification seal.

---

## 5. Functional Requirements
Break down by feature/epic. Reference user stories.

### 5.1 Member & Connection Input
- [ ] Member catalogue with sizes & section properties (public catalogues or user-entered)
- [ ] Grade selection; environment/corrosion notes as metadata
- [ ] Connection templates: single shear plate, end plate, angle cleat, base plate (anchors)
- [ ] Connection substrate material (per connection). MVP supports steel members only (Grades 250–450 MPa), but connections may eventually specify substrate = steel/concrete/timber.

### 5.2 Calculation Engine
- [ ] Limit-state checks per **AS 4100:2020** for bolts, plates, welds (**steel-to-steel** connections in MVP)
- [ ] Bolt groups: patterns, spacing, edge/end distances, hole/slot types
- [ ] Combined actions (V, N, M) with interaction rules where applicable
- [ ] Standards module: clause mapping, φ/safety factors, version tags
- [ ] Tolerances & rounding consistent with practice

### 5.3 Results & Output
- [ ] Summary card (governing check, utilisation, recommended detailing)
- [ ] Printable A4 view (SVG diagram + inputs + BOM (plates/bolts/weld))
- [ ] “Show method” expands to outline formulae and references (no verbatim tables)

### 5.4 Data & Audit
- [ ] No PII; optional local save of last inputs/results
- [ ] Version banner shows standards set used; warn when outdated

### 5.5 Load Case Input & Helper (MVP)
**Goal:** Convert simple user inputs (weight or volume of material on top) into design actions **V, N, M** required by the calc engine.

**What the user can provide:**
- **Weight** (kN) **or** **Volume** (m³) **+ material density** (γ in kN/m³; e.g., concrete ≈ 24 kN/m³)
- **Geometry:** span **L** (m), member **spacing/tributary width** **b** (m)
- **Support/connection type:** shear connection (pin) vs moment connection
- **Factored?** Whether actions include ULS factors (MVP expects **design actions**)

**Conversions used (MVP):**
- If **volume given**: `W [kN] = γ [kN/m³] × V [m³]`
- If **surface load**: `q_area [kN/m²] = γ × t [m]`
- **Line load on member**: `q_line [kN/m] = q_area × b`

**Reactions & internal actions (simply supported):**
- End reactions: `R = q_line × L / 2` → **V** at each shear connection
- Max moment at midspan: `M_max = q_line × L² / 8` (used only if the connection is **moment-resisting**)
- Point load at midspan alternative: `R = P / 2`, `M_max = P × L / 4`

**What weight/volume alone misses:** Span **L**, tributary width **b**, support/connection type, eccentricities, and whether values are **factored**. The helper will ask for these to compute **V, N, M**.

**MVP UI behaviour:**
- Load Helper with presets (e.g., concrete topping, sheeting, roofing) auto-fills density
- User enters: volume **or** thickness, span **L**, spacing **b**, support type → app computes **V** (and **M** if needed)
- Output to calc engine: **V_kN**, optional **M_kNm**, optional **N_kN**

### 5.6 Capacity Mode (no given loads)
**Goal:** When no loads are provided, compute **maximum allowable design actions** for the chosen member + connection template under AS 4100:2020.

**Behaviour:**
- User selects: member (family/size/grade), connection type, allowed bolt diameters/classes, plate thickness bounds, weld options, and any geometric limits (max plate width/length)
- Engine designs a feasible layout and returns capacity outputs:
  - **V_design,max** (shear)
  - **N_design,max** (tension/compression where applicable)
  - **M_design,max** (moment connections)
  - Optional **V–M interaction envelope** for moment-resisting details
- Output includes governing limit state (e.g., bolt shear, bearing, net section, block shear, weld strength, plate bending) with clause refs and utilisation at capacity = 1.0

**Acceptance criteria:**
- [ ] For two reference details (shear plate, end plate), capacities match benchmark hand-calcs within ±3%
- [ ] Governing check and clause references are shown
- [ ] Printable summary includes a clear “Capacity mode” header and assumptions

#### 5.6.1 Presets
| ID | Connection | Typical members | Bolts (AS/NZS 1252) | Pattern (rows×cols) | Plate (min) | Weld (MVP) | Hole type | Notes |
|---|---|---|---|---|---|---|---|---|
| **SP-16-2R** | Shear plate | UB/UC/PFC | M16 class 8.8 | 2×1 (vertical stack) | t ≥ 10 mm, Grade ≥ 300 | Fillet 6 mm to web | Standard | Baseline shear plate for light–med shear. Engine optimises pitch/edge. |
| **SP-20-3R** | Shear plate | UB/UC/PFC | M20 class 8.8 | 3×1 | t ≥ 12 mm, Grade ≥ 300 | Fillet 6–8 mm | Standard | Higher shear capacity; may govern on bolt shear/bearing or net section. |
| **EP-20-2x2** | End plate | UB/UC | M20 class 8.8 | 2×2 | t ≥ 12 mm, Grade ≥ 300 | Fillet 6 mm (web/flange) | Standard | Moment-capable; engine returns **V_max**, **M_max** and interaction where applicable. |
| **EP-24-2x3** | End plate | UB/UC | M24 class 8.8 | 2×3 | t ≥ 16 mm, Grade ≥ 350 | Fillet 8 mm | Standard | Higher moment capacity; plate bending or bolt tension may govern. |
| **AC-12-2R** | Angle cleat (single) | UB/PFC | M12 class 8.8 | 2×1 | t ≥ 8 mm, Grade ≥ 300 | Fillet 6 mm | Standard/Short-slot | Compact shear detail; check tear-out and block shear. |
| **AC-16-2x2** | Angle cleat (double) | UB/UC/PFC | M16 class 8.8 | 2×2 | t ≥ 10 mm, Grade ≥ 300 | Fillet 6–8 mm | Standard | For larger shear; verify gauge limits against member leg/seat.

**Preset behaviour:**
- User picks a preset, selects member (family/size/grade), and the engine proposes a feasible layout **within template bounds** (pitch, gauge, edge ≥ AS 4100 minima).
- Engine computes **capacity outputs** (V_design,max / N_design,max / M_design,max as applicable), lists the **governing limit state**, and cites **AS 4100:2020** clause references.

**Parameter bounds (handled by engine):**
- Edge/end distances based on bolt diameter & hole type (standard holes vs slots adjust minima).
- Plate thickness lower bounds from preset; upper bounds can be user-limited.
- Bolt property class fixed per preset; diameter fixed by preset selection. Users can clone/edit presets if needed.

**Test vectors (for verification ±3%):**
1. **SP-16-2R** on **200UC46.2**, Grade 350; plate t=10 mm. Expect governing **bolt shear/bearing**; report **V_design,max** and controlling clause.
2. **EP-20-2x2** on **250UB31.4**, Grade 350; plate t=12 mm; fillet 6 mm. Expect **bolt tension or plate bending** to govern at **M_design,max**.

---

## 6. Non-Functional Requirements (NFRs)
**Performance:** P95 compute < 300 ms for typical cases.

**Availability:** N/A offline; if hosted, >99.5% monthly.

**Security & Privacy:**
- [ ] No server-side storage (MVP); local-only data
- [ ] TLS if hosted; Subresource Integrity (SRI) for third-party libs

**Compliance & Standards:**
- [ ] **MVP fixed to AS 4100:2020** (hot-rolled structural steel members & steel-to-steel connections). Clause pointers included in outputs.
- [ ] Other standards (e.g., AS 5216 for anchors to concrete, AS/NZS 4600 for cold-formed) may be referenced for terminology only and targeted in post-MVP expansions.
- [ ] Do not reproduce standards content; provide clause pointers only

**Accessibility:** WCAG 2.1 AA for core flows.

**Internationalisation:** Not required (MVP).

**Observability:** Console errors; optional client-side error capture.

---

## 7. System Architecture (MVP)
**High-level:** Pure client-side **PWA** → calculation modules (**TypeScript**) → local storage (settings). No backend required.

**Tech choices (tentative):**
- **Frontend:** HTML + **TypeScript** + **Vite** (no framework), or Svelte + Vite if components help
- **UI:** Minimal CSS (Tailwind optional) + small helpers
- **Drawings:** **SVG** generation for plate/bolt layouts
- **PDF:** Browser print-to-PDF (or jsPDF) from the results view
- **Data:** JSON catalogues for member sizes (or user-entered), standards metadata/version tags
- **Auth:** None (MVP)
- **Hosting:** Static (GitHub Pages / Cloudflare Pages / Netlify). Installable PWA for offline use.

**Diagrams:** Add `/docs/architecture/diagram.drawio` with data flow (Inputs → Calc Engine → Results/Diagram → Print/PDF).

**API style:** N/A (no server). Public functions exposed from `calc/` modules.

---

## 8. Data Model
**Canonical units (MVP):** millimetres (mm), kilonewtons (kN), megapascals (MPa), degrees (°). Store/compute in SI; convert only at UI.

### 8.1 Entities & Interfaces (TypeScript)
```ts
// Families limited to structural steel per constraint
export type MemberFamily = 'UB' | 'UC' | 'PFC' | 'SHS' | 'RHS' | 'EA' | 'GA' | 'LipC';
export type SteelGrade = '250' | '300' | '350' | '400' | '450'; // MPa yield

export interface SectionProps {
  A: number;    // area mm^2
  Ix?: number;  // second moment about x, mm^4
  Iy?: number;  // second moment about y, mm^4
  Zx?: number;  // section modulus x, mm^3
  Zy?: number;  // section modulus y, mm^3
  tf?: number;  // flange thickness mm
  tw?: number;  // web thickness mm
  r?: number;   // root radius mm
  d?: number;   // depth mm
  b?: number;   // flange width mm
}

export interface Member {
  id: string;
  family: MemberFamily;
  size: string;         // e.g., '200UC46.2'
  grade: SteelGrade;
  section: SectionProps;
}

export type ConnectionType = 'ShearPlate' | 'EndPlate' | 'AngleCleat' | 'BasePlate';
export type Substrate = 'steel' | 'concrete' | 'timber'; // MVP default 'steel'; others captured for future design

export interface BoltSpec {
  id: string;
  standard: 'AS/NZS 1252';
  propertyClass: '8.8' | '10.9';
  diameter_mm: 12 | 16 | 20 | 24 | 30 | 36; // initial set (all kept)
  holeType: 'std' | 'shortSlot' | 'longSlot';
  washerSet: 'std' | 'hardened';
}

export interface WeldSpec {
  process: 'fillet' | 'butt';             // both allowed in MVP
  legSize_mm?: number;                    // for fillet
  throat_mm?: number;                     // derived if not provided
  electrode_strength_MPa?: number;
}

export interface PlateSpec {
  materialGrade: SteelGrade;
  thickness_mm: number;
  width_mm?: number;
  length_mm?: number;
}

export interface LoadCase {
  /** Shear force (design or characteristic) */
  V_kN?: number;
  /** Axial force: tension (+), compression (-) */
  N_kN?: number;
  /** Bending moment about major axis */
  M_kNm?: number;
  /** Optional user label, e.g., 'ULS combo 1' */
  combo?: string;
  /** If provided, denotes value already factored by γ. Otherwise UI may apply default factoring (future). */
  gamma?: number;
  /** Flag to indicate whether inputs are 'factored' design actions */
  factored?: boolean;
}

export interface StandardVersion {
  code: 'AS4100';
  version: '2020';
}

export interface ConnectionTemplate {
  id: string;
  type: ConnectionType;
  name: string;
  parameters: Record<string, unknown>; // plate bounds, bolt rows/cols limits, etc.
}

export interface CalcInputs {
  member: Member;
  connection: ConnectionTemplate;
  substrate: Substrate;  // MVP default 'steel'; others allowed as metadata for future design
  bolts: BoltSpec[];
  weld?: WeldSpec;
  plate?: PlateSpec;
  loads: LoadCase;
  standards: StandardVersion[]; // MVP: [{ code: 'AS4100', version: '2020' }]
  assumptions?: string[]; // simplifying assumptions
}

export interface UtilisationCheck {
  name: string;          // e.g., 'Bolt shear'
  clauseRef: string;     // e.g., 'AS4100-2020 Cl.9.x'
  demand: number;        // factored action
  capacity: number;      // design capacity
  utilisation: number;   // demand/capacity
  governing?: boolean;
}

export interface CalcResult {
  inputsEcho: CalcInputs;
  plate_dims?: { t: number; w: number; l: number };
  bolt_layout?: { rows: number; cols: number; pitch: number; gauge: number; edge: { e1: number; e2: number } };
  weld_specs?: WeldSpec;
  checks: UtilisationCheck[];
  governingCheck: string; // name of governing check
  ok: boolean;
  bom: { plates?: number; bolts?: number; washers?: number; nuts?: number; weld_length_mm?: number };
  notes: string[];        // warnings, assumptions
}

```

### 8.2 Load Case Input (explained)

- V_kN (Shear): In-plane shear at the connection.

- N_kN (Axial): Tension (+) increases bolt/weld demand; Compression (–) may relieve bolt tension but affects bearing and plate checks.

- M_kNm (Moment): Eccentric/end-plate/base-plate actions that create tension/compression couples across the bolt group.

- combo: A label for the user’s load combination (e.g., ULS 1). The engine treats it as metadata.

- gamma & factored: If factored=true (and/or gamma provided), inputs are treated as design actions. For MVP, we expect design actions (no auto-factoring).

### 8.3 Open Questions

- Member catalogue source: bring your own CSVs (per supplier) or bundle a minimal open set?

- Default densities for Load Helper presets (concrete, sheeting, roofing) — provide a small table?

- SVG diagram style conventions (dimensions, annotation fonts) for print clarity?

---

## 9. Module Contracts (no HTTP API in MVP)

Public TypeScript function signatures for deterministic calculations. Keep pure & testable.


```ts
/** Validate user inputs against template & standard ranges */
export function verifyInputs(inputs: CalcInputs): { issues: string[]; warnings: string[] };

/** Optimise a bolt layout for a given template & plate bounds */
export function designBoltLayout(inputs: CalcInputs): CalcInputs; // fills bolt_layout via connection.parameters constraints

/** Run limit-state checks and return structured result with clause refs */
export function designConnection(inputs: CalcInputs): CalcResult;

/** Individual check helpers (unit-tested) */
export function checkBoltShear(inputs: CalcInputs): UtilisationCheck;
export function checkBoltBearing(inputs: CalcInputs): UtilisationCheck;
export function checkNetSection(inputs: CalcInputs): UtilisationCheck;
export function checkBlockShear(inputs: CalcInputs): UtilisationCheck;
export function checkWeldStrength(inputs: CalcInputs): UtilisationCheck;
export function checkPlateBending(inputs: CalcInputs): UtilisationCheck;

/** Standards utilities */
export function getStandardFactor(standard: StandardVersion, key: string): number; // φ, etc.

```

Notes

All modules must be pure (no DOM access); UI orchestrates.

Each check returns a clauseRef and a utilisation; designConnection composes and selects the governing check.

Keep a standards/ mapping file that resolves AS4100:2020 factors and any formula switches (no verbatim text).

---

## 10. UX Flows & Screens

 Flow 1: Select Member → Select Connection → Enter Loads → Review Results → Print

Member: family, size, grade (structural steel only)

Connection: type (shear plate / end plate / angle cleat / base plate*), substrate (steel default; others as metadata)

Loads: V, N, M with “factored?” toggle (MVP expects design actions)

Results: governing check, utilisation, plate/bolt/weld recommendations, clause refs

Print/PDF: one-page summary (inputs + SVG + BOM + standards/version)

 Flow 2: “Show method” → clause pointers + formulas used → close

 Flow 3: Settings → standards set/version (locked to AS4100:2020 for MVP) → units display

Wireframe notes:

Single page with 3 columns on desktop: Inputs | Results | Diagram. Stack on mobile.

Diagram: simplified SVG showing plate outline, bolt pattern (rows/cols, pitch, gauge), edge distances.

Validation inline (red callouts for edge distance, spacing, thickness below min, etc.).

Printable Summary (A4) contents:

Header: Project/Job, Date, Standards (AS4100:2020), Version of app

Inputs: Member (family/size/grade), Connection type, Substrate, Loads (combo label)

Diagram (SVG) with key dims; BOM (plate thk/size, bolt dia/class/count, weld size/length)

Checks table: name, clauseRef, demand, capacity, utilisation (highlight governing)

Disclaimer: engineering aid only; verify per project requirements

---

## 11. Content & Copy

Product name/Tagline: TBD

Legal pages: Privacy Policy, Terms of Use (links or stubs)

Email templates: N/A for MVP (no auth)

---

## 12. Acceptance Plan

Definition of Done (MVP):

 US-01..US-05 pass acceptance criteria

 Results match benchmark hand-calcs on 5 representative cases within ±3%

 Printable summary renders correctly on A4

 PWA works offline after first load

UAT checklist:

 Field tech completes Flow 1 in < 60s

 Engineer verifies method and factors for at least 3 cases

Implementation task breakdown (first slice):

verifyInputs: ranges for member grade, plate thickness, bolt diameters, edge/pitch/gauge minima (AS4100 rules)

designBoltLayout: choose rows/cols/pitch/gauge within bounds; compute edge distances; return candidate layout

check functions: bolt shear/bearing, net section, block shear, plate bending, weld strength

designConnection: compose checks, compute utilisation, select governing, produce SVG & BOM

UI: input form, results panel, SVG render, print stylesheet

Tests: unit tests per check; integration tests for two exemplars (shear plate & end plate)

---

## 13. Risks & Mitigations
Risk	Likelihood	Impact	Mitigation
Standards updates (versions)	Med	High	Version tag library; banner when outdated; quick patch process
Copyright/licensing of standards	High	Med	Store parameters/logic only; cite clauses; no verbatim reproduction
Misuse without engineering oversight	Med	High	Prominent disclaimer; method view; encourage peer review
Input errors (garbage in/out)	High	Med	Strong validation; ranges; sensible defaults
Over-simplified templates	Med	Med	Start narrow; expand with tests and engineer review

---

14. Timeline & Milestones

T0 (Week 0–1): Architecture & scaffolding

T1 (Week 2–3): verifyInputs + designBoltLayout + two check modules

T2 (Week 4): Full designConnection, SVG + Print, UAT & launch

Gantt or checklist acceptable.

---

15. AI-Assisted “Vibe Coding” Hooks

Working agreements:

Keep this doc updated. Treat it as the contract with AI helpers.

For each task, write a prompt stub in /prompts/ describing context, goal, constraints, and success criteria.

Prompt stub template (save as /prompts/FEATURE_NAME.md):

```markdown
Context: (what exists, where code lives)
Goal: (what to build/change)
Constraints: (tech choices, NFRs, style)
Inputs: (APIs, data models)
Acceptance: (tests, UX behaviors)
Follow-ups: (next tasks if successful)

```

---

16. Deployment Strategy (MVP)

Environments: Local (open index.html) → Preview (Pages) → Production (Pages)

CI/CD: GitHub Actions to build static bundle and deploy on tag

Secrets: None (MVP); set up .env.example for future

Backups & migrations: N/A (no DB). Keep a standards/ versioned folder.

---

17. Backlog Parking Lot

Advanced CAD-like visualisation

 Sketch-to-calcs: upload a photo/PDF of a simple annotated frame sketch → parse members, spans, and dimensions → map to member catalogue → run connection/fixing checks

 Multi-standard compliance (Eurocode, US AISC)

 Mobile offline-first mode with on-device caching

Sketch-to-calcs: Feasibility & Phasing

Phase 1 (Template-driven OCR):

Constrain input to annotated templates with clear fonts/symbols.

Use OCR + geometry heuristics (OpenCV/Tesseract) to extract dimensions and members.

Output structured JSON model for calc engine.

Phase 2 (Vector inputs):

Accept DXF/SVG uploads from CAD.

Parse layers/linetypes for members and dimensions with high fidelity.

Map directly to member catalogue and load cases.

Phase 3 (ML assist):

Add lightweight CV/ML model to classify connection types and validate OCR results.

Overlay interpreted schematic on upload for user confirmation/edit before calculation.

Constraints:

Always show interpreted model back to user for validation.

Deterministic standards-based calc engine remains source of truth.

Prominent disclaimer: engineering aid only, not a certified design output.

---

18. Changelog.

Changelog (human-readable)

2025-08-20: Created initial requirements.md; added Load Helper (§5.5), Capacity Mode (§5.6) with presets, and Sketch-to-calcs phasing (§19).