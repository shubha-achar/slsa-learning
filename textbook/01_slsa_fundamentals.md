# Chapter 1 — SLSA fundamentals: evidence, not magic

> **Review date:** 2026-11-04  
> **Version:** SLSA v1.2

**Chapter map:** [1. The motivating problem](#1-the-motivating-problem) · [2. The model](#2-prerequisites-and-the-smallest-useful-model) · [3. Tracks and levels](#3-tracks-and-levels-do-not-flatten-them-into-one-score) · [4. Attestations](#4-provenance-versus-signature-versus-attestation) · [5. Provenance schema](#5-the-v1-provenance-schema-read-it-as-questions) · [6. Verification](#6-verification-evidence-plus-policy) · [7. Threat model](#7-threat-model-and-limitations) · [8. Tooling and STM](#8-practical-tooling-careful-claims) · [9. VSA](#9-vsa-a-different-artifact) · [10. Misconceptions](#10-misconceptions) · [11. Prediction prompt](#11-prediction-prompt) · [12. Exercises](#12-exercises-and-comprehension-check) · [13. Further study](#13-further-study)

**Navigate:** [Home](../README.md) · [Cheatsheet](../cheatsheets/01_slsa_fundamentals.md) · [References](../REFERENCES.md)

## 1. The motivating problem

Suppose you download `widget-2.4.0`. A checksum tells you whether the bytes changed **since someone computed that checksum**. A signature can tell you which key signed those bytes. Neither, by itself, answers: *Which source revision? Which build instructions? Which dependencies? Which builder? Did an attacker alter the build while it ran?*

SLSA (Supply-chain Levels for Software Artifacts) is a vocabulary and set of requirements for answering those questions with increasing assurance. The central evidence is **provenance**: structured metadata describing how an artifact was produced.

### Intuition: a package’s “chain of custody”

Think of a museum object:

1. The object has an identity (a digest).
2. A custody record says where it came from and who handled it.
3. A tamper-evident seal protects the custody record.
4. A museum’s approved list says which seal issuer and custodian are trusted.
5. A visitor checks the object, the record, the seal, and the museum’s expectations.

SLSA supplies the vocabulary for steps 2–5. It does **not** make an untrusted build platform trustworthy merely by adding JSON.

## 2. Prerequisites and the smallest useful model

You need five ideas:

- **Digest:** a cryptographic fingerprint of immutable bytes (normally SHA-256).
- **Source revision:** a precise snapshot such as a Git commit.
- **Builder:** the platform and transitive infrastructure that transforms source into an artifact.
- **Attestation:** signed, machine-readable metadata about an artifact.
- **Verifier:** a policy engine that checks evidence against a root of trust and expectations.

The basic flow is:

```text
source + parameters -> builder -> artifact
                          |           |
                          +-> provenance <- digest
                                |
                         DSSE signature
                                |
              verifier checks identity + expectations
                                |
                         accept / reject
```

The **root of trust** is the verifier's configured set of trusted signer/builder identities and the maximum level assigned to each. **Expectations** are the package-specific facts the verifier expects, such as the canonical repository, allowed build type, and release ref.

The analogy has limits: a museum might have one curator, while SLSA distinguishes the builder from the signer and requires policy to bind their identities.

### Map the analogy to the actual objects

| Museum analogy | Supply-chain object |
|---|---|
| The object | Artifact bytes, identified by a cryptographic digest |
| Custody record | Provenance predicate inside an in-toto Statement |
| Tamper-evident seal | DSSE signature over the exact serialized payload |
| Approved issuer/custodian list | Verifier's root of trust |
| Visitor's acceptance criteria | Artifact-specific expectations and policy |

**Pause and predict:** If a checksum matches but the artifact came from an untrusted repository, should the verifier accept it? Why or why not?

## 3. Tracks and levels: do not flatten them into one score

SLSA v1.2 has separate tracks. A source claim is not a build claim.

### Build track

| Level | Plain-language focus | Typical protection boundary |
|---|---|---|
| **L0** | No SLSA Build guarantees. | No level-specific guarantee. |
| **L1** | Provenance exists and identifies the output by digest and describes how it was produced. | No authenticity or accuracy requirements for provenance. |
| **L2** | Hosted build platform produces authentic provenance; consumers validate it. | It requires some anti-tampering control, but no minimum strength; it does not provide L3's unforgeability or isolation. |
| **L3** | Hosted platform adds unforgeable provenance and isolated builds. | It does not prove the build platform itself is uncompromised or honest. |

These are summaries, not a shortcut around the normative requirements. L2 requires the platform's control plane to generate provenance from platform-obtained data, with limited exceptions: tenants may provide the output names and cryptographic digests in `subject`, as well as fields not required at L2. The platform must have some control against tenant tampering. L3 additionally requires all external parameters to be enumerated, prevents user build steps from accessing provenance-signing secrets, and requires isolated, ephemeral builds. The producer must choose a capable platform, follow a consistent process, distribute provenance, and meet every applicable requirement. A badge, action name, or field value is not proof of a level.

**Pause and predict:** Which level adds protection against one build influencing another build on the same platform?

### Source track

| Level | Focus |
|---|---|
| **L1** | Use a source-control system (SCS) and produce discrete source revisions. |
| **L2** | Preserve change history and generate source provenance. |
| **L3** | Enforce organizational technical controls. |
| **L4** | Require two-party review. |

At **every Source level L1–L4**, the SCS must issue an accessible Source Verification Summary Attestation (Source VSA)—an attestation stating the revision's level (see §9 for the general VSA concept). If a revision has no Source VSA, it is Source L0. Source provenance attestations are an additional requirement from Source L2 onward. Source L4 does not mean Build L4: v1.2's Build track is L0–L3. A project can have strong source controls and a weak build, or the reverse.

## 4. Provenance versus signature versus attestation

These terms answer different questions:

- **Provenance** is the *content*: facts about inputs, process, builder, time, and outputs.
- **Signature** is a cryptographic mechanism: it authenticates bytes under a key. It does not make the signed facts true.
- **Attestation** is the larger object: an authenticated statement plus its envelope. It can contain provenance, an SBOM, a test result, or another predicate.

Raw signing of an artifact says “this key signed these bytes.” A signed provenance attestation says “this attester signed these explicit claims about the artifact.” The verifier must still decide whether the signer and claims are acceptable.

### in-toto Statement and DSSE

The recommended SLSA suite uses:

1. an **in-toto Statement**: `_type`, `subject[]`, `predicateType`, and `predicate`;
2. a **SLSA Provenance v1** predicate;
3. a **DSSE** envelope that authenticates the payload type and exact serialized payload bytes using pre-authentication encoding; JSON canonicalization is not required.

The Statement’s `subject[].digest` binds the predicate to exact artifact bytes. DSSE authenticates the payload and signer; it does not replace policy or provenance semantics.

## 5. The v1 provenance schema: read it as questions

The stable predicate type URI is:

`https://slsa.dev/provenance/v1`

Use that exact URI; the documentation URL and predicate type are not interchangeable.

For SLSA Build L1 with this format, `buildDefinition` and `runDetails` are required. So are `buildDefinition.buildType`, `buildDefinition.externalParameters`, and `runDetails.builder.id`. Fields described below but not in that required set are optional. Provenance parsing treats unset, null, and empty values equivalently.

### BuildDefinition

- `buildType`: what kind of build this is (a URI).
- `externalParameters`: top-level inputs controlled from outside the builder (for example, repository and requested ref); their structure is defined by `buildType`. They may be best-effort at lower levels but MUST be complete at Build L3.
- `internalParameters`: builder-controlled parameters relevant to reproduction or interpretation.
- `resolvedDependencies[]`: artifacts needed at build time, ideally with URI and digest. Completeness is best effort, including at Build L3.

### RunDetails

- `builder.id`: URI identifying the build platform, not simply the person or key that signed.
- `builder.builderDependencies[]`: dependencies of the builder itself, when described.
- `builder.version`: builder component versions.
- `metadata`: optional invocation and timing information such as `invocationId`, `startedOn`, and `finishedOn`. SLSA Provenance v1 removed the old `completeness` indicators; completeness is implied by the trusted builder identity and level requirements.
- `byproducts[]`: additional outputs of the run (for example logs), with descriptors/digests where applicable.

`buildConfig` and `metadata.completeness` are not SLSA Provenance v1 fields. Configuration can be represented through `externalParameters`, dependencies, or byproducts as appropriate to the build type.

### Illustrative JSON (not a level claim)

The following is intentionally **illustrative, incomplete for production policy, and does not claim any SLSA level**. It shows the v1 shape and the subject binding; it is not a DSSE envelope.

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [{
    "name": "registry.example/widget",
    "digest": {
      "sha256": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
    }
  }],
  "predicateType": "https://slsa.dev/provenance/v1",
  "predicate": {
    "buildDefinition": {
      "buildType": "https://example.com/build/widget/v1",
      "externalParameters": {
        "source": "https://example.com/widget@4f2c..."
      },
      "internalParameters": {
        "target": "release"
      },
      "resolvedDependencies": [{
        "uri": "https://example.com/widget@4f2c...",
        "digest": {"sha1": "4f2c..."}
      }]
    },
    "runDetails": {
      "builder": {
        "id": "https://ci.example.com/builder",
        "builderDependencies": [],
        "version": {"runner": "2026.11"}
      },
      "metadata": {
        "invocationId": "run-123",
        "startedOn": "2026-11-04T08:00:00Z",
        "finishedOn": "2026-11-04T08:03:00Z"
      },
      "byproducts": []
    }
  }
}
```

The shortened `4f2c...` source-revision values in this illustrative JSON are placeholders, not complete digests.

**Trace the example:** (1) `subject[0].digest.sha256` is the output's identity; (2) `predicateType` says how to parse the predicate; (3) `predicate.buildDefinition` describes build inputs; (4) `predicate.runDetails.builder.id` names the claimed platform. The example contains no signature: a DSSE envelope would authenticate the exact serialized Statement payload. The `buildType` specification would need to define the `source` parameter used here.

Notice the division of labor: the Statement binds the artifact; the predicate explains the build; the envelope (not shown) authenticates the Statement.

**Pause and predict:** If someone changes only the binary but leaves this Statement unchanged, which field-level comparison detects the mismatch?

## 6. Verification: evidence plus policy

A practical verifier should, in substance:

1. Obtain the artifact and compute its digest.
2. Find an attestation and parse the in-toto Statement.
3. Check the expected Statement `_type` and predicate type (`https://slsa.dev/provenance/v1` for SLSA provenance).
4. Verify the DSSE signature using a root of trust and check that the signer/builder identity is trusted for the claimed builder.
5. Require the Statement subject digest to equal the artifact digest.
6. Look up the trusted level for that builder; do not infer it from a claim in the JSON. Unrecognized or weakly trusted builders must not silently become L2/L3.
7. Compare facts to **expectations**: canonical repository, revision/ref, build type, parameters, artifact name, and acceptable dependencies. Verifiers SHOULD reject unrecognized external parameters.
8. Reject unexpected external parameters or missing required facts.
9. Evaluate every normative requirement for the requested level and record a verification result.
10. Recursively assess dependencies if required by policy; resolved-dependency completeness is best effort.

