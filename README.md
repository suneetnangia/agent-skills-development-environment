# Agent Skills Development Environment (ASDE)

This document focuses on the [Agent Skills](agent_skills.md) (referenced as Skills here onwards) development experience, particularly when Skills are developed by business domain experts (referred to as Skill developers hereafter) rather than software developers. By providing Skill developers with a cohesive environment to develop and iterate on Skill definitions, we reduce the overall time to market for AI solutions and improve the quality of experiences for our customers.

## Out of Scope

This document does not cover release and deployment side of Skills, that should be covered by a separate document which will compliment the thinking here.

## Development

At its core, the development of a Skill should follow the [Agent Skills](agent_skills.md) document, which covers the best practices for authoring Skills.

One of the key parts of the Skill development exercise is to ensure a Skill has strong task adherence, inference generation, triggering thresholds as per the intention of the Skill developer.
This is a difficult task to achieve because of the variables present in the environment in which the Skill is loaded, these variables may include:

1. Model (S/LLM) Variations
2. Context Differences (System Prompts, Other Skills/Tools)
3. Orchestration/Harness Differences (Copilot, Claude, Custom Orchestrator)

Adjusting and validating Skills to accommodate the above variables should be a part of the inner loop i.e. Skill developers should be able to iterate and refine their Skill definitions as they build the Skill. It is also important that they should be able to achieve this without being exposed to the coding artifacts and complex configurations of the underlying harnesses/orchestrators.

### Dependencies

To achieve the goals outlined above, we need a few dependencies to be in place when Skills are being developed.

#### Orchestrator/Harness Access

Access to specific harnesses/orchestrators, including, Custom Orchestrator, M365 Copilot, Claude, ChatGPT. This enables validating Skill on the target surfaces as soon as possible, as not all surfaces are same i.e. they use different orchestrating engines, configurations and support for protocols including UX.

#### Skills and Tools Library 

Access to the same set of Skills and Tools which may be present along side the Skill in development. This brings it closer to a real-world environment, in almost all scenarios your Skill will be sharing space with other Skills and tools, which can impact how your Skill is triggered and which tools it calls.

### Evaluation

A Skill must be evaluated for its task adherence, inference generation, triggering thresholds and evaluation play an important role here. Evaluation is generally split into the following steps which can be independently implemented and made available to Skill developers.

1. **Stimuli** contains prompts, expected answers, grader configuration and harness/orchestrator configuration, these are the test cases with configuration to evaluate appropriately.
2. **Execution** it takes the stumuli as input and runs the configured harness/orchestrator (the harnesses/orchestrators required by the stumuli). This results in a generation of inference data or trajectories in an agreed format (e.g. JSON, CSV).
3. **Grading/Evaluation** inspects the inference data or trajectories and provides scores according to the graders/metrics configured. Graders can be LLM based with rubric config or static, the latter is usually fast and deterministic in scoring e.g. regex matching.


## The Full Picture

