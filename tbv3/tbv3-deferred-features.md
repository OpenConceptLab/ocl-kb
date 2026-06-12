# TBv3 Knowledge Base — Post-v3 Features and Deferred Design Topics

This file documents features, design ideas, and open questions that are explicitly **out of scope for v3 launch** but are worth preserving for future planning cycles. Each entry is a stub — enough context to resume the discussion without losing the original intent.

---

## Operations Panel

### Operations Panel (FHIR and OCL API Operations UI)
**Origin:** ocl_issues#1624

A developer-oriented panel that allows users to invoke FHIR and OCL API operations directly from the TermBrowser UI without leaving the page.

**Intended scope (from #1624):**
- Server selector (defaults to current server)
- Operation selector supporting:
  - FHIR: `$translate`, `$lookup`, `$validate-code`
  - OCL: `$cascade`, `$checksum`
- Schema-driven input form: each operation's request parameters are defined by a schema; the panel generates the input form dynamically
- Parameter inputs reuse existing TBv3 selector components (Repository + version, Concept, Mapping, JSON object)
- Submit button to execute the operation
- Results display: Raw and JSON views
- Copy request URL; Copy or Download results

**Post-v3 rationale:** No active design or code activity in oclweb3. Not in MVP scope. The checksum operation is partially referenced in the broader Diffs and Checksums epic (#1690) but the panel UI itself is unspecified and unbuilt.

---

## Search

### Mapping Constraint Search
**Origin:** `02_capabilities/search-and-filter.md`, ocl_issues#1607

Allow users to filter concepts by mapping constraints — for example, "show concepts that have no SAME-AS mapping to CIEL" or "show concepts mapped to a retired concept in source X." This requires a cross-concept mapping index and cannot be done with the current search API without significant backend work.

Surfaced by Andy Kanter during the v3 planning process. Preserved here for future search planning.

---

## Validation

### Content Validation Engine (Schema Rule Engine, Report Panel, Background Scan)
**Origin:** `02_capabilities/validate-content.md`

The full content validation capability — built-in rule categories (Required Properties, Allowed Classes, Allowed Datatypes, Mapping Constraints, Cardinality, Reference Structure), three-level severity model (Error/Warning/Info), Validation Report Panel, background scan of collection references, and error-blocking of version creation/expansion — is post-v3. Field-level form validation (blocking save on missing required fields) is in scope for v3 and already implemented.

This requires a dedicated design session before it can be built. See `validate-content.md` for the full specification, which is preserved here for when the engine is implemented.

---

### User-Defined Validation Schemas and Per-Rule Severity
**Origin:** `02_capabilities/validate-content.md`

Allow source/collection admins to define their own validation schemas (custom rule sets) and override the severity level of individual built-in rules (e.g., downgrade an Error to a Warning for their use case). The v3 system uses a fixed set of built-in rules only.

**Relevant context:** The `validate-content.md` file describes a full rule list UI with toggles and severity selectors — this is the intended future state. For v3, only predefined schema templates (OpenMRS, FHIR) are supported.

---

### Validation Report Panel for Sources
**Origin:** `02_capabilities/validate-content.md`

The Validation Report Panel is currently specified only for collections. How validation applies in the source authoring context (concept/mapping editor), and what a source-level report panel looks like, requires dedicated design work. This should be queued as a TBv3 topic.

---

## Concept Proposals

### `$match` Operation for Proposers
**Origin:** `03_workflows/manage-concept-proposals-workflow.md`

Before submitting a proposal, the proposal form should use OCL's `$match` operation to surface potential existing concepts that match the proposed concept — reducing duplicate proposals. The proposer sees "Did you mean one of these?" before finalizing their submission.

---

### Grouping Similar Proposals for Admins
**Origin:** `03_workflows/manage-concept-proposals-workflow.md`

A mechanism for source admins to detect and group duplicate or similar proposals in the review queue (e.g., multiple users proposing the same concept in different locales). Could be based on name similarity scoring or the same `$match` logic used for proposers.

---

### Post-Approval Actions for Proposers
**Origin:** `02_capabilities/manage-concept-proposals.md`

After a concept proposal is approved, give the proposer direct actions from the proposal detail page: view the newly created concept, add it to one of their collections, clone it to their own source, etc. Currently the proposer receives a notification with a link to the concept but no additional workflow from the proposal UI.

---

### Public-Facing Proposal View (Community Visibility / Upvoting)
**Origin:** `02_capabilities/manage-concept-proposals.md`

Allow non-admin, non-proposer users to view a source's open proposals and endorse or upvote them — giving source admins signal about community demand. This is a more open/community-model approach than the current proposer-to-admin flow.

---

## Expansions

### Include Drafts Parameter in Expansion Creation
**Origin:** `04_surfaces/expansion-form.md`, `02_capabilities/manage-versions-and-expansions.md`

The **Include Drafts** toggle (would set `include_drafts: true` in expansion parameters) allows an expansion to pull in HEAD/unreleased content from referenced sources. This is useful for testing against in-progress source content before an official version is released.

Deferred because: the use case requires more design work around how users discover and understand that an expansion contains draft content, and what permissions or warnings are appropriate when sharing such an expansion. For now, expansions always use released source versions only.

**When to revisit:** When users request pre-release testing workflows or when the "shadow expansion" / staging use case becomes a priority.

---

## Dependency Notifications

### Per-Dependency Notification Preferences
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2348)

Allow users to configure, per dependency (or globally), which events trigger notifications (new version, deprecation, deletion, nested collection update), which channels are used (in-app, email, webhook), and at what frequency (immediate, daily digest, weekly summary). v3 M44 ships with smart defaults only (in-app + optional email, immediate) and no per-dependency configuration UI.

**When to revisit:** If users report notification fatigue or request digest/quiet-hours behavior once the M44 notification flow is in production.

---

### Notification History / Export
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2348)