The root of trust answers “which identities and builders do I recognize?” Expectations answer “what did I intend to release?” Both are needed. A valid signature from the wrong builder or for the wrong repository is not an acceptable release.

**Worked attack:** An attacker substitutes a different binary but leaves the original provenance attached. Recomputing the downloaded artifact's digest and comparing it to `subject[].digest` fails. If the attacker also makes a new statement but names an unapproved source repository, the signature check may succeed while the expectation check rejects the claim.

### What verification cannot do

SLSA does not prove the source is bug-free, the dependency is safe, the reviewer is wise, or the builder has no undiscovered vulnerability. Build L3 does not cover compromise of the build platform itself; trust in that platform remains a verifier decision. Provenance can be incomplete, misleading, or unavailable, and a verifier’s policy can be misconfigured.

## 7. Threat model and limitations

SLSA targets supply-chain tampering and confusion: altered source, substituted dependencies, unauthorized build inputs, post-build artifact/provenance replacement, and attacks on build isolation. Higher levels strengthen provenance integrity and isolation.

It is not a complete secure-development program. It does not replace least privilege, patching, dependency review, reproducible-build work, incident response, or runtime security. It also does not magically detect a malicious change that was legitimately approved and built.

## 8. Practical tooling: careful claims

### GitHub Artifact Attestations

