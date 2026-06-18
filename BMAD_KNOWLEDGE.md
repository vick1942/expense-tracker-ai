# BMAD Method — Complete Knowledge Reference
> Compiled: 2026-06-08 | Source: docs.bmad-method.org (32 pages)

---

## Table of Contents
1. [What Is BMAD?](#1-what-is-bmad)
2. [Installation & Setup](#2-installation--setup)
3. [Four Phases of Development](#3-four-phases-of-development)
4. [Workflow Tracks](#4-workflow-tracks)
5. [Named Agents (The Team)](#5-named-agents-the-team)
6. [Core Skills & Tools](#6-core-skills--tools)
7. [Analysis Phase Tools](#7-analysis-phase-tools)
8. [Planning Phase](#8-planning-phase)
9. [Solutioning Phase](#9-solutioning-phase)
10. [Implementation Phase](#10-implementation-phase)
11. [Quality & Review Tools](#11-quality--review-tools)
12. [Project Context](#12-project-context)
13. [Customization System](#13-customization-system)
14. [Working with Existing Projects](#14-working-with-existing-projects)
15. [Document Management](#15-document-management)
16. [Web Bundles](#16-web-bundles)
17. [Official Modules](#17-official-modules)
18. [Community & Custom Modules](#18-community--custom-modules)
19. [Upgrading to v6](#19-upgrading-to-v6)
20. [Getting Help](#20-getting-help)
21. [Quick Reference: All Commands](#21-quick-reference-all-commands)
22. [Decision Guide: When to Use What](#22-decision-guide-when-to-use-what)

---

## 1. What Is BMAD?

BMAD Method is an AI-powered software development framework that uses **specialized named agents** to guide projects through planning, architecture, and implementation phases. It installs directly into your project and works with AI IDEs like Claude Code, Cursor, and Windsurf.

**Core idea:** Each phase produces documents that feed the next. Agents never work in a vacuum — they have structured context from prior phases.

**Key principle:** Always start a **fresh chat** for each workflow. Context contamination from previous workflows degrades output quality.

**Directory structure after install:**
```
your-project/
├── _bmad/              # Framework files, configs, customizations
├── _bmad-output/
│   ├── planning-artifacts/    # PRDs, architecture, UX specs
│   └── implementation-artifacts/  # Epics, stories, investigations
└── .claude/skills/     # Skill files (Claude Code)
    # OR .agents/skills/ (Cursor/Windsurf)
```

---

## 2. Installation & Setup

### Prerequisites
- Node.js 20.12+
- Git (recommended)
- AI-powered IDE (Claude Code, Cursor, Windsurf, etc.)

### Install Command
```bash
npx bmad-method install
```
For prerelease:
```bash
npx bmad-method@next install
```

### Interactive Setup Prompts
1. Installation directory selection
2. Module selection (core, bmm, bmb, cis, gds, tea)
3. Confirmation for stable releases
4. AI tool/IDE integration choices
5. Per-module configuration

### CI/Automation Flags
| Flag | Purpose |
|------|---------|
| `--yes` | Skip all prompts |
| `--modules <list>` | Specify exact module set |
| `--tools <list>` | IDE selection |
| `--pin <module>=<tag>` | Lock specific version |
| `--next=<module>` | Use development branch |
| `--set <module>.<key>=<value>` | Configure options |

### Version Control Axes
**Module Channels:**
- `stable` — Latest released (default)
- `next` — Main branch HEAD at install time
- `pinned` — Specific tagged version

**Installer Binary:**
- `@latest` — Stable installer
- `@next` — Prerelease installer

### Updating
Re-run `npx bmad-method install`. Options:
- **Quick Update** — Refresh without major changes
- **Modify Install** — Full interactive reconfiguration

### Rate Limiting
GitHub allows 60 anonymous API calls/hour. Set `GITHUB_TOKEN` env var for 5000/hour limit when installing multiple external modules.

### Manifest
Install details saved to `_bmad/_config/manifest.yaml` for reproducible deployments.

---

## 3. Four Phases of Development

```
Phase 1: Analysis (OPTIONAL)
  └─ Brainstorming, Research, Product Brief, PRFAQ
         ↓
Phase 2: Planning (REQUIRED)
  └─ PRD creation, UX design
         ↓
Phase 3: Solutioning (METHOD/ENTERPRISE)
  └─ Architecture, Epics & Stories breakdown
         ↓
Phase 4: Implementation
  └─ Sprint planning, Story dev, Code review, Retrospective
```

### Phase 1: Analysis (Optional)
Validates product concepts before committing to development. "Every tool here is optional, but skipping analysis entirely means your PRD is built on assumptions instead of insight."

Tools: Brainstorming, Market/Domain/Technical Research, Product Brief, PRFAQ

### Phase 2: Planning (Required)
Defines requirements and user experience.
- `bmad-prd` — Create/update/validate Product Requirements Document
- `bmad-ux` — UX design (visual + behavioral)

### Phase 3: Solutioning
Bridges planning and implementation. Required for complex/enterprise projects.
- Makes technical decisions explicit before any code is written
- Prevents multiple agents from making conflicting architectural choices
- "10× faster to catch alignment problems during solutioning than during implementation"

### Phase 4: Implementation
Executes work iteratively.
- Sprint planning → Story dev → Code review (all in separate chats)
- Each story: Create → Implement → Review

---

## 4. Workflow Tracks

| Track | Scope | Phases Used | Outputs |
|-------|-------|-------------|---------|
| **Quick Flow** | 1–15 stories, simple features | Skips 1-3 | Tech-spec only |
| **BMad Method** | 10–50+ stories, products | 1–4 | PRD + Architecture + UX |
| **Enterprise** | 30+ stories, compliance systems | 1–4 | PRD + Architecture + Security + DevOps |

### Quick Dev (Quick Flow)
`bmad-quick-dev` — For bug fixes, refactorings, small targeted changes.

**When to use:**
- Bug fixes with identifiable root causes
- Small refactorings confined to few files
- Minor feature tweaks or config adjustments
- Dependency version updates

**Process:**
1. Fresh chat session
2. State your intent (free-form, file paths, issue URLs all work)
3. Clarify & approve the spec
4. Review & push (agent self-reviews, patches issues, commits locally)

**Outputs:** Modified source files, passing tests, conventional commit message.

**Deferred work:** Multiple independent goals → creates `deferred-work.md` to keep sessions focused.

**Escalate to full method when:** Changes affect multiple systems, scope requires discovery, or team documentation is needed.

---

## 5. Named Agents (The Team)

Agents are specialized AI personas that activate by name or by trigger code. "Pull any leg away and the experience collapses" — skills, named agents, and customization are all required.

### Activation
Say the agent's name naturally: "Hey Mary, let's brainstorm" — system detects intent and launches directly.

### The Seven Default Agents

| Agent | Name | Skill ID | Role | Triggers |
|-------|------|----------|------|---------|
| Analyst | **Mary** | `bmad-agent-analyst` | Research, brainstorming, market analysis, product briefs | `BP`, `MR`, `DR`, `TR`, `CB`, `WB`, `DP` |
| Product Manager | **John** | `bmad-agent-pm` | PRD creation, epics, stories, course correction | `PRD`, `CE`, `IR`, `CC` |
| Architect | **Winston** | `bmad-agent-architect` | System architecture, implementation readiness | `CA`, `IR` |
| Developer | **Amelia** | `bmad-agent-dev` | Code, code review, sprint planning, forensic investigation | `DS`, `QD`, `QA`, `CR`, `SP`, `CS`, `ER`, `IN` |
| UX Designer | **Sally** | `bmad-agent-ux-designer` | UX design | `CU` |
| Technical Writer | **Paige** | `bmad-agent-tech-writer` | Documentation, diagrams | `DP`, `WD`, `MG`, `VD`, `EC` |
| *(Orchestrator)* | — | Built-in | Routes work, manages agent roster | — |

### Trigger Types
- **Workflow Triggers** — Load structured step-by-step workflows (`PRD`, `DS`, `CA`)
- **Conversational Triggers** — Start free-form conversations needing descriptive context (Technical Writer's `WD`, `MG`, `VD`, `EC`)

### Activation Sequence (8 steps)
1. Resolve configuration overrides
2. Execute setup steps
3. Adopt persona
4. Load organizational context and user settings
5. Greet user
6. Perform post-greeting configuration
7. Dispatch to matched capability OR
8. Present menu options

### Party Mode
`bmad-party-mode` — Run multiple agents simultaneously. Agents debate, agree, disagree, and build on each other's perspectives.

**Best for:**
- Major decisions with competing tradeoffs
- Brainstorming sessions
- Post-mortems
- Sprint retrospectives

**How it works:** System auto-selects relevant installed agents per message. Real back-and-forth dialogue until satisfied.

---

## 6. Core Skills & Tools

Skills are typed directly into your IDE without needing an agent session. Available in every installation regardless of modules.

### Skills vs Agent Menu Triggers
- **Skills** (`bmad-help`) — Typed directly, immediately launch agent/workflow/task
- **Agent menu triggers** (`PRD`, `DS`) — Short codes used inside an active agent session

### File Locations
- Claude Code: `.claude/skills/`
- Cursor & Windsurf: `.agents/skills/`

### Complete Core Tool List

| Skill | Category | Purpose |
|-------|----------|---------|
| `bmad-help` | Guidance | Inspects project state, detects what's done, recommends next step. Handles 80%+ of questions. |
| `bmad-brainstorming` | Ideation | Structured creative sessions, targets 100+ ideas before organizing |
| `bmad-party-mode` | Collaboration | Multi-agent discussions with natural cross-talk |
| `bmad-spec` | Documentation | Converts any input into canonical SPEC (Why, Capabilities, Constraints, Non-goals, Success signal) |
| `bmad-advanced-elicitation` | Refinement | Iterative improvement via analytical techniques |
| `bmad-shard-doc` | Doc management | Splits large markdown by level-2 headers |
| `bmad-index-docs` | Doc management | Generates organized indexes for doc folders |
| `bmad-review-adversarial-general` | Review | Critical analysis, minimum 10 required findings |
| `bmad-review-edge-case-hunter` | Review | Mechanical path-tracing for boundary conditions |
| `bmad-editorial-review-prose` | Review | Copy-editing for clarity (Microsoft Writing Style Guide) |
| `bmad-editorial-review-structure` | Review | Organizational assessment: cuts, merges, reorganization |
| `bmad-customize` | Configuration | Creates verified override files for agent/workflow behavior |

---

## 7. Analysis Phase Tools

### Brainstorming (`bmad-brainstorming`)
AI as coach/facilitator — ideas come from you, AI creates conditions for your best thinking.

**Process:**
1. Initialization — topic, objectives, limitations
2. Method selection — user-selected, AI-suggested, random, or sequential
3. Guided work — questioning and collaborative support
4. Organization — categorize themes, rank by priority
5. Implementation — define next actions and metrics

**Use when:** Unclear direction, exploring multiple options, overcoming creative obstacles.

### Research Workflows
Three focused tracks:
- **Market Research** — competitors, trends, user sentiment
- **Domain Research** — subject-matter expertise, terminology
- **Technical Research** — feasibility, architecture, implementation

**Use when:** Unfamiliar domain, unvalidated technical capability.

### Product Brief
1-2 page executive summary. Collaborative with Business Analyst (Mary). Captures: vision, target audience, value proposition, scope.

**Use when:** You have existing conviction about the concept but need efficient documentation.

### PRFAQ (Working Backwards)
Amazon's methodology: write the press release before coding. Forces answering difficult customer/stakeholder questions.

**Use when:** Stress-testing concepts, validating value propositions, high-stakes decisions.

### Advanced Elicitation (`bmad-advanced-elicitation`)
Structured re-examination of AI-generated content. Forces a specific reasoning angle vs. generic retry.

**Process:**
1. LLM suggests 5 relevant methods for the content
2. User selects one (or reshuffles)
3. Method applied, improvements displayed
4. Accept, discard, or repeat

**Available methods:**
| Method | What it does |
|--------|-------------|
| Pre-mortem Analysis | Assumes failure, traces root causes backward — **recommended starting point** |
| First Principles Thinking | Removes assumptions, rebuilds from fundamentals |
| Inversion | Finds conditions guaranteeing failure, then avoids them |
| Red Team vs Blue Team | Adversarial critique + defense |
| Socratic Questioning | Challenges claims systematically |
| Constraint Removal | Eliminates boundaries to explore possibilities |
| Stakeholder Mapping | Re-examines from multiple perspectives |
| Analogical Reasoning | Applies lessons from parallel domains |

---

## 8. Planning Phase

### PRD (`bmad-prd`)
Create, update, or validate a Product Requirements Document.
- Answers "What should we build and why?"
- Targets stakeholders with business logic
- Feeds all downstream phases

### UX Design (`bmad-ux`)
Designs visual and behavioral user experience components. Only needed if making UX changes or designing new patterns.

---

## 9. Solutioning Phase

### Why Solutioning Matters
Without it, multiple agents working on different epics make independent conflicting choices:
- Agent 1 uses REST → Agent 2 uses GraphQL → Integration problems
- Agent 1 uses snake_case DB → Agent 2 uses camelCase → Inconsistency

Architecture = shared context all agents consult before coding.

### Architecture (`bmad-create-architecture`)
Makes technical decisions explicit through ADRs (Architecture Decision Records).

**ADR contents:**
- Context
- Alternatives considered
- Chosen solution
- Rationale
- Trade-offs

**Common ADR topics:** API style, database selection, authentication, state management, styling frameworks, testing tools.

**Anti-patterns:**
- Implicit ad-hoc decisions
- Over-documentation causing paralysis
- Stale documents not updated with learnings

### Epics & Stories (`bmad-create-epics-and-stories`)
Breaks architecture into manageable implementation units.

### Implementation Readiness Check (`bmad-check-implementation-readiness`)
Gates entry to Phase 4. Validates PRD, UX, Architecture, and Epics are complete.

---

## 10. Implementation Phase

### Sprint Planning (`bmad-sprint-planning`)
Organizes stories into sprints.

### Story Development (`bmad-dev-story`)
Trigger: `DS` from Developer (Amelia)

Each story cycle (all separate chats):
1. `bmad-create-story` → create story file with full context
2. `bmad-dev-story` → implement the story
3. `bmad-code-review` → review the changes

### Code Review (`bmad-code-review`)
Trigger: `CR` from Developer (Amelia)

Supports effort levels:
- `low/medium` — Fewer, high-confidence findings
- `high→max` — Broader coverage, may include uncertain findings
- `ultra` — Deep multi-agent review in the cloud

Pass `--comment` to post inline PR comments, `--fix` to apply findings.

### Retrospective (`bmad-retrospective`)
Post-sprint review and learning capture.

### Correct Course (`bmad-correct-course`)
Mid-sprint adjustment when scope or direction changes.

### Forensic Investigation (`bmad-investigate`)
Trigger: `IN` from Developer (Amelia)

Disciplined debugging approach with formal case file.

**Key principles:**
- **Evidence grading:**
  - *Confirmed* — Directly observed, cited with specific reference
  - *Deduced* — Logically derived from confirmed evidence
  - *Hypothesized* — Plausible but unconfirmed
- **Stronghold first** — Start with one verified fact, expand outward
- **Hypothesis discipline** — All hypotheses stay in case file; update status when resolved
- **Premise verification** — User's problem description is a hypothesis, not established fact

**Output:** `{implementation_artifacts}/investigations/{slug}-investigation.md`

---

## 11. Quality & Review Tools

### Checkpoint Preview (`bmad-checkpoint-preview`)
Interactive human-in-the-loop code review. Walks through a change by design concern, not file order.

**5-step workflow:**
1. **Orientation** — Intent summary, files modified, modules affected, boundary crossings
2. **Walkthrough** — Changes by design concern with rationale and code refs (top-down)
3. **Detail Pass** — 2-5 high-risk locations tagged: `[auth]`, `[schema]`, `[billing]`, `[public API]`, `[security]`
4. **Testing** — 2-5 manual observation methods with expected outcomes
5. **Wrap-Up** — Approve, rework, or continue discussion

**Not a substitute for:** linters, type checkers, or test suites.

**Primary use:** Handoff after `bmad-quick-dev` completes.

### Adversarial Review (`bmad-review-adversarial-general`)
Reviewers **must** find issues. "No 'looks good' allowed."

**Mechanism:**
- Reviewer adopts cynical stance, assumes problems exist
- System halts at zero findings — requires deeper analysis or documented justification
- Minimum 10 findings required

**Limitation:** AI instructed to find problems → false positives occur. Human judgment required to distinguish legitimate findings from noise/hallucinations.

**Iterations:** Multiple passes yield more findings, but diminishing returns after 2-3 rounds.

### Edge Case Hunter (`bmad-review-edge-case-hunter`)
Mechanical path-tracing for boundary conditions and unhandled scenarios. Different mechanism from adversarial review → different findings.

### Testing

**Built-in QA (`bmad-qa-generate-e2e-tests`):**
Trigger `QA` from Developer. Auto-detects test framework (Jest, Vitest, Playwright, Cypress).

Five-step process:
1. Detect framework
2. Identify features to test
3. Generate API tests (status codes, responses, happy paths, edge cases)
4. Create E2E tests (semantic locators, visible-outcome assertions)
5. Execute and auto-fix failures

Design rules: semantic locators, no CSS selectors, no ordering dependencies, no hardcoded delays.

**Best for:** Small-medium projects, quick coverage without advanced setup.

**Test Architect (TEA) module** — For enterprise-grade needs (see Modules section).

---

## 12. Project Context

### What It Is
`_bmad-output/project-context.md` — An implementation guide/constitution for AI agents. Ensures consistent code generation across all stories and workflows.

### Workflows That Load It
`bmad-create-architecture`, `bmad-create-story`, `bmad-dev-story`, `bmad-code-review`, `bmad-quick-dev`, `bmad-sprint-planning`, `bmad-retrospective`, `bmad-correct-course`

### File Structure
```markdown
# Project Context

## Technology Stack & Versions
- TypeScript 5.x — strict mode, no `any` types
- React 18 — functional components only
- ...

## Critical Implementation Rules
- Components in `/src/components/` with co-located tests
- State: Zustand for global, useState for local
- API: REST with axios, all errors handled via error boundary
- ...
```

Focus on **non-obvious** patterns agents might overlook, not universal best practices.

### Creation Methods
| Method | When to Use |
|--------|-------------|
| Manual | Clear rules defined upfront before architecture |
| `bmad-generate-project-context` after architecture | Capture decisions made during solutioning |
| `bmad-generate-project-context` on existing project | Discover and formalize existing patterns |

### When to Create
- **New project with preferences** → Before architecture phase
- **New project without firm preferences** → After architecture phase
- **Existing project** → Generate to capture current conventions
- **Quick Flow project** → Before or during implementation

**Keep lean** — large files eat into implementation workflow context.

---

## 13. Customization System

### Three-Layer Override Model
```
Priority: personal > team > defaults

1. _bmad/custom/{skill-name}.user.toml   ← personal (gitignored, highest priority)
2. _bmad/custom/{skill-name}.toml        ← team (committed to git)
3. Skill's built-in customize.toml       ← defaults
```

### Merge Rules (by value type)
| Type | Rule |
|------|------|
| Scalars (strings, ints, booleans, floats) | Override wins |
| Tables | Deep recursive merge |
| Arrays with `code`/`id` on every item | Merge by key; matching replaces, new appends |
| Other arrays | Append mode (base + team + user) |

**Critical constraint:** No removal mechanism. Overrides cannot delete base items.

### Per-Skill Customization
File: `_bmad/custom/bmad-{skill}.toml`

**Customizable fields:**
```toml
[agent]
# Scalars (override wins)
icon = "🔧"
communication_style = "terse and direct"

# Arrays (append-only)
persistent_facts = [
  "All PRDs require legal sign-off before engineering kickoff.",
  "file:{project-root}/docs/compliance/hipaa-overview.md",
]

# Menu items (merge by code)
[[agent.menu]]
code = "MY"
description = "My custom task"
skill = "my-custom-skill"
```

**Read-only:** `agent.name` and `agent.title` (hardcoded in SKILL.md).

### Workflow Customization
```toml
[workflow]
activation_steps_prepend = [...]
activation_steps_append = [...]
persistent_facts = [...]
on_complete = "publish to Confluence"  # runs after workflow finishes
```

### Central Configuration
File: `_bmad/custom/config.toml`

Four-layer merge:
1. `_bmad/custom/config.user.toml` (personal, highest)
2. `_bmad/custom/config.toml` (team)
3. `_bmad/config.user.toml` (installer-generated user)
4. `_bmad/config.toml` (installer-generated base)

Use for:
- Rebrand agents: override `[agents.{code}]` fields
- Add fictional agents: new `[agents.{code}]` entries
- Pin team settings: override `[modules.{code}]` or `[core]`

### Resolution Script
```bash
python3 {project-root}/_bmad/scripts/resolve_customization.py \
  --skill {skill-root} \
  --key agent
```
Requires Python 3.11+ (uses stdlib `tomllib`).

### Use Case Matrix
| Need | Surface |
|------|---------|
| Add MCP tool calls to agent | Per-skill `persistent_facts` |
| Add menu item | Per-skill `[[agent.menu]]` |
| Swap output template | Per-skill scalar override |
| Rebrand agent descriptor | Central config `[agents.<code>]` |
| Add fictional agent to roster | Central config new `[agents.<code>]` entry |
| Pin team install settings | Central config `[modules.<code>]` |
| Shape agent behavior across ALL its workflows | Agent-level `persistent_facts` |
| Enforce single workflow conventions | Workflow-level `persistent_facts` |
| Auto-publish artifacts | Workflow `on_complete` hook |

### Six Org-Level Recipes
1. **Shape agent behavior** — Agent-level `persistent_facts` for integrated tools (Context7, Linear)
2. **Enforce workflow conventions** — Workflow `persistent_facts` (e.g., "every brief must include Owner, Target Release, Security Review Status")
3. **Publish outputs automatically** — `on_complete` hook routes to Confluence/Jira
4. **Replace output templates** — `brief_template = "{project-root}/docs/enterprise/brief-template.md"`
5. **Customize agent roster** — Rebrand, add fictional agents, pin team settings
6. **Advanced integrations** — `external_sources`, `external_handoffs`, `doc_standards`, swappable templates

### Decision Rule
> "If the rule applies everywhere an engineer does dev work → customize the dev agent. If it applies only when someone writes a product brief → customize the product-brief workflow. If it changes who's in the room → edit central config."

### Anti-Pattern Warning
**Never copy the full `customize.toml`** — it locks in old default values and blocks future updates from shipping new defaults. Only override deltas.

### `bmad-customize` Skill
Guided authoring for per-skill overrides via natural language. Hand-author central config overrides manually.

---

## 14. Working with Existing Projects

### Three Essential Steps
1. **Clean up planning artifacts** — Remove completed PRD docs from `docs/`, archive or rely on version control
2. **Create project context** — Run `bmad-generate-project-context` to scan codebase patterns
3. **Maintain quality docs** — Keep `docs/` succinct and accurate (intent, business rules, architecture)

### When to Run `bmad-document-project`
- Documentation is absent or outdated
- AI agents need context about current code
- Skip if comprehensive current docs + `docs/index.md` exist

**You can run it at any stage**, even during active development.

### Quick Flow on Existing Codebases
Works fine. Quick Flow will:
- Auto-detect existing technology stacks
- Examine code patterns
- Ask "Should I follow these existing conventions?"
- Respect your choice (maintain consistency OR establish new standards)

### Approach Selection
| Change Type | Approach |
|-------------|---------|
| Bug fix, single feature | `bmad-quick-dev` |
| Major system change | Full BMad Method with appropriate rigor |
| UX changes | Add `bmad-ux` workflow |
| Architecture concerns | Have Winston reference existing docs + scan codebase |

---

## 15. Document Management

### Document Sharding (`bmad-shard-doc`)
Splits large markdown files by level-2 headings.

**Before:**
```
prd.md (50k tokens)
```
**After:**
```
prd/
├── index.md (table of contents with section descriptions)
├── overview.md
├── user-requirements.md
└── ...
```

**Workflow detection priority:**
1. Looks for whole document first
2. Then checks for sharded index file
3. Whole document takes precedence — delete original if you want sharded version used

**Status:** Deprecated — will be unnecessary as workflows evolve and LLMs gain subprocess support.

### Index Docs (`bmad-index-docs`)
Generates organized indexes for documentation folders.

### Document Project (`bmad-document-project`)
Scans and documents current project state for existing codebases.

---

## 16. Web Bundles

### What They Are
Pre-built AI-optimized planning tools for **Google Gemini Gems** or **ChatGPT Custom GPTs**. Install at bmadcode.com/web-bundles.

### Bundle Contents
- `SKILL.md` — Protocol file (uploaded as knowledge)
- `INSTRUCTIONS.md` — Persona block (pasted into platform instructions)
- Supporting data files (CSVs, templates, validation checklists)

### Why Use Web Bundles
**"Plan in the web, build in the IDE"**

| Dimension | Web LLM | IDE |
|-----------|---------|-----|
| Cost model | Flat-rate subscription | Metered tokens |
| Strengths | Canvas, Deep Research, images | Codebase context, terminal |
| Best for | Brainstorming, PRDs, research | Implementation, code review |

### Current Bundles (7)
1. Brainstorming Coach (Osborn/Minto personas)
2. Product Brief Coach (Mary — BMad analyst)
3. PRFAQ Coach (Working Backwards methodology)
4. PRD Coach (Cagan persona)
5. UX Coach (Norman persona)
6. Market & Industry Research (Porter/Christensen frameworks)
7. *(More in development)*

### Session Workflow
1. Open Gem/GPT → persona initiates conversational discovery
2. Persona asks about scope, existing materials, constraints
3. Work progresses in Canvas with continuous updates
4. Export/transfer completed artifact to repo or IDE

### Requirements
- Gemini Gems: Gemini Advanced subscription
- ChatGPT GPTs: Plus, Pro, Business, or Enterprise plan

### Customization Rule
"Customize the instructions, attach the knowledge" — modify the `INSTRUCTIONS.md` block, never the knowledge files. This allows future updates via simple file replacement.

### Building Custom Bundles
Use `bmad-os-skill-to-bundle` utility to convert existing BMad skills into web bundles.

### When NOT to Use
- Work requiring codebase reading/modification
- Mid-implementation scenarios needing context retention
- Teams without paid subscriptions

---

## 17. Official Modules

Installed via `npx bmad-method install`. Select during setup.

### Core + BMM (Installed by Default)
- **core** — Base framework, all core tools
- **bmm** — BMad Method agile suite (all planning/dev workflows)

### Optional Official Modules

#### BMad Builder (`bmb`)
Create custom agents, workflows, and domain-specific modules.
- Agent Builder, Workflow Builder, Module Builder
- Interactive YAML configuration
- npm publishing support
- npm: `bmad-builder`

#### Creative Intelligence Suite (`cis`)
AI tools for structured creativity and early-stage innovation.
- Agents: Innovation Strategist, Design Thinking Coach, Brainstorming Coach, Problem Solver
- Frameworks: SCAMPER, Reverse Brainstorming, problem reframing
- npm: `bmad-creative-intelligence-suite`

#### Game Dev Studio (`gds`)
Structured game development for Unity, Unreal, Godot, and custom engines.
- GDD generation, Quick Dev prototyping, narrative design
- 21+ game type coverage
- npm: `bmad-game-dev-studio`

#### Test Architect Enterprise (`tea`)
Enterprise-grade test strategy, automation guidance, release gate decisions.
- Murat agent (Master Test Architect)
- Nine structured workflows:
  - Test Design — strategy tied to requirements
  - ATDD — acceptance-test-driven development
  - Automate — advanced test generation with utilities
  - Test Review — quality and coverage validation
  - Traceability — requirements mapping for compliance
  - NFR Assessment — performance and security evaluation
  - CI Setup — pipeline configuration
  - Framework Scaffolding — infrastructure setup
  - Release Gate — data-driven deployment decisions
- npm: `bmad-method-test-architecture-enterprise`

**Use TEA when:** Compliance documentation needed, risk-based prioritization required, formal quality gates, complex domains, outgrown single-workflow testing.

---

## 18. Community & Custom Modules

### Sources
1. Community registry (BMad plugins marketplace)
2. Third-party Git repositories
3. Local file paths

### Install from Community
During `npx bmad-method install` → "Would you like to browse community modules?" → Browse by category, featured, or keyword.

Community modules pinned to approved commits for security.

### Install from Custom Source

**Interactive:** After community modules step, answer "Would you like to install from a custom source?"

**Non-interactive:**
```bash
# Single custom source
npx bmad-method install --directory . --custom-source /path/to/my-module --tools claude-code --yes

# Multiple sources (comma-separated)
npx bmad-method install --directory . \
  --custom-source /path/one,https://github.com/org/repo,/path/two \
  --tools claude-code --yes
```

**Supported formats:**
- HTTPS: `https://github.com/org/repo`
- SSH: `git@github.com:org/repo.git`
- Subdirectory: `https://github.com/org/repo/tree/main/my-module`
- Local: `/Users/me/projects/my-module` or `~/projects/my-module`

### Module Discovery Modes
| Mode | Trigger | Behavior |
|------|---------|---------|
| Discovery Mode | Source contains `.claude-plugin/marketplace.json` | Lists all plugins, allows selective install |
| Direct Mode | No marketplace.json | Scans for skills (dirs with `SKILL.md`), installs as single module |

### Local Development Workflow
```bash
npx bmad-method install --directory ~/my-project \
  --custom-source ~/my-module-repo/skills \
  --tools claude-code --yes
```
Local sources reference paths, not copies. Changes auto-detected on reinstall.

**Note:** If source directory is deleted, installed files in `_bmad/` are preserved but module skipped during updates until path restored.

### Output Structure
```
your-project/
└── _bmad/
    ├── core/
    ├── bmm/
    ├── my-module/
    │   ├── my-skill/
    │   │   └── SKILL.md
    │   └── module-help.csv
    └── _config/
        └── manifest.yaml
```

### Creating Custom Modules
1. Run `bmad-module-builder` to scaffold structure
2. Add skills, agents, workflows
3. For discovery mode: include `.claude-plugin/marketplace.json` in repo root
4. Publish to Git or share folder collection
5. Others install via `--custom-source <your-repo-url>`
6. **Test locally first** with local path before publishing

---

## 19. Upgrading to v6

### Eligibility
- Have BMAD v4 installed (`.bmad-method` folder present)
- Want to adopt v6 architecture

### Five-Step Migration

**Step 1 — Execute Installation**
Run `npx bmad-method install` following normal install procedure.

**Step 2 — Address Legacy Installation**
Installer auto-detects v4 deployment. Two options:
- Automated backup-and-removal of `.bmad-method`
- Manual cleanup
*Manually-renamed folders require user deletion.*

**Step 3 — Eliminate IDE Skills**
Remove outdated v4 commands from `.claude/commands/` (especially nested bmad folders). v6 skills install to `.claude/skills/`.

**Step 4 — Relocate Planning Documents**
Transfer existing PRD, brief, architecture, UX design to `_bmad-output/planning-artifacts/` with descriptive filenames.

*Mid-planning projects: consider restarting — v6 workflows with web search yield superior results.*

**Step 5 — Handle Active Development**
1. Complete installation
2. Place `epics.md` or `epics/epic*.md` in planning artifacts
3. Run developer sprint-planning workflow
4. Identify completed epics/stories

### Key Architectural Shifts in v6
- Core framework renamed from "method"
- Config now uses modular YAML files (not direct editing)
- Document handling fully flexible with automatic scanning
- Unified module structure under `_bmad/`

### Module Status Changes
- Game dev modules (Phaser, Unity, Godot) → unified BMGD Module
- Infrastructure/DevOps deprecated (pending replacement)
- Creative writing module awaiting v6 adaptation

---

## 20. Getting Help

### Tier 1: BMad-Help (Fastest)
```
bmad-help
```
Inspects project state, recommends next step. Handles 80%+ of questions.

Alternative syntax: `/bmad-help` or `$bmad-help` depending on platform.

### Tier 2: Source Documentation
**Agent-based approach:** Clone the [BMAD-METHOD GitHub repo](https://github.com/bmad-code-org/BMAD-METHOD) and ask your AI tool to analyze it directly.

**Non-agent approach:** Fetch `llms-full.txt` consolidated doc file into chat (for ChatGPT, Claude.ai without local file access).

**Tips:**
- Ask specific questions, not broad ones
- Verify surprising claims against source material
- Check Discord if claims seem questionable

### Tier 3: Community
| Channel | Purpose |
|---------|---------|
| `help-requests` forum | General questions |
| `#suggestions-feedback` | Feature ideas |
| [Discord](https://discord.gg/gk8jAdXWmj) | Real-time community |
| [GitHub Issues](https://github.com/bmad-code-org/BMAD-METHOD/issues) | Bug reports, feature tracking |

---

## 21. Quick Reference: All Commands

### Installation
```bash
npx bmad-method install              # Standard install
npx bmad-method@next install         # Prerelease
npx bmad-method install --yes        # Non-interactive
```

### Analysis Phase
| Command | What It Does |
|---------|-------------|
| `bmad-brainstorming` | Facilitated creative session |
| `bmad-market-research` | Market analysis workflow |
| `bmad-domain-research` | Domain expertise research |
| `bmad-technical-research` | Technical feasibility research |
| `bmad-product-brief` | Create product brief (Mary) |
| `bmad-prfaq` | Working Backwards press release |
| `bmad-advanced-elicitation` | Deepen any output with reasoning methods |

### Planning Phase
| Command | What It Does |
|---------|-------------|
| `bmad-prd` | Create/update/validate PRD |
| `bmad-edit-prd` | Edit existing PRD |
| `bmad-validate-prd` | Validate PRD completeness |
| `bmad-ux` | UX design workflow |

### Solutioning Phase
| Command | What It Does |
|---------|-------------|
| `bmad-create-architecture` | System architecture design |
| `bmad-create-epics-and-stories` | Break into epics and stories |
| `bmad-check-implementation-readiness` | Gate check before implementation |

### Implementation Phase
| Command | What It Does |
|---------|-------------|
| `bmad-sprint-planning` | Organize sprint |
| `bmad-create-story` | Create story file |
| `bmad-dev-story` | Implement a story |
| `bmad-code-review` | Review code changes |
| `bmad-retrospective` | Sprint retrospective |
| `bmad-correct-course` | Mid-sprint adjustment |
| `bmad-investigate` | Forensic debugging |
| `bmad-qa-generate-e2e-tests` | Generate and run E2E tests |
| `bmad-quick-dev` | Quick fix/small change workflow |

### Review & Quality
| Command | What It Does |
|---------|-------------|
| `bmad-checkpoint-preview` | Human-in-the-loop code review walkthrough |
| `bmad-review-adversarial-general` | Must-find-issues review |
| `bmad-review-edge-case-hunter` | Boundary condition analysis |
| `bmad-editorial-review-prose` | Copy-editing |
| `bmad-editorial-review-structure` | Structure/organization review |

### Project Management
| Command | What It Does |
|---------|-------------|
| `bmad-help` | Project-aware guidance |
| `bmad-generate-project-context` | Scan and capture project patterns |
| `bmad-document-project` | Document existing codebase |
| `bmad-shard-doc` | Split large documents |
| `bmad-index-docs` | Generate doc folder indexes |
| `bmad-sprint-status` | Current sprint status |

### Configuration & Customization
| Command | What It Does |
|---------|-------------|
| `bmad-customize` | Create agent/workflow overrides |
| `bmad-party-mode` | Multi-agent discussion |
| `bmad-spec` | Convert any input to canonical SPEC |

### Agents (invoke by name or skill ID)
| Skill ID | Agent | Invoke |
|----------|-------|--------|
| `bmad-agent-analyst` | Mary | "Hey Mary..." |
| `bmad-agent-pm` | John | "Hey John..." |
| `bmad-agent-architect` | Winston | "Hey Winston..." |
| `bmad-agent-dev` | Amelia | "Hey Amelia..." |
| `bmad-agent-ux-designer` | Sally | "Hey Sally..." |
| `bmad-agent-tech-writer` | Paige | "Hey Paige..." |

---

## 22. Decision Guide: When to Use What

### "Should I use Quick Dev or the full BMad Method?"
```
Change affects 1-3 files, no cascading effects?
  └─ YES → bmad-quick-dev
  └─ NO  → Full method with appropriate track (BMad/Enterprise)
```

### "Which planning track?"
```
1-15 stories, simple feature? → Quick Flow
10-50+ stories, product? → BMad Method
30+ stories, compliance/enterprise? → Enterprise track
```

### "When do I need Solutioning (Phase 3)?"
```
Multiple epics that could be implemented by different agents?
  └─ YES → Solutioning is REQUIRED
  └─ NO  → Can skip for Quick Flow
```

### "Do I need a project-context.md?"
```
New project, have strong tech preferences? → Create BEFORE architecture
New project, flexible on tech? → Create AFTER architecture
Existing project? → Run bmad-generate-project-context NOW
Quick Flow project? → Create before or during implementation
```

### "Which review tool?"
```
Need to find bugs/issues in code? → bmad-review-adversarial-general
Need boundary/edge case analysis? → bmad-review-edge-case-hunter
Need human walkthrough of changes? → bmad-checkpoint-preview
Need prose quality check? → bmad-editorial-review-prose
Need structural reorganization? → bmad-editorial-review-structure
```

### "Analysis: which tool for my situation?"
```
Unclear direction, need to explore? → bmad-brainstorming
Have conviction, need to document? → bmad-product-brief
Need to validate/stress-test? → bmad-prfaq
Unfamiliar market/domain? → bmad-market-research / bmad-domain-research
Technical uncertainty? → bmad-technical-research
Want to deepen any existing output? → bmad-advanced-elicitation
```

### "Where should I do planning work — web or IDE?"
```
Long planning sessions (PRD, research, brief)? → Web Bundle (saves IDE tokens)
Work requiring codebase context? → IDE
Team lacks paid web subscriptions? → IDE only
```

### "Testing approach?"
```
Small-medium project, quick coverage? → bmad-qa-generate-e2e-tests (built-in)
Compliance requirements, risk-based? → Install TEA module
Complex domain, need pre-planning? → Install TEA module
```

---

## Appendix: Key Files Reference

| File | Purpose |
|------|---------|
| `_bmad/_config/manifest.yaml` | Install record, module versions, sources |
| `_bmad-output/project-context.md` | Technical constitution for all agents |
| `_bmad-output/planning-artifacts/` | PRD, architecture, UX specs, epics |
| `_bmad-output/implementation-artifacts/` | Stories, investigations |
| `_bmad/custom/config.toml` | Team-level central configuration |
| `_bmad/custom/config.user.toml` | Personal configuration (gitignored) |
| `_bmad/custom/bmad-{skill}.toml` | Team-level per-skill overrides |
| `_bmad/custom/bmad-{skill}.user.toml` | Personal per-skill overrides (gitignored) |
| `.claude/skills/` | Installed skill files (Claude Code) |
| `.agents/skills/` | Installed skill files (Cursor/Windsurf) |
| `docs/` | Project documentation (keep lean + accurate) |

---

*Source: https://docs.bmad-method.org — 32 pages scraped 2026-06-08*
