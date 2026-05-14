# agent-engines

## What this is

`agent-engines` is an Espanso package of semicolon-prefixed triggers that route your prompts to the correct AI engine. Each trigger expands to a precise, opinionated prompt engineered for one of ten engines: **ado-gateway** (ADO read/write), **spec-engine** (specification from handoff or freeform request), **delivery-engine** (vertical slice planning), **code-quality-engine** (review and refactor), **test-engine** (test planning and E2E), **ops-engine** (CI/CD review and threat modeling), **doc-engine** (README, ADR, AGENTS.md), **repo-engine** (onboarding), **thinking-engine** (assumption auditing, gap analysis), and **caveman** (step-by-step triage mode). Pipeline triggers chain multiple engines end-to-end; modifier triggers adjust the behavior of whatever engine is currently active.

## Install

**Option A — external package install (recommended):**

```sh
espanso package install --external /path/to/agent-engines
```

**Option B — clone into packages directory:**

```sh
git clone https://github.com/moeller-projects/espanso-packages "$(espanso path config)/match/packages/espanso-packages"
# then symlink or copy agent-engines/ into $(espanso path config)/match/packages/agent-engines/
```

After installing, restart Espanso:

```sh
espanso restart
```

## Convention

- All triggers are prefixed with `;` (semicolon) and use kebab-case.
- **String triggers** expand immediately on typing the keyword, then you replace `<placeholders>` inline before sending.
- **Regex triggers** embed the parameter in the keyword itself — type the trigger _plus_ the value (e.g. `;ado-read-wi-12345`, `;tests-plan-cart-service`) and Espanso substitutes it into the prompt automatically. No post-expansion editing is needed for the captured value.
- `word: true` is set on every match, so triggers only expand when typed as a standalone word (not mid-sentence).

## Trigger reference

### Single-skill

| Trigger | Engine | What it routes to |
|---|---|---|
| `;ado-read-wi-<wi-id>` ¹ | ado-gateway | Fetch a work item and emit the handoff JSON |
| `;ado-read-pr` | ado-gateway | Parse a PR URL and return the pr-comments handoff |
| `;ado-read-wi-pr-<wi-id>` ¹ | ado-gateway | Fetch work item + linked PR comments, emit handoff envelope |
| `;ado-write-dry` | ado-gateway | Create a work item — dry-run only, no `--confirm` |
| `;ado-write-confirm` | ado-gateway | Execute the previous dry-run with `--confirm` |
| `;spec-from-handoff` | spec-engine | Produce an implementation-ready spec from an ado-gateway handoff |
| `;spec-from-request` | spec-engine | Turn a freeform request into a spec |
| `;plan` | delivery-engine | Break a confirmed spec into atomic vertical slices |
| `;review` | code-quality-engine | Review a file for correctness, maintainability, modernization |
| `;refactor` | code-quality-engine | Refactor a function with minimal mutation |
| `;tests-plan-<component>` ¹ | test-engine | Design a test plan for the named component |
| `;tests-e2e-<flow>` ¹ | test-engine | Add Playwright E2E tests for the named user flow |
| `;ops-ci` | ops-engine | Review a GitHub Actions workflow |
| `;ops-threat-<service>` ¹ | ops-engine | Threat-model the deployment path for the named service |
| `;doc-readme-<service>` ¹ | doc-engine | Write a README for the named service |
| `;doc-adr` | doc-engine | Capture an architectural decision record |
| `;doc-agents` | doc-engine | Diff-only update to AGENTS.md |
| `;onboard` | repo-engine | Map domains, entry points, conventions, hotspots |
| `;think` | thinking-engine | Pressure-test MVP scope, generate options |
| `;missing` | thinking-engine | Surface what is missing from the current plan |
| `;caveman-full-<service>` ¹ | caveman | Full-mode triage for the named service |
| `;caveman-rest` | caveman | Caveman mode for the rest of the current review |

