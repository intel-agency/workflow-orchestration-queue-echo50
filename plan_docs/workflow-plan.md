# Workflow Execution Plan: project-setup

## 1. Overview

| Field | Value |
|---|---|
| **Workflow Name** | `project-setup` (Dynamic Workflow) |
| **Project Name** | workflow-orchestration-queue (OS-APOW — Opencode-Server Agent Workflow) |
| **Repository** | `intel-agency/workflow-orchestration-queue-echo50` |
| **Working Branch** | `dynamic-workflow-project-setup` |
| **Total Main Assignments** | 6 |
| **Event Assignments** | 3 (`create-workflow-plan`, `validate-assignment-completion`, `report-progress`) |
| **Workflow Trigger** | `orchestrate-dynamic-workflow` command |

**Summary:** This plan orchestrates the initial setup of the workflow-orchestration-queue repository. It progresses through repository initialization, application planning, project structure creation, AGENTS.md authoring, debriefing, and finally PR approval and merge. The system being built is a headless agentic orchestration platform that transforms GitHub Issues into autonomous AI worker tasks via a Sentinel Orchestrator, FastAPI webhook Notifier, and DevContainer-based workers.

---

## 2. Project Context Summary

### 2.1 Product Description

**workflow-orchestration-queue (OS-APOW)** is a headless agentic orchestration platform that shifts AI from a passive co-pilot to an autonomous background production service. It translates GitHub Issues into "Execution Orders" fulfilled by AI agents in isolated DevContainers — achieving "Zero-Touch Construction" where a user opens a Specification Issue and receives a functional, test-passed PR without manual intervention.

### 2.2 Architecture (4 Pillars)

1. **The Ear (Work Event Notifier):** FastAPI webhook receiver with HMAC SHA256 validation, intelligent triage, and WorkItem manifest generation. (`notifier_service.py`)
2. **The State (Work Queue):** GitHub Issues as the database ("Markdown-as-a-Database"), using labels for state management (`agent:queued`, `agent:in-progress`, `agent:success`, `agent:error`) and Assignees as distributed locks.
3. **The Brain (Sentinel Orchestrator):** Persistent async Python background service that polls GitHub, claims tasks, manages worker lifecycle via shell bridge, and handles cost guardrails. (`orchestrator_sentinel.py`)
4. **The Hands (Opencode Worker):** Isolated DevContainer executing markdown-based instruction modules against the cloned codebase, with vector indexing and test verification.

### 2.3 Tech Stack

- **Language:** Python 3.12+
- **Frameworks:** FastAPI, Uvicorn, Pydantic, HTTPX
- **Package Manager:** uv (Rust-based)
- **Infrastructure:** Docker, Docker Compose, DevContainers
- **Scripting:** PowerShell Core (pwsh), Bash
- **AI Runtime:** opencode-server CLI (GLM-5 / Claude 3.5 Sonnet)
- **State/Queue:** GitHub Issues, Labels, Milestones, Projects

### 2.4 Development Phases

| Phase | Name | Description | Status |
|---|---|---|---|
| 0 | Seeding & Bootstrapping | Manual clone, seed plan docs, initial orchestration | Current |
| 1 | The Sentinel (MVP) | Polling engine, shell-bridge dispatcher, status feedback | Upcoming |
| 2 | The Ear (Webhook Automation) | FastAPI webhook receiver, HMAC validation, triage | Upcoming |
| 3 | Deep Orchestration | Architect sub-agent, hierarchical decomposition, self-healing | Upcoming |

### 2.5 Key Design Decisions

- **Script-First Integration:** Uses existing `devcontainer-opencode.sh` rather than reimplementing Docker SDK
- **Polling-First Resiliency:** Sentinel polls GitHub every 60s; webhooks are an optimization layer
- **Provider-Agnostic Interfaces:** `ITaskQueue` / `IWorkQueue` abstract base classes prevent vendor lock-in
- **Security:** Network isolation, ephemeral credentials, log scrubbing, HMAC signature validation

### 2.6 Existing Seed Code

The `plan_docs/` directory contains reference implementations:
- **`orchestrator_sentinel.py`** — 309-line Sentinel with `GitHubQueue`, `WorkItem` models, polling loop, shell-bridge integration
- **`notifier_service.py`** — 147-line FastAPI app with `ITaskQueue` interface, HMAC verification, webhook routing
- **`interactive-report.html`** — React-based interactive visualization of the architecture and development plan

