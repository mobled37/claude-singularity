# Claude Singularity

**From papers to research proposals — automatically.**

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that runs an 8-stage multi-agent pipeline to analyze academic papers, identify research gaps, and generate novel method proposals with experiment plans.

```
Papers + Goal  ──→  Analysis  ──→  Parallel Ideation  ──→  Unified Proposal + Experiments
```

## Why

Turning a literature review into a concrete research direction typically takes weeks of reading, comparing weaknesses, brainstorming approaches, formalizing methods, and designing experiments. This plugin automates that entire pipeline with specialized AI agents — each operating at the right level of reasoning for its task.

The key insight: **idea blending, math formulation, and architecture design are run in parallel** as independent proposals, then synthesized. This produces more diverse and robust methods than a single sequential thought process.

## Quick Start

### Install

```bash
# From local clone
claude install-plugin /path/to/claude-singularity

# Or directly from GitHub
claude install-plugin https://github.com/mobled37/claude-singularity
```

### Run

```bash
claude "/claude-research:research-pipeline ~/papers/my-project --goal 'Improve transformer efficiency for long sequences'"
```

### Prepare Papers

Put your reference PDFs in a directory. Prefix the baseline paper with `baseline_`:

```
~/papers/my-project/
├── baseline_attention_is_all_you_need.pdf   # primary method to improve
├── linear_attention.pdf
├── flash_attention.pdf
└── state_space_models.pdf
```

If no `baseline_` prefix is found, the pipeline will ask you to identify one.

## Pipeline

```
                    ┌─────────────────────────────────────┐
                    │     Sequential: Analysis             │
                    │                                      │
                    │  Stage 1  Paper Review                │
                    │     │     Weakness analysis per paper │
                    │     ▼                                │
                    │  Stage 2  Gap Search                  │
                    │     │     Find & download related     │
                    │     │     papers from the web         │
                    │     ▼                                │
                    │  Stage 3  Summarization               │
                    │           Structured summaries        │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │     Parallel: Idea Proposal          │
                    │                                      │
                    │  ┌──────┐  ┌──────┐  ┌──────────┐   │
                    │  │  4   │  │  5   │  │    6     │   │
                    │  │ Idea │  │ Math │  │  Arch    │   │
                    │  │Blend │  │Expert│  │  Expert  │   │
                    │  └──┬───┘  └──┬───┘  └────┬─────┘   │
                    │     └─────────┼───────────┘         │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │     Sequential: Convergence          │
                    │                                      │
                    │  Stage 7  Proposal Synthesis          │
                    │     │     Unify 4+5+6 into one       │
                    │     ▼     coherent method             │
                    │  Stage 8  Experiment Plan             │
                    │           Baselines & evaluation      │
                    └─────────────────────────────────────┘
```

### Agents

| Stage | Agent | Model | Role |
|-------|-------|-------|------|
| 1 | Paper Reviewer | Opus | Deep weakness analysis per paper |
| 2 | Gap Scout | Sonnet | Web search + PDF download of related work |
| 3 | Paper Summarizer | Sonnet | Structured summaries & comparison tables |
| 4 | Idea Blender | Opus | Creative cross-paper technique combinations |
| 5 | Math Expert | Opus | Independent mathematical formulation |
| 6 | Arch Expert | Opus | Independent architecture design |
| 7 | Proposal Synthesizer | Opus | Merge parallel proposals into unified method |
| 8 | Experiment Planner | Opus | Baselines, datasets, metrics, ablation design |

## Output

All outputs are saved to `.omc/research-pipeline/{session-id}/`:

| File | Content |
|------|---------|
| `stage-1-paper-review.md` | Weaknesses with severity ratings and IDs |
| `stage-2-gap-search.md` | Compensatory papers found + download status |
| `stage-3-summaries.md` | Structured summaries + comparative table |
| `stage-4-idea-blending.md` | 3-5 ranked blending strategies |
| `stage-5-math-formulation.md` | Formal problem definition + proposed method |
| `stage-6-architecture.md` | Component breakdown + data flow design |
| `stage-7-synthesis.md` | Unified method (convergence + conflict resolution) |
| `stage-8-experiments.md` | Full experiment plan with compute estimates |
| `final-report.md` | Complete synthesized research proposal |

Downloaded related papers are saved to `{paper-directory}/related_papers/`.

## Options

```
--goal "<goal>"           # Required: research objective
--baseline "<filename>"   # Override baseline paper detection
--stages "1-3"            # Run only specific stages
--skip-search             # Skip Stage 2 (gap search)
--top-k 3                 # Number of blending strategies (default: 5)
```

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Claude model access (Opus recommended for best results; Sonnet works for Stages 2, 3)
- PDF files of reference papers

## Project Structure

```
claude-singularity/
├── .claude-plugin/
│   ├── plugin.json             # Plugin manifest
│   └── marketplace.json        # Marketplace metadata
├── agents/
│   ├── paper-reviewer.md       # Stage 1: Weakness analysis
│   ├── gap-scout.md            # Stage 2: Paper search + download
│   ├── paper-summarizer.md     # Stage 3: Structured summaries
│   ├── idea-blender.md         # Stage 4: Creative synthesis
│   ├── math-expert.md          # Stage 5: Mathematical formulation
│   ├── arch-expert.md          # Stage 6: Architecture design
│   ├── proposal-synthesizer.md # Stage 7: Proposal unification
│   └── experiment-planner.md   # Stage 8: Experiment design
├── skills/
│   └── research-pipeline/
│       └── skill.md            # Pipeline orchestration skill
├── CLAUDE.md                   # Project instructions
└── README.md
```

## How It Works

1. **Analysis** (Stages 1-3, sequential): Each paper is read in full via Claude's PDF support. The reviewer identifies weaknesses with severity ratings. The gap scout searches the web for papers addressing those weaknesses and downloads their PDFs. The summarizer produces structured comparisons across all papers.

2. **Parallel Ideation** (Stages 4-6, simultaneous): Three independent Opus agents each receive the same analysis outputs but propose solutions from different angles — creative blending, mathematical rigor, and practical architecture. They do not see each other's work.

3. **Convergence** (Stages 7-8, sequential): A synthesis agent identifies where the three proposals converge (high confidence), resolves conflicts, and merges the best elements into one coherent method. The experiment planner then designs a full evaluation against the baseline.

The pipeline state is persisted to `pipeline.json`, enabling resume from any interrupted stage.

## License

MIT
