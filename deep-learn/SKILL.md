---
name: deep-learn
description: >
  Interactive learning skill for deeply understanding research papers, tech blog posts, and articles.
  Use this skill whenever a user wants to: read and understand a research paper or academic article,
  get a deep explanation of a technical blog post, explore concepts from an article interactively,
  understand algorithms, theory, or implementation details behind a paper, chat with an agent to
  clarify technical concepts step by step, build intuition for a topic through guided Q&A, or
  create a personal study/review document from a reading session.

  Trigger on phrases like: "help me understand this paper", "explain this article to me",
  "let's deep dive into X", "walk me through this", "I want to learn about this paper",
  "summarize this paper then let me ask questions", or whenever a user shares a URL, PDF, or
  article text with the clear intent to learn from it. Even if they just say "I found this
  interesting paper" — offer to run a deep-learn session.
---

# Deep Learn — Interactive Article Understanding Skill

This skill creates a structured, conversational learning session around a paper, blog post, or article. The session flows naturally through three phases, though the learner can revisit earlier phases at any time.

**Phase 1 → Summarize**: Produce a comprehensive structured overview, an architecture/algorithm diagram, and a sharp initial concept tree.
**Phase 2 → Deep Dive**: Interactive concept-by-concept exploration — the tree grows as the conversation unfolds.
**Phase 3 → Review Summary**: Generate a saved Markdown study document capturing the whole session.

---

## Phase 1: Content Ingestion & Summarization

### Step 1: Read the content

Handle input as follows:
- **URL** — Use WebFetch to retrieve the article.
- **PDF file path** — Use the Read tool on the provided path.
- **Local file (.md, .txt)** — Use the Read tool.
- **Pasted text** — The content is already in the conversation; proceed directly.

If the content is very long, read the abstract, introduction, method, experiments, and conclusion first.

### Step 2: Produce the structured summary

Write the summary using the format below. The architecture diagram is embedded as a subsection inside the summary — placed immediately after `⚙️ Algorithms & Approaches`, where the learner already has enough context to understand what it depicts.

Omit any section that genuinely doesn't apply to the article.

```
# 📄 [Title]
*[Type: Research Paper / Tech Blog / Article] · [Authors if available] · [Year if available]*

## 🎯 Problem Definition
What problem is being addressed? Why does it matter? What gap in prior work does it fill?

## ✨ Main Contributions & Innovations
- [Key contribution 1]
- [Key contribution 2]
- ...

## 🔗 Related Work
What prior work does this build on? How does this work position itself relative to prior approaches?

## ⚙️ Algorithms & Approaches
Core technical methods, architectures, models, or approaches. Explain clearly — use plain
language first, then add technical detail. Use pseudocode or equations if helpful.

### 🔬 Architecture Diagram
A Mermaid diagram capturing the core system, algorithm, or conceptual structure.
Choose the diagram type that best fits the article:
- System/pipeline → flowchart LR (data flow through components)
- Algorithm/process → flowchart TD (key steps in order)
- Taxonomy/comparison → graph TD (hierarchy or grouping)

Keep labels concise (3–6 words); annotate key edges. Goal: illuminating, not exhaustive.
Only omit if the article is purely conceptual with nothing structural to visualize.

Example (two-component pipeline):
[mermaid]
flowchart LR
    Q[Question] --> R[Retriever\nDPR]
    R -->|top-k passages| G[Generator\nBART]
    KB[(Wikipedia\nIndex)] --> R
    G --> A[Answer]
[/mermaid]

## 🏁 Main Conclusions
What did the authors conclude? Key takeaways.

## 📊 Experimental Results
Key experiments, benchmarks, metrics, and what they showed.

## ⚠️ Limitations
If the paper or article explicitly identifies limitations of the proposed approach —
methodological constraints, failure cases, scalability concerns, scope restrictions,
unresolved questions — summarize them here concisely.
Omit this section if no limitations are identified or discussed.
```

### Step 3: Initialize the concept tree

After the summary, display a **small, sharp concept tree** — 3 to 5 nodes maximum at the start.

The initial tree is not a table of contents. It captures the *essential tensions, design choices, and innovations* in the article — the things a smart reader would want to understand to truly grasp the work.

**How to choose good initial nodes:**
- Ask: what are the 3–5 ideas that, if understood deeply, would unlock everything else?
- Prefer nodes that represent *choices* or *tradeoffs* the authors made, not just topics
- Name nodes concisely (3–7 words) using the paper's own key terms
- Avoid vague names like "Related Work" or "Experiments" — those are sections, not concepts

**Good initial tree example for RAG:**
```
📚 Concept Tree
└── RAG [✓ summarized]
    ├── Parametric vs. non-parametric memory
    ├── Dense retrieval with learned embeddings
    ├── RAG-Sequence vs. RAG-Token
    └── Generator conditioned on retrieved docs
```

