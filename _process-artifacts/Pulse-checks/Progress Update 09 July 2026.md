## Where We Are — Headlines for OCL-CIEL Team

## 09 July 2026

### What's shipped

* **Mapper MVP is live.** The core AI-assisted mapping engine — the thing the whole ICD-11/CIEL Bridge strategy depends on — is built and closed out (81 issues).  
* **ICD-11 Bridge pipeline proven end-to-end, twice**: once as a technical demo, and again on a real pre-publication dataset. The generalized "map anything to a mapped source" capability (not just CIEL→ICD-11) is also in place.  
* **CIEL Lab v2 is production-capable, not just a prototype.** It was used to push an actual CIEL release live, and its Concept Manager and QA workflow pieces are done. The team cut over to managing CIEL concept pages primarily in OCL last month, running bug-free enough that Ellen has already reported better performance and visibility.  
* **Collection & Reference Creation in TermBrowser v3 has had major improvements** — both the basic and enhanced versions. This was the foundational Implementers-track capability everything else builds on.  
* **Content Curation's technical (non-AI) QA workflow is built**, and already drove a major cleanup of retired SNOMED codes.  
* Plus: OCL Early Access launched, and the DBMI research presentation happened.

**The throughline:** the hard technical proof-of-concept work across all four SOW streams is done. What's left is mostly about scaling those proven capabilities into repeatable, complete, or broadly-usable workflows — a different kind of work (process, decisions, breadth) than what's been completed so far.

---

### What hasn't been achieved yet