¹ **Regex trigger** — append the parameter directly: e.g. `;ado-read-wi-12345`, `;tests-plan-cart-service`, `;ops-threat-payment-svc`. The value is substituted into the prompt automatically.

### Pipelines

| Trigger | Pipeline | Steps |
|---|---|---|
| `;pipe-ticket-<wi-id>` ¹ | Full ticket → ship loop | Fetch WI → onboard → spec → slice → test plan → WI comment |
| `;pipe-review` | Full review → patch loop | Review diff → sort findings → patch plan → test plan → residual risk |
| `;pipe-onboard` | Onboard → plan → deliver | Onboard repo → identify critical path → delivery plan → surface unknowns |
| `;pipe-threat` | Threat model → remediation plan | Threat-model boundary → identify surfaces → rank findings → mitigations → abort if unacceptable |
| `;pipe-doc` | Spec → doc → ADR | Consume spec → README → ADR → AGENTS.md diff → surface inconsistencies |

### Modifiers

| Trigger | Effect |
|---|---|
| `;mod-risk` | Add a risk tier (LOW / MEDIUM / HIGH / CRITICAL) to every output block for the rest of this task. Flag any HIGH or CRITICAL items before proceeding. |
| `;mod-scope` | Restate the current scope in IN / OUT format and lock it. Reject any request that expands scope without an explicit scope-change acknowledgment. |
| `;mod-depth` | Increase analysis depth for the rest of this task. Surface additional edge cases, adversarial inputs, and second-order effects. Do not compress findings. |
| `;mod-stop` | Stop the current pipeline. List every open question, blocker, and assumption that must be resolved before proceeding. Do not continue until I respond. |
| `;mod-brief` | For the rest of this task: decisions and actions only. No background prose. Use bullet lists. Maximum 5 lines per block. |
| `;mod-dry` | Treat the next write operation as a dry-run. Preview the full payload and stop. Do not pass --confirm until I explicitly say "proceed". |
| `;mod-confirm` | Execute the previously previewed dry-run with --confirm. Abort if validation fails or if any field differs from the preview. |
| `;mod-caveman` | Drop to caveman mode for the rest of this task. One command at a time. Confirm output before the next step. Narrate what you observe, not what you expect. |

## Anti-patterns

Do not type these — they won't route

| Vague prompt | Why it fails | Use instead |
|---|---|---|
| "Fix the bug" | No engine knows which bug, which file, or which layer | `;review` with the specific file path |
| "Write tests" | No scope, no level (unit/integration/E2E), no target behavior | `;tests-plan` then `;tests-e2e` |
| "Update the docs" | No target audience, no structure, no verification step | `;doc-readme` or `;doc-adr` |
| "Plan the feature" | No confirmed spec to plan against; slices will be incoherent | `;spec-from-request` then `;plan` |
| "Is this safe?" | No boundary defined; threat model will be incomplete | `;ops-threat` with the deployment path scoped |

## Upgrading to forms

Regex triggers (e.g. `;ado-read-wi-12345`) already eliminate post-expansion editing for the captured parameter. For triggers with multiple `<placeholders>` (e.g. `<org>/<project>`), a future version will use Espanso form variables so the editor pops up a dialog to fill each field before expansion. Example of what this will look like for `;ado-write-dry`:

```yaml
- trigger: ";ado-write-dry"
  word: true
  label: "ado-gateway: create work item dry-run"
  replace: "Create an ADO work item titled \"{{title}}\" under {{org}}/{{project}}, type=User Story, area={{area}}. Dry-run only. Do not pass --confirm."
  vars:
    - name: title
      type: form
      params:
        layout: "Title: {{title}}"
    - name: org
      type: form
      params:
        layout: "Org: {{org}}"
```

This is not applied in v0.1.0 to keep the package dependency-free and immediately usable.

## Versioning

Bump the `version:` field in `_manifest.yml` using MAJOR.MINOR.PATCH whenever prompts change wording, new triggers are added, or triggers are removed. If you install a new version of the package without bumping the version, Espanso will skip the update — run `espanso package install --force --external <path>` to override.
