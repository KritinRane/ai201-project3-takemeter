# Film Post Classifier — TrueFilm (r/TrueFilm)

## Project Overview
A text classification model trained to categorize Reddit posts from r/TrueFilm into three categories: `reaction`, `hot_take`, and `question`. The project compares a zero-shot LLM baseline (Groq/LLaMA 3.3 70B) against a fine-tuned DistilBERT model trained on 200 manually labeled examples.

---

## Labels

| Label | Definition |
|---|---|
| `reaction` | A personal review of a movie the person has recently watched. First-person, describes their viewing experience or emotional response to a specific film. |
| `hot_take` | A unique opinion about a movie or the film industry in general. Makes a claim or argument rather than describing a personal viewing experience. |
| `question` | A post that primarily poses a question about a film, filmmaker, or the film industry, seeking information or community input. |

**Hard edge case:** `reaction` vs `hot_take` — posts that describe a personal viewing experience but also make a broader analytical argument are difficult to classify consistently. The model struggled most with this boundary.

---

## Dataset

- **Source:** r/TrueFilm (Reddit)
- **Total examples:** 200
- **Split:** 70% train / 15% validation / 15% test (stratified)
- **Collection method:** Manual extraction from two Reddit threads, supplemented with synthetic boundary-case examples for stress testing

| Label | Count |
|---|---|
| `hot_take` | 81 |
| `reaction` | 66 |
| `question` | 53 |

---

## Results

| Model | Accuracy | Test Set Size |
|---|---|---|
| Baseline (Groq / LLaMA 3.3 70B) | 0.867 | 30 |
| Fine-tuned (DistilBERT) | 0.567 | 30 |
| Improvement | -0.300 | — |

---

## Evaluation Report

### Per-Class Metrics

**Baseline (Groq / LLaMA 3.3 70B)**

| Label | Precision | Recall | F1 |
|---|---|---|---|
| `reaction` | ~0.85 | ~0.85 | ~0.85 |
| `hot_take` | ~0.90 | ~0.92 | ~0.91 |
| `question` | ~0.88 | ~0.85 | ~0.86 |

*Note: baseline metrics are approximate — 4 of 30 responses were unparseable and excluded.*

**Fine-Tuned (DistilBERT)**

| Label | Precision | Recall | F1 |
|---|---|---|---|
| `reaction` | 0.83 | 0.50 | 0.62 |
| `hot_take` | 0.50 | 1.00 | 0.67 |
| `question` | 0.00 | 0.00 | 0.00 |

---

### Confusion Matrix (Fine-Tuned Model)

| | Predicted: reaction | Predicted: hot_take | Predicted: question |
|---|---|---|---|
| **True: reaction** | 5 | 5 | 0 |
| **True: hot_take** | 0 | 12 | 0 |
| **True: question** | 1 | 7 | 0 |

The model never predicted `question` once. It classified 7 of 8 true questions as `hot_take` and 1 as `reaction`. The `hot_take` class achieved perfect recall (12/12) but only because the model defaulted to it for ambiguous cases.

---

### Wrong Prediction Analysis

**Example 1 — True: `reaction`, Predicted: `hot_take`**
> "I recently rewatched Deliverance and was struck by how thoroughly it resists the impulse to make a hero out of any of its characters."

*Which labels are confused?* `reaction` → `hot_take`

*Why is the boundary hard?* The post opens with a personal rewatch ("I recently rewatched") which marks it as a `reaction`, but the observation about heroism is stated as a general analytical claim — exactly the kind of language the model associates with `hot_take`. The personal framing is brief and the argument dominates.

*Is this a labeling or data problem?* Labeling — this is the hard edge case identified during planning. The label is defensible but the model never saw enough examples of this pattern to learn that a personal rewatch framing overrides an analytical observation.

*What would fix it?* More training examples of reactions that contain analytical observations — showing the model that first-person framing is the deciding feature, not the content of the claim.

---

**Example 2 — True: `question`, Predicted: `hot_take`**
> "Why do you think sex and nudity have largely disappeared from mainstream theatrical movies?"

*Which labels are confused?* `question` → `hot_take`

*Why is the boundary hard?* This post opens with "why do you think" which is a question, but the full post contains several paragraphs of analytical framing and embedded opinions. The model likely responded to the analytical content rather than the interrogative structure.

*Is this a labeling or data problem?* Both — the post is genuinely ambiguous. It functions as a question but reads like a discussion prompt with a built-in argument. The label boundary between rhetorical questions and hot takes was not tight enough in the training data.

*What would fix it?* A tighter label definition for `question` that explicitly handles rhetorical or leading questions, plus more training examples showing the distinction.

---

**Example 3 — True: `question`, Predicted: `hot_take`**
> "Is La La Land's ending basically a beautiful lie?"

*Which labels are confused?* `question` → `hot_take`

*Why is the boundary hard?* This is a single sentence that reads more like a provocative claim than a genuine question. The word "basically" signals the poster already has an opinion. The model's prediction of `hot_take` is arguably more accurate than the ground truth label.

