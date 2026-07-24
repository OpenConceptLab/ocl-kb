# Capability: Manage References


## Scope

This capability covers all ways a user can define what content belongs in a collection: adding references from outside the collection context (e.g., from search results or concept detail), creating references from inside the collection, previewing reference results without saving, tracing a concept back to the reference that brought it in, transforming existing references, and removing references.

---

## Adding References from Outside the Collection (Add to Collection)

This is the most common path for Terminology Implementers building a dictionary. The user encounters a concept elsewhere in TBv3 and wants to add it to a collection they own.

### Entry Points
- Concept Detail (split view or full page) → "Add to Collection" CTA ✅ implemented
- Concept row in Search Results → action button or right-click menu ⬜ not yet wired
- Concept row while browsing a repository → action button or right-click menu ⬜ not yet wired
- Hierarchy view node → "Add to Collection" action ⬜ not yet wired

### Interaction Flow
1. User clicks "Add to Collection"
2. A dialog opens (`src/components/common/AddToCollectionDialog.jsx`):
   - **Collection selector**: searchable autocomplete of all collections the user has edit access to (personal + org), grouped by owner; live-filtered by collection name, ID, short code, or owner — no debounce. Each option shows the collection name on the left and a `RepoChip` on the right; the chip tooltip activates after a 1-second hover. Only HEAD collections are shown (you can only add references to HEAD).
   - **Cascade selector** (rendered via `src/components/common/CascadeSelector.jsx`):
     - **None** — add the concept only; no mappings or target concepts
     - **Source Mappings** — add the concept plus all its mappings within the same source (`method=sourcemappings`)
     - **All source concepts and mappings** — traverse Q-AND-A and CONCEPT-SET mappings recursively, returning all reachable concepts and mappings (`method=sourcetoconcepts&mapTypes=Q-AND-A,CONCEPT-SET&cascadeLevels=*&returnMapTypes=*`)
     - **Custom** — expands an inline form (left-bordered) to specify `method`, `mapTypes`, `excludeMapTypes`, `cascadeLevels`, and `returnMapTypes` manually
   - **Advanced → Transform** checkbox (default: unchecked) — applies the OpenMRS transform (`transformReferences=openmrs_concept_reference`) on top of the selected cascade. Shown with a `?` help icon tooltip explaining its purpose.
   - **Preview API call** (default collapsed): toggled by a chip button; shows the full `PUT /:owner/collections/:collection/references/` request with query string and request body, updating live as collection and cascade options change
3. User confirms → `PUT /[owner]/collections/[collection]/references/` with `data.expressions` containing the concept URL (version stripped) and `cascade` set to the selected method; cascade query params appended as needed
4. Results displayed inline: successful additions shown in light blue rows (concept ID + message); failures shown in a two-column table (Reference | Error) with human-readable error descriptions unwrapped from the API's expression-keyed envelope, including conflicting reference IDs and conflicting concept names
5. On completion the form locks; user can close

### Version Consistency Warning
> **M42 MVP deliverable — not yet implemented.** This is the primary version locking UX for the M42 showcase.

- Before submission, detect whether the concept version that a user intends to add to a collection based on the user's context (let's call this the **intended concept version**) would be different than the **resolved concept version** based on the collection HEAD's auto-expansion's locked source version for that source
- If a mismatch is detected, show a warning in the dialog before the user confirms
- If no mismatch (or the collection has no locked source version yet), proceed silently with no warning

**Happy path (best practice):**
- **Add unversioned reference and rebuild the expansion** — the expansion re-evaluates all unversioned references against the latest locked source version, so the new reference will resolve correctly
- The warning dialog should include a "Go to collection to rebuild" CTA that takes the user to the collection's Versions + Expansions tab; the rebuild itself is triggered there, not inline in this dialog

**Less happy paths (presented as secondary options):**
- **Add unversioned reference without rebuilding** — the reference is added but will resolve to the old locked version v[Y] until the expansion is rebuilt
- **Add version-pinned reference** — not recommended unless the user explicitly wants to pin to a specific version; shown with a caution label

> **M42 scope boundary:** The warning UI is M42. The guided rebuild workflow (diff-then-confirm, alongside expansion) is **post-M42** — users are directed to the Versions + Expansions tab to trigger the rebuild manually.

> **Technical constraint — CIEL and concept versioning:** CIEL's current import process creates a new concept version and mapping version for every resource on every import, even when content has not meaningfully changed. A naive version-number comparison will therefore always show all concepts as "changed." The version mismatch check must use **smart checksums** (not raw concept version comparison) to distinguish real clinical changes from no-op version bumps. This is a two-pass check: (1) identify if a newer concept version exists → (2) compare smart checksums to determine if there is a meaningful difference. See `01_objects/version.md` and ADR-003 for checksum details.

---

## Add References (Within Collection)

> **Ticket:** ocl_issues#2431 — CTA / Reference / Add (within Collection)

Used when the user is already in the collection context and wants to define what content to include. The primary tools are a direct expression input field and an optional source/collection seed that builds the base path automatically.

> **Design direction (2026-05-07):** This dialog should evolve toward a **manage references** interface — a unified surface where adding new references and editing existing ones are both possible in-context, rather than two separate views. The long-term model is closer to in-line spreadsheet editing (grouped by source) than a standalone add dialog. The current dialog is the M42 starting point; the manage/view unification is a near-term follow-on.
>
> **Expanded reference syntax:** OCL's underlying data model uses the **expanded reference syntax** (per-field: system, version, code, include, cascade), not the simplified inline URL string. The simplified syntax is a display/data-entry shorthand. This tool should be designed around the expanded model — it is compatible with everything OCL can do, including canonical URLs and FHIR. See the Resolve Reference operation doc for the full expanded syntax spec.

### M42 Scope

