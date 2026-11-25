# Workspace platform research: workspace.findyourdent.com

Research date: 2026-11-25. Status: research input for specs. Nothing here is a decision yet.

## The question

What does it take to build workspace.findyourdent.com: a multi-tenant platform, run on the owner's VPS, where Indonesian dental providers (private dentists, clinics, hospital dental units) manage their findyourdent.com directory page and run their practice. Requested modules: patient management (unified calendar, electronic medical records, patient CRM), partner-dentist management (per-specialty treatment scheduling, with WebCeph as the orthodontic benchmark), inventory, assets, finance, and a WhatsApp-based AI agent for qualification, booking, reminders, follow-ups and closing. Five roles: superadmin/owner, partner dentist, dental nurse, operations admin, viewer. It reuses the findyourdent.com design system, adds a `/workbuddy` area, and is the improved successor of dentalclinic.web.id, which stays online.

## How this note was made, and how far to trust it

- Six research tracks ran in parallel, one per numbered section below. Each was told to use primary sources only (source code, official docs, regulation text, guideline owners) and to cite every claim inline.
- **[UNVERIFIED]** marks a claim that could not be confirmed at a primary source. **[INFERENCE]** marks reasoning or a recommendation, not a fact.
- Sections 1 and 6 cite the two local repositories as `repo/path:line`. Section 1 defines aliases for which checkout or commit each citation refers to, because neither default checkout was current.
- Most web pages were read through a fetch tool that summarises, so quoted wording can be a paraphrase. Large Indonesian laws were read from scanned PDFs with an OCR layer. Re-read the live page before quoting anything in a contract or a filing.
- Sections 3 and 4 inform software design. They are not tax or legal advice. Items marked **(lawyer)** need an Indonesian health-law, privacy or tax professional.
- Re-checked independently after the tracks reported, all confirmed:
  - main product: Firestore rules deny all client access; the `__session` cookie has no `Domain` attribute; provider types are `klinik`, `privateDentist`, `hospital`; the local `main` checkout is about 140 commits behind `origin/main`
  - dentalclinic: scope decision "The agent never sends anything ... No WhatsApp Business API"
  - PostgreSQL 19.0 released 2026-11-19; the MinIO repository archived 2026-04-25; TanStack Start's Node hosting steps
  - Meta's developer pricing page: service messages charged again from 2026-10-01 with 1,000 free per number per month, and utility templates inside the service window charged again
  - Permenkes 5/2026: in force since 2026-05-04, revoking Permenkes 62/2017 and 54/2015
- Not re-checked: WebCeph's pricing page returned HTTP 403 to the re-check, so the WebCeph prices in section 2 rest on the track's own read.

## What the research found

**Existing systems (section 1)**

- The main product has no provider dashboard and no API a second system could call. An owner edits a listing only at `/tempat/:placeId/klaim` and `/daftar-harga`, behind a Google sign-in cookie, App Check and rate limits.
- Ownership there is one approved Claim by one Google account per Google `placeId`. There are no tenants, staff seats or roles.
- The design system is portable as tokens (colors, type scale, radius, Figtree, brand files), not as a component library. Nine patient-facing components exist; no table, dialog, calendar or sidebar.
- The main product runs React Router 7 with Biome on Cloud Run and Firestore. dentalclinic already uses the intended workspace stack (TanStack Start, shadcn, oxlint, oxfmt) but depends on Cloudflare Workers, D1, FTS5 and Worker cron.
- dentalclinic is a single-tenant demo of one fictional clinic with read-only views and a nightly reset. Its reusable parts are rules, not screens: role guards in SQL, one refusal for restricted and missing data, masking, a hash-chained audit log, propose-then-approve, the recall rule with WhatsApp opt-in, three finance measures, and device pairing.
- WorkBuddy is Tencent's desktop agent workspace. dentalclinic ships two Expert packages that reach its `/mcp` endpoint with a paired token. `/workbuddy` most plausibly means the Expert landing, install, pairing and MCP home for real tenants **[INFERENCE]**.

**Clinical and practice-management benchmarks (section 2)**

- WebCeph cannot be a data source: its partner API creates patients and records and returns a link, but returns no landmarks, analyses or images. Integration means a deep link plus manual upload of the exported report.
- Orthodontic systems centre on a staged case (dates, visit sequence, hardware log, contract schedule), not on the ceph image. The same shape fits every specialty, so one generic treatment-case entity with stage templates covers them **[INFERENCE]**.
- Guideline owners publish interval rules that a recall engine can hold as data: orthodontic adjustment every 6 to 10 weeks (AAO), supportive periodontal care every 3 to 12 months (EFP), implant placement and loading gates (ITI), endodontic healing check at 6 to 12 months (AAE), child recall every 6 months or 3 if high risk (AAPD), adult routine recall 3 to 24 months (NICE).
- No WebCeph-style tool was confirmed for endodontics, oral surgery or pediatrics. Other specialties have file-based or device tools.
- Indonesian dental products publish Rp149.000 to Rp1.499.000 per clinic per month. Their public pages show no phased treatment plans, lab-case tracking, trigger-based recall, waitlist or specialty visit templates.

**Inventory, assets and finance (section 3)**

- Partner-dentist fees carry a tax rule that changes the design: under PMK 168/2023 the PPh 21 withholding base is 50% of the doctor fee paid by patients, before the clinic's share. The tax base therefore differs from the payout, and the fee engine must carry both numbers. Withholding slips are electronic (BP21).
- Dentist and clinic services are VAT-exempt under PP 49/2022.
- Device regulation changed in May 2026: Permenkes 5/2026 revoked the earlier izin edar and calibration regulations, so the calibration interval has to be configurable, not hard-coded.
- QRIS is capped at Rp10 million per transaction, so large orthodontic or implant invoices need another channel.
- Dedicated dental procurement tools deplete stock by scan, tag or count. Only clinic systems deduct materials per procedure, which is what cost-per-procedure and ROI need.
- Dental equipment carries a compliance calendar: twice-yearly ASPAK updates, BAPETEN records for X-ray units, sterilizer cycle logs.
- Fiscal depreciation rules name no dental equipment, so the platform needs a tenant-selectable fiscal group and a separate management life for ROI.
- Recommended **[INFERENCE]**: one stock ledger, a domain sub-ledger with journal export to an accounting product instead of an in-house general ledger, and money that never transits the platform's own account. Section 3h lists a proposed finance feature set in order of value.
- Every KPI benchmark found is a US figure from the ADA. No Indonesian benchmark was found.

**Indonesian regulation and SATUSEHAT (section 4)**

- Electronic medical records are mandatory for every tenant type, including solo dentists; the deadline passed on 2023-12-31.
- Health data must be stored and processed in Indonesia, backups included.
- The vendor needs health-sector PSE registration and must register and verify the record system with Kemenkes/SATUSEHAT before tenants can select it.
- Record rules: NIK at registration; name, time and signature per entry; corrections only within 2×24 hours; retention at least 25 years from the last visit.
- The operative dental data standard is the SATUSEHAT dental module: FDI numbering and the odontogram as coded observations.
- Health data is specific personal data under the PDP law: DPIA, 3×24-hour breach and data-subject clocks, written approval for sub-processors.
- Advertising and ethics rules prohibit discounts, testimonials, superlatives, and recruiting patients through agents. Clinical advice over chat is telemedicine, reserved to licensed clinicians.

**WhatsApp agentic operations (section 5)**

- A business's own customer-service AI is allowed on the WhatsApp Business Platform. Meta's own Business Agent excludes the health vertical.
- Since 2026-10-01 free-form service replies are charged beyond 1,000 per number per month. Indonesia list rates: marketing IDR 586.33; utility and service IDR 356.65 per message.
- In the track's cost model (300 enquiries and 400 appointments a month) Meta fees are about IDR 1.67 million per clinic per month, far above the model cost **[INFERENCE]**.
- Recommended path: become a Meta Tech Provider and run Embedded Signup, with coexistence so clinics keep their existing WhatsApp Business app number **[INFERENCE]**.
- WhatsApp Flows can do in-chat booking with a calendar picker, but needs each clinic's Meta business verification.
- Chatwoot's MIT core covers a shared inbox; roles, branding and AI features are paid. n8n and Typebot licences restrict tenant-facing commercial use.
- Text reminders raise attendance (Cochrane, RR 1.14); multiple reminders beat single ones.

**Platform architecture (section 6)**

- TanStack Start is stable and self-hosts on Node through the Nitro plugin. `/workbuddy` is safer as a route subtree of the same app, because base-path handling has an open upstream issue.
- Recommended shape **[INFERENCE]**: PostgreSQL with a shared schema and forced row-level security; exclusion constraints against double-booking; Better Auth for organizations, roles, passkeys and TOTP; pg-boss for jobs; Postgres NOTIFY to server-sent events for the live inbox and calendar; Kamal for deploys; Cloudflare in front.
- Single sign-on with the main site does not come for free. The workable first step is to verify a Firebase ID token on the workspace and issue the workspace's own session.
- Writes to the directory should go through a new versioned API on the main product, since direct Firestore writes would bypass its moderation and claim rules.
- MinIO is archived and unmaintained. Multi-resource calendar views are paid in FullCalendar and Schedule-X.

## Where the request collides with existing rules and the law

| The request | What it collides with | Section | Ways through **[INFERENCE]** |
| --- | --- | --- | --- |
| AI agent does "closing" | Permenkes 1787/2010 Ps. 5 bans discounts, testimonials, superlatives and encouraging use of a provider; the PDGI ethics code bans recruiting patients through agents; the main product bans promotion and offers in dentist-facing features | 4f, 1b | Scope the agent as the clinic's administrative assistant: availability, booking, recall, neutral service and price information. Get a lawyer's and an ethics view before any sales behaviour |
| Agent sends messages by itself | dentalclinic's recorded decision: the agent only proposes, a human approves and sends | 1d | Make autonomous sending an explicit recorded decision, possibly split by message type |
| Agent answers patients' questions | Clinical advice at a distance is telemedicine, reserved to STR/SIP holders in a facility | 4d, 4f | Refuse-and-hand-over for anything clinical; disclose that it is automated |
| Chat handled by a hosted model | Health data must be processed in Indonesia; transfer abroad needs a PDP mechanism and possibly government permission | 4d, 4e | An AI gateway that keeps the record in Indonesia and strips identifiers and clinical detail; lawyer to confirm |
| "A VPS I have" | Health data, backups and logs must sit in an Indonesian data centre, with a second Indonesian location for backups | 4a, 4d | Confirm where the VPS is; plan a second in-country site |
| Radiographs in object storage | Cloudflare R2 has no Indonesian placement; MinIO is archived | 6f, 4d | An S3-compatible store hosted in Indonesia; decision open |
| Manage the directory page from the workspace | No API, cookie and App Check only, one claimant per place | 1b, 6c | A versioned service API on the main product; grow the Claim into "one tenant, many staff" |
| One login across both sites | Host-only cookie; Firebase cannot act as a redirect-based identity provider | 1b, 6c | Same Google identity with separate sessions first; a redirect hand-off later |
| WebCeph as the orthodontic benchmark | Its API returns no results; it processes data in Korea | 2a | Link out and attach exports; build the staged orthodontic case natively |
| Reuse the design system | Tokens only; no back-office components; dark mode never shipped | 1a, 6h | Copy tokens and font, add stock shadcn components, publish a registry later |
| Role names in the request | Both glossaries define other names and avoid "admin", "owner" and "reminder" | 1c | Write the workspace glossary before specs |
| Budget for the AI agent | Meta's per-message fees, not model cost, dominate | 5a | Fewer, denser messages; staff replies from the phone app are free under coexistence |

## Decisions only the owner can make

1. **Agent autonomy.** Does the WhatsApp agent send on its own, or does a human approve each message? (Sections 1, 5c) The sales-scope half of this question is decided: see "Owner decisions so far".
2. **Hosting location.** Is the VPS in an Indonesian data centre, and where does the second backup site go? (4d, 6g) Decided: see "Owner decisions so far".
3. **Model hosting.** Offshore model API with pseudonymisation, or an in-country model? (4e) Decided: see "Owner decisions so far".
4. **Tenant model.** Is a tenant one Provider (`placeId`), or can it hold several? Can a hospital dental unit be a tenant? Can one person belong to several tenants? (1e, 6c)
5. **Role names and scope.** Canonical names for the five roles, what a viewer sees, and whether a partner dentist sees only patients in their care. (1c, 2d, 6c)
6. **Sign-in.** Reuse the main product's Firebase project (Google only), or a workspace identity that also covers staff without Google accounts? (1b, 6c)
7. **Directory integration.** May the workspace write to the main product's Firestore, or must the main product expose an API first? (1e, 6c)
8. **`/workbuddy`.** Expert landing and pairing only, or a new multi-tenant MCP server? Do the existing Experts keep pointing at dentalclinic? (1d)
9. **Relationship between the two live products.** Separate audiences, cross-links, or shared Experts? (1e)
10. **Specialties first.** Which specialties do the first partner dentists practise? Nine stage templates at once is not justified without that mix. (2b)
11. **Meta onboarding.** Become a Tech Provider, or go through a Solution Partner? Depends on which payment methods Indonesian clinics can attach. (5b)
12. **Inbox.** Adopt Chatwoot's core behind the workspace UI, or build a narrower inbox? (5e)
13. **Brand.** The main product's teal and Figtree, or dentalclinic's green and Instrument Sans? Does dark mode ship? (1a)
14. **SATUSEHAT timing.** Verified at launch, or launch without and add it? Tenants face accreditation-linked sanctions without an integrated record system. (4a, 4c) Decided: future backlog, see "Owner decisions so far".
15. **Fee contracts.** How do target clinics contract partner dentists today: the base, percentages by procedure, guarantees, and who bears discounts and gateway fees? No public primary source answers this; it needs the owner's own data. (3d)
16. **Payments.** Which gateway first, and does money ever pass through the platform? The research recommends each tenant connects its own merchant account. (3e)
17. **Accounting.** Which accounting product do target clinics use, if any? That decides the export target. (3f)

## Owner decisions so far

Recorded 2026-11-25, in reply to the summary above.

| Topic | Decision | Effect on the work |
| --- | --- | --- |
| Agent sales scope | Agreed: the WhatsApp agent is the clinic's administrative assistant (availability, booking, recall, neutral service and price information). No "closing" behaviour | Discounts, testimonials, superlatives and persuasion are out of the agent's scope. Whether it sends on its own is still open (decision 1) |
| Regulatory registrations (health-sector PSE, Kemenkes and SATUSEHAT verification) | Future backlog, not part of the first implementation | Tracked in [findyourdent-workspace#1](https://github.com/ahmadwaliradhi/findyourdent-workspace/issues/1) |
| Data residency and offshore model use | Noted. Not treated as a constraint on the implementation | Storage, backup and model choices are made on engineering grounds. The findings in section 4 stand as research |
| Main-domain work (service API, tenant link, sign-in support) | Gets its own ticket | Tracked in [findyourdent-workspace#2](https://github.com/ahmadwaliradhi/findyourdent-workspace/issues/2) |

**[INFERENCE]** Keeping object storage and the model call behind adapters makes a later move in-country a configuration change, not a rewrite.

## Suggested order of work

All of this is **[INFERENCE]** from the sections below.

1. **Before code.** Settle the decisions still open above. Write the workspace glossary. Agree the main-domain work in [findyourdent-workspace#2](https://github.com/ahmadwaliradhi/findyourdent-workspace/issues/2), since step 2 depends on it. The regulatory filings are on the backlog ([findyourdent-workspace#1](https://github.com/ahmadwaliradhi/findyourdent-workspace/issues/1)).
2. **Foundation.** Tenancy, sign-in, the five roles, audit trail, consent registry, and directory-page management through an API on the main product.
3. **Core practice.** Patient registry, dentist × chair calendar, the record with an odontogram in the SATUSEHAT structure, invoices with payment allocation.
4. **Agent.** WhatsApp inbox under coexistence, an administrative agent for booking and recall, human handoff.
5. **Depth.** Treatment-case engine for the specialties the first dentists practise, the recall engine, the partner-dentist fee engine with PPh 21, the stock ledger, the asset register with its compliance calendar, SATUSEHAT submission, and the `/workbuddy` MCP server for tenants.

## What stayed unverified and matters most

- PP 33/2026 (PDP implementing regulation) and the status of the PDP supervisory agency: reported by secondary sources only. (4e)
- Whether Permenkes 24/2022, Permenkes 1787/2010 and Permenkes 290/2008 are still in force unchanged. (4a, 4f)
- Whether patient chat content may be processed by an offshore model or messaging provider, and under which mechanism. (4e)
- Whether a public STR/SIP lookup or API exists. (4d)
- Which payment methods Meta's Billing Hub accepts for Indonesian clinics; whether Indonesia is a Cloud API local-storage region. (5)
- WebCeph's partner API approval and its future price. (2a)
- Which commit is live on each existing site, and whether the workspace host exists in DNS or in Firebase's authorized domains. (1)
- Per-study sizes for intraoral, panoramic and CBCT images; the licence of the free shadcn resource calendar. (6)
- What PER-12/PJ/2026 changed in the withholding-slip rules; whether "dokter" in PMK 168/2023 covers dentists in tax-office practice; where the equipment calibration duty sits now that Permenkes 54/2015 is revoked; whether a clinic holding chair-side anesthetics needs a pharmacy unit with a pharmacist. (3)

Each section ends with its own fuller list of open questions.


---

## 1. Existing systems: findyourdent.com and dentalclinic.web.id

Read-only pass on 2026-11-25 over the two local repositories the owner confirmed as the code behind findyourdent.com and dentalclinic.web.id. Code outranks docs; every disagreement found is stated. dentalclinic.web.id is **not** being shut down: the workspace is its improved successor and the two coexist, so this section treats it as a source of decisions and reusable parts, not as a migration source.

**Checkouts read and citation aliases.** Neither repository's default on-disk checkout is current, so each fact names its checkout.

| Alias in citations | What it is |
| --- | --- |
| `findyourdent/…` | Main checkout on disk: branch `main` @ `0ceeced` (2026-10-18), 138 commits behind `origin/main`, 19 files with uncommitted design edits (`git status`, `git rev-list --left-right --count main...origin/main`). Also holds the untracked `findyourdent-design-handoff/` |
| `findyourdent@om/…` | Committed content of `origin/main` @ `a2eb5f3` (2026-11-25). Read with `git -C findyourdent show origin/main:<path>`; also checked out at `findyourdent/.claude/worktrees/mattpocock-skills-impl-c1d874/`. That worktree carries uncommitted work in 22 files (a violation-badge feature), so on-disk lines differ for `packages/contracts/src/firestore.ts`, `infra/gcp/README.md`, `apps/web/app/server/provider-page.server.ts` and `apps/web/app/routes/tempat.$placeId.tsx`. Line numbers here are the committed ones; the uncommitted work is excluded |
| `dentalclinic/…` | Main checkout: branch `feat/13-landing-page` @ `3a1ef1c`. Scaffold plus the first landing page only (`dentalclinic/src/routes/` holds `__root.tsx` and `index.tsx`; the only migration is `dentalclinic/db/migrations/0001_scaffold.sql:1-5`) |
| `dentalclinic@mp/…` | `dentalclinic/.claude/worktrees/mattpocock-skills-implement-ba76d2/` @ `f276275`, clean. Engine, MCP, Experts, ADRs, tests |
| `dentalclinic@ds/…` | `dentalclinic/.claude/worktrees/delete-all-sessions-16d3bb/` @ `0b8316a`, clean, one commit behind `origin/main`. Latest glossary, README and role rename |
| `dentalclinic@om/…` | Committed `origin/main` @ `56cd151`, read with `git show` (not checked out anywhere) |
| `dentalclinic@hr/…` | `dentalclinic/.claude/worktrees/dental-clinic-hackathon-research-4417f6/`, which holds only the untracked research note |

- Neither repository mentions the other or a workspace subdomain: `git grep -i` for `workspace.findyourdent` and `dentalclinic.web` over `findyourdent` `origin/main`, and for `findyourdent` over `dentalclinic` `origin/main`, returns nothing.
- Both repositories are public on GitHub (`findyourdent@om/AGENTS.md:15`, `dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:70`).

### a. Design system of the main product

**Where it is defined**

- Design package, tracked: `findyourdent/docs/design/` with `HANDOFF.md`, `design-system/{README.md, tokens.json, tokens.css, components/, assets/Brand/, fonts/}` and `mockups/html/` (inventory at `findyourdent/docs/design/HANDOFF.md:13-24`). The token files, the brand book and `HANDOFF.md` are identical on `main` and `origin/main`; only `components/Card/README.md` changed since (`git diff --name-status main origin/main -- docs/design`).
- `findyourdent/findyourdent-design-handoff/` is the same package before it was committed: untracked (0 tracked files), `HANDOFF.md`, `tokens.css` and `tokens.json` byte-identical to `docs/design/` (`diff -q`), plus two folders that were not committed: `mockups/screenshots/{desktop,mobile}/` (24 PNG each) and `mockups/source/` (canvas `*.dc.html`, `canvas.json`). Spec 08 confirms the package was committed without them (`findyourdent@om/docs/specs/08-design-revamp.md:11`); HANDOFF says the canvas files are not needed for implementation (`findyourdent/docs/design/HANDOFF.md:24`).
- `tokens.json` is the declared source of truth, with light and dark values (`findyourdent/docs/design/HANDOFF.md:16`); `tokens.css` is generated from it (`findyourdent/docs/design/design-system/tokens.css:1`).
- Runtime: `findyourdent@om/apps/web/app/app.css` holds the tokens as CSS variables and maps them to the Tailwind theme; components are 9 files in `findyourdent@om/apps/web/app/components/ui/` (`badge`, `button`, `card`, `error-notice`, `field`, `notice`, `testimony-card`, `textarea`, `urgency-notice`; `git ls-tree origin/main`).

**Stack**

| Item | Fact | Source |
| --- | --- | --- |
| Tailwind | 4.3.3 through `@tailwindcss/vite`; CSS-first, no config file (`"config": ""`); theme in `@theme inline` | `findyourdent@om/apps/web/package.json:49,55`, `findyourdent@om/apps/web/components.json:7`, `findyourdent@om/apps/web/app/app.css:85-133` |
| shadcn | style `new-york`, base color `neutral`, CSS variables on, alias root `~/` | `findyourdent@om/apps/web/components.json:3,9-10,14-18` |
| Primitive base | `radix-ui` 1.8.1 (unified package), used only for `Slot` in Button; `class-variance-authority`, `clsx`, `tailwind-merge` | `findyourdent@om/apps/web/package.json:28-29,37,43`, `findyourdent@om/apps/web/app/components/ui/button.tsx:2`, `findyourdent@om/apps/web/app/lib/utils.ts:4-6` |
| Components | Hand-written in shadcn style (cva, `data-slot`), not stock shadcn: Button variants `primary, secondary, outline, ghost, danger, link`, sizes `sm, md, lg` at 44/48/56 px, plus `loading`, `icon`, `block` | `findyourdent@om/apps/web/app/components/ui/button.tsx:14-33,37-44` |
| Icons | `lucide-react` 1.48.0, stroke 1.75, 20 or 24 px, fixed status-icon mapping. HANDOFF's "lucide-react is not installed" is stale | `findyourdent@om/apps/web/package.json:36`, `findyourdent/docs/design/design-system/README.md:43`, `findyourdent@om/apps/web/app/components/ui/badge.tsx:28-33`, `findyourdent/docs/design/HANDOFF.md:102` |
| Font | Figtree variable (300 to 900), SIL OFL 1.1, self-hosted at `/fonts/Figtree-Variable.woff2`, preloaded | `findyourdent@om/apps/web/app/app.css:17-23`, `findyourdent@om/apps/web/app/root.tsx:23`, `findyourdent/docs/design/design-system/README.md:29` |
| Dark mode | Dark values exist under `[data-theme="dark"], .dark`; the app ships light only | `findyourdent/docs/design/design-system/tokens.css:37-69`, `findyourdent@om/apps/web/app/app.css:26`, `findyourdent/docs/design/HANDOFF.md:9` |
| Framework and lint | React Router 7.18.4 (SSR on Express), Biome 2.5.15. Both differ from the workspace's stated stack | `findyourdent@om/apps/web/package.json:24,31,41`, `findyourdent@om/package.json:11-12,19` |

**Color tokens** (`findyourdent/docs/design/design-system/tokens.css:5-31` light, `:39-65` dark; the light values are repeated in `findyourdent@om/apps/web/app/app.css:30-56`)

| Token | Light | Dark | Token | Light | Dark |
| --- | --- | --- | --- | --- | --- |
| `surface-100` (page) | `#f7f5f0` | `#0d1a1d` | `danger` | `#b3261e` | `#f08a82` |
| `surface-200` (card) | `#ffffff` | `#14262a` | `on-danger` | `#ffffff` | `#2a0806` |
| `surface-300` (sunken) | `#ece8df` | `#091416` | `danger-soft` / `-line` | `#fdebe8` / `#c4463d` | `#3a1815` / `#a8524c` |
| `ink` | `#12262b` | `#eaf2f0` | `warning` | `#774600` | `#f2c25c` |
| `ink-muted` | `#4b5c60` | `#a9bcbc` | `warning-soft` / `-line` | `#fff3d1` / `#a8740a` | `#35290a` / `#9a7418` |
| `border` | `#d8d3c6` | `#26403f` | `success` | `#1b6b3c` | `#6fd09a` |
| `border-strong` | `#7a8788` | `#6f8b8b` | `success-soft` / `-line` | `#e3f4ea` / `#3f8f5f` | `#12301f` / `#3c9a64` |
| `brand` | `#0b6e6a` | `#5cc8bf` | `info` | `#1d5a94` | `#8dbdf0` |
| `brand-hover` | `#085753` | `#7ad6ce` | `info-soft` / `-line` | `#e5eff9` / `#4d86bd` | `#12283d` / `#3f78ad` |
| `on-brand` | `#ffffff` | `#052624` | `star` | `#a86a00` | `#f0b429` |
| `brand-tint` / `on-brand-tint` | `#dcefec` / `#084a47` | `#143c3a` / `#bfeee9` | `focus-ring` | `#0b6e6a` | `#5cc8bf` |

- `highlight: #ffe08a` exists only in the app stylesheet, not in the token files (`findyourdent@om/apps/web/app/app.css:57`).
- Type scale, as size/line-height, weight, tracking (`findyourdent/docs/design/design-system/tokens.css:98-124`): display 40/44 700 -0.02em; h1 30/36 700 -0.015em; h2 24/30 700 -0.01em; h3 20/26 650; h4 17/24 650; body-lg 18/28; body 16/24; body-strong 16/24 600; body-sm 14/20; label 14/20 600; caption 12/16 500; overline 12/16 700 0.08em; price 22/28 700; score 16/20 700.
- Spacing on a 4 px base: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64; touch target 44 px (`findyourdent/docs/design/design-system/tokens.css:72-82`).
- Radius: sm 6, md 10, lg 14, xl 20, pill 9999 px (`findyourdent/docs/design/design-system/tokens.css:83-87`); in the app `--radius: 10px` feeds `--radius-md` (`findyourdent@om/apps/web/app/app.css:64,87-90`). Usage: buttons, inputs and notices md; cards lg; chips pill (`findyourdent/docs/design/design-system/README.md:33`).
- Shadows, light: xs `0 1px 1px rgba(18,38,43,0.05)`; sm `0 1px 3px rgba(18,38,43,0.08), 0 1px 2px rgba(18,38,43,0.04)`; md `0 8px 20px -6px rgba(18,38,43,0.16), 0 2px 4px rgba(18,38,43,0.06)`. Dark: `0 1px 1px rgba(0,0,0,0.4)`, `0 1px 3px rgba(0,0,0,0.5)`, `0 8px 20px -6px rgba(0,0,0,0.6)` (`findyourdent/docs/design/design-system/tokens.css:32-34,66-68`). The app stores them as `--shadow-*-value` to avoid colliding with Tailwind's own names (`findyourdent@om/apps/web/app/app.css:58-61,91-93`).
- shadcn mapping: `--background` = surface-100; `--card`, `--popover` = surface-200; `--foreground` = ink; `--muted` = surface-300; `--muted-foreground` = ink-muted; `--primary` = brand; `--secondary`, `--accent` = brand-tint; `--destructive` = danger; `--input` = border-strong; `--ring` = focus-ring (`findyourdent@om/apps/web/app/app.css:64-82`, `findyourdent/docs/design/design-system/README.md:49-57`).
- App-only extensions: `--color-primary-hover`, `-soft` and `-line` color families, `--spacing-touch: 44px`, `--breakpoint-desktop: 900px`, a `desktop-unfolded` custom variant, and one global `:focus-visible` ring of 3 px with 2 px offset (`findyourdent@om/apps/web/app/app.css:9-15,94-96,106,115-116,122-132,146-149`).
- Layout: mobile-first at 360 px; content max 1200 px, gutter 16 px, 40 px from 900 px (`findyourdent/docs/design/HANDOFF.md:9,55`, `findyourdent@om/apps/web/app/components/site-shell.tsx:11`). Docs disagree: the brand book still says one 672 px column (`findyourdent/docs/design/design-system/README.md:31`). Code wins.
- Hard rules: no raw hex or Tailwind palette colors in components; 44 px touch targets; text never under 12 px; every status carries an icon and a word; money in tabular figures as `Rp 450.000`; `prefers-reduced-motion` respected (`findyourdent/docs/design/HANDOFF.md:86-92`).
- Brand assets: logo mark (3 variants), wordmark (2), favicon, app icons, `og-image.png` (`findyourdent/docs/design/design-system/assets/Brand/README.md:3-9`); served from `apps/web/public/` and `public/brand/` (`findyourdent/docs/design/HANDOFF.md:19`).
- Reference components `bundle.js`, `bundle.css`, `index.d.ts` cover Button, Badge, Card, Field, UrgencyNotice, TestimonyCard; HANDOFF says to rebuild them with Tailwind and not ship the bundle (`findyourdent/docs/design/HANDOFF.md:18`).

**Portability to a separate repository**

- What exists: no shared UI package (the repository's only packages are `apps/web`, `packages/contracts` and `services/agent`), no shadcn `registry.json`, no Tailwind config file (`git ls-tree -r origin/main`). The UI lives inside `apps/web` behind the `~/` alias.
- Portable without change: `tokens.json` and `tokens.css` (plain custom properties), the `:root` and `@theme inline` blocks of `app.css`, the font file, the brand SVG and PNG files, and `mockups/html/fyd.css`.
- Not portable as-is: the 9 `ui/*.tsx` files import `~/lib/utils` and, for `urgency-notice`, a type from `@findyourdent/contracts` (`findyourdent@om/apps/web/app/components/ui/urgency-notice.tsx:1-3`). They cover patient-facing screens only; no table, dialog, select, tabs, calendar or sidebar component exists (`git ls-tree origin/main apps/web/app/components/ui`).
- dentalclinic already runs the workspace's intended stack (TanStack Start, Tailwind 4.3.5, shadcn `new-york`/`neutral`, oxlint, oxfmt) with a different, greener palette and Instrument Sans from Google Fonts: `--primary: #0b6e5f`, `--radius: 0.75rem` (`dentalclinic@mp/package.json:24-25,43-45`, `dentalclinic@mp/components.json:3,9`, `dentalclinic@mp/src/styles.css:5,35-57`, `dentalclinic@ds/src/routes/__root.tsx:19`).
- **[INFERENCE]** Cheapest path: copy the `:root` and `@theme inline` blocks, the font and the brand files into the workspace, then add stock shadcn components. Stock components will pick up the brand through the standard variable names, but anything that calls `buttonVariants({ variant: "default" })` or `size: "icon"` needs adapting, because the main product's Button has no `default`, `destructive` or `icon` values.
- **[INFERENCE]** Durable path: publish tokens, font and the base components from the public main repository as a shadcn registry or a small versioned package generated from `tokens.json`, so both products track one source. Neither exists today.
- **[INFERENCE]** A back office needs tokens the patient site never defined (sidebar, chart, table density) and a decision on dark mode, whose values exist but have never shipped.

### b. Main product's provider domain

**What the product is.** A patient-facing site: a Patient describes a complaint, gets Guidance with Emergency Escalation, then a Shortlist of nearby Providers with Testimonies and price signals (`findyourdent@om/README.md:4`). Its tagline already names dentists and clinics as an audience (`findyourdent@om/README.md:2`, `findyourdent@om/GLOSSARY.md:3`).

**Provider types and identity**

- Three types: `klinik`, `privateDentist`, `hospital` (`findyourdent@om/packages/contracts/src/firestore.ts:53`; definitions at `findyourdent@om/GLOSSARY.md:11-25`).
- Without a Claim the type is inferred: Google type `hospital`, else a name starting `drg.` or `dokter gigi` means Private Dentist, else Klinik (`findyourdent@om/apps/web/app/lib/provider-type.ts:5-11`).
- A Provider is keyed by its Google `placeId`, which is also the Firestore document id; a place not on Google Maps cannot be listed or claimed (`findyourdent@om/packages/contracts/src/firestore.ts:46-47`, `findyourdent@om/docs/adr/0005-hybrid-places-and-listed-providers.md:12`).
- A Listed Provider is created by its first published Testimony or by an approved Claim; only Listed Providers appear in the directory and feed (`findyourdent@om/GLOSSARY.md:61-63`).
- Google data is never stored beyond `placeId` and coordinates for up to 30 days (`findyourdent@om/packages/contracts/src/firestore.ts:4-6`).

**What a provider page contains**

| Group | Fields | Source |
| --- | --- | --- |
| Listed Provider record | `name` (+ `nameSource`), `region` (province, kabupaten/kota, optional kecamatan), `location`, `providerType` (+ source), `claimStatus`, `claimId`, `ownerFacts`, `pendingOwnerFacts`, `aggregates`, `listedBy` | `findyourdent@om/packages/contracts/src/firestore.ts:56-63,165-190` |
| Owner facts (after an approved Claim) | `name`, `providerType`, `region`, `address`, `phone`, `whatsapp`, `website`, `openingHours`, `services` (procedure codes), `moderationStatus` | `findyourdent@om/packages/contracts/src/firestore.ts:87-101` |
| Aggregates | Testimony count, Proven count, aspect averages (service, hygiene, materials and equipment), price stats per procedure, Payment Method counts | `findyourdent@om/packages/contracts/src/firestore.ts:72-77,148-162` |
| Page payload | Summary, price signals, owner Price List, owner facts, Google Maps link, up to 100 published Testimonies with the owner's reply | `findyourdent@om/apps/web/app/server/provider-page.server.ts:25,29-52,54-65` |
| Live Google block (shown, never stored) | Address, open-now, hours, phone, WhatsApp link, website, rating | `findyourdent@om/apps/web/app/lib/provider.ts:17-32` |

- Gap in code: `website` is in the stored type and passed to the page, but the owner-facts validator does not accept it, so an owner cannot currently set it (`findyourdent@om/packages/contracts/src/firestore.ts:94`, `findyourdent@om/apps/web/app/server/provider-page.server.ts:75`, `findyourdent@om/apps/web/app/server/claims.server.ts:73-76,286-290`).
- Owner-declared `services` are limited to the shared 27 procedure codes, at most 30 entries (`findyourdent@om/apps/web/app/server/claims.server.ts:292-294`, `findyourdent@om/apps/web/app/lib/claims.ts:23`; count from `findyourdent@om/packages/contracts/data/procedures.json`).

**How a provider claims and edits a listing today**

- Entry: a link on the provider page to `/tempat/:placeId/klaim` (`findyourdent@om/apps/web/app/routes/tempat.$placeId.tsx:247`, `findyourdent@om/apps/web/app/routes.ts:42`).
- Claim: sign in with Google; choose a role (`owner`, `practisingDentist`, `manager`); give a contact phone; upload one permit (SIP or izin praktik; JPEG, PNG, WebP or PDF up to 10 MB) (`findyourdent@om/apps/web/app/lib/claims.ts:6,10-12`, `findyourdent@om/apps/web/app/server/claims.server.ts:211-267`).
- Verification is manual: an admin reads the permit and phones the number on the Google listing, with a two-working-day target (`findyourdent@om/docs/specs/05-claim-and-price-list.md:49`, `findyourdent@om/apps/web/app/lib/claims.ts:15`). Claim states: `submitted`, `approved`, `rejected`, `revoked` (`findyourdent@om/packages/contracts/src/firestore.ts:394`).
- One active Claim per place and one claimant account per place, so a listing has exactly one editor; there are no team seats (`findyourdent@om/packages/contracts/src/firestore.ts:396-400`, `findyourdent@om/apps/web/app/server/claims.server.ts:5-11,142-154`).
- After approval the same `/klaim` page becomes the owner's management page: edit facts, reply once per Testimony, answer a policy-violation case, and open the Price List page (`findyourdent@om/apps/web/app/routes/tempat.$placeId.klaim.tsx:68-69,368-419,532-669`).
- Price List: the owner uploads one PDF or up to five photos at `/tempat/:placeId/daftar-harga`; Gemini extracts rows; the owner corrects and confirms; only confirmed rows are stored (`findyourdent@om/apps/web/app/server/price-lists.server.ts:1-11`, `findyourdent@om/packages/contracts/src/firestore.ts:420-453`).
- Owner facts and replies pass AI Moderation and are published or held for an admin (`findyourdent@om/apps/web/app/server/claims.server.ts:365-372,392-399`).
- There is no provider dashboard, no multi-location view and no onboarding beyond this flow. The admin side is one allowlisted `/admin` route with a queue and action API (`findyourdent@om/apps/web/app/routes.ts:32-36`).
- Standing constraints on every dentist-facing feature: no promotion tools, no sponsored placement, no discounts or offers, no badge other than "Diklaim pemilik", no way to remove or hide a Testimony (`findyourdent@om/docs/specs/05-claim-and-price-list.md:56`, `findyourdent@om/docs/adr/0002-providers-cannot-delete-testimonies.md:3`).
- Patient management, appointments and marketing tools for clinics were explicitly deferred (`findyourdent@om/docs/specs/05-claim-and-price-list.md:67`).

**Authentication and roles**

- Firebase Auth with Google as the only sign-in provider; the browser signs in with a popup and posts the ID token to `/auth/session` (`findyourdent@om/apps/web/server/auth.ts:6-7,36`, `findyourdent@om/apps/web/app/lib/firebase.client.ts:57`, `findyourdent@om/apps/web/app/routes/auth.session.ts:4-8`).
- The server issues a 5-day Firebase session cookie stored inside one `__session` cookie: `httpOnly`, `sameSite=lax`, no `domain` attribute, so it is host-only and is not sent to a subdomain (`findyourdent@om/apps/web/server/auth.ts:9,40-46`, `findyourdent@om/apps/web/server/browser-state.ts:30-35`). Firebase Hosting strips every other cookie (`findyourdent@om/apps/web/server/browser-state.ts:5-8`).
- No custom claims and no role table (`git grep -i "customClaims\|setCustomUserClaims" origin/main` is empty). The only roles are: admin, by an email allowlist from the `ADMIN_EMAILS` secret; approved claimant of one place; signed-in Patient (`findyourdent@om/apps/web/server/auth.ts:71-75,87-104`, `findyourdent@om/apps/web/server/context.ts:84,147-150`).
- Paid routes also require an App Check token (reCAPTCHA Enterprise) and pass rate limits; owner writes are capped at 30 a day per account (`findyourdent@om/apps/web/app/server/protection.server.ts:13-23`, `findyourdent@om/infra/gcp/README.md:23`, `findyourdent@om/apps/web/server/context.ts:51`).

**Data stores**

- Firestore, Native mode, Jakarta; 13 top-level collections: `providers`, `testimonies`, `testimonyReports`, `claims`, `priceLists`, `moderationDecisions`, `nameCorrections`, `adminActions`, `counters`, `rateLimits`, `proofFingerprints`, `submissionSignals`, `policyViolations` (`findyourdent@om/packages/contracts/src/firestore.ts:29-43`, `findyourdent@om/infra/gcp/README.md:20`).
- Security rules deny every client read and write; all access goes through the web service with the Admin SDK (`findyourdent@om/firestore.rules:2-8`, `findyourdent@om/storage.rules:2-7`).
- Cloud Storage bucket `findyourdent-uploads`, private, with lifecycle deletes for Price List originals (30 days) and pending proofs (7 days) (`findyourdent@om/infra/gcp/README.md:21`).
- `adminActions` is an append-only audit log of admin decisions (`findyourdent@om/packages/contracts/src/firestore.ts:558-579`).
- Shared reference data in `@findyourdent/contracts`: 27 procedures, 38 provinces and 514 kabupaten/kota, a kecamatan list, 18 insurers; the package is `private` and exports TypeScript source (`findyourdent@om/packages/contracts/package.json:2-12`, `findyourdent@om/packages/contracts/src/index.ts:105-108,117-118,137`; counts read from the JSON files).

**Services and APIs the workspace could reach**

| Surface | What it does | Access today |
| --- | --- | --- |
| `POST /api/providers/:placeId/claims` | Submit a Claim with permit | Session cookie, App Check, rate limit (`findyourdent@om/apps/web/app/routes.ts:18`, `findyourdent@om/apps/web/app/server/claims.server.ts:124-136`) |
| `POST /api/providers/:placeId/owner-facts` | Save owner facts, moderated | Approved claimant only (`findyourdent@om/apps/web/app/routes.ts:19`, `findyourdent@om/apps/web/app/server/claims.server.ts:349-407`) |
| `POST /api/providers/:placeId/price-lists` and `/:priceListId` | Upload, then confirm, a Price List | Approved claimant only (`findyourdent@om/apps/web/app/routes.ts:24-25`) |
| `POST /api/testimonies/:testimonyId/reply` | One public reply per Testimony | Approved claimant only (`findyourdent@om/apps/web/app/routes.ts:16`) |
| `POST /api/providers/:placeId/policy-violations/:violationId/response` | Reply to a violation case | Approved claimant only (`findyourdent@om/apps/web/app/routes.ts:20-23`) |
| Agent service `fyd-agent`: `/v1/guidance`, `/v1/moderation`, `/v1/moderation/proof-of-visit`, `/v1/price-list-extraction`, `/v1/shortlist-reasons` | FastAPI plus Google ADK on Gemini | IAM-only; callable by the `fyd-web` service account alone (`findyourdent@om/services/agent/src/findyourdent_agent/app.py:116-117,151-152,168-169,187-188,203`, `findyourdent@om/services/agent/pyproject.toml:7-8`, `findyourdent@om/infra/gcp/README.md:107`) |

- No Cloud Functions: `firebase.json` configures only Hosting, Firestore, Storage and emulators (`findyourdent@om/firebase.json:2-31`).
- No partner API, API keys or CORS headers anywhere (`git grep -i "access-control\|cors" origin/main -- apps services` is empty). Every route is same-origin and cookie-based, so a server on another host cannot call them as they stand.

**Booking, leads, WhatsApp, AI**

- No booking, appointment or lead capture exists (`git grep -i "booking\|appointment\|janji temu\|reservasi"` over app, server and contracts code returns nothing relevant).
- WhatsApp appears only as `wa.me` links: a contact button on the provider page from the owner's or Google's mobile number, and a share link for Guidance (`findyourdent@om/apps/web/app/server/provider.server.ts:29-32`, `findyourdent@om/apps/web/app/routes/tempat.$placeId.tsx:112,152-154`, `findyourdent@om/apps/web/app/components/guidance-result.tsx:540`).
- AI in production: Guidance and Shortlist reasons on `gemini-3.8-flash`; Moderation, proof checks and Price List extraction on `gemini-3.5-flash-lite` (`findyourdent@om/services/agent/src/findyourdent_agent/config.py:15-18`). All of it is patient-facing or moderation; none is clinic operations.

**i18n and hosting**

- i18next with `id` (default) and `en`; language comes from the cookie, and `Accept-Language` is deliberately ignored (`findyourdent@om/apps/web/app/i18n/config.ts:6-8`, `findyourdent@om/apps/web/app/i18n/language.server.ts:4-11`). Each locale file is 1,535 lines (`git show origin/main:apps/web/app/i18n/locales/id.json | wc -l`).
- Hosting: Firebase Hosting rewrites every path to Cloud Run `fyd-web` in `asia-southeast2`; the agent runs as `fyd-agent` in the same region (`findyourdent@om/firebase.json:5-12`, `findyourdent@om/infra/gcp/README.md:19,88,106-107`).
- Domain: `findyourdent.com` is registered at Hostinger, which hosts its DNS; apex and `www` point at Firebase Hosting (`findyourdent@om/infra/gcp/README.md:103-104`).
- Firebase Auth authorized domains are a console-managed list of four hosts, none of them a `workspace` host (`findyourdent@om/infra/gcp/README.md:124`).
- Deploy is manual with `infra/gcp/deploy.sh`, which builds `HEAD` and refuses uncommitted changes (`findyourdent@om/infra/gcp/README.md:97`). A GitHub Actions deploy exists only on an unmerged branch (`findyourdent/.claude/worktrees/mattpocock-skills-issue-133-349c0e/docs/adr/0012-github-actions-deploys-main-to-cloud-run.md:3`).
- Cloud Run Jobs run coordinate refresh and proof expiry daily from Cloud Scheduler; a third job recomputes aggregates once, by hand (`findyourdent@om/infra/gcp/README.md:108-110`).
- The web service is a plain Node 24 Express server in a Docker image, so it is container-portable (`findyourdent@om/apps/web/Dockerfile:6,24,42`, `findyourdent@om/apps/web/server/app.ts:46-50`).

### c. Glossary: terms to reuse and conflicts

**Main product terms the workspace should reuse** (`findyourdent@om/GLOSSARY.md`)

| Term | Meaning | Line |
| --- | --- | --- |
| Patient | Member of the public looking for care. Avoid "user", "customer" | 7-9 |
| Klinik, Private Dentist, Hospital | The three kinds of provider. **[INFERENCE]** They map one-to-one to the workspace's three tenant types | 11-21 |
| Provider | Any of the three, identified by its Google place. Avoid "listing", "merchant" | 23-25 |
| Listed Provider | A Provider with a first-party record; the thing a directory page shows | 61-63 |
| Testimony, Proof of Visit, Proven Testimony | Patient account of a visit and its proof. "Review" is reserved for Google reviews | 29-39 |
| Testimony Policy Violation, Violation case | Confirmed or suspected incentive for Testimonies | 45-51 |
| Moderation, Hold | AI screening and the admin-decision state | 53-59 |
| Region | Province, kota/kabupaten, optional kecamatan, by Kemendagri code. "Location" is reserved for coordinates | 65-67 |
| Guidance, Emergency Escalation, Urgency Tier, Shortlist | Patient-side AI vocabulary; never "diagnosis" or "triage" | 71-85 |
| FKTP, Payment Method | BPJS first-level facility; how a visit was paid (Pribadi, BPJS, Asuransi swasta). Never "accepts BPJS" | 89-95 |
| Dental terms | 64 headwords for anatomy, conditions, procedures and orthodontics, with patient wording first | 97-311 |

- "Claim" and "Price List" are used throughout code and specs but have no glossary entry (`findyourdent@om/GLOSSARY.md:62` uses "Claim" inside another definition only).

**dentalclinic terms the workspace should reuse** (`dentalclinic@ds/GLOSSARY.md`)

| Term | Meaning | Line |
| --- | --- | --- |
| Role; Pemilik; Dokter Gigi Mitra; Perawat Gigi; Front Office | Access levels and what each sees | 39-61 |
| SIP | Five-year practice licence per clinic; warn 90 days before expiry | 51-53 |
| Recall; Due for Recall; Rencana Perawatan; Rencana Tindak Lanjut | Bringing a patient back; the rule that makes a patient due | 65-79 |
| WA Opt-in | Separate, timestamped consent for WhatsApp contact; no message is drafted without it | 81-83 |
| Proposal; Approval Decision; Recall Message; Stok Opname Request | Agent prepares, a human with the right Role accepts within 24 hours | 91-105 |
| Restricted; Refusal | Data a Role may not see; one fixed reply for both restricted and non-existent data | 107-109, 131-133 |
| Source; Source Conflict; Injection Flag; Topic Owner | Citable record types and how conflicts and embedded instructions are handled | 111-125 |
| Jejak Audit | Append-only record of every question, answer and decision | 127-129 |
| Aset; Payback; Margin per Tindakan; Overhead % | Equipment and the three finance measures | 21-35 |
| Identity Pairing; Demo Login | Linking an agent client to a staff member; one-click demo sign-in | 15-17, 87-89 |

**Conflicts and collisions**

| Topic | Main product | dentalclinic | Note for the workspace |
| --- | --- | --- | --- |
| Dental nurse role name | none | `TGM` in the main checkout and `@mp` (which lists "perawat gigi" under Avoid); `Perawat Gigi` at `@ds` and `origin/main`, by migration 0010 (`dentalclinic@mp/GLOSSARY.md:55-57`, `dentalclinic@ds/GLOSSARY.md:55-57`, `dentalclinic@ds/db/migrations/0010_perawat_gigi.sql:1-16`, `dentalclinic@ds/src/brain/staff.ts:4`) | Latest code says Perawat Gigi. The README still says TGM, even on `origin/main` (`dentalclinic@ds/README.md:75`); trust the code |
| Owner | Claim roles `owner`, `practisingDentist`, `manager` (`findyourdent@om/packages/contracts/src/firestore.ts:400`) | `Pemilik`, with "admin" and "owner" under Avoid (`dentalclinic@ds/GLOSSARY.md:43-45`) | The request's "superadmin/owner" needs one canonical name |
| Admin | The platform team's allowlisted accounts (`findyourdent@om/apps/web/server/auth.ts:71-75`) | "admin" is an avoided word; reception is `Front Office` (`dentalclinic@ds/GLOSSARY.md:59-61`) | The request's "operations admin" collides with both. "Viewer" exists in neither |
| Partner dentist | `Private Dentist` = runs an independent practice, a provider type (`findyourdent@om/GLOSSARY.md:15-17`) | `Dokter Gigi Mitra` = practises at a clinic under fee-share, a staff role (`dentalclinic@ds/GLOSSARY.md:47-49`) | Different things; one person can be both |
| Patient | Anonymous or Google-signed-in member of the public; health text is not stored (`findyourdent@om/GLOSSARY.md:7-9`, `findyourdent@om/docs/data-model.md:8`) | A registered record with No RM, NIK and phone (`dentalclinic@mp/db/migrations/0003_roles.sql:3-10`) | **[INFERENCE]** Same word, two lifecycle stages; the workspace needs a word for the pre-registration contact |
| Procedure codes | 27 English snake_case codes such as `tooth_filling`, `root_canal_treatment` (`findyourdent@om/packages/contracts/data/procedures.json`) | Six Indonesian codes: `scaling`, `komposit`, `pencabutan`, `psa`, `ortodonti`, `panoramik` (`dentalclinic@mp/db/seed.sql:221-228`) | Only `scaling` matches |
| Price vocabulary | Price List rows with min, max and unit; "price paid" from Testimonies (`findyourdent@om/packages/contracts/src/firestore.ts:420-427`) | One `tariff` and one `bmhp` per procedure (`dentalclinic@mp/db/migrations/0005_keuangan_aset.sql:4-10`) | A clinic tariff list is the natural source of a public Price List |
| Reminder and follow-up | none | Listed under Avoid; the term is Recall (`dentalclinic@ds/GLOSSARY.md:65-67`) | The request's own wording ("reminders, follow-ups") is the avoided wording |
| Product category | "Clinic management" avoided as a synonym for Klinik (`findyourdent@om/GLOSSARY.md:13`) | "ERP" and "clinic management system" avoided for the product (`dentalclinic@ds/GLOSSARY.md:7-9`) | The workspace is the thing both glossaries avoid naming |
| Language | Bilingual, `id` default (`findyourdent@om/apps/web/app/i18n/config.ts:6-8`) | Site Indonesian only; answers follow the question's language (`dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:12`) | Decide per surface |
| "Claim" | Ownership claim on a listing | none | **[INFERENCE]** Will collide with insurance or BPJS claims once finance exists |

### d. dentalclinic today, its decisions, and what "WorkBuddy" is

**What it is.** "Dental Clinic Brain": a staff-facing question-answering and proposal engine over the data of one fictional clinic, reached by staff through an external agent client, with a small Indonesian website for approvals, audit and read-only views (`dentalclinic@ds/README.md:5,9-13`). It was built for a hackathon with a same-day deadline (`dentalclinic@hr/docs/research/2026-11-07-hackathon-dental-clinic-research.md:1,11`): 46 of the 47 commits on `origin/main` are dated 2026-11-07 and none is later (`git log origin/main --format=%cd --date=short`).

**Stack and deploy**

- One TanStack Start app on one Cloudflare Worker with D1; a custom domain route for `dentalclinic.web.id` plus `workers.dev`; a cron at `0 17 * * *` (midnight WIB) (`dentalclinic@mp/wrangler.jsonc:3-18`, `dentalclinic@mp/package.json:24-25`).
- The Worker entry sends `/mcp` to the MCP handler and everything else to TanStack Start (`dentalclinic@mp/src/server.ts:6-13`).
- Tailwind 4.3.5, shadcn conventions, oxlint and oxfmt, Vitest inside workerd (`dentalclinic@mp/package.json:9-13,43-49`).
- GitHub Actions is the only path to production: checks on every pull request; on `main`, build, then remote D1 migrations, then upload, then a smoke test (`dentalclinic@mp/docs/adr/0003-deploy-on-merge-with-github-actions.md:3`, `dentalclinic@mp/.github/workflows/ci.yml:63-65,99,104`).

**Routes** (`dentalclinic@mp/src/routes/`)

| Path | Behaviour | Source |
| --- | --- | --- |
| `/` | Indonesian landing page: launch links, Expert zips, Demo Login, accountability sheet, test results, video | `dentalclinic@ds/README.md:234-239`, `dentalclinic@mp/test/page.test.ts:19-121` |
| `POST /demo/:id`, `/keluar` | Demo Login as one of four seeded staff; 7-day session; sign out | `dentalclinic@mp/src/routes/demo.$id.ts:9-18`, `dentalclinic@mp/src/brain/staff.ts:13,16` |
| `/persetujuan`, `POST /persetujuan/:id` | Approvals inbox; accept or decline by form post; hidden from Front Office | `dentalclinic@mp/src/routes/persetujuan.$id.ts:12-21`, `dentalclinic@mp/src/routes/_klinik.tsx:35` |
| `/pasien`, `/jadwal`, `/stok`, `/aset` | Read-only views filtered by Role and masked | `dentalclinic@mp/src/routes/_klinik.tsx:16-24`, `dentalclinic@mp/src/web.ts:22-28` |
| `/jejak-audit` | Audit log with a hash-chain integrity check | `dentalclinic@mp/src/web.ts:30-33` |
| `/perangkat`, `POST /pairing`, `POST /pairing/token` | Identity Pairing and device list | `dentalclinic@mp/src/routes/pairing.ts:8-11`, `dentalclinic@mp/src/routes/pairing.token.ts:14-19` |
| `POST /mcp` | Stateless MCP server behind a bearer token | `dentalclinic@mp/src/mcp.ts:11-16,87-89` |

- There are no create or edit screens for patients, visits, stock or assets, by decision (`dentalclinic@mp/docs/adr/0002-brain-not-erp.md:3`).
- There is no real sign-up or authentication: only Demo Login for `rina`, `andi`, `tari`, `nina` (`dentalclinic@mp/src/brain/staff.ts:16-29`, `dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:39`).
- It is single-tenant: no table carries a clinic or tenant column (all of `dentalclinic@mp/db/migrations/`).

**Data model** (D1, SQLite `STRICT`)

| Table | Columns that matter | Source |
| --- | --- | --- |
| `staff` | `id`, `name`, `role` (CHECK on four roles) | `dentalclinic@mp/db/migrations/0002_klinik_brain.sql:4-8` |
| `sessions`, `tokens`, `pairings` | SHA-256 hashes only; token scope `read propose`, expiry | `dentalclinic@mp/db/migrations/0004_web.sql:4-8`, `dentalclinic@mp/db/migrations/0007_identity_pairing.sql:5-21` |
| `patients` | `no_rm`, `name`, `nik`, `phone`, `dentist_id`, `wa_opt_in_at` | `dentalclinic@mp/db/migrations/0003_roles.sql:3-10` |
| `visits` | patient, sequence, dentist, assistant, date, free-text `notes`, follow-up date | `dentalclinic@mp/db/migrations/0003_roles.sql:12-21` |
| `rencana_perawatan` | treatment, total and completed visits | `dentalclinic@mp/db/migrations/0003_roles.sql:23-29` |
| `appointments` | patient, dentist, `starts_at` only | `dentalclinic@mp/db/migrations/0004_web.sql:10-15` |
| `fee_rules` | one `share_percent` per dentist | `dentalclinic@mp/db/migrations/0003_roles.sql:32-35` |
| `procedures`, `procedure_counts` | code, name, tariff, `bmhp`; monthly counts per dentist | `dentalclinic@mp/db/migrations/0005_keuangan_aset.sql:4-10,26-32` |
| `aset`, `aset_procedures` | name, price, purchase date; procedures that use it | `dentalclinic@mp/db/migrations/0005_keuangan_aset.sql:12-23` |
| `monthly_books` | `overhead`, `collections` per closed month | `dentalclinic@mp/db/migrations/0005_keuangan_aset.sql:35-39` |
| `stock_items` | item, quantity, unit, last count date | `dentalclinic@mp/db/migrations/0004_web.sql:18-23` |
| `sources`, `source_search`, `source_access` | Citable records by URI with FTS5 index; `area`, `subject`, `fact`, `expires_on`; per-staff grants | `dentalclinic@mp/db/migrations/0002_klinik_brain.sql:17-41`, `dentalclinic@mp/db/migrations/0003_roles.sql:40-49` |
| `proposals` | kind `recall_message` or `stok_opname`, text, proposer, expiry, decision | `dentalclinic@mp/db/migrations/0009_stok_opname.sql:3-17` |
| `jejak_audit`, `jejak_audit_chain` | One row per question, proposal and decision; `prev_hash`, `hash` | `dentalclinic@mp/db/migrations/0008_proposals.sql:19-37`, `dentalclinic@mp/db/migrations/0006_jejak_audit_chain.sql:5-11` |

- All data is a fictional seed for "Klinik Gigi Senyum Sejahtera", restored every night; only the audit log survives the reset (`dentalclinic@mp/db/seed.sql:1`, `dentalclinic@mp/src/brain/nightly.ts:9-17`).
- The research note sketched a much wider model (clinics, odontogram, invoices, stock movements); it was not built (`dentalclinic@hr/docs/research/2026-11-07-hackathon-dental-clinic-research.md:350-352`). The same note holds a competitor table and regulation notes (`:171-232`) that belong to other tracks and were not re-verified here.

**Engine behaviour**, each rule pinned by a test

| Rule | Source | Test |
| --- | --- | --- |
| Role access is an SQL guard applied to every query, tool and view | `dentalclinic@mp/src/brain/access.ts:15-29,44-57,64-78` | `dentalclinic@mp/test/roles.test.ts:17-88` |
| Restricted and non-existent data get the identical Refusal; the true reason is only in the audit row | `dentalclinic@mp/src/brain/ask.ts:42-65` | `dentalclinic@mp/test/roles.test.ts:30,60` |
| Answers are composed only from retrieved text, each sentence checked against its Source and numbered | `dentalclinic@mp/src/brain/ask.ts:79-132` | `dentalclinic@mp/test/ask.test.ts:112` |
| Keyword retrieval on FTS5, no model and no vectors; a Source must match at least half the keywords | `dentalclinic@mp/src/brain/search.ts:33-59` | `dentalclinic@mp/test/ask.test.ts:47-63` |
| NIK and phone keep their last 4 digits, email keeps its domain | `dentalclinic@mp/src/brain/mask.ts:1-7` | `dentalclinic@mp/test/roles.test.ts:88` |
| Due for Recall: unfinished plan and over 14 days since last visit, or a passed follow-up date, and no visit booked | `dentalclinic@mp/src/brain/recall.ts:28-75` | `dentalclinic@mp/test/recall.test.ts:17` |
| Recall Message only for patients with WA Opt-in; text carries no clinical detail | `dentalclinic@mp/src/brain/recall.ts:117-151` | `dentalclinic@mp/test/recall.test.ts:51-71` |
| The agent only proposes; a human accepts on the site; an accepted Recall Message shows a `wa.me` link the human taps | `dentalclinic@mp/src/brain/proposals.ts:143,180-183`, `dentalclinic@mp/src/mcp.ts:50` | `dentalclinic@mp/test/approvals.test.ts:26-92` |
| Finance measures with formula and Sources: Payback, Margin per Tindakan (tariff − BMHP − dentist fee), overhead against a 59% reference | `dentalclinic@mp/src/brain/kpi.ts:100-102,131-143` | `dentalclinic@mp/test/kpi.test.ts:9-66` |
| A cited Source that expires within 90 days, such as a SIP, adds a warning | `dentalclinic@mp/src/brain/ask.ts:102-108` | `dentalclinic@mp/test/honest.test.ts:130` |
| Instruction-like sentences in Sources are flagged, logged and not followed | `dentalclinic@mp/src/brain/ask.ts:109-112` | `dentalclinic@mp/test/honest.test.ts:56` |
| Audit rows form a hash chain that the audit page verifies | `dentalclinic@mp/src/brain/jejak-audit.ts:138-146,188` | `dentalclinic@mp/test/audit.test.ts:18` |

- MCP tools: `who_am_i`, `ask`, `due_for_recall`, `propose_recall_message`, `propose_stok_opname` for every Role; `kpi` for the Pemilik only. No tool accepts, declines or sends (`dentalclinic@mp/src/mcp.ts:20-85`, `dentalclinic@mp/test/mcp.test.ts:52-67`).
- Identity Pairing: an 8-character code valid 10 minutes, confirmed by the signed-in staff member; a token with scope `read propose`, valid 90 days, stored hashed, revocable per device (`dentalclinic@mp/src/brain/pairing.ts:9-15`, `dentalclinic@mp/test/pairing.test.ts:16-138`).

**Architecture decisions**

- ADR 0001: accountability (permissions, masking, conflicts, citation checks, audit) lives in deterministic code behind MCP, not in prompts; no LLM runs on the Worker and the site has no chat (`dentalclinic@mp/docs/adr/0001-deterministic-engine-behind-mcp.md:3`).
- ADR 0002: a "brain" over mock data, not an ERP; ERP screens, SATUSEHAT and BPJS are roadmap (`dentalclinic@mp/docs/adr/0002-brain-not-erp.md:3`).
- ADR 0003: deploy on merge, migrations before upload, no preview deployments because there is one database (`dentalclinic@mp/docs/adr/0003-deploy-on-merge-with-github-actions.md:3`).
- Scope decisions: no WhatsApp Business API; the agent never sends; proposals are limited to Recall Message and Stok Opname Request; out of scope are record correction, treatment-plan scheduling, PII reveal, record deletion, SATUSEHAT and BPJS (`dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:47-49`).

**What "WorkBuddy" is in this codebase**

- The vendor describes WorkBuddy as "a full-scenario AI agent desktop workspace for professionals, built by Tencent" ([WorkBuddy Overview](https://www.workbuddy.ai/docs/workbuddy/Overview)). An Expert is an AI role configured for a domain, and an Expert Team is a lead plus members ([Expert Center](https://www.workbuddy.ai/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Expert-Center)). Connectors link it to external services over MCP, and a custom MCP connector can be added from the desktop app ([Connector](https://www.workbuddy.ai/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Connector)).
- CodeBuddy is Tencent's coding assistant; its CLI plugin format reads a manifest at `.codebuddy-plugin/plugin.json` and also accepts `.workbuddy-plugin/` and `.claude-plugin/` ([CodeBuddy Code plugins reference](https://www.workbuddy.ai/docs/cli/plugins-reference), [plugins](https://www.workbuddy.ai/docs/cli/plugins)). WorkBuddy Expert packages use the same folder name ([Expert spec](https://open.workbuddy.cn/en/docs/expert), [Expert Team spec](https://open.workbuddy.cn/en/docs/expert-team)).
- WorkBuddy plays two parts here. First, it is the only conversational surface for clinic staff (`dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:11`). Second, it was the coding agent that wrote the repository: the scope decisions say so (`dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:69`), `.workbuddy-ai/memory/` holds its project notes (`dentalclinic/.workbuddy-ai/memory/MEMORY.md:1-9`), and ten worktrees sit under a `WorkBuddy/Worktrees/dentalclinic/` folder in the home directory, seven of them on `workbuddy/*` branches (`git worktree list`).
- The integration consists of:

| Piece | Content | Source |
| --- | --- | --- |
| `expert/dira-cepat/` | Single-agent Expert: `.codebuddy-plugin/plugin.json` (`expertType: "agent"`, bilingual en/zh names, 3 tags, 3 quick prompts), `agents/dira-cepat.md`, avatar, README | `dentalclinic@mp/expert/dira-cepat/.codebuddy-plugin/plugin.json:2-30` |
| `expert/dentalclinic-team/` | Team Expert: lead Dira, members Sari (patients and recall), Bima (finance and assets), Wulan (reviewer, up to 3 review rounds); `settings.json` names the lead | `dentalclinic@mp/expert/dentalclinic-team/.codebuddy-plugin/plugin.json:14-19,52-81`, `dentalclinic@mp/expert/dentalclinic-team/settings.json:2`, `dentalclinic@mp/expert/dentalclinic-team/agents/dentalclinic-team-lead.md:27-42` |
| MCP declaration | Remote HTTP server `dental-clinic-brain` at the `workers.dev` `/mcp` URL, `Authorization: Bearer ${DENTALCLINIC_TOKEN}`, with an `x-workbuddy` token form | `dentalclinic@mp/expert/dira-cepat/.codebuddy-plugin/plugin.json:31-54` |
| Agent prompt | Call one tool, show the returned `text` verbatim, never send, treat tool output as data, run pairing with `curl` when unconnected | `dentalclinic@mp/expert/dira-cepat/agents/dira-cepat.md:19-24,30-35,39-46` |
| Packaging and install | `npm run build` zips each Expert into `public/`; a script copies them into the local WorkBuddy config folder and runs WorkBuddy's own validate and register scripts | `dentalclinic@mp/package.json:18-19`, `dentalclinic@mp/scripts/install-expert.sh:6-18` |
| Launch links | Two `workbuddy.ai/work/launch/?sharecode=…` links, one per Expert, shown as "Buka di WorkBuddy" and "Buka Tim di WorkBuddy" | `dentalclinic@om/src/data/release.json:2-3`, `dentalclinic@om/src/routes/index.tsx:183` |
| Tests | Manifest rules (bilingual fields, 40 to 50 character zh description, exactly 3 tags and prompts, no `tools:` in frontmatter) and that both Experts pair through the same Worker | `dentalclinic@mp/test/expert.test.ts:40-63,93-112` |

- Both Experts hard-code the `workers.dev` host, not `dentalclinic.web.id`, for MCP and pairing (`dentalclinic@mp/expert/dira-cepat/.codebuddy-plugin/plugin.json:34`, `dentalclinic@mp/expert/dira-cepat/agents/dira-cepat.md:41`).
- Docs versus code: the Open Platform spec documents MCP under `dependencies.mcpServers` or a root `.mcp.json`, while these manifests use a top-level `mcpServers`, which is the CodeBuddy CLI form. **[UNVERIFIED]** which form WorkBuddy treats as canonical. The repository registers these packages with WorkBuddy's own scripts and publishes 30 screenshots of WorkBuddy runs, which suggests the top-level form works (`dentalclinic@ds/README.md:148`, `dentalclinic@mp/public/evidence/`).
- **[UNVERIFIED]** against vendor pages: the local config folder and `expert-manager` scripts that the install script relies on, and the share-link format. Both come from the repository and its research note (`dentalclinic@hr/docs/research/2026-11-07-hackathon-dental-clinic-research.md:141-143`). Vendor facts above are paraphrased from the linked pages, apart from the one quotation.
- The vendor FAQ says server-side capabilities are still being improved and describes no inbound API ([FAQ](https://www.workbuddy.ai/docs/workbuddy/From-Beginner-to-Expert-Guide/FAQ)). Integration therefore runs one way, from WorkBuddy to the product over MCP, which is the premise of ADR 0001.
- The research note names a comparable product that hosts its Expert landing page on a dedicated `workbuddy.` host, with install button, launch link, device pairing and limits table (`dentalclinic@hr/docs/research/2026-11-07-hackathon-dental-clinic-research.md:161-165`).
- **[INFERENCE]** `/workbuddy` on the workspace most plausibly means the home of this integration for real tenants: the Expert landing and install page, the pairing screen (today `/perangkat`), connected-device management and the MCP endpoint. It is unlikely to mean an embedded chat, since no vendor page read here documents embedding or an inbound API.

### e. Gap list

**(i) dentalclinic capabilities and decisions worth carrying forward, and how the two products can coexist**

| Carry forward | Why it is reusable | Source |
| --- | --- | --- |
| Deterministic engine behind MCP; agent proposes, human approves on the site, 24-hour expiry | The safety model for any agent that touches patient data | `dentalclinic@mp/docs/adr/0001-deterministic-engine-behind-mcp.md:3`, `dentalclinic@mp/src/mcp.ts:50` |
| Role guards in SQL, one Refusal for restricted and missing data, masking | Maps onto the five workspace roles | `dentalclinic@mp/src/brain/access.ts:15-93` |
| Hash-chained Jejak Audit | An audit trail for medical-record access | `dentalclinic@mp/src/brain/jejak-audit.ts:138-146` |
| Due for Recall rule, WA Opt-in gate, no clinical detail in messages | Direct input to WhatsApp reminders and follow-ups | `dentalclinic@mp/src/brain/recall.ts:28-75,149-151` |
| Payback, Margin per Tindakan, Overhead %, SIP expiry warning | First finance and asset measures, with formulas | `dentalclinic@mp/src/brain/kpi.ts:100-102,131-143` |
| Identity Pairing | The pattern for linking any agent client to a staff identity | `dentalclinic@mp/src/brain/pairing.ts:9-15` |
| Test matrix T01 to T25 and the Workers-runtime test set-up | Ready acceptance cases for permissions and honesty | `dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:61` |

- Stays unique to dentalclinic.web.id: the public demo of one fictional clinic, Demo Login, the nightly reset, the hackathon evidence page, and the "brain, not ERP" position (`dentalclinic@mp/docs/adr/0002-brain-not-erp.md:3`, `dentalclinic@mp/src/brain/nightly.ts:9-17`).
- Overlaps with the workspace: the role model, patients, visits, appointments, stock, assets, procedures and fee rules (read-only and mock there; editable, real and multi-tenant in the workspace), WhatsApp recall, audit, and the WorkBuddy integration.
- Shareable rather than rebuilt: the glossary; the MCP tool contract (keeping tool names and the answer envelope stable would let the same Experts talk to either backend); the Expert packages, re-pointed at a new URL; the engine's rules and tests.
- Not shareable as-is: `src/brain/` reads D1 through `cloudflare:workers` and relies on SQLite FTS5 and Worker cron (`dentalclinic@mp/src/brain/staff.ts:1`, `dentalclinic@mp/db/migrations/0002_klinik_brain.sql:24-28`, `dentalclinic@mp/wrangler.jsonc:18`). **[INFERENCE]** Running it on a VPS needs a database adapter and a different search index.
- There is nothing to import: dentalclinic holds only fictional seed data that is reset nightly (`dentalclinic@mp/db/seed.sql:1`).
- **[INFERENCE]** The code suggests a natural split: dentalclinic.web.id as the open demo and sandbox for the agent experience, the workspace as the real multi-tenant product, with links between them. The code does not say what the owner intends.

**(ii) Main-product surfaces the workspace must integrate with**

- Identity link: a tenant must map to one or more `placeId` values, and today ownership is one approved Claim by one Google account per place (`findyourdent@om/packages/contracts/src/firestore.ts:396-400`).
- Directory page content: `ownerFacts`, the Price List, the Testimony reply and the violation response, each with its moderation and rate-limit rules (`findyourdent@om/apps/web/app/server/claims.server.ts:349-407`, `findyourdent@om/apps/web/app/server/price-lists.server.ts:1-11`).
- Sign-in: Firebase Auth, Google only, host-only cookie, console-managed authorized domains (`findyourdent@om/apps/web/server/browser-state.ts:30-35`, `findyourdent@om/infra/gcp/README.md:124`).
- Shared reference data: procedure codes, Region codes, insurers (`findyourdent@om/packages/contracts/package.json:7-12`).
- Design system: tokens, font, brand files, component conventions (section a).
- Ethics rules that bind every dentist-facing feature (`findyourdent@om/docs/specs/05-claim-and-price-list.md:56`).
- Deep links back to the public site: `/tempat/:placeId`, `/tempat/:placeId/klaim`, `/tempat/:placeId/daftar-harga` (`findyourdent@om/apps/web/app/routes.ts:41-43`).
- DNS at Hostinger for the new subdomain (`findyourdent@om/infra/gcp/README.md:104`).

**(iii) What the request assumes that neither repository has**

Rows restate facts cited in sections b and d; only new sources are cited inline.

| Assumed | Main product | dentalclinic |
| --- | --- | --- |
| Multi-tenancy | No tenant entity; one claimant per place | One fictional clinic, no clinic column |
| Staff accounts, invitations, real sign-in | Google sign-in for Patients and one claimant | Demo Login only (`dentalclinic@mp/src/brain/staff.ts:16`) |
| Roles superadmin, operations admin, viewer | Admin is the platform team | Four roles, none of these three |
| Scheduling calendar | None | `appointments` has a start time only, no duration, chair or status |
| Electronic medical records | None; health text is deliberately not stored | Free-text `visits.notes`; no odontogram, coding or attachments |
| Patient CRM, leads, booking from the directory | Contact links only | Recall proposals only |
| Partner-dentist fees and per-specialty scheduling | None | One `share_percent` per dentist; no specialty field |
| Inventory | None | A stock card of four columns; no movements, expiry, reorder or suppliers |
| Asset management | None | Name, price, purchase date; no maintenance or depreciation schedule |
| Finance | None | Monthly overhead and collections; no invoices, payments, payouts or tax |
| WhatsApp agent: inbound messages, qualification, booking, closing | `wa.me` links | Explicitly no WhatsApp Business API; the agent never sends (`dentalclinic@mp/docs/decisions/2026-11-07-scope-decisions.md:47`) |
| Third-party integrations dashboard | None | None |
| VPS deployment | Cloud Run and Firebase Hosting | Cloudflare Workers and D1 |
| A server-to-server API into the directory | None; cookie and App Check only | Not applicable |
| Back-office UI components | Nine patient-facing components | Button, Input, Table (`dentalclinic@mp/src/components/ui/`) |
| Tenant billing or plans | Declared out of scope (`findyourdent@om/docs/specs/05-claim-and-price-list.md:70`) | None |

### Implications for the workspace platform

- **[INFERENCE]** The directory integration is the first hard design decision. Because every owner route needs the public site's cookie and an App Check token, and no API or CORS exists (section b), the workspace cannot edit a directory page without one of: a new authenticated service API on the main product, direct Firestore access with its own service account (which would bypass the moderation path in `findyourdent@om/apps/web/app/server/claims.server.ts:365-372` unless that logic is shared), or deep-linking owners to the existing `/klaim` page.
- **[INFERENCE]** Single sign-on across `findyourdent.com` and `workspace.findyourdent.com` does not come for free: the session cookie is host-only and Firebase Hosting forwards only `__session` (`findyourdent@om/apps/web/server/browser-state.ts:5-8,30-35`). A separate sign-in on the workspace against the same Firebase project is the smaller change, and needs the new host added to the authorized-domains list.
- **[INFERENCE]** Tenant-to-listing verification should reuse the Claim rather than invent a second ownership check, since the Claim already carries the permit review and the phone call (`findyourdent@om/apps/web/app/lib/claims.ts:10-15`). The one-claimant-per-place rule then has to grow into "one tenant, many staff".
- **[INFERENCE]** A real clinic has data the main product must never receive. The main product stores no health text by rule (`findyourdent@om/docs/data-model.md:8`), so patient records belong only in the workspace's own database, with the directory getting public facts alone.
- **[INFERENCE]** The "closing" and CRM parts of the WhatsApp agent need an explicit ethics boundary, because the main product bans promotion, offers and discounts for dentist-facing features (`findyourdent@om/docs/specs/05-claim-and-price-list.md:13,56`) and dentalclinic bans clinical detail in outbound messages (`dentalclinic@mp/src/brain/recall.ts:149-151`).
- **[INFERENCE]** dentalclinic's propose-then-approve model with WA Opt-in is stricter than an autonomous WhatsApp agent. Moving to an agent that sends by itself reverses ADR 0001 and the scope decisions, so it should be a recorded decision, not a default.
- **[INFERENCE]** The workspace can start from dentalclinic's project skeleton (same framework, lint, UI conventions and test approach) and the main product's tokens, but must replace D1, Worker cron and FTS5 for a VPS.
- **[INFERENCE]** One procedure vocabulary is needed before tariffs, fees and public Price Lists can connect: the main product's 27 codes are the public standard; dentalclinic's six do not match it (section c).
- **[INFERENCE]** `/workbuddy` needs per-tenant tokens: today a token binds to a staff member of the only clinic (`dentalclinic@mp/db/migrations/0007_identity_pairing.sql:5-12`), and the Experts carry a fixed server URL.
- **[INFERENCE]** A glossary for the workspace should be written before specs, because the request's role names and "reminder" wording conflict with both existing glossaries.

### Open questions

1. How should the two live products relate for the owner's users: separate audiences, cross-links, shared Experts, or an optional import? What the code shows: no link in either direction; dentalclinic serves staff of one fictional clinic through Demo Login, so it has no real accounts and no real data to import; the main product serves Patients and one claimant per listing. The code does not show which audience the owner wants on which product.
2. Which role names are canonical: Pemilik or owner or superadmin; Front Office or operations admin; and what a viewer may see?
3. Is a workspace tenant a Provider (one `placeId`), or can it hold several places, and can a hospital dental unit be a tenant when the Hospital is the Google place?
4. Does the workspace sign in with the main product's Firebase project (Google only) or with its own identity system that also covers staff without Google accounts?
5. May the workspace write directly to the main product's Firestore, or must the main product expose an API first?
6. Should the workspace adopt the main product's teal palette and Figtree, or dentalclinic's green and Instrument Sans? Should dark mode ship?
7. What does `/workbuddy` have to deliver: Expert landing and pairing only, or a new multi-tenant MCP server, and do the existing Experts keep pointing at dentalclinic?
8. Does the WhatsApp agent send on its own, or keep the human-approval rule?
9. Which commit is live on each site? findyourdent.com deploys are manual and `origin/main` is 67 commits past tag `v1.0.0` (`git describe --tags origin/main`). dentalclinic.web.id deploys on every merge to `main`, so it should be serving `56cd151`, but that was not checked against the live site. **[UNVERIFIED]**
10. Is `workspace.findyourdent.com` already in DNS or in the Firebase authorized-domains list? Both are console state, not in either repository. **[UNVERIFIED]**
11. Can Expert packages be published to WorkBuddy's marketplace from Indonesia? The research note says publishing is restricted (`dentalclinic@hr/docs/research/2026-11-07-hackathon-dental-clinic-research.md:144`); no vendor page read here confirms it. **[UNVERIFIED]**
12. Should the uncommitted main-checkout edits in `findyourdent/` and the violation-badge work in the `origin/main` worktree be treated as abandoned or pending? They were excluded here.


---

## 2. Clinical and practice-management benchmarks

Method note: every link below was opened in this session (2026-11-25). Pages were read either as raw page text (WebCeph, NICE, ITI, EFP, AAPD, AAP, Dolphin brochure, coDiagnostiX IFU, Florida Probe, Dentalink, Dentrix Ascend help, Open Dental schema) or through a fetch tool that summarises the page and returns quotes (the other Open Dental manual pages, Dentally, CareStack, Curve, Ortho2, AAO, AAE, ADA, BSSPD, Smilecloud, FDA, the other Indonesian vendors); quotes from the second group were not re-checked against raw HTML. Items tagged **[UNVERIFIED]** could not be confirmed at a primary source (blocked, JS-only, or only seen in a search snippet); **[INFERENCE]** marks my own reasoning. Blocked sources: AAOMS patient site (Cloudflare challenge, HTTP 403), ISO catalogue page for ISO 3950 (403), Planet DDS Cloud 9 product page (rendered headings only), Dentrix Ascend support site (TLS failure; the Henry Schein help host was read instead).

### a. WebCeph and what an orthodontic module needs beyond it

**What WebCeph is (vendor: AssembleCircle Corp., Seongnam, Korea)**

- Positioning: "Web-based Orthodontic Analysis Software" with apps on Google Play and the App Store linked from the home page ([home](https://webceph.com/en/)).
- Core functions listed in the vendor guide ([guide](https://webceph.com/en/guide/)):
  - Automated cephalometric tracing and "various cephalometric analysis".
  - Visual treatment objectives: "simulate orthodontic treatment and orthognathic surgery, and save the result as an image or video file" (VTO / STO).
  - Automated ceph superimposition to "evaluate treatment progress".
  - Case Review and Caseroom: cases organised into templates and reviewed later.
  - Image viewer with Single, Dual and Template modes; gallery export and print as PDF.
  - Analysis Wizard and Measurement Wizard for custom analyses and measurements.
  - Face simulation from clinical photos.
  - WebNotes (notes) and a Scheduler with drag-and-click appointments.
  - Toosapp, "a mobile application that connects dentists with their patients", used to send educational material.
  - Community features: forum, webinars, study groups.
- Plans and prices (USD, "VAT 10% excluded"; the Indonesian page shows the same USD prices, "Belum termasuk PPN 10%") ([pricing EN](https://webceph.com/en/pricing/), [pricing ID](https://webceph.com/id/pricing/)):

| Plan | Price | What it adds (vendor wording, condensed) |
|---|---|---|
| WebCeph (free) | FREE / month | Automated ceph tracing, various ceph analyses, VTO/STO simulation, automated superimposition, image gallery, case review and Caseroom |
| Plus | $9.99 / month | Ad-free; automated time-lapse video; extra tracing landmarks; landmark table with Excel export; maxilla / mandible / multiple superimposition; clinic logo instead of watermark on PDF report; STO/VTO landmark movement table (Excel export); profile superimposition video; advanced Caseroom; multi-stage analysis comparison chart; image crop; Toosapp PLUS |
| Premium | $14.99 / month | Plus features + PA cephalometric viewer, frontal face viewer, lateral face viewer |
| Elite | $19.99 / month | Premium features + 3D STL model viewer and analysis (up to 5 GB STL storage), occlusogram viewer, patient analysis report, automated photo upload, desktop application, full-screen case presentation |

- Regulatory and AI statements: FDA 510(k) K220903; Korean device certification; ISO 13485:2016; the "About" page states the AI uses SVM, gradient boosting and CNN models to detect landmarks, reports mean deviation of 1.5 mm or less on lateral cephs and 2.5 mm or less on PA cephs and clinical photos against expert ground truth, 10,550 training items including augmentation, and that results are reference information to assist the clinician's judgment (my translation of the Korean text) ([about](https://webceph.com/en/about/)). FDA lists K220903 as "WebCeph", applicant Assemblecircle Corp., decision date 08/17/2022, product code LLZ, substantially equivalent ([FDA 510(k) database](https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfpmn/pmn.cfm?ID=K220903)).
- Data hosting: the About page says the product is web-based and AI analysis runs in a cloud environment "based on Amazon Web Services" ([about](https://webceph.com/en/about/)); the pricing page says "All patient data is securely stored in the cloud" ([pricing](https://webceph.com/en/pricing/)); the global privacy policy names Amazon Web Services Korea LLC as the cloud processor and states personal information is transferred to and processed in Korea ([privacy policy](https://webceph.com/en/policy/global/privacy)). The API sample response shows `"region": "ap-northeast-2"` ([API](https://webceph.com/en/api/)).
- API and integration (decisive for build-vs-integrate) ([API page](https://webceph.com/en/api/)):
  - "API access requires prior approval from WEBCEPH. A subscription plan alone does not grant API access"; integration is offered "to members of WEBCEPH Premium level or higher"; the partner API is "currently provided free of charge as a beta service and is planned to transition to a paid service".
  - Endpoints: API status, get user info, get patient list (latest 10, search by name or patient ID), list records, create patient, add record, upload image. Limit: 30 requests per minute per user.
  - Hard limits: "We do not provide data via API such as landmark points, analysis results, or other clinical and diagnostic information"; "image export ... is not provided through the API"; the API "cannot be used for bulk retrieval" or for downloading records and photos already uploaded.
  - Intended pattern: a shortcut in the practice-management software that opens the patient in WebCeph, or direct upload from imaging software. Responses return a per-patient `link` URL.
  - API keys may not be shared with third parties or resold; support "may be limited to official documentation".
- Export paths that do exist are manual: landmark and movement tables to Excel, PDF analysis report, images and videos ([pricing](https://webceph.com/en/pricing/), [guide](https://webceph.com/en/guide/)).
- Not found on vendor pages: patient-count or storage caps for the free plan, team/seat pricing, SLA. **[UNVERIFIED]**

**What an orthodontic module needs beyond ceph analysis (from orthodontic practice systems)**

- Treatment card per visit. Dolphin Management's brochure describes a treatment card with customisable layouts, quick phrases, "personalized band/elastic sizes", images, graphing of any column "such as Oral Hygiene or Cooperation", and scheduling "multiple next-appointments directly from the Treatment Card" ([Dolphin Management brochure, rev 07/19](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Open Dental's Ortho Chart is a grid of visit rows with configurable tabs/columns, read-only date and provider, per-row electronic signature that is invalidated if the row changes, single-editor locking and an audit trail ([Ortho Chart](https://www.opendental.com/manual/orthochart.html)).
- Appliance and hardware tracking. Dolphin's tooth chart records "bracket type with torque information, arch wire sizes, elastics, powerchains, TADs and annotations" ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Open Dental stores `orthohardware` rows typed Bracket / Wire / Elastic per exam date and tooth range, with a spec pick list (example "NITI 16x25") and `orthorx` prescriptions for fast entry ([Open Dental schema 26.3](https://www.opendental.com/OpenDentalDocumentation26-3.xml)). Ortho2 tracks appliance type, service location and dates, optionally by barcode ([Appliances & Milestones](https://help.ortho2.com/patient/panel-milestones)).
- Staged treatment plan. Dolphin: treatment plans "include sequencing, number of visits, procedures, months between visits, internal visit notes", picked from predefined global plans, with progress shown on the treatment card ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Ortho2 Edge Cloud: workflows "sequence standardized tasks and milestones" and the Treatment Hub gives access to "treatment plan and historical tooth chart data" ([Edge Cloud](https://www.ortho2.com/our-solutions/edge-cloud/)); milestone dates are "updated automatically as you schedule patient appointments or work with the treatment chart" ([Milestones](https://help.ortho2.com/patient/panel-milestones)).
- Case object with dates and fee schedule. Open Dental's Ortho Case holds banding (or transfer) date, expected debond date, total fee, primary/secondary insurance and patient portions; a fee breakdown into banding amount, debond amount and all-visits amount whose "percentages must equal 100"; visits planned and "Visits Completed: Automatically calculates as visit appointments are set complete"; one active case per patient; the case deactivates when the debond procedure is completed ([Ortho Case](https://www.opendental.com/manual/orthocase.html)). Its Auto Ortho data adds Date Start, Tx Months Total, Months in Treatment and Months Rem ([Auto Ortho](https://www.opendental.com/manual/orthoauto.html)).
- Contracts and payment plans. Ortho2: "You create a contract for each treatment phase that a patient goes through", obligations "can be allocated among multiple responsible parties and insurance plans" ([Financials](https://help.ortho2.com/finance)); the contract structure editor sets "the initial fees, periodic fees, and retention fees for each member of the contract" ([Contract Structure Editor](https://help.ortho2.com/finance/contract-edit-structure)); Edge Cloud lets patients "electronically adjust the terms of a contract before signing it" ([Edge Cloud](https://www.ortho2.com/our-solutions/edge-cloud/)). Ortho2 also documents open-ended contracts, auto receipt agreements, payments on file and late fees as separate help articles ([Financials](https://help.ortho2.com/finance)). Dolphin: "treatment/contract proposals, complex contracts, precise adjustments, family statements", plus X-Charge integration "for automatic credit card processing" ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Open Dental links a payment plan and a production schedule to the ortho case (`orthoplanlink`, `orthoschedule`) and supports weekly to quarterly charge frequency on payment plans ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Visit intervals (professional body). AAO: modern wires allow "6 to 10-week intervals between orthodontic appointments"; aligners are "each worn for one to two weeks before moving to the next one" ([AAO, modified 2025-10-21](https://aaoinfo.org/whats-trending/what-if-im-too-busy-for-orthodontic-visits/)). Retention: "Retainers are typically worn immediately after active treatment ends", then reduced to night wear; "many patients wear retainers nightly for life"; removable and fixed types ([AAO, modified 2026-01-05](https://aaoinfo.org/whats-trending/will-i-need-to-wear-retainers/)).
- Ortho-specific scheduling and flow. Dolphin: 2 to 60 minute increments, "unlimited chairs/columns and unlimited office locations", day/location templates, doctor-time tracking, a "Sooner if Possible" list and patient-flow tracking ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Ortho2: "Doctor time visually represented for each procedure" ([Edge Cloud](https://www.ortho2.com/our-solutions/edge-cloud/)).
- Notation: Open Dental stores ortho hardware tooth ranges "in Universal (1-32) notation" and they "are displayed to the user as Palmer notation"; brackets use single teeth, wires a range, elastics usually two teeth ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Cloud 9 (Planet DDS) treatment card, aligner tab and financial arrangements: **[UNVERIFIED]**, the product page returned only headings ([page](https://www.planetdds.com/cloud9)). tops Software was not read. **[UNVERIFIED]**

**Orthodontic stage template assembled from the above** (stage names are **[INFERENCE]**; each cell cites what a benchmark records)

| Stage | What benchmark systems record | Interval or rule |
|---|---|---|
| Records and diagnosis | Ceph, photos, analyses, VTO/STO ([WebCeph](https://webceph.com/en/guide/)); exam questionnaires and treatment or contract proposals ([Dolphin](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)) | Before contract |
| Contract | Total fee, insurance and patient portions, initial / periodic / retention fees, responsible parties ([Open Dental](https://www.opendental.com/manual/orthocase.html), [Ortho2](https://help.ortho2.com/finance/contract-edit-structure)) | One contract per treatment phase ([Ortho2](https://help.ortho2.com/finance)) |
| Banding, bonding or aligner delivery | Banding date, banding fee share, hardware placed ([Open Dental](https://www.opendental.com/manual/orthocase.html), [schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)) | Starts the months-in-treatment counter ([Auto Ortho](https://www.opendental.com/manual/orthoauto.html)) |
| Adjustment visits | Visit row with band and elastic sizes, hygiene and cooperation columns ([Dolphin](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)); signed grid row per visit ([Open Dental Ortho Chart](https://www.opendental.com/manual/orthochart.html)); visits completed vs planned ([Open Dental Ortho Case](https://www.opendental.com/manual/orthocase.html)) | 6 to 10 weeks; aligners changed every 1 to 2 weeks ([AAO](https://aaoinfo.org/whats-trending/what-if-im-too-busy-for-orthodontic-visits/)) |
| Progress records | Superimposition, multi-stage comparison chart, time-lapse video ([WebCeph pricing](https://webceph.com/en/pricing/)) | Timing set by clinician **[UNVERIFIED]** |
| Debond | Debond date vs expected debond, debond fee share; case closes ([Open Dental](https://www.opendental.com/manual/orthocase.html)) | Expected date set at case start |
| Retention | Retainer type removable or fixed; retention fee ([AAO](https://aaoinfo.org/whats-trending/will-i-need-to-wear-retainers/), [Ortho2](https://help.ortho2.com/finance/contract-edit-structure)) | Starts immediately after active treatment; long-term night wear ([AAO](https://aaoinfo.org/whats-trending/will-i-need-to-wear-retainers/)) |
- Note on source age: the Dolphin brochure is dated 07/19; treat it as a functional reference, not a current feature list.

### b. Specialty-by-specialty benchmark

| Specialty | Data captured | Visit-sequence template with intervals | Benchmark tool(s) | Build vs integrate |
|---|---|---|---|---|
| Orthodontics | Ceph tracing, analyses, superimposition, VTO/STO, photo series ([WebCeph guide](https://webceph.com/en/guide/)); visit grid rows with provider and signature ([Open Dental](https://www.opendental.com/manual/orthochart.html)); brackets, wires, elastics by tooth range ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)); hygiene and cooperation scores ([Dolphin](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)) | Records and consult → banding / aligner delivery → adjustments every 6 to 10 weeks; aligner change every 1 to 2 weeks ([AAO](https://aaoinfo.org/whats-trending/what-if-im-too-busy-for-orthodontic-visits/)) → debond → retainers immediately, night wear long term ([AAO](https://aaoinfo.org/whats-trending/will-i-need-to-wear-retainers/)) | WebCeph; Dolphin Imaging + Management; Ortho2 Edge Cloud. ClinCheck, OnyxCeph, DentalMonitoring **[UNVERIFIED]** | **[INFERENCE]** Build case, staged visits, hardware/aligner log, contract schedule, photo archive. Integrate ceph by deep link and manual PDF/Excel import; WebCeph API cannot return results ([API](https://webceph.com/en/api/)) |
| Periodontics | Six sites per tooth: probing depth, gingival margin, auto CAL, MGJ, furcation 0 to 3, mobility, bleeding, suppuration, plaque, calculus; skipped and implant teeth; up to six exams compared ([Open Dental perio](https://www.opendental.com/manual/perio.html)). Stage I to IV by interdental CAL, radiographic bone loss, tooth loss; grade A to C with smoking and HbA1c modifiers ([AAP chart](https://sites.perio.org/wp-content/uploads/2019/08/Staging-and-Grading-Periodontitis.pdf)) | Step 1 behaviour change, risk-factor control, professional plaque removal → Step 2 subgingival instrumentation (quadrant-wise or full mouth within 24 hours) → re-evaluation, endpoints "No periodontal pockets ≥ 5 mm with bleeding on probing" and "No deep pockets [≥ 6 mm]" → Step 3 re-instrumentation or surgery → Step 4 supportive care "at intervals of 3 to a maximum of 12 months" tailored to risk ([EFP S3 guideline infographic](https://www.efp.org/fileadmin/uploads/efp/Photos/Continuing_Education/Teatment-stage-Perio_Steps-ALL_071022-3.pdf)). Healing period before re-evaluation in weeks: **[UNVERIFIED]** | Florida Probe: constant-force probe, 0.2 mm precision, records recession, pocket depth, bleeding, suppuration, furcation, plaque, mobility, MGJ, risk assessment ([Florida Probe](https://floridaprobe.com/fpsystem01.htm)); Curve perio charting with 6 pathways and Bola AI voice entry ([Curve](https://www.curvedental.com/perio-charting)) | **[INFERENCE]** Build perio chart, exam comparison, stage/grade helper, supportive-care recall type. Voice entry later |
| Endodontics | Diagnosis and prognosis per procedure, multi-visit grouping ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)). Canal count, working length, obturation, pre/post-op radiograph fields **[INFERENCE]** | "Endodontic treatment can often be performed in one or two visits", about 90 minutes each; temporary filling; "you must return to your general dentist" for crown or restoration; healing check "in 6-12 months" ([AAE](https://www.aae.org/patients/root-canal-treatment/what-is-a-root-canal/root-canal-explained/)). ESE follow-up schedule **[UNVERIFIED]** | No WebCeph-style cloud tool confirmed. AI periapical detection products **[UNVERIFIED]** | **[INFERENCE]** Build endo visit template, pending-restoration task, 6 to 12 month review recall |
| Prosthodontics, crown and bridge, lab | Lab case: lab, provider, due date, created / sent / received / quality-checked timestamps, instructions, fee, invoice number; per-lab service turnaround days ([Lab Case](https://www.opendental.com/manual/labcaseedit.html), [schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)); prosthesis initial vs replacement and original date ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)) | Prep and temporary → lab turnaround (per-lab setting; due date skips weekends and holidays) → seat; multi-visit procedures show "Complete (In Process)" until all are done ([Open Dental](https://www.opendental.com/manual/procsmultipleappts.html)). Complete dentures "can often require up to six visits by the patient, and a similar number of laboratory stages" ([BSSPD](https://www.bsspd.org/For+Patients/Removable+Prosthodontics.aspx)). Typical crown turnaround days **[UNVERIFIED]** | PMS lab-case tracking as above. exocad, 3Shape **[UNVERIFIED]** | **[INFERENCE]** Build lab-case entity tied to the delivery appointment with stage checklist. Integrate lab portals later |
| Implantology | Implant teeth flagged in perio chart ([Open Dental](https://www.opendental.com/manual/perio.html)). Site, system, size, lot, placement type, loading protocol, graft, CBCT and STL files **[INFERENCE]** | Placement: immediate (Type 1) at extraction; early with soft-tissue healing "(4–8 weeks)" or partial bone healing "(12–16 weeks)"; late "more than 6 months after tooth extraction". Loading: immediate "within 1 week", early "between 1 week and 2 months", conventional "more than 2 months" ([ITI consensus](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/implant-placement-and-loading-protocols/1802), [ITI 2009 definitions](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/loading-protocols-2009-/1208), [type definitions](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/implants-in-postextraction-sites/1209)) | coDiagnostiX: imports "3D CBCT or CT scan data (DICOM standard) and model scan data" and exports data for surgical guides ([IFU v14.4](https://www.straumann.com/content/dam/media-center/digital/en-us/documents/knowledge-center/codiagnostix/coDiagnostiX-10-IFU-EN-v14.4.pdf)). Blue Sky Plan, DTX Studio **[UNVERIFIED]** | **[INFERENCE]** Build implant case timeline and implant record. Integrate planning by file attachment, not by embedding |
| Oral surgery, extractions | Third-molar radiographic assessment in late adolescence ([AAPD](https://www.aapd.org/globalassets/media/policies_guidelines/bp_periodicity.pdf)). Indication, technique, anaesthesia, sutures, complications, post-op instructions **[INFERENCE]** | Consult and radiograph → surgery → review. Socket landmarks for onward planning: 4 to 8 weeks soft tissue, 12 to 16 weeks partial bone, more than 6 months complete bone healing ([ITI](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/implant-placement-and-loading-protocols/1802)). Suture-removal and review day counts **[UNVERIFIED]** (AAOMS page blocked) | None identified | **[INFERENCE]** Build entirely: consent, post-op instruction message, review task |
| Pediatric | Caries-risk assessment, radiographs, fluoride, sealants, growth and occlusion, anticipatory guidance, transition to adult care ([AAPD, revised 2022](https://www.aapd.org/globalassets/media/policies_guidelines/bp_periodicity.pdf)); primary teeth 51 to 85 in FDI ([Open Dental](https://www.opendental.com/manual/graphicaltoothchart.html)); permanent / deciduous / mixed dentition toggles ([Dentally](https://help.dentally.com/en/articles/3565932-how-to-chart-in-dentally)) | First exam "at the time of the eruption of the first tooth and no later than 12 months of age"; higher-risk children benefit from recall "every 3 months" rather than every 6 ([AAPD](https://www.aapd.org/globalassets/media/policies_guidelines/bp_periodicity.pdf)); longest interval under 18 is 12 months ([NICE CG19](https://www.nice.org.uk/guidance/cg19/chapter/Recommendations)) | None identified | **[INFERENCE]** Build guardian link, mixed-dentition odontogram, risk-based recall |
| Conservative, restorative | Tooth, surfaces (B/F, V, M, O/I, D, L), status TP / C / EC / EO / R / Cn, diagnosis, prognosis ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)); ICDAS 0 to 6 lesion scale and ADA caries-risk forms for ages 0 to 6 and over 6 ([ADA, updated 2023-07-18](https://www.ada.org/resources/ada-library/oral-health-topics/caries-risk-assessment-and-management)) | Usually single visit. Recall is risk-based: 3, 6, 9 or 12 months under 18; 3 to 24 months in 3-month steps for adults; interval and patient agreement must be recorded ([NICE CG19](https://www.nice.org.uk/guidance/cg19/chapter/Recommendations)) | AI radiograph reading: Overjet Dental Assist, FDA K210187, decision 05/19/2021 ([FDA](https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfpmn/pmn.cfm?ID=K210187)); Open Dental polls Pearl's API for AI annotations on images ([schema, `pearlrequest`](https://www.opendental.com/OpenDentalDocumentation26-3.xml)) | **[INFERENCE]** Build odontogram and caries-risk form. Integrate AI reading later behind an attachment hook |
| Aesthetic (whitening, veneers) | Pre-treatment exam and history; existing restorations, since "Only natural teeth can be whitened" ([ADA, updated 2026-11-11](https://www.ada.org/resources/ada-library/oral-health-topics/whitening)). Shade before/after and photo set **[INFERENCE]** | Exam → in-office peroxide "may be applied for up to 30 minutes", or at-home trays with 10% to 38% carbamide peroxide, reviewed regimens "2 to 10 hours for periods of 6 to 28 days" ([ADA](https://www.ada.org/resources/ada-library/oral-health-topics/whitening)). Veneer sequence (mock-up → prep → lab → bond) **[UNVERIFIED]** | Smilecloud: Lite, Starter, Business, Dental Group plans; biometric smile design, before/after video simulations (10 or 30 per month), DICOM to 3D conversion, team workspace ([Smilecloud pricing](https://www.smilecloud.com/pricing/)); euro prices **[UNVERIFIED]**. DSD app **[UNVERIFIED]** | **[INFERENCE]** Build photo protocol, shade fields, package/quota tracking. Integrate smile design by link or file |

Supporting detail for the table:

- AAP staging thresholds: Stage I CAL 1 to 2 mm, Stage II 3 to 4 mm, Stage III and IV 5 mm or more; grade primary criterion is bone loss or CAL over 5 years (none, under 2 mm, 2 mm or more); "Clinicians should initially assume grade B disease" ([AAP chart](https://sites.perio.org/wp-content/uploads/2019/08/Staging-and-Grading-Periodontitis.pdf)). These are computable from perio-chart and history fields. **[INFERENCE]**
- ITI defines combined protocols as placement type plus loading letter (for example Type 1A, Type 4C) and states Type 1A "should only be performed by clinicians with a high level of clinical skill and experience" ([ITI consensus](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/implant-placement-and-loading-protocols/1802)); a 2026-02-17 ITI article uses the same Type 1 to 4 and A / B / C scheme ([ITI blog](https://blog.iti.org/clinical-insights/state-of-the-art-on-implant-placement-and-loading-protocols-in-partially-edentulous-patients/)). Older ITI definitions (2003) used 48 hours and 3 to 6 months, so the protocol label should store the actual dates ([ITI 2009 page](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/loading-protocols-2009-/1208)).
- NICE: recall interval "should be determined specifically for each patient", chosen at the end of a review or "on completion of a specific treatment journey", and reviewed at each oral health review ([NICE CG19](https://www.nice.org.uk/guidance/cg19/chapter/Recommendations)).
- Recall rules per specialty differ only in trigger and interval, so one recall engine can serve all of them **[INFERENCE]**:

| Recall type | Trigger | Interval | Source |
|---|---|---|---|
| Routine adult review | End of oral health review or treatment journey | 3 to 24 months in 3-month steps, risk-based, recorded with patient agreement | [NICE CG19](https://www.nice.org.uk/guidance/cg19/chapter/Recommendations) |
| Routine child review | Same | 3, 6, 9 or 12 months; 6 months common, 3 months for high risk | [NICE CG19](https://www.nice.org.uk/guidance/cg19/chapter/Recommendations), [AAPD](https://www.aapd.org/globalassets/media/policies_guidelines/bp_periodicity.pdf) |
| Supportive periodontal care | Endpoints of active therapy reached | 3 to a maximum of 12 months, by risk profile | [EFP](https://www.efp.org/fileadmin/uploads/efp/Photos/Continuing_Education/Teatment-stage-Perio_Steps-ALL_071022-3.pdf) |
| Endodontic healing check | Root canal treatment finished | 6 to 12 months | [AAE](https://www.aae.org/patients/root-canal-treatment/what-is-a-root-canal/root-canal-explained/) |
| Orthodontic adjustment | Previous adjustment or aligner delivery | 6 to 10 weeks | [AAO](https://aaoinfo.org/whats-trending/what-if-im-too-busy-for-orthodontic-visits/) |
| Retention check | Debond | Not specified by AAO; set by the orthodontist | [AAO](https://aaoinfo.org/whats-trending/will-i-need-to-wear-retainers/) |
| Implant stage gate | Extraction or implant placement date | 4 to 8 weeks, 12 to 16 weeks, over 6 months (placement); 1 week, 2 months (loading thresholds) | [ITI](https://academy.iti.org/academy/consensus-database/consensus-statement/-/consensus/implant-placement-and-loading-protocols/1802) |

### c. Core practice-management features (Open Dental as reference, plus cloud systems)

**Charting and notation**

- Open Dental supports four numbering systems chosen by one preference: Universal (1 to 32, A to T), FDI (11 to 48 permanent, 51 to 85 primary), Haderup, Palmer ([Graphical Tooth Chart](https://www.opendental.com/manual/graphicaltoothchart.html)). Procedure rows store the tooth as "1-32 or A-T" whatever the display preference, and saved treatment-plan copies store the already-converted display string ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)). The ISO 3950 text itself was not opened. **[UNVERIFIED]**
- Chart colours follow procedure status (treatment planned, complete, existing current provider, existing other provider, referred, condition); a time slider replays history; supernumerary teeth are not drawn ([Graphical Tooth Chart](https://www.opendental.com/manual/graphicaltoothchart.html)).
- `toothinitial` records missing, hidden, primary, shifts in mm, rotation and tipping in degrees, and drawings ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- A procedure "can NEVER be deleted", only set to status deleted; statuses are TP, C, EC, EO, R, D, Cn; each row carries tooth, surfaces, tooth range, priority, provider, clinic, diagnosis, prognosis, fee, discount, `DateTP` and `IsLocked` ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Dentally separates a base chart from treatment plans: "Each open treatment plan has its own tab in the patient's chart", completed plans move to History, and a BPE tab holds basic perio exam history ([Dentally, 2026-04-01](https://help.dentally.com/en/articles/3565932-how-to-chart-in-dentally)).

**Treatment plans, phases, alternatives, consent**

- Open Dental: "There can be one active treatment plan per patient" and "multiple inactive treatment plans"; saved plans are permanent signed copies; phasing is done with priority labels (examples include Plan A, Plan B, AltPlan); "Sign TP: Electronically sign a saved treatment plan" ([Treatment Plan](https://www.opendental.com/manual/treatmentplan.html)). Saved plans copy procedures into `proctp`; `treatplan` stores patient and practice signatures with timestamps and a presenter user ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- CareStack: plans hold alternates and phases give sequence, "Break treatment into phases, letting your patient know what comes first"; patients can "review and sign the plan electronically via the Patient Portal"; a treatment coordinator can be assigned ([CareStack treatment planning](https://carestack.com/dental-software/features/treatment-planning)).
- Dentally: "Create Estimate" builds an estimate or consent form from the charted plan using templates; it becomes available "for the patient to sign electronically on their Portal" and is saved to the patient's correspondence ([Dentally, 2026-09-07](https://help.dentally.com/en/articles/3566525-how-to-create-an-estimate-consent-form-for-a-patient)).
- Next-visit queueing: Open Dental creates "Planned Appointments" from the plan ("+Plan Appt"), and a "P" marks procedures attached to one ([Treatment Plan](https://www.opendental.com/manual/treatmentplan.html)); lab cases attach to the planned appointment until it is scheduled ([Lab Case](https://www.opendental.com/manual/labcaseedit.html)).
- Open Dental consent forms are sheet templates generated from the Chart or Treatment Plan module, can pull selected procedures, and "Electronic Signatures are supported"; paper copies are scanned into Imaging ([Consent Forms](https://www.opendental.com/manual/consentforms.html)).
- Clinical notes: the Open Dental chart offers group notes (one note across several same-day procedures) and exam sheets ([Chart Module](https://www.opendental.com/manual/chart.html)); auto notes can be inserted into ortho chart rows and lab-case instructions ([Ortho Chart](https://www.opendental.com/manual/orthochart.html), [Lab Case](https://www.opendental.com/manual/labcaseedit.html)). Dentally practice and personal note templates: **[UNVERIFIED]** (seen in search results only).

**Imaging**

- Open Dental's Imaging module stores radiographs, photos, documents and mounts in categories; it previews "JPEG, GIF, DICOM, etc." and PDF; "Exported DICOM images reflect the original file"; folders can be shared to the patient portal; designated categories show as thumbnails in the chart ([Imaging Module](https://www.opendental.com/manual/images.html)). `document` rows carry image type, optional tooth numbers, provider and a bound signature ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).

**Recall**

- Open Dental: every patient gets a prophy recall, switchable to perio, with custom types; "When a recall types' trigger procedures are set complete, the patient's next recall appointment due date is calculated"; interval can be overridden per patient; recall can be disabled always, until a date, or until the family balance is below an amount ([Recall](https://www.opendental.com/manual/recall.html)). Schema: one recall per patient per type; `recalltrigger` links procedure codes to recall types; recall types carry default interval, time pattern and procedures for the appointment ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Dentally: per-patient "Dentist recall interval" and "Hygienist recall interval"; due date "updates automatically" unless edited ([Dentally, 2026-08-24](https://help.dentally.com/en/articles/3563875-how-to-set-a-patient-s-recall-preferences)); a `[patient_recall_booking_link]` tag sends "a secure unique link which will take the patient straight to booking without having to log in", and dentist plus hygienist recalls within 28 days are offered together ([Dentally, 2026-08-24](https://help.dentally.com/en/articles/6791631-how-to-use-online-booking-for-recall-appointments)).

**Calendar: operatories, provider time, blocks, waitlist, online booking**

- Each operatory "is a single operatory or column in the appts module", with default dentist, hygienist, clinic, hygiene flag and online-booking flag ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)). Provider on a new appointment resolves in order: provider scheduled in the operatory, provider assigned to the operatory, patient's primary provider ([Operatories](https://www.opendental.com/manual/operatories.html)).
- Appointment time pattern separates doctor time (X) from assistant time (/) in 5-minute units; statuses are Scheduled, Complete, UnschedList, Broken, Planned and patient notes; a planned appointment "never gets scheduled, but instead gets copied"; a separate hygienist provider is optional ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Schedules: one table of time blocks typed Practice, Provider, Blockout, Employee or WebSchedASAP, optionally linked to operatories ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)); weeks can be copied, pasted and repeated, holidays are protected, and views filter by clinic ([Schedule](https://www.opendental.com/manual/schedule.html)).
- Block scheduling: blockout types carry flags "No appointment scheduling (NS)" and disable cut/copy/paste; appointment types can be restricted to allowed blockout types ([Blockouts](https://www.opendental.com/manual/blockouts.html)). Appointment types hold default procedure codes, pattern and colour ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Waitlist: "The ASAP List shows patients who would like to be contacted when an earlier appointment becomes available", covering appointments and recalls, filterable by provider, clinic and site, with text offers and online rescheduling ([ASAP List](https://www.opendental.com/manual/asaplist.html)).
- Online booking: CareStack offers direct booking that lets patients "find an appropriate slot in your calendar and fully book the appointment" and indirect booking where the patient requests a slot ([CareStack patient portal](https://carestack.com/dental-software/features/patient-portal)). Open Dental restricts online booking to operatories flagged for it ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).

**Patient portal**

- CareStack portal: view and confirm appointments, book, message, complete forms and medical history, "view and sign recommended treatment plans", view balance and pay, add family members; "Only the responsible party will see the entire account information" ([CareStack](https://carestack.com/dental-software/features/patient-portal)).
- Open Dental eServices page redirected and its list was not captured. **[UNVERIFIED]**

**Patient CRM**

- Households: a guarantor heads each family; super families group several families; new members inherit contact data, billing type and providers ([Family Module](https://www.opendental.com/manual/family.html)); `patient` stores guarantor, primary and secondary provider, clinic, and status values Patient, NonPatient, Inactive, Archived, Deceased, Prospective ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Communication log: "a general purpose log of all communications with a patient" with type, mode (phone, email, mail, in person, text, fax), sent or received, note, signature; automatic entries for texts and confirmation or ASAP messages ([Commlog](https://www.opendental.com/manual/commlog.html)).
- Referral tracking: referred from, referred to, custom; referrals attach to procedures; reports include "Referral Analysis Report: Track production for referrals in a date range" and referred-procedure tracking ([Referrals](https://www.opendental.com/manual/referrals.html)). Dolphin reports referral sources "based on patients, practitioners and billing parties" ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)).
- Segmentation: Dolphin's report tool selects "from over 500 data fields" and any generated list "can be sent a customized text" ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)). Dentally groups reports into financial, patient, appointment, treatment and practice categories ([Dentally, 2026-08-24](https://help.dentally.com/en/articles/9317584-how-to-access-reports-in-dentally)).
- Lifetime value: no page read defines a patient lifetime-value metric. **[UNVERIFIED]** that any benchmark system ships one.

**Coverage of what was read, by system** (blank means not read, not absent)

| Capability | Open Dental (reference) | Dentally (cloud) | CareStack (cloud) | Curve (cloud) | Dentrix Ascend (cloud) |
|---|---|---|---|---|---|
| Odontogram and notation | Four notations incl. FDI | Base chart plus plan tabs; PD / DD toggles | Odontogram or Advanced Planner | | |
| Plans, phases, alternates | One active, many inactive, saved signed copies; priorities | One tab per open plan; clone, templates | Plans (alternates) and phases | | |
| Perio | Full six-site chart, indices, voice | BPE history tab | | Six-site chart, 6 pathways, Bola voice | |
| Consent and e-sign | Sheets, e-signature, kiosk | Estimate / consent from plan, portal signing | Portal signing of plans | | |
| Recall | Types, triggers, intervals, disable rules | Dentist and hygienist intervals, booking link | | | |
| Online booking | Operatory flag for web scheduling | Portal booking, recall link | Direct and indirect booking | | |
| Per-provider finance | Production and Income, Provider Payroll | | | | Provider A/R Totals |

Sources for the matrix are the links cited earlier in this section and in section d, plus [Curve perio charting](https://www.curvedental.com/perio-charting). tab32 and Sensei Cloud were not read. **[UNVERIFIED]**

**Data-model reference** (all from the [Open Dental 26.3 schema documentation](https://www.opendental.com/OpenDentalDocumentation26-3.xml), 433 tables)

| Entity (table) | Key fields | Why it matters here |
|---|---|---|
| Procedure (`procedurelog`) | patient, appointment, planned appointment, tooth, surfaces, tooth range, status, priority, provider, clinic, diagnosis, prognosis, fee, discount, date planned, date complete, lock flag | One row type serves charting, planning, billing and production |
| Multi-visit link (`procmultivisit`) | group id, procedure, derived "In Process" flag | Crown prep and seat, dentures, staged work |
| Tooth state (`toothinitial`) | tooth, type (missing, hidden, primary, shifts, rotation, tipping, drawing), movement value | Odontogram state independent of procedures |
| Treatment plan (`treatplan`, `proctp`) | status saved / active / inactive, heading, note, patient and practice signatures, presenter; copied procedure lines with fee and patient amount | Signed, immutable plan snapshots |
| Perio (`perioexam`, `periomeasure`) | exam date and provider; one row per tooth per measure type (mobility, furcation, gingival margin, MGJ, probing, skip tooth, bleeding / suppuration / plaque / calculus flags) with six site values | Compact perio storage that feeds staging and grading |
| Lab (`labcase`, `laboratory`, `labturnaround`) | lab, appointment, planned appointment, due, created, sent, received, checked, instructions, fee; published and actual turnaround days | Lab-case tracking |
| Recall (`recalltype`, `recalltrigger`, `recall`) | default interval, time pattern, procedures; trigger codes; due date calculated and overridable, disable-until rules, ASAP priority | Recall engine |
| Calendar (`operatory`, `schedule`, `scheduleop`, `appointment`, `appointmenttype`) | operatory per clinic with default providers; time blocks typed provider / blockout / practice; appointment pattern, status, type, priority | Multi-chair, multi-provider scheduling |
| Ortho (`orthocase`, `orthoschedule`, `orthoplanlink`, `orthohardware`, `orthochartrow`) | case fee, banding, debond, expected debond, transfer flag; banding / visit / debond amounts; links to payment plans; hardware by exam date; signed chart rows | Ortho case and contract |
| Money (`paysplit`, `adjustment`, `payplan`) | split amount, provider, procedure, clinic, unearned type; adjustment amount, type, provider, procedure; plan guarantor, APR, frequency, signature | Per-provider collections and payment plans |
| CRM (`patient`, `commlog`, `referral`, `refattach`) | guarantor, super family, primary and secondary provider, status; comm type, mode, direction, user, signature; referral source flagged as patient, doctor or non-person, attached as to / from / custom | Households, comm log, referral tracking |
| Imaging (`document`, `mount`) | category, image type, tooth numbers, provider, signature | Attachments tagged to teeth |

### d. Visiting, partner and associate dentists

**Schedules and availability**

- Provider time is a dated schedule block that can be tied to specific operatories, and each operatory belongs to a clinic; the provider scheduled in an operatory takes precedence when an appointment is created ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml), [Operatories](https://www.opendental.com/manual/operatories.html)).
- Rotating or alternating patterns are built by copying and repeating days or weeks; "Offices can define any kind of rotating or alternating schedule they want"; editing needs a dedicated schedule permission ([Schedule](https://www.opendental.com/manual/schedule.html)).
- Lab due dates skip weekends and holidays but "Provider schedule is not taken into account" ([Lab Case](https://www.opendental.com/manual/labcaseedit.html)); a visiting dentist's next clinic day therefore has to be checked separately. **[INFERENCE]**
- Dolphin supports unlimited locations, per-location schedule templates and optional "Location Based Security" restricting staff to specific locations; charges can name "billing and treating dentist/ location at time of posting" ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)).

**How "production", "collections" and "adjustments" are defined per provider**

| System | Production | Adjustments | Collections | Attribution and caveats |
|---|---|---|---|---|
| Open Dental, Production and Income report ([manual](https://www.opendental.com/manual/reportprodinc.html)) | "Procedure fees for completed procedures (i.e., the gross fee billed to the patient)"; capitation excluded | Adjustments on completed procedures or posted to accounts; PPO write-offs dated by payment date, procedure date, or estimate-then-true-up | "Pt Income": patient payments that are allocated; "Ins Income": claim payments; total income is their sum; unearned (prepayments) is a separate column | Provider report "groups the totals by provider"; clinic comes from the procedure. Total production = Production + Sched + Adj − Write-off. Totals "may not match Aging of A/R amounts" |
| Open Dental, Provider Payroll report ([manual](https://www.opendental.com/manual/reportprovpayroll.html)) | UCR production = ProcFee × (UnitQty + BaseUnits), dated when set complete | "Prod Adj" excludes bad-debt adjustments; estimated write-off and change in write-off shown separately | Net income = allocated patient income + insurance income; unallocated patient income shown apart | Net production = gross production + estimated write-off + adjustments + change in write-off. Calculations were "custom-created for a specific use and may not apply to all offices"; requires all income to be allocated |
| Dentrix Ascend, Provider A/R Totals ([Henry Schein help](https://hsps.pro/DentrixAscend/Help/Understanding_the_Provider_AR_Totals_Report.htm)) | Production Total = Procedure Charges + production charge adjustments + production credit adjustments | Each adjustment is designated production or collection "according to their category and subcategory" | Collection Total = Payments applied to procedures + collection charge adjustments + collection credit adjustments | One row per provider plus an "<Unapplied Credits>" row; % Collected = Collection Total / Production Total × 100; ending balance = beginning + production − collection |

- The data rule underneath: a payment split is "Always attached to a payment. Always affects exactly one patient account and one provider", can point at one procedure, and carries its own clinic; an adjustment has its own provider and optional procedure ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)). Per-provider collections are only as good as payment allocation. **[INFERENCE]**
- Dolphin's Practice Analysis tracks "treatment starts, production, charges, receipts, adjustments, status changes, new-patient-added and accounts receivable" with drill-down to transactions ([brochure](https://www.dolphinimaging.com/Areas/Product/Documents/Management/ManagementBrochure.pdf)).

**Provider-level access**

- Open Dental: running the production report for anyone but yourself requires "Production and Income - View All Providers"; without it users see only the provider linked to their user ([manual](https://www.opendental.com/manual/reportprodinc.html)). Users and providers are separate entities that can be linked; `ClinicIsRestricted` limits a user to listed clinics; a provider-clinic link table records which providers are restricted to which clinics ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Record-level controls seen: ortho chart permission "Ortho Chart Edit (same user, signed)" ([Ortho Chart](https://www.opendental.com/manual/orthochart.html)); procedure `IsLocked` allows only append and sign ([schema](https://www.opendental.com/OpenDentalDocumentation26-3.xml)).
- Role sets seen in local products: dentist, dental nurse, cashier, clinic manager ([Dentalink](https://dentalink.id/fitur.html)); doctor, receptionist, admin with clinical data limited to doctors ([Dentiva](https://dentiva.id/)).
- None of the systems read documents a rule of the form "a visiting dentist sees only patients they treated". **[UNVERIFIED]** that such scoping exists in the benchmarks.

### e. Indonesian market: local products, from their own pages

| Product (vendor page) | Published price | Stated features | Not stated on the pages read |
|---|---|---|---|
| DIGMAN Apps, Carigi Indonesia ([page](https://carigi-indonesia.odoo.com/digital-management-apps)) | Rp 200rb / month; Rp 180rb (3 mo), 170rb (6 mo), 160rb (12 mo) | "RME Standar Kemenkes RI", "Integrasi SATUSEHAT & BPJS", "Odontogram & Indeks OHIS", "Analisis Cephalometric & Periochart", consent, cashier, instalments, insurance claims, stock, Google Calendar scheduling, online reservation page, WhatsApp notifications, payroll and attendance, referral letters, multi-branch; claims "400+ Klinik Aktif"; hospital and dental-lab editions "Coming Soon" | Dentist revenue share, patient app, treatment phases, lab-case tracking, recall rules |
| Dentalink ([home](https://dentalink.id/), [features](https://dentalink.id/fitur.html), [pricing](https://dentalink.id/harga.html)) | Rp199.000 / month; Rp1.990.000 / year. Pricing page also lists per-module amounts (odontogram Rp50.000, treatment packages Rp25.000, WhatsApp marketing Rp50.000, central warehouse Rp25.000, SATUSEHAT Rp50.000 per month); whether these are add-ons is unclear | Odontogram for permanent and primary teeth; RME with ICD-10; upload of periapical, panoramic, cephalometric and intraoral images; visual calendar with dentist schedule, treatment duration and dental-unit availability; treatment packages with visit quota (example "Paket Ortodonti 12x Kontrol"); POS; insurance module; automatic "bagi hasil (fee sharing) dokter mitra berdasarkan persentase tindakan"; roles for dentist, nurse, cashier, manager; WhatsApp H-1 reminders; SATUSEHAT integration listed; registered PSE | Perio chart, ceph analysis, treatment phases or alternatives, lab cases, e-consent, recall engine, patient portal, DICOM; notation not named (the example uses tooth 36) |
| Dentiva ([page](https://dentiva.id/)) | Starter Rp299.000, Pro Rp599.000, Enterprise Rp1.499.000 per month; annual pays 10 months | FDI odontogram with 52 positions, per-surface coding; SOAP with ICD-10 and ICD-9-CM, record locking and versioning; tablet e-signature consent as PDF; X-ray viewer; per-doctor daily capacity; WhatsApp reminders one day and one hour ahead; inventory with FEFO batches; billing with partial payment; cost and margin per treatment; SATUSEHAT in FHIR; roles doctor, receptionist, admin | Treatment plan, perio, ortho module, ceph, online booking, lab tracking, dentist fees, BPJS, patient app |
| Klinik Pintar ([clinic app](https://klinikpintar.id/aplikasiklinik), [dental page](https://klinikpintar.id/aplikasiklinik/faskes/klinik-gigi)) | Mandiri Rp0; Siap Rp300 ribu; Tumbuh Jaringan and Tumbuh Ranap Rp600 ribu per month | Odontogram in the RME module; queue; online reservation (Siap and up); inventory; cashier; SATUSEHAT (all plans); BPJS PCare and Mobile JKN (Siap and up); cross-branch integration and a commission module (top plans) | Calendar by chair, WhatsApp patient reminders, treatment plan, perio, ortho, imaging, consent, recall |
| Denta by Assist.id ([product](https://assist.id/produk/aplikasi-klinik-gigi-denta), [home](https://assist.id/)) | "Hubungi Kami" (no price) | "Odontogram Digital Interaktif", integrated dental records, photos and radiographs, online scheduling with reminders, queue, automatic billing, inventory, O-HIS reporting, BPJS and SATUSEHAT badges | SOAP, treatment plan, dentist fees, consent, ortho or perio, multi-branch for Denta |
| DentalCare Pro ([page](https://dentalcarepro.id/)) | Basic Rp149.000; Professional Rp299.000; Enterprise Rp999.000 per month; 14-day trial | Odontogram, records, doctor schedule and appointments, WhatsApp reminders, inventory, cashier, owner dashboard, BPJS (Professional), multi-branch and API access (Enterprise) | SATUSEHAT, treatment plan, perio, ortho, imaging, consent, online booking, recall, dentist fees |
| Sesnic ([page](https://sesnic.id/)) | Standart 275.000; Expert 375.000 per month; 30-day demo | Registration, records, anamnesis, diagnosis, treatment, prescription, odontogram, payment, reports, dashboards; Expert adds online registration and reservation reminder | SATUSEHAT, BPJS, inventory, calendar, consent, imaging, dentist fees |
| Periksa.id (general clinic; [home](https://periksa.id/), [clinic](https://periksa.id/solusi/klinik)) | No price shown | SOAP RME with SATUSEHAT, cashier with QRIS and VA, insurance billing, pharmacy stock, WhatsApp reminder "H-1 appointment", broadcast promo, booking confirmation; voice-to-SOAP "Medical AI" | Any dental feature |

- KlinikDent (klinikdent.id) timed out; its Rp99.000 price was seen only in a search snippet. **[UNVERIFIED]**
- Gaps relative to sections a to d, across every local page read: no phased or alternative treatment plans, no lab-case tracking, no recall engine with procedure triggers, no waitlist, no specialty visit templates, no per-provider production and collection definitions, no DICOM handling. Perio charting and ceph analysis are claimed by one vendor only (DIGMAN); explicit dentist fee sharing by one (Dentalink) and a commission module by one (Klinik Pintar). **[INFERENCE]** from the "not stated" column; absence on a marketing page is not proof of absence in the product.
- Price band for dental-specific local products: about Rp149.000 to Rp1.499.000 per clinic per month (table above).
- Pricing model: all published prices are flat per clinic per month with user or patient caps by tier (Dentiva: 3 users and 1,000 patients on Starter; DentalCare Pro: 2 users on Basic); none prices per dentist or per chair ([Dentiva](https://dentiva.id/), [DentalCare Pro](https://dentalcarepro.id/)).
- Common table stakes locally: odontogram, electronic record and cashier appear on all seven dental product pages; five of seven list SATUSEHAT (not DentalCare Pro or Sesnic); four of seven name WhatsApp for patient reminders or notifications (DIGMAN, Dentalink, Dentiva, DentalCare Pro), while Denta and Sesnic mention reminders without a channel (table above).
- Compliance signalling is part of local positioning: Dentalink advertises PSE registration and Kemenkes RME alignment on its home page ([Dentalink](https://dentalink.id/)). Verification of those claims belongs to the regulation track.

### Implications for the workspace platform

- **[INFERENCE]** Treat WebCeph as an external tool, not a dependency. Its API creates patients, records and image uploads and returns a link, but returns no landmarks, analyses or images (section a). The ortho module should store a WebCeph link per record set and accept manual upload of the exported PDF report and Excel tables.
- **[INFERENCE]** The differentiator is the staged case, not the image analysis. Ortho2, Dolphin and Open Dental all centre on a case with dates, a visit sequence, hardware log and a fee schedule (section a). Build one generic "treatment case" entity with stage templates, then specialise it per row of the specialty table.
- **[INFERENCE]** Stage templates should carry interval rules as data: minimum and typical gap, trigger event, and the citation behind them (6 to 10 weeks ortho, 3 to 12 months supportive perio care, 4 to 8 / 12 to 16 weeks / over 6 months implant placement, over 2 months conventional loading, 6 to 12 months endodontic review). Dentists must be able to override per patient, as NICE and the EFP both require tailoring (section b).
- **[INFERENCE]** Model procedures the Open Dental way: immutable procedure rows with status TP / complete / existing / referred / condition, tooth and surface fields, priority for phasing, and saved plan snapshots for signature (section c). This gives treatment plans, alternatives, consent and production reporting from one table.
- **[INFERENCE]** Store teeth in one canonical notation and display FDI by default for Indonesia; Open Dental's store-Universal, display-by-preference design shows this works (section c). Include primary and mixed dentition from day one because pediatric recall and charting depend on it (section b).
- **[INFERENCE]** The perio chart is worth building natively: six sites per tooth with the Open Dental measure types maps directly onto AAP staging and grading inputs, and only one local competitor claims a perio chart (sections b, c, e).
- **[INFERENCE]** Lab-case tracking is a clear local gap: four timestamps, due date from per-lab turnaround, and a link to the delivery appointment (sections b, e). It also feeds the finance track's lab-cost data.
- **[INFERENCE]** Recall should be a rules engine, not a reminder field: recall types with trigger procedures, default interval, per-patient override and disable conditions, plus a waitlist flag (section c). No local product page describes this (section e).
- **[INFERENCE]** Calendar model: operatory (dental unit) columns, provider time blocks linked to operatories and clinics, blockouts restricted by appointment type, and a doctor-time versus assistant-time pattern (section c). This is what makes a visiting dentist's availability bookable across clinics (section d).
- **[INFERENCE]** Define production, adjustments and collections explicitly and per provider before building dashboards. Open Dental and Dentrix Ascend differ on where adjustments land, and both attribute collections through payment splits tied to one provider (section d). Require every payment line to name a provider and, where possible, a procedure.
- **[INFERENCE]** Partner-dentist access needs its own rule set. Benchmarks restrict by clinic, by report permission and by signed-record locking, but none documents "own patients only" scoping (section d). This must be designed with the regulation track.
- **[INFERENCE]** Imaging: start with categorised attachments (radiograph, photo, document) with tooth tags and a signed-record flag; accept DICOM as a stored file with preview later. AI radiograph reading and implant planning are integrate-later items; Open Dental itself integrates Pearl by polling rather than building detection (sections b, c).
- **[INFERENCE]** Pricing reference: local dental products sit between about Rp149.000 and Rp1.499.000 per month, while WebCeph alone costs up to $19.99 per user per month plus VAT (sections a, e). A specialty module can be priced as an add-on above the local base band.

### Open questions

- Will WebCeph approve a partner API key for this platform, and what will the planned paid tier cost? The API page says approval is separate from subscription and pricing is pending (section a).
- Which specialties do the first partner dentists actually practise? The table supports nine; building all stage templates at once is not justified without that mix.
- Does the regulation track require a specific odontogram symbol set and notation for Indonesian electronic records? This track only confirmed that vendors claim Kemenkes-standard odontograms (section e).
- Should partner dentists see only patients they have treated, all patients of a clinic, or a configurable scope? No benchmark answers this (section d).
- Are fees for multi-visit work (crowns, dentures, ortho) recognised at the first visit, the last visit, or spread across visits? Open Dental forces the practice to choose (section b); the finance track needs the same decision.
- Is voice or AI-assisted charting in scope for the first release? Curve and Open Dental both rely on a third party (Bola AI) for it (sections b, c).
- Which lab workflow is typical for target clinics: external labs only, or in-house lab too? DIGMAN lists a dental-laboratory edition as coming soon (section e).
- Items still to confirm at primary sources before the note is final: AAOMS post-extraction review timing, ESE endodontic follow-up schedule, EFP re-evaluation timing in weeks, veneer visit sequence, ISO 3950 text, Cloud 9 and tops feature sets, Dentrix Ascend clinical features, tab32 and Sensei Cloud.


---

## 3. Inventory, asset and financial management

Research date: 2026-11-25. Tags: **[UNVERIFIED]** = not confirmed at a primary source in this session; **[INFERENCE]** = this track's own reasoning. Regulation PDFs were read from the linked official files (text layer, or page images where scanned). This note informs software design; it is **not tax or legal advice** — items marked "consultant" must be confirmed by a licensed Indonesian tax consultant (konsultan pajak) or health-law counsel before build.

Access limits met in this session: gs1.org pages returned HTTP 403; xendit.co pricing page 403; developer.xero.com doc pages are JS-only (empty to the fetcher); api-doc.jurnal.id did not respond and developers.mekari.com redirected to a maintenance page; the PP 28/2024 PDF on BPK exceeds the fetcher's size limit; PubMed/PMC pages were blocked by cookie/CAPTCHA walls; one ADA "Guidelines for Practice Success" page returned 403.

### a. Consumables inventory — what dedicated systems and PMS modules actually do

**Feature matrix (from each vendor's own pages)**

| Capability | Evidence from first-party docs |
|---|---|
| Item master + units | Open Dental's supply record holds Description (required; vendor advises writing the pack unit into it, e.g. "box of 100"), Catalog Item Number, Category, Supplier, Price, Stock Level ("minimum number of items to keep on hand"), On Hand Qty, Order Qty, Hidden; units "should match how they are ordered"; no barcode field is documented ([Open Dental – Supplies](https://opendental.com/manual/supplies.html)). Hybrent keeps "a controlled item master with preferred vendors and formulary items" ([Hybrent](https://procurementpartners.com/solutions/hybrent)). CureMint offers "Custom Formulary and Shopping Lists" ([CureMint](https://www.curemint.io/)). |
| Par level / reorder point | Open Dental: Stock Level = minimum on hand, with a free-form "Supplies Needed" list any staff member can add to ([Open Dental – Supply Inventory](https://opendental.com/manual/supplyinventory.html)). Sowingo: thresholds trigger "real-time low level inventory alerts" ([Sowingo – General Inventory](https://sowingo.com/general-inventory/)). CureMint: "Quickly Reorder Items Below Par Level" ([CureMint](https://www.curemint.io/)). Method: physical reorder tags placed at pre-calculated reorder points; staff drop the tag in a "Need to Order" bin; scanning the collected tags builds the order with predetermined quantities; tags return to the shelf at receipt ([Method – Reorder Tag System](https://www.methodusa.com/platform/inventory-management/reorder-tag-system)). |
| Lot / expiry / FEFO | Hybrent: barcode scanning "to order and deplete inventory with tracking for lot number, serial number, and expiration date" ([Hybrent](https://procurementpartners.com/solutions/hybrent)). Sowingo: expiration-date alerts for general stock and medications; implants tracked "by lot number and expiration date", by storage location, with automated FIFO and recall lookup by lot number to find the affected patient ([Sowingo – Implants](https://sowingo.com/implants/), [Sowingo – Medications](https://sowingo.com/medications/)). Open Dental's supply pages document no lot or expiry field ([Open Dental – Supplies](https://opendental.com/manual/supplies.html)). |
| Barcode scanning | Sowingo: scan products in and out with the mobile app ([Sowingo – Medications](https://sowingo.com/medications/)). Hybrent: scan to order and to deplete ([Hybrent](https://procurementpartners.com/solutions/hybrent)). Method: scan reorder tags ([Method](https://www.methodusa.com/platform/inventory-management/reorder-tag-system)). |
| Per-procedure consumption | MedicaSimple (dental/clinic PMS): materials are linked to treatments and "deducted from the main depot automatically when clinical work is completed"; each branch has its own depot and deduction hits the clinic where treatment was done; material spend "stays linked to the cost of each treatment" ([MedicaSimple – Stock management](https://medicasimple.com/features/stock-management)). Pabau (clinic PMS): products are linked to a service with a "standard amount of the consumable product required"; they appear pre-selected under the service at point of sale, can be removed per sale, and a "Consumable deduction" option deducts stock "when booking this service" ([Pabau – Automatic consumption](https://support.pabau.com/en/pabau2/how-to-use-automatic-consumption)). None of the dedicated dental procurement tools read here (Sowingo, Hybrent, CureMint, Method) documents a procedure bill of materials; they deplete by scan or count. |
| Purchase orders / suppliers | Open Dental: an order is "pending" until Date Placed is entered; total = quantity × price, shipping entered separately; suppliers and categories must exist first; suppliers attached to orders cannot be deleted ([Open Dental – Supply Orders](https://opendental.com/manual/supplyorders.html), [Supply Inventory](https://opendental.com/manual/supplyinventory.html)). Sowingo: POs with preferred suppliers, marketplace price comparison "across 10+ suppliers", 3-way matching, vendor links via API/EDI/cXML ([Sowingo](https://sowingo.com/)). CureMint: cross-supplier search and ordering, "Automated Review and Approval Workflows", per-office budgets ([CureMint](https://www.curemint.io/)). Hybrent: approval workflows; EDI and non-EDI vendors; punch-out ([Hybrent](https://procurementpartners.com/solutions/hybrent)). |
| Receiving | Open Dental: Date Received per order or per line for partial shipments; unreceived lines stay "pending" ([Open Dental – Supply Orders](https://opendental.com/manual/supplyorders.html)); the page does not state that receiving updates On Hand ([same page](https://opendental.com/manual/supplyorders.html)). Hybrent: "Track supply receipts with real time inventory values" ([Hybrent](https://procurementpartners.com/solutions/hybrent)). |
| Invoice matching / AP | Hybrent: "two-way or three-way match analysis"; approved invoices pushed to accounting software ([Hybrent](https://procurementpartners.com/solutions/hybrent)). CureMint: "Automated 3-Way Matching and Duplicate Bill Detection", GL coding, bill approval, integration with the financial system ([CureMint](https://www.curemint.io/)). |
| Multi-location transfers, counts | Hybrent: "manage transfers and cycle counts", visibility across facilities ([Hybrent](https://procurementpartners.com/solutions/hybrent)). Open Dental: "Enter Quantities" grid for typing On Hand and Order numbers ([Open Dental – Supplies](https://opendental.com/manual/supplies.html)). |
| Wastage / shrinkage | Sowingo positions expiry alerts as waste prevention and keeps a detailed activity log exportable to Excel ([Sowingo – General Inventory](https://sowingo.com/general-inventory/)). Method: "Track when products are used and monitor for unexplained losses or theft" ([Method](https://www.methodusa.com/platform/inventory-management/reorder-tag-system)). |
| Price history / spend | Sowingo: "tracking your previous prices paid and order details" ([Sowingo – General Inventory](https://sowingo.com/general-inventory/)). CureMint: spend dashboard vs budget per office ([CureMint](https://www.curemint.io/)). |
| Implant / device-to-patient traceability | Sowingo attaches implant products to patients, logs the referring doctor, reports implants placed per doctor, labels consignment stock ([Sowingo – Implants](https://sowingo.com/implants/)). |

- A mainstream dental PMS can ship a deliberately thin inventory module: Open Dental's is supplier/supply/order/equipment lists with manually typed quantities and no documented lot, expiry, barcode or automatic depletion ([Open Dental – Supplies](https://opendental.com/manual/supplies.html), [Supply Orders](https://opendental.com/manual/supplyorders.html)).
- ZenSupplies/ZenOne: no first-party documentation was retrieved in this session — **[UNVERIFIED]**, not characterised here.
- Pattern across the products read **[INFERENCE]**:
  - Procurement-first tools (Sowingo, Hybrent, CureMint, Method) are strong on ordering, approvals, receiving and invoice matching, and deplete stock by scan, tag or count — they do not know which procedure used an item.
  - Clinic PMS products with stock (MedicaSimple, Pabau) deplete from a per-service material list at completion or sale, which is what makes cost per procedure possible — but they publish little on lot-level traceability.
  - No product read combines both with Indonesian identifiers (izin edar) — that combination is the gap this platform can fill.

**Identifiers on the box: UDI and GS1**

- A UDI = Device Identifier (DI: "mandatory, fixed portion… identifies the labeler and the specific version or model") + Production Identifier (PI: lot/batch, serial number, expiration date, manufacture date) ([FDA – UDI Basics](https://www.fda.gov/medical-devices/unique-device-identification-system-udi-system/udi-basics)). The US rule owner is FDA; its database (GUDID) stores the DI only, not the PI ([FDA – UDI Basics](https://www.fda.gov/medical-devices/unique-device-identification-system-udi-system/udi-basics)).
- FDA has accredited three UDI issuing agencies — GS1, HIBCC, ICCBBA. GS1 format: `(01)` DI, numeric, 14 digits (the GTIN); `(11)` production date YYMMDD; `(17)` expiration date YYMMDD; `(10)` batch/lot, alphanumeric ≤20; `(21)` serial, alphanumeric ≤20. Example: `(01)51022222233336(11)141231(17)150707(10)A213B1(21)1234`. HIBCC uses a different syntax (`+` DI, `$`/`$$` lot and expiry) ([FDA – UDI formats by FDA-accredited issuing agency, v1.4, 1 Sep 2025](https://www.fda.gov/media/96648/download)).
- GS1's own UDI and Application Identifier pages could not be opened (403 / unrendered) — GS1 General Specifications details beyond the FDA table are **[UNVERIFIED]**.
- Indonesia: a keyword search of the full text of Permenkes 5/2026 for "UDI", "identifikasi unik" and "ketertelusuran" found nothing ([Permenkes 5/2026 PDF](https://peraturan.bpk.go.id/Download/414598/Permenkes%205%20Tahun%202026.pdf)). Whether Kemenkes has any separate UDI obligation is **[UNVERIFIED]**. Imported dental products will often carry GS1/HIBCC codes anyway **[INFERENCE]**.

### b. Indonesian supply-side rules that shape the module

**Medical-device registration (izin edar) — regulation changed in May 2026**

- Permenkes 5/2026 "Perbekalan Kesehatan" — enacted 17 Apr 2026, promulgated and effective 4 May 2026, status Berlaku — revoked, among others, Permenkes 62/2017 (izin edar alkes/IVD/PKRT), Permenkes 54/2015 (testing and calibration of medical devices), Permenkes 1191/2010 (device distribution), Permenkes 5/2019 (medicine procurement via e-catalogue), and most of Permenkes 4/2014 (good distribution practice, except Pasal 2 and annex) ([BPK – Permenkes 5/2026](https://peraturan.bpk.go.id/Details/350852/permenkes-no-5-tahun-2026)).
- Under it, health supplies "hanya dapat diedarkan setelah memperoleh perizinan berusaha berupa izin edar" (Pasal 50(1)); the izin edar holder must label the product with safety, benefit and usage information (Pasal 52); products that are sub-standard, lack an izin edar, have a revoked or lapsed izin edar, or are expired must be destroyed, and facilities that hold them must do so to standard (Pasal 89–90) ([Permenkes 5/2026 PDF](https://peraturan.bpk.go.id/Download/414598/Permenkes%205%20Tahun%202026.pdf)).
- Transition: existing arrangements must conform within one year of promulgation (Pasal 122); earlier implementing provisions stay valid where not contradictory (Pasal 124) ([Permenkes 5/2026 PDF](https://peraturan.bpk.go.id/Download/414598/Permenkes%205%20Tahun%202026.pdf)).
- Historical detail from the now-revoked Permenkes 62/2017 (useful for data modelling; whether each point is carried forward is **[UNVERIFIED]**): risk classes A (low) to D (high) (Pasal 7); izin edar valid at most 5 years, or the distributor-appointment period, 3 years for OEM (Pasal 24); label must show trade name, izin edar number, product type, manufacturer and izin-edar holder, batch/production code/serial, "steril" and method, expiry date where applicable (Pasal 41(2)) ([Permenkes 62/2017](https://jdih.kemkes.go.id/storage/documents/pdfs/2017permenkes062.pdf)).
- Number format and lookup: Kemenkes' Farmalkes FAQ states the izin edar number is "11 digit angka" preceded by **AKD** (domestic device) or **AKL** (imported device) — PKD/PKL for household health supplies — and that it can be checked at infoalkes.kemkes.go.id or the Mobile Alkes app ([Farmalkes FAQ](https://farmalkes.kemkes.go.id/faq)). e-Info Alkes is described as the ministry's official platform, operated by Direktorat Produksi dan Distribusi Alat Kesehatan (Ditjen Farmalkes), with a "Cek Izin" search and lists of domestic/imported devices and distributors ([infoalkes.kemkes.go.id](https://infoalkes.kemkes.go.id/)). The meaning of each digit group is not published on those pages.

**Medicines and anesthetics held in a clinic** (Permenkes 34/2021, Standar Pelayanan Kefarmasian di Klinik; listed as in force on Farmalkes per search result, status not re-opened this session)

- Pharmaceutical service in a clinic runs in an Instalasi Farmasi; an outpatient clinic *may* provide it and, if it does, must have an Apoteker in charge; if it does not, it may cooperate with another clinic or a pharmacy, and prescriptions are served there (Pasal 3(1), (4)–(6); Pasal 4(6)) ([Permenkes 34/2021](https://www.peraturan.go.id/files/bn1376-2021.pdf)).
- Management cycle for outpatient clinics: selection, planning, procurement, receiving, storage, destruction and recall, control, administration (Pasal 4(2)). Sources allowed: pharmaceutical industry, PBF wholesaler, device distributor, device shop, and pharmacy only to cover scarcity/stock-out (Pasal 5(1)–(2)). Orders use a *surat pesanan* signed by the Apoteker with SIPA number, which may be electronic (Pasal 5(3)–(5)) ([Permenkes 34/2021](https://www.peraturan.go.id/files/bn1376-2021.pdf)).
- Narcotics, psychotropics and hard drugs (obat keras) are handed over only on prescription (Pasal 6(3)–(4)); prescriptions are confidential and kept at least 5 years (Pasal 10) ([Permenkes 34/2021](https://www.peraturan.go.id/files/bn1376-2021.pdf)).
- Annex (scanned pages 22–41 read): planning by consumption method `A = (B + C + D) − E` (A plan, B average monthly use, C buffer stock, D lead-time stock, E remaining stock), plus ABC (≈70/20/10 % of spend) and VEN analysis; an electronic ordering system "harus bisa menjamin ketertelusuran produk" and have electronic backup; receiving checks packaging/label, match to order and invoice, and **batch number and expiry date**; storage rules include temperature monitoring with a calibrated thermometer and a temperature log, issue by **FIFO/FEFO**, containers showing name, batch number and expiry, and special marking and separate storage for items 3–6 months from expiry; narcotic and psychotropic orders use dedicated forms in triplicate; high-alert and look-alike/sound-alike items are stored separately and labelled ([Permenkes 34/2021 annex](https://www.peraturan.go.id/files/bn1376-2021.pdf)).
- The annex's administration section (stock cards, stock opname, narcotic reporting) was not read — stock-card field requirements are **[UNVERIFIED]**. Whether a dental clinic that keeps local anesthetics only for chair-side use (no dispensing) needs an Instalasi Farmasi/Apoteker is not answered by the articles read — **consultant**.

**Government e-catalogue**: the ministerial rule on medicine planning and procurement via the electronic catalogue (Permenkes 5/2019) is among those revoked by Permenkes 5/2026 ([BPK – Permenkes 5/2026](https://peraturan.bpk.go.id/Details/350852/permenkes-no-5-tahun-2026)). It governs public procurement; no source read makes it a duty for private clinics, so it is out of scope for v1 **[INFERENCE]**.

### c. Asset management and the compliance duties attached to dental equipment

**What products do**

- Open Dental's Equipment list exists "to track equipment for payment of property taxes": description, auto-generated unique serial number, model year, date purchased, date sold, purchase cost, estimated market value, location (e.g. OP-2), status; no depreciation or maintenance fields are documented ([Open Dental – Equipment](https://opendental.com/manual/equipment.html)).
- Sowingo's compliance module logs sterilizer biological-indicator incubation cycles and outcomes, chemical-indicator strips, washer/ultrasonic test strips, optional autoclave-resistant pouch labels to "trace instruments back to the patient", one-click audit trail, plus "equipment, licensing, insurance and permit tracking", renewal alerts and certification history ([Sowingo – Compliance](https://sowingo.com/compliance/)).
- Accurate Online exposes a `fixed-asset` API group (see f) ([Accurate OpenAPI](https://account.accurate.id/open-api/json.do)).

**Indonesian duties, by owner**

| Duty | Source and content | Design consequence **[INFERENCE]** |
|---|---|---|
| Testing / calibration of devices | Permenkes 54/2015 is "Tidak Berlaku", revoked by Permenkes 5/2026 ([BPK – Permenkes 54/2015](https://peraturan.bpk.go.id/Details/116264/permenkes-no-54-tahun-2015)); the only occurrence of "kalibrasi" in Permenkes 5/2026 is that revocation clause ([Permenkes 5/2026 PDF](https://peraturan.bpk.go.id/Download/414598/Permenkes%205%20Tahun%202026.pdf)). PP 28/2024 is in force ([BPK – PP 28/2024](https://peraturan.bpk.go.id/Details/294077/pp-no-28-tahun-2024)); a search result quotes its Pasal 815 as requiring hospital equipment to be tested and calibrated periodically by an authorised institution, but the PDF could not be opened — **[UNVERIFIED]**, as are the current interval and its application to clinics (the revoked rule was reported as at least yearly). | Calibration interval must be a configurable field per asset type, default 12 months, with certificate upload and provider name; do not hard-code a legal citation. |
| ASPAK reporting | Permenkes 31/2018: ASPAK is the web application gathering facility, infrastructure and medical-device data of health facilities (Pasal 1(4)); it "harus diselenggarakan" by health facilities, which appoint an operator and input/update data (Pasal 5(1)–(2)); accounts come from the district/provincial health office or the ministry according to who licensed the facility (Pasal 6); data must be updated at least twice a year, each 30 June and 31 December, then validated by the health office (Pasal 7); ASPAK may interoperate with other applications (Pasal 4(2)–(3)) ([Permenkes 31/2018](https://www.peraturan.go.id/files/bn1012-2018.pdf)). Current status not re-checked at BPK — **[UNVERIFIED]**. | Asset register should export an ASPAK-shaped device list and raise reminders ahead of 30 Jun / 31 Dec. |
| Dental X-ray (BAPETEN) | Peraturan BAPETEN 4/2020: dental units are defined as intraoral, extraoral 2D and extraoral 3D (Pasal 1); personnel = radiology medical staff, health workers and a Petugas Proteksi Radiasi (Pasal 9); a dentist must be available for intraoral units, a dental-radiology specialist for extraoral 2D/3D units, or a dentist with additional competence granted clinical authority where no specialist exists (Pasal 10); the licence holder must keep records including X-ray inventory (technical specification, tube replacement), personal dose evaluations, patient dose, safety-verification results, radiation-meter calibration certificates, training records and **maintenance and repair data** (Pasal 15–16); yearly online reports through B@LIS (Pasal 17(2)); dose records kept at least 30 years after the worker stops or until age 75 (Pasal 36(2)); external quality control via Uji Kesesuaian (Pasal 64) ([Peraturan BAPETEN 4/2020](https://www.peraturan.go.id/files/bn1218-2020.pdf)). Later amendment status **[UNVERIFIED]**. | X-ray units need an asset subtype with licence number/expiry, compliance-test date and certificate, PPR and dosimeter records, tube-replacement log, annual B@LIS reminder. |
| X-ray licensing framework | Peraturan BAPETEN 5/2025 (set 19 Dec 2025) implements PP 28/2025 on risk-based licensing and revokes Peraturan BAPETEN 3/2021 and 1/2022 (Pasal 182); licence validity follows statute for radiation-source use, and existing licences stay valid until expiry (Pasal 11 and transitional provisions) ([Peraturan BAPETEN 5/2025](https://www.peraturan.go.id/files/peraturan-bapeten-no-5-tahun-2025.pdf)). Exact validity years and requirements for dental units — **[UNVERIFIED]**. | Store licence validity as data, not as a rule. |
| Sterilization / infection control | Permenkes 27/2017 (in force per [BPK](https://peraturan.bpk.go.id/Details/112075/permenkes-no-27-tahun-2017)) sets the Spaulding categories; critical items must be disinfected and sterilized; steam sterilization at 121 °C, 106 kPa, 20 min unwrapped / 30 min wrapped ([Permenkes 27/2017 annex](https://jdih.kemkes.go.id/storage/documents/pdfs/2017permenkes027.pdf)). A text search of that PDF finds no requirement on chemical/biological indicators or per-load sterilization records ([same PDF](https://jdih.kemkes.go.id/storage/documents/pdfs/2017permenkes027.pdf)) — Indonesian source silent. | Fall back to CDC (next row). |
| Sterilizer monitoring (CDC, where Indonesia is silent) | Mechanical monitoring "for every sterilizer load"; a chemical indicator inside every package; a spore test "at least weekly" and "for every load with an implantable device"; keep mechanical, chemical and biological monitoring records; after a positive spore test remove the sterilizer from service, recall and quarantine implantable items, and re-process suspect loads back to the last negative test where possible ([CDC – Sterilization monitoring](https://www.cdc.gov/dental-infection-control/hcp/dental-ipc-faqs/sterilization-monitoring.html)). | Autoclave = asset with a cycle log (load ID, operator, parameters, CI/BI result); pack label ID links load → instrument set → patient visit, enabling recall. |

**Fiscal depreciation (PMK 72/2023, in force since 17 Jul 2023 per [BPK](https://peraturan.bpk.go.id/Details/257823/pmk-no-72-tahun-2023))**

- Tangible assets used to earn income with useful life over one year are depreciated in equal parts over the prescribed life (straight line); non-building assets may instead use declining balance on book value with the remainder written off in the final year, if applied consistently (Pasal 2(1)–(2)) ([PMK 72/2023](https://peraturan.go.id/files/permenkeu-no-72-tahun-2023.pdf)).

| Group | Useful life | Straight line | Declining balance |
|---|---|---|---|
| Non-building Kelompok 1 | 4 years | 25 % | 50 % |
| Kelompok 2 | 8 years | 12.5 % | 25 % |
| Kelompok 3 | 16 years | 6.25 % | 12.5 % |
| Kelompok 4 | 20 years | 5 % | 10 % |
| Building, permanent | 20 years | 5 % | — |
| Building, non-permanent | 10 years | 10 % | — |

Source: Pasal 2(3) ([PMK 72/2023](https://peraturan.go.id/files/permenkeu-no-72-tahun-2023.pdf)).

- Asset types per group are in the annex (Pasal 3); an asset **not listed** uses Kelompok 3 unless the taxpayer obtains a DJP determination for another group (Pasal 4) ([PMK 72/2023](https://peraturan.go.id/files/permenkeu-no-72-tahun-2023.pdf)).
- Depreciation starts in the month of the expenditure (or completion month for assets under construction; month of first use only with DJP approval) (Pasal 5) ([PMK 72/2023](https://peraturan.go.id/files/permenkeu-no-72-tahun-2023.pdf)).
- Annex pages 25–28 (read as images): Kelompok 1, all business types — wooden/rattan furniture; office machines incl. laptop, computer, printer, scanner; other electronics; motorcycles; "alat perlengkapan khusus (tools) bagi industri/jasa yang bersangkutan"; communication devices. Kelompok 2, all business types — metal furniture, air conditioners, cars. Kelompok 3 and 4 list industry machinery only. **No entry names medical or dental equipment** ([PMK 72/2023 annex](https://peraturan.go.id/files/permenkeu-no-72-tahun-2023.pdf)).
- Hence a dental unit, autoclave, compressor or X-ray set either defaults to Kelompok 3 (16 years) under Pasal 4 or is argued to be "special tools for the service concerned" (Kelompok 1) — an interpretation call **[INFERENCE]** → **consultant**; the platform must let the tenant pick the fiscal group per asset class and keep a separate commercial (management) useful life.

### d. Partner-dentist fees ("jasa medis")

**How pay is modelled in established sources**

- ADA's worked example for one crown (production $1,500, adjusted production $1,000, collected $950): 35 % of collections = $227.50; 35 % of adjusted production = $350; 30 % of production = $450; also salary, salary plus commission (e.g. $7,000/month plus 10 % of the commission base), and net-profit models where pay is based on billings "less any expenses as specified in your contract" such as lab fees and discounts ([ADA – Dentist compensation](https://www.ada.org/resources/careers/dentist-compensation)). "Adjusted production" = "the amount the office is contractually allowed to collect" ([same page](https://www.ada.org/resources/careers/dentist-compensation)).
- CareStack Provider Payroll: batches by Monthly/Weekly/Biweekly/Custom period; summary of Gross Production, Net Production, Additions, Deductions, Gross Amount; detail of Earned Share, Minimum Guarantee, Estimated Gross, Lab Cost Percentage, Estimated Lab Cost; lab cases pulled by received date in the period and total lab cost editable in review (edits tracked); a **daily minimum guarantee** with adjustable working days, flagged and routed to Pending Review; one active payroll per provider per date range, void-and-regenerate; per-location payout breakdown; click-through to the underlying calculation ([CareStack – Provider Payroll](https://carestack.com/dental-software/features/provider-payroll)).
- Open Dental Provider Payroll report (Summary / Patient Detail): UCR Production, Est Write-off, Prod Adj (excluding bad debt), Change in Write-off, Net Prod, Pat Inc Alloc (payments attached to a procedure), Pat Inc Unalloc, Ins Income, Ins Not Final, Net Income = allocated patient income + insurance income; used where "providers earn a percentage of net income"; all income must be allocated first; usually run per provider for the pay period; the manual warns the calculations "may not apply to all offices" ([Open Dental – Provider Payroll](https://opendental.com/manual/reportprovpayroll.html)).
- Dentally: pay associates from Monthly Accounts → "Totals by practitioner" → **Net payments**; deposits count only once the work is performed, and corrections, deposits used and refunds issued in the month are already netted; figures are fixed after month end ([Dentally – Monthly Accounts](https://help.dentally.com/en/articles/7866192-how-to-use-the-monthly-accounts-report-to-pay-associates)). The older Payment Allocation Totals report is "no longer the recommended report" ([Dentally – legacy report](https://help.dentally.com/en/articles/4601844-how-do-i-use-dentally-reports-to-pay-associates)).
- Indonesia's own regulation illustrates a split: the hospital keeps 20 % of each doctor fee paid by the patient and pays 80 % to the doctor at month end ([PMK 168/2023, annex example V.2](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf)).

**Design dimensions for a fee engine [INFERENCE]** (each grounded in the sources above)

| Dimension | Options to support | Why |
|---|---|---|
| Base | gross production / adjusted (net of discounts and write-offs) / collections allocated to the procedure | ADA's three bases give $450 / $350 / $227.50 on the same crown. |
| Timing | on completion; on payment received; deposits recognised only when work is done | Dentally's deposit rule; Open Dental's allocated-income basis. |
| Deductions before or after the percentage | lab fee, material cost (from the procedure BOM in section a), gateway fee, discount borne by clinic/dentist/shared | ADA net-profit model; CareStack lab-cost fields. |
| Rate schedule | flat %, per-procedure-category %, fixed fee per procedure, tiered by monthly base | Per-procedure and tiered schedules are not documented in the vendor pages read; treat as a product decision. |
| Floor | daily or monthly minimum guarantee, flagged for review | CareStack. |
| Multi-dentist procedure | split lines per performer (operator, anesthesia, assistant dentist) with shares summing to 100 % | Open Dental's pay report is built from payment splits attached to a procedure and a provider; none of the pages read documents multi-performer splitting, so this is a product decision. |
| Refunds / corrections | reverse in the period of the refund; never rewrite a closed period | Dentally fixes month figures; CareStack void-and-regenerate. |
| Instalments | fee accrues pro rata to each instalment collected (collections base) or fully on completion (production base) | Follows from the base choice. |
| Auditability | every payout line drills down to procedure, payment and rule version | CareStack click-through. |

**Indonesian tax the payout must implement**

- *Classification*: "Bukan Pegawai" covers experts doing independent work, expressly including **dokter** (PMK 168/2023 Pasal 3(2)a) ([PMK 168/2023](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf)). The regulation says "dokter" and does not separately name dentists; applying it to dokter gigi is the natural reading **[INFERENCE]** → consultant. Dentists who are employees fall under the monthly effective-rate scheme, not covered here.
- *Base*: 50 % of gross income (Pasal 12(3)). For "jasa dokter yang melakukan praktik di rumah sakit dan/atau klinik", gross = the doctor fee paid by the patient through the hospital/clinic **before** deduction of costs or the facility's profit share (annex IV.2.b) ([PMK 168/2023](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf)).
- *Rate*: Pasal 17(1)(a) income-tax rates × that base (Pasal 16(3)), computed for the income of one tax period or when the income becomes payable (annex IV.1) ([PMK 168/2023](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf)). Brackets: 5 % up to Rp60 million; 15 % above Rp60 m to Rp250 m; 25 % above Rp250 m to Rp500 m; 30 % above Rp500 m to Rp5 billion; 35 % above Rp5 billion ([DJP – Buku PPh 21/26, 2024](https://static.pajak.go.id/download/kalkulator/Buku_PPh2126_Release_20240108.pdf)).
- *Not cumulative*: DJP states the non-employee base is "tidak dikumulatif dengan penghasilan masa-masa sebelumnya" and the calculation is a single formula, "tidak akumulatif" ([DJP – Buku PPh 21/26](https://static.pajak.go.id/download/kalkulator/Buku_PPh2126_Release_20240108.pdf)).
- *Official examples*: doctor fees paid by patients Rp45,000,000 in January → base Rp22,500,000 → 5 % → Rp1,125,000, repeated month by month, with a withholding slip made each month ([PMK 168/2023 annex V.2](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf)); a Rp400,000,000 single payment → base Rp200,000,000 → (5 % × 60 m) + (15 % × 140 m) = Rp24,000,000 ([DJP – Buku PPh 21/26](https://static.pajak.go.id/download/kalkulator/Buku_PPh2126_Release_20240108.pdf)).
- *Consequence* **[INFERENCE]**: the tax base is the patient-paid doctor fee, not the dentist's share. With patient-paid fees of Rp100 m and a 40 % share (Rp40 m), withholding = 5 % × Rp50 m = Rp2.5 m, i.e. 6.25 % of the payout. The engine therefore needs both figures per period. Where the clinic bills one bundled price (fee + materials + lab), what counts as "jasa dokter yang dibayar oleh pasien" is not defined in the texts read → **consultant**.
- *Calculation sketch* **[INFERENCE]** (per dentist, per tax period; to be validated by the consultant before coding):

```text
doctor_fee_gross = Σ "jasa dokter" paid by patients through the clinic in the period   # before clinic share and costs
pph21_base       = 50% × doctor_fee_gross                                               # PMK 168/2023 Ps. 12(3)
pph21            = progressive(pph21_base; 5% ≤60m, 15% ≤250m, 25% ≤500m, 30% ≤5bn, 35% above)   # per period, not cumulative
payout_gross     = contract_rule(base, deductions, schedule, guarantee)                # the clinic–dentist agreement
payout_net       = payout_gross − pph21 − other agreed deductions
slip             = BP21 { recipient, tax-object code, period, gross, base, tax }       # one per recipient/code/period
```

- *Withholder duties*: compute, withhold, remit and report each tax period; make a withholding slip and give it to the recipient; keep a calculation worksheet per recipient (Pasal 20(1)); effective 1 Jan 2024; JDIH status Berlaku with no amendment listed ([PMK 168/2023](https://jdih.kemenkeu.go.id/api/download/e60a82e0-b218-40f5-9d18-b924aa1e11ce/2023pmkeuangan168.pdf), [JDIH Kemenkeu record](https://jdih.kemenkeu.go.id/dok/pmk-168-tahun-2023/files)).
- *Withholding slip in the current DJP system*: PER-11/PJ/2025 (22 May 2025) names four forms — BPA1 (permanent employees/pensioners), BPA2 (civil servants etc.), **BP21** (other resident recipients, final and non-final Pasal 21) and BP26; they are electronic documents made in the **eBupot module** of the taxpayer portal or an integrated application, with electronic signature (Pasal 6); BP21 "dapat dibuat untuk setiap transaksi atau untuk 1 (satu) Masa Pajak", must be given to the recipient each time it is made, and one slip covers one recipient, one tax-object code and one tax period (Pasal 7) ([PER-11/PJ/2025](https://jdih.kemenkeu.go.id/api/download/a94edee5-e585-4eeb-b9e7-a76f616c92fb/PER-11_PJ_2025.pdf), [JDIH record](https://jdih.kemenkeu.go.id/dok/per-11pj2025/files)).
- PER-12/PJ/2026 amended PER-11/PJ/2025 on 28 Sep 2026 and is in force ([JDIH – PER-12/PJ/2026](https://jdih.kemenkeu.go.id/dok/per-12pj2026)); the full-text link on that record served a different regulation, so **what it changed is [UNVERIFIED]** — check before building slip exports.
- Not confirmed at a primary source this session — **[UNVERIFIED]**: remittance and filing deadlines, tax-object codes for doctor fees, treatment of recipients without a valid NPWP/NIK, and whether Coretax offers an API/bulk import for BP21.
- *VAT*: "jasa pelayanan kesehatan medis" is a strategic taxable service **exempt** from VAT (PP 49/2022 Pasal 10(a)); it covers services of "dokter umum, dokter spesialis, dokter gigi, dan dokter gigi spesialis" and of facilities including "klinik kesehatan" (Pasal 11(2)–(4)) ([PP 49/2022](https://peraturan.go.id/files/Salinan+PP+Nomor+49+Tahun+2022.pdf)); in force, no amendment shown ([BPK – PP 49/2022](https://peraturan.bpk.go.id/Details/232943/pp-no-49-tahun-2022)). Retail sale of goods (oral-care products, medicines) is not named in Pasal 11 **[INFERENCE]** → consultant on VAT registration and invoicing for goods.

### e. Clinic revenue cycle in Indonesia

**Billing constructs seen in dental systems**: payments are allocated to procedures and providers, with unallocated patient income tracked separately ([Open Dental – Provider Payroll](https://opendental.com/manual/reportprovpayroll.html)); deposits become earned only when treatment is performed ([Dentally](https://help.dentally.com/en/articles/7866192-how-to-use-the-monthly-accounts-report-to-pay-associates)). Open Dental's payment-plan and unearned-income manual pages were not reachable at the URLs tried — instalment-plan mechanics in that product are **[UNVERIFIED]**.

**QRIS rules (Bank Indonesia)**

- QRIS is BI's national payment-QR standard, mandatory for all payment providers using QR; maximum **Rp10,000,000 per transaction**; modes: merchant-presented static and dynamic, customer-presented, remote (TTM: QR shared by image or invoice), Tuntas, TAP (NFC) and cross-border ([BI – QRIS](https://www.bi.go.id/id/fungsi-utama/sistem-pembayaran/ritel/kanal-layanan/qris/default.aspx)). Legal basis: PADG 21/18/PADG/2019 as last amended by PADG 3/2025 (19 Feb 2025) ([BI – PADG 3/2025](https://www.bi.go.id/id/publikasi/peraturan/Pages/PADG_032025.aspx)).
- MDR from **1 Oct 2026** (BI press release 28/159/DKom of 17 Aug 2026): micro merchants 0 % up to Rp500,000 and 0.3 % above; small/medium/large merchants 0 % up to Rp100,000 and 0.7 % above; education 0.6 % above Rp100,000; fuel stations 0.4 % above Rp100,000; BLU/PSO/G2P/P2G 0 %. MDR is borne by the merchant and "tidak boleh dibebankan kepada konsumen". No health-care category exists ([BI – MDR QRIS, updated 19 Oct 2026](https://www.bi.go.id/id/publikasi/ruang-media/cerita-bi/Pages/mdr-QRIS.aspx)). BI's main QRIS page still shows the earlier flat 0.7 % table ([BI – QRIS](https://www.bi.go.id/id/fungsi-utama/sistem-pembayaran/ritel/kanal-layanan/qris/default.aspx)).

**Gateways (official docs)**

| | Midtrans | Xendit | DOKU | Durianpay |
|---|---|---|---|---|
| QRIS | Dynamic QRIS; existing GoPay merchants get it automatically ([docs](https://docs.midtrans.com/docs/qris-payment-method-in-midtrans.md)) | Min Rp1, max Rp10,000,000; settlement T+1 business day; full, partial and multiple partial refunds; "Merchant Initiated Transaction" is marked "-" in the capability table, i.e. no recurring pull ([docs](https://docs.xendit.co/docs/qris.md)) | QRIS under SNAP direct API ([index](https://developers.doku.com/llms.txt)) | Rp1 – Rp10,000,000 ([docs](https://docs.durianpay.id/docs/payment-methods)) |
| Virtual accounts | BCA, BNI, Mandiri, Permata, BRI, CIMB, Danamon, BSI, SeaBank, Saqu ([pricing](https://midtrans.com/id/pricing)) | BCA, BNI, BRI, BSI, Mandiri, Permata and others ([index](https://docs.xendit.co/llms.txt)) | Per-bank VA pages; each sub-account gets a static BRI VA ([sub-account v2](https://developers.doku.com/wallet-as-a-service/sub-account/sub-account-v2.md)) | BCA, Mandiri, Permata, BNI, BRI, CIMB, BSI, Nobu; Rp10,000 – Rp10 billion, account-dependent ([docs](https://docs.durianpay.id/docs/payment-methods)) |
| Cards / instalments | Card instalment by `installment_term`; "online" (acquirer = issuer) and "offline" with BIN list; not for debit cards ([docs](https://docs.midtrans.com/reference/card-feature-installment.md)) | Cards and an Installment guide exist ([index](https://docs.xendit.co/llms.txt)); content not read | Cards, tokenization ([index](https://developers.doku.com/llms.txt)); no instalment page in index | Visa/Mastercard from Rp15,000; paylater via Indodana ([docs](https://docs.durianpay.id/docs/payment-methods)) |
| Payment links | Dashboard, CSV bulk or API; expiry date; maximum usage; optional customer-entered amount with min/max, suitable for down payments ([docs](https://docs.midtrans.com/docs/payment-link-overview.md)) | Payment links / payment sessions ([index](https://docs.xendit.co/llms.txt)) | FlexiBill sends invoices by email and/or WhatsApp with a payment link ([docs](https://developers.doku.com/flexibill/subscription-and-billing.md)) | Dashboard or API ([index](https://docs.durianpay.id/llms.txt)) |
| Recurring | Subscription API supports only `credit_card` and `gopay`; schedule by interval, unit, max_interval, start_time; failed charges retried 3 times hourly by default; IDR only ([docs](https://docs.midtrans.com/reference/create-subscription.md)) | Plans with weekly/monthly/yearly cycles, retry settings, fallback payment link, deactivation after failures; needs a channel supporting merchant-initiated transactions ([docs](https://docs.xendit.co/docs/how-subscriptions-work.md)) | FlexiBill: one-time or `RECURRING` bills and plan-based subscriptions; `PAID` webhook ([docs](https://developers.doku.com/flexibill/subscription-and-billing.md)) | Subscription API reference only ([index](https://docs.durianpay.id/llms.txt)) |
| Split / payout | No split-payment page in the docs index; partner/multi-outlet and payout pages only ([index](https://docs.midtrans.com/llms.txt)) | xenPlatform split rules: flat or percent per route, multiple destinations, applied after settlement on amount net of fees; a 100 % rule fails; refunds do not auto-return splits ([docs](https://docs.xendit.co/docs/split-payments)); payouts to banks and e-wallets ([docs](https://docs.xendit.co/docs/send-payouts-overview.md)) | Split settlement to up to 8 bank accounts per transaction, `PERCENTAGE` or `FIX`, computed on net settlement, aggregator merchants only ([docs](https://developers.doku.com/accept-payments/finance-and-settlement/split-settlement)); sub-accounts with split rules and bank payout via BI-FAST/online ([docs](https://developers.doku.com/wallet-as-a-service/sub-account/sub-account-v2.md)) | Batch disbursement API with optional approve step, bank-account validation webhook, item statuses done/invalid/failed ([docs](https://docs.durianpay.id/docs/disbursement-apis-implementation.md)) |
| Webhooks | POST to notification URL; verify `SHA512(order_id+status_code+gross_amount+ServerKey)`; statuses incl. settlement, pending, expire, refund, partial_refund; retries up to 5 times over about 2–210 minutes; duplicates and out-of-order delivery possible ([docs](https://docs.midtrans.com/docs/https-notification-webhooks.md)) | `x-callback-token` header; up to 6 retries with exponential backoff; de-duplicate on payment IDs; may arrive out of order ([docs](https://docs.xendit.co/docs/handling-webhooks.md)) | Notification, retry and sample pages listed ([index](https://developers.doku.com/llms.txt)) | HMAC-SHA256 signature over `disbursement_id|amount` ([docs](https://docs.durianpay.id/docs/disbursement-apis-implementation.md)) |
| Published fees | VA Rp4,000; QRIS 0.7 %; GoPay 2 %; ShopeePay 2 %; DANA/OVO 1.5 %; cards 2.9 % + Rp2,000; minimarket Rp5,000; Akulaku 1.7 %; Kredivo 2 %; fees exclude VAT except QRIS, GoPay, ShopeePay ([pricing](https://midtrans.com/id/pricing)) | Fee types: payment-method fee kept on refund; processing fee on every card attempt and on successful local payments, payouts and refunds; minimum monthly fee USD 50 or local equivalent after the first month ([docs](https://docs.xendit.co/docs/transaction-fees.md)); rate card page blocked — **[UNVERIFIED]** | Not retrieved — **[UNVERIFIED]** | No fee page in docs index — **[UNVERIFIED]** |

- The Midtrans QRIS price (0.7 %) predates or ignores BI's October 2026 zero-rate tiers; gateway price ≠ BI MDR, so the applicable fee must be confirmed per contract **[INFERENCE]**.
- Orthodontic or implant invoices above Rp10 million cannot be settled in one QRIS payment (BI limit above), so high-value plans need VA, card instalment or scheduled payment links **[INFERENCE]**.

**Private insurance / TPA (brief)**: Allianz's FAQ for its AdMedika network describes the cashless flow — patient presents insurer card and ID at a network hospital or clinic; the provider verifies membership and benefit; an initial medical report goes to the TPA so a **Guarantee Letter** is issued to the provider; patient signs the service form; any excess is paid by the patient before leaving, and later excess is billed within 14 calendar days ([Allianz – FAQ Prosedur Cashless](https://www.allianz.co.id/content/dam/onemarketing/azli/wwwallianzcoid/layanan/klaim/klaim-asuransi-kesehatan/V1-5-FAQ-Prosedur-Cashless.pdf)). That document concerns inpatient cover; dental-specific benefit limits, TPA portals/EDC steps and corporate direct-billing terms were not found at first-party sources — **[UNVERIFIED]**.

### f. Accounting: build a ledger vs integrate

| Product | What its official material exposes |
|---|---|
| Accurate Online | OAuth 2.0 with app registration ([overview](https://accurate.id/api-integration/)); every call needs `Authorization: Bearer` and data calls an `X-Session-ID` obtained from `open-db.do`, on a per-database host ([examples](https://accurate.id/api-integration/api-example)). OpenAPI schema groups include sales-invoice, sales-receipt, sales-return, customer; purchase-order, purchase-invoice, purchase-payment, receive-item, vendor; item, unit, warehouse, item-adjustment, **item-transfer**, **stock-opname-order/result**; **bill-of-material**; glaccount, **journal-voucher**, other-deposit, other-payment, expense, tax; **fixed-asset**; branch, department, project; webhook history/renew ([OpenAPI JSON](https://account.accurate.id/open-api/json.do)). |
| Zahir Online | States an API for contacts, accounts, departments, general ledger, cash in/out, projects, warehouses, products and categories, tax, units, purchasing ([Zahir Online API](https://www.zahironline.com/?p=7846)); the linked documentation page rendered only a placeholder ([zahironline.com/api](https://www.zahironline.com/api/)) — endpoint detail **[UNVERIFIED]**. |
| Xero | Official OpenAPI spec: Accounting API v20.1.0 at `https://api.xero.com/api.xro/2.0`, paths incl. /Accounts, /BankTransactions, /BankTransfers, /BatchPayments, OAuth scopes such as `accounting.transactions` ([Xero-OpenAPI](https://raw.githubusercontent.com/XeroAPI/Xero-OpenAPI/master/xero_accounting.yaml)); only the first part of the file was read. Invoices, manual journals, tracking categories and Indonesian tax localisation — **[UNVERIFIED]** (docs site JS-only). |
| Jurnal by Mekari | Documentation unreachable in this session — **[UNVERIFIED]** entirely. |

- Standards (IAI): three tiers — SAK (tier 1), SAK Entitas Privat (tier 2) and SAK EMKM (tier 3); SAK EP replaces SAK ETAP, effective **1 January 2025** with early adoption allowed, and adopts IFRS for SMEs (2015); it applies to entities without public accountability that issue general-purpose statements; SAK EMKM's "small and medium" follows the quantitative ranges of UU 20/2008 ([IAI – Sosialisasi SAK Entitas Privat](https://web.iaiglobal.or.id/assets/files/file_berita/Materi%20Presentasi_Sosialisasi%20SAK%20Entitas%20Privat.pdf)). SAK EMKM's own text was not opened — its simplifications are **[UNVERIFIED]**.
- **[INFERENCE]** Build a *sub-ledger*, not a general ledger: an immutable, double-entry posting journal for the domain events only this platform knows (invoice, payment, deposit, refund, fee accrual, PPh 21 payable, stock receipt, consumption, write-off, depreciation), plus a mapped chart of accounts, then export journals. Accurate's API already accepts journal vouchers, items, transfers, stock opname and fixed assets, so a clinic with an accountant keeps statutory books there; a solo dentist gets management reports without any accounting product. A full GL (period close, tax reports, bank reconciliation, statutory statements under SAK EP/EMKM) duplicates products that already exist and is where regulatory drift is highest.

### g. KPIs and ROI

All benchmark values below are US sources (ADA); **no Indonesian benchmark from PDGI, Kemenkes or an academic source was located — [UNVERIFIED] for local use**. Treat them as default targets the tenant can edit.

| KPI | Definition | Benchmark and source |
|---|---|---|
| Production (gross / adjusted) | Fees for completed procedures; adjusted = amount the office may actually collect | Average gross billings per general dentist $965,660, specialist $1,213,040 (2025 Survey of Dental Practice) ([ADA HPI](https://www.ada.org/resources/research/health-policy-institute/dental-practice-research)) |
| Collection rate | (Collections ÷ adjusted production) × 100 ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) | 98 % of billable/adjusted production ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)) |
| Overhead | Expenses ÷ collections | ≤ 63 % of total income ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)); "averages around 62 %", Levin Group target 59 % ([ADA News](https://adanews.ada.org/new-dentist/2022/march/ask-the-expert-practical-strategies-to-reduce-dental-practice-expenses/)) |
| Dental consumables % | Dental supplies ÷ collections | ≤ 5–6 %; office supplies ≤ 2 %; items over $500 count as equipment ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) |
| Lab cost % | Lab bills ÷ collections | 6–8 % general practice (excl. CAD/CAM and ortho); about 4 % may signal low case acceptance; up to 11 % in cosmetic-heavy practices ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) |
| Staff cost % | Payroll + benefits ÷ collections | 25–28 % in an efficient practice, lenders want under 30 %; assistants 7–8 %, clerical 7–8 %, hygiene 9–10 % ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) |
| Occupancy % | Rent + utilities ÷ collections | 8–10 % ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) |
| Case acceptance | Accepted ÷ presented treatment plans | 75–80 % ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)) |
| Cancellation / no-show rate | (Cancellations + no-shows) ÷ scheduled appointments | ≤ 5 % ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)). Published academic rates (about 14 % in one academic dental clinic; about 23 % across medical specialties) were seen only in search snippets — **[UNVERIFIED]** |
| New patients | Count per month | Should grow 10–15 % a year ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)) |
| Recall | Share of recall patients active / with next visit booked | 85 % active; 90 % scheduled ([ADA – KPIs](https://www.ada.org/resources/practice/practice-management/measuring-success20160428t155202)) |
| AR aging | Receivables by 0–30/31–60/61–90/90+ days | Warning signs: receivables above two months of production; high balance over 90 days ([ADA](https://www.ada.org/resources/careers/career-planning/how-to-purchase-with-confidence)) |
| Busyness / capacity | Self-reported; days a new patient waits | 32 % of US dentists "not busy enough" in Q1 2026; new-patient wait 12.4 business days; dental equipment and supply prices +6 % in the 12 months to Feb 2026 ([ADA HPI – Q1 2026](https://www.ada.org/-/media/project/ada-organization/ada/ada-org/files/resources/research/hpi/state_us_dental_economy_q12026.pdf)) |
| Chair utilization | Booked chair-hours ÷ available chair-hours **[INFERENCE]** | No primary benchmark found — **[UNVERIFIED]** |

Derived metrics **[INFERENCE]** (formulas this platform can compute because it holds procedures, BOM consumption, lab bills, fee shares and assets):
- *Procedure contribution margin* = price charged − discount − material cost consumed − lab fee − dentist fee share − payment fee.
- *Per-dentist profitability* = Σ contribution margin of their procedures − allocated chair-time cost; chair-time cost = (rent + staff + depreciation of the operatory's assets) ÷ available chair-hours.
- *Asset payback (months)* = acquisition cost ÷ average monthly contribution margin of procedures that require the asset, net of maintenance; *asset ROI* = (cumulative margin − maintenance − depreciation) ÷ acquisition cost; *utilization* = procedures or hours used ÷ available; *downtime* = hours out of service ÷ scheduled hours.
- *Inventory*: days on hand = stock value ÷ average daily consumption; expiry write-off % = expired value ÷ purchases; variance = system quantity − counted quantity.

### h. Proposed finance feature set — all **[INFERENCE]**, ordered by value

1. **Invoice, payment and allocation core** — every payment allocated to procedure lines and performers, with deposits held as unearned until treatment (Dentally and Open Dental both make allocation the precondition for correct pay, section d). Everything below depends on it.
2. **Partner-dentist fee engine with PPh 21** — rule versions per dentist (base, deductions, schedule, guarantee), period statements with drill-down, and withholding computed on the patient-paid doctor fee at 50 % × Pasal 17 rates, non-cumulative, with BP21-ready data (section d). Highest owner pain; the tax base differs from the payout, which spreadsheets get wrong.
3. **Payment collection via gateway** — payment links over WhatsApp, VA and QRIS with webhook reconciliation (signature check, idempotency, out-of-order tolerance per Midtrans/Xendit docs), fees captured per payment; respect the Rp10 million QRIS cap (section e).
4. **Instalment and deposit plans** — schedule, reminders, per-instalment payment link; avoid dependence on card-only subscription APIs (Midtrans subscription = card and GoPay only) (section e).
5. **Receivables and claims** — AR aging per patient and per payer; guarantee-letter tracking with excess billed to the patient (Allianz flow), corporate invoices (section e).
6. **Cost per procedure** — procedure BOM auto-deduction at completion (MedicaSimple/Pabau pattern) plus lab bills, feeding contribution margin (sections a, g).
7. **Expense and supplier payables** — PO → receipt → invoice match (Hybrent/CureMint pattern), due dates, spend vs budget by category against ADA ratio targets (sections a, g).
8. **Asset finance** — register with acquisition cost, fiscal group selectable per PMK 72/2023 and separate management life, monthly depreciation, maintenance cost, payback and utilization per asset (sections c, g).
9. **Owner dashboard** — production, collections, collection rate, overhead mix, per-dentist and per-procedure margin, AR aging, no-show, new patients; role-scoped so a partner dentist sees only own statements (section g).
10. **Daily cash close** — per cashier/shift: cash, QRIS, VA, card totals vs gateway settlement reports; variance log (settlement timing, e.g. Xendit QRIS T+1) (section e).
11. **Accounting export** — journal export and Accurate Online sync (journal-voucher, item, fixed-asset APIs); chart-of-accounts mapping; no in-house GL (section f).
12. **Refund and correction workflow** — approvals, reversal postings in the current period, automatic fee clawback line in the next dentist statement; gateway refund where the channel supports it (Xendit QRIS refunds; Xendit split refunds do not auto-reverse) (sections d, e).
13. **Split settlement to partner dentists** (later, optional) — Xendit split rules or DOKU sub-accounts could pay shares at settlement, but withholding tax must be deducted first and each dentist must be onboarded as a sub-account; keep clinic-side payout as the default (sections d, e).

### Implications for the workspace platform

All items **[INFERENCE]**, each tied to cited facts above.

- **One stock ledger, three write paths**: receipts (PO receiving), consumption (procedure BOM at treatment completion, editable before posting as in Pabau), adjustments (count, waste, expiry, transfer). Every movement carries location, lot, expiry, unit cost, actor and reason — this single table feeds FEFO picking, recall, cost per procedure and the accounting export (sections a, b).
- **Item master fields**: name, category, base unit and purchase unit with conversion (Open Dental relies on free-text pack units — avoid that), par/reorder point and reorder quantity per location, preferred supplier and last prices, flags (medicine / device / implant / high-alert / cold-chain), `izin_edar_no` (prefix AKL/AKD + 11 digits, validated by pattern only) with expiry of the licence as optional data, GTIN, and tracking mode (none / lot+expiry / serial).
- **Scanner input**: parse GS1 element strings `(01)(17)(10)(21)(11)` from DataMatrix/Code-128 using the FDA-published field sizes; accept manual lot/expiry entry because Indonesian UDI obligation is unconfirmed and many local consumables will lack GS1 codes. Phone camera first; reorder-tag (kanban) mode as the low-effort option for small practices (Method's pattern).
- **Do not automate izin edar lookup in v1**: infoalkes is a public search site with no documented API in the pages read; store the number and a manual "verified on" date.
- **Medicines sub-module**: lot and expiry mandatory, FEFO suggestion, near-expiry flag at a configurable 3–6 months (Permenkes 34/2021 annex), temperature log for the medicine fridge as an asset checklist, expired stock moved to a quarantine location and written off with a destruction record (Permenkes 5/2026 Pasal 89–90). Apoteker-signed order workflow only for tenants that run an Instalasi Farmasi.
- **Asset register = equipment + compliance calendar**: per asset — category, serial, location/chair, supplier, acquisition cost and date, warranty end, izin edar no., maintenance plan and service history, calibration due date and certificate, downtime log, disposal (date, proceeds). Subtypes: X-ray unit (BAPETEN licence, compliance test, dosimetry, tube log, yearly B@LIS report) and sterilizer (cycle log with mechanical/chemical/biological results, pack-to-patient link). ASPAK export and 30 Jun / 31 Dec reminders.
- **Regulatory values as tenant-editable configuration with a "source and date" note**, not constants: calibration interval, PPh 21 brackets and 50 % base, depreciation groups, QRIS limit and MDR, gateway fees. Three of the rules researched changed in 2026 alone (Permenkes 5/2026 in May; QRIS MDR in October; PER-12/PJ/2026 in September).
- **Two depreciation books**: fiscal (PMK 72/2023 group and method chosen by the tenant's tax adviser) and management (real expected life) — ROI and payback use the management book.
- **Fee engine outputs two numbers per dentist per period**: payout owed (by the contract rule) and PPh 21 base (patient-paid doctor fee). Invoice lines therefore need a "jasa dokter" component separable from materials, lab and facility charges, or a tenant-level policy on how a bundled price maps to it (consultant-defined).
- **Period locking**: once a dentist statement or month is closed, later refunds and corrections post as new lines in the open period (Dentally fixes month figures after month end; CareStack allows one active payroll per provider and period, replaced only by void-and-regenerate).
- **Gateway abstraction**: a provider interface with create-payment-link / VA / QRIS, webhook verify, refund, settlement report; start with one provider with published pricing and payment-link API (Midtrans has both documented); keep Xendit/DOKU adapters possible for split settlement later. Money never transits the platform's own account in v1 — each tenant connects its own merchant account — which avoids payment-service licensing questions (consultant).
- **Role mapping**: owner/superadmin — all finance; operations admin — invoices, payments, purchasing, stock; dental nurse — consumption confirmation, sterilizer log, counts; partner dentist — own production, fee statement and withholding slips only; viewer — dashboards without patient identifiers.
- **Benchmarks shipped as editable defaults** labelled "US ADA reference", since no Indonesian benchmark was found.

### Open questions

1. Which regulation now sets the testing/calibration duty and interval for clinic equipment after Permenkes 54/2015 was revoked (PP 28/2024 article text; any 2026 implementing rule)? Does it bind klinik pratama and solo practices?
2. Does Permenkes 5/2026 (or its implementing rules, due within a year of 4 May 2026) keep the AKL/AKD format, risk classes, 5-year validity and label elements of Permenkes 62/2017? Is there any Indonesian UDI/traceability mandate planned?
3. Must a dental clinic that stocks anesthetics and antibiotics for in-chair use only run an Instalasi Farmasi with an Apoteker, and what stock records (kartu stok, narcotic/psychotropic reporting) apply to it?
4. PPh 21: does "dokter" in PMK 168/2023 cover dokter gigi in DJP practice; how is "jasa dokter yang dibayar oleh pasien" determined when the clinic charges a bundled price; what are the tax-object code, deadlines and no-NPWP treatment; what did PER-12/PJ/2026 change; is there a Coretax import/API for BP21? (tax consultant)
5. How do target clinics actually contract partner dentists today — base (gross tindakan vs net of bahan/lab), typical percentages by procedure type, guarantees, who bears discounts and gateway fees? (owner's existing data and interviews; not found in public primary sources)
6. Fiscal group for dental units, autoclaves, compressors, X-ray/CBCT under PMK 72/2023 — Kelompok 1 "special tools" vs default Kelompok 3 vs a DJP determination? (tax consultant)
7. VAT position for goods sold by the clinic and for mixed invoices; PKP registration threshold relevance. (tax consultant)
8. BAPETEN: licence validity period, Petugas Proteksi Radiasi and compliance-test requirements for intraoral, panoramic and CBCT units under Peraturan BAPETEN 5/2025; is Peraturan BAPETEN 4/2020 amended?
9. ASPAK: current regulation status and whether an import/API exists for bulk device data.
10. Which gateway do target clinics already hold accounts with; actual contracted fees (Xendit, DOKU, Durianpay rate cards not verified); whether the platform may orchestrate payouts to dentists without holding funds.
11. Which TPAs/insurers matter for dental in the target cities, and do they expose provider portals or only EDC/phone approval?
12. Jurnal by Mekari, Zahir and Xero API capabilities (not verifiable this session) — which accounting product do target clinics use, if any?
13. Indonesian KPI benchmarks (consumables %, lab %, no-show, chair utilization) — none found; consider deriving anonymised cross-tenant benchmarks once data exists.


---

## 4. Indonesian regulation and health-data interoperability

> Scope and method. Research date 2026-11-25. Every claim is cited to a page opened in this session; article numbers are *Pasal* (Ps.). Regulation texts were read from the official PDFs (peraturan.go.id / peraturan.bpk.go.id / keslan.kemkes.go.id); the BPK copies of the large laws are scans with an OCR text layer, so quoted wording may carry OCR noise. SATUSEHAT pages were read through a fetch-and-summarise tool, so element-level details (codes, version dates) must be re-checked on the live page before implementation. **[UNVERIFIED]** = not confirmed at a primary source; **[INFERENCE]** = my reasoning. This is design input, not legal advice — items marked **(lawyer)** need confirmation by an Indonesian health-law / privacy lawyer.
>
> Sources I could not read: JDIH Kemenkes PDF of SE HK.02.01/MENKES/1030/2023 (HTTP 404); `tte.komdigi.go.id` PSrE list (403); the SATUSEHAT "Akses Token" and "TTE" pages (redirected to index); the 2014 Kemenkes dental-record guide (only a non-official copy exists, 403); PP 33/2026 (not present in the BPK database search on this date).

### a. Electronic medical records — Permenkes 24/2022 and its enforcement instruments

**Status.** Permenkes 24/2022 *tentang Rekam Medis* was enacted and promulgated 31 Aug 2022 (BN 2022/829), is listed "Berlaku" with no amending/revoking instrument, and revoked Permenkes 269/2008 ([BPK record](https://peraturan.bpk.go.id/Details/245544)). Its statutory parents (UU 29/2004, UU 36/2014) were revoked by UU 17/2023 ([BPK record UU 17/2023](https://peraturan.bpk.go.id/Details/258028/uu-no-17-tahun-2023)); the electronic-record duty is now restated at PP level, and PP 28/2024 Ps. 787 delegates further detail to a ministerial regulation ([PP 28/2024 Ps. 777–787](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)). No replacement Permenkes was found in this session **[UNVERIFIED — absence not proven; (lawyer)]**.

| Topic | Rule | Source |
|---|---|---|
| Who must run RME | Every *Fasilitas Pelayanan Kesehatan*; the list expressly includes **tempat praktik mandiri dokter/dokter gigi**, puskesmas, **klinik**, **rumah sakit**, apotek, lab, balai; also facilities providing telemedicine | [Permenkes 24/2022 Ps. 3–4](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Deadline | All facilities "paling lambat 31 Desember 2023" | [Ps. 45](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| PP-level duty | RME "wajib diselenggarakan secara elektronik"; paper only while a technical obstacle lasts; RME must be integrated with the national health information system (SIKN), incl. telemedicine providers | [PP 28/2024 Ps. 779, 781](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf) |
| Who may supply the system | Kemenkes, the facility itself, or a *Penyelenggara Sistem Elektronik* (PSE) "melalui kerja sama"; that PSE **must be registered as a health-sector PSE** at the communications ministry | [Ps. 9(1),(3)](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Standards | System must be compatible/interoperable per Kemenkes standards and follow Kemenkes **variables and metadata** | [Ps. 10–11](https://www.peraturan.go.id/files/bn829-2022.pdf); [PP 28/2024 Ps. 780(d)–(e)](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf) |
| **System registration at Kemenkes** | Facility *or PSE* "wajib melakukan registrasi Sistem Elektronik" at Kemenkes, attaching: system name, documentation, features, **data storage location**, variables/metadata, list of user facilities; changes must be reported | [Ps. 12](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Minimum activities | Patient registration, distribution, clinical entry, processing (coding/reporting/analysis), claim data entry, storage, quality assurance, transfer of contents; in a solo practice these are the dentist's responsibility | [Ps. 13](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Registration data | Identity: at least MR number, name, **NIK**; social data: religion, occupation, education, marital status | [Ps. 14](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Clinical entry | Complete, clear, made after service, carrying **name, time and signature** of the health worker; sequential; corrections allowed | [Ps. 16](https://www.peraturan.go.id/files/bn829-2022.pdf); same at statute level: [UU 17/2023 Ps. 296(3)–(4)](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf) |
| Coding | Clinical classification codes per latest ICD | [Ps. 18(2)](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Minimum contents | Patient identity; physical and supporting exam results; diagnosis, treatment, follow-up plan; name and signature of the provider. PP adds that the record includes the **consent to treatment** | [Ps. 26(6)](https://www.peraturan.go.id/files/bn829-2022.pdf); [PP 28/2024 Ps. 778(3)](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf) |
| Ownership | Record *document* belongs to the facility; *contents* belong to the patient; patient has a right of access | [Ps. 25–26](https://www.peraturan.go.id/files/bn829-2022.pdf); [UU 17/2023 Ps. 297](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf) |
| Storage media | Server, **certified** cloud, or other certified digital media; **backup mandatory**, in a different location from the facility, periodic, documented in SOP | [Ps. 20](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Outsourced storage | Allowed via a PSE that has **storage facilities in Indonesia**; PSE needs a **recommendation from the Kemenkes data unit**; PSE may not open, take, manipulate, damage or exploit the data; this goes into an **integrity pact / NDA** attached to the contract; facility must keep **unrestricted access** | [Ps. 22](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Interoperate with Kemenkes | Stored RME "harus terhubung/terinteroperabilitas" with the Kemenkes platform; referral transfer goes through that platform; facility must open access to all RME contents to Kemenkes | [Ps. 21, 24, 28](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Security principles | Confidentiality, integrity, availability | [Ps. 29](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Access rights | Granted by the facility head as SOP policy; three rights: input, correction, view. **Correction only within 2×24 h of input**; later administrative corrections need approval of the records officer and/or facility head; viewing is for internal staff for service/administration | [Ps. 30](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| E-signature | RME "dapat dilengkapi" with electronic signature as verification/authentication tool (optional wording) | [Ps. 31](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Confidentiality | Binds everyone involved, incl. "pihak lain yang memiliki akses", even after the patient's death | [Ps. 32](https://www.peraturan.go.id/files/bn829-2022.pdf); [PP 28/2024 Ps. 785, 789](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf) |
| Release with consent | For care, at the patient's request, or for administration/insurance; request in writing or electronically; limited to need; insurance/admin release consent is taken **at registration** in writing or via the system; next-of-kin rules for incapable patients | [Ps. 33–34](https://www.peraturan.go.id/files/bn829-2022.pdf); [PP 28/2024 Ps. 783](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf) |
| Release without consent | Law enforcement, ethics/discipline, medical audit, outbreaks, education/research, danger to others; identity withheld except outbreak/safety cases; needs ministerial approval unless by court order | [Ps. 35–37](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Waiver | Patient/family who publicise the contents in mass media are deemed to waive secrecy; facility gets a right of reply | [Ps. 38](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| **Retention** | RME data kept **at least 25 years from the last visit**; may be kept longer if still used; destruction per law | [Ps. 39](https://www.peraturan.go.id/files/bn829-2022.pdf) |
| Sanctions | Written warning and/or recommendation to revoke accreditation, for breach of Ps. 3 | [Ps. 42](https://www.peraturan.go.id/files/bn829-2022.pdf) |

**Later decrees / circulars**
- **KMK HK.01.07/MENKES/1423/2022** (14 Sep 2022) sets the *Pedoman Variabel dan Meta Data*; diktum KEDUA makes it a mandatory reference for facilities, health workers and health-sector PSEs. Data sets: IGD, **Rawat Jalan**, Rawat Inap, Laboratorium, Apotek. Diagnoses reference ICD-10; procedures ICD-9-CM; NIK is 16 digits (16×"9" when absent); consent variable references Permenkes 290/2008 ([KMK 1423/2022](https://keslan.kemkes.go.id/unduhan/fileunduhan_1664868356_130610.pdf)).
- **SE Dirjen Yankes HK.02.02/D/7093/2023**: lists the same facility types (incl. solo dentists); RME systems "wajib mengikuti standar": KMK 1423/2022, ICD-9-CM, ICD-10, SNOMED CT, LOINC, KFA, HL7 FHIR; must be able to interoperate with SATUSEHAT; trial and registration via the SATUSEHAT portal; non-compliance after 31 Dec 2023 "akan berpengaruh terhadap hasil penilaian akreditasi" ([SE 7093/2023](https://keslan.kemkes.go.id/unduhan/fileunduhan_1683252264_905012.pdf)).
- **SE Menkes HK.02.01/MENKES/1030/2023** (RME and administrative sanctions): a Kemenkes article confirms the sanctions are written warning and recommendation to adjust — up to revoke — accreditation status, imposed through the Dirjen ([Kemenkes Ditjen Yankes, 7 Mar 2024](https://keslan.kemkes.go.id/read/1704/desk-sistem-informasi-rumah-sakit-rme-dan-satusehat)). Exact trigger dates in the SE (e.g. 31 Mar / 31 Jul 2024 thresholds) **[UNVERIFIED — primary PDF returned 404]**.
- SATUSEHAT's KYC page restates that under "PMK No. 24/2022" facilities must send medical records to SATUSEHAT ([SATUSEHAT KYC](https://satusehat.kemkes.go.id/platform/docs/id/kyc/)).

### b. Dental-specific record standards

- **KMK 1423/2022 has no dental data set**: the word "odontogram" does not occur; teeth appear only as a free-text physical-exam item ("Gigi geligi") and a radiology exam type ([KMK 1423/2022](https://keslan.kemkes.go.id/unduhan/fileunduhan_1664868356_130610.pdf)).
- **The operative dental data standard today is Kemenkes' SATUSEHAT "Modul Penerapan Gigi"** (v1.5, 7 Aug 2024; history from v1.0, 20 Jan 2024), scoped to *Rawat Jalan Gigi*; it cites Permenkes 89/2015 (oral health efforts) and Permenkes 30/2022 (OHI-S) as basis ([SATUSEHAT Gigi](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rawat-jalan-gigi/)).
- **Tooth numbering: FDI two-digit** (11–48 permanent, 51–85 primary), carried in `Observation.bodySite` as SNOMED CT "Structure of … tooth" concepts mapped to the FDI number ([SATUSEHAT Gigi](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rawat-jalan-gigi/); [Lampiran Terminologi Gigi, Lamp. 1](https://satusehat.kemkes.go.id/platform/docs/id/terminology/lampiran-terminologi/rawat-jalan-gigi/)).
- **Odontogram representation**: one `Observation` (category exam) per tooth finding with components — surface (LOINC 32889-8; values M/O/D/V/L), tooth condition (SNOMED 278544002), restoration material, restoration, prosthesis material, prosthesis; plus occlusion, torus palatinus/mandibularis, palate, diastema, anomalous teeth, and D/M/F counts ([SATUSEHAT Gigi](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rawat-jalan-gigi/)).
- **Symbol set** (three-letter abbreviations bound to SNOMED CT or Kemenkes local `OV…` codes at `http://terminology.kemkes.go.id/CodeSystem/clinical-term`): conditions `sou, non, une, pre, imv, ano, dia, att, abr, car, cfr, nvt, rrx, mis` …; materials `amf, gif, cof, fis, acr, rfx`; restorations `fmc, poc, mpc, gmc, rct, ipx, meb, pob, pon, abu, inl, onl`; prostheses `prd, fld, cla` … The published list has duplicate abbreviations (`non`, `rrx`, `gif`) — key on the code, not the abbreviation ([Lampiran Terminologi Gigi](https://satusehat.kemkes.go.id/platform/docs/id/terminology/lampiran-terminologi/rawat-jalan-gigi/)).
- **OHI-S**: index teeth 16, 11, 26, 36, 31, 46; debris/calculus scores with local codes ([SATUSEHAT Gigi](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rawat-jalan-gigi/)). Dental radiology LOINC list is a separate downloadable sheet ([Radiologi Gigi](https://satusehat.kemkes.go.id/platform/docs/id/terminology/loinc/radiologi/lampiran-gigi/)).
- **Legacy national standard**: the *Standar Nasional Rekam Medik Kedokteran Gigi* (mid-2000s) and Kemenkes' *Panduan Rekam Medik Kedokteran Gigi* (Direktorat Bina Upaya Kesehatan Dasar, 2014; record = identity + odontogram + treatment table + attachments; FDI numbering) are widely cited but no official copy could be opened **[UNVERIFIED]**. That the SATUSEHAT abbreviations reproduce that guide's symbols is **[INFERENCE]**.
- **PDGI**: the ethics code has an article on patient records (Ps. 19 "Rekam Medik Pasien") and on secrecy (Ps. 18) ([KODEKGI, PB PDGI 2017–2020 edition, copy hosted by FKG Unair](https://fkg.unair.ac.id/en/wp-content/uploads/2021/06/Buku-Kode-Etik-Kedokteran-Gigi-2020.pdf)). Current ownership split of the odontogram standard between Kemenkes, the Konsil and PDGI is not stated in any primary text read **[UNVERIFIED]**; de facto the binding interoperability definition is Kemenkes' **[INFERENCE]**.

### c. SATUSEHAT platform (official developer documentation)

**Versions.** Docs portal is **v7.23** (release note dated 2 Apr 2026); guides and terminology are revised quarterly (Mar/Jun/Sep/Dec) ([What's new](https://satusehat.kemkes.go.id/platform/docs/id/whats-new/); [v7 notes](https://satusehat.kemkes.go.id/platform/docs/id/whats-new/7/)). *Resume Medis Rawat Jalan* is at v7.0 (10 Jan 2026) ([Rawat Jalan](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rme-rawat-jalan/)); *Standar Terminologi* at v12.0 (31 Oct 2026) ([Standar Terminologi](https://satusehat.kemkes.go.id/platform/docs/id/terminology/standar-terminologi/)). API paths: FHIR R4 at `…/fhir-r4/v1` ([Patient API](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/onboardings/apis/patient/)), OAuth at `…/oauth2/v1` ([Autentikasi](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/authentication/)). No dated compliance deadline appears in the docs read.

**Onboarding — vendor ("Sistem RME")** ([Registrasi Sistem RME](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-vendor/); [Data Sistem RME](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-vendor/data-rme-vendor/))
1. Preconditions: **PSE registration number** from Kominfo/Komdigi ("WAJIB"; private entities via OSS); **SNOMED CT affiliate** application number; **sandbox test evidence** — submissions up to `Encounter` and `Condition` returning 200/201, with screenshots, a screen recording e-mailed to the helpdesk, and monitoring-dashboard screenshots; administrative PDFs (authorisation letter, system documentation, signed-and-stamped variables/metadata statement, list of client facilities or a "not yet partnered" statement).
2. Portal form fields include system name, features, facility types served, **data storage location (must be in Indonesia for SATUSEHAT interoperability data)**, on-prem vs cloud, **backup yes/no and backup location (also in Indonesia)**.
3. **Security survey** (all questions mandatory; stated purpose: confirm the system meets personal-data-protection requirements) ([Survei Keamanan](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-vendor/survei-vendor/)); then Kemenkes verification with a trackable status. Stage vocabulary: *Terdaftar → Terkoneksi → Terintegrasi* ([v7.18 note](https://satusehat.kemkes.go.id/platform/docs/id/whats-new/7/)).

**Onboarding — client facility** ([Registrasi Fasyankes](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-institution/))
- Facility must update its "RME system" data in the Kemenkes facility registry: **DFO** (clinics, puskesmas, labs), **REGFASYANKES** (TPMD/TPMDG = solo doctors/dentists), **RS Online** (hospitals), and pick a system that is already verified on SATUSEHAT.
- The facility's account is created from its SATUSEHAT contact form (activation link to the "email integrasi"); only vendors self-register on the portal.
- **Production credentials** (Organization ID, Client ID, Client Secret) appear about 3 working days after a correct update ([Registrasi Fasyankes](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-institution/)); a verified vendor account can see the production codes of facilities that selected its system ([Kode Akses Production](https://satusehat.kemkes.go.id/platform/docs/id/api-code/access-production/)).

**Auth** ([Autentikasi](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/authentication/))
- OAuth 2 `client_credentials`, server-to-server. Sandbox `https://api-satusehat-stg.dto.kemkes.go.id/oauth2/v1`; production `https://api-satusehat.kemkes.go.id/oauth2/v1`.
- **One client ID/secret works for exactly one Organization ID** (otherwise a business-rule error); credentials are confidential and non-transferable. Token lifetime and rate limits: page not readable **[UNVERIFIED]**.

**Identity and master data**
- **Patient**: look up the IHS number in the Master Patient Index — `GET /Patient?identifier=https://fhir.kemkes.go.id/id/nik|{nik}`, or name+birthdate+gender, or mother's NIK for newborns; `POST /Patient` when not found; store the returned id. Identifier systems: `…/id/nik`, `…/id/nik-ibu`, `…/id/ihs-number`, also passport and KK ([Patient API](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/onboardings/apis/patient/)). The IHS number may be stored in the partner's system; MPI is the stated single source of truth; Dukcapil now only validates NIKs ([MPI](https://satusehat.kemkes.go.id/platform/docs/id/master-data/master-patient-index/preliminary/)). Missing patient NIK → POST to MPI; missing practitioner NIK → e-mail helpdesk ([v7.20 note](https://satusehat.kemkes.go.id/platform/docs/id/whats-new/7/)).
- **Practitioner**: read-only search by NIK or name+birthdate+gender to obtain the practitioner IHS id; no POST documented; **STR/SIP data are not mentioned** in the API page ([Practitioner API](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/onboardings/apis/practitioner/)).
- **Organization / Location**: the facility registers its own structure (prerequisite steps) ([Rawat Jalan](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rme-rawat-jalan/)).
- **KYC**: facility staff can verify a patient's SATUSEHAT Mobile profile once so the patient can see their PHR; integration uses a KYC library/URL generation ([KYC](https://satusehat.kemkes.go.id/platform/docs/id/kyc/)).

**Outpatient encounter — resources** ([Rawat Jalan](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rme-rawat-jalan/))
- Prerequisites: auth, Organization, Location, Practitioner IHS, Patient IHS.
- Sequence (29 steps) from `Encounter` (arrived → in-progress → finished) through `Condition`, `Observation`, `AllergyIntolerance`, `Procedure`, `ServiceRequest`, `Medication*`, `ClinicalImpression`, `CarePlan`, to `Composition` (resume). Mandatory elements are starred per resource; closing an Encounter as `finished` requires `Encounter.diagnosis`. Timestamps in UTC; dates before 3 Jun 2014 rejected. Radiology images go via a DICOM router to the national repository.
- **Minimum the vendor must prove** for verification is Encounter + Condition ([Registrasi Sistem RME](https://satusehat.kemkes.go.id/platform/docs/id/registration-guide/regis-vendor/)). Which additional resources are legally mandatory per visit is not stated in the pages read **[UNVERIFIED]**.

**Dental use case** ([SATUSEHAT Gigi](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rawat-jalan-gigi/)): 16 steps — Patient (GET) → Encounter → anamnesis (`Condition`, `AllergyIntolerance`, `MedicationStatement`, `Observation`) → physical exam + OHI-S (`Observation`) → **odontogram (`Observation`)** → supporting exams (`ServiceRequest`, `Specimen`, `ImagingStudy`, `DiagnosticReport`) → diagnosis (`Condition`) → **procedure and education (`Procedure`)** → pharmacy → prognosis (`ClinicalImpression`) → follow-up (`ServiceRequest`) → Encounter update (PUT). Dental procedure/diagnosis coding defers to the outpatient module.

**Terminologies**

| Terminology | Use | Availability / licensing | Source |
|---|---|---|---|
| ICD-10, ICD-9-CM (+ICD-O etc.) | Diagnoses, procedures | Published in the portal's terminology section | [docs navigation](https://satusehat.kemkes.go.id/platform/docs/id/bibliography/); [SE 7093/2023](https://keslan.kemkes.go.id/unduhan/fileunduhan_1683252264_905012.pdf) |
| KPTL | National procedure list, with an ICD-9-CM→KPTL mapping added in v7.21 (30 Nov 2025) | Portal | [v7 notes](https://satusehat.kemkes.go.id/platform/docs/id/whats-new/7/) |
| LOINC | Vitals, lab, radiology (incl. dental radiology sheet) | Portal appendices | [Rawat Jalan](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rme-rawat-jalan/) |
| SNOMED CT | Complaints, history, dental findings, teeth | Indonesia is a SNOMED International member; Kemenkes pays the fee; free for use within Indonesia. **Affiliate licence via MLDS required**: phase 1 facilities (the facility applies even if a third party runs the EMR), phase 3 vendors. Duties: annual usage report, manage sublicences; no redistribution of RF2 or translation without Kemenkes | [SNOMED CT Affiliate License](https://satusehat.kemkes.go.id/platform/docs/id/terminology/snomed-ct/affiliate-license/) |
| KFA | Medicines/devices (codes prefixed 93 product, 92 template) | REST API + browser | [Rawat Jalan](https://satusehat.kemkes.go.id/platform/docs/id/interoperability/rme-rawat-jalan/) |

**Consent.** The docs expose a FHIR `Consent` resource (mandatory elements incl. status, scope, category, policy URI, provision actor/data; `provision.type` permit/deny) ([Consent](https://satusehat.kemkes.go.id/platform/docs/id/fhir/resources/consent/)). A dedicated opt-in/opt-out consent endpoint, its default state and whether calling it is mandatory were not found in the pages read **[UNVERIFIED]**.

### d. Health-law framework — UU 17/2023 and PP 28/2024

- **Status.** UU 17/2023 (8 Aug 2023) in force; revoked UU 29/2004, 36/2009, 36/2014, 44/2009 and others; three Constitutional Court decisions (49, 111, 182/PUU-XXII/2024) concern the Konsil, Kolegium, professional organisations and the health-education transition, not records or data ([BPK record](https://peraturan.bpk.go.id/Details/258028/uu-no-17-tahun-2023)). PP 28/2024 (26 Jul 2024) in force, no amendment listed; revoked PP 46/2014 (health information systems) and PP 47/2016 (facilities) ([BPK record](https://peraturan.bpk.go.id/Details/294077/pp-no-28-tahun-2024)).
- **Tenant types.** First-level facilities: puskesmas, *klinik pratama*, **praktik mandiri** of medical/health personnel; advanced: hospital, *klinik utama*, balai, praktik mandiri; every facility needs a business licence ([UU 17/2023 Ps. 165, 167](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf); [PP 28/2024 Ps. 762–763](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
- **Facility duties.** Run medical records; report through the health information system; keep patients' health secrets; **may not employ personnel without a practice licence** ([UU 17/2023 Ps. 173, 177](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)).
- **STR / SIP.** STR is issued by the Konsil on behalf of the Minister and is **valid for life**; SIP is issued by the regency/city government, tied to a practice location, **valid 5 years**, lapses if the location changes or STR is revoked ([UU 17/2023 Ps. 260–264](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)). No public STR/SIP verification API is documented in the SATUSEHAT pages read ([Practitioner API](https://satusehat.kemkes.go.id/platform/docs/id/api-catalogue/onboardings/apis/practitioner/)); existence of a public lookup at the Konsil or SATUSEHAT SDMK **[UNVERIFIED]**.
- **Records at statute level.** Personnel must make records; outside solo practice the facility is responsible; document owned by the facility, patient may access; Kemenkes manages national record data ([UU 17/2023 Ps. 296–298](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)).
- **Informed consent.** Every individual health service needs consent after adequate explanation (diagnosis, indication, action and purpose, risks, alternatives, risk of no action, prognosis); oral or written; **written before invasive and/or high-risk actions**, signed by the patient/representative and **witnessed by a medical/health worker**; cost explanation also required ([UU 17/2023 Ps. 293–294](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)). Permenkes 290/2008 is still referenced by KMK 1423/2022; its current status **[UNVERIFIED]**.
- **Health information systems (SIK).** Operators include facilities and "masyarakat"; all must **integrate with SIKN** using ministerial data and interoperability standards ([UU 17/2023 Ps. 345](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf); [PP 28/2024 Ps. 946–947](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - **Processing location**: SIK operators "wajib melakukan pemrosesan data dan informasi Kesehatan di wilayah Indonesia"; processing abroad only per regulations; transfer abroad for specific, limited purposes **with central-government permission** ([UU 17/2023 Ps. 346(6)–(7), 349(7)](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)).
  - **Storage**: database in a **data centre inside Indonesia**; server or cloud; **backup mandatory**; third-party hosting allowed if in Indonesia, under a cooperation agreement covering at least confidentiality and service levels, with data "dimiliki dan dikendalikan secara penuh" by the SIK operator and processor duties met ([PP 28/2024 Ps. 966](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - **Lawful basis and notice**: processing individual health data needs the owner's consent and/or another PDP-law basis; operator must notify the owner; objections in writing/electronically ([PP 28/2024 Ps. 963](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf); [UU 17/2023 Ps. 351](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)).
  - **Reliability**: feasibility testing, confidentiality, access policy, **reliability certification**, periodic audit; **processing history log** accessible to the data owner ([UU 17/2023 Ps. 347, 349(11)](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf); [PP 28/2024 Ps. 977, 986](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - **Transfers to other parties**: by cooperation agreement (rights/obligations, purpose, security, processing mechanism) with prior risk analysis ([PP 28/2024 Ps. 970–971](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
- **Telehealth / telemedicine.**
  - *Telekesehatan* covers clinical and **non-clinical** services (the latter incl. promotive/preventive services and "sistem informasi dan administrasi Kesehatan"); clinical services are *Telemedisin* ([UU 17/2023 Ps. 25](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf); [PP 28/2024 Ps. 557](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - Telemedicine is provided **by a facility** (hospital, puskesmas, clinic, solo practice, lab, pharmacy), alone or with a **registered PSE**, and **by personnel holding STR and SIP**; teleconsultation = remote clinical consultation to help establish diagnosis/management ([UU 17/2023 Ps. 172](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf); [PP 28/2024 Ps. 558, 561–562](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - A self-developed telehealth/telemedicine application "harus teregistrasi" at Kemenkes ([PP 28/2024 Ps. 555(4), 560(3)](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).
  - AI ("kecerdasan buatan") is named as one form of health-information technology use ([PP 28/2024 Ps. 1019(4)](https://peraturan.bpk.go.id/Download/353434/PP%20Nomor%2028%20Tahun%202024.pdf)).

### e. Personal data protection and electronic-system rules

**UU 27/2022 (PDP).** In force since 17 Oct 2022; MK decision 151/PUU-XXII/2024 reads "dan" in Ps. 53(1)(b) as "dan/atau" ([BPK record](https://peraturan.bpk.go.id/Details/229798/uu-no-27-tahun-2022)). The 2-year adjustment period (Ps. 74) ended **17 Oct 2024**. All cites below: [UU 27/2022](https://peraturan.bpk.go.id/Download/224884/UU%20Nomor%2027%20Tahun%202022.pdf).

| Topic | Rule (Pasal) |
|---|---|
| Health data | "data dan informasi kesehatan" is **specific personal data**; so are children's data and biometrics (Ps. 4(2)) |
| Extraterritorial reach | Applies to acts outside Indonesia with legal effect in Indonesia or on Indonesian data subjects (Ps. 2) |
| Lawful bases | Explicit consent; contract; legal obligation; vital interest; public task; legitimate interest (Ps. 20(2)) |
| Consent form | Written or recorded, electronic allowed; where bundled with other purposes it must be clearly distinguishable, understandable, plain language — otherwise void; contract clauses requesting processing without explicit consent are void; controller must be able to **show proof of consent** (Ps. 22–24) |
| Notice content | Legality, purpose, data types, retention period, processing period, rights (Ps. 21) |
| Children / disability | Parent/guardian consent for children; special handling for persons with disabilities (Ps. 25–26) |
| Data-subject rights | Information, correction, access/copy, erasure, withdraw consent, object to automated decisions/profiling, restrict, compensation, portability (Ps. 5–13); requests via recorded application (Ps. 14) |
| Response clocks | Correction, access, stop after withdrawal, restriction — each **3×24 hours** (Ps. 30, 32, 40, 41) |
| Records / accountability | Record all processing activities; demonstrate accountability; supervise all parties under the controller's control (Ps. 31, 37, 47) |
| DPIA | Mandatory for high-risk processing, which includes **specific data**, automated decisions with significant effect, large scale, systematic evaluation/scoring/monitoring, **new technology** (Ps. 34) |
| Security | Technical/operational measures proportionate to risk; confidentiality; prevent unlawful access (Ps. 35–39) |
| Erasure vs retention | Delete/destroy when purpose ends, on withdrawal or request — "kecuali ditentukan lain oleh peraturan perundang-undangan" (Ps. 16(2)(g), 43–44) |
| **Breach notification** | Written notice **within 3×24 hours** to the data subject **and** the agency: what data, when/how, remediation; public notice in certain cases (Ps. 46) |
| Controller / processor | Controller determines purposes and controls processing; processor acts **on the controller's instruction** and on its behalf; controller remains responsible; **sub-processors need the controller's prior written approval**; a processor acting outside instructions becomes responsible itself; security/record duties (Ps. 29, 31, 35–39) apply to processors too (Ps. 1, 51–52) |
| DPO | Required where core activities involve regular systematic large-scale monitoring and/or **large-scale processing of specific data**; may be internal or external (Ps. 53–54) |
| Cross-border | Destination country must have equal or higher protection; else adequate binding safeguards; else data-subject consent (Ps. 56) |
| Sanctions | Administrative: warning, suspension of processing, deletion, **fine up to 2% of annual revenue/receipts** per violation variable, imposed by the agency (Ps. 57). Criminal: unlawful collection/use up to 5 years / Rp 5 bn; disclosure up to 4 years / Rp 4 bn; falsification up to 6 years / Rp 6 bn (Ps. 67–68) |
| Supervisor | An agency established by and answering to the President (Ps. 58–60) |

**Enforcement status today.**
- The implementing regulation is reported as **PP 33/2026**, enacted 16 Jul 2026 and taking effect six months after promulgation (≈16 Jan 2027) ([Hukumonline, 8 Oct 2026](https://www.hukumonline.com/berita/a/memandang-pdp-sebagai-investasi-korporasi-lt6ac761d6b8b14)) — **[UNVERIFIED at a primary source]**: a search of the BPK database for PP 33/2026 returned no record on this date ([BPK search](https://peraturan.bpk.go.id/Search?keywords=pelindungan+data+pribadi&tentang=&nomor=33&tahun=2026)). Its article-level content (DPO, DPIA, processor contracts, fines) was not read.
- Press reports say the Presidential Regulation creating the PDP agency was still pending in Oct 2026 **[UNVERIFIED — news search snippets only]**. Without the agency, Ps. 46 notices and Ps. 57 fines have no confirmed addressee/enforcer **(lawyer)**.

**PP 71/2019 (electronic systems and transactions).** In force; revoked PP 82/2012 ([BPK record](https://peraturan.bpk.go.id/Details/122030/pp-no-71-tahun-2019)). Cites: [PP 71/2019](https://peraturan.bpk.go.id/Download/112816/PP%20Nomor%2071%20Tahun%202019.pdf).
- A private-scope PSE includes operators regulated/supervised by a ministry and anyone running an internet portal/app that offers services, provides communication services, or **processes personal data for serving the public in connection with electronic transactions** (Ps. 2(5)).
- **Registration is mandatory before the system is used** (Ps. 6); via the OSS licensing system.
- Personal-data principles, consent and a duty to tell the data owner of protection failures (Ps. 14).
- **Location**: *public-scope* PSEs must keep systems and data in Indonesia (Ps. 20); **private-scope PSEs may host in or outside Indonesia**, but must ensure effective supervision and law enforcement and give access for those purposes (Ps. 21). Sector health rules (section d) are stricter and govern health data **[INFERENCE]**.
- Audit trail for all system activities (Ps. 22); security procedures and immediate reporting of serious incidents to law enforcement and the supervising ministry (Ps. 24).
- Sanctions: warning, fine, suspension, **access termination**, removal from the list (Ps. 100).

**Permenkominfo 5/2020 (private-scope PSE)**, in force, amended by Permenkominfo 10/2021 ([BPK record](https://peraturan.bpk.go.id/Details/203049/permenkominfo-no-5-tahun-2020)). Cites: [Permenkominfo 5/2020](https://peraturan.bpk.go.id/Download/197017/Nomor%205%20Tahun%202020.pdf).
- Registration through OSS with: system name, sector, URL, domain/IP, business model, functions, **personal data processed**, **location of management/processing/storage** (Ps. 2–3).
- Unregistered PSE → **access blocking**; inaccurate/unchanged data → warning, suspension, blocking (Ps. 7).
- Must give ministries (supervision) and law enforcement access to systems/data (Ps. 21, 36). The amendment's content was not read **[UNVERIFIED]**.

### f. Rules constraining an AI WhatsApp agent that markets and sells dental services

**Permenkes 1787/2010 — advertising and publication of health services** ([text, BN 2010/673](https://www.peraturan.go.id/files/bn673-2010.pdf)); current status not confirmed in the BPK database **[UNVERIFIED]**.
- Facilities may advertise, subject to professional codes and the advertising code (Ps. 3); content must be accurate, evidence-based, informative, educative, responsible; media ads carry the facility's **name, address and date** (Ps. 4).
- **Prohibited** (Ps. 5), among others: false/misleading statements; creating **inappropriate expectations**; comparing with or disparaging other facilities; **superlatives / "satu-satunya"**; unproven methods; advertising services of personnel/facilities **without a licence** (h); information given in a way that **"mendorong penggunaan jasa tenaga kesehatan" at that facility** (l); **sales promotion in any form incl. discounts**, rewards and MLM (m); **testimonials** in ads or mass-media publication (n); use of **academic titles/professional designations** (o).
- Health workers may not advertise or model for facilities except public-service ads (Ps. 8).
- Internet advertising/publication "tidak boleh digunakan sebagai sarana konsultasi medis jarak jauh (telemedicine)" (Ps. 10).
- Oversight by a Kemenkes assessment team and provincial health offices; referral to ethics/discipline bodies (Ps. 11–12).

**KODEKGI (PDGI dental ethics code)** ([PB PDGI 2017–2020 edition](https://fkg.unair.ac.id/en/wp-content/uploads/2021/06/Buku-Kode-Etik-Kedokteran-Gigi-2020.pdf)); whether a newer edition exists **[UNVERIFIED]**.
- Ps. 3 scope: dentists may not promote themselves "dalam bentuk apapun" (self-praise, promises) to attract patients; may not show off through mass, electronic or **social media**; explanatory items list as violations: superlatives and **guarantees of treatment success**, failing to prevent **patient testimonials** with name/photo/results, distributing **discount** flyers/banners, advertising self, colleagues or facilities.
- Ps. 5 (*Rabat dan komisi*): dentists are "tidak diperkenankan **menjaring pasien** secara pribadi, melalui pasien atau **agen**".
- Ps. 4: no misleading claims to third-party payers. Ps. 18: secrecy; release for insurance needs patient consent.

**Clinical boundary.** Clinical consultation at a distance is telemedicine, reserved to facilities and STR/SIP holders (section d); consent to treatment must follow an explanation by the provider and be written for invasive/high-risk care ([UU 17/2023 Ps. 293](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)).

**AI governance instruments.**
- **SE Menkominfo 9/2023 on AI ethics** (19 Dec 2023, status "Berlaku"): addressed to AI programming businesses and public/private PSEs; values incl. inclusivity, humanity, security, accessibility, transparency, credibility/accountability, personal-data protection; responsibilities incl. protecting the public's data and not letting AI make decisions on humanitarian matters; styled as ethical guidance ([JDIH Komdigi](https://jdih.komdigi.go.id/produk_hukum/view/id/883/t/surat+edaran+menteri+komunikasi+dan+informatika+nomor+9+tahun+2023)).
- **Kemenkes**: no binding AI-in-health regulation identified. A June 2026 Kemenkes forum aimed to *produce* policy recommendations and national guidelines (informed-consent standards, bias, overdiagnosis, data protection) ([Kemenkes, 9 Jun 2026](https://keslan.kemkes.go.id/read/3798/kemenkes-dorong-pengembangan-ekosistem-ai-kesehatan-yang-aman-adil-dan-bertanggung-jawab)); on 10 Nov 2026 the Vice Minister said Kemenkes will regulate all digital health platforms (telemedicine, record-keeping, tariffs, SIP of consulting doctors) and published a *Sandbox Kesehatan* guidebook covering AI and telemedicine ([Kemenkes, 11 Nov 2026](https://www.kemkes.go.id/id/wamenkes-dante-seluruh-platform-kesehatan-digital-harus-diregulasi)).
- Reported but not opened: KMK 951/2026 on software/AI medical devices; a national AI Presidential Regulation **[UNVERIFIED]**.
- PDP: right to object to solely automated decisions with legal/significant effect ([UU 27/2022 Ps. 10](https://peraturan.bpk.go.id/Download/224884/UU%20Nomor%2027%20Tahun%202022.pdf)).

**Outbound messages / consumer protection.**
- PP 80/2019 (in force, [BPK record](https://peraturan.bpk.go.id/Details/126143/pp-no-80-tahun-2019)): electronic ads may be sent for marketing, must respect consumer rights and privacy/data-protection law; **whoever makes, provides the means for, or disseminates an electronic ad is responsible for its content** ([PP 80/2019 Ps. 32–35](https://peraturan.bpk.go.id/Download/117172/PP%20Nomor%2080%20Tahun%202019.pdf)).
- No Indonesian opt-out/anti-spam article was found in the texts read; the operative constraints are PDP consent, purpose limitation and the 3×24 h stop-on-withdrawal rule ([UU 27/2022 Ps. 20–22, 40](https://peraturan.bpk.go.id/Download/224884/UU%20Nomor%2027%20Tahun%202022.pdf)). UU 8/1999 (consumer protection) was not opened **[UNVERIFIED]**.

### g. BPJS Kesehatan

- Perpres 82/2018 on Jaminan Kesehatan is amended three times (Perpres 75/2019, 64/2020, 59/2024) ([BPK record](https://peraturan.bpk.go.id/Details/94711/perpres-no-82-tahun-2018)). The original text's list of non-covered services includes **aesthetic services and "pelayanan meratakan gigi atau ortodonsi"** ([Perpres 82/2018, exclusions article](https://peraturan.bpk.go.id/Download/254897/Perpres%20Nomor%2082%20Tahun%202018.pdf)) — article number and post-amendment wording **[UNVERIFIED]**.
- Dental care at primary level under capitation, the scope of covered dental procedures, and the P-Care / Antrean bridging API and its onboarding were **not confirmed at any official BPJS source [UNVERIFIED]**.
- SATUSEHAT lists separate "Klaim BPJS" and "Klaim Non-BPJS" use cases ([docs navigation](https://satusehat.kemkes.go.id/platform/docs/id/bibliography/)); Permenkes 24/2022 treats claim data entry as an RME activity ([Ps. 13, 19](https://www.peraturan.go.id/files/bn829-2022.pdf)).

### h. Electronic signatures and consent documents

- UU 1/2024 (second amendment of UU ITE, in force 2 Jan 2024; the listed MK decision 115/PUU-XXII/2024 concerns Ps. 28(3)/45A(3), not signatures; decision 105/PUU-XXII/2024 not read) ([BPK record](https://peraturan.bpk.go.id/Details/274494/uu-no-1-tahun-2024)). Cites: [UU 1/2024](https://peraturan.bpk.go.id/Download/332870/UU%20Nomor%201%20Tahun%202024.pdf).
  - Everyone may use a certification provider (PSrE); a PSrE operating in Indonesia must be an Indonesian legal entity domiciled in Indonesia (Ps. 13); PSrE services include e-signature, **e-seal, e-timestamp**, registered e-delivery, **signature preservation**, digital identity (Ps. 13A).
  - **High-risk electronic transactions "menggunakan Tanda Tangan Elektronik yang diamankan dengan Sertifikat Elektronik"** (Ps. 17(2a)); detail delegated to a PP — whether medical consent is "high-risk" is not defined in the text read **(lawyer)**.
  - PSEs must protect children using the system (Ps. 16A).
- PP 71/2019 ([text](https://peraturan.bpk.go.id/Download/112816/PP%20Nomor%2071%20Tahun%202019.pdf)): an e-signature is valid if creation data are linked to and controlled only by the signer, later changes to signature and document are detectable, the signer is identifiable, and consent to the document is shown (Ps. 59(3)); signatures are **certified** (certificate from an Indonesian PSrE + certified creation device) or **uncertified** (Ps. 60) — the elucidation says the difference affects evidentiary weight.
- Health rules: each record entry carries the provider's signature ([UU 17/2023 Ps. 296(4)](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)); e-signature in RME is optional and must follow the e-signature laws ([Permenkes 24/2022 Ps. 31](https://www.peraturan.go.id/files/bn829-2022.pdf)); written treatment consent is signed by patient and witnessed by a health worker ([UU 17/2023 Ps. 293(8)](https://peraturan.bpk.go.id/Download/314883/UU%20Nomor%2017%20Tahun%202023.pdf)); PDP consent may be written or recorded electronically ([UU 27/2022 Ps. 22](https://peraturan.bpk.go.id/Download/224884/UU%20Nomor%2027%20Tahun%202022.pdf)).
- The current list of recognised PSrE (names, tiers) could not be opened **[UNVERIFIED]**. SATUSEHAT lists a "TTE" use case in its navigation, content not readable **[UNVERIFIED]** ([docs navigation](https://satusehat.kemkes.go.id/platform/docs/id/bibliography/)).

### Compliance requirements matrix

| # | Requirement | Source article | What the platform must do | Module affected | Launch? |
|---|---|---|---|---|---|
| 1 | PSE registration (health sector) before use | PP 71/2019 Ps. 6; Permenkominfo 5/2020 Ps. 2–3, 7; Permenkes 24/2022 Ps. 9(3) | Register the company's system via OSS; keep registered data (storage location, personal data processed) current | Company / legal, platform ops | **Must** |
| 2 | Register the RME system at Kemenkes; get it verified on SATUSEHAT | Permenkes 24/2022 Ps. 12; SATUSEHAT vendor guide | Prepare docs, variables/metadata statement, facility list, security survey, sandbox evidence (Encounter + Condition) | Interop service, compliance pack | **Must** (precondition for tenants to select the system) |
| 3 | Health data stored and processed in Indonesia, incl. backups | UU 17/2023 Ps. 346(6); PP 28/2024 Ps. 966; Permenkes 24/2022 Ps. 20, 22; SATUSEHAT form | Indonesian VPS/data centre for DB, object storage, backups, logs; no clinical data to offshore services without a lawful transfer path | Infrastructure, AI gateway, backups | **Must** |
| 4 | Off-site periodic backup with SOP | Permenkes 24/2022 Ps. 20(4)–(5); PP 28/2024 Ps. 966(4) | Second Indonesian site; scheduled, tested restores; documented | Infrastructure | **Must** |
| 5 | Tenant contract terms for hosted records | Permenkes 24/2022 Ps. 22; PP 28/2024 Ps. 966(5); UU 27/2022 Ps. 51 | Cooperation agreement + integrity pact/NDA + processing terms; tenant keeps full control and unrestricted access/export; sub-processor approval list | Legal templates, tenant onboarding, export tool | **Must** |
| 6 | Kemenkes recommendation for storage PSE | Permenkes 24/2022 Ps. 22(2) | Obtain / document the recommendation process | Compliance | Before first hosted tenant **(lawyer: procedure)** |
| 7 | Minimum record content, NIK, signed and timed entries | Permenkes 24/2022 Ps. 14, 16, 26(6); UU 17/2023 Ps. 296 | Schema with NIK, social data, per-entry author + timestamp + sign-off; consent attached to record | Patient registry, EMR | **Must** |
| 8 | Correction window and immutable history | Permenkes 24/2022 Ps. 30(6)–(7); Ps. 29 | Lock entries 48 h after input; later change = approved amendment; append-only versions | EMR, audit | **Must** |
| 9 | Role-based access set by facility head | Permenkes 24/2022 Ps. 30; UU 17/2023 Ps. 347(2)(c) | Per-tenant RBAC (input / correct / view); vendor staff have no default access; break-glass logged | Auth/RBAC | **Must** |
| 10 | Audit trail and processing history, accessible to data owner | PP 71/2019 Ps. 22; UU 27/2022 Ps. 31–32; PP 28/2024 Ps. 977 | Tamper-evident access/processing logs; patient-facing access report | Audit service | **Must** |
| 11 | 25-year retention; controlled destruction | Permenkes 24/2022 Ps. 39; UU 27/2022 Ps. 16(2)(g), 43–44 | Retention clock from last visit; legal-hold on erasure requests; tenant offboarding hand-over | Data lifecycle | **Must** (design), tooling later |
| 12 | SATUSEHAT integration per tenant | Permenkes 24/2022 Ps. 21; PP 28/2024 Ps. 781, 947; SATUSEHAT auth rules | Per-organisation credentials vault; Organization/Location/Practitioner/Patient sync; Encounter + Condition first, then dental module; retry queue and monitoring | Interop service | Encounter/Condition **at launch**; full dental module **later** |
| 13 | Standard terminologies | SE 7093/2023 no. 2(b); SATUSEHAT terminology | ICD-10, ICD-9-CM/KPTL, SNOMED CT, LOINC, KFA tables, versioned and updated quarterly; SNOMED affiliate licence (vendor + each facility) | Terminology service | ICD-10/ICD-9-CM **must**; rest with interop |
| 14 | Odontogram per Kemenkes model | SATUSEHAT Gigi v1.5 + Lampiran | Store FDI tooth, surface, condition, restoration, prosthesis as coded values mappable to `Observation` | Dental chart | Data model **must**; submission later |
| 15 | Lawful basis, notice, provable consent | UU 27/2022 Ps. 20–24; PP 28/2024 Ps. 963 | Consent/notice registry per purpose (care, insurance release, WhatsApp contact, marketing, AI processing); versioned text; proof export | Consent service, CRM, WhatsApp | **Must** |
| 16 | Data-subject requests within 3×24 h | UU 27/2022 Ps. 30, 32, 40, 41 | DSR inbox routed to the tenant (controller) with timers; stop-processing switch per patient | CRM, admin console | **Must** (basic) |
| 17 | Breach notification within 3×24 h | UU 27/2022 Ps. 46; PP 71/2019 Ps. 24(3) | Incident runbook; per-tenant impact reports; notify tenants immediately so they can notify subjects/agency | Security ops | **Must** |
| 18 | DPIA and data-protection officer | UU 27/2022 Ps. 34, 53 | DPIA before launch (specific data + new technology + automated interaction); appoint DPO function (may be external) | Governance | **Must** **(lawyer: scale test)** |
| 19 | Cross-border transfer controls | UU 27/2022 Ps. 56; UU 17/2023 Ps. 349(7) | Inventory of every offshore processor (LLM, messaging, e-mail, monitoring); minimise/pseudonymise; legal mechanism per flow | AI gateway, integrations | **Must** |
| 20 | Advertising and ethics limits | Permenkes 1787/2010 Ps. 4–5, 8, 10; KODEKGI Ps. 3, 5 | Content policy engine: no testimonials, discounts, superlatives, guarantees, comparisons; tenant-approved templates; facility name/address | AI agent, campaign tools, directory | **Must** |
| 21 | No clinical advice by the agent | UU 17/2023 Ps. 172(4); PP 28/2024 Ps. 561–562 | Scope guardrails, triage-to-human, disclosure that it is an automated assistant, transcripts retained | AI agent | **Must** |
| 22 | Practitioner licence checks | UU 17/2023 Ps. 173(3), 260–264; Permenkes 1787/2010 Ps. 5(h) | Capture STR number, SIP number, issuing regency, practice address, SIP expiry (5-year); document upload; expiry alerts; match practitioner NIK to SATUSEHAT IHS | Partner-dentist registry, directory | **Must** (manual verification) |
| 23 | Written consent for invasive/high-risk treatment | UU 17/2023 Ps. 293(5),(8) | Consent forms with required explanation items, patient signature + clinician witness; hash, timestamp, immutable storage | Consent documents | **Must**; certified e-signature option later |
| 24 | Child patients | UU 27/2022 Ps. 25; Permenkes 24/2022 Ps. 26(4); UU 1/2024 Ps. 16A | Guardian linkage and guardian consent for under-18 records and messaging | Patient registry, WhatsApp | **Must** |
| 25 | Telehealth app registration | PP 28/2024 Ps. 555(4), 560(3) | Determine whether the assistant/app is a self-developed telehealth application requiring Kemenkes registration | Compliance | **(lawyer)** before launch |

### Implications for the workspace platform

All items below are **[INFERENCE]**, each tied to facts cited above.

1. **Host everything clinical in Indonesia — including backups and logs.** Three independent layers say so (UU 17/2023 Ps. 346(6); PP 28/2024 Ps. 966; the SATUSEHAT vendor form), and the general PSE rule that permits offshore hosting (PP 71/2019 Ps. 21) does not displace them. An own-VPS model fits only if the VPS provider's data centre is in Indonesia and a second Indonesian location holds backups.
2. **The AI assistant is the main cross-border risk.** WhatsApp conversations with patients contain health data (specific personal data). Sending them to an offshore model API or to the messaging provider's servers is processing/transfer abroad under UU 17/2023 Ps. 346(7)/349(7) and UU 27/2022 Ps. 56. Design an AI gateway that (a) keeps the system of record in Indonesia, (b) strips identifiers and clinical detail before any offshore call, (c) records the legal mechanism per flow, and (d) can switch to an in-country model. **(lawyer)** must confirm whether offshore inference on pseudonymised chat text is permissible and whether a government permit is required.
3. **Role split: tenant = controller of patient records; the company = processor — but not for everything.** Record ownership and responsibility sit with the facility/dentist (UU 17/2023 Ps. 296–297; Permenkes 24/2022 Ps. 13(5), 25). For the public directory, lead capture before a clinic relationship exists, cross-tenant analytics and AI "closing" logic that the company designs, the company determines purposes and is likely a controller or joint controller (UU 27/2022 Ps. 1, 51(6)). Keep these data domains physically and contractually separate. **(lawyer)**
4. **Multi-tenancy must mirror SATUSEHAT's one-credential-per-organisation rule**: a secrets vault keyed by tenant Organization ID, per-tenant outbound queues, and no shared client credentials. Because solo dentists register through REGFASYANKES and clinics through DFO, tenant onboarding needs a guided checklist and a "selected our system in the registry?" status field.
5. **Get the system verified on SATUSEHAT before selling it as an RME.** Tenants can only pick verified systems, and facilities face accreditation-linked sanctions for not running an integrated RME (Permenkes 24/2022 Ps. 42; SE 7093/2023). Verification needs PSE registration and a SNOMED affiliate application first — sequence: PSE → SNOMED/MLDS → sandbox Encounter+Condition → portal registration and security survey.
6. **Model the odontogram on the SATUSEHAT Gigi structure from day one** (FDI tooth → surface → condition/restoration/prosthesis codes; OHI-S; D/M/F), storing SNOMED/local codes rather than abbreviations, since the published abbreviation list has duplicates. This avoids a later migration and doubles as the de facto national standard, given KMK 1423/2022 has no dental data set.
7. **Records must be append-only with a 48-hour edit window and per-entry sign-off**, and retained 25 years from the last visit — which outlives most SaaS contracts. Offer export in a standard format and an offboarding hand-over so the tenant can keep meeting Ps. 39 after leaving; erasure requests under UU 27/2022 must be answered with restriction, not deletion, while the retention duty runs.
8. **The "sales closing" behaviour of the agent collides with advertising and ethics rules.** Discounts, testimonials, before/after praise, superlatives, success guarantees and anything that "encourages use of" a specific provider's services are prohibited in health-service advertising (Permenkes 1787/2010 Ps. 5) and in the dental ethics code, which also bars recruiting patients through agents (KODEKGI Ps. 3, 5). Position the agent as an administrative assistant acting in the clinic's name: availability, booking, reminders, neutral service and price information, post-visit instructions approved by the dentist. Commission-per-patient pricing for the platform or directory is an ethics exposure for partner dentists. **(lawyer + PDGI ethics view)**
9. **Hard clinical boundary.** Diagnosis, treatment recommendations or reading radiographs over chat is telemedicine reserved to STR/SIP holders in a facility (PP 28/2024 Ps. 561–562) and internet advertising must not double as remote consultation (Permenkes 1787/2010 Ps. 10). The agent needs refusal-and-handover behaviour, an identified human clinician for anything clinical, and disclosure that it is automated (SE Menkominfo 9/2023 transparency value; UU 27/2022 Ps. 10).
10. **Consent is a product feature, not a checkbox.** Separate, provable consents are needed for: care record processing; insurance/admin release at registration (Permenkes 24/2022 Ps. 34(7)); WhatsApp contact; marketing follow-ups; AI processing; guardian consent for minors. Withdrawal must stop processing within 3×24 h — build a per-patient suppression flag that the WhatsApp scheduler checks before every outbound message.
11. **Treatment consent can launch with uncertified e-signatures plus strong evidence** (signer identification, document hash, timestamp, clinician witness), because validity turns on PP 71/2019 Ps. 59(3) and certification mainly raises evidentiary weight; leave a PSrE integration point for certified signatures/e-seals in case Ps. 17(2a) "high-risk transaction" rules are held to cover medical consent. **(lawyer)**
12. **Partner-dentist verification is manual at launch.** There is no documented STR/SIP API; capture numbers, issuing region and expiry, require document upload, and use the SATUSEHAT Practitioner lookup by NIK only as an existence check. Blocking an expired SIP from scheduling protects the tenant from Ps. 173(3).
13. **Governance artefacts required before go-live**: DPIA, DPO appointment (internal or external), record of processing, incident runbook with a 72-hour clock, processor/sub-processor register, security-survey answers. These same artefacts are what the SATUSEHAT security survey and PP 28/2024 Ps. 986 (testing, certification, audit) look for.
14. **BPJS integration is out of scope for launch.** Orthodontic and aesthetic services — a large share of private dental revenue — are excluded from JKN, the bridging API could not be confirmed, and claims are a separate SATUSEHAT use case. Keep a payer field and claim-ready coding (ICD-10/ICD-9-CM) so it can be added for clinics that are BPJS primary-care providers.
15. **Regulatory change is active**: PP 33/2026 (PDP implementing rules) is reported to bite from January 2027, the PDP agency is still forming, Kemenkes has announced platform-wide digital-health regulation and AI guidance, and SATUSEHAT revises its guides quarterly. Budget a compliance review each quarter and keep terminology/profile versions configurable.

### Open questions

1. Is Permenkes 24/2022 still the operative RME regulation, or has a Permenkes under PP 28/2024 Ps. 787 replaced/amended it (retention period, e-signature, storage PSE recommendation)? **(lawyer)**
2. What exactly does the "recommendation from the Kemenkes data unit" for storage PSEs (Permenkes 24/2022 Ps. 22(2)) require in practice, and is it satisfied by SATUSEHAT system verification?
3. Does a multi-tenant SaaS vendor count as a "penyelenggara Sistem Informasi Kesehatan" in its own right under UU 17/2023 Ps. 345, or only its tenants? This decides who carries the certification/audit duties in Ps. 347.
4. May patient chat content be processed by an offshore LLM or messaging provider, and under which mechanism (PDP Ps. 56 safeguards, consent, or a government permit under UU 17/2023 Ps. 349(7))?
5. Full text of PP 33/2026: DPIA procedure, DPO criteria ("large scale"), mandatory controller–processor contract content, breach-notice format, transition rules — and who receives breach notices until the PDP agency operates.
6. Does the AI assistant (non-clinical scheduling/CRM) fall under *Telekesehatan* such that the application must be registered with Kemenkes (PP 28/2024 Ps. 555(4))? Is the Sandbox Kesehatan route advisable?
7. Is Permenkes 1787/2010 still in force unchanged, and how do Kemenkes and PDGI's ethics council treat directory listings, lead-generation fees, chat-based promotion and patient reviews today? Is there a KODEKGI edition newer than 2017–2020?
8. Which SATUSEHAT resources are mandatory per outpatient/dental visit for a facility to count as "Terintegrasi", and is there a submission deadline per encounter? Is the patient-consent (opt-in/opt-out) API mandatory before sending?
9. Official text and status of the dental record guide (Kemenkes 2014) and whether any Konsil/Kolegium rule prescribes odontogram content or update frequency beyond the SATUSEHAT module.
10. Does UU 1/2024 Ps. 17(2a) (certified e-signature for high-risk transactions) apply to treatment consent or record sign-off once its implementing PP is issued? Which PSrE are currently recognised?
11. BPJS: current dental benefit scope at primary care after Perpres 59/2024, and the technical/contractual route for P-Care bridging by a third-party system.
12. Whether a judicial-review decision exists on Permenkes 24/2022 (BPK lists none) and whether SE 1030/2023's sanction thresholds have been updated since 2024.


---

## 5. WhatsApp-based agentic AI operations

Research date 2026-11-25. Source notes:
- Meta developer docs were read from Meta's own Markdown renderings (same URL + `.md`); links below point to the canonical page. Their code samples use Graph API **v27.0** ([messaging limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/messaging-limits)).
- Meta legal texts read: *Meta Terms for WhatsApp Business Platform* and *WhatsApp Business Messaging Policy*, both "last modified/updated September 23, 2026" ([terms](https://www.facebook.com/legal/Meta-Terms-for-WhatsApp-Business-Platform?locale=en_US), [policy](https://whatsappbusiness.com/policy/)). `whatsapp.com/legal/business-solution-terms` and `business.whatsapp.com/policy` now 301-redirect to those two URLs.
- Tags: **[INFERENCE]** = my reasoning; **[UNVERIFIED]** = not confirmed at a primary source in this session.
- Known conflict between Meta's own pages: the marketing pricing page still says service messages are not charged and the free-entry-point window is 72 hours incl. Facebook Page CTA ([marketing page](https://whatsappbusiness.com/products/platform-pricing/)); the developer pricing doc (updated for 1 Oct 2026) says otherwise ([pricing doc](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)). The developer doc and its rate cards are treated as authoritative below.

### 5a. WhatsApp Business Platform today (Cloud API)

**Capabilities**
- Cloud API covers messaging (text, media, interactive), **Calling** (make/receive VoIP calls) and **Groups**; the Business Management API covers accounts, numbers, templates, billing and analytics; a separate Marketing Messages API sends optimised marketing templates; "Meta Business Agent" is Meta's own configurable AI agent ([about the platform](https://developers.facebook.com/documentation/business-messaging/whatsapp/about-the-platform)).
- Account model changed: the WABA is splitting into a **WhatsApp account (WAAC)** (phone number, profile, identity) and a **Messaging account** (templates, billing, webhook subscriptions); existing `waba_id` values now denote Messaging account IDs; GA 23 Sep 2026, all businesses by mid-Oct 2026. One number can now be shared by several partners with separate billing ([account model](https://developers.facebook.com/documentation/business-messaging/whatsapp/account-model-evolution)).
- Throughput: 80 messages/second per number by default, auto-upgrade to 1,000 mps; coexistence numbers fixed at 20 mps ([throughput](https://developers.facebook.com/documentation/business-messaging/whatsapp/throughput)). Pair rate limit: 1 message per 6 s to the same user, bursts of up to 45 borrowed from future quota ([about the platform](https://developers.facebook.com/documentation/business-messaging/whatsapp/about-the-platform)).
- Data handling: Cloud API decrypts and forwards messages to the business; Meta states it acts as data processor/service provider; messages retained max 30 days; optional "local storage" lets a business choose the at-rest country ([data privacy](https://developers.facebook.com/documentation/business-messaging/whatsapp/data-privacy-and-security)). Whether Indonesia is a selectable local-storage region: **[UNVERIFIED]** (region list lives in the registration reference, not opened).

**Pricing model (per-message since 1 Jul 2025; major change 1 Oct 2026)**
- Meta charges per *delivered* business-to-user message, by category and by the recipient's country calling code; user-to-business messages are never charged ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).
- Five categories: template messages = **marketing / utility / authentication**; non-template messages (only inside the 24-hour window) = **service** (any non-template message not powered by Meta Business Agent, including "a 3rd-party AI solution or people") or **Meta Business Agent** (since 1 Jul 2026, USD 2.00 per 1M tokens, ~20–25k tokens ≈ 4–5 US cents per message) ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).
- **Since 1 Oct 2026 service messages are charged again** (they had been free since 1 Nov 2024), with a **free tier of 1,000 delivered service messages per business phone number per month** (no roll-over); **utility templates sent inside an open customer service window are also charged again** (they had been free since 1 Jul 2025). Without a payment method, service messages beyond the free tier are not delivered ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).
- Service rates equal utility/authentication rates per market and have no volume tiers; volume tiers exist only for utility and authentication, aggregated per business portfolio ([non-template messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/non-template-messages), [pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).
- Pricing calendar: changes only on 1 Jan / 1 Apr / 1 Jul / 1 Oct; minimum notice 1 month (rate card), 3 months (model add-on), 6 months (model change). IDR is one of 16 billing currencies; currency is fixed at Messaging-account creation ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).
- Eligible governments and non-profits keep free service messages through 31 Dec 2027 — not applicable to private clinics ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).

**Indonesia (+62) list rates, effective 1 Oct 2026** — from the "USD list rates" and "IDR list rates" files linked on the [pricing doc](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing) (files are spreadsheets served from signed Meta CDN links, parsed in this session):

| Category | USD / message | IDR / message | Notes |
|---|---|---|---|
| Marketing | 0.0411 | 586.33 | template, outside or inside window |
| Utility | 0.0250 | 356.65 | list rate applies to first 750,000/month; tiers beyond |
| Authentication | 0.0250 | 356.65 | |
| Authentication-International | 0.1360 | 1,940.13 | applies only to businesses Meta flags as eligible (eligibility page not opened: **[UNVERIFIED]**) |
| Service | 0.0250 | 356.65 | after the 1,000 free per number per month |
| Meta Business Agent | USD 2.00 / 1M tokens | — | not available to health vertical (see 5c) |

**Customer service window and free entry points**
- A 24-hour customer service window (CSW) opens when the user messages or calls and resets on each user message; non-template messages are only allowed while it is open; outside it only templates ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), [send messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages)).
- Free entry point (FEP): if the user arrives via a Click-to-WhatsApp ad (Android/iOS only) and the business replies within the CSW, that reply and all marketing/utility/authentication/service messages are free while the FEP window is open — "up to 7 days" per the developer doc ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)). A plain `wa.me` link from a directory website is **not** an FEP **[INFERENCE from the doc naming only CTWA ads]**.

**Templates: categories, approval, quality, pacing**
- Utility must be non-promotional **and** specific to/requested by the user or essential; mixed or unclear content is marketing; "retargeting" (renew, come back, saved search) is marketing "even if requested by users" ([categorization](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-categorization)).
- Since 9 Apr 2025 a template submitted as utility that Meta judges to be marketing is approved *as marketing*; approved utility templates are re-scanned and re-categorised with 1 day's notice (no notice for accounts warned for misuse) ([categorization](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-categorization)).
- Review uses automated + manual checks; decision "can take up to 24 hours"; appeals decided within 24 hours; common rejections: malformed variables, starting/ending with a variable, requesting sensitive identifiers, threatening content, duplicates ([template review](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-review)).
- Quality score per template: GREEN / YELLOW / RED / UNKNOWN ([template quality](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-quality)). A RED template is paused 3 h (1st), 6 h (2nd), then disabled ([template pausing](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-pausing)).
- Template pacing holds new/low-history marketing and (after a utility pause) utility templates at an undisclosed threshold until feedback arrives; held sends return `held_for_quality_assessment` ([template pacing](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-pacing)). Portfolio pacing applies to portfolios with <500K templates sent in 365 days ([portfolio pacing](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/portfolio-pacing)).
- Per-user marketing caps: WhatsApp may drop marketing templates to a user who is unlikely to engage (error 131049; wait ≥24 h before retry) ([per-user limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/marketing-templates/per-user-limits)); users can stop/resume all marketing from a business, reported via the `user_preferences` webhook ([marketing templates](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/marketing-templates)).
- **Booking appointment templates** (utility family): structured appointment card with required start date/time, optional end, location, phone, booking URL and booking-management URL (reschedule/cancel), fixed "View details" button ([booking appointment templates](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/utility-templates/booking-appointment-templates)).

**Opt-in**
- Policy text: "You may only contact people on WhatsApp if: (a) they have given you their mobile phone number or username; and (b) you have received opt-in permission from the recipient confirming that they wish to receive subsequent messages or calls from you." A separate opt-in is required to initiate calls; opt-out instructions must be provided and honoured ([policy](https://whatsappbusiness.com/policy/)).
- Opt-in may be general (not WhatsApp-specific) if lawful, must name the business, and may be collected by website, SMS, IVR, in person or on paper ([opt-in guide](https://developers.facebook.com/documentation/business-messaging/whatsapp/getting-opt-in)).

**Messaging limits, quality, restrictions**
- Limit = unique users reachable *outside* a CSW per moving 24 h, set per **business portfolio** (shared by all its numbers): new portfolios start at **250**, then 2,000 → 10,000 → 100,000 → unlimited ([messaging limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/messaging-limits)).
- To reach 2,000: verify the business, or partner-led verification, or send 2,000 delivered templates to unique users within 30 days with high quality; after that, auto-scaling within 6 hours when ≥50% of the limit was used in the last 7 days with high quality ([messaging limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/messaging-limits)).
- Since 7 Oct 2025 the "Flagged" number state no longer exists and a quality drop no longer downgrades the limit ([limits change log](https://developers.facebook.com/documentation/business-messaging/whatsapp/upcoming-messaging-limits-changes)). Message quality is based on the last 7 days of blocks, reports, mutes and archives ([send messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages)).
- Enforcement ladder for policy/spam violations: warning → 1- or 3-day block on templates and on adding numbers → 5/7/30-day block on all messages → account lock (appeal only) → permanent disablement; severe harm or "excessive negative feedback" can mean immediate offboarding; appeals typically 24–48 h; `account_update` webhook reports violations ([policy enforcement](https://developers.facebook.com/documentation/business-messaging/whatsapp/policy-enforcement)).

**Numbers, display names, verification**
- A number must be owned by the business, have country+area code, and receive SMS or voice for the OTP; a number already used in WhatsApp must be deleted there first (except coexistence, below) ([business phone numbers](https://developers.facebook.com/documentation/business-messaging/whatsapp/business-phone-numbers/phone-numbers)).
- Display name is mandatory, reviewed automatically when the number reaches a higher limit, changeable 10 times per 30 days, and needs re-registration within 14 days of approval ([display names](https://developers.facebook.com/documentation/business-messaging/whatsapp/display-names)).
- Business verification is not required to start (250 limit) but unlocks the 2,000 tier, is a prerequisite to *send Flows*, and is required for Official Business Account (blue tick), which additionally needs 30 days on the platform, two-step verification and an approved display name; OBA is not granted to WhatsApp Business app (coexistence) numbers ([messaging limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/messaging-limits), [Flows prerequisites](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/gettingstarted), [OBA](https://developers.facebook.com/documentation/business-messaging/whatsapp/official-business-accounts)).

**Coexistence (WhatsApp Business app + Cloud API on one number)**
- Supported through Embedded Signup with the `whatsapp_business_app_onboarding` feature (on by default in ES v5); requires app version ≥2.24.17 and that the onboarding party is a Tech Provider or Solution Partner ([coexistence](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users)).
- Messages are mirrored both ways; up to 6 months of 1:1 history and all contacts can be synchronised, but only within 24 hours of onboarding; staff messages sent from the phone app arrive as `smb_message_echoes` webhooks ([coexistence](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users), [smb_message_echoes](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/smb_message_echoes)).
- Messages sent from the app stay free and are not bound by the 24-hour window; API messages are charged normally. Side effects: broadcast lists disabled, disappearing/view-once/live-location turned off, groups and app calls not mirrored to the API, linked devices unlinked once (re-link; Windows and WearOS clients unsupported), 20 mps cap ([coexistence](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users)).

**WhatsApp Flows (in-chat forms) — can it book appointments? Yes.**
- Flows are multi-screen forms defined in Flow JSON, optionally backed by a business **endpoint** for dynamic data exchange (encrypted payloads, health checks) ([Flows](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows), [endpoint guide](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/implementingyourflowendpoint)).
- Components include `DatePicker` and `CalendarPicker` (Flow JSON ≥6.1) with `min-date`, `max-date`, `unavailable-dates`, `include-days` and an on-select `data_exchange` action — enough to show live availability from a clinic calendar; plus Dropdown, RadioButtons, TextInput, OptIn, media upload ([components](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/components)). Meta lists an official "Book Appointments with WhatsApp Flows" tutorial among its examples ([Flows examples](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/examples)).
- A Flow is sent as a template `FLOW` button (business-initiated) or as an interactive message inside the CSW ([sending a Flow](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow)).
- Constraints: sender must have a **verified business** and high message quality ([Flows prerequisites](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/gettingstarted)); endpoint error-rate/latency/availability are monitored and a degraded Flow is throttled to 10 sends/hour, then blocked ([Flow health](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/healthmonitoring)); partners should "use a dedicated endpoint and an encryption key pair for each Messaging account" ([endpoint guide](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/implementingyourflowendpoint)).

**Interactive messages (inside the CSW)**
- Reply buttons: up to 3, label ≤20 chars ([reply buttons](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/interactive-reply-buttons-messages)). List: up to 10 rows across up to 10 sections, button label ≤20 chars, row description ≤72 chars ([list messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/interactive-list-messages)). Also CTA-URL button, location request, media carousel, address ([send messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages)).
- Ice breakers and slash-commands can be set per number, but ice breakers are auto-dismissed when the user arrives via a `wa.me` link with pre-filled text ([conversational components](https://developers.facebook.com/documentation/business-messaging/whatsapp/business-phone-numbers/conversational-components)).

**Calling API**
- Prerequisites to enable the Calling API on a number: the number is "in use with Cloud API (not the WhatsApp Business app)" — so **coexistence numbers are excluded** — and the business has a messaging limit of at least 2,000 ([calling](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling)).
- User-initiated calls are free and available wherever Cloud API is; business-initiated calls are unavailable for business numbers in the US, Canada, Egypt, Vietnam and Nigeria (Indonesia not excluded) ([calling](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling), [calling pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling/pricing)).
- Business-initiated calls need user call permission: max 2 permission requests per 7 days, temporary permission lasts 7 days, 4 consecutive unanswered calls revoke it ([call permissions](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling/user-call-permissions)). Billed in 6-second pulses by destination country and volume tier; IDR rate card exists (per-minute Indonesia rate not extracted: **[UNVERIFIED]**) ([calling pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling/pricing)).
- Signalling is Graph API + webhooks with WebRTC media, or SIP ([calling](https://developers.facebook.com/documentation/business-messaging/whatsapp/calling)).

**Webhooks and lead attribution**
- Failed deliveries are retried with decreasing frequency for up to 7 days and can produce duplicates; mTLS is supported ([webhooks](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/overview)).
- **Ad attribution**: the first inbound message from a Click-to-WhatsApp ad carries a `referral` object: `source_url`, `source_id` (ad ID), `source_type: "ad"`, `headline`, `body`, media fields and `ctwa_clid` (omitted for WhatsApp Status placements) ([text message webhook](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/text)).
- **Directory attribution**: click-to-chat links are `https://wa.me/<number>` and `https://wa.me/<number>?text=<urlencoded text>` (international format, no leading zeros or symbols) ([WhatsApp Help Center](https://faq.whatsapp.com/5913398998672934)). API-managed QR codes/short links: max 2,000 per number, pre-filled text ≤140 chars, and Meta provides **no analytics** for them ([QR codes](https://developers.facebook.com/documentation/business-messaging/whatsapp/qr-codes)). No `referral` object is documented for plain `wa.me` links, so attribution must be carried in the pre-filled text or by a first-party redirect **[INFERENCE]**.
- Status webhooks carry a `pricing` object (`billable`, `category`, `type`, `subtype: free_tier`) usable for per-tenant cost metering ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)).

### 5b. Multi-tenant onboarding: what the SaaS must be

**Meta's partner types** ([partners overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/overview))

| | Tech Provider | Tech Partner | Solution Partner (Meta's setup guide now titles this "[Technology Partner with the Reseller add-on](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/get-started-for-solution-partners)") |
|---|---|---|---|
| Can serve onboarded clients via API | Yes | Yes | Yes |
| Credit line / bills clients for Meta fees | No — each client adds its own payment method; Meta bills the client | No | Yes — partner is "Bill To Party", liable for all client spend ([credit lines](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/share-and-revoke-credit-lines)) |
| Meta Business Partner badge / accelerator | No | Yes | Yes |
| Direct Support | Yes | Yes | Yes |
| Partner-led business verification of clients | No | No | Only "Select" / "Premier" tiers |

- **Anyone whose app is used by other businesses must have Advanced access** to `whatsapp_business_management` and `whatsapp_business_messaging` via App Review; a direct developer using only its own number does not ([app review](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/app-review)).
- **Becoming a Tech Provider**: Meta app with WhatsApp use case + business portfolio → (1) verify the operator's business (name, address, phone, email, website, documents) → (2) App Review with two screen recordings (send a message; create a template) and written justification ([become a Tech Provider](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/get-started-for-tech-providers)). Meta states average App Review turnaround "about 24 hours" ([app review](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/app-review)); Twilio tells ISVs that Meta app approval plus partner linking "usually take 3-4 weeks" ([Twilio ISV guide](https://www.twilio.com/docs/whatsapp/isv/tech-provider-program)). Business-verification duration is not stated by Meta: **[UNVERIFIED]**.
- **Tech Partner upgrade** needs ≥2,500 average daily messages, ≥10 active messaging accounts, quality thresholds and Blueprint training ([upgrade](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/upgrade-to-tech-partner)). Solution Partner is "a lengthy process" per Meta ([partners overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/overview)).

**Embedded Signup (ES)**
- ES is a Facebook-Login-for-Business popup in the SaaS: the clinic logs in, accepts terms, picks/creates its business portfolio and WhatsApp account, verifies its number, sets a display name; the SaaS receives `waba_id`, `phone_number_id` and a code to exchange server-side for a client-scoped business token, then subscribes webhooks and registers the number ([ES overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/overview), [TP onboarding steps](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-customers-as-a-tech-provider)).
- **Ownership**: "Business customers onboarded via Embedded Signup own all of their WhatsApp assets" and keep full WhatsApp Manager access that the partner cannot restrict ([ES overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/overview)). The older On-Behalf-Of model (partner-owned WABAs) is deprecated ([OBO deprecation](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/obo-model-deprecation)).
- **Versions**: ES v2/v3 were deprecated on 15 Oct 2026; build on v4 or v5 ([ES overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/overview)).
- **Onboarding caps**: 10 new WhatsApp accounts per rolling 7 days by default; 200 after Business Verification + App Review + Access Verification; more requires Meta Business Partner status ([ES overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/overview)).
- **Billing for Tech Providers**: each clinic must attach a payment method to its own Messaging account before it can send ([ES overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/overview)). Prepaid billing exists only in India (UPI) ([prepaid billing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/prepaid-billing)); the terms mention cards and invoiced credit lines ([terms](https://www.facebook.com/legal/Meta-Terms-for-WhatsApp-Business-Platform?locale=en_US)). Which Indonesian payment instruments Billing Hub accepts: **[UNVERIFIED]**.
- **Multi-Partner Solutions** let a Tech Provider pair with a Solution Partner so clients use the partner's credit line ([partners overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/overview)).
- Rate limit per app per client Messaging account: 5,000 Business Management API requests/hour once a number is registered ([about the platform](https://developers.facebook.com/documentation/business-messaging/whatsapp/about-the-platform)).

**Going through a BSP instead** (official pricing/docs pages)

| Provider | Platform fee | Meta message fees | Multi-tenant mechanics | Source |
|---|---|---|---|---|
| Direct (own Tech Provider app) | none to Meta | Meta list rates, billed by Meta to each clinic | own ES + webhooks | [partners overview](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/overview) |
| 360dialog Partner Platform | Starter €250/mo incl. 5 channels (+€49 each); Growth €500 incl. 10 (+€25); Premium €1,000 incl. 20 (+€15) | "Billed at the Official Meta rate"; +7% if marketing is sent via the standard endpoint; card payments +4% | Partner Hub/API; >3 numbers requires the partner to be a Meta Tech Provider with a Multi-Partner Solution; 200 clients per rolling 7 days per solution | [pricing](https://docs.360dialog.com/partner/get-started/pricing), [TP program](https://docs.360dialog.com/partner/get-started/tech-provider-program) |
| Twilio | USD 0.005 per message, inbound and outbound; USD 0.001 per failed message | Meta fees passed through | ISV must join Meta's Tech Provider program, link app to Twilio as Partner Solution, use ES + Twilio Senders API | [pricing](https://www.twilio.com/en-us/whatsapp/pricing), [ISV guide](https://www.twilio.com/docs/whatsapp/isv/tech-provider-program) |
| Infobip | none listed; custom plans | Own per-message price list; Indonesia: utility/authentication/free-form USD 0.03375, marketing USD 0.0572, international authentication USD 0.155428 (page updated 16 Nov 2026) — about 35–39% above Meta's list rates above **[INFERENCE: arithmetic]** | not documented on pricing page | [pricing](https://www.infobip.com/whatsapp-business/pricing) |
| Qiscus (ID) | quote only; depends on products, agents and monthly active customers | "per pesan berdasarkan tarif Meta" | finished omnichannel inbox product; no reseller/API terms on the page | [pricing](https://www.qiscus.com/pricing) |
| Mekari Qontak (ID) | pricing page returned only its title to the fetcher (JS); a search snippet says "from IDR 400,000 per user per month" **[UNVERIFIED]** | **[UNVERIFIED]** | finished CRM/omnichannel product | [pricing page (not readable)](https://qontak.com/en/pricing) |
| Kata.ai (ID) | no public pricing; home page describes a custom enterprise agent platform; WhatsApp BSP status **[UNVERIFIED]** | **[UNVERIFIED]** | enterprise projects | [home](https://kata.ai/) |

Trade-offs:
- BSPs do **not** remove the Meta work: both 360dialog and Twilio require the ISV to become a Meta Tech Provider anyway (sources in table). What they add is a credit line/consolidated invoicing, support escalation and a simplified API; what they cost is a per-channel or per-message margin and a second vendor in the data path **[INFERENCE]**.
- Indonesian inbox vendors (Qiscus, Qontak) sell a competing finished dashboard, not infrastructure for a third-party SaaS **[INFERENCE from their pricing pages]**.
- With per-partner Messaging accounts in the new account model, a clinic can later add or remove a partner on the same number without migrating it ([account model](https://developers.facebook.com/documentation/business-messaging/whatsapp/account-model-evolution)), lowering lock-in either way **[INFERENCE]**.

### 5c. Policy limits on AI and on healthcare

**AI Providers clause — exact wording** (Meta Terms for WhatsApp Business Platform §4.7, document last modified 23 Sep 2026; Meta's pricing doc dates the clause to the terms update of 15 Jan 2026) ([terms](https://www.facebook.com/legal/Meta-Terms-for-WhatsApp-Business-Platform?locale=en_US), [AI Providers pricing policy](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/ai-providers)):

> "Providers and developers of artificial intelligence or machine learning technologies, including but not limited to large language models, generative artificial intelligence platforms, general-purpose artificial intelligence assistants, or similar technologies as determined by Meta in its sole discretion ("AI Providers"), are strictly prohibited from accessing or using the WhatsApp Business Platform, whether directly or indirectly, for the purposes of providing, delivering, offering, selling, or otherwise making available such technologies when such technologies are the primary (rather than incidental or ancillary) functionality being made available for use, as determined by Meta in its sole discretion; provided, however, that such technologies may be made available to businesses in certain countries as set forth here. Notwithstanding the foregoing, you may retain an AI Provider as your Solution Provider. In such cases, you may not directly or indirectly allow WhatsApp Business Platform Data, including any anonymous, aggregate, or derived forms of WhatsApp Business Platform Data, to be used to create, develop, train, or improve any machine learning or artificial intelligence systems, models, or technologies, including large language models (collectively, "AI Models"); provided that you may use WhatsApp Business Platform Data to fine-tune an AI Model that is for your exclusive use, so long as this does not result in WhatsApp Business Platform Data being used to create, develop, train, or improve any other AI Models. This Section survives termination of these Terms."

(Assembled from consecutive verbatim segments returned by the fetch tool; re-read the live page before quoting it in a contract.)

- **Prohibited**: using the platform to distribute an AI assistant as the *primary* product (general-purpose chatbots). The carve-out countries are Brazil, Italy, 29 European markets and Türkiye — Indonesia is not listed ([AI Providers pricing policy](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/ai-providers)).
- **Allowed**: a business's own customer-service automation where AI is incidental to its service. Meta's pricing doc itself defines service messages as non-template messages "powered by a 3rd-party AI solution or people" and prices them normally; the AI-provider policy "does NOT change how Meta charges all other businesses" ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), [AI Providers pricing policy](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/ai-providers)). A clinic answering its own patients about its own services with an LLM falls in this class **[INFERENCE]**; the determination is "in Meta's sole discretion", so the product must stay scoped to the clinic's business (no open-ended general assistant behaviour) **[INFERENCE]**.
- **Data-use duty that binds the SaaS**: WhatsApp conversation data (even anonymised/aggregated) may not train or improve AI models, except fine-tuning a model for the clinic's exclusive use (clause above). Cross-tenant learning on chat transcripts is therefore off-limits **[INFERENCE]**. Anthropic's API commitment is that retained data "is never used for model training without your express permission" ([Anthropic data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)).
- A dedicated "AI-enabled phone numbers" page is listed in Meta's doc index but returned 404 in this session; any labelling/registration duty for AI-driven numbers is **[UNVERIFIED]** ([doc index](https://developers.facebook.com/documentation/business-messaging/whatsapp/llms.txt)).

**Human escalation** — policy text: "You may use automation when responding during the 24-hour window, but must also have available prompt, clear, and direct escalation paths." Accepted paths listed: in-chat human agent transfer, phone number, email, web support, in-store visits, support form ([policy](https://whatsappbusiness.com/policy/)).

**Health content and sensitive data**
- Policy (verbatim, checked against the page HTML): "Don’t use WhatsApp for telemedicine or to send or request any health related information, if applicable regulations prohibit distribution of such information to systems that do not meet heightened requirements to handle health related information." ([policy](https://whatsappbusiness.com/policy/)). The restriction is conditional on local law — the Indonesian-law track decides whether it bites.
- The same policy lists "Medical and healthcare products" and "Drugs, whether prescription, recreational, or otherwise" among goods that may not be bought, sold or promoted; its "Regulated Verticals" (country-gated, adults only) are online gambling, over-the-counter drugs and alcohol — dental *services* are in neither list ([policy](https://whatsappbusiness.com/policy/)).
- Policy: "Don't share or ask people to share full length individual payment card numbers, financial account numbers, personal ID card numbers, or other sensitive identifiers" ([policy](https://whatsappbusiness.com/policy/)); template review rejects templates that request such identifiers or documents containing them ([template review](https://developers.facebook.com/documentation/business-messaging/whatsapp/templates/template-review)). Asking for a full NIK or BPJS number in chat or in a template conflicts with this **[INFERENCE]**.
- Terms §4.2: the business may not give Meta "Prohibited Information" (data subject to legal safeguarding/distribution limits); Meta "is not a Business Associate or Subcontractor" under HIPAA and "the WhatsApp Business Platform is not HIPAA compliant" ([terms](https://www.facebook.com/legal/Meta-Terms-for-WhatsApp-Business-Platform?locale=en_US)).
- Commerce enforcement category `HEALTHCARE`: businesses may not transact in certain healthcare products incl. medical devices, but "healthcare services who do not engage in direct sale are allowed" ([violations](https://developers.facebook.com/documentation/business-messaging/whatsapp/policy-enforcement-violations)). Clinics are allowed; selling devices/medicines through the chat is the risk area **[INFERENCE]**.

**Meta's own AI agent** — Meta Business Agent exists (knowledge sources, custom connectors incl. "booking an appointment", thread-control handoff) but eligibility excludes "Finance, Government, **Health**, Alcohol, Gambling, over-the-counter drugs, and matrimony services", requires an "eligible country" and blocks numbers "already running another AI agent" ([Business Agent overview](https://developers.facebook.com/documentation/meta-business-agent/overview)). Dental clinics are therefore not eligible on the vertical rule alone **[INFERENCE]**; whether Indonesia is an eligible country is **[UNVERIFIED]**.

**Unofficial client libraries (WhatsApp Web automation)**
- Consumer Terms of Service (effective 4 Jan 2021) forbid, "directly or through automated means", reverse engineering, "bulk messaging, auto-messaging, auto-dialing", creating accounts by automated means, and creating software/APIs that function substantially the same as the service; violations allow WhatsApp to disable or suspend the account ([WhatsApp ToS](https://www.whatsapp.com/legal/terms-of-service)).
- Help Center (pages were served in Indonesian; paraphrased, not verbatim): accounts may be temporarily or permanently blocked for using unofficial WhatsApp apps or websites, because WhatsApp cannot validate their security/privacy practices ([unofficial apps](https://faq.whatsapp.com/1217634902127718)); WhatsApp's products are not intended for bulk or automated messaging, and it reserves the right to take legal action ([unauthorized automated messaging](https://faq.whatsapp.com/5957850900902049)).
- Platform terms §4.1 also forbid scraping, reverse engineering and building look-alike APIs ([terms](https://www.facebook.com/legal/Meta-Terms-for-WhatsApp-Business-Platform?locale=en_US)).
- Enforcement risk for a SaaS built on such libraries: the ban lands on the *clinic's* number (its patient contact channel), with no appeal path the SaaS controls **[INFERENCE]**. Coexistence now gives the same "keep using the phone app" benefit legitimately (5a).

### 5d. Agent design (Claude as primary reference)

**Models and prices (Anthropic first-party API, USD per million tokens)** ([pricing](https://platform.claude.com/docs/en/about-claude/pricing), [models overview](https://platform.claude.com/docs/en/about-claude/models/overview))

| Model (API ID) | Input | Cache read | 5-min cache write | Output | Docs positioning |
|---|---|---|---|---|---|
| Claude Haiku 5.5 (`claude-haiku-5-5`), prompts ≤100k tokens | 0.10 | 0.01 | 0.125 | 0.50 | "high-volume, latency-sensitive tasks such as classification, extraction, and routing" |
| Claude Sonnet 5.5 (`claude-sonnet-5-5`) | 2.00 | 0.10 | 2.50 | 10.00 | "best combination of speed and intelligence" |
| Claude Opus 5.5 (`claude-opus-5-5`) | 4.00 | 0.20 | 5.00 | 20.00 | default recommendation "for most workloads" |
| Claude Fable 5.2 (`claude-fable-5-2`) | 8.00 | 0.20 | 10.00 | 40.00 | demanding reasoning; requires 30-day retention |

- All four: 1M-token context, 128K max output, tool use, vision, multilingual ([models overview](https://platform.claude.com/docs/en/about-claude/models/overview)). Anthropic's own multilingual table lists Indonesian at 97–99% of English performance across the models shown ([multilingual support](https://platform.claude.com/docs/en/build-with-claude/multilingual-support)).
- Anthropic's customer-support guide: Opus 5.5 "is well suited to balance intelligence, latency, and cost"; for flows with "RAG, tool use, or long-context prompts, Claude Haiku 5.5 might be more suitable to optimize for latency"; effort `low` is the fastest setting ([support agent guide](https://platform.claude.com/docs/en/about-claude/use-case-guides/customer-support-chat)).
- Batch API: 50% off, asynchronous, most batches finish within 1 hour — fits nightly reminder copy, conversation summaries and eval runs, not live chat ([batch](https://platform.claude.com/docs/en/build-with-claude/batch-processing)).

**Building blocks**
- **Tool-calling loop**: Claude returns `stop_reason: "tool_use"` with `tool_use` blocks; the application executes them and returns `tool_result` blocks; repeat until a final answer. Client tools run in the operator's code (clinic data never has to leave the VPS except as tool results) ([tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)).
- **Strict tool use / structured outputs**: `strict: true` on a tool guarantees inputs match the JSON Schema via grammar-constrained sampling (Anthropic's own example is a booking system needing an integer); `output_config.format` with `json_schema` guarantees schema-valid JSON replies ([strict tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use), [structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)).
- **Prompt caching**: prefix order is `tools` → `system` → `messages`; default TTL 5 minutes (refreshed on each hit), optional 1-hour TTL at 2× write price; minimum cacheable prefix 512 tokens on the current models; caches are isolated per organization and per workspace ([prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)).
- **Conversation memory/state**: the Messages API is stateless — the application resends history. Options documented: server-side compaction (beta; summarises older turns), the client-side memory tool (files stored in the operator's own infrastructure), context editing ([compaction](https://platform.claude.com/docs/en/build-with-claude/compaction), [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool)).
- **Agent SDK vs Managed Agents vs plain API**: the Claude Agent SDK is "Claude Code as a library" (built-in file/shell tools, hooks, subagents, MCP, sessions) ([Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)); Managed Agents is a hosted harness in beta, "not eligible for ZDR", transcripts persist until deleted ([Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview)); the MCP connector (beta) attaches remote MCP servers to a Messages request ([MCP connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector)). For a WhatsApp CS agent with ~10 domain tools on the operator's VPS, the plain Messages API + tool loop is the lightest fit; the coding-oriented Agent SDK toolset is unnecessary **[INFERENCE]**.
- **Guardrails** (Anthropic guidance): let the model say "I don't know" and restrict it to supplied documents ([reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)); pre-screen user input with a lightweight model, and treat tool results/third-party text as untrusted data that must not override the system prompt ([jailbreaks and prompt injection](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)).
- **Evaluation targets** from Anthropic's support guide: escalation accuracy ≥95%, relevance ≥90% by LLM grading, 70–80% deflection as a typical aim; build code-graded, LLM-graded and human-graded evals before launch ([support agent guide](https://platform.claude.com/docs/en/about-claude/use-case-guides/customer-support-chat), [develop tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)).
- **Data handling**: Anthropic commits that retained API data "is never used for model training without your express permission"; Zero Data Retention and HIPAA-ready (BAA) arrangements exist on request; Fable/Mythos 5.x models require 30-day retention ([Anthropic data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)). Exact default retention period for standard API traffic was not extracted: **[UNVERIFIED]**.

**Other providers (official docs)**
- OpenAI: function tools defined by JSON Schema, model returns `function_call`, app returns `function_call_output`, `strict: true` recommended, parallel calls configurable ([function calling](https://developers.openai.com/api/docs/guides/function-calling)); prices e.g. `gpt-6-luna` USD 0.10 in / 0.01 cached / 0.50 out, `gpt-6.2-sol` USD 2 / 0.10 / 10 per 1M tokens ([OpenAI pricing](https://developers.openai.com/api/docs/pricing)).
- Google Gemini: same declare → `function_call` → `function_result` pattern with `auto/any/none/validated` modes ([function calling](https://aistudio.google.com/docs/function-calling)); prices e.g. `gemini-3.5-flash-lite` USD 0.30 in / 2.50 out, `gemini-3.8-flash` USD 0.75 / 3.75 through 31 Dec 2026 then doubling ([Gemini pricing](https://aistudio.google.com/docs/pricing)).
- All three expose the same loop, so the agent runtime can sit behind a provider-neutral tool interface **[INFERENCE]**.

**Five jobs → tools** **[INFERENCE: design mapping; platform facts cited in 5a]**

| Job | Agent tools (all tenant-scoped, strict schemas) | WhatsApp surface | Guardrail / handoff trigger |
|---|---|---|---|
| Qualify | `lookup_patient(phone)`, `create_lead(source, ref_code, referral)`, `get_clinic_profile()`, `get_price_list(service)`, `classify_intent()` | free-form service messages; list (≤10 rows) for service choice | pain, bleeding, swelling, trauma, pregnancy, medication, minors, "is this normal?" → scripted safety text + `handoff()`; never diagnose |
| Book | `get_availability(dentist, service, range)`, `hold_slot()`, `create_appointment()`, `reschedule()`, `cancel()` | reply buttons (≤3) for slots, or a Flow with CalendarPicker; booking-appointment template as confirmation | double-booking conflict, out-of-policy slots, deposit rules → human |
| Remind | `schedule_template(template, at, vars)`, `get_optout_status()` (called by scheduler, not the LLM) | utility template(s); buttons Confirm / Reschedule | any reply reopens the CSW → agent resumes with context |
| Follow up | `get_treatment_plan_status()` (non-clinical fields only), `send_template()` | utility for post-visit logistics; marketing for recall/offers | clinical complaint after treatment → immediate handoff to dentist/nurse |
| Close | `create_quote()`, `create_payment_link(amount, appointment)`, `get_payment_status()`, `mark_won(value)` | CTA-URL button to payment page | discounts beyond price list, instalment/insurance questions → owner/admin |
| Always | `handoff(reason, summary)`, `add_internal_note()`, `set_conversation_state()` | policy-required escalation path | human reply (dashboard or phone app echo) pauses the AI |

**Cost per conversation** — see the worked model under Implications.

### 5e. The "single dashboard": build vs adopt

| Project | Licence (file read) | What it gives | Limits for this product |
|---|---|---|---|
| **Chatwoot** | MIT for everything outside `enterprise/`; `enterprise/` needs a paid subscription per seat and may not be redistributed ([LICENSE](https://raw.githubusercontent.com/chatwoot/chatwoot/develop/LICENSE), [enterprise LICENSE](https://raw.githubusercontent.com/chatwoot/chatwoot/develop/enterprise/LICENSE)) | Shared inbox with assignment, labels, resolve; WhatsApp Cloud channel incl. "Quick setup with Meta" (Embedded Signup, supports coexistence), templates and 24-h window handling, campaigns ([WhatsApp channel guide](https://www.chatwoot.com/hc/user-guide/articles/1677832735-how-to-setup-a-whats_app-channel)); self-hosted ES needs the operator's own Meta app ID, configuration ID and secret ([ES config](https://developers.chatwoot.com/self-hosted/configuration/features/integrations/whatsapp-embedded-signup)); **Agent Bots**: webhook receives `message_created` etc., bot replies via API, new conversations start `pending`, bot sets `open` to hand to a human, agent sets `pending` to hand back ([agent bots](https://www.chatwoot.com/hc/user-guide/articles/1677497472-how-to-use-agent-bots)); Platform APIs (self-hosted only) create accounts/users programmatically ([API intro](https://developers.chatwoot.com/api-reference/introduction)) | Paid-only on self-hosted: Captain AI (OpenAI-based), voice calls, custom branding, agent capacity, **roles & permissions**, SSO, SLA, audit logs — USD 19 or 99 per agent/month ([self-hosted plans](https://www.chatwoot.com/pricing/self-hosted-plans)); WhatsApp calling needs paid EE ≥ v4.15.0 ([WhatsApp channel guide](https://www.chatwoot.com/hc/user-guide/articles/1677832735-how-to-setup-a-whats_app-channel)). Support for Flows, list/button messages and booking templates in the Chatwoot UI: **[UNVERIFIED]**. Bahasa Indonesia UI completeness: **[UNVERIFIED]** |
| **Zammad** | AGPL-3.0 ([LICENSE](https://raw.githubusercontent.com/zammad/zammad/develop/LICENSE)) | Ticketing helpdesk with a WhatsApp Cloud API channel ([Zammad admin docs](https://admin-docs.zammad.org/en/latest/channels/whatsapp/index.html)) | Ticket-centric; AGPL obliges offering modified source to network users; template/bot features **[UNVERIFIED]** |
| **n8n** | Sustainable Use License: use "only for your own internal business purposes"; may be provided to others only "free of charge for non-commercial purposes"; `.ee` files need an Enterprise licence ([LICENSE.md](https://raw.githubusercontent.com/n8n-io/n8n/master/LICENSE.md)) | Workflow automation, many connectors | Not licensable as a tenant-facing/white-labelled workflow builder inside a paid SaaS without a commercial (Embed) agreement **[INFERENCE from licence text]**; n8n's licence FAQ page 404'd in this session, so its own examples are **[UNVERIFIED]** |
| **Typebot** | FSL-1.1-Apache-2.0: any purpose except a "Competing Use" (offering the software, or substantially similar functionality, to others in a commercial product); each version becomes Apache-2.0 after two years ([LICENSE](https://raw.githubusercontent.com/baptisteArno/typebot.io/main/LICENSE)) | Visual chatbot flows deployable to WhatsApp with the user's own Meta app; 3-button limit handling; several blocks unsupported on WhatsApp ([Typebot WhatsApp docs](https://docs.typebot.com/deploy/whatsapp/overview)) | No inbox/handoff described; exposing its builder to tenants looks like a Competing Use **[INFERENCE]** |

What would still be custom whichever is adopted **[INFERENCE]**: the Meta Tech Provider app and tenant onboarding; the agent runtime and tool layer; clinic calendar/patient/price-list/payment domain; reminder scheduler and template lifecycle per tenant; lead attribution; the five clinic roles mapped onto inbox permissions (Chatwoot's granular roles are a paid feature); per-tenant WhatsApp cost metering from status webhooks; Bahasa Indonesia UX; audit trail of AI actions.

### 5f. Third-party integrations (official docs)

| Integration | Verified facts | Design note [INFERENCE] |
|---|---|---|
| **Google Calendar API** | Push channels for Events/CalendarList/ACL/Settings via `watch`; HTTPS with valid certificate required; notifications have **no body** — a follow-up API call is needed; channels expire and there is "no automatic way to renew"; "expect a small percentage of messages to get dropped" ([push](https://developers.google.com/workspace/calendar/api/guides/push)). Incremental sync with `syncToken`; HTTP 410 means wipe and full re-sync ([sync](https://developers.google.com/workspace/calendar/api/guides/sync)). Quotas: 10,000 requests/min per project, 600/min per user; 403/429 `usageLimits`; exponential backoff ([quota](https://developers.google.com/workspace/calendar/api/guides/quota)) | Keep the platform's own appointment table as source of truth; Google Calendar is a two-way mirror per dentist (push + periodic syncToken poll + channel-renewal job); availability checks must read the local table, never Google live |
| **Instagram DM** | Requires Instagram professional account and `instagram_business_manage_messages`; Advanced Access to serve accounts the app doesn't own; only reply after the user messages first; 24-hour standard window, 7-day human-agent window; docs use `graph.instagram.com/v27.0` ([Instagram messaging API](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api)) | Second channel in the same inbox; separate App Review scope; no templates, so reminders stay on WhatsApp |
| **Meta Lead Ads** | Leads via `leadgen` Page webhook (delay "up to a few minutes") or bulk read; needs `leads_retrieval`, `pages_manage_metadata`, `pages_show_list`, `pages_read_engagement`, `ads_management` ([retrieving leads](https://developers.facebook.com/docs/marketing-api/guides/lead-ads/retrieving)) | Lead form → create lead → send opt-in-covered utility/marketing template to start the WhatsApp thread |
| **Conversions API for Business Messaging** | `action_source: business_messaging`, `messaging_channel: whatsapp`, `user_data` = `whatsapp_business_account_id` + `ctwa_clid`; events include `LeadSubmitted`, `QualifiedLead`, `Purchase`; one dataset per Page, owned by the client ([CAPI business messaging](https://developers.facebook.com/docs/marketing-api/conversions-api/business-messaging)); Meta's WhatsApp app-review page describes `whatsapp_business_manage_events` as the permission for logging such events ([app review](https://developers.facebook.com/documentation/business-messaging/whatsapp/solution-providers/app-review)) | "Closing" attribution for clinics that run CTWA ads: emit QualifiedLead at booking and Purchase at payment; only meaningful when a `ctwa_clid` exists |
| **Google Business Profile** | Chat and call history were removed on 31 Jul 2024; Google suggests WhatsApp/SMS contact where eligible and Reserve with Google in some countries ([GBP chat change](https://support.google.com/business/answer/14919056?hl=en)). Owners can add up to 10 booking links per category; third-party provider links can appear automatically ([GBP booking links](https://support.google.com/business/answer/6218037)) | No messaging API to integrate; put the clinic's WhatsApp link and public booking URL on the profile. Reserve with Google availability for Indonesian dental clinics: **[UNVERIFIED]** |
| **SMS fallback** | Twilio lists Indonesia outbound SMS at USD 0.4414 per segment (Nov 2026) ([Twilio SMS ID](https://www.twilio.com/en-us/sms/pricing/id)) | ~17× a WhatsApp utility template (0.4414 / 0.025) — use only for delivery failures on critical reminders |
| **Email fallback** | not researched in this track | **[UNVERIFIED]** — low priority for patient reminders |

### 5g. Evidence for the reminder cadence

- **Cochrane review** (Gurol-Urganci et al., *Cochrane Database Syst Rev* 2013;(12):CD007458; 8 RCTs, 6,615 participants): text reminders vs no reminder increased attendance, RR 1.14 (95% CI 1.03–1.26), moderate-quality evidence; text vs phone-call reminders similar, RR 0.99 (0.95–1.02); attendance 67.8% (none), 78.6% (text), 80.3% (call); text cost per attendance 55% and 65% lower than calls in two studies; authors call the evidence "insufficient to conclusively inform policy decisions" and ask future trials to report message content and timing ([PubMed record, PMID 24310741](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=24310741&rettype=abstract&retmode=text)).
- **Meta-analysis** (Robotham et al., *BMJ Open* 2016; 26 studies): notified patients more likely to attend, RR 1.23 (1.10–1.38), and 25% less likely to no-show, RR 0.75 (0.68–0.82); **multiple notifications** raised attendance more than single ones (78% vs 52% across 5 studies, against 62% vs 55% for single across 8); nine studies sent reminders ≤48 h before the appointment, three earlier; no pooled conclusion on optimal timing ([PMC5093388](https://pmc.ncbi.nlm.nih.gov/articles/PMC5093388/)).
- **Dental RCT, India** (Prasad & Anand, *Int Dent J* 2012;62(1):21-6; n=206, four dental departments): on-time attendance 79.2% with SMS reminder vs 35.5% control ([PMID 22251033](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=22251033&rettype=abstract&retmode=text)). Exact send times are not in the abstract: **[UNVERIFIED]**.
- **Paediatric dental RCT, USA** (Nelson et al., *J Am Dent Assoc* 2011;142(4):397-405; n=318): no-show 17.7% with SMS vs 8.2% with voice message (P=.01) — text was *less* effective than a voice reminder in a university paediatric clinic ([PMID 21454845](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=21454845&rettype=abstract&retmode=text)).
- **Public dental service, Australia** (Stormon et al., *Health Soc Care Community* 2022; 118,266 appointments, before/after): adult attendance rose from 73.5 to 77.7 per 100; in the child service attendance fell 4% — mixed effect ([PMID 34850473](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=34850473&rettype=abstract&retmode=text)).
- Caveats: all are SMS/voice, not WhatsApp; none is Indonesian; PubMed/PMC HTML pages were bot-walled, so three papers were read as abstracts through NCBI's E-utilities endpoint and full-text details were not checked.

### Implications for the workspace platform

All items **[INFERENCE]**, each tied to facts cited above.

1. **Recommended onboarding path: become a Meta Tech Provider and run Embedded Signup yourself; add a Solution Partner only if billing forces it.**
   - Sequence: verify the operator's legal entity → build prototype → App Review for the two WhatsApp permissions (+ `whatsapp_business_manage_events` if CAPI is planned) → Access Verification → ES v4/v5 with coexistence enabled. Plan 3–6 weeks of calendar time for Meta steps (Meta says ~24 h per App Review; Twilio's ISV guide says 3–4 weeks in practice) (5b).
   - Each clinic owns its WhatsApp account and Messaging account and pays Meta directly in IDR; the platform never fronts message fees (5b). This matches a directory operator that should not carry credit risk for hundreds of clinics.
   - Pilot ceiling is 10 new clinics per rolling 7 days until verifications complete, then 200 (5b) — pace sales accordingly.
   - Fallback: if clinics cannot attach an accepted payment method in Billing Hub (open question), pair the same Meta app with a Solution Partner through a Multi-Partner Solution (e.g. 360dialog Growth: €500 + €25 per channel beyond 10) and re-bill.
   - Do not build on WhatsApp-Web libraries: the ban risk sits on the clinic's number (5c), and coexistence already lets staff keep the phone app (5a).

2. **Default tenant setup = coexistence.** Clinics keep their existing WhatsApp Business app number; staff replies from the phone remain free and outside the 24-h rule, while the API handles AI replies and templates (5a). Product consequences: use `smb_message_echoes` as the "human took over" signal; run history sync immediately (24-hour limit); warn about broadcast lists and linked devices during signup; no blue tick and no Calling API for these numbers (voice stays in the phone app, unlogged).

3. **AI positioning must stay "the clinic's own customer service".** Scope the agent to clinic services, schedule, prices and logistics; refuse general-assistant use; never train or fine-tune shared models on tenant chats; expose a human-handoff path in every conversation (policy requirement) (5c). Meta's own Business Agent is not an option for health-vertical numbers, so a third-party LLM agent is the only route — and is explicitly priced as "service" messages (5a, 5c).

4. **Keep clinical data out of WhatsApp payloads.** Templates and AI replies carry logistics only (date, time, dentist, address, amount); medical history, images, diagnoses and ID numbers go through a secure web link to the platform, not through chat or Flows (policy on health information and sensitive identifiers; platform "not HIPAA compliant") (5c). Final rule depends on the Indonesian-regulation track.

5. **Booking UX: start conversational, add Flows per verified clinic.** Reply buttons/list messages work for every tenant on day one; Flows with CalendarPicker need the clinic's Meta business verification, which solo dentists may not pass, plus a per-tenant encryption key and a fast availability endpoint (5a). Use the booking-appointment template for confirmations with a manage-booking URL.

6. **Template catalogue per tenant, seeded by the platform.** Utility: booking confirmation, reminder(s), reschedule/cancel confirmation, post-visit logistics, opt-in confirmation. Marketing (opt-in + stop control): recall ("time for your check-up"), promotions, reactivation — Meta's retargeting rule makes recall marketing in most wordings (5a). Never mix a promo into a reminder (auto-recategorised to marketing at 1.6× the price).

7. **Reminder cadence supported by evidence: at least two reminders, with a confirm/reschedule action.** Multiple notifications outperform single ones; ≤48 h is the commonest timing; one dental RCT found voice better than text, so add a human call (or WhatsApp user-callable number) for unconfirmed high-value appointments (5g). Suggested default: T-48 h utility template with Confirm/Reschedule buttons, T-3 h short reminder; any reply reopens the free-form window.

8. **Messaging limits are not a constraint; quality is.** 250 unique business-initiated recipients per 24 h covers a clinic's reminders even before verification (5a). The real risks are blocks/reports (opt-in hygiene, per-user marketing caps) and template misclassification.

9. **Directory attribution.** Each "Chat on WhatsApp" button on the directory should hit a first-party redirect that logs the click and forwards to `wa.me/<clinic>?text=…` containing a short reference code; parse it on the first inbound message; fall back to click-time matching when the patient edits the text. Paid-ad leads use the `referral` object and CAPI (5a, 5f).

10. **Dashboard: adopt the pattern, weigh adopting the code.** Chatwoot's MIT core gives inbox + WhatsApp channel + agent-bot handoff + Platform APIs, but the five-role model, branding and AI features sit behind per-agent enterprise pricing, and dental-specific objects are absent (5e). Either embed Chatwoot CE headlessly behind the platform's own UI, or build a narrower inbox natively; in both cases the agent runtime, scheduler, onboarding and metering are custom. Avoid n8n/Typebot as tenant-facing components on licence grounds; internal operator automations are a separate question (5e).

11. **Rough per-clinic monthly cost model** (variable costs only; list prices in 5a and 5d).

   Assumptions: 1 number; 300 new enquiry conversations/month from organic `wa.me` links (no FEP); 12 business replies per conversation, all via API; 400 appointments/month; per appointment 1 confirmation + 2 reminders + 1 post-visit message (utility); 300 recall/promo marketing templates; Meta's implied rate USD 1 ≈ IDR 14,266 (356.65 / 0.025); per conversation 15 model calls, each with 4,000 cached prefix tokens, 2,000 uncached input tokens, 300 output tokens (incl. thinking); one cache write per conversation.

   | Line | Volume | Unit | Monthly |
   |---|---|---|---|
   | Service messages | 3,600 − 1,000 free = 2,600 | IDR 356.65 | IDR 927,290 (USD 65.00) |
   | Utility templates | 400 × 4 = 1,600 | IDR 356.65 | IDR 570,640 (USD 40.00) |
   | Marketing templates | 300 | IDR 586.33 | IDR 175,899 (USD 12.33) |
   | **Meta total** | | | **≈ IDR 1.67 million (USD 117)** |
   | LLM, Haiku 5.5 | 300 conv × ~USD 0.006 | | ≈ USD 2 |
   | LLM, Sonnet 5.5 | 300 conv × ~USD 0.12 | | ≈ USD 36 |
   | LLM, Opus 5.5 | 300 conv × ~USD 0.24 | | ≈ USD 73 |

   - Per-call arithmetic (Sonnet 5.5): 4,000 × 0.10 + 2,000 × 2.00 + 300 × 10.00 per million = USD 0.0074; × 15 + cache write 0.01 ≈ USD 0.12. Same method for the other models.
   - Reading: **Meta fees dominate**, and since 1 Oct 2026 the largest line is free-form replies, not templates. Levers: fewer, denser AI messages (one message with buttons instead of three bubbles); staff replies from the coexistence phone app are free; CTWA-sourced conversations are free for up to 7 days; the 1,000 free service messages are per number, so a second number doubles the allowance.
   - Token counts for Bahasa Indonesia on the current tokenizer are assumptions; measure with the token-counting endpoint before pricing plans. Platform-level BSP fees, if used, add roughly €25–49 per clinic (360dialog) or ~USD 45 per clinic at ~9,000 messages in+out (Twilio).
   - Keep the system prompt + tool definitions byte-identical across tenants and put tenant facts after the cache breakpoint, so the shared prefix stays warm across all clinics despite the 5-minute TTL (5d).

12. **Model routing.** Use a small model (Haiku-class) as an always-on screen/intent router and for summaries; a mid model (Sonnet-class) for the tool-using conversation; reserve larger models for evals and hard cases. All three providers expose the same tool loop, so keep the tool layer provider-neutral (5d).

### Open questions

- Which payment methods can an Indonesian clinic attach to its Messaging account in Billing Hub (local debit cards, virtual accounts)? Determines whether a Solution Partner credit line is needed.
- Does Meta require any registration or disclosure for AI-operated numbers ("AI-enabled phone numbers" page was 404), and is there an expected in-chat AI disclosure?
- Does Indonesian health-data law make the Business Messaging Policy's health-information clause apply (i.e., may appointment details naming a treatment be sent over WhatsApp)? Owned by the regulation track.
- How many target clinics (especially solo practitioners) can pass Meta Business Verification — needed for Flows, the 2,000 tier and business-initiated calling?
- How will Meta categorise recall messages ("6-month check-up due") — utility or marketing? Test with real template submissions.
- Is Indonesia a supported Cloud API local-storage region, and is that needed for data-residency compliance?
- Calling: coexistence numbers cannot use the Calling API (5a) — is a second, API-only number per clinic worth it for call logging/recording, and what is the IDR per-minute rate for Indonesia?
- Is Indonesia an "eligible country" for Meta Business Agent (moot for health today, relevant if the vertical rule changes)?
- Chatwoot CE: are Flows, list/button messages and booking templates sendable from the stock UI/API, and how complete is its Bahasa Indonesia locale?
- Real Bahasa Indonesia token counts and tool-call counts per conversation, from a pilot, to replace the cost-model assumptions.
- Mekari Qontak and Kata.ai current pricing and partner terms (pages not machine-readable in this session).


---

## 6. Platform architecture on a self-managed VPS

Research date 2026-11-25. Tags: **[UNVERIFIED]** = not confirmed at a primary source in this session; **[INFERENCE]** = my reasoning or recommendation. Local-repo references are relative to `the folder that holds the three repositories`.

### a. TanStack Start outside Cloudflare

- **Status and version.** The docs overview states "TanStack Start is stable" and links a migration guide from the release candidate ([Start overview](https://tanstack.com/start/latest/docs/framework/react/overview)). Latest tagged release: `@tanstack/react-start` 1.169.7 on 2026-11-19 ([router releases](https://github.com/TanStack/router/releases)); `@tanstack/react-router` is 1.170.46, MIT ([npm](https://registry.npmjs.org/@tanstack/react-router/latest)).
- **Node target.** With Vite, add the Nitro plugin (`plugins: [tanstackStart(), nitro(), viteReact()]`), then `vite build` and `node .output/server/index.mjs`; the docs note the Nitro Vite plugin "is still under active development" ([Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)). An Rsbuild build instead emits `dist/server/index.js` exporting a `fetch(request)` handler, served with `srvx`, and "Express or any other custom Node.js server works too" ([Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)).
- **Docker.** The section is titled "Node.js / Docker" but contains no Dockerfile or container-specific instructions ([Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)); the image is therefore ours to write (multi-stage Node build that copies `.output/`) **[INFERENCE]**.
- **Bun.** Bun deployment guidelines "only work with React 19"; use `nitro({ preset: 'bun' })` or a custom `server.ts` copied from the example repo ([Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)).
- **Server functions.** `createServerFn()` defaults to GET, `{ method: 'POST' }` for mutations, `.validator()` before `.handler()`; they are "same-origin RPC endpoints"; Start provides `createCsrfMiddleware()` to block cross-site calls; every function touching private data needs its own auth check because route `beforeLoad` guards "are not a security boundary" ([server functions](https://tanstack.com/start/latest/docs/framework/react/guide/server-functions)).
- **Middleware.** Request middleware covers server routes, SSR and server functions; function middleware adds `.client()` and `.validator()`; global middleware is registered in `src/start.ts` via `createStart`; CSRF protection is installed by default only while that file does not exist, after which `createCsrfMiddleware()` must be added by hand ([middleware](https://tanstack.com/start/latest/docs/framework/react/guide/middleware)).
- **SSR/streaming.** Full-document SSR and streaming are listed core features ([Start overview](https://tanstack.com/start/latest/docs/framework/react/overview)); server functions can stream a typed `ReadableStream` or an `async function*` generator ([streaming guide](https://tanstack.com/start/latest/docs/framework/react/guide/streaming-data-from-server-functions)).
- **Server routes.** Defined with a `server` property on `createFileRoute`; handlers return a `Response` ([server routes](https://tanstack.com/start/latest/docs/framework/react/guide/server-routes)). This is where webhooks (WhatsApp) and the auth handler mount.
- **SSE / WebSockets.** None of the Start pages opened (server routes, server functions, streaming) mention SSE or WebSockets. The underlying Nitro server documents both: WebSockets via CrossWS with `features: { websocket: true }` and `defineWebSocketHandler`, and SSE via `createEventStream` as the "simpler alternative when you only need server-to-client streaming" ([Nitro WebSocket](https://nitro.build/docs/websocket)). Returning a `text/event-stream` `Response` from a Start server route should work because handlers return standard `Response` objects **[INFERENCE]**; whether Nitro WebSocket route files can be co-hosted inside a Start build is **[UNVERIFIED]**.
- **Base path (`/workbuddy`).**
  - Vite: `base` rewrites JS-imported asset URLs, CSS `url()` and HTML asset references; `import.meta.env.BASE_URL` exposes it ([Vite build](https://vite.dev/guide/build.html)).
  - Router: `basepath` (default `/`) "is useful for mounting a router instance at a subpath" ([RouterOptions](https://tanstack.com/router/latest/docs/framework/react/api/router/RouterOptionsType)); the router implements it internally as a rewrite applied on client and server ([URL rewrites](https://tanstack.com/router/latest/docs/framework/react/guide/url-rewrites)).
  - Start: issue #4888 "[start] Incomplete basepath handling" (opened 2025-08-08, still open, last activity 2026-07-15) records that Vite `base` and router `basepath` must both be set, that server file routes did not receive the basepath automatically in the original report, and that Start documentation for this is still outstanding ([router#4888](https://github.com/TanStack/router/issues/4888)).
  - Options: (1) `/workbuddy` as a route subtree of the same app (no base path at all); (2) a second Start app built with `base: '/workbuddy/'` + `basepath: '/workbuddy'`, routed by the reverse proxy. Option 1 avoids the open base-path issue and keeps one session cookie and one deploy; option 2 buys independent release cadence at the cost of the caveats above **[INFERENCE]**.

### b. Data layer (PostgreSQL)

- **Version.** PostgreSQL docs "current" is 19.0; the site banner reads "November 19, 2026: PostgreSQL 19 Released!" ([RLS docs](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)).

| Tenancy model | What the PostgreSQL docs say | Consequence for this product **[INFERENCE]** |
|---|---|---|
| Shared schema + RLS | With RLS enabled "all normal access … must be allowed by a row security policy"; no policy = default-deny; superusers, `BYPASSRLS` roles and (unless `FORCE ROW LEVEL SECURITY`) table owners bypass; `USING` filters reads, `WITH CHECK` validates writes; permissive policies OR together, restrictive AND ([RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)) | One migration set, one pool; isolation depends on the app role being non-owner, non-superuser, and on every table carrying `tenant_id` |
| Schema per tenant | "Unlike databases, schemas are not rigidly separated: a user can access objects in any of the schemas … if they have privileges"; resolution via `search_path` ([schemas](https://www.postgresql.org/docs/current/ddl-schemas.html)) | N× migrations; `search_path` switching per request; cross-tenant reporting needs unions |
| Database per tenant | Databases "are isolated from each other"; "not possible to access more than one database per connection"; recommended for "unrelated projects or users"; roles are cluster-level ([databases](https://www.postgresql.org/docs/current/manage-ag-overview.html)) | Strongest isolation and per-tenant restore, but a pool per tenant and no cross-tenant queries; reserve for hospital customers that contractually require it |

- **RLS caveats.** Unique/FK checks "always bypass row security" (covert-channel risk); policies that sub-select other tables can race under `READ COMMITTED`; policies should reference only the current row where possible ([RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)).
- **Drizzle.** Latest stable 0.45.4; 1.0.0-rc.6 (Nov 20) is a pre-release ([drizzle releases](https://github.com/drizzle-team/drizzle-orm/releases)). RLS: `pgTable.withRLS(...)` (v1; `.enableRLS()` deprecated), `pgPolicy` with `as/to/for/using/withCheck`, `pgRole` (+ `.existing()`), role management opt-in via `entities.roles`; the per-request `set_config` + `set local role` transaction wrapper shown is a community example, not core API ([Drizzle RLS](https://orm.drizzle.team/docs/rls)). Migrations: `generate` + `migrate` (SQL files + history table), `push`, `pull`, or a runtime `migrate(db)` ([Drizzle migrations](https://orm.drizzle.team/docs/migrations)).
- **Alternative: Prisma.** The official RLS client-extension example wraps each query in a transaction with `set_config(..., TRUE)` but states "This extension is provided as an example only. It is not intended to be used in production environments," and explicit `$transaction()` "may not work as intended" ([prisma-client-extensions RLS](https://github.com/prisma/prisma-client-extensions/tree/main/row-level-security)).
- **Double-booking.** Exclusion constraints guarantee that no two rows satisfy all listed operators; implemented with GiST/SP-GiST; a `predicate` makes it partial ([CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html)). The canonical pattern is `EXCLUDE USING GIST (room WITH =, during WITH &&)` with `btree_gist`; range constructors default to `[)` bounds ([range types](https://www.postgresql.org/docs/current/rangetypes.html)). `UNIQUE (id, valid_at WITHOUT OVERLAPS)` "behaves like `EXCLUDE USING GIST (id WITH =, valid_at WITH &&)`" ([CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html)). Two constraints (dentist × time, chair × time), each `WHERE status <> 'cancelled'`, on a `tstzrange` column **[INFERENCE]**; whether drizzle-kit can express `EXCLUDE` natively or needs a hand-written SQL migration is **[UNVERIFIED]**.
- **Audit logging.** pgAudit provides session logging (`pgaudit.log` classes READ, WRITE, FUNCTION, ROLE, DDL, MISC) and object logging via `pgaudit.role`, written to the standard PostgreSQL log; versions track PG majors (v19.X ↔ PG 19, v18.X ↔ PG 18). Limits: "Audit logging is best-effort and not transactional," "It is not possible to reliably audit superusers" ([pgaudit](https://github.com/pgaudit/pgaudit)). For "who opened which patient record", an application-level `audit_event` table (INSERT/SELECT-only grants, written in the same transaction as the access) is the transactional trail; pgAudit covers DDL, role changes and out-of-band SQL **[INFERENCE]**.
- **Encryption.** PostgreSQL documents column encryption via pgcrypto (key and plaintext "present on the server for a brief time"), data-partition encryption at filesystem/block level (dm-crypt + LUKS; no protection while mounted), TLS in transit, and client-side encryption when the DBA cannot be trusted; no built-in TDE is described ([encryption options](https://www.postgresql.org/docs/current/encryption-options.html)). pgcrypto is a trusted extension; "all the data and passwords move between pgcrypto and client applications in clear text", so use SSL/local connections ([pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html)).
- **Backup / PITR.** Continuous archiving needs `wal_level=replica`, `archive_mode=on`, an `archive_command`/`archive_library`, and a base backup; it restores "an entire database cluster, not a subset"; "`archive_timeout` settings of a minute or so are usually reasonable" ([continuous archiving](https://www.postgresql.org/docs/current/continuous-archiving.html)).

| Tool | Version / license | Notable |
|---|---|---|
| pgBackRest | v2.60.1 (2026-11-16), MIT ([pgbackrest.org](https://pgbackrest.org/)) | Full/diff/incremental incl. block-level, parallel, S3/Azure/GCS/SFTP repos, repo encryption, multiple repositories, backup from standby, resume; sponsorship-funded ("would not exist without sponsorship") ([pgbackrest.org](https://pgbackrest.org/)) |
| WAL-G | Apache-2.0 (LZO part GPL-3.0+) ([wal-g](https://github.com/wal-g/wal-g)) | S3/GCS/Azure/Swift/file/SSH; libsodium or OpenPGP encryption ([wal-g](https://github.com/wal-g/wal-g)) |
| Barman | GPL-3.0, maintained by EnterpriseDB ([barman](https://github.com/EnterpriseDB/barman)) | Remote backups of multiple servers from a central host ([barman](https://github.com/EnterpriseDB/barman)); PITR/WAL-streaming/`barman-cloud` details **[UNVERIFIED]** (docs site did not render) |

- **Connection pooling.** PgBouncer (latest 1.26.1): in transaction pooling `SET/RESET`, `LISTEN`, session advisory locks and SQL `PREPARE` are "Never" supported; protocol-level prepared plans work only with `max_prepared_statements` > 0 ([PgBouncer features](https://www.pgbouncer.org/features.html)). Transaction-local `set_config(..., true)` for RLS context should be safe under transaction pooling, but the page does not address `SET LOCAL` **[UNVERIFIED]**. Realtime `LISTEN` needs a direct, non-pooled connection **[INFERENCE]**.

### c. Authentication, SSO with the Firebase-backed main site, authorization

**Better Auth** (v1.7.28, MIT — [npm](https://registry.npmjs.org/better-auth/latest); "framework-agnostic authentication (and authorization) framework for TypeScript" — [repo](https://github.com/better-auth/better-auth)):

| Capability | Finding |
|---|---|
| TanStack Start | Handler mounted at `src/routes/api/auth/$.ts` calling `auth.handler(request)`; `tanstackStartCookies()` must be the last plugin; sessions read with `auth.api.getSession({ headers })` in server functions ([TanStack integration](https://www.better-auth.com/docs/integrations/tanstack)) |
| Database | Direct `pg` Pool, or Drizzle/Prisma adapters; `generate` emits schema/SQL ([installation](https://www.better-auth.com/docs/installation)) |
| Organizations | `organization()` plugin: organizations, members, invitations, teams; `activeOrganizationId` on the session; default roles owner/admin/member; a user can hold multiple roles ([organization](https://www.better-auth.com/docs/plugins/organization)) |
| Custom roles | `createAccessControl(statements)` + `ac.newRole(...)`; `dynamicAccessControl` stores runtime roles in `organizationRole`; checks via `auth.api.hasPermission` ([organization](https://www.better-auth.com/docs/plugins/organization)) |
| Cross-subdomain cookies | `crossSubDomainCookies` with a root `domain`; docs advise "the most specific scope needed" and "Be cautious of untrusted subdomains" ([cookies](https://www.better-auth.com/docs/concepts/cookies)) |
| 2FA | TOTP, OTP (own `sendOTP`), backup codes, 30-day trusted device; OAuth/passkey/magic-link sign-ins "are not gated by 2FA by default"; passwordless users need `allowPasswordless: true` to enrol ([2FA](https://www.better-auth.com/docs/plugins/2fa)) |
| Passkeys | `@better-auth/passkey`, built on SimpleWebAuthn; `rpID`, `rpName`, `origin`; conditional UI ([passkey](https://www.better-auth.com/docs/plugins/passkey)) |
| SSO (as relying party) | `@better-auth/sso`: OIDC + SAML 2.0, per-organization providers, domain-based sign-in, organization provisioning ([SSO](https://www.better-auth.com/docs/plugins/sso)); Generic OAuth plugin for any OAuth2/OIDC provider via `discoveryUrl` ([generic OAuth](https://www.better-auth.com/docs/plugins/generic-oauth)) |
| As identity provider | `@better-auth/oauth-provider`: OAuth 2.1 + OIDC Core authorization server (auth-code, client credentials, refresh, PKCE S256, dynamic client registration, consent, UserInfo, back-channel logout); stability level not stated on the page ([OAuth provider](https://www.better-auth.com/docs/plugins/oauth-provider)) |
| Admin | `admin` plugin: create/list users, set role, ban, revoke sessions, impersonate (default 1 h, cannot impersonate admins by default) ([admin](https://www.better-auth.com/docs/plugins/admin)) |
| Audit hooks | `hooks.before/after` via `createAuthMiddleware` with `ctx.path`, `ctx.context.newSession`, `runInBackground` ([hooks](https://www.better-auth.com/docs/concepts/hooks)); `databaseHooks` on user/session/account create/update ([database](https://www.better-auth.com/docs/concepts/database)). No built-in audit log was found on these pages; it is written from hooks **[INFERENCE]** |

**Self-hosted identity servers:**

| Server | License | Footprint / notes |
|---|---|---|
| Keycloak | Apache-2.0 ([repo](https://github.com/keycloak/keycloak)) | SSO, identity brokering (OIDC/SAML), LDAP/AD federation, admin console, fine-grained authorization ([keycloak.org](https://www.keycloak.org/)); Java service with its own database **[INFERENCE]** |
| Zitadel | AGPL-3.0-only; `proto/` and docs Apache-2.0; login UI and client packages MIT; commercial licence on request ([LICENSING.md](https://raw.githubusercontent.com/zitadel/zitadel/main/LICENSING.md)) | Organizations (B2B multi-tenancy), OIDC, SAML, passkeys, event-sourced audit trail, PostgreSQL ≥ 14 ([repo](https://github.com/zitadel/zitadel)); ~512 MB RAM, 4 cores recommended for password hashing ([production guide](https://zitadel.com/docs/self-hosting/manage/production)) |
| authentik | Core MIT; `authentik/enterprise` under a separate EE licence ([repo](https://github.com/goauthentik/authentik)) | SAML, OAuth2/OIDC, LDAP, RADIUS ([repo](https://github.com/goauthentik/authentik)); ≥ 2 cores / 2 GB RAM; docs version 2026.8 ([compose install](https://docs.goauthentik.io/install-config/install/docker-compose/)) |

- A separate IdP adds a second stateful service, database and upgrade track for one application; it pays off only when several products must share one login or customers demand SAML at scale **[INFERENCE]**.

**How the main site signs users in (local repo, auth-related parts only):**

- Stack: React Router 7.18.4 + Express 5 with `firebase` 12.19.0 and `firebase-admin` 14.5.0 (`findyourdent/apps/web/package.json:20-36`), served from Cloud Run `asia-southeast2` behind Firebase Hosting (`findyourdent/firebase.json:5-12`).
- Google is the only sign-in provider (`findyourdent/apps/web/server/auth.ts:6-7`); the browser runs `signInWithPopup(auth, new GoogleAuthProvider())` and keeps no Firebase session of its own (`findyourdent/apps/web/app/lib/firebase.client.ts:44-57`).
- The ID token is POSTed to `/auth/session` (`findyourdent/apps/web/app/routes/auth.session.ts:5-6`), verified, rejected if `auth_time` is older than 5 minutes, and exchanged for a 5-day Firebase session cookie (`findyourdent/apps/web/server/auth.ts:9,32-41`); each request verifies it with `verifySessionCookie(value, true)` (`findyourdent/apps/web/server/auth.ts:60`).
- The session cookie value is stored inside a single `__session` cookie (path `/`, `SameSite=Lax`, `HttpOnly`, no `Domain` attribute, i.e. host-only) because Firebase Hosting strips every other cookie (`findyourdent/apps/web/server/browser-state.ts:4-8,20-25`); Firebase confirms "Only the specially-named `__session` cookie is permitted to pass through" ([Hosting cache](https://firebase.google.com/docs/hosting/manage-cache)).
- Admins are an email allowlist, not roles or custom claims (`findyourdent/apps/web/server/auth.ts:69-72`).

**Where provider listing data lives:**

- Firestore, reached only through the web service with `firebase-admin`; client rules deny everything (`findyourdent/firestore.rules:2-7`), same for Storage (`findyourdent/storage.rules:2-6`).
- Collections: `providers`, `testimonies`, `claims`, `priceLists`, `moderationDecisions`, … (`findyourdent/packages/contracts/src/firestore.ts:23-34`). A listed provider is `providers/{placeId}` (Google place id as document id) with `claimStatus`, `claimId`, `ownerFacts`, `pendingOwnerFacts`, `aggregates` (`findyourdent/packages/contracts/src/firestore.ts:112-138`); owner-editable facts are name, type, region, address, phone, WhatsApp, website, opening hours, services (`findyourdent/packages/contracts/src/firestore.ts:78-92`).
- Ownership = an approved `claims/{claimId}` whose `claimantUid` is the Firebase uid (`findyourdent/packages/contracts/src/firestore.ts:254-273`; check in `findyourdent/apps/web/app/server/claims.server.ts:134-145`).
- Owner writes already have HTTP endpoints (`api/providers/:placeId/claims`, `/owner-facts`, `/price-lists` — `findyourdent/apps/web/app/routes.ts:16-19`); owner facts pass AI moderation and are published or held as `pendingOwnerFacts` (`findyourdent/apps/web/app/server/claims.server.ts:325-372`). Paid/owner routes also require an App Check token (`findyourdent/apps/web/app/server/protection.server.ts:23,89-92`).

**Firebase building blocks (official docs):**

- Session cookies: JWTs "generated using authorized service accounts", lifetime 5 minutes–2 weeks, same claims as the ID token; verification "requires the Admin SDK to be initialized with a project ID" and works from cached public certs; `checkRevoked` adds a network request ([manage cookies](https://firebase.google.com/docs/auth/admin/manage-cookies)).
- ID-token verification: Admin SDK `verifyIdToken()`, or any JWT library — `alg` RS256, `kid` from Google's `securetoken` certs, `aud` = project ID, `iss` = `https://securetoken.google.com/<projectId>`, `sub` = uid ([verify ID tokens](https://firebase.google.com/docs/auth/admin/verify-id-tokens)). The issuer's discovery document exposes `jwks_uri` and `response_types_supported: ["id_token"]` but no authorization or token endpoint ([securetoken discovery, placeholder project](https://securetoken.google.com/example-project/.well-known/openid-configuration)), so Firebase Auth cannot act as a redirect-based OIDC provider for the workspace **[INFERENCE]**.
- Custom tokens: server-minted JWTs, expire after one hour, signed by a service-account key or via IAM `signBlob`; client calls `signInWithCustomToken()` ([custom tokens](https://firebase.google.com/docs/auth/admin/create-custom-tokens)).
- Admin SDK outside Google: `GOOGLE_APPLICATION_CREDENTIALS` pointing at a service-account key file; "Extremely high security awareness is required when working with service account keys" ([admin setup](https://firebase.google.com/docs/admin/setup)).
- Authorized domains: a new origin must be added under Authentication → Settings → Authorized domains; a custom `authDomain` also needs `https://<domain>/__/auth/handler` whitelisted with the OAuth provider ([Google sign-in](https://firebase.google.com/docs/auth/web/google-signin)). `signInWithRedirect` breaks under third-party-storage blocking unless `authDomain` matches the app domain or auth requests are proxied; `signInWithPopup` is the listed alternative ([redirect best practices](https://firebase.google.com/docs/auth/web/redirect-best-practices)).
- Custom claims: `setCustomUserClaims`, ≤ 1000 bytes, propagate on next token issue, "only used to provide access control" ([custom claims](https://firebase.google.com/docs/auth/admin/custom-claims)).
- Identity Platform: Firebase can be a relying party to an external OIDC provider only in "upgraded projects" (code flow recommended) ([Firebase OIDC](https://firebase.google.com/docs/auth/web/openid-connect)); multi-tenancy gives per-tenant user silos and providers within one project ([Identity Platform multi-tenancy](https://docs.cloud.google.com/identity-platform/docs/multi-tenancy)).

**SSO options between the Firebase main site and the Postgres workspace** (all rows **[INFERENCE]** built on the facts above):

| # | Option | How | Pros | Cons |
|---|---|---|---|---|
| S1 | Same identity, separate sessions | Workspace runs the Firebase JS popup (its domain added to Authorized domains), posts the ID token to its own server, verifies it (project ID + Google public keys; no service account), then creates a Better Auth session keyed by Firebase `uid` | No change to the main site; no secret on the VPS; uid matches `claims.claimantUid` | One extra Google click; two logouts; staff without Google accounts need a second method (email + password/passkey) |
| S2 | Shared parent-domain cookie | Main site sets `__session` with a parent `Domain`; workspace parses it and verifies the Firebase session cookie | True SSO | Main-site cookie becomes readable by every subdomain; workspace must track the main site's cookie format; revocation check from the VPS needs Google credentials **[UNVERIFIED]**; couples release cycles |
| S3 | Redirect hand-off | Workspace redirects to a main-site endpoint; main site (already signed in) returns a short-lived signed assertion or a Firebase custom token; workspace exchanges it for its own session | Seamless; cookies stay host-only; explicit, auditable trust | New endpoint + signing key on the main site; replay/redirect hardening is on us |
| S4 | Standards OIDC with one IdP | Workspace (Better Auth OAuth provider) or Zitadel/Keycloak becomes the IdP; main site consumes it via Identity Platform OIDC | Cleanest long-term for many apps | Requires the Identity Platform upgrade, changes the main site's "Google only" rule and uid continuity; heaviest |
| S5 | Google directly | Workspace uses Better Auth's Google provider ("sign in with ID token" supported — [Better Auth Google](https://www.better-auth.com/docs/authentication/google)) and links to the Firebase uid by Google account id/email | No Firebase runtime dependency | Mapping Firebase uid ↔ Google subject via `firebase.identities` is **[UNVERIFIED]** (the reference only types `identities` as an object — [DecodedIdToken](https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.decodedidtoken)) |

**Data-integration options** (**[INFERENCE]** unless cited):

| # | Option | Pros | Cons |
|---|---|---|---|
| D1 | Admin SDK from the VPS (direct Firestore read/write) | Fastest to build; server-side realtime listeners exist (`onSnapshot` in the Node client — [Firestore listen](https://firebase.google.com/docs/firestore/query-data/listen)) | Long-lived service-account key on the VPS ([admin setup](https://firebase.google.com/docs/admin/setup)); bypasses the main site's moderation and claim rules, which live in its server code, not in Firestore rules (`findyourdent/firestore.rules:2-7`) |
| D2 | Versioned API on the main product (`/api/v1/...`) with service-to-service auth and an "acting uid" | Moderation, claim checks and rate limits stay in one place; contract is testable; no Google key on the VPS | Main site needs a machine-auth path (today it expects the session cookie + App Check); adds latency and an availability dependency |
| D3 | Event/outbox sync | Workspace writes an outbox row in the same Postgres transaction and a worker delivers to D2 with idempotency keys; reverse direction via Firestore triggers, which are "delivered at least once" with "Ordering is not guaranteed" ([Firestore triggers](https://firebase.google.com/docs/firestore/extend-with-functions-2nd-gen)) | Eventual consistency; handlers must be idempotent and order-tolerant; needs a reconciliation job |

**Role and permission model for the five roles** (**[INFERENCE]**; relies on Better Auth `createAccessControl` roles/`hasPermission` ([organization](https://www.better-auth.com/docs/plugins/organization)) for the coarse layer and PostgreSQL RLS `USING`/`WITH CHECK`/restrictive policies ([RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)) for the record layer):

| Resource → | Owner / superadmin | Partner dentist | Dental nurse | Operations admin | Viewer |
|---|---|---|---|---|---|
| Tenant settings, members, roles, billing plan | manage | – | – | read | – |
| Directory page (owner facts, price list) | manage | – | – | edit | read |
| Patient demographics / CRM | manage | read/edit own patients | read/edit | manage | read (masked contact) |
| Schedule (dentist × chair) | manage | read all, edit own | read all, edit | manage | read |
| Clinical record (chart, notes, radiographs) | read all; write if also a clinician; break-glass logged | read/write for patients in own care; sign and lock | read for today's assigned patients; write assistant notes/vitals; cannot sign | no clinical notes; sees visit + procedure codes for billing | none |
| Inventory / assets | manage | read | consume/adjust | manage | read |
| Finance (invoices, payouts, fee split) | manage | own fee statements | – | manage | aggregated reports |
| WhatsApp inbox / AI agent | manage, configure agent | assigned conversations | handle | handle, assign | read |
| Audit log | read | own actions | – | – | – |

Record-level rules for medical records **[INFERENCE]**:
1. **Tenant wall**: a `RESTRICTIVE` policy `tenant_id = current_setting('app.tenant_id')` on every table, `FORCE ROW LEVEL SECURITY`, app role without `BYPASSRLS` ([RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)).
2. **Care relationship**: permissive policies on clinical tables keyed on a `care_team(patient_id, member_id, valid_during)` row or a same-day appointment; keep the predicate on the current row plus one indexed lookup because of the documented race/performance caveats ([RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)).
3. **Signed notes are immutable**: no `UPDATE`/`DELETE` policy once `signed_at` is set; amendments are new rows.
4. **Break-glass**: owner access outside a care relationship requires a reason, is time-boxed and writes an audit event in the same transaction.
5. **Platform operator ≠ tenant owner**: operator support uses Better Auth admin impersonation (1 h default — [admin](https://www.better-auth.com/docs/plugins/admin)) and is always audited; operators get no standing clinical access.
6. **2FA** is enforced for owner, dentist and operations-admin roles by a hook, since OAuth sign-ins are not 2FA-gated by default ([2FA](https://www.better-auth.com/docs/plugins/2fa)).

### d. Scheduling UI

| Option | Multi-resource (dentist × chair) views | License / price | Notes |
|---|---|---|---|
| FullCalendar v7.1.5 (2026-11-12) | Timeline and Vertical Resource View are Premium ([pricing](https://fullcalendar.io/pricing), [premium](https://fullcalendar.io/docs/premium)) | Standard MIT; Premium "Starting at $480" for 1–10 developers, renewal 50%; AGPLv3 only if frontend and backend are fully open source, otherwise commercial ([pricing](https://fullcalendar.io/pricing), [license](https://fullcalendar.io/license)) | Official shadcn registry items; `*-scheduler` items add `resource-timeline.tsx` / `resource-timegrid.tsx` and need `schedulerLicenseKey` ([FullCalendar shadcn](https://fullcalendar.io/docs/shadcn)) |
| Schedule-X 4.9.2 | Resource scheduler (hourly/daily, drag, resize, lazy loading) is premium (`@sx-premium/resource-scheduler`) ([resource scheduler](https://schedule-x.dev/docs/calendar/resource-scheduler)) | Core MIT ([repo](https://github.com/schedule-x/schedule-x), [npm](https://registry.npmjs.org/@schedule-x/calendar/latest)); premium €479/yr (2–3 devs) or €999 lifetime, one licence per project, self-hosted SaaS = one project ([premium](https://schedule-x.dev/premium)) | Preact core with a React adapter; depends on `temporal-polyfill` ([npm](https://registry.npmjs.org/@schedule-x/calendar/latest)) |
| react-big-calendar 1.20.0 | Source tree has `ResourceHeader.js`, `TimeGridHeaderResources.js` ([src](https://github.com/jquense/react-big-calendar/tree/master/src)); prop semantics **[UNVERIFIED]** | MIT ([npm](https://registry.npmjs.org/react-big-calendar/latest)) | Localizers: Moment, Globalize, date-fns v2, Day.js; README is silent on time zones ([README](https://raw.githubusercontent.com/jquense/react-big-calendar/master/README.md)) |
| Bryntum | Scheduler product | Scheduler $460 and Calendar $319 per developer/year (shown for 3 developers); the store also mentions MIT Community editions ([store](https://bryntum.com/store/)) | OEM/SaaS terms not on the store page |
| DayPilot | Lite Calendar has a resources view with custom columns; Lite Scheduler has resource rows ([DayPilot Lite](https://javascript.daypilot.org/open-source/)) | Lite Apache-2.0 with attribution; Pro SaaS edition from $1,199 (1 dev) / $2,899 (5 devs) ([Lite](https://javascript.daypilot.org/open-source/), [buy](https://javascript.daypilot.org/buy/)) | Real-time drag validation is Pro-only ([Lite](https://javascript.daypilot.org/open-source/)); own styling system, not Tailwind **[INFERENCE]** |
| ReUI Event Calendar (shadcn registry) | Resource view: "one column per resource", drag/resize/drag-create across columns ([ReUI](https://reui.io/docs/components/base/event-calendar)) | Installed with `shadcn add @reui/event-calendar`; licence text **[UNVERIFIED]** | date-fns v4 + `@date-fns/tz`; `timeZone` prop (IANA); RFC 5545 subset incl. EXDATE/RDATE, 1,000-occurrence cap; never persists data ([ReUI](https://reui.io/docs/components/base/event-calendar)) |

- **Recurrence.** RFC 5545 `RECUR`: `FREQ` required; `INTERVAL`, `COUNT`, `UNTIL`, `BYDAY`…; if `DTSTART` carries a `TZID`, `UNTIL` "MUST be specified as a date with UTC time"; local times without `TZID` are floating ([RFC 5545](https://www.rfc-editor.org/rfc/rfc5545)). `rrule` 2.8.1 is BSD-3-Clause ([npm](https://registry.npmjs.org/rrule/latest)). pg-boss also schedules by RRULE ([pg-boss](https://github.com/timgit/pg-boss)).
- **Indonesia's time zones.** IANA zones: `Asia/Jakarta` (Java, Sumatra) and `Asia/Pontianak` (west/central Borneo); `Asia/Makassar` (east/south Borneo, Sulawesi, Bali, Nusa Tenggara, west Timor); `Asia/Jayapura` (Papua, Maluku) ([zone1970.tab](https://data.iana.org/time-zones/tzdb/zone1970.tab)); current offsets +07 WIB, +08 WITA, +09 WIT, with no DST rules in the Indonesia section ([tz `asia`](https://raw.githubusercontent.com/eggert/tz/main/asia)). Store instants as `timestamptz`/`tstzrange`, keep an IANA zone per clinic location, expand recurrences in that zone, and render in the clinic's zone rather than the browser's **[INFERENCE]**.

### e. Background work and real-time

| Engine | Backing store | License | Operational weight / notes |
|---|---|---|---|
| pg-boss 12.39.0 | PostgreSQL ≥ 13, Node ≥ 22.12 or Bun | MIT ("always will be") | `SKIP LOCKED`; cron and RRULE scheduling, deferral, retries with exponential backoff, dead-letter queues with redrive, priority, pub/sub fan-out, queue policies for rate limiting/debounce/concurrency ([pg-boss](https://github.com/timgit/pg-boss), [npm](https://registry.npmjs.org/pg-boss/latest)) |
| Graphile Worker 0.18.0 | PostgreSQL, Node ≥ 22.18 | MIT | `LISTEN/NOTIFY` latency "typically under 3ms"; `job_key` de-duplication; crontab with backfill; 25 attempts over ~3 days; jobs can be created from triggers ([docs](https://worker.graphile.org/docs), [npm](https://registry.npmjs.org/graphile-worker/latest)) |
| BullMQ 6.7.10 | Redis ≥ 6.2, Valkey or Dragonfly; docs also state it "ships with a ready-made PostgreSQL backend" | MIT; Pro (groups, batches, observables) is commercial | Delayed jobs, job schedulers, rate limiting, flows; Redis must run `maxmemory-policy=noeviction` ([docs](https://docs.bullmq.io/), [connections](https://docs.bullmq.io/guide/connections), [npm](https://registry.npmjs.org/bullmq/latest)) |
| Temporal | Persistence DB + visibility store (Compose default: PostgreSQL + Elasticsearch) | MIT ([repo](https://github.com/temporalio/temporal)) | "Stateful control plane"; production image requires manual schema management; keep off the public internet ([self-hosted guide](https://docs.temporal.io/self-hosted-guide/deployment)) |
| Inngest | Single binary; default SQLite + in-memory Redis; external Postgres/Redis optional | SSPL v1 with Apache-2.0 grant on each release's third anniversary ([LICENSE](https://github.com/inngest/inngest/blob/main/LICENSE.md)) | `inngest start` labelled Beta in its help output; no automatic row cleanup; no guaranteed support for self-hosted ([self-hosting](https://www.inngest.com/docs/self-hosting)) |
| Trigger.dev | Webapp + Postgres + Redis, supervisor + runners | Apache-2.0 ([repo](https://github.com/triggerdotdev/trigger.dev)) | Self-hosted lacks warm starts, auto-scaling and checkpoints ([self-hosting](https://trigger.dev/docs/self-hosting/overview)) |
| Restate | Pre-built server binary/Docker ([repo](https://github.com/restatedev/restate)) | BSL 1.1; may not be offered as a "Public Restate Platform Service"; converts to Apache-2.0 four years after release ([LICENSE](https://raw.githubusercontent.com/restatedev/restate/main/LICENSE)) | Storage engine and footprint not confirmed here **[UNVERIFIED]** |

- **Real-time.** `NOTIFY` is delivered only on commit; payload < 8000 bytes ("put it in a database table and send the key"); 8 GB queue, and when full, transactions calling `NOTIFY` fail at commit; ordering preserved per commit order ([NOTIFY](https://www.postgresql.org/docs/current/sql-notify.html)). SSE/WebSocket options are in section a. Sync engine: Electric "syncs data out of Postgres into local clients over HTTP" via Shapes, writes go through your own API, with a TanStack DB integration ([Electric intro](https://electric.ax/docs/intro)); its licence and deployment architecture were not on the page opened **[UNVERIFIED]**.
- Pattern **[INFERENCE]**: webhook route verifies the signature, stores the raw event and enqueues a job in one transaction, returns 200; a worker process (same image, different command) runs agent steps as resumable jobs; reminders are deferred jobs created when the appointment is booked; each commit emits `NOTIFY tenant_<id>` with row ids; one SSE endpoint per browser tab fans out and triggers TanStack Query invalidation.

### f. Files and imaging

- **MinIO.** The repository "was archived by the owner on Apr 25, 2026"; README: "THIS REPOSITORY IS NO LONGER MAINTAINED", community edition "distributed as source code only", legacy binaries "will not receive updates", AGPLv3; the vendor points to AIStor Free/Enterprise ([minio/minio](https://github.com/minio/minio)). Not a candidate for a new build **[INFERENCE]**.
- **Garage** (docs reference v2.6.0): S3-compatible, single dependency-free binary, minimum 1 GB RAM/16 GB disk, designed for 3-zone replication ([Garage](https://garagehq.deuxfleurs.fr/)); `--single-node` exists but "should not be used in production, as it provides no redundancy" ([quick start](https://garagehq.deuxfleurs.fr/documentation/quick-start/)). Licence (believed AGPLv3) **[UNVERIFIED]**.
- **SeaweedFS** (release 4.53 mentioned): Apache-2.0; master + volume + filer servers; S3 gateway; separate Enterprise Edition ([seaweedfs](https://github.com/seaweedfs/seaweedfs)).
- **Cloudflare R2**: presigned GET/HEAD/PUT/DELETE valid 1 s–7 days, only on the `<ACCOUNT_ID>.r2.cloudflarestorage.com` S3 endpoint (not custom domains), HTML-form `POST` uploads unsupported, bucket CORS required for browsers ([R2 presigned URLs](https://developers.cloudflare.com/r2/api/s3/presigned-urls/)); $0.015/GB-month standard, Class A $4.50/M, Class B $0.36/M, no egress fee ([R2 pricing](https://developers.cloudflare.com/r2/pricing/)); jurisdictions are only `eu`, `us`, `fedramp`, and location hints (`apac`, …) are "a best effort and not a guarantee" — there is no Indonesia-specific placement ([R2 data location](https://developers.cloudflare.com/r2/reference/data-location/)).
- **Virus scanning.** ClamAV `clamd` is a multi-threaded daemon on a Unix or TCP socket; the TCP socket is unauthenticated and unencrypted ([ClamAV scanning](https://docs.clamav.net/manual/Usage/Scanning.html)); Docker guidance: minimum 3 GiB RAM, preferred 4 GiB, with reloads briefly doubling engine memory unless `ConcurrentDatabaseReload no`; port 3310 ([ClamAV Docker](https://docs.clamav.net/manual/Installing/Docker.html)).
- **Imaging stack.**

| Component | Role | License |
|---|---|---|
| Cornerstone3D 5.0 | Browser rendering library: stack/volume viewports, streaming volume loading, MPR-style orientations, annotation/segmentation tools, CPU fallback | MIT ([Cornerstone3D](https://www.cornerstonejs.org/docs/getting-started/overview)) |
| OHIF Viewer | "Zero-footprint" viewer app built on Cornerstone3D; works with DICOMweb archives | MIT ([OHIF](https://github.com/OHIF/Viewers)) |
| Orthanc | DICOM server | Core GPLv3+; scalability/cloud plugins (e.g. PostgreSQL, web viewer) AGPLv3+; calling it over REST/DICOM from a proprietary platform is permitted ([Orthanc licensing](https://orthanc.uclouvain.be/book/faq/licensing.html)) |

- **Study sizes.** DICOM native pixel data size follows rows × columns × frames × samples per pixel × bytes per sample ([DICOM PS3.3 C.7.6.3](https://dicom.nema.org/medical/dicom/current/output/chtml/part03/sect_C.7.6.3.html)). Order-of-magnitude planning figures, all **[UNVERIFIED]** (no vendor or standards page with per-study sizes could be opened): intraoral periapical/bitewing ≈ 2–10 MB uncompressed (e.g. a ~2,500 × 1,850 px 16-bit frame ≈ 9 MB); panoramic ≈ 5–30 MB; CBCT ≈ 30–400+ MB per volume (512³ voxels × 2 bytes ≈ 268 MB before compression). Measure real exports from the pilot clinics' devices before fixing quotas.
- Many small-clinic sensors export JPEG/PNG/TIFF rather than DICOM, so the viewer must handle plain images as well as DICOM **[INFERENCE]**.

### g. Deployment and operations

- **Reverse proxy.** Caddy (Apache-2.0 — [repo](https://github.com/caddyserver/caddy)): "By default, Caddy serves all sites over HTTPS", ACME via Let's Encrypt/ZeroSSL with auto-renewal; needs ports 80/443 and DNS pointing at the host, or the DNS challenge ([automatic HTTPS](https://caddyserver.com/docs/automatic-https)). Traefik (MIT): dynamic configuration from Docker, Let's Encrypt incl. wildcard ([repo](https://github.com/traefik/traefik)).

| Layer | License | Zero-downtime behaviour | Notes |
|---|---|---|---|
| Kamal 2.12.0 | MIT ([repo](https://github.com/basecamp/kamal)) | kamal-proxy routes to the new container once `GET /up` returns 200, then stops the old one ("gapless deployments") ([install](https://kamal-deploy.org/docs/installation/), [proxy](https://kamal-deploy.org/docs/configuration/proxy/)) | SSH-driven, needs an image registry; `ssl: true` Let's Encrypt only for single-server + `host`; behind a trusted proxy set `forward_headers: true`; secrets via `.kamal/secrets` ([proxy](https://kamal-deploy.org/docs/configuration/proxy/), [install](https://kamal-deploy.org/docs/installation/)) |
| Coolify (v4.x branch) | Apache-2.0 ([repo](https://github.com/coollabsio/coolify)) | Start-first rolling update for Dockerfile/Nixpacks/image apps; "not supported for Docker Compose applications"; blocked by host-published ports or fixed container names ([rolling updates](https://coolify.io/docs/knowledge-base/rolling-updates)) | Control plane itself needs 2 cores / 2 GB / 10 GB ([installation](https://coolify.io/docs/get-started/installation)); Git deploys, databases with backups, automatic HTTPS ([intro](https://coolify.io/docs/get-started/introduction)) |
| Dokploy | Apache-2.0, except `/proprietary` directories under a separate licence ([LICENSE](https://raw.githubusercontent.com/Dokploy/dokploy/canary/LICENSE.MD)) | Not described on the page opened | Built on Docker + Traefik; has an Enterprise section with licence keys ([docs](https://docs.dokploy.com/docs/core)) |
| Plain Docker Compose + Caddy | – | None built in; a deploy restarts the container **[INFERENCE]** | Fewest moving parts |

- **Secrets.** Compose secrets are mounted as files at `/run/secrets/<name>` and granted per service; environment variables "are often available to all processes" and "can also be printed in logs" ([Compose secrets](https://docs.docker.com/compose/how-tos/use-secrets/)).
- **Cloudflare in front.**
  - Tunnel: `cloudflared` makes outbound-only connections, so the firewall can block all inbound traffic and the origin needs no public IP ([Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/)).
  - Proxied DNS: use Full (strict) — origin cert must be valid, publicly trusted or Cloudflare Origin CA, hostname-matching; Authenticated Origin Pulls recommended ([Full strict](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/)).
  - Limits: request body 100 MB on Free/Pro, 200 MB Business ([limits](https://developers.cloudflare.com/workers/platform/limits/)); Proxy Read Timeout 125 s (524 on expiry) ([connection limits](https://developers.cloudflare.com/fundamentals/reference/connection-limits/)); WebSockets on all plans, closed when idle and on edge restarts ([WebSockets](https://developers.cloudflare.com/network/websockets/)). SSE streams therefore need a heartbeat well under 125 s, and large radiograph uploads must go direct to object storage via presigned URLs rather than through the proxied app **[INFERENCE]**.
- **Host hardening.**
  - Ubuntu applies security updates automatically via `unattended-upgrades` (daily, security pockets only); `Automatic-Reboot` defaults to `false` ([Ubuntu automatic updates](https://ubuntu.com/server/docs/how-to/software/automatic-updates/)).
  - SSH: ed25519 keys, optional FIDO/TOTP second factor, drop-ins in `/etc/ssh/sshd_config.d/`, validate with `sshd -t` ([Ubuntu OpenSSH](https://ubuntu.com/server/docs/how-to/security/openssh-server/)).
  - Firewall: published Docker ports are routed in the `nat` table "before the firewall rules can be applied", bypassing ufw ([Docker packet filtering](https://docs.docker.com/engine/network/packet-filtering-firewalls/)); bind to `127.0.0.1` or do not publish, and expose only the proxy **[INFERENCE]**.
  - CIS: benchmarks exist for Ubuntu Linux 24.04 LTS (2.0.0), Docker (1.9.0), PostgreSQL 17 (1.1.0) ([CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)); Ubuntu's `sudo usg fix cis_level1_server` applies the Level 1 server profile ([USG CIS](https://documentation.ubuntu.com/security/compliance/usg/cis-comply/)); USG being an Ubuntu Pro entitlement is **[UNVERIFIED]** (seen only in search snippets).
- **Observability.**

| Tool | License | Footprint / status |
|---|---|---|
| OpenTelemetry JS | – | Traces Stable, Metrics Stable, Logs Development; all active/maintenance Node LTS ([OTel JS](https://opentelemetry.io/docs/languages/js/)) |
| Grafana / Loki / Tempo | AGPLv3 (Loki: AGPL-3.0-only with Apache exceptions) ([Grafana licensing](https://grafana.com/licensing/), [Loki](https://github.com/grafana/loki)) | Internal use unaffected by AGPL network clause in practice **[INFERENCE]** |
| GlitchTip | MIT ([LICENSE](https://gitlab.com/glitchtip/glitchtip-backend/-/raw/master/LICENSE)) | 512 MB RAM recommended, PostgreSQL 14+, Valkey optional ([install](https://glitchtip.com/documentation/install)) |
| Sentry self-hosted | FSL, converts to Apache-2.0 after two years | Minimum 4 cores, 16 GB RAM + 16 GB swap, 20 GB disk ([Sentry self-hosted](https://develop.sentry.dev/self-hosted/)) — too heavy for the app VPS **[INFERENCE]** |
| SigNoz | Licence not confirmed **[UNVERIFIED]** | OpenTelemetry-native logs/metrics/traces ([SigNoz](https://github.com/SigNoz/signoz)) |
| Uptime Kuma | MIT | HTTP/TCP/ping/keyword monitors, 90+ notification services ([Uptime Kuma](https://github.com/louislam/uptime-kuma)); must run off-box to detect a dead VPS **[INFERENCE]** |

- **Uptime and DR targets** (**[INFERENCE]**; no primary source prescribes numbers for this product — align with the regulation track): RPO ≤ 5 minutes (WAL archiving with `archive_timeout` ≈ 60 s to an off-site encrypted repository); RTO ≤ 4 hours (scripted rebuild of a fresh VPS + pgBackRest restore); monthly automated restore test; quarterly full DR drill; target 99.5% monthly availability on one VPS (≈ 3.6 h/month), with a printable "today's schedule" export as the clinic's offline fallback.
- **Single-VPS limits and split order** (**[INFERENCE]**): (1) object storage and backups off-box from day one; (2) move PostgreSQL to its own host (plus a streaming replica) when RAM contention appears or the RTO must drop below ~1 hour; (3) move workers/agent runs and ClamAV to a second node when they affect request latency; (4) add a second app node behind the proxy last. A single VPS has no redundancy for host, disk or provider-zone failure.

### h. Sharing the design system with a separate repository

| Mechanism | What it requires | Fit |
|---|---|---|
| shadcn registry | A registry served as JSON; consumers map a namespace in `components.json` `registries` (`"@acme": "https://…/{name}.json"`), optionally with `headers` using `${ENV}` tokens for private access, then `npx shadcn@latest add @acme/button` ([namespaces](https://ui.shadcn.com/docs/registry/namespace)). Items are typed (`registry:ui`, `registry:block`, `registry:theme`, `registry:style`, …) and carry `files`, `dependencies`, `registryDependencies`, and `cssVars` with `theme`/`light`/`dark` keys; the `tailwind` key is deprecated in favour of `cssVars.theme` for Tailwind v4 ([registry-item.json](https://ui.shadcn.com/docs/registry/registry-item-json)). "The registry works with any project type and any framework" ([registry](https://ui.shadcn.com/docs/registry)) | Copy-and-own: the workspace gets the main product's components and tokens as source and may diverge; updates are a deliberate re-`add`, not a version bump **[INFERENCE]** |
| Tailwind v4 theme file | Tokens live in CSS under `@theme` (namespaces `--color-*`, `--font-*`, `--spacing-*`, `--radius-*`, …), generating both utilities and `:root` variables; "sharing them across projects is just a matter of throwing them into their own CSS file", which can be published to npm and `@import`ed ([Tailwind theme](https://tailwindcss.com/docs/theme); docs version v4.3) | Smallest unit of sharing: one `theme.css` keeps colour/typography/radius identical across both repos |
| Published workspace package | An npm package (private registry or Git dependency) exporting built components + the theme CSS; needs versioning, a build, and the consumer's Tailwind must scan the package's class names (the `@source` directive) **[UNVERIFIED]** | Single source of truth with semver, but adds a release pipeline and couples React/Tailwind versions across repos **[INFERENCE]** |

- The main product already has a shadcn `components.json` (style `new-york`, base colour `neutral`, CSS at `app/app.css`) (`findyourdent/apps/web/components.json:3-9`), so a registry or theme file can be produced from it without restructuring **[INFERENCE]**. The main app is React Router, not TanStack Start (`findyourdent/apps/web/package.json:36`), so only framework-neutral components and tokens are portable **[INFERENCE]**.

### Implications for the workspace platform

Everything in this section is **[INFERENCE]** built on the cited findings above.

**Recommended reference architecture**

```
Browser ── Cloudflare (proxied DNS + WAF, Full strict; or Tunnel) ── VPS (Ubuntu LTS, Docker)
  kamal-proxy / Caddy ─┬─ web: TanStack Start (Nitro node) — SSR, server fns, /api/auth, /api/webhooks/*, /api/rt (SSE)
                        │        routes: /…  and  /workbuddy/…  (same route tree)
                        └─ worker: same image, pg-boss consumers (reminders, WhatsApp events, agent steps, outbox → main-site API)
  PostgreSQL (RLS, btree_gist, pgaudit) ── pgBackRest ─→ local repo + off-site S3-compatible repo (encrypted)
  optional on-box: GlitchTip, OTel collector          off-box: object storage (images), uptime monitor, main-site API
```

1. **One TanStack Start app** on the Nitro Node target; `/workbuddy` is a route subtree, not a base-path app, until the open base-path issue is resolved.
2. **PostgreSQL, shared schema, `tenant_id` + forced RLS**, Drizzle (stable 0.45.x now, 1.0 when final) with a separate migration role; exclusion constraints for dentist and chair overlaps; app-level audit table plus pgAudit; LUKS volume + TLS + encrypted backups; application-level envelope encryption only for a few high-risk identifiers.
3. **Better Auth inside the app** (organization = tenant, custom roles via access control, TOTP + passkeys, admin plugin for operators). SSO phase 1 = option S1 (Firebase ID token → workspace session, uid continuity with claims); phase 2 = option S3 if seamless hand-off is required.
4. **Directory data stays owned by the main product**: workspace calls a new versioned main-site API (D2) through a Postgres outbox (D3); the Admin SDK on the VPS is limited to a one-off backfill, if used at all.
5. **pg-boss** worker for reminders, webhook processing and agent steps; Postgres `LISTEN/NOTIFY` → SSE for inbox and calendar; no Redis in v1.
6. **Images in external S3-compatible storage via presigned PUT**, quarantine prefix → scan job → publish; Cornerstone3D for 2D radiographs first, OHIF + a DICOM server only when CBCT viewing is in scope.
7. **Kamal** for gapless deploys of web + worker images; Cloudflare in front with the origin closed to everything but Cloudflare.
8. **Design system**: publish the main product's tokens as one Tailwind v4 theme CSS file and its components through a private shadcn registry.

**Alternatives considered**

| Decision | Chosen | Alternatives | Why not (now) |
|---|---|---|---|
| App shape | One Start app, `/workbuddy` subtree | Second app under base path | Start base-path handling still has an open issue; server routes not auto-prefixed |
| Tenancy | Shared schema + RLS | Schema-per-tenant; DB-per-tenant | N× migrations and pools; keep DB-per-tenant as an enterprise/hospital option |
| ORM | Drizzle | Prisma | Prisma's RLS extension is explicitly example-only; Drizzle models policies and roles in schema |
| Auth | Better Auth in-app | Keycloak / Zitadel / authentik | Second stateful service + DB; AGPL (Zitadel) or EE split (authentik); no multi-app need yet |
| SSO | S1 → S3 | S2 shared cookie; S4 OIDC via Identity Platform; S5 Google-only | S2 widens cookie scope and couples formats; S4 needs the Identity Platform upgrade and changes main-site auth; S5 mapping unverified |
| Data integration | Main-site API + outbox | Admin SDK direct | Would bypass moderation/claim rules and put a Google key on the VPS |
| Calendar UI | Shadcn-registry resource calendar (ReUI) spike, FullCalendar Premium as the paid fallback | Schedule-X premium; DayPilot Lite; react-big-calendar; Bryntum | Paid tiers for resource views (Schedule-X, FullCalendar, Bryntum); DayPilot Lite lacks live drag validation and Tailwind styling; RBC resource/time-zone support unverified. ReUI's licence must be confirmed first |
| Jobs | pg-boss | Graphile Worker; BullMQ; Temporal; Inngest; Trigger.dev; Restate | Graphile is a close second (pre-1.0 version number); BullMQ adds Redis; workflow engines add services and SSPL/BSL terms — revisit if agent runs need long waits/signals |
| Real-time | NOTIFY → SSE | Nitro WebSockets; Electric | WS co-hosting in Start unverified; a sync engine is a second service — reconsider for offline-first |
| Object storage | External S3-compatible bucket | MinIO; Garage single-node; SeaweedFS; local disk | MinIO archived; Garage single-node has no redundancy; SeaweedFS adds three services; local disk ties images to one host. R2 has no Indonesia placement — residency decision pending |
| Deploy | Kamal | Coolify; Dokploy; Compose + Caddy | Coolify can't roll Compose apps and uses 2 GB itself; Dokploy has a proprietary directory; plain Compose restarts on deploy |
| Error tracking | GlitchTip | Sentry self-hosted; SigNoz | Sentry needs 16 GB RAM; SigNoz licence unverified |

**Minimal VPS sizing estimate**

Assumptions: pilot of ≤ 30 clinics, ≤ 150 concurrent staff sessions, ≤ 50k appointments/month, ≤ 100k WhatsApp messages/month, LLM inference via external API, images in external object storage, one region.

| Component | RAM budget | Basis |
|---|---|---|
| OS, Docker, proxy, cloudflared | ~1 GB | estimate |
| Web (2 containers overlapping during deploy) | 1–1.5 GB | estimate |
| Worker (jobs + agent orchestration) | 0.5–1 GB | estimate |
| PostgreSQL | 4 GB | estimate |
| GlitchTip + OTel collector | ~0.75 GB | GlitchTip 512 MB recommended ([install](https://glitchtip.com/documentation/install)) |
| ClamAV (if on-box) | 3–4 GB | ClamAV minimum 3 GiB, preferred 4 GiB ([ClamAV Docker](https://docs.clamav.net/manual/Installing/Docker.html)) |

- **Minimum viable**: 4 vCPU / 8 GB RAM / 80 GB NVMe, with malware scanning and error tracking off-box or deferred.
- **Recommended pilot**: 8 vCPU / 16 GB RAM / 160 GB NVMe (headroom for ClamAV, GlitchTip, deploy overlap, local backup repo sized at 2–3× the database).
- **Object storage growth** (assumption-driven): 30 clinics × 20 radiographs/day × ~2 MB compressed × 300 days ≈ 360 GB/year; at R2 list price that is ≈ $5.40/month per 360 GB ([R2 pricing](https://developers.cloudflare.com/r2/pricing/)). CBCT changes this by an order of magnitude and should be quota-gated.
- A second small VPS (2 vCPU / 4 GB) for uptime monitoring, backup verification restores and later ClamAV is the first horizontal step.

### Open questions

1. Is seamless SSO with the main site a launch requirement, or is "same Google account, separate session" (S1) acceptable? Will the main site ever add non-Google sign-in?
2. Do nurses and operations admins have Google accounts, or must the workspace issue email/password + passkey identities that never exist in Firebase?
3. Is "superadmin/owner" one role, or a platform operator role plus a per-tenant owner role? Can a tenant own several listed providers (`placeId`s), and can one person belong to several tenants?
4. Can the main product expose a versioned machine-to-machine API (and relax the App Check requirement for it), or must the workspace write Firestore directly?
5. Data residency: must radiographs and backups stay in Indonesia? (R2 offers no Indonesian jurisdiction; decision belongs with the regulation track.)
6. Which imaging devices and export formats do pilot clinics use (DICOM vs JPEG/TIFF; CBCT in scope for v1)? Real file sizes are needed to replace the unverified estimates.
7. Is malware scanning of uploads a hard requirement (it costs 3–4 GB RAM), or is content-type validation plus image re-encoding sufficient for v1?
8. Calendar component: confirm the ReUI licence terms and whether FullCalendar Premium's per-developer licence is acceptable as the fallback.
9. Target PostgreSQL major: 19 was released on 2026-11-19; confirm extension and hosting-image availability (pgaudit v19.X exists) or start on 18.
10. Does `/workbuddy` share authentication, tenants and release cadence with the workspace, or is it a distinct product that only shares the host?
11. Will hospital customers require database-per-tenant isolation or their own SAML/OIDC identity provider at onboarding?
