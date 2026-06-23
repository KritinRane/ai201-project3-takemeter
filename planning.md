# Film Post Classifier — Planning

## Community
r/TrueFilm — an in-depth film discussion subreddit. Chose this community because posts fall into clearly distinct categories (personal reactions, analytical opinions, and questions) making it a good fit for a classification task.

---

## Labels

**1. reaction** — a personal review or response to a movie the person has recently watched. First-person, anchored in a specific viewing experience.

**2. hot_take** — a unique or analytical opinion about a movie or the film industry in general. Makes a freestanding claim or argument, not anchored in a personal viewing experience.

**3. question** — a post that primarily poses a question about a film, filmmaker, or the film industry, seeking information or community input.

---

## Hard Edge Cases

**reaction vs hot_take** — the hardest boundary. A post can start with a personal viewing experience ("I rewatched Deliverance recently") but then pivot to a general analytical claim. The rule: if the post is primarily anchored in a personal experience, label it `reaction` even if it contains analysis. If the personal framing is absent or minimal, label it `hot_take`.

**rhetorical questions vs hot_takes** — posts like "Is La La Land's ending basically a beautiful lie?" are structured as questions but function as arguments. Rule: if the post is a single sentence with an embedded opinion, label it `hot_take`. If it genuinely invites community input, label it `question`.

---

## Data Collection Plan

- Source: r/TrueFilm Reddit threads
- Target: 50 examples per label (150 minimum, 200 total)
- Method: Manual extraction from threads, supplemented with synthetic boundary-case examples for stress testing
- If a label is underrepresented after collection, generate synthetic examples to balance

**Actual distribution collected:**
- `hot_take`: 81
- `reaction`: 66
- `question`: 53

`hot_take` was overrepresented because r/TrueFilm skews toward analytical argument. The imbalance was noted but not fully corrected.

---

## Evaluation Metrics

- **Primary metric:** overall accuracy on the held-out test set
- **Secondary metrics:** per-class F1, precision, recall via `classification_report`
- **Verification:** if Groq pre-labels match my own judgment on a sample of examples, the labels are considered consistent
- F1 is the most important per-class metric because it balances precision and recall — useful when classes are imbalanced

---

## AI Tool Plan

**1. Label stress-testing:** Ask an LLM to generate 5–10 posts at the boundary between `reaction` and `hot_take` to expose weaknesses in the label definitions before annotation.

**2. Annotation assistance:** Use Groq (LLaMA 3.3 70B) to pre-label a batch of examples, then review and correct the labels manually. Groq's labels are a starting point, not ground truth.

**3. Pattern analysis:** After training, paste misclassified examples into Claude and ask it to identify common themes across failures. Verify any patterns against the data before including them in the analysis.

---

## Hard Annotation Decisions

- Posts that open with "I rewatched X" but spend most of their length making analytical arguments → labeled `reaction` if the rewatch framing is present, `hot_take` if it is absent
- Single-sentence rhetorical questions with embedded opinions → labeled `hot_take`
- Posts that ask a question but include several paragraphs of analytical setup → labeled `question` if the question is genuine, `hot_take` if the question is rhetorical
- Synthetic boundary-case examples (rows 51–60 and 168–200 in the dataset) were flagged in the `source` column as `synthetic` to distinguish them from real Reddit posts