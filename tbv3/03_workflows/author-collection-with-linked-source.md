# Workflow: Author a Collection with a Linked (HEAD) Source

## Purpose

A Terminology Implementer owns both a source and a collection — for example, the PIH source and the PIH dictionary collection. They are actively developing content: adding new concepts to the source and adding references to those concepts in the collection. The problem today is that new concepts added to a source's HEAD are invisible in the collection until the source is versioned and released, which makes the authoring loop feel broken.

This workflow describes how a **HEAD-link flag** on the collection's source relationship allows the collection HEAD to resolve references against the source's HEAD during development, and how the lock transitions cleanly to a released version upon publishing.

> **Relationship to the Update Collection workflow:** This workflow is for implementers who *own* the source they are developing content in (e.g., their own PIH source). If you are instead tracking updates from an external source you don't own (e.g., a new CIEL release), see [Update Collection to Latest Source Version](update-collection-source-version.md).

---

## SOW Coverage

- Linked source: Workflow to resolve references to HEAD when appropriate (during authoring) and transition to released repos upon version creation/release [Tracker: 60]

---

## Roles

- **Primary actor**: Terminology Implementer (owner of the collection; does not need to own the source)
- **Supporting actors**: System (auto-expansion, resolve reference, release orchestration)

---

## Entry Conditions

- User is authenticated and owns (or has edit access to) a collection
- The collection references the source (or will)
- The source has authorized the collection's owner (org or user) to access its HEAD (the source is open to its own owner by default, or has explicitly granted access to another owner)

---

## Core Concept: The HEAD-Link Flag

Normally, when a collection resolves its references against a source, it resolves against a **locked released version** of that source. HEAD content is not visible — it has not been published.

A **HEAD-link** is a flag on the collection's relationship to a specific source that says:

> "While I am working in HEAD of this collection, resolve references to this source against HEAD of that source."

This is not a new entity or a separate data structure. It is a property of a collection-to-source linkage. It uses the existing resolve reference operation and locking behavior, just with an additional mode for authoring.

**Implementation note:** Technically, the HEAD-link is a property of the collection's **auto-expansion**, not just the collection-to-source relationship. The auto-expansion for collection HEAD is configured to resolve references against source HEAD. This matters at release time: when the collection is published, the auto-expansion must be re-evaluated (not copied) so references transition from HEAD to the newly released source version.

**Key design decisions:**

| Decision                         | Choice                                                                                                  | Rationale                                                                                                                                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Where is the flag configured?    | **Collection level** — per source relationship | The collection configures which source to HEAD-link to. Whether it is permitted depends on whether the collection's owner has been granted HEAD access by the source. Resolution is always consistent regardless of which individual user triggers it — it is the collection's owner that is authorized, not the user. |
| Authorization basis              | **Source-level owner authorization**                                  | The source controls which owners (orgs or users) may access its HEAD — its own owner by default, or explicitly granted to specific external owners. A collection may use HEAD-link only if its owner is in that authorized list. Cross-owner grants are an exception case that implies a working relationship: the external owner's collections will always depend on the source owner to cut a release before their collection can be published. |
| Does the reference text change?  | **No** — references are unchanged                                                                | The reference expression remains the same; only the resolution behavior differs based on whether you are in HEAD of the collection or in a released version |
| What other sources are affected? | **Only the HEAD-linked source**                                                                   | Other sources (e.g., CIEL) remain locked to their pinned released version                                                                                   |

---

## Workflow

### Phase 1: Authoring (Working in HEAD)

The user authors in the collection; when new content is needed (e.g., a new answer concept), it is created in the linked source as part of that collection authoring action. The HEAD-link ensures the new source content is immediately visible in the collection without a release cycle.

> **Important:** Edits to the source do not flow automatically to the collection. Only content explicitly referenced by the collection is visible — unreferenced source HEAD content has no effect. The linked source is the *destination* for new concepts; the collection is the *driver* of what gets created there.
>
> A single source can be HEAD-linked from multiple collections simultaneously. For example, PIH might maintain both a main PIH dictionary collection and a Vaccinations collection, both HEAD-linked to the same PIH source. Content created while authoring either collection lands in the shared PIH source.

```
AUTHORING PHASE — working in HEAD of both source and collection
═══════════════════════════════════════════════════════════════════

  PIH Collection [HEAD]
  ┌──────────────────────────────────────────────────────────────┐
  │  ref: /PIH/sources/PIH/concepts/123/   ─────────────────┐   │
  │  ref: /PIH/sources/PIH/concepts/124/   ─────────────────┤   │
  │  ref: /CIEL/sources/CIEL/concepts/5089/ ───────────────┐ │   │
  └───────────────────────────────────────────────────────────────┘
                 │ HEAD-link flag active                    │   │
                 ▼                                          │   │
  PIH Source [HEAD]  ◄────────────────────────────────────────┘
  ┌────────────────────────┐
  │ concept/123  ← new!   │   resolved immediately ✓
  │ concept/124  ← new!   │   user sees these in collection
  └────────────────────────┘
                                                            │
                                                            ▼
                                               CIEL [v2026-01]  ← locked, unchanged
                                               ┌──────────────────────┐
                                               │ concept/5089         │
                                               └──────────────────────┘
```