GitHub documents Artifact Attestations as producing signed provenance and states that the feature by itself provides SLSA v1.0 Build L2 (GitHub’s documentation may use that versioned wording). Reusable workflows can add stronger isolation properties. Treat this as a product-specific claim that must be checked against the current documentation, configuration, permissions, and exact artifact path—not as a universal v1.2 level claim for every workflow. Consumers generally verify these attestations with `gh attestation verify`.

### `slsa-github-generator`

Its README says the project is no longer actively maintained, suggests GitHub Artifact Attestations, and warns that existing workflows may not receive updates. Existing attestations remain useful evidence, but maintenance status and runtime compatibility must be assessed before adoption.

### `slsa-verifier`

Its README says it is no longer actively maintained. It describes verification of provenance from supported builders and points GitHub Actions users toward Artifact Attestations plus `gh attestation verify`. It documents only limited support for GitHub `attest-build-provenance` attestations (two specific Bazel builder IDs), not arbitrary GitHub attestations. Do not assume it is the correct verifier for every attestation; check format, signer, builder, and current support.

### STM CLI / “sign the metadata” shortcuts

A hypothetical **STM CLI**—a command-line tool that signs caller-supplied provenance fields—or a GitHub Action can improve authenticity of *the caller's claims*. Such a tool does not, by itself, establish SLSA Build L2: a caller-controlled action may be able to lie about source, inputs, builder, or isolation. In particular:

