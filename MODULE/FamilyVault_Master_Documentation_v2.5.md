# FamilyVault — Master Product Documentation

**Version 2.5 — Unified Comprehensive Specification**
**Merged with v3 Technical Content: Database Schemas, Permission Matrix, Event Catalog, Compliance, and Operations**
**Canonical reference for all product, engineering, design, and legal contributors**

> **Version 2.4 Change Note:** This unified document merges v2.2 (feature specifications and narrative) with v3 (technical schemas, compliance, and operations). New content includes: complete database schemas (30 tables), full 13-role × 13-entity permission matrix, 250+ activity log events, GDPR Articles 30/33/35/20 compliance procedures, CCPA/CPRA framework, STRIDE threat model, OWASP Top 10 mitigations, Incident Response Plan (P0-P3), and Business Continuity Plan. All v3 technical content is now integrated inline — no appendices, no separate documents. Changes include: clarification of contradictory routing rules (intra-family disputes, rejection-reason visibility, voice tier status); Memorial Mode timing corrected to activate after the reversal window; Vault Constitution editing permissions clarified; Phase 11 QR exit criterion dependency corrected; WebSocket requirement added to Phase 0 exit criteria; Named tier excluded from MVP until Phase 8; GEDCOM Phase 2 vs Phase 13 scope delineated; formal definitions added for the 72-hour unavailability threshold (UNAVAIL-001), Personal Drive, Mentions Feed, five guided story flows, era_year field schema, Immediate privacy tier, privacy tier inheritance rules, Vault Heir deferral, Acting Family Admin designation, cross-vault search policy, and Biographer payment; Death Witness predecease and no-witness scenarios fully specified; GDPR lawful basis table added to SAR-006; Comments notification category removed (feature undefined); health blocks prohibited in offline mode; Secret Vault heir death-detection logic hardened; hospice B2B use case added; death protocol quiet period notification recipients defined; Book Snapshot vs Ghost Mode distinction clarified; Profile Completion Score denominator behaviour specified; Interview PIN expiry mid-session behaviour defined; simultaneous-death stewardship guidance added; Vault Constitution data model specified; Phase 11 scope-split assessment required; EDGE-003 through EDGE-020 formally defined; SAR-008 formally deprecated; DEATH-002 references Section 6.5.2 as canonical; non-suppressible notification list centralised to M7-SEC; B2B Read Grant rules centralised to B2B-006; print snapshot rules centralised to PRINT-001; HLTH-001 health liberation exclusion centralised; Section 10 "Section 15" stale reference corrected; MCP-001-A 2FA SMS exception added; 18 missing event log entries added including BLOCK_REORDER_UNDONE, PROFILE_PHOTO_UPDATED, BOOK_SNAPSHOT_CREATED, PROFILE_MERGE_INITIATED/COMPLETED/REVERSED, interview review events, LOW_QUALITY_ANSWER_FLAGGED, DECEASED_PROFILE_RIGHTS_DELEGATED, VAULT_HEIR_STEWARDSHIP_DEFERRED, ACTING_FAMILY_ADMIN_DESIGNATED, DEATH_WITNESS_INVALID, DEATH_PROTOCOL_MANUAL_INITIATED, SECRET_VAULT_HEIR_DECEASED_CONFIRMED, VAULT_ESCROW_DEPOSIT_FAILED.

This document is the authoritative, merged synthesis of all three FamilyVault specification documents:
- Document 1: Comprehensive Product Inventory
- Document 2: Build Sequence & Phase Plan
- Document 3: Comprehensive Build Contract

Where any section of this document overlaps with another, the following authority order applies: the Build Contract governs technical standards and legal or security obligations; the Product Inventory governs feature scope and user-facing behaviour; the Build Sequence governs delivery order and phase dependencies.

---

## What Is FamilyVault — A Comprehensive Overview

### The Problem FamilyVault Solves

Every family is, at its core, a story. It is a collection of voices, choices, silences, migrations, losses, triumphs, recipes, wars survived, children named, and wisdom accumulated over generations. That story is not stored in databases or cloud drives. It lives in the memory of a grandmother who learned to cook over an open fire in a village that no longer exists. It lives in the recorded voice of a grandfather who described the smell of his father's workshop to no one in particular on a Tuesday afternoon. It lives in the handwritten letter that a mother left for her daughter, sealed in an envelope, never opened because no one knew it existed when she passed away.

The tragedy of human memory is not merely that individuals forget — it is that when a person dies, an irreplaceable library burns. The details of their childhood, the texture of their marriage, the names of their school friends, the precise fears that shaped their choices, the lessons they learned but never found the moment to teach — all of it vanishes. And unlike the loss of a physical object, this loss is invisible. No one files a report. No one holds a funeral for a story that was never told.

The scale of this loss is staggering. Billions of people alive today have grandparents, great-grandparents, and ancestors whose voices, faces, and inner lives are already beyond reconstruction. The photographs exist, but the person behind the photograph — their character, their humour, their regrets — is gone. Future generations will know that their ancestors existed. They will not know who they were.

FamilyVault exists to solve this problem. It exists to ensure that no family's story is ever again lost to time, to death, to the chaos of scattered documents, and to the simple human failure to ask the right questions while there was still time to receive an answer.

### What FamilyVault Is

FamilyVault is a multi-generational, privacy-first digital platform designed to help families capture, preserve, organise, and pass on their stories, memories, health histories, and legacy documents across unlimited generations. It is, at the most fundamental level, a family memory system — a place where a family's complete story can be built, maintained, and inherited, as a living record that grows richer with every addition and never loses anything that has already been recorded.

The platform is built around the concept of a vault — a single, sovereign digital space owned exclusively by a family and governed entirely by the person they designate as their Vault Keeper. This is not a social media platform, a photo-sharing application, or a genealogy tool in the traditional sense. It is a dedicated, deeply intentional environment built specifically for the act of preserving human memory at its most intimate and consequential level.

FamilyVault operates at the intersection of several powerful ideas: the permanence of the written and spoken word, the intimacy of oral history, the rigour of legal document management, the warmth of collaborative storytelling, and the capability of modern artificial intelligence to make all of it accessible even to people who are elderly, technologically inexperienced, or cognitively impaired. It is designed to be used by a Nigerian grandmother who has never owned a smartphone as comfortably as it is used by a thirty-five-year-old software engineer in Singapore who wants to document their family's history before it disappears.

### The Core Promise to Families

FamilyVault makes four foundational promises to every family that entrusts it with their most precious memories.

The first promise is permanence. Nothing recorded in FamilyVault can ever be accidentally destroyed. The platform is built on an append-only architectural doctrine — every action, every edit, every deletion creates a new layer in the record rather than erasing the layer beneath it. If a story is accidentally deleted, it is recoverable. If a version of a document was edited five years ago, the original is still accessible in its exact original form. If a media file was recorded in 2025, it will still be there in 2075 and, through the platform's open export standard, in 2125 as well. The platform is designed not for the lifespan of a technology company but for the lifespan of a family, which spans centuries.

The second promise is sovereignty. The family owns its data absolutely and permanently. FamilyVault is a custodian, not a proprietor. The Vault Keeper — the person designated to steward the family's vault — has complete and unchallengeable control over every piece of content, every privacy setting, every access decision, and every inheritance instruction. No algorithm, no platform administrator, no advertiser, and no third party can override those decisions except in the specific, legally defined circumstances of Trust and Safety obligations or lawful government orders. The platform does not mine family memories for advertising. It does not profile users for commercial purposes. It does not sell data. The family's story belongs to the family, period.

The third promise is continuity. FamilyVault is built to outlast the technology company that built it. The platform publishes and maintains an open export standard — the FamilyVault Archive Format — which is a fully documented, publicly available format that allows any family to export their entire vault, including all content and all media, in a form that can be read and browsed without any connection to FamilyVault's servers, without any subscription, and without any software other than a standard web browser. Annual escrow deposits ensure that every vault owner can access their data even if FamilyVault ceases to exist tomorrow. Corporate continuity policies are binding commitments, not marketing statements.

The fourth promise is dignity. FamilyVault is designed with the understanding that the content it holds is not casual. It holds final letters written to children who will read them after their parents are gone. It holds health histories that could inform life-or-death medical decisions. It holds stories of trauma, survival, and love that are the most personal material a human being produces. Every design decision — from the way the death protocol is triggered to the way an elderly user is guided through their first voice recording — is made with that gravity in mind.

### How FamilyVault Works

The platform is organised around five primary functions, each of which serves the central goal of preserving and transmitting family memory.

**Story Creation and Capture** is the heart of the platform. FamilyVault provides a rich, structured environment for recording family stories. This includes a guided question bank of one thousand adaptive questions organised across ten categories — childhood and origins, faith and religion, love and marriage, work and purpose, family and lineage, hardship and resilience, world and history, wisdom and legacy, culture and identity, and health and body. These questions are not presented as a survey. They are presented one at a time, at the right moment, in the right context, as a gentle invitation to share a piece of a life.

Answers can be recorded in text, in voice, in video, or as a combination of all three. A grandfather who cannot type can simply speak his answers into his phone; the platform records the audio, transcribes it, and stores both. A daughter who wants to capture not just what her father said but how his voice sounded can do so. A family can upload old photographs, annotate them with names and stories, and anchor those photographs to the specific moments in the text they relate to. Every piece of content is stored not as a flat file but as a structured, versioned, cross-referenced record that can be navigated, searched, and related to other records across the entire vault.

The content system supports a full editorial hierarchy — folders, books, chapters, sections, and blocks — that allows a family to organise their story as a proper narrative work, with the structure, depth, and coherence of a real book, not the fragmentary chaos of a shared folder or a social media timeline.

**The Question Bank and Interview Mode** solve one of the most profound practical problems in oral history: people do not know what to ask, and subjects do not know what to share. The question bank is built from decades of oral history practice and is designed to prompt the kind of rich, reflective, emotionally resonant answers that become priceless with time. The questions are adaptive — the system uses profile data to present relevant questions and suppress irrelevant ones. A widowed person receives questions about spousal loss and remembrance; a person raised in a Nigerian Yoruba household receives questions about clan and lineage and ancestor relationships that would not appear in a system designed only for Western nuclear families.

For elderly users, cognitively impaired subjects, and anyone who finds technology intimidating, FamilyVault provides Interview Mode — a stripped-down, accessible interface that presents one large question at a time, accepts a voice recording with a single tap, and guides the user through their session with no navigation chrome, no menus, and no distractions. An Interviewer — a family member, a care home staff member, a social worker, or a paid biographer — can be assigned to conduct Interview Mode sessions on behalf of a profile subject. The interviewer cannot access any other part of the vault. They see only the current question and the record button. Completed sessions are reviewed by the Vault Keeper before entering the vault, preserving quality and consent.

**The Family Tree and Relationship System** provides the structural backbone of the vault. FamilyVault builds three distinct but interconnected relationship layers: the Family Tree (a navigable visual graph of bloodline relationships), the Social Web (a separate overlay mapping non-blood relationships — mentors, close friends, chosen family), and the Story Graph (automatically generated from name mentions across stories, creating an organic cross-reference map of who appeared in whose life). These three layers can be viewed simultaneously or separately, and they power the AI-assisted features that help surface connections a family might not have noticed.

The family tree is explicitly designed to accommodate the full diversity of human family structures. It supports polygamous relationships, clan-based structures, multi-parent households, extended family networks, and cultural naming conventions from right-to-left scripts to patronymics to clan designations. The platform does not impose a Western nuclear family template. A family from any cultural background — Nigerian, South Asian, Middle Eastern, East Asian, Indigenous, or any other — can represent their actual family structure accurately and completely.

**Privacy and Access Control** give families the granular control they need to share different parts of their story with different people. Every piece of content in the vault has its own privacy tier, ranging from completely private (visible only to the Vault Keeper) to family-tier (visible to confirmed family members) to named viewer lists (visible only to specific individuals) to time-locked (visible only after a specified date or event) to public (shareable as a link to anyone). These tiers are enforced at four separate technical layers — the database query, the service layer, the response serialiser, and the cache — ensuring that a privacy decision is never accidentally violated by a system edge case.

The Secret Vault feature allows the Vault Keeper to store content in a separately encrypted area of the vault, protected by its own encryption key, accessible only to designated individuals. Final letters written to specific family members, sensitive legal documents, private confessions or resolutions that a person wants to be read only after they are gone — all of this can be stored with confidence that it cannot be accessed prematurely, accidentally, or without explicit authorisation.

**The Death Protocol and Legacy System** are what distinguish FamilyVault most profoundly from any other platform. FamilyVault is built with the understanding that the most important moment in a family vault's life is not when it is created but when the person who created it dies. The platform provides a comprehensive, deeply human death protocol that allows a Vault Keeper to pre-designate one or more Death Witnesses — trusted individuals who can report the keeper's death — and one or more Vault Heirs — the designated successors who will receive stewardship of the vault.

When a Death Witness reports a death, the system initiates a carefully managed transition. A reversal window prevents any irreversible action from occurring for 24 hours, ensuring that false triggers cannot cause harm. A quiet period follows, during which the family can grieve without being flooded by system notifications. The Primary Vault Heir receives a private, human-toned message that acknowledges their grief and makes clear there is no urgency. Wills, final letters, and time-locked documents are released on the schedule the deceased Vault Keeper configured. The vault enters Memorial Mode — the deceased person's biography is frozen as a permanent historical record, but family members can continue adding memories and tributes to a separate Memorial Addendum.

The fallback stewardship chain ensures that the vault never becomes ownerless, even if the designated heir is unreachable. A deterministic sequence of successors — from primary heir to backup heirs to designated Family Admins to the longest-serving family members — ensures continuity. If the entire succession chain is exhausted without an acceptance, the vault enters Protected Sealed State, where all content is preserved encrypted and inaccessible to anyone except through a court-approved legal process or a Trust and Safety manual review. Nothing is lost. Nothing is destroyed. The vault waits, intact, for a legitimate successor to be identified.

### The Health Intelligence System

One of FamilyVault's most distinctive and consequential features is the Health Archive — a separately encrypted, access-controlled area within the vault that stores family health history across multiple generations. This is not a medical records system. It is a family health intelligence layer, designed to help living family members understand the health patterns that run through their lineage.

The health archive stores conditions, causes of death, medications, genetic markers, and mental health history across profiles. An AI-powered Health Pattern Engine — operating only on data the requesting user is explicitly authorised to access — analyses this information to identify recurring patterns: conditions that appear across multiple generations, early onset patterns that might be informative for living descendants, genetic markers that appear with unusual frequency in a family line.

Health alerts generated by this engine are always presented with a confidence score, a specific list of the ancestor profiles that informed the pattern, and a mandatory medical disclaimer. The platform never presents health patterns as diagnoses. It presents them as potentially significant family history that a person might want to discuss with their doctor. The difference is important and is enforced at the product level, not just as a terms-of-service caveat.

Health data is protected more rigorously than any other content in the vault. It is stored in a separate encrypted tablespace with its own encryption key. Access requires an explicit health authorisation grant separate from general vault membership. Health data is excluded from Living Memory Liberation — the feature that allows content about deceased individuals to gradually become more visible over time — because health information about a deceased person remains sensitive regardless of when they died. Health data is excluded from all AI analysis for any profile subject who has set a No-AI-Training flag. And health data is excluded from any general export that does not include an explicit health bundle with its own locally unlockable encrypted package.

### The Artificial Intelligence Layer

FamilyVault uses artificial intelligence as a servant of human memory, never as a replacement for it. The platform's AI capabilities are organised into distinct, permission-scoped, transparent layers that assist without overreaching.

The AI Writing Assistance feature helps users who struggle to find the words to express a memory or a story. It offers suggestions, expansions, and style matching — but all AI-generated text is presented in a clearly distinct visual state before acceptance, requires a deliberate explicit click to accept, and is permanently flagged as AI-generated in the content record even after acceptance. There is no pathway by which AI-generated content can enter the vault without the explicit, logged, deliberate consent of the author.

The AI Question Generation feature reads the existing content in a profile and generates contextually appropriate follow-up questions — not from the standard question bank, but specifically derived from what the subject has already said. If a grandmother mentioned working in a textile factory in the 1960s, the AI might suggest: "You mentioned the factory where you worked as a young woman — what was the name of your supervisor, and was she kind to you?" This kind of contextual prompting, impossible to implement in a static question bank, is what turns a guided interview into a genuine conversation.

The AI Pattern Cards feature analyses the content across a family vault and surfaces interesting patterns, connections, and observations — recurring themes across generations, geographic migrations that followed economic patterns, family members who lived in the same city at the same time without knowing it, naming traditions that span five generations. These cards are displayed with confidence scores and evidence citations. Low-confidence observations are clearly marked as speculative. All pattern cards respect the permission system — the AI is shown only what the requesting user is permitted to see, and pattern cards are never generated from or about profiles where the No-AI-Training flag is set.

The AI Transcription feature converts voice recordings to text automatically, using high-accuracy transcription models with a fallback chain for reliability. Transcriptions are presented as drafts for review and correction before being stored. The original audio is always preserved alongside the transcription — the voice, with all its emotion, hesitation, and humanity, is never discarded in favour of text.

The entire AI infrastructure operates through a proprietary Model Context Protocol orchestration layer that abstracts the specific AI providers from the application code. This means the platform can switch AI providers — from Claude to GPT-4o or any future provider — without any change to the application logic, and without any change to the privacy or transparency guarantees. No provider-specific SDK is permitted outside the AI layer. This architectural discipline ensures that no AI provider ever sees data it should not see, and that no privacy commitment is accidentally broken by a provider-specific quirk.

### Who Uses FamilyVault

FamilyVault is designed for an unusually broad range of users, and its architecture deliberately accommodates the full spectrum of human technical ability, cultural background, and life circumstance.

The primary user is any person who cares about preserving their family's story — which is to say, any person at all. A young professional in Lagos who wants to record her grandmother's stories before the grandmother dies. A retired civil servant in Nairobi who wants to write a proper memoir for his children and grandchildren. A family in Houston whose patriarch was a Holocaust survivor and whose stories have never been formally recorded. A second-generation immigrant in London who wants to document the journey that brought their parents from Pakistan to England and why it matters. A family in rural Nigeria who wants to preserve the oral traditions, clan histories, and ancestral records that have been passed down verbally for generations.

Beyond individual families, FamilyVault serves institutional users through its Business-to-Business platform. Funeral homes can create vaults for recently deceased individuals and invite family members to contribute memories, then compile those contributions into a printed memorial book at 30, 60, or 90 days after the death. Elder care facilities can assign vaults to their residents and use Interview Mode to conduct regular oral history sessions — giving residents a meaningful creative project and giving families a record of their loved one's life as it was lived in those final years. Estate law firms can use FamilyVault as a secure repository for wills, letters of wishes, and legal documents, with time-locks that ensure the documents are released only when the death protocol is triggered. Libraries and genealogical societies can use the platform for community oral history projects, with public-tier exports for research use.

The platform's approach to accessibility is particularly important for serving older users. Interview Mode, Cognitive Accessibility Mode, and Large Font Mode are not afterthoughts. They are built as first-class, dedicated interfaces that are tested with real elderly users before every release. The minimum font size in accessibility modes is 20 points. There is no navigation chrome in Interview Mode. The entire interface reduces to one question and three buttons. This simplicity is not a limitation — it is a design achievement, because the most valuable recordings in a family vault are often the ones made by people who would never have been able to navigate a standard digital interface.

### The Cultural Dimension

FamilyVault is explicitly and deliberately built to serve non-Western families as fully as it serves Western ones. This is not a diversity statement. It is a product decision with deep technical implications.

A standard genealogy platform or family memory application is typically designed around the Western nuclear family model — two parents, biological children, linear descent. This model is inadequate for a majority of the world's families. In much of Africa, Asia, the Middle East, and among Indigenous communities globally, family structures are fundamentally different. Polygamous households, extended family networks where grandparents and aunts and uncles are as central as parents, clan-based identity systems where membership in a lineage is a defining social fact, ancestral relationships with deceased family members that are understood as active rather than historical — none of these are well served by a platform built around a Western template.

FamilyVault solves this not by adding a few extra fields but by rethinking the relationship model from the ground up. Relationship types are fully configurable. Multiple co-parent relationships are first-class citizens in the family tree. Clan, lineage, and tribal designations are first-class profile fields. Names support right-to-left scripts, non-Latin characters, and multi-word cultural naming conventions. The question bank is regionally adaptive — questions that are relevant for a Western context but irrelevant or culturally inappropriate for a Yoruba family context are deprioritised, not forced. Elders and ancestors are not required to have a death date to be in the family tree, because many cultures understand ancestors as living presences in the family, not as historical records.

This cultural adaptability extends to calendar systems. The platform supports Gregorian, Hijri, Hebrew, Lunisolar, and other calendar systems for anniversaries and memorial dates. It extends to naming conventions, supporting patronymics, matronymics, clan names, and mononyms. It extends to the default onboarding experience, where profile labels are presented as relationship-neutral options — Parent, Grandparent, Guardian and Elder, Self, Someone Else — rather than the Western-specific Father, Mother, Grandfather, Grandmother labels that assume a particular family structure.

### The Business Model and Institutional Commitments

FamilyVault operates on a freemium subscription model. The free tier is genuinely useful — it supports five profiles, three books, text and photo content, and access to the core question bank categories. This is not a crippled demo. It is a real product that a family can use to start preserving their story today. The paywall is designed to trigger at natural moments of expansion — when a family wants to add a sixth profile, record a video, create a secret vault, or print a book — not as an artificial barrier on the core experience.

Paid plans provide unlimited profiles, all content types including voice and video, advanced AI features, the full question bank including the health category, secret vaults, print credits, and priority access to new features. The Legacy plan, positioned for families with the most demanding preservation needs, provides the full feature set plus enhanced AI capabilities and premium export options.

The platform also supports section-level monetisation, allowing Vault Keepers to set access fees for specific sections of their vault. This enables authors, oral historians, and community memory keepers to earn revenue from their work while maintaining full control over their content. Revenue is split 70% to the Vault Keeper and 30% to FamilyVault, with Stripe handling all payment processing.

The institutional Business-to-Business plan allows funeral homes, elder care facilities, hospice organisations, estate law firms, and other institutions to purchase blocks of vaults and manage them as a portfolio. Institutional pricing scales with volume, and the platform's white-label option allows institutions to display their own logo and colours on the vault interface while maintaining the mandatory FamilyVault attribution that ensures their clients understand where their data is stored and that they always have the right to export it.

### Technical Architecture and Longevity Design

FamilyVault is built on a modern, production-grade technology stack chosen specifically for reliability, performance, and longevity. The frontend is built on Next.js 14 with TypeScript strict mode enforced across the entire codebase. The backend is a Node.js and Fastify API. The primary database is PostgreSQL 16. Search is powered by Typesense. File storage uses Cloudflare R2. Video is handled through Cloudflare Stream. The mobile applications are built in React Native with Expo.

Every architectural decision is made with the understanding that family memories must outlast the technology trends of any particular era. The data is stored in open, documented formats — JSON for structured content, standard media codecs for audio and video, and the FamilyVault Archive Format (FVAF) for complete vault exports. The FVAF specification is publicly documented and open-licensed, so any third party can build a reader. A reference reader is open-sourced and included in every vault export, meaning a family can browse their complete vault archive in any standard web browser with no internet connection required and no FamilyVault infrastructure dependencies.

The platform's security architecture is built around the principle that family memories are among the most sensitive data a person produces. All vault content is encrypted at rest using AES-256-GCM with per-vault keys. Health data uses separate per-profile encryption keys. Secret Vault content uses a third, separate encryption key stored in a secrets manager, never in the database. All data in transit uses TLS 1.3. IP addresses are never stored in plain text — they are stored as SHA-256 hashes with a per-vault salt, preventing reconstruction while still allowing security anomaly detection. The platform undergoes quarterly external penetration testing, daily automated vulnerability scanning, and a comprehensive access control testing regime that covers all 13 user roles against every entity type in the system.

The immutability architecture is the platform's most distinctive technical feature. The entire system is built on the principle that no content can ever be hard-deleted. Every write operation creates a new version node. Every deletion creates a branch in the version chain rather than removing the content. The original is always recoverable. Every version of every document is preserved in its exact original state, with a SHA-256 cryptographic hash that allows verification of authenticity. Ghost Mode — the ability to reconstruct any document, profile, or content block exactly as it appeared at any historical timestamp — is not a debugging feature. It is a first-class product capability, available to users to review the complete history of any piece of content in their vault.

### The Organisational Commitment

FamilyVault's founders understand that they are asking families to make a profound act of trust. They are asking people to record their most intimate memories, their most sensitive health information, their most private relationships, and their most consequential legal documents, and to hand custody of all of it to a software company. That trust cannot be justified by a privacy policy alone. It requires structural, legally binding commitments.

FamilyVault commits to depositing a complete, encrypted, self-contained export of every vault with an independent escrow agent annually. If the company ceases operations, the escrow agent releases exports to vault owners within 30 days. The company commits to providing a minimum of 90 days advance notice and a full data export before any shutdown. The company commits to ensuring that any acquiring company signs a Vault Data Commitment Agreement that binds them to honour all existing privacy tiers, ownership rights, and export commitments for a minimum of 10 years post-acquisition. The open export standard ensures that no family is ever locked in. A family can export their complete vault at any time, in a format that any future technology can read.

These are not aspirational statements. They are binding product rules, technical obligations, and in many cases contractual commitments. They are specified in this document with the same rigour and enforceability as the security and privacy rules. They are reviewed by legal counsel annually. They are the foundation on which everything else is built — because a platform that asks families to trust it with their most precious memories has a moral obligation to honour that trust not just today but across generations, which is, after all, exactly the span of time that FamilyVault exists to serve.

---

## Table of Contents

1. [Platform Philosophy & Core Principles](#1-platform-philosophy--core-principles)
2. [User Roles & Permission Matrix](#2-user-roles--permission-matrix)
3. [Content Architecture & Data Models](#3-content-architecture--data-models)
4. [Data Models & Database Schema](#4-data-models--database-schema)
5. [Configuration Management](#5-configuration-management)
6. [Complete Feature Specifications](#6-complete-feature-specifications)
7. [Technology Stack](#7-technology-stack)
8. [Build Phases — Phase 0 through Phase 20](#8-build-phases)
9. [Activity Log Event Types](#9-activity-log-event-types)
10. [Build Contract — All Rules](#10-build-contract)
11. [Data Retention Schedule](#11-data-retention-schedule)
12. [Master Acceptance Criteria](#12-master-acceptance-criteria)

---

## 1. Platform Philosophy & Core Principles

### 1.1 The Immutable Truth Doctrine

FamilyVault operates under a single, non-negotiable architectural principle: records are append-only and content is never destructively removed except where a legally required erasure obligation expressly overrides normal immutability rules. Every action taken inside the vault is permanent in the sense that its record cannot be erased. Deletion, editing, moving, and archiving are all additive operations — they create new states rather than removing old ones. This doctrine governs every database schema, every API design, every user interface decision, and every business rule throughout the platform.

- Every content block has an append-only history chain. No version is ever overwritten.
- Every action — read, write, edit, delete, move, share, lock, unlock — produces a permanent log entry.
- Deletion is a branch event, not a destructive event. The deleted item remains in the Branch Archive.
- Editing creates a diff layer — the original text is preserved, and the new text is stored alongside it.
- Every log entry records: who, what, when (to the millisecond), where (exact content address), why (reason field required for all non-CREATE actions per IMM-C-004), from what device, from what SHA-256 hashed IP address (per SEC-008), and in what session.
- The audit log itself is immutable — no user, including the Vault Keeper, can delete log entries.
- Time-zone aware timestamps are stored in UTC with local time displayed per user preference.

### 1.2 The Family Sovereignty Principle

Families own their data completely and permanently. FamilyVault is a custodian, not an owner. The Vault Keeper has supreme control over every aspect of the vault, and that control can be transferred, inherited, and legally documented. No platform action may override Vault Keeper decisions except for legal compliance and defined Trust and Safety interventions required by platform safety policy.

### 1.3 Privacy-First by Architecture

All content is private by default. Sharing requires explicit, deliberate action. Every piece of content — down to an individual sentence block — has its own privacy tier. The platform never assumes visibility; it requires grants.

### 1.4 Generational Durability Guarantee

FamilyVault content must survive 100 or more years of technological change. All data is stored in open, documented formats — JSON, plain text, and standard media codecs. There is no proprietary lock-in. Data exports are always available. The vault must be transferable, printable, and readable without the application itself.

### 1.5 Subject Rights Doctrine

Every living person whose likeness, name, story, or health data appears in a vault has rights that the platform must honour. This is not merely a legal obligation — it is a core ethical commitment.

- Any living person named in a vault may submit a Subject Access Request to receive a copy of all content referencing them.
- Any living person may request correction of factually incorrect content referencing them. The Vault Keeper reviews and decides; the request and decision are permanently logged regardless of outcome.
- Any living person may request removal of content referencing them from shared or public tiers. The Vault Keeper decides; if they decline, the subject is informed of their right to escalate to their regional data authority.
- Profile subjects who have claimed their profile may set a No-AI-Training flag — their content is excluded from any model fine-tuning or AI pattern analysis.
- Deceased profile subjects have rights inherited by their designated next-of-kin hierarchy. The Vault Keeper must maintain an ordered next-of-kin chain for each deceased profile.
- Minor protections are tiered. Under-13 profiles require verifiable guardian consent before collection beyond minimal necessary metadata, and may not be processed by AI or placed in shared or public tiers without that consent. Ages 13 through 17 receive enhanced protections for imagery, health data, and public or shared publication.

### 1.6 Long-Term Stewardship Guarantee

FamilyVault acknowledges that families are being asked to trust their most precious memories to a software company. This trust must be backed by concrete, legally binding stewardship commitments.

- **Data Escrow**: A complete, encrypted, self-contained export of every vault is deposited with an independent escrow agent annually. If FamilyVault ceases operations, the escrow agent releases the exports to vault owners within 30 days.
- **Open Export Standard**: FamilyVault publishes and maintains a fully documented open export format called the FamilyVault Archive Format (FVAF). Any third party may build a reader without a licence.
- **Continuity Plan**: FamilyVault maintains a corporate continuity policy. In the event of acquisition, the acquirer must honour all existing vault privacy tiers and ownership rights for a minimum of 10 years.
- **90-Day Wind-Down Notice**: If FamilyVault decides to shut down, all paying customers receive a minimum of 90 days advance notice and a full export of their vault before shutdown.
- **Vault Backup Transparency Dashboard**: Every vault owner can see the last backup date, number of backup copies, geographic locations, last restore-test result, and data integrity score. This dashboard is updated daily.

### 1.7 Non-Western Family Structure Doctrine

FamilyVault must be as useful for a Nigerian Yoruba family with polygamous family structures as for a Western nuclear family. The data model and user interface must never assume a single cultural template for family relationships.

- Relationship types are fully configurable — no hard-coded Mother or Father as the only parent options.
- Multiple co-parent relationships are first-class citizens in the family tree.
- Clan, lineage, and tribal designations are supported as first-class profile fields.
- Names support right-to-left scripts, non-Latin characters, and multi-word cultural naming conventions including patronymics, matronymics, and clan names.
- The question bank is regionally adaptive — questions irrelevant to a selected cultural context are deprioritised rather than removed.
- Elders and ancestors are not required to have a death date to be in the tree — many cultures consider ancestors active members of the family.

---

## 2. User Roles & Permission Matrix

### 2.1 Role Definitions

FamilyVault operates with exactly 13 defined roles. The canonical role set is fixed. Interim Authority is transient but must be represented in access design and testing.

| Role | Description | Scope | Maximum Count |
|------|-------------|-------|---------------|
| **Vault Keeper** | Supreme owner. Full control over all settings, content, access, monetisation, and transfer. | Entire vault | 1 (transferable) |
| **Vault Heir** | Designated successor. Ordered heirs receive Vault Keeper access only after triggered inheritance and stewardship acceptance. | Entire vault (post-transfer) | 1 primary plus up to 2 ordered backups |
| **Family Admin** | Trusted family member with elevated access. Can manage profiles, approve edits, and invite members. | Vault-wide minus private sections | Up to 5 |
| **Verified Member** | Confirmed family member. Can view all family-tier content and contribute to open sections. | Family-tier content | Unlimited |
| **Contributor** | Invited to collaborate on specific sections or books. Scoped, time-limited access. | Defined sections only | Unlimited |
| **Reader** | Can read assigned content only. Cannot edit, comment, or download. | Defined sections only | Unlimited |
| **Guest** | Temporary access to a specific public or shared link. No account required. | Specific shared link | Unlimited |
| **Interviewer** | Assigned to conduct guided interviews of a specific profile. Cannot read other content. | Interview tool only, per profile | Per profile |
| **Death Witness** | Designated to trigger the post-death protocol. One-action role. | Trigger only | 1 to 2 |
| **Transcriber** | Can listen to audio or video and submit transcriptions. Cannot read surrounding content. | Media and transcription only, per item | Per item |
| **Biographer** | Paid external collaborator. Full read of assigned book, contribution rights, no admin access. | Assigned book only | Per book |
| **B2B Partner Admin** | Institutional partner managing assigned client vaults only. Cannot cross-access between clients. | Assigned client vaults | Per institution |
| **Interim Authority** | Temporary, system-assigned stewardship role used during protected custody when no heir has yet accepted. May keep the vault operational but may not transfer ownership or alter payout or escrow authority. | Operational vault stewardship only | 0 or 1 temporary |

### 2.2 Role-Specific Access Boundaries

**Vault Heir (pre-trigger):** Pre-trigger Vault Heir permissions are limited to designation visibility, contact verification, and later accept, defer, or decline stewardship. Full Vault Keeper powers begin only after formal stewardship acceptance.

**Vault Heir deferral — operational definition:** A Vault Heir who chooses to "defer" stewardship is indicating they are not ready to accept now but have not declined permanently. Deferral has the following operational rules:
- The vault continues operating normally under Interim Authority during the deferral period.
- The deferring heir retains their position in the stewardship chain — they are not skipped.
- A deferred heir must provide a deferral duration when deferring: 7 days, 14 days, or 30 days. No open-ended deferral is permitted.
- At the end of the deferral period, the heir receives a second notification prompting them to accept, defer again (maximum one additional deferral), or decline.
- A single heir may defer a maximum of two times. On the third notification, only Accept or Decline are available. If no response is received within 30 days of the third notification, the heir is treated as having timed out and the next heir in the chain is contacted.
- Deferral does not grant the heir any temporary access to vault content.
- All deferral events are logged as VAULT_HEIR_STEWARDSHIP_DEFERRED with the deferral duration and deferral count.

**Guest:** Access is limited to the specific shared link or reunion-scope content granted. Guests cannot search the wider vault, export, print, or browse the audit log.

**Interviewer:** Access is limited to the assigned interview session, assigned question set, recording and upload controls, and session status. Interviewers cannot browse general vault content or surrounding profile narrative.

**Death Witness:** Access is limited to initiating and confirming the death reporting flow. Death Witnesses do not gain operational vault permissions by reporting a death.

**Transcriber:** Access is limited to the isolated media asset, timing markers, speaker labels, and transcription submission and review notes. Transcribers cannot read surrounding chapters, profile books, or unrelated metadata beyond what is needed to produce an accurate transcript.

**Biographer:** Access is limited to assigned books, assigned media within those books, contribution tools, and revision history for the assigned scope. Biographers cannot manage members, change monetisation, or access private or locked content outside assignment.

**Biographer payment:** Payment between the Vault Keeper and a Biographer is entirely outside the FamilyVault platform in V1. FamilyVault does not intermediate, process, or track Biographer compensation. The Vault Keeper arranges payment directly with the Biographer (via bank transfer, PayPal, invoice, or any other mechanism they agree on). The Biographer's access grant is time-limited (set by the Vault Keeper at invitation — minimum 1 week, maximum 12 months, renewable). Revoking a Biographer's access does not affect any external payment arrangement. A post-V1 Biographer marketplace feature (allowing Vault Keepers to hire and pay verified Biographers through the platform) is a candidate for a future release and must be fully specified, including payment processing, escrow, dispute resolution, and vetting, before implementation.

**B2B Partner Admin:** Access is limited to assigned client vault administration, provisioning, status review, billing-related metadata, and institution-specific workflows. No cross-client vault browsing is permitted.

**Interim Authority:** May approve pending contributions, resolve non-private moderation queues, maintain stewardship operations, and keep the vault safe during the protected state. Interim Authority may not transfer ownership, alter payout accounts, decrypt secret vault material beyond already-authorised operational scope, change export recipients, or rewrite the vault constitution.

### 2.3 Role Change and Demotion Rules

The Vault Keeper may demote a Family Admin to Verified Member or lower at any time. On demotion:

- The demoted user's access is immediately reduced to their new role scope.
- Content they created retains their name as author but is now governed by the Vault Keeper's approval for any future edits.
- Pending actions — pending contribution approvals and pending invitations they sent — are transferred to the Vault Keeper's queue.
- The demotion is logged as ROLE_CHANGED with the required reason field.
- The demoted user receives a notification of their role change.

### 2.4 Multi-Vault Membership

A user may hold different roles in multiple vaults simultaneously. The following rules apply:

- Login lands the user on their default vault — the vault where they are Vault Keeper, or their most recently active vault if they have no keeper role. A vault switcher is accessible from the top navigation on all screens.
- Notification digests are per-vault by default. A user with membership in three vaults receives three separate digests unless they enable Combined Digest in cross-vault notification settings.
- A profile claimed in multiple vaults via VaultLink has one canonical identity in the owner vault. Contributions and biographies in other vaults are federated overlays, not separate identities.
- The user's account settings — email, password, two-factor authentication, and passkey — are shared across all vault memberships. Role permissions are per-vault and do not cross-pollinate.
- **Cross-vault search:** There is no unified cross-vault search in V1. Search (powered by Typesense) is scoped per vault. A user active in multiple vaults must search each vault separately using the per-vault search bar. The vault switcher in the top navigation allows moving between vaults to perform separate searches. A cross-vault unified search surface is a post-V1 consideration and must be specified separately if introduced, including data isolation requirements, permission scoping across vaults, and Typesense index architecture.

### 2.5 Core Permission Matrix

The platform implements a comprehensive 13-role permission system across 13 entity types. This section provides the summary matrix; complete per-entity matrices follow.

**Matrix Legend:** ✓ = Full | ○ = Scoped/Limited | △ = Conditional | ✗ = None

#### 2.5.1 Vault-Level Permissions

| Action | Keeper | Heir(Pre) | Heir(Post) | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|-----------|------------|----------|--------|-------|--------|-------|---------|
| View vault metadata | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Edit vault settings | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ○ |
| Delete vault | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Export vault data | ✓ | ✗ | ✓ | △ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Manage members | ✓ | ✗ | ✓ | ○ | ✗ | ✗ | ✗ | ✗ | ○ |
| View audit log | ✓ | ✗ | ✓ | Partial | Own | ✗ | ✗ | ✗ | ✓ |
| Configure death protocol | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Configure monetization | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Configure AI providers | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Manage B2B integrations | ✓ | ✗ | ✓ | △ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Set vault constitution | ✓ | ✗ | ✓ | ✓ | Suggest | ✗ | ✗ | ✗ | ✓ |

#### 2.5.2 Content (Block) Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|----------|--------|-------|--------|-------|---------|
| View block | ✓ | ✓ | ✓ | ○ | ○ | ○ | ✓ |
| Create block | ✓ | ✓ | ✓ | ○ | ✗ | ✗ | ✓ |
| Edit block | ✓ | ✓ | ○ (own) | ○ (own) | ✗ | ✗ | ✓ |
| Delete (branch) block | ✓ | ✓ (reason) | ○ (own) | ✗ | ✗ | ✗ | ✓ |
| Restore block | ✓ | ✓ | ○ (own) | ✗ | ✗ | ✗ | ✓ |
| Change privacy tier | ✓ | △ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Lock/unlock block | ✓ | △ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Pin/unpin block | ✓ | ✓ | Own | ✗ | ✗ | ✗ | ✓ |
| Enable AI assistance | ✓ | ✓ | Own | ✗ | ✗ | ✗ | ✓ |
| Create template block | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Instantiate template block | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Delete (branch) template block | ✓ | ✓ (reason) | Own | ✗ | ✗ | ✗ | ✓ |

#### 2.5.3 Profile Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Interviewer | Biographer | Interim |
|--------|--------|----------|--------|-------|--------|-------------|------------|---------|
| View profile | ✓ | ✓ | ✓ | ○ | ○ | Target | Target | ✓ |
| Create profile | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Edit profile | ✓ | Assigned | Own | Scoped | ✗ | ✗ | ✗ | ✓ |
| Edit health data | ✓ | With auth | Own auth | ✗ | ✗ | ✗ | ✗ | ✗ |
| Merge profiles | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Set deceased status | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Configure interview mode | ✓ | △ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Claim profile | ✓ | ✓ | Subject | Subject | ✗ | ✗ | ✗ | ✓ |

#### 2.5.4 Book & Folder Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Biographer | Interim |
|--------|--------|----------|--------|-------|------------|---------|
| Create book/folder | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Edit metadata | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Delete (branch) | ✓ | With reason | ✗ | ✗ | ✗ | ✓ |
| Set co-authors | ✓ | ✓ | ✗ | ✗ | Book only | ✓ |
| Create snapshot | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ |
| Lock folder | ✓ | △ | ✗ | ✗ | ✗ | ✓ |

#### 2.5.5 Special Role Permissions

**Death Witness:**
- Can trigger death protocol (with second witness if configured)
- Cannot view vault content beyond confirmation of keeper identity
- Cannot edit anything
- Access expires after protocol completion or reversal

**Transcriber:**
- Can view assigned media assets
- Can submit transcripts for approval
- Cannot view other vault content
- Cannot edit media metadata

**Secret Vault Heir:**
- Pre-death: Can view designation status, update contact verification
- Pre-death: Can accept/defer/decline stewardship (after death protocol)
- Pre-death: NO operational vault access
- Post-acceptance: Full Vault Keeper privileges
- Secret Vault access: Independent of stewardship status

**B2B Partner Admin:**
- Can manage assigned institutional vaults only
- Row-level security enforced at database
- Cannot access vaults outside assigned portfolio
- Can assign/remove institutional users within portfolio

#### 2.5.6 Time Lock Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|----------|--------|-------|--------|-------|---------|
| Create time lock | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| View time lock status | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Modify time lock condition | ✓ | With reason | ✗ | ✗ | ✗ | ✗ | ✓ |
| Revoke time lock | ✓ | With reason | ✗ | ✗ | ✗ | ✗ | ✓ |
| Release conditional lock | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |

#### 2.5.7 Memorial Addenda Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|----------|--------|-------|--------|-------|---------|
| Submit memorial addendum | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ |
| View memorial addendum | ✓ | ✓ | ✓ | ○ | ○ | ✗ | ✓ |
| Approve/reject memorial addendum | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Delete (branch) memorial addendum | ✓ | With reason | ✗ | Own | ✗ | ✗ | ✓ |

#### 2.5.8 Family Reunion QR Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|----------|--------|-------|--------|-------|---------|
| Create reunion QR code | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| View QR code status | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Revoke reunion QR code | ✓ | With reason | ✗ | ✗ | ✗ | ✗ | ✓ |
| Scan QR code (Guest) | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ |

#### 2.5.9 Functional Role Limitations

| Role | Can Read | Can Write | Can Admin | Time-Limited |
|------|----------|-----------|-----------|--------------|
| Interviewer | Target profile only | Interview transcripts only | No | Per-session |
| Transcriber | Assigned media only | Transcripts only | No | Per-asset |
| Biographer | Assigned book only | Book content only | No | Per-book |
| Guest | Shared content only | Nothing | No | Per-link |
| Contributor | Scoped content | Contributions for review | No | Invitation duration |

#### 2.5.10 Complete Permission Matrix Summary

The full implementation must test all 13 roles against each entity type:

| Entity Type | 13 Roles Tested |
|-------------|-----------------|
| Vault | Keeper, Heir(Pre), Heir(Post), FamAdmin, VrfMem, Contr, Reader, Guest, Interviewer, Witness, Transcriber, Biographer, Interim |
| Profile | All 13 roles |
| Folder | All 13 roles |
| Book | All 13 roles |
| Chapter | All 13 roles |
| Block | All 13 roles |
| Asset | All 13 roles |
| Health Record | Keeper, FamAdmin, VrfMem (own only) |
| Shared Link | Keeper, FamAdmin, VrfMem (own shares) |
| Webhook Subscription | Keeper only |
| Export | Keeper, Heir(Post), FamAdmin, Interim |
| B2B Read Grant | Keeper, FamAdmin, B2B Admin |
| Audit Log | Keeper (full), Heir(Post) (full), FamAdmin (partial), Others (own only) |

**Note on Test Coverage:** Integration tests must verify: correct access for permitted roles; correct 401 for unauthenticated; correct 403 for authenticated but forbidden; correct 404-style response for locked or existence-sensitive content (no distinction between "locked" and "does not exist").

---

#### 2.5.11 Question Bank Permissions

| Action | Keeper | FamAdmin | VrfMem | Contr | Reader | Guest | Interim |
|--------|--------|----------|--------|-------|--------|-------|---------|
| View system questions | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ |
| Create personal question | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Create vault question | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Create personal category | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Create vault category | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Override (edit) system question | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Delete (branch) own question | ✓ | ✓ | Own | ✗ | ✗ | ✗ | ✓ |
| Delete (branch) vault question | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Move question between categories | ✓ | ✓ | Own | ✗ | ✗ | ✗ | ✓ |
| Generate / Download Question Book | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Order Question Book physical print | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |


## 3. Content Architecture & Data Models

### 3.1 The Content Hierarchy

All content is organised in a strict hierarchy:

**Vault → Folder → Book → Chapter → Section → Block → Line → Character Position**

Every level has its own: identifier, privacy tier, creation log, edit history, branch archive, and access control list.

### 3.1.1 Vault

- One per account. The root container for all family content.
- Has a unique, permanent Vault ID that never changes, even after ownership transfer.
- Contains all folders, books, profiles, and the master audit log.
- Has a **Vault Constitution** — a master document co-authored by family members. The Vault Constitution is a statement of intent and family values, not a technically enforced ruleset. Violations are resolved by the Vault Keeper, not by automated enforcement. **Editing rights:** Only the Vault Keeper and Family Admins may directly write to the Vault Constitution (consistent with the permission matrix in Section 2.5). Verified Members and Contributors may *propose* amendments by submitting a Constitution Contribution via the standard contribution review workflow; the Vault Keeper or a Family Admin must approve the proposal before it is incorporated. The CONSTITUTION_UPDATED event is logged when an approved edit is committed. The phrase "co-authored by family members" refers to this proposal-and-approval contribution model, not to unrestricted direct editing rights.
- Has a Health Archive folder (auto-generated, AI-maintained).
- Has a Deleted Archive (system folder, always present, never user-deletable).
- Has a Pending Profiles folder (profiles created by mention, not yet claimed).
- Has a Subject Rights Queue — all outstanding Subject Access Requests and correction requests managed here.
- Has a Backup Status panel — last backup timestamp, integrity score, and restore-test status.

**Vault Renaming:** The vault display name can be changed at any time by the Vault Keeper. The change is logged as VAULT_RENAMED. The underlying Vault ID never changes.

### 3.1.2 Folder

- Container for books. Can be nested one level deep.
- Has name, description, cover image, privacy tier, creation date, and created-by fields.
- Can be locked — all books inside become inaccessible simultaneously.
- Can be shared as a unit — share a folder, and recipients see only their permitted books.
- Can be printed as a unit — all permitted books exported together.
- Supports ordering — the user can manually reorder folders and books within folders.

### 3.1.3 Book

- The primary storytelling unit. A book is one person's story, one topic, or one collection.
- Has: title, subtitle, cover image or colour, authors, date range (era), chapter list, and privacy tier.
- Has a spine — a summary paragraph shown on the shelf view.
- Supports co-authorship — multiple profiles credited as authors.
- Can be version-snapshotted — a point-in-time read-only copy for archiving or printing. **Relationship to Ghost Mode and the version chain:** A Book Snapshot is a user-initiated, explicitly named artefact. It is distinct from Ghost Mode reconstruction (which is an always-available, on-demand historical reconstruction from the version chain) in the following ways: (1) A Book Snapshot has a user-assigned name and can be shared as a read-only link with specific recipients; (2) A Book Snapshot is a materialised copy — it renders and stores the complete readable state of the book at snapshot time, making it independently accessible without replaying the version chain; (3) Ghost Mode is for internal historical review and verification — it requires vault access and is not shareable externally. When a user wants to "archive a version for the record" or "send a frozen copy to a family member for review," they use a Book Snapshot. When a user wants to "see what this book looked like in January 2026," they use Ghost Mode. Print orders use the same snapshot mechanism as Book Snapshots (stored as PRINT_VERSION_SNAPSHOT). General Book Snapshots are stored as BOOK_SNAPSHOT records with their own event log entry (see Section 9).
- Has a completion percentage — tracks answered versus unanswered questions.
- Has a reading-time estimate — computed from total word count.

### 3.1.4 Chapter, Section, and Block

The canonical content structure includes the following block types:

| Block Type | Description |
|------------|-------------|
| Text | Rich text with bold, italic, headings, inline links, and footnotes |
| Photo with Caption | Image with caption text and alt text |
| Question and Answer | A question from the question bank with the author's answer |
| Timeline Event | A dated event on the era timeline |
| Document | Generic uploaded document |
| Legal Document | Will, deed, or letter of wishes — encrypted separately |
| Link | External hyperlink with preview |
| Quote | A quoted passage with attribution |
| Cross-Reference | A reference to another block, profile, or book |
| Voice | Audio recording with waveform and optional transcript |
| Video | Video recording or upload with transcript |
| Mixed | A composition of text, voice, video, and photo in one block |
| Interview Transcript | Records an Interviewer session with timestamps per question |
| Memory Contribution | Submitted by Verified Members for Vault Keeper approval |
| Witness Statement | Co-signed by multiple profiles, cryptographically logged |
| B2B-Authored Content | Content authored by an institutional partner. Watermarked. |

**Template Blocks**

Any block type can be marked as a template by setting `is_template = TRUE`. Template blocks behave as follows:

- **Creation:** Vault Keepers and Family Admins can create template blocks from any existing block or from scratch
- **Privacy:** Template blocks default to `private` tier and are only visible to Vault Keepers and Family Admins (the creator plus other VK/FA)
- **Storage:** Template blocks are stored in the `blocks` table with `section_id = NULL` (not assigned to any section)
- **Version chain on instantiation:** When a template block is copied into book content, the copy receives a new `version_chain_id`; the original template retains its own chain
- **UI Location:** Template blocks appear in a Template Library UI accessible during book editing (when adding blocks to a section)
- **Permissions:** Template blocks can be edited or deleted only by their creator, Vault Keepers, and Family Admins

### 3.1.5 Version Chain Snapshot Model

Each content entity (block, section, chapter, book, profile) maintains a VersionChain. Each write produces a VersionNode storing:

- `actor_user_id`
- `timestamp_utc` (millisecond precision)
- `action_type`
- `device_fingerprint`
- `session_id`
- `integrity_hash` (SHA-256 computed over a canonical JSON serialisation of the complete entity state)

The `content_snapshot` is a canonical JSON representation of the complete entity state at write time, serialised using RFC 8785 Canonical JSON (JCS) — keys sorted lexicographically, Unicode normalised, numbers deterministic. The integrity hash is SHA-256 over that canonical serialisation. Non-RFC-8785 serialisation is a critical bug. Ghost Mode reconstructs any entity at any historical timestamp by replaying version nodes, and output is verified against the stored integrity hash.

**Content snapshot contents by block type:**
- **Text blocks:** The normalised rich-text tree, metadata, privacy level, anchors, and referenced asset identifiers.
- **Photo blocks:** Asset key, immutable checksum, caption, alt text, anchor metadata, and display settings.
- **Voice and video blocks:** Asset keys, immutable checksums, transcript text, speaker labels, caption or summary metadata, moderation state, and media-specific settings.
- **Mixed blocks:** The full ordered composition of text and media children.
- **Witness Statement and Interview Transcript blocks:** Question timestamps, speaker attribution, signatory metadata where applicable, and transcript state.

### 3.1.6 Orphaned Media Archive

When an inline media anchor's surrounding text is deleted, the asset is not deleted. It moves to the Orphaned Media Archive — a first-class, read-only, permanently retained archive accessible to Vault Keepers and Family Admins. It shows:

- Original content address (block identifier, line number, character offset)
- Original context (50 characters before and after the insertion point)
- Orphaning reason, orphaning actor, and timestamp
- Restore options

Retention is permanent unless superseded by a legally required erasure event.

### 3.1.7 Shared Profiles and VaultLinks

A shared profile has exactly one `owner_vault_id`. Core identity fields — name, birth and death dates, canonical relationships, and primary photo — are writable only in the owner vault. Linked vaults store local overlays (display preferences, local annotations) but may not diverge the canonical identity record.

VaultLink creation is owner-led: one vault creates the canonical profile, issues a link invitation, the second vault accepts, and the second vault receives a read-only federated profile reference plus any locally permitted overlays.

If the owner vault becomes dormant or has its sharing authority revoked, linked vaults receive a VAULT_LINK_SUSPENDED or VAULT_LINK_REVOKED notification. Silent broken references are forbidden.

### 3.1.8 Secret Vault

A Secret Vault is a separately encrypted storage area within the vault, protected by its own per-vault secret-storage key held in the secrets manager. The Vault Keeper designates a secret-vault heir (`secret_vault_heir_profile_id`) who receives access upon the death protocol.

Without the correct key, Secret Vault contents are indistinguishable from non-existent content in all API responses (returns a 404-equivalent). Secret Vault content is excluded from general exports unless the requesting user holds the appropriate key.

**Secret Vault Heir Lifecycle on Stewardship Transfer:**

- Upon death protocol trigger, the designated Secret Vault heir retains access independently of who assumes general vault stewardship. The Secret Vault heir's access key is not revoked by a stewardship transfer — these are two separate designations.
- The new Vault Keeper cannot revoke Secret Vault heir access unless they also hold the Secret Vault key. The Secret Vault heir's access persists until: the heir explicitly surrenders access, a court order requires it, or the heir predeceases the Vault Keeper.
- If the Secret Vault heir has predeceased the Vault Keeper, the Secret Vault enters a Protected Sealed State at the time the death protocol resolves. The new steward is informed that Secret Vault content exists but is sealed. Access is only available via legal proceedings to retrieve the encrypted key material from escrow.

  **Important — heir deceased-status verification:** Detecting that a Secret Vault heir has predeceased the Vault Keeper requires a reliable real-world death signal, not merely a vault profile marked as deceased (which may have been set for editorial reasons unrelated to an actual death). The system uses the following hierarchy to establish heir pre-death with sufficient confidence: (1) the heir's own vault has had a confirmed death protocol triggered (DEATH_PROTOCOL_TRIGGERED event on the heir's own vault), or (2) the heir's profile in this vault has been marked deceased AND has a documented death date set AND the death date is more than 30 days in the past, or (3) the Vault Keeper explicitly confirms the heir's death during the Secret Vault heir configuration screen. In cases (2) and (3), the system logs the basis for the deceased determination as SECRET_VAULT_HEIR_DECEASED_CONFIRMED. If none of these conditions are met at the time the death protocol resolves, the system treats the heir as potentially reachable and attempts contact via the stored email address before entering Protected Sealed State.
- The Vault Keeper should always designate at least one backup Secret Vault heir. If no valid heir exists at succession, the Secret Vault content is preserved encrypted but inaccessible until a court-approved heir process is completed.

### 3.2 Profile Data Model

Profiles include all standard biographical fields plus the following extended fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cultural_context` | Enum + String | No | Primary cultural framework: WESTERN_NUCLEAR / AFRICAN_EXTENDED / SOUTH_ASIAN / EAST_ASIAN / MIDDLE_EASTERN / INDIGENOUS / OTHER (custom) |
| `lineage_clan_name` | String (200) | No | Clan, lineage, or tribal designation |
| `naming_convention` | Enum | No | PATRONYMIC / MATRONYMIC / CLAN_FIRST / GIVEN_FAMILY / MONONYM / OTHER |
| `subject_rights_email` | String (200) | No | Contact email for Subject Access Request notifications to the person referenced by this profile |
| `sar_opt_in` | Boolean | Yes | Whether the profile subject has been notified of their Subject Access Request rights |
| `no_ai_training_flag` | Boolean | Yes | Whether this profile's content is excluded from AI training and pattern analysis |
| `guardian_profile_id` | UUID | No | For minors: the profile identifier of their designated guardian |
| `next_of_kin_chain` | UUID array | No | Ordered chain of designated next-of-kin profiles for deceased-profile rights handling |
| `cognitive_mode` | Enum | No | STANDARD / SIMPLIFIED / INTERVIEW_ONLY / LARGE_FONT — UX accessibility override for this profile |
| `interview_mode_active` (SQL: `interview_mode_active`) | Boolean | No | Whether Interview Mode is currently active for this profile |
| `b2b_institution_id` | UUID | No | If profile was created under a B2B institutional account |
| `religion_affiliation` | Enum or String | No | Religion or faith affiliation used for adaptive question filtering |
| `marital_status` | Enum | No | Marital-status field used for adaptive question filtering. States: MARRIED / WIDOWED / DIVORCED / NEVER_MARRIED |
| `calendar_preference` | Enum | No | GREGORIAN / HIJRI / HEBREW / LUNISOLAR / OTHER (custom) — preferred calendar system for anniversaries and memorial dates |
| `under_13_flag` | Boolean | Yes | Whether the profile subject is under 13 for child protection purposes |
| `guardian_consent_status` | Enum | No | Guardian consent state for protected minor processing and sharing |
| `backup_secret_vault_heir_profile_id` | UUID | No | Optional backup Secret Vault heir. Required if primary Secret Vault heir is designated (strongly recommended). |
| `shared_profile_owner_vault_id` | UUID | No | Identifies the owner vault for federated shared profiles |
| `memorial_liberation_policy` | Enum | No | Living Memory Liberation policy for this profile |
| `portability_contact_email` | String | No | Email for delivering data portability packages to claimed profile subjects |
| `is_adopted` | Boolean | No | Enables adoption-specific question filtering |

**Canonical field naming uses snake_case throughout. The canonical health-access field name is `health_authorized`.**

### 3.3 The Question Bank

#### 3.3.1 Overview and Architecture

The Question Bank is the platform's library of prompts driving interview sessions, story flows, and printable Question Books. It operates across three ownership tiers:

- **System Questions** — 1,000 platform-authored questions across 10 categories. Globally immutable. No individual user, Vault Keeper, or admin operation may modify a system question's text (QB-001, MANDATORY). Users can only *override* a system question at their own scope — the global record is never touched.
- **Vault Questions** — created by Vault Keepers or Family Admins. Shared with all vault members. Subject to the vault's Constitution.
- **Personal Questions** — created by any individual user. Private to the creator across all vaults they belong to.

This architecture guarantees every user always has the full, unmodified 1,000-question bank, plus their own personal layer that never contaminates anyone else's view.

#### 3.3.2 System Question Categories

| Category | Question Count | Adaptive Filters | Sample Questions |
|----------|---------------|-----------------|-----------------|
| 1. Childhood and Origins | 100 | Birth location, era, siblings | Where were you born? What is your earliest memory? |
| 2. Faith and Religion | 100 | `religion_affiliation` (off if None) | What role did faith play in your upbringing? |
| 3. Love and Marriage | 100 | `marital_status`, children | How did you meet your partner? |
| 4. Work and Purpose | 100 | Career type, era | What was your first job? |
| 5. Family and Lineage | 100 | Family structure, cultural context | What do you know about your grandparents? |
| 6. Hardship and Resilience | 100 | None — universal | What was your hardest year? |
| 7. World and History | 100 | Birth era, location | Where were you when a relevant world event happened? |
| 8. Wisdom and Legacy | 100 | None — universal | What do you want to be remembered for? |
| 9. Culture and Identity | 100 | Cultural context, language | What does your heritage mean to you? |
| 10. Health and Body (Private) | 100 | Health archive enabled | What health conditions have you lived with? |

**Category 10** questions are stored in the Health Archive, encrypted separately, and never shown in the general story flow unless the Vault Keeper explicitly enables health storytelling.

#### 3.3.3 Question Source Mode

When starting an interview, a new profile, or generating a Question Book, the user selects a **Question Source Mode**:

| Mode | What is shown | Use case |
|------|--------------|----------|
| `ALL` | System + vault + personal + active overrides | Default |
| `SYSTEM_ONLY` | System questions only | Standardised/institutional |
| `CUSTOM_ONLY` | Vault and personal questions only | Fully personalised |
| `SYSTEM_PLUS_CATEGORY` | System questions from selected categories + all custom | Focused sessions |

Stored in `question_set_configs.source_mode`, persists per context.

#### 3.3.4 Custom Questions and Categories

**Creating questions:** Any user can create personal questions (visible only to them). Vault Keepers and Family Admins can create vault questions (visible to all vault members). Owners can create additional system questions via the admin console.

**Custom categories:** Users create named custom categories scoped to USER or VAULT. A question can appear in multiple categories via `question_category_assignments` (non-destructive join table). Moving a question creates/removes an assignment row — the question's `primary_category_id` is never changed.

**Editing a system question (User Override):** Clicking **Edit** on a system question creates a `question_overrides` row scoped to the requesting profile. The original is never modified. The UI shows: *"You are creating a personal copy. The original is unchanged for all other users."* Override badge and **Revert to original** are always visible while active.

#### 3.3.5 Adaptive Filter Rules

- **Childless profiles:** Parenting and grandparent questions deprioritised (not removed). A `sensitivity_flag` is stored per question.
- **Marital status:** WIDOWED → spousal loss questions. DIVORCED → life-chapter questions. NEVER_MARRIED → chosen-family questions.
- **Adopted profiles:** Biological-heritage questions offered with "as far as you know" qualifier rather than hidden.
- **Filter logic:** OR-inclusive by default. Vault Keeper can switch to AND-strict mode per profile.

#### 3.3.6 Tier Limits

| Feature | Free | Premium | Enterprise |
|---------|------|---------|------------|
| Custom questions | 50 | Unlimited | Unlimited |
| Custom categories | 5 | Unlimited | Unlimited |
| Vault questions | ✗ | ✓ | ✓ |
| AI-suggested questions / month | 20 | 100 | Unlimited |
| Question Book PDF export | ✓ | ✓ | ✓ |
| Question Book physical print | ✓ | ✓ | ✓ |

#### 3.3.7 Question Audit Trail

Every question creation is logged with full provenance (stored in the `questions` table):

| Field | Purpose |
|-------|---------|
| `created_by_profile_id` | Who created it |
| `created_at` | UTC timestamp |
| `device_fingerprint_at_creation` | Device identifier |
| `session_id_at_creation` | Session UUID |
| `geo_country_at_creation` | ISO 3166-1 alpha-2 country code |
| `geo_region_at_creation` | State/province — never street-level |
| `ip_hash_at_creation` | SHA-256 hash — never raw IP |

Every mutation fires an activity log event: `QUESTION_CREATED`, `QUESTION_EDITED`, `QUESTION_OVERRIDE_CREATED`, `QUESTION_CATEGORY_CREATED`, `QUESTION_MOVED`, `QUESTION_BRANCHED`.

#### 3.3.8 The Question Book

A **Question Book** is a formatted printable or downloadable document containing a curated list of questions with no answers. Use cases: a gift to a family member, preparation before a recorded interview, a family gathering activity, a companion to physical photo albums.

The user selects questions interactively in the Question Book Builder (Section 6.3.5). The final set is materialised as a `book_snapshots` row with `snapshot_type = 'QUESTION_BOOK'`. Delivery: PDF download or physical print (Section 6.14, 30-minute recall per PRINT-001). Physical print spec: A5 or US Digest softcover, 80gsm uncoated paper, 16–200 pages.


## 4. Data Models & Database Schema

### 4.1 Database Architecture Overview

FamilyVault uses **PostgreSQL 16** as the primary database. All tables use UUID version 4 primary keys. All timestamps are stored in UTC with timezone. The `activity_logs` table resides on a separate tablespace from application data.

**Key Constraints:**
- All primary keys are UUID v4
- All foreign keys use `ON DELETE RESTRICT`. No cascade deletes are permitted. All parent-entity removal is handled through application-level branch operations.
- No hard deletes on content tables (branching only)
- activity_logs table: INSERT-only, no UPDATE or DELETE permissions

**Vault Creation Bootstrap Sequence**

The circular FK dependencies (vaults ↔ profiles ↔ version_chains) use `DEFERRABLE INITIALLY DEFERRED` constraints. All inserts must occur within a single transaction in this order — FK checks are deferred until `COMMIT`:

1. `INSERT INTO version_chains` (`entity_type='vault'`, `entity_id=<vault_id>`, `vault_id=<vault_id>`)
2. `INSERT INTO version_chains` (`entity_type='profile'`, `entity_id=<profile_id>`, `vault_id=<vault_id>`)
3. `INSERT INTO profiles` (`vault_id=<vault_id>`, `version_chain_id=<profile_chain_id>`, ...)
4. `INSERT INTO vaults` (`keeper_profile_id=<profile_id>`, `version_chain_id=<vault_chain_id>`, ...)
5. `COMMIT` — all deferred FK checks fire and pass

### 4.2 Core Tables

#### 4.2.1 vaults
```sql
CREATE TYPE privacy_tier_enum AS ENUM ('public', 'family', 'immediate', 'named', 'private', 'locked');

CREATE TABLE vaults (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    keeper_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    constitution_id UUID REFERENCES vault_constitutions(id),
    privacy_tier privacy_tier_enum DEFAULT 'private',
    vault_heir_primary_profile_id UUID REFERENCES profiles(id),
    vault_heir_backup1_profile_id UUID REFERENCES profiles(id),
    vault_heir_backup2_profile_id UUID REFERENCES profiles(id),
    acting_family_admin_profile_id UUID REFERENCES profiles(id),
    death_witness_1_profile_id UUID REFERENCES profiles(id),
    death_witness_2_profile_id UUID REFERENCES profiles(id),
    ip_forensic_salt VARCHAR(64) NOT NULL,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_vaults_keeper ON vaults(keeper_profile_id);
CREATE INDEX idx_vaults_deleted ON vaults(is_deleted_branch) WHERE is_deleted_branch = TRUE;
```

#### 4.2.2 profiles
```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    display_name VARCHAR(255) NOT NULL,
    legal_name VARCHAR(255),
    email VARCHAR(255),
    phone VARCHAR(50),
    date_of_birth DATE CHECK (date_of_birth <= CURRENT_DATE),
    date_of_death DATE,
    is_deceased BOOLEAN DEFAULT FALSE,
    CONSTRAINT death_after_birth CHECK (date_of_death IS NULL OR date_of_death >= date_of_birth),
    
    -- Cultural context fields
    cultural_context VARCHAR(100),
    lineage_clan_name VARCHAR(255),
    naming_convention VARCHAR(100),
    religion_affiliation VARCHAR(100),
    calendar_preference VARCHAR(20) DEFAULT 'GREGORIAN',
    
    -- Relationship fields
    marital_status VARCHAR(50),
    current_spouse_profile_id UUID REFERENCES profiles(id),
    previous_spouse_profile_ids UUID[],
    
    -- Lifecycle & legal fields
    under_13_flag BOOLEAN DEFAULT FALSE,
    guardian_consent_status VARCHAR(50),
    guardian_profile_id UUID REFERENCES profiles(id),
    consent_verified_at TIMESTAMPTZ,
    
    -- Access control fields
    privacy_tier privacy_tier_enum DEFAULT 'private',
    -- Note: health_authorized is stored in vault_memberships per DB-010 (membership record is authoritative)
    sar_opt_in BOOLEAN DEFAULT TRUE,
    portability_contact_email VARCHAR(255),
    is_adopted BOOLEAN DEFAULT FALSE,
    b2b_institution_id UUID,
    no_ai_training_flag BOOLEAN DEFAULT FALSE,
    no_ai_training_set_at TIMESTAMPTZ,
    no_ai_training_set_by UUID REFERENCES profiles(id),
    
    -- Succession & legacy fields
    is_vault_keeper BOOLEAN DEFAULT FALSE,
    vault_keeper_order INTEGER,
    next_of_kin_chain UUID[],
    secret_vault_heir_profile_id UUID REFERENCES profiles(id),
    backup_secret_vault_heir_profile_id UUID REFERENCES profiles(id),
    memorial_liberation_policy VARCHAR(20),
    memorial_liberation_enabled BOOLEAN DEFAULT FALSE,
    
    -- Accessibility fields
    cognitive_mode VARCHAR(20) DEFAULT 'STANDARD',
    interview_mode_active BOOLEAN DEFAULT FALSE,  -- Canonical field name per spec
    interview_mode_pin_hash VARCHAR(255),
    interview_mode_questions UUID[],
    read_to_me_enabled BOOLEAN DEFAULT TRUE,
    font_size_preference INTEGER DEFAULT 16,
    
    -- Federation fields (canonical: owner_vault_id for shared profiles)
    owner_vault_id UUID,  -- References owner vault for federated/shared profiles (FED-001)
    vault_link_id UUID,
    local_privacy_overlay JSONB DEFAULT '{}',
    
    -- Subject rights contact
    subject_rights_email VARCHAR(255),  -- Contact for SAR notifications
    
    -- Branch deletion fields (IMM-C-001: profiles are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Partial unique index: only one keeper per vault (PostgreSQL syntax)
CREATE UNIQUE INDEX idx_one_keeper_per_vault ON profiles(vault_id, is_vault_keeper) WHERE is_vault_keeper = TRUE;

CREATE INDEX idx_profiles_vault ON profiles(vault_id);
CREATE INDEX idx_profiles_deceased ON profiles(is_deceased);
CREATE INDEX idx_profiles_keeper ON profiles(is_vault_keeper) WHERE is_vault_keeper = TRUE;
CREATE INDEX idx_profiles_deleted ON profiles(is_deleted_branch) WHERE is_deleted_branch = TRUE;
```

#### 4.2.3 blocks
```sql
CREATE TABLE blocks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    section_id UUID REFERENCES sections(id),  -- NULL for: (1) TEMPLATE blocks (is_template=TRUE), (2) ORPHANED blocks during section deletion—sync transaction branches all children via SECTION_CASCADE_BRANCH
    block_type VARCHAR(50) NOT NULL,
    content JSONB NOT NULL,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    named_viewers UUID[],
    is_pinned BOOLEAN DEFAULT FALSE,
    pin_position INTEGER,
    is_locked BOOLEAN DEFAULT FALSE,
    lock_reason TEXT,
    is_monetized BOOLEAN DEFAULT FALSE,
    price_cents INTEGER,
    ai_generated_flag BOOLEAN DEFAULT FALSE,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_blocks_vault ON blocks(vault_id);
CREATE INDEX idx_blocks_section ON blocks(section_id);
CREATE INDEX idx_blocks_type ON blocks(block_type);
CREATE INDEX idx_blocks_privacy ON blocks(privacy_tier);
CREATE INDEX idx_blocks_deleted ON blocks(is_deleted_branch) WHERE is_deleted_branch = TRUE;
```

**Section ID Nullable Exceptions:**
- **TEMPLATE blocks:** Blocks marked with `is_template = TRUE`. Template blocks are displayed in a template library UI (a browsable gallery of reusable content blocks accessible only to Vault Keepers and Family Admins), not in book content. They must be assigned to a section before being added to book content.
- **ORPHANED blocks:** During section deletion-branching, child blocks are temporarily orphaned (section_id set to NULL) during the synchronous transaction. All child blocks are branched via `SECTION_CASCADE_BRANCH` event before the parent section is marked deleted. No background job is required.

#### 4.2.4 chapters
```sql
CREATE TABLE chapters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    book_id UUID NOT NULL REFERENCES books(id) ON DELETE RESTRICT,
    title VARCHAR(255) NOT NULL,
    chapter_number INTEGER NOT NULL,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    UNIQUE(book_id, chapter_number)
);

CREATE INDEX idx_chapters_book ON chapters(book_id);
```

#### 4.2.5 sections
```sql
CREATE TABLE sections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    chapter_id UUID NOT NULL REFERENCES chapters(id) ON DELETE RESTRICT,
    title VARCHAR(255) NOT NULL,
    section_number INTEGER NOT NULL,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    UNIQUE(chapter_id, section_number)
);

CREATE INDEX idx_sections_chapter ON sections(chapter_id);
```

#### 4.2.6 books
```sql
CREATE TABLE books (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    folder_id UUID REFERENCES folders(id),
    title VARCHAR(255) NOT NULL,
    subtitle VARCHAR(500),
    spine_summary TEXT,
    cover_asset_id UUID REFERENCES assets(id),
    era_start_date DATE,
    era_end_date DATE,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    author_profile_ids UUID[],
    is_template BOOLEAN DEFAULT FALSE,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_books_vault ON books(vault_id);
CREATE INDEX idx_books_folder ON books(folder_id);
```

#### 4.2.7 folders
```sql
CREATE TABLE folders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    parent_folder_id UUID REFERENCES folders(id),
    name VARCHAR(255) NOT NULL,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    is_locked BOOLEAN DEFAULT FALSE,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    CONSTRAINT no_deep_nesting CHECK (parent_folder_id IS NULL OR parent_folder_id != id)
);

CREATE INDEX idx_folders_vault ON folders(vault_id);
CREATE INDEX idx_folders_parent ON folders(parent_folder_id);
```

#### 4.2.8 assets
```sql
CREATE TABLE assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    uploader_profile_id UUID NOT NULL REFERENCES profiles(id),
    storage_key VARCHAR(500) NOT NULL,
    filename VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    md5_checksum VARCHAR(32) NOT NULL,
    sha256_checksum VARCHAR(64) NOT NULL,
    privacy_tier privacy_tier_enum DEFAULT 'private',
    is_anchored BOOLEAN DEFAULT FALSE,
    anchor_block_id UUID REFERENCES blocks(id),
    anchor_context_before TEXT,
    anchor_context_after TEXT,
    is_orphaned BOOLEAN DEFAULT FALSE,
    orphaning_reason TEXT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_assets_vault ON assets(vault_id);
CREATE INDEX idx_assets_uploader ON assets(uploader_profile_id);
CREATE INDEX idx_assets_orphaned ON assets(is_orphaned) WHERE is_orphaned = TRUE;
```

#### 4.2.9 version_chains
```sql
CREATE TABLE version_chains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(50) NOT NULL,
    entity_id UUID NOT NULL,
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_version_chains_entity ON version_chains(entity_type, entity_id);
```

#### 4.2.10 version_nodes
```sql
CREATE TABLE version_nodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    node_number INTEGER NOT NULL,
    actor_user_id UUID NOT NULL,
    timestamp_utc TIMESTAMPTZ DEFAULT NOW(),
    device_fingerprint VARCHAR(255),
    session_id UUID,
    action_type VARCHAR(50) NOT NULL,
    reason TEXT,  -- NULL for CREATE actions, NOT NULL for EDIT/DELETE per IMM-C-004
    content_snapshot JSONB NOT NULL,
    integrity_hash VARCHAR(64) NOT NULL,
    diff_from_previous JSONB,
    previous_node_id UUID REFERENCES version_nodes(id),
    UNIQUE(chain_id, node_number),
    CONSTRAINT reason_required_for_non_create CHECK (action_type = 'CREATE' OR reason IS NOT NULL)
);

CREATE INDEX idx_version_nodes_chain ON version_nodes(chain_id);
CREATE INDEX idx_version_nodes_actor ON version_nodes(actor_user_id);
```

#### 4.2.11 activity_logs
```sql
CREATE TYPE severity_enum AS ENUM ('DEBUG', 'INFO', 'WARN', 'CRITICAL');
CREATE TYPE event_category_enum AS ENUM (
    'CONTENT', 'MEDIA', 'PROFILE', 'SHARING', 'PRIVACY', 
    'VAULT', 'MEMBERSHIP', 'ADMIN', 'SECURITY', 'AUTH',
    'BILLING', 'PRINT', 'EXPORT', 'LEGAL', 'SYSTEM', 'LEGACY',
    'SUBJECT_RIGHTS', 'AI', 'INTERVIEW', 'TRUST_SAFETY'
);

-- Event type is VARCHAR to support 250+ canonical events (Section 9)
-- Event catalog is the authoritative source; database enforces foreign key to event_catalog
CREATE TABLE event_catalog (
    event_name VARCHAR(100) PRIMARY KEY,
    category event_category_enum NOT NULL,
    severity severity_enum NOT NULL,
    requires_reason BOOLEAN NOT NULL DEFAULT FALSE,
    description TEXT,
    first_introduced_version VARCHAR(10),
    
    -- Deprecation tracking (event types can be retired without breaking FK references)
    is_deprecated BOOLEAN DEFAULT FALSE,
    deprecated_at TIMESTAMPTZ,
    deprecated_reason TEXT
);

CREATE TABLE activity_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id UUID NOT NULL UNIQUE,
    event_type VARCHAR(100) NOT NULL REFERENCES event_catalog(event_name),
    event_category event_category_enum NOT NULL,
    timestamp_utc TIMESTAMPTZ DEFAULT NOW(),
    severity severity_enum NOT NULL,
    actor_user_id UUID,
    actor_type VARCHAR(20) DEFAULT 'USER',
    vault_id UUID,
    entity_type VARCHAR(50),
    entity_id UUID,
    session_id UUID,
    device_fingerprint VARCHAR(255),
    ip_address_hash VARCHAR(64),  -- SHA-256 hash with per-vault salt (SEC-008)
    reason TEXT,  -- Required for non-CREATE actions per IMM-C-004
    metadata JSONB DEFAULT '{}',
    requires_reason BOOLEAN DEFAULT FALSE,
    CONSTRAINT reason_required_for_non_create CHECK (
        requires_reason = FALSE OR reason IS NOT NULL
    )
) TABLESPACE activity_log_ts;

CREATE INDEX idx_activity_logs_vault ON activity_logs(vault_id);
CREATE INDEX idx_activity_logs_actor ON activity_logs(actor_user_id);
CREATE INDEX idx_activity_logs_type ON activity_logs(event_type);
CREATE INDEX idx_activity_logs_category ON activity_logs(event_category);
CREATE INDEX idx_activity_logs_timestamp ON activity_logs(timestamp_utc);
CREATE INDEX idx_activity_logs_severity ON activity_logs(severity);
```

#### 4.2.12 relationships
```sql
CREATE TABLE relationships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    from_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    to_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    relationship_type VARCHAR(50) NOT NULL,
    is_biological BOOLEAN,
    is_adoptive BOOLEAN,
    is_step BOOLEAN,
    start_date DATE,
    end_date DATE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    CONSTRAINT no_self_relationship CHECK (from_profile_id != to_profile_id)
);

CREATE INDEX idx_relationships_from ON relationships(from_profile_id);
CREATE INDEX idx_relationships_to ON relationships(to_profile_id);
CREATE INDEX idx_relationships_type ON relationships(relationship_type);
```

#### 4.2.13 mcp_providers
```sql
CREATE TABLE mcp_providers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    provider_type VARCHAR(50) NOT NULL,
    capabilities TEXT[] NOT NULL,
    config JSONB NOT NULL,
    credentials_ref VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    is_primary BOOLEAN DEFAULT FALSE,
    health_check_url VARCHAR(500),
    last_health_check TIMESTAMPTZ,
    health_status VARCHAR(20) DEFAULT 'unknown',
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_mcp_providers_type ON mcp_providers(provider_type);
CREATE INDEX idx_mcp_providers_active ON mcp_providers(is_active);
```

#### 4.2.14 search_index
```sql
-- Search Architecture: Typesense is the canonical search engine (Section 2.4).
-- This Postgres search_index serves three purposes:
-- 1) Audit/trail: Immutable record of what was indexed and when
-- 2) Backup search: Fallback if Typesense is unavailable
-- 3) Erasure compliance: Target for GDPR pseudonymisation (SAR-005)
-- Sync: Typesense is source of truth; this table is populated asynchronously
-- via CDC from the content tables. Privacy tier changes propagate to both.
CREATE TABLE search_index (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(50) NOT NULL,
    entity_id UUID NOT NULL,
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    content_vector TSVECTOR,
    encrypted_plaintext_content BYTEA,  -- AES-256-GCM encrypted; decryption requires vault search key
    privacy_tier privacy_tier_enum DEFAULT 'private',
    is_pseudonymized BOOLEAN DEFAULT FALSE,
    pseudonym_id UUID,
    access_control_hash VARCHAR(64),  -- Hash of authorized viewer IDs for quick filtering
    last_indexed TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ
);

CREATE INDEX idx_search_vector ON search_index USING GIN(content_vector);
CREATE INDEX idx_search_vault ON search_index(vault_id);
```

#### 4.2.15 health_records
```sql
CREATE TABLE health_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    record_type VARCHAR(50) NOT NULL,
    encrypted_data BYTEA NOT NULL,
    encrypted_by UUID NOT NULL,
    encryption_key_ref VARCHAR(255) NOT NULL,
    ai_analysis_results JSONB,
    is_ai_analyzed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    is_deleted_branch BOOLEAN DEFAULT FALSE
) TABLESPACE health_data_ts;

CREATE INDEX idx_health_records_profile ON health_records(profile_id);
```

#### 4.2.16 time_locks
```sql
-- Time-lock content for delayed release (wills, final letters, conditional secrets)
CREATE TYPE time_lock_type_enum AS ENUM ('DATE', 'CONDITIONAL', 'DEATH_PROTOCOL');
CREATE TYPE time_lock_status_enum AS ENUM (
    'PENDING',           -- Not yet triggered
    'TRIGGERED',         -- Condition met, waiting release
    'RELEASED',          -- Content unlocked to recipient
    'DEFERRED_REVERSAL', -- Death protocol: waiting for reversal window
    'DEFERRED_QUIET',    -- Death protocol: waiting for quiet period
    'SEALED',            -- Recipient deceased, no next-of-kin chain
    'REVOKED'            -- Cancelled by creator before trigger
);

CREATE TABLE time_locks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    content_block_id UUID REFERENCES blocks(id) ON DELETE RESTRICT,  -- Optional: may be document-level
    content_entity_type VARCHAR(50) NOT NULL,  -- 'block', 'section', 'book', 'secret_vault'
    content_entity_id UUID NOT NULL,
    
    -- Lock configuration
    lock_type time_lock_type_enum NOT NULL,
    unlock_date TIMESTAMPTZ,  -- For DATE type
    unlock_condition TEXT,    -- For CONDITIONAL type (human-readable condition)
    condition_resolver_profile_id UUID REFERENCES profiles(id) ON DELETE RESTRICT,  -- Who can mark condition met
    
    -- Recipient configuration
    recipient_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    recipient_next_of_kin_chain UUID[],  -- Fallback chain if recipient deceased
    
    -- Status tracking
    status time_lock_status_enum NOT NULL DEFAULT 'PENDING',
    status_changed_at TIMESTAMPTZ,
    released_at TIMESTAMPTZ,
    released_by UUID,  -- Profile who triggered release (for CONDITIONAL type)
    
    -- Death protocol interaction (SV-002)
    death_protocol_triggered_at TIMESTAMPTZ,
    death_protocol_reversal_window_closes TIMESTAMPTZ,
    death_protocol_quiet_period_ends TIMESTAMPTZ,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    
    -- Creator tracking
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: time locks are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    
    -- Constraints
    CONSTRAINT date_required_for_date_type CHECK (
        lock_type != 'DATE' OR unlock_date IS NOT NULL
    ),
    CONSTRAINT resolver_required_for_conditional CHECK (
        lock_type != 'CONDITIONAL' OR condition_resolver_profile_id IS NOT NULL
    )
);

CREATE INDEX idx_time_locks_vault ON time_locks(vault_id);
CREATE INDEX idx_time_locks_recipient ON time_locks(recipient_profile_id);
CREATE INDEX idx_time_locks_status ON time_locks(status);
CREATE INDEX idx_time_locks_unlock_date ON time_locks(unlock_date) WHERE lock_type = 'DATE';
```

#### 4.2.17 notifications
```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    recipient_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    notification_type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    body TEXT NOT NULL,
    action_url VARCHAR(500),
    is_read BOOLEAN DEFAULT FALSE,
    is_suppressible BOOLEAN DEFAULT TRUE,
    sent_at TIMESTAMPTZ,
    read_at TIMESTAMPTZ,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notifications_recipient ON notifications(recipient_profile_id);
CREATE INDEX idx_notifications_unread ON notifications(recipient_profile_id, is_read) WHERE is_read = FALSE;
```

#### 4.2.18 vault_memberships
```sql
CREATE TABLE vault_memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    role VARCHAR(50) NOT NULL DEFAULT 'reader',
    invited_by UUID REFERENCES profiles(id),
    invited_at TIMESTAMPTZ DEFAULT NOW(),
    accepted_at TIMESTAMPTZ,
    health_authorized BOOLEAN DEFAULT FALSE,
    health_authorized_by UUID REFERENCES profiles(id),
    health_authorized_at TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT TRUE,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(vault_id, profile_id)
);

CREATE INDEX idx_memberships_vault ON vault_memberships(vault_id);
CREATE INDEX idx_memberships_profile ON vault_memberships(profile_id);
CREATE INDEX idx_memberships_role ON vault_memberships(role);
```

#### 4.2.19 vault_constitutions
```sql
CREATE TABLE vault_constitutions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    content JSONB NOT NULL DEFAULT '[]',
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(vault_id)
);

CREATE INDEX idx_constitutions_vault ON vault_constitutions(vault_id);
```

#### 4.2.20 interview_sessions
```sql
CREATE TABLE interview_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    subject_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    interviewer_profile_id UUID REFERENCES profiles(id),
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    assigned_question_ids UUID[],
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    reviewed_by UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    reviewed_at TIMESTAMPTZ,
    review_decision VARCHAR(50),
    review_notes TEXT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_interviews_vault ON interview_sessions(vault_id);
CREATE INDEX idx_interviews_subject ON interview_sessions(subject_profile_id);
CREATE INDEX idx_interviews_status ON interview_sessions(status);
```

#### 4.2.21 interview_question_assignments
```sql
CREATE TABLE interview_question_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    session_id UUID NOT NULL REFERENCES interview_sessions(id) ON DELETE RESTRICT,
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
    question_id UUID NOT NULL,
    question_text TEXT NOT NULL,
    assigned_by UUID NOT NULL REFERENCES profiles(id),
    assigned_at TIMESTAMPTZ DEFAULT NOW(),
    answer_text TEXT,
    answer_audio_asset_id UUID REFERENCES assets(id),
    answered_at TIMESTAMPTZ,
    skipped BOOLEAN DEFAULT FALSE,
    skip_reason TEXT,
    order_index INTEGER NOT NULL,
    is_deleted_branch BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_question_assignments_session ON interview_question_assignments(session_id);
CREATE INDEX idx_question_assignments_order ON interview_question_assignments(session_id, order_index);
```

#### 4.2.22 shared_links
```sql
-- Public shareable links for viral growth loop (Section 6.4.2)
CREATE TYPE shared_link_status_enum AS ENUM ('ACTIVE', 'EXPIRED', 'REVOKED', 'CONSUMED');

CREATE TABLE shared_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    token VARCHAR(255) NOT NULL UNIQUE,  -- URL-safe random token
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- Content scope
    entity_type VARCHAR(50) NOT NULL,  -- 'block', 'section', 'book', 'profile'
    entity_id UUID NOT NULL,
    
    -- Access control
    status shared_link_status_enum NOT NULL DEFAULT 'ACTIVE',
    max_uses INTEGER,  -- NULL = unlimited
    use_count INTEGER DEFAULT 0,
    expires_at TIMESTAMPTZ,
    
    -- Tracking
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    revoked_at TIMESTAMPTZ,
    revoked_by UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- Branch deletion fields (IMM-C-001: shared links are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_shared_links_token ON shared_links(token);
CREATE INDEX idx_shared_links_vault ON shared_links(vault_id);
CREATE INDEX idx_shared_links_status ON shared_links(status) WHERE status = 'ACTIVE';
```

#### 4.2.23 webhook_subscriptions
```sql
-- Developer API webhook subscriptions (Section 6.11)
CREATE TYPE webhook_status_enum AS ENUM ('ACTIVE', 'PAUSED', 'DISABLED', 'FAILED');

CREATE TABLE webhook_subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- Endpoint configuration
    endpoint_url VARCHAR(500) NOT NULL,
    secret_key_ref VARCHAR(255) NOT NULL,  -- Reference to secrets manager (SEC-009)
    
    -- Event filtering (comma-separated event names from Section 9)
    subscribed_events TEXT NOT NULL,  -- e.g., 'BLOCK_CREATED,BLOCK_EDITED,VAULT_HEIR_STEWARDSHIP_ACCEPTED'
    
    -- Status
    status webhook_status_enum NOT NULL DEFAULT 'ACTIVE',
    status_reason TEXT,  -- Why disabled/failed
    
    -- Retry configuration
    retry_count INTEGER DEFAULT 0,
    last_failure_at TIMESTAMPTZ,
    last_failure_response TEXT,
    
    -- Metadata
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: webhook subscriptions are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_webhook_subscriptions_vault ON webhook_subscriptions(vault_id);
CREATE INDEX idx_webhook_subscriptions_status ON webhook_subscriptions(status);
```

#### 4.2.24 b2b_read_grants
```sql
-- B2B institutional access grants (Section 6.10, B2B-006)
CREATE TYPE b2b_grant_status_enum AS ENUM ('ACTIVE', 'PENDING_REVIEW', 'EXPIRED', 'REVOKED');

CREATE TABLE b2b_read_grants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    institution_id UUID NOT NULL,  -- References external B2B institution registry
    
    -- Grant scope (configurable per B2B-006)
    granted_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    grant_scope VARCHAR(50) NOT NULL,  -- 'vault', 'book', 'profile', 'health'
    scope_entity_id UUID,  -- NULL if vault-wide
    
    -- Timing (B2B-006: 30-day default, 365-day max)
    granted_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    
    -- Status
    status b2b_grant_status_enum NOT NULL DEFAULT 'ACTIVE',
    status_changed_at TIMESTAMPTZ,
    status_changed_by UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- Stewardship transfer tracking
    stewardship_transfer_at TIMESTAMPTZ,  -- When transfer triggered PENDING_REVIEW
    reapproval_deadline TIMESTAMPTZ,      -- 14-day re-approval window
    
    -- Tracking
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: B2B grants are non-hard-deletable; reason required per IMM-C-004)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT NOT NULL,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_b2b_grants_vault ON b2b_read_grants(vault_id);
CREATE INDEX idx_b2b_grants_institution ON b2b_read_grants(institution_id);
CREATE INDEX idx_b2b_grants_status ON b2b_read_grants(status);
CREATE INDEX idx_b2b_grants_expiry ON b2b_read_grants(expires_at) WHERE status = 'ACTIVE';
```

#### 4.2.25 subject_rights_requests
```sql
-- GDPR/CCPA Subject Access Request queue (Section 6.7, SAR-001)
CREATE TYPE sar_type_enum AS ENUM ('ACCESS', 'CORRECTION', 'DELETION', 'PORTABILITY');
CREATE TYPE sar_status_enum AS ENUM ('PENDING', 'IN_REVIEW', 'FULFILLED', 'DENIED', 'PARTIAL', 'ESCALATED');
CREATE TYPE sar_response_enum AS ENUM ('FULFILLED', 'DENIED', 'PARTIAL');

CREATE TABLE subject_rights_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    
    -- Requester info
    requester_profile_id UUID REFERENCES profiles(id) ON DELETE RESTRICT,  -- NULL if external requester
    requester_email VARCHAR(255) NOT NULL,
    requester_verification_status VARCHAR(50),  -- 'verified', 'pending', 'failed'
    
    -- Request details
    request_type sar_type_enum NOT NULL,
    request_scope TEXT,  -- JSON: which profiles/content
    request_body TEXT,
    
    -- Status tracking
    status sar_status_enum NOT NULL DEFAULT 'PENDING',
    submitted_at TIMESTAMPTZ DEFAULT NOW(),
    deadline_at TIMESTAMPTZ NOT NULL,  -- 30 days per SAR-001
    responded_at TIMESTAMPTZ,
    response_type sar_response_enum,
    response_reason TEXT,  -- If denied/partial
    
    -- Assignment
    assigned_to UUID REFERENCES profiles(id) ON DELETE RESTRICT,  -- DPO or admin handling request
    
    -- Tracking
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: SAR records are non-hard-deletable)
    -- Note: SARs should never be branched (legal compliance records), but this prevents hard DELETE
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_sar_vault ON subject_rights_requests(vault_id);
CREATE INDEX idx_sar_status ON subject_rights_requests(status);
CREATE INDEX idx_sar_deadline ON subject_rights_requests(deadline_at) WHERE status IN ('PENDING', 'IN_REVIEW');
```

#### 4.2.26 book_snapshots
```sql
-- Point-in-time read-only copies for archiving/printing (Section 6.3.3)
CREATE TYPE snapshot_type_enum AS ENUM ('USER_SNAPSHOT', 'PRINT_VERSION', 'GHOST_MODE_RECONSTRUCTION', 'QUESTION_BOOK');
CREATE TYPE snapshot_status_enum AS ENUM ('PENDING', 'READY', 'RENDERED', 'EXPIRED');

CREATE TABLE book_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    book_id UUID REFERENCES books(id) ON DELETE RESTRICT,  -- NULL for QUESTION_BOOK snapshots (not attached to story books)
    question_set_id UUID REFERENCES question_set_configs(id) ON DELETE RESTRICT,  -- Required for QUESTION_BOOK snapshots
    
    -- Snapshot metadata
    name VARCHAR(255) NOT NULL,  -- User-assigned name
    snapshot_type snapshot_type_enum NOT NULL,
    status snapshot_status_enum NOT NULL DEFAULT 'PENDING',
    
    -- Content (materialized copy)
    snapshot_data JSONB NOT NULL,  -- Complete rendered book state
    included_block_ids UUID[],
    
    -- Sharing (for USER_SNAPSHOT type)
    share_token VARCHAR(255),  -- NULL if not shared
    share_expires_at TIMESTAMPTZ,
    
    -- Print-specific fields (for PRINT_VERSION type)
    print_order_id UUID,
    recall_window_closes_at TIMESTAMPTZ,  -- 30-minute recall per PRINT-001
    
    -- Versioning
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: book snapshots are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_book_snapshots_vault ON book_snapshots(vault_id);
CREATE INDEX idx_book_snapshots_book ON book_snapshots(book_id);
CREATE INDEX idx_book_snapshots_share ON book_snapshots(share_token) WHERE share_token IS NOT NULL;
```

#### 4.2.27 contributions
```sql
-- Pending contributions from Verified Members (Section 6.3.2)
CREATE TYPE contribution_type_enum AS ENUM ('STORY', 'INTERVIEW_SESSION', 'MEMORY', 'BIOGRAPHY_EDIT', 'CONSTITUTION_AMENDMENT');
CREATE TYPE contribution_status_enum AS ENUM ('PENDING', 'APPROVED', 'REJECTED', 'EDITED_WHILE_PENDING');

CREATE TABLE contributions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    
    -- Contributor
    contributor_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    contribution_type contribution_type_enum NOT NULL,
    
    -- Target
    target_entity_type VARCHAR(50),  -- 'block', 'section', 'book', 'profile', 'constitution'
    target_entity_id UUID,
    
    -- Content
    title VARCHAR(255),
    content TEXT,
    metadata JSONB,  -- Type-specific data (e.g., interview session ID)
    
    -- Status workflow
    status contribution_status_enum NOT NULL DEFAULT 'PENDING',
    submitted_at TIMESTAMPTZ DEFAULT NOW(),
    submitted_at_version INTEGER DEFAULT 1,  -- Resubmission counter
    
    -- Review tracking
    reminder_sent_at TIMESTAMPTZ,  -- 14-day reminder
    escalated_at TIMESTAMPTZ,      -- 21-day Family Admin escalation
    delay_notice_sent_at TIMESTAMPTZ,  -- 30-day contributor notification
    
    -- Resolution
    reviewed_by UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    reviewed_at TIMESTAMPTZ,
    rejection_reason TEXT,
    
    -- Branch deletion fields (IMM-C-001: contributions are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_contributions_vault ON contributions(vault_id);
CREATE INDEX idx_contributions_status ON contributions(status);
CREATE INDEX idx_contributions_pending ON contributions(vault_id, status) WHERE status = 'PENDING';
```

#### 4.2.28 family_reunion_qr
```sql
-- QR codes for family reunion events (Section 6.4.2)
CREATE TYPE qr_status_enum AS ENUM ('ACTIVE', 'EXPIRED', 'REVOKED', 'SUSPENDED_DEATH');

CREATE TABLE family_reunion_qr (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- QR configuration
    qr_code_token VARCHAR(255) NOT NULL UNIQUE,
    event_name VARCHAR(255),
    max_scans INTEGER,  -- NULL = unlimited
    scan_count INTEGER DEFAULT 0,
    
    -- Status
    status qr_status_enum NOT NULL DEFAULT 'ACTIVE',
    expires_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    
    -- Death protocol tracking
    death_protocol_flag_set_at TIMESTAMPTZ,  -- When DEATH_PROTOCOL_TRIGGERED set flag
    
    -- Created guests tracking
    created_profile_ids UUID[],  -- Profiles created via this QR
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: reunion QR codes are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_reunion_qr_vault ON family_reunion_qr(vault_id);
CREATE INDEX idx_reunion_qr_token ON family_reunion_qr(qr_code_token);
CREATE INDEX idx_reunion_qr_status ON family_reunion_qr(status);
```

#### 4.2.29 memorial_addenda
```sql
-- Post-death collaborative memorial contributions (Section 6.5.4)
CREATE TYPE memorial_entry_status_enum AS ENUM ('PENDING_MODERATION', 'APPROVED', 'REJECTED');

CREATE TABLE memorial_addenda (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    deceased_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    
    -- Entry
    contributor_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    entry_type VARCHAR(50) NOT NULL,  -- 'tribute', 'memory', 'photo', 'story'
    content TEXT NOT NULL,
    
    -- Moderation
    status memorial_entry_status_enum NOT NULL DEFAULT 'PENDING_MODERATION',
    submitted_at TIMESTAMPTZ DEFAULT NOW(),
    moderated_by UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    moderated_at TIMESTAMPTZ,
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: memorial addenda are non-hard-deletable)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT
);

CREATE INDEX idx_memorial_vault ON memorial_addenda(vault_id);
CREATE INDEX idx_memorial_deceased ON memorial_addenda(deceased_profile_id);
CREATE INDEX idx_memorial_status ON memorial_addenda(status);
```

#### 4.2.30 escrow_deposits
```sql
-- Annual encrypted vault exports for continuity guarantee (Section 1.6)
CREATE TYPE escrow_status_enum AS ENUM ('PENDING', 'DEPOSITED', 'VERIFIED', 'FAILED');

CREATE TABLE escrow_deposits (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    
    -- Deposit metadata
    deposit_year INTEGER NOT NULL,
    status escrow_status_enum NOT NULL DEFAULT 'PENDING',
    
    -- File location
    storage_provider VARCHAR(50) NOT NULL,  -- 'aws_s3', 'gcs', 'azure'
    storage_path VARCHAR(500) NOT NULL,
    storage_bucket VARCHAR(255) NOT NULL,
    
    -- Integrity
    file_size_bytes BIGINT,
    sha256_hash VARCHAR(64),
    encryption_key_ref VARCHAR(255) NOT NULL,  -- Reference to secrets manager
    
    -- Verification
    deposited_at TIMESTAMPTZ,
    verified_at TIMESTAMPTZ,
    verification_log TEXT,
    
    -- Access tracking
    released_to_owner_at TIMESTAMPTZ,  -- If company shutdown
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Branch deletion fields (IMM-C-001: escrow deposits are permanent legal audit artefacts)
    -- CHECK constraint ensures these records can never be branched (permanent retention required)
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    CONSTRAINT escrow_permanent CHECK (is_deleted_branch = FALSE),
    
    -- Version chain for immutability
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    
    UNIQUE(vault_id, deposit_year)
);

CREATE INDEX idx_escrow_vault ON escrow_deposits(vault_id);
CREATE INDEX idx_escrow_status ON escrow_deposits(status);
```

---

#### 4.2.31 question_categories
```sql
CREATE TYPE question_owner_type_enum AS ENUM ('SYSTEM', 'USER', 'VAULT');

CREATE TABLE question_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID REFERENCES vaults(id) ON DELETE RESTRICT,
    owner_type question_owner_type_enum NOT NULL DEFAULT 'USER',
    owner_profile_id UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    display_order INTEGER DEFAULT 0,
    system_category_number INTEGER,
    created_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    CONSTRAINT system_no_vault CHECK (owner_type != 'SYSTEM' OR vault_id IS NULL),
    CONSTRAINT vault_requires_vault_id CHECK (owner_type != 'VAULT' OR vault_id IS NOT NULL),
    CONSTRAINT user_categories_no_vault CHECK (owner_type != 'USER' OR vault_id IS NULL),
    CONSTRAINT user_categories_require_owner CHECK (owner_type != 'USER' OR owner_profile_id IS NOT NULL)
);
CREATE INDEX idx_question_categories_vault ON question_categories(vault_id);
CREATE INDEX idx_question_categories_owner ON question_categories(owner_profile_id);
```

#### 4.2.32 questions
```sql
CREATE TYPE question_source_type_enum AS ENUM ('SYSTEM', 'VAULT', 'USER');
CREATE TYPE question_status_enum AS ENUM ('ACTIVE', 'DEPRECATED', 'ARCHIVED');

CREATE TABLE questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID REFERENCES vaults(id) ON DELETE RESTRICT,
    source_type question_source_type_enum NOT NULL DEFAULT 'USER',
    text TEXT NOT NULL,
    follow_up_prompt TEXT,
    placeholder_answer_hint TEXT,
    primary_category_id UUID REFERENCES question_categories(id) ON DELETE RESTRICT,
    system_category_number INTEGER,
    sensitivity_flag VARCHAR(50),
    adaptive_filter_rules JSONB DEFAULT '{}',
    display_order INTEGER DEFAULT 0,
    status question_status_enum NOT NULL DEFAULT 'ACTIVE',
    created_by_profile_id UUID REFERENCES profiles(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    device_fingerprint_at_creation VARCHAR(255),
    session_id_at_creation UUID,
    geo_country_at_creation VARCHAR(100),
    geo_region_at_creation VARCHAR(100),
    ip_hash_at_creation VARCHAR(64),
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    branching_reason TEXT,
    CONSTRAINT system_questions_no_vault CHECK (source_type != 'SYSTEM' OR vault_id IS NULL),
    CONSTRAINT user_questions_no_vault CHECK (source_type != 'USER' OR vault_id IS NULL),
    CONSTRAINT vault_questions_require_vault CHECK (source_type != 'VAULT' OR vault_id IS NOT NULL)
);
CREATE INDEX idx_questions_vault ON questions(vault_id);
CREATE INDEX idx_questions_source ON questions(source_type);
CREATE INDEX idx_questions_category ON questions(primary_category_id);
CREATE INDEX idx_questions_creator ON questions(created_by_profile_id);
CREATE INDEX idx_questions_status ON questions(status);
CREATE INDEX idx_questions_sys_cat ON questions(system_category_number) WHERE system_category_number IS NOT NULL;
```

#### 4.2.33 question_category_assignments
```sql
CREATE TABLE question_category_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_id UUID NOT NULL REFERENCES questions(id) ON DELETE RESTRICT,
    category_id UUID NOT NULL REFERENCES question_categories(id) ON DELETE RESTRICT,
    vault_id UUID REFERENCES vaults(id) ON DELETE RESTRICT,
    assigned_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    assigned_at TIMESTAMPTZ DEFAULT NOW(),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    display_order INTEGER DEFAULT 0,
    assignment_type VARCHAR(20) NOT NULL DEFAULT 'secondary',
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    UNIQUE(question_id, category_id)
);
CREATE INDEX idx_qca_question ON question_category_assignments(question_id);
CREATE INDEX idx_qca_category ON question_category_assignments(category_id);
```

#### 4.2.34 question_overrides
```sql
-- User-scoped personalisation of system questions. The original is NEVER modified (QB-001).
CREATE TABLE question_overrides (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- base_question_id MUST point to a SYSTEM question only (enforced at service layer per QOV-1)
    base_question_id UUID NOT NULL REFERENCES questions(id) ON DELETE RESTRICT,
    scoped_to_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    vault_id UUID REFERENCES vaults(id) ON DELETE RESTRICT,
    custom_text TEXT,
    custom_follow_up TEXT,
    custom_placeholder TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    device_fingerprint_at_creation VARCHAR(255),
    session_id_at_creation UUID,
    geo_country_at_creation VARCHAR(100),
    geo_region_at_creation VARCHAR(100),
    ip_hash_at_creation VARCHAR(64),
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ,
    UNIQUE(base_question_id, scoped_to_profile_id)
);
CREATE INDEX idx_question_overrides_base ON question_overrides(base_question_id);
CREATE INDEX idx_question_overrides_profile ON question_overrides(scoped_to_profile_id);
```

**Note:** `question_overrides` is intended for SYSTEM questions only. Service-layer validation must reject `base_question_id` pointing to non-SYSTEM questions (per QOV-1).

#### 4.2.35 question_set_configs
```sql
CREATE TYPE question_source_mode_enum AS ENUM ('ALL', 'SYSTEM_ONLY', 'CUSTOM_ONLY', 'SYSTEM_PLUS_CATEGORY');

CREATE TABLE question_set_configs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vault_id UUID NOT NULL REFERENCES vaults(id) ON DELETE RESTRICT,
    owner_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE RESTRICT,
    name VARCHAR(255),
    source_mode question_source_mode_enum NOT NULL DEFAULT 'ALL',
    included_category_ids UUID[],
    excluded_question_ids UUID[],
    pinned_question_ids UUID[],
    qbook_answer_space_style qbook_answer_space_enum DEFAULT 'ruled',
    qbook_include_category_headers BOOLEAN DEFAULT TRUE,
    qbook_include_page_numbers BOOLEAN DEFAULT TRUE,
    qbook_cover_title VARCHAR(255),
    qbook_cover_subtitle VARCHAR(255),
    context_type VARCHAR(50),
    context_entity_id UUID,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    version_chain_id UUID NOT NULL REFERENCES version_chains(id) ON DELETE RESTRICT,
    is_deleted_branch BOOLEAN DEFAULT FALSE,
    branched_at TIMESTAMPTZ
);
CREATE INDEX idx_qsc_vault ON question_set_configs(vault_id);
CREATE INDEX idx_qsc_owner ON question_set_configs(owner_profile_id);
CREATE INDEX idx_qsc_context ON question_set_configs(context_type, context_entity_id);
```


### 4.3 content_snapshot Specification

The `content_snapshot` is a canonical serialised JSON representation of the complete entity state at write time. The `integrity_hash` is SHA-256 over that canonical serialisation.

**Canonical serialisation requirements (RFC 8785):**
- Keys sorted lexicographically
- Unicode strings normalised (NFC)
- Number representation deterministic (no scientific notation for integers)
- No insignificant whitespace
- Array order preserved as stored

**TypeScript Interface:**
```typescript
interface ContentSnapshot {
  entity_type: 'block' | 'section' | 'profile' | 'book' | 'chapter' | 'asset' | 'folder' | 
               'time_lock' | 'shared_link' | 'webhook_subscription' | 'contribution' | 
               'b2b_read_grant' | 'book_snapshot' | 'family_reunion_qr' | 'memorial_addendum' | 'escrow_deposit' |
               'question_category' | 'question' | 'question_category_assignment' | 'question_override' | 'question_set_config';
  entity_id: string; // UUID
  vault_id: string; // UUID
  version_number: number;
  timestamp_utc: string; // ISO 8601 with millisecond precision
  payload: BlockPayload | SectionPayload | ProfilePayload | BookPayload | ChapterPayload | AssetPayload | FolderPayload |
           TimeLockPayload | SharedLinkPayload | WebhookSubscriptionPayload | ContributionPayload |
           B2BReadGrantPayload | BookSnapshotPayload | FamilyReunionQRPayload | MemorialAddendumPayload | EscrowDepositPayload |
           QuestionCategoryPayload | QuestionPayload | QuestionCategoryAssignmentPayload | QuestionOverridePayload | QuestionSetConfigPayload;
  privacy_tier?: 'public' | 'family' | 'immediate' | 'named' | 'private' | 'locked';  // Optional for non-content entities
  named_viewers?: string[]; // UUID array
  parent_entity_id?: string; // UUID
  child_entity_ids?: string[]; // UUID array
  referenced_asset_ids?: string[]; // UUID array
}
```

**SectionPayload:**
```typescript
interface SectionPayload {
  title: string;
  section_number: number;
  privacy_tier: 'public' | 'family' | 'immediate' | 'named' | 'private' | 'locked';
  chapter_id: string; // UUID
  vault_id: string; // UUID
  // Branch state for Ghost Mode verification
  is_deleted_branch: boolean;
  branched_at?: string; // ISO 8601 timestamp
  branching_reason?: string;
}
```

**Additional Payload Interfaces for Versioned Entities:**

```typescript
interface TimeLockPayload {
  lock_type: 'DATE' | 'CONDITIONAL' | 'DEATH_PROTOCOL';
  unlock_date?: string;
  unlock_condition?: string;
  recipient_profile_id: string;
  status: 'PENDING' | 'TRIGGERED' | 'RELEASED' | 'DEFERRED_REVERSAL' | 'DEFERRED_QUIET' | 'SEALED' | 'REVOKED';
}

interface SharedLinkPayload {
  token: string;
  entity_type: string;
  entity_id: string;
  status: 'ACTIVE' | 'EXPIRED' | 'REVOKED' | 'CONSUMED';
  max_uses?: number;
  use_count: number;
  expires_at?: string;
}

interface WebhookSubscriptionPayload {
  endpoint_url: string;
  subscribed_events: string[];
  status: 'ACTIVE' | 'PAUSED' | 'DISABLED' | 'FAILED';
}

interface ContributionPayload {
  contributor_profile_id: string;
  contribution_type: 'STORY' | 'INTERVIEW_SESSION' | 'MEMORY' | 'BIOGRAPHY_EDIT' | 'CONSTITUTION_AMENDMENT';
  target_entity_type?: string;
  target_entity_id?: string;
  status: 'PENDING' | 'APPROVED' | 'REJECTED' | 'EDITED_WHILE_PENDING';
  content?: string;
}

interface B2BReadGrantPayload {
  institution_id: string;
  grant_scope: string;
  scope_entity_id?: string;
  expires_at: string;
  status: 'ACTIVE' | 'PENDING_REVIEW' | 'EXPIRED' | 'REVOKED';
}

interface BookSnapshotPayload {
  book_id: string;
  name: string;
  snapshot_type: 'USER_SNAPSHOT' | 'PRINT_VERSION' | 'GHOST_MODE_RECONSTRUCTION' | 'QUESTION_BOOK';
  status: 'PENDING' | 'READY' | 'EXPIRED';
  included_block_ids: string[];
}

interface FamilyReunionQRPayload {
  qr_code_token: string;
  event_name?: string;
  max_scans?: number;
  scan_count: number;
  status: 'ACTIVE' | 'EXPIRED' | 'REVOKED' | 'SUSPENDED_DEATH';
}

interface MemorialAddendumPayload {
  deceased_profile_id: string;
  contributor_profile_id: string;
  entry_type: string;
  content: string;
  status: 'PENDING_MODERATION' | 'APPROVED' | 'REJECTED';
}

interface EscrowDepositPayload {
  deposit_year: number;
  status: 'PENDING' | 'DEPOSITED' | 'VERIFIED' | 'FAILED';
  storage_provider: string;
  storage_path: string;
  file_size_bytes?: number;
  sha256_hash?: string;
}
```

**Critical:** Non-RFC-8785 serialisation is a critical bug — two calls to hash the same entity must always produce identical bytes, or Ghost Mode hash verification will emit false TAMPERED flags.

---

## 5. Configuration Management

### 5.1 PlatformConfig Interface

All previously hardcoded business rules are configurable via the PlatformConfig registry:

```typescript
interface PlatformConfig {
  // Free Tier Limits
  free_tier_max_profiles: number;              // default: 5
  free_tier_max_books: number;                 // default: 3
  free_tier_max_chapters_per_book: number;     // default: 10
  free_tier_max_audio_minutes: number;         // default: 60
  free_tier_max_video_minutes: number;         // default: 0 (video is paywalled)
  free_tier_max_storage_mb: number;            // default: 1000
  free_tier_max_ai_queries_monthly: number;    // default: 50
  
  // Death Protocol Timing
  death_witness_window_hours: number;          // default: 4
  death_reversal_window_hours: number;         // default: 24
  death_quiet_period_days: number;             // default: 7
  death_reunion_code_expiry_days: number;      // default: 90
  vault_heir_response_timeout_days: number;    // default: 30 (UNAVAIL-001)
  vault_heir_deferral_max_count: number;        // default: 2
  vault_heir_deferral_max_days: number;         // default: 30
  
  // Revenue Share
  revenue_share_creator_percent: number;       // default: 70
  revenue_share_platform_percent: number;      // default: 30
  revenue_escrow_transition_hold_days: number; // default: 90 (heir acceptance period)
  revenue_unclaimed_property_months: number;   // default: 12 (unclaimed property holding)
  
  // Rate Limits
  api_rate_limit_per_minute: number;           // default: 100
  api_rate_limit_per_hour: number;             // default: 5000
  webhook_retry_max_hours: number;             // default: 72
  offline_queue_ttl_days: number;              // default: 30
  keeper_unavailability_threshold_hours: number; // default: 72 (UNAVAIL-001)
  
  // Accessibility
  cognitive_mode_min_font_size: number;        // default: 20
  interview_mode_default_pin_length: number;   // default: 6
  interview_pin_max_attempts: number;          // default: 5
  read_to_me_default_speed: number;            // default: 1.0
  
  // API Support
  api_version_support_months: number;          // default: 24
  api_deprecation_notice_days: number;         // default: 90
  
  // Regional/Legal
  data_residency_default_region: string;       // default: 'eu-west-1'
  gdpr_erasure_propagation_hours: number;      // default: 72
  sar_response_deadline_days: number;          // default: 30
  minor_consent_verification_days: number;     // default: 14
  
  // Question Bank
  free_tier_max_custom_questions: number;       // default: 50
  free_tier_max_custom_categories: number;      // default: 5
  max_question_text_length: number;             // default: 500
  question_bank_ai_suggestions_monthly: number; // default: 20

  // Content
  low_quality_answer_threshold: number;        // default: 0.3
  transcription_fallback_timeout_seconds: number; // default: 10
  print_order_recall_minutes: number;          // default: 30
  vault_export_max_profiles_batch: number;     // default: 500
  
  // Contribution Review Workflow (Section 6.3.2)
  contribution_reminder_days: number;          // default: 14 (reminder sent to Vault Keeper)
  contribution_escalation_days: number;        // default: 21 (Family Admin may act)
  contribution_delay_notice_days: number;      // default: 30 (contributor notified)
  contribution_resubmission_limit_per_day: number; // default: 3 (rate limit per contributor per section)
  contribution_resubmission_window_hours: number;  // default: 24
  
  // Vault Heir Notification Cadence (Section 6.5.1)
  heir_reminder_days: number;                  // default: 14 (reminder sent to heir)
  heir_timeout_days: number;                   // default: 30 (heir position lapses)
  
  // B2B Read Grants (B2B-006)
  b2b_grant_default_days: number;              // default: 30
  b2b_grant_max_days: number;                  // default: 365
  b2b_grant_reapproval_window_days: number;    // default: 14 (after stewardship transfer)
  
  // Escrow retry policy
  escrow_deposit_retry_hours: number;          // default: 24
  escrow_deposit_max_retries: number;          // default: 3
}
```

### 5.2 Configuration Resolution Hierarchy

Configuration values are resolved in this priority order:
1. Jurisdiction-specific override
2. Plan-specific override (Free vs Premium vs Enterprise)
3. Environment-specific value (dev/staging/production)
4. Global default

### 5.3 Background Job Inventory

The following scheduled jobs and timers must be implemented for system correctness:

| Job Name | Schedule | Owner | Description | Config Keys | Failure Handling |
|----------|----------|-------|-------------|-------------|------------------|
| `contribution.reminder` | Daily | Notification Service | Send 14-day reminders for pending contributions | `contribution_reminder_days` | Retry 3x, then escalate to admin |
| `contribution.escalation` | Daily | Notification Service | Escalate 21-day pending to Family Admins | `contribution_escalation_days` | Log only |
| `contribution.delay_notice` | Daily | Notification Service | Notify contributors at 30 days | `contribution_delay_notice_days` | Log only |
| `heir.reminder` | Daily | Death Protocol Service | Send 14-day heir reminder | `heir_reminder_days` | Retry 3x |
| `heir.timeout` | Daily | Death Protocol Service | Timeout heir after 30 days, contact next | `heir_timeout_days` | Log as VAULT_HEIR_TIMEOUT |
| `time_lock.release` | Every minute | Time Lock Service | Release DATE-type time-locks at unlock_date | N/A | Log as TIME_LOCK_RELEASE_FAILED |
| `time_lock.death_check` | Hourly | Death Protocol Service | Check deferred time-locks after reversal/quiet periods | N/A | Alert on-call |
| `b2b_grant.expiry` | Daily | B2B Service | Expire grants past expires_at | N/A | Log as B2B_READ_GRANT_EXPIRED |
| `b2b_grant.reapproval` | Daily | B2B Service | Alert on grants in PENDING_REVIEW past reapproval_deadline | N/A | Log and revoke |
| `webhook.retry` | Exponential | Webhook Service | Retry failed webhook deliveries | `webhook_retry_max_hours` | Disable after max hours |
| `webhook.disable` | Hourly | Webhook Service | Disable endpoints failed >72 hours | `webhook_retry_max_hours` | Notify Vault Keeper |
| `reunion_qr.expiry` | Daily | Sharing Service | Expire QR codes past expires_at | `death_reunion_code_expiry_days` | Log as REUNION_QR_EXPIRED |
| `reunion_qr.death_suspend` | Event-driven | Death Protocol Service | Suspend QR codes on DEATH_PROTOCOL_TRIGGERED | N/A | Immediate |
| `sar.deadline_alert` | Daily | SAR Service | Alert on requests approaching 30-day deadline | `sar_response_deadline_days` | Escalate to DPO |
| `ip_salt.rotation` | Event-driven | Security Service | Rotate IP salt on stewardship acceptance | N/A | Block stewardship completion |
| `escrow.deposit` | Annual | Escrow Service | Create encrypted vault export for escrow | `escrow_deposit_retry_hours`, `escrow_deposit_max_retries` | Alert engineering |
| `escrow.verify` | Weekly | Escrow Service | Verify escrow deposit integrity | N/A | Retry per config |
| `device_fingerprint.purge` | Daily | Privacy Service | Purge old device fingerprints per retention | `device_fingerprint_retention_days` | Log only |
| `search_index.pseudonymize` | Event-driven | Privacy Service | Pseudonymise search index on GDPR erasure | N/A | Retry 3x, alert on-call |
| `ai_training.flag_sync` | Every 5 min | AI Service | Verify No-AI-Training flag propagation | N/A | Alert on-call (SAR-004) |
| `revenue.escrow_release` | Daily | Revenue Service | Release held revenue to heirs after transition period | `revenue_escrow_transition_hold_days` | Log and alert finance |
| `print.order_recall` | Per-order | Print Service | Check 30-minute recall window expiry | `print_order_recall_minutes` | Irreversible after expiry |
| `notification.digest` | User-config | Notification Service | Send daily/weekly digests | User preference | Retry once, mark failed |
| `notification.fatigue` | Weekly | Notification Service | Auto-switch to Weekly Digest after 30 days | N/A | Log only |
| `memorial.anniversary` | Daily | Legacy Service | Send death anniversary/birthday alerts | N/A | Retry 3x |
| `vault.dormant_check` | Daily | Trust & Safety | Flag vaults dormant >1 year | N/A | Manual review queue |

### 5.4 Feature-to-Schema Traceability

| Feature Section | Primary Tables | Events (Section 9) | Config Keys | Contract Rules |
|-----------------|----------------|-------------------|-------------|----------------|
| 6.1 Onboarding | `vaults`, `profiles`, `vault_memberships` | VAULT_CREATED, PROFILE_CREATED, INVITATION_SENT, INVITATION_ACCEPTED | `free_tier_*` | AS-001, AS-003, AS-004 |
| 6.2 Authentication | `vaults`, `profiles`, `activity_logs` | LOGIN_SUCCESS, LOGIN_FAILED, SESSION_EXPIRED, PASSWORD_CHANGED, MFA_ENABLED, MFA_DISABLED | `api_rate_limit_*`, `interview_pin_*` | SEC-001–004, AS-002 |
| 6.3 Story Creation | `blocks`, `sections`, `chapters`, `books`, `folders`, `interview_sessions`, `interview_question_assignments`, `contributions` | BLOCK_CREATED, BLOCK_EDITED, BLOCK_BRANCHED, SECTION_CASCADE_BRANCH, INTERVIEW_SESSION_STARTED, INTERVIEW_SESSION_COMPLETED, CONTRIBUTION_SUBMITTED, CONTRIBUTION_APPROVED, CONTRIBUTION_REJECTED | `low_quality_answer_threshold`, `contribution_*` | IMM-C-001–012, VC-001–004 |
| 6.4 Privacy/Sharing | `blocks`, `shared_links`, `family_reunion_qr`, `relationships` | BLOCK_PRIVACY_CHANGED, SHARED_LINK_CREATED, SHARED_LINK_ACCESSED, REUNION_QR_CREATED, REUNION_QR_EXPIRED, PROFILE_MERGE_INITIATED, PROFILE_MERGE_COMPLETED | N/A | SEC-014–016, AS-002 |
| 6.5 Death Protocol | `time_locks`, `profiles`, `vaults`, `memorial_addenda` | DEATH_PROTOCOL_TRIGGERED, DEATH_PROTOCOL_CONFIRMED, DEATH_PROTOCOL_FINALIZED, VAULT_QUIET_PERIOD_STARTED, VAULT_QUIET_PERIOD_ENDED, MEMORIAL_MODE_ACTIVATED, VAULT_HEIR_STEWARDSHIP_ACCEPTED, TIMELOCK_DEFERRED_DEATH_PROTOCOL, WILL_UNLOCKED, FINAL_LETTER_DELIVERED, MEMORIAL_ADDENDUM_SUBMITTED | `death_*`, `heir_*` | DEATH-001–005, SV-001–003 |
| 6.6 Trust & Safety | `profiles`, `vaults`, `activity_logs` | CONTENT_FLAGGED, CONTENT_REVIEWED, CONTENT_REMOVED, VAULT_SUSPENDED, VAULT_UNSUSPENDED | N/A | M1–M15 |
| 6.7 Offline Mode | `assets` (offline cache) | SYNC_STARTED, SYNC_COMPLETED, SYNC_FAILED, CONFLICT_DETECTED | `offline_queue_ttl_days` | N/A |
| 6.8 Smart Import | `profiles`, `relationships` | GEDCOM_IMPORT_STARTED, GEDCOM_IMPORT_COMPLETED, PROFILE_MERGE_INITIATED | N/A | EDGE-007–008 |
| 6.9 Monetisation | `book_snapshots` | REVENUE_SHARE_PAID, REVENUE_SHARE_CLAWBACK_INITIATED, TAX_DOCUMENT_ISSUED | `revenue_share_*` | PRINT-001 |
| 6.10 B2B | `b2b_read_grants`, `profiles`, `vaults` | B2B_READ_GRANT_ACCESS, B2B_READ_GRANT_EXPIRED, B2B_READ_GRANT_REVIEWED, B2B_READ_GRANT_APPROVED, B2B_READ_GRANT_REJECTED | `b2b_grant_*` | B2B-001–006 |
| 6.11 Developer API | `webhook_subscriptions` | WEBHOOK_DELIVERED, WEBHOOK_FAILED | `webhook_*` | API-001–003 |
| 6.12 Completion/Achievement | `profiles` | MILESTONE_COMPLETED, BADGE_AWARDED | N/A | N/A |
| 6.13 Data Backup | `escrow_deposits` | VAULT_BACKUP_COMPLETED, VAULT_BACKUP_FAILED, VAULT_EXPORTED | `escrow_*` | CFG-001 |
| 6.14 Print System | `book_snapshots` | PRINT_ORDER_SUBMITTED, PRINT_ORDER_CANCELLED, PRINT_PROVIDER_ACCEPTED, PRINT_SHIPPED | `print_order_recall_minutes` | PRINT-001 |
| 6.15 SAR/GDPR | `subject_rights_requests`, `profiles`, `blocks` | SAR_SUBMITTED, SAR_RESPONDED, GDPR_ERASURE_REQUESTED, GDPR_PSEUDONYMIZED, PORTABILITY_PACKAGE_GENERATED, NO_AI_TRAINING_FLAG_SET | `sar_*`, `gdpr_*` | SAR-001–007 |
| 6.16 AI | `blocks`, `profiles`, `mcp_providers` | AI_SUGGESTION_ACCEPTED, AI_SUGGESTION_REJECTED, AI_PATTERN_GENERATED | N/A | AI-001–007 |
| 6.17 Health | `health_records`, `profiles` | HEALTH_RECORD_CREATED, HEALTH_RECORD_ACCESSED | N/A | SEC-007, SAR-003 |

### 5.5 Formal State Machine Definitions

#### 5.5.1 Death Protocol State Machine

```
States: IDLE → WITNESS_PENDING → TRIGGERED → REVERSAL_WINDOW → MEMORIAL → QUIET_PERIOD → STEWARDSHIP_PENDING → ACTIVE

IDLE:
  - death_witness_1_confirms → WITNESS_PENDING (if 2 witnesses configured)
  - death_witness_confirms → TRIGGERED (if only 1 witness)

WITNESS_PENDING:
  - death_witness_2_confirms within 4h → TRIGGERED
  - timeout 4h → WITNESS_TIMEOUT (reset to IDLE)

TRIGGERED:
  - entry: emit DEATH_PROTOCOL_TRIGGERED
  - auto-advance after 0s → REVERSAL_WINDOW

REVERSAL_WINDOW (24h):
  - reversal_requested by any witness → IDLE (emit DEATH_PROTOCOL_REVERSED)
  - timeout 24h → MEMORIAL
  - time_lock releases deferred (SV-002)

MEMORIAL:
  - entry: Memorial Mode activated (frozen biography, addendum enabled)
  - emit MEMORIAL_MODE_ACTIVATED
  - auto-advance after 0s → QUIET_PERIOD

QUIET_PERIOD (7d):
  - entry: suppress non-critical notifications
  - timeout 7d → STEWARDSHIP_PENDING
  - time_lock releases: wills/final letters release to designated recipients
  - emit TIME_LOCK_RELEASED for each released document

STEWARDSHIP_PENDING:
  - vault_heir_accepts → ACTIVE (emit VAULT_HEIR_STEWARDSHIP_ACCEPTED)
  - heir_timeout 30d → contact next heir (emit VAULT_HEIR_TIMEOUT)
  - no heirs remain → PROTECTED_SEALED (emit VAULT_PROTECTED_SEALED_ENTERED)

ACTIVE:
  - entry: stewardship-specific actions
  - rotate IP salt
  - B2B grants → PENDING_REVIEW
  - Vault Heir now has full stewardship authority
```

#### 5.5.2 Contribution Review State Machine

```
States: PENDING → REMINDED → ESCALATED → (APPROVED | REJECTED)

PENDING:
  - T+14d → REMINDED (send reminder to Vault Keeper)
  - reviewer_approves → APPROVED (emit CONTRIBUTION_APPROVED)
  - reviewer_rejects → REJECTED (emit CONTRIBUTION_REJECTED)
  - contributor_edits → PENDING (reset clock, emit CONTRIBUTION_EDITED_WHILE_PENDING)

REMINDED:
  - T+21d → ESCALATED (Family Admin may act on non-private)
  - reviewer_acts → (APPROVED | REJECTED)

ESCALATED:
  - T+30d → notify contributor of delay
  - vault_keeper_acts → (APPROVED | REJECTED)
  - family_admin_approves_non_private → APPROVED

REJECTED:
  - contributor_resubmits → PENDING (rate limit: 3 per 24h)
```

#### 5.5.3 Time-Lock State Machine

```
States: PENDING → TRIGGERED → (RELEASED | DEFERRED_REVERSAL | DEFERRED_QUIET | SEALED | REVOKED)

PENDING:
  - unlock_date reached → TRIGGERED
  - condition_resolved → TRIGGERED
  - death_protocol_triggered → DEFERRED_REVERSAL (if during reversal window)
  - revoked_by_creator → REVOKED

TRIGGERED:
  - no_death_protocol → RELEASED (emit TIME_LOCK_TRIGGERED)
  - death_protocol_quiet_ended → RELEASED
  - recipient_deceased → SEALED (emit TIME_LOCK_SEALED)

DEFERRED_REVERSAL:
  - reversal_window_closes → DEFERRED_QUIET
  - death_reversed → PENDING (reset unlock)

DEFERRED_QUIET:
  - quiet_period_ends → RELEASED

SEALED:
  - next_of_kin_claims → RELEASED
  - court_order → RELEASED (emit VAULT_PROTECTED_SEALED_EXITED)
```

#### 5.5.4 B2B Read Grant State Machine

```
States: ACTIVE → (PENDING_REVIEW | EXPIRED | REVOKED)

ACTIVE:
  - expires_at reached → EXPIRED (emit B2B_READ_GRANT_EXPIRED)
  - revoked_by_keeper → REVOKED
  - stewardship_transfer → PENDING_REVIEW (emit B2B_READ_GRANT_REVIEW_PENDING)

PENDING_REVIEW:
  - new_keeper_approves within 14d → ACTIVE (emit B2B_READ_GRANT_APPROVED)
  - new_keeper_rejects → REVOKED (emit B2B_READ_GRANT_REJECTED)
  - reapproval_deadline reached → REVOKED
```

#### 5.5.5 Public/Shared Link Access State Machine

```
States: ACTIVE → (CONSUMED | EXPIRED | REVOKED)

ACTIVE:
  - guest_accesses_with_valid_token → CONSUMED (if max_uses=1)
  - guest_accesses → use_count++ (if use_count >= max_uses → EXPIRED)
  - expires_at reached → EXPIRED
  - revoked_by_keeper → REVOKED
  - entity_privacy_tier_changed_from_public → REVOKED

CONSUMED:
  - terminal state

Security Invariants:
- Token must be cryptographically random (256-bit entropy)
- Rate limit: 100 requests per IP per hour
- Failed token attempts logged (max 10 per IP per hour)
```

---

## 6. Complete Feature Specifications

### 6.1 Onboarding and Cold Start Experience

The single highest-impact feature for user retention. Every design decision in onboarding must optimise for one outcome: the user hears or sees a family member's voice or story within 10 minutes of account creation.

#### 6.1.1 The First Ten Minutes — Onboarding Flow

1. **Welcome screen:** Tagline, one-sentence value proposition, and a "Start Your Family's Story" call to action.
2. **Account creation:** Email, passkey, or social sign-in. No lengthy forms.
3. **Vault name:** "The [Family Name] Vault" — prefilled from the user's name and editable.
4. **First Profile Wizard:** "Let's start with someone important. Who do you want to preserve first?" — culturally adaptive relationship cards including options such as Parent, Grandparent, Guardian/Elder, Self, and Someone Else.
5. **One First Question:** The selected profile is shown one opening question immediately. For a parent profile: "Where were you born, and what is your earliest memory of that place?" — just one. Inviting, not overwhelming.
6. **Response prompt:** Three buttons — Record Voice / Type Answer / Upload a Photo. If they record a voice memo here, the onboarding is essentially complete.
7. **After first response:** "You've just started preserving [Name]'s story. There are 200 more questions waiting — but there's no rush. Want to invite [Name] to answer some themselves?"
8. **Invitation prompt:** Pre-written invite message, shareable link or SMS or email send.
9. **Dashboard:** Show the vault with one entry already in it. The shelf is no longer empty.

#### 6.1.2 Progressive Disclosure

- Free users see a simplified dashboard — shelf, profiles panel, and a prioritised current-book view while retaining access to all books included in their plan.
- Advanced features (family tree, secret vaults, health archive) are surfaced contextually, not upfront.
- The question bank is introduced gradually — three questions at a time, never a list of a thousand.
- A Family Progress Bar on the dashboard shows completion across all profiles — gamifies gently.
- The simplified dashboard for free users may visually emphasise one current book, but it must surface access to all three free-tier books. The shelf is simplified, not reduced in entitlement.

#### 6.1.3 Re-Engagement Flow

- **Day 3 email:** "You started [Name]'s story — here's the next question for them."
- **Day 7 email:** If an invitee has not responded, prompt the current vault owner to contribute what they know so the story can still progress. This targets the vault owner to contribute about a non-responsive invitee.
- **Day 14 email:** If a profile invitation remains unclaimed, send a distinct claim-state nudge to the profile owner or steward. This targets the profile owner or claim flow to follow up on an unclaimed invitation state. The Day 7 and Day 14 triggers, copy, and intent are distinct.
- **Monthly digest:** "This month in your family's history" — pulls any stories tagged to this calendar month across any era.
- **Anniversary alerts:** Death anniversaries and birthdays of deceased profiles — "Today would have been [Name]'s 82nd birthday. Would you like to add a memory?"
  - Leap year handling: February 29 birthdays and anniversaries alert on February 28 in non-leap years (never silently skipped).
  - Approximate dates (for example, "sometime in March 1940") alert on March 1.
  - Timezone: All anniversary calculations use the Vault Keeper's registered timezone for alert delivery, and the alerting user's local timezone for display.
  - Daylight saving transitions: Alerts are calculated in wall-clock time, not UTC offsets, to prevent double-firing or skipping on daylight saving boundaries.
  - Anniversary snooze: Alerts can be snoozed for 1 day, 1 week, or "skip this year." Snooze state is per-profile per event and resets annually. Permanently dismissing an anniversary alert is not supported.

### 6.2 Authentication and Identity

Full authentication system includes the following:

- Email and password registration with email verification
- Social sign-in: Google, Apple, Facebook (OAuth 2.0)
- Passkey and biometric authentication (WebAuthn)
- Two-factor authentication: TOTP authenticator app, SMS fallback (Twilio), email fallback
- Magic link login with 15-minute expiry
- JWT access tokens (15-minute expiry) plus refresh tokens (90-day rotation, single-use)
- Device trust — remember device 30, 90, or 365 days
- Session management — view all active sessions, revoke any
- Account lockout: 10 failed attempts, 30-minute cooldown
- Password breach detection via HaveIBeenPwned API
- Profile Claim Flow: claim token generation, invitation, account linking, Vault Keeper approval
- **Interview Mode Authentication:** A simplified PIN-only entry mode for elderly or cognitively impaired users assigned to Interview Mode. The Vault Keeper sets the PIN and the interface strips all navigation except the current question prompt. PINs are stored as bcrypt hashes. Minimum PIN length is 6 digits. Failed PIN attempts lock the session after 5 attempts — Vault Keeper must reset. PIN lockout recovery supports remote reset by Vault Keeper or authorised Family Admin and presents a user-visible recovery path.
  - **PIN session expiry mid-recording:** Interview Mode PIN sessions have a 4-hour session expiry (matching the maximum recording duration). If a PIN session expires while a recording is in progress, the following behaviour applies: (1) the active recording is preserved in memory and submitted to the server immediately; (2) the session is paused with a clear on-screen message in large font: "Your session has ended. Your recording has been saved. Please ask [Vault Keeper name] to restart your session."; (3) the partially-completed session enters PAUSED_SESSION_EXPIRED state visible to the Vault Keeper; (4) the Vault Keeper can restart the session for the same question, with the preserved recording available for review. Under no circumstances is an in-progress recording discarded on session expiry.
- **Guardian Login:** When a guardian logs in, they see a "Managing [Child's Name]" banner and all actions are scoped accordingly.
- **B2B Single Sign-On:** Institutional partners authenticate via SAML 2.0 or OIDC against their own identity provider. Must fall back to FamilyVault native authentication if the institution's identity provider becomes unavailable.
- All authentication events logged to audit trail with device fingerprint, session identifier, and hashed IP address.

### 6.3 Story Creation Engine

#### 6.3.1 Interview Mode

A dedicated UX mode designed for non-tech-savvy elderly users, cognitively impaired subjects, or anyone who finds the standard interface overwhelming.

- Interface is stripped to: one large question, three large buttons (Record / Type / Skip), and a progress indicator.
- Font size is minimum 20 points. No navigation chrome. No menus.
- Voice recording is the primary action — one tap to record, one tap to stop.
- Maximum recording duration: 4 hours per single voice recording (approximately 500 MB at standard quality). The UI shows a recording duration counter and warns at 3 hours and 45 minutes.
- A per-question recording limit of 30 minutes is enforced for Interview Mode to prevent runaway sessions.
- After recording, the UI shows a simple waveform and plays it back for confirmation.
- The interviewer cannot see the vault — they can only see the current question and the record button.
- Sessions can be paused and resumed. Progress is saved per question.
- A Family Admin or Vault Keeper assigns questions to an Interview Mode session.
- Completed interview sessions are reviewed and approved before entering the vault.
- **Interview Mode must verify active internet connectivity before starting.** If offline, the session must not start. If connectivity is lost during a session, the session pauses immediately. Partial responses are preserved in memory and submitted when connectivity is restored.

**Baseline Interview Mode interface (Phase 11 delivery):** A functional but unstyled web route (`/interview/[session_id]`) presenting one question at a time with Record, Type, and Skip controls. Phase 19 upgrades this to the full polished accessibility-compliant interface.

#### 6.3.2 Contribution Review Workflow

The following rules apply to all contribution types — story contributions, interview sessions, memory contributions, and biography edits:

- **Review timeout:** If a contribution is not reviewed within 14 days, a reminder is sent to the Vault Keeper and all Family Admins. If not reviewed within 21 days, any Family Admin may review non-private contributions. If not reviewed within 30 days, the contributor is notified of the delay.
- **Contributor edits during review:** If a contributor edits their submission while it is under review, the contribution returns to pending status and the 14-day clock resets. The reviewer is notified of the edit.
- **Review order:** Contributions are reviewed in chronological submission order by default. The Vault Keeper can re-order the review queue.
- **Rejection and resubmission:** A rejected contribution can be resubmitted with modifications. If a rejection reason was provided by the Vault Keeper, that reason is visible to the contributor. The Vault Keeper is not required to provide a reason. If no reason was given, the contributor may send a single request asking for one; the Vault Keeper may decline to explain and that decision is final. There is no limit on resubmission attempts, but rate limiting prevents spam (maximum 3 resubmissions per 24 hours per contributor per section).

#### 6.3.3 Cognitive Accessibility Mode

- Activated per profile by Vault Keeper or Family Admin.
- Questions are rewritten in simple, short sentences with no complex vocabulary.
- Only one sentence displayed at a time — a "Next sentence" button advances the text.
- Voice input is default — text is secondary.
- No session time limits. Auto-save every 30 seconds.
- A "Read to me" mode reads the question aloud using text-to-speech before expecting a response.
- Secret Vault content is not accessible from Cognitive Accessibility Mode or Interview Only Mode. A user in Cognitive Mode may be designated as a Secret Vault heir, but access to the Secret Vault requires switching to Standard mode and completing a guardian-approval step where applicable.


### 6.3.4 Question Bank Management

Accessible from **Library → Question Bank**. Three-pane interface: (1) **Category Sidebar** — system categories (lock icon), custom categories (edit/delete), **+ New Category**; (2) **Question List** — questions with source badge, drag handle, action menu; (3) **Preview Panel** — question as it appears in a session with Override badge and Revert control.

**Create question:** **+ Add Question** → enter text (max 500 chars), optional follow-up and hint, select primary category and visibility (Personal / Vault). → `QUESTION_CREATED` logged with device, session, country, region, IP hash.

**Create category:** **+ New Category** → name, description, scope. → `QUESTION_CATEGORY_CREATED` logged.

**Override system question:** Action menu → **Edit personal copy**. Notice: *"You are creating a personal copy. The original is unchanged for all other users."* → `QUESTION_OVERRIDE_CREATED` logged. Override badge and **Revert to original** always visible.

**Move question:** Drag or use **Move to**. Creates/removes a `question_category_assignments` row — non-destructive. Question continues in its original category with **Also in: [X]** indicator. → `QUESTION_MOVED` logged.

**Source mode selector** (shown on interview start and Question Book creation):
```
[ ALL  |  System only  |  Custom only  |  Select categories... ]
```
Persists in `question_set_configs`. Changes → `QUESTION_SOURCE_MODE_CHANGED` logged.

### 6.3.5 Question Book Builder

A **Question Book** is a printable or downloadable document containing questions — no answers. Access from: **Question Bank → Generate Question Book**, **Book → Print → Question Book**, or **New Interview → Print blank questions first**.

**Step 1 — Source mode:** Select ALL / SYSTEM_ONLY / CUSTOM_ONLY / SYSTEM_PLUS_CATEGORY.

**Step 2 — Select questions:** Browse by category. Toggle individual questions. Drag to reorder. Add inline custom questions. Running count shown.

**Step 3 — Layout:**

| Option | Values | Default |
|--------|--------|---------|
| Answer space | None / Ruled lines / Full page | Ruled lines |
| Category headers | Show / Hide | Show |
| Page numbers | Yes / No | Yes |
| Cover page | Custom title + subtitle / None | None |
| Font size | Standard / Large | Standard |

**Step 4 — Preview:** Paginated on-screen view. Click any question to toggle.

**Step 5 — Generate:**
- **Download PDF** → renders immediately. → `QUESTION_BOOK_DOWNLOADED` logged.
- **Order Physical Print** → enters print pipeline (Section 6.14), `snapshot_type = 'QUESTION_BOOK'`, 30-minute recall per PRINT-001. → `QUESTION_BOOK_PRINT_ORDERED` logged.

Snapshot stored as `book_snapshots` row (`snapshot_type = 'QUESTION_BOOK'`). → `QUESTION_BOOK_GENERATED` logged on creation.

**Physical print spec:** A5 or US Digest softcover, saddle-stitched, 80gsm uncoated white, 16–200 pages.


### 6.4 Privacy and Access Control

#### 6.4.1 Privacy Tiers

The six privacy tiers are defined as follows:

| Tier | Who Can See It |
|------|----------------|
| **Public** | Anyone with a link, including Guests. No login required. |
| **Family** | All Verified Members and above in this vault. |
| **Immediate** | Members with a declared first-degree relationship to the profile that owns the content. "First-degree" means: spouse/partner, parent, child, or sibling as recorded in the Family Tree. In multi-parent or polygamous family structures, all co-parents and their direct children qualify. The Vault Keeper always has access to Immediate-tier content regardless of relationship. For profiles where no first-degree relationships are defined, Immediate behaves identically to Family tier. |
| **Named** | Only the specific individuals in the Named Viewer List for this content item (Phase 8+). |
| **Private** | Only the Vault Keeper and Family Admins with explicit access. |
| **Locked** | Indistinguishable from non-existent content in all API responses. Access requires the specific unlock key or condition. |

Privacy enforcement occurs at four mandatory layers simultaneously:

1. **Database query filter** — only fetch rows the user can see.
2. **Service layer check** — verify entity-level privacy.
3. **Serialiser layer** — strip fields the user cannot see.
4. **Cache key namespacing** — cache entries are per-user, never shared across users with different permission levels.

The Locked privacy tier must produce identical API responses to the non-existent content case. A user must not be able to distinguish between "this content is locked" and "this content does not exist" through any API response, error code, response time, or response size.

**Privacy Tier Inheritance and Conflict Resolution:** Every level of the content hierarchy has its own privacy tier. The following rules govern conflicts between levels:

- **Child can be more restrictive than parent:** A Block inside a Family-tier Book may be set to Private or Named. A user with access to the Book will not see that Block if their permissions are insufficient. The Block simply does not appear in the Book's content listing for that user.
- **Child cannot be less restrictive than parent:** A Block inside a Private Book may not be set to Family or Public. Attempting to set a less restrictive tier on a child entity than its parent returns a 400 error with code PRIVACY_TIER_ESCALATION_FORBIDDEN. The minimum permitted privacy tier for any child entity is its parent entity's tier.
- **Navigation visibility:** If a user has access to a Book but a specific Block inside it is set to Named tier and they are not on the Named list, the Block is entirely absent from that user's view of the Book. No placeholder, no "restricted content" indicator. The absence is silent, consistent with the privacy-first architecture.
- **Locked tier propagation:** If a Book is set to Locked, all child Chapters, Sections, and Blocks within it are treated as Locked regardless of their individual tier settings. Locked at any ancestor level overrides all descendant tiers.
- **Privacy tier changes are logged:** Every privacy tier change at any level is logged as BLOCK_PRIVACY_CHANGED or SECTION_PRIVACY_CHANGED with the reason field required.

#### 6.4.2 Invitation and Viral Growth Loop

- When a profile is tagged in a story, the system generates a personalised invitation: "Your name appeared in a story written by [Vault Keeper]. Click here to see what was written about you and add your own side."
- Invitation links are content-scoped — the recipient sees only the section they were tagged in, plus a call to action to create their own vault or claim their profile.
- **Memory Request feature:** Any Verified Member can send a named person a request for a specific memory. The request arrives as a single prompted question — no login required to respond.
- **Referral system:** When a Verified Member's invitation is accepted and the new user completes onboarding, the referrer receives a "Family Builder" badge and a print credit.
- **Referral fraud detection:** Self-referral is detected by matching the referrer's email domain, hashed IP subnet, and device fingerprint against the referred account. A two-of-three match triggers human review before the referral credit is awarded. Maximum 50 referral credits earned per calendar year per account.
- **Family Reunion mode:** Vault Keeper can generate a QR code for an event. Attendees scan, create accounts, and are automatically connected as family members pending Vault Keeper approval. On death protocol trigger, all active Family Reunion QR codes are immediately expired, automated welcome emails are suppressed, and pending reunion guests are placed in SUSPENDED_PENDING state until the new Vault Keeper reviews them.

#### 6.4.3 Notification Management System

- Every notification type has an on/off toggle per user — globally and per vault.
- **Non-suppressible notifications:** See the canonical list defined in contract rule M7-SEC (Section 10.15). This feature section defers to M7-SEC as the single source of truth. Any additions to the non-suppressible list must be made in M7-SEC first and propagated to Phase 8 and Phase 20 exit criteria. The current canonical list is: new device login alert, password reset confirmation, two-factor authentication code delivery, Subject Access Request submission confirmation, death protocol trigger confirmation, vault stewardship change, account locked or unlocked, and suspicious session activity alerts. These are delivered immediately, bypassing all notification preference filters.
- **Notification frequency:** Immediate / Daily Digest / Weekly Digest / Never — per category.
- **Categories:** Contributions, Invitations, Health Alerts, Milestone Completions, System Alerts, Marketing (always off by default), Anniversaries, and Questions Assigned. **Note: a "Comments" feature is not defined in V1.** The Comments notification category has been removed from this list. If a comments feature is introduced in a future version, it must be fully specified (comment data model, block-level vs. book-level scope, moderation rules, and notification events) before re-adding this category. Any existing "Comments" references in notification settings are deprecated and must be removed from the UI.
- **Quiet Hours:** User sets a time window during which no push notifications are sent. Quiet Hours do not apply to non-suppressible notifications.
- **Notification fatigue protection:** If a user has not opened any notification in 30 days, the system automatically switches them to Weekly Digest and notifies them.
- **Notification digest quality:** The digest shows a curated "3 most important things from this week" rather than every event.
- **Multi-vault digests:** A user with membership in multiple vaults receives separate digests per vault by default unless they enable Combined Digest.

### 6.5 Death Protocol and Legacy System

#### 6.5.1 Death Protocol Human Flow

The death protocol is one of the most sensitive flows in the application. Every design decision must be made with grief in mind.

**Step 1 — Trigger:** A Death Witness (pre-designated by Vault Keeper) clicks "I need to report a death" from any authenticated device. If two Death Witnesses are designated, both confirmations must complete within 4 hours. If only one witness is designated, that single confirmation is sufficient.

**Death Witness unavailability — handling rules:**

- **If a designated Death Witness predeceases the Vault Keeper:** When a Vault profile for a designated Death Witness is marked as deceased (via the standard deceased-marking flow), the system checks whether that profile holds a Death Witness designation in any vault. If it does, the Vault Keeper of the affected vault is immediately notified: "Your designated Death Witness [Name] has been marked as deceased in your vault. Please designate a replacement Death Witness to ensure your vault can be properly transitioned." The deceased witness's designation is flagged DEATH_WITNESS_INVALID. The vault remains fully functional but the Vault Keeper receives a persistent settings warning until they designate a valid replacement. Note: a profile being marked deceased in the vault is treated as a proxy signal for real-world death for this purpose — this is sufficient given that the Vault Keeper controls who is marked deceased.

- **If no Death Witness has ever been designated:** No death protocol trigger via the standard flow is possible. In this scenario, the following alternative trigger paths are available: (a) A Vault Heir may contact FamilyVault Trust and Safety directly with proof of death (death certificate or equivalent) to request a manual protocol initiation. (b) A Family Admin may do the same. Trust and Safety reviews the submission within 5 business days and, if valid, initiates the death protocol manually on behalf of the vault. (c) An estate attorney or designated legal contact with documented authority may also initiate the manual Trust and Safety path. The manual path is subject to the same 24-hour reversal window and subsequent quiet period as the automated path. All manual path actions are logged as DEATH_PROTOCOL_MANUAL_INITIATED (CRITICAL severity).

- **Proactive guidance:** During onboarding and at annual vault health reminders, the platform prompts the Vault Keeper to designate a Death Witness if none is set. The Backup Status panel in Settings > Vault Health shows a red warning if no valid Death Witness is designated.

**Step 2 — Confirmation:** The system shows the Vault Keeper's name and profile photo. "Are you reporting the death of [Name]?" Two buttons: "Yes, this is correct" / "No, go back." No accidental triggers.

**Step 3 — Date:** "When did [Name] pass away?" Date picker. Optional: "I don't know the exact date" — records as approximate.

**Step 4 — Reversal Window:** The death protocol is considered triggered only after the required confirmation set is complete. Once the trigger is finalised, a 24-hour reversal window opens. No irreversible release, webhook, or ownership action may occur during this window.

**Step 5 — Vault Heir Notification:** A private, human-toned message is sent to the Primary Vault Heir after trigger finalisation: "You have been designated as the successor for [Vault Name]. There is no urgency. The vault is safe. When you are ready, you can accept stewardship." Ordered backup heirs are contacted only if the earlier heir declines, **times out (defined as 30 days of no response after the initial notification)**, or is disqualified. At 14 days of no response, a single reminder is sent. At 30 days of no response, the heir is considered to have timed out, their position in the chain lapses for this succession event, and the next heir in the fallback order is contacted. The 30-day response timeout is the canonical heir SLA for stewardship acceptance. This timeout applies equally to backup heirs.

**Step 6 — Memorial Mode Activation:** Memorial Mode activation is deferred until after the 24-hour reversal window closes (i.e., it follows Step 7 in the event that the protocol is finalised and not reversed). If the death protocol is reversed during the reversal window, no Memorial Mode activation occurs and no freezing of the biography book takes place. Once the reversal window closes without reversal, the Vault Keeper's profile enters Memorial Mode: their biography book is frozen against ordinary editing, the last page is automatically left blank — a "final chapter to be written by those who loved them" prompt — and collaborative memorial submissions become allowed through a separate Memorial Addendum area. The frozen biography remains historically stable while new memorial contributions are collected in the memorial addendum.

**Step 7 — Quiet Period:** After the reversal window closes and Memorial Mode is activated, the 7-day quiet period begins. Broader family notifications follow the quiet-period rules configured by the Vault Keeper.

**Quiet period notification recipients:** The Vault Keeper pre-configures who receives the broader family death notification in Settings > Stewardship > Quiet Period Notifications. The configurable options are:
- **All Verified Members and above** (default): every person with a confirmed vault membership role of Verified Member or higher receives a notification after the quiet period ends.
- **Family Admins only:** only the Family Admins receive the post-quiet-period notification; other Verified Members do not.
- **Named list:** only a specific set of individuals configured by the Vault Keeper receive the notification. This uses the same Named Viewer List mechanism as content privacy.
- **No automated notifications:** the Vault Keeper or heir chooses to inform people manually; no system notifications are sent.

If the Vault Keeper never configured this setting before their death, the system defaults to **All Verified Members and above**. Guests and pending members do not receive death notifications. The notification content is human-toned: "[Vault Keeper Name] has passed away. Their family vault has entered memorial mode. You can continue to add your memories and tributes to [Vault Name]." The notification is sent exactly once, at the close of the quiet period.

**Step 8 — Will and Final Letters Release:** If the Vault Keeper created a will or final letters, the system holds them until the quiet period completes (minimum 7 days after reversal-window closure), then releases them to the designated recipients per the Vault Keeper's instructions.

**Step 9 — Handover:** The Vault Heir receives an onboarding guide: "Here is what [Name] left for you. Here is how to take over stewardship of the family vault."

#### 6.5.2 Stewardship Fallback Order

The stewardship fallback order is deterministic:

1. Primary Vault Heir
2. Backup Vault Heir 1
3. Backup Vault Heir 2
4. Pre-designated Acting Family Admin. **Designation UI and schema:** The Vault Keeper designates an Acting Family Admin via Settings > Stewardship > Acting Family Admin. The designation is stored as `acting_family_admin_profile_id` on the vault record. When the Vault Keeper sets or changes this designation, the event ACTING_FAMILY_ADMIN_DESIGNATED is logged (WARN severity, requires reason field). This event must be added to the Activity Log table alongside VAULT_HEIR_DESIGNATED. The designated Acting Family Admin is shown a confirmation prompt at designation time so they are aware of the role. Removing the designation is logged as ACTING_FAMILY_ADMIN_DESIGNATION_REMOVED.
5. Longest-serving Family Admin (tiebreaker: explicitly pre-designated acting admin wins; then earliest role-grant event UUID lexical ascending; then lowest lexical user UUID as final deterministic tiebreaker)
6. Earliest approved Verified Member stewardship applicant
7. Protected custodial suspension pending manual legal review

On stewardship acceptance: the vault's forensic IP salt epoch rotates, all active B2B read grants enter pending-review status, and VAULT_HEIR_STEWARDSHIP_ACCEPTED is recorded.

#### 6.5.3 Time-Lock Secrets During Death Protocol

- **Time-lock opens before death protocol trigger:** Content unlocks normally to the designated viewer as configured. No death protocol interaction.
- **Time-lock opens during the 24-hour reversal window:** Unlock is deferred until the reversal window closes. All time-lock interactions with death protocol states are logged as TIMELOCK_DEFERRED_DEATH_PROTOCOL.
- **Time-lock opens during the 7-day quiet period:** Unlock is deferred until the quiet period ends.
- **Time-lock opens after stewardship transfer:** The new Vault Keeper can view the time-lock schedule but cannot change the designated recipient. Legal documents release to the recipients as originally configured by the deceased Vault Keeper.
- **If the designated time-lock recipient is also deceased:** The time-locked content releases to the deceased recipient's next-of-kin chain. If no chain exists, content moves to Protected Sealed State.

#### 6.5.4 Deceased Profile Rights Hierarchy

- Upon marking a profile as deceased, the system prompts: "Who should manage this person's profile rights going forward?" — presents a ranked list of family members.
- The first available person in the next-of-kin chain can approve memory contributions, respond to subject access requests referencing the deceased, request corrections on their behalf, manage profile-level privacy settings within the deceased-rights scope, and approve public-family memorial publication where platform policy requires subject-side approval.
- If the first nominee declines or becomes unreachable, rights advance to the next nominee in the chain without rewriting the chain history.
- If the entire chain is exhausted, authority reverts to the Vault Keeper or current steward. If no steward exists, the request enters protected custodial review.
- **Living Memory Liberation:** An explicit, configurable profile setting (default OFF). It may be pre-set by the Vault Keeper and automatically proposes a privacy change only when the configured years-since-death threshold is reached. Health Archive exclusion from Living Memory Liberation is governed canonically by contract rule HLTH-001 (Section 10.15) — this feature section defers to that rule. Summary: Health Archive data is excluded from Living Memory Liberation regardless of the liberation policy setting, and any code path that changes health data privacy tiers as part of liberation processing is a critical bug.

#### 6.5.4a Catastrophic Succession Failure — Guidance

A scenario in which the Vault Keeper and all designated Vault Heirs and Family Admins die in the same event (e.g., a shared accident) is acknowledged as a possible — if rare — real-world occurrence. The platform cannot fully automate a resolution for this scenario, but must handle it gracefully:

- If the Death Witness triggers the death protocol and the fallback chain reaches step 7 (Protected Custodial Suspension) because no heir or admin is reachable, the vault enters Protected Sealed State per Section 6.5.5.
- Trust and Safety is alerted immediately on VAULT_PROTECTED_SEALED_ENTERED.
- A surviving family member, estate attorney, or other legal representative can contact FamilyVault to initiate a court-authorised successor process.
- **Guidance to Vault Keepers (surfaced in onboarding and annual vault health review):** Vault Keepers are strongly encouraged to designate at least one Death Witness and at least one Vault Heir from a different household or social group than themselves. The vault health dashboard shows a warning if all designated heirs and the Death Witness share the same household address on record. The warning text is: "Your designated successors may all be in the same household. Consider designating at least one heir from a different household to protect your vault in unexpected circumstances."

#### 6.5.5 Protected Sealed State

A vault entity (Secret Vault, time-locked content, or entire vault) enters Protected Sealed State when it cannot be transferred to a legitimate successor and no further automated action can proceed safely.

**Entry conditions:**
- Secret Vault heir is deceased with no backup heir.
- All stewardship fallback chain members have declined and no Verified Member applicant exists.
- Time-lock content's designated recipient is deceased with exhausted next-of-kin chain.

**In Protected Sealed State:**
- All content remains encrypted and preserved.
- No new access keys are issued.
- The state is visible to Trust and Safety with a full audit trail.
- Exit requires: a legal process producing a court-approved successor, a Trust and Safety manual review identifying a valid claimant, or a vault owner pre-designating an "emergency contact" who can initiate a court process.

Event: VAULT_PROTECTED_SEALED_ENTERED (CRITICAL severity) is emitted on entry. VAULT_PROTECTED_SEALED_EXITED (CRITICAL severity) is emitted on exit with court order reference.

### 6.6 Content Moderation and Trust and Safety

- Private-first architecture means most content is never publicly visible — this significantly reduces the moderation surface area.
- For public-tier and shared-link content: automated scanning for illegal content (child sexual abuse material and terrorism content) before storage. Required by law.
- **Intra-family dispute — two distinct routing flows:**
  - **Flow A — Member-versus-Member:** If a family member flags content *written by another family member* (not the Vault Keeper) as harmful or false, the flag is routed to the Vault Keeper for review. The Vault Keeper has 14 days to resolve it; if unresolved, the Trust and Safety team mediates.
  - **Flow B — Subject-versus-Keeper:** If a profile subject flags content *written or curated by the Vault Keeper* about themselves as harmful or false, the flag bypasses the Vault Keeper entirely and goes directly to Trust and Safety triage. This prevents forcing the subject to appeal to the contested party first. Trust and Safety has 14 days to mediate before escalation to the regional data authority pathway.
  These two flows must be implemented as separate system workflows with separate routing, notification, and escalation paths.
- Content that violates platform Terms of Service (illegal content, doxing, non-consensual imagery) may be removed by FamilyVault administrators — this is the only platform override of vault sovereignty.
- All platform-initiated removals are logged permanently, visible to the Vault Keeper, and the Vault Keeper receives the reason in writing.
- **Appeal mechanism:** The Vault Keeper may appeal platform removals within 30 days to a Trust and Safety review panel. The appeal is reviewed by a different Trust and Safety agent than the one who made the original removal decision. A Vault Keeper may submit one appeal per removal; if denied, the decision is final unless new material evidence is submitted. If content is restored after a successful appeal, the Vault Keeper and all Family Admins receive a restoration notification.
- The Trust and Safety admin tool must support: report intake, review, legal hold, escalation, removal, suspension, CyberTipline and NCMEC reporting where applicable, appeal handling, and immutable internal audit trails.
- Rate limiting on invitations and collaboration requests to prevent spam or harassment via the platform.

**Mental Health Content Rules:**

- Mental health history stored in Health Archive is treated with all health data protections. It is never surfaced in general story content or AI pattern cards without health authorisation.
- Suicide mentions or self-harm content in general story blocks outside the Health Archive: when the AI content analysis layer detects such content, it does not remove or flag it for removal. However, if the content is in a public or shared tier, it is escalated to Trust and Safety for review.
- FamilyVault does not have a duty-of-care relationship with users that triggers mandatory reporting to authorities for historical health content. However, if real-time content suggests an imminent risk of harm to a currently active user, the platform's crisis pathway must be defined before the Health Archive goes live.
- Historical self-harm content in vault content is treated as personal history and is not subject to automatic removal. It is subject to the same privacy tiers as all other content.

### 6.7 Offline Mode and Conflict Resolution

- Mobile app supports offline mode: read all cached content, create new blocks, record voice and video.
- Offline writes are queued in a local SQLite store with: content, timestamp, block identifier, vault identifier, and session token.
- On reconnection, the offline queue is synced to the server.
- **Conflict resolution:** Last Write Wins is not used. If two users edited the same block while one was offline, both edits are preserved as competing versions. The Vault Keeper is presented with a side-by-side merge view and must resolve manually.
- **Three-or-more-way conflicts:** If N users each edited the same block while offline, all N versions are preserved as competing branches. The Vault Keeper sees an N-way merge view showing all versions with timestamps, authors, and diff highlights. For media blocks, competing recordings cannot be merged — the Vault Keeper must choose one as primary (others become archived versions).
- If the Vault Keeper does not resolve a conflict within 30 days, a reminder notification is sent. After 90 days unresolved, the conflict is escalated to Family Admin for resolution.
- Unresolved conflicts never block vault use — they display as a warning badge on the affected block.
- Offline status indicator: a persistent banner shows "You are offline — changes will sync when reconnected."
- Offline data is encrypted at rest on the device using the vault's derived key. If the device is wiped, offline data is lost — this is intentional security behaviour, documented to users.
- **Health archive blocks in offline mode:** Health records use a separate per-profile health key (SEC-007, DB-010) that is not derived from the vault master key and is not cached on the device. Therefore, **health-related blocks (Question and Answer blocks from Category 10, Health Record entries) cannot be created in offline mode.** If a user attempts to create a health-type block while offline, the system returns an offline-prohibition error: "Health records cannot be created offline. Please reconnect to save health information." This prohibition applies regardless of whether the user holds a `health_authorized` grant. The offline queue system must explicitly exclude health block types from its accepted content queue.
- **Interview Mode sessions are prohibited from operating in Offline Mode.** Interview Mode must detect offline state at session start and prevent launch.
- **72-Hour Vault Keeper Unavailability Threshold (Rule UNAVAIL-001):** This threshold is implemented once and reused by approval, review, and conflict-resolution flows. Defined as follows:
  - **Trigger:** The 72-hour clock starts when an action is queued for the Vault Keeper's sole decision and no Vault Keeper login or API activity is detected. Applicable contexts: (a) a contribution pending Keeper review, (b) a conflict resolution awaiting the Keeper's manual merge, (c) an intra-family dispute flag awaiting the Keeper's response.
  - **What it unlocks:** After 72 hours of Keeper unavailability, any Family Admin may act as reviewing authority for non-private contributions and non-private conflict resolutions. For subject-versus-keeper disputes, the 72-hour threshold escalates directly to Trust and Safety.
  - **Notification:** At the 72-hour mark, all Family Admins are notified that temporary acting authority has opened. The Vault Keeper is also notified of the threshold trigger.
  - **Restoration:** Once the Vault Keeper logs back in, acting authority reverts to them for any items not yet resolved by a Family Admin.
  - **Does not apply to:** Secret Vault decisions, death protocol decisions, or stewardship transfers — these always require the Vault Keeper directly or the formal death protocol chain.

### 6.8 Smart Import and Deduplication Engine

- **GEDCOM import:** Parses GEDCOM 5.5 and 5.5.1. Maps to FamilyVault profile schema. Creates pending profiles for all individuals. Preserves original GEDCOM as an archive file. GEDCOM health-related imports (cause of death, medical condition extensions) route into the Health Archive as imported health data pending review, not into the general biography by default.
- **Photo import:** Bulk import from Google Photos, iCloud, and phone camera roll. AI detects faces and suggests profile tagging.
- **Contact import:** From phone contacts and Google Contacts. Creates pending profiles. No automatic invitation — Vault Keeper reviews first.
- **Deduplication:** Before any import creates a profile, the system checks for: same name plus approximate birth year, or same name plus same relationship type. Presents matches as "Is this the same person?" — merge or create new.

**GEDCOM deduplication edge cases:**
- **Partial overlap between two GEDCOM imports:** Present as candidate merge with field-by-field comparison showing which import has more complete data.
- **Name changes (maiden versus married):** Match on both names if `maiden_name` field is populated; present as likely match for Vault Keeper confirmation.
- **Conflicting birth dates:** Show both dates and ask which is authoritative; the other is stored as an "alternate date" annotation.
- **Circular references in GEDCOM (A is child of B, B is child of A):** Detected during parse, flagged as GEDCOM_CIRCULAR_REF error. The conflicting relationship is not imported and is shown in the import error report.

**Profile Merge Rules:**
- **Pending plus Pending merge:** All content attributed to either pending profile is re-attributed to the merged profile. Reversible for 14 days.
- **Pending plus Claimed merge:** The claimed profile's identity is canonical. The pending profile's content is migrated. The claimant must approve the merge.
- **Claimed plus Claimed merge:** Not permitted without explicit written consent from both claimants. Either claimant may decline, in which case the profiles remain separate with a cross-reference note.
- Post-merge content retains its original author attribution. No content changes ownership as a result of a profile merge.
- Story Graph anti-spam: if the same name is mentioned three or more times in stories, only one pending profile is created and one invitation is triggered (not per mention).

### 6.9 Monetisation System

#### 6.9.1 Subscription Plans and Paywall Timing

- **Free tier:** Genuinely useful — 5 profiles, 3 books, text blocks, photo captions, basic questions, and **voice recording** (available as of the Phase 6 media release). Voice recording is a free-tier feature. The paywall moment must feel earned, not imposed.
- **Paywall triggers:** Attempting to add a sixth profile; attempting to add **video**; attempting to create a secret vault; attempting to print; attempting to access the full question bank. Voice recording is explicitly *not* a paywall trigger — it is included in the free tier.
- At each paywall trigger, the upgrade prompt is contextual: "You've created 5 profiles — your family is growing. Upgrade to add unlimited profiles and never miss a story."
- **Trial:** 30-day free trial of the Family plan triggered at the first paywall moment, not at signup. This maximises trial usage — users have context for the value before they trial.
- Annual billing is shown by default (savings highlighted). Monthly is available but not prominent.
- The Legacy plan is shown only after the Family plan has been active for three or more months.

#### 6.9.2 Revenue Share Mechanics

- Section-level monetisation: Vault Keeper can set any section as paid access. Price set by Vault Keeper ($0.99 to $99 per access).
- Revenue split: 70% to Vault Keeper, 30% to FamilyVault. Stripe processes all payments.
- Tax: FamilyVault issues a 1099-K (United States) or equivalent regional form to any Vault Keeper who earns more than the applicable threshold in a calendar year. Vault Keepers must provide a tax identifier to enable monetisation.
- Revenue is accrued in USD. For Vault Keepers with non-USD bank accounts, Stripe performs conversion at the time of payout using the then-current exchange rate.
- **Outstanding revenue on vault transfer:** Accrued but unpaid revenue transfers to the new Vault Keeper at the next monthly billing cycle (billing cycles run on the first of each calendar month).
- **On death:** Outstanding revenue is held for 90 days while the Vault Heir assumes stewardship. The 90-day hold clock runs concurrently with the stewardship fallback chain, not sequentially. If stewardship is not accepted within 90 days, revenue escrow moves to the 12-month unclaimed property track. If stewardship is not resolved within 12 months, the revenue is held in trust per applicable jurisdiction law and FamilyVault retains the 30% platform share. The 70% Vault Keeper share is reported to the relevant tax authority as unclaimed.
- Revenue dashboard: Vault Keeper sees earnings per section, total earned, pending payouts, and tax documentation status.
- All revenue-share defaults, section-pricing bounds, and developer API rate-limit defaults must be documented in governed commercial policy and referenced by configuration, not hidden as hardcoded constants.

### 6.10 B2B and Institutional Use Cases

- **Target verticals:** Funeral homes, elder care facilities, hospice organisations, estate law firms, oral history archives, libraries, and genealogical societies.
- **B2B plan:** Institution purchases a block of vaults (10, 50, 100, or 500). Each vault is assigned to a client family. Institution has a B2B Partner Admin role.
- **Funeral home use case:** Staff creates a vault for a recently deceased person. Invites family to contribute memories. At 30, 60, or 90 days post-death, a curated memory book is auto-compiled and offered for print purchase.
- **Elder care use case:** Facility resident is assigned a vault. An Interviewer (facility staff member) conducts monthly Interview Mode sessions. Family members receive quarterly updates.
- **Hospice organisation use case:** A hospice care provider is assigned a block of vaults for current patients. Each resident at or near end-of-life is assigned their own vault. An Interviewer (hospice staff member, volunteer, or social worker) conducts guided Interview Mode sessions to capture the resident's life story, final messages, and legacy wishes — often over multiple short sessions suited to the resident's capacity. Family members are invited as Verified Members to contribute their own memories and tributes. The Death Witness designation is particularly important in this context: a trusted hospice staff member or the patient's designated family contact is pre-designated as Death Witness so that the death protocol triggers promptly and without requiring family members to navigate the platform under acute grief. The Memorial Mode and Final Letters features allow residents to leave structured goodbyes. The print-on-demand feature allows the hospice to offer families a memorial book at the time of death, compiled from vault content. Hospice-tier B2B accounts include a pre-configured vault template with the Interview Mode sessions and Final Letters prompts pre-populated for end-of-life context.

- **Estate law use case:** Attorney creates a vault section containing the will, letter of wishes, and property documentation. Time-locked until death protocol triggers. Attorney is notified upon trigger.
  - If death protocol never triggers (no designated witnesses, witnesses unreachable), time-locked legal documents remain locked indefinitely. The attorney or designated legal contact may manually request Trust and Safety review.
  - If the attorney loses account access before the death event, the attorney role can be transferred by the Vault Keeper at any time. The Vault Keeper is notified 30 days before any B2B attorney access grant expires.
  - All legal documents in the estate law use case should be configured with a Conditional Secret (condition: death protocol resolved) rather than a date-based time lock.
- **Library and archive use case:** Institution creates vaults for community oral history projects. Public-tier export for research.
- **B2B billing:** Annual per-vault licensing. Volume discounts at 50 or more vaults. Custom SLAs for 500 or more vault enterprise clients.
- **B2B branding:** Institution's logo and colours on the vault shell (white-label option). The "powered by FamilyVault" attribution remains visible to end users in all interfaces and data exports.
- **B2B data isolation:** No cross-vault data sharing between a B2B institution's clients. Full data isolation at the database level with row-level security.
- **B2B Read Grants:** See the canonical rules defined in contract rule B2B-006 (Section 10.12). This feature section defers to B2B-006 as the single source of truth. Any changes to Read Grant durations, lapse rules, or re-approval windows must be made in B2B-006 first. Summary: 30-day default duration, 365-day maximum, pending-review on stewardship transfer, 14-day re-approval window, no auto-renewal on expiry during transition.

### 6.11 Developer API and Third-Party Ecosystem

- **Public API:** Authenticated via OAuth 2.0 with defined scopes. Vault Keeper grants API access per scope.
- **Scopes:** vault:read, profiles:read, stories:read, stories:write (requires Keeper approval per write), media:read, health:read (requires special health scope grant).
- **Webhooks:** Developers can subscribe only to explicitly granted events. Sensitive events such as `death.protocol.triggered` require specific steward approval and fire only after the reversal window closes. Webhook delivery uses HMAC-SHA256 signing. Failed deliveries are retried with exponential backoff for up to 72 hours. After 72 hours, the Vault Keeper is notified and the webhook endpoint is automatically disabled.
- **SDK:** TypeScript and JavaScript SDK published to npm. Python SDK planned post-V1.
- **Rate limits:** Free API: 100 requests per hour. Paid API: 10,000 requests per hour. Enterprise: custom.
- **API marketplace:** Third parties may list approved integrations in the FamilyVault marketplace.
- **API changelog:** All breaking changes announced minimum 6 months in advance. Version 1 is supported for minimum 24 months after version 2 launch.
- A valid OAuth token does not bypass role-based permission checks. OAuth scope checks are a separate, additional permission layer evaluated after all standard role and privacy checks.

### 6.12 Content Completion and Achievement System

- **Profile Completion Score:** Calculated as (answered questions / total applicable questions) × 100. Shown on every profile card. Low-quality answers (single character, whitespace-only) are detected server-side, flagged as LOW_QUALITY_ANSWER, and do not increment the answered count.
  - **Denominator behaviour:** The denominator (total applicable questions) is recalculated dynamically whenever profile data changes in a way that affects adaptive question filtering (e.g., `marital_status` changes, `religion_affiliation` is set, `is_adopted` is toggled). This means the completion percentage may decrease when the user adds profile information that unlocks new applicable questions. This is expected and correct behaviour. The UI must communicate this to the user with a tooltip or contextual note: "Adding more profile details unlocks additional relevant questions. Your completion score reflects how much of your full story has been captured." The denominator is never fixed at profile creation time — it always reflects the current set of applicable questions.
- **Vault Completion Score:** Average across all profiles. Shown on dashboard.
- **Milestones:** First Story, First Voice Memo, First Video, First Family Member Invited, First Secret Vault, All 5 Default Stories Started, 50 Questions Answered, 100 Questions Answered, First Print Order.
- **Milestone rewards:** Print credits, profile badges, unlocking of advanced decorative features (not paywalled features).
- **Family Leaderboard (optional, opt-in):** Shows which family members have contributed the most stories. Gentle, never competitive.
- **Streak tracking:** Consecutive days with at least one vault activity. Shown as a subtle indicator.
- "This vault is X% complete" is shown to Vault Keeper on dashboard, including a "What's missing?" breakdown.

### 6.13 Data Backup Transparency Dashboard

Accessible from Settings > Vault Health. Shows:

- Last full backup timestamp
- Number of backup copies
- Geographic locations of backups (region-level, not exact)
- Last successful restore test date
- Data integrity score (hash verification status)
- Export format: FVAF version
- Escrow status: last deposit date
- Last search-index re-index time for erasure-sensitive operations
- Status of escrow package verification

Email notification if backup has not run in 72 hours. One-click export: Vault Keeper can trigger an immediate full export at any time.

**Export format:** FVAF (FamilyVault Archive Format) — a ZIP containing all content as JSON, all media as original files, and a Health Bundle encrypted with export-time wrapped keys where health data is included. A self-contained reader HTML is included so the archive can be browsed offline without the application or FamilyVault infrastructure.

### 6.14 Print Order System

- Print options: softcover, hardcover, spiral. Page size A4, US Letter, or US Trade.
- Print preview: paginated WYSIWYG view before ordering.
- Print providers: Lulu Direct API (primary), Blurb (secondary).
- Print order version snapshot rules are defined canonically in contract rule PRINT-001 (Section 10.15). This section defers to PRINT-001 as the single source of truth. Summary: snapshot at payment confirmation, stored as PRINT_VERSION_SNAPSHOT, subsequent edits do not affect the order, multi-book orders snapshot at the same timestamp, 30-minute recall window, mandatory content-list review checkbox, order irreversible after 30 minutes. Any changes to these rules must be made in PRINT-001 first.
- Deceased profile content in print: if a print order includes content about a deceased profile, the privacy tier at time of print confirmation is applied. Memorial-tier content is not included unless the Vault Keeper explicitly selects it.

---

## 7. Technology Stack

| Layer | Technology | Purpose | Rationale |
|-------|-----------|---------|-----------|
| Frontend Framework | Next.js 14 (App Router) | Web application | Server-side rendering and static site generation, file routing, excellent performance |
| Mobile | React Native (Expo) | iOS and Android | Code share with web, strong media support |
| User Interface Library | Tailwind CSS + shadcn/ui | Component styling | Utility-first, consistent design system |
| State Management | Zustand + React Query | Client state and server sync | Lightweight, predictable, excellent caching |
| Backend Framework | Node.js + Fastify | REST and WebSocket API | High performance, TypeScript-native |
| Database (Primary) | PostgreSQL 16 | Relational data plus JSONB | ACID compliant, JSONB for flexible metadata |
| Database (Search) | Typesense | Full-text search | Fast, typo-tolerant, privacy-friendly, self-hostable |
| Database (Cache) | Redis | Session cache, rate limiting, queues | Industry standard, pub/sub support |
| File Storage | Cloudflare R2 | Documents, images, audio | S3-compatible, no egress fees |
| Video Storage | Cloudflare Stream | Video upload, transcoding, delivery | Adaptive bitrate, global CDN |
| CDN | Cloudflare | Global content delivery | Best-in-class edge network |
| Authentication | Custom + Passkey (WebAuthn) | Authentication | Passwordless-first, maximum security |
| B2B Authentication | SAML 2.0 / OIDC | Institutional SSO | Enterprise identity provider compatibility |
| Encryption (at Rest) | AES-256-GCM | Data at rest | Industry standard symmetric encryption |
| Encryption (in Transit) | TLS 1.3 | Data in transit | Modern, forward-secret |
| Encryption (Vault) | Per-vault key (AES-256-GCM) | Vault-level end-to-end encryption | Vault key derived from owner credential and wrapped for stewardship transfer, escrow recovery, and authorised offline export use |
| Queue and Jobs | BullMQ (Redis-backed) | Background jobs | Reliable, retryable, observable |
| Email | Resend (primary), SendGrid (fallback) | Transactional email | Developer-first, React Email templates |
| SMS | Twilio | SMS, two-factor authentication, voice | Reliable, global, well-documented |
| Push Notifications | FCM + APNs | Mobile notifications | Platform-native reliability |
| Payments | Stripe | Subscriptions plus one-time payments | Industry standard, excellent webhooks |
| Print | Lulu Direct API (primary), Blurb (secondary) | Print-on-demand | Global printing, quality hardcovers |
| AI (Large Language Model) | Claude (primary), GPT-4o (fallback) | Narrative AI, pattern detection | Via MCP capability layer |
| AI (Transcription) | Whisper (primary), Google STT (fallback) | Voice-to-text | High accuracy, self-hostable option |
| AI (Orchestration) | Internal MCP Service | Provider-agnostic AI routing | Allows provider swap without code change |
| Monitoring | Sentry + Datadog | Errors and APM | Full observability stack |
| Analytics | PostHog (self-hosted) | Product analytics | Privacy-first, self-hostable |
| CI/CD | GitHub Actions | Automated testing and deployment | Native GitHub integration |
| Infrastructure | Fly.io + Cloudflare Workers | Hosting and edge compute | Low latency, global distribution |
| Container | Docker | Consistent environments | Industry standard |
| Offline (Mobile) | SQLite (Expo SQLite) | Local offline queue | Device-native, encrypted |
| Testing | Vitest + Playwright + Supertest | Unit, end-to-end, API tests | Comprehensive coverage stack |
| Documentation | Mintlify | API docs and guides | Auto-generated from code |
| TypeScript | 5.x strict mode | Type safety across stack | Prevents runtime errors |
| Export Format | FVAF (open standard) | Portable vault archive | Self-contained, application-independent |
| Data Escrow | Annual deposit to escrow agent | 100-year stewardship guarantee | Trust through legal commitment |

---

## 8. Build Phases

FamilyVault is built in 21 phases — Phase 0 through Phase 20 inclusive — over approximately 20 to 24 months for a full production release, with a usable MVP available at Phase 5. Each phase produces independently deployable, tested, and documented software. No phase begins until the previous phase has passed all exit criteria.

| Phase | Name | Duration | Team | Gate |
|-------|------|----------|------|------|
| 0 | Foundation and Infrastructure | 3 weeks | 2 Engineers | — |
| 1 | Authentication and Identity | 3 weeks | 2 Engineers | — |
| 2 | Profile System | 4 weeks | 3 Engineers | — |
| 3 | Core Story Engine (Text) | 5 weeks | 4 Engineers | — |
| 4 | Immutability Engine | 4 weeks | 3 Engineers | — |
| 5 | MVP Release | 2 weeks | Full team | **MVP** |
| 6 | Media System (Voice and Video) | 5 weeks | 4 Engineers | — |
| 7 | Family Tree and Relationships | 4 weeks | 3 Engineers | — |
| 8 | Privacy and Access Control | 4 weeks | 3 Engineers | — |
| 9 | AI Integration Layer | 6 weeks | 3 Engineers + AI Specialist | — |
| 10 | Health Archive and Disease Intelligence | 3 weeks | 2 Engineers + AI Specialist | — |
| 11 | Collaboration and Contribution System | 4 weeks | 3 Engineers | — |
| 12 | Monetisation and Print System | 5 weeks | 3 Engineers | — |
| 13 | MCP Provider System and Integrations | 5 weeks | 3 Engineers | — |
| 14 | Onboarding, Viral Growth, and Achievements | 3 weeks | 3 Engineers + Product Designer | — |
| 15 | Subject Rights, Moderation, and Stewardship | 3 weeks | 2 Engineers + Legal Consultant | — |
| 16 | B2B and Developer API | 4 weeks | 3 Engineers | — |
| 17 | Mobile App (React Native) | 8 weeks | 3 Engineers | — |
| 18 | Offline Mode and Conflict Resolution | 4 weeks | 2 Engineers | — |
| 19 | Cognitive Accessibility and Interview Mode | 3 weeks | 2 Engineers + UX Designer | — |
| 20 | Production Hardening and Launch | 3 weeks | Full team | **V1.0** |

**Total estimated build time:** 85 weeks (approximately 20 months) with a team of 6 to 8 engineers. MVP (Phases 0 through 5) is achievable in 21 weeks (approximately 5 months).

---

### Phase 0 — Foundation and Infrastructure

**Duration:** 3 weeks | **Team:** 2 Engineers

**Objectives:** Establish the complete development environment, infrastructure topology, CI/CD pipeline, database schemas, and core shared libraries. No feature work begins until this phase is complete.

**Entry Criteria:**
- Engineering team hired and onboarded
- Source code repository created and access configured
- Cloud provider accounts provisioned (Cloudflare, Fly.io, Neon or Supabase)
- Domain names registered and DNS configured

**Key Deliverables:**
- Monorepo scaffold (Turborepo): apps/web, apps/api, apps/mobile, packages/shared, packages/db, packages/types
- TypeScript strict mode configured across all packages
- Core database tables and known table stubs created. Foundational and core tables are created here; later phases may add additional tables where approved data models require them, provided migration and rollback rules are followed.
- Redis and Typesense instances provisioned with empty index schemas
- Fastify API scaffold with Zod validation, rate limiting, CORS, and health check
- Next.js 14 frontend scaffold with App Router, Tailwind, shadcn/ui, React Query, and Zustand
- GitHub Actions CI/CD pipeline: lint → type-check → test → deploy
- Shared libraries: /crypto, /audit, /validation, /constants, /logger
- Audit log writer: append-only INSERT-only, verified in tests from day one
- **Health Archive placeholder:** `health_records` table stub with schema (encrypted, separate tablespace), empty in Phase 0. Populated by Phase 2 GEDCOM import and fully operational in Phase 10. This stub ensures GEDCOM health imports in Phase 2 have a valid destination.
- **MCP Provider Interface stub:** `/packages/mcp/capability-interface.ts` defines the canonical capability interface (`llm.generate`, `transcription.transcribe`, `email.send`, `storage.file`, `sms.send`). Phase 6 AI transcription must route through this interface stub from day one — direct SDK calls are forbidden per Contract rule AI-010.

**Before MVP launch at Phase 5, FamilyVault must have a documented and support-tested manual stewardship and death runbook.** Minimum required scenarios: (1) Vault Keeper death — manual heir contact, vault suspension, and protected custody initiation; (2) Emergency access suspension — how to suspend a compromised account without automated tooling; (3) Heir contact — process for reaching designated heirs without automated notification; (4) Protected custody transfer — steps for placing vault in protected custodial state pending legal review. The runbook must be support-tested with at least one dry-run exercise before Phase 5 sign-off.

**Exit Criteria:**
- All database tables created, migration runs cleanly from zero
- API /health returns 200. Frontend deploys to staging URL.
- CI/CD pipeline runs on test commit. All three services communicate.
- Audit log writer creates a verifiable entry. Zero TypeScript errors in strict mode.
- **WebSocket or Server-Sent Events (SSE) layer is established and verified:** A test connection can be opened from the frontend to the API, a test message delivered, and the connection gracefully closed. This is a blocking exit criterion because Phase 4 soft-lock for concurrent editing requires real-time push to other users. The choice of WebSocket versus SSE must be documented in the Phase 0 architecture decision record.

---

### Phase 1 — Authentication and Identity

**Duration:** 3 weeks | **Team:** 2 Engineers

**Objectives:** Build the complete authentication system. Upon completion, users can create accounts and log in.

**Key Deliverables:**
- Email and password registration with email verification (Resend)
- Social sign-in: Google, Apple, Facebook (OAuth 2.0)
- Passkey and biometric authentication (WebAuthn / SimpleWebAuthn library)
- Two-factor authentication: TOTP authenticator app, SMS fallback (Twilio), email fallback
- Magic link login with 15-minute expiry
- JWT access tokens (15-minute expiry) plus refresh tokens (90-day rotation, single-use with rotation)
- Refresh token rotation must tolerate benign concurrent use from the same token family. The implementation must prevent false compromise lockouts caused by network retries or simultaneous mobile requests, while still detecting replay outside the allowed grace semantics.
- Device trust — remember device 30, 90, or 365 days
- Session management — view all active sessions, revoke any
- Account lockout: 10 failed attempts, 30-minute cooldown
- Password breach detection via HaveIBeenPwned API
- Profile Claim Flow: claim token generation, invitation, account linking, Vault Keeper approval
- Interview Mode Authentication: PIN-only simplified entry for designated profiles
- B2B SSO: SAML 2.0 / OIDC configuration for institutional partners. Must fall back to native FamilyVault authentication when the identity provider is unavailable.
- Device fingerprint collection with explicit user consent, as documented in the Privacy Notice acceptance at account creation. Device fingerprint hashes retained for session duration plus 90 days, then purged. DEVICE_FINGERPRINT_PURGED event logged on purge.
- All authentication events logged to audit trail with device fingerprint, session identifier, and hashed IP address

**Exit Criteria:**
- Full authentication flow works: register → verify → login → two-factor authentication → session → refresh → logout
- All 12 authentication event types verified in audit log
- Permission boundary tests: unauthenticated requests return 401
- B2B SSO fallback unit test: SAML/OIDC flow falls back to native FamilyVault authentication when identity provider is unavailable

---

### Phase 2 — Profile System

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build the complete profile system. Upon completion, users can create and manage profiles with all cultural contexts, relationship types, and accessibility modes.

**Key Deliverables:**
- Profile creation: quick-create, full-create, relationship wizard
- Culturally adaptive or neutral default profile flows at onboarding (Parent, Grandparent, Guardian/Elder, Self, Someone Else)
- GEDCOM import: parse GEDCOM 5.5 and 5.5.1, map to profile schema, create pending profiles
- GEDCOM health-related imports route to Health Archive review queue, not into general biography by default
- Contact import from Google Contacts and phone contacts
- Duplicate detection: alert if name plus birth year or name plus relationship matches existing profile
- Profile types: SELF / FAMILY / FRIEND / NEIGHBOUR / COLLEAGUE / MENTOR / DECEASED / PENDING
- Cultural context fields: `cultural_context`, `lineage_clan_name`, `naming_convention`
- Deceased Profile Mode: memorial mode, freeze, collaborative memory, anniversary alerts
- Pending Profile Mode: created by mention, "Unresolved Characters" dashboard widget
- Cognitive mode field: STANDARD / SIMPLIFIED / INTERVIEW_ONLY / LARGE_FONT
- Subject rights fields: `sar_opt_in`, `no_ai_training_flag`, `subject_rights_email`, `next_of_kin_chain`, `under_13_flag`, `guardian_consent_status`, `memorial_liberation_policy`
- Under-13 profile controls and guardian consent flow
- `calendar_preference` field supporting GREGORIAN, HIJRI, HEBREW, LUNISOLAR, and OTHER
- Profile sections: Basic Info, Biography Book, Personal Drive, Era Timeline, Health, Relationships, Media Gallery, Mentions Feed, Questions Queue, Activity Feed
  - **Personal Drive** — A private-by-default document storage area attached to the profile. It holds documents, notes, and files that belong to this individual but do not fit neatly into the narrative Biography Book. It is distinct from the Biography Book in that it has no chapter or question structure — it is a flat file store organised by folders. Default privacy tier: Private. It has its own privacy tier independent of the Biography Book, its own version chain per stored document, and is excluded from print orders unless explicitly included by the Vault Keeper. It does not appear in Story Graph cross-references.
  - **Mentions Feed** — A system-generated, read-only feed that aggregates all content blocks across the vault where this profile's name has been detected (via the name-detection engine). It is event-driven: the feed updates within 5 minutes whenever a new block mentioning the profile is published or approved. When a Pending Profile is claimed, the Mentions Feed for the newly claimed profile is generated from all historical mentions already in the Story Graph. When a mention is removed or its block is branched (deleted), the corresponding entry is removed from the Mentions Feed. The Mentions Feed is a view over Story Graph edges filtered to this profile — it is not a separate data store.
- Family tree graph structure: nodes plus edges with relationship type and direction
- Social web overlay: non-blood relationships mapped separately from family tree

**Exit Criteria:**
- Can create profile of every type. GEDCOM import creates pending profiles correctly.
- Duplicate detection fires on matching name plus relationship.
- Deceased mode freezes profile and logs freeze event.
- Under-13 controls enforced at privacy-tier and processing-decision time.

---

### Phase 3 — Core Story Engine (Text)

**Duration:** 5 weeks | **Team:** 4 Engineers

**Objectives:** Build the complete text-based story creation engine. Upon completion, users can write, edit, and structure family stories with the full question bank.

**Key Deliverables:**
- Full content hierarchy: Folder → Book → Chapter → Section → Block
- Rich text editor (Block type: Text): bold, italic, headings, inline links, footnotes
- All 16 block types as defined in Section 3.1.4 — including Interview Transcript and B2B-Authored Content block type stubs required for immutability compliance from first use
- Interview Mode data structures: `block_type: INTERVIEW_TRANSCRIPT`, `interview_session` table, `interview_question_assignment` table created as schema stubs. Full Interview Mode UX ships in Phases 11 and 19.
- Block operations: create, edit, reorder, pin, lock, privacy-change — all with version nodes and audit log
- The reason field enforced on every edit, delete, and privacy change — server-side validation
- Question bank: all 10 categories of 100 questions each, loaded and categorised
- Adaptive question filtering: hides irrelevant questions based on profile data
- Five guided default story flows generated from culturally adaptive or neutral relationship labels. The five flows are:
  1. **The Beginning** — Focused on childhood, origins, and earliest memories. Draws from Category 1 (Childhood and Origins) and Category 9 (Culture and Identity). Applies to all profile types.
  2. **The Work** — Focused on career, purpose, and public life. Draws from Category 4 (Work and Purpose) and Category 8 (Wisdom and Legacy). Applies to all adult profile types.
  3. **The Family** — Focused on relationships, love, marriage, and lineage. Draws from Category 3 (Love and Marriage) and Category 5 (Family and Lineage). Adaptive: questions about spouse deprioritised for NEVER_MARRIED status; questions about children deprioritised for childless profiles.
  4. **The World** — Focused on the historical era the person lived through. Draws from Category 7 (World and History) and Category 6 (Hardship and Resilience). Adaptive: era-relevant world event questions drawn from the profile's birth year and location.
  5. **The Legacy** — Focused on what the person wants remembered and passed on. Draws from Category 8 (Wisdom and Legacy) and Category 2 (Faith and Religion). Religion questions deprioritised if `religion_affiliation` is None.
  Each flow contains 10–15 curated questions drawn from the adaptive question bank. Flows are presented as a guided starting point — the subject may answer any or all questions, skip freely, and return to incomplete flows at any time.
- Name detection in text: scans for names matching known profiles or pending profiles
- Story Graph: tracks cross-references between profiles based on name mentions
- Anti-spam identity resolution: multiple mentions of same name → one pending profile
- Era timeline: blocks tagged to `era_year`, displayed on decade-by-decade timeline. **`era_year` field definition:** The `era_year` field is stored as a structured object with two sub-fields: `era_year_start` (integer, a specific calendar year, e.g. 1947) and `era_year_end` (integer, optional — if set, represents a year range; if omitted, the era is treated as a single year). A human-readable era label (free text, maximum 100 characters, e.g. "Post-war Britain" or "1940s childhood") may optionally be stored alongside the year value as `era_label`. The timeline display groups blocks by decade using `era_year_start` for positioning. Blocks with a year range span the relevant decade cells. The `era_year_start` field is what search and timeline queries filter on. Category 7 (World and History) adaptive filtering uses `era_year_start` to surface era-relevant world events. For blocks where the era is approximate or unknown, `era_year_start` may be set to null — these blocks appear in an "Undated" section of the timeline.
- Vault Constitution: a special co-authored root document. **Data model:** The Vault Constitution is not a standard Block or Book. It is a dedicated document entity with its own `vault_constitutions` table, with the following schema: `vault_id` (UUID, foreign key), `content` (JSONB, stores the constitutional text as an ordered array of named sections, each section being a Text-block-compatible rich-text node), `version_chain_id` (UUID, links to its VersionChain — the Constitution has its own append-only version chain), `created_at`, `updated_at`. The Constitution is created automatically when a vault is created (empty by default). It is edited via a dedicated Constitution editor, not via the standard block editor. Write permissions follow the rules defined in Section 3.1.1 (Vault Keeper and Family Admin can write directly; Verified Members and Contributors submit proposals via the contribution review workflow). The CONSTITUTION_UPDATED event is fired when any approved edit is committed. The Constitution does not appear in the standard shelf view or book navigation — it is accessible only via Settings > Vault > Constitution.
- Vault shelf user interface: book covers, spine summaries, folder organisation
- Completion percentage: per profile and vault-wide
- Block-level deduplication: SHA-256 of normalised text content. If a block hash matches an existing block in the same vault within the last 30 days, the user is warned.

**Exit Criteria:**
- Can create full content hierarchy from vault to block. Rich text editor works.
- Version node created on every write — verified in immutability test suite.
- Reason field rejected if empty — API test verifies 400 response.
- Question bank accessible. Adaptive filtering hides irrelevant questions.

---

### Phase 4 — Immutability Engine

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build and harden the complete immutability and version chain system. This phase validates that the core architectural promise holds under all conditions.

**Key Deliverables:**
- Version chain: VersionChain and VersionNode tables, append-only at application level
- Character-level diff: Myers diff algorithm, stored as independently verifiable diff
- SHA-256 integrity hash: computed and stored on every VersionNode using RFC 8785 Canonical JSON serialisation
- Tamper detection: hash verified on read for Ghost Mode and version history. Mismatch → TAMPERED flag → Vault Keeper notification. Tamper detection must never silently fail.
- Ghost Mode: render any entity as it appeared at any historical timestamp — byte-for-byte identical to stored hashes
- Deleted Archive: first-class browsable interface showing deleted item, original location in dot-notation, actor, timestamp, reason, and surrounding context
- Branch restoration: creates new VersionNode on current head — does not modify historical chain
- Block reorder undo: previous order stored in VersionNode. Up to 20 reorder operations can be undone within a single editing session. Standard Ctrl+Z / Cmd+Z keyboard shortcut. **Event log distinction:** A reorder undo creates a BLOCK_REORDER_UNDONE event (not BLOCK_REORDERED), so the audit log can distinguish deliberate reordering from undo operations. A reorder undo does create a new VersionNode on the version chain — the event type on that node is BLOCK_REORDER_UNDONE. This allows Ghost Mode and the audit log to accurately represent the user's intent.
- Immutability test suite: dedicated test suite covering all 12 IMM-C rules, runs on every pull request — non-skippable
- Database permission enforcement: app user has INSERT-only on `activity_logs`. Hard-delete forbidden at database permission level.
- Media anchor immutability: character position plus 50-character context preserved. Orphaned media moves to Orphaned Archive, never deletes.
- Soft-lock for concurrent editing: a content block being actively edited by User A is soft-locked for 15 minutes (visible as "Being edited by [Name]"). User B can read but not edit. Hard conflicts resolved via the offline merge flow in Phase 18.

**Exit Criteria:**
- All 12 IMM-C contract rules verified by automated test suite
- Ghost Mode produces byte-identical output verified against stored hashes
- Hard-delete attempt on any content table fails at database permission level
- Deleted Archive shows all branched items with full metadata

---

### Phase 5 — MVP Release

**Duration:** 2 weeks | **Team:** Full team | **Gate: MVP**

**Objectives:** Integrate, harden, and deploy Phases 0 through 4 as a coherent, usable product. The MVP is a text plus photo plus question-driven family story vault with full immutability and privacy controls.

**MVP Feature Set:**
- Account creation, login, two-factor authentication, and session management
- Create up to 5 profiles (Free tier limit)
- Create up to 3 books with chapters, sections, and text and photo blocks
- Question bank accessible — basic categories 1 through 6, 8, and 9 included in Free tier. Full question bank (all 10 categories including Health and Body) requires upgrade. Accessing a premium-only category triggers the paywall upgrade prompt.
- Five guided default story flows
- Name detection and pending profile creation
- Full immutability: version history, deleted archive, Ghost Mode
- Privacy tiers: Public, Family, Immediate, Private, Locked. **Note: the Named tier is excluded from the MVP.** The Named label exists in the enum schema from Phase 3, but Named Viewer List management (the UI and API that make the Named tier functional) ships in Phase 8. In the MVP, setting content to Named tier is not permitted — the tier selection control hides the Named option until Phase 8. Attempting to set Named via the API returns a 403 with error code FEATURE_NOT_YET_AVAILABLE. The Named tier is fully enabled as part of Phase 8 exit criteria.
- Invitation flow: invite family members to profiles
- Era timeline: view content by decade
- Vault shelf view and book navigation

**MVP Exclusions (Post-MVP phases):**
- Voice and video recording (Phase 6)
- AI features (Phase 9)
- Secret vaults and time locks (Phase 8)
- Print and monetisation (Phase 12)
- Mobile app (Phase 17)

**Exit Criteria:**
- End-to-end test suite passes all MVP user flows
- Performance: all pages load under 2 seconds on a 4G connection
- Security: penetration test on authentication and privacy systems performed by an external security firm — zero critical findings, all high findings resolved
- Manual stewardship and death runbook documented, support-tested, and dry-run completed
- Product Owner signs off after real-family user testing session

---

### Phase 6 — Media System (Voice and Video)

**Duration:** 5 weeks | **Team:** 4 Engineers

**Objectives:** Add voice and video recording, upload, transcoding, playback, and transcription to the platform.

**Key Deliverables:**
- Voice recording: in-app recorder, waveform visualisation, one-tap record and stop
- Maximum recording duration: 4 hours per single voice recording. The UI shows a recording duration counter and warns at 3 hours and 45 minutes.
- Video recording: in-app camera, or upload from device
- Cloudflare Stream: video upload, adaptive bitrate transcoding, global delivery
- Cloudflare R2: audio storage with signed URL serving (15-minute expiry)
- Block types: Voice, Video, Mixed (text plus voice plus video plus photo in one block)
- Transcription flow: record → prompt → queue for transcription → review → replace block or annotate
- AI transcription (Whisper primary, Google STT fallback) — must route through the MCP Provider Interface stub created in Phase 0 (`packages/mcp/capability-interface.ts`). Direct Whisper or Google SDK imports in application code are forbidden.
- Crowdsource transcription: Transcriber role assigned per audio or video item
- Transcriber isolation: Transcriber can access only the isolated media asset, transcription tools, and assigned metadata — never surrounding story content
- Asset checksums: MD5 plus SHA-256 computed at upload. Corruption detection on serve.
- Media anchors: character position plus context, updated as surrounding text changes
- Orphaned Media Archive: media whose anchor text was deleted
- Media gallery per profile: all photos, audio, and video in chronological grid
- Video transcoding failure handling: if Cloudflare Stream transcoding fails, the original upload is preserved in R2. After 3 failed transcode attempts, the asset is flagged TRANSCODE_FAILED and the Vault Keeper is notified.

**Exit Criteria:**
- Voice, video, mixed, and interview-transcript block types can be created, versioned, restored, and rendered in Ghost Mode.
- Transcriber isolation is verified: the transcriber can access only the isolated media asset, transcription tools, and assigned metadata, never surrounding story content.
- Orphaned Media Archive is browsable, retains original context, and can restore anchors without data loss.
- Media-version rules are tested for caption-only edits, transcript-only edits, and asset replacement while preserving prior binary history unless a legal-erasure exception applies.

---

### Phase 7 — Family Tree and Relationships

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build the visual interactive family tree and the three-layer relationship system.

**Key Deliverables:**
- Family Tree (Layer 1): bloodline mapping, navigable visual graph, click any node to enter profile. First-class product feature with permission-scoped traversal and culturally flexible relationship labels.
- Social Web (Layer 2): non-blood relationships mapped as a separate overlay
- Story Graph (Layer 3): auto-generated from name mentions — organic cross-reference web
- Parallel View: split-screen two family members' timelines side by side
- Era overlay: see what two people were doing in the same year
- Cross-reference detection: "Dad mentioned Uncle Roy in 3 stories — connect them?"
- Relationship badge system: shown on all profile cards
- Non-Western family structures: polygamous, clan-based, extended, and multi-parent relationships all supported
- Ancestor profiles: no death date required. Treated as active members of the family tree.
- VaultLink federation: shared profile display rules between family tree, social web, and story graph verified for consistency

**Exit Criteria:**
- Family Tree is available as a first-class user interface feature with permission-scoped traversal and culturally flexible relationship labels.
- Shared-profile display rules between all three layers are verified for consistency.
- No cross-profile or cross-vault relationship leak occurs when the requester lacks permission to see the linked profile.

---

### Phase 8 — Privacy and Access Control

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build the complete privacy enforcement stack, secret vaults, time locks, and all access control systems.

**Key Deliverables:**
- Privacy tiers enforced at all 4 layers: database query, service, serialiser, and cache key namespace
- Locked tier: indistinguishable from non-existent content in API response and error codes
- Secret Vault: separate encrypted storage with a dedicated per-vault secret-storage key. Explicit secret-heir configuration or explicit fallback acknowledgement is required on activation.
- Time-Lock Secrets: content unlocks on a specified date
- Conditional Secrets: content unlocks when a condition is manually resolved by a designated resolver
- Named Viewer Lists: cherry-pick individuals for specific content
- Shared Links: scoped, expirable, revocable. When a shared link is revoked, all currently active Guest sessions using that link are terminated within 60 seconds.
- Contribution scoping: invite a contributor to a specific section only
- Notification management system: per-type toggles, frequency settings, Quiet Hours, fatigue protection, non-suppressible security notification list
- Family Reunion QR code permission model: QR-linked guest account permissions established here. The QR code user interface feature itself ships in Phase 14. On DEATH_PROTOCOL_TRIGGERED event, all active reunion codes for the vault are immediately set to EXPIRED status (synchronously, in the same transaction). Automated welcome emails are suppressed. Pending-approval reunion guests are placed in SUSPENDED_PENDING state.
- All 13 roles mapped through the permission matrix and tested against key entity classes

**Exit Criteria:**
- All 13 roles are mapped through the permission matrix and tested against key entity classes.
- Security notifications remain non-suppressible while marketing and engagement notifications obey user settings and legal unsubscribe rules.
- Secret Vault activation enforces explicit secret-heir configuration or explicit fallback acknowledgement, and Family Reunion revocation behaviour is defined for both pending and already-approved guest access.

---

### Phase 9 — AI Integration Layer

**Duration:** 6 weeks | **Team:** 3 Engineers + AI Specialist

**Objectives:** Build the MCP orchestration service and all AI-powered features.

**Key Deliverables:**
- MCP Orchestration Service: all AI calls routed through internal capability layer
- No provider SDK imported outside MCP layer — enforced at code review
- LLM providers: Claude (primary), GPT-4o (fallback). Auto-failover after 10-second timeout plus 1 retry. On failover: partial results from the failed provider are discarded. The job restarts on the fallback provider from the beginning of the job context.
- AI Writing Assistance: prompt expansion, style matching, suggestion user interface (cannot be accidentally accepted). Suggestions must have a different background colour, a border, and an explicit Accept button that requires deliberate click.
- AI Question Generation: context-aware follow-up questions based on existing answers
- AI Name Detection Enhancement: NLP-powered name detection beyond simple string matching
- Story Graph AI: AI detects implicit relationships not caught by name matching alone
- AI Pattern Cards: family patterns detected across vault content, confidence scores, evidence citations. Displayed on dashboard and eligible profile views. Low-confidence patterns (below 0.4) clearly marked as speculative.
- No-AI-Training flag propagation: flagged profiles excluded from all AI analysis within 5 minutes. Pattern cards derived before a No-AI-Training flag was set must be invalidated and regenerated within 5 minutes of flag activation.
- AI budget tracking: per-vault compute budget, Free plan cap with graceful degradation order (writing assistance degrades first, then follow-up question generation, then NLP name detection falls back to deterministic matching, then Story Graph inference degrades)
- AI transparency: every AI block flagged AI_GENERATED permanently. Accepted = logged as AI_SUGGESTION_ACCEPTED.
- Permission-scoped AI context: AI never sees content the requesting user cannot see
- AI_PROVIDER_FAILOVER logged with: primary provider, failure reason, fallback provider, job type, and retry count

**Exit Criteria:**
- AI degradation order follows the canonical dependency order and does not hard-fail features when budget limits are reached.
- AI Pattern Cards render with confidence and evidence, have a defined placement, and respect dismiss and share controls.
- In-flight No-AI-Training changes are honoured, outputs are discarded, and provider-discard signalling is verified where supported.

---

### Phase 10 — Health Archive and Disease Intelligence

**Duration:** 3 weeks | **Team:** 2 Engineers + AI Specialist

**Objectives:** Build the encrypted health archive and AI-powered health pattern engine.

**Key Deliverables:**
- Health archive: separate encrypted storage with per-profile health key. Separate tablespace from general content tables.
- Health record fields: conditions, cause of death, medications, genetic markers, mental health history
- Health chapter questions: Category 10 of question bank, private by default
- AI Health Pattern Engine: scans across ancestors for recurring conditions
- Health alerts: include pattern description, ancestor profiles, confidence score, and mandatory medical disclaimer. Never presented as diagnosis.
- De-identification before AI calls: names replaced with profile IDs, rehydrated after AI response. Verified by automated test on every AI health call.
- Explicit `health_authorized` access: a separate access grant required to view health data
- Health data excluded from No-AI-Training profiles
- Health Archive exclusion from Living Memory Liberation: Health Archive data is explicitly excluded from Living Memory Liberation regardless of the liberation policy setting. Any code path that changes health data privacy tiers as part of liberation processing is a critical bug.
- Mental health content handling policy must be reviewed by legal counsel and the crisis pathway documented and tested before Health Archive goes live.

**Exit Criteria:**
- Health Pattern Engine analyses only authorised profiles and suppresses outputs that would inferentially reveal inaccessible health data.
- Health export uses wrapped export keys and unlocks locally in the offline FVAF reader without calling FamilyVault services.
- Imported GEDCOM health data lands in Health Archive review queues with correct encryption and access boundaries.

---

### Phase 11 — Collaboration and Contribution System

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build the full collaboration system including invitations, contribution review, co-authorship, and the Death Protocol.

**⚠ Scope Assessment — Required Before Phase 11 Begins:** Phase 11 carries an unusually large deliverable set relative to other phases: contribution invitation, scoped invitations, contribution review workflow, co-authorship, Interviewer role, Interview Mode baseline interface, Witness Statements, the full 9-step Death Protocol UX, Vault Heir designation and fallback chain, Death Witness designation, Will and Final Letters, Memorial Mode, and Deceased Profile Rights Hierarchy. This scope is significantly larger than peer phases. Before Phase 11 begins, the Product Owner and Engineering Lead must conduct a formal scope-split assessment and document one of the following decisions: (a) Phase 11 proceeds as specified with an extended timeline (recommended: 6–7 weeks, not 4) and/or an additional fourth engineer; (b) the Death Protocol and Legacy System deliverables are split into a separate Phase 11b (1–2 weeks, 2 engineers) immediately following Phase 11a; (c) Interview Mode baseline interface is deferred to Phase 19 (where the full polished interface ships anyway) and Phase 11 focuses on Collaboration, Contribution Review, and Death Protocol only. This assessment must be documented and signed off before Phase 11 sprint planning begins.

**Key Deliverables:**
- Contribution invitation: invite any profile to add their side of a story
- Scoped invitations: contribution limited to the specific block or section
- Contribution review workflow: Vault Keeper or Family Admin approves all contributions before publishing (full rules from Section 6.3.2)
- Co-authorship: multiple profiles credited as book and chapter authors
- Interviewer role: assigned per profile, accesses only the Interview Mode interface
- Interview Mode baseline interface: functional web route (`/interview/[session_id]`) presenting one question at a time with Record, Type, and Skip controls
- Witness Statement blocks: co-signed by multiple profiles, cryptographically logged
- Death Protocol: full 9-step human UX flow per Section 6.5.1
- Ordered Vault Heir designation, fallback-chain handling per DEATH-002, and onboarding sequence
- Death Witness designation and one-action trigger interface
- If two witnesses are configured, both must confirm within 4 hours (DEATH-001)
- 24-hour reversal window begins only after trigger finalisation
- Will and final letter creation, storage, and triggered release
- Memorial Mode activation: frozen biography book with separate Memorial Addendum area
- Deceased Profile Rights Hierarchy: `next_of_kin_chain` assignment and deceased-rights delegation
- On stewardship acceptance: IP salt epoch rotates, B2B read grants enter pending-review status, VAULT_HEIR_STEWARDSHIP_ACCEPTED recorded
- Death protocol interaction with Family Reunion QR codes: the QR code expiry-on-death behaviour is tested in Phase 20 (not Phase 11), because the Family Reunion QR code feature itself ships in Phase 14. Phase 11 verifies that the DEATH_PROTOCOL_TRIGGERED event correctly sets the `reunion_qr_expiry_on_death` flag in the vault state machine, and that no QR codes for that vault can be activated while that flag is set. The full end-to-end QR expiry test (SUSPENDED_PENDING guests, suppressed welcome emails) is an exit criterion of Phase 20.

**Exit Criteria:**
- Death protocol executes according to the final state machine: witness confirmation window, reversal window, quiet period, heir sequencing, and stewardship acceptance.
- No-heir and no-admin fallback flows tested end-to-end, including Interim Authority notification copies.
- Memorial Mode freezes biography content while allowing collaborative memorial addendum submissions.
- Contribution review workflow verified: 14-day reminder, 21-day Family Admin escalation, 30-day contributor delay notification, resubmission flow, and rate limiting all tested end-to-end.
- Stewardship acceptance rotates the IP salt epoch and forces review of outstanding B2B read grants. B2B Read Grant stewardship transfer flow is verified: new Vault Keeper receives grant review list, 14-day re-approval window is enforced, and expired grants during transition are logged as B2B_READ_GRANT_EXPIRED.

---

### Phase 12 — Monetisation and Print System

**Duration:** 5 weeks | **Team:** 3 Engineers

**Objectives:** Build the complete monetisation stack, print-on-demand system, and revenue sharing mechanics.

**Key Deliverables:**
- Plan tiers: Free, Family, Legacy — enforced at API level
- Paywall moments: contextual, triggered at natural upgrade points (sixth profile, video attempt, secret vault, print)
- 30-day trial: triggered at first paywall moment, not at signup
- Stripe integration: subscriptions, one-time payments, webhooks, dunning
- Section monetisation: Vault Keeper sets paid access price on any section
- Revenue split: 70/30. Tax document issuance: 1099-K (US) and regional equivalents. Tax obligations follow the Vault Keeper's registered jurisdiction.
- Tax identifier collection: required before enabling section monetisation
- Revenue handling on vault transfer and death protocol including outstanding payout management
- Revenue escrow during stewardship gap per REV-001: 90-day hold, 12-month maximum for unclaimed, then unclaimed property treatment
- Revenue dashboard: earnings per section, pending payouts, and tax documents
- Print-on-demand: Lulu Direct API (primary), Blurb (secondary)
- Print options: softcover, hardcover, spiral. Page size A4, US Letter, US Trade.
- Print preview: paginated WYSIWYG view before ordering
- Print order version snapshot per PRINT-001: snapshot at payment confirmation, 30-minute recall window, mandatory content list review checkbox
- Print credits: awarded via referral program and manual grant
- Referral program mechanics: referral link generation, link tracking, credit award on successful new user completion of onboarding. The "Family Builder" badge ships in Phase 14.

**Exit Criteria:**
- Revenue handling on ownership transfer and death protocol is verified, including pending payouts and grant review prompts.
- Tax-document behaviour is parameterised against jurisdictional rules instead of a hard-coded threshold.
- Print and export privacy enforcement is verified for all privacy tiers and deceased and memorial content states.
- Print order version snapshot verified: content edits after order confirmation do not affect the order. 30-minute recall window tested. Multi-book timestamp consistency verified.

---

### Phase 13 — MCP Provider System and Integrations

**Duration:** 5 weeks | **Team:** 3 Engineers

**Objectives:** Build the full MCP provider registry, capability conflict resolution, and third-party integrations.

**Key Deliverables:**
- MCP registry: provider registration, capability mapping, conflict detection on registration
- Capability conflict resolution: Vault Keeper designates PRIMARY and SECONDARY per capability. Unresolved conflicts block the conflicted capability.
- Provider health dashboard: latency percentiles, error rates, and capability status per provider
- Vault Keeper provider configuration: all AI, email, and SMS providers configurable per vault
- GEDCOM import integration: Phase 13 adds **AI-powered duplicate detection** as an enhancement layer on top of the basic GEDCOM import built in Phase 2. Phase 2 delivers deterministic deduplication (same name plus birth year, same name plus relationship type). Phase 13 delivers semantic or AI-assisted deduplication (variant name spellings, inferred identity from multiple indirect signals). The GEDCOM deduplication rules in Section 6.8 — partial overlap, name changes, conflicting birth dates, and circular references — are Phase 2 deliverables except where explicitly noted as AI-enhanced. The AI duplicate detection in Phase 13 supplements but does not replace Phase 2 logic.
- Google Photos import: face detection for profile tagging suggestions
- iCloud photo import
- Ancestry.com import: GEDCOM bridge
- Email provider switching: Resend to SendGrid with zero downtime
- Transcription provider switching: Whisper to Google STT

**Exit Criteria:**
- Provider switching and capability conflict resolution work without application code changes.
- Search, AI, and provider integrations respect erasure propagation, No-AI exclusions, and re-index requirements.

---

### Phase 14 — Onboarding, Viral Growth, and Achievements

**Duration:** 3 weeks | **Team:** 3 Engineers + Product Designer

**Objectives:** Build the complete onboarding experience, viral growth mechanisms, and achievement system.

**Key Deliverables:**
- First-10-minutes onboarding flow: Welcome → Account → Vault Name → First Profile → First Question → First Response → Invitation Prompt → Dashboard (per Section 6.1.1)
- Progressive disclosure: simplified dashboard for new users, advanced features surfaced contextually. Simplified dashboard still surfaces all free-tier books.
- Re-engagement email sequences: Day 3, Day 7, Day 14 (distinct flows with different targets and intent per Section 6.1.3)
- Anniversary alerts: death anniversaries, birthdays of deceased profiles, with leap year and daylight saving handling
- Monthly "this month in your family's history" digest
- Memory Request feature: send a specific question to a named person without requiring login
- Family Reunion QR code: event-based bulk onboarding (permission model from Phase 8, QR user interface feature here)
- Referral program — Family Builder badge: achievement surface for the referral system built in Phase 12. Phase 14 adds the badge award, dashboard display, and referral-milestone integration.
- Achievement milestone system: First Story, First Voice Memo, First Family Member Invited, and more
- Profile Completion Score and Vault Completion Score
- "What's missing?" breakdown on dashboard
- Viral growth loop: profile tagging → personalised invite → content-scoped view → claim profile call to action
- Onboarding starter profiles must be culturally adaptive or neutral rather than Western-only hard-coded labels

**Exit Criteria:**
- Simplified dashboard still surfaces all free-tier books while maintaining progressive disclosure.
- Day 3, Day 7, Day 14, monthly, and anniversary re-engagement triggers are implemented as distinct flows with legal email controls.
- Onboarding starter profiles are culturally adaptive or neutral rather than Western-only hard-coded labels.

---

### Phase 15 — Subject Rights, Moderation, and Stewardship

**Duration:** 3 weeks | **Team:** 2 Engineers + Legal Consultant

**Objectives:** Build the subject rights system, content moderation framework, and long-term stewardship infrastructure.

**Key Deliverables:**
- Subject Access Request system: any living person named in vault can submit a request
- Subject Access Request queue: managed in Vault > Subject Rights Queue. Vault Keeper reviews and responds. Responses required within one calendar month.
- Correction request flow: subject requests correction → Vault Keeper decides → logged regardless of outcome
- Removal request flow: subject requests removal from shared tiers → Vault Keeper decides → escalation path to data authority if declined
- No-AI-Training flag: Vault Keeper or profile subject sets per profile. Excluded from all AI pattern analysis.
- Guardian consent flow: under-13 profiles require verifiable guardian consent for collection beyond minimal metadata. Ages 13 through 17 retain enhanced shared-tier protections.
- Cross-vault GDPR erasure propagation per FED-003: VAULT_LINK_ERASURE_PROPAGATED events, 72-hour linked vault propagation, daily reconciliation job
- Data portability per SAR-007: claimed profile subjects receive structured machine-readable export of personal data
- Content moderation: automated scanning of public-tier content for child sexual abuse material and terrorism material. Connected to Trust and Safety operations and mandatory CyberTipline and NCMEC reporting workflow where applicable.
- Intra-family dispute flow: flag → Vault Keeper → 14-day resolution window → Trust and Safety escalation
- Platform removal mechanism: Trust and Safety admin tool. Logged permanently. Vault Keeper notified in writing. Appeal mechanism.
- Trust and Safety tool: intake, review, removal, escalation, appeal, immutable logging, legal hold, role-based internal access control
- Data Escrow: automated annual export deposited to escrow agent
- FVAF export standard: open, documented format with self-contained HTML reader
- Backup Transparency Dashboard: live status of all backup and integrity metrics
- Corporate continuity disclosure: visible in Settings > About > Stewardship Commitment
- 90-day wind-down notice system: operational procedure documented and tested
- Public Privacy Notice, Subject Rights Notice, and Stewardship Commitment: live and linked before launch sign-off

**Exit Criteria:**
- Subject-rights workflows cover access, correction, removal, erasure, portability, guardian and under-13 consent, and search-index and cache propagation.
- Public privacy notice, subject-rights notice, and stewardship commitment are live and linked before launch sign-off.
- CSAM detection is connected to Trust and Safety operations and mandatory CyberTipline and NCMEC reporting workflow where applicable.
- Trust and Safety admin tooling supports intake, review, removal, escalation, appeal, immutable logging, and legal hold.

---

### Phase 16 — B2B and Developer API

**Duration:** 4 weeks | **Team:** 3 Engineers

**Objectives:** Build the institutional partner system and public developer API.

**Key Deliverables:**
- B2B plan: vault portfolio management for institutions
- B2B Partner Admin role: manages assigned client vaults, isolated from each other at row-level security
- B2B SSO: SAML 2.0 / OIDC configuration (built in Phase 1, integrated and fully tested here)
- Institutional use case templates: funeral home, elder care, estate law, library and archive
- B2B white-label: institution logo and colours on vault shell. "Powered by FamilyVault" attribution mandatory.
- B2B billing: annual per-vault licensing, volume discounts, custom SLA infrastructure
- B2B Read Grant lifecycle: 30-day default, 365-day maximum, expiry logging, pending-review status on stewardship transfer, 14-day re-approval window
- Public API: OAuth 2.0 with defined scopes
- Webhooks: scope-gated events, HMAC-SHA256 signed, retry logic with exponential backoff up to 72 hours. `death.protocol.triggered` webhook fires only after reversal-window closure and only for explicitly granted subscriptions.
- TypeScript and JavaScript SDK published to npm
- API rate limits: Free (100 per hour), Paid (10,000 per hour), Enterprise (custom)
- API changelog automation: breaking changes detected and documented on every deployment
- Developer documentation: Mintlify-generated API docs with examples

**Exit Criteria:**
- Webhook subscriptions are scope-gated, signed, retry-tested, and sensitive events require explicit opt-in.
- `death.protocol.triggered` webhook fires only after reversal-window closure and only for explicitly granted subscriptions.
- B2B Read Grants support default and maximum durations, expiry, revocation, and re-approval after stewardship transfer.
- B2B SSO fallback to native authentication is tested in this phase and on every B2B-related release.

---

### Phase 17 — Mobile App (React Native)

**Duration:** 8 weeks | **Team:** 3 Engineers

**Objectives:** Build native iOS and Android apps with full feature parity for core vault features.

**Key Deliverables:**
- React Native (Expo) app for iOS and Android
- Full authentication flow including biometric Face ID and Touch ID
- Vault shelf, book navigation, and story creation (text, voice, video)
- In-app audio and video recording with waveform visualisation
- Push notifications: FCM (Android) and APNs (iOS)
- Family tree and profile views
- Interview Mode: dedicated mobile interface — large buttons, large text, one question at a time. Interview Mode on mobile requires active internet connectivity. The interface checks connectivity at session start and prevents launch if offline.
- Offline queue: SQLite-backed write queue for offline content creation
- Encrypted local storage for offline content (vault-derived key)
- App Store and Google Play submission and review management

**Exit Criteria:**
- Mobile authentication survives concurrent refresh races without false compromise logouts.
- Offline queue survives app restarts and can be resumed after re-authentication when token expiry occurs.
- Interview Mode and accessibility flows are verified on supported mobile devices.

---

### Phase 18 — Offline Mode and Conflict Resolution

**Duration:** 4 weeks | **Team:** 2 Engineers

**Objectives:** Build robust offline mode for mobile, including conflict resolution for concurrent edits.

**Key Deliverables:**
- Read all cached content offline: vault shelf, books, profiles
- Create new blocks offline: text, voice, video — queued in local SQLite store
- Offline queue fields: content, timestamp, block identifier, vault identifier, session token
- Offline queue TTL and relogin recovery: queue preserved across relogin, expired-session re-authentication flow, user-facing warnings
- On reconnection: sync queue to server with retry logic
- Conflict detection: two users edited same block while one was offline → competing versions preserved
- Three-or-more-way conflict handling: all N versions preserved as competing branches. Media block conflicts present all recordings for keeper selection. 30-day unresolved conflict reminder and 90-day Family Admin escalation tested end-to-end.
- Side-by-side merge view: Vault Keeper reviews competing versions and resolves manually
- Offline status banner: persistent "You are offline — changes will sync when reconnected"
- Offline data encryption: device-level AES-256 using vault-derived key
- Data loss policy: if device is wiped, offline data is lost — documented and communicated to users
- The 72-hour Vault Keeper unavailability threshold (Rule UNAVAIL-001 — see Section 6.7 for the canonical definition): implemented once and reused by approval, review, and conflict-resolution flows

**Exit Criteria:**
- Offline queue TTL, relogin recovery, expired-session behaviour, and user-facing warnings are implemented and tested.
- The 72-hour Vault Keeper unavailability threshold is implemented once and reused by approval, review, and conflict-resolution flows.
- Conflict flows preserve competing versions across all supported block types, including media and transcripts.

---

### Phase 19 — Cognitive Accessibility and Interview Mode

**Duration:** 3 weeks | **Team:** 2 Engineers + UX Designer

**Objectives:** Build the full polished cognitive accessibility mode and complete Interview Mode user experience.

**Key Deliverables:**
- Cognitive Accessibility Mode: questions rewritten in simple sentences, one at a time, Read-to-Me text-to-speech
- Interview Mode web interface: stripped user interface, one large question, three large buttons (Record, Type, Skip), progress indicator
- Interview Mode mobile interface: same UX, optimised for tablet and large phone
- Minimum font size: 20 points in accessibility modes. No navigation chrome.
- Voice is default input in all accessibility modes — text is secondary
- Auto-save every 30 seconds. No session time limits.
- Interview session review workflow: Family Admin or Vault Keeper reviews completed session before it enters vault. Interviewees notified when session is accepted, rejected, or returned for clarification.
- PIN-only authentication for Interview Mode profiles (set by Vault Keeper)
- PIN lockout recovery: remote reset by Vault Keeper or authorised Family Admin, logged, with user-visible recovery path
- Assignable question sets for Interview Mode: Vault Keeper selects which questions the interviewer asks
- Interview Mode offline prohibition test: verified that Interview Mode sessions cannot be started in offline state. Verified that mid-session disconnection pauses the session (not discards it). Verified that reconnection resumes from last answered question.
- Accessibility testing: tested with elderly user panel. WCAG 2.1 AA compliance across all interfaces.

**Exit Criteria:**
- Interview review service level agreement, reminder and escalation flow, acceptance and rejection notifications, and PIN reset mechanisms are implemented.
- Accessibility modes pass WCAG 2.1 AA and elderly-user validation for both standard and recovery flows.

---

### Phase 20 — Production Hardening and Launch

**Duration:** 3 weeks | **Team:** Full team | **Gate: V1.0 Launch**

**Objectives:** Final hardening, security audit, performance optimisation, and production launch.

**Key Deliverables:**
- External penetration test: all critical and high findings resolved before launch
- Load test: 10,000 concurrent users simulated — all SLAs verified
- Disaster Recovery test: full DR failover to secondary region, RTO verified (less than 4 hours), RPO verified (less than 1 hour). Disaster recovery regions must respect data residency obligations.
- GDPR and privacy compliance audit: designated privacy counsel or DPO reviews all data flows, consent mechanisms, Subject Access Request flows, erasure propagation, portability, and regional minor protections
- COPPA compliance review: minor profile protections, under-13 controls, and guardian-consent verified
- Accessibility audit: WCAG 2.1 AA verified across all interfaces including recovery and accessibility modes
- Final review of all Build Contract rule families — zero open MANDATORY violations
- All 13-role access matrix verified across all key entity types
- App Store and Google Play launch submissions
- Data escrow first annual deposit — FVAF reader verified functional offline and health-bundle local unlock verified
- Public launch: marketing site, press outreach, referral program activation
- IP salt rotation on stewardship acceptance, B2B SSO fallback, media-erasure handling, and search-index pseudonymisation all verified
- Secret Vault heir succession, time-lock death protocol interaction, cross-vault GDPR erasure propagation, and revenue escrow chain all tested end-to-end
- All EDGE resolution register items (EDGE-001 through EDGE-020) implemented with appropriate test coverage per their classification. Specifications for all 20 items are in Section 10.15.

**Exit Criteria:**
- All 12 IMM-C rules, including legal-erasure handling for binary assets, are verified by automated suites and launch review.
- GDPR and privacy sign-off performed by designated privacy counsel or DPO, with data portability, erasure propagation, consent mechanisms, and privacy notices verified.
- Access-control testing covers all 13 roles across all key entity types.
- Phase 20 acceptance criteria are identical in substance to the Build Contract master acceptance criteria (Section 12).
- Product Owner, Legal and Privacy, and Security Lead have all signed off on their respective domains.

---

## 9. Activity Log Event Types

The activity log is append-only and permanent. Every user-initiated and system-initiated action generates an event.

| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Content | BLOCK_CREATED | No | INFO |
| Content | BLOCK_EDITED | Yes | INFO |
| Content | BLOCK_BRANCHED (deleted) | Yes | WARN |
| Content | BLOCK_RESTORED | Yes | INFO |
| Content | BLOCK_REORDERED | No | INFO |
| Content | BLOCK_PINNED | No | INFO |
| Content | BLOCK_UNPINNED | No | INFO |
| Content | BLOCK_PRIVACY_CHANGED | Yes | WARN |
| Content | BLOCK_LOCKED | No | INFO |
| Content | BLOCK_UNLOCKED | Yes | INFO |
| Content | BLOCK_MONETISED | No | INFO |
| Content | BLOCK_DEMONETISED | No | INFO |
| Content | BLOCK_AI_FLAG_SET | No | INFO |
| Content | BLOCK_AI_SUGGESTION_ACCEPTED | No | INFO |
| Content | BLOCK_AI_SUGGESTION_REJECTED | No | INFO |
| Content | BLOCK_REORDER_UNDONE | No | INFO |
| Content | SECTION_CASCADE_BRANCH | Yes | WARN |
| Content | BLOCK_TEMPLATE_INSTANTIATED | No | INFO |
| Content | QUESTION_CREATED | No | INFO |
| Content | QUESTION_EDITED | Yes | INFO |
| Content | QUESTION_BRANCHED | Yes | WARN |
| Content | QUESTION_OVERRIDE_CREATED | No | INFO |
| Content | QUESTION_OVERRIDE_BRANCHED | No | INFO |
| Content | QUESTION_CATEGORY_CREATED | No | INFO |
| Content | QUESTION_CATEGORY_EDITED | Yes | INFO |
| Content | QUESTION_CATEGORY_BRANCHED | Yes | WARN |
| Content | QUESTION_MOVED | No | INFO |
| Content | QUESTION_SOURCE_MODE_CHANGED | No | INFO |
| Content | QUESTION_BOOK_GENERATED | No | INFO |
| Content | QUESTION_BOOK_DOWNLOADED | No | INFO |
| Content | QUESTION_BOOK_PRINT_ORDERED | No | INFO |
| Content | BOOK_SNAPSHOT_CREATED | No | INFO |
| Media | MEDIA_UPLOADED | No | INFO |
| Media | MEDIA_ANCHORED | No | INFO |
| Media | MEDIA_ORPHANED | No | WARN |
| Media | MEDIA_TRANSCRIPTION_QUEUED | No | INFO |
| Media | MEDIA_TRANSCRIPTION_SUBMITTED | No | INFO |
| Media | MEDIA_TRANSCRIPTION_APPROVED | No | INFO |
| Media | MEDIA_TRANSCRIPTION_REJECTED | Yes | INFO |
| Media | MEDIA_DELETED_BRANCH | Yes | WARN |
| Profile | PROFILE_CREATED | No | INFO |
| Profile | PROFILE_EDITED | Yes | INFO |
| Profile | PROFILE_CLAIMED | No | INFO |
| Profile | PROFILE_PHOTO_UPDATED | Yes | INFO |
| Profile | PROFILE_MERGE_INITIATED | No | WARN |
| Profile | PROFILE_MERGE_COMPLETED | No | WARN |
| Profile | PROFILE_MERGE_REVERSED | Yes | WARN |
| Profile | PROFILE_DECEASED_MARKED | No | CRITICAL |
| Profile | PROFILE_FROZEN | No | WARN |
| Profile | PROFILE_MEMORIAL_MODE_ACTIVATED | No | INFO |
| Profile | PROFILE_NEXT_OF_KIN_SET | No | INFO |
| Profile | PROFILE_COGNITIVE_MODE_CHANGED | No | INFO |
| Profile | PROFILE_NO_AI_FLAG_SET | No | INFO |
| Access | MEMBER_INVITED | No | INFO |
| Access | MEMBER_ACCEPTED | No | INFO |
| Access | MEMBER_REVOKED | Yes | WARN |
| Access | ROLE_CHANGED | Yes | WARN |
| Access | CONTRIBUTOR_SCOPED_INVITE | No | INFO |
| Access | CONTRIBUTION_SUBMITTED | No | INFO |
| Access | CONTRIBUTION_APPROVED | No | INFO |
| Access | CONTRIBUTION_REJECTED | Yes | INFO |
| Access | SHARED_LINK_CREATED | No | INFO |
| Access | SHARED_LINK_ACCESSED | No | INFO |
| Access | SHARED_LINK_REVOKED | No | INFO |
| Access | INTERVIEW_MODE_SESSION_STARTED | No | INFO |
| Access | INTERVIEW_MODE_SESSION_COMPLETED | No | INFO |
| Access | B2B_PARTNER_ASSIGNED | No | INFO |
| Access | B2B_PARTNER_REMOVED | Yes | WARN |
| Access | GUEST_ACCOUNT_SUSPENDED_PENDING | No | INFO |
| Privacy | SECTION_PRIVACY_CHANGED | Yes | WARN |
| Privacy | NAMED_LIST_MEMBER_ADDED | No | INFO |
| Privacy | NAMED_LIST_MEMBER_REMOVED | Yes | INFO |
| Privacy | TIME_LOCK_SET | No | INFO |
| Privacy | TIME_LOCK_TRIGGERED | No | INFO |
| Privacy | CONDITIONAL_LOCK_RESOLVED | No | INFO |
| Privacy | SECRET_VAULT_CREATED | No | INFO |
| Privacy | SECRET_VAULT_ACCESSED | No | INFO |
| Privacy | SECRET_VAULT_SEALED | No | CRITICAL |
| Privacy | SECRET_VAULT_HEIR_DECEASED_CONFIRMED | No | WARN |

### 9.1 Additional Event Types (Complete Catalog)

The following event types extend the base catalog to provide comprehensive audit coverage:

#### Subject Rights Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Subject Rights | SAR_SUBMITTED | No | WARN |
| Subject Rights | SAR_RESPONDED | Yes | WARN |
| Subject Rights | SAR_FULFILLED | No | INFO |
| Subject Rights | SAR_DENIED | Yes | WARN |
| Subject Rights | SAR_EXTENSION_REQUESTED | Yes | INFO |
| Subject Rights | CORRECTION_REQUESTED | No | WARN |
| Subject Rights | CORRECTION_RESOLVED | Yes | WARN |
| Subject Rights | CORRECTION_APPLIED | Yes | INFO |
| Subject Rights | REMOVAL_REQUESTED | No | WARN |
| Subject Rights | REMOVAL_RESOLVED | Yes | WARN |
| Subject Rights | REMOVAL_CONTENT_PRIVATIZED | No | INFO |
| Subject Rights | PORTABILITY_REQUESTED | No | WARN |
| Subject Rights | PORTABILITY_PACKAGE_GENERATED | No | INFO |
| Subject Rights | PORTABILITY_PACKAGE_DELIVERED | No | INFO |
| Subject Rights | NO_AI_TRAINING_FLAG_SET | No | INFO |
| Subject Rights | NO_AI_TRAINING_FLAG_CLEARED | Yes | INFO |
| Subject Rights | NO_AI_TRAINING_PROPAGATED | No | INFO |
| Subject Rights | NO_AI_TRAINING_PROPAGATION_FAILED | No | CRITICAL |

#### Legacy & Succession Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Legacy | VAULT_HEIR_DESIGNATED | No | INFO |
| Legacy | VAULT_HEIR_REMOVED | Yes | WARN |
| Legacy | VAULT_HEIR_ACCEPTED | No | CRITICAL |
| Legacy | VAULT_HEIR_DECLINED | No | CRITICAL |
| Legacy | VAULT_HEIR_DEFERRED | No | INFO |
| Legacy | VAULT_HEIR_STEWARDSHIP_ACCEPTED | No | CRITICAL |
| Legacy | VAULT_HEIR_STEWARDSHIP_DEFERRED | No | INFO |
| Legacy | DEATH_WITNESS_DESIGNATED | No | INFO |
| Legacy | DEATH_WITNESS_REMOVED | Yes | INFO |
| Legacy | DEATH_PROTOCOL_TRIGGERED | Yes | CRITICAL |
| Legacy | DEATH_PROTOCOL_CONFIRMED | Yes | CRITICAL |
| Legacy | DEATH_PROTOCOL_REVERSAL_REQUESTED | Yes | WARN |
| Legacy | DEATH_PROTOCOL_REVERSAL_PROCESSED | Yes | CRITICAL |
| Legacy | DEATH_PROTOCOL_FINALIZED | Yes | CRITICAL |
| Legacy | DEATH_WITNESS_INVALID | No | WARN |
| Legacy | DEATH_PROTOCOL_MANUAL_INITIATED | No | CRITICAL |
| Legacy | VAULT_QUIET_PERIOD_STARTED | No | INFO |
| Legacy | VAULT_QUIET_PERIOD_ENDED | No | INFO |
| Legacy | VAULT_OWNERSHIP_TRANSFERRED | Yes | CRITICAL |
| Legacy | WILL_CREATED | No | INFO |
| Legacy | WILL_UPDATED | Yes | INFO |
| Legacy | WILL_UNLOCKED | No | CRITICAL |
| Legacy | FINAL_LETTER_CREATED | No | INFO |
| Legacy | FINAL_LETTER_DELIVERED | No | CRITICAL |
| Legacy | TIMELOCK_DEFERRED_DEATH_PROTOCOL | No | WARN |
| Legacy | VAULT_PROTECTED_SEALED_ENTERED | No | CRITICAL |
| Legacy | VAULT_PROTECTED_SEALED_EXITED | No | CRITICAL |
| Legacy | ACTING_FAMILY_ADMIN_DESIGNATED | Yes | WARN |
| Legacy | ACTING_FAMILY_ADMIN_DESIGNATION_REMOVED | Yes | WARN |
| Legacy | STEWARDSHIP_ESCALATION_TIMER_STARTED | No | INFO |
| Legacy | STEWARDSHIP_ESCALATION_TIMER_EXPIRED | No | WARN |
| Legacy | MEMORIAL_LIBERATION_TRIGGERED | No | INFO |
| Legacy | MEMORIAL_ADDENDUM_SUBMITTED | No | INFO |
| Legacy | MEMORIAL_ADDENDUM_APPROVED | No | INFO |
| Legacy | MEMORIAL_ADDENDUM_REJECTED | Yes | INFO |
| Legacy | DECEASED_PROFILE_RIGHTS_DELEGATED | No | INFO |

#### Vault Management Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Vault | VAULT_CREATED | No | INFO |
| Vault | VAULT_EXPORTED | No | WARN |
| Vault | VAULT_IMPORTED | No | INFO |
| Vault | VAULT_BACKUP_COMPLETED | No | INFO |
| Vault | VAULT_BACKUP_FAILED | No | CRITICAL |
| Vault | VAULT_INTEGRITY_CHECK_PASSED | No | INFO |
| Vault | VAULT_INTEGRITY_CHECK_FAILED | No | CRITICAL |
| Vault | VAULT_RENAMED | Yes | INFO |
| Vault | VAULT_IP_SALT_ROTATED | No | INFO |
| Vault | VAULT_LINK_SUSPENDED | No | WARN |
| Vault | VAULT_LINK_REVOKED | No | WARN |
| Vault | VAULT_LINK_ERASURE_PROPAGATED | No | WARN |
| Vault | VAULT_ESCROW_DEPOSITED | No | INFO |
| Vault | VAULT_ESCROW_DEPOSIT_FAILED | No | CRITICAL |
| Vault | CONSTITUTION_UPDATED | Yes | INFO |
| Vault | PROVIDER_CHANGED | Yes | WARN |
| Vault | AI_FEATURE_TOGGLED | Yes | INFO |
| Vault | FAMILY_REUNION_QR_GENERATED | No | INFO |
| Vault | FAMILY_REUNION_QR_EXPIRED | No | INFO |
| Vault | FAMILY_REUNION_QR_ACCESSED | No | INFO |

#### AI System Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| AI | AI_QUERY_MADE | No | INFO |
| AI | AI_PATTERN_DETECTED | No | INFO |
| AI | AI_HEALTH_ALERT_SENT | No | WARN |
| AI | AI_HEALTH_ALERT_DISMISSED | No | INFO |
| AI | AI_BUDGET_CAP_REACHED | No | WARN |
| AI | AI_PROVIDER_FAILOVER | No | WARN |
| AI | AI_PROVIDER_FAILOVER_RECOVERED | No | INFO |
| AI | AI_SUGGESTION_ACCEPTED | No | INFO |
| AI | AI_SUGGESTION_REJECTED | No | INFO |
| AI | AI_TRANSCRIPTION_COMPLETED | No | INFO |
| AI | AI_TRANSCRIPTION_FAILED | No | WARN |
| AI | AI_PATTERN_CARD_GENERATED | No | INFO |
| AI | AI_PATTERN_CARD_INVALIDATED | No | INFO |
| AI | AI_CONTEXT_CONSTRUCTED | No | DEBUG |
| AI | AI_CONTEXT_FILTERED | No | DEBUG |
| AI | AI_DEIDENTIFICATION_APPLIED | No | DEBUG |
| AI | AI_EXTERNAL_JOB_DISCARD_REQUESTED | No | WARN |

#### Billing & Monetization Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Billing | PLAN_UPGRADED | No | INFO |
| Billing | PLAN_DOWNGRADED | Yes | INFO |
| Billing | PAYMENT_SUCCEEDED | No | INFO |
| Billing | PAYMENT_FAILED | No | WARN |
| Billing | PAYMENT_RETRY_SCHEDULED | No | INFO |
| Billing | PAYMENT_METHOD_ADDED | No | INFO |
| Billing | PAYMENT_METHOD_REMOVED | No | INFO |
| Billing | PRINT_CREDIT_USED | No | INFO |
| Billing | PRINT_CREDIT_PURCHASED | No | INFO |
| Billing | SECTION_PAID_ACCESS_GRANTED | No | INFO |
| Billing | SECTION_PAID_ACCESS_REVOKED | No | INFO |
| Billing | REVENUE_SHARE_PAID_OUT | No | INFO |
| Billing | REVENUE_SHARE_ESCROWED | No | INFO |
| Billing | REVENUE_SHARE_CLAWBACK_INITIATED | Yes | WARN |
| Billing | TAX_DOCUMENT_ISSUED | No | INFO |
| Billing | B2B_READ_GRANT_ACCESS | No | INFO |
| Billing | B2B_READ_GRANT_EXPIRED | No | WARN |
| Billing | B2B_READ_GRANT_REVIEWED | No | INFO |
| Billing | B2B_READ_GRANT_APPROVED | No | INFO |
| Billing | B2B_READ_GRANT_REJECTED | Yes | INFO |
| Billing | REFERRAL_ACCEPTED | No | INFO |
| Billing | REFERRAL_FRAUD_DETECTED | No | WARN |
| Billing | REFERRAL_CREDIT_AWARDED | No | INFO |

#### Print & Export Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Print | PRINT_ORDER_CREATED | No | INFO |
| Print | PRINT_ORDER_SUBMITTED | No | INFO |
| Print | PRINT_ORDER_PAYMENT_CONFIRMED | No | INFO |
| Print | PRINT_ORDER_RECALL_REQUESTED | Yes | INFO |
| Print | PRINT_ORDER_RECALL_EXPIRED | No | INFO |
| Print | PRINT_ORDER_SHIPPED | No | INFO |
| Print | PRINT_ORDER_DELIVERED | No | INFO |
| Print | PRINT_ORDER_CANCELLED | Yes | WARN |
| Print | PRINT_VERSION_SNAPSHOT_CREATED | No | INFO |
| Print | PRINT_PROVIDER_FAILED_OVER | No | WARN |
| Export | EXPORT_REQUESTED | No | INFO |
| Export | EXPORT_GENERATED | No | INFO |
| Export | EXPORT_DOWNLOADED | No | INFO |
| Export | EXPORT_EXPIRED | No | INFO |
| Export | FVAF_PACKAGE_VALIDATED | No | INFO |
| Export | FVAF_PACKAGE_CORRUPTION_DETECTED | No | CRITICAL |

#### Authentication & Security Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Auth | LOGIN_SUCCESS | No | INFO |
| Auth | LOGIN_FAILURE | No | WARN |
| Auth | LOGOUT | No | INFO |
| Auth | PASSWORD_CHANGED | No | INFO |
| Auth | PASSWORD_RESET_REQUESTED | No | INFO |
| Auth | PASSWORD_RESET_COMPLETED | No | INFO |
| Auth | MFA_ENABLED | No | INFO |
| Auth | MFA_DISABLED | Yes | WARN |
| Auth | MFA_CHALLENGE_ISSUED | No | INFO |
| Auth | MFA_CHALLENGE_PASSED | No | INFO |
| Auth | MFA_CHALLENGE_FAILED | No | WARN |
| Auth | PASSKEY_CREATED | No | INFO |
| Auth | PASSKEY_USED | No | INFO |
| Auth | PASSKEY_REVOKED | Yes | INFO |
| Auth | DEVICE_TRUST_GRANTED | No | INFO |
| Auth | DEVICE_TRUST_REVOKED | No | INFO |
| Auth | SESSION_REVOKED | No | INFO |
| Auth | ACCOUNT_LOCKED | No | WARN |
| Auth | ACCOUNT_UNLOCKED | No | INFO |
| Auth | SUSPICIOUS_ACTIVITY_DETECTED | No | WARN |
| Auth | PIN_LOCKOUT | No | WARN |
| Auth | PIN_RESET_COMPLETED | No | INFO |
| Security | DEVICE_FINGERPRINT_PURGED | No | INFO |
| Security | IP_SALT_ROTATED | No | INFO |
| Security | ENCRYPTION_KEY_ROTATED | No | CRITICAL |
| Security | CSAM_SCAN_ALERT | No | CRITICAL |
| Security | LEGAL_HOLD_APPLIED | Yes | CRITICAL |
| Security | LEGAL_HOLD_REMOVED | Yes | CRITICAL |

#### Interview Mode Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| Interview | INTERVIEW_SESSION_REVIEWED | No | INFO |
| Interview | INTERVIEW_SESSION_APPROVED | No | INFO |
| Interview | INTERVIEW_SESSION_REJECTED | Yes | INFO |
| Interview | INTERVIEW_SESSION_RETURNED | Yes | INFO |
| Interview | INTERVIEW_QUESTIONS_ASSIGNED | No | INFO |
| Interview | INTERVIEW_QUESTIONS_MODIFIED | Yes | INFO |
| Interview | INTERVIEW_OFFLINE_ATTEMPT_BLOCKED | No | WARN |
| Interview | INTERVIEW_SESSION_PAUSED | No | INFO |
| Interview | INTERVIEW_SESSION_RESUMED | No | INFO |
| Interview | LOW_QUALITY_ANSWER_FLAGGED | No | INFO |

#### Trust & Safety Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| TrustSafety | CONTENT_REPORTED | No | WARN |
| TrustSafety | CONTENT_REVIEW_QUEUED | No | INFO |
| TrustSafety | CONTENT_REVIEWED | No | INFO |
| TrustSafety | CONTENT_REMOVED | Yes | CRITICAL |
| TrustSafety | CONTENT_RESTORED | Yes | WARN |
| TrustSafety | USER_SUSPENDED | Yes | CRITICAL |
| TrustSafety | USER_BANNED | Yes | CRITICAL |
| TrustSafety | USER_REINSTATED | Yes | CRITICAL |
| TrustSafety | APPEAL_SUBMITTED | No | INFO |
| TrustSafety | APPEAL_REVIEWED | Yes | WARN |
| TrustSafety | APPEAL_GRANTED | Yes | WARN |
| TrustSafety | APPEAL_DENIED | Yes | WARN |
| TrustSafety | NCMEC_REPORT_SUBMITTED | Yes | CRITICAL |
| TrustSafety | LEGAL_ORDER_RECEIVED | Yes | CRITICAL |
| TrustSafety | LEGAL_ORDER_COMPLIED | Yes | CRITICAL |
| TrustSafety | EMERGENCY_DISCLOSURE_MADE | Yes | CRITICAL |

#### System & Infrastructure Events
| Category | Event Type | Requires Reason | Severity |
|----------|-----------|-----------------|----------|
| System | MIGRATION_STARTED | No | INFO |
| System | MIGRATION_COMPLETED | No | INFO |
| System | MIGRATION_FAILED | No | CRITICAL |
| System | BACKUP_STARTED | No | INFO |
| System | BACKUP_COMPLETED | No | INFO |
| System | BACKUP_FAILED | No | CRITICAL |
| System | RESTORE_STARTED | No | WARN |
| System | RESTORE_COMPLETED | No | WARN |
| System | DEPLOYMENT_STARTED | No | INFO |
| System | DEPLOYMENT_COMPLETED | No | INFO |
| System | DEPLOYMENT_ROLLED_BACK | Yes | CRITICAL |
| System | SCALING_EVENT | No | INFO |
| System | PROVIDER_HEALTH_CHECK_PASSED | No | DEBUG |
| System | PROVIDER_HEALTH_CHECK_FAILED | No | WARN |
| System | WEBHOOK_DELIVERED | No | DEBUG |
| System | WEBHOOK_FAILED | No | WARN |
| System | CACHE_INVALIDATED | No | DEBUG |
| System | SEARCH_INDEX_UPDATED | No | DEBUG |
| System | MAINTENANCE_MODE_ENABLED | Yes | WARN |
| System | MAINTENANCE_MODE_DISABLED | Yes | INFO |
| System | DATA_INTEGRITY_WARNING | No | WARN |
| System | FEATURE_FLAG_ENABLED | No | INFO |
| System | FEATURE_FLAG_DISABLED | No | INFO |

**Total Event Types: 250+**

### 9.2 Event Log Retention Summary

Per IMM-C-002, activity log entries are immutable and permanent. However, storage tiering applies based on severity:

| Severity | Hot Storage | Warm Storage | Cold Storage (Immutable Archive) |
|----------|-------------|--------------|----------------------------------|
| CRITICAL | 7 years | Permanent | Permanent |
| WARN | 2 years | 7 years | Permanent |
| INFO | 90 days | 2 years | Permanent |
| DEBUG | 30 days | 90 days | Permanent |

**Note:** All activity log entries are retained permanently per IMM-C-002. Storage tiering optimizes cost — older logs move to cheaper storage but remain queryable and immutable. No log entry is ever deleted.

All events include: timestamp (UTC), actor, action, target entity, IP address hash, device fingerprint, session ID, and reason (when required).

---

## 10. Build Contract

### 10.0 Classification System

| Classification | Meaning | Violation Consequence |
|---------------|---------|----------------------|
| MANDATORY | Non-negotiable. Violation requires immediate rollback and post-mortem. | Deploy blocked. Post-mortem required. |
| CRITICAL | Near-mandatory. Exceptions require Product Owner approval in writing. | Product Owner sign-off required. Logged. |
| STANDARD | Required for all production code. Development and staging exceptions allowed. | Must be resolved before next release. |
| ADVISORY | Strongly recommended. Deviation must be documented. | Document reason in pull request description. |

### 10.1 Immutability Contract

**IMM-C-001 — MANDATORY:** No row in any content table (blocks, chapters, books, profiles, assets, folders, relationships) may ever be hard-deleted from the database. DELETE statements on these tables are forbidden. The only permitted action is setting a branch flag and creating a version node.

**IMM-C-002 — MANDATORY:** No row in the activity_logs table may ever be updated or deleted. The database user that the application runs as must not be granted UPDATE or DELETE permissions on activity_logs. This is enforced at the database permission level, not just application level.

**IMM-C-003 — MANDATORY:** Every write operation to any content entity (create, update, move, lock, privacy change) must create a new VersionNode before the write is committed. The write and the version node creation are a single database transaction. If the transaction fails, both are rolled back.

**IMM-C-004 — MANDATORY:** Every non-CREATE write operation must include a reason field from the actor. The API must reject any edit or delete request that does not include a non-empty reason string (minimum 5 characters). This validation is performed server-side, not client-side only.

**IMM-C-005 — MANDATORY:** Every VersionNode must store: actor_user_id, timestamp_utc to millisecond precision, device_fingerprint, session_id, action_type, and integrity_hash (SHA-256 of content_snapshot). Any version node missing any of these fields is invalid and must not be committed.

**IMM-C-006 — MANDATORY:** Inline media anchors must record exact character position (block_id, line_number, char_offset) and context anchors (50 characters before and 50 characters after the insertion point). Both are stored at anchor creation time. Context anchors must update as surrounding text is edited. If surrounding text is deleted, the asset does not delete — it moves to the Orphaned Media Archive with its original context preserved.

**IMM-C-007 — CRITICAL:** Ghost Mode (render any entity at any historical timestamp) must produce byte-for-byte identical output to what that entity contained at that timestamp, reconstructed from the version chain. Ghost Mode output must be verified against stored integrity hashes.

**IMM-C-008 — MANDATORY:** The Deleted Archive must be a first-class browsable interface, not a modal or overflow menu. It must be accessible from the main navigation. It must show: the deleted item, its original location in dot-notation, the actor who deleted it, the timestamp, the reason, and the surrounding context at the time of deletion.

**IMM-C-009 — MANDATORY:** Branch restoration must create a new VersionNode on the current head, not modify any existing version. The restoration event is logged as a RESTORE action. The previous "deleted" branch version remains in the chain — it does not disappear because it was restored.

**IMM-C-010 — CRITICAL:** The integrity_hash stored on each VersionNode must be verified on read for any Ghost Mode or version history view. If the hash does not match the content, the system must flag the record as TAMPERED and notify the Vault Keeper immediately. Tamper detection must never silently fail.

**IMM-C-011 — MANDATORY:** The immutability test suite is a mandatory, non-skippable step in the CI/CD pipeline. Any pull request that causes any immutability test to fail is automatically rejected. No human override is permitted. The test suite covers all 12 IMM-C rules with dedicated test cases for each.

**IMM-C-012 — MANDATORY:** Binary assets remain append-only and non-destructively retained except where a valid legal erasure obligation requires removal of a living subject's personal data. In those cases the original object, renditions, thumbnails, derived caches, and search and index derivatives must be deleted or cryptoshredded without undue delay, and the erasure action must be logged permanently.

### 10.2 Data Architecture Contract

**DB-001 — MANDATORY:** PostgreSQL 16 is the primary database. No other SQL database may be used for primary data storage without a formal architecture review and Product Owner sign-off.

**DB-002 — MANDATORY:** All tables must have created_at and updated_at timestamp columns with timezone. All timestamps are stored in UTC. The application layer converts to local time for display only.

**DB-003 — MANDATORY:** All primary keys are UUID version 4. No sequential integer identifiers for any entity visible to the client. Sequential identifiers may be used for internal join tables only.

**DB-004 — MANDATORY:** All foreign keys must have explicit constraints defined at the database level. Orphaned foreign keys are not permitted. Cascade rules must be defined explicitly — no implicit defaults.

**DB-005 — MANDATORY:** The activity_logs table must be on a separate tablespace from application data. The database user for the application is granted INSERT only on activity_logs. A separate read-only analytics user is granted SELECT for log queries.

**DB-006 — MANDATORY:** All database migrations must be versioned and idempotent. Migration files are never modified after being applied. Rollback scripts must accompany every migration. No migration deploys to production without successful staging execution.

**DB-007 — CRITICAL:** Database connection pooling (PgBouncer or equivalent) is required in production. Direct connections from application servers to the primary database are not permitted except for administrative purposes.

**DB-008 — MANDATORY:** All content columns that store rich text must use a defined JSON schema (not raw HTML, not raw Markdown). The schema version is stored alongside the content. Schema migrations are applied lazily on read, eagerly on write.

**DB-009 — MANDATORY:** privacy_tier columns must use a defined enum type at the database level. Free-text privacy values are not permitted. Any privacy check must use the enum, never a string comparison.

**DB-010 — MANDATORY:** The health_records table must be on a separate tablespace from general content tables. The health data encryption key must be separate from the vault encryption key. Health data access requires explicit `health_authorized` flag on the requesting user's vault membership record.

**VC-001 — MANDATORY:** The VersionChain and VersionNode tables are append-only at the application level. The application must never UPDATE any row in version_nodes. A new node is always created.

**VC-002 — CRITICAL:** Diffs stored in VersionNode must be character-level for text content. The diff format must be a standard diff algorithm (Myers diff or equivalent). The diff must be independently verifiable — applying the diff to the previous version must produce the current content exactly.

**VC-003 — MANDATORY:** Every entity has exactly one VersionChain. The chain is created when the entity is created. The chain is never deleted. Chain integrity (correct linked list of version nodes) must be verified on any full export.

**VC-004 — MANDATORY:** content_snapshot is a canonical serialised JSON representation of the complete entity state at write time. integrity_hash is SHA-256 over that canonical serialisation. Canonical serialisation must use RFC 8785 Canonical JSON (JCS): keys sorted lexicographically, Unicode strings normalised, and number representation deterministic. Non-RFC-8785 serialisation is a critical bug — two calls to hash the same entity must always produce identical bytes, or Ghost Mode hash verification will emit false TAMPERED flags.

**AS-001 — MANDATORY:** All assets must be uploaded via the Asset Service. Direct uploads to storage providers from the client are not permitted without a server-generated pre-signed URL, and the Asset record must be created before the pre-signed URL is issued.

**AS-002 — MANDATORY:** All private assets must be served via signed URLs with a maximum 15-minute expiry. Public assets may be served via CDN. An asset's public or private status is determined by its privacy_tier field, which mirrors the owning block's privacy_tier.

**AS-003 — CRITICAL:** All uploaded assets must have MD5 and SHA-256 checksums computed and stored at upload time. A GET of any asset must verify the checksum. Corruption must be logged and flagged, never silently served.

**AS-004 — MANDATORY:** No asset may be hard-deleted from object storage except under the IMM-C-012 legal-erasure exception. Deletion sets `is_deleted_branch=true` on the asset record and moves the storage key to a deleted-assets bucket. The original key is preserved with a 90-day minimum retention after branching. When legal erasure applies, deletion or cryptoshredding must cascade to renditions, CDN caches, thumbnails, search records, and export manifests.

### 10.3 API Contract

**API-001 — MANDATORY:** All API endpoints are versioned under `/api/v1/`. No unversioned endpoints are permitted in production. When version 2 is introduced, version 1 remains operational for a minimum of 24 months.

**API-002 — MANDATORY:** Every API endpoint that reads content must perform permission checks in this exact order: (1) Is the user authenticated? (2) Is the user's role permitted for this action? (3) Does the entity's privacy_tier permit access for this user? (4) Is the user in the named_viewer list if required? All four checks must pass. Failing authentication checks returns 401. Failing role or scope checks returns 403 with a specific error code. Failing privacy checks for locked or existence-sensitive content must return the same 404-style response as non-existent content.

**API-002A — MANDATORY (Public Tier Exception):** Content with `privacy_tier = 'public'` may be accessed without authentication when presented with a valid shared link token. The shared link endpoint (`/api/v1/shared/{token}`) bypasses authentication check (1) but enforces: (a) token validity and expiry, (b) entity privacy_tier is still 'public' (not changed since link creation), (c) rate limiting per IP address, (d) no access to child entities with more restrictive tiers. This exception enables the Guest access model for viral sharing per Section 6.4.1.

**API-003 — MANDATORY:** The API must never return content that the requesting user does not have permission to see. Permission filtering happens at the query level, not the serialiser level. A user must never receive an error that reveals the existence of content they cannot access — locked content returns the same 404 as non-existent content.

**API-004 — STANDARD:** All API responses follow a consistent envelope: `{ data: ..., meta: { request_id, timestamp, version }, errors: [] }`. Errors always return a machine-readable error_code alongside a human-readable message.

**API-005 — MANDATORY:** All write endpoints require a CSRF token for browser clients. API key clients are exempt but must use the Authorization header. Cookies are SameSite=Strict. CORS is restricted to permitted origins.

**API-006 — MANDATORY:** All API endpoints have rate limits defined and enforced at the API gateway level. Rate limit headers are returned on every response. Exceeding the limit returns 429 with Retry-After header.

**API-007 — MANDATORY:** Every write endpoint that modifies content must write an activity log entry atomically within the same database transaction as the content write. An activity log failure must roll back the content write. A content write that succeeds without a log entry is a critical bug.

**API-008 — MANDATORY:** The Public Developer API must enforce OAuth 2.0 scope checks as a separate permission layer, evaluated after all standard role and privacy checks. A valid OAuth token does not bypass role-based permission checks.

**API-009 — CRITICAL:** Webhook delivery must use HMAC-SHA256 signing. Webhook consumers must verify signatures. Failed webhook deliveries must be retried with exponential backoff up to 72 hours. After 72 hours of failures, the Vault Keeper is notified and the webhook endpoint is automatically disabled. Webhook subscriptions are granted per scope, not globally. Sensitive events such as `death.protocol.triggered`, stewardship acceptance, or legal-document release require explicit per-event consent by the active steward.

**API-010 — CRITICAL:** `death.protocol.triggered` must not be emitted to third parties until the death protocol trigger is fully confirmed and the reversal window has closed.

**API-011 — MANDATORY:** Every API and background workflow must use a documented error taxonomy with stable machine-readable error codes, retry classification, user-safe messaging rules, and alert-routing severity. No production-critical workflow may rely on ad-hoc or undocumented error semantics.

**API Performance SLAs:**

| Endpoint Category | P50 Target | P99 Target | Timeout |
|------------------|-----------|-----------|---------|
| Authentication (login, register) | Less than 200ms | Less than 500ms | 5s |
| Content read (blocks, chapters) | Less than 150ms | Less than 400ms | 5s |
| Content write (create, edit) | Less than 200ms | Less than 500ms | 5s |
| Family tree and graph queries | Less than 300ms | Less than 800ms | 10s |
| Search (Typesense) | Less than 100ms | Less than 300ms | 3s |
| Media upload (pre-signed URL) | Less than 200ms | Less than 500ms | 5s |
| AI-assisted features | Less than 2,000ms | Less than 8,000ms | 30s |
| Print order creation | Less than 500ms | Less than 2,000ms | 30s |
| Export and full vault download | Less than 5,000ms | Less than 30,000ms | 120s |
| Health endpoints (/health, /ready) | Less than 50ms | Less than 100ms | 1s |
| Developer API (OAuth) | Less than 200ms | Less than 600ms | 10s |

### 10.4 Security Contract

**SEC-001 — MANDATORY:** All session tokens are stateless JWTs with 15-minute access token expiry. Refresh tokens are stateful (stored in database), 90-day expiry, single-use with rotation. Refresh tokens are stored as bcrypt hashes, never in plain text. Rotation must tolerate benign concurrent use from the same token family to prevent false compromise lockouts.

**SEC-002 — MANDATORY:** Two-factor authentication must be enforced for all Vault Keeper and Vault Heir accounts. Family Admin accounts are strongly encouraged (CRITICAL). Other roles may opt in (STANDARD).

**SEC-003 — MANDATORY:** All authentication events are logged: login success, login failure, two-factor authentication success, two-factor authentication failure, passkey creation, passkey use, device trust granted, session revoked, password changed, password reset, account locked, account unlocked.

**SEC-004 — MANDATORY:** Interview Mode PIN authentication: PINs are stored as bcrypt hashes. Minimum PIN length is 6 digits. PIN is set by the Vault Keeper, not the interview subject. Failed PIN attempts lock the interview session after 5 attempts — Vault Keeper must reset.

**SEC-004A — MANDATORY:** PIN lockout recovery must support remote reset by Vault Keeper or authorised Family Admin, log the reset event, and present a user-visible recovery path. A locked-out interview subject must not be left without any documented recovery mechanism.

**SEC-005 — MANDATORY:** All Secret Vault content must be encrypted at rest using AES-256-GCM with a per-vault encryption key. The encryption key is stored in the secrets manager (not in the database). The application retrieves the key at runtime for each request.

**SEC-006 — MANDATORY:** All data in transit uses TLS 1.3 minimum. TLS 1.0 and 1.1 are disabled globally. TLS 1.2 is permitted only for legacy print provider integrations with documented justification and Product Owner sign-off.

**SEC-007 — MANDATORY:** Health chapter data is encrypted with a separate per-profile health key. This key is only decrypted when an explicitly `health_authorized` user requests it. Health data must never appear in general content queries.

**SEC-008 — MANDATORY:** IP addresses must never be stored in plain text in activity logs. All IPs are stored as SHA-256 hashes with a per-vault salt. This prevents IP reconstruction while allowing anomaly detection. The per-vault IP salt must rotate on any effective control transfer, including stewardship acceptance after death protocol. Rotation is logged as VAULT_IP_SALT_ROTATED.

**SEC-009 — MANDATORY:** API keys for MCP providers must never be stored in the application database. They are stored in the secrets manager with a reference identifier stored in the mcp_providers table.

**SEC-010 — MANDATORY:** Offline data stored on mobile devices must be encrypted using AES-256 with a key derived from the vault master key. The key must not be stored in device keychain without biometric protection. Device wipe results in permanent offline data loss — this is intentional and must be documented to users.

**SEC-011 — MANDATORY:** All user-supplied content is validated and sanitised server-side before any processing. Client-side validation is for user experience only.

**SEC-012 — MANDATORY:** All database queries use parameterised queries or prepared statements. String concatenation into SQL is forbidden.

**SEC-013 — MANDATORY:** All rich text content uses a strict allowlist of permitted HTML elements and attributes. Sanitisation happens server-side. SVG uploads are forbidden (cross-site scripting vector). File uploads are validated by magic bytes, not file extension.

**SEC-014 — MANDATORY:** The privacy enforcement stack has four mandatory layers: (1) Database query filter, (2) Service layer check, (3) Serialiser layer, (4) Cache key namespacing — cache entries are per-user, never shared across users with different permission levels.

**SEC-015 — MANDATORY:** The Locked privacy tier must produce identical API responses to the non-existent content case. A user must not be able to distinguish between "this content is locked" and "this content does not exist" through any API response, error code, response time, or response size.

**SEC-016 — MANDATORY:** Batch endpoints must apply per-entity permission checks. A user requesting 100 blocks must receive only the blocks they are permitted to see. The response count may differ from the requested count — this is correct behaviour.

**SEC-017 — MANDATORY:** AI context construction must enforce the same permission stack as direct API queries. The AI orchestration service must never pass content to an LLM provider that the requesting user does not have permission to see.

**SEC-018 — MANDATORY:** The Public Developer API must enforce a separate OAuth scope check layer after all standard permission checks. A Vault Keeper may revoke API access at any time with immediate effect. Revoked tokens must be invalidated in the token cache within 60 seconds.

**SAR-001 — MANDATORY:** The platform must respond to Subject Access Requests without undue delay and no later than one calendar month. The Subject Access Request queue must track: submission date, respondent, response date, response type (fulfilled, denied, or partial), and reason. This data is retained permanently. Queue enforcement requires both scheduled timers and a daily reconciliation job.

**SAR-002 — MANDATORY:** Content removed from shared tiers following a removal request must not be hard-deleted — it must be moved to Private tier. The removal event is permanently logged.

**SAR-003 — CRITICAL:** Under-13 profile data may not be stored in shared or public privacy tiers and may not be processed by AI without verifiable guardian consent. Ages 13 through 17 remain subject to enhanced protections for imagery, health data, and public and shared publication. These checks are enforced at privacy-tier and processing-decision time, not just at creation time.

**SAR-004 — MANDATORY:** The No-AI-Training flag, once set on a profile, must propagate to all AI services within 5 minutes. A batch job must verify flag propagation and log the result. Any AI service that processes flagged content after the 5-minute propagation window is in violation.

**SAR-005 — MANDATORY:** GDPR erasure and pseudonymisation workflows must cover current records, historical version nodes, search indexes, caches, thumbnails, analytics derivatives, AI embeddings, and export manifests where they contain personal data. A request is not complete until downstream stores are reconciled.

**SAR-006 — MANDATORY:** A publicly accessible Privacy Notice, Subject Rights Notice, and Stewardship Commitment must be live before regulated-region data collection begins. Annual legal review is required. The Privacy Notice must specify the lawful basis for each category of personal data processing, as required by GDPR Article 13/14. The following lawful bases apply per processing activity:

| Processing Activity | Lawful Basis | Notes |
|---------------------|-------------|-------|
| Account registration and authentication | Contract performance (Art. 6(1)(b)) | Necessary to provide the service |
| Vault content storage and display | Contract performance (Art. 6(1)(b)) | Core service delivery |
| Subject Access Request processing | Legal obligation (Art. 6(1)(c)) | GDPR Chapter III obligations |
| GDPR erasure and portability | Legal obligation (Art. 6(1)(c)) | GDPR Chapter III obligations |
| Device fingerprint and session logging | Legitimate interest (Art. 6(1)(f)) | Security and fraud prevention; LIA documented and renewed annually |
| Marketing and re-engagement emails | Consent (Art. 6(1)(a)) | Opt-in at registration; withdrawable at any time |
| Health data processing (special category) | Explicit consent (Art. 9(2)(a)) | Separate explicit consent required before Health Archive activation |
| AI pattern analysis on vault content | Legitimate interest (Art. 6(1)(f)) | Subject to No-AI-Training opt-out; LIA documented |
| Minor profile data (under-13) | Consent of parent or guardian (Art. 8) | Verifiable guardian consent required |
| Deceased profile data | Legitimate interest (Art. 6(1)(f)) | Family preservation purpose; subject to next-of-kin rights |

The Privacy Notice must be reviewed by qualified privacy counsel annually and updated whenever a new processing activity is introduced. The lawful basis for each activity must be traceable from the Privacy Notice to the technical implementation.

**SAR-007 — MANDATORY:** Claimed profile subjects must be able to obtain a structured, commonly used, machine-readable export of personal data concerning that claimed profile. This portability package is separate from the full-vault FVAF export but must remain interoperable with FVAF.

**SAR-008 — DEPRECATED:** This rule number is formally retired. The content originally covered by SAR-008 is fully addressed by SAR-003, which is the canonical rule. SAR-008 must not be referenced in any new code, test, or documentation. Any existing test or code referencing SAR-008 must be updated to reference SAR-003. In a compliance audit, SAR-008 does not exist as an active rule — only SAR-003 applies.

### 10.5 AI System Contract

**AI-001 — MANDATORY:** No AI-generated content may be silently published to any profile or block. All AI suggestions require explicit human acceptance before being written to the content store. The accept action is logged as AI_SUGGESTION_ACCEPTED in the activity log.

**AI-002 — MANDATORY:** Every AI-generated text block must carry a permanent AI_GENERATED flag in its block record. This flag cannot be removed, even if the content is substantially edited. The edit creates a new version, but the AI_GENERATED origin is preserved in the version chain.

**AI-003 — MANDATORY:** AI-generated health alerts must include: the pattern description, the specific ancestor profiles that informed it, a confidence score (0.0 to 1.0), and a mandatory medical disclaimer. The alert must not be presented as a diagnosis and must not use diagnostic language.

**AI-004 — CRITICAL:** AI-generated family pattern cards must include a confidence score and a list of evidence (block identifiers and profile identifiers) that informed the pattern. Low-confidence patterns (below 0.4) must be clearly marked as speculative and visually distinguished from high-confidence patterns.

**AI-005 — MANDATORY:** AI writing assistance must present the generated text in a clearly distinct suggestion state before acceptance. The user interface must make it visually impossible to accidentally accept AI content. Suggestions must have a different background colour, a border, and an explicit Accept button that requires deliberate click.

**AI-006 — MANDATORY:** The AI orchestration service must never pass content to an LLM provider that the requesting user does not have permission to see. Context construction is permission-scoped.

**AI-007 — MANDATORY:** Health chapter data passed to any external AI system must be pseudonymized (names replaced with profile identifiers) before being sent. Name rehydration happens after AI response, in the internal AI service. This pseudonymization must be verified by an automated test on every AI health call.

**AI-008 — MANDATORY:** AI queries that reference the story graph or family tree must only return results that the requesting user has permission to see. AI results must be post-filtered through the same permission stack as direct API queries.

**AI-009 — MANDATORY:** Profiles with the No-AI-Training flag set must be excluded from all AI pattern analysis, question generation, and family graph AI features. Exclusion is enforced at the AI orchestration layer, not just the UI layer.

**AI-010 — MANDATORY:** All AI capability calls must go through the MCP Orchestration Service. Direct calls from application code to external AI provider APIs are forbidden. This is enforced at code review.

**AI-011 — CRITICAL:** Provider failover must be automatic. If the primary LLM provider returns a 5xx error or times out after 10 seconds, the request must retry once on the primary, then automatically route to the fallback provider. The failover event must be logged as AI_PROVIDER_FAILOVER. Partial results from the failed provider are discarded; the job restarts on the fallback provider from the beginning of the job context.

**AI-012 — MANDATORY:** AI call costs must be tracked per vault. Monthly AI cost per vault is logged. Vaults on the Free plan have an AI compute budget cap. Exceeding the cap gracefully degrades AI features and notifies the Vault Keeper. It must never hard-fail or cause data loss. AI budget cap and graceful degradation are launch-blocking behaviour, not deferrable.

**AI-013 — MANDATORY:** Cross-profile health analysis may only use profiles that the requester is authorised to access. If an aggregate output would reveal inaccessible health information by inference, the result must be suppressed.

**AI-014 — MANDATORY:** When a No-AI-Training flag is enabled during an in-flight external job, output storage is forbidden and the platform must issue a best-effort discard and deletion request to the external provider for transient artefacts.

**AI-015 — MANDATORY:** A profile with the No-AI-Training flag set is excluded from all AI analysis including aggregate and cross-profile pattern cards, family health pattern engine, and story graph AI inference. The exclusion is enforced at the AI orchestration layer before any data is sent to any provider. Pattern cards computed over a cohort that includes No-AI-Training profiles must suppress any output that would inferentially reveal the excluded profile's data by omission. Pattern cards must be invalidated and regenerated within 5 minutes of a No-AI-Training flag being activated (same window as SAR-004). AI-009 governs individual-profile exclusion; AI-015 governs aggregate and cross-profile exclusion.

### 10.6 MCP Provider Architecture Contract

**MCP-001 — MANDATORY:** No application code outside the MCP layer may import or reference a specific provider's SDK or API directly. All provider access is through the capability interface. This is enforced at code review — any pull request violating this is automatically rejected.

**MCP-001-A — MANDATORY (Two-Factor Authentication SMS Exception):** Two-factor authentication SMS delivery is explicitly excluded from the `sms.send` MCP capability routing. 2FA SMS is a security-critical, latency-sensitive function that must not be subject to MCP provider health checks, capability conflict resolution, or Vault Keeper-configurable provider switching. 2FA SMS routes directly through the designated authentication SMS provider (Twilio, as specified in the technology stack) via the authentication service — bypassing the MCP layer entirely. This is the only permitted exception to MCP-001. The MCP `sms.send` capability is used exclusively for non-security transactional SMS (e.g., interview session reminders, re-engagement nudges). Any future addition of a non-2FA security-critical SMS use case must be reviewed against this exception before routing through MCP. The two routing paths must be clearly separated in code with comments identifying the exception.

**MCP-002 — CRITICAL:** Adding a new provider must require only: registering the provider in the MCP registry, implementing the capability interface, adding credentials to the secrets manager, and adding a health check. No application code changes may be required.

**MCP-003 — CRITICAL:** Switching the primary provider for any capability must require only a configuration change in the MCP registry. No code deployment required for provider switches.

**MCP-004 — MANDATORY:** Every capability code (for example, `llm.generate`, `email.send`, `storage.file`) must have a defined interface specification: input schema, output schema, error types, and timeout values. Provider implementations must conform to this specification.

**MCP-005 — MANDATORY:** When two registered providers share a capability, the system must detect the conflict immediately on provider registration. The Vault Keeper is notified and must resolve the conflict before either provider is used for that capability.

**MCP-006 — MANDATORY:** Conflict resolution must be explicit: the Vault Keeper selects PRIMARY and SECONDARY for each conflicted capability. The selection is logged. Unresolved conflicts block the conflicted capability until resolved.

### 10.7 Logging and Retention Contract

**LOG-001 — MANDATORY:** Every user-initiated and system-initiated action must produce an activity log entry. No action may silently succeed. The log entry must be written in the same transaction as the action.

**LOG-002 — MANDATORY:** Every log entry must include at minimum: event_id (UUID), event_type (enum), event_category (enum), timestamp_utc (millisecond precision), severity, actor_user_id (or SYSTEM for automated actions), and the applicable vault, entity, session, and device fields for that event class. Where a field is not applicable, the logging policy must use a documented null or sentinel value.

**LOG-003 — MANDATORY:** The audit log must be queryable by: vault_id, actor_user_id, event_type, event_category, entity_id, timestamp range, and severity. Query performance must meet the API SLA for log read endpoints.

**LOG-004 — MANDATORY:** device_fingerprint is a privacy-reviewed, hashed and salted identifier derived only from an approved set of device and session signals documented by security and privacy counsel. It must never store raw browser fingerprint material in clear text.

**LOG-005 — MANDATORY:** device_fingerprint collection requires explicit user consent documented in the Privacy Notice acceptance at account creation. Users who withdraw consent must have their stored device_fingerprint hashes pseudonymised within 72 hours. Retention limit: device_fingerprint hashes are retained for the duration of the active session plus 90 days, then purged. The 90-day retention must be disclosed in the Privacy Notice. device_fingerprint data must never be shared with AI providers, B2B partners, or third-party analytics platforms. A DEVICE_FINGERPRINT_PURGED event is logged when hashes are purged on consent withdrawal or retention expiry.

### 10.8 Testing Contract

**Test Coverage Requirements:**

| Test Category | Coverage Requirement |
|--------------|---------------------|
| Unit tests — domain logic | Minimum 90% line coverage on all domain services |
| Unit tests — version chain | 100% coverage on all version chain creation paths |
| Unit tests — permission checks | 100% coverage on all permission check functions |
| Unit tests — Subject Access Request workflows | 100% coverage on all subject rights flows |
| Integration tests — API endpoints | 100% of endpoints have at least one integration test |
| Integration tests — happy path | All primary user flows covered end-to-end |
| Integration tests — permission boundaries | All 13 roles tested against all entity types |
| Immutability tests | Dedicated non-skippable suite: delete creates branch, version node on every write, reason enforced, log written atomically |
| Offline sync tests | Conflict detection and merge flow tested for all block types |
| AI contract tests | All AI capabilities have mock provider tests. Failover path tested. No-AI-Training propagation verified. |
| Security tests (automated) | Run on every pull request. No critical issues may be merged. |
| Performance tests | Run on every staging deploy. All response time SLAs verified. |
| Load tests | Run before every major release. 10,000 concurrent users simulated. |
| Accessibility tests | Run on every user interface change. WCAG 2.1 AA compliance verified. |
| Interview Mode tests | End-to-end: Vault Keeper assigns session → Interviewer completes → Keeper reviews → content enters vault. |

**Definition of Done (Universal):** No feature, sprint, or story may be marked Done unless all of the following are true:

1. Code is merged to main branch. No open pull request comments.
2. All tests pass (unit, integration, security). Coverage thresholds met.
3. All activity log entries for the feature are verified to be written correctly in tests.
4. All version chain operations are verified: every write creates a version node.
5. Permission boundaries tested: correct access for permitted roles, correct 401, 403, and 404-style outcomes for unauthenticated, forbidden, and non-existent or locked cases.
6. The reason field is enforced on all applicable actions.
7. Performance targets met in staging environment.
8. No new critical or high vulnerability findings from automated security scan.
9. Documentation updated: API docs, data model if changed.
10. Product Owner has reviewed and approved the feature in staging.
11. If the feature touches Subject Access Requests or subject rights: Subject Access Request workflow tested end-to-end.
12. If the feature touches AI: AI transparency rules verified. No-AI-Training propagation tested.
13. If the feature is accessible: WCAG 2.1 AA compliance verified.
14. If the feature touches offline: offline sync and conflict resolution tested.
15. If the feature touches Secret Vault: Secret Vault heir lifecycle, Cognitive Mode exclusion, and time-lock interaction tested.
16. If the feature touches death protocol: reunion code expiry, time-lock deferral, revenue escrow chain, and B2B grant review tested.
17. If the feature touches AI Pattern Cards: No-AI-Training aggregate exclusion and flag invalidation-propagation tested.
18. If the feature touches notifications: non-suppressible security notification delivery verified under all suppression settings.

**Regression Criteria — What Must Never Break:**

**TEST-REG-001 — MANDATORY:** Any code change that causes a hard-delete of any content record to succeed is a critical regression. The immutability test suite must run on every pull request and any failure blocks merge.

**TEST-REG-002 — MANDATORY:** Any code change that allows content to be served to a user who does not have permission must be treated as a critical security regression. The permission boundary test suite is mandatory on every pull request.

**TEST-REG-003 — MANDATORY:** Any code change that causes an action to succeed without producing an activity log entry is a critical regression.

**TEST-REG-004 — MANDATORY:** Any code change that allows AI content to be published without human acceptance is a critical regression.

**TEST-REG-005 — MANDATORY:** Any code change that allows a No-AI-Training profile's content to be processed by an AI service is a critical regression.

### 10.9 Deployment and Operations Contract

**Environment Standards:**

| Environment | Standard |
|-------------|---------|
| Development | Local only. No access to real user data. Seeded with synthetic data only. No real provider API keys. |
| Staging | Production-identical infrastructure at 20% scale. Real feature testing. No real user data. Full security controls active. |
| Production | Full scale. Real user data. All security controls active. Change window required for schema changes. |
| Disaster Recovery | Production replica in separate geographic region. Automated failover tested quarterly. RTO less than 4 hours. RPO less than 1 hour. Disaster recovery regions must respect data residency obligations. Cross-border failover for regulated data requires an approved transfer mechanism. |

**DEP-001 — MANDATORY:** Zero-downtime deployments are required for all production deploys. Blue-green or canary deployment strategy. No maintenance windows for application code deploys.

**DEP-002 — MANDATORY:** Database schema changes (migrations) require a change window. Migrations must be backward-compatible with the current production code for a minimum of one full deploy cycle. No breaking schema changes without a multi-phase migration plan.

**DEP-003 — CRITICAL:** Production deployments require at least two engineers present: a deployer and a monitor. If P99 latency exceeds 2 times baseline during rollout, the deploy is automatically rolled back.

**DEP-004 — MANDATORY:** All production infrastructure is defined as code (Terraform or equivalent). No manual infrastructure changes in production.

**DEP-005 — MANDATORY:** Every production deployment must have a documented rollback path for code and configuration, with compatibility rules for in-flight schema changes and queued jobs. A deploy is incomplete if rollback viability has not been verified in staging or canary conditions.

**Incident Response SLAs:**

| Severity | Definition | Response Time | Resolution Target |
|----------|-----------|--------------|------------------|
| P0 — Critical | Data loss, security breach, total outage, death protocol failure | 15 minutes | 4 hours |
| P1 — High | Major feature unavailable, authentication failure affecting multiple users, AI serving incorrect private content | 30 minutes | 8 hours |
| P2 — Medium | Single feature degraded, performance degradation, non-critical integration failure | 2 hours | 24 hours |
| P3 — Low | Minor user interface issue, non-critical API error, cosmetic bug | 24 hours | 72 hours |

**OPS-001 — MANDATORY:** Authentication, vault reads and writes, exports, death protocol, subject-rights queues, AI orchestration, webhook delivery, and backup and escrow jobs must emit metrics, logs, traces, and alert thresholds sufficient to prove service health and detect degradation before user harm occurs.

### 10.10 Code Standards Contract

**CODE-001 — MANDATORY:** TypeScript strict mode is required across all packages. `tsconfig.json` must include: `strict: true`, `noImplicitAny: true`, `strictNullChecks: true`, `noUncheckedIndexedAccess: true`. Any pull request that introduces TypeScript errors in strict mode is automatically rejected.

**CODE-002 — MANDATORY:** No `any` type is permitted in production code. Type assertions require a comment explaining why the assertion is safe. Usage of `unknown` is preferred over `any` when type narrowing is required.

**CODE-003 — STANDARD:** All domain logic is in service functions, not in route handlers or React components. Route handlers validate input with Zod and call service functions. React components call API hooks, not services directly.

**CODE-004 — STANDARD:** Shared types (entities, enums, error codes, event types) live exclusively in `packages/shared/types`. No type is defined in more than one place.

**CODE-005 — STANDARD:** All data fetching uses React Query (TanStack Query). No direct fetch calls in components. Queries are typed end-to-end from the API response type.

**CODE-006 — STANDARD:** All forms use React Hook Form with Zod resolvers. No uncontrolled inputs in forms that touch the API. Form submission is disabled while a mutation is in-flight.

**CODE-007 — MANDATORY:** Accessibility: all interactive elements have aria-label or aria-labelledby. All images have alt text. All form inputs have associated labels. Keyboard navigation must work for all primary flows. WCAG 2.1 AA is the minimum standard.

**CODE-008 — MANDATORY:** Interview Mode and Cognitive Accessibility Mode user interfaces must be built as separate route groups from the standard user interface. No standard navigation chrome leaks into these modes. Tested with real elderly users before each release.

### 10.11 Long-Term Stewardship Contract

**STW-001 — MANDATORY:** A complete, encrypted, self-contained export of every vault must be deposited with a designated escrow agent annually. The escrow deposit must be logged as VAULT_ESCROW_DEPOSITED in the vault's audit log. If the escrow deposit fails, the Vault Keeper is notified within 24 hours.

**STW-002 — MANDATORY:** The FamilyVault Archive Format (FVAF) specification must be maintained as a public, versioned document. The specification must be sufficient for a third party to build a compliant reader without any assistance from FamilyVault. A reference reader implementation must be open-sourced.

**STW-003 — MANDATORY:** Each vault export must be self-contained: all content as JSON, all media as original files, a locally unlockable Health Bundle where health data is included, and the FVAF reference reader HTML file. The export must be browsable without an internet connection. FVAF exports must include a local-unlock Health Bundle using export-time wrapped keys so health content remains offline-accessible without FamilyVault infrastructure.

**STW-004 — CRITICAL:** If FamilyVault decides to shut down, all active vault owners must receive a minimum of 90 days advance notice via email and in-app notification. A full FVAF export must be made available for download to all vault owners before shutdown. This is a binding commitment.

**STW-005 — CRITICAL:** In the event of acquisition, the acquirer must sign a Vault Data Commitment Agreement as a condition of the acquisition. This agreement must commit to: honouring all existing privacy tiers, maintaining vault ownership rights, supporting FVAF exports, and providing 90-day notice before any service discontinuation.

**STW-006 — MANDATORY:** The Backup Transparency Dashboard must be updated with real data daily. If the dashboard shows data that is more than 48 hours stale, the on-call engineer is automatically paged. The dashboard must be accessible to the Vault Keeper at all times, even during partial outages.

**STW-007 — STANDARD:** FamilyVault should maintain a trust page at `/trust` that discloses: backup frequency, escrow agent identity, geographic backup locations (region-level), last restore test date, FVAF version, and a link to the Vault Data Commitment Agreement template.

### 10.12 B2B and Institutional Contract

**B2B-001 — MANDATORY:** B2B partner institutions must have their vault portfolio completely isolated at the database level. A row-level security policy must prevent B2B Partner Admins from accessing any vault outside their assigned portfolio. Cross-portfolio data leakage is a critical security violation.

**B2B-002 — MANDATORY:** B2B white-label branding must never remove or obscure the "Powered by FamilyVault" attribution. Brand customisation is permitted for colours and logo. The FamilyVault name must remain visible to end users in the footer and in any data export.

**B2B-003 — CRITICAL:** B2B clients (the families or individuals whose vaults are managed by an institution) must be informed that their vault is hosted on FamilyVault and that they may export their data at any time. Institutions may not hide this information from their clients.

**B2B-004 — MANDATORY:** B2B SAML 2.0 / OIDC authentication must fall back to FamilyVault native authentication if the institution's identity provider becomes unavailable. B2B users must never be permanently locked out due to their institution's identity provider failure. Phase 1 owns the unit test of the fallback mechanism. Phase 16 owns the full integration test in the B2B context. Both phases are required; neither substitutes for the other.

**B2B-005 — STANDARD:** B2B custom SLA terms (for 500 or more vault enterprise clients) must be documented in a separate SLA addendum signed by both parties. The technical obligations in this contract remain binding regardless of the custom SLA terms.

**B2B-006 — MANDATORY:** B2B Read Grants are scope-limited and time-limited. Default duration is 30 days, maximum duration is 365 days, expiry is logged, and any stewardship transfer moves outstanding grants into pending-review status until re-approved or lapsed.

### 10.13 Death Protocol Contract

**DEATH-001 — MANDATORY:** If two witnesses are configured, both must confirm within 4 hours. The protocol becomes triggered only after the required confirmation set is complete. The 24-hour reversal window begins only at trigger finalisation.

**DEATH-002 — MANDATORY:** Stewardship fallback order is deterministic. The canonical definition is Section 6.5.2 — this contract rule references it as the single source of truth. Tiebreakers for step 5 (multiple Family Admins with identical role-grant timestamps): first, explicitly pre-designated acting admin wins; second, earliest role-grant event UUID (lexical ascending) wins; third, lowest lexical user UUID as the final deterministic tiebreaker. These tiebreakers must be implemented, tested, and logged — the system must never enter an undefined state during stewardship transition. Any future change to the fallback order must be made in Section 6.5.2 first; DEATH-002 will always defer to that section.

**DEATH-003 — MANDATORY:** If no Vault Heir is designated, all Interim Authority actions must be copied to all Family Admins and any Verified Members who formally applied for stewardship.

**DEATH-004 — MANDATORY:** Memorial Mode freezes the canonical biography while allowing collaborative memorial addendum submissions under moderated workflow. The system must distinguish frozen biography content from post-death memorial contributions.

**DEATH-005 — MANDATORY:** Stewardship acceptance rotates the IP salt epoch, prompts review of active B2B read grants, and records VAULT_HEIR_STEWARDSHIP_ACCEPTED.

### 10.14 Federation and Erasure Contract

**FED-001 — MANDATORY:** A federated or shared profile has exactly one owner_vault_id. Only the owner vault may write canonical identity fields. Linked vaults may store local overlays but not fork identity truth.

**FED-002 — MANDATORY:** If a source vault becomes dormant, suspended, or loses sharing authority, linked vaults must receive a deterministic VAULT_LINK_SUSPENDED or VAULT_LINK_REVOKED outcome. Silent broken references are forbidden.

**FED-003 — MANDATORY:** When a valid GDPR erasure order is processed on a profile that is shared via VaultLink to other vaults, the erasure system must emit VAULT_LINK_ERASURE_PROPAGATED to all linked vaults within 24 hours. Linked vaults must purge their local overlay data for the erased subject within 72 hours of receiving the event. A daily reconciliation job must check for unconfirmed propagation events older than 72 hours and escalate to Trust and Safety.

### 10.15 Additional Rules

**SV-001 — MANDATORY:** The Secret Vault heir (`secret_vault_heir_profile_id`) retains their access key independently of vault stewardship transfer. The new Vault Keeper cannot revoke Secret Vault heir access without possessing the Secret Vault key. If the Secret Vault heir is deceased at the time of Vault Keeper death, the Secret Vault enters Protected Sealed State. The system must emit SECRET_VAULT_SEALED event when this condition is detected.

**SV-002 — MANDATORY:** Time-locked content must not unlock during the 24-hour reversal window. All time-lock interactions with death protocol states are logged as TIMELOCK_DEFERRED_DEATH_PROTOCOL.

**SV-003 — MANDATORY:** The Secret Vault must not be accessible from Cognitive Accessibility Mode or Interview Only Mode. The navigation, routing, and API must reject Secret Vault requests from sessions in these modes.

**REV-001 — MANDATORY:** Outstanding Vault Keeper revenue is held in escrow during the entire stewardship transition. Revenue follows the stewardship fallback chain. If all heirs decline and the vault enters protected custodial suspension, revenue is held for a maximum of 12 months, after which the 70% share is treated as unclaimed property and the 30% platform share is retained by FamilyVault. This timeline must be disclosed in the Terms of Service.

**INT-001 — MANDATORY:** Interview Mode sessions must verify active internet connectivity before starting. If the device is offline, the session must not start. If connectivity is lost during a session, the session is paused immediately. The offline queue system must explicitly exclude Interview Mode session content from its queue mechanism.

**QR-001 — MANDATORY:** When DEATH_PROTOCOL_TRIGGERED event fires, all active Family Reunion QR codes for that vault must be expired immediately (synchronously, in the same transaction). The system must not send any vault communications from a deceased Vault Keeper's account after trigger finalisation.

**HLTH-001 — MANDATORY:** Health Archive data (health_records table and all derived AI outputs from health data) is explicitly excluded from Living Memory Liberation regardless of the profile's liberation_policy setting. Any code path that changes health data privacy tiers as part of liberation processing is a critical bug.

**M7-SEC — MANDATORY:** The following notification types are non-suppressible by any user toggle, Quiet Hours setting, or digest preference: new device login alert, password reset confirmation, two-factor authentication delivery, Subject Access Request submission confirmation, death protocol trigger confirmation, vault stewardship change, account locked or unlocked, and suspicious session activity. These are delivered immediately, bypassing all notification preference filters.

**PRINT-001 — MANDATORY:** A print order must capture a complete version snapshot of all included content at the time of payment confirmation. A 30-minute recall window is available after confirmation and before transmission to the print provider. After 30 minutes, the order is irreversible. Multi-book orders snapshot all books at the same timestamp.

**SEAL-001 — MANDATORY:** Protected Sealed State is a formal system state applicable to Secret Vaults, time-locked content, or entire vaults. Entry conditions and exit requirements are per Section 6.5.5. Entry is logged as VAULT_PROTECTED_SEALED_ENTERED (CRITICAL). Exit is logged as VAULT_PROTECTED_SEALED_EXITED (CRITICAL) with court order reference.

**CFG-001 — MANDATORY:** Product thresholds, free-tier limits, review windows, quiet periods, rate limits, and regional and legal defaults must be sourced from a configuration registry with environment, plan, jurisdiction, and effective-date support. Specification values are launch defaults unless this contract explicitly marks them fixed.

**COM-001 — MANDATORY:** Automated and marketing email flows must honour applicable unsubscribe, consent, sender-identification, and lawful-basis requirements. Security and legal notices may remain non-suppressible where law or security practice requires delivery.

**TS-001 — MANDATORY:** The Trust and Safety tool must support report intake, CSAM and illegal-content escalation, CyberTipline and NCMEC reporting where applicable, legal hold, removal and suspension, appeal handling, immutable operator logs, and role-based internal access control.

**BIZ-001 — MANDATORY:** Revenue-share defaults, section-pricing bounds, and developer API rate-limit defaults must be documented in governed commercial policy and referenced by configuration, not hidden as hardcoded application constants. Where commercial terms become legally binding, the governing policy version must be traceable from the contract or customer agreement.

**EDGE-001 — STANDARD:** A content block being actively edited must display a soft-lock indicator to other users showing who is editing it. The soft-lock expires after 15 minutes of inactivity. A second user may read but not edit a soft-locked block. Soft-locks are not recorded in the version chain.

**EDGE-002 — MANDATORY:** Self-referral detection compares referrer and referred-user on: email domain match, hashed IP subnet match, and device_fingerprint match. Any two-of-three match triggers human review before referral credit is awarded. Device fingerprint match alone is not sufficient to trigger review — it must be combined with email domain or IP subnet match to avoid false positives from legitimate referrals between household members on different devices. Maximum 50 referral credits per account per calendar year.

**EDGE-003 — STANDARD:** A Vault Keeper who has been inactive (no login, no API activity) for 180 days receives an account health warning email. After 365 days of inactivity, the vault enters a low-activity flag state visible only to Trust and Safety. No content is modified or access restricted by the inactivity flag alone.

**EDGE-004 — STANDARD:** If a profile is tagged in a story but that profile is later merged into another profile, all Story Graph edges referencing the original profile are re-attributed to the surviving profile. The original mention text in the story is not altered — only the graph edge target changes.

**EDGE-005 — STANDARD:** If a Vault Keeper downgrades from a paid plan to the Free tier, content that exceeds free-tier limits (profiles 6+, books 4+, video blocks) is not deleted. It is placed in a read-only locked state. The Vault Keeper is informed which content is locked and can re-upgrade at any time to restore access. Content does not degrade silently.

**EDGE-006 — STANDARD:** If a shared link expires while a Guest session is active, the Guest session is terminated within 60 seconds of expiry. The Guest receives an in-app message: "This shared link has expired. Contact the vault owner for continued access."

**EDGE-007 — STANDARD:** Circular family tree relationships (Person A is the parent of Person B, Person B is the parent of Person A) are detected at the time of relationship creation and blocked with a clear error. The error message explains the conflict and links to both profiles.

**EDGE-008 — STANDARD:** If a GEDCOM import produces more than 500 pending profiles in a single import, the import is paused after 500 and the Vault Keeper is notified. They must confirm continuation in batches of 500 to prevent unmanageable inbox flooding.

**EDGE-009 — STANDARD:** If a time-locked content item's designated recipient deletes their FamilyVault account before the unlock date, the time-lock is not automatically redirected. The Vault Keeper is notified that the designated recipient's account is no longer active and is prompted to designate a new recipient or move the content to the next-of-kin chain.

**EDGE-010 — STANDARD:** If the print provider (Lulu Direct) is unavailable at order submission time, the order is queued for up to 72 hours of retries. The Vault Keeper is notified of the delay. If the primary provider is still unavailable after 72 hours, the system automatically falls back to the secondary provider (Blurb) and notifies the Vault Keeper of the provider change.

**EDGE-011 — STANDARD:** A Biographer whose time-limited access grant expires while a contribution is still in review is not removed from the contribution's authorship record. Their access to new content is revoked, but the in-flight contribution retains their name as author and completes the review workflow normally.

**EDGE-012 — STANDARD:** If a vault export (FVAF) is triggered during an active offline sync conflict (unresolved competing versions), the export includes all competing versions with a conflict marker in the FVAF manifest. The export is not blocked by unresolved conflicts.

**EDGE-013 — STANDARD:** If an AI transcription job fails for all providers (primary and fallback), the voice recording is preserved in its original form and the Vault Keeper is notified. The block is created as a Voice block without a transcript, with a status indicator: "Transcription unavailable — you can retry or transcribe manually."

**EDGE-014 — DEPRECATED:** This rule is formally retired. Per API-010, `death.protocol.triggered` is not emitted until the reversal window has fully closed, making post-trigger reversal impossible. The `death.protocol.reversed` event is reserved for internal audit logging only and is never emitted to third-party webhook subscribers. Any code or documentation referencing the reversal webhook scenario must be updated to remove this expectation.

**EDGE-015 — STANDARD:** If a B2B institution's account is suspended or terminated by FamilyVault (non-payment, terms violation), the client vaults within that institution's portfolio are not deleted. They are transferred to a temporary FamilyVault custodial state and each vault owner is notified that their institution's account has been suspended. Vault owners have 90 days to claim their vault independently or transfer to another account.

**EDGE-016 — STANDARD:** A profile subject who submits a Subject Access Request and receives a response may submit a follow-up request if they believe the response was incomplete. Follow-up requests are treated as new SAR submissions with the same one-month response SLA.

**EDGE-017 — STANDARD:** If a Vault Keeper's email address becomes undeliverable (hard bounce), the platform flags the account after 3 hard bounces and notifies all Family Admins that the Vault Keeper's email is undeliverable. The account is not suspended, but a persistent warning is shown in the vault settings panel.

**EDGE-018 — STANDARD:** If a health pattern alert is generated for a profile and that profile subsequently sets the No-AI-Training flag, the alert is immediately invalidated and removed from the Health Alerts panel. The pattern card must not remain visible after the flag is set.

**EDGE-019 — STANDARD:** If a Memorial Addendum contribution is submitted for a deceased profile by a person who is not a Verified Member of the vault (e.g., via a shared memorial link), the contribution enters a separate external memorial queue. The new Vault Keeper or a Family Admin reviews and approves these contributions before they enter the vault. External memorial contributions are not auto-approved under any circumstance.

**EDGE-020 — STANDARD:** If a vault's annual escrow deposit fails (escrow agent unreachable, export generation error), the system retries daily for 7 days. After 7 failed attempts, the Vault Keeper is notified by email and in-app, and the Trust and Safety team is alerted. The failure is logged as VAULT_ESCROW_DEPOSIT_FAILED (CRITICAL severity). The vault continues to operate normally — the escrow failure does not affect vault access or functionality.

---

**QB-001 — MANDATORY:** System questions are globally immutable. No user action, API call, or admin operation may issue an UPDATE to any `questions` row where `source_type = 'SYSTEM'`. User personalisation always creates a `question_overrides` row. Any attempt to UPDATE a system question must be rejected and logged as `SECURITY_VIOLATION_ATTEMPTED`.

**QB-002 — MANDATORY:** Question provenance is non-optional. Every INSERT to `questions` must supply `device_fingerprint_at_creation`, `session_id_at_creation`, and `ip_hash_at_creation`. A nightly audit query detects NULL provenance and emits `DATA_INTEGRITY_WARNING`.

**QB-003 — STANDARD:** Overrides are profile-scoped and non-transferable. A `question_overrides` row is never visible to any profile other than `scoped_to_profile_id`. The API must filter overrides to the authenticated profile before returning question text.

**QB-004 — STANDARD:** Question Books are immutable snapshots. Once a `book_snapshots` row with `snapshot_type = 'QUESTION_BOOK'` reaches `status = 'RENDERED'`, its `snapshot_data` may not be modified. Recall branches the row per PRINT-001.

**QB-005 — STANDARD:** Moving a question is non-destructive. Creating or removing a `question_category_assignments` row must never modify or delete the underlying `questions` row. A question always retains its `primary_category_id` regardless of assignment changes.


### 10.16 Security Testing Requirements

| Security Test | Frequency and Standard |
|--------------|----------------------|
| Penetration test — full scope | Quarterly. External firm. All critical and high findings resolved before next release. |
| Dependency vulnerability scan | Daily automated (Snyk or equivalent). No critical CVEs in production. High CVEs resolved within 7 days. |
| SAST (static analysis) | Every pull request. No new critical issues merged. |
| DAST (dynamic analysis) | Every staging deploy. Reports reviewed by security lead. |
| Authentication security audit | Every authentication system change. OWASP Authentication Cheat Sheet checklist. |
| Secret scanning | Every commit. GitHub Advanced Security or equivalent. No secrets committed. |
| SQL injection test | Every database schema change. Automated plus manual review. |
| Access control matrix test | Every permission system change. All 13 roles times all entity types verified. |
| Privacy boundary test | Every new feature. Verify locked equals non-existent. Verify No-AI flag propagation. |
| Subject Rights workflow test | Every release. Subject Access Request submission to response flow fully exercised in staging. |
| Offline data encryption test | Every mobile release. Verify offline store is encrypted. Verify key derivation. |

---

### 10.17 Legal Compliance & Operations

#### 10.17.1 GDPR Article 30 — Record of Processing Activities (RoPA)

**ROP-001 — MANDATORY:** FamilyVault must maintain a comprehensive Register of Processing Activities as required by GDPR Article 30. The register must be available to supervisory authorities upon request.

**Required Records:**

| Processing Activity | Purpose | Data Subjects | Data Categories | Recipients | Retention | Legal Basis |
|---------------------|---------|---------------|-----------------|------------|-----------|-------------|
| Account registration | Service provision | All users | Identity, contact, auth credentials | Internal | Account lifetime + 7 years | Contract performance (Art. 6(1)(b)) |
| Content storage | Core service | All users | Family history, photos, videos, audio | Internal, escrow agent | Permanent unless erasure requested | Contract performance |
| AI pattern analysis | Feature enhancement | Opted-in users | Pseudonymized content patterns | AI providers (pseudonymized) | 5 years | Legitimate interest |
| Health data processing | Health insights | Health Archive users | Medical history, conditions | Internal only | Permanent | Explicit consent (Art. 9(2)(a)) |
| Subject Access Requests | Legal compliance | Requesters | All personal data | Internal, requester | Permanent | Legal obligation (Art. 6(1)(c)) |
| Activity logging | Security, audit | All users | Action history, IP hashes | Internal, regulators | Category-specific | Legal obligation |

**ROP-002 — MANDATORY:** The RoPA must be reviewed and updated within 30 days of any new processing activity introduction.

**ROP-003 — MANDATORY:** Cross-border transfers must be documented with the transfer mechanism used (SCCs, adequacy decision, or BCRs).

---

#### 10.17.2 GDPR Article 33 — Data Breach Notification Procedure

**BRN-001 — MANDATORY:** FamilyVault must maintain a documented Personal Data Breach Response Procedure.

**Breach Classification:**

| Severity | Definition | Notification Timeline | Actions |
|----------|-----------|---------------------|---------|
| Critical | Unauthorized access to >1000 users; sensitive data exfiltration | Supervisory authority: 72 hours (GDPR standard); internal target: 24 hours; Users: 48 hours | Immediate containment, forensic investigation, regulatory notification |
| High | Unauthorized access to 100-1000 users; encrypted data exposure | Supervisory authority: 48 hours; Users: 72 hours | Containment, investigation, risk assessment |
| Medium | Unauthorized access to <100 users; no sensitive data | Supervisory authority: 72 hours; Users: 7 days | Investigation, remediation, documentation |
| Low | No personal data involved; infrastructure only | Internal documentation only | Remediation, lessons learned |

**BRN-002 — MANDATORY:** The breach response team must include: Security Lead, Privacy Counsel/DPO, Engineering Lead, Communications Lead, and Legal Counsel.

**BRN-003 — MANDATORY:** All breaches must be logged in the Security Incident Register with: detection timestamp, containment timestamp, root cause, affected data subjects count, data categories affected, and remediation actions.

**BRN-004 — MANDATORY:** User notifications must include: nature of breach, likely consequences, measures taken, contact details for more information, and recommended user actions.

---

#### 10.17.3 GDPR Article 35 — Data Protection Impact Assessment (DPIA)

**DPIA-001 — MANDATORY:** A DPIA is required before implementing:
- Health Archive feature (systematic health monitoring)
- AI pattern analysis on family data
- Large-scale processing of special category data
- Systematic monitoring of publicly accessible areas (if applicable)
- New uses of biometric data
- Significant changes to data retention periods

**DPIA-002 — MANDATORY:** The DPIA must include:
1. Systematic description of processing operations
2. Assessment of necessity and proportionality
3. Risk assessment to rights and freedoms
4. Measures to address risks (safeguards, security measures)
5. Residual risk after mitigation

**DPIA-003 — MANDATORY:** High residual risks must be referred to the supervisory authority for prior consultation before processing begins.

**DPIA-004 — MANDATORY:** DPIAs must be reviewed annually and whenever there is a change in processing that may affect risk.

---

#### 10.17.4 GDPR Article 20 — Data Portability

**PRT-001 — MANDATORY:** Data subjects have the right to receive their personal data in a structured, commonly used, machine-readable format (JSON, CSV).

**Portability Package Contents:**
- Profile information (all fields)
- Content blocks authored by or about the subject
- Media assets (original files)
- Activity log entries concerning the subject
- Relationship graph (pseudonymized references to others)
- Version history for all portable content

**PRT-002 — MANDATORY:** The platform must support direct transmission of portability packages to another controller when technically feasible.

**PRT-003 — MANDATORY:** Portability requests must be fulfilled within 30 days, extendable to 60 days for complex requests with notification to the data subject.

---

#### 10.17.5 CCPA/CPRA Compliance Framework

**CCPA-001 — MANDATORY:** California residents have the following rights:
- Right to Know: Categories and specific pieces of personal information collected
- Right to Delete: Personal information (with exceptions)
- Right to Opt-Out: Of sale/sharing of personal information
- Right to Non-Discrimination: For exercising privacy rights
- Right to Correct: Inaccurate personal information (CPRA)
- Right to Limit: Use of sensitive personal information (CPRA)

**CCPA-002 — MANDATORY:** Required Disclosures:
- Categories of personal information collected
- Categories of sources
- Business/commercial purposes
- Categories of third parties shared with
- Specific pieces collected (upon request)

**CCPA-003 — MANDATORY:** "Do Not Sell or Share My Personal Information" link must be prominently displayed on the homepage and in account settings. *Note: While FamilyVault's Privacy Promise states that user data is never sold, this link is provided as a precautionary transparency measure to ensure full CCPA/CPRA compliance and user clarity. Legal counsel should review whether this link is required given the platform's no-sale policy.*

**CCPA-004 — MANDATORY:** Sensitive Personal Information under CPRA includes: health data, biometric information, precise geolocation, racial/ethnic origin, and contents of communications. Separate consent required for use beyond essential service provision.

**CCPA-005 — MANDATORY:** Response timeline: 45 days (extendable once by 45 days with notification).

---

#### 10.17.6 Security Threat Model

**STRIDE Framework:**

| Threat Category | Example | Mitigation |
|-----------------|---------|------------|
| **S**poofing | Impersonating Vault Keeper | Multi-factor authentication, session binding to device fingerprint |
| **T**ampering | Modifying version chain integrity hash | Immutable logs, cryptographic verification, append-only architecture |
| **R**epudiation | Denying action was taken | Comprehensive activity logging, reason field required for all writes |
| **I**nformation Disclosure | Unauthorized access to Secret Vault | Encryption at rest, separate encryption keys, role-based access |
| **D**enial of Service | Overwhelming API with requests | Rate limiting, DDoS protection, autoscaling |
| **E**levation of Privilege | Guest escalating to Vault Keeper | Strict role hierarchy, permission checks on every endpoint |

**OWASP Top 10 Mitigations:**

| Risk | Mitigation |
|------|------------|
| A01: Broken Access Control | Permission matrix enforced at API gateway, service layer, and database query level |
| A02: Cryptographic Failures | AES-256-GCM for encryption, TLS 1.3 minimum, SHA-256 for hashing |
| A03: Injection | Parameterized queries only, strict input validation with Zod |
| A04: Insecure Design | Privacy-by-design, threat modeling for all features |
| A05: Security Misconfiguration | Infrastructure as code, automated security scanning |
| A06: Vulnerable Components | Daily dependency scanning, automated CVE alerts |
| A07: Auth Failures | Stateless JWT with short expiry, refresh token rotation, MFA for privileged roles |
| A08: Integrity Failures | Immutable version chains, integrity hash verification |
| A09: Logging Failures | Mandatory activity logging, tamper-evident audit trail |
| A10: SSRF | Strict egress filtering, no direct user-controlled URLs to backend services |

---

#### 10.17.7 Incident Response Plan

**IRP Severity Levels:**

**P0 — Critical Incident:**
- Definition: Active data breach, total service outage, ransomware, death protocol failure
- Response Time: 15 minutes
- Resolution Target: 4 hours
- Actions: Page on-call team, activate war room, notify executives and legal, prepare regulatory notifications
- Communication: Internal every 30 minutes; user status page updates every hour

**P1 — High Incident:**
- Definition: Major feature unavailable, authentication issues affecting multiple users, AI serving incorrect private content
- Response Time: 30 minutes
- Resolution Target: 8 hours
- Actions: Alert on-call engineer, begin investigation, notify stakeholders
- Communication: Internal hourly; user status page if >1 hour

**P2 — Medium Incident:**
- Definition: Single feature degraded, performance issues, non-critical integration failure
- Response Time: 2 hours
- Resolution Target: 24 hours
- Actions: Ticket assignment, investigation, fix scheduling
- Communication: Internal twice daily

**P3 — Low Incident:**
- Definition: Minor UI issues, cosmetic bugs, documentation errors
- Response Time: 24 hours
- Resolution Target: 72 hours
- Actions: Backlog prioritization
- Communication: Standard ticket updates

**IRP-001 — MANDATORY:** Every incident must have:
- Incident commander assigned
- Timeline of events
- Root cause analysis (within 48 hours for P0/P1)
- Remediation actions
- Prevention measures for recurrence

**IRP-002 — MANDATORY:** Post-incident reviews required for all P0 and P1 incidents within 1 week.

---

#### 10.17.8 Business Continuity Plan

**BCP Objectives:**
- **Recovery Time Objective (RTO):** 4 hours for critical services
- **Recovery Point Objective (RPO):** 1 hour maximum data loss
- **Geographic Distribution:** Primary and DR regions with data residency compliance

**BCP Scenarios:**

| Scenario | Impact | Response |
|----------|--------|----------|
| Primary region failure | Full service outage | Automatic traffic rerouting to DR region within 1 hour; full service recovery within 4 hours (RTO) |
| Database corruption | Data integrity issues | Restore from point-in-time backup (max 1 hour loss) |
| Major provider outage | AI, storage, or email failure | MCP provider failover automatic |
| Cyber attack | Security compromise | Isolate affected systems, activate incident response |
| Physical disaster | Data center destruction | DR region activation, communication to users |

**BCP-001 — MANDATORY:** Quarterly DR failover drills with full service verification.

**BCP-002 — MANDATORY:** Annual BCP review and update.

**BCP-003 — MANDATORY:** All critical team members must have documented secondary contacts and escalation paths.

---

## 11. Data Retention Schedule

| Data Type | Retention Policy |
|-----------|-----------------|
| All activity log entries | Permanent — no expiry. Archived to cold storage after 2 years. Always queryable. |
| Authentication events | Permanent. Separate retention required by SOC 2. |
| Health alert logs | Permanent. Subject to GDPR right to erasure for personal data only (pseudonymized records retained). |
| AI suggestion logs | 5 years minimum. Required for audit and model improvement review. |
| Subject Rights request logs | Permanent. Required for regulatory compliance. |
| Provider health check logs | 90 days hot. 1 year warm. Archive thereafter. |
| Background job logs | 30 days hot. Errors: 1 year. |
| Print job logs | Permanent. Required for order disputes. |
| Export logs | Permanent. Required for data provenance. |
| Offline sync conflict logs | 2 years. Required for dispute resolution. |
| Revenue and billing logs | 7 years minimum. Tax compliance requirement. |
| API developer access logs | 2 years. Security audit trail. |
| Death protocol logs | Permanent. Required for stewardship, legal, and audit traceability. |
| Interview session transcripts and recordings | Permanent unless a lawful erasure obligation requires removal of personal data derivatives. |
| Orphaned Media Archive records | Permanent. Required for immutability and restoration traceability. |
| Trust and Safety action logs | Permanent. Required for legal defensibility, appeals, and safety audit trail. |
| B2B read-grant access logs | 2 years minimum, or longer where contract, legal, or incident-review needs require retention. |

---

## 12. Master Acceptance Criteria

The following acceptance criteria are the authoritative and complete checklist for production launch sign-off. Every item must be satisfied before V1.0 is released. This list is the master acceptance criteria for V1.0. It supersedes all prior references to "Section 15 of the Build Contract" — that section number referred to the original standalone Build Contract document which has been merged into this master document. All internal cross-references that previously pointed to "Section 15" now point to this Section 12.

1. All 12 IMM-C immutability rules (IMM-C-001 through IMM-C-012) are verified by automated test suite with zero failures.
2. All mandatory contract rule families — DB, VC, AS, API, SEC, SAR, AI, MCP, LOG, TEST, DEP, CODE, STW, B2B, DEATH, FED, COM, Trust and Safety, and Legal-Operations (ROP, BRN, DPIA, PRT, CCPA, IRP, BCP) — have zero open violations.
3. External penetration test: zero critical findings and all high findings resolved.
4. Load test at 10,000 concurrent users meets all API SLA targets at P99.
5. Disaster Recovery failover is verified with compliant residency pairing and RTO (less than 4 hours) and RPO (less than 1 hour) targets met.
6. Privacy counsel or designated DPO sign-off verifies: Privacy Notice, Subject Rights Notice, Stewardship Commitment, consent mechanisms, Subject Access Request workflows, erasure propagation, data portability, and regional minor protections.
7. COPPA compliance review explicitly verifies under-13 handling, guardian-consent controls, and restricted processing behaviour.
8. WCAG 2.1 AA accessibility audit passes for all primary user flows, including recovery flows and accessibility modes.
9. Interview Mode and Cognitive Accessibility Mode are validated with elderly-user testing and all critical UX issues are resolved.
10. Access-control verification covers all 13 roles across all key entity types.
11. Data escrow deposit succeeds, FVAF reader is functional offline, and health-bundle local unlock is verified.
12. Product Owner, Legal and Privacy, and Security Lead have all signed off on their respective domains.
13. All Pass 6 mandatory rules (SV-001 through SV-003, REV-001, FED-003, INT-001, AI-015, QR-001, HLTH-001, M7-SEC, LOG-005, PRINT-001) have zero open violations.
14. Protected Sealed State lifecycle (SEAL-001) is implemented, tested, and SECRET_VAULT_SEALED and VAULT_PROTECTED_SEALED_ENTERED events are verified in the event log.
15. All EDGE resolution register items (EDGE-001 through EDGE-020) are implemented with appropriate test coverage per their classification. EDGE-001 and EDGE-002 are MANDATORY/STANDARD rules defined in Section 10.15. EDGE-003 through EDGE-020 are STANDARD rules defined in Section 10.15. All 20 items have enforceable specifications.
16. Secret Vault heir succession, time-lock death protocol interaction, cross-vault GDPR erasure propagation, and revenue escrow chain have all been tested end-to-end.
17. Mental health content handling policy (Mental Health Content Rule M14) has been reviewed by legal counsel and the crisis pathway is documented and tested before Health Archive goes live.
18. B2B SSO fallback to native authentication is tested on every B2B-related release. B2B Read Grant stewardship transfer flow is verified.
19. All 21 build phases (Phase 0 through Phase 20) have passed their stated exit criteria and the Phase 20 launch checklist is fully executed.
20. Manual stewardship and death runbook is documented, support-tested, and dry-run completed prior to launch.
21. All Section 10.17 Legal Compliance & Operations rules (ROP-001–003, BRN-001–004, DPIA-001–004, PRT-001–003, CCPA-001–005, IRP-001–002, BCP-001–003) have zero open violations and have been reviewed and signed off by designated privacy counsel or DPO.

---

*End of FamilyVault Master Product Documentation — Version 2.3 (Unified with v3 Technical Content)*

*This document is the single source of truth for all product, engineering, legal, and compliance decisions for FamilyVault. All contributors must adhere to the standards and rules contained herein.*
