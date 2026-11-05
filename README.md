# SLSA, learned from first principles

This textbook-first guide introduces **SLSA v1.2**. It starts with the intuition—“can I explain where this artifact came from, and can I detect tampering?”—then moves to the data model, levels, and verification.

## Learning path

1. Read [the chapter](textbook/01_slsa_fundamentals.md) in order (about 60–90 minutes).
2. Pause at each prediction prompt and answer the exercises before opening the chapter's answer key.
3. Keep [the cheatsheet](cheatsheets/01_slsa_fundamentals.md) open while tracing the worked statement in Chapter §5.
4. For implementation work, use the normative sources in [REFERENCES.md](REFERENCES.md), not this tutorial alone.
5. Revisit the questions tomorrow and again in a week; explain each answer without looking.

## Prerequisites

You should know what a cryptographic hash, public-key signature, CI job, source revision, and JSON object are. No prior SLSA or in-toto knowledge is required.

## Three-format workflow

- **Textbook:** build a mental model, then learn the formal vocabulary.
- **Worked artifact:** trace the illustrative statement in Chapter §5 from subject digest to predicate; then identify what a DSSE envelope would authenticate.
- **Cheatsheet:** retrieve definitions, level distinctions, schema fields, and verification steps.

## Objectives

By the end, you can:

- explain why provenance differs from a signature (Chapter §§1–4);
- distinguish Build L0–L3 from Source L1–L4 (Chapter §3);
- recognize an in-toto Statement, DSSE envelope, and SLSA Provenance v1 predicate (Chapter §§4–5);
- explain why a verifier needs both a root of trust and artifact-specific expectations (Chapter §6);
- avoid claiming a SLSA level without proving every applicable requirement (Chapter §§3, 6, 8);
- assess GitHub Artifact Attestations, `slsa-github-generator`, `slsa-verifier`, and VSA trade-offs (Chapter §§8–9).

## Scope note

This artifact teaches SLSA v1.2 as published at the review date. Product behavior, repositories, trust registries, and GitHub documentation can change; confirm them before adopting a policy.
