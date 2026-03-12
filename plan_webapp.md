# Web App Plan: OLMo 3 Data Curation Visualizer + Autoresearch Experimenter

## Overview
A single-page interactive web app that pairs with the X article. It lets readers:
1. **Visualize** OLMo 3's data curation pipeline step-by-step
2. **Simulate** data mixing and quality upsampling with sliders
3. **Connect** to autoresearch for hands-on experimentation

## Tech Stack
- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS + shadcn/ui components
- **Charts**: Recharts (lightweight, React-native)
- **Animations**: Framer Motion (step transitions)
- **Deployment**: Vercel (free tier, instant deploy)
- **No backend needed** — all simulation runs client-side

---

## Page Sections

### Section 1: Hero / Intro
- Title: "How OLMo 3 Curated 5.9 Trillion Tokens"
- Subtitle: "Visualize the pipeline. Experiment with the ideas. Train your own models."
- CTA: "Explore the Pipeline ↓" + "Try with Autoresearch →"

### Section 2: Interactive Pipeline Visualizer
A horizontal stepper/flow diagram. User clicks each step to expand details.

**Steps (left to right):**

#### Step 1: Raw Data Sources
- Visual: Stacked bar showing data sources with token counts
  - Common Crawl: 8.14T tokens (87.4%)
  - olmOCR PDFs: 972B (10.4%)
  - Stack-Edu: 137B (1.5%)
  - arXiv: 21.4B (0.2%)
  - FineMath: 34.1B (0.4%)
  - Wikipedia: 3.69B (0.04%)
- Interaction: Hover each source for description and doc count

#### Step 2: Text Extraction & Heuristic Filtering
- Visual: Funnel diagram
  - Start: 252.6B documents
  - After URL/spam filtering
  - After length filtering
  - After repetition removal
  - After language detection
  - End: 38.8B documents (84.6% reduction)
- Interaction: Toggle individual filters on/off to see impact on doc count
- Key insight callout: "This step alone removes 84.6% of documents"

#### Step 3: Deduplication
- Visual: Three-stage waterfall
  - 38.7B docs → Exact Dedup → 12.8B docs (67% removed)
  - 12.8B docs → Fuzzy Dedup (MinHash) → 9.8B docs (23% removed)
  - 9.8B docs → Substring Dedup (suffix-array) → 9.7B docs (14% text bytes removed)
- Interaction: Animated counter showing docs dropping at each stage
- Tooltip explaining each method (hash, MinHash+Jaccard, suffix-array)

#### Step 4: Topic & Quality Classification
- Visual: 2D grid heatmap (24 topics × 20 quality tiers = 480 cells)
  - X-axis: Quality tiers (vigintiles, 5-percentile buckets)
  - Y-axis: 24 WebOrganizer topics
  - Cell color intensity: token count in that bucket
- Interaction: Click any cell to see sample description of that topic+quality combo
- Sidebar: Explain positive/negative training examples for the quality classifier

#### Step 5: Constrained Data Mixing (Interactive!)
- Visual: Interactive mixing console
  - Sliders for major data sources (Web, Code, Math, PDFs, Wiki)
  - Real-time pie chart updating as sliders move
  - Constraint indicators turning red when violated:
    - "Max 7x upsampling" warning
    - "Must sum to 6T" constraint bar
    - "Source exhausted" alerts
