# Beacon release review

## Pipeline audit
Gap / risk / repaired dependency: Production review could run before validation; `needs: validate` now blocks it until validation succeeds.
Production reviewer control to configure (not demonstrated locally): Require an independent reviewer in GitHub Settings > Environments > production and prevent self-review before real production use.
Secret handling and scope: Retained `${{ secrets.REGISTRY_TOKEN }}` only in the review step; no secret added, used or printed. Repository permissions remain `contents: read`.

## Traceability
Why PATCH is appropriate: v2.3.1 is a backward-compatible bug fix to v2.3.0, so only PATCH increases.
Candidate image and annotated Git tag: `beacon-service:v2.3.1`; annotated `v2.3.1` points to the final repair commit.
Record tag -> code commit in PDF using git rev-parse commands.
Image-tag mutability limitation: Version tags can still be overwritten; production should pin a verified registry digest. The checker does not prove image existence.

## Incident recovery
Evidence and rollback vs forward decision: The fixture reports 35% failures, rising errors and two of three readiness checks failing. Choose rollback because the previous version is healthy and schema-compatible; an unverified forward fix prolongs broad impact.
Known-good target: `beacon-service:v2.3.0`, not the failing v2.3.1 candidate.
Proposed kubectl command (not executed): `kubectl set image deployment/beacon beacon=beacon-service:v2.3.0`
Recovery checks after action: Wait for rollout completion, confirm the restored image/version, verify readiness and health, and ensure request failures return to baseline.

## Release note and decision
Change scope: Repaired one job dependency, the candidate image and rollback target; application, tests, checker, contract and incident fixture are unchanged.
Local validation proof: `python3 -m unittest -v` passes; `python3 check_release.py all` reports pipeline, image and rollback PASS. These are static/local checks only.
Open risk / owner / mitigation: Release owner must configure independent review, verify registry digests and validate the target cluster before requesting production approval.
Current production decision and blocker: Sign-off NOT REQUESTED; NOT DEPLOYED. Local PASS does not authorize production; recovery is proposed only.
Terraform vs Kubernetes responsibilities: Terraform provisions infrastructure; Kubernetes manages application rollout and runtime state. Neither is executed in this exercise.