```
┌───────────────────────────────────────────────────────────────────────────────────────┐
│                     Agent Skills Development Environment (ASDE)                       │
│                                                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                        INNER LOOP (Skill Developer)                             │  │
│  │                                                                                 │  │
│  │   ┌─────────────────┐         ┌──────────────────────────────────────────┐      │  │
│  │   │  Skill          │         │  Dependencies                            │      │  │
│  │   │  Definition     │────────>│                                          │      │  │
│  │   │  (Authored by   │         │  • Orchestrator/Harness Access           │      │  │
│  │   │   Domain Expert)│         │    (e.g. Custom, M365 Copilot, Claude)   │      │  │
│  │   │                 │         │                                          │      │  │
│  │   └────────┬────────┘         │  • Skills & Tools Library                │      │  │
│  │            │                  │    (Co-loaded Skills, MCP Tools)         │      │  │
│  │            │                  └───────────────────▲──────────────────────┘      │  │
│  │            │                                      │                             │  │
│  │            ▼                                      │                             │  │
│  │   ┌───────────────────────────────────────────────┼──────────────────────────┐  │  │
│  │   │                          EVALUATION           │                          │  │  │
│  │   │                                               │                          │  │  │
│  │   │  ┌──────────────┐    ┌──────────────────┐     │  ┌────────────────────┐  │  │  │
│  │   │  │  1. Stimuli  │──> │   2. Execution   │─────┘  │  3. Grading        │  │  │  │
│  │   │  │              │    │                  │───────>│                    │  │  │  │
│  │   │  │  • Prompts   │    │  • Run harness/  │        │  • LLM-based       │  │  │  │
│  │   │  │  • Expected  │    │    orchestrator  │        │    (rubric)        │  │  │  │
│  │   │  │    answers   │    │  • Generate      │        │  • Static          │  │  │  │
│  │   │  │  • Grader    │    │    trajectories  │        │    (regex, etc.)   │  │  │  │
│  │   │  │    config    │    │    (JSON/CSV)    │        │  • Scores &        │  │  │  │
│  │   │  │  • Harness   │    │                  │        │    pass/fail       │  │  │  │
│  │   │  │    config    │    │                  │        │                    │  │  │  │
│  │   │  └──────────────┘    └──────────────────┘        └─────────┬──────────┘  │  │  │
│  │   │                                                            │             │  │  │
│  │   └────────────────────────────────────────────────────────────┼─────────────┘  │  │
│  │                                                                │                │  │
│  │            <───────────────── Feedback ────────────────────────┘                │  │
│  │            (Refine Skill Definition)                                            │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                       │
│  ┌───────────────────────────────────────────────────────────────────────────────┐    │
│  │                        DEPLOYMENT GATE                                        │    │
│  │                                                                               │    │
│  │   Skill must pass evaluation across all target surfaces before deployment.    │    │
│  │   Cost/time scales with: models × harnesses × Skills × tools                  │    │
│  │                                                                               │    │
│  └───────────────────────────────────────────────────────────────────────────────┘    │
│                                      |                                                │
│                                      ▼                                                │
│                          ┌───────────────────────┐                                    │
│                          │  Deployable Skill     │                                    │
│                          │  (Internal / External)│                                    │
│                          └───────────────────────┘                                    │
└───────────────────────────────────────────────────────────────────────────────────────┘
```

## Solution Packaging

While software developers are comfortable with code and configurations, most Skill developers are not. Their focus is on business domain knowledge and workflow definitions rather than coding artifacts.

We must provide a platform that is friendly to Skill developers, enabling them to configure the dependencies and workflows described in this document in a no-code way. The diagram above illustrates the end-to-end workflow required for developing a viable Skill.


### Skill Development Environment

A web-based application serves as the primary interface for Skill developers, abstracting the underlying infrastructure and wiring the dependencies together. The application should provide the following capabilities:

#### Skill Authoring

- A guided editor for writing and editing Skill definitions (name, description, instructions) without requiring knowledge of the deployment mechanisms.
- Real-time syntax validation and previews of how the Skill will behave when loaded into an orchestrator.
- Version history and diff views to track changes across iterations.

#### Orchestrator/Harness Integration

- A configuration panel to select target harnesses/orchestrators (Custom, M365 Copilot, Claude, ChatGPT) against which the Skill will be tested.
- The application manages authentication, API keys, and connection details so that Skill developers do not need to handle infrastructure setup.
- Access to sandbox environments where the Skill can be loaded and exercised on the selected surface.

#### Skills and Tools Library Management

- A catalogue view of all available Skills and Tools that can be co-loaded alongside the Skill in development.
- Drag-and-drop or checkbox selection to compose the set of companion Skills and Tools for a given test run.
- Visibility into potential conflicts or overlapping triggers with co-loaded Skills.

#### Evaluation Pipeline

- A test-case builder for authoring stimuli (prompts, expected answers, grader selection) through a form-based UI.
- One-click execution that dispatches stimuli to the configured harness, collects trajectories, and runs the selected graders — all without the developer writing scripts or CLI commands.
- A results dashboard displaying scores, pass/fail status, and detailed evidence from graders, with the ability to drill down into individual trajectories.

#### Feedback and Iteration

- Direct links from evaluation results back to the Skill definition editor, highlighting which parts of the Skill may need refinement.
- Side-by-side comparison of evaluation runs to visualise the impact of changes across iterations.