- a signature is not the same thing as a hosted, trusted build platform;
- `builder.id` identifies the claimed platform; it is not simply “the signer”;
- a signing action must establish how facts were observed and protected, not merely serialize user input;
- a common but incorrect assumption that “signed provenance = L2” is incomplete.

Assess the exact implementation and its trust boundary. For any generator repository, verify current maintenance, release, and compatibility status from its README/repository before making a security or level claim.

## 9. VSA: a different artifact

A **Verification Summary Attestation (VSA)** records that a verifier evaluated an artifact and a policy, and summarizes the verified level(s), verifier, policy, and sometimes dependency levels. Its `verificationResult` may be `PASSED` or `FAILED`; `verifiedLevels` reports the highest verified level per track for the artifact, not its dependencies. It can let consumers trust a verifier without receiving every underlying attestation, and can hide sensitive build details.

A VSA is not the original build provenance and does not make an untrusted provenance trustworthy by itself. It shifts trust to the VSA verifier and its policy. To rely on a VSA, verify its signature against a configured root, subject digest, VSA predicate type, signer–verifier relationship, expected `resourceUri`, `verificationResult == "PASSED"`, and expected `verifiedLevels`.

## 10. Misconceptions

1. **“A hash proves origin.”** It proves identity of bytes, not who produced them.
2. **“A signature proves the build was secure.”** It authenticates a signer, not every fact.
3. **“`builder.id` is the signing key.”** It names a build platform; policy binds platform and signer.
4. **“Any provenance means L1.”** A level requires the applicable normative requirements, not merely JSON.
5. **“GitHub Actions means L2/L3.”** Platform, workflow isolation, provenance generation, signer, and policy all matter.
6. **“Source L4 is Build L4.”** The tracks measure different properties.
7. **“A VSA is provenance.”** It is a signed summary of verification.

