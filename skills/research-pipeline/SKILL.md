---
name: research-pipeline
description: Research paper analysis pipeline with parallel idea proposal and synthesis
argument-hint: <paper-directory> --goal "<research goal>"
---

[RESEARCH PIPELINE - ITERATION {{ITERATION}}/{{MAX}}]

Your previous attempt did not output the completion promise. Continue working on the pipeline.

<Purpose>
Research Pipeline is a multi-agent pipeline that systematically analyzes academic papers, identifies research gaps, and proposes novel methods with experiment plans. It processes a directory of reference papers through 8 stages: 3 sequential analysis stages, 3 parallel idea-proposal stages, a synthesis stage, and an experiment planning stage. Modeled after the ralph execution pattern with persistent state tracking.
</Purpose>

<Use_When>
- User wants systematic research paper analysis with gap identification
- User says "research pipeline", "analyze papers", "paper review", "research assistance"
- User has a directory of reference papers and a research goal
- User wants to go from literature review to experiment design automatically
</Use_When>

<Do_Not_Use_When>
- User wants to review a single paper only -- just read and analyze it directly
- User wants a general literature survey without method proposal -- use sciomc instead
- User wants to write a paper -- this generates research direction, not paper text
</Do_Not_Use_When>

<Why_This_Exists>
Research ideation requires structured analysis that most researchers do manually over weeks: reading papers, finding weaknesses, searching for complementary work, blending ideas, formalizing methods, designing architectures, and planning experiments. This pipeline automates the full sequence with specialized agents at each stage. Stages 4-6 run in parallel because idea blending, mathematical formulation, and architecture design are independent creative proposals that benefit from diverse perspectives — the synthesizer then unifies them.
</Why_This_Exists>

<Pipeline_Overview>
```
[Sequential Phase: Analysis]
Stage 1: Paper Review ──────→ weaknesses per paper
Stage 2: Gap Search ────────→ papers compensating weaknesses
Stage 3: Summarization ─────→ structured summaries (all papers)
                │
                ▼
[Parallel Phase: Idea Proposal]  ← Stages 4, 5, 6 run SIMULTANEOUSLY
┌───────────────┼───────────────┐
│               │               │
▼               ▼               ▼
Stage 4:      Stage 5:      Stage 6:
Idea          Math          Architecture
Blender       Expert        Expert
│               │               │
└───────────────┼───────────────┘
                │
                ▼
[Sequential Phase: Convergence]
Stage 7: Proposal Synthesis ─→ unified method from 4+5+6
Stage 8: Experiment Plan ────→ baselines & evaluation design
```

Stages 1-3 are **sequential** (each depends on previous).
Stages 4-6 are **parallel** (all read from stages 1-3, independent of each other).
Stages 7-8 are **sequential** (7 synthesizes 4-6, 8 plans experiments from 7).
State is persisted to `.omc/research-pipeline/{session-id}/` for resume capability.
</Pipeline_Overview>

<Paper_Directory_Convention>
- All reference PDFs must be in the specified directory
- **Baseline paper**: prefix filename with `baseline_` (e.g., `baseline_attention_is_all_you_need.pdf`)
- Alternative: place the baseline in a `baseline/` subdirectory
- If no baseline is detected, **STOP and ask the user** which paper is the baseline before proceeding
- The baseline paper is the primary method the user wants to improve upon or compare against
</Paper_Directory_Convention>

<Execution_Policy>
- Stages 1-3 execute **strictly sequentially** -- each must complete before the next begins
- Stages 4, 5, 6 execute **in parallel** -- fire all three agents simultaneously after Stage 3 completes
- Stage 7 waits for ALL of stages 4, 5, 6 to complete before starting
- Stage 8 executes after Stage 7 completes
- Use Opus-tier agents for stages requiring deep analysis (1, 4, 5, 6, 7)
- Use Sonnet-tier agents for standard work (2, 3, 8)
- Read PDF files using the Read tool (Claude Code supports PDF reading)
- For the parallel phase, read agent definition files from `agents/` directory for detailed prompts
- Persist all outputs to the session directory
- If a stage fails, retry once with more context before reporting failure
</Execution_Policy>

