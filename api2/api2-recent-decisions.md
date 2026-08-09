# oclapi2 Recent Decisions and Open Work

Decisions and open work items identified after the KB was generated in March 2026. Organized by topic.

---

## 2026-04-28 — CIEL-OCL Squad Call

### 1. Concept Name and Description Retirement (Data Model Change)

**Decision:** Add `retired: boolean` (default `false`) to the Name and Description sub-objects on concepts. This makes retirement status **managed content** — an explicit field in the current version rather than something inferred by diffing historical versions.

**Background:** OCL's current model does not store a retired flag on names or descriptions. If a name is absent from an import, it is treated as deleted. There is no way to represent "this name once existed and was officially retired." The CIEL release process generates MySQL dumps for OpenMRS from OCL exports, and OpenMRS stores a `voided` flag per concept name row. Historical instance data references specific concept name IDs in `obs.value_coded_name_id`, so names cannot simply disappear — they need to be marked retired and carried forward.

**Prior art consulted:** SNOMED CT has active/inactive on descriptions with a coded inactivation reason. ICD-11 has a Boolean deprecated flag on titles. LOINC and ICD-10-CM do not manage name-level status. OpenMRS uses `voided`. The Boolean approach matches the OpenMRS pattern and is sufficient for CIEL needs; a status string is not needed.

**What needs to be built:**
- Add `retired` field to the Name and Description models in oclapi2 (default `false` on all existing records — no migration script required)
- Retire/unretire API endpoints for names and descriptions (analogous to mapping retirement)
- Export JSON must include `retired` on each name and description sub-object
- Import behavior: if a name is absent from an import payload, the system should soft-retire it (carry it forward with `retired: true`) rather than deleting it
- Bulk import must accept the `retired` field on name/description sub-objects
- OCL Module (OpenMRS Java module) must be updated to read the `retired` field and set `voided` on concept names during import; submit as a PR (reviewer: Ian)
- TBv3: toggle to show/hide retired names in concept detail view; retire/unretire action in the concept editor

**Reference:** [ocl_online#77](https://github.com/OpenConceptLab/ocl_online/issues/77)  
**Who populates initial values:** Burke (on the next CIEL build from CIEL Labs)

---

### 2. Duplicate Retired Mappings — Import Bug Fix · [#2490](https://github.com/OpenConceptLab/ocl_issues/issues/2490) · [#2491](https://github.com/OpenConceptLab/ocl_issues/issues/2491)

**Background:** A bulk import of CIEL mappings on staging failed because two retired mappings existed with identical key fields (from-concept, to-concept, map-type). The import tried to unretire one (setting `retired: false`) but could not determine which of the two duplicates to update, crashing the job. The duplicate accumulated because:

- OCL's duplicate check only validates new mappings against **active** resources, not retired ones
- CIEL Labs now emits `retired: true` on some mappings; prior CIEL Admin imports always wrote `retired: false`
- A mapping was manually created on staging on 2026-04-03 (by jamlung) that matched an existing retired mapping, creating the second retired duplicate. This occurred as a result of automated/unintended repeated imports of CIEL every three days from a CIEL Staging import done on March 3, 2026.

**Decisions:**
1. When an import tries to create a new active mapping that matches an existing retired mapping, **unretire** the existing one instead of creating a duplicate
2. Extend the duplicate check to validate against both active and retired resources
3. When multiple retired duplicates with identical key fields exist, unretire the most recently updated one and log a warning

**Open question:** Whether to also deduplicate existing retired mapping duplicates in production (likely via a one-time cleanup script). Staging is not representative of production for this issue — Burke submitted a custom import to staging, so the duplicate may not exist in prod. Verify before running cleanup.