## 11. Prediction prompt

Before reading the key, consider a variation on the worked attack in §6: an attacker substitutes a malicious binary, then obtains a valid signature from a different, recognized builder. The old provenance subject digest still matches the original release, but not the substituted bytes. Does the valid signature change the verification result? What evidence should the verifier evaluate?

## 12. Exercises and comprehension check

1. What does a digest establish that a signature alone does not? What does a signature establish that a digest does not?
2. Name the three layers in the recommended SLSA attestation suite.
3. Which field binds a Statement to artifact bytes?
4. Where do `externalParameters` and `resolvedDependencies` live?
5. Why is a trusted root needed if DSSE already verifies a signature?
6. What additional protections does Build L3 require beyond authentic provenance?
7. A signed CLI accepts `builder.id` as a command-line argument. Is that alone evidence of L2? Explain.
8. Which track concerns two-party review, and what evidence is required even at Source L1?
9. When is a VSA useful, and what must a consumer check before relying on one?
10. **Scenario:** A hosted CI job signs provenance, but user-defined build steps can read the signing secret. What level-specific conclusion can you draw?
11. **Scenario:** A trusted platform produces provenance whose `subject[].digest` matches the binary, but the repository is not the expected upstream repository. What should the verifier do?
12. **Scenario:** A report claims Build L2 because an artifact was signed by STM after a self-hosted build. Which evidence is missing before that level can be assessed?

<details>
<summary>Answer key — open after attempting the questions</summary>

**Pause-and-predict prompts:** (1) No: a matching checksum establishes byte identity, not that the source repository is trusted; the repository expectation fails. (2) Build L3 requires isolation between builds, so the platform must prevent one build from influencing another, including later builds. (3) Compare the freshly computed artifact digest to `subject[].digest`; the mismatch detects the substitution.

**Prediction prompt:** No. A valid signature from another recognized builder does not make the subject digest match the substituted artifact. The verifier checks the signature and builder trust, then the artifact digest; the digest mismatch rejects the attestation.

1. A digest identifies bytes; a signature authenticates a message under a key.
2. DSSE envelope, in-toto Statement, and predicate (normally SLSA Provenance).
3. `subject[].digest`, checked against a digest computed from the artifact.
4. `predicate.buildDefinition`.
5. A cryptographically valid signature may come from an untrusted or wrong identity; the root maps identities/builders to trust and levels.
6. Unforgeable provenance and isolated builds, including preventing user-defined build steps from accessing provenance-signing secrets; external parameters must be fully enumerated.
7. No. Caller-provided facts can be false; L2 requires a hosted platform and authentic provenance generated from platform-obtained facts with anti-tampering controls.
8. Source L4; an SCS must issue an accessible Source VSA for every revision at Source L1 or above. Without a VSA, the revision is Source L0.
9. When consumers need a compact, policy-bound verification result or detailed provenance must remain private. Check signature/root, subject digest, VSA predicate type, signer–verifier relationship, expected resource URI, passing result, and expected verified levels.
10. That configuration fails Build L3's secret-isolation requirement. L2 would still require evaluating its separate authenticity and platform requirements; hosted CI plus a signature alone is not enough to conclude L2.
11. Reject it under the package-specific expectations even if the signature is valid.
12. The signature proves the attester signed the message, not that the signer witnessed the build. Assess the build platform, data collection, signer/builder binding, and every L2 requirement; the signature alone is insufficient.

</details>

## 13. Further study

Read the normative pages in [REFERENCES.md](../REFERENCES.md), then inspect one real attestation and document: artifact digest, predicate type, source revision, resolved dependencies, builder identity, signer, root of trust, and expectations. Next study in-toto Statement/DSSE parsing, SLSA threats and mitigations, and your package ecosystem's verification policy.
