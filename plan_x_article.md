# X Article Plan: Data Curation for LLM Pre-Training (OLMo 3 + Autoresearch)

## Core Thesis
"If compute is finite, better data curation buys you capability per token — and now you can experiment with it yourself."

## Target Audience
ML practitioners, indie researchers, people interested in pre-training their own models on a budget.

---

## Article Structure (X Thread — 12-15 posts)

### Post 1 — Hook
What's the cost of getting data wrong vs. spending a week building the curation pipeline?

I read the entire OLMo 3 technical report (~117 pages) so you don't have to. Here's how Allen AI curated 5.9 TRILLION tokens — and how you can experiment with the same ideas on your laptop using @karpathy's autoresearch.

🧵👇

### Post 2 — The Central Question
The central question OLMo 3 answers:

If compute is finite, how does better data curation buy you capability per token?

Three pillars:
- What data to pre-train on?
- What capability do you get for a given data mix?
- What's the compute cost?

### Post 3 — The Pipeline (Visual Post)
Every token in OLMo 3's 6T corpus earned its place through this pipeline:

Raw Web Data → Text Extraction → Heuristic Filtering → Deduplication → Topic & Quality Classification → Constrained Mixing → Quality-Aware Upsampling

[Attach: simplified pipeline diagram from webapp]

Common Crawl alone contributes 76.1% of all pre-training tokens. But raw != ready.

### Post 4 — Deduplication: The 75% Reduction
They start with 38.7B documents and end with 9.7B. A 75% reduction.

Three stages:
1. Exact dedup (text hashes) → 38.7B → 12.8B docs
2. Fuzzy dedup (MinHash + Jaccard) → 12.8B → 9.8B docs
3. Substring dedup (suffix-array, ≥500 bytes) → removes 14% of text bytes

This isn't optional cleanup. This IS the data strategy.

### Post 5 — Topic & Quality Classification (The Novel Part)
Here's what I found most interesting as a data scientist.

They use WebOrganizer to classify into 24 topics (Science, Politics, etc.), then a fastText quality classifier trained on:
- Positive: OpenHermes-2.5, ELI5, UltraChat-200k, WildChat
- Negative: DCLM-RefinedWeb

Result: 480 disjoint subsets (24 topics × 20 quality tiers). Fine-grained control.

### Post 6 — Constrained Data Mixing (The Magic)
How do you decide *how much* of each topic?

Train a swarm of 200+ tiny 30M-param models on 3B tokens each with different data mixes. Fit regression models per task. Optimize.

But here's the catch — it's a *constrained* optimization:
- No source repeated more than 4-7x
- Everything sums to exactly 6T tokens
- Math has 200B tokens → max 1.4T (200B × 7) = capped at 23%

Without constraints, the optimizer would say "80% math!" and you'd repeat every math doc 24x.

### Post 7 — Conditional Mixing (Saving Compute)
They don't re-run the full swarm every time data changes.

Conditional mixing: freeze the existing optimized mix as a "virtual domain", only optimize over new data.

3 rounds:
- Round 1: 24 web categories (~120 models)
- Round 2: Freeze web, optimize 15 programming languages (~80 models)
- Round 3: Freeze everything, integrate 24 PDF topics (~125 models)

### Post 8 — Quality-Aware Upsampling (Not Binary Filtering)
Once mixing says "35% Science/Math", which docs within that topic?

Not binary keep/discard. A smooth exponential curve:
- Bottom 40%: discarded
- 40-60th percentile: appears 1x
- 60-80th: repeated 2-4x
- 80-95th: repeated 4-6x
- Top 5%: repeated 7x (the gold)

Like Spotify — you hear your #1 song more than your #50, but not on infinite repeat.

### Post 9 — Results: It Actually Works
Quality-aware upsampling vs flat filtering:
- Math Easy: 0.858-0.870 BPB (flat) → 0.740 BPB (quality-aware)

Average improvement: 0.056 BPB. Peak: 0.209 BPB on specific benchmarks.

The secret: maintain diversity (keep 60% of data) while intelligently varying repetition.

### Post 10 — The Three Training Stages
All this data feeds three stages:

1. **Pre-training** (5.9T tokens, 1024 H100s) — the base
2. **Mid-training** (100B tokens, 128 H100s) — steer toward code, math, QA
3. **Long-context** (50B tokens, 256 H100s) — 4.5M docs >32k tokens from science PDFs

Total wall-clock: ~56 days. Cost at $2/H100-hour: ~$2.75M.

### Post 11 — Now YOU Can Experiment (Autoresearch Bridge)
You obviously can't replicate this at scale. But you CAN experiment with the *ideas*.

@karpathy's autoresearch repo lets an AI agent autonomously run ML experiments overnight:
- Propose change → train 5 min → evaluate → keep or discard → repeat
- ~12 experiments/hour = ~100 while you sleep

The missing piece: using OLMo 3's data curation insights to guide WHAT the agent experiments with.

### Post 12 — The Experiment You Can Run Today
Here's what I built: a simple web app that:

1. Visualizes OLMo 3's data curation pipeline step-by-step
2. Lets you simulate topic/quality filtering on sample data
3. Generates autoresearch-compatible experiment configs

Fork the repo, plug in your data, let the agent iterate on data curation strategies overnight.

[Link to webapp + repo]

### Post 13 — Key Takeaway
The lesson from OLMo 3 isn't "get 1024 H100s."

It's that principled data curation — dedup, classify, constrained mixing, quality upsampling — is the highest-leverage work you can do before training.

Every token should earn its place. Now you have the tools to experiment with that idea.

---

## Writing Prompt for Structuring the Article

Use this prompt with Claude/GPT to refine the thread:

```
You are helping me write an X (Twitter) thread about data curation for LLM pre-training.

Context:
- I read the OLMo 3 technical report (Allen AI) which details how they curated 5.9T tokens
- I want to connect these insights to Karpathy's autoresearch repo for hands-on experimentation
- My audience is ML practitioners who want to understand and experiment with data curation

The thread should:
1. Open with a strong hook about the cost of bad data
2. Walk through the pipeline: filtering → dedup → classification → mixing → upsampling
3. Use concrete numbers from the report (38.7B→9.7B docs, 480 subsets, etc.)
4. Include the Spotify analogy for quality-aware upsampling
5. Bridge to autoresearch as the "experiment with these ideas yourself" tool
6. Close with a link to the companion web app
7. Be conversational but technically precise — no fluff
8. Each post should be self-contained enough to be screenshotted
9. Use specific numbers and results rather than vague claims
10. Total thread: 12-15 posts, each under 280 characters where possible (some can be longer with images)

Key data points to include:
- 76.1% of tokens from Common Crawl
- 75% document reduction from deduplication (38.7B → 9.7B)
- 480 disjoint subsets (24 topics × 20 quality tiers)
- Swarm of 200+ 30M-param proxy models
- Quality-aware upsampling: 0.740 vs 0.858-0.870 BPB on Math Easy
- 3 conditional mixing rounds (120 + 80 + 125 models)
- Max upsampling factor of 7x
- Total cost: ~$2.75M, 56 days on 1024 H100s

Tone: practitioner sharing lessons learned, not academic summary. Write like someone who just finished a 117-page report at 2am and is excited about what they found.
```

---

## Visual Assets Needed
1. Pipeline flow diagram (simplified from Figure 8 in the report)
2. Quality-aware upsampling curve (recreated from Figure 10)
3. Data composition pie chart (from Table 4)
4. Screenshot of the companion web app
5. Before/after dedup numbers infographic
