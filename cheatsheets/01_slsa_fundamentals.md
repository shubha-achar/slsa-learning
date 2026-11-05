# SLSA v1.2 — quick reference

[Full chapter](../textbook/01_slsa_fundamentals.md) · [Home](../README.md) · [References](../REFERENCES.md)

## Definitions

- **Artifact:** immutable bytes, usually identified by cryptographic digest.
- **Provenance:** predicate describing how an artifact was produced.
- **Attestation:** a statement about an artifact, authenticated by an envelope.
- **Signature:** authenticates signed bytes and signer; does not prove the asserted facts.
- **Builder:** trusted platform boundary that ran the build and is responsible for provenance.
- **Root of trust:** configured trusted identities and the maximum level assigned to each.
- **Expectations:** package-specific facts the verifier expects, such as source repository and build type.
- **VSA:** a verifier's signed summary of a policy evaluation; its result can pass or fail.

## Tracks and levels

| Track | Level | Short meaning |
|---|---|---|
| Build | L0 | No guarantees |
| Build | L1 | Provenance exists and identifies output digest |
| Build | L2 | Hosted platform generates authentic provenance; consumer validates it |
| Build | L3 | Unforgeable provenance and isolated builds |
| Source | L1 | Version-controlled, discrete source revisions |
| Source | L2 | Reliable history and source provenance |
| Source | L3 | Organizational technical controls |
| Source | L4 | Two-party review |

**Source VSA rule:** the source-control system must issue an accessible Source VSA for every revision at L1–L4. Without one, the revision is Source L0. Source provenance begins at Source L2. Never infer a level from a product name, action, badge, or one field; prove every requirement.

## Attestation and schema map

```text
DSSE envelope (authenticates payload type + serialized payload bytes)
  └── in-toto Statement
      ├── _type = https://in-toto.io/Statement/v1
      ├── subject[].name
      ├── subject[].digest  <-- must match artifact digest
      ├── predicateType = https://slsa.dev/provenance/v1
      └── predicate
          ├── buildDefinition
          │   ├── buildType
          │   ├── externalParameters
          │   ├── internalParameters
          │   └── resolvedDependencies[]
          └── runDetails
              ├── builder: id, builderDependencies[], version
              ├── metadata: invocationId, startedOn, finishedOn
              └── byproducts[]
```

For Build L1, `buildDefinition`, `runDetails`, `buildType`, `externalParameters`, and `builder.id` are required. Other listed fields are optional. `externalParameters` must be complete at L3; dependency completeness remains best effort. `buildConfig` and `metadata.completeness` are not Provenance v1 fields. Unset, null, and empty values parse equivalently.

## Provenance verification checklist (chapter §6)

1. Compute the digest of the artifact being verified.
2. Locate and parse the attestation Statement.
3. Check the expected Statement `_type` and `predicateType`.
4. Verify the signature using a root of trust; bind signer to trusted `builder.id`.
5. Match `subject[].digest` to the computed artifact digest.
6. Look up the builder's trusted level; never accept an untrusted level claim.
7. Compare canonical repository/revision, build type, external parameters, artifact name, and acceptable dependencies to expectations.
8. Reject unexpected external parameters or missing required facts.
9. Evaluate all normative requirements for the requested level and record the result.
10. Recursively assess dependencies if required by policy; resolved-dependency completeness is best effort.

## Related artifact: VSA

A VSA is a separate signed verification result, not provenance. To rely on it, check:

- signature against a configured root of trust;
- subject digest and `https://slsa.dev/verification_summary/v1`;
- `verifier.id` matches the signer;
- expected `resourceUri`;
- `verificationResult` is `PASSED`;
- expected `verifiedLevels`.

## What each thing establishes

| Thing | Answers | Does not answer |
|---|---|---|
| Digest | Are these the same bytes? | Who built them? |
| Raw signature | Which key signed these bytes? | Whether build facts are true |
| Provenance | How does the producer say it was built? | Whether claims are authentic |
| Signed attestation | Who authenticated these claims? | Whether the signer/builder is trusted |
| VSA | What policy result did a verifier report? | Complete underlying build history |

## Common traps

- A valid signature from the wrong signer is not acceptance.
- `builder.id` identifies a build platform; it is not simply the signer identity.
- Caller-provided JSON signed by a CLI does not establish Build L2.
- Build L2 needs hosted execution and authentic provenance; it does not require L3's unforgeability.
- Build L3 adds unforgeable provenance and isolated builds; it does not prove the platform itself is uncompromised.
- GitHub Artifact Attestations documentation and verification commands are product-specific and version-sensitive.
- `slsa-github-generator` and `slsa-verifier` READMEs say they are no longer actively maintained. Check current status before adoption.
- `slsa-verifier` support is builder/format-specific; GitHub Artifact Attestations generally use `gh attestation verify`.
- Source and Build levels are independent.
- A VSA can report `FAILED`; it is not the original provenance.

**Decision rule:** accept only when the evidence is authentic, matches the artifact and expectations, and satisfies policy; reject clear mismatches; escalate missing or ambiguous evidence rather than treating it as success.