| Feature | M42? |
|---|---|
| Add by ID(s) mode (comma-separated concept or mapping IDs) | ✅ M42 |
| Expression mode | ✅ M42 |
| Source / Collection quick-search seed | ✅ M42 |
| Version hidden by default; "Pin to a specific version" toggle | ✅ M42 |
| Reference Type selector (Concepts / Mappings) | ✅ M42 |
| Include / Exclude toggle | ✅ M42 |
| Cascade selector | ✅ M42 |
| Preview panel (`ReferencePreviewPanel` — #2007) | ⬜ Post-M42 |
| Intensional filter builder (Property / Operator / Value) | ⬜ Post-M42 |
| Search-and-select individual concepts from a source | ⬜ Post-M42 |

### Entry Point

Collection → References tab → `+ Add References` button

- Visible to users with edit access only
- Disabled (with tooltip) when viewing a released version; HEAD only
- Not shown to read-only users

### Component

`src/components/collections/AddReferencesDialog.jsx`

---

### Dialog Sections

#### 1. Source / Collection (seed field)

Searchable autocomplete of all sources and collections in OCL — not scoped to the current user's ownership. Typing searches by name, ID, short code, and owner.

- **Optional** — the user can type an expression directly without using the seed at all
- On selection: Version selector and Reference Type selector appear; the Expression field is populated with the base path
- **If the user has already typed a non-empty expression** before selecting a source/collection, show a warning inline: *"This will replace the current expression."* — selection only proceeds if the user confirms (or the field is empty)
- After a source/collection is selected, show an action link to open that source/collection at the selected version context. This lets users browse/search Concepts or Mappings directly and add from that source/collection context if they do not know the IDs to enter.

#### 2. Version (hidden by default)

References default to **versionless** — no version control is shown after a source is selected. Versionless is the strongly encouraged best practice, and the UI should not create friction around it.

A **"Pin to a specific version"** checkbox appears below the source/collection selector, unchecked by default. When unchecked, a muted helper text is shown:
> *Resolves to {resolved_version_id} via the collection's expansion strategy.*

(The resolved version shown is the collection's locked source version if one exists, or the latest released version of the selected source/collection otherwise.)

Checking "Pin to a specific version" reveals the version dropdown:
- **Released version IDs** — listed in descending order. Selecting one produces a version-pinned expression path (e.g., `/orgs/CIEL/sources/CIEL/v2024-08-01/`).
- HEAD is not a selectable option.

**When the collection has a locked version and the user pins to a different version:** show a warning inline below the dropdown:
> ⚠️ *Your collection is pinned to {locked_version_id}. This pinned reference will differ from the versionless resolution.*

**When the collection has no locked version:** No locked-version warning is shown unless the API later reports a version inconsistency.

#### 3. Reference Type Selector

Appears after a source/collection is selected. Dropdown with two options:

| Option | Appended to expression |
|---|---|
| **Concepts** (default) | `concepts/` |
| **Mappings** | `mappings/` |

Changing this updates the seeded expression path. If the user has manually edited the expression past the base path, changing the type shows the same overwrite warning as above.

#### 4. Expression Field

Visible and editable only in **Expression** mode. Hidden in **Add by ID(s)** mode.

The seeded base path format depends on whether a source or collection was selected:

| Seed type | Seeded format (Versionless / unversioned) | Seeded format (specific version) |
|---|---|---|
| **Source** | `/{ownerType}/{owner}/sources/{source}/concepts/` | `/{ownerType}/{owner}/sources/{source}/{version}/concepts/` |
| **Collection** | `/{ownerType}/{owner}/collections/{collection}/concepts/` | `/{ownerType}/{owner}/collections/{collection}/{version}/concepts/` |

- Placeholder when empty: `e.g. /orgs/CIEL/sources/CIEL/concepts/1234/`
- User may append anything after the base path: a concept or mapping ID, query params (`?conceptClass=Test&locale=en`), etc.
- Submitting only the base path (e.g., `/orgs/CIEL/sources/CIEL/concepts/`) without further qualification is valid — it creates a reference that resolves to all concepts in that repo at the given version

#### 5. Add by ID(s) vs. Expression Mode

The dialog supports two mutually exclusive authoring modes:

| Mode | Behaviour |
|---|---|
| **Add by ID(s)** (default) | User pastes comma-separated IDs; the Expression field is hidden and the generated base path context is represented by the upstream Source / Collection, Version, and Reference Type fields |
| **Expression** | User edits and submits the Expression field as a single reference expression |

Switching Source / Collection, Version, or Reference Type updates the generated Expression/base path. If the user manually edited the Expression field in Expression mode, changing those upstream fields shows the overwrite warning described above.

Add by ID(s) entry:

- **Requires** a source or collection to be seeded (IDs must be expanded into full URLs; not available without a repo context)
- Textarea accepting comma-separated IDs for the currently selected Reference Type: `1234, 5678, 9012`
- When Reference Type is **Concepts**, IDs expand under `concepts/`; when Reference Type is **Mappings**, IDs expand under `mappings/`
- On submit, each ID is expanded into a separate extensional expression using the seeded repo path:
  - From a source: `/{ownerType}/{owner}/sources/{source}/{version}/{resourceType}/{id}/`
  - From a collection: `/{ownerType}/{owner}/collections/{collection}/{version}/{resourceType}/{id}/`
  - `{resourceType}` is `concepts` or `mappings`, based on the selected Reference Type
- Submitted as N expressions in a single batch API call; Preview also batches all N

**Add by ID(s) and Expression are mutually exclusive:** the user must choose one mode before submit. Add by ID(s) no longer silently takes precedence over Expression.

#### 6. Include / Exclude

Dropdown, defaults to **Include**.

Applies to all expressions in the submission — you cannot mix include and exclude references in a single dialog open.

| Option | Behaviour |
|---|---|
| **Include** | Standard inclusion reference (`include: true`) |
| **Exclude** | Exclusion reference (`include: false`); shown with Exclude chip in the References list |

#### 7. Cascade Selector

Same `CascadeSelector` component used in `AddToCollectionDialog`. Applies to all expressions in the submission.

Use the existing `src/components/common/CascadeSelector.jsx` design pattern:

- Preset dropdown: **None**, **Source Mappings**, **All source concepts and mappings** (`sourcetoconcepts` with OpenMRS Q-AND-A / CONCEPT-SET defaults), and **Custom**
- **Custom** expands an inline, left-bordered form for `method`, `mapTypes`, `excludeMapTypes`, `cascadeLevels`, and `returnMapTypes`
- **Advanced** chip toggles the Transform checkbox
- **Transform** appends `transformReferences=openmrs_concept_reference`, matching the current CascadeSelector behavior
- **Preview API call** chip toggles a compact request preview for implementation/debug visibility

This Transform checkbox is distinct from broader reference transforms handled by #2433 from the References tab.

#### 8. Preview Panel (Post-M42)

> **Deferred (2026-05-07):** Previewing a set of references in the Add References dialog is complex and not required for M42 MVP. The per-reference preview (clicking an existing reference row in the References tab to see its resolved results in the detail split view) is the M42 model. The bulk preview panel below is specified for future reference but should not be built as part of the initial Add References dialog.

`ReferencePreviewPanel` (#2007), collapsed by default, toggled open via `[Preview ▼]`.

- In expression mode: fires on the single expression value
- In Add by ID(s) mode: fires on all N expanded expressions as a batch
- Version mismatch and zero-result warnings are surfaced here (see `§ Reference Preview`)
- Preview fires only when the user opens Preview or clicks **Refresh Preview**. If expression, version, type, include/exclude, or cascade params change after preview has been run, the existing preview is marked stale until the user refreshes it.
- Preview supports two display modes:
  - **References → Resolved Concepts/Mappings**: one expandable row per reference expression with resolved counts and status. Expanding uses the same concept sub-row / mapping sub-row / orphaned mapping group pattern as `04_surfaces/references-tab.md § Expandable row layout (concept sub-rows)`.
    - Suppress the Orphaned Mappings group when no orphaned mappings exist.
  - **Concepts/Mappings → References**: one row per resolved resource, with a **References** column that shows an expandable count (e.g., `+ 2 References`) and expands inline to show the contributing reference expressions. Concept rows can still show nested mapping rows using the same References tab expansion pattern.
    - Nested mapping rows should indicate that the mapping was included by the concept's cascade.

---

### Interaction Flow

1. User clicks `+ Add References` from the References tab toolbar
2. Dialog opens; expression field is empty with placeholder
3. *(Optional)* User types in the Source / Collection seed field and selects a source
4. Helper text appears below the source: *"Resolves to {version} via the collection's expansion strategy."* The "Pin to a specific version" checkbox is shown unchecked. User checks it only if they want to pin to a specific released version; locked-version warning shown if the pinned version differs from the collection's expansion-locked version.
5. Reference Type selector appears — user selects Concepts or Mappings (default: Concepts)
6. Expression field is populated with the seeded base path; user edits as needed (adds ID, query params, etc.)
   — *or* —
   User uses Add by ID(s) and pastes comma-separated concept or mapping IDs, depending on the selected Reference Type
7. User sets Include/Exclude and cascade options
8. User clicks **Add Reference(s)**
10. `PUT /:ownerType/:owner/collections/:collection/references/` with `{ data: { expressions: [...] } }` + cascade query params
11. Results displayed inline: successes in light blue rows; failures in a Reference / Error table (same pattern as `AddToCollectionDialog`)
12. Form locks on completion; user closes

---

### Submit Behaviour

| Mode | `expressions` array |
|---|---|
| Expression only | `[expressionFieldValue]` |
| Add by ID(s) only | `[/{source}/{version}/{resourceType}/{id1}/, ..., /{source}/{version}/{resourceType}/{idN}/]`, where `{resourceType}` is `concepts` or `mappings` |
| Both filled | Not allowed; the dialog must be in either Expression mode or Add by ID(s) mode |

Include/Exclude is sent as `include: false` on the reference body for exclude references. Cascade params appended as query params per `CascadeSelector` output.

---

### Validation

- **Add button disabled** when: expression field is empty in Expression mode or no IDs are present in Add by ID(s) mode
- **Add by ID(s) requires a source or collection seed**: the ID textarea is disabled with tooltip *"Select a source or collection above to use Add by ID(s)"* until a repo is seeded
- **Mode exclusivity**: Expression mode hides Add by ID(s) inputs; Add by ID(s) mode hides the Expression field and uses Source / Collection, Version, and Reference Type as the visible generated-path context
- **Version mismatch**: when the user has checked "Pin to a specific version" and selected a version that differs from the collection's locked source version, the warning is shown inline below the version dropdown. The three action choices (add unversioned + rebuild / add without rebuilding / pin to this version) are presented in the dialog's submit area if a mismatch is detected (same pattern as `AddToCollectionDialog` — see `§ Version Consistency Warning`)

---

### Post-M42: Intensional Filter Builder

The Intensional reference pattern (`/orgs/:org/sources/:source/concepts/?q=...`) can be authored via the Expression field today using raw query params. A structured filter builder UI (Property | Operator | Value rows) that generates the query string automatically is deferred. See `tbv3-deferred-features.md`.

---

### Visual Mockup

Clickable workflow mockups:
- `04_surfaces/mockups/add-references-workflow.html`
- `04_surfaces/mockups/add-references-workflow-ocl.html` (OCL-styled copy using oclweb3/design-system cues)

*Default state (no seed selected):*
```
┌──────────────────────────────────────────────────────────┐
│ Add References                                      [X]  │
├──────────────────────────────────────────────────────────┤
│ Source / Collection                                      │
│ [🔍 Search sources and collections...           ]       │
│                                                          │
│ [Add by ID(s)] [Expression]                             │
│                                                          │
│ Add by ID(s)                                            │
│ [Select a source or collection to add by ID(s)]          │
│                                                          │
│ Include / Exclude        Cascade                         │
│ [Include            ▼]  [None               ▼]         │
│                                                          │
│ [Preview ▼]                                             │
├──────────────────────────────────────────────────────────┤
│                          [Cancel]  [Add Reference(s)]   │
└──────────────────────────────────────────────────────────┘
```

*After source selected (collection has a locked version — versionless default):*
```
┌──────────────────────────────────────────────────────────┐
│ Add References                                      [X]  │
├──────────────────────────────────────────────────────────┤
│ Source / Collection                                      │
│ [CIEL (CIEL)                                    ]       │
│ Resolves to v2024-08-01 via the collection's expansion  │
│                                                          │
│ [ ] Pin to a specific version          Reference Type    │
│                                        [Concepts      ▼] │
│                                                          │
│ [Add by ID(s)] [Expression]                             │
│ Add by ID(s)                                            │
│ IDs (comma-separated, for selected Reference Type):      │
│ [1234, 5678, 9012                               ]       │
│                                                          │
│ Include / Exclude        Cascade                         │
│ [Include            ▼]  [Source Mappings    ▼]         │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                          [Cancel]  [Add Reference(s)]   │
└──────────────────────────────────────────────────────────┘
```

*After checking "Pin to a specific version":*
```
│ [✓] Pin to a specific version          Reference Type    │
│ [v2024-08-01                        ▼] [Concepts      ▼] │
```

*User pinned to a version that differs from the collection's locked version:*
```
│ [✓] Pin to a specific version                            │
│ [v2023-10-01                        ▼]                  │
│ ⚠ Your collection is pinned to CIEL v2024-08-01.        │
│   This pinned reference will differ from the            │
│   versionless resolution.                               │
```

*(Version dropdown open):*
```
│ ┌─────────────────────────────────────────────────────┐ │
│ │ ✓ v2024-08-01                                       │ │
│ │   v2023-10-01                                       │ │
│ │   v2022-06-15                                       │ │
│ └─────────────────────────────────────────────────────┘ │
```

*Add by ID(s) mode:*
```
│ Add by ID(s)                                            │
│ IDs (comma-separated, for selected Reference Type):      │
│ [1234, 5678, 9012, 3456                         ]       │
│ Expression is hidden while Add by ID(s) mode is active. │
```

---

## Reference Preview (Without Persisting)

> **Ticket:** ocl_issues#2007 — CTA / Reference / Preview

Users can evaluate what a reference expression would resolve to before committing it. This is non-destructive: nothing is added to or removed from the collection.

### M42 Scope

| Entry Point | M42? |
|---|---|
| Preview panel within the Add to Collection dialog (`AddToCollectionDialog.jsx`) | ✅ M42 |
| Preview panel within the Add References dialog (`AddReferencesDialog.jsx`) | ✅ M42 (built as part of #2431) |
| Preview action on an existing reference row in the References list | ✅ M42 |

---

### API

**Endpoint:** `POST /:ownerType/:owner/collections/:collection/references/preview/`

The endpoint accepts multiple expressions in a single call and returns one result object per expression. This is important for batch operations (e.g., previewing Add by ID(s) entries before adding them all at once via the Add References dialog).

**Request body:**
```json
{
  "data": {
    "expressions": [
      "/orgs/CIEL/sources/CIEL/concepts/1234/",
      "/orgs/CIEL/sources/CIEL/concepts/5678/"
    ]
  },
  "cascade": "sourcemappings"
}
```

**Query params (appended as needed):**
- `cascade` — cascade method string (e.g., `sourcemappings`, `sourcetoconcepts`)
- `transformReferences` — optional transform (e.g., `openmrs_concept_reference`)
- Additional cascade params as needed (`mapTypes`, `cascadeLevels`, `returnMapTypes`)

**Response:** Array of preview result objects, one entry per input expression:
```json
[
  {
    "reference": "/orgs/CIEL/sources/CIEL/concepts/1234/",
    "new": {
      "concepts": [ /* ConceptListSerializer[] — first 25 */ ],
      "mappings": [ /* MappingListSerializer[] — first 25 */ ],
      "concepts_count": 1,
      "mappings_count": 3
    },
    "existing": {
      "concepts": [ /* ConceptListSerializer[] — first 25 */ ],
      "mappings": [ /* MappingListSerializer[] — first 25 */ ],
      "concepts_count": 0,
      "mappings_count": 0
    },
    "version_status": {
      "consistent": true,
      "canonical_source_version": "CIEL v2024-08-01",
      "resolved_source_version": "CIEL v2024-08-01"
    },
    "errors": [],
    "exclude": false
  },
  {
    "reference": "/orgs/CIEL/sources/CIEL/concepts/5678/",
    "new": { "concepts": [], "mappings": [], "concepts_count": 0, "mappings_count": 0 },
    "existing": { "concepts": [], "mappings": [], "concepts_count": 0, "mappings_count": 0 },
    "version_status": null,
    "errors": [
      { "type": "source_not_found", "message": "Source CIEL not found or not accessible." }
    ],
    "exclude": false
  }
]
```

> **API enhancement required:** The current `CollectionReferencesPreview` view returns a flat `concepts` / `mappings` array with no collection-membership awareness, version checking, or error taxonomy. The endpoint needs to be updated to:
> 1. Accept multiple expressions and return one result object per expression
> 2. Split resolved resources into `new` vs. `existing` groups by checking each against the collection's current expansion
> 3. Populate `version_status` by comparing the resolved source version against the auto-expansion's locked source version (see [Version Consistency](#version-consistency-warning) above)
> 4. Populate `errors` with typed error objects for any expression that cannot be evaluated
>
> The `concepts` / `mappings` arrays within each group are capped at 25 items each; use the `*_count` fields for display totals.

---

### Warnings and Errors

The preview response surfaces two distinct concern types per expression. These are displayed in the panel above the result rows, one alert per affected expression.

#### Version Inconsistency (Warning)

Shown when one or more result objects have `version_status.consistent === false`. This is the same condition described in [Version Consistency Warning](#version-consistency-warning) above — surfaced here so the user sees the conflict before deciding to submit.

The warning is a single collapsible alert regardless of how many expressions are affected. It does not repeat per-expression.

**Collapsed (default):**
> ⚠️ **{N} reference(s) will resolve to a different source version than your collection is pinned to.** ▼

**Expanded (user clicks ▼):**

A table drops down listing each affected expression:

| Reference | Resolves to | Collection pinned to |
|---|---|---|
| `CIEL/concepts/1234/` | CIEL v2023-10-01 | CIEL v2024-08-01 |
| `CIEL/concepts/5678/` | CIEL v2023-10-01 | CIEL v2024-08-01 |

- Expression column shows the concept ID portion of the URL (not the full path) for readability
- Resolved version and collection version are shown as source+version chips
- If all affected references resolve to the same mismatched version, the table makes this obvious at a glance

The alert is informational only within the preview panel. The action choices (Add unversioned + rebuild / Add unversioned without rebuilding / Pin to this version) are presented in the parent dialog's submit area, not inline in the panel.

If `version_status` is `null` for all results (collection has no locked source version yet), no version warning is shown.

> **Unresolvable Reference (Error) is post-M42** — see `tbv3-deferred-features.md § Reference Preview: Unresolvable Reference Error Taxonomy`. For M42, expressions that fail to resolve are shown as empty results; no per-type error messaging is displayed.

#### Zero-result reference (not an error, a warning)

When `errors` is empty but `new.concepts_count + new.mappings_count + existing.concepts_count + existing.mappings_count === 0`, show a soft warning rather than an error:

> ⚠️ This reference resolves to 0 resources. It will be saved as an empty reference.

This is a valid state (intensional references may legitimately return zero results today but resolve in the future), so saving is not blocked — the user just needs to acknowledge.

---

### Component: `ReferencePreviewPanel`

**File:** `src/components/common/ReferencePreviewPanel.jsx`

**Props:**

| Prop | Type | Description |
|---|---|---|
| `collectionUrl` | `string` | Full collection URL; used to build the preview API path |
| `expressions` | `string[]` | One or more reference expressions to preview in a single call |
| `cascadeParams` | `object` | Cascade options object from `CascadeSelector` |

> When called from a single-expression context (e.g., Add to Collection dialog), pass a one-element array.

**States:**

| State | UI |
|---|---|
| Idle (not yet triggered) | Nothing rendered; parent controls when to mount/show |
| Loading | `CircularProgress` spinner + "Evaluating reference…" caption |
| Success — results | Warning/error alerts (if any) + summary bar + grouped result rows |
| Error (request failure) | `Alert severity="error"` with the API error message |

**State mockups:**

*Loading:*
```
┌─────────────────────────────────────────────────────┐
│  ⟳  Evaluating reference…                           │
└─────────────────────────────────────────────────────┘
```

*Results — version mismatch warning, collapsed (default):*
```
┌─────────────────────────────────────────────────────┐
│  ⚠  4 references will resolve to a different source │
│     version than your collection is pinned to.  ▼  │
├─────────────────────────────────────────────────────┤
│  12 new  ·  3 already in collection                 │
├─────────────────────────────────────────────────────┤
│  ✅ New (12)                                         │
│  ...                                                │
└─────────────────────────────────────────────────────┘
```

*Results — version mismatch warning, expanded:*
```
┌─────────────────────────────────────────────────────┐
│  ⚠  4 references will resolve to a different source │
│     version than your collection is pinned to.  ▲  │
│                                                     │
│  Reference            Resolves to    Pinned to      │
│  ─────────────────────────────────────────────────  │
│  CIEL/concepts/1234/  [CIEL v2023]   [CIEL v2024]   │
│  CIEL/concepts/5678/  [CIEL v2023]   [CIEL v2024]   │
│  CIEL/concepts/9012/  [CIEL v2023]   [CIEL v2024]   │
│  CIEL/concepts/3456/  [CIEL v2023]   [CIEL v2024]   │
├─────────────────────────────────────────────────────┤
│  12 new  ·  3 already in collection                 │
├─────────────────────────────────────────────────────┤
│  ✅ New (12)                                         │
│  ─────────────────────────────────────────────────  │
│  [CIEL]  1234 · Malaria, unspecified   [2 mappings] │
│      [SAME-AS]  ICD-10-WHO: A09                     │
│      [NARROWER-THAN]  SNOMED: 61462000              │
│  ─────────────────────────────────────────────────  │
│  [CIEL]  1380 · Nutrition counselling               │
│  ─────────────────────────────────────────────────  │
│  [CIEL]  5089 · Weight                 [1 mapping]  │
│      [SAME-AS]  LOINC: 3141-9                       │
│  … and 9 more concepts                              │
├─────────────────────────────────────────────────────┤
│  ⚠  Already in collection (3)                       │
│  ─────────────────────────────────────────────────  │
│  [CIEL]  1855 · Amount of substance                 │
│  [CIEL]  2006 · Diagnosis                           │
│  [CIEL]  3031 · Body weight                         │
└─────────────────────────────────────────────────────┘
```

*Results — zero-result warning (empty intensional reference):*
```
┌─────────────────────────────────────────────────────┐
│  ⚠  This reference resolves to 0 resources. It will │
│     be saved as an empty reference.                 │
└─────────────────────────────────────────────────────┘
```

---

### Result Grouping

Results arrive pre-grouped from the API. Each expression's result object contains `new` and `existing` sub-objects; resources with `errors` have empty `new` and `existing` objects.

| Group | Icon | Source in response |
|---|---|---|
| **New** | ✅ green | `result.new` — resources not yet in the collection's current expansion |
| **Already in collection** | ⚠️ yellow | `result.existing` — resources already in the collection's current expansion |
| **Unresolvable** | ❌ red | `result.errors` is non-empty |

When previewing multiple expressions, results for all expressions are merged into these groups (New, Already in Collection, Unresolvable). Version inconsistency and zero-result warnings are shown per-expression above the merged result list.

---

### Summary Bar

Shown above the result rows (below any per-expression warning/error alerts):

> `X new · Y already in collection · Z unresolvable`

Zero-count segments are omitted. When multiple expressions are previewed, counts are aggregated across all expressions.

---

### Result Row Display

Each row shows one concept or mapping. No selection controls — rows are read-only.

**Concept row:**
```
[CIEL] 1234 · Malaria, unspecified          [1 mapping]
```
- Source chip (abbreviated source name)
- Concept ID · Display name
- Mapping count badge if mappings > 0

**Mapping row (shown below its parent concept, indented):**
```
    [SAME-AS]  ICD-10-WHO: A09
```
- Map type chip
- Target code · Target source (external) or concept chip (internal)

**Pagination / overflow:**
- If the total across all groups exceeds 100 rows: show first 100, then a muted footer: `"…and N more resources. Add the reference to see the full list."`
- Within a group, if the `concepts` or `mappings` array is capped at 25 but the `*_count` is higher, show an inline "…and N more [concepts/mappings]" note within that group

---

### Integration: Add to Collection Dialog

In `AddToCollectionDialog.jsx`:

- Add a collapsible "Preview" section below the `CascadeSelector` and above the Submit button
- Collapsed by default; toggled open by a `Preview` chip-button
- When open, `ReferencePreviewPanel` is mounted with `expressions={[conceptUrl]}`, `cascadeParams`, and `collectionUrl`
- Preview updates live when cascade params change (re-fires on each cascade change while panel is open)
- **Version consistency:** If the preview returns `version_status.consistent === false`, the version mismatch alert is shown in the panel. The Submit area below should also reflect the three action choices described in [Version Consistency Warning](#version-consistency-warning) (Add unversioned + rebuild / Add unversioned without rebuilding / Pin to version)

---

### Integration: Add References Dialog

In `AddReferencesDialog.jsx` (to be built as part of #2431):

- "Preview results" toggle below the expression/cascade fields; collapsed by default
- When open, `ReferencePreviewPanel` is mounted with the current expressions array and cascade params
- Preview fires on user action only. After changes, show a stale-results warning and a **Refresh Preview** action instead of auto-refreshing.
- Multiple expressions from Add by ID(s) are passed as a single batch; results are merged in the panel

---

## Reference Presentation in the References Tab

> **Milestone note:** The flat list view and detail split view are part of the current milestone. Bulk Transform and Remove remain in MVP scope but land in a later MVP milestone. Inline row expansion in the list itself and Group by Source are also later-milestone enhancements — see `04_surfaces/references-tab.md` and the pending tickets in `tbv3-ticket-execution-log.md` Section 6. The concept → mapping tree is already used inside the detail split view's Expansion tab for MVP.

### List View
- Table columns: Expression | Type (Extensional/Intensional/Cascade/Exclude) | Resolved Count | Version Pinned? | Status
- Resolved Count: number of concepts/mappings this reference evaluates to in the current expansion
- Version Pinned: badge shown if the reference is locked to a specific source version; warning badge if that version differs from the auto-expansion's locked source version
- Status: Active | Warning | Error

### Reference Detail (Row Click → Split View)
- Full expression (not truncated)
- Reference type and cascade settings
- Uses the same split-view interaction model as Concept Detail
- Header-level `Actions ▾` menu includes reference utilities such as download and share
- Footer shows the last-resolved timestamp plus quick download/share actions
- Expansion tab shows the resolved resources as an expandable concept → mapping tree, including an orphaned mappings group when applicable
- "Trace back": any concept in the expansion can link to the reference(s) that brought it in

---

## Reference Transforms (HEAD Only)

> **Ticket:** ocl_issues#2433 - CTA / Reference / Transform
> **Reference ticket:** ocl_issues#2274 - checkbox-based actions for references

Transforms let an editor change the version pinning strategy or expression pattern for selected collection references in bulk. They are only available on the collection's HEAD version. This is distinct from Cascade's Transform option, which is used while adding references and sends `transformReferences` to the preview/add API.

> **Design context — scope and audience:** Most users who want to move their collection from one CIEL version to another should use the [Update Collection Source Version workflow](../03_workflows/update-collection-source-version.md) instead. Reference Transforms are a power-user tool for targeted, individual reference changes — not the primary path for collection maintenance. When in doubt, direct users to the source version update workflow first.
>
> **Implementation approach:** A wizard-style multi-step flow (one expandable panel per reference group, multiple transform selectors, async preview per group) was reviewed in design discussion and found to be too complex for the common case. The M42 implementation should favor a simpler single-dialog pattern: selected reference summary by group, one transform type selector per group, target version input where needed, and a before/after preview table. The more elaborate per-group wizard UX is post-M42 if it proves necessary.
>
> **Resource-versioned reference cleanup — backend-first:** The cleanup of deprecated resource-versioned references (Transform #3 below) is being handled via a backend data migration analysis ([#2504](https://github.com/OpenConceptLab/ocl_issues/issues/2504)) before any UI is built. If the blast radius is contained and references can be unambiguously resolved, Sunny will run a backend migration that eliminates the need for the UI transform entirely. The UI implementation of Transform #3 is **blocked pending #2504 analysis and recommendation**.

### #2433 Scope

- Entry point: Collection -> References tab -> select one or more references with checkboxes -> `Transform`
- Applies to selected collection references only; no row-level transform action in MVP
- Supports bulk operation with before/after preview before commit
- Mixed selections are grouped by reference/pinning type so each group can expose only the transform options that apply to it
- Preserves reference metadata that is not part of the selected transform, including include/exclude state and cascade settings, unless the transform type explicitly changes it
- Successful transform queues re-expansion of the collection HEAD
- Released collection versions are immutable; transform controls are disabled with the same HEAD-only messaging used by Add and Remove

### #2433 Available Transforms

1. **Unpin to HEAD / non-versioned reference** - Removes source or resource version pinning so the reference resolves dynamically through the collection expansion strategy
   - Repo-versioned source path: `/:owner/sources/:source/:repoVersion/concepts/:id/` -> `/:owner/sources/:source/concepts/:id/`
   - Deprecated resource-versioned path: `/:owner/sources/:source/concepts/:id/:resourceVersion/` -> `/:owner/sources/:source/concepts/:id/`
   - Use when: cleaning up old or overly pinned references, including the #2274 "Transform to non-versioned reference" workflow
2. **Lock to repo version** - Changes an unversioned reference to a repo-versioned reference
   - `/:owner/sources/:source/concepts/:id/` -> `/:owner/sources/:source/:repoVersion/concepts/:id/`
   - Default target version: the collection expansion's currently locked source version for that source, when available
   - If no locked version exists, user must choose a released source version; default to the latest released version if the API can determine it
   - Use when: the user wants to freeze selected HEAD-resolving references to a specific released source version
3. **Convert deprecated resource-versioned reference to repo-versioned reference** - Rewrites old resource-versioned references into the modern repo-versioned pattern
   - `/:owner/sources/:source/concepts/:id/:resourceVersion/` -> `/:owner/sources/:source/:repoVersion/concepts/:id/`
   - If the resource version maps to exactly one released repo version, preselect that version
   - If it maps to multiple possible repo versions, user must choose a target version or rule before preview can be committed
   - Use when: the user wants to keep version pinning while removing deprecated resource-versioned URLs
   - **Status: blocked pending #2504.** The backend migration analysis will determine whether this transform is needed as a UI feature at all. Do not implement until #2504 is resolved. See design context above.
4. **Expression rewrite** *(post-M42)* - Applies a controlled expression rewrite across selected references
   - Intended for consistent pattern changes, such as replacing a base repo path, converting a concepts expression to a mappings expression, or updating query parameters on intensional references
   - Must validate that each rewritten expression is parseable and previewable before commit
   - Free-form text replacement is allowed only if the preview reports each before/after expression and blocks invalid rewrites

> **Post-v3:** A transform that lets a user update the pinned version of a repo-versioned reference (e.g., from `v2023` to `v2024`) without changing the reference pattern is deferred to post-v3. See `tbv3-deferred-features.md`.

### #2433 Eligibility

| Transform | Valid selected references | Invalid selected references |
|---|---|---|
| Unpin to HEAD / non-versioned | Repo-versioned references; deprecated resource-versioned references | Already unversioned references; references without a source/collection version segment |
| Lock to repo version | Unversioned source/collection references | Already pinned references; expressions whose target repo/version cannot be identified |
| Convert deprecated resource-versioned to repo-versioned *(blocked — see #2504)* | Deprecated resource-versioned references | Unversioned references; already repo-versioned references |
| Expression rewrite *(post-M42)* | References that match the configured rewrite rule and can be parsed after rewrite | Non-matching references; invalid rewritten expressions; references whose preview fails |

Bulk transforms may include invalid selected references. Invalid references are skipped, not silently changed, and are shown in the preview report with a reason.

### #2433 Mixed Selection Grouping

When selected references include different expression or pinning patterns, the dialog groups them before asking for transform inputs. The grouping should be based on what options are valid, not only on display labels.

| Group | Examples | Available transform options |
|---|---|---|
| Unversioned references | `/:owner/sources/:source/concepts/:id/` | Lock to repo version; Expression rewrite *(post-M42)* |
| Repo-versioned references | `/:owner/sources/:source/:repoVersion/concepts/:id/` | Unpin to HEAD / non-versioned; Expression rewrite *(post-M42)* |
| Deprecated resource-versioned references | `/:owner/sources/:source/concepts/:id/:resourceVersion/` | Unpin to HEAD / non-versioned; Convert deprecated resource-versioned to repo-versioned *(blocked — #2504)*; Expression rewrite *(post-M42)* |
| Intensional/filter references | `/:owner/sources/:source/concepts/?q=...` | Transform options depend on whether the base repo path is pinned; Expression rewrite *(post-M42)* |
| Unsupported/unknown references | Expressions that cannot be parsed or classified | No transform; shown as skipped with reason |

Dialog behavior for mixed selections:
- Show a selected-reference summary by group, e.g. `8 selected: 3 unversioned, 4 repo-versioned, 1 unsupported`
- Each group has its own transform selector and required inputs
- A user may transform one group while leaving another group unchanged
- Groups with no valid transform remain visible as skipped so the user can see why they are not affected
- The final commit submits only rows that are Ready within groups that have an active transform selected

### #2433 Transform Dialog

> **M42 implementation:** Keep this dialog simple. The goal is a single lightweight dialog, not a per-group wizard. The per-group expandable panel design (described in full below) is post-M42 if the simpler pattern proves insufficient.

**M42 dialog (simple pattern):**
1. User selects references in the References tab
2. User clicks `Transform`
3. Dialog opens with:
   - Selected count and a summary line grouping references by type (e.g., `8 selected: 5 unversioned, 2 repo-versioned, 1 unsupported`)
   - For each group with valid transform options: transform type selector + target version input if needed
   - A before/after preview table (current expression → proposed expression, with resolved version/count where available)
   - Clear status per row: Ready, Skipped (with reason), Warning, Error
4. Commit button enabled when at least one row is Ready
5. On commit: only Ready rows are submitted; results shown inline with transformed/skipped/failed counts; collection HEAD re-expansion is queued

**Post-M42 wizard (full per-group flow):**
Each reference group gets its own expandable panel with independent transform selector, target version picker, expression rewrite controls, and per-row async preview. This full design is specified in detail in the original #2433 ticket for reference if needed.

Preview table row (both patterns):
- Current expression
- Proposed expression
- Current resolved source/version and result count, when available
- Proposed resolved source/version and result count, when previewable
- Status: Ready, Skipped, Warning, Error, Preview pending, or Preview too large
- Reason for skipped/error rows

### #2433 Preview and Safety Rules

- Preview is required before commit; users must see before/after expressions for every row that will be submitted
- The dialog must make partial application explicit: `X will be transformed`, `Y will be skipped`, `Z need attention`
- If a proposed transform changes resolved content count, show a warning rather than blocking by default
- If a proposed transform resolves to zero resources, show a warning and require explicit confirmation
- If a proposed transform cannot be previewed or produces an invalid expression, block that row from commit
- If every selected reference is invalid for the selected transform, disable commit and explain why
- Transform preview should use the same reference preview semantics as Add References where possible, so version mismatch and zero-result warnings behave consistently

### #2433 Large Reference Preview Handling

Some references resolve to very large result sets, and OCL may not be able to resolve them synchronously during transform preview. The transform dialog must handle this without timing out or blocking the whole selection.

- The preview has two layers:
  - **Structural preview**: validate the current expression, proposed expression, transform eligibility, target repo/version, include/exclude state, and cascade metadata without resolving all referenced resources
  - **Resolution preview**: compare resolved source/version and resolved resource counts when the API can provide them quickly
- A row can be `Ready` based on structural preview even when full resolution preview is unavailable, provided the expression is valid and the transform is deterministic
- If full resolution is too expensive, show `Preview too large` or `Resolution preview queued` in the resolution columns instead of an error
- Large-preview rows must clearly state that resolved counts/content changes could not be confirmed before commit
- If the API supports async preview jobs, the dialog should poll or refresh job status and update rows from `Preview pending` to Ready, Warning, Error, or Preview too large
- Async preview must be scoped per row or group so one large reference does not prevent smaller references from becoming Ready
- If an async preview is still pending, the user can commit only rows whose structural preview is Ready; pending rows remain unsubmitted unless the user explicitly chooses to include structurally ready rows without resolved counts
- Transform commit should not require downloading or rendering the resolved resource list for very large references
- After commit, the queued collection re-expansion remains the authoritative confirmation of final resolved content

### #2433 API Requirements

- Preferred API shape: a bulk transform endpoint that accepts selected reference IDs plus transform parameters and returns per-reference preview/commit results
- Preview responses should distinguish structural validation from full resolution, including statuses for async pending and too-large-to-resolve references
- If only lower-level update/delete/add APIs exist, the client must still present the operation as one bulk transform with per-reference results and must avoid losing include/exclude and cascade metadata
- Commit must be idempotent enough that retrying after a partial failure does not duplicate references
- The server response must expose enough data to refresh changed reference rows without a full page reload; a full list refresh is acceptable as a fallback

### #2433 Acceptance Criteria

- [ ] References tab supports checkbox selection for reference rows and a header checkbox for visible rows
- [ ] Transform control is enabled when one or more references are selected on HEAD and disabled on released versions
- [ ] Mixed selected references are grouped by reference/pinning type with a summary line in the dialog
- [ ] Transform dialog supports Unpin to HEAD / non-versioned reference
- [ ] Transform dialog supports Lock to repo version
- [ ] *(Blocked — pending #2504)* Transform dialog supports converting deprecated resource-versioned references to repo-versioned references — implement only if #2504 determines backend migration is insufficient
- [ ] *(Post-M42)* Transform dialog supports expression rewrite with before/after preview
- [ ] Invalid selected references are skipped with clear reasons
- [ ] User sees before/after expression, resolved version, and resolved count before commit
- [ ] Large references that cannot be resolved synchronously show structural preview plus a clear async/too-large resolution status
- [ ] Commit applies only eligible/ready rows and reports transformed/skipped/failed counts
- [ ] Successful transform queues or triggers collection HEAD re-expansion and updates the References tab

### #2433 Open Questions

- Confirm backend endpoint and request/response shape for transform preview and commit
- ~~Confirm whether expression rewrite is in M42 MVP or should be a later milestone~~ — **resolved: post-M42**
- ~~Confirm how to map deprecated resource-level versions to repo versions when more than one released repo version contains the resource version~~ — **resolved: blocked pending #2504 analysis; backend migration preferred over UI**
- Confirm whether transform should preserve the original reference ID or create a replacement reference with a new ID
- Confirm whether "Lock to repo version" should default to the expansion's locked version, latest released version, or require explicit user choice when no expansion lock exists
- Confirm API threshold and status vocabulary for large references that cannot be synchronously resolved during preview
- Confirm whether users may commit structurally valid transforms when resolved counts are unavailable, or whether that should require an explicit confirmation checkbox

### Legacy Name Mapping

- Earlier SOW notes called `Unpin to HEAD / non-versioned reference` "Transform to Unversioned"
- Earlier SOW notes called `Convert deprecated resource-versioned reference to repo-versioned reference` "Transform to Repo-Versioned"
- Earlier SOW notes called `Lock to repo version` "Lock to Repo Version"

---

## Removing References

> **Ticket:** ocl_issues#2008 — CTA / Reference / Delete ✅ implemented

### Bulk Remove from References Tab

**Entry point:** Collection → References tab → checkbox selection → "Remove selected" (bulk action only; no row action menu)

**Confirmation dialog** (shown before deletion):
> "Remove [X] reference(s)?"
> "This will remove [Y] concepts and [Z] mappings from the collection expansion."

- Only possible on **HEAD** version (released versions are immutable)
- Successful removal queues a re-expansion of HEAD
- Dialog has Cancel and Remove (destructive) buttons — no type-to-confirm required

**API:** `DELETE /:ownerType/:owner/collections/:collection/references/` with body `{ "ids": [id1, id2, ...] }`

**Implementation (oclweb3):**
- `DeleteReferencesDialog` component: `src/components/collections/DeleteReferencesDialog.jsx`
- Delete button rendered as `toolbarControl` in `Search.jsx` when `resource === 'references'` and items are selected
- HEAD-only: button is disabled with tooltip "Not available in saved versions. Switch to HEAD to edit." when not on HEAD; derived from `props.url?.includes('/HEAD/')`
- Reference IDs sourced from selected result objects matched by `r.id`; concept/mapping counts summed from `reference.concepts` and `reference.mappings` fields

---

### Remove from Collection via Concept/Mapping Detail ✅ implemented

A secondary removal path: a user viewing a concept or mapping inside a collection can remove it directly from its detail split view without navigating to the References tab.

**Entry point:** Collection → Concepts/Mappings tab → select concept or mapping → detail panel → "Remove from collection" button (header area)

**Behavior:**
- Only visible when viewing a concept/mapping in a collection context (URL contains `/collections/`) AND on HEAD AND user has edit access
- Clicking the button opens `RemoveFromCollectionDialog`, which lists the reference expression(s) that bring this concept/mapping into the collection
- On confirm, deletes those references via the same `DELETE /references/` API call (using IDs from `concept.references` / `mapping.references`)
- On success, closes the detail panel

**References section in detail view:**
When a concept or mapping is fetched in a collection context, the API is called with `includeReferences=true`. The response includes a `references` array on the concept/mapping object. This is rendered as a "References" section at the bottom of the detail view (below checksums), showing each reference expression that brings the item into the collection.

**Implementation (oclweb3):**
- `RemoveFromCollectionDialog` component: `src/components/collections/RemoveFromCollectionDialog.jsx`
- `ResourceReferences` display component: `src/components/common/ResourceReferences.jsx` — used in both `ConceptDetails.jsx` and `MappingDetails.jsx`
- Collection context detected in `ConceptHome` / `MappingHome` via `props.repo?.type?.includes('Collection') || props.url?.includes('/collections/')`
- Fetch param: `includeReferences: true` added to concept/mapping GET when in collection context
- Remove button in `ConceptHeader.jsx` / `MappingHeader.jsx` — shown instead of "Add to Collection" when `isInCollection && repo?.version === 'HEAD'`

---

### Open Design Question: Removing a Concept Covered by Multiple References

> **Raised by:** [ocl_issues#2644](https://github.com/OpenConceptLab/ocl_issues/issues/2644) (MSF report, comment [2026-07-2x](https://github.com/OpenConceptLab/ocl_issues/issues/2644#issuecomment-5068497102)). **Needs team discussion before oclweb3 builds this out further.**

Neither remove path documented above (Bulk Remove from References Tab, Remove from Collection via Concept/Mapping Detail) addresses what happens when the concept/mapping being removed is brought into the collection by **more than one reference** (e.g. a specific reference *and* a broader cascade/wildcard reference that also resolves to it). Simply deleting one reference in that case doesn't remove the concept from the expansion — the other reference still resolves to it.

**How oclweb2 (legacy, current production) handles this today:** its "Remove Resources by Reference" dialog (`CollectionHomeChildrenList.jsx`) detects the multi-reference case (`shouldMakeExclusion`) and recommends **"Exclude concept(s)/mapping(s) from collection"** instead of a plain delete. Exclude creates a *new, independent* `CollectionReference` row with `include: false` targeting just that concept — it does not modify or link to the reference(s) that pulled the concept in.

**The problem, per the backend model (`oclapi2/core/collections/models.py`):**
- `CollectionReference.include=False` rows have no foreign-key/grouping relationship to the include reference(s) they were created to counteract (models.py:521-561).
- Every exclude reference in a collection version is permanently re-applied on **every subsequent** `add_references` call (`Expansion.add_references`, models.py:1367-1373) — it's a standing filter, not a one-time/paired operation.
- Neither `Collection.delete_references` (models.py:310-320, matches by expression string) nor `Expansion.delete_references` (models.py:1304-1336, matches by ID) has any logic to find and remove an exclude reference associated with the reference being deleted.
- Net effect: once a concept has been "excluded," that exclude reference is orphaned the moment the original broader/cascade reference is later removed. It keeps suppressing the concept from the expansion indefinitely until a user manually finds and deletes the stray "Exclude concept – &lt;ID&gt;" reference — which is exactly what MSF hit (concepts silently missing from OpenMRS after reference cleanup).
- Compounding factor: `Concept.cascade()` (`oclapi2/core/concepts/models.py:1405-1424`) traverses **all** mapping types by default, including SAME-AS. A cascade/hierarchy reference can re-pull an "excluded" concept back into the expansion via a SAME-AS-linked sibling concept, independent of the exclude reference — MSF resolved one of their two cases only after deleting a conflicting SAME-AS mapping, not by fixing the exclude reference itself.

**Open questions for the team:**
1. Should an exclude reference be linked (FK or grouping) to the include reference(s) it was created to counteract, so removing the include reference offers/forces cleanup of its paired exclude reference?
2. Should exclude references be scoped/paired at all, or is the current "standing filter" behavior intentional — and if so, how should the UI make existing exclude references for a concept impossible to miss when a user is removing/re-adding that concept?
3. Should cascade traversal (e.g. via SAME-AS mappings) respect exclude references consistently, regardless of which reference path re-introduces the concept?
4. Does this problem, and its fix, need to be re-litigated for oclweb3's `DeleteReferencesDialog` / `RemoveFromCollectionDialog`, which currently don't implement the multi-reference detection oclweb2 has at all (see above) — should oclweb3 implement the same recommend-exclude behavior, a fixed version of it, or a different mechanism entirely?

See [ocl_issues#2644](https://github.com/OpenConceptLab/ocl_issues/issues/2644) for the full MSF report and reproduction details. A separate, narrower UI bug from the same report (oclweb2's "Remove Resources by Reference" dialog not showing success feedback on the plain-delete path) is tracked/fixed independently and is not part of this design question.

---

## Linked Source: Resolve to HEAD During Updates

> **Decision (2026-05-13, ADR-007):** HEAD-resolution is allowed for any source — not limited to sources the user owns — provided the source has explicitly enabled it via a repository-level setting (`allow_head_as_linked_source: true`). Source owners control this opt-in; it is off by default.

### What Linked Source Does

A **linked source** is a per-collection, per-source configuration that overrides which version of a source is used when resolving unversioned references during expansion. By default, unversioned references resolve against the collection's locked source version. Linked source lets the collection point at a different version — including HEAD.

Primary use cases:
- **Pre-release testing:** A source administrator tests new unreleased content in a collection before a formal source release
- **Continuous integration:** A collection that intentionally tracks upstream HEAD (e.g., a CIEL development test collection)

Linked source is **not** the recommended path for Terminology Implementers doing routine collection updates. The standard Update Collection workflow ([`update-collection-source-version.md`](../03_workflows/update-collection-source-version.md)) is the correct path for responding to a new CIEL release.

### Source-Level Setting: `allow_head_as_linked_source`

Source repositories have a boolean setting `allow_head_as_linked_source` (default: `false`).

- When `false`: collections may only link to a specific released version of this source; HEAD is not available as a linked source target
- When `true`: collections may link to this source's HEAD, resolving unversioned references against live (unreleased) content

Source owners configure this in the repository Settings tab under "Advanced." CIEL's public source has this off; private/development source repos typically have it on.

### Linked Source Configuration in a Collection

Collections configure linked sources per referenced source:

- **Setting location:** Collection → Settings → "Linked Sources" section
- **Per-source row:** Source name | Linked version (dropdown: released versions, plus HEAD if `allow_head_as_linked_source = true` on the source)
- **Default:** No linked source configured; behavior is the standard locked source version resolution

### Effect on Reference Resolution

| Collection's linked source config | Reference type | Resolves to |
|---|---|---|
| Not configured | Unversioned | Locked source version (default) |
| Linked to released version v2026-04 | Unversioned | v2026-04 |
| Linked to HEAD (source must allow it) | Unversioned | Source HEAD at expansion time |
| Any | Repo-versioned | The pinned repo version (linked source does not affect these) |

### Snapshot at Version Creation

When a collection version is created, `resolved_repo_versions` on the auto-expansion captures the exact source version used at that moment. For HEAD-linked sources, the HEAD version at creation time becomes the expansion's locked version — the linked source configuration is a HEAD-only setting and does not travel into released versions.

### UI Indicators

- Collection header shows a "Linked Source" chip per configured source (e.g., `CIEL: HEAD` or `CIEL: v2026-04`)
- If the linked source is HEAD and the source has unreleased changes since the last expansion, a "Needs Refresh" indicator appears on affected expansions

### M44 Scope

M44 delivers the foundation: the source-level opt-in setting, the collection-level linked source configuration, and the resolution behavior change. The full **linked source test workflow** — a guided "try this new version, see impact, commit or cancel" UI (from #2347) — is post-M44. The "test without committing" use case in M44 is covered by the **Create Similar Expansion** capability: the user creates a custom expansion with the new source version pinned, reviews the diff, and decides before committing to a rebuild.

---

## Permissions

| Action | Required Permission |
|---|---|
| Preview a reference | Read access to collection and source |
| Add reference (Add to Collection) | Edit access to the collection |
| Create new reference (References tab) | Edit access to the collection |
| Transform reference | Edit access to the collection |
| Remove reference (bulk, from References tab) | Edit access to the collection |
| Remove from collection (via Concept/Mapping detail) | Edit access to the collection |
