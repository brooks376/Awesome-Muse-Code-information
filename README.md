# Awesome Muse Code [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A comprehensive, community-maintained resource on **Muse Code** — Meta's terminal coding agent powered by Muse Spark 1.2.

[![Last Updated](https://img.shields.io/badge/last%20updated-August%202026-blue)](#changelog)
[![Status](https://img.shields.io/badge/status-early%20beta-orange)](#current-status)
[![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux-lightgrey)](#installation)
[![Model](https://img.shields.io/badge/model-Muse%20Spark%201.2-purple)](#muse-spark-12--the-underlying-model)
[![Context Window](https://img.shields.io/badge/context-1M%20tokens-green)](#muse-spark-12--the-underlying-model)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

---

## Table of Contents

- [What is Muse Code](#what-is-muse-code)
- [Current Status](#current-status)
- [Installation](#installation)
- [Architecture Overview](#architecture-overview)
  - [Agent Loop & Async Background Agents](#agent-loop--async-background-agents)
  - [Local Event Log](#local-event-log)
  - [Git Worktree Fan-Out](#git-worktree-fan-out)
- [Muse Spark 1.2 — The Underlying Model](#muse-spark-12--the-underlying-model)
  - [Co-Training with Muse Code](#co-training-with-muse-code)
  - [Long-Horizon Training](#long-horizon-training)
  - [Self-Improvement Loop](#self-improvement-loop)
  - [Input Modalities](#input-modalities)
- [Built-in Commands](#built-in-commands)
- [Pricing](#pricing)
  - [Standard Tier](#standard-tier)
  - [Contributor Tier](#contributor-tier)
  - [Rate Limits](#rate-limits)
  - [Cost Estimation Example](#cost-estimation-example)
- [Benchmarks](#benchmarks)
  - [Terminal-Bench 2.1](#terminal-bench-21)
  - [DeepSWE 1.1](#deepswe-11)
  - [Meta Internal Coding Bench](#meta-internal-coding-bench)
  - [Artificial Analysis Intelligence Index](#artificial-analysis-intelligence-index)
  - [Benchmark Caveats](#benchmark-caveats)
- [Muse Code vs Competitors](#muse-code-vs-competitors)
- [Case Study: GPU Kernel Optimization](#case-study-gpu-kernel-optimization)
- [Demos from Launch](#demos-from-launch)
- [Known Limitations & Open Questions](#known-limitations--open-questions)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Official Sources](#official-sources)
- [Independent Analysis & Community Resources](#independent-analysis--community-resources)
- [Changelog](#changelog)
- [Contributing](#contributing)
- [Disclaimer](#disclaimer)

---

## What is Muse Code

**Muse Code** is Meta's beta terminal coding agent, released on August 5, 2026, by Meta Superintelligence Labs (MSL). It is designed to take on complete software engineering tasks across large repositories — planning changes, writing code, and validating results — all from the command line.

Muse Code is powered by **Muse Spark 1.2**, a proprietary coding-focused model co-trained with the agent itself. Unlike general-purpose AI assistants that happen to write code, Muse Code is purpose-built as an agentic coding tool with persistent background agents, parallel subagent orchestration, and a crash-safe runtime.

It competes directly with Anthropic's Claude Code and OpenAI's Codex as one of the three major terminal-based AI coding agents on the market in 2026.

> 📘 For a source-traced, independent developer reference with evidence grading and methodology documentation, see **[Muse Code: Independent Guide to Meta's Terminal Coding Agent](https://muse-code.dev/)**.

### Key Highlights

- **Terminal-native**: pure CLI tool, no app interface — install and run from your terminal
- **Persistent subagents**: specialized background agents stay alive for an entire session
- **Parallel execution**: fan-out to multiple subagents working in isolated git worktrees
- **Crash-safe**: append-only local event log enables replay-exact session recovery
- **1M token context window**: the full context capacity of Muse Spark 1.2
- **Co-trained**: model and harness trained together for tighter integration
- **Aggressive pricing**: contributor tier at $0.10/$0.20 per million tokens (with data-sharing consent)

---

## Current Status

| Field | Detail |
|-------|--------|
| **Status** | Early Beta |
| **Release Date** | August 5, 2026 |
| **Developed by** | Meta Superintelligence Labs (MSL) |
| **Led by** | Alexandr Wang (Chief AI Officer, Meta) |
| **Model** | Muse Spark 1.2 (`muse-spark-1.2`) |
| **Platforms** | macOS, Linux |
| **Windows Support** | Not officially announced (WSL may work) |
| **API Access** | Meta Model API (public preview, expanded global access) |
| **Open Source** | No — proprietary, closed-weight, cloud-hosted |
| **Third-Party Access** | Available via OpenRouter |

---

## Installation

Muse Code installs via a single shell command on macOS and Linux:

```bash
curl -fsSL https://dev.meta.ai/install.sh | bash
```

Then navigate to your project and start the agent:

```bash
cd /path/to/your/project
muse
```

### Before You Install

> ⚠️ **Security reminder**: piping a remote script into a shell is a powerful operation. Before executing:
> - Verify the official domain (`dev.meta.ai`)
> - Review current authentication and terms of service
> - Inspect the script source if possible
> - Test in a non-critical repository first

### System Requirements

- **Operating Systems**: macOS, Linux
- **Authentication**: Meta developer account required
- **Network**: Cloud-hosted inference — requires internet connection
- **Git**: Required for worktree isolation features

For the official installation guide, see [Meta's developer documentation](https://developer.meta.com/ai/resources/blog/build-with-muse-code/).

---

## Architecture Overview

Muse Code is not a thin wrapper around a language model. It is a purpose-built agent runtime with a distinct architecture. Understanding the separation between the agent (Muse Code) and the model (Muse Spark 1.2) is essential.

| Dimension | Muse Code (Agent) | Muse Spark 1.2 (Model) |
|-----------|-------------------|------------------------|
| **Role** | Plans, acts, validates, manages sessions | Generates model outputs for the agent loop |
| **State** | Local event log, goals, tool results, agent state | Input context and generated output within inference |
| **Actions** | Invokes tools, coordinates isolated worktrees | Does not edit a repository without a harness |
| **Access** | Terminal beta (macOS/Linux) | Muse Code, Meta Model API, partner platforms |

> 📐 For a detailed technical breakdown of how the runtime works, see [How Muse Code Works — Independent Analysis](https://muse-code.dev/how-muse-code-works/).

### Agent Loop & Async Background Agents

Most coding agents spawn a fresh subagent for every task and discard it afterward. Muse Code takes a different approach: it maintains a set of **async background agents** that remain active for the duration of an entire session.

This means:
- Context accumulated during a session (files inspected, decisions made, patterns learned) carries forward instead of resetting
- Background agents decide for themselves when to report back to the main agent
- Reduced latency because the agent doesn't have to re-gather repository context for every new request
- Less manual steering on complex, multi-step tasks

Meta's architecture description from the research announcement:

> The system uses "a simple agent loop plus a set of async background agents to enhance the main agent's capability."

### Local Event Log

Every action Muse Code takes is recorded in an **append-only local event log**:

- Model calls
- Tool runs
- Approvals
- Code edits

This log serves as the single source of truth for the session and enables two critical properties:

1. **Replay-exact**: the session can be reconstructed precisely from the log
2. **Restart-safe**: after a crash, the agent can resume from the last recorded step

This design is particularly relevant for long-horizon tasks. If a session crashes 20 hours into a complex operation, the agent picks up exactly where it stopped — no lost work, no re-prompting.

For enterprise engineering teams, this local audit trail may also serve as an important compliance and traceability feature.

### Git Worktree Fan-Out

When a task is large enough to parallelize, Muse Code splits it across **separate subagents**, each working in its own **isolated git worktree**.

Git worktrees allow multiple branches to be checked out in different directories simultaneously without touching the primary working copy. This means:

- The developer's working copy is never modified by subagent activity
- Multiple subagents can write code in parallel without state corruption
- Each subagent operates in a clean, isolated environment

Mark Zuckerberg described this in the launch announcement:

> In testing, the system built six features for a game simultaneously with no collisions among the agents.

#### Architecture Diagram

```
┌──────────────────────────────────────────────────┐
│                   Muse Code                       │
│                                                   │
│  ┌─────────────┐    ┌──────────────────────────┐  │
│  │  Main Agent  │◄──►│  Async Background Agents │  │
│  │  Loop        │    │  (persistent per session) │  │
│  └──────┬───────┘    └──────────────────────────┘  │
│         │                                          │
│    ┌────┴────┐                                     │
│    │ Fan-Out │ (when task is large enough)          │
│    └────┬────┘                                     │
│    ┌────┼─────────┬──────────────┐                 │
│    ▼    ▼         ▼              ▼                 │
│  ┌───┐┌───┐   ┌───┐          ┌───┐                │
│  │SA1││SA2│   │SA3│   ...    │SAn│                │
│  └─┬─┘└─┬─┘   └─┬─┘          └─┬─┘                │
│    ▼    ▼       ▼              ▼                   │
│  ┌───┐┌───┐   ┌───┐          ┌───┐                │
│  │WT1││WT2│   │WT3│          │WTn│                │
│  └───┘└───┘   └───┘          └───┘                │
│  (isolated git worktrees)                          │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │         Local Event Log (append-only)         │  │
│  │  model calls · tool runs · approvals · edits  │  │
│  └──────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
       │
       ▼
 ┌───────────┐
 │Muse Spark │
 │   1.2     │ (inference via Meta Model API)
 └───────────┘
```

---

## Muse Spark 1.2 — The Underlying Model

Muse Spark 1.2 is a **proprietary, closed-weight** reasoning model from Meta, released alongside Muse Code on August 5, 2026. It is a coding-focused update to Muse Spark 1.1 (released July 9, 2026).

| Specification | Detail |
|---------------|--------|
| **Model ID** | `muse-spark-1.2` |
| **Contributor Model ID** | `muse-spark-1.2-contributor` |
| **Type** | Proprietary reasoning model |
| **Context Window** | 1,048,576 tokens (1M) |
| **Input** | Text, images, video, audio, PDF documents |
| **Output** | Text |
| **Open Weights** | No |
| **Predecessor** | Muse Spark 1.1 (July 9, 2026) |
| **Family** | Muse Spark (original → 1.1 → 1.2) |
| **Provider** | Meta Model API, OpenRouter |

### Co-Training with Muse Code

A key differentiator: Muse Spark 1.2 was **co-trained with the Muse Code harness itself**. The training process included:

- **Rejection-sampled harness trajectories**: training on successful agent runs
- **Recipe optimizations**: for goals, context compaction, and subagent coordination
- **Muse Code toolset integration**: to maximize harness compatibility

Meta's argument is that this co-training produces better tool use, fewer retries, and higher-quality output than wrapping a general-purpose model in a generic agent loop. This claim has not been independently verified by testing Muse Spark 1.2 inside rival harnesses.

### Long-Horizon Training

Muse Spark 1.2 was extensively trained on long-horizon coding tasks, including:

- Whole-repository generation
- Large end-to-end projects
- Auto-research workflows

The model uses **planning** to sequence work, **goal conditioning** to maintain direction, and **context compaction** to retain essential knowledge while managing its 1M token context window.

### Self-Improvement Loop

Meta used Muse Spark 1.1 to generate challenging coding environments and instruction-following templates. The model then graded candidate solutions on how well they met requirements, producing a scalable training dataset for 1.2. This self-improvement loop helped Muse Spark 1.2 follow complex instructions more precisely than its predecessor.

### Input Modalities

Muse Spark 1.2 accepts multiple input types:

- **Text**: standard prompt input
- **Images**: screenshots, diagrams, design mockups
- **Video**: in one demo, a user fed a fly-through video of a home, and Muse Code built a vacation rental marketing site from it
- **Audio**: voice-based input
- **PDF documents**: for reading and processing documentation

---

## Built-in Commands

Muse Code ships with several **bundled skills** — built-in commands that define its intended workflow:

### `/plan`

Creates an **approval-gated plan** before any code is written.

- Breaks down the task into structured steps
- Requires developer approval before execution begins
- Prevents the agent from taking unreviewed actions on the codebase

### `/grill`

**Stress-tests a plan** until it holds up.

- Challenges the proposed approach for weaknesses
- Identifies edge cases and failure modes
- Refines the plan before execution

### `/goal`

Drives the agent toward **successful completion of a stated objective**.

- Keeps the agent oriented during long sessions
- Maintains focus across multi-step tasks
- Tracks progress against the specified objective

This three-command workflow — **plan, attack the plan, then execute** — mirrors the pattern experienced users of competing tools converged on manually. Muse Code ships it as first-class functionality.

---

## Pricing

Muse Spark 1.2 is offered through the Meta Model API in **two pricing tiers**. There is no subscription — billing is purely pay-as-you-go based on token usage.

### Standard Tier

| Component | Price per 1M Tokens |
|-----------|---------------------|
| **Input tokens** | $1.25 |
| **Cached input tokens** | $0.15 |
| **Output tokens** | $4.25 |

- Prompts and completions are **not used** to improve Meta's products
- No long-context premium
- Rate limit: 3,000 requests/min, 4M tokens/min per team

### Contributor Tier

| Component | Price per 1M Tokens |
|-----------|---------------------|
| **Input tokens** | $0.10 |
| **Cached input tokens** | $0.002 |
| **Output tokens** | $0.20 |

- In exchange, Meta **may use your prompts and completions** to train future models
- Roughly **12.5x cheaper** on input and **21.25x cheaper** on output
- Rate limit: 60 requests/min (2% of Standard tier)
- Available in select countries only
- Model ID: `muse-spark-1.2-contributor`

> ⚠️ **Important**: any prompts containing proprietary code, client data, PII, or trade secrets should use the Standard tier. The Contributor tier sends your data into Meta's training pipeline.

### Rate Limits

| Parameter | Standard | Contributor |
|-----------|----------|-------------|
| Requests per minute | 3,000 | 60 |
| Tokens per minute | 4,000,000 | Not disclosed |

### Cost Estimation Example

For a single agent step with ~60,000 tokens of repository context in and ~3,000 tokens of plan-and-patch out:

| Scenario | Cost per Step | Cost per 1,000 Steps |
|----------|---------------|----------------------|
| Standard, cold prompt | ~$0.088 | ~$87.75 |
| Standard, cached context | ~$0.022 | ~$21.75 |
| Contributor, cold prompt | ~$0.007 | ~$6.60 |
| Contributor, cached context | ~$0.001 | ~$0.72 |

Web search grounding is billed separately at $2.50 per 1,000 queries on top of token costs.

---

## Benchmarks

Meta published benchmark results alongside the launch. All scores below are **Meta-reported** unless otherwise noted. Independent verification is limited at this early stage.

### Terminal-Bench 2.1

Terminal-Bench measures real-world agentic tasks in a shell environment: compiling code, setting up servers, system administration, security tasks, and data science pipelines.

| Model | Score |
|-------|-------|
| **Muse Spark 1.2 (Muse Code)** | **82.9%** |
| Muse Spark 1.1 | 80% |

*(Other model scores on Terminal-Bench 2.1 are from Meta's own harness, not each lab's reported figures.)*

### DeepSWE 1.1

Artificial Analysis' independent test of deep, agentic software-engineering work — planning and carrying out substantial coding tasks end to end.

| Model | Score |
|-------|-------|
| **Muse Spark 1.2** | **59.3%** |

### Meta Internal Coding Bench

Meta's own internal coding benchmark. No external methodology disclosure available.

Muse Spark 1.2 shows improvement over Muse Spark 1.1 on this internal measure.

### Artificial Analysis Intelligence Index

| Model | Index Score | Notes |
|-------|-------------|-------|
| Muse Spark 1.2 (xhigh) | 54 | Above median (32) |
| Muse Spark 1.1 (xhigh) | 51 | Previous version |
| Muse Spark (original) | 43 | First release |

The 3-point gain from 1.1 to 1.2 on the Intelligence Index is concentrated in agentic evaluations:
- GDPval-AA v2: +260 Elo (1371 → 1631)
- Terminal-Bench v2.1: +2 points (78% → 80%)
- τ³-Banking: +2 points (25% → 27%)

Minor regressions: SciCode (-2 points), Humanity's Last Exam (-1 point).

### Benchmark Caveats

- Meta published evaluation charts **without a full external methodology write-up** for all benchmarks
- Comparison figures for rival models come from **Meta's own harness**, not each lab's self-reported numbers
- No independent party has yet reproduced the Terminal-Bench 2.1 result outside of Muse Code
- The Artificial Analysis Intelligence Index uses its own methodology (v4.1)
- A model that only performs well inside its own harness is a product advantage, not necessarily a general capability
- Scores are specific to model, harness, effort, environment, date, and cost

---

## Muse Code vs Competitors

A comparison of the three major terminal-based AI coding agents as of August 2026:

| Feature | Muse Code | Claude Code | OpenAI Codex |
|---------|-----------|-------------|--------------|
| **Developer** | Meta (MSL) | Anthropic | OpenAI |
| **Status** | Early Beta | GA | GA |
| **Model** | Muse Spark 1.2 | Claude Opus 4.8 | GPT-5.5 |
| **Context Window** | 1M tokens | 1M tokens | 200K (1.05M long mode) |
| **Open Source** | No (proprietary) | No (proprietary) | Yes (Apache 2.0 CLI) |
| **Platform** | macOS, Linux | macOS, Linux, Windows | macOS, Linux, Windows (WSL) |
| **IDE Integration** | Terminal only | VS Code, JetBrains, Desktop | VS Code, Terminal |
| **Subagents** | Persistent async, git worktree isolation | Agent Teams, message-passing | Up to 8 parallel cloud sandboxes |
| **Crash Recovery** | Append-only event log, replay-exact | Session resumption | Cloud sandbox persistence |
| **Base Subscription** | None (pay-as-you-go) | $20/mo (Pro) | $20/mo (Plus) |
| **API Input Price** | $1.25/1M (standard) | ~$15/1M (Opus 4.8) | ~$2.50/1M (GPT-5.5) |
| **API Output Price** | $4.25/1M (standard) | ~$75/1M (Opus 4.8) | ~$10/1M (GPT-5.5) |
| **Budget Tier** | $0.10/$0.20 (contributor) | N/A | N/A |
| **Data Privacy** | Standard tier: not trained on; Contributor: trained on | Not trained on | Not trained on (API) |
| **Built-in Commands** | `/plan`, `/grill`, `/goal` | Hooks, subagent spawning | `/mode`, sandboxed execution |

> **Note**: Pricing and features change frequently. Verify current pricing on each provider's official documentation. Comparison data is assembled from multiple third-party analyses and may not reflect the latest updates from each vendor.

### Where Muse Code Stands Out

- **Pricing**: the Contributor tier at $0.10/$0.20 is significantly cheaper than any competing frontier-tier model, if you accept the data-sharing tradeoff
- **No subscription**: pure pay-as-you-go eliminates the $20/mo base cost that Claude Code and Codex require
- **Persistent subagents**: architectural bet that session-long agent persistence reduces redundant work
- **Crash recovery**: the event log design is a genuine architectural advantage for long-running tasks
- **Co-training**: model and harness trained together, potentially tighter integration

### Where Competitors Lead

- **Maturity**: Claude Code and Codex are generally available; Muse Code is in early beta
- **Ecosystem**: Claude Code has VS Code/JetBrains integration, lifecycle hooks, and a larger plugin ecosystem
- **Platform support**: competitors offer broader OS support including Windows
- **Track record**: Claude Code reportedly authors 326K+ GitHub commits per day (as of March 2026)
- **Independent benchmarks**: more extensive third-party evaluation data exists for competing tools
- **Open source**: OpenAI's Codex CLI is Apache 2.0; Meta's offering is fully proprietary

---

## Case Study: GPU Kernel Optimization

Meta tested Muse Code on a demanding long-horizon task: **iteratively optimizing GPU kernels on NVIDIA Hopper GPUs**.

### Setup

- **Task**: write, compile, profile, and progressively improve GPU kernel performance
- **Benchmarks**: KDA (Key-Dimension-Attention) and MLA (Multi-Latent-Attention) kernels
- **Duration**: 1,000+ tool calls over sessions lasting up to 24 hours
- **Constraint**: models were prohibited from importing third-party kernel libraries (e.g., FLA); they had to implement optimizations in Triton directly

### Results

- Muse Spark 1.2 achieved substantial improvements over the baseline implementation on both KDA and MLA kernels
- For KDA, the model paired a chunk-parallel preparation kernel with a sequential inter-chunk scan, combining fusion and tiling with KDA-specific optimizations such as re-centering the gated cumulative decay at the chunk midpoint
- For MLA, the model designed a two-kernel Triton pipeline combining kernel fusion and tiling with MLA-specific optimizations

### Interpretation

This case study demonstrates Muse Code's ability to sustain coherent, goal-directed work across an extended session. However:

- The case study is evidence of a **narrow capability** (GPU kernel optimization), not proof of reliability across arbitrary production repositories
- The baseline used (FLA Triton implementation) and specific benchmark parameters (batch size 1, 64 heads, sequence length 8192, latent dimension 512) are narrow testing conditions
- No independent party has reproduced these results

---

## Demos from Launch

Meta showcased several demonstrations during the Muse Code launch:

1. **Vacation Rental Website**: a user fed a fly-through video (MP4) of a home into the terminal, and Muse Code interpreted the footage to produce a visually rich vacation home marketing and booking page

2. **Six-Feature Game Build**: Muse Code simultaneously built six features for a single game using parallel subagents in isolated worktrees, with no collisions among the agents

3. **Photon Sphere / Embervault / Avo Lawn**: interactive demos showcased on Meta's research blog demonstrating various coding capabilities

---

## Known Limitations & Open Questions

Based on launch materials, independent analyses, and community discussion:

1. **Beta maturity**: this is an early beta — expect rough edges, missing features, and potential instability

2. **No Windows support**: Meta's announcement lists macOS and Linux only. Windows users may need WSL

3. **Proprietary and closed**: unlike Meta's Llama model family, Muse Code and Muse Spark 1.2 are fully proprietary. You cannot inspect the model weights or run inference locally

4. **Contributor tier privacy**: the discounted tier sends your code and prompts into Meta's training pipeline — a tradeoff that enterprises with proprietary codebases must consciously evaluate

5. **Parallel subagent conflict resolution**: in a 1,000+ tool-call run, one subagent may validate a change at hour two that conflicts with another subagent's change at hour five. Meta's launch materials do not address this failure mode in detail

6. **Benchmark harness dependency**: it is unclear whether Muse Spark 1.2's strong Terminal-Bench scores transfer to other coding harnesses. A model that only performs optimally inside its own harness is a product, not a general capability

7. **Regional availability**: the Contributor tier is available in select countries only, and Meta has not published a specific country list

8. **No IDE integration**: Muse Code is terminal-only — no VS Code, JetBrains, or desktop app integration at launch

9. **Rate limits on Contributor tier**: 60 requests per minute (vs. 3,000 for Standard) makes the Contributor tier impractical for production workloads

---

## Frequently Asked Questions

### What is Muse Code?

Muse Code is Meta's beta terminal coding agent. It plans changes, writes code, validates results, coordinates persistent background agents, and records work in a local event log. It is developed by Meta Superintelligence Labs (MSL).

### Is Muse Code the same as Muse Spark 1.2?

No. Muse Code is the **agent and runtime**; Muse Spark 1.2 is the **coding-focused model** that powers it. The agent manages goals, tools, state, worktrees, and the interaction loop. The model provides inference.

### Is Muse Code free?

No. Muse Code uses the Meta Model API, which is pay-as-you-go. Standard tier: $1.25/$4.25 per million input/output tokens. Contributor tier: $0.10/$0.20 (with data-sharing consent).

### Is Muse Code open source?

No. Unlike Meta's Llama models, both Muse Code and Muse Spark 1.2 are proprietary. The agent is a closed-source binary and the model weights are not available.

### Which operating systems are supported?

macOS and Linux. Meta has not announced native Windows support.

### Can Muse Code resume after a crash?

According to Meta, yes. The append-only local event log makes sessions replay-exact and restart-safe, allowing a run to continue from the last recorded step after interruption.

### Does Muse Code use subagents?

Yes. It uses persistent async background agents that stay active for an entire session, plus parallel subagents that can work in isolated git worktrees for large tasks.

### What is the Contributor tier?

A heavily discounted pricing tier ($0.10/$0.20 per million tokens) that allows Meta to use your prompts and completions to train future models. It has a 60 req/min rate limit and is available in select countries only.

### How does Muse Code compare to Claude Code?

See the [comparison table](#muse-code-vs-competitors) above. In short: Muse Code is cheaper and has an innovative persistent-agent architecture, but is less mature (beta vs. GA), has fewer platform integrations, and lacks the ecosystem breadth that Claude Code has built since its 2025 launch.

### Who leads Muse Code development?

Alexandr Wang, Meta's Chief AI Officer and head of Meta Superintelligence Labs. Wang joined Meta in mid-2025 as part of CEO Mark Zuckerberg's effort to rebuild the company's AI strategy.

---

## Official Sources

These are the primary sources from Meta:

| Source | Description | Link |
|--------|-------------|------|
| **Research Blog** | Full technical announcement with architecture, training details, and benchmarks | [research.meta.ai](https://research.meta.ai/blog/introducing-muse-code-and-muse-spark-1-2) |
| **Developer Blog** | Installation, model IDs, pricing, context window, cookbooks | [developer.meta.com](https://developer.meta.com/ai/resources/blog/build-with-muse-code/) |
| **Product Page** | Muse Code product access point | [developer.meta.com/ai/products/muse-code](https://developer.meta.com/ai/products/muse-code/) |
| **Model Page** | Muse Spark 1.2 model identity and access | [developer.meta.com/ai/models/muse-spark](https://developer.meta.com/ai/models/muse-spark/) |
| **Evaluation Methodology** | Meta's evaluation methodology report | [research.meta.ai/static/muse-spark-1-2-methodology](https://research.meta.ai/static/muse-spark-1-2-methodology) |

---

## Independent Analysis & Community Resources

Independent analyses, comparisons, and community resources:

### Independent Guides

- **[Muse Code — Independent Developer Guide](https://muse-code.dev/)** — source-traced reference separating confirmed facts, Meta's claims, evaluation limits, and community questions. Includes [Muse Spark 1.2 model analysis](https://muse-code.dev/muse-spark-1-2/), [architecture breakdown](https://muse-code.dev/how-muse-code-works/), and a [full source ledger](https://muse-code.dev/sources/).

### Benchmark Analysis

- [Artificial Analysis — Muse Spark 1.2](https://artificialanalysis.ai/models/muse-spark-1-2) — intelligence index, performance, and pricing analysis
- [BenchLM — Muse Spark 1.2](https://benchlm.ai/models/muse-spark-1-2) — benchmark coverage and leaderboard position
- [AI Release Tracker — Muse Spark 1.2](https://aireleasetracker.com/model/meta/muse-spark-1.2) — benchmark specs and release timeline

### Media Coverage

- [TechCrunch](https://techcrunch.com/2026/08/05/meta-launches-muse-code-an-ai-agent-for-large-code-bases/) — Meta launches Muse Code, an AI agent for large code bases
- [CNBC](https://www.cnbc.com/2026/08/05/meta-debuts-muse-code-to-take-on-anthropic-and-openai-.html) — Meta debuts first AI coding agent to take on Anthropic and OpenAI
- [VentureBeat](https://venturebeat.com/orchestration/meta-enters-the-ai-coding-wars-with-muse-spark-1-2-and-muse-code-with-persistent-async-background-agents) — Meta enters the AI coding wars
- [The Register](https://www.theregister.com/ai-and-ml/2026/08/06/meta-wants-to-get-inside-your-terminal-with-its-new-coding-agent/5283717) — Meta wants to get inside your terminal
- [Engadget](https://www.engadget.com/2231285/meta-introduces-muse-code-its-take-on-a-coding-agent/) — Meta introduces Muse Code
- [9to5Mac](https://9to5mac.com/2026/08/05/meta-launches-muse-code-ai-coding-agent-for-macos-and-linux/) — Meta launches Muse Code AI coding agent for macOS and Linux
- [MacRumors](https://www.macrumors.com/2026/08/05/meta-muse-code-for-mac/) — Meta's New Mac Coding Agent
- [Neowin](https://www.neowin.net/news/meta-takes-on-claude-code-and-codex-with-its-new-muse-code-agent-with-a-privacy-tradeoff/) — Meta takes on Claude Code and Codex
- [MarkTechPost](https://www.marktechpost.com/2026/08/05/meta-superintelligence-labs-releases-muse-code/) — Meta AI Releases Muse Code (Beta)

### Technical Deep Dives

- [Simon Willison](https://simonwillison.net/2026/Aug/5/muse-code-and-muse-spark-12/) — Muse Code and Muse Spark 1.2 analysis
- [Orcarouter](https://www.orcarouter.ai/blog/meta-muse-code-terminal-coding-agent) — Benchmarks, pricing, and the real catch
- [ExplainX](https://www.explainx.ai/blog/meta-muse-code-coding-agent-muse-spark-1-2-launch-august-2026) — Full launch analysis

### Pricing Resources

- [OpenRouter — Muse Spark 1.2](https://openrouter.ai/meta/muse-spark-1.2) — pricing, throughput, and API access

### API & Integration

- [Vercel AI Gateway — Muse Spark 1.2](https://vercel.com/ai-gateway/models/muse-spark-1.2-contributor) — API gateway access

### Community Tools

- [MuseCodeIcon](https://github.com/yourbossdonpely/musecligui) — free, MIT-licensed desktop GUI (Windows, macOS, Linux) for the Muse Code CLI, with coding threads, approvals, diffs and usage meters. Unofficial community project, not affiliated with Meta.

---

## Changelog

| Date | Update |
|------|--------|
| **2026-08-06** | Initial release of this repository |
| **2026-08-05** | Muse Code beta and Muse Spark 1.2 launched by Meta |

---

## Contributing

This is a community-maintained resource. Contributions are welcome.

### How to Contribute

1. **Fork** this repository
2. Create a **feature branch** (`git checkout -b add-new-resource`)
3. Add your changes — new resources, corrections, updated benchmarks, community findings
4. **Submit a Pull Request** with a clear description

### What We're Looking For

- Independent benchmark results and reproductions
- Community experience reports (both positive and negative)
- Updated pricing or feature changes
- New tools, integrations, or third-party resources
- Corrections to any factual errors
- Translations

### Guidelines

- Cite sources for factual claims
- Distinguish between Meta's claims and independently verified information
- Keep a neutral, informational tone
- Follow the existing structure and formatting

---

## Disclaimer

This repository is **not affiliated with, endorsed by, owned by, or operated by Meta**. It is an independent, community-maintained collection of publicly available information about Muse Code and Muse Spark 1.2.

Product names, trademarks, and registered trademarks are the property of their respective owners. All information is provided "as is" without warranty of any kind.

Pricing, features, benchmarks, and availability are subject to change. Always verify current information against official sources before making decisions.

---

<p align="center">
  <sub>Maintained by the community · <a href="https://muse-code.dev/">muse-code.dev</a> · Not affiliated with Meta</sub>
</p>
