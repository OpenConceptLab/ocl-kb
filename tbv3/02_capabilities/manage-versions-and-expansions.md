# Capability: Manage Versions and Expansions

## Scope

Creating, releasing, and managing repository versions. Creating, configuring, and managing collection expansions. Dependency notifications. Locking controls (Rebuild, Create Similar Expansion). Applies to both Sources and Collections, with collection-specific behaviors called out.

---

## Versions + Expansions Tab

This tab is only fully functional for Collections (per SOW). For Sources, the Versions tab shows versions only.

### Tab Location
- Repository → "Versions + Expansions" tab (Collections)
- Repository → "Versions" tab (Sources)

### Versions + Expansions Tab Layout (Collections)

Two-panel layout within the tab:
- **Left panel**: List of all versions (including HEAD) — versions are the primary element; each version row shows its status, whether it has expansions, and which expansion is the default
- **Right panel**: Expansions for the selected version — expansions are children of the selected version; the panel shows all expansions for that version with their parameters and resolved repo versions

Selecting a version in the left panel updates the right panel to show that version's expansions. Each version row should make it clear when an expansion exists and which one is in use — expansions should not be hidden, but versions remain at the forefront of the UI.

See `04_surfaces/versions-expansions-tab.md` for the full layout specification.

---

## Creating a Version

### Entry Point
- Versions tab → "New Version" button (owners/admins only)
- Accessible from both Sources and Collections

### Interaction Flow
1. Dialog opens with:
   - **Version ID**: text input (e.g., "v2024-01-15"); validated for uniqueness within the repo
   - **Description**: textarea for release notes
   - **Released**: checkbox (default: unchecked — creates as Draft first)
   - **Expansion behavior** (Collections only): radio — "Copy current HEAD expansion as-is (default)" | "Re-evaluate all references now"
2. On submit:
   - For Collections: creates the **auto-expansion** using the selected behavior (async); progress indicator shown while it computes
   - For Sources: creates a snapshot of HEAD; HEAD continues as a working copy; indexing and export generation run asynchronously and are shown as a processing state in the version row
3. Success: new version appears in the list; if collection, expansion shows as "Processing" until complete

### Releasing a Draft Version
- Draft versions have a "Release" action button in the version list
- One-click; confirmation dialog: "Release this version? It will become publicly accessible and immutable."
- On release:
  - Version status badge updates to Released
  - For collections: auto-expansion is finalized
  - Notification sent to subscribers (see Dependency Notifications below)

---

## Expansions (Collections Only)

### Auto-Expansion
- Created automatically whenever a collection version is created
- Labeled "auto-expansion" in the UI
- Parameters: `active_only = true`, no pinned repo versions (uses canonical source versions)
- Users do not need to create an expansion manually unless they want different parameters
- Auto-expansion is the default expansion for the version

### Custom Expansions
- Owners can create additional expansions with custom parameters
- Use cases: "What does this collection look like pinned to CIEL 2023 instead of 2024?", "Include retired concepts for audit"

### Creating a Custom Expansion

**Entry Point:** Versions + Expansions tab → Select a version → "New Expansion" button

See `04_surfaces/expansion-form.md` for the full form layout and field specifications.

**Summary of fields:**
- **Expansion ID** (mnemonic) — required; immutable after creation
- **Canonical URL** — optional
- **Parameters**: Active Only (default: on), Source Version Overrides (per-source version table), Date, and advanced FHIR params (collapsed)
- **Set as Default** checkbox

On submit: expansion created; shown as "Processing" while being evaluated. On complete: concept/mapping counts appear; browsable.

### Setting Default Expansion
- Any expansion for a version can be set as default
- "Set as default" action available from expansion row action menu (⋮)
- Only one default per version at a time; setting a new default removes the flag from the current one

### Rebuild Expansion
- Available on any expansion via action menu (⋮) → "Rebuild"
- Re-evaluates the expansion with the same parameters (picks up any underlying source content changes)
- **Confirmation dialog** (shown before triggering):
  > "Rebuild [expansion ID]?"
  > "This will re-evaluate all references using the same parameters. This cannot be undone. To compare before committing, use 'Create Similar' instead."
- **API:** `POST /:owner/collections/:collection/:version/expansions/:expansion/re-evaluate/`
- Shows as "Processing" while rebuilding; non-blocking (user can navigate away)
- On complete: updated counts shown; Needs Refresh indicator removed

### Create Similar Expansion
- Available on any expansion via action menu (⋮) → "Create Similar"
- Opens the "New Expansion" dialog pre-filled with the current expansion's parameters
- User can modify any parameter before saving
- Use case: "Try this expansion but pin CIEL to v2025 instead of v2024"

### Update Expansion Metadata
- Available on any expansion via action menu (⋮) → "Edit"
- Only **Canonical URL** is editable after creation — the expansion ID (mnemonic) is immutable
- Evaluation parameters are **not** editable after creation — use "Create Similar" to create a new expansion with different parameters
- **API:** `PUT /:owner/collections/:collection/:version/expansions/:expansion/` with `{ canonical_url }`
- Does not trigger re-evaluation; the expansion's resolved content is unchanged

### Delete Expansion
- Available on any expansion via action menu (⋮) → "Delete" (owners only)
- Not available on the **default expansion** for a version — set a different expansion as default first
- **Confirmation dialog:**
  > "Delete expansion [ID]?"
  > "This cannot be undone. Concepts and mappings in this collection version will not be affected."
- **API:** `DELETE /:owner/collections/:collection/:version/expansions/:expansion/`

---