### 2.7 Key Risks

| Risk | Impact | Mitigation |
|---|---|---|
| GitHub API Rate Limiting | High | GitHub App tokens (5,000 req/hr), aggressive caching, long-polling |
| LLM Looping/Hallucination | High | Max steps timeout, cost guardrails, retry counter (>3 → stalled) |
| Concurrency Collisions | Medium | GitHub Assignees as distributed lock semaphore |
| Container Drift | Medium | Periodic `docker-compose down && up` between Epics |
| Security Injection | Medium | HMAC validation, network isolation, credential scrubbing |

---

## 3. Assignment Execution Plan

### 3.1 Event: pre-script-begin — `create-workflow-plan`

| Field | Detail |
|---|---|
| **Short ID** | `create-workflow-plan` |
| **Goal** | Create a comprehensive workflow execution plan for the `project-setup` dynamic workflow |
| **Timing** | Before any main assignment begins |
| **Key Acceptance Criteria** | Dynamic workflow file read; all assignments traced; all plan_docs read; plan covers every assignment; plan presented and committed |
| **Dependencies** | None (first task) |
| **Output** | `plan_docs/workflow-plan.md` committed to working branch |
| **Status** | **IN PROGRESS (this document)** |

---

### 3.2 Assignment 1: `init-existing-repository`

| Field | Detail |
|---|---|
| **Short ID** | `init-existing-repository` |
| **Goal** | Initialize the repository by creating a branch, importing branch protection, creating a GitHub Project, importing labels, renaming workspace/devcontainer files, and opening a PR |
| **Prerequisites** | GitHub auth with `repo`, `project`, `read:project`, `read:user`, `user:email` scopes; `administration: write` for rulesets; `gh` CLI authenticated |
| **Dependencies** | None (first main assignment) |
| **Key Acceptance Criteria** | 1. New branch `dynamic-workflow-project-setup` created; 2. Branch protection ruleset imported from `.github/protected-branches_ruleset.json`; 3. GitHub Project created with Board template (Not Started, In Progress, In Review, Done columns); 4. Project linked to repository; 5. Labels imported from `.github/.labels.json`; 6. Devcontainer name and workspace file renamed to match repo name; 7. PR opened from branch to `main` |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | This repo is a GitHub template clone. The branch should be `dynamic-workflow-project-setup`. The `scripts/test-github-permissions.ps1` and `scripts/import-labels.ps1` scripts are available. The `GH_ORCHESTRATION_AGENT_TOKEN` (not `GITHUB_TOKEN`) must be used for ruleset import due to `administration: write` scope requirements. |
| **Risks** | Ruleset import may fail if PAT lacks `administration: write` scope; GitHub Project creation may require specific org permissions; PR creation requires at least one commit on the branch |

**Detailed Step Summary:**
1. Create branch `dynamic-workflow-project-setup` (must be first)
2. Import branch protection ruleset (idempotent — skip if exists)
3. Create GitHub Project for issue tracking (Board template, 4 columns)
4. Import labels via `scripts/import-labels.ps1`
5. Rename devcontainer name and workspace file to match repo name
6. Create PR from branch to `main`
7. Record PR number as `$pr_num` for later use

---

### 3.3 Assignment 2: `create-app-plan`

| Field | Detail |
|---|---|
| **Short ID** | `create-app-plan` |
| **Goal** | Create a comprehensive application plan documented as a GitHub Issue, based on the plan docs and application template |
| **Prerequisites** | Repository initialized (init-existing-repository complete); plan_docs analyzed |
| **Dependencies** | `init-existing-repository` must complete |
| **Key Acceptance Criteria** | 1. Application template thoroughly analyzed; 2. Plan follows template from Appendix A; 3. Detailed phase breakdown; 4. All components and dependencies planned; 5. Tech stack documented; 6. Testing, documentation, containerization addressed; 7. Risks and mitigations identified; 8. Plan documented as GitHub Issue; 9. Milestones created and issues linked; 10. Issue added to GitHub Project |
| **Pre-assignment Event** | `gather-context` assignment |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | The application template/plan docs are in `plan_docs/` — specifically the Implementation Specification v1 serves as the "application template." The project is Python-based (FastAPI, Pydantic, HTTPX, uv). Planning only — NO code implementation. Tech stack should be documented in `plan_docs/tech-stack.md` and architecture in `plan_docs/architecture.md`. The issue template is at `.github/ISSUE_TEMPLATE/application-plan.md`. |
| **Risks** | The `orchestration:plan-approved` label must NOT be applied by this assignment (it's applied by the post-script-complete event); may need iteration for orchestrator approval |

**Detailed Step Summary:**
1. Analyze the Implementation Specification v1 and all plan_docs
2. Identify languages, frameworks, tools → document in `plan_docs/tech-stack.md`
3. Document architecture, components, design decisions in `plan_docs/architecture.md`
4. Design plan with detailed phases using the application-plan issue template
5. Create GitHub Issue documenting the plan
6. Create milestones based on phases (Phase 0, 1, 2, 3)
7. Link issue to GitHub Project and assign to "Phase 1: Foundation" milestone
8. Apply labels: `planning`, `documentation`
9. Iterate with orchestrator until approved

---

### 3.4 Assignment 3: `create-project-structure`

| Field | Detail |
|---|---|
| **Short ID** | `create-project-structure` |
| **Goal** | Create the actual project structure and scaffolding: solution layout, infrastructure configs, CI/CD, Docker, documentation |
| **Prerequisites** | Application plan exists (create-app-plan complete); project structure documented |
| **Dependencies** | `create-app-plan` must complete |
| **Key Acceptance Criteria** | 1. Solution/project structure created per plan's tech stack; 2. All required files and directories established; 3. Docker, docker-compose configs created; 4. CI/CD pipeline structure established; 5. Documentation structure created; 6. Dev environment validated; 7. Initial commit with scaffolding; 8. Repository summary created (`.ai-repository-summary.md`); 9. All GitHub Actions pinned to SHA |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | Python project with uv. Structure should follow the Implementation Spec: `pyproject.toml`, `uv.lock`, `src/` with `notifier_service.py`, `orchestrator_sentinel.py`, `models/`, `interfaces/`, `scripts/` (shell bridge, auth), `local_ai_instruction_modules/`, `docs/`. Use `uv` for dependency management. Docker healthchecks must use Python stdlib (no curl). All actions in CI workflows must be pinned to SHA. |
| **Risks** | Dockerfile must handle editable installs correctly (`COPY src/` before `uv pip install -e .`); CI workflows must use SHA-pinned actions; build validation must pass |

**Detailed Step Summary:**
1. Create Python project structure (pyproject.toml, src/, models/, interfaces/)
2. Create Docker infrastructure (Dockerfile, docker-compose.yml with Python stdlib healthcheck)
3. Create dev environment (scripts, test structure, build tools)
4. Create documentation (README.md, docs/, API docs, ADRs)
5. Initialize CI/CD (.github/workflows with SHA-pinned actions)
6. Quality validation (build, lint, Docker configs)
7. Create `.ai-repository-summary.md`
8. Initial commit and push

---

### 3.5 Assignment 4: `create-agents-md-file`

| Field | Detail |
|---|---|
| **Short ID** | `create-agents-md-file` |
| **Goal** | Create a comprehensive `AGENTS.md` file at the repository root following the agents.md open specification |
| **Prerequisites** | Repository initialized; application plan exists; project structure created; build/test tooling in place |
| **Dependencies** | `create-project-structure` must complete |
| **Key Acceptance Criteria** | 1. `AGENTS.md` exists at repository root; 2. Contains project overview, tech stack, setup/build/test commands; 3. Contains code style, project structure, testing instructions; 4. Commands validated by running them; 5. Complements (not duplicates) README.md; 6. Committed and pushed |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | Must validate all commands work. The AGENTS.md should document: `uv sync` for deps, `uv run pytest` for tests, `uv run ruff check` for linting, `uv run python -m src.notifier_service` to run notifier, etc. Must cross-reference with `.ai-repository-summary.md`, README.md, and plan_docs. |
| **Risks** | Build/test commands must actually work before documenting them; need to ensure consistency with existing documentation |

**Detailed Step Summary:**
1. Gather project context (README, .ai-repository-summary.md, plan docs, CI workflows)
2. Inventory tech stack (Python 3.12, FastAPI, uv, Pydantic, HTTPX, Docker, etc.)
3. Validate all build/setup/test commands
4. Draft AGENTS.md with: overview, setup commands, project structure, code style, testing, architecture, PR guidelines, common pitfalls
5. Cross-reference with existing docs
6. Final validation — re-run all listed commands
7. Commit and push

---

### 3.6 Assignment 5: `debrief-and-document`

| Field | Detail |
|---|---|
| **Short ID** | `debrief-and-document` |
| **Goal** | Perform comprehensive debriefing: capture learnings, deviations, metrics, and improvement recommendations |
| **Prerequisites** | All preceding assignments complete |
| **Dependencies** | `create-agents-md-file` must complete |
| **Key Acceptance Criteria** | 1. Detailed report created following the 12-section template; 2. All deviations documented; 3. Execution trace saved as `debrief-and-document/trace.md`; 4. Report reviewed and approved; 5. Committed and pushed |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | Must flag any plan-impacting findings as ACTION ITEMS. The debrief should capture: any issues with GitHub permissions, branch protection import, project creation, CI validation failures, etc. The execution trace should include all terminal commands run. Plan Adjustment Mandate applies — review upcoming phases for continued validity. |
| **Risks** | Must be thorough — missing deviations or action items will compound in later phases |

**Detailed Step Summary:**
1. Create debrief report using the 12-section structured template
2. Capture execution trace (commands, files, interactions) → `debrief-and-document/trace.md`
3. Review with orchestrator/stakeholder
4. Flag ACTION ITEMS for plan adjustments
5. Commit and push

---

### 3.7 Assignment 6: `pr-approval-and-merge`

| Field | Detail |
|---|---|
| **Short ID** | `pr-approval-and-merge` |
| **Goal** | Complete full PR approval and merge: resolve review comments, obtain approval, merge, and clean up |
| **Inputs** | `$pr_num` — extracted from `init-existing-repository` output |
| **Prerequisites** | All preceding assignments complete; PR is open with all changes |
| **Dependencies** | `debrief-and-document` must complete; `$pr_num` from `init-existing-repository` |
| **Key Acceptance Criteria** | 1. CI verification passes (remediation loop up to 3 attempts); 2. Code review delegated to `code-reviewer` subagent; 3. Auto-reviewer comments waited for; 4. PR comment protocol executed (`ai-pr-comment-protocol.md`); 5. All threads resolved (GraphQL verification); 6. Stakeholder approval obtained; 7. PR merged; 8. Branch deleted; 9. Related issues closed |
| **Post-assignment Events** | `validate-assignment-completion`, `report-progress` |
| **Project-Specific Notes** | This is an automated setup PR — self-approval by orchestrator is acceptable per the dynamic workflow spec. CI remediation loop (Phase 0.5) is mandatory even for self-approval. On successful merge, delete `dynamic-workflow-project-setup` branch and close related setup issues. Must follow `ai-pr-comment-protocol.md` and `pr-review-comments.md`. |
| **Risks** | CI failures may require up to 3 fix cycles; merge conflicts if main has changed; must commit all local changes before merge |

**Detailed Step Summary:**
1. Phase 0: Pre-flight (confirm PR number, auth, GraphQL snapshot)
2. Phase 0.5: CI verification & remediation (up to 3 attempts)
3. Phase 0.75: Code review delegation & auto-reviewer wait
4. Phase 1: Resolve review comments (execute pr-review-comments)
5. Phase 2: Secure approval
6. Phase 3: Merge, delete branch, close issues
7. Set `result` output: `"merged"`, `"pending"`, or `"failed"`

---

### 3.8 Event: post-assignment-complete — `validate-assignment-completion`

| Field | Detail |
|---|---|
| **Short ID** | `validate-assignment-completion` |
| **Goal** | Validate that each completed assignment has met all acceptance criteria |
| **Timing** | After each main assignment completes |
| **Key Acceptance Criteria** | 1. All required files exist; 2. Verification commands pass (build, test, lint); 3. Validation report created; 4. Pass/fail determined with remediation steps if failed |
| **Dependencies** | Must be delegated to independent QA agent (`qa-test-engineer`) |
| **Project-Specific Notes** | For Python assignments: `uv sync`, `uv run pytest`, `uv run ruff check`. For GitHub operations: delegate `github-expert` to verify live state. Reports saved to `docs/validation/VALIDATION_REPORT_<assignment-name>_<timestamp>.md`. |

---

### 3.9 Event: post-assignment-complete — `report-progress`

| Field | Detail |
|---|---|
| **Short ID** | `report-progress` |
| **Goal** | Generate structured progress reports, capture outputs, validate acceptance criteria, and create checkpoints |
| **Timing** | After each main assignment completes (alongside validate-assignment-completion) |
| **Key Acceptance Criteria** | 1. Structured progress report generated; 2. Step outputs captured; 3. Acceptance criteria validated; 4. Checkpoint state saved; 5. Action items filed as GitHub issues |
| **Project-Specific Notes** | Must include Deviations & Findings and Plan-Impacting Discoveries sections. All action items MUST be filed as GitHub issues (non-negotiable). Progress format: `=== STEP COMPLETE: <step-name> ===` |

---

### 3.10 Event: post-script-complete — Apply `orchestration:plan-approved` Label

| Field | Detail |
|---|---|
| **Action** | Apply `orchestration:plan-approved` label to the application plan issue created during `create-app-plan` |
| **Timing** | After ALL assignments and post-assignment events complete successfully |
| **Dependencies** | All 6 main assignments complete; all post-assignment events complete |
| **Purpose** | Signals that the plan is ready for epic creation, triggering the next phase of the orchestration pipeline |

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    project-setup Dynamic Workflow                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─ PRE-SCRIPT-BEGIN EVENT ──────────────────────────────────────────────┐ │
│  │                                                                       │ │
│  │  [create-workflow-plan]                                               │ │
│  │    → Read dynamic workflow file                                       │ │
│  │    → Trace all 6 assignments + 2 event assignments                   │ │
│  │    → Read all plan_docs/                                              │ │
│  │    → Produce workflow-plan.md                                         │ │
│  │    → Commit to branch                                                 │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─ MAIN ASSIGNMENTS (Sequential) ───────────────────────────────────────┐ │
│  │                                                                       │ │
│  │  [1] init-existing-repository                                         │ │
│  │       ├─ Create branch dynamic-workflow-project-setup                 │ │
│  │       ├─ Import branch protection ruleset                             │ │
│  │       ├─ Create GitHub Project (Board)                                │ │
│  │       ├─ Import labels                                                │ │
│  │       ├─ Rename devcontainer/workspace files                          │ │
│  │       └─ Open PR ($pr_num captured)                                   │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  │  [2] create-app-plan  (PLANNING ONLY — no code)                       │ │
│  │       ├─► [gather-context] (pre-assignment-begin)                     │ │
│  │       ├─ Analyze Implementation Spec + plan docs                      │ │
│  │       ├─ Create tech-stack.md, architecture.md                        │ │
│  │       ├─ Create planning issue (application-plan template)            │ │
│  │       ├─ Create milestones, link to Project                           │ │
│  │       └─ Iterate for approval                                         │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  │  [3] create-project-structure                                         │ │
│  │       ├─ Create Python project scaffolding (pyproject.toml, src/)     │ │
│  │       ├─ Create Docker infrastructure                                 │ │
│  │       ├─ Create CI/CD workflows (SHA-pinned actions)                  │ │
│  │       ├─ Create documentation structure                               │ │
│  │       ├─ Create .ai-repository-summary.md                             │ │
│  │       └─ Quality validation (build, lint, Docker)                     │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  │  [4] create-agents-md-file                                            │ │
│  │       ├─ Gather context, inventory tech stack                         │ │
│  │       ├─ Validate all build/test commands                             │ │
│  │       ├─ Draft AGENTS.md (overview, setup, structure, style, tests)   │ │
│  │       └─ Cross-reference with existing docs                           │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  │  [5] debrief-and-document                                             │ │
│  │       ├─ Create 12-section debrief report                             │ │
│  │       ├─ Capture execution trace                                      │ │
│  │       ├─ Flag ACTION ITEMS for plan adjustments                       │ │
│  │       └─ Iterate for approval                                         │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  │  [6] pr-approval-and-merge  (self-approval OK for setup PR)           │ │
│  │       ├─ Phase 0: Pre-flight check                                    │ │
│  │       ├─ Phase 0.5: CI remediation (up to 3 attempts)                │ │
│  │       ├─ Phase 0.75: Code review delegation                           │ │
│  │       ├─ Phase 1: Resolve review comments                             │ │
│  │       ├─ Phase 2: Secure approval                                     │ │
│  │       └─ Phase 3: Merge, delete branch, close issues                  │ │
│  │            │                                                          │ │
│  │            ├─► [validate-assignment-completion] ──► QA agent          │ │
│  │            └─► [report-progress] ──► File issues for action items     │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─ POST-SCRIPT-COMPLETE EVENT ──────────────────────────────────────────┐ │
│  │                                                                       │ │
│  │  Apply label `orchestration:plan-approved` to the planning issue      │ │
│  │  (created during create-app-plan)                                     │ │
│  │  → Triggers next phase of orchestration pipeline                      │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Open Questions

### 5.1 GitHub Authentication & Permissions

1. **PAT Scope for Ruleset Import:** Does the `GH_ORCHESTRATION_AGENT_TOKEN` environment variable exist and does the associated PAT have `administration: write` scope? If not, the branch protection ruleset import in `init-existing-repository` will fail.
2. **GitHub Project Creation:** Does the authenticated user have permissions to create org-level Projects? Some organizations restrict Project creation to admins.

### 5.2 Application Template Location

3. **Application Template File:** The `create-app-plan` assignment references `plan_docs/ai-new-app-template.md`. This file does not exist in the current `plan_docs/` directory. The Implementation Specification v1 appears to serve as the de facto application template. Should the agent treat the Implementation Specification as the application template, or is there a separate template expected?

### 5.3 Existing Code Integration

4. **Seed Code Disposition:** The `plan_docs/` directory contains reference implementations (`orchestrator_sentinel.py`, `notifier_service.py`, `interactive-report.html`). Should `create-project-structure` move these into the proper `src/` directory structure, or should fresh implementations be scaffolded with these as reference?

### 5.4 CI/CD and Testing

5. **CI Workflow Baseline:** The template repo already has GitHub Actions workflows (`orchestrator-agent.yml`, `publish-docker.yml`, `prebuild-devcontainer.yml`). The `create-project-structure` assignment should create ADDITIONAL CI workflows for the Python project (lint, test, build) without conflicting with existing ones. Need to confirm the expected separation.

### 5.5 Branch Strategy

6. **Branch for Project Setup:** The `init-existing-repository` assignment specifies creating branch `dynamic-workflow-project-setup`. However, the Implementation Specification references a `develop` branch strategy. Should a `develop` branch also be created during setup, or is that deferred to Phase 1?

### 5.7 PR Self-Approval

7. **Self-Approval Scope:** The dynamic workflow states self-approval is acceptable for the setup PR. However, the `pr-approval-and-merge` assignment mandates code review delegation to a `code-reviewer` subagent. Is the self-approval meant to bypass the HUMAN stakeholder approval only, while still requiring the subagent code review? This needs clarification to avoid conflicting interpretations.

### 5.8 Existing AGENTS.md

8. **Existing AGENTS.md:** The repository root already contains an `AGENTS.md` file (the one you're reading instructions from). The `create-agents-md-file` assignment will overwrite this. Should the new `AGENTS.md` replace it entirely, or should project-specific content be appended/merged with the existing template instructions?

---

## 6. Execution Checklist

- [ ] Pre-script-begin: `create-workflow-plan` (THIS DOCUMENT)
- [ ] Assignment 1: `init-existing-repository`
- [ ] Post: `validate-assignment-completion` for init
- [ ] Post: `report-progress` for init
- [ ] Assignment 2: `create-app-plan`
- [ ] Post: `validate-assignment-completion` for plan
- [ ] Post: `report-progress` for plan
- [ ] Assignment 3: `create-project-structure`
- [ ] Post: `validate-assignment-completion` for structure
- [ ] Post: `report-progress` for structure
- [ ] Assignment 4: `create-agents-md-file`
- [ ] Post: `validate-assignment-completion` for agents
- [ ] Post: `report-progress` for agents
- [ ] Assignment 5: `debrief-and-document`
- [ ] Post: `validate-assignment-completion` for debrief
- [ ] Post: `report-progress` for debrief
- [ ] Assignment 6: `pr-approval-and-merge`
- [ ] Post: `validate-assignment-completion` for PR
- [ ] Post: `report-progress` for PR
- [ ] Post-script-complete: Apply `orchestration:plan-approved` label