A searchable, filterable history of all dependency-change notifications a user has received, including what action was taken (updated, dismissed, snoozed), with export. M44 ships the Notification Center (current unread/read list) but not a long-term searchable history or export.

---

### Downstream Notification to Dependent Collection Owners (Reverse-Dependency Graph)
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2348 and #2349)

When a collection accepts an update and publishes a new version, owners of *other* collections that nest or depend on this collection are not notified in M44. This requires a reverse-dependency graph (collections depending on collections, not just on sources) which is not part of the M44 dependency graph (source → collection only).

**When to revisit:** Once the source → collection dependency graph from M44 is in production and nested-collection dependency tracking is prioritized.

---

## Version Locking / Update Collection

### Rollback After Accepting a Dependency Update
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2349)

After accepting an update and publishing a new collection version, provide a one-click rollback to the prior collection state (dependency version + expansion) if the update causes problems downstream. This depends on ocl_issues#2284 (Restore/Revert Collection HEAD to Previous Version), which is itself deferred and not an M44 blocker.

M44's safety net is the existing version history: the prior released collection version remains accessible and unchanged, so a manual revert is possible but not a guided one-click workflow.

**When to revisit:** When #2284 is scheduled.

---

### Scheduled / Batch Dependency Updates
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2349)

Two related capabilities, both deferred:
- **Scheduled updates:** let a user schedule "Accept Update" for a future date/time, with automatic pre-test and auto-rollback if the test fails.
- **Batch updates:** when multiple dependencies have pending updates, review and apply them together with a combined impact assessment, rather than one at a time.