## Dependency Notifications

_Maps to SOW Tracker #62 (dependency change notification) and #63 (review and accept upstream updates)._

### Detection: How TBv3 Determines Dependency Staleness

A collection has a dependency on a source when it contains one or more references whose expressions resolve to concepts in that source.

**Phase 1 (M44) — on-demand synchrony check:** When a user visits a collection, TBv3 runs a function that compares the source versions currently resolved by the collection's expansion against the latest released versions of those sources. If any dependency has a newer release available, the function returns a change summary for each stale dependency. No persistent dependency graph or event records are required — this is a synchronous check against version metadata.

The change summary includes:
- **Added concepts**: concepts in the new version not present in the version the collection is resolved to
- **Retired/removed concepts**: concepts that have become inactive or were deleted
- **Modified concepts**: concepts whose names, properties, or mappings changed
- **Affected references**: count of references in the collection that point to changed concepts and mappings

No staleness signal is returned if the collection is already resolved to the latest version of all its dependencies, or if a newer version exists but contains no changes affecting the collection's references.

**Phase 2 (post-M44) — proactive notification infrastructure:** Proactively push staleness information to users when a new source version is released. Design TBD — likely uses the events framework with repo-level following (a collection automatically follows sources it depends on, distinct from a user explicitly following a source). Tracked separately after Part 1 ships.

### What Triggers the Staleness Banner (Phase 1)
- The user visits a collection
- At least one of the collection's resolved source dependencies has a newer released version available
- The newer version contains at least one change affecting concepts referenced by the collection

### Staleness Content
- Which source has a newer version (source chip with name and org)
- The new version ID and release date
- A summary of what changed: "[N] added, [N] retired, [N] modified concepts affect your collection"
- A direct link: "Review updates →" (launches the Update Collection workflow — see `03_workflows/update-collection-source-version.md`)

### Where Staleness Is Surfaced

**Phase 1 (M44):**

**Collection header banner**
- When a user navigates to a collection that has at least one dependency with a newer version available, a persistent yellow banner appears at the top of the collection page:
  > ⚠️ **[Source Name]** released a new version. [N] of your references may be affected. [Review updates →]
- Banner is dismissible per session ("Snooze"); reappears on next visit until the update workflow is completed or dismissed permanently by the owner

**"Needs Refresh" badge on expansions**
- Expansions whose resolved source versions are behind the latest available releases are marked **Needs Refresh**
- Needs Refresh indicator: a yellow dot + "Needs Refresh" badge next to the expansion in the Versions + Expansions tab
- Tooltip: "This expansion was built before [Source] released [versionID]. Rebuild to pick up changes."
- Rebuilding the expansion clears the Needs Refresh indicator

**Phase 2 (post-M44):**

**In-app notification bell (header)**
- Badge count on unread notifications
- Clicking opens the Notification Center

**Notification Center**
- List of all notifications for the user, newest first
- Each item shows: source chip, message, timestamp, "Review updates →" CTA
- Actions: Mark as read, Dismiss

**Notification list item format:**
```
[Source chip: CIEL / OpenConceptLab]
"CIEL released v2024-11-01 on Nov 1, 2024"
"23 concepts in your collection are affected (4 retired, 19 modified)"
[Review updates →]                          2 hours ago
```

**Email notification** (if enabled in user settings; note: sending from ocl.org carries domain reputation risk if messages are marked as spam — design volume carefully)
- Subject: "[Source Name] released a new version — review updates to [Collection Name]"
- Body: same content as in-app notification plus a direct link

---

## Version Locking Controls

### Context
Version locking (see `05_decisions/adrs.md` ADR-003 for full rationale) ensures a collection's auto-expansion consistently resolves references to the same source version. It is the **auto-expansion** that gets locked — not the collection itself. The locking controls in this capability are the user-facing mechanisms.

### "Lock to Current Version" Action
- Available in the collection's References tab when unversioned references exist and a locked source version is identified for the auto-expansion
- Applies the "Lock to Repo Version" transform to all unversioned references for the selected source
- Confirmation dialog showing: current locked source version, number of references that will be locked, preview of transformed expressions

### Locked Source Version Indicator
- Shown in the collection header and Versions + Expansions tab
- Format: "CIEL: v2024-08-01" (or multiple entries if the collection references multiple sources)
- Clicking the indicator opens a summary of all referenced sources and their locked versions

---

## Permissions

| Action | Required Permission |
|---|---|
| View versions and expansions | Read access to repository |
| Create a version | Edit/owner access to repository |
| Release a version | Owner access to repository |
| Create custom expansion | Edit/owner access to collection |
| Rebuild expansion | Edit/owner access to collection |
| Set default expansion | Owner access to collection |
| Delete expansion | Owner access to collection |
| Dismiss dependency notification | Owner access to collection |

---

## Restore/Revert Collection HEAD (Deferred)

> **Status: Deferred (2026-05-13).** The Restore/Revert capability (#2284) — which would allow a collection owner to reset HEAD references to match a previous version's state — is not in scope for M44 or v3. Two design decisions must be resolved before implementation:
>
> 1. **Version history behavior:** Option A — newer versions remain in history; HEAD is silently updated to match the target version's state. Option B — a new version is created that is an explicit copy of the target, preserving a clear audit trail.
> 2. **Expansion handling:** Option A — references are re-evaluated at restore time against current source content. Option B — the target version's expansion is copied as-is, preserving the exact resolved content from that point in time.
>
> When the team is ready to prioritize this feature, resolve both decisions first and use this file's spec sections as the implementation anchor.
