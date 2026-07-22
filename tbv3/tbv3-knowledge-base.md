# TBv3 Knowledge Base

- **Generated:** 2026-03-20
- **Scope:** 262 issues from the OCL TermBrowser v3 project
- **Source:** [OpenConceptLab/ocl_issues](https://github.com/OpenConceptLab/ocl_issues) GitHub repository
- **Companion data:** `tbv3-raw-archive.json` (full issue bodies + comments), `tbv3-issue-classification.csv` (208-row analysis)

---

## About This File and the Requirements Library

This knowledge base was synthesized from 262 GitHub issues and represents the historical and architectural context of TBv3 as tracked in the issue tracker.

For the **authoritative, actively maintained requirements library** — including data schemas, UI specifications, user workflows, and architecture decisions — see the folder structure alongside this file:

| Folder | Contents |
|---|---|
| [`00_context/`](00_context/) | Product vision, user roles, glossary — always load these |
| [`01_objects/`](01_objects/) | Concept, Mapping, Reference, Repository, Version, Expansion, Concept Proposal schemas |
| [`02_capabilities/`](02_capabilities/) | User-facing features: search, browse, author, manage references/versions/proposals, compare, validate |
| [`03_workflows/`](03_workflows/) | End-to-end user journeys: build dictionary, update collection, author source, manage proposals |
| [`04_surfaces/`](04_surfaces/) | Page and component specs: design system, concept detail, repository page, versions tab |
| [`05_decisions/`](05_decisions/) | Architecture Decision Records (ADRs) |

See [`tbv3-folder-structure.md`](tbv3-folder-structure.md) for how to navigate the library for different use cases (UI feature, workflow, API, test generation, design review).

For features explicitly deferred to post-v3, see [`tbv3-deferred-features.md`](tbv3-deferred-features.md).

For current priorities and open requirements work, see [`PRIORITIES.md`](PRIORITIES.md) and [`TODO.md`](TODO.md).

---

## Table of Contents

1. [Executive Summary](#section-1-executive-summary)
2. [Architecture & Foundation](#section-2-architecture--foundation)
3. [Consumer Experience](#section-3-consumer-experience)
   - 3a. Dashboard & Navigation
   - 3b. Repository Views
   - 3c. Concept Views
   - 3d. User & Org Profiles
4. [Authoring Experience](#section-4-authoring-experience)
   - 4a. Repository Authoring
   - 4b. Concept Authoring
   - 4c. Mapping Authoring
   - 4d. Organization Management
   - 4e. Expansion Management
   - 4f. Reference Management
5. [Cross-Cutting Concerns](#section-5-cross-cutting-concerns)
6. [Legacy & Migration Notes](#section-6-legacy--migration-notes)
7. [Open Questions & Unresolved Discussions](#section-7-open-questions--unresolved-discussions)
8. [Resource Library](#section-8-resource-library)
- [Appendix A: Issue Index](#appendix-a-issue-index)
- [Appendix B: Contributor Map](#appendix-b-contributor-map)

---

# OCLv3 Knowledge Base: Sections 1 & 2

---

## Section 1: Executive Summary

### 1.1 What Is OCLv3 / TermBrowser v3

OCLv3 -- referred to in project tracking as "TermBrowser v3" or "TBv3" -- is the next-generation web interface for the Open Concept Lab (OCL) platform. It replaces the existing TermBrowser v2 (oclweb2) with a ground-up redesign (oclweb3) that delivers a modern user experience for browsing, authoring, and managing health terminology content. The effort spans three codebases: the OCL API (oclapi2), the new web frontend (oclweb3), and the OCL Mapper (oclmap), with supporting work in ocl-docs and infrastructure.

The v3 initiative is not simply a UI refresh. It introduces foundational platform capabilities that were absent or incomplete in v2: a canonical URL registry for resolving FHIR-style references (#1696), a typed repository model that distinguishes CodeSystems from ValueSets from Dictionaries (#1695), a roles-and-permissions framework (#1812), API throttling (#2121), and a redesigned authentication flow built on Keycloak SSO (#1702, #2145). These "v3-foundation" subsystems reshape how OCL organizes, governs, and exposes terminology content.

### 1.2 Key Stakeholders and Contributors

The contributor footprint across the 208 classified issues reveals a tight core team with specialized roles:

| Contributor | Primary Focus | Notable Involvement |
|---|---|---|
| **@paynejd** (Jonathan Payne) | Product owner, architecture, API design | Canonical URL registry design (#1696, #1732), repo type specification (#1687, #1695), diffs/checksums strategy (#1690), throttling requirements (#2121) |
| **@snyaggarwal** (Sunny Aggarwal) | API & frontend engineering | URL registry implementation (#1732, #1816), $resolveReference (#1747), Keycloak SSO (#1702), throttling implementation (#2121), bulk of oclweb3 feature work |
| **@jamlung-ri** (Joe Amlung) | UX research, user stories, QA | User stories for repo type (#1695), diffs (#1690), canonical URL registry; QA testing of SSO flows (#1702); design input on search, hierarchy, org views |
| **@paulsonder** (Paul Sonder) | UI/UX design, frontend design specs | Assigned to nearly all Design-labeled issues; search UI (#1694), concept detail views, mapping views, organization and user profile designs |
| **@bmamlin** (Burke Mamlin) | Clinical domain expertise, FHIR alignment | URL registry review (#1732), global registry population strategy (#1743) |
| **@rkorytkowski** (Rafal Korytkowski) | Infrastructure, legacy API knowledge | Global registry population (#1743) |

The pattern across issues is consistent: @paynejd defines requirements and architectural decisions, @snyaggarwal implements them on both API and web sides, @jamlung-ri translates requirements into user stories and performs QA, and @paulsonder produces design specifications. @bmamlin and @rkorytkowski serve as domain and infrastructure advisors on select foundational tickets.

### 1.3 Timeline and Milestones

**Origins (pre-2022).** The earliest issues in the TBv3-classified set date to 2018 (#12, #13), but these are legacy TermBrowser enhancement requests that were never part of a v3 initiative. The v3 label begins appearing around mid-2022.

**Foundation phase (Oct 2022 -- Nov 2023).** The first explicitly v3-scoped issue is #1371 (Canonical URL Restrictions, Round 1), created October 2022. Through 2023, the team defined the architectural foundations: canonical URL handling, repo types, diffs/checksums, and the initial design system. The v3 project board was established, and the "TermBrowser v3 MVP" milestone was created to track the path to a production-ready replacement for v2.

**Build phase (Nov 2023 -- mid 2024).** November 2023 marks a burst of epic creation: #1687 (repo_type), #1690 (diffs/checksums), #1694 (search UI), #1695 (repo type epic), #1696 (canonical URL registry), and the dashboard/registration epic #1688. The URL registry API was specified (#1732) in January 2024 and deployed to QA by January 17, 2024. By February 2024, URL registries were integrated into $resolveReference and deployed behind a feature toggle. The SSO/Keycloak integration (#1702) was completed and deployed to QA by January 2024.

**Feature expansion (mid 2024 -- late 2024).** From mid-2024 onward, the focus shifted to consumer-facing features: repository views, concept details, organization profiles, user profiles, and search experiences. A large batch of "CTA" (call-to-action) authoring tickets was created in November 2024 (#1977--#2031), covering CRUD operations for repositories, concepts, mappings, expansions, references, and organizations.

**Hardening and launch preparation (early 2025 -- present).** In 2025, work has concentrated on API throttling (#2121, created Feb 2025), authentication improvements (#2132, #2138, #2145), and QA testing of authoring workflows. Many CTA tickets are now in "Testing (QA)" status. The Repo Type epic (#1695) was closed on March 18, 2026, indicating that the foundational repo type work is considered complete.

### 1.4 Current State Overview

Based on the 208 classified issues and the 169-item project board:

**Disposition breakdown (from classification):**

- **keep**: the majority of issues are active or relevant to ongoing v3 work
- **close-stale / close-outdated / close-superseded**: a significant cohort of pre-v3-era issues recommended for cleanup
- **review**: issues requiring further triage before a keep/close decision

**Project board status (for the ~110+ items on the v3 board):**

- **Done / Closed**: Dashboard and registration (#1688 and children), SSO Keycloak flow (#1702), URL registry API (#1732), URL registry in TB (#1742), $resolveReference integration (#1747, #1788), ES caching for registry (#1816), Repo Type epic (#1695), multiple user/org profile and search results tickets
- **Testing (QA)**: API throttling (#2121), SSO URL redirects (#2145), concept edit (#1991), repository create/edit/version/release, mapping CRUD, org delete, repo comparison (#1853), concept history (#2290), namespace mappings (#1972), importer UI (#2161)
- **Todo**: Repository templates (Source #2025, Collection #2026, ValueSet #2027, CodeSystem #2028, ConceptMap #2029), repo header (#1871), repo version selector (#1872), repo overview (#1951), concept add-mapping (#1995)
- **Backlog / In Development**: repo_type API attribute (#1687), repo_type in TBv3 (#1727), chip views (#1776), references list (#1949), add references (#2005)
- **Open / No Status**: Canonical URL restrictions (#1371), global registry population (#1743), registry permissions (#1744), namespace selector (#1821), add canonical URL UI (#1827), roles/permissions (#1812), diffs/checksums (#1690), forced sign-in/out UX (#2132), django-auth sign-in (#2138), and many consumer/authoring epics

**Key blockers and gaps:**

1. The **Canonical URL Registry** epic (#1696) has API-side plumbing complete but critical operational and UX pieces remain open: populating the global registry (#1743), defining permissions (#1744), UI for adding URLs (#1827), and the namespace selector (#1821).
2. **Roles and Permissions** (#1812) remains entirely in requirements-gathering stage with no child tickets and no code activity -- it is a prerequisite for multi-tenant governance.
3. **Diffs and Checksums** (#1690) has partial API implementation (source compare, duplicate prevention) but the changelog, squash, collection diff, and TB integration components are unbuilt.
4. **repo_type** API attribute (#1687) is still open despite the parent epic (#1695) being closed -- the migration of existing data and the TB-side implementation (#1727) remain in backlog.
5. **Auth improvements** (#2132, #2138) are newly filed (March 2025) and address UX gaps in session management that affect all OCL web apps.

---

## Section 2: Architecture & Foundation

The "v3-foundation" label identifies a set of cross-cutting subsystems that underpin the TermBrowser v3 experience. These are not features visible to end users in isolation; they are platform capabilities that other features depend on. This section synthesizes the design decisions, implementation status, and open questions for each.

### 2.1 Canonical URL Registry

**Epic:** #1696 | **Children:** #1371, #1732, #1742, #1743, #1744, #1747, #1788, #1816, #1821, #1827

#### What It Is and Why It Matters

In FHIR and broader health terminology standards, resources are identified by canonical URLs -- persistent, globally unique URIs like `http://hl7.org/fhir/ValueSet/example`. OCL's v2 platform had no mechanism to centrally track or resolve these URLs. Canonical URLs on repositories were optional, not enforced as unique, and there was no global registry. This meant that the `$resolveReference` operation -- which resolves a canonical URL to a specific OCL repository -- could produce ambiguous results when multiple repos in different namespaces shared the same canonical URL (#1371).

The Canonical URL Registry introduces a first-class API resource (`/url-registry/`) that maps canonical URLs to OCL repositories, enabling unambiguous resolution across the platform. This is critical for FHIR compliance, for mapping workflows that reference external code systems by canonical URL, and for the broader goal of making OCL a reliable canonical-aware terminology server.

#### What Was Built

**URL Registry API resource (#1732 -- CLOSED).** Specified in January 2024 and deployed to QA by January 17, 2024. The resource model includes:
- `url` (required canonical URL), `namespace` (optional, e.g. `/orgs/CIEL/`), `name`, `extras`, and audit fields
- CRUD endpoints at both global (`/url-registry/`) and namespace-scoped (`/:ownerType/:owner/url-registry/`) levels
- A `$lookup` operation for finding a registry entry by URL

Design decisions from the discussion (#1732 comments):
- The `owner_url` field was added to all entries, with global entries having `owner_url: "/"` and null `owner`/`owner_type` -- this addressed the question of how to represent entries not owned by a specific namespace.
- The `type` field value was changed to `"URLRegistryEntry"` per @paynejd's feedback.
- `$lookup` was scoped to a single registry (global or namespace-specific), with the cross-registry resolution logic delegated to `$resolveReference`.

**$resolveReference integration (#1747 -- CLOSED).** By March 2024, 100% of API logic for finding a repository was routed through `$resolveReference`, including mapping resolution. The key design decision was to incorporate canonical URL resolution directly into the existing `$resolveReference` operation rather than creating a separate `$resolveCanonical` operation (#1732 comments, @paynejd, Jan 19 2024). This meant that all existing consumers of `$resolveReference` automatically gained canonical-aware resolution without API changes.

The resolution order within a namespace follows a three-step lookup: (1) owner-specific registry, (2) owner's repos by canonical URL, (3) global registry.

**Registry entry in $resolveReference response (#1788 -- CLOSED).** When a registry entry is used for resolution, the entry is now included directly in the response body (not the HTTP header), because `$resolveReference` can process multiple expressions in a single request (#1788 comments, @snyaggarwal, Mar 7 2024).

**ES caching for registry (#1816 -- CLOSED).** Registry entries now cache their resolved repository in Elasticsearch, enabling search by resolved repo ID and updating automatically when repos are created/updated with canonical URLs. Deployed to all environments by March 28, 2024 (#1816 comments). An open technical question was raised about `$lookup` vs. `$resolve` semantics -- `$lookup` does not handle the `|` version syntax while `$resolve` does (#1816 comments, @snyaggarwal, Mar 22 2024).

**Canonical URL registries in TermBrowser (#1742 -- CLOSED).** The registry browsing UI was implemented in TBv3 and deployed on QA with refinements including an "Owner chip" to replace the target owner column and a combined repo ID/type display (#1696 comments, @paynejd, Apr 2 2024).

**Mapping resolution switchover (#1747 -- CLOSED).** Mappings were the last API subsystem to switch to `$resolveReference` for repo resolution. The approach passes the source URL through `$resolveReference` with `version='HEAD'` to resolve to the HEAD version (#1747 comments, @snyaggarwal, Jan 31 2024).

#### What Is Pending

**Canonical URL restrictions (#1371 -- OPEN).** The foundational policy questions remain unresolved:
- Should canonical URLs be auto-generated/always present?
- Should they be unique across the system?
- Should updates be restricted (changing a canonical URL could break references across repo versions)?
- What happens when a repo with multiple released versions changes its canonical URL -- does the change apply retroactively?

This is labeled "discussion-needed" and has been open since October 2022. It is the policy backbone that the registry implementation needs to fully operationalize.

**Populating the global registry (#1743 -- OPEN).** The operational question of how to seed and maintain the global registry across environments remains open. The discussion (#1743 comments, @paynejd, Jan 30 2024) surfaces two competing philosophies:
1. **Curated registry**: Only canonical URLs that owners explicitly want discoverable outside their org.
2. **Automatic registry**: Any public repo's canonical URL is automatically registered, with community curation for conflicts.

The current thinking favors minimizing manual curation: public repos with new canonical URLs would auto-register, while private repos would not. Key unresolved edge cases include: what happens when a registered repo is deleted, how to handle conflict resolution, and how to support local OCL implementations.

**Registry permissions (#1744 -- OPEN).** No body or comments -- this is a placeholder for designing who can read, write, and manage registry entries, and how permissions interact with `$resolveReference` behavior. Created January 2024, no progress.

**Namespace selector UI (#1821 -- OPEN).** The TB-side tool for selecting which namespace to query when browsing/testing canonical URLs. Assigned to @paulsonder, no implementation yet.

**Add Canonical URL UI (#1827 -- OPEN).** The basic UI for creating new registry entries in TBv3. No implementation yet.

#### Open Design Questions

- How should `$lookup` and `$resolve` semantics be harmonized, particularly around version syntax (`|`) support? (#1816)
- Should the global registry support batch loading for initial population? (#1732, noted as "out of scope for MVP")
- What is the governance model for the global registry -- first-come-first-served with community escalation, or admin-curated? (#1743)
- How do feature toggles for breaking `$resolveReference` changes get managed across environments? (#1732 comments)

---

### 2.2 Repo Type System

**Epic:** #1695 (CLOSED) | **Children:** #1686 (CLOSED), #1687 (OPEN), #1701 (CLOSED), #1727 (OPEN)

#### What It Is and Why It Matters

OCL's data model historically had `source_type` and `collection_type` as freeform descriptive attributes (e.g., "Dictionary", "Code List", "Interface Terminology"). These did not carry behavioral implications -- a source typed as "CodeSystem" behaved identically to one typed as "External." The Repo Type system introduces a new coded `repo_type` attribute that determines what resources and behaviors are available for a given repository. This is the mechanism that allows OCL to treat a source configured as a FHIR CodeSystem differently from one configured as a plain dictionary, and to present repo-type-specific UI templates, validation rules, and workflows.

The system also introduced a unified `/repos/` endpoint (#1686 -- CLOSED) that searches across both sources and collections, replacing the need for clients to query two separate endpoints.

#### What Was Built

**Unified repos endpoint (#1686 -- CLOSED).** A global endpoint for searching across repository types, implemented with Elasticsearch support.

**Canonical URL filter (#1701 -- CLOSED).** A `canonical_url` filter parameter for repos, sources, and collections endpoints.

**Repo Type epic closure (#1695 -- CLOSED, March 2026).** The parent epic is marked complete. The user stories (#1695 comments, @jamlung-ri, Nov 2023) envisioned: accessing repositories by type (projects, dictionaries, etc.), creating repos pre-configured with type-specific schemas, and eventually exploring public-facing repositories from other organizations.

**Repo type filter in repos (#2252 -- CLOSED).** A filterable facet for repo_type in the repos endpoint, deployed with ES index support.

#### What Is Pending

**repo_type API attribute (#1687 -- OPEN, Backlog).** The core attribute implementation remains open. The specification defines supported repo types:

| Repo Type | Sources | Collections |
|---|---|---|
| Source | Yes | No |
| Collection | No | Yes |
| CodeSystem | Yes | TBD |
| ValueSet | No | Yes |
| ConceptMap | Yes | TBD |
| Dictionary | No (?) | Yes |

A migration mapping from existing `source_type`/`collection_type` values to `repo_type` is defined in the issue body. Open questions from @snyaggarwal's review (Nov 15, 2023) remain unanswered:
- How is `repo_type` updated -- via API, TermBrowser, or both?
- Does it need to appear in TB headers/forms?
- Should new repos accept `repo_type` in the API, default based on the mapping table, or both?
- Do bulk import and CSV converters need changes?
- Is `repo_type` filterable (answered: yes, per #2252)?

Additional discussion on FHIR-specific repo types was flagged by @jamlung-ri (Dec 5, 2023) but not elaborated.

**repo_type in TBv3 (#1727 -- OPEN, Backlog).** The frontend implementation of repo_type display and selection. Depends on #1687 being completed.

#### Key Design Decision

The decision to keep `source_type`/`collection_type` as purely descriptive (optional) fields while introducing `repo_type` as the behavioral driver avoids a breaking migration. Existing content retains its descriptive type; the new `repo_type` is populated via a deterministic mapping from existing values but carries actual behavioral weight going forward.

---

### 2.3 Roles & Permissions

**Epic:** #1812 (OPEN)

#### What It Is and Why It Matters

OCL v2 has a simple two-tier permission model: organization members can be either members or admins, and repository-level permissions are inherited from the owning org/user. The v3 roles-and-permissions initiative (#1812) aims to implement a more granular system that can support:
- Role-based access to specific repositories within an organization
- Differentiated permissions for content authoring vs. administrative actions
- Governance workflows (e.g., change requests, approval chains) that depend on knowing what a user can and cannot do

The requirements document is being developed externally (Google Doc linked in #1812).

#### Current State

This epic is in the earliest stage of all v3-foundation items. It has:
- No child tickets
- No comments
- No assignees
- No milestone

However, it does show oclapi2 code activity per the classification data, suggesting preliminary investigation or scaffolding has occurred. The issue was created March 15, 2024, and last updated February 21, 2025, indicating periodic review but no forward movement on design or implementation.

#### Why It Is Blocked

The external requirements document has not yet been translated into actionable tickets. Until the permission model is defined (which roles exist, what each role can do at the org/repo/resource level, how roles are assigned and inherited), implementation cannot begin. This is a dependency for several other features: registry permissions (#1744), authoring workflows, change request management (#1622), and the "walled garden" / trusted sources concept (#1706).

---

### 2.4 API Throttling

**Epic:** #2121 (OPEN, Testing on QA)

#### What It Is and Why It Matters

API throttling limits the rate at which clients can call OCL API endpoints. Without throttling, the platform is vulnerable to both accidental abuse (runaway scripts, poorly-designed integrations) and intentional denial-of-service. As OCL prepares for broader public use with TBv3, throttling becomes essential for service reliability.

#### What Was Built

The throttling implementation was designed based on analysis of two weeks of production and staging logs (#2121 comments, @snyaggarwal, Mar 13 2025). The initially proposed tiers were:

1. **Guest**: 400 requests/minute, 10,000 requests/day (tracked by IP)
2. **Lite (default)**: 500 requests/minute, 20,000 requests/day
3. **Premium**: 1,000 requests/minute, 50,000 requests/day

The final implementation (per @paynejd's direction, Mar 17 2025) simplified to two tiers:

1. **Guest** (anonymous): 400/min, 10k/day, tracked by IP address
2. **Standard** (authenticated): 500/min, 20k/day

The premium tier was deferred. The implementation was deployed to QA by March 18, 2025.

#### Key Design Decisions

- **Data-driven limits**: Thresholds were set based on actual usage patterns, not arbitrary values.
- **Two-tier simplification**: Rather than launching with three tiers that require a mechanism to assign users to premium status, the team shipped with guest/standard and deferred premium to a later phase -- likely to be revisited when #1812 (roles/permissions) provides the infrastructure for tiered user classification.
- **IP-based tracking for anonymous users**: A pragmatic choice that works for most scenarios but may be inadequate for shared IP environments (NAT, VPN).

#### What Is Pending

- Production deployment (currently QA only)
- Premium tier and custom rate limits for specific API consumers
- Integration with the roles/permissions system (#1812) for programmatic tier assignment
- Rate limit response headers and client-facing documentation

---

### 2.5 Auth & SSO

**Issues:** #1702 (CLOSED), #2132 (OPEN), #2138 (OPEN), #2145 (OPEN, Testing on QA)

#### What It Is and Why It Matters

Authentication is the gateway to all non-public OCL functionality. The v3 auth strategy centers on Keycloak as the SSO provider, replacing the Django-native authentication that powered v2. This shift enables standardized OAuth2/OIDC flows, custom branding of login/registration screens, and a single sign-on experience across OCL web applications (TBv3, OCL Mapper, potentially v2 during the transition period).

#### What Was Built

**Keycloak SSO integration (#1702 -- CLOSED).** The registration, sign-in, and forgot-password flows were implemented with a Keycloak theme and deployed to QA by January 2024. QA testing by @jamlung-ri (Dec 5, 2023) confirmed that organization and location fields were included in registration, and account verification and password reset emails functioned correctly. The issue was closed January 11, 2024.

**SSO URL redirects (#2145 -- OPEN, Testing on QA).** Simple URLs (`/#/signin`, `/#/signup`) that redirect to the actual SSO provider. Implemented for both v2 and v3, deployed to QA, staging, and production (for v2) as of March 27, 2025. This smooths the user experience by providing predictable entry points regardless of the underlying SSO configuration.

#### What Is Pending

**Forced sign-in/sign-out UX (#2132 -- OPEN).** The current behavior when a session expires or a forced sign-out occurs is described as having a "poor user experience" that "can often result in lost work." This issue was filed March 19, 2025, with no assignees, no comments, and no design work. It applies to all OCL web applications, not just TBv3.

**Django-auth sign-in form (#2138 -- OPEN).** For environments or deployments that do not use Keycloak (e.g., local development, on-premise installations), a Django-native sign-in form is needed for oclweb3 and oclmap. Assigned to @snyaggarwal, filed March 24, 2025, no implementation yet.

#### Key Design Decisions

- **Keycloak as the canonical auth provider**: This aligns OCL with industry-standard identity management and enables future features like social login, MFA, and federated identity.
- **Dual-path auth support**: The existence of #2138 indicates that Keycloak is not a hard requirement -- the platform must also support Django-native auth for flexibility. This dual-path approach adds complexity but is necessary for OCL's diverse deployment scenarios (hosted SaaS vs. on-premise).
- **Progressive deployment**: SSO URL redirects (#2145) were deployed to v2 production first, demonstrating a pattern of shipping auth changes to the stable environment before v3.

#### Open Questions

- What should the session timeout behavior be, and how should in-progress work be preserved when a session expires? (#2132)
- Should django-auth and Keycloak auth be runtime-switchable, or deployment-time configured? (#2138)
- How does auth interact with API throttling tiers (#2121) -- are authenticated users identified via their SSO token for rate limiting?

---

### 2.6 Diffs & Checksums

**Epic:** #1690 (OPEN)

#### What It Is and Why It Matters

The diffs and checksums subsystem provides the primitives for answering the question: "What changed?" It spans several capabilities:

1. **Preventing duplicate resource versions** during imports by using checksums to detect when an import would create a version identical to the existing one.
2. **Comparing repository versions** to produce diffs showing what was added, removed, or changed between releases.
3. **Generating changelogs** automatically from those diffs.
4. **Squashing** redundant resource versions that accumulated before duplicate prevention was implemented.
5. **Resource-level comparison** that leverages checksums to highlight meaningful changes between two versions of a concept.

For users managing large terminologies like CIEL, this is essential. Without it, every import creates new resource versions regardless of whether anything changed, making version history meaningless and making it impossible to quickly identify what is different between releases (#1690 comments, @jamlung-ri, Nov 20 2023).

#### What Was Built

Per @paynejd's detailed status update (Feb 13, 2024):

**Duplicate prevention**: Implemented for both concepts and mappings, deployed behind a toggle (enabled on QA). The HTTP response code for a duplicate import that does not create a new version was finalized. Validation with bulk imports on QA was completed.

**Local repo diff**: Checksums are available in exports, and a checksum operation is available in the operations panel. Next steps (unfinished) include creating JavaScript/Python methods for local checksum calculation.

**Repo diff API**: Implemented at `/sources/compare/` for concepts (including hierarchy as a concept attribute). Mappings were subsequently incorporated. Still pending: diffing repo metadata, collection version diff, and a unified comparison view.

**User stories for v3** (#1690 comments, @jamlung-ri, Nov 20 2023):
- See a summary of changes between CIEL versions (X retired, X new concepts, X new mappings)
- Clearly identify when a concept changed across several releases
- See an itemized list of changed concepts between two versions
- Dive into exact changes for a concept that changed since last check-in

An additional user story from @paynejd (Mar 8, 2024): after a bulk import, fetch checksums for a repo version or specific resources to verify import success without waiting for a full export.

#### What Is Pending

The majority of the checksum package remains unbuilt:

- **Changelog generation** (#1814, referenced but not in the architecture dataset): Define requirements and implement MVP for generating a changelog between two repo versions.
- **Squash operation**: No research or requirements defined yet. The question "How deep of a hole are we in?" regarding redundant versions remains unanswered.
- **Collection version diff**: Not implemented.
- **Repo metadata diff**: No approach decided.
- **Checksum-enabled resource comparison in TBv3**: Explicitly deferred ("not planning to do this in v2"), not yet designed for v3.
- **Generic `$compare` operation**: For diffing arbitrary sets of resources (not just repo versions). Not specified.
- **Local checksum calculation**: JavaScript/Python methods not yet created.

#### Key Design Decisions

- **Standard vs. smart checksums**: The system distinguishes between "standard" checksums (exact content comparison) and "smart" checksums (semantically meaningful comparison that ignores irrelevant differences). Both are needed: standard for deduplication, smart for human-facing diffs.
- **CIEL as the reference use case**: The entire design is grounded in CIEL's workflow -- large imports, version-over-version comparison, identifying clinically meaningful changes. This ensures the implementation addresses real-world scale but may need adaptation for other use cases.
- **Toggle-based deployment**: Duplicate prevention behavior was deployed behind a feature toggle, enabling validation on QA without risking production data integrity. This pattern -- deploy behind toggle, validate, then enable -- appears repeatedly across v3-foundation work.

#### Open Questions

- What constitutes a "meaningful" change for smart checksums across different resource types?
- How should the `$compare` operation handle asymmetric comparisons (e.g., comparing a repo version against an arbitrary list of concepts from an import)?
- What is the expected performance envelope for repo diffs on large terminologies like CIEL (millions of resources)?
- How will the changelog capability integrate into the TBv3 release workflow?

---

## Cross-Cutting Observations

### Dependency Chain

Several foundation subsystems have implicit dependencies that create a critical path:

1. **Roles & Permissions (#1812)** is a prerequisite for registry permissions (#1744), API throttling tiers (premium), and authoring governance workflows.
2. **Canonical URL restrictions (#1371)** must be resolved before the global registry (#1743) can be reliably populated.
3. **repo_type API implementation (#1687)** must be completed before the TBv3 repo_type UI (#1727) and the repository templates (#2025--#2029) can ship.
4. **Diffs & checksums (#1690)** depend on the resource comparison tool design, which in turn depends on the concept detail view (#1620, #1621) being finalized.

### Pattern: API-First, Toggle-Gated

A consistent development pattern emerges across all foundation subsystems: the API capability is built first, deployed to QA behind a feature toggle, validated through iterative feedback between @paynejd and @snyaggarwal, and then the TBv3 frontend is connected. This pattern is visible in the URL registry (#1732 -> #1742), $resolveReference (#1747), and throttling (#2121). It has the advantage of decoupling backend readiness from frontend design timelines, but it means several API capabilities deployed months ago still lack their TBv3 exposure.

### Maturity Gradient

| Subsystem | API | TBv3 UI | Policy/Governance |
|---|---|---|---|
| Canonical URL Registry | Mostly complete | Partial (browse works; add/namespace pending) | Open (restrictions, permissions, population strategy) |
| Repo Type | Partially complete (endpoint yes, attribute pending) | Not started | Defined but unimplemented |
| Roles & Permissions | Not started | Not started | Requirements in progress |
| API Throttling | Complete (2 tiers) | N/A (backend only) | Minimal (2 tiers, no premium) |
| Auth & SSO | Complete (Keycloak) | Mostly complete | Open (session UX, dual-path) |
| Diffs & Checksums | Partial (source compare, dedup) | Not started | Design defined, mostly unbuilt |

This gradient shows that the v3 foundation is strongest on the auth/SSO and canonical URL registry fronts, moderately advanced on throttling and repo type, and earliest-stage on roles/permissions and diffs/checksums. The common gap across all subsystems is the governance/policy layer -- the rules about who can do what, under what conditions, with what constraints.


---

## Section 3: Consumer Experience (Viewing & Browsing)

The consumer experience in TBv3 represents the most substantial redesign effort in the project. It encompasses everything a user sees when they arrive at OCL -- from the landing page dashboard, through global search, into repository and concept detail views, and across user and organization profiles. The design philosophy, articulated across dozens of tickets and comment threads, centers on three principles: (1) align the UI with FHIR terminology conventions so that health informatics users feel at home, (2) adopt GitHub-like patterns for familiarity in profile pages, activity feeds, and repository organization, and (3) build a modular component system (chips, tooltips, search templates) that can be reused consistently throughout the application.

This section synthesizes the requirements, design decisions, implementation status, and open questions across approximately 80 issues that define the consumer persona's experience.

---

### 3a. Dashboard & Navigation

#### The Dashboard Vision

The TBv3 dashboard is the first thing a user sees and serves fundamentally different purposes depending on authentication state. The dashboard epic (#1688) -- now closed -- organized work across registration, sign-in flows, and the dashboard itself. The foundational ticket (#1612, closed) established that the dashboard acts as the home screen with two modes: signed-in and signed-out, with the signed-in version adapting content based on user behavior such as recently viewed repos, recently edited content, and recently updated repositories.

Early design work (August 2023) produced initial mockups shared via Zeplin, with skeleton layouts developed to allow parallel design and development. By December 2023, the dashboard was "feeling good" on the v3 QA environment, though still relatively basic (#1612). Design feedback from paulsonder included removing the search field border, adding placeholder text, and setting a max-width of 1024px on the content area to prevent text from sprawling across wide monitors.

#### Anonymous User Dashboard

The anonymous dashboard (#1861, closed December 2024) serves as a landing page that prompts exploration, content discovery, account creation, and learning about OCL. Its acceptance criteria include: access to global search, a feed of content highlights (not personalized), an OCL blog feed, sign-in/registration prompts, a tour option, and a nav menu. Importantly, it must be "prepared for non-OCL Online implementation" -- meaning local OCL deployments need sensible defaults.

The highlights feed for anonymous users is generated from events by superusers (specifically, the `ocladmin` user's followed entities), filtered to `CREATED`, `RELEASED`, or `FOLLOWED` event types (#1861). Blog posts from the OCL website are pulled into the dashboard feed, with dates displayed in a written-out format (e.g., "Oct 23, 2024") per a decision made during implementation (#1861). A design for unauthenticated users attempting to access restricted content was also provided (#1861).

The "Take a tour" feature remains outstanding and is tracked separately.

#### Authenticated User Dashboard

The authenticated dashboard (#1862, closed December 2024) extends the anonymous version with a personalized feed and user navigation menu. Separate designs exist for OCL Online and for other instances (local deployments). The feeds and events specification is maintained in an external spreadsheet that defines what events appear and under what conditions.

Implementation was deployed to QA in September 2024, prompting iterative feedback. Key design decisions that emerged:

- **"Your Recent Activity"** appears as a sidebar feed. Initial feedback requested more spacing between the profile picture and the feed title, and questioned whether the profile picture was duplicative (#1862).
- **Activity feed behavior:** Events from followed repositories appear in the "All" and "Following" tabs. Org-level events (e.g., a member creating a source version) do not automatically appear unless the user follows the specific entity. This was an intentional architectural decision, though it surprised testers (#1862).
- **Dashboard width:** Initially constrained by design, it was expanded to 1440px after feedback that the layout was too narrow (#1862).
- **"What's New" label:** The "Recent Activity" heading was renamed to "What's New" to match the design spec (#1941, closed).
- **Empty state bug:** A bug where the signed-in empty state showed nothing on staging was fixed (#1967, closed). The corrected empty state shows highlights and OCL branding artwork.

All resources in the activity feed are hyperlinked for navigation (#1862). The feed supports tabs for "All", "Following", and "Organizations" event streams.

#### Dashboard Feed Design

The initial proof-of-concept feed (#1708, closed) was scoped as "recently updated repositories" belonging to the user's orgs or owned repos, sorted by last updated. This was later subsumed into the authenticated dashboard work (#1862).

A separate feed for "Updated Reference Content" (#1771, closed) was planned to show the latest updates to reference terminology content (e.g., standard code systems). This raised questions about how content is marked as "reference" (potentially via the global canonical URL registry), whether change summaries are auto-generated, and what triggers feed entries (likely released versions). The specification was folded into #1861.

#### Registration and Sign-in

Registration was enhanced to include Company and Location fields (#1663, closed January 2024), requiring a custom KeyCloak theme for the registration form. The sign-in/sign-out flows using KeyCloak theming were completed (#1702, closed).

Custom email templates for account verification and password reset (#1858, closed July 2024) were designed by paulsonder and implemented. These include OCL branding, environment-specific subject lines (e.g., "OCL-QA"), and styled HTML bodies. A limitation was discovered: KeyCloak does not support variables in email subject lines (keycloak/keycloak#11883), so environment-specific subjects could not be fully automated. The logo also could not be embedded as originally designed due to a KeyCloak limitation with image embedding in emails (#1858).

#### Community Website Link

A simple but useful feature (#1724, closed January 2024): a link in the Account menu that opens the community website (openconceptlab.org) in a new tab, with an icon per the design spec.

#### Search UI and the Searchlight Component

Search is arguably the most cross-cutting consumer feature, touching global search, in-repo search, and resource selection throughout the application. The search epic (#1694, open) organizes work across multiple child tickets.

**Searchlight** (#1704, closed February 2024) is the unified search component in the header bar. Its core user story: a user can search inside a repository or across all of OCL from one search bar, selecting their preferred scope via arrow-down navigation. The design was implemented and deployed quickly, with refinements including: showing the "Search OCL for..." option only after typing begins, and removing the loading screen when clearing a previous search.

User stories for the broader search experience (#1694) were documented in November 2023 and include: limiting results by free text or filters, specifying which field to search within, exact match support (using quotation marks -- noted as not currently working), navigating concepts without losing search context (via independent left- and right-side panels), selecting multiple results for batch actions (compare, clone, add to repo, download), changing sort order, and getting brief insights about repos/users/owners.

paynejd provided a comprehensive taxonomy of search-like behaviors throughout the app (#1694), identifying: regular search results (global, org-level, user profile, repo-level), less-regular results (clone/add-to-collection workflows, concept details associations, collection reference addition, concept diff), and search-like selection behaviors (concept/mapping create/edit forms, dropdown lists populated by OCL sources). This mapping underscores the need for standardized, reusable search components.

**Search Results Templates** (#1703, closed February 2024) established the template patterns: side filters (always open for global search, closed by default for in-repo views), table and card layouts, and split view. A "Clear N filters" button was implemented. Templates were built for concepts, repositories, orgs, and users. ConceptMap/Mapping search templates were noted as still TBD.

**Searchlight Enhancements** (#1754, open) track additional improvements: the ability to search a user's repos from their profile, and search variations that route to specific tabs in global search (e.g., typing "CIEL" and being routed directly to organization search results). The design is modular, allowing "in-page" search to be handled separately (#1754). Development was approved in July 2024 but status is Todo.

**Search a Repository** (#1877, open) addresses in-page search functionality with requirements for: searching results including hierarchical displays, skipping through results, highlighting matches, clearing search, and viewing paginated results. An early design video demonstrated distinguishing between searching a repo via global search vs. searching while remaining on the page. Known v3 gaps include: results limited to 10,000, owner chips missing avatars, missing latest released version ID column, and Searchlight lacking repository-specific search.

**Loading Performance** (#1741, open) notes that the bouncy ball loading icon appears with every search, potentially making performance feel slower than it is. The proposal defines three levels: instant results, briefly loading, and taking longer than expected, with the loading icon appearing only for the latter two.

**Sort Order Issues** (#1975, open) documents problems discussed at OHIE: duplicate sort values (up and down for each attribute), a confusing default sort by ID, separate sorts for ID and Numerical ID, and multiple sorting mechanisms (Sort By menu and table headers). paulsonder is assigned to simplify this.

**Mapped Concept Icons** (#1974, open) would bring back a v2 feature: an info icon explaining why a concept appears in search results when the match is via a mapping relationship rather than direct text match.

---

### 3b. Repository Views

#### Repository Consumer Epic

The repository consumer epic (#1625, open) is the largest design unit in the consumer experience. jamlung-ri provided an exhaustive catalog of repository information needs in December 2023, organized into tiers:

**Basic required info:** short name plus ID, OCL type (dictionary, source, collection, CodeSystem, ValueSet, ConceptMap, etc.), owner (org or user), latest released version number.

**Secondary optional info:** full repo name with description snippet, content summary (concept count, version count), date of last released version, canonical URL, permission level (public/private), default and supported locales, and a future "trusted"/"verified" indicator (e.g., the real CIEL vs. someone's CIEL subset).

**GitHub-like attributes (aspirational):** activity sparklines, favorite/subscribe counts, most common topics (top 3-5 concept classes).

**Source-specific info:** all basic and secondary info plus content summary (concept/mapping counts, version counts, most common concept classes, locales ordered by frequency).

**Collection-specific info:** same as source except using reference counts instead of version counts, and most commonly referenced sources instead of locales.

A comprehensive table of repository CTAs was documented (#1625): View, Search, Compare, Export, Bookmark/Star, Operations, and Share -- each with user stories and requirements.

**FHIR Acceptance Criteria** were added in August 2024 (#1625): view repo identity (ID, canonical URL, version, name, title), view metadata (publication status, experimental, date, publisher, contact, description, useContext, jurisdiction, purpose, copyright, caseSensitive, hierarchyMeaning, compositional, content, supplements, count), view narrative text for human-readable rendering, view extensions for non-FHIR attributes, and view core content (CodeSystem filters/properties/concepts, ValueSet compose/expansion, ConceptMap groups).

#### Repo Type System

Repo type (#1727, open; #1699, open; #1752, open) is foundational to the consumer experience. The design decision that "Repo IDs should always be shown with Types" (#1776) reflects the reality that in FHIR, ValueSets and CodeSystems can share the same ID. OCL distinguishes between: Source, Collection, ValueSet, Code System, Concept Map, and Dictionary.

A key behavioral decision flagged as "discussion-needed" (#1752): when clicking a repo in global search, the user should navigate to the latest release instead of HEAD. This also affects what the "Latest Release" column shows for repos with no release. This ticket is related to the broader #1699 epic about defaulting to the latest released repo version and hiding individual resource versions.

#### Repository Header

The repo header (#1871, open, Todo status) specifies key attributes: full name, identifier, description, primary and supported locales, website, access permissions, created/updated dates, picture/icon, selected version, and ability to "View all attributes." This remains in the design backlog.

#### Repository Version Selector

The version selector (#1872, open, Todo) replaces the v2 versions tab. The design decision (#1955, closed, Done) was to deprecate a separate versions tab in favor of positioning the version picker in the page hierarchy so that content below inherits the selected version. The picker includes more information than v2 to function as both a list and a menu. Designs include states for: no version selected (with current version highlighted), and multiple versions selected (enabling comparison).

Implementation feedback (#1955) identified issues: private versions showing no way to navigate back without the browser back button, and empty icons appearing when no default expansion exists for a collection version (suggesting "N/A" placeholder text with a tooltip).

The version metadata was moved from a separate area to the header and expansions table (#2275, closed, Done).

#### Repository Summary

The repo summary panel (#1950, closed December 2024; originally from #1478) sits on the right side of the repo template. It shows counts of concepts, mappings, concept classes, datatypes, map types, languages, and name types. For a selected repo version, it shows active/retired breakdowns.

**Design evolution:** The original v2 implementation (#1478) was acknowledged as "scrappy." paulsonder suggested bringing the summary into the repo right bar, but performance concerns emerged -- CIEL's summary took about 20 seconds to load (#1478). The summary also needed to account for expanding details (originally in tooltip hovers) and for collection-specific content showing which source versions are being pulled from (#1478).

**UI decisions from v2:** Skeleton loading was preferred over spinner loaders -- "Skeleton all the way" (paulsonder, #1478). Retired concepts indicator bars should show Active out of Total rather than leading with retired/red (#1478). Mapped-to/from controls should sit inside accordions for obvious interactivity (#1478).

**Implementation in v3:** Each summary statistic line is intended to be clickable, filtering the concept/mapping list accordingly (#1950). Icons were updated to match designs. A "re-run summary" link was added for cases where summary calculations fail (#1950), addressing two failure modes: HEAD version not updating counts, and version calculation jobs failing (the 95%+ case per snyaggarwal).

**Open question:** How the canonical registry should be represented in the summary panel remains under discussion (#1950).

#### Repository Overview / About

The overview tab (#1951, open) houses "About" information in structured lists aligned with FHIR spec organization. The intent is to "visually and mentally start aligning our users towards a more consistent presentation" of content metadata (#1951). An extensive list of fields was compiled, including both OCL-native attributes (created on/by, custom attributes, locales, website, external ID, processing status, auto-ID settings) and FHIR attributes (content_type, revision_date, case_sensitive, collection_reference, compositional, meta, hierarchy_meaning) (#1951).

A separate "About" page (#1892, open, Todo) handles verbose descriptions, license information, and repository activity feeds. It mirrors the approach used for User and Org pages with defined empty states.

An empty state design was created for repos with no About content (#1951).

#### Repository Comparison

Version comparison (#1853, open, Testing/QA status) allows consumers to see differences between two repository versions. User stories define three levels: summary of differences (stat counts), list of specific changes (new/retired concepts), and expansion-level comparison (comparing expansions from potentially the same version).

Out-of-scope-for-now user stories include: seeing changes relevant to content the implementer uses, comparing FHIR resources, and comparing $lookup responses.

Design was delivered in September 2024 with videos demonstrating loading states and version selection UX. Implementation is in QA as of May 2025, with last steps including grouping the concept list by change type, adding icons for concept comparison view, and adding concept detail previews (#1853).

A follow-up ticket (#2181, open) tracks MVP gaps: mappings and JSON diff in the comparison view. The JSON diff approach draws cues from jsondiff.com (#1853).

#### Objects Chip System

The chip/object view (#1776, open, Backlog) is a cross-cutting design system component that represents repositories, concepts, users, and organizations as compact, recognizable visual elements. This went through extensive design iteration from March through October 2024.

**Core design problem:** OCL is multi-tenant, so the same code system ID and canonical URL can be defined multiple times. A chip must convey: repo ID (always required), repo type (always required), owner (usually), and version (usually). Without all four, you often cannot identify which repo is being discussed (#1776).

**Key design decisions:**

- Repo ID and repo type are the most important and should be emphasized visually, even though the owner comes first in the tree hierarchy (#1776).
- Chips should not wrap text per Material guidelines; truncation with tooltips handles overflow (#1776).
- A "no version" state simply omits the version content rather than displaying "no version" text (#1776).
- Version-less references need distinct handling: "latest" is a special keyword always pointing to the latest release, "latest (v2.77)" is a resolved reference, and bare "latest" is unresolved (#1776).

**Chip variants were designed for:** Organizations, Users, Repositories, and Concepts, each with corresponding tooltip designs showing extended information (#1776, September 2024). After the October 2024 dev/UAT call, slight updates were requested.

**Usage guidelines:** Chips are used in repository headers and dashboard event cards. Chips always show corresponding tooltips. A proposal to avoid chips in data tables was confirmed -- data tables use a different layout (#1776). paulsonder raised an important design system concern: when to use chips vs. other representations, noting that a list full of "fully loaded chips" would be difficult to read.

**Implementation questions** from snyaggarwal (#1776): what representation to show where (rules), what is mandatory without tooltip, and what goes inside the tooltip. A key rule: "Repo should never be shown without repo type and may be always with version (if applicable)."

#### Repo Type Filter

A repo type filter in repository listings (#2252, closed, Done) was implemented to allow filtering by repository type in search results.

#### Following and Bookmarking

Following/bookmarking (#1780, closed December 2024) underwent significant conceptual evolution. The initial framing was "bookmarks" -- a way to tag resources for quick navigation and narrower search results. Discussion evolved this into a "Follow" model, more aligned with GitHub conventions (#1780).

**Conceptual distinction:** Pins are how an owner highlights content they own to other viewers. Bookmarks/follows are how a user tags any owner/repo for personal tracking. These must remain separate (#1780). The API backend model is flexible: the existing Pin model (with resource_type, resource_uri, and order) was proposed to be extended with a `type` field for Pin, Bookmark, or Follow, with type-specific validations (#1780).

**Followable resources:** Organizations, repositories, users, and concepts. Concept following supports monitoring for updated mappings/attributes. Concept Proposals would be auto-followed once implemented (#1780).

**Events framework:** Events were documented in a separate spreadsheet. Initial events implemented include: Org Create/Delete, Source Create, Source Version Create, Follow/Unfollow User (#1780). Events for deleted resources initially showed "Created None" / "Deleted None" when the underlying resource no longer existed -- this was fixed (#1780).

**UI implementation:** An Input Chip design was used for follow/unfollow states. The follow/unfollow feature was deployed to QA in August 2024. Dashboard integration followed, with the "Your recent activity" section and the main feed reflecting followed entities.

#### Download and Export

Repository download/export (#1873, open) requires: compressed OCL-formatted JSON, CSV, and FHIR representation downloads. Mapping download (#1880, open) requires OCL-formatted JSON and potentially a FHIR representation of a single mapping, with clarity on which repo version applies.

#### Mapping Listing

The mapping listing view (#1758, open, marked OHIE Must-Haves) went through multiple design cycles. The original v2-derived design was noted as the "only place in the termbrowser where we display a concept like this" with just an ID, prompting a call for showing display names instead (#1758).

**Requirements (#1758):** filterable list, key attributes always visible (source/target repos, from/to codes, map type), visual indicator for custom attributes with expand option, indication of whether from/to concept exists in OCL with navigation, and retired styling for mappings and concepts.

**Design evolution:** paulsonder proposed grouping by source, with hover states and clickable concepts (#1758). jamlung-ri suggested incorporating repo chips (#1776) for better previews and proposed an MUI Collapsible Table approach: groupings by From Source, To Source, then Map Type as width-spanning collapsed rows that expand to show concept details (#1758).

The new mapping list design (#1948, open, Backlog) uses AG Grid with a Material theme for grouped data presentation. Grouping is by From Source + Target Source, with expandable rows. The design includes resolved/unresolved styling and retired concept indicators (#1948). Implementation was deployed to QA in November 2024 without grouping initially. Feedback addressed: long source IDs overflowing cells, broken hover tooltips on unresolvable repos, and concept tooltips not yet implemented (#1948).

**Grouping and filtering discussion (#1948):** Two grouping approaches -- (from source + target source) and (from source + from concept) -- serve different use cases. Filtering by map type is essential; filtering by target source and additional metadata (e.g., mappings with comments) is useful. Pagination should use infinite scroll given the hierarchical nature of mapping data.

**In-namespace mappings** (#1972, open, Testing/QA) addresses showing mappings within the same namespace in concept details.

#### Mapping Detail View

The mapping detail view (#1952, closed December 2024) was designed and implemented with iterative QA feedback. Key details:

- From/to concept boxes with source context, clickable to navigate to the concept
- Unresolved sources/concepts show no hyperlinks and no hover tooltips
- Truncated source text shows a tooltip on hover
- Properties section shows sort weight and extra attributes, hidden when empty
- Standard and smart checksums moved to the footer (borrowing from GitHub's checksum display approach)
- Source preview tooltips show short name/ID rather than relative URL
- Custom attributes appear with a "Custom" tag

A transition smoothness issue between mapping selections was noted but deferred to a future ticket (#1952).

#### References and Expansions

References and expansions are among the most complex consumer views, reflecting OCL's unique approach to building collections/value sets from reference expressions.

**Reference List** (#1949, open, In Development) shows references with their expressions, filters, and results. The design includes two views: expression/filter view and results view. Feedback highlighted the need to show whether references point to concepts or mappings, to clearly distinguish intensional references from grouped extensional references, and to show mapping references. This was acknowledged as "one of the hardest and most complex views" in TBv3 (#1949). A v2-based version was deployed to QA and Staging in late 2025 (#1949).

**Reference Detail** (#1953, open) shows individual reference attributes and a "View Expansion" tab with resolved concepts and mappings, including orphaned mappings and the resolved source URL. The design supports selecting different expansions. Closed duplicate: #1894.

**Reference Preview** (#1895, open) enables evaluating a reference on-the-fly without saving to an expansion -- critical for understanding how OCL would resolve a reference in isolation.

**Reference Search** (#1896, open) faces a technical limitation: references are not indexed in Elasticsearch, limiting search capabilities. Adding ES indexing with facets was proposed (#1896).

**Expansion View** (#1889, open) shows concepts/mappings resulting from expanding references, with optional grouping by reference/repo, filtering, expansion parameters, repo versions used, canonical URL resolution details, and all metadata.

**Collection References Tab** (#2123, open) tracks the broader consumer view of repository references and expansions.

**History for concepts and mappings** (#2290, open, User Testing) implements version history viewing in TBv3.

---

### 3c. Concept Views

#### Concept Detail Header

The concept detail header (#1620, open, Backlog) is assigned to paulsonder and tracked under the TermBrowser v3 MVP milestone. Specific requirements have not been extensively documented in the available tickets, but it follows the same header pattern established for repositories: key identifying information, version context, and action buttons.

#### Names and Descriptions

The names and descriptions card (#1734, open) has detailed requirements and went through thorough design review:

**Display requirements:** Group by locale/language (2-3 letter codes, lowercase by convention). Within a locale, sort by: "Preferred within locale" flag and Fully Specified Names first, then tagged names, then untagged names. Show name/description with truncation for long text, name/description type (as a chip), external ID with copy functionality, visual indicator for "preferred name within locale" flag, and ability to copy URL for each name/description (#1734).

**Design iteration:** paulsonder delivered initial designs in April 2024 showing the card in various states. jamlung-ri's feedback requested: multiple names within one locale (e.g., several EN names), long descriptions, auto-generation indicator for external IDs, and a "finished with names" workflow that submits all changes in a single API call (#1734).

**Key versioning decision:** Changes should be saved only when a user clicks "Finished editing," creating one new version with all changes rather than a separate version for each individual name addition. This was confirmed as the preferred approach (#1734). A dialog warning about unsaved changes when leaving edit mode was also planned.

**Long text handling:** MUI's textarea-autosize component was selected for fields that may have long descriptions (#1734).

#### Concept History

Concept version history (#1739, open) is tracked for design under the TermBrowser v3 MVP milestone. Implementation of history viewing for concepts and mappings in TBv3 is in User Testing (#2290).

#### Associations Card

The concept associations card (#1621, open, Testing/QA) shows mappings/associations for a concept. This is essentially cascade results with no recursion (as noted by paynejd in #1694). The design discussion included how to better display hierarchy in this panel and whether hierarchy editing should be incorporated (#1822, open), though hierarchy is technically different from mappings.

#### Collection Membership

Collection membership (#1738, open, marked discussion-needed) shows which collections contain a given concept. Specific requirements and design work are pending.

#### Hierarchy and Cascade

Hierarchy/cascade visualization spans multiple tickets. The cascade preview visualizer (#1455, open) is a top-level item. The hierarchy/cascade component (#1700, open) addresses "seeing context for a concept" -- critical for hierarchical code systems like ICD.

jamlung-ri documented current hierarchy/cascade appearances in OCL (#1700): Add to Collection (cascade only, no visualization), Clone workflow, concept details in global search, operations panel, hierarchy tree view (ICD), and cascade visualizer (for both ICD hierarchy and CIEL mappings).

The hierarchy as part of Add/Edit Mappings (#1822, open) emerged from discussions about showing parent/child relationships alongside mappings in concept details, noting that hierarchy relationships are technically different from mappings.

---

### 3d. User & Org Profiles

#### User Profile

The user profile went through two rounds of design and development. **Round 1** (#1756, closed March 2024) established the basic layout: left panel with avatar, name, organizations, and right panel with repos. Designs were provided for both viewing and editing states.

**Key decisions from Round 1:**

- Edit button placement follows GitHub's pattern: below the name/username (#1756)
- Left nav with icons is kept consistent across pages (#1756)
- Organizations without logos use 2-character abbreviation with hover tooltip, preferred over GitHub's random placeholder images (#1756)
- "My Repositories" shows repos owned by the user and by orgs where the user is a member; "User Profile" shows only repos owned directly by the user (#1756)
- A new API endpoint was needed to list user's repos plus repos owned by orgs the user belongs to (#1756)
- Owner column should be removed from the User Profile repo list since all repos are owned by the viewing user (#1756)

**Empty state and activity feed** (#1800, closed July 2024) addressed the problem that new users see an empty, valueless page. Activity feed events include: Joined OCL, Updated Profile, Created an Organization, Became a member of [org], Created/Published a Repository. An "About Me" section encourages users to write a blurb with a "Tell us about yourself" prompt (#1800).

**User summary statistics** available from the API (#1756): count of sources, collections, org memberships, directly owned orgs, and bookmarks. The team noted the lack of a verbose summary API for users/orgs and suggested creating one (#1756).

**Profile access permissions** (#1852, closed June 2024): Anonymous users cannot view other profiles; logged-in users can see any profile. Actions like "Copy API Token" and "Edit Profile" are restricted to the profile owner. The 403 vs 401 distinction was clarified: 403 is for authenticated users without access (no sign-in prompt needed), 401 is for unauthenticated users (already has sign-in/register) (#1852).

**User bio** (#1942, closed October 2024): enforced a character limit (under 200 characters), with the short bio displayed next to avatar and username.

**User settings page** (#1798, open, discussion-needed): a single entry point for customizing the OCL experience, including personal settings and links to collaborative object settings (e.g., Organization editing). Design shows a left-hand navigation with different setting categories, some editable in-page and others linking to specific edit UIs.

**User canonical registry** (#1888, open): adds a "View Canonical URL Registry" CTA to user profiles when registry entries exist. paulsonder prototyped a modal approach that keeps the user in context rather than navigating to Settings. The indicator appears only when registry entries are configured. A "Learn more" link would point to either help documentation or the canonical URL tool itself (#1888).

**Form / Edit user profile** (#1617, closed) and **Profile / User** (#1627, closed) completed the core user profile implementation.

#### Organization Views

Organization views followed a similar two-round pattern. **Round 1** (#1757, closed March 2024) established the basic org home page with an overview tab (About section, pinned repos) and a repositories tab.

**Design challenges:** Empty organizations posed a significant UX problem. When an org has no About section, no bookmarks, and no repos, the page was barren (#1757). paulsonder proposed combining the About and Repos sections (following GitHub's approach), adding pinned repositories, a featured repositories section (showing most recently updated repos unless the owner specifies others), and an empty state design (#1757).

**Pinned vs. Featured distinction** (#1757): jamlung-ri raised the question of differentiating pinning (more highlighted) from featuring. A "truly empty" state design was created for brand-new organizations with only minimum info, drawing inspiration from GitHub's blank repo page with immediate calls to action (#1757).

**Round 2** (#1799, closed August 2024) addressed: aggregate statistics across repos (novel concepts, published versions, most common concept classes), org ID in the header, member overflow for 8+ members, CIEL icon display, and empty state handling.

Detailed implementation in Round 2 included:

- "View all attributes" as a two-column table in a modal/popup for core and custom attributes (#1799)
- "View Canonical URL Registry" link when a registry exists (#1799)
- Browse Members dialog triggered from the org statistics panel (#1799)
- Default org avatar for orgs without uploaded logos (#1799)
- Org ID (short code) added to the header after it was noticed missing (#1799)
- Created date and metadata display in the About panel (#1799)
- Attribute redundancy eliminated between header and additional attributes popup (#1799)

**Outstanding items for org views:** Activity feed (dependent on events framework), number of followers statistic, "Add a member" button (no design yet), and download/share button consolidation.

**Branding re-convergence (2026-07-22):** The deferred org landing-page branding feature (see Organization > Organization Landing Page Branding in `tbv3-deferred-features.md`) is now trending back toward this Round 1 GitHub-inspired About+Repos direction rather than a full banner/color-editing model. Andy is prototyping a GitHub-README-style mockup for CL.

**Global Search for Orgs** (#1864, open) tracks enhancements: mini org avatar in results, org summary (sources/collections/members count), filters/facets, and card view. paulsonder proposed keeping global search results inside Searchlight (as a modal overlay) rather than navigating to a separate page, which jamlung-ri supported for both UX and workflow benefits (e.g., the "add to collection" workflow could use a windowed global searchlight) (#1864).

**Org/User pinned repos sizing** (#1966, closed, Done) fixed a bug where pinned repos were shrinking too much on the page.

#### My Repositories

"My Repositories" (#1802, open) is an epic covering the user's personal repository management view. The initial implementation (#1755, closed March 2024) was deployed to QA with: table view of repos, pinned repos, and repo creation. Feedback led to: adding an Owner column, fixing pinned repo display (showing pin ID instead of repo name), and handling blank descriptions gracefully (show description, then full name, then nothing) (#1755).

Follow-up tickets include: Card View for My Repositories (#1803, open) and Create/Update from My Repositories (#1804, open, Testing/QA).

The distinction between "My Repositories" and the User Profile page was debated (#1755, #1756): My Repositories shows repos from all orgs the user belongs to plus the user's own repos, while the User Profile shows only directly owned repos. This mirrors GitHub's approach.

#### Sharing

Concept sharing (#1865, open, Backlog) defines a Share button with dropdown options: "Share this Page" (human-readable URL preserving search context), "Copy as FHIR Coding" (canonical pointer in FHIR Coding format, only for CodeSystem/Source with canonical URL), and "Copy OCL API" (API URL). The FHIR Coding option follows the FHIR spec format with system, version, code, and display (#1865). An open question remains about how sharing works within the context of a collection (#1865).

Repository sharing (#1916, open) requires human-readable URLs that navigate to the same location, an optional Share button, and FHIR representation links if available.

---

### Summary of Status and Key Patterns

#### Completed Work
- Dashboard for both anonymous and authenticated users (#1688 epic closed, children #1612, #1663, #1702, #1708, #1771, #1858, #1861, #1862, #1941, #1967 all closed)
- Searchlight component (#1704 closed) and search results templates (#1703 closed)
- Community website link (#1724 closed)
- User Profile Round 1 (#1756 closed) and Round 2 (#1800, #1627, #1617, #1852, #1942 closed)
- Org Home Round 1 (#1757 closed) and Round 2 (#1799 closed)
- My Repositories initial implementation (#1755 closed)
- Following/bookmarking (#1780 closed)
- Repo Summary (#1950 closed), Versions (#1955 closed), Mapping Detail (#1952 closed)
- Design for repo+version display (#1715 closed, subsumed into #1776)
- Custom emails (#1858 closed)
- Repo Type Filter (#2252 closed)
- Repo version metadata in header (#2275 closed)

#### In Progress / Testing
- Repo version comparison (#1853, Testing/QA)
- Concept associations card (#1621, Testing/QA)
- In-namespace mappings (#1972, Testing/QA)
- Concept/mapping history (#2290, User Testing)
- Collection References tab (#2273, User Testing)
- References list (#1949, In Development)

#### Open Design / Planned
- Concept detail header (#1620, Backlog)
- Objects chip system (#1776, Backlog)
- Repo header (#1871, Todo), version selector (#1872, Todo), overview (#1951, Todo)
- About page (#1892, Todo)
- Mapping listing with grouping (#1758, #1948, discussion-needed)
- Search enhancements (#1754, #1877, #1866)
- Download/export (#1873, #1880)
- Sharing (#1865, #1916)
- Reference detail/preview/search (#1953, #1895, #1896)
- User settings (#1798, discussion-needed)
- Hierarchy/cascade (#1455, #1700, #1822)
- Names and descriptions (#1734)
- Collection membership (#1738, discussion-needed)
- Sort order fixes (#1975)
- Loading performance benchmarks (#1741)

#### Design Principles Observed

1. **Skeleton loading over spinners** -- confirmed early and applied throughout (#1478)
2. **GitHub as UX precedent** -- profile layouts, edit buttons, empty states, following model all draw from GitHub patterns
3. **FHIR alignment** -- repo overview structured to match FHIR resource metadata, acceptance criteria include FHIR search parameters, sharing includes FHIR Coding format
4. **Modular components** -- chips, tooltips, search templates designed for reuse across all resource types
5. **"Finished editing" pattern** -- batch save rather than per-field versioning, creating one version with all changes
6. **Infinite scroll for hierarchical/grouped data** -- preferred pagination method for mappings and similar nested content
7. **Context preservation** -- users should be able to search and browse without losing their current position (split panels, modal overlays, Searchlight staying in-page)


---

# Section 4: Authoring Experience (Creating & Editing)

The authoring experience in TBv3 represents the transition from OCL as a read-mostly terminology browser to a full authoring environment. The system organizes authoring actions around a consistent CTA (Call-to-Action) pattern, where each user action is modeled as a discrete operation accessible from contextual menus, toolbars, and entry points throughout the UI. This section covers six authoring domains -- repositories, concepts, mappings, organizations, expansions, and references -- plus the design specifications that bind them together.

All authoring takes place within a Material Design 3 (M3) Side Sheet component (#1376), which provides a consistent editing surface that overlays the current view. Buttons in the Side Sheet are sticky, validation is inline, and dropdown fields leverage the Searchlight component for resource selection. This Side Sheet pattern is foundational: it means authoring never navigates users away from their current context, a deliberate design decision to support the "view content in list or card view in the same UX" user story (#1376).


## 4a. Repository Authoring

The Repository authoring epic (#1977) covers the full lifecycle of repository management: create, edit, version, release, copy, and type-specific templates. Repository CTAs are primarily accessible from a "More" menu in the repository header (#1977).

### Repository Creation

Repository creation (#1978) supports two entry points: from an Organization page (pre-selecting the org as owner) or from the global "Create" menu (where the user selects an owner). The workflow proceeds through a repo type selection step, then a type-specific form, then creation with navigation to the new repository. Designs for this flow are in Zeplin (#1978).

A key design decision arose around the default repo type selection. The initial designs showed "Code System" as the default selected type, but the team decided "Source" should be the default, since it is the most general OCL repo type (#1978, comment from paulsonder). Joe (jamlung-ri) approved the flow structure, noting content placeholders should be left in designs for him to populate (#1978).

Several implementation details surfaced during review:

- **Auto-ID fields**: Sunny (snyaggarwal) flagged the need for design elements around auto-ID assignment, which in v2 lived in "Advanced Settings" (#1978). Joe provided a detailed user story: terminology authors need to configure auto-ID assignment methods (None, Sequential, UUID) for concept IDs, mapping IDs, external IDs, and name/description external IDs. Sequential IDs require a starting number, and future iterations may support prefixes/suffixes (#1978).
- **Repo About details**: Initially missing from the create form, this was added back to the Zeplin designs (#1978).
- **Owner and source context**: The owning source is set based on where the user initiated creation. The entry point for creating a concept should always be inside a repo, not from global search (#1376, paynejd comment).

Repository creation is in Testing/QA status (#1978, #1615).

### Repository Editing

Repository editing (#1979) reuses the same form as creation but with a modified header state and pre-populated fields. The form is accessible from the "Edit" action in the repository header menu. The key constraint is that repo type and certain identifiers cannot be changed after creation (#1979). This is also in Testing/QA status.

**Validation Schema Change UX.** One unaddressed UX gap in the repository edit form concerns the validation schema field. The current UX treats changing the validation schema as a synchronous operation -- the user selects a new schema, saves the form, and the change appears immediate. In reality, re-validating all content against a new schema can be a long-running asynchronous operation (20--30 minutes or more for a large source). The edit form should account for this: while a schema change is in progress, the schema field should be unselectable and display a clear in-progress indicator (with an option to cancel the operation). If a schema change fails and the task log is still available, a small note near the schema field should surface the failure inline -- for example, "(Schema change to 'OpenMRS' failed YYYY-MM-DD HH:MM:SS. Click here for report)" -- so the user does not have to hunt for the error in a separate log view. This is related to the broader validation schema errors issue (#962, Marked for Review).

### Versioning and Release

**Create version** (#1980) generates a snapshot of a repository's content and metadata at a specific moment. The user clicks "Create Version" from the repository page, completes a form, and is returned to the repository with the new version selected. Designs were reviewed and approved with only minor comments (#1980). In Testing/QA.

**Release (or un-release) a version** (#1981) allows users to mark a version as released or unreleased. The action is in the repository header menu, requires confirmation via a dialog, and results in a visual "Released" chip in the summary panel. The version menu also indicates release status for each version (#1981).

An important open question emerged around release status and FHIR alignment. Joe noted that OCL's Released/Unreleased model is similar to FHIR's `status` attribute (draft | active | retired | unknown) and asked whether the design could accommodate a future migration to FHIR's model (#1981). Jon (paynejd) pointed out that OCL already implements the FHIR status attribute in the API and suggested making it live in v3. This discussion was tagged `discussion-needed` and remains unresolved -- the team has not yet decided whether to align OCL's release status with FHIR's status model or display both (#1981). This is in Testing/QA.

### Copy (Fork) and Advanced Creation

**Make a copy** (#1982) allows users to create a new repository pre-populated with metadata and content from an existing one. The owner field is left empty and focused, prompting the user to select a namespace. Joe noted potential issues with the underlying Create Repository form fields (e.g., "Hierarchy Meaning" placement), and Paul redirected that feedback to #1978. This feature is in Backlog status.

Three newer tickets (April 2025) define a more sophisticated repo creation starting point:

- **Repo Creation Starting Point UI** (#2155): A decision screen where users select HOW they want to create a repo -- from scratch, fork an existing repo version, or load content from a file/URL. The pathway is pre-selected based on context (e.g., which CTA brought the user here) but can be changed.
- **Fork a repo version** (#2156): After selecting "Fork" from #2155, the user finds a repository and selects a version. The selected repo version provides base metadata (and possibly content) for the new repo. Repo type is inherited, so the user skips that selection step.
- **Import content from existing repo** (#2157): Following a fork or other repo creation from an existing repo, users choose whether to: (a) not bring in content, (b) clone content (create new concepts/mappings), or (c) reference content (create references to existing concepts/mappings). The reference option is only available for collection-based repos. Long-running imports are queued with a Task ID.

All three are open and assigned to the TermBrowser v3 MVP milestone.

### Repository Templates

Five repository templates define the page structure and tab configuration for each repo type:

| Template | Tabs | Status |
|----------|------|--------|
| **Source** (#2025) | Overview, Concepts (ID, Display name, Concept Class, Datatype), Mappings (grouped by Target Source) | Todo |
| **Collection** (#2026) | Overview, Composition (Filters, Intensional/Extensional), Concepts (with Expansion picker, includes Owner column), Mappings | Todo |
| **ValueSet** (#2027) | Same structure as Collection | Todo |
| **CodeSystem** (#2028) | Not yet defined (empty body) | Todo |
| **Concept Map** (#2029) | Not yet defined (empty body) | Todo |

The Collection template's high-level structure was approved by Joe (#2026). Paul asked Sunny whether there was enough detail for development; the question of what additional information developers need per template remains open (#2026).


## 4b. Concept Authoring

The Concept authoring epic (#1989) and its foundational design ticket (#1376) represent one of the most design-intensive areas of TBv3. Concept authoring is the first place TBv3 advances the "Smart Dictionaries" vision, where the tool helps users follow best practices rather than simply accepting arbitrary input.

### Design Foundation: Create and Update Concepts and Mappings

The design specification (#1376) establishes the core interaction model. Concept and mapping CRUD operations take place in the Side Sheet component. The design went through a multi-phase sprint process (documented by paulsonder in December 2022) and evolved significantly through 2024.

Key design decisions from the #1376 thread:

**Form layout for names/synonyms**: Concepts can have 100+ names, so the team considered squeezing each name (Locale, Type, Name, External ID, Preferred) into a single line or using a table-based editing component. The decision was to explore MUI's data grid component (`@mui/x-react-data-grid`) for this purpose (#1376, paynejd checklist).

**Smart defaults and best practices guidance** (#1376):
- Pre-populate language selections based on repo defaults (decided)
- Require one (and only one) Fully Specified Name per concept (decided)
- Concept templates: users select a template at the top of the form, with "Blank" as one option (decided for launch)
- ID fields should be "aware" of the repo's ID auto-assignment settings (decided)
- First concept name defaults to "Fully specified" and "preferred" (decided)
- Additional "Smart Dictionary" guardrails are planned but not yet specified

**Non-editable context fields**: Owner and source are displayed at the top of the concept form but are not editable -- they are determined by where the user initiated the create action (#1376). When editing a concept, the Concept ID is not editable (#1376, snyaggarwal).

**Concept URL display**: Sunny requested that the URL where the concept will live be displayed in the form (#1376).

The design shows three modes for the concept detail view: Read, Edit, and Create (#1376, paulsonder April 2024 designs). Joe requested a visual for inline editing of name fields (e.g., what happens when a user clicks on a name in Edit Mode) (#1376). This interaction pattern -- whether clicking text makes it editable inline vs. requiring a three-dot menu -- was raised but the resolution is not explicitly documented in the thread.

The design is dependent on the Searchlight component (#1830) for dropdown resource selection within forms (#1376).

### Concept CTA Operations

| CTA | Ticket | Status | Notes |
|-----|--------|--------|-------|
| Edit | #1991 | Testing/QA | Assigned to snyaggarwal |
| Clone to repo | #1992 | Open | No body or comments; MVP milestone |
| Add to repo | #1993 | Open | No body or comments; MVP milestone |
| Create similar | #1994 | Open | No body or comments; MVP milestone |
| Add mapping | #1995 | Todo | "Quick add mapping" from concept details; paynejd directed to follow TBv2 design pattern with the Searchlight-based approach |
| Create mapped concept | #1996 | Open | No body or comments; no milestone |
| Retire | #1997 | Testing/QA | No body; MVP milestone |
| Import code list | #1998 | Open | No body or comments; no milestone |

The "Add mapping" CTA (#1995) is notable because paynejd explicitly stated it should implement the "quick add mapping" feature for the concept details page, following TBv2's design. This is a priority feature for the OpenMRS community.

**Quick Add Mappings in Collection (#1361).** A related but distinct requirement covers the collection context: from a concept viewed within a collection the user owns, the user should be able to click a button to add a new mapping -- analogous to the quick-add mapping in sources (#1359). The key complication is that collections do not own a source, so any new mapping created this way must have a designated source to live in. The design must therefore prompt the user to select or confirm a target source for the mapping before it can be saved. This issue is Marked for Review, meaning it has not yet been redesigned or specified for v3.

"Clone to repo," "Add to repo," and "Create similar" (#1992, #1993, #1994) are all specified at the title level only with no acceptance criteria, body content, or comments. These represent planned operations that have not yet been designed or discussed.

"Import code list" (#1998) similarly has no specification. This would allow users to import a list of codes directly into a source, bypassing the standard bulk import workflow.


## 4c. Mapping Authoring

The Mapping authoring epic (#1999) covers five operations: create, edit, create similar, retire, and delete. As of mid-2025, mapping authoring has received significant attention for both the form design and the interaction model.

### Mapping Form Design

Joe provided a detailed specification for the mapping form structure in June 2025 (#1999):

**Form sections:**
1. **Context** (already implemented): Organization and Source that will contain the mapping
2. **Header** (mostly implemented): Contains Map Type with a "Suggested" section if available
3. **From/To Concept** (user input): The core of the form
   - Default behavior borrows the Quick Add Mapping field structure from concept details (#1995)
   - Optional toggle to convert From or To section to free-text mode, which reveals additional mapping fields
   - Autofill of resolved concept name when available, with user override capability
   - Retains the "swap From/To" button from v2
4. **Resolution Line**: A dynamic preview showing what the mapping row will look like in the mappings list view, updating in real-time during editing
5. **Attributes**: External ID, Sort Weight, Custom Attributes
6. **Submit button with Update Comment**: An Update Comment field next to the submit button, with suggested autofill logic (e.g., "Updated Map Type, To Concept, External ID; Added Sort Weight, Custom Attribute(s)")

Joe provided a crude mockup via Google Drawings for this form layout (#1999).

### Critical Mapping Form Gap

Jon (paynejd) flagged the current Create/Edit mapping form as "a non-starter" in May 2025 (#2000). The from/to concept fields must be replaced with modern components that use the Searchlight component for resource search and selection. This is a blocking requirement for the mapping authoring experience.

### Mapping CTA Operations

| CTA | Ticket | Status |
|-----|--------|--------|
| Create | #2000 | Testing/QA |
| Edit | #2001 | Testing/QA |
| Create similar | #2002 | Open |
| Retire | #2003 | Testing/QA |
| Delete | #2004 | Testing/QA |

Four of the five mapping CTAs are in Testing/QA, making this one of the more advanced authoring areas. "Create similar" (#2002) remains open with no body or comments.

Note that mapping authoring deliberately excludes a "Clone" or "Copy" action at the mapping level. Mappings are authored in the context of a source, and the "Create similar" action serves as the mechanism for duplicating a mapping with modifications.

### Open Questions

- The from/to concept fields need Searchlight integration (#2000) -- the current implementation is insufficient
- Whether and how to handle mappings where from/to concepts point to resources outside OCL (i.e., external canonical URLs) (#1376, paynejd: "From Concept and To Concept do not always point to repos or concepts that OCL knows about")
- Mapping ID and External ID fields need to be "aware" of repo ID settings, similar to concepts (#1376)


## 4d. Organization Management

The Organization management epic (#2013) covers the full organizational lifecycle. Organizations are the primary namespace mechanism in OCL, and their management has distinct considerations around branding, membership, and content ownership.

### Organization Creation

Creating an organization (#2014) is accessible from the "Create" menu in the navigation bar or from the Quick Links drawer. The form designs are in Zeplin.

Key design discussions:

**Custom attributes**: Paul asked whether orgs should support custom attributes. Joe confirmed it is uncommon but useful, noting that custom attributes could be displayed in the same format as FHIR attributes in a repo overview, using a structured list component (#2014). The team agreed to support this with low emphasis.

**Avatar/identity fallbacks**: The team discussed whether avatars should be mandatory. The consensus is that avatars should be encouraged but not mandatory, with a generated fallback (like GitHub's Identicons or Jazzicons) for orgs and users that do not upload an image (#2014). Paul noted the distinction between org and user identity fallbacks -- the approach should be consistent across both entity types (#2014).

### Organization Editing and Membership

**Edit** (#2016): Uses the same form as creation but for updating an organization's metadata. Requires owner or member-with-edit-permissions access. Design docs are TBC.

**Add members** (#2015): Users search for new members and add them to the organization. Requires owner or member-with-edit-permissions access. Design docs are TBC.

**Create repository** (#2017) from an org page pre-selects the organization as the owner in the repo creation flow. This connects to the repository creation workflow described in Section 4a.

### Organization Customization

**Pin repository** (#2018): Allows prominently positioning repos on the org page. The team discussed the purpose of pins, concluding that with v3's new Dashboard and Follows features, pins serve primarily to highlight content for visitors to the org page, not for personal bookmarking (#2018). The current 4-pin limit was deemed sufficient; increasing it could defeat the highlighting purpose (#2018). Status: Backlog.

**Customize appearance** (#2019): For MVP, limited to avatar and header background image uploads. Paul scoped this deliberately to avoid the complexity of v2's white-label capabilities. Joe documented what v2 features would be lost:
- Banner settings (height, solid color, different banners per tab)
- Text color customization (title, description, other text)
- Show/hide controls (logo, signatures, controls, pins)

The team agreed these can be dropped for v3, but the new approach must still support existing branded orgs like WHO-DDCC, CIEL, and PEPFAR MSP (#2019).

**Edit canonical registry** (#2020): Allows org owners/editors to add or change entries in the organization's canonical URL registry. This connects to the broader Canonical URL Registry infrastructure (covered separately).

### Organization Deletion and Sharing

**Delete** (#2021): Requires confirmation by typing the organization's name, carrying over the v2 pattern. The destructive nature justified keeping this safety measure for v3 (#2021). Status: Testing/QA.

**Share** (#2022): The MVP is "Copy URL to Clipboard." The team discussed richer sharing (email, social media, OpenGraph previews) but agreed to start simple and iterate post-beta (#2022).


## 4e. Expansion Management

The Expansion management epic (#1983) covers five operations for managing expansions within collections. Expansions are the materialized "snapshots" of a collection's references -- they resolve references into concrete lists of concepts and mappings.

### CTA Operations

| CTA | Ticket | Status | Milestone |
|-----|--------|--------|-----------|
| Create ($expand) | #1984 | Open | TBv3 MVP |
| Set as default | #1985 | Open | TBv3 MVP |
| Delete | #1986 | Open | TBv3 MVP |
| Update metadata | #1987 | Open | None |
| Rebuild expansion | #1988 | Open | TBv3 MVP |

All five expansion CTAs have empty bodies and no comments. This is the least specified of the authoring areas -- the CTA titles define the intended operations, but no acceptance criteria, workflows, or designs have been documented. The epic itself (#1983) has no body.

Despite this sparse specification, the operations are conceptually straightforward:
- **Create** triggers the `$expand` operation to materialize a collection's references into a concrete expansion
- **Set as default** designates which expansion is used when consumers access the collection without specifying one
- **Delete** removes an expansion
- **Update metadata** modifies an expansion's descriptive attributes without rebuilding it
- **Rebuild** re-evaluates all references in the expansion against current source content

The expansion API surface is more developed than the UI specification. The Collection Expansions API (#2272) added a cross-version endpoint (`GET /:ownerType/:owner/collections/expansions/`) that returns all expansions within a collection across all versions, with pagination support. This was deployed to QA/Staging in December 2025.

Related work on the TBv2 Collection Versions/Expansions Tab (#2288) provides insight into the expected functionality. Joe's testing feedback identified several issues including missing hyperlink formatting, missing "Released" icon indicators, incomplete "Create Similar Expansion" parameters, and discrepancies in resolved repo versions between rebuild and new expansion operations (#2288). The feedback also suggested consolidating status buttons into a single "Status: Processing/Ready" indicator.


## 4f. Reference Management

Reference management is one of the most complex authoring areas in TBv3, spanning two distinct interaction patterns: authoring references within a collection context and adding references from outside a collection. This area is actively under development as part of the "[CIEL Implementers] Collection & Reference Creation E2E Workflow" milestone.

### Add References Within Collection

The primary reference authoring interface (#2005) provides the full-featured reference building workflow when a user is already inside a collection. The specification is comprehensive:

**Reference definition options:**
- Select individual concepts or groups from available sources
- **Cascade rules**: Include children, include descendants, specify depth levels for fine-grained control over concept hierarchy inclusion
- **Mapping options**: Include/exclude mappings with filters and advanced configuration
- **Version control**: Lock references to specific concept versions, freeze options to prevent updates

**Preview before finalizing**: A comprehensive preview shows validation and grouping, a consolidated summary table (sortable and searchable), and visual indicators for versioning, cascading, and inclusion/exclusion status.

**UI/UX design requirements** documented in #2005:
- Use small icons/indicators rather than verbose text for options (versioned, cascaded, include/exclude)
- Vertical-friendly layout to display many references at once
- Reference field should have expanded/prominent space
- Support for up to 8 properties (class or extra attributes)
- Reduce cognitive load around cascade options

As of June 2025, Joe confirmed readiness to begin development, referencing mockups and "Puzzle Pieces" components in a Miro board (#2005).

### Add Reference to Collection (Quick Add)

A separate "Quick Add" pathway (#2006) supports adding references from outside the collection context -- from search results, another repository, or concept discovery views. Entry points include searching across organizations/collections and viewing concepts from other collections.

The Quick Add workflow:
1. Select concept(s) individually or select all in a repository
2. Select target collection
3. Quick Add Reference (streamlined, minimal configuration)
4. Optionally configure cascade, transform, and version locking
5. Preview results
6. Submit

This distinction between full authoring (#2005) and quick add (#2006) is a deliberate design decision to support both power users building complex collections and casual users who just need to add a few concepts.

### Reference CTAs

| CTA | Ticket | Status |
|-----|--------|--------|
| Preview | #2007 | Open |
| Delete | #2008 | Open |
| Create expansion ($expand) | #2009 | Open |
| Delete (duplicate) | #2011 | Open |
| Update metadata | #2012 | Open |

Note: #2011 was a duplicate of #2008 and has been closed. #2008 now has a full spec (requirements written April 2026) and implementation underway in oclweb3. See `02_capabilities/manage-references.md` for details.

### Collection References Tab

The Collection References tab implementation (#2273) adds listing, search, and filtering capabilities for references within a collection. This is one of the more technically detailed implementations:

**Filter syntax** (deployed to QA/Staging, January 2026):
- `?versioning=` (all, unversioned, repository, resource)
- `?repo_version=` (specific version)
- `?cascade=` (boolean or specific cascade definition)
- `?definition_type=` (all, intensional, extensional)
- `?inclusion_type=` (all, include, exclude)

The filter design went through a refinement cycle. Sunny's initial implementation used separate `resource_versioned` and `repo_versioned` boolean parameters. Jon consolidated these into a single `versioning` facet and renamed "Transform" to "Definition Type" with clearer labels ("Extensional / Enumerated" and "Intensional / Rule-based") (#2273). Jon also noted that `transform = extensional/intensional` is semantically inaccurate and ideally should be changed in the database.

Sorting was deferred because it would require adding References to Elasticsearch, which is a separate effort (#2273).

### Versioned Reference Deprecation

A significant infrastructure change (#2268) removes all pathways for creating new "resource versioned references" -- the deprecated format that embeds a resource version in the URL (e.g., `/concepts/123/123156/`). This work encompasses:

- Blocking the API from creating new resource-versioned references
- Updating the "transform references to extensional" workflow to follow the input reference model
- Removing UI links that direct users to resource-versioned URLs in both TBv2 and TBv3

The API continues to support evaluation of existing resource-versioned references until full migration. The TermBrowser helps users convert resource-versioned references into valid ones so they never reach the API. Communications include a deprecated response header and a planned email announcing the 90-day retirement timeline (#2268). Deployed to QA/Staging in December 2025; TB redirect handling remains pending. Status: Testing/QA.

### Collection Reference Cleanup

**Clean Up Collection References** (#2282) adds a UI workflow to modernize a collection's HEAD references by removing version specifications and applying cascade parameters. This addresses the common problem of collections accumulating versioned references that block automatic updates.

The workflow:
1. User accesses "Clean Up Collection References" from the Collection HEAD Actions dropdown
2. Configures target (versioned-only or all concept references), cascade preset, and preview/backup options
3. Reviews a detailed preview showing total references, concept/mapping breakdowns, and example transformations
4. System auto-saves current HEAD as a new version, processes references, creates a new expansion
5. User investigates results and commits or discards changes

Safety features include automatic backup, comparison/validation tools, and error handling for large operations (>10,000 references). Related CLI work exists in the `oclcli` tool (`ocl tool recreate_collection_with_versionless_references`), noted by Filipe Lopez (#2282).

### Collection Revert

**Restore/Revert Collection HEAD** (#2284) would allow users to restore a collection's HEAD to match a previous version's state. The specification documents two approaches:
- **Option A**: HEAD changes to match the selected version; all newer versions remain in history
- **Option B**: A new version is created that copies the selected version's state

The expansion handling question is also unresolved: should the restore re-evaluate references against current concept states (fresh expansion) or copy the exact previous expansion (historical fidelity)?

As of March 2026, Jon questioned whether this ticket is still relevant, indicating it may be deprioritized (#2284).


## 4g. Design Specifications

Several design-focused tickets define the visual and interaction patterns that underpin the authoring experience.

### Concept Detail View Header

The concept detail view header (#1620) establishes what information is visible when a user opens a concept: the publishing organization, repository (including version), concept code, concept class, and datatype. A key requirement is showing both the original source of the concept and the repo the user pulled it from (e.g., a collection), enabling dependency tracing (#1620).

**External ID handling** went through extensive design iteration. The team explored:
- A chip-style indicator with tooltip (#1620, paulsonder)
- Click-to-copy with tooltip confirmation ("External ID copied to clipboard")
- Placement options: header area, actions menu, or bottom of detail view

The agreed approach: show a truncated External ID in the header, full ID in tooltip on hover, click to copy to clipboard (#1620). This was approved by all three team members.

Design to-dos still open: incorporate External IDs and Created/Updated Date into the header (#1620).

Status: Backlog.

### Concept Detail View Associations Card

The Associations card (#1621) displays hierarchy and mapping associations from the same source. A first draft (view-only) was deployed to QA in September 2024.

Key features and discussions:

**Retired concept/mapping differentiation** (#1621, #1620): A retired mapping means "this target concept is no longer mapped to my concept," while a retired concept means "this target concept has been retired but may still be a valid mapping." The design must distinguish these states clearly.

**Inverse mappings**: Joe requested support for displaying inverse (backward) mappings -- cases where the current concept is the target of a mapping from another concept (e.g., Q-AND-A inverse maps in CIEL). This is important for seeing the full context of answer concepts and concept set membership (#1621).

**Navigation from associations**: The team decided to support "open in same context" for navigating from an association to its from/to concept, keeping the current view intact and potentially using a breadcrumb or persisted header for navigation back to the original concept (#1621). This was confirmed as ready to proceed in March 2025.

**Testing feedback** from Joe (September 2024) included:
- Three-dot menu on each mapping row with "Open From Concept," "Open To Concept," "Compare Concepts," and "Retire Mapping" options
- Sticky map type headers during scrolling within long lists (fixed)
- Table column spacing adjustments (fixed)
- Tooltip explanations for defined-in-OCL vs. not-defined indicators (fixed)

**Retire Mapping from Associations panel** (#2131): The three-dot context menu on each mapping row in the Associations panel must include a "Retire Mapping" action. When selected, a text field should appear for the user to enter a retirement reason (replicating the behavior of the standalone mapping details page). This applies to both TBv2 and TBv3. The retirement reason is optional but the confirmation step must always be present.

**Scroll and focus preservation after CRUD** (#2131): After creating, editing, or retiring an association (mapping), the panel must preserve the user's scroll position and return focus to the modified row rather than scrolling back to the top of the page. This is particularly impactful for concepts with many associations (e.g., 440+ mappings), where finding a recently edited item after a scroll-to-top is difficult.

**Scrollbar proliferation on concept detail** (#2131): The concept details page currently renders 4+ scrollbars simultaneously for concepts with large association lists (e.g., PIH:7936). Removing the max-height constraint from the Associations panel eliminates this problem, allowing the panel to expand to its natural height and use only the page-level scrollbar. Trade-off: the "Add Mapping" button moves to the bottom of the panel, so it should be duplicated at the top as well.

**Associations panel loading performance** (#2131): Loading 440 associations for a single concept (PIH:7936) took over 15 seconds in TBv2. Query optimization or database indexing is required, and the same constraint must be addressed in v3. Skeleton loading should be shown while associations load; the panel must not block rendering of other concept detail sections.

Status: Testing/QA.

### Detail View: Compare

The compare tool (#1623) applies to both concepts and mappings. Requirements include:
- From concept/mapping details: open the compare tool pre-populated with the current resource
- From concept/mapping history: compare two versions of the same resource
- From search results or repo browsing: compare two selected resources
- (Optional) Allow changing which resource is on the left or right side, especially important for multi-server capabilities

Status: Open, no comments. Assigned to paulsonder with TermBrowser v3 MVP milestone.

### Detail View: Change Request

The change request mechanism (#1622) has an empty body and no comments. It is assigned to paulsonder with no milestone. This suggests it is a placeholder for a future collaborative editing workflow, but no design or requirements work has been initiated.

### Form Designs

Three form design tickets track the UI specifications:

- **Create repo form** (#1615): Empty body, no comments. Testing/QA status (per hierarchy document). Assigned to paulsonder.
- **Create org form** (#1616): Designs updated and shared in Zeplin. Joe asked about switching between avatar vs. background image editing; Paul clarified it depends on whether the user clicks the background or the avatar area -- no switching needed (#1616). Testing/QA status.
- **Add references form** (#1618): Empty body, no comments. Assigned to paulsonder with TermBrowser v3 MVP milestone.


## Cross-Cutting Patterns and Observations

### The CTA Model

TBv3's authoring experience is organized around a systematic CTA (Call-to-Action) model. Each discrete user action (Create, Edit, Retire, Delete, etc.) is tracked as a separate ticket, with consistent naming: `CTA / {Object Type} / {Action}`. This model serves several purposes:
- Each action can be independently designed, implemented, and tested
- Status tracking is granular (some CTAs for a given object may be in Testing/QA while others remain unspecified)
- Entry points for each action are explicitly documented

### Implementation Status Summary

| Area | Testing/QA | Open/Todo | Backlog |
|------|-----------|-----------|---------|
| Repository | Create, Edit, Version, Release (#1978-#1981) | Templates, Fork, Import (#2025-#2029, #2155-#2157) | Copy (#1982) |
| Concept | Edit, Retire (#1991, #1997) | Clone, Add, Create similar, Add mapping, Create mapped, Import (#1992-#1996, #1998) | -- |
| Mapping | Create, Edit, Retire, Delete (#2000-#2001, #2003-#2004) | Create similar (#2002) | -- |
| Organization | Delete (#2021) | All others (#2014-#2020, #2022) | Pin repo (#2018) |
| Expansion | -- | All (#1984-#1988) | -- |
| Reference | Versioned ref deprecation (#2268), References tab (#2273) | All CTAs (#2007-#2012), Cleanup (#2282) | Revert (#2284, relevance questioned) |

### Specification Gaps

Many CTA tickets (particularly for concepts, mappings, and expansions) have empty bodies with no acceptance criteria, user stories, or design documentation. These represent "title-level" specifications where the intended operation is clear but the detailed design work has not been done. The most significant gaps:

- All five expansion CTAs (#1984-#1988): no specification at all
- Concept clone/add/create similar (#1992-#1994): no specification
- Reference CTAs (#2007-#2012): no specification
- Several org CTAs (#2015-#2017, #2020): acceptance criteria exist but design docs are "TBC"

### Key Unresolved Decisions

1. **FHIR status alignment**: Should v3 replace Released/Unreleased with FHIR's draft/active/retired/unknown model? (#1981)
2. **Mapping from/to concepts for external resources**: How to handle mappings where concepts are not in OCL (#1376)
3. **Collection revert approach**: Option A (HEAD changes, versions preserved) vs. Option B (new version created) (#2284) -- and whether to pursue this at all
4. **Expansion rebuild discrepancies**: Different resolved repo versions between rebuild and new expansion operations (#2288)
5. **Inline editing interaction model**: Whether clicking text in Edit Mode makes it directly editable or requires a menu action (#1376)


---

# OCLv3 Knowledge Base: Sections 5-7

## Section 5: Cross-Cutting Concerns

Cross-cutting concerns are capabilities that span multiple functional areas of the TermBrowser. Unlike features confined to a single domain (e.g., concept editing or repository management), these concerns influence the user experience across the entire application and require coordinated design and implementation.

### 5.1 Multi-lingual Support

Multi-lingual support in OCL operates at two distinct levels: the language of the TermBrowser UI itself, and the language of the terminology content being browsed.

**UI Internationalization.** The epic for internationalizing the TermBrowser (#1186) envisions user-selectable UI languages, admin-managed translation uploads, and community-contributed translations. The practical first step -- extracting the English language bundle into an external YAML/JSON file (#1471) -- was originally scoped for v2 but deliberately deferred. As @snyaggarwal noted in July 2024: "this is happening as we build v3. So we can close this out." The implication is that v3 is being built with i18n as a structural assumption rather than a retrofit, and tools like Transifex (used within the OpenMRS community) are under consideration for managing translations.

**User Language Preferences.** The effects of a user's language preference on TermBrowser behavior remain under-documented (#1769). The team acknowledges the need to specify what the Language field on the User Profile actually controls -- whether it filters concept display names, changes UI labels, affects search ranking, or all of the above. This question intersects with search preferences (#1770), where the team has yet to define how language selection interacts with search result ordering.

**Search Preferences (#1770).** The broader search preferences requirement asks: what user-configurable settings persist across search sessions, and what behavioral effects do they produce? Key open questions that need resolution:

- **Scope of effects.** Does a search preference affect result ranking only, result filtering, display language, or all three? For example, a user with a French language preference might expect: (a) French names displayed in results, (b) French-named concepts ranked higher, and/or (c) results filtered to only concepts with French names. These are meaningfully different behaviors and the intended combination has not been specified.

- **Multiple selections.** Can a user select multiple preferred languages (e.g., English and French) or multiple preferred vocabularies (e.g., CIEL and SNOMED CT)? If so, how are conflicts resolved -- by union (show anything matching any preference), by ranked order (prefer the first match), or by some other rule? The interaction model for multi-select preferences has not been designed.

- **Preferred vocabularies.** Beyond language, search preferences likely extend to preferred source vocabularies. A user working primarily with CIEL might want CIEL concepts surfaced before equivalent concepts from other sources. This relates to the Walled Garden / Trusted Sources initiative (#1706), but at an individual user level rather than a platform-level curation. It is unresolved whether vocabulary preferences are part of the user profile, the search UI, or both.

- **Persistence and scope.** Are preferences set once in the user profile and applied globally, or can they be adjusted per-session from the search UI itself? The relationship between the User Profile language field (#1769) and any search-time preference controls has not been defined.

These open questions make #1770 a prerequisite dependency for fully specifying both the user profile form and the advanced search behavior (#1607).

**Content-level language handling** surfaces throughout the system. Advanced search requirements (#1607) explicitly mention cross-language use cases: "English user performing an advanced search to only match French names/descriptions -- e.g. working to localize terminology in their own application." The locale selector itself has bugs in v2 (#1681), where searching for common two-letter codes like "es" (Spanish) fails because exact matches are not prioritized over partial matches.

Multi-lingual support touches search (#1607, #1770), user profiles (#1769), content display (#1186), and import/export flows. It is one of the few cross-cutting concerns that was explicitly called out as a v3 structural requirement rather than a feature to be layered on later.

### 5.2 Import & Export

Import and export in OCL encompasses bulk data ingestion, repository downloads, individual resource downloads, and the emerging NPM/FHIR package ecosystem.

**Bulk Import.** The bulk import UI has accumulated a long list of improvement ideas (#863), including FHIR resource import support and auto-detection of hierarchy. A specific pain point is the import task state polling mechanism (#2040), which currently fires every 5 seconds regardless of whether the previous response has returned -- a straightforward fix to wait 5 seconds after the last completed response. NPM file upload support (#1959) extends bulk import to handle FHIR Implementation Guide packages, linking to the broader FHIR packaging work (#1917). The new Importer UI for v3 (#2161) is milestoned to the OCL Mapper MVP and is currently in QA testing.

**Repository Download and Export.** The Repository Download + Export epic (#1873) is milestoned for the TBv3 MVP and represents a redesign of how users obtain repository content. A long-standing request (#900) asks for the "retired" concept filter to be exposed in the export UI, a feature already supported by the API since issue #22. Individual resource downloads are being designed for concepts (#1879) and mappings (#1880), with formats including OCL JSON and FHIR representations. The ability to download or view content as FHIR (#591, #535) traces back to early 2021 and is being subsumed into v3's more comprehensive download architecture.

**Collection Reference Results.** When users add references to a collection, the results can be lengthy and difficult to parse. Two companion issues address this: the ability to copy/download add-reference results (#1537) and the ability to resolve references in the results page to show human-readable concept names and IDs (#1538). Both originated from the Kenya OpenMRS Hackathon and were tagged as v3 requirements. The broader challenge of making the add-references results panel useful (#1202) and resolving reference display text (#1375) feeds into the same problem space.

**Import from Existing Repositories.** A newer v3-era requirement (#2157) enables creating a repository by importing content from an existing one, complementing the fork-a-repo-version feature (#2156). Both are milestoned for the TBv3 MVP and represent a shift toward repository-level content reuse rather than concept-by-concept reference building.

### 5.3 Performance

Performance concerns in v3 center on perceived responsiveness and the scalability of complex operations.

**Loading Indicators and Benchmarks.** The "OCL Bouncy Ball" loading icon (#1741) is both a design deliverable (a branded loading animation) and a performance benchmarking initiative. The ticket calls for establishing measurable performance benchmarks alongside the visual indicator, acknowledging that users need clear feedback when operations take time.

**Cascade Performance.** The cascade preview visualizer (#1455) is designed as a reusable component that must handle potentially large hierarchical datasets across multiple view modes (hierarchical table, flattened table, visual, raw JSON). Performance is an implicit constraint here -- cascade operations can traverse deep hierarchies and return hundreds of entries, as demonstrated by the MSF-OCB collection example in #1763 where a single concept cascade returned 14 entries across multiple sources.

**Concept Details Loading.** The v2 hot fixes ticket (#2131) documents specific performance issues: loading 440 associations for a single concept (PIH:7936) takes over 15 seconds. The ticket proposes query optimization or indexing as solutions, and notes that this behavior also needs to be addressed in v3.

**API Throttling.** OCL API throttling (#2121) is in QA testing and represents an infrastructure-level performance concern -- protecting the API from overload while maintaining responsive UI interactions. This is particularly relevant for operations like bulk import and cascade evaluation that generate heavy API traffic.

### 5.4 Search Behavior

Search is the primary entry point for most OCL interactions and has undergone significant evolution from v2 to v3.

**Sort Order.** Sort order issues (#1975) affect how results are ranked and displayed across the application. The companion issue on general sorting behavior (#1939) covers concepts, mappings, and repositories. The question of whether global search should navigate to the latest released version rather than HEAD (#1752) remains tagged as "discussion-needed" -- it reflects a fundamental tension between consumers (who want stable released content) and authors (who need to see their latest work).

**Source Properties and Filters.** A major v3 search enhancement (#2174) migrates concept_class and datatype to become source-defined "properties" and exposes them as search filters. The implementation distinguishes between properties (schema definitions), filters (facet controls), and summary display configuration. As @snyaggarwal clarified: "Properties is just schema. Filters control facets. Include_in_summary only controls the default summary display." This allows source administrators to define which properties appear in concept summaries -- for example, CIEL might surface class and datatype, while LOINC surfaces its six component parts (Component, Property, Time, System, Scale, Method).

**Search Result Presentation.** The removal of alternating table background colors (#2120) is a small but telling change -- the v3 search results are being redesigned from scratch rather than inheriting v2 patterns. The search results icon for mapped concepts (#1974) adds visual indicators showing when a concept has mappings, helping users identify relationship-rich concepts at a glance.

**Search Result Highlighting and Fuzzy Match Formatting** (#1937): Users have reported confusion when exact matches and fuzzy matches look identical in search results. The current ElasticSearch-based highlighting bolds entire words even when only a partial match occurs -- for example, "Warburg" is fully highlighted when the query only matched "arburg." The requirement for v3 is to highlight only the actually-matching characters within a word, not the entire word. This distinction helps users quickly assess match quality and understand why a result was returned.

**Advanced Search.** The advanced search MVP (#1607) remains one of the most ambitious search requirements. Compiled requirements include: restricting search to specific fields, searching custom attributes (with dynamic population of attribute keys), cross-language name matching, and finding concepts based on mapping characteristics. Andy Kanter's request -- "I'd love to see where there is not SAME-AS mapping to reference terms that we have a SAME-AS map to CIEL" -- captures the kind of relationship-aware search that distinguishes OCL from simpler terminology browsers.

### 5.5 Sharing & Collaboration

Sharing and collaboration features in v3 aim to make OCL content easily shareable both within and outside the platform.

**Deep Linking.** The deep linking requirement (#1522) is foundational: URLs should update to reflect the current view, copy-pasting a URL should reproduce the same view, and right-click-to-copy should work on links. The ticket explicitly notes that "there are places now where this doesn't work" and that selecting text sometimes triggers actions instead of allowing copy. Deep linking requirements were folded into the concept detail header design (#1620).

**Resource Sharing.** Individual share actions are being designed for concepts (#1865), mappings (#1878), repositories (#1916), and organizations (#2022). The concept share feature (#1865) is milestoned for the TBv3 MVP and is currently in the backlog. The pattern is consistent: each resource type gets a Share action that generates a URL or enables copying a reference in various formats.

**Following and Activity.** The "Following" feature (#1780) allows users to follow organizations, users, or repositories. This was closed as complete for its initial scope and is tied to the dashboard activity feeds (see Section 5.8). The design discussion around #1780 considered what actions should generate follow notifications and how to prevent information overload.

**Walled Garden.** The Walled Garden and Trusted Sources initiative (#1706) aims to surface content most relevant to each user, prioritizing reference content like CIEL and LOINC over the "noise" of all other user-generated content. The concept is related to canonical URL work and represents a content curation strategy that affects search ranking, dashboard content, and repository discoverability.

### 5.6 UI Patterns & Design System

Across the v3 issue corpus, several recurring UI patterns and design system components emerge that standardize interactions across functional areas.

**Chip Components.** The Objects (Chip view) pattern (#1776) provides a compact visual representation of OCL resources -- repositories, concepts, mappings -- used in search results, canonical URL resolution displays, and anywhere a resource needs to be referenced inline. The chip was designed iteratively during the canonical URL registry work (#1742) and is now a reusable design system element.

**Split View / Detail Panel.** The split view concept (#534) dates back to January 2021 and envisions a sidebar/detail layout similar to the ICD-10 browser. While the original ticket is classified as pre-v3, the pattern has been realized in v3's concept detail view alongside search results, as specified in the concept detail header (#1620), associations card (#1621), and names/descriptions panel (#1734).

**Custom View Configuration.** A family of tickets addresses how content views are customized: filter configuration (#530, #788), resource details page customization (#536), and custom view configuration broadly (#808). In v2, this was implemented through an admin configuration system that controlled which attributes appeared in headers (#1355), with built-in logic for default visible attributes and collapsible overflow. V3 inherits this pattern but redesigns it through the source properties/filters framework (#2174).

**Form Patterns.** Form design tickets for creating repos (#1615), orgs (#1616), and adding references (#1618) are all milestoned for the TBv3 MVP and follow a consistent design language. The tension between UI strictness and API flexibility (#958) -- where the UI intentionally restricts options to encourage best practices while the API allows power users more freedom -- was explicitly discussed. As @jamlung-ri noted: "the UI is more restrictive than the API, and I feel like we do that to encourage best practices in the UI for newer users."

**Extras Display.** The treatment of JSON "extras" attributes (#2402) is an active design challenge. The current raw JSON display is inadequate for structured data like CIEL's advanced description attributes. The proposal introduces a label/display/related_description_external_id schema that enables elegant rendering while maintaining data fidelity.

**Version Navigation.** The Version-to-HEAD mode transition (#1830) addresses a fundamental UX concern: how does a user know they are viewing a read-only published version versus the editable HEAD? The ticket calls for a scenario table mapping content types, user roles, and expected display behavior.

### 5.7 Hierarchy & Visualization

Hierarchy and visualization represent one of the most technically demanding cross-cutting concerns, touching search results, concept details, cascade operations, and collection management.

**Cascade as the Hierarchy Engine.** The pivot from a dedicated hierarchy view to cascade-powered hierarchy (#1592) is a key architectural decision. Cascade is "way more powerful than using hierarchy directly and allows for a lot of user customization." The target behavior loads children dynamically on node expansion, defaults to repo-level settings, and lets users switch between rendering hierarchy versus specific mapping types. The component is envisioned as reusable in both search results and the associations panel.

**Cascade Preview Visualizer.** The cascade preview component (#1455) supports four view modes -- hierarchical table, flattened table, visual, and raw JSON. It is a prerequisite for the Add References dialog with preview capabilities and is intended for deployment across multiple surfaces: the operations panel, the associations panel in concept details, and collection management workflows.

**Hierarchy in Mapping Workflows.** Hierarchy as part of Add/Edit Mappings (#1822) brings cascade visualization directly into the mapping creation flow, helping users understand the structural context when establishing relationships between concepts. This is milestoned for the TBv3 MVP.

**Hierarchy/Cascade Component.** The dedicated design ticket (#1700) for "Seeing context for a concept" integrates hierarchy into the search and browse experience. It sits under the Search UI epic (#1694) and is paired with the Searchlight component, indicating that hierarchy context is considered part of the search experience rather than a standalone feature.

### 5.8 Notifications & Activity Feeds

The notification and activity system in v3 is built around the dashboard and the following/subscription model.

**Dashboard Feeds.** The signed-in user dashboard includes a "Your recently updated repos" feed (#1708) that shows repositories the user owns or has recently interacted with. This was closed as complete. A companion feed for "Updated Reference Content" (#1771) notifies users when reference sources like CIEL or LOINC have been updated -- this directly supports the long-standing need to know when upstream content has changed (see #12, #13).

**Following Model.** The Following feature (#1780) enables users to subscribe to updates from organizations, users, or repositories. This feeds into the dashboard activity display and was designed as part of the Consumer: Repository epic (#1625). The feature is closed, indicating the initial implementation is complete.

**Activity as Discovery.** The dashboard feeds serve a dual purpose: notification of changes to content the user cares about, and discovery of new relevant content. The anonymous user dashboard (#1861) and authenticated user dashboard (#1862) present different activity views -- anonymous users see community-wide activity, while authenticated users see personalized feeds based on their followed entities and owned content.

---

## Section 6: Legacy & Migration Notes

The TBv3 project inherits a long history of feature development, design iteration, and architectural decisions from v2 and earlier. This section traces how that legacy shapes v3 by examining issues classified as "TBv2," "Not Clear," or marked for closure as outdated or stale.

### 6.1 The MSP Era: Domain-Specific Beginnings (Issues #667-#672)

Three issues from March 2021 (#667, #669, #672) concern the PEPFAR Metadata Sharing Platform (MSP), an OCL-powered application for browsing health indicator metadata. These tickets -- showing fiscal year data for reference indicators, fixing denominator display, and managing fiscal year filters -- represent a period when the TermBrowser was being extended to serve very specific domain use cases. None have any code activity or v3 board presence. Their existence in the issue tracker reflects the historical reality that ocl_issues serves as the issue repository for multiple OCL-adjacent applications, not just the core TermBrowser. They are artifacts of a time when the TermBrowser's scope was less clearly bounded.

### 6.2 The Configuration Ambition (2021): Customization as Strategy

A cluster of issues from early-to-mid 2021 reveals an ambitious vision for TermBrowser customization that influenced but did not directly carry into v3:

- **Filter Configuration** (#530) envisioned admin-configurable filters using Material UI Select components, with hardcoded options as a stepping stone to dynamic facets. The work was deprioritized repeatedly.
- **Split View** (#534) proposed a sidebar/detail layout modeled on the ICD-10 browser. Design artifacts were created in Miro and Zeplin but the concept was never built as described; instead, v3 adopted its own interpretation of detail panels.
- **Resource Download Formats** (#535) and **Resource Details Customization** (#536) proposed flexible, user-driven control over what information is displayed and how content is exported. These aspirations survive in v3's source properties/filters work (#2174) and download designs (#1873, #1879, #1880).
- **Custom View Configuration** (#808) and **Configuration Sandbox / Custom Domain** (#913) pushed further toward a vision of white-labeled, organization-branded TermBrowser instances. The custom domain concept has not resurfaced in v3 planning.

The pattern that emerges is one of narrowing scope: v2 aspired to be a configurable platform; v3 is focusing on being a well-designed product with strategic extension points.

### 6.3 The Connector Architecture (2022): Abstraction That Did Not Persist

Issue #1185 proposed abstracting TermBrowser connectors into independent objects, supporting OCL, HAPI FHIR, OCL FHIR, ICD-11 FHIR, OpenMRS, and DHIS2 as interchangeable backends. Separately, #1003 envisioned a standalone TermBrowser running without oclapi2, integrated directly with SVCM (Sharing ValueSets, Codes, and Maps) servers. Neither concept survived into v3. The v3 architecture assumes OCL as the backend, with FHIR served through OCL's own FHIR endpoints rather than through pluggable connectors.

### 6.4 Concept Modifications and Expansions: An Evolving Architecture

The concept modifications epic (#816) from June 2021 laid out an ambitious architectural roadmap: separating collection definitions from expansions, implementing custom expansion parameters, supporting multiple reference types with canonical URLs, and handling references to resources not defined in OCL. The companion TermBrowser ticket (#909) raised questions that remain relevant: releasing an expansion, retiring an expansion, exporting an expansion.

This architectural vision has largely been realized in v3's reference/expansion model (#1854, #1855), but through a different implementation path. The v2-era issues (#816, #909) are classified as "Not Clear" or "close-superseded" because their specific scope has been absorbed by v3's Consumer: Reference and Consumer: Expansion epics.

### 6.5 The Collection Update Problem: Seven Years and Counting

Issues #12 and #13, created in February 2018, describe the ability to check for and apply updates to collection references. The problem is deceptively simple: when a concept in a source is updated, collections referencing that concept should be able to detect and adopt the change. Andy Kanter described the scope in August 2019:

> "The update to latest released concept version is an incredibly important feature... There should be a way to filter the concept view to only those concepts needing an update. A full report of changes with the ability to select which updates to apply is probably best practice."

The discussion (#12 comments) revealed that `is_latest_concept_version` tracks HEAD changes but not released version changes, meaning users cannot distinguish between an unreleased edit and a formally released update. @karuhanga identified the gap: "We need a stronger constraint of a source version having been released before we consider a concept as having been updated."

These issues remain open. The dashboard feed for updated reference content (#1771) addresses the notification aspect, and the expansion rebuild mechanism (#1988) provides the action, but the comprehensive review-and-accept-changes workflow Kanter described has not been built. This remains one of the oldest unresolved design problems in OCL.

### 6.6 Mapping Display: Repeated Redesign

The mapping display has been redesigned across multiple issue generations:
- **OCL TermBrowser Mapping Revamp** (#793, June 2021) -- closed as outdated
- **Concept Modifications tracing** (#1763, February 2024) -- where the team discovered that collection-context concept details were pulling source-context mappings instead of collection-modified mappings
- **Mapping Listing View** (#1758) -- the v3 implementation, milestoned for MVP

Each iteration surfaced the same fundamental tension: mappings in OCL are independent resources that exist in sources, but users often think of them as attributes of concepts. The v3 mapping listing view (#1758) and the concept detail associations card (#1621) represent the latest attempt to present mappings in a way that serves both mental models.

### 6.7 What v2 Issues Tell Us About v3 Priorities

Analyzing the disposition of pre-v3 issues reveals the project's evolution:

**Closed as Outdated (26 issues):** Issues like #233 (the original "TermBrowser" epic), #530 (filter config), #536 (details customization), #808 (custom views), #882 (hierarchy UI), and #985 (async counts) represent v2-era specifications that have been superseded by v3's ground-up redesign. Their concepts often persist in v3 but under different framing.

**Closed as Stale (15 issues):** Issues like #535 (download formats), #591 (FHIR download), #865 (FHIR UI), #888 (resource comparison), #913 (custom domains), #1003 (SVCM standalone), and #1297 ($resolveReference in operations panel) represent features that were desired but never gained enough momentum. Some have v3 successors; others were quietly abandoned.

**Marked for Review (9 issues):** Issues like #534 (split view), #816 (concept modifications), #863 (bulk import improvements), #962 (validation schema errors), #1113 (favorites), #1281 (collection management), and #1361 (quick add mappings in collection) represent ideas that still have potential relevance but need re-evaluation in the v3 context.

---

## Section 7: Open Questions & Unresolved Discussions

### 7.1 The Permissions Model

The Roles and Permissions epic (#1812) is tagged as a "v3-foundation" item but has no milestone and sits in "No Status" on the project board. It has code activity in oclapi2, suggesting backend work is underway, but the TermBrowser-side design remains undefined.

The permissions question surfaces repeatedly across the issue corpus:
- **Registry permissions** (#1744): What permissions govern who can view, create, and modify canonical URL registry entries? How should `$resolveReference` behave for registries the user cannot access?
- **User profile visibility** (#1852): Who can view user profiles? The issue was closed after discussion, but the broader permissions framework is unresolved.
- **Repository release permissions** (#1981): Tagged "discussion-needed" -- who can release or un-release a version?
- **Private content in search** (#1887): V3 search did not return private content for non-admin users, exposing a gap in how permissions are applied to search indexing.

The absence of a defined permissions model is the single largest unresolved architectural question in v3.

### 7.2 The Canonical URL Registry: Resolved Design, Open Governance

The canonical URL registry was one of v3's most thoroughly designed features, with detailed mockups (#1742), implementation stages defined by @paynejd (Stage 1: staff manage global; Stage 2: users manage own namespaces; Stage 3: canonicals integrated throughout app), and a functioning QA deployment.

Unresolved questions remain around governance:
- **How does the registry behave if a canonical URL changed between repo versions?** (#1742 comment by @paynejd)
- **Canonical URL restrictions** (#1371): What characters and patterns are valid? The underscore issue (#1874) is one specific case where FHIR conventions conflict with existing OCL content.
- **Global registry population** (#1743): The task of populating the global registry across all environments is milestoned for MVP but assigned to three people (bmamlin, rkorytkowski, snyaggarwal), suggesting it requires cross-team coordination and possibly community input.

### 7.3 Collection Reference Interpretation

The Display/Interpretation of Collection References epic (#1374) is tagged "discussion-needed" and captures a fundamental UX problem: how do you convert a technical reference expression into human-readable language? As @jamlung-ri described: "I wanted to search for a specific code in a repository (collection) and diagnose what references brought the concept into this collection so that I can target my collection editing process."

Related unresolved questions:
- How should the "Remove Reference" prompt communicate the difference between removing a reference (which may affect many concepts) and excluding a specific concept (#1373)?
- Should reference display text include concept names in addition to IDs (#1375)?
- How should the TermBrowser handle collection-based references versus source-based references when adding concepts (#1772)?
- How should the consumer model for references (#1854) handle the fact that OCL's reference model is "not entirely FHIR conformant," as @jamlung-ri noted: "It may not necessarily be FHIR-illegal, but it at least doesn't go along with most of the value set building norms that FHIR would do."

### 7.4 Concept Comparison Approaches

The resource comparison capability has been an open question since #888 (August 2021). The v3 approach includes:
- **Detail view / Compare** (#1623): Milestoned for MVP, in "No Status"
- **Repo / Compare Versions** (#1853): In Testing/QA with oclweb3 commits
- **Repo Comparison MVP Gaps** (#2181): Acknowledges that mapping comparison and JSON comparison are missing from the initial implementation

The question of what constitutes a meaningful "diff" between terminology resources -- where changes to names, descriptions, mappings, and extras all carry different semantic weight -- remains a design problem without a fully articulated solution.

### 7.5 The Extras Schema Problem

Issue #2402 (tagged "discussion-needed") exposes a structural tension in OCL's data model. The `extras` field on concepts is a generic JSON key-value store, but real-world usage demands structured, linked, locale-aware data. The CIEL use case requires multiple "advanced description parts" that share an external_id and must be reassembled into OpenMRS-compatible descriptions.

The proposed solution introduces label/display/related_description_external_id patterns within extras, raising the question: at what point does extras become a schema-driven data structure rather than a free-form extension point? This decision has implications for the concept detail panel display, import/export formats, and API behavior.

### 7.6 Search Defaults: HEAD vs. Latest Released

Whether global search should navigate to the latest released version or HEAD (#1752) is tagged "discussion-needed" and milestoned for MVP. The tension:
- **Consumers** want to see stable, published content. Navigating to HEAD exposes them to in-progress work.
- **Authors** want to find their own content, which may only exist on HEAD.
- **The current behavior** (navigating to HEAD) can confuse consumers who don't understand the HEAD/version distinction.

This question connects to the Version-to-HEAD mode transition design (#1830) and affects how search results are presented, how deep links resolve, and how the dashboard surfaces content.

### 7.7 Deferred but Not Rejected

Several significant feature requests have been deferred without being formally rejected:

- **Concept Cloning** (#1444): The ability to copy a concept from one source to another, maintaining a "cloned-from" link. Requirements were expanded to include cascading (cloning a concept's mappings and answer sets). The $cascade operator was enhanced with `omitIfExistsIn` and `equivalencyMapType` parameters as prerequisites, but the full $clone operation and UI remain unbuilt.

- **Change Requests** (#1614): Discovered during v3 information architecture work as "an Object to represent and manage changes to Concepts and other content in OCL." A deep dive was proposed but the design was never completed. The concept of formal change request management -- distinct from version history -- represents a governance layer that could transform OCL from a terminology editing tool into a terminology governance platform.

- **Operations Panel** (#1624): The ability to execute FHIR operations ($translate, $lookup, $validate-code) and OCL operations ($cascade, $checksum) from within the TermBrowser. Requirements are well-defined but the feature has not been built for v3.

- **Walled Garden / Trusted Sources** (#1706): The vision of curating what content users see by default, prioritizing reference content over community noise. The ticket notes "It is unclear if this will be part of V3 quite yet or to what extent."

- **Concept Modification Tracking** (#1763, #1318): The ability to see when a concept in a collection has been modified from its source version, and to highlight those modifications visually. This is a prerequisite for trustworthy collection management but requires resolving the source-vs-collection context ambiguity that persists from v2.

- **Object-focused Summary Metrics** (#1857): Standardized metrics for organizations (repo count, member count, followers), users (repo count, org count, contribution history), and repositories (concept count, mapping count, version count). The "official/verified" flag for repositories was noted as not easily implementable.

### 7.8 Recurring Themes in Unresolved Discussions

Across the "discussion-needed" issues, several themes recur:

1. **Context ambiguity**: When viewing a concept, is the context the source or the collection? Different answers yield different mappings, different modification indicators, and different cascade results. This is the single most frequently revisited design problem.

2. **Power users vs. new users**: The UI/API restriction gap (#958) reflects a deliberate choice to guide new users while allowing API power users more flexibility. V3 must decide where this boundary lives for each feature.

3. **FHIR conformance vs. OCL flexibility**: OCL's data model predates and sometimes diverges from FHIR conventions. Issues like underscore warnings (#1874), reference model non-conformance (#1854), and the canonical URL restrictions (#1371) all navigate this tension.

4. **Notification without overwhelm**: The dashboard feed and following model need to communicate meaningful changes without becoming noisy. The seven-year-old collection update problem (#12, #13) shows how hard it is to determine what constitutes a meaningful change worth notifying about.


---

## Section 8: Resource Library

This section catalogs all external resources referenced in v3 issues — design documents, specifications, API references, and application links.


### Google Document (9 resources)

- **#667** — [MSP: Show years in which a reference indicator is active when viewing a referenc](https://docs.google.com/document/d/1-KruJnWfqnDJp3UfktVfxey1LloW2TW9OTZaJSXjkaM/edit#)
- **#534** — [New View: Split View](https://docs.google.com/document/d/135GycK_fkNna0NHw85DKf0hkLjk-UU5a5BT1Gdh47xg/edit#heading=h.b71eyyyedpl1)
  - Context: Initial requirements will be compiled here: https://docs.google.com/document/d/135GycKfkNna0NHw85DKf0hkLjk-UU5a5BT1Gdh47xg/edit#heading=h.b71eyyyedpl1
- **#1763** — [Concept tracing information - Modification indicators](https://docs.google.com/document/d/157ChmfmYLGLEg_fO4cE7oMqtGpL4PchUkEhNfUUMxdo/edit#bookmark=id.pbgpc2ccacvm)
  - Context: Complexities from this OCL Arch Call(https://docs.google.com/document/d/157ChmfmYLGLEgfO4cE7oMqtGpL4PchUkEhNfUUMxdo/edit#bookmark=id.pbgpc2ccacvm):
- **#2121** — [OCL API Throttling](https://docs.google.com/document/d/1LonHS-89tEPHBH0ajcSzmOHl9mk0TIqFPJe-wGXdPro/edit?tab=t.0#heading=h.vg1f9unmeq86)
  - Context: To be applied on all APIs. More(https://docs.google.com/document/d/1LonHS-89tEPHBH0ajcSzmOHl9mk0TIqFPJe-wGXdPro/edit?tab=t.0#heading=h.vg1f9unmeq86)
- **#1854** — [Consumer: Reference](https://docs.google.com/document/d/1QvWX0H6TP1_yJSiNkE9xWIrYmn9opp6O0-qQp57m4zc/edit)
- **#1812** — [Roles and Permissions for OCLv3](https://docs.google.com/document/d/1qJ3331fwKs4TYRN5_YIInO--R2SA_3Oy21ol9ArnKVs/edit)
  - Context: https://docs.google.com/document/d/1qJ3331fwKs4TYRN5YIInO--R2SA3Oy21ol9ArnKVs/edit
- **#816** — [Concept Modifications](https://docs.google.com/document/d/1uhnnKxB_X_XLGeO7KSEN1obdL-UpkD82BraYE_ZpTzY/edit)
  - Context: Implementing requirements that are laid out here: https://docs.google.com/document/d/1uhnnKxBXXLGeO7KSEN1obdL-UpkD82BraYEZpTzY/edit
- **#1696** — [Canonical URL Registry](https://docs.google.com/document/d/1zrQOQBC868uRdrZA-mhzvNzsepVYFhHrMDuJU7mKRjY/edit#heading=h.pvobb1bu30xm)
  - Context: https://docs.google.com/document/d/1zrQOQBC868uRdrZA-mhzvNzsepVYFhHrMDuJU7mKRjY/edit#heading=h.pvobb1bu30xm
- **#1706** — [Walled Garden and Trusted Sources](https://docs.google.com/document/d/1zrQOQBC868uRdrZA-mhzvNzsepVYFhHrMDuJU7mKRjY/edit#heading=h.wl2btr79egq1)
  - Context: Existing documentation here: https://docs.google.com/document/d/1zrQOQBC868uRdrZA-mhzvNzsepVYFhHrMDuJU7mKRjY/edit#heading=h.wl2btr79egq1

### Google Presentation (7 resources)

- **#536** — [Resource Details Page Customization Options](https://docs.google.com/presentation/d/1GLwB65ybluTxQrqUNSATIXmSrBKl_rIPL4_eNBYw9a0/edit?usp=sharing)
  - Context: Requirements and ideas for this are being compiled here: https://docs.google.com/presentation/d/1GLwB65ybluTxQrqUNSATIXmSrBKlrIPL4eNBYw9a0/edit?usp=sh
- **#1696** — [Canonical URL Registry](https://docs.google.com/presentation/d/1j49g6zNfwMag4gmoxEGfI0HG-6tQ4zEZpgwHooiUK4w/edit)
  - Context: Initial Requirements and Design: https://docs.google.com/presentation/d/1j49g6zNfwMag4gmoxEGfI0HG-6tQ4zEZpgwHooiUK4w/edit
- **#1888** — [User - View canonical registry](https://docs.google.com/presentation/d/1j49g6zNfwMag4gmoxEGfI0HG-6tQ4zEZpgwHooiUK4w/edit#slide=id.g2be25b8701d_0_16)
- **#1742** — [Canonical URL registries in the TermBrowser](https://docs.google.com/presentation/d/1j49g6zNfwMag4gmoxEGfI0HG-6tQ4zEZpgwHooiUK4w/edit#slide=id.g2be25b8701d_1_34)
  - Context: https://docs.google.com/presentation/d/1j49g6zNfwMag4gmoxEGfI0HG-6tQ4zEZpgwHooiUK4w/edit#slide=id.g2be25b8701d134
- **#1079** — [Add filter button for Extra Attributes in the Associations Section of the Concep](https://docs.google.com/presentation/d/1nkGq1Zx_SDJa-8NblyaHIr5Ot33xKWem-vyTtLjQu1k/edit#slide=id.gfd536d8015_0_0)
  - Context: See mockup below or in this slide deck(https://docs.google.com/presentation/d/1nkGq1ZxSDJa-8NblyaHIr5Ot33xKWem-vyTtLjQu1k/edit#slide=id.gfd536d801500)
- **#530** — [Filter Configuration](https://docs.google.com/presentation/d/1xdZvvgWAB8yem4xLWC5kMOouQHeFE_c3F4cR2uQBDXo/edit#slide=id.gb4fd7ce096_0_14)
  - Context: Further mockups and ideas can be found in this slide deck: https://docs.google.com/presentation/d/1xdZvvgWAB8yem4xLWC5kMOouQHeFEc3F4cR2uQBDXo/edit#sli
- **#2288** — [Quick update to TBv2 Collection Versions/Expansions Tab](https://docs.google.com/presentation/d/1xjC5OejDI2KGceka_IznkOUAhCsDKcRS1isIhIYmGHU/edit?usp=sharing)
  - Context: https://docs.google.com/presentation/d/1xjC5OejDI2KGcekaIznkOUAhCsDKcRS1isIhIYmGHU/edit?usp=sharing

### Google Spreadsheet (3 resources)

- **#1687** — [Implement repo_type attribute for sources and collections](https://docs.google.com/spreadsheets/d/1IHvFEhugjq3S39NweE-ej-GVgcjjREyH5_uQTvp7iDM/edit#gid=0)
  - Context: https://docs.google.com/spreadsheets/d/1IHvFEhugjq3S39NweE-ej-GVgcjjREyH5uQTvp7iDM/edit#gid=0
- **#1861, #1862, #1627, #1688, #1780** — [Global Dashboard for Anonymous User](https://docs.google.com/spreadsheets/d/1idF7p-0lYizHTPhYgSoxigM9F9MJpzOUeq-TbR5-abI/edit)
  - Context: Recent Highlights feed is documented here: https://docs.google.com/spreadsheets/d/1idF7p-0lYizHTPhYgSoxigM9F9MJpzOUeq-TbR5-abI/edit
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://docs.google.com/spreadsheets/d/1idF7p-0lYizHTPhYgSoxigM9F9MJpzOUeq-TbR5-abI/edit?gid=0)
  - Context: -   Sunny to review additional changes in the Events and Feeds spreadsheet(https://docs.google.com/spreadsheets/d/1idF7p-0lYizHTPhYgSoxigM9F9MJpzOUeq-

### Google Drawing (1 resources)

- **#1999** — [Author: Mapping](https://docs.google.com/drawings/d/1YW7_O7X8F7NG5WfPpTqdALrLmXQ6vA7_ZATgD10uKCY/edit)
  - Context: Very Crude Mockup (link if anyone needs to edit(https://docs.google.com/drawings/d/1YW7O7X8F7NG5WfPpTqdALrLmXQ6vA7ZATgD10uKCY/edit)):

### FHIR/HL7 (5 resources)

- **#1981** — [CTA / Repository / Release (or Un-release) a version](http://hl7.org/fhir/R4B/codesystem-definitions.html#CodeSystem.status)
- **#1874** — [In TB, warn users that underscore not officially supported by FHIR in canonical ](https://build.fhir.org/datatypes.html#uri)
- **#1865** — [Concept / Share](https://build.fhir.org/terminologies.html)
- **#1625** — [Consumer: Repository](https://www.hl7.org/fhir/codesystem.html#search)
- **#1854** — [Consumer: Reference](https://www.hl7.org/fhir/valueset.html)

### OCL App (58 resources)

- **#1724** — [Community website link](http://www.openconceptlab.org.)
- **#1854** — [Consumer: Reference](https://api.openconceptlab.org/users/jamlung/collections/loinc-scale-type/1/references/7088422/)
- **#1318** — [Highlight concept attributes/associations that were modified in the collection](https://api.qa.openconceptlab.org/orgs/OpenmrsTest/collections/test1/concepts/163361/106081/)
- **#667** — [MSP: Show years in which a reference indicator is active when viewing a referenc](https://api.staging.openconceptlab.org/orgs/PEPFAR-Test8/collections/MER_REFERENCE_INDICATORS_FY18/concepts/?verbose=true)
- **#667** — [MSP: Show years in which a reference indicator is active when viewing a referenc](https://api.staging.openconceptlab.org/orgs/PEPFAR-Test8/sources/MER/concepts/HTS_TST/5eea8e00debb3d007da648aa/)
- **#667** — [MSP: Show years in which a reference indicator is active when viewing a referenc](https://api.staging.openconceptlab.org/orgs/PEPFAR-Test8/sources/MER/concepts/HTS_TST/versions/?verbose=true)
- **#2019** — [CTA / Organization / Customize appearance](https://app.openconceptlab.org/#/orgs/CIEL/)
- **#1621** — [Concept Detail view / Associations card](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/1499/)
- **#1734** — [Concept Detail View / Names + Descriptions](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/160330/)
- **#1734** — [Concept Detail View / Names + Descriptions](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/162311/)
- **#1621** — [Concept Detail view / Associations card](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/162478/)
- **#1738** — [Concept detail view / Collection membership](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/168682/)
- **#1621** — [Concept Detail view / Associations card](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL/concepts/5090/)
- **#1763** — [Concept tracing information - Modification indicators](https://app.openconceptlab.org/#/orgs/MSF-OCB/collections/Trauma/HEAD/expansions/autoexpand-HEAD/concepts/1290/)
- **#1763** — [Concept tracing information - Modification indicators](https://app.openconceptlab.org/#/orgs/MSF/sources/MSF/concepts/1290/)
- **#2131** — [TBv2 concept details & associations hot fixes](https://app.openconceptlab.org/#/orgs/PIH/sources/PIH/concepts/7936/)
- **#1892** — [Repository - About](https://app.openconceptlab.org/#/orgs/SNOMED-International/sources/SNOMED-GPS/)
- **#1621, #1822** — [Concept Detail view / Associations card](https://app.openconceptlab.org/#/orgs/WHO/sources/ICD-10-WHO/concepts/Z98/)
- **#2288** — [Quick update to TBv2 Collection Versions/Expansions Tab](https://app.openconceptlab.org/#/users/jamlung/collections/demo/versions)
- **#1684** — ["Add Expression" workflow appears to make invalid references?](https://app.openconceptlab.org/#/users/jamlung/collections/my-labs/references)
- **#1478** — [Repository version summary](https://app.qa.openconceptlab.org/#/orgs/CIEL/sources/CIEL/summary)
- **#2019** — [CTA / Organization / Customize appearance](https://app.qa.openconceptlab.org/#/orgs/PEPFAR-Test10b/)
- **#1799** — [Org (Detail view) - round 2](https://app.qa.openconceptlab.org/#/orgs/PEPFAR/sources/DATIM-MOH-DAA-FY23/)
- **#2019** — [CTA / Organization / Customize appearance](https://app.qa.openconceptlab.org/#/orgs/WHO-DDCC/)
- **#1887** — [v3 Search does not return private content](https://app.qa.openconceptlab.org/#/orgs/private-org/.)
- **#1887** — [v3 Search does not return private content](https://app.qa.openconceptlab.org/#/orgs/private-org/sources/private-source/)
- **#1887** — [v3 Search does not return private content](https://app.qa.openconceptlab.org/#/search/?q=private&isTable=true&isList=false&page=1&type=sources)
- **#1755** — [My repositories](https://app.qa.openconceptlab.org/#/users/jamlung/collections/)
- **#1322** — [Revise display string representation of references](https://app.qa.openconceptlab.org/#/users/ocladmin/collections/joe-test-02Sep2022/references)
- **#1355** — [Repo Header showing unnecessary information](https://app.qa.openconceptlab.org/#/users/ocladmin/sources/asdd/)
- **#1673** — [Datatype Dropdown bug - cannot use one of my common datatypes in a source](https://app.staging.openconceptlab.org/#/orgs/OCL/sources/Datatypes/)
- **#1738** — [Concept detail view / Collection membership](https://app.staging.openconceptlab.org/#/orgs/PEPFAR-MER-FY23/sources/MER/concepts/aMnJ58kqylT/)
- **#1799** — [Org (Detail view) - round 2](https://app.v3.qa.openconceptlab.org/#/orgs/CIEL/)
- **#1757** — [Org Home View Round 1](https://app.v3.qa.openconceptlab.org/#/orgs/WHO-DDCC/)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/orgs/WHO/sources/ICD-10-WHO/)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/orgs/WHO/sources/ICD-10-WHO/concepts/A00/)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/orgs/WHO/sources/ICD-10-to-11-Maps-WHO/0.1.0/mappings)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/orgs/WHO/sources/ICD-10-to-11-Maps-WHO/mappings)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/orgs/WHO/sources/ICD-10-to-11-Maps-WHO/mappings/30)
- **#1887** — [v3 Search does not return private content](https://app.v3.qa.openconceptlab.org/#/orgs/private-org/repos?q=private)
- **#1887** — [v3 Search does not return private content](https://app.v3.qa.openconceptlab.org/#/orgs/private-org/sources/private-source/)
- **#1754** — [Searchlight enhancements](https://app.v3.qa.openconceptlab.org/#/search/?q=ciel&type=orgs)
- **#1887** — [v3 Search does not return private content](https://app.v3.qa.openconceptlab.org/#/search/?q=private&type=repos)
- **#1966** — [Org/User Page: Pinned repos are shrinking too much](https://app.v3.qa.openconceptlab.org/#/users/jamlung/)
- **#1955** — [Repo: Versions](https://app.v3.qa.openconceptlab.org/#/users/jamlung/collections/openmrs-demo/)
- **#1950** — [Repo - Summary](https://app.v3.qa.openconceptlab.org/#/users/jamlung/collections/openmrs-demo/not-private/)
- **#1955** — [Repo: Versions](https://app.v3.qa.openconceptlab.org/#/users/jamlung/collections/openmrs-demo/private/)
- **#1948** — [Repo - Mappings list](https://app.v3.qa.openconceptlab.org/#/users/jamlung/sources/test/mappings)
- **#1952** — [Detail view: Mappings](https://app.v3.qa.openconceptlab.org/#/users/jamlung/sources/test/mappings/1792951)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://app.v3.qa.openconceptlab.org/#/users/jon/)
- **#1972** — [Show mappings in same namespace in concept details](https://app.v3.qa.openconceptlab.org/#/users/jon/sources/test-source/v1/concepts/1/)
- **#2192** — [Cannot change concept datatype](https://chat.openconceptlab.org/)
- **#1694** — [Search UI, incl. preview and search results](https://chat.openconceptlab.org/t/ocl-search-journey-ahead/55)
- **#1788** — [Add registry entry to $resolveReference response](https://docs.openconceptlab.org/en/latest/oclapi/apireference/resolveReference.html)
- **#1732** — [Implement URL registry API resource](https://docs.openconceptlab.org/en/latest/oclapi/apireference/urlregistry.html)
- **#1918** — [Double-clicking Add Version crashes the page](https://errbit.openconceptlab.org/apps/616f8ee7e685dd0007ed00a5/problems/66bcb816e4267500113c97fd)
- **#669** — [MSP: Indicator denominator not displaying properly](https://msp.qa.aws.openconceptlab.org/indicators/indicatorDetail?id=iUHHsl342Nj)
- **#534** — [New View: Split View](https://msp.qa.openconceptlab.org/referenceIndicator)

### GitHub (160 resources)

- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://docs.github.com/en/rest/users/followers?apiVersion=2022-11-28)
- **#1732** — [Implement URL registry API resource](https://github.com/OpenConceptLab/ocl-docs/blob/main/docs/source/oclapi/apireference/resolveReference.md)
- **#1858** — [Custom OCL account verification/reset password emails](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/01e13dda-6c21-4ca3-9a72-04cc3ad00e47)
- **#1858** — [Custom OCL account verification/reset password emails](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/03f23cd1-a417-4167-96b2-e576f8c0febc)
- **#1858** — [Custom OCL account verification/reset password emails](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/1d2f19a0-8bc5-41d1-8ad2-929ece88e3b4)
- **#1776** — [Objects (Chip view)](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/29e64574-40c8-46b7-ad11-af3e0537b1c7)
- **#1696** — [Canonical URL Registry](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/4bbf73fd-9015-43a0-a90b-065edc1304e4)
- **#1715** — [Design for repo+version and repo+version+expansion](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/6ece70ae-8520-4bd4-b5e1-683c317c8856)
- **#1858** — [Custom OCL account verification/reset password emails](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/7196ea60-85c9-4f64-b60f-e86fc8bb0db2)
- **#1776** — [Objects (Chip view)](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/966eb9da-323f-4b9a-8206-dd913f74eedc)
- **#1858** — [Custom OCL account verification/reset password emails](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/d54894e3-66a0-4057-b033-7e1f6e774771)
- **#1758** — [Mapping Listing View](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/e4bca844-412e-4755-8206-2a690bbd9c5b)
- **#1776** — [Objects (Chip view)](https://github.com/OpenConceptLab/ocl_issues/assets/1189689/f5b35776-b183-46cd-87a3-c5af51970d03)
- **#1704** — [Searchlight component](https://github.com/OpenConceptLab/ocl_issues/assets/1858162/346a5ff9-20ca-479b-b710-eff2e4f5761c)
- **#1758** — [Mapping Listing View](https://github.com/OpenConceptLab/ocl_issues/assets/1858162/c4d92165-d596-4a3f-a03a-aff4e3076819)
- **#1788** — [Add registry entry to $resolveReference response](https://github.com/OpenConceptLab/ocl_issues/assets/1858162/d143b829-4293-4c29-ae12-f9524e0c7507)
- **#1704** — [Searchlight component](https://github.com/OpenConceptLab/ocl_issues/assets/1858162/e7bce19c-688a-4899-a726-c31629405aeb)
- **#1612** — [TermBrowser dashboard](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/00f35e1d-07dc-43b3-8957-684b81b02dcf)
- **#1756** — [User Profile Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/04dccbac-c330-414b-a52f-2e3167dc983c)
- **#1617** — [Form / Edit user profile](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/0dac4600-42b3-4c2f-a950-8cf3df8c4102)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/1939b739-004e-49f1-9044-4585563a412b)
- **#1681** — [Search for some locales is broken](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/19c5822a-5970-4664-96aa-7cbd7a03a603)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/1af3e25a-c937-485d-9716-eab1df047c57)
- **#1754** — [Searchlight enhancements](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/1e2200b8-6071-4818-b3b0-1b19d0813b8e)
- **#1861** — [Global Dashboard for Anonymous User](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/1f892f13-ef9b-4f56-9932-2312dfd3e71b)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/3ecac373-09ec-4f7f-8e06-6be41b1b334d)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/4652b032-7636-4a6a-904c-3f71af964c96)
- **#1756** — [User Profile Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/4eae66e5-c905-438f-96ed-60873173086e)
- **#1756** — [User Profile Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/59e2a831-3aa6-477a-a3b5-3d953a01618f)
- **#1852** — [User Profile access - permission to view all profiles](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/5baac3b5-fe82-42f2-94e6-fadd7d136696)
- **#1684** — ["Add Expression" workflow appears to make invalid references?](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/60b9416f-bc47-4979-b897-3da6609a8325)
- **#1757, #1799** — [Org Home View Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/731a80f6-8e71-4bf1-b55b-3b9614b9cced)
- **#1755** — [My repositories](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/73d33b46-bc70-453d-bc59-8a95360b6343)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/89ca81ff-6783-41c1-ba0c-85b7ea7018fd)
- **#1862** — [Global Dashboard for Authenticated User](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/93be6bca-b6d0-4382-b02f-b910e7dee767)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/97d07282-eaf7-4bca-94a8-51037e85269a)
- **#1684** — ["Add Expression" workflow appears to make invalid references?](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/9de55b19-66f9-4076-8f97-00f9bb5339f6)
- **#1763** — [Concept tracing information - Modification indicators](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/b3883ae9-649c-49fb-9f31-ec605467743d)
- **#1681** — [Search for some locales is broken](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/b65fa8e8-a2bd-41e0-9f1e-5511edc7b9a6)
- **#1865** — [Concept / Share](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/c859aa82-f3b3-45a9-a0f5-492e2ca53539)
- **#1757** — [Org Home View Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/cf94ad79-ce83-4d29-8504-464dce4373b0)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/ea9d90a5-e4f5-487c-90a7-1de6d4fab591)
- **#1864** — [Global Search / Org](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/eadc3965-a3e2-44bd-99a5-b9ee90422f60)
- **#1763** — [Concept tracing information - Modification indicators](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/ef34ebb1-a59a-4d5d-abb7-344567927010)
- **#1694** — [Search UI, incl. preview and search results](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/efabb07e-bfcd-4f01-bee4-f6c43c02f18f)
- **#1757** — [Org Home View Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/55999682/f6a114cd-cad0-4347-892d-9f94445dd6d4)
- **#1681** — [Search for some locales is broken](https://github.com/OpenConceptLab/ocl_issues/assets/860834/7b9fcde8-ced1-4c0b-926f-b58cad10dad4)
- **#1734** — [Concept Detail View / Names + Descriptions](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/15bebc89-0f1b-4e78-86fd-9e34634bf880)
- **#1756, #1800** — [User Profile Round 1](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/18c36c89-089c-431e-a65f-0d3be375d1ac)
- **#1742** — [Canonical URL registries in the TermBrowser](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/1a92fffd-8d08-418f-a228-b08452cc508a)
- **#1621** — [Concept Detail view / Associations card](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/272044fc-74d2-4d1c-98e1-1999c1c11e73)
- **#1742** — [Canonical URL registries in the TermBrowser](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/323d7dff-e6f4-480a-8a0f-5bb4610e5be1)
- **#1612** — [TermBrowser dashboard](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/528a282e-dc4c-499d-ae1e-7cd394f1d4b4)
- **#1376** — [Create and Update Concepts and Mappings](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/57e508a8-d62e-417e-a576-e1657b691428)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/8d76100d-a311-4745-a957-06857556f2dc)
- **#1798** — [User settings page](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/94ea3cff-6a36-48db-8c27-da731a5ada18)
- **#1734** — [Concept Detail View / Names + Descriptions](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/957cba26-9b08-4395-bf28-8abedc38da4f)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/9ba91226-91a5-49e4-b594-81cee230ecb4)
- **#1864** — [Global Search / Org](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/a4b6a98e-5836-4d8b-acf6-bc43de6b06d4)
- **#1376** — [Create and Update Concepts and Mappings](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/c9ee5482-110e-47c1-9142-7384d5d9dc1b)
- **#1798** — [User settings page](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/cf4c19a5-9684-4a80-9cfc-4aadbc33a5a0)
- **#1620** — [Concept detail view / Header](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/d9ed809c-d386-44b8-8385-fae2e2b3bd3f)
- **#1376** — [Create and Update Concepts and Mappings](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/e1f3d679-14b0-496d-bac2-0dae6dd72299)
- **#1776** — [Objects (Chip view)](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/e7df003e-8de1-4343-936a-1e668e0e4d66)
- **#1776** — [Objects (Chip view)](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/eeb2784f-6103-4b80-9f61-95b5c40e0f5b)
- **#1376** — [Create and Update Concepts and Mappings](https://github.com/OpenConceptLab/ocl_issues/assets/95474174/f1c4537e-e6d9-43c4-bdd4-bd84d3b94e00)
- **#1743** — [Populate global URL registry on all environments](https://github.com/OpenConceptLab/ocl_issues/files/14095091/canonical_mappings.csv)
- **#1743** — [Populate global URL registry on all environments](https://github.com/OpenConceptLab/ocl_issues/files/14095093/canonical_references.csv)
- **#1743** — [Populate global URL registry on all environments](https://github.com/OpenConceptLab/ocl_issues/files/14095095/canonical_sources.csv)
- **#1743** — [Populate global URL registry on all environments](https://github.com/OpenConceptLab/ocl_issues/files/14095096/canonical_collections.csv)
- **#1743** — [Populate global URL registry on all environments](https://github.com/OpenConceptLab/oclapi2/blob/9cec42f672bf081ad513ecac729fc947f1c16f77/core/sources/tests/tests.py#L773)
- **#1322** — [Revise display string representation of references](https://github.com/OpenConceptLab/oclapi2/blob/master/core/collections/tests/tests.py#L2285)
- **#2282** — [Clean Up Collection References with Cascade Options](https://github.com/OpenConceptLab/oclcli)
- **#2275** — [Move repo version metadata to the header and expansions table](https://github.com/OpenConceptLab/oclweb2/commit/b4c233018624c698be019938c6378b3a381953ac)
- **#1617, #1865, #1887, #1780** — [Form / Edit user profile](https://github.com/jamlung-ri)
- **#1896** — [Reference / Search](https://github.com/notifications/unsubscribe-auth/AAJCOOIMPXTGDUY5U42472TZMXVPFAVCNFSM6AAAAABK7B3QI6VHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDEMZSGM2TGMJXG4)
- **#1972** — [Show mappings in same namespace in concept details](https://github.com/notifications/unsubscribe-auth/AAJCOOIQKU2PXLTVJIUTU5L2D4B7VAVCNFSM6AAAAABSC54ETCVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDKMJXGQ2TQMJZHE)
- **#1887** — [v3 Search does not return private content](https://github.com/notifications/unsubscribe-auth/AAJCOOJB2YSYJVDTHCHE64TZM2ELHAVCNFSM6AAAAABK5D6FWGVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDEMZTGU4TINJRGE)
- **#1948** — [Repo - Mappings list](https://github.com/notifications/unsubscribe-auth/AAJCOOJW23BTLJA7SLQOW5DZ6DDEVAVCNFSM6AAAAABQFS6PDWVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDINBWHA4DCNRZGU)
- **#1617** — [Form / Edit user profile](https://github.com/notifications/unsubscribe-auth/AAJCOOKFNV5CXZJS3HW7EA3ZJFK2JAVCNFSM6AAAAAA2OCH7I6VHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDCOBYG4ZTMOJTGM)
- **#1355** — [Repo Header showing unnecessary information](https://github.com/notifications/unsubscribe-auth/AAJCOOKS7FWVQ2RQWJ44PZTV7XRGNANCNFSM6AAAAAAQUFH5KI)
- **#1981** — [CTA / Repository / Release (or Un-release) a version](https://github.com/notifications/unsubscribe-auth/AAJCOOMSAUEO2ZYDKQTXI432FBQ6FAVCNFSM6AAAAABSNRGZOOVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDKMZWGQ2TKMZSGA)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/notifications/unsubscribe-auth/AAJCOOMVR2EBJ2VFKIBA7QDZJI4J3AVCNFSM6AAAAABDU5DNOKVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDCOJQGUYTAMZQGM)
- **#1888** — [User - View canonical registry](https://github.com/notifications/unsubscribe-auth/AAJCOON2X2N3HAF3IS2AQY3ZMUVV7AVCNFSM6AAAAABK5EQXMWVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDEMZRGA4DIMZRGU)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/notifications/unsubscribe-auth/AAJCOONARKLTSA6YJHGXMD3YVBDPDAVCNFSM6AAAAABDU5DNOKVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMYTSNRQHA3TINRRHE)
- **#12** — [Ability to check if new versions of resources referenced in a collection are ava](https://github.com/notifications/unsubscribe-auth/AAJCOONFYEZ6CEGKAXVKZG3QF4M3XANCNFSM4ESF7NYQ)
- **#1444** — [Concept Cloning](https://github.com/notifications/unsubscribe-auth/AAJCOONQ4V45LU4OVBLX5J3WNIIW3ANCNFSM6AAAAAAS62IBAI)
- **#1865** — [Concept / Share](https://github.com/notifications/unsubscribe-auth/AAJCOOPJXAXUL3KJQXNVBMTZJDY4VAVCNFSM6AAAAABJ2LOYRSVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDCOBXHE3DQNJVGI)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/notifications/unsubscribe-auth/AAJCOOPT3J4CFU2ZA2AHTWDYU6FUPAVCNFSM6AAAAABDU5DNOKVHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMYTSNRQGAZTENRZGI)
- **#12** — [Ability to check if new versions of resources referenced in a collection are ava](https://github.com/notifications/unsubscribe-auth/ABInOWpm81nYjJhVeqFeEGYBCnViC1YPks5vItL2gaJpZM4SRftx)
- **#1627** — [Profile / User](https://github.com/notifications/unsubscribe-auth/AWYND7UY5ETDWMPLTT2HYPLZQOBZDAVCNFSM6AAAAABJJATU56VHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDENZWGA2DKNJZHE)
- **#1478** — [Repository version summary](https://github.com/notifications/unsubscribe-auth/AWYND7VDDTG7U26DJJCOBU3W5LSSRANCNFSM6AAAAAAUX5RFT4)
- **#1478, #1981, #1627** — [Repository version summary](https://github.com/paulsonder)
- **#12, #1355, #1780** — [Ability to check if new versions of resources referenced in a collection are ava](https://github.com/paynejd)
- **#1948, #1972, #1627** — [Repo - Mappings list](https://github.com/snyaggarwal)
- **#1972** — [Show mappings in same namespace in concept details](https://github.com/user-attachments/assets/032a8ae7-87d8-4a5a-8ff3-1fa59ba090ad)
- **#1972** — [Show mappings in same namespace in concept details](https://github.com/user-attachments/assets/053e8665-87b7-4973-ada9-88614f7e1310)
- **#1799** — [Org (Detail view) - round 2](https://github.com/user-attachments/assets/08ddf281-6944-4e7f-a35d-1fa7cc87236b)
- **#1887** — [v3 Search does not return private content](https://github.com/user-attachments/assets/095b7f03-23a3-4a6f-96f2-d0b9ad7863b9)
- **#1621** — [Concept Detail view / Associations card](https://github.com/user-attachments/assets/0ae41253-3e77-41bf-b27c-fd137ff931fc)
- **#1617** — [Form / Edit user profile](https://github.com/user-attachments/assets/0e08d2be-da3a-4719-8e15-048242ead70b)
- **#1950** — [Repo - Summary](https://github.com/user-attachments/assets/0e3b69fb-3214-4d5b-84b0-9ae6ae7b00de)
- **#1862** — [Global Dashboard for Authenticated User](https://github.com/user-attachments/assets/11a5ace8-d799-44c6-80b0-bb4e3737e013)
- **#1952** — [Detail view: Mappings](https://github.com/user-attachments/assets/13ddc217-ac7b-4b18-adc3-95113afbc9a4)
- **#1966** — [Org/User Page: Pinned repos are shrinking too much](https://github.com/user-attachments/assets/14b2b289-cf16-4cee-89d7-0095fa02fba3)
- **#1981** — [CTA / Repository / Release (or Un-release) a version](https://github.com/user-attachments/assets/1637c8e4-1b73-4fec-b980-5a0bd2937b42)
- **#2131** — [TBv2 concept details & associations hot fixes](https://github.com/user-attachments/assets/1a89166d-78b6-46b1-a617-2ec8eae12aab)
- **#1955** — [Repo: Versions](https://github.com/user-attachments/assets/1ebaeb01-27ce-4903-a91b-57a4c269f2a8)
- **#1999** — [Author: Mapping](https://github.com/user-attachments/assets/1f6b59c1-64d4-4913-8aef-adb3619ace36)
- **#1955** — [Repo: Versions](https://github.com/user-attachments/assets/20baeaa3-2127-4710-8773-dd6341dc2cf2)
- **#1862** — [Global Dashboard for Authenticated User](https://github.com/user-attachments/assets/2d6c7d09-ef7a-461f-acf5-7163964507b9)
- **#1967** — [Signed-in Dashboard Empty State not working properly](https://github.com/user-attachments/assets/2ddd2d01-13eb-4c0b-8d09-80291a8a9e17)
- **#1952** — [Detail view: Mappings](https://github.com/user-attachments/assets/363dd483-c8dd-42d5-a556-f6a9420c2903)
- **#1478** — [Repository version summary](https://github.com/user-attachments/assets/3df9718f-ff62-4544-9f01-898fcc810878)
- **#1939** — [Sorting when displaying Concepts, Mappings, Repositories, etc.](https://github.com/user-attachments/assets/3ef9afcb-5c2a-4ef1-8bab-84cb43835f3a)
- **#1937** — [Fuzzy search result formatting - slight adjustment](https://github.com/user-attachments/assets/3f5dc4d7-b920-4003-bd23-c9951710759a)
- **#1952** — [Detail view: Mappings](https://github.com/user-attachments/assets/4837773b-e1ab-499a-997d-1b31598bbb89)
- **#1621** — [Concept Detail view / Associations card](https://github.com/user-attachments/assets/4ba314bf-f023-41f3-977a-60871b657131)
- **#2290** — [Implement history for concept & mapping details in TBv3](https://github.com/user-attachments/assets/4cdd65d9-c528-4cd8-b9c8-9bb5fe2c911c)
- **#1980** — [CTA / Repository / Create version](https://github.com/user-attachments/assets/59625b13-07e6-4f2c-9c22-f0bbe1320035)
- **#2120** — [Remove alternating table bgcolor in search results](https://github.com/user-attachments/assets/5b6e3261-32e2-4209-a106-7baa4caf827e)
- **#1776** — [Objects (Chip view)](https://github.com/user-attachments/assets/5d07269b-6574-4cba-a584-f42144396aaf)
- **#1948** — [Repo - Mappings list](https://github.com/user-attachments/assets/6d365e07-5714-4afb-934c-0b950b753889)
- **#1967** — [Signed-in Dashboard Empty State not working properly](https://github.com/user-attachments/assets/6e0c3449-2202-404f-a408-d40984c5b92a)
- **#1979** — [CTA / Repository / Edit](https://github.com/user-attachments/assets/6eeb5ea0-ebd0-4914-ba62-1790fc48e0b5)
- **#2268** — [Remove all possible pathways to creating new resource versioned references](https://github.com/user-attachments/assets/7029f764-c090-4b15-af78-8de5b965601b)
- **#1776** — [Objects (Chip view)](https://github.com/user-attachments/assets/716f4937-6b6e-4f13-903a-e26783e45985)
- **#2290** — [Implement history for concept & mapping details in TBv3](https://github.com/user-attachments/assets/7432f662-ec20-40fc-805c-3648382a03eb)
- **#2288** — [Quick update to TBv2 Collection Versions/Expansions Tab](https://github.com/user-attachments/assets/7ad674a8-4f16-400a-959a-5db7179a5b5e)
- **#1980** — [CTA / Repository / Create version](https://github.com/user-attachments/assets/7f1e9fa5-1f95-4b49-ac68-0a837a3a91b9)
- **#1888** — [User - View canonical registry](https://github.com/user-attachments/assets/80842cc4-7f6c-447b-bcb0-2061b245c13b)
- **#1776** — [Objects (Chip view)](https://github.com/user-attachments/assets/80a9fd89-598d-4e72-b213-4929055038cf)
- **#2000** — [CTA / Mapping / Create](https://github.com/user-attachments/assets/823dc8b3-a905-45b0-b20b-0ff060a98ea1)
- **#2014** — [CTA / Organization / Create](https://github.com/user-attachments/assets/836ca913-fce0-49e1-ad97-dfca94011083)
- **#1887** — [v3 Search does not return private content](https://github.com/user-attachments/assets/83efff56-4e4a-4e5c-b4ad-0c611691f7f0)
- **#1776** — [Objects (Chip view)](https://github.com/user-attachments/assets/8890d926-bb99-49b2-9e26-295086835c23)
- **#2131** — [TBv2 concept details & associations hot fixes](https://github.com/user-attachments/assets/904f0e03-99a4-4553-9e20-f97b7145170b)
- **#1617** — [Form / Edit user profile](https://github.com/user-attachments/assets/9514d51f-4f0c-4da4-9c10-642a60b09862)
- **#2290** — [Implement history for concept & mapping details in TBv3](https://github.com/user-attachments/assets/97756afa-7cc8-4ff1-b846-b8b5bb50d1dd)
- **#1948** — [Repo - Mappings list](https://github.com/user-attachments/assets/9af51889-5872-4aec-9ffa-a5e1aa395c22)
- **#1977** — [Author: Repository](https://github.com/user-attachments/assets/a64c79e2-cbdc-4e2d-a4fb-e38896e50fb0)
- **#1981** — [CTA / Repository / Release (or Un-release) a version](https://github.com/user-attachments/assets/a7944637-49c8-46e9-90b7-f53793149cd3)
- **#1995** — [CTA / Concept / Add mapping](https://github.com/user-attachments/assets/afde266e-1ecd-43fe-9bea-bb6ff801f7f5)
- **#1862** — [Global Dashboard for Authenticated User](https://github.com/user-attachments/assets/b16abb72-dea4-4489-8f97-48ad5e02a86d)
- **#1776** — [Objects (Chip view)](https://github.com/user-attachments/assets/bdbf2cb7-063f-446b-824e-3475009dd691)
- **#2021** — [CTA / Organization / Delete](https://github.com/user-attachments/assets/c416fcbc-1f00-4578-ba00-00e77f9b4411)
- **#1888** — [User - View canonical registry](https://github.com/user-attachments/assets/c510d30d-48c9-49b0-a5c2-8b7097955f1c)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://github.com/user-attachments/assets/cc06976e-9718-4ccb-9b7a-5b42baa811ef)
- **#1887** — [v3 Search does not return private content](https://github.com/user-attachments/assets/d44daf65-d435-4b4d-9cbe-1e3074ec53d9)
- **#1981** — [CTA / Repository / Release (or Un-release) a version](https://github.com/user-attachments/assets/d4c41f78-1751-4311-a953-0cb5b9ed6daf)
- **#1999** — [Author: Mapping](https://github.com/user-attachments/assets/ddc04e04-3f5f-4e0b-a678-5218e32a86e9)
- **#1966** — [Org/User Page: Pinned repos are shrinking too much](https://github.com/user-attachments/assets/de010114-0672-462e-8213-c3de3f421dff)
- **#1955** — [Repo: Versions](https://github.com/user-attachments/assets/dfb91c87-5c50-47bf-a804-77c30b664bb9)
- **#1918** — [Double-clicking Add Version crashes the page](https://github.com/user-attachments/assets/e0f0e9eb-2473-4d54-a292-3744162dfcb1)
- **#1620** — [Concept detail view / Header](https://github.com/user-attachments/assets/e8e30db8-856b-4c5f-b979-0348a0b966a2)
- **#1948** — [Repo - Mappings list](https://github.com/user-attachments/assets/ec1dacec-f360-4bc3-900a-1243229897fc)
- **#1620** — [Concept detail view / Header](https://github.com/user-attachments/assets/ec8788de-2d9d-4c9f-b1dc-31edb2b05068)
- **#2056** — [Concept Details Query Gone Wrong](https://github.com/user-attachments/assets/f6fb9372-417b-43c8-80c2-023d2f325e22)
- **#2014** — [CTA / Organization / Create](https://github.com/user-attachments/assets/f98be3ac-225b-47b6-ac83-08e710b9a360)
- **#1952** — [Detail view: Mappings](https://github.com/user-attachments/assets/fc9a740f-458a-49bf-82cb-fc7dea8baad5)

### External (108 resources)

- **#1865, #1742** — [Concept / Share](http://loinc.org)
- **#1742** — [Canonical URL registries in the TermBrowser](http://loinc.org\|2.60)
- **#1322** — [Revise display string representation of references](http://who.int/cg/CodeSystem/extended-content/concepts/G/’)
- **#534** — [New View: Split View](https://app.zeplin.io/project/61d73b9f4187e7636b96e741)
- **#1980** — [CTA / Repository / Create version](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e)
- **#1702** — [Sign-In/out, Registration Flows with KeyCloak theme](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/655370f3e0c5f8727e58581f)
- **#1756** — [User Profile Round 1](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/65bbb39b5edc2467493a2c00)
- **#1758** — [Mapping Listing View](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/65bbb500bbf6143e38d6e807)
- **#1756** — [User Profile Round 1](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/65bbb5e016ed139e67cde990)
- **#1978** — [CTA / Repository / Create](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/667400da447767e439b82ee9)
- **#1799** — [Org (Detail view) - round 2](https://app.zeplin.io/project/623204ca71595ab4a7e3f67e/screen/668514841ad8f42d5b476269)
- **#1471** — [Extract out English language bundle from TermBrowser](https://help.transifex.com/en/collections/3511306-for-developers#file-formats)
- **#534** — [New View: Split View](https://icd.who.int/browse10/2019/en#/)
- **#2014** — [CTA / Organization / Create](https://identicon.net/)
- **#1478** — [Repository version summary](https://images.zenhubusercontent.com/5981a15ff8b79f8376e1a78e/4c063b2c-ba1b-4202-bbce-4d4608a564a1)
- **#1022** — [Unable to add members to an org in the UI](https://images.zenhubusercontent.com/5981a15ff8b79f8376e1a78e/6f9ff62c-bb4c-4419-a91f-916850abaa41)
- **#1022** — [Unable to add members to an org in the UI](https://images.zenhubusercontent.com/5981a15ff8b79f8376e1a78e/be2d2a07-8cf7-4a20-9c3a-195c4a9fbe9e)
- **#1355** — [Repo Header showing unnecessary information](https://images.zenhubusercontent.com/5df114a3d094b314eeec671c/ad1ff33a-67ba-466b-b6ab-81ca90cab5fb)
- **#1079** — [Add filter button for Extra Attributes in the Associations Section of the Concep](https://images.zenhubusercontent.com/5df114a3d094b314eeec671c/fa51045b-370d-4e8c-b719-5524847309cc)
- **#1612** — [TermBrowser dashboard](https://images.zenhubusercontent.com/62f2740361fc271207ca9fb3/0f3d630b-ba11-4ab2-a304-e34f73186a30)
- **#1776** — [Objects (Chip view)](https://loinc.org/)
- **#1742** — [Canonical URL registries in the TermBrowser](https://loinc.org\|2.60”)
- **#1742** — [Canonical URL registries in the TermBrowser](https://loinc.org”)
- **#1376** — [Create and Update Concepts and Mappings](https://m3.material.io/components/side-sheets/overview)
- **#534** — [New View: Split View](https://miro.com/app/board/o9J_ljd25Io=/?fromRedirect=1)
- **#1627** — [Profile / User](https://miro.com/app/board/uXjVK5g8WOU=/)
- **#1776** — [Objects (Chip view)](https://miro.com/app/board/uXjVKdKjbls=/)
- **#1708** — [Dashboard Feed - Your recently updated repos](https://miro.com/app/board/uXjVNefU-jA=/?moveToWidget=3458764565896758814&cot=14)
- **#2155** — [Repo Creation Starting Point UI](https://miro.com/app/board/uXjVNeppQ4Y=/?moveToWidget=3458764625606267189&cot=14)
- **#2157** — [Create repo --> Import content from existing repo](https://miro.com/app/board/uXjVNeppQ4Y=/?moveToWidget=3458764625606597413&cot=14)
- **#2005** — [Add References within Collection](https://miro.com/app/board/uXjVNeppQ4Y=/?moveToWidget=3458764631968376805&cot=14)
- **#2005** — [Add References within Collection](https://miro.com/app/board/uXjVNeppQ4Y=/?moveToWidget=3458764632047177832&cot=14)
- **#2005** — [Add References within Collection](https://miro.com/app/board/uXjVNeppQ4Y=/?moveToWidget=3458764632695311569&cot=14)
- **#1734** — [Concept Detail View / Names + Descriptions](https://mui.com/base-ui/react-textarea-autosize/)
- **#1478** — [Repository version summary](https://mui.com/material-ui/react-progress/#circular-color)
- **#1478** — [Repository version summary](https://mui.com/material-ui/react-skeleton/)
- **#1758** — [Mapping Listing View](https://mui.com/material-ui/react-table/#collapsible-table)
- **#1617** — [Form / Edit user profile](https://mui.com/material-ui/react-text-field/#input-adornments)
- **#1376** — [Create and Update Concepts and Mappings](https://mui.com/x/react-data-grid/)
- **#1607** — [Advanced search MVP](https://pubmed.ncbi.nlm.nih.gov/advanced/)
- **#816, #909** — [Concept Modifications](https://talk.openmrs.org/t/reference-vital-sign-ranges/38055/1)
- **#1853** — [Repo / Compare Versions](https://www.jsondiff.com/)
- **#1853** — [Repo / Compare Versions](https://www.loom.com/share/1b5213914b8e43f0a681fd572d44f027?sid=ec3c44b1-8cba-4b8f-a597-1dcb50cb62df)
- **#1877** — [Search a Repository](https://www.loom.com/share/25cb8b5f61104fafa5107b35a7d95b30)
- **#1888** — [User - View canonical registry](https://www.loom.com/share/4be2ade301e84c908dcb38019edee9f5?sid=7cc0592f-1c35-4594-be3f-a4abfab1a3c9)
- **#1673** — [Datatype Dropdown bug - cannot use one of my common datatypes in a source](https://www.loom.com/share/80f984f6880e4f41894284fd394e5a48?from_recorder=1&focus_title=1)
- **#1684** — ["Add Expression" workflow appears to make invalid references?](https://www.loom.com/share/8c4e3233f8184716b67e77f172dfc4d4)
- **#1888** — [User - View canonical registry](https://www.loom.com/share/d103cc7cd884429e9ec3908befd1a858?sid=27499cec-58cb-49c3-8e8e-25b589b0f33d)
- **#1853** — [Repo / Compare Versions](https://www.loom.com/share/fbf949410843407f935601ada7df35ea?sid=92f107f8-e0eb-436c-ae7e-cf5300c94903)
- **#2014** — [CTA / Organization / Create](https://www.npmjs.com/package/jazzicon)
- **#1471** — [Extract out English language bundle from TermBrowser](https://www.transifex.com/)
- **#1861** — [Global Dashboard for Anonymous User](https://zpl.io/1yoLRmE)
- **#1620** — [Concept detail view / Header](https://zpl.io/1yorRPm)
- **#1754** — [Searchlight enhancements](https://zpl.io/3031x7N)
- **#1979** — [CTA / Repository / Edit](https://zpl.io/30X7gvY)
- **#1861** — [Global Dashboard for Anonymous User](https://zpl.io/30Xqmvd)
- **#1979** — [CTA / Repository / Edit](https://zpl.io/5NEyBn8)
- **#1757** — [Org Home View Round 1](https://zpl.io/5NZJeG8)
- **#1942, #1780** — [Changes to User Bio on the User Profile page](https://zpl.io/7Gwxv3q)
- **#1755** — [My repositories](https://zpl.io/9N4wEZB)
- **#1948** — [Repo - Mappings list](https://zpl.io/9NqGAjz)
- **#1703** — [Search results templates](https://zpl.io/9qBJqwW)
- **#1950** — [Repo - Summary](https://zpl.io/AOmPyXn)
- **#1776** — [Objects (Chip view)](https://zpl.io/DpGXPBq)
- **#1757, #1799** — [Org Home View Round 1](https://zpl.io/DpGd5Kr)
- **#1862, #1941, #1627** — [Global Dashboard for Authenticated User](https://zpl.io/DplDQ6O)
- **#1950, #1951** — [Repo - Summary](https://zpl.io/DplNkR3)
- **#1861** — [Global Dashboard for Anonymous User](https://zpl.io/DplYYKe)
- **#1978** — [CTA / Repository / Create](https://zpl.io/GnEgo5m)
- **#1627, #1799** — [Profile / User](https://zpl.io/JQEZrOr)
- **#1862** — [Global Dashboard for Authenticated User](https://zpl.io/KEG08jR)
- **#1888** — [User - View canonical registry](https://zpl.io/MLd4kD3)
- **#1776** — [Objects (Chip view)](https://zpl.io/MLdwlRK)
- **#1742** — [Canonical URL registries in the TermBrowser](https://zpl.io/NEZXkwd)
- **#1724** — [Community website link](https://zpl.io/O0Jz9rY)
- **#1953** — [Detail view: References](https://zpl.io/OX0EAYQ)
- **#1799** — [Org (Detail view) - round 2](https://zpl.io/RmMWm9z)
- **#1780** — [Following Organizations, Users, or Repositories (related to Events/Feeds)](https://zpl.io/RmMYBKz)
- **#1621** — [Concept Detail view / Associations card](https://zpl.io/RmrQPqv)
- **#1955** — [Repo: Versions](https://zpl.io/WQ47Xde)
- **#1758** — [Mapping Listing View](https://zpl.io/WQ4gNXx)
- **#1756** — [User Profile Round 1](https://zpl.io/XKLWoYl)
- **#1951** — [Repo - Overview](https://zpl.io/YPYD8eM)
- **#1617** — [Form / Edit user profile](https://zpl.io/Zq0xA9l)
- **#1776** — [Objects (Chip view)](https://zpl.io/dR4l6vl)
- **#1627, #1780** — [Profile / User](https://zpl.io/dRxAQL7)
- **#1948** — [Repo - Mappings list](https://zpl.io/dRxKmWn)
- **#1949** — [Repo - References list](https://zpl.io/dRxQzK7)
- **#1800** — [User (Detail view) - Empty State + Activity Feed](https://zpl.io/eznmnPW)
- **#1887** — [v3 Search does not return private content](https://zpl.io/eznn8ql)
- **#1982** — [CTA / Repository / Make a copy](https://zpl.io/ezwroPm)
- **#1978** — [CTA / Repository / Create](https://zpl.io/gg8yrY1)
- **#1755** — [My repositories](https://zpl.io/ggeQBvM)
- **#1866** — [Repo / Search - Enhancements](https://zpl.io/ggeeNPN)
- **#1978** — [CTA / Repository / Create](https://zpl.io/jpZ8J1x)
- **#1755, #1756** — [My repositories](https://zpl.io/jpj5R4r)
- **#1627, #1780** — [Profile / User](https://zpl.io/kKDrLeG)
- **#2123** — [Consumer - View repository references / expansions](https://zpl.io/m8DPK16)
- **#1799** — [Org (Detail view) - round 2](https://zpl.io/n6OQGeY)
- **#1952** — [Detail view: Mappings](https://zpl.io/n6ozLpY)
- **#1616, #2014** — [Form / Create org](https://zpl.io/p011vMy)
- **#1853** — [Repo / Compare Versions](https://zpl.io/p01EPNr)
- **#1612** — [TermBrowser dashboard](https://zpl.io/p1DqwQQ)
- **#1627, #1756** — [Profile / User](https://zpl.io/q1mvlLZ)
- **#1376** — [Create and Update Concepts and Mappings](https://zpl.io/vn1GWY7)
- **#1967** — [Signed-in Dashboard Empty State not working properly](https://zpl.io/vnMEY0P)
- **#1704** — [Searchlight component](https://zpl.io/wyL09DN)
- **#1376** — [Create and Update Concepts and Mappings](https://zpl.io/zwkrdpM)

### Image (18 resources)

- **#1763** — [Concept tracing information - Modification indicators](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBL0ZJQXc9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--55dbc07bf7c3fd2b1516556bb2827fafc140e08b/image.png)
- **#1763** — [Concept tracing information - Modification indicators](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBL0pJQXc9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--3eea7477d821f027259573fe87b16eeec03eb230/image.png)
- **#1941** — [Change Recent Activity label](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBL1JGQnc9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--dfa8cb8f812952b952fc7d236ff9b2131533dfef/image.png)
- **#1858** — [Custom OCL account verification/reset password emails](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMEIrQlE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--ff1d8ee140d1e2339cd8930702319fbdcaf44398/image.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMS9pQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--d270a669a67b16d560de882905eb999cf711bc6d/Add%20to%20Collection%20-%20Cascade%20only%20-%20no%20visualization%20but%20users%20dont%20know%20what%20content%20will%20get%20added.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMkRpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--cb24576d6d6d00a15004a9676c4dc61a20a1fe62/Clone%20workflow.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMkhpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--5ea5f4b41093cb2a7b7861838aff34de434fb79a/Concept%20details%20in%20Global%20Search.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMkxpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--9af8665ce565c8cfcec6568078c7204193fb9719/Operations%20Panel.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMlBpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--9c2d02d1bce3fa5d0f584305a3939af4f7dcdcba/Hierarchy%20tree%20view%20%28ICD%29.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMlRpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--5a8582066c7ed19e683c89e9b8f8ed85f03d87d6/Cascade%20Vizualizer%20%28ICD%20hierarchy%20only%29.png)
- **#1700** — [Hierarchy/Cascade Component - Seeing context for a concept](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBMmJpQVE9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--abd0865d33c3cf0780d88142217807dbaae766c9/Cascade%20Vizualizer%20%28CIEL%20mappings%29.png)
- **#1708** — [Dashboard Feed - Your recently updated repos](https://api.zenhub.com/attachedFiles/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBNWN6QWc9PSIsImV4cCI6bnVsbCwicHVyIjoiYmxvYl9pZCJ9fQ==--a999e4ec2a9e901b0fb354996bf4f86358d85986/image.png)
- **#1887** — [v3 Search does not return private content](https://private-user-images.githubusercontent.com/55999682/348826289-095b7f03-23a3-4a6f-96f2-d0b9ad7863b9.gif?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjEyMTk0OTYsIm5iZiI6MTcyMTIxOTE5NiwicGF0aCI6Ii81NTk5OTY4Mi8zNDg4MjYyODktMDk1YjdmMDMtMjNhMy00YTZmLTk2ZjItZDBiOWFkNzg2M2I5LmdpZj9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA3MTclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNzE3VDEyMjYzNlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTZjMTMyNDg4YWMzY2I5MjgzMTEyMzVjMTYwMTkwYzdkZDVmODBmMjc3MGQxMzQxZTA5MTJiY2UwYjhjZmM1ZGYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.vB3j3YB3up8Mk4-S9TKrrre6FkBgS_HA0PUPZIYv8Iw)
- **#1948** — [Repo - Mappings list](https://private-user-images.githubusercontent.com/95474174/381525912-6d365e07-5714-4afb-934c-0b950b753889.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MzAyOTcxNzcsIm5iZiI6MTczMDI5Njg3NywicGF0aCI6Ii85NTQ3NDE3NC8zODE1MjU5MTItNmQzNjVlMDctNTcxNC00YWZiLTkzNGMtMGI5NTBiNzUzODg5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDEwMzAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQxMDMwVDE0MDExN1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPThkMzkwNjcxMGJkOTBhMTI2N2Q3MWE3M2FkYzEyMDU3ZDhhODEwYjRlNDY0MWFmYmVkMjJhNWY5NTViMjIwYTUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.b8yeNnBAldEKsauWk0PiZBViTJ_rt9WAWXGjCDDl-nE)
- **#1202** — [API and UI: Better results panel when adding references to collections](https://user-images.githubusercontent.com/55999682/152218422-0348c541-7da3-46f2-8f89-82f7917d9982.png)
- **#1478** — [Repository version summary](https://user-images.githubusercontent.com/55999682/229603549-90ee9b88-904d-490b-a18f-a4208a416312.png)
- **#1478** — [Repository version summary](https://user-images.githubusercontent.com/55999682/229603746-514bfe8c-a4bf-48df-9479-0185b25dd80b.png)
- **#1376** — [Create and Update Concepts and Mappings](https://user-images.githubusercontent.com/95474174/205044105-2dd3ce37-4d7d-464b-acdb-504ffd9d5684.png)

---

## Appendix A: Issue Index

Cross-reference of all issues cited in this knowledge base.

| Issue | Title | State | Sections |
|-------|-------|-------|----------|
| [#12](https://github.com/OpenConceptLab/ocl_issues/issues/12) | Ability to check if new versions of resources referenced ... | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#13](https://github.com/OpenConceptLab/ocl_issues/issues/13) | Ability to update collection references to latest version... | OPEN | Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#22](https://github.com/OpenConceptLab/ocl_issues/issues/22) | Unknown | ? | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#233](https://github.com/OpenConceptLab/ocl_issues/issues/233) | TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#530](https://github.com/OpenConceptLab/ocl_issues/issues/530) | Filter Configuration | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#534](https://github.com/OpenConceptLab/ocl_issues/issues/534) | New View: Split View | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#535](https://github.com/OpenConceptLab/ocl_issues/issues/535) | Resource Download Formats | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#536](https://github.com/OpenConceptLab/ocl_issues/issues/536) | Resource Details Page Customization Options | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#591](https://github.com/OpenConceptLab/ocl_issues/issues/591) | Download/View as FHIR | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#667](https://github.com/OpenConceptLab/ocl_issues/issues/667) | MSP: Show years in which a reference indicator is active ... | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#669](https://github.com/OpenConceptLab/ocl_issues/issues/669) | MSP: Indicator denominator not displaying properly | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#672](https://github.com/OpenConceptLab/ocl_issues/issues/672) | MSP: Changing Fiscal Year filter when navigating to a pre... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#788](https://github.com/OpenConceptLab/ocl_issues/issues/788) | Filter Configuration Feature Additions (Not Immediate) | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#793](https://github.com/OpenConceptLab/ocl_issues/issues/793) | OCL TermBrowser Mapping Revamp | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#808](https://github.com/OpenConceptLab/ocl_issues/issues/808) | Custom View Configuration in TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#816](https://github.com/OpenConceptLab/ocl_issues/issues/816) | Concept Modifications | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#863](https://github.com/OpenConceptLab/ocl_issues/issues/863) | Bulk Import UI Improvements | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#865](https://github.com/OpenConceptLab/ocl_issues/issues/865) | FHIR UI in TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#882](https://github.com/OpenConceptLab/ocl_issues/issues/882) | Hierarchy UI in TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#888](https://github.com/OpenConceptLab/ocl_issues/issues/888) | OCL Resource Comparison | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#900](https://github.com/OpenConceptLab/ocl_issues/issues/900) | Expose "retired" option when Exporting in UI | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#909](https://github.com/OpenConceptLab/ocl_issues/issues/909) | Concept Modification/Expansion Support in TermBrowser | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#913](https://github.com/OpenConceptLab/ocl_issues/issues/913) | Configuration Sandbox / Custom Domain for TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#958](https://github.com/OpenConceptLab/ocl_issues/issues/958) | Compare UI validation/restrictions to API | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#962](https://github.com/OpenConceptLab/ocl_issues/issues/962) | Expose validation schema error messages to user | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#985](https://github.com/OpenConceptLab/ocl_issues/issues/985) | Add async counts into org sources, collections and member... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1003](https://github.com/OpenConceptLab/ocl_issues/issues/1003) | Support standalone integration of TermBrowser with SVCM s... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1022](https://github.com/OpenConceptLab/ocl_issues/issues/1022) | Unable to add members to an org in the UI | OPEN | Section 8: Resource Library |
| [#1079](https://github.com/OpenConceptLab/ocl_issues/issues/1079) | Add filter button for Extra Attributes in the Association... | OPEN | Section 8: Resource Library |
| [#1113](https://github.com/OpenConceptLab/ocl_issues/issues/1113) | Favorite Icon, allowing users to "Favorite" any repo | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1185](https://github.com/OpenConceptLab/ocl_issues/issues/1185) | Abstract TermBrowser connectors into their own objects | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1186](https://github.com/OpenConceptLab/ocl_issues/issues/1186) | Multi-lingual support in OCL TermBrowser | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1202](https://github.com/OpenConceptLab/ocl_issues/issues/1202) | API and UI: Better results panel when adding references t... | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1281](https://github.com/OpenConceptLab/ocl_issues/issues/1281) | Collection Management UI and Workflows | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1297](https://github.com/OpenConceptLab/ocl_issues/issues/1297) | Add $resolveReference to Operations Panel | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1318](https://github.com/OpenConceptLab/ocl_issues/issues/1318) | Highlight concept attributes/associations that were modif... | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1322](https://github.com/OpenConceptLab/ocl_issues/issues/1322) | Revise display string representation of references | OPEN | Section 8: Resource Library |
| [#1355](https://github.com/OpenConceptLab/ocl_issues/issues/1355) | Repo Header showing unnecessary information | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1361](https://github.com/OpenConceptLab/ocl_issues/issues/1361) | Quick Add Mappings in Collection | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1371](https://github.com/OpenConceptLab/ocl_issues/issues/1371) | Canonical URL Restrictions (Round 1) | OPEN | Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1373](https://github.com/OpenConceptLab/ocl_issues/issues/1373) | Remove References from Collection | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1374](https://github.com/OpenConceptLab/ocl_issues/issues/1374) | Display/Interpretation of Collection References | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1375](https://github.com/OpenConceptLab/ocl_issues/issues/1375) | Reference display text update - include concept name in a... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1376](https://github.com/OpenConceptLab/ocl_issues/issues/1376) | Create and Update Concepts and Mappings | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#1444](https://github.com/OpenConceptLab/ocl_issues/issues/1444) | Concept Cloning | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1455](https://github.com/OpenConceptLab/ocl_issues/issues/1455) | Cascade preview visualizer/component | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1471](https://github.com/OpenConceptLab/ocl_issues/issues/1471) | Extract out English language bundle from TermBrowser | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1478](https://github.com/OpenConceptLab/ocl_issues/issues/1478) | Repository version summary | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1522](https://github.com/OpenConceptLab/ocl_issues/issues/1522) | Deep linking | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1537](https://github.com/OpenConceptLab/ocl_issues/issues/1537) | Ability to copy/download the results from adding referenc... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1538](https://github.com/OpenConceptLab/ocl_issues/issues/1538) | Resolve references (where possible) in the add references... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1592](https://github.com/OpenConceptLab/ocl_issues/issues/1592) | Switch TermBrowser hierarchy view to use cascade | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1607](https://github.com/OpenConceptLab/ocl_issues/issues/1607) | Advanced search MVP | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1612](https://github.com/OpenConceptLab/ocl_issues/issues/1612) | TermBrowser dashboard | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1614](https://github.com/OpenConceptLab/ocl_issues/issues/1614) | Change requests and management | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1615](https://github.com/OpenConceptLab/ocl_issues/issues/1615) | Form / Create repo | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1616](https://github.com/OpenConceptLab/ocl_issues/issues/1616) | Form / Create org | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1617](https://github.com/OpenConceptLab/ocl_issues/issues/1617) | Form / Edit user profile | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1618](https://github.com/OpenConceptLab/ocl_issues/issues/1618) | Form / Repo / Add references | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1620](https://github.com/OpenConceptLab/ocl_issues/issues/1620) | Concept detail view / Header | OPEN | Section 3: Consumer Experience, Section 4: Authoring Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1621](https://github.com/OpenConceptLab/ocl_issues/issues/1621) | Concept Detail view / Associations card | OPEN | Section 3: Consumer Experience, Section 4: Authoring Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1622](https://github.com/OpenConceptLab/ocl_issues/issues/1622) | Detail view / Change request | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#1623](https://github.com/OpenConceptLab/ocl_issues/issues/1623) | Detail view / Compare | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1624](https://github.com/OpenConceptLab/ocl_issues/issues/1624) | Operations panel | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1625](https://github.com/OpenConceptLab/ocl_issues/issues/1625) | Consumer: Repository | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1627](https://github.com/OpenConceptLab/ocl_issues/issues/1627) | Profile / User | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1663](https://github.com/OpenConceptLab/ocl_issues/issues/1663) | Add Company and Location to user registration form | CLOSED | Section 3: Consumer Experience |
| [#1673](https://github.com/OpenConceptLab/ocl_issues/issues/1673) | Datatype Dropdown bug - cannot use one of my common datat... | OPEN | Section 8: Resource Library |
| [#1681](https://github.com/OpenConceptLab/ocl_issues/issues/1681) | Search for some locales is broken | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1684](https://github.com/OpenConceptLab/ocl_issues/issues/1684) | "Add Expression" workflow appears to make invalid referen... | OPEN | Section 8: Resource Library |
| [#1686](https://github.com/OpenConceptLab/ocl_issues/issues/1686) | Unknown | ? | Sections 1-2: Executive Summary & Architecture |
| [#1687](https://github.com/OpenConceptLab/ocl_issues/issues/1687) | Implement repo_type attribute for sources and collections | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1688](https://github.com/OpenConceptLab/ocl_issues/issues/1688) | Dashboard, Registration, Sign-in | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1690](https://github.com/OpenConceptLab/ocl_issues/issues/1690) | Diffs and Checksums | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#1694](https://github.com/OpenConceptLab/ocl_issues/issues/1694) | Search UI, incl. preview and search results | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1695](https://github.com/OpenConceptLab/ocl_issues/issues/1695) | Repo Type | CLOSED | Sections 1-2: Executive Summary & Architecture |
| [#1696](https://github.com/OpenConceptLab/ocl_issues/issues/1696) | Canonical URL Registry | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1699](https://github.com/OpenConceptLab/ocl_issues/issues/1699) | Latest Released Repo Version | OPEN | Section 3: Consumer Experience |
| [#1700](https://github.com/OpenConceptLab/ocl_issues/issues/1700) | Hierarchy/Cascade Component - Seeing context for a concept | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1701](https://github.com/OpenConceptLab/ocl_issues/issues/1701) | Unknown | ? | Sections 1-2: Executive Summary & Architecture |
| [#1702](https://github.com/OpenConceptLab/ocl_issues/issues/1702) | Sign-In/out, Registration Flows with KeyCloak theme | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1703](https://github.com/OpenConceptLab/ocl_issues/issues/1703) | Search results templates | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1704](https://github.com/OpenConceptLab/ocl_issues/issues/1704) | Searchlight component | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1706](https://github.com/OpenConceptLab/ocl_issues/issues/1706) | Walled Garden and Trusted Sources | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1708](https://github.com/OpenConceptLab/ocl_issues/issues/1708) | Dashboard Feed - Your recently updated repos | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1715](https://github.com/OpenConceptLab/ocl_issues/issues/1715) | Design for repo+version and repo+version+expansion | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1724](https://github.com/OpenConceptLab/ocl_issues/issues/1724) | Community website link | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1727](https://github.com/OpenConceptLab/ocl_issues/issues/1727) | Implement repo_type attribute in TBv3 | OPEN | Section 3: Consumer Experience, Sections 1-2: Executive Summary & Architecture |
| [#1732](https://github.com/OpenConceptLab/ocl_issues/issues/1732) | Implement URL registry API resource | CLOSED | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1734](https://github.com/OpenConceptLab/ocl_issues/issues/1734) | Concept Detail View / Names + Descriptions | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1738](https://github.com/OpenConceptLab/ocl_issues/issues/1738) | Concept detail view / Collection membership | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1739](https://github.com/OpenConceptLab/ocl_issues/issues/1739) | Concept detail view / History | OPEN | Section 3: Consumer Experience |
| [#1741](https://github.com/OpenConceptLab/ocl_issues/issues/1741) | OCL Bouncy Ball Loading icon and performance benchmark | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1742](https://github.com/OpenConceptLab/ocl_issues/issues/1742) | Canonical URL registries in the TermBrowser | CLOSED | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1743](https://github.com/OpenConceptLab/ocl_issues/issues/1743) | Populate global URL registry on all environments | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1744](https://github.com/OpenConceptLab/ocl_issues/issues/1744) | Permissions for registries and $resolveReference behavior | OPEN | Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1747](https://github.com/OpenConceptLab/ocl_issues/issues/1747) | Switch mappings to use $resolveReference to resolve repo ... | CLOSED | Sections 1-2: Executive Summary & Architecture |
| [#1752](https://github.com/OpenConceptLab/ocl_issues/issues/1752) | Selecting repo in global search should navigate to latest... | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1754](https://github.com/OpenConceptLab/ocl_issues/issues/1754) | Searchlight enhancements | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1755](https://github.com/OpenConceptLab/ocl_issues/issues/1755) | My repositories | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1756](https://github.com/OpenConceptLab/ocl_issues/issues/1756) | User Profile Round 1 | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1757](https://github.com/OpenConceptLab/ocl_issues/issues/1757) | Org Home View Round 1 | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1758](https://github.com/OpenConceptLab/ocl_issues/issues/1758) | Mapping Listing View | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1763](https://github.com/OpenConceptLab/ocl_issues/issues/1763) | Concept tracing information - Modification indicators | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1769](https://github.com/OpenConceptLab/ocl_issues/issues/1769) | User Language Preferences | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1770](https://github.com/OpenConceptLab/ocl_issues/issues/1770) | Search Preferences - How does this affect the user | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1771](https://github.com/OpenConceptLab/ocl_issues/issues/1771) | Dashboard Feed - Updated Reference Content | CLOSED | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1772](https://github.com/OpenConceptLab/ocl_issues/issues/1772) | UI Method for adding collection-based reference to a coll... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1776](https://github.com/OpenConceptLab/ocl_issues/issues/1776) | Objects (Chip view) | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1780](https://github.com/OpenConceptLab/ocl_issues/issues/1780) | Following Organizations, Users, or Repositories (related ... | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1788](https://github.com/OpenConceptLab/ocl_issues/issues/1788) | Add registry entry to $resolveReference response | CLOSED | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1798](https://github.com/OpenConceptLab/ocl_issues/issues/1798) | User settings page | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1799](https://github.com/OpenConceptLab/ocl_issues/issues/1799) | Org (Detail view) - round 2 | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1800](https://github.com/OpenConceptLab/ocl_issues/issues/1800) | User (Detail view) - Empty State + Activity Feed | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1802](https://github.com/OpenConceptLab/ocl_issues/issues/1802) | My Repositories + My Orgs | OPEN | Section 3: Consumer Experience |
| [#1803](https://github.com/OpenConceptLab/ocl_issues/issues/1803) | My Repositories - Card View | OPEN | Section 3: Consumer Experience |
| [#1804](https://github.com/OpenConceptLab/ocl_issues/issues/1804) | My Repositories - Create/Update | OPEN | Section 3: Consumer Experience |
| [#1812](https://github.com/OpenConceptLab/ocl_issues/issues/1812) | Roles and Permissions for OCLv3 | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1814](https://github.com/OpenConceptLab/ocl_issues/issues/1814) | Unknown | ? | Sections 1-2: Executive Summary & Architecture |
| [#1816](https://github.com/OpenConceptLab/ocl_issues/issues/1816) | URL Registry Entry should cache resolve result in ES for ... | CLOSED | Sections 1-2: Executive Summary & Architecture |
| [#1821](https://github.com/OpenConceptLab/ocl_issues/issues/1821) | Implement namespace selector for canonical URL registry tool | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#1822](https://github.com/OpenConceptLab/ocl_issues/issues/1822) | Hierarchy as part of Add/Edit Mappings on Concept Details | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1827](https://github.com/OpenConceptLab/ocl_issues/issues/1827) | Canonical URL Registry - Add Canonical URL (basic) | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#1830](https://github.com/OpenConceptLab/ocl_issues/issues/1830) | Version-to-HEAD Mode Transition/Navigation | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1852](https://github.com/OpenConceptLab/ocl_issues/issues/1852) | User Profile access - permission to view all profiles | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1853](https://github.com/OpenConceptLab/ocl_issues/issues/1853) | Repo / Compare Versions | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1854](https://github.com/OpenConceptLab/ocl_issues/issues/1854) | Consumer: Reference | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1855](https://github.com/OpenConceptLab/ocl_issues/issues/1855) | Consumer: Expansion | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1857](https://github.com/OpenConceptLab/ocl_issues/issues/1857) | Object-focused Summary Metrics | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1858](https://github.com/OpenConceptLab/ocl_issues/issues/1858) | Custom OCL account verification/reset password emails | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1861](https://github.com/OpenConceptLab/ocl_issues/issues/1861) | Global Dashboard for Anonymous User | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1862](https://github.com/OpenConceptLab/ocl_issues/issues/1862) | Global Dashboard for Authenticated User | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1864](https://github.com/OpenConceptLab/ocl_issues/issues/1864) | Global Search / Org | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1865](https://github.com/OpenConceptLab/ocl_issues/issues/1865) | Concept / Share | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1866](https://github.com/OpenConceptLab/ocl_issues/issues/1866) | Repo / Search - Enhancements | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1871](https://github.com/OpenConceptLab/ocl_issues/issues/1871) | Repo / Header | OPEN | Section 3: Consumer Experience, Sections 1-2: Executive Summary & Architecture |
| [#1872](https://github.com/OpenConceptLab/ocl_issues/issues/1872) | Repo Version Selector | OPEN | Section 3: Consumer Experience, Sections 1-2: Executive Summary & Architecture |
| [#1873](https://github.com/OpenConceptLab/ocl_issues/issues/1873) | Repository / Download + Export | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1874](https://github.com/OpenConceptLab/ocl_issues/issues/1874) | In TB, warn users that underscore not officially supporte... | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1877](https://github.com/OpenConceptLab/ocl_issues/issues/1877) | Search a Repository | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1878](https://github.com/OpenConceptLab/ocl_issues/issues/1878) | Mapping / Share | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1879](https://github.com/OpenConceptLab/ocl_issues/issues/1879) | Concept / Download | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1880](https://github.com/OpenConceptLab/ocl_issues/issues/1880) | Mapping / Download | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1887](https://github.com/OpenConceptLab/ocl_issues/issues/1887) | v3 Search does not return private content | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1888](https://github.com/OpenConceptLab/ocl_issues/issues/1888) | User - View canonical registry | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1889](https://github.com/OpenConceptLab/ocl_issues/issues/1889) | Expansion / View | OPEN | Section 3: Consumer Experience |
| [#1892](https://github.com/OpenConceptLab/ocl_issues/issues/1892) | Repository - About | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1894](https://github.com/OpenConceptLab/ocl_issues/issues/1894) | Reference / View | CLOSED | Section 3: Consumer Experience |
| [#1895](https://github.com/OpenConceptLab/ocl_issues/issues/1895) | Reference / Preview | OPEN | Section 3: Consumer Experience |
| [#1896](https://github.com/OpenConceptLab/ocl_issues/issues/1896) | Reference / Search | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1916](https://github.com/OpenConceptLab/ocl_issues/issues/1916) | Repo / Share | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1917](https://github.com/OpenConceptLab/ocl_issues/issues/1917) | Unknown | ? | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1918](https://github.com/OpenConceptLab/ocl_issues/issues/1918) | Double-clicking Add Version crashes the page | OPEN | Section 8: Resource Library |
| [#1937](https://github.com/OpenConceptLab/ocl_issues/issues/1937) | Fuzzy search result formatting - slight adjustment | OPEN | Section 8: Resource Library |
| [#1939](https://github.com/OpenConceptLab/ocl_issues/issues/1939) | Sorting when displaying Concepts, Mappings, Repositories,... | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1941](https://github.com/OpenConceptLab/ocl_issues/issues/1941) | Change Recent Activity label | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1942](https://github.com/OpenConceptLab/ocl_issues/issues/1942) | Changes to User Bio on the User Profile page | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1948](https://github.com/OpenConceptLab/ocl_issues/issues/1948) | Repo - Mappings list | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1949](https://github.com/OpenConceptLab/ocl_issues/issues/1949) | Repo - References list | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1950](https://github.com/OpenConceptLab/ocl_issues/issues/1950) | Repo - Summary | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1951](https://github.com/OpenConceptLab/ocl_issues/issues/1951) | Repo - Overview | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1952](https://github.com/OpenConceptLab/ocl_issues/issues/1952) | Detail view: Mappings | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1953](https://github.com/OpenConceptLab/ocl_issues/issues/1953) | Detail view: References | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1955](https://github.com/OpenConceptLab/ocl_issues/issues/1955) | Repo: Versions | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1959](https://github.com/OpenConceptLab/ocl_issues/issues/1959) | Support NPM file upload in bulk import UI | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1966](https://github.com/OpenConceptLab/ocl_issues/issues/1966) | Org/User Page: Pinned repos are shrinking too much | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1967](https://github.com/OpenConceptLab/ocl_issues/issues/1967) | Signed-in Dashboard Empty State not working properly | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#1972](https://github.com/OpenConceptLab/ocl_issues/issues/1972) | Show mappings in same namespace in concept details | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1974](https://github.com/OpenConceptLab/ocl_issues/issues/1974) | Search Results icon for Mapped Concepts | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1975](https://github.com/OpenConceptLab/ocl_issues/issues/1975) | Fix Sort order issues | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1977](https://github.com/OpenConceptLab/ocl_issues/issues/1977) | Author: Repository | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1978](https://github.com/OpenConceptLab/ocl_issues/issues/1978) | CTA / Repository / Create | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#1979](https://github.com/OpenConceptLab/ocl_issues/issues/1979) | CTA / Repository / Edit | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#1980](https://github.com/OpenConceptLab/ocl_issues/issues/1980) | CTA / Repository / Create version | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#1981](https://github.com/OpenConceptLab/ocl_issues/issues/1981) | CTA / Repository / Release (or Un-release) a version | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1982](https://github.com/OpenConceptLab/ocl_issues/issues/1982) | CTA / Repository / Make a copy | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#1983](https://github.com/OpenConceptLab/ocl_issues/issues/1983) | Author: Expansion | OPEN | Section 4: Authoring Experience |
| [#1984](https://github.com/OpenConceptLab/ocl_issues/issues/1984) | CTA / Expansion / Create ($expand) | OPEN | Section 4: Authoring Experience |
| [#1985](https://github.com/OpenConceptLab/ocl_issues/issues/1985) | CTA / Expansion / Set as default | OPEN | Section 4: Authoring Experience |
| [#1986](https://github.com/OpenConceptLab/ocl_issues/issues/1986) | CTA / Expansion / Delete | OPEN | Section 4: Authoring Experience |
| [#1987](https://github.com/OpenConceptLab/ocl_issues/issues/1987) | CTA / Expansion / Update metadata | OPEN | Section 4: Authoring Experience |
| [#1988](https://github.com/OpenConceptLab/ocl_issues/issues/1988) | CTA / Expansion / Rebuild expansion | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#1989](https://github.com/OpenConceptLab/ocl_issues/issues/1989) | Author: Concept | OPEN | Section 4: Authoring Experience |
| [#1991](https://github.com/OpenConceptLab/ocl_issues/issues/1991) | CTA / Concept / Edit | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#1992](https://github.com/OpenConceptLab/ocl_issues/issues/1992) | CTA / Concept / Clone to repo | OPEN | Section 4: Authoring Experience |
| [#1993](https://github.com/OpenConceptLab/ocl_issues/issues/1993) | CTA / Concept / Add to repo | OPEN | Section 4: Authoring Experience |
| [#1994](https://github.com/OpenConceptLab/ocl_issues/issues/1994) | CTA / Concept / Create similar | OPEN | Section 4: Authoring Experience |
| [#1995](https://github.com/OpenConceptLab/ocl_issues/issues/1995) | CTA / Concept / Add mapping | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#1996](https://github.com/OpenConceptLab/ocl_issues/issues/1996) | CTA / Concept / Create mapped concept | OPEN | Section 4: Authoring Experience |
| [#1997](https://github.com/OpenConceptLab/ocl_issues/issues/1997) | CTA / Concept / Retire | OPEN | Section 4: Authoring Experience |
| [#1998](https://github.com/OpenConceptLab/ocl_issues/issues/1998) | CTA / Concept / Import code list | OPEN | Section 4: Authoring Experience |
| [#1999](https://github.com/OpenConceptLab/ocl_issues/issues/1999) | Author: Mapping | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2000](https://github.com/OpenConceptLab/ocl_issues/issues/2000) | CTA / Mapping / Create | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2001](https://github.com/OpenConceptLab/ocl_issues/issues/2001) | CTA / Mapping / Edit | OPEN | Section 4: Authoring Experience |
| [#2002](https://github.com/OpenConceptLab/ocl_issues/issues/2002) | CTA / Mapping / Create similar | OPEN | Section 4: Authoring Experience |
| [#2003](https://github.com/OpenConceptLab/ocl_issues/issues/2003) | CTA / Mapping / Retire | OPEN | Section 4: Authoring Experience |
| [#2004](https://github.com/OpenConceptLab/ocl_issues/issues/2004) | CTA / Mapping / Delete | OPEN | Section 4: Authoring Experience |
| [#2005](https://github.com/OpenConceptLab/ocl_issues/issues/2005) | Add References within Collection | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#2006](https://github.com/OpenConceptLab/ocl_issues/issues/2006) | Add Reference to Collection | OPEN | Section 4: Authoring Experience |
| [#2007](https://github.com/OpenConceptLab/ocl_issues/issues/2007) | CTA / Reference / Preview | OPEN | Section 4: Authoring Experience |
| [#2008](https://github.com/OpenConceptLab/ocl_issues/issues/2008) | CTA / Reference / Delete | OPEN | Section 4: Authoring Experience |
| [#2009](https://github.com/OpenConceptLab/ocl_issues/issues/2009) | CTA / Reference / Create ($expand) | OPEN | Section 4: Authoring Experience |
| [#2011](https://github.com/OpenConceptLab/ocl_issues/issues/2011) | CTA / Reference / Delete | OPEN | Section 4: Authoring Experience |
| [#2012](https://github.com/OpenConceptLab/ocl_issues/issues/2012) | CTA / Reference / Update metadata | OPEN | Section 4: Authoring Experience |
| [#2013](https://github.com/OpenConceptLab/ocl_issues/issues/2013) | Author: Organization | OPEN | Section 4: Authoring Experience |
| [#2014](https://github.com/OpenConceptLab/ocl_issues/issues/2014) | CTA / Organization / Create | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2015](https://github.com/OpenConceptLab/ocl_issues/issues/2015) | CTA / Organization / Add members | OPEN | Section 4: Authoring Experience |
| [#2016](https://github.com/OpenConceptLab/ocl_issues/issues/2016) | CTA / Organization / Edit | OPEN | Section 4: Authoring Experience |
| [#2017](https://github.com/OpenConceptLab/ocl_issues/issues/2017) | CTA / Organization / Create repository | OPEN | Section 4: Authoring Experience |
| [#2018](https://github.com/OpenConceptLab/ocl_issues/issues/2018) | CTA / Organization / Pin repository | OPEN | Section 4: Authoring Experience |
| [#2019](https://github.com/OpenConceptLab/ocl_issues/issues/2019) | CTA / Organization / Customize appearance | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2020](https://github.com/OpenConceptLab/ocl_issues/issues/2020) | CTA / Organization / Edit canonical registry | OPEN | Section 4: Authoring Experience |
| [#2021](https://github.com/OpenConceptLab/ocl_issues/issues/2021) | CTA / Organization / Delete | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2022](https://github.com/OpenConceptLab/ocl_issues/issues/2022) | CTA / Organization / Share | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2025](https://github.com/OpenConceptLab/ocl_issues/issues/2025) | Template / Repository / Source | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#2026](https://github.com/OpenConceptLab/ocl_issues/issues/2026) | Template / Repository / Collection | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#2027](https://github.com/OpenConceptLab/ocl_issues/issues/2027) | Template / Repository / ValueSet | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#2028](https://github.com/OpenConceptLab/ocl_issues/issues/2028) | Template / Repository / CodeSystem | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#2029](https://github.com/OpenConceptLab/ocl_issues/issues/2029) | Template / Repository / Concept Map | OPEN | Section 4: Authoring Experience, Sections 1-2: Executive Summary & Architecture |
| [#2031](https://github.com/OpenConceptLab/ocl_issues/issues/2031) | Author: Searchlight | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#2040](https://github.com/OpenConceptLab/ocl_issues/issues/2040) | Issue import task state update after previous state reque... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2056](https://github.com/OpenConceptLab/ocl_issues/issues/2056) | Concept Details Query Gone Wrong | OPEN | Section 8: Resource Library |
| [#2120](https://github.com/OpenConceptLab/ocl_issues/issues/2120) | Remove alternating table bgcolor in search results | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2121](https://github.com/OpenConceptLab/ocl_issues/issues/2121) | OCL API Throttling | OPEN | Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2123](https://github.com/OpenConceptLab/ocl_issues/issues/2123) | Consumer - View repository references / expansions | OPEN | Section 3: Consumer Experience, Section 8: Resource Library |
| [#2131](https://github.com/OpenConceptLab/ocl_issues/issues/2131) | TBv2 concept details & associations hot fixes | OPEN | Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2132](https://github.com/OpenConceptLab/ocl_issues/issues/2132) | Improve forced sign-in/sign-out behavior | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#2138](https://github.com/OpenConceptLab/ocl_issues/issues/2138) | Implement user sign-in form for django-auth in oclweb3/oc... | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#2145](https://github.com/OpenConceptLab/ocl_issues/issues/2145) | Simple URLs to redirect to SSO for Signin/up | OPEN | Sections 1-2: Executive Summary & Architecture |
| [#2155](https://github.com/OpenConceptLab/ocl_issues/issues/2155) | Repo Creation Starting Point UI | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2156](https://github.com/OpenConceptLab/ocl_issues/issues/2156) | Fork a repo version | OPEN | Section 4: Authoring Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2157](https://github.com/OpenConceptLab/ocl_issues/issues/2157) | Create repo --> Import content from existing repo | OPEN | Section 4: Authoring Experience, Section 8: Resource Library, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2161](https://github.com/OpenConceptLab/ocl_issues/issues/2161) | Importer UI | OPEN | Sections 1-2: Executive Summary & Architecture, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2174](https://github.com/OpenConceptLab/ocl_issues/issues/2174) | Implement source properties/filters in TB search | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2181](https://github.com/OpenConceptLab/ocl_issues/issues/2181) | Repo Comparison MVP Gaps - Mappings and JSON | OPEN | Section 3: Consumer Experience, Sections 5-7: Cross-Cutting, Legacy & Open Questions |
| [#2192](https://github.com/OpenConceptLab/ocl_issues/issues/2192) | Cannot change concept datatype | CLOSED | Section 8: Resource Library |
| [#2252](https://github.com/OpenConceptLab/ocl_issues/issues/2252) | Repo Type Filter in Repos | CLOSED | Section 3: Consumer Experience, Sections 1-2: Executive Summary & Architecture |
| [#2268](https://github.com/OpenConceptLab/ocl_issues/issues/2268) | Remove all possible pathways to creating new resource ver... | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2272](https://github.com/OpenConceptLab/ocl_issues/issues/2272) | Collection's all Expansions API | OPEN | Section 4: Authoring Experience |
| [#2273](https://github.com/OpenConceptLab/ocl_issues/issues/2273) | Implement Collection References tab in TBv3 | OPEN | Section 3: Consumer Experience, Section 4: Authoring Experience |
| [#2275](https://github.com/OpenConceptLab/ocl_issues/issues/2275) | Move repo version metadata to the header and expansions t... | CLOSED | Section 3: Consumer Experience, Section 8: Resource Library |
| [#2282](https://github.com/OpenConceptLab/ocl_issues/issues/2282) | Clean Up Collection References with Cascade Options | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2284](https://github.com/OpenConceptLab/ocl_issues/issues/2284) | Restore/Revert Collection HEAD to Previous Version | OPEN | Section 4: Authoring Experience |
| [#2288](https://github.com/OpenConceptLab/ocl_issues/issues/2288) | Quick update to TBv2 Collection Versions/Expansions Tab | OPEN | Section 4: Authoring Experience, Section 8: Resource Library |
| [#2290](https://github.com/OpenConceptLab/ocl_issues/issues/2290) | Implement history for concept & mapping details in TBv3 | OPEN | Section 3: Consumer Experience, Section 8: Resource Library, Sections 1-2: Executive Summary & Architecture |
| [#2402](https://github.com/OpenConceptLab/ocl_issues/issues/2402) | TBv3 OCL `extras` JSON pretty display - Improve advanced ... | OPEN | Sections 5-7: Cross-Cutting, Legacy & Open Questions |

**Total issues referenced:** 246

---

## Appendix B: Contributor Map

Key contributors to the OCLv3 project, derived from issue assignments and comment activity.

| Contributor | Role | Issues Assigned | Issues Commented | Focus Areas |
|-------------|------|-----------------|------------------|-------------|
| [@paynejd](https://github.com/paynejd) | Product Owner / Technical Lead | 18 | 48 | API, Canonical URLs, Collections, Concepts, Design |
| [@snyaggarwal](https://github.com/snyaggarwal) | Lead Engineer (API + Frontend) | 87 | 68 | API, Collections, Concepts, Design, Mappings |
| [@jamlung-ri](https://github.com/jamlung-ri) | Product Manager / Design Lead | 50 | 88 | API, Collections, Design, Repositories |
| [@paulsonder](https://github.com/paulsonder) | UX/UI Designer | 88 | 55 | Collections, Concepts, Dashboard, Design, Hierarchy |
| [@rkorytkowski](https://github.com/rkorytkowski) | Domain Expert / Legacy Maintainer | 4 | 1 | Canonical URLs, Collections, Concepts, Mappings |
| [@bmamlin](https://github.com/bmamlin) | Domain Expert / Advisor | 1 | 1 | Canonical URLs |

**Additional contributors** (1-2 issues): @karuhanga, @ibacher, @ManojLL, @filiperochalopes