**What the user experiences:**

1. While managing the collection, user adds new content (e.g., adds an answer concept to a question) → concept is created in PIH Source HEAD
2. A reference to the new concept is added to the collection HEAD simultaneously
3. Auto-expansion resolves the reference → resolves to PIH Source HEAD via the HEAD-link
4. User navigates to the collection → sees the new concept immediately
5. Repeat for each new concept — no release required to see the work

**What happens under the hood:**

- Resolve reference is called with a flag to use HEAD of the linked source
- The reference expression itself is unversioned (e.g., `/PIH/sources/PIH/concepts/123/`) — this same expression is what will appear in the released version
- Other sources (CIEL) are not affected; they resolve against their locked released version as usual

---

### Phase 2: Releasing (Batch Release)

When the user is ready to publish a new version of their dictionary, OCL first checks whether a source release is actually needed:

- **If the collection references no unreleased source content** — all referenced content is already in the latest released source version — the collection publishes immediately, locking to that released version. No source release required.
- **If the collection references HEAD content not yet in any release** — the source must be released first. Two steps then happen in order:

1. **Release the source** — unpublished (HEAD) content must be versioned so the collection can point to it
2. **Release the collection** — the HEAD-link switches from HEAD to the newly released source version

> **Why not auto-release the source?** OCL warns and asks the user to release the source manually rather than doing it automatically. A source's HEAD may contain work-in-progress content unrelated to this collection — auto-releasing would publish it all. Releasing a source may also trigger QA pipelines and validation steps that need to complete first.

```
RELEASE PHASE — batch release of source then collection
═══════════════════════════════════════════════════════════════════

STEP 1: Release PIH Source

  PIH Source [HEAD]
  ┌────────────────────┐
  │ concept/123        │  ──publish──►  PIH Source [v19.3]
  │ concept/124        │                ┌────────────────────┐
  └────────────────────┘                │ concept/123  ✓     │
                                        │ concept/124  ✓     │
                                        └────────────────────┘

STEP 2: Release Collection

  PIH Collection [HEAD]  ──publish──►  PIH Collection [v3.0]

  The HEAD-link switches on release:
  ┌──────────────────────────────────────────────────────────────┐
  │  BEFORE  (HEAD):   resolve PIH refs → PIH Source HEAD        │
  │  AFTER   (v3.0):   resolve PIH refs → PIH Source v19.3  ✓   │
  └──────────────────────────────────────────────────────────────┘

RESULT: PIH Collection v3.0
  ┌──────────────────────────────────────────────────────────────┐
  │  ref: /PIH/sources/PIH/concepts/123/  →  PIH v19.3  ✓       │
  │  ref: /PIH/sources/PIH/concepts/124/  →  PIH v19.3  ✓       │
  │  ref: /CIEL/sources/CIEL/concepts/5089/ → CIEL v2026-01 ✓   │
  └──────────────────────────────────────────────────────────────┘
  All references resolve. HEAD-link is no longer active in v3.0.
```

**Happy path steps:**