**Bad initial tree (too broad, just section headers):**
```
📚 Concept Tree
└── RAG [✓ summarized]
    ├── Introduction
    ├── Architecture
    ├── Experiments
    └── Conclusion
```

**Concept tree conventions:**
- `[✓]` — explored
- `[←]` — current focus (only used *while actively explaining* a topic, not after)
- `[ ]` — not yet explored

Show only the root and initial top-level nodes at the start. Sub-concepts are added as the learner explores them.

End Phase 1 with: *"Which of these would you like to dig into first? Or ask me anything about the article."*

---

## Phase 2: Interactive Deep Dive

This is the heart of the skill. The learner asks questions; you respond and grow the concept tree.

### Answering questions

When the learner asks about a concept, sub-concept, or background topic:

1. **Identify what they're asking about** and locate it in the current concept tree
2. **Answer at the right depth** — watch for cues about their background. Use concrete examples, analogies, or pseudocode. When explaining background topics, always reconnect back to the article.
3. **Grow the tree** — add sub-concepts discovered in the answer; mark the explored node

### How the tree grows

The tree unfolds during conversation, not all at once. When you explain a concept in depth:
- Mark the explored node `[✓]`
- Add 2–4 sub-concepts that emerged from your explanation as `[ ]` children
- Use `[←]` on the concept *while you are currently explaining it* (it becomes `[✓]` once you finish)

This makes the tree feel like a live map of the session — you can see what's been covered and what's still open.

**Example of tree growth across two turns:**

*Initial:*
```
📚 Concept Tree
└── RAG [✓ summarized]
    ├── Parametric vs. non-parametric memory [ ]
    ├── Dense retrieval with learned embeddings [ ]
    ├── RAG-Sequence vs. RAG-Token [ ]
    └── Generator conditioned on retrieved docs [ ]
```

*After learner asks about dense retrieval:*
```
📚 Concept Tree
└── RAG [✓ summarized]
    ├── Parametric vs. non-parametric memory [ ]
    ├── Dense retrieval with learned embeddings [✓]
    │   ├── Shared embedding space (question + passage) [✓]
    │   ├── Inner product search (MIPS) [✓]
    │   └── FAISS for fast approximate lookup [ ]
    ├── RAG-Sequence vs. RAG-Token [ ]
    └── Generator conditioned on retrieved docs [ ]
```

*While explaining FAISS (mid-response):*
```
    │   └── FAISS for fast approximate lookup [←]
```

*After finishing FAISS explanation:*
```
    │   └── FAISS for fast approximate lookup [✓]
    │       └── Approximate nearest-neighbor tradeoffs [ ]
```

### When to show the tree

- Always show the updated tree after a concept is newly explored or new sub-concepts are added
- Show the full tree when the learner seems to be navigating ("what else can we explore?")
- Skip it if the conversation is flowing through a single sub-concept chain — show only the relevant branch

### Gentle guidance

After answering, you may briefly suggest one natural next step: a sub-concept to go deeper on, or a sibling concept. Keep it short and optional — the learner leads.

### Diagrams in deep dives

If a concept would benefit from a diagram (e.g., the learner asks "how does FAISS work?"), generate a new focused Mermaid or ASCII diagram just for that concept. Keep it small — this is a teaching aid, not a reference document.

---

## Phase 3: Review Summary

Trigger when the learner says something like "summarize our chat", "give me a review summary", "create a study note", or "save this session".

### What to produce

Write a tutor-style Markdown document:

```markdown
# 📖 Study Notes: [Article Title]
*Session date: [today's date]*

## Original Summary
[The Phase 1 structured summary, with the architecture diagram embedded under
the Algorithms & Approaches section as it appeared in the session]

## Concepts Explored
[For each explored node in the tree, write a concise section:]

### [Concept Name]
[Key explanation points, important clarifications, "aha moments" from the conversation.
Write as a self-contained explanation the learner can use as a refresher — not raw Q&A.]

## Final Concept Tree
[The final state of the tree]

## Key Takeaways
- [3–5 most important insights from this session]

## Further Reading / Rabbit Holes
[Optional: related papers, concepts, or topics that came up and are worth exploring]
```

### Saving the file

Save to the workspace folder as `[article-title-slug]-study-notes.md`. Provide the file path and a clickable link so the learner can find it.

---

## General Guidance

**Maintain the concept tree in your working memory across the whole session** — it's the session backbone. Even if the learner jumps around, the tree tracks what's been covered.

**Adjust depth to the learner** — expert questions deserve precise technical answers; beginner questions deserve analogies and worked examples.

**Connect background knowledge back to the article** — whenever you explain a supporting concept (e.g., backprop, Bayes' theorem, FAISS), always end with: "So in this paper, they use this by…"

**The diagram is a teaching anchor** — reference it when answering questions. "Remember the diagram from earlier? The [component] is the part where…"
