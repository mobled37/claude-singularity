# claude-research

A Claude Code plugin for systematic academic research assistance. Provides an 8-stage multi-agent pipeline that analyzes reference papers, identifies research gaps, downloads related papers, proposes novel methods from multiple perspectives, synthesizes them, and generates experiment plans.

## Installation

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated

### Install the plugin

```bash
claude install-plugin /path/to/claude-singularity
```

Or install directly from GitHub:

```bash
claude install-plugin https://github.com/mobled37/claude-singularity
```

### Verify installation

```bash
claude /claude-research:research-pipeline --help
```

## Usage

### 1. Prepare your papers

Create a directory with your reference PDFs. Mark the baseline paper with a `baseline_` prefix:

```
~/papers/my-project/
├── baseline_attention_is_all_you_need.pdf
├── linear_attention.pdf
├── flash_attention.pdf
└── state_space_models.pdf
```

### 2. Run the pipeline

```bash
claude "/claude-research:research-pipeline ~/papers/my-project --goal 'Improve transformer efficiency for long sequences'"
```

### 3. Pipeline stages

```
[Sequential: Analysis]
  Stage 1: Paper Review      -> Weakness analysis per paper
  Stage 2: Gap Search        -> Find & download compensatory papers
  Stage 3: Summarization     -> Structured summaries of all papers

[Parallel: Idea Proposal]     <- Stages 4, 5, 6 run simultaneously
  Stage 4: Idea Blending     -> Creative cross-paper synthesis
  Stage 5: Math Expert       -> Independent mathematical formulation
  Stage 6: Arch Expert       -> Independent architecture design

[Sequential: Convergence]
  Stage 7: Proposal Synthesis -> Unified method from 4+5+6
  Stage 8: Experiment Plan    -> Baselines & evaluation design
```

### 4. Output

All outputs are saved to `.omc/research-pipeline/{session-id}/`:

| File | Content |
|------|---------|
| `stage-1-paper-review.md` | Paper weaknesses |
| `stage-2-gap-search.md` | Compensatory papers found |
| `stage-3-summaries.md` | All paper summaries |
| `stage-4-idea-blending.md` | Creative blend proposals |
| `stage-5-math-formulation.md` | Mathematical method |
| `stage-6-architecture.md` | Architecture design |
| `stage-7-synthesis.md` | Unified method |
| `stage-8-experiments.md` | Experiment plan |
| `final-report.md` | Synthesized final report |

Downloaded related papers are saved to `{paper-directory}/related_papers/`.

### Options

```
--goal "<goal>"           # Required: research objective
--baseline "<filename>"   # Override baseline paper detection
--stages "1-3"            # Run only specific stages
--skip-search             # Skip Stage 2 (if you already have all papers)
--top-k 3                 # Number of blending strategies in Stage 4 (default: 5)
```

## Paper Directory Convention

- Place all reference PDFs in one directory
- **Baseline paper**: prefix filename with `baseline_` (e.g., `baseline_transformer.pdf`)
- If no `baseline_` prefix found, the pipeline will ask which paper is the baseline
- Stage 2 downloads related papers into `{paper-directory}/related_papers/`

## How it works

The pipeline uses specialized Claude agents at each stage:

- **Opus-tier agents** for deep analysis (Stages 1, 4, 5, 6, 7)
- **Sonnet-tier agents** for search and summarization (Stages 2, 3, 8)

Stages 4-6 run **in parallel** to produce independent, diverse proposals. Stage 7 then synthesizes these into a unified method that combines the best ideas from each perspective.

## License

MIT