- Interaction: User can try to find a better mix than OLMo 3's
- Show OLMo 3's actual mix as a "reference solution" overlay
- Display: predicted BPB score (simplified linear model from the report's regression approach)

#### Step 6: Quality-Aware Upsampling (Interactive!)
- Visual: The upsampling curve (recreated from Figure 10)
  - X-axis: Data quality percentile (0-100)
  - Y-axis: Upsampling factor (0-7x)
  - Pink curve: quality-aware upsampling
  - Gray dashed: flat filtering comparison
  - Red dashed: max upsampling (7x) line
- Interaction:
  - Drag control points to reshape the curve
  - See real-time token count calculation
  - Compare your curve vs OLMo 3's curve
  - Display: "Your mix produces X tokens. Target: 6T."
- Below: bar chart comparing BPB scores for flat vs quality-aware (from report data)

#### Step 7: Final Mix Output
- Visual: Sankey/alluvial diagram showing data flow from sources → pipeline → final mix
- Numbers: Final Dolma 3 Mix composition (Table 4 from report)
  - Common Crawl: 4.51T (76.1%)
  - olmOCR PDFs: 805B (13.6%)
  - Stack-Edu: 409B (6.89%)
  - arXiv: 50.8B (0.86%)
  - FineMath: 152B (2.56%)
  - Wikipedia: 2.51B (0.04%)

### Section 3: Training Stages Timeline
- Visual: Horizontal timeline with three stages
  - Pre-training: 5.9T tokens, 1024 H100s, ~35 days
  - Mid-training: 100B tokens, 128 H100s, ~1.5 days
  - Long-context: 50-100B tokens, 256 H100s, ~1 day
- Each stage expandable to show data composition (Tables 4, 5, 11 from report)
- Cost calculator: user can input their GPU count and see estimated time

### Section 4: "Try It Yourself" — Autoresearch Bridge
This is the actionable section that connects the OLMo 3 insights to hands-on experimentation.

#### 4a: What is Autoresearch?
- Brief explainer (2-3 sentences)
- Diagram of the autonomous loop: Propose → Train (5 min) → Evaluate → Keep/Discard → Repeat
- "~100 experiments while you sleep"

#### 4b: Data Curation Experiments You Can Run
A curated list of experiment ideas, each with:
- **Hypothesis** (one sentence)
- **What to modify** in autoresearch
- **How to measure** success (val_bpb)
- **Estimated runs** needed

**Experiment Ideas:**

1. **Quality filtering threshold sweep**
   - Hypothesis: "Aggressive quality filtering on ClimbMix data improves val_bpb"
   - Modify: `prepare.py` data loading to add quality score filtering
   - Metric: Compare val_bpb across thresholds (top 50%, 25%, 10%)

2. **Deduplication impact**
   - Hypothesis: "Even simple exact-dedup on training shards improves token efficiency"
   - Modify: Add hash-based dedup to the data loader
   - Metric: val_bpb at same token count

3. **Topic rebalancing**
   - Hypothesis: "Upsampling math/code content improves BPB"
   - Modify: Adjust sampling weights per data domain in the loader
   - Metric: val_bpb and per-domain loss

4. **Upsampling curve shape**
   - Hypothesis: "Quality-aware upsampling beats flat filtering at small scale"
   - Modify: Implement a simple quality scorer + exponential upsampling curve
   - Metric: val_bpb with same total tokens but different repetition strategies

5. **Sequence packing strategies**
   - Hypothesis: "BOS-aligned packing with quality sorting improves learning"
   - Modify: Sort documents by quality before packing in data loader
   - Metric: val_bpb convergence speed

#### 4c: Quick Start
```bash
# Clone and setup autoresearch
git clone https://github.com/karpathy/autoresearch
cd autoresearch
uv sync
uv run prepare.py

# Run baseline
uv run train.py

# Start autonomous experimentation
# (open Claude Code / Cursor and reference program.md)
```

#### 4d: Generate Experiment Config (Interactive)
- User selects which experiment type from the list above
- App generates a modified `program.md` snippet with:
  - Clear hypothesis
  - Specific changes to make in `train.py` or data loading
  - Success criteria
- Copy-to-clipboard button

### Section 5: Key Takeaways
- "Every token should earn its place"
- Three lessons:
  1. Dedup aggressively (75% of raw data is redundant)
  2. Classify before mixing (480 buckets > random sampling)
  3. Upsample by quality, don't filter binary (keep diversity, vary repetition)
- Link to full OLMo 3 report
- Link to X thread
- Link to autoresearch repo

---

## File Structure

```
webapp/
├── app/
│   ├── layout.tsx          # Root layout with fonts, metadata
│   ├── page.tsx            # Main page composing all sections
│   └── globals.css         # Tailwind base + custom styles
├── components/
│   ├── hero.tsx            # Hero section
│   ├── pipeline/
│   │   ├── pipeline-stepper.tsx     # Main stepper container
│   │   ├── raw-data-step.tsx        # Step 1: data sources bar chart
│   │   ├── filtering-step.tsx       # Step 2: funnel diagram
│   │   ├── dedup-step.tsx           # Step 3: waterfall
│   │   ├── classification-step.tsx  # Step 4: 2D heatmap
│   │   ├── mixing-step.tsx          # Step 5: interactive sliders
│   │   ├── upsampling-step.tsx      # Step 6: interactive curve
│   │   └── final-mix-step.tsx       # Step 7: sankey diagram
│   ├── training-timeline.tsx        # Section 3: timeline
│   ├── autoresearch/
│   │   ├── explainer.tsx            # What is autoresearch
│   │   ├── experiment-cards.tsx     # Experiment ideas
│   │   └── config-generator.tsx     # Interactive config generator
│   ├── takeaways.tsx                # Key takeaways
│   └── ui/                          # shadcn components (slider, card, etc.)
├── lib/
│   ├── data.ts             # All hardcoded data from OLMo 3 report
│   ├── mixing-model.ts     # Simplified regression model for BPB prediction
│   └── upsampling.ts       # Upsampling curve math (parametric curve fitting)
├── public/
│   └── og-image.png        # Social share image
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── next.config.js
```

---

## Data Constants (from Report)

```typescript
// lib/data.ts

export const DATA_SOURCES = [
  { name: "Common Crawl", type: "Web pages", poolTokens: "8.14T", poolDocs: "9.67B", mixTokens: "4.51T", mixPercent: 76.1 },
  { name: "olmOCR PDFs", type: "Academic docs", poolTokens: "972B", poolDocs: "101M", mixTokens: "805B", mixPercent: 13.6 },
  { name: "Stack-Edu", type: "GitHub code", poolTokens: "137B", poolDocs: "167M", mixTokens: "409B", mixPercent: 6.89 },
  { name: "arXiv", type: "Papers (LaTeX)", poolTokens: "21.4B", poolDocs: "3.95M", mixTokens: "50.8B", mixPercent: 0.86 },
  { name: "FineMath 3+", type: "Math web pages", poolTokens: "34.1B", poolDocs: "21.4M", mixTokens: "152B", mixPercent: 2.56 },
  { name: "Wikipedia", type: "Encyclopedic", poolTokens: "3.69B", poolDocs: "6.67M", mixTokens: "2.51B", mixPercent: 0.04 },
];

export const DEDUP_STAGES = [
  { name: "Raw corpus", docs: 38.7e9, method: "" },
  { name: "Exact dedup", docs: 12.8e9, method: "Document text hashes", reduction: "67%" },
  { name: "Fuzzy dedup", docs: 9.8e9, method: "MinHash + Jaccard similarity", reduction: "23%" },
  { name: "Substring dedup", docs: 9.7e9, method: "Fuzzy suffix-array (≥500 bytes)", reduction: "14% text bytes" },
];

export const TOPIC_CATEGORIES = 24; // WebOrganizer categories
export const QUALITY_TIERS = 20;    // Vigintile buckets
export const TOTAL_SUBSETS = 480;   // 24 × 20

export const TRAINING_STAGES = [
  { name: "Pre-training", tokens: "5.9T", gpus: 1024, days: 35, data: "Dolma 3 Mix" },
  { name: "Mid-training", tokens: "100B", gpus: 128, days: 1.5, data: "Dolma 3 Dolmino Mix" },
  { name: "Long-context", tokens: "50-100B", gpus: 256, days: 1, data: "Dolma 3 Longmino Mix" },
];

export const MAX_UPSAMPLING = 7;
export const TARGET_TOKENS = 6e12; // 6T
```

---

## Implementation Phases

### Phase 1: Static Scaffold (MVP)
- [ ] Next.js project setup with Tailwind + shadcn
- [ ] Hero section
- [ ] Pipeline stepper with static content for all 7 steps
- [ ] Training timeline (static)
- [ ] Key takeaways footer
- **Goal**: Readable companion to the X thread. No interactivity yet.

### Phase 2: Interactive Pipeline
- [ ] Step 2 (Filtering): Toggle filters on/off
- [ ] Step 3 (Dedup): Animated counters
- [ ] Step 5 (Mixing): Working sliders with constraint validation
- [ ] Step 6 (Upsampling): Draggable curve with real-time token calculation
- **Goal**: Users can explore and build intuition about each decision.

### Phase 3: Autoresearch Integration
- [ ] Autoresearch explainer section
- [ ] Experiment idea cards
- [ ] Config generator (generates modified program.md snippets)
- [ ] Quick-start copy-paste commands
- **Goal**: Bridge from "understanding OLMo 3" to "trying it yourself."

### Phase 4: Polish & Deploy
- [ ] Responsive design (mobile-friendly for X readers)
- [ ] OG image for social sharing
- [ ] Performance optimization (lazy-load charts)
- [ ] Vercel deployment
- [ ] Link in X thread

---

## Key Design Decisions

1. **Client-side only**: No server needed. All data is hardcoded from the report. Simulation math (mixing constraints, upsampling curves) runs in the browser.

2. **Progressive disclosure**: Pipeline steps collapse/expand. Users don't see all complexity at once.

3. **Numbers from the report**: Every visualization uses actual numbers from OLMo 3 Tables 4, 5, 11. No made-up data.

4. **Autoresearch-compatible output**: The config generator produces snippets that work directly with autoresearch's `program.md` format.

5. **Mobile-first**: X readers will tap through on mobile. All interactions must work on touch devices. Sliders and drag interactions use touch-friendly sizes.