1. User decides to publish a new version of the dictionary
2. System checks: does the collection reference any PIH Source content that exists only in HEAD (not in the latest released version)?
3. **If no unreleased content referenced:** collection is published immediately; references lock to the latest released PIH Source version. Done.
4. **If unreleased content is referenced:** user is prompted to release PIH Source first (see [Unreleased Content Warning](#unreleased-content-warning) below)
5. User releases PIH Source (e.g., as v19.3) — a deliberate manual step, not automatic
6. User releases the collection — system re-evaluates references and locks to PIH Source v19.3
7. Collection version is published

> **UX goal:** The release experience should feel like one action from the user's perspective. If the workflow forces users to navigate away to release a source, run through QA, and return before releasing the collection, most users will abandon it and revert to source-only authoring. The design should guide the user through source release as an inline dependency step, not a separate context switch.

---

## Unreleased Content Warning

If the user attempts to release the collection before releasing the source, the system surfaces a warning rather than a hard block:

> ⚠️ **Heads up:** Your collection references content in PIH Source that has not been published yet (N concepts in HEAD). If you release the collection now, those references will not resolve in the published version.
>
> **Recommended:** Release PIH Source first, then return to release the collection.
>
> [Release PIH Source →]  [Continue Anyway]

This is a warning, not a blocker. The user can proceed, but should understand the consequence: any references pointing to unreleased source content will return no result in the published collection version.

**Implementation considerations:**

- Detecting which HEAD content is referenced by the collection requires diffing source HEAD against its latest release and then checking collection references against that diff — may have performance implications at scale and needs validation.
- Source HEAD may contain changes the user did not intend to publish alongside this collection. The user should remain in control of when a source version is cut.
- The check should be scoped to content actually referenced by the collection, not all HEAD changes in the source.

---

## What Stays Locked During Authoring

Not all source relationships become HEAD-links. The HEAD-link is opt-in, per source, at the collection level. A typical setup might look like:

| Source                  | Relationship type  | Behavior during authoring HEAD        | Behavior on release                               |
| ----------------------- | ------------------ | ------------------------------------- | ------------------------------------------------- |
| PIH Source (own source) | HEAD-linked        | Resolves to PIH Source HEAD           | Transitions to latest released PIH Source version |
| CIEL                    | Locked to v2026-01 | Resolves to CIEL v2026-01 (unchanged) | Stays at v2026-01 unless explicitly updated       |

CIEL stays on its pinned version throughout. Only the source explicitly configured as HEAD-linked gets the HEAD resolution behavior.

---

## Implementation Notes

**What already exists in OCL:**

- The resolve reference operation already supports a HEAD parameter and handles the case where a source has no released version (falls back to HEAD)
- Source version locking on collection release already exists
- The API can already represent a collection expansion locked to HEAD and a collection expansion locked to a released version via expansion parameters — the underlying content model for HEAD-link is in place today for same-owner scenarios
- Owner-based URL registry (for multi-org resolution) already exists

**What needs to be built:**

- UI to configure a source relationship as HEAD-linked in the collection (Collection → Settings → "Linked Sources" section), gated on the collection's owner being in the source's authorized owner list
- Owner-level HEAD access control: today HEAD is accessible to anyone with repo access; future permissions work must introduce owner-to-owner authorization so the source can explicitly grant HEAD access to specific external owners (OCL has no precedent for this type of access control yet; the current cross-owner gap means collection and source must share the same owner for now)
- Business logic layer in auto-expansion to resolve unversioned references against source HEAD when the linked source is configured to HEAD
- Re-evaluation step on collection release — currently OCL copies the HEAD auto-expansion on version creation rather than re-evaluating; for HEAD-link to work, the release step must re-evaluate references and lock to the source version current at release time

---

## Open Issues

| Issue                                      | Description                                                                                                                                                                                                                                                                                                                                                                |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Auto-expansion re-evaluation on release    | When creating a released collection version, OCL currently copies the HEAD auto-expansion. For the HEAD-link transition (HEAD → released version) to work correctly on publish, the release step must re-evaluate references rather than copy them. This is a behavior change in the current release flow.                                                                |
| Dirty reference detection                  | If a user edits or deletes a concept in the source HEAD, the collection HEAD does not automatically know to refresh. Re-running all references on every source change is too slow at scale. A smarter invalidation mechanism (e.g., source-level change timestamps compared to expansion timestamps) is needed. For the MVP, users may need to manually trigger a refresh. |
| ~~Multi-org / cross-owner access~~ (Resolved 2026-05-13) | Authorization is owner-based at the source level — the source grants HEAD access to its own owner by default and can explicitly grant access to specific external owners (orgs or users). A collection's owner must be in the source's authorized list; individual user access is not the gate. Cross-owner grants are an exception case, not an open/public model, and carry an implied commitment: the external owner's collections will always depend on the source owner to release before their collection can publish. The URL registry handles cross-org URL resolution. (Further clarified 2026-05-29.) |
| Source release scope | When a collection's HEAD-link requires a source release before publishing, the source HEAD may contain work-in-progress content the user did not intend to release alongside this collection (other concepts still under review, QA steps not yet run, etc.). Options: (a) warn and require manual source release — current design, safest; (b) auto-release the source before the collection — risky, captures unintended changes; (c) partial/cherry-pick release — technically hard, no current UI. The smart-check (publish against the latest released version when possible) reduces how often a source release is actually needed. |
| Owner-level access control | OCL currently has no mechanism to grant HEAD access to a specific external owner (org or user). Today HEAD is accessible to anyone with repo access; versionless references point to latest rather than HEAD, but HEAD itself is still reachable. Future permissions work must introduce owner-to-owner authorization. Until then, cross-owner HEAD-link requires the collection and source to share the same owner. |
| Concept modifications | If a collection references a source concept and adds local modifications (e.g., PIH adds a local display name or answer set to a CL concept), how those modifications interact with the HEAD-link and the release transition has not been determined. The reference expansion structure is designed to accommodate this, but it has not been fully designed as part of this workflow. Needs to be addressed as part of broader collection management work. |

---

## Related

- [Build a Concept Dictionary](build-concept-dictionary.md) — initial collection setup
- [Update Collection to Latest Source Version](update-collection-source-version.md) — tracking external source updates (e.g., new CIEL version)
- [Object: Version](../01_objects/version.md) — HEAD vs. released version lifecycle
- [Object: Reference](../01_objects/reference.md) — how references are resolved and locked
- [Capability: Manage Versions and Expansions](../02_capabilities/manage-versions-and-expansions.md) — release controls and auto-expansion behavior