1. **CIEL Bridge — proven on one dataset, not yet proven at scale or breadth**

   1. The pipeline works on pre-pub data, but hasn't yet been run against a full country dataset or a broader evaluation dataset, and it hasn't been showcased running against target repos other than ICD-11/CIEL.
      1. We've proven it can work, not that it reliably works across the range of real-world inputs it needs to handle.
   2. The GDHF conference submission (a visibility/positioning goal, not a technical one) is also not yet moving.
   3. **Re-ranking is built; the eval isn't.** Multiple re-rankers are configurable (including custom Hugging Face models), but the planned formal eval never ran — turns out that's lower priority than building a **unified/composite score** that blends re-ranker output with other signals. That score also needs to hold up *without* AI, since not everyone will pay for the advanced algorithms.
   4. ICD-11 hierarchy display inside the Mapper was intentionally left out of Bridge scope — folded into the Hierarchy Display work under Implementers (#3).

2. **CIEL Source Management — the release happened once, but not as a repeatable pipeline**

   1. Filipe (with Andy filling in) got one real release out through CIEL Lab v2, which proved the tool works.
   2. But turning that into a documented, repeatable release workflow — and the "real-life release using Labs v2" showcase that would confirm it's not a one-off — hasn't happened. The concept *release* process itself (distinct from the concept management pages, which are done) still needs its own conversation.
   3. Remaining production gaps: minor UX items (diagnosis filtering, result sorting), occasional validation errors, and a slight indexing lag between the Lab and OCL HEAD.
   4. **Bulk validation coverage has a gap** — a SNOMED retired-code cleanup surfaced RxNorm content with no rule at all. Work is underway to generalize bulk validation into ad-hoc rules, and to turn QA worklists into an actual remediation workflow (mass-pushing reviewed/AI-proposed fixes), similar to the Mapper's bulk edit.
   5. **Open question:** how to persist a reviewer's decision to *ignore* a bulk-validation finding so it doesn't resurface every re-run. High-value beyond CIEL (relevant to OMOP/Odyssey too), and the same underlying problem as Mapping Decision Memory below.
   6. **V3's CIEL/org landing page isn't ready to replace V2's** branded one — on the roadmap only as "not deferred," no concrete plan yet.
   7. The remaining gap on Filipe's side is mostly hours left in his allocation, not scope. Open question: does the unfinished work carry into a next phase, or pause?

3. **CIEL/OpenMRS Implementers — the biggest remaining surface area**
   This is where the most SOW weight (45%) sits and where the most is still open:

* **Collection Maintenance** (responding to a CIEL release) is furthest along and closest to a showcase — very little left in to-do/requirements beyond one possible bug. Next step is ticketing any remaining gaps not yet on the board.
* **Hierarchy Display** needed –
  * browse a repo hierarchically ([https://github.com/OpenConceptLab/ocl\_issues/issues/2291](https://github.com/OpenConceptLab/ocl_issues/issues/2291))
    * view a concept in its hierarchy – e.g. concept details page (v3 & mapper) or associations panel
  * Two phases: **Phase 1** (browse a source top-down, like v2) is ready to ticket with no more design. **Phase 2** (view a concept in context — position, parents, light navigation) needs a small design pass. A more ambitious idea (siblings, cross-repo navigation, ECL-style queries) is aspirational, not near-term.
  * **Collection Maintenance and Hierarchy Display (phase 1) are the two priorities starred for showcase before this period ends.**
* Team-based mapping workflows are early-stage, though the Mapper itself already covers a large share of what's needed here.
* Concept Proposals hasn't started — already flagged as a bandwidth risk against Collection Maintenance, and now **falling off the map as a reality for this period** given people OOO. Foundation and momentum exist to pick it back up later; Odyssey (OMOP) is independently exploring similar workflows, a possible joint effort down the line.
* Mapping Decision Memory (AI-assisted recall of past mapping decisions) hasn't shipped, though there's reportedly good progress behind the scenes. Same underlying capability as the decision-persistence problem under Source Management (#2.5) — remembering and honoring a prior human decision.
* Linked Sources — letting collections track a source's HEAD rather than a frozen version — is early and has a near-term deadline. Won't finish this period either, but requirements agreement is strong — better positioned for a future phase than a few weeks ago.

4. **OCL Mapper — built, but not ready to open up**

   1. The core product works, but two things are blocking it from being more than an internal tool:
      1. The public showcase/wrap-up work isn't done
      2. Need a plan for covering or charging for AI/compute costs per user.
   2. That access-and-pricing gap is the real ceiling on community rollout, not remaining engineering work. Subscriptions and per-use AI billing are now being actively scoped as real engineering work, not just a policy decision — and will need to land across every tool once built.

5. **Content Curation — the tool works, the curation itself is early**

   1. The QA tooling is done, but actual curation of the \~15k-concept backlog is only an estimated 10–15% complete, and the bulk validation tool surfaced a new category of work (non-English locale casing issues) that expanded the backlog further.
   2. No showcase milestone is scoped yet for "curation complete." The RxNorm rule gap and the decision-persistence problem under Source Management (#2.4–2.5) are really Content Curation issues too — both surfaced through Filipe's CIEL Lab QA work.

6. **TermBrowser v3 platform (broader MVP, beyond the CIEL-specific showcases above)**

   1. This is the largest single body of remaining work — roughly a third done — and about half of what's left hasn't even been spec'd yet. That's less "work in progress" and more "work not yet defined."
   2. That said, the on-the-ground read is more optimistic than the ticket count suggests — v2 is barely needed day to day anymore. The framing has shifted from "how much is left to build" to **where's the cutoff for shipping to a broader audience** — that cutoff needs to be made concrete for both TBv3 and Mapper.
   3. A **bug bash** — using TBv3 like an end user rather than reviewing tickets — has been proposed to surface V2→V3 gaps faster than the backlog would.

---

### What's next

* **Near-term focus:** ship Collection Maintenance and Hierarchy Display (phase 1) as the two starred showcases before wrapping this period.
* **After that:** a cross-tool step-back to find remaining gaps everywhere, then build the roadmap for the next phase.
* **Define a concrete "feature-complete MVP" cutoff** for TermBrowser v3 and Mapper.
* **Three prerequisites for opening up paid AI access:** the MVP cutoff above, subscription/AI billing capability (being scoped now), and a community site overhaul (draft plan exists — SSO, drop-in blog structure).
* **Still unresolved:** a dedicated conversation on the CIEL concept *release* process (separate from the concept management pages, which are done).
