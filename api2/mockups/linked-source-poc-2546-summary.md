# HEAD-Link Content Model PoC — Summary

Issue [#2546](https://github.com/OpenConceptLab/ocl_issues/issues/2546) · Milestone: Linked Sources · Related to [#2347](https://github.com/OpenConceptLab/ocl_issues/issues/2347) (Linked Source workflow design) · Environment: `api.staging.openconceptlab.org`

## Goal

Validate whether OCL's existing API and expansion parameters can already represent a "linked source" content model — i.e. a collection in one org that pulls concepts from a source in the **same org**, either always tracking that source's **HEAD** (including unreleased changes) or pinned to a specific **released version**. The goal was to confirm the mechanism end-to-end via the API/CLI before any new UI or endpoints are designed.

## Setup

A dedicated, throwaway test org was created on staging (left in place for now, no real data involved):

- Org: `POC-LinkedSource-2546`
- A small test source with a canonical URL, with a HEAD that has unreleased changes *and* at least one released version
- A test collection in the same org, referencing that source
- Two named expansions on the collection: one using `system-version` pinned to **HEAD**, one pinned to the source's **released version**
- A collection version (`v1.0`) to confirm the mechanism also works on non-HEAD collection versions

## Key Findings

### ✅ Works — `system-version` expansion parameter can represent HEAD-linking and version-pinning

Using the **canonical URL + pipe syntax** — `<canonical_url>|HEAD` or `<canonical_url>|<version>` — the `system-version` parameter correctly overrides the source version used to evaluate a collection reference.

```
POST /orgs/POC-LinkedSource-2546/collections/.../expansions/
{
  "mnemonic": "head-linked",
  "parameters": {
    "system-version": "https://example.org/CodeSystem/test-source|HEAD"
  }
}
```

This expansion's concept list included the unreleased concept added after the last release — confirming true HEAD-linking.

### ✅ Works — Released-version pinning behaves as expected

The equivalent expansion using `"https://example.org/CodeSystem/test-source|v1.0"` resolved *only* to concepts present in that released version — the unreleased concept was correctly excluded.

Both expansions worked identically when evaluated against a non-HEAD collection version (`v1.0`), not just the collection's HEAD.

### ❌ Fails silently — Relative URL + `/HEAD/` syntax does not work

The "obvious" syntax of a relative repo URL with an explicit `/HEAD/` segment (e.g. `/orgs/POC-LinkedSource-2546/sources/test-source/HEAD/`) is **not** a supported `system-version` value. It is not rejected with an error — it is silently recorded in `unresolved_repo_versions` and has no effect on the expansion.

Only the canonical-URL + pipe-syntax form (`|HEAD` / `|<version>`) is recognized.

### ⚠️ Doc correction — Default resolution (no `system-version`) is "latest released version", not HEAD

When a collection reference does not specify a `version` and no `system-version` parameter applies, OCL falls back to the source's **latest released version** — falling back to HEAD only if the source has *no* released versions at all. This contradicted the previous wording in `collectionReferenceEvaluation.md` (which said HEAD is used by default); that doc has been corrected.

### ⚠️ Unimplemented — `check-system-version` / `force-system-version`

These keys exist in `default_expansion_parameters()` as placeholders but have no implemented behavior in the expansion evaluation pipeline today.

## Mechanism Reference Table

| Approach | Syntax | Result |
|---|---|---|
| HEAD-link via expansion parameter | `system-version: <canonical_url>|HEAD` | ✅ Resolves to source HEAD, including unreleased concepts |
| Released-version link via expansion parameter | `system-version: <canonical_url>|<version>` | ✅ Resolves only to that released version's concepts |
| Relative URL + `/HEAD/` | `system-version: /orgs/.../sources/.../HEAD/` | ❌ Silently unresolved (no error, no effect) |
| No `system-version` specified | *(omit parameter)* | ⚠️ Default: latest released version (HEAD only if none released) |

## Implications for HEAD-Link UI & New Endpoints (#2347)

**UI considerations**

- A "Link to HEAD" toggle/option for a linked source must translate to `system-version: <canonical_url>|HEAD` — relative URLs won't work, so the source must have a resolvable canonical URL.
- UI copy explaining "default" behavior should say *latest released version*, not HEAD, to match actual behavior.
- Surface `unresolved_repo_versions` from the verbose expansion response so users can see when a linked reference silently failed to resolve.

**API / endpoint gaps**

- No validation/error is returned for malformed `system-version` values — consider surfacing a warning or rejecting unsupported syntaxes.
- No dedicated "linked source" resource/endpoint exists yet — today this is achieved purely via expansion parameters on a per-expansion basis, which may not be discoverable or persisted at the collection level.
- `check-system-version` / `force-system-version` would need real implementations if the linked-source design relies on them (e.g. to validate a referenced source version still exists).
- Performance of HEAD-linked expansions on a real-sized source (e.g. CIEL) was not tested in this PoC — only a small test source was used.

## Test Artifacts

- Staging org `POC-LinkedSource-2546` left in place (test source, collection, two named expansions, collection version `v1.0`).
- Doc correction applied to `docs/source/oclapi/apireference/collectionReferenceEvaluation.md` (Stage 1 system resolution wording).

---

Generated as a working summary for issue #2546 · staging environment, no production data touched