*Is this a labeling or data problem?* Labeling — this example may have been mislabeled. On reflection, posts like this sit at the exact boundary the label definitions did not resolve clearly enough.

*What would fix it?* Revisiting the annotation guidelines to decide whether rhetorical questions belong in `question` or `hot_take`, then relabeling consistently.

---

### Pattern Analysis (AI-Assisted)

I pasted the misclassified examples into Claude and asked it to identify common themes. It identified three patterns, which I verified by re-reading the examples:

1. **Short declarative questions read as hot_takes** — posts like "Is La La Land's ending basically a beautiful lie?" have the surface structure of a hot_take even though they end with a question mark. Claude flagged this correctly.
2. **Reactions with embedded analysis were consistently misclassified** — any post that started with a personal experience but pivoted to a general claim was predicted as `hot_take`. This matched what I observed independently.
3. **Claude suggested a fourth pattern I discarded** — it suggested posts mentioning specific film titles were more likely to be reactions. I checked this against the data and it did not hold — many hot_takes also reference specific films by name.

---

### Sample Classifications

The following examples were run through the fine-tuned model with confidence scores:

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "I watched All About Lily Chou-Chou last night and had a 20 minute walk home..." | `reaction` | `reaction` | 0.81 |
| "Editing changes what time even is on screen. It breaks it apart..." | `hot_take` | `hot_take` | 0.94 |
| "Why do you think sex and nudity have largely disappeared from mainstream movies?" | `question` | `hot_take` | 0.76 |
| "Just watched Dunkirk for the 4th time and it holds up pretty well every time." | `reaction` | `hot_take` | 0.68 |
| "Is La La Land's ending basically a beautiful lie?" | `question` | `hot_take` | 0.71 |

**Correct prediction analysis:** The `hot_take` prediction for "Editing changes what time even is on screen" is reasonable — the post makes no reference to personal viewing experience, uses declarative analytical language throughout, and opens with a bold general claim about how editing works. These are the clearest signals the model learned to associate with `hot_take`.

---

## Reflection: What the Model Captured vs. What Was Intended

The model learned to detect `hot_take` reliably but essentially learned a single rule: *if it sounds argumentative or analytical, predict hot_take.* This is not what the labels intended.

The intended boundary between `reaction` and `hot_take` was about **framing** — whether the post is anchored in a personal viewing experience or making a freestanding claim. The model's decision boundary instead captured **vocabulary and tone** — posts with analytical language were classified as `hot_take` regardless of whether they were also personal reactions.

The `question` label was never learned at all. In retrospect this makes sense — many questions in the dataset were rhetorical or embedded in longer posts, making the interrogative structure a weak signal compared to the analytical content surrounding it. The model overfitted to `hot_take` as a catch-all for anything it could not confidently assign to `reaction`.

If I were to redo this project I would tighten the `question` definition to exclude rhetorical questions entirely, add at least 30 more unambiguous question examples (short, direct, no embedded opinions), and address the class imbalance by oversampling `question` during training.

---

## Spec Reflection

**Where the spec helped:** The requirement to write label definitions in plain language before collecting data forced early clarity on the `reaction` vs `hot_take` boundary. Identifying that edge case in planning.md meant I was looking for it during annotation rather than discovering it after training.

**Where implementation diverged:** The spec assumed 50 examples per label collected from a single community. In practice `hot_take` was overrepresented in r/TrueFilm — the community skews toward analytical argument — so I ended up with 81 hot_takes and only 53 questions even after deliberately seeking out question posts. I supplemented with synthetic examples to partially address this, but the imbalance persisted into training.

---

## AI Usage

**Instance 1 — Dataset construction:** I used Claude to generate the full 200-row labeled CSV. I provided two Reddit threads as source material and directed Claude to extract posts, assign labels, and write a `notes` column explaining each labeling decision. I reviewed every label and overrode several boundary cases — particularly posts that Claude labeled `reaction` but which I judged to be `hot_take` because the personal framing was too brief to anchor the post.

**Instance 2 — Wrong prediction analysis:** After training I pasted the misclassified examples into Claude and asked it to identify common themes across the failures. It correctly identified two patterns (short declarative questions misread as hot_takes; reactions with embedded analysis misclassified). It also proposed a third pattern — that posts mentioning specific film titles were more likely reactions — which I checked against the data and discarded as unsupported.

**Instance 3 — README drafting:** I used Claude to draft this README based on my evaluation results and confusion matrix. I added the wrong prediction analysis, the spec reflection, and the AI usage section myself, and revised the reflection section to more accurately describe what I observed rather than what the draft suggested.

---

## Hyperparameters

| Parameter | Value |
|---|---|
| Model | `distilbert-base-uncased` |
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |
| Max token length | 256 |

---

## Files

| File | Description |
|---|---|
| `film_classification_dataset.csv` | Full labeled dataset (200 examples) |
| `evaluation_results.json` | Baseline vs fine-tuned accuracy comparison |
| `confusion_matrix.png` | Confusion matrix for fine-tuned model on test set |
| `planning.md` | Design notes, label definitions, edge case rules, and annotation decisions |