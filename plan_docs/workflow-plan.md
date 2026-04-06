# Workflow Execution Plan: Project Setup

## 1. Overview

**Workflow Name:** project-setup  
**Workflow File:** `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md`  
**Project Name:** workflow-orchestration-queue  
**Project Description:** A headless agentic orchestration platform that transforms GitHub Issues into autonomous AI-driven development workflows, eliminating the need for human-in-the-loop interactions.

**Total Assignments:** 6 main assignments + 2 post-assignment events  
**High-level Summary:** This workflow initializes the repository, creates an application plan, sets up the project structure, configures documentation, and merges the setup PR to establish a fully functional development environment for the workflow-orchestration-queue system.

## 2. Project Context Summary

### Key Facts from plan_docs/

**Technology Stack:**
- **Language:** Python 3.12+ (primary), PowerShell Core/Bash (scripts)
- **Framework:** FastAPI (webhook receiver), UV (dependency management)
- **Architecture:** Event-driven, distributed system with four pillars:
  - The Ear (FastAPI webhook receiver)
  - The State (GitHub Issues as database)
  - The Brain (Sentinel orchestrator)
  - The Hands (Opencode worker in DevContainer)
- **Infrastructure:** Docker, Docker Compose, DevContainers
- **API Integration:** GitHub REST API, GitHub App webhooks

**Repository Details:**
- **Repository:** intel-agency/workflow-orchestration-queue-echo50
- **Template Source:** workflow-orchestration-queue-echo50 template
- **Branch:** main
- **Current State:** Fresh clone with plan documents seeded

**Special Constraints:**
- **Self-Bootstrapping:** The system will build itself using its own orchestration capabilities
- **Markdown-as-Database:** GitHub Issues/Labels used as persistence layer
- **Script-First Integration:** Must use existing devcontainer-opencode.sh as primary API
- **Security Critical:** All GitHub Actions must be pinned to commit SHAs (not version tags)
- **Polling-First Architecture:** Resiliency through polling, webhooks as optimization

**Known Risks:**
- GitHub API rate limiting (mitigated by GitHub App Installation tokens)
- Concurrency collisions (mitigated by GitHub Assignees as distributed locks)
- LLM looping/hallucination (mitigated by max_steps timeout and cost guardrails)
- Container drift (mitigated by periodic docker-compose down/up cycles)

## 3. Assignment Execution Plan

### Pre-script-begin Event

| Field | Content |
|---|---|
| **Assignment** | `create-workflow-plan`: Create Workflow Plan |
| **Goal** | Create a comprehensive workflow execution plan for the project-setup dynamic workflow |
| **Key Acceptance Criteria** | • Dynamic workflow file fully read and understood<br>• All referenced workflow assignments traced and read<br>• All plan_docs/ files read and summarized<br>• Workflow execution plan produced and approved<br>• Plan committed to plan_docs/workflow-plan.md |
| **Project-Specific Notes** | This is a self-bootstrapping system. The workflow plan must account for the fact that the system will use its own orchestration to build subsequent phases. Plan docs use a custom format (OS-APOW files) rather than standard ai-new-app-template.md. |
| **Prerequisites** | None (first step) |
| **Dependencies** | None |
| **Risks / Challenges** | Non-standard plan document format may require adaptation of standard planning templates |
| **Events** | None |

### Main Assignment 1

| Field | Content |
|---|---|
| **Assignment** | `init-existing-repository`: Initialize Existing Repository |
| **Goal** | Initialize repository by creating branch, importing branch protection, creating GitHub Project, importing labels, and creating initial PR |
| **Key Acceptance Criteria** | • New branch created (dynamic-workflow-project-setup)<br>• Branch protection ruleset imported<br>• GitHub Project created and linked<br>• Labels imported from .github/.labels.json<br>• Workspace/devcontainer files renamed<br>• PR created to main |
| **Project-Specific Notes** | Must use GH_ORCHESTRATION_AGENT_TOKEN (not GITHUB_TOKEN) for branch protection ruleset import due to administration:write scope requirement. Project name should match repository name. |
| **Prerequisites** | GitHub CLI authenticated, proper permissions (repo, project, administration:write) |
| **Dependencies** | None (first main assignment) |
| **Risks / Challenges** | • Branch protection import may fail if token lacks administration:write scope<br>• PR creation requires at least one commit on branch |
| **Events** | **post-assignment-complete:**<br>• validate-assignment-completion<br>• report-progress |

### Main Assignment 2

| Field | Content |
|---|---|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Create comprehensive application plan based on OS-APOW planning documents |
| **Key Acceptance Criteria** | • Application template analyzed<br>• Plan documented in GitHub Issue using application-plan.md template<br>• Milestones created and linked<br>• Issue added to GitHub Project<br>• Appropriate labels applied (planning, documentation) |
| **Project-Specific Notes** | Plan docs are in custom format (OS-APOW files). Must extract and synthesize information from:<br>• OS-APOW Architecture Guide v3.md<br>• OS-APOW Development Plan v4.md<br>• OS-APOW Implementation Specification v1.md<br>No coding in this assignment - planning only. |
| **Prerequisites** | init-existing-repository completed |
| **Dependencies** | Requires PR from assignment 1 to be open |
| **Risks / Challenges** | Non-standard plan format requires careful analysis and synthesis |
| **Events** | **pre-assignment-begin:** gather-context<br>**on-assignment-failure:** recover-from-error<br>**post-assignment-complete:** report-progress |

### Main Assignment 3

| Field | Content |
|---|---|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create actual project scaffolding: solution structure, Docker configs, CI/CD foundation, documentation structure |
| **Key Acceptance Criteria** | • Solution/project structure created following Python 3.12+ tech stack<br>• Dockerfile and docker-compose.yml created<br>• Development environment configured<br>• Documentation structure established (README, docs/, ADRs)<br>• CI/CD foundation created with SHA-pinned actions<br>• Repository summary created<br>• Initial commit made |
| **Project-Specific Notes** | Tech stack is Python 3.12+, FastAPI, UV. Must create:<br>• pyproject.toml (UV package management)<br>• src/ directory structure<br>• tests/ directory<br>• Dockerfile for FastAPI app<br>• docker-compose.yml for local dev<br>• .github/workflows/ with SHA-pinned actions<br>• .ai-repository-summary.md at root |
| **Prerequisites** | create-app-plan completed |
| **Dependencies** | Requires application plan issue for context |
| **Risks / Challenges** | • All GitHub Actions must use commit SHAs (critical constraint)<br>• Docker healthcheck must use Python stdlib, not curl<br>• UV editable install requires source tree copy before install |
| **Events** | **post-assignment-complete:**<br>• validate-assignment-completion<br>• report-progress |

### Main Assignment 4

| Field | Content |
|---|---|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create or update AGENTS.md configuration file with project-specific instructions |
| **Key Acceptance Criteria** | • AGENTS.md file created/updated with project context<br>• File includes agent-specific instructions and guardrails<br>• File committed to repository |
| **Project-Specific Notes** | AGENTS.md already exists in template. May need to update with workflow-orchestration-queue-specific context. This assignment was not found in the assignments index - may need to be handled as part of create-project-structure or skipped if AGENTS.md is already appropriate. |
| **Prerequisites** | create-project-structure completed |
| **Dependencies** | Requires project structure to be in place |
| **Risks / Challenges** | Assignment definition not found in canonical repository - may need adaptation |
| **Events** | **post-assignment-complete:**<br>• validate-assignment-completion<br>• report-progress |

### Main Assignment 5

| Field | Content |
|---|---|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Create comprehensive debriefing report capturing key learnings, insights, and areas for improvement |
| **Key Acceptance Criteria** | • Detailed report created using structured template<br>• All deviations from assignments documented<br>• Report reviewed and approved<br>• Execution trace saved to debrief-and-document/trace.md<br>• Report committed and pushed |
| **Project-Specific Notes** | Must include all 12 sections of the debrief template. Must flag plan-impacting findings as ACTION ITEMS. This is a self-bootstrapping system, so learnings about the orchestration process are critical. |
| **Prerequisites** | All previous assignments completed |
| **Dependencies** | Requires all prior work to be complete |
| **Risks / Challenges** | Thoroughness required - all deviations must be documented |
| **Events** | **post-assignment-complete:**<br>• validate-assignment-completion<br>• report-progress |

### Main Assignment 6

| Field | Content |
|---|---|
| **Assignment** | `pr-approval-and-merge`: PR Approval and Merge |
| **Goal** | Complete full PR approval and merge process including CI verification, code review, comment resolution, and merge |
| **Key Acceptance Criteria** | • All CI/CD status checks pass (up to 3 remediation attempts)<br>• Code review delegated to code-reviewer subagent<br>• All review comments resolved using ai-pr-comment-protocol.md<br>• Stakeholder approval obtained<br>• PR merged successfully<br>• Source branch deleted<br>• Related issues closed |
| **Project-Specific Notes** | **CRITICAL:** This is an automated setup PR - self-approval by orchestrator is acceptable. Must pass $pr_num from init-existing-repository assignment. Must execute CI remediation loop (Phase 0.5) with up to 3 fix cycles. All actions in workflows must be SHA-pinned. |
| **Prerequisites** | All previous assignments completed, PR number available |
| **Dependencies** | Requires PR number from assignment 1 |
| **Risks / Challenges** | • CI may fail initially - must be prepared for remediation loop<br>• Must follow ai-pr-comment-protocol.md exactly<br>• All local changes must be committed and pushed before merge |
| **Events** | **post-assignment-complete:**<br>• validate-assignment-completion<br>• report-progress |

