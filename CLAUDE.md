# Claude Research Pipeline Plugin

A Claude Code plugin for systematic academic research assistance. Provides an 8-stage pipeline (3 sequential analysis, 3 parallel proposal, 2 convergence) that analyzes reference papers, identifies gaps, proposes novel methods from multiple perspectives, synthesizes them, and generates experiment plans.

## Plugin Structure

```
claude_singularity/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   └── research-pipeline/
│       └── SKILL.md          # Main sequential pipeline skill
├── agents/
│   ├── paper-reviewer.md       # Stage 1: Paper weakness analysis
│   ├── gap-scout.md            # Stage 2: Compensatory paper search
│   ├── paper-summarizer.md     # Stage 3: Structured summarization
│   ├── idea-blender.md         # Stage 4: Cross-paper idea synthesis (parallel)
│   ├── math-expert.md          # Stage 5: Mathematical formulation (parallel)
│   ├── arch-expert.md          # Stage 6: Architecture design (parallel)
│   ├── proposal-synthesizer.md # Stage 7: Unify parallel proposals
│   └── experiment-planner.md   # Stage 8: Baseline experiment generation
└── CLAUDE.md                 # This file
```

## Usage

```
/claude-research:research-pipeline <paper-directory> --goal "<research goal>"
```

## Pipeline Stages

```
[Sequential: Analysis]
1. Paper Review     → Weaknesses per paper
2. Gap Search       → Papers compensating weaknesses
3. Summarization    → Structured summaries

[Parallel: Idea Proposal]  ← Stages 4, 5, 6 fire SIMULTANEOUSLY
4. Idea Blending    → Creative synthesis proposals
5. Math Expert      → Independent math formulation
6. Arch Expert      → Independent architecture design

[Sequential: Convergence]
7. Proposal Synthesis → Unified method from 4+5+6
8. Experiment Plan    → Baselines & evaluation design
```

## Paper Directory Convention

- Place all reference PDFs in one directory
- **Baseline paper**: prefix filename with `baseline_` (e.g., `baseline_transformer.pdf`)
- If no `baseline_` prefix found, the pipeline will ask which paper is the baseline

## Output

All outputs are saved to `.omc/research-pipeline/{session-id}/`:
- `pipeline.json` - Pipeline state and progress
- `stage-1-paper-review.md` - Paper weaknesses
- `stage-2-gap-search.md` - Compensatory papers found
- `stage-3-summaries.md` - All paper summaries
- `stage-4-idea-blending.md` - Creative blend proposals (parallel)
- `stage-5-math-formulation.md` - Mathematical method (parallel)
- `stage-6-architecture.md` - Architecture design (parallel)
- `stage-7-synthesis.md` - Unified method from 4+5+6
- `stage-8-experiments.md` - Experiment plan with baselines
- `final-report.md` - Synthesized final report