<Steps>
0. **Initialization**:
   a. Parse arguments: extract `<paper-directory>` and `--goal "<goal>"` from `{{PROMPT}}`
   b. Verify the paper directory exists and contains PDF files
   c. List all PDFs and identify the baseline paper (look for `baseline_` prefix or `baseline/` subdirectory)
   d. If no baseline detected, STOP and ask the user which paper is the baseline
   e. Generate a session ID: `research-{YYYYMMDD}-{short-hash}`
   f. Create session directory: `.omc/research-pipeline/{session-id}/`
   g. Create `pipeline.json` state file with all 8 stages marked `pending`
   h. Save the list of papers and the identified baseline to `pipeline.json`

---

## Sequential Phase: Analysis (Stages 1-3)

1. **Stage 1 - Paper Review** (Identify weaknesses):
   a. Read EACH paper in the directory using the Read tool (PDF support)
   b. Read the agent prompt template from `agents/paper-reviewer.md`
   c. For each paper, analyze and document:
      - **Key contributions**: What novel ideas does this paper introduce?
      - **Methodology**: What approach/algorithm/model is used?
      - **Results**: What are the main experimental results?
      - **Weaknesses**: What are the limitations, assumptions, failure cases?
      - **Open questions**: What does the paper leave unanswered?
   d. For the **baseline paper**, provide extra-detailed analysis:
      - Specific quantitative limitations (e.g., "O(n²) complexity for sequence length")
      - Assumptions that could be relaxed
      - Scenarios where the method underperforms
      - Missing comparisons or evaluations
   e. Spawn an Opus-tier agent:
      ```
      Agent(prompt="[Read agents/paper-reviewer.md for full instructions]
      You are an expert research paper reviewer. Read and analyze each of these papers thoroughly.
      Papers: [list all PDF paths]
      Baseline paper: [baseline filename]
      Research goal: [goal]
      Output a structured review with weaknesses for each paper.
      Use weakness IDs (W-B1, W-B2 for baseline; W-{N}1 for others) for cross-referencing.",
      subagent_type="general-purpose", model="opus")
      ```
   f. Save output to `stage-1-paper-review.md`
   g. Update `pipeline.json`: stage 1 status = `complete`
   h. **Verification**: Confirm output contains weakness analysis for EVERY paper