### Post-script-complete Event

| Field | Content |
|---|---|
| **Event** | Apply `orchestration:plan-approved` label to application plan issue |
| **Goal** | Signal that the plan is ready for epic creation by applying the orchestration:plan-approved label |
| **Key Acceptance Criteria** | • Application plan issue located (from create-app-plan assignment)<br>• Label `orchestration:plan-approved` applied<br>• Output recorded as #events.post-script-complete.plan-approved |
| **Project-Specific Notes** | This label triggers the next phase of the orchestration pipeline. The plan issue was created during create-app-plan assignment. |
| **Prerequisites** | All main assignments and post-assignment events completed |
| **Dependencies** | Requires plan issue number from create-app-plan |
| **Risks / Challenges** | Must locate correct issue if multiple issues exist |
| **Events** | None |

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ PRE-SCRIPT-BEGIN                                                │
│ └─> create-workflow-plan                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ MAIN ASSIGNMENTS (Sequential)                                   │
│                                                                 │
│ 1. init-existing-repository                                     │
│    └─> [Extract PR number for step 6]                          │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
│                              ↓                                  │
│ 2. create-app-plan                                              │
│    └─> [Extract plan issue number for post-script-complete]    │
│    └─> pre-assignment-begin: gather-context                     │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
│                              ↓                                  │
│ 3. create-project-structure                                     │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
│                              ↓                                  │
│ 4. create-agents-md-file                                        │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
│                              ↓                                  │
│ 5. debrief-and-document                                         │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
│                              ↓                                  │
│ 6. pr-approval-and-merge [uses PR from step 1]                 │
│    └─> Phase 0.5: CI Verification & Remediation (up to 3x)     │
│    └─> Phase 0.75: Code Review Delegation                      │
│    └─> Phase 1: Resolve Review Comments                        │
│    └─> Phase 2: Secure Approval                                │
│    └─> Phase 3: Merge & Wrap-Up                                │
│    └─> post-assignment-complete:                               │
│        ├─> validate-assignment-completion                       │
│        └─> report-progress                                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ POST-SCRIPT-COMPLETE                                            │
│ └─> Apply orchestration:plan-approved label to plan issue      │
└─────────────────────────────────────────────────────────────────┘
```

## 5. Open Questions

1. **create-agents-md-file Assignment:** This assignment was not found in the canonical assignments index. Should it be:
   - Skipped entirely (AGENTS.md already exists and is appropriate)?
   - Merged into create-project-structure?
   - Handled as a custom assignment?

2. **GH_ORCHESTRATION_AGENT_TOKEN:** The init-existing-repository assignment requires this token with administration:write scope for branch protection ruleset import. Is this token available in the environment, or should we fall back to GITHUB_TOKEN and handle potential permission errors?

3. **Application Plan Template:** The plan_docs/ directory uses custom OS-APOW format rather than the standard ai-new-app-template.md. Should the create-app-plan assignment:
   - Adapt the standard template to this format?
   - Create a hybrid document?
   - Follow the standard template and reference OS-APOW files as supporting documents?

4. **Self-Approval Protocol:** For pr-approval-and-merge, the workflow states "self-approval by orchestrator is acceptable." Should this be:
   - Automatic approval without formal review?
   - Delegated to code-reviewer agent for formality?
   - Explicit orchestrator approval comment on PR?

## 6. Approval

**Plan Status:** ✅ Ready for Review

This workflow execution plan has been created following the create-workflow-plan assignment guidelines. It covers:
- All 6 main assignments in execution order
- All pre/post events
- Project-specific context from OS-APOW planning documents
- Dependencies and risks
- Open questions requiring stakeholder input

**Next Steps:**
1. Review this plan for accuracy and completeness
2. Address open questions
3. Once approved, proceed with init-existing-repository assignment

---

**Plan Prepared By:** Orchestrator Agent  
**Date:** 2026-04-06  
**Workflow:** project-setup  
**Status:** Pending Approval
