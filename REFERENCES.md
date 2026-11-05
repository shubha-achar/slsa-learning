# References

Direct sources, retrieved or checked for this learning artifact on 2026-11-04. Links are authoritative; this tutorial intentionally paraphrases rather than reproduces their text.

1. **SLSA specification v1.2** — [https://slsa.dev/spec/v1.2/](https://slsa.dev/spec/v1.2/)  
   Version, tracks, levels, terminology, attestation formats, and navigation to normative requirements.
2. **Build: Track Basics** — [https://slsa.dev/spec/v1.2/build-track-basics](https://slsa.dev/spec/v1.2/build-track-basics)  
   Informative Build L0–L3 intent and security focus.
3. **Build: Requirements** — [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements)  
   Normative producer/platform responsibilities, hosted/isolation/provenance requirements, and RFC 2119 terms.
4. **Build: Provenance** — [https://slsa.dev/spec/v1.2/build-provenance](https://slsa.dev/spec/v1.2/build-provenance)  
   Provenance v1 predicate, parsing rules, BuildDefinition, RunDetails, and stable predicate URI.
5. **Build: Distributing provenance** — [https://slsa.dev/spec/v1.2/distributing-provenance](https://slsa.dev/spec/v1.2/distributing-provenance)  
   Technical requirements for distributing provenance to consumers.
6. **Source: Requirements for producing source** — [https://slsa.dev/spec/v1.2/source-requirements](https://slsa.dev/spec/v1.2/source-requirements)  
   Source L1–L4, Source VSA requirements, source roles, history, technical controls, and two-party review.
7. **Attestation model** — [https://slsa.dev/spec/v1.2/attestation-model](https://slsa.dev/spec/v1.2/attestation-model)  
   Artifact/attestation/envelope/Statement/predicate layers and recommended DSSE + in-toto suite.
8. **Build: Verifying artifacts** — [https://slsa.dev/spec/v1.2/verifying-artifacts](https://slsa.dev/spec/v1.2/verifying-artifacts)  
   Subject digest, predicate type, roots of trust, builder lookup, expectations, and threat coverage.
9. **Verification Summary Attestation v1** — [https://slsa.dev/spec/v1.2/verification_summary](https://slsa.dev/spec/v1.2/verification_summary)  
   VSA purpose, verifier/policy binding, verified levels, dependency summaries, and predicate URI.
10. **GitHub Docs: Artifact attestations** — [https://docs.github.com/en/actions/concepts/security/artifact-attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations)  
    GitHub’s current generation, claimed SLSA level wording, and `gh attestation verify` workflow. Product claims are version-sensitive.
11. **slsa-github-generator README** — [https://github.com/slsa-framework/slsa-github-generator/blob/main/README.md](https://github.com/slsa-framework/slsa-github-generator/blob/main/README.md)  
    Generator purpose, compatibility, maintenance status (“no longer actively maintained”), migration suggestion, and verification notes.
12. **slsa-verifier README** — [https://github.com/slsa-framework/slsa-verifier/blob/main/README.md](https://github.com/slsa-framework/slsa-verifier/blob/main/README.md)  
    Maintenance status, supported provenance verification workflows, builder limitations, VSA section, limited GitHub-attestation support, and current GitHub verification guidance.
13. **DSSE protocol: Signature Definition** — [https://github.com/secure-systems-lab/dsse/blob/master/protocol.md#signature-definition](https://github.com/secure-systems-lab/dsse/blob/master/protocol.md#signature-definition)  
    Pre-authentication encoding and the exact payload bytes covered by the signature; DSSE does not require JSON canonicalization.
14. **in-toto Attestation Framework: Statement** — [https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md](https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md)  
    Statement fields, subject binding, and predicate conventions.

## Source-fact uncertainty

- SLSA normative pages are the authority for levels and schema; summaries here deliberately omit many requirement details.
- GitHub product behavior, the supported attestation formats, and command-line UX are operational facts that can change. Re-check the linked GitHub docs before deployment.
- “STM CLI” in the chapter is a hypothetical command-line tool that signs caller-supplied provenance fields. The CLI/GitHub Action discussion is a threat-model assessment, not a claim that one specific implementation satisfies or fails a level. Establish facts about the exact implementation, signer boundary, observed inputs, and maintenance status before making a determination.
- No citation date was invented for pages that do not publish one; the review date above records when this artifact was checked.