2. **Stage 2 - Gap Search** (Find compensatory papers):
   a. Read Stage 1 output (`stage-1-paper-review.md`)
   b. Read the agent prompt template from `agents/gap-scout.md`
   c. Extract the top weaknesses across all papers (prioritize baseline weaknesses)
   d. For each major weakness, search for papers that address it:
      - Use WebSearch to find relevant papers (Semantic Scholar, arXiv, Google Scholar)
      - Cross-disciplinary search is encouraged (the area doesn't matter)
      - For each found paper, note: title, authors, year, venue, URL, and how it addresses the weakness
   e. Spawn a Sonnet-tier agent:
      ```
      Agent(prompt="[Read agents/gap-scout.md for full instructions]
      You are a research paper scout. Given these weaknesses from our literature review:
      [Include weaknesses from Stage 1]
      Research goal: [goal]
      Search the web for papers that address each weakness. Use WebSearch and WebFetch.
      Cross-disciplinary papers are welcome -- the area doesn't matter as long as the technique is relevant.",
      subagent_type="general-purpose", model="sonnet")
      ```
   f. Save output to `stage-2-gap-search.md`
   g. Update `pipeline.json`: stage 2 status = `complete`
   h. **Verification**: Confirm at least 1 compensatory paper found per major weakness

3. **Stage 3 - Paper Summarization** (Structured summaries):
   a. Read Stage 1 output (original paper analyses) and Stage 2 output (found papers)
   b. Read the agent prompt template from `agents/paper-summarizer.md`
   c. Create structured summaries for ALL papers (both original references and newly found ones)
   d. For each paper, produce:
      - **One-paragraph summary** (problem, approach, key result)
      - **Core technique** (the main algorithmic/methodological contribution)
      - **Key equations/formulations** (if applicable)
      - **Strengths and limitations** (from Stage 1 or new analysis)
      - **Relevance to goal** (how this paper relates to the research goal)
   e. Create a **comparative table** showing all papers side-by-side
   f. Spawn a Sonnet-tier agent:
      ```
      Agent(prompt="[Read agents/paper-summarizer.md for full instructions]
      You are an expert research summarizer. Create structured summaries of all these papers.
      Paper analyses: [Include Stage 1 output]
      Found papers: [Include Stage 2 output]
      Research goal: [goal]
      Also create a comparative table and complementarity map.",
      subagent_type="general-purpose", model="sonnet")
      ```
   g. Save output to `stage-3-summaries.md`
   h. Update `pipeline.json`: stage 3 status = `complete`

---

## Parallel Phase: Idea Proposal (Stages 4, 5, 6)

**CRITICAL: Fire all three agents SIMULTANEOUSLY after Stage 3 completes.**
Each agent receives the same inputs (stages 1-3 outputs) but works independently on a different aspect of the proposal.

4. **Stage 4 - Idea Blending** (Creative synthesis — runs in PARALLEL):
   a. Read ALL previous stage outputs (Stages 1-3)
   b. Read the agent prompt template from `agents/idea-blender.md`
   c. Identify complementary techniques across papers and propose 3-5 concrete **blending strategies**
   d. Each strategy includes: blend name, source papers, core idea, expected benefit, challenges, novelty
   e. Rank by expected impact and feasibility
   f. **Spawn simultaneously with Stages 5 and 6**:
      ```
      Agent(prompt="[Read agents/idea-blender.md for full instructions]
      You are a creative research ideation expert.
      Paper reviews: [Stage 1 output]
      Gap search results: [Stage 2 output]
      Paper summaries: [Stage 3 output]
      Research goal: [goal]
      Propose 3-5 ways to blend ideas from these papers into novel approaches.
      Think creatively -- cross-disciplinary combinations are encouraged.
      Rank strategies by impact and feasibility.",
      subagent_type="general-purpose", model="opus")
      ```
   g. Save output to `stage-4-idea-blending.md`
   h. Update `pipeline.json`: stage 4 status = `complete`

5. **Stage 5 - Mathematical Formulation** (Formal method — runs in PARALLEL):
   a. Read ALL previous stage outputs (Stages 1-3)
   b. Read the agent prompt template from `agents/math-expert.md`
   c. Based on the paper analyses and identified gaps, independently propose a mathematical formulation:
      - **Problem formulation**: Define the problem formally (input space, output space, objective)
      - **Proposed method**: Define the method with equations, using LaTeX notation
      - **Loss function / Objective**: Define the training objective or optimization goal
      - **Theoretical properties**: Analyze complexity, convergence, bounds if applicable
      - **Key assumptions**: State assumptions clearly
   d. NOTE: This agent works INDEPENDENTLY from Stage 4 — it proposes its own mathematical approach based on the paper analyses, not on blending strategies
   e. **Spawn simultaneously with Stages 4 and 6**:
      ```
      Agent(prompt="[Read agents/math-expert.md for full instructions]
      You are a mathematical expert in machine learning and optimization.
      Paper reviews with weaknesses: [Stage 1 output]
      Compensatory papers: [Stage 2 output]
      Paper summaries with key equations: [Stage 3 output]
      Research goal: [goal]
      Based on the identified weaknesses and available techniques, propose a rigorous
      mathematical formulation for a novel method. Use LaTeX notation.
      Define the problem, propose the method formally, specify the loss function,
      and analyze theoretical properties.
      You are working INDEPENDENTLY -- propose your own approach from the paper analyses.",
      subagent_type="general-purpose", model="opus")
      ```
   f. Save output to `stage-5-math-formulation.md`
   g. Update `pipeline.json`: stage 5 status = `complete`

6. **Stage 6 - Architecture Design** (System architecture — runs in PARALLEL):
   a. Read ALL previous stage outputs (Stages 1-3)
   b. Read the agent prompt template from `agents/arch-expert.md`
   c. Based on the paper analyses and identified gaps, independently propose a model/system architecture:
      - **High-level architecture diagram** (ASCII or structured description)
      - **Component breakdown**: Each module/component with its role
      - **Data flow**: How data moves through the system
      - **Training and inference procedures**
      - **Comparison with baseline architecture**
   d. NOTE: This agent works INDEPENDENTLY from Stages 4 and 5 — it proposes its own architectural approach
   e. **Spawn simultaneously with Stages 4 and 5**:
      ```
      Agent(prompt="[Read agents/arch-expert.md for full instructions]
      You are a senior ML systems architect.
      Paper reviews with weaknesses: [Stage 1 output]
      Compensatory papers: [Stage 2 output]
      Paper summaries with architectures: [Stage 3 output]
      Research goal: [goal]
      Based on the identified weaknesses and available techniques, propose a concrete
      model/system architecture for a novel method that addresses the key gaps.
      Include component breakdown, data flow, training procedure, and comparison with baseline.
      You are working INDEPENDENTLY -- propose your own architectural approach from the paper analyses.",
      subagent_type="general-purpose", model="opus")
      ```
   f. Save output to `stage-6-architecture.md`
   g. Update `pipeline.json`: stage 6 status = `complete`

**Parallel execution code pattern:**
```
// Fire ALL THREE simultaneously — do NOT wait between them
Agent(prompt="[Stage 4: Idea Blending]...", model="opus")   // Fire
Agent(prompt="[Stage 5: Math Expert]...", model="opus")      // Fire
Agent(prompt="[Stage 6: Arch Expert]...", model="opus")      // Fire
// Wait for ALL three to complete before proceeding to Stage 7
```

---

## Sequential Phase: Convergence (Stages 7-8)

7. **Stage 7 - Proposal Synthesis** (Unify parallel proposals):
   a. Read ALL outputs from the parallel phase: Stage 4 (idea blending), Stage 5 (math formulation), Stage 6 (architecture design)
   b. Also read Stages 1-3 outputs for reference
   c. **Synthesize the three independent proposals into a unified, coherent method**:
      - **Identify convergence**: Where do the three proposals agree? These are high-confidence directions.
      - **Resolve conflicts**: Where do they disagree? Choose the strongest approach with justification.
      - **Integrate strengths**: Take the best ideas from each:
        - From Stage 4 (Idea Blender): The most creative and novel combination strategies
        - From Stage 5 (Math Expert): The most rigorous mathematical formulation and theoretical properties
        - From Stage 6 (Arch Expert): The most practical and implementable architecture design
      - **Produce a unified proposal** that includes:
        - **Method name and one-paragraph description**
        - **Core idea** (from idea blending, refined)
        - **Mathematical formulation** (from math expert, refined to match the chosen architecture)
        - **Architecture design** (from arch expert, refined to implement the chosen math formulation)
        - **Novelty statement**: What is genuinely new about this unified approach
        - **Comparison with baseline**: How this improves on the baseline paper
   d. Spawn an Opus-tier agent:
      ```
      Agent(prompt="You are a senior research director who synthesizes diverse technical proposals
      into unified, coherent research directions.

      Three independent expert agents have analyzed the same papers and proposed different aspects
      of a novel method. Your job is to synthesize their outputs into ONE unified proposal.

      Idea Blending proposals (creative combinations): [Stage 4 output]
      Mathematical formulation (formal method): [Stage 5 output]
      Architecture design (system design): [Stage 6 output]

      Background:
      Paper reviews: [Stage 1 output summary]
      Research goal: [goal]

      Instructions:
      1. Identify where the three proposals CONVERGE (high-confidence directions)
      2. Resolve CONFLICTS between proposals (choose strongest with justification)
      3. Integrate the best from each:
         - Best creative ideas from the Idea Blender
         - Most rigorous math from the Math Expert
         - Most practical architecture from the Arch Expert
      4. Produce a UNIFIED proposal with: method name, core idea, math formulation,
         architecture, novelty statement, and baseline comparison.

      The unified proposal must be internally consistent -- the math must match the architecture,
      and both must implement the core idea.",
      subagent_type="general-purpose", model="opus")
      ```
   e. Save output to `stage-7-synthesis.md`
   f. Update `pipeline.json`: stage 7 status = `complete`
   g. **Verification**: Confirm the synthesis references all three input proposals and produces a coherent unified method

8. **Stage 8 - Experiment Planning** (Baseline experiments):
   a. Read Stage 7 output (unified proposal) and Stage 1 output (baseline paper analysis)
   b. Also read Stage 3 (all paper summaries) for baseline and comparison methods
   c. Read the agent prompt template from `agents/experiment-planner.md`
   d. Design a comprehensive experiment plan:
      - **Datasets**: Which datasets to use (from baseline paper + additional relevant ones)
      - **Baselines**: List of methods to compare against, including:
        - The baseline paper's method (primary comparison)
        - Other methods from reference papers
        - Relevant SOTA methods from Stage 2
        - Ablation variants of the proposed method
      - **Metrics**: Evaluation metrics (from baseline paper + additional ones)
      - **Main experiments**: Core comparison table specification
      - **Ablation studies**: What components to ablate and why
      - **Analysis experiments**: Visualization, qualitative analysis, failure case analysis
      - **Compute budget estimate**: Approximate GPU hours and resources needed
      - **Expected results**: Hypothesis for what the results should show
   e. Spawn an Opus-tier agent:
      ```
      Agent(prompt="[Read agents/experiment-planner.md for full instructions]
      You are an expert experiment designer for ML research.
      Unified method proposal: [Stage 7 output]
      Baseline paper analysis: [Stage 1 baseline section]
      All paper summaries: [Stage 3 output]
      Research goal: [goal]
      Design a comprehensive experiment plan with baselines, datasets, metrics, ablations,
      and expected results. The baseline paper's method is the PRIMARY comparison target.",
      subagent_type="general-purpose", model="opus")
      ```
   f. Save output to `stage-8-experiments.md`
   g. Update `pipeline.json`: stage 8 status = `complete`

---

9. **Final Report Generation**:
   a. Read all 8 stage outputs
   b. Generate a synthesized `final-report.md` containing:
      - Executive summary of the research direction
      - Literature analysis (from Stages 1-3)
      - Independent proposals overview (from Stages 4-6)
      - **Unified method proposal** (from Stage 7 — the core output)
      - Experiment plan (from Stage 8)
      - Full references list
   c. Update `pipeline.json`: overall status = `complete`
   d. Print a summary of what was produced to the user
</Steps>

<State_Management>
### pipeline.json Format

```json
{
  "id": "research-YYYYMMDD-xxxxxx",
  "goal": "The user's research goal",
  "paperDir": "/absolute/path/to/papers",
  "baselinePaper": "baseline_paper_name.pdf",
  "papers": [
    {"filename": "paper1.pdf", "isBaseline": false},
    {"filename": "baseline_method.pdf", "isBaseline": true}
  ],
  "status": "in_progress",
  "currentPhase": "analysis",
  "stages": [
    {
      "id": 1,
      "name": "paper-review",
      "phase": "analysis",
      "parallel": false,
      "description": "Review papers and identify weaknesses",
      "status": "pending",
      "outputFile": "stage-1-paper-review.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 2,
      "name": "gap-search",
      "phase": "analysis",
      "parallel": false,
      "description": "Search for papers compensating weaknesses",
      "status": "pending",
      "outputFile": "stage-2-gap-search.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 3,
      "name": "summarization",
      "phase": "analysis",
      "parallel": false,
      "description": "Create structured summaries of all papers",
      "status": "pending",
      "outputFile": "stage-3-summaries.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 4,
      "name": "idea-blending",
      "phase": "proposal",
      "parallel": true,
      "parallelGroup": "proposal",
      "description": "Propose creative idea combinations",
      "status": "pending",
      "outputFile": "stage-4-idea-blending.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 5,
      "name": "math-formulation",
      "phase": "proposal",
      "parallel": true,
      "parallelGroup": "proposal",
      "description": "Independent mathematical method formulation",
      "status": "pending",
      "outputFile": "stage-5-math-formulation.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 6,
      "name": "architecture",
      "phase": "proposal",
      "parallel": true,
      "parallelGroup": "proposal",
      "description": "Independent model/system architecture design",
      "status": "pending",
      "outputFile": "stage-6-architecture.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 7,
      "name": "synthesis",
      "phase": "convergence",
      "parallel": false,
      "description": "Synthesize parallel proposals into unified method",
      "status": "pending",
      "outputFile": "stage-7-synthesis.md",
      "startedAt": null,
      "completedAt": null
    },
    {
      "id": 8,
      "name": "experiments",
      "phase": "convergence",
      "parallel": false,
      "description": "Baseline experiments and evaluation plan",
      "status": "pending",
      "outputFile": "stage-8-experiments.md",
      "startedAt": null,
      "completedAt": null
    }
  ],
  "createdAt": "ISO timestamp",
  "updatedAt": "ISO timestamp"
}
```

### Resume Behavior

When the skill is re-invoked:
1. Check for existing `pipeline.json` in `.omc/research-pipeline/`
2. If found, read it and determine current phase:
   - If in analysis phase (1-3): resume from first incomplete sequential stage
   - If in proposal phase (4-6): check which parallel stages are incomplete, re-fire them
   - If in convergence phase (7-8): resume from first incomplete sequential stage
3. If not found, start fresh from initialization
</State_Management>

<Output_Formats>
### Stage 1 Output: Paper Review
```markdown
# Paper Review Report

## Research Goal
{goal}

## Baseline Paper: {name}
### Key Contributions
- ...
### Methodology
- ...
### Results
- ...
### Weaknesses (Detailed)
1. **[W-B1] {short name}**: {weakness} — {evidence/reasoning}
   - Severity: HIGH/MEDIUM/LOW
2. **[W-B2]**: ...
### Open Questions
- ...

## Paper: {name}
### Key Contributions
...
### Weaknesses
1. **[W-{id}1]**: ...
...

## Summary of Key Weaknesses Across All Papers
| Paper | ID | Weakness | Severity |
|-------|----|----------|----------|
| ... | ... | ... | ... |
```

### Stage 2 Output: Gap Search
```markdown
# Gap Search Report

## Weakness → Compensatory Paper Mapping

### [W-B1]: {weakness description}
**Found Papers:**
1. **{Title}** ({Authors}, {Year}, {Venue})
   - URL: {url}
   - Relevance: {how it addresses the weakness}
   - Key technique: {what technique it offers}

## Cross-Disciplinary Findings
{Papers from unexpected domains that offer relevant techniques}
```

### Stage 7 Output: Proposal Synthesis (KEY OUTPUT)
```markdown
# Unified Method Proposal

## Method Name: {name}

## One-Paragraph Summary
{What the method does, why it's novel, what it achieves}

## Convergence Analysis
### Where proposals agree:
- {convergence point 1}
- {convergence point 2}

### Conflicts resolved:
- {conflict}: chose {approach} because {justification}

## Core Idea
{From Idea Blender, refined with Math/Arch insights}

## Mathematical Formulation
{From Math Expert, refined to match chosen architecture}

## Architecture Design
{From Arch Expert, refined to implement chosen math}

## Novelty Statement
{What is genuinely new — not incremental combination}

## Comparison with Baseline
| Aspect | Baseline | Proposed | Improvement |
|--------|----------|----------|-------------|
| ... | ... | ... | ... |
```

### Stage 8 Output: Experiment Plan
```markdown
# Experiment Plan

## Datasets
| Dataset | Size | Task | Source |
|---------|------|------|--------|

## Baselines
| Method | Paper | Why Include |
|--------|-------|-------------|
| {Baseline paper method} | {cite} | PRIMARY comparison target |

## Main Experiments
| Experiment | Goal | Metrics |
|------------|------|---------|

## Ablation Studies
| Ablation | What's Removed | Tests |
|----------|----------------|-------|

## Expected Results & Compute Budget
```
</Output_Formats>

<Tool_Usage>
- Use `Read` tool to read PDF papers (Claude Code supports PDF reading natively)
- Use `Read` tool to read agent prompt templates from `agents/` directory
- Use `Agent` tool with `subagent_type="general-purpose"` for each stage's agent
- **For parallel phase (Stages 4-6)**: Fire THREE Agent calls in a SINGLE message block
- Use `Write` tool to save stage outputs and state
- Use `Edit` tool to update pipeline.json status
- Use `WebSearch` (via agent) for Stage 2 paper searching
- Use `WebFetch` (via agent) to access found paper abstracts/details
- Use `Glob` to list PDF files in the paper directory
</Tool_Usage>

<Examples>
<Good>
User: "/claude-research:research-pipeline ~/papers/transformers --goal 'Improve transformer efficiency for long sequences'"
Pipeline detects 5 PDFs including `baseline_attention_is_all_you_need.pdf`.
Stage 1 identifies O(n²) attention as key weakness. (sequential)
Stage 2 finds linear attention, sparse attention, and state-space model papers. (sequential)
Stage 3 summarizes all 5 original + 4 found papers. (sequential)
--- Parallel phase fires 3 agents simultaneously ---
Stage 4 (Idea Blender): proposes blending sparse attention with state-space models
Stage 5 (Math Expert): formalizes a sub-quadratic attention mechanism with provable bounds
Stage 6 (Arch Expert): designs a transformer variant with local-global attention hierarchy
--- Parallel complete, synthesis begins ---
Stage 7 (Synthesis): unifies into "SSM-Sparse Transformer" — takes the local-global hierarchy from Arch,
  the provable bounds from Math, and the SSM integration from Idea Blender
Stage 8: plans experiments on LRA benchmark comparing against vanilla transformer, Linformer, S4
Why good: Parallel proposals provide diverse perspectives, synthesis creates a stronger unified method.
</Good>

<Good>
Parallel diversity scenario:
Stage 4 proposes "attention + graph neural networks"
Stage 5 proposes "kernel-based O(n) attention formulation"
Stage 6 proposes "mixture-of-experts with adaptive routing"
Stage 7 finds convergence on "adaptive computation" theme, synthesizes into a
  kernel-attention method with MoE routing — mathematically grounded with practical architecture.
Why good: Independent proposals led to a richer synthesis than any single sequential path would.
</Good>

<Bad>
Running stages 4-6 sequentially: "Let me first blend ideas, then formalize the math based on the blend,
then design architecture based on the math."
Why bad: Stages 4-6 MUST run in parallel. They should be INDEPENDENT proposals.
Stage 5 should NOT read Stage 4's output. Stage 6 should NOT read Stage 5's output.
The synthesis (Stage 7) handles integration.
</Bad>

<Bad>
No baseline identified: Starting the pipeline without confirming which paper is the baseline.
Why bad: Must stop and ask if no baseline_* prefix is found. The baseline is critical for experiment design.
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- STOP if no PDF files found in the specified directory
- STOP if no baseline paper identified and user hasn't specified one
- STOP if paper reading fails (corrupted PDF, too large)
- STOP if Stage 2 web search finds zero relevant papers (unlikely but possible)
- If one parallel agent (4/5/6) fails but others succeed, proceed to synthesis with available outputs — note the gap
- Continue if a stage produces lower-quality output -- flag it and proceed
- If the same stage fails twice, report the issue and ask for user guidance
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Paper directory validated with PDFs found
- [ ] Baseline paper identified
- [ ] All 8 stages completed (status = complete in pipeline.json)
- [ ] Stages 4, 5, 6 were fired in parallel (not sequentially)
- [ ] Stage 7 synthesis references all three parallel proposals
- [ ] Stage 7 produces internally consistent unified method (math matches architecture)
- [ ] Each stage output file exists and is non-empty
- [ ] Final report generated with all sections
- [ ] User informed of completion with summary of proposed research direction
</Final_Checklist>

<Advanced>
## Configuration

The pipeline behavior can be customized via arguments:

```
--goal "<goal>"           # Required: research objective
--baseline "<filename>"   # Override baseline paper detection
--stages "1-3"           # Run only specific stages (for partial re-runs)
--skip-search            # Skip Stage 2 (if you already have all papers)
--top-k 3               # Number of blending strategies in Stage 4 (default: 5)
```

## Integration with OMC

If oh-my-claudecode is installed, the pipeline can use specialized agents:
- Stage 1: `oh-my-claudecode:scientist` (opus) for deep paper analysis
- Stage 2: `oh-my-claudecode:document-specialist` (sonnet) for web search
- Stage 4: `oh-my-claudecode:scientist` (opus) for creative synthesis
- Stage 5: `oh-my-claudecode:scientist` (opus) for mathematical formulation
- Stage 6: `oh-my-claudecode:architect` (opus) for architecture design
- Stage 7: `oh-my-claudecode:scientist` (opus) for proposal synthesis

Without OMC, all stages use `general-purpose` agents with detailed prompts.

## Why Parallel Stages 4-6?

Running idea blending, mathematical formulation, and architecture design in parallel provides:
1. **Diversity**: Each expert approaches the problem from a different angle without being anchored by the others
2. **Independence**: Math expert proposes what's theoretically elegant; Arch expert proposes what's practically implementable; Idea Blender proposes what's creatively novel
3. **Better synthesis**: Stage 7 has three independent perspectives to merge, avoiding the tunnel vision of a single sequential path
4. **Speed**: Three Opus agents running in parallel is ~3x faster than sequential

## Troubleshooting

**PDF reading fails?**
- Ensure PDFs are not password-protected
- Large PDFs may need page-range reading: Read(file, pages="1-20")
- If a paper is too large, the agent will summarize available pages

**One parallel agent failed?**
- Stage 7 can synthesize from 2/3 proposals — note the gap in the synthesis
- Consider re-running just the failed stage before synthesis

**Synthesis is incoherent?**
- Check if the three parallel proposals are too divergent
- Re-run Stage 7 with explicit instructions to prioritize one proposal as the backbone

**Stage 2 finds irrelevant papers?**
- Provide more specific research goal
- Cross-disciplinary results are intentional (area doesn't matter for technique relevance)
</Advanced>

Original task:
{{PROMPT}}
