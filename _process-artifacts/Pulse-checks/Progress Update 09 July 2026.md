
## Where We Are — Headlines for OCL-CIEL Team

## 09 July 2026

### What's shipped

* **Mapper MVP is live.** The core AI-assisted mapping engine — the thing the whole ICD-11/CIEL Bridge strategy depends on — is built and closed out (81 issues).
* **ICD-11 Bridge pipeline proven end-to-end, twice**: once as a technical demo, and again on a real pre-publication dataset. The generalized "map anything to a mapped source" capability (not just CIEL→ICD-11) is also in place.
* **CIEL Lab v2 is production-capable, not just a prototype**. It was used to push an actual CIEL release live, and its Concept Manager and QA workflow pieces are done.
* **Collection & Reference Creation in TermBrowser v3 has had major improvements** — both the basic and enhanced versions. This was the foundational Implementers-track capability everything else builds on.
* **Content Curation's technical (non-AI) QA workflow is built.**
* Plus: OCL Early Access launched, and the DBMI research presentation happened.

**The throughline:** the hard technical proof-of-concept work across all four SOW streams is done. What's left is mostly about scaling those proven capabilities into repeatable, complete, or broadly-usable workflows — which is a different kind of work (process, decisions, breadth) than what's been completed so far.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### What hasn't been achieved yet

1. **CIEL Bridge — proven on one dataset, not yet proven at scale or breadth**

   1. The pipeline works on pre-pub data, but hasn't yet been run against a full country dataset or a broader evaluation dataset, and it hasn't been showcased running against target repos other than ICD-11/CIEL.
      1. We've proven it can work, not that it reliably works across the range of real-world inputs it needs to handle.
   2. The GDHF conference submission (a visibility/positioning goal, not a technical one) is also not yet moving.
2. **CIEL Source Management — the release happened once, but not as a repeatable pipeline**

   1. Filipe (with Andy filling in) got one real release out through CIEL Lab v2, which proved the tool works.
   2. But turning that into a documented, repeatable release workflow — and the "real-life release using Labs v2" showcase that would confirm it's not a one-off — hasn't happened(?)
3. **CIEL/OpenMRS Implementers — the biggest remaining surface area**
   This is where the most SOW weight (45%) sits and where the most is still open:

* Collection Maintenance (responding to a CIEL release) is furthest along and closest to a showcase.
* Hierarchy Display needed –
  * browse a repo hierarchically ([https://github.com/OpenConceptLab/ocl\_issues/issues/2291](https://github.com/OpenConceptLab/ocl_issues/issues/2291))
    * view a concept in its hierarchy – e.g. concept details page (v3 & mapper) or associations panel
* Team-based mapping workflows are early-stage.
* Concept Proposals hasn't started — and was already flagged in the last review as a bandwidth risk against Collection Maintenance.
* Mapping Decision Memory (AI-assisted recall of past mapping decisions) hasn't started.
* Linked Sources — a newer capability (letting collections track a source's HEAD rather than a frozen version) — is early and has a near-term deadline.

4. **OCL Mapper — built, but not ready to open up**

   1. The core product works, but two things are blocking it from being more than an internal tool:
      1. The public showcase/wrap-up work isn't done
      2. Need a plan for covering or charging for AI/compute costs per user.
   2. That access-and-pricing gap is the real ceiling on community rollout, not remaining engineering work.
5. **Content Curation — the tool works, the curation itself is early**

   1. The QA tooling is done, but actual curation of the \~15k-concept backlog is only an estimated 10–15% complete, and the bulk validation tool surfaced a new category of work (non-English locale casing issues) that expanded the backlog further.
   2. No showcase milestone is scoped yet for "curation complete."
6. **TermBrowser v3 platform (broader MVP, beyond the CIEL-specific showcases above)**

   1. This is the largest single body of remaining work — roughly a third done — and notably, about half of what's left hasn't even been spec'd yet.
   2. This is less "work in progress" and more "work not yet defined"