M44 handles each pending source update as its own review-and-accept flow (with version-skipping supported within a single source's update chain).

---

### Update History / Audit Log
**Origin:** M44 scoping pass, 2026-06-12 (cut from ocl_issues#2349)

A dedicated, exportable audit log of all dependency updates applied to a collection: date/time, who applied it, before → after versions, impact assessment, and whether it was later rolled back. M44 relies on the collection's existing version history as the implicit record; no separate update-specific audit log is built.

---

### Facilitated Per-Concept Accept/Reject in the Update Collection Workflow
**Origin:** Joe + Jon design discussion, 2026-04-16

The current `update-collection-source-version.md` workflow offers a binary choice: accept the new source version entirely, or keep the current locked version. A more granular version would let users review the diff of changed/added/retired concepts and select which specific changes to accept individually — e.g., accept some newly added CIEL concepts but not others, or handle each retired concept differently.

**Why deferred:** The complexity is significant, especially for sources like CIEL that use cascading references. There is no trivial "facilitated accept/reject per concept" for cascade-based collections because the concept boundaries aren't cleanly enumerable from the reference list. Additionally, the happy path should encourage accepting all changes rather than granular picking — encouraging exclusions or per-reference version pinning silently prevents future updates.

**When to revisit:** When a real user need emerges with specific, articulable scope (e.g., "I only want new concepts in class X," which could map cleanly to reference management).

---

### Retired Concept Handling During Source Update — Open Question
**Origin:** Joe + Jon design discussion, 2026-04-16 — flagged for Wednesday OCL squad call

When a collection accepts a new CIEL version that retires a concept, the concept still appears in the expansion (the reference is unchanged) but is now tagged as "Retired." Two perspectives in tension:

- **Governance argument (Andy Kanter):** Showing the concept as Retired is the correct signal — implementers should know CIEL deprecated it. Pinning to the pre-retirement version hides that important metadata.
- **Implementation argument (Burke / OpenMRS):** A retired concept flowing through to an OpenMRS export might cause problems — unclear if downstream systems can still collect data against a retired concept.

**If FHIR/OpenMRS is fine with retired concepts:** The happy path is "accept all → rebuild → concept appears as Retired → user is informed but not blocked." Pinning to an old version remains available as an edge-case power-user action.

**If retired concepts break downstream systems:** We may need a more prominent warning during the diff review step and a simpler path to handle retired concepts without requiring the user to manually edit references.

**Action:** Discuss with Andy + Burke on Wednesday OCL squad call.

---

## References

### "Shopping Cart" Model for Batch Reference Building
**Origin:** Joe + Jon design discussion, 2026-04-16

Rather than the current one-at-a-time (dialog-per-reference) flow, a "shopping cart" model allows users to queue up multiple references across multiple sources and pages before applying them to a collection all at once. Key characteristics:

**Collection target selection:** When a user first clicks "Add to Collection" anywhere in OCL, they pick a target collection. That selection anchors the session — subsequent "Add" actions across sources accumulate in the same cart automatically.

**Non-blocking sidebar / panel:** The cart is shown as a collapsible sidebar that doesn't take over the page. Users can close it and continue browsing; the cart persists. This is especially important for large async expansions — users don't have to wait.

**Async mismatch detection:** Once the target collection is selected, OCL asynchronously checks each queued item for version mismatches against the auto-expansion's locked source version. Flags appear in the cart without interrupting the user's browsing.

**Checkout / finalize step:** The "checkout" is a review screen where the user can:
- See a preview of what will resolve from each reference
- Apply cascade settings or transforms in bulk
- Review and resolve any version conflict flags before applying

**Rebuild workflow (post-apply):** After applying references to HEAD:
1. The rebuild (re-expansion) runs **alongside** the current auto-expansion — it does not overwrite in place
2. A diff is computed between the pre-rebuild and post-rebuild expansions
3. User sees a simple summary: "X synonyms added, zero clinical impact" or a list of changed/added concepts, with a system recommendation (GitHub-merge metaphor)
4. User confirms → new auto-expansion becomes active; old one can be retained or discarded

**Multi-source:** The cart naturally handles references to multiple sources (CIEL + OpenMRS distro, etc.) in one session. The rebuild after checkout covers all sources at once.

**Post-M42 rationale:** This is a substantial UX paradigm shift and involves new state management (cart persistence, async mismatch checking, rebuild-alongside logic). For M42, the version consistency warning appears inline in the Add References dialog and the rebuild is triggered manually from the Versions + Expansions tab.

---

### Add References Dialog: Intensional Filter Builder
**Origin:** `02_capabilities/manage-references.md § Add References (Within Collection)`, ocl_issues#2431

A structured UI for building intensional reference expressions without typing raw query params. Would present rows of Property | Operator | Value inputs that generate the query string automatically (e.g., `?conceptClass=Test&locale=en`). For M42, users author intensional expressions by typing query params directly into the Expression field.

**Post-M42 rationale:** Requires design work on which filter properties to surface, how operators map to OCL query params, and how to handle complex multi-condition filters. The expression field is a sufficient workaround for power users in the near term.

---

### Add References Dialog: Search-and-Select Concepts
**Origin:** `02_capabilities/manage-references.md § Add References (Within Collection)`, ocl_issues#2431

A mode within the Add References dialog that lets users browse or search a source's concepts and select individual ones to add, rather than typing expressions or pasting IDs. Selected concepts would be queued and submitted as a batch of extensional expressions.

**Post-M42 rationale:** Significant additional UI work on top of the expression input and Bulk Add modes already in M42 scope. Bulk Add (comma-separated concept or mapping IDs, based on the selected Reference Type) covers the most common multi-resource case for now.

---

### Reference Preview: Unresolvable Reference Error Taxonomy
**Origin:** `02_capabilities/manage-references.md § Reference Preview`, ocl_issues#2007

When a reference expression cannot be evaluated at all (source not found, invalid URL, validation failure, etc.), the preview panel should surface a typed error rather than just showing an empty result. This requires the `CollectionReferencesPreview` API endpoint to return a structured `errors` array per expression.

**Proposed `errors` entry shape:**
```json
{ "type": "source_not_found", "message": "Source CIEL not found or not accessible." }
```

**Error type taxonomy:**

| `type` | Cause | Display message |
|---|---|---|
| `source_not_found` | The source in the expression URL does not exist or is not accessible | "Source not found or not accessible." |
| `concept_not_found` | The concept ID in the expression does not exist in the source | "Concept not found in source." |
| `invalid_expression` | The expression URL is malformed or uses a deprecated resource-versioned pattern | "Invalid reference expression." |
| `validation_error` | The resolved concept fails the collection's validation schema (e.g., OpenMRS required fields) | "Concept does not pass this collection's validation schema." |
| `cascade_error` | The cascade params reference a map type or method that is not valid for this source | "Cascade configuration is not valid for this source." |

Multiple errors per expression are possible; the frontend should display all. The `errors` array is in addition to (not replacing) the `new` / `existing` groups — an expression can have errors and also partially resolve.

**Post-M42 rationale:** Requires `CollectionReferencesPreview` to catch and classify per-expression failures instead of silently returning an empty result. For M42, unresolvable expressions are shown as empty results with no per-type explanation.

---

### Transform: Update Pinned Source Version
**Origin:** `02_capabilities/manage-references.md`

A transform that lets a user update the pinned version on a repo-versioned reference (e.g., from `orgs/CIEL/sources/CIEL/2023-01-01/` to `orgs/CIEL/sources/CIEL/2024-08-01/`) without changing the reference pattern or the concept it points to. Useful for collections that have deliberately pinned to a specific CIEL version and want to advance the pin without going through the full Update Collection workflow.

---

## Repository Configuration

### Extra Attribute Template Configuration
**Origin:** `01_objects/repository.md`

Allow a repository to define templates for Extra attribute keys — pre-defined keys with data types and human-readable labels. This would make the "Extras" section of concept/mapping forms more structured and consistent within a repository. Currently Extras are fully freeform.

---

### FHIR Repository Templates and Subtypes
**Origin:** `01_objects/repository.md`, ocl_issues #2027, #2028, #2029

FHIR-specific repository creation templates (ValueSet, CodeSystem, ConceptMap) and the underlying FHIR subtype architecture are post-v3. In v3, repository creation uses generic Source and Collection templates only. The FHIR-aligned types remain as collection/source type dropdown values but are not surfaced as distinct creation pathways with pre-filled FHIR metadata.

Two open architecture questions that must be resolved before deep FHIR support is added:
1. **ConceptMap**: Should it be a Source subtype, a Collection subtype, or valid as both? Currently modeled as a Collection subtype.
2. **CodeSystem**: Should it be a distinct top-level repo type (separate from Source), or remain a Source subtype? Currently modeled as a Source subtype (equivalent to FHIR CodeSystem).

Issues #2027 (ValueSet template), #2028 (CodeSystem template), and #2029 (ConceptMap template) are on the MVP milestone in ocl_issues but are **explicitly deferred** pending this architecture decision. They should be moved to the post-v3 backlog.

---

## Dropdown Configuration

### Configure Dropdowns UI (Per-Repository Dropdown Management)
**Origin:** `02_capabilities/configure-repository.md`, `02_capabilities/author-concept.md`

The Configure Dropdowns UI — allowing repository owners to manage per-field dropdown value lists (Concept Class, Datatype, Name Type, Map Type, Locale) with add/edit/delete/reorder controls — is post-v3. In v3, dropdowns are populated from OCL global sources (`OCL/Classes`, `OCL/Datatypes`, etc.) without per-repository customization.

This also defers concept templates (pre-filled form combinations based on a selected template). Both features depend on the same underlying `dropdown_config` field in the source model.

---

### Link Dropdowns to an OCL Source or Collection
**Origin:** `02_capabilities/author-concept.md`

Instead of manually managing dropdown values for concept class, datatype, name types, and map types in repository settings, allow owners to point to an existing OCL source or collection as the authoritative list of values. This enables centralized vocabulary management — e.g., an organization maintains a "Concept Classes" source and all their repos reference it.

OCL's own default sources for each field type also need to be documented.

---

## Linked Sources

## Organization

### Organization Landing Page Branding (Custom Banner / Background Image)
**Origin:** Andy Kanter feedback, 2026-05-20

OCL V2 lets organizations configure a custom banner image and background for their organization landing page. Andy uses this for the CL organization — `CLterminology.org` redirects directly to the OCL org page, making it CL's only public-facing website. The org page must be visually attractive; he's less concerned about branding within the source content pages themselves.

**Deferred rationale:** No design or code work has been done on org branding in TBv3. Core org page navigation and content functionality takes priority for MVP. Branding can be added after launch.

**When to revisit:** Before the V2 → V3 full cutover, since the org page serves as CL's canonical public URL.

---

### HEAD-Resolution Scope: Own vs. Not-Own Sources
**Origin:** `02_capabilities/manage-references.md`, `03_workflows/build-concept-dictionary.md`

When a collection is being updated and HEAD-resolution is used (to pick up new content before locking to a released version), it is not yet decided whether this applies:
- Only to sources the collection owner also owns (e.g., their own custom source)
- Or also to sources they do not own (e.g., CIEL)

The Linked Source pattern (allowing HEAD references for sources the user owns) is also not MVP. These questions deserve a dedicated design session with implications for the Update Collection workflow and the data model.

---
