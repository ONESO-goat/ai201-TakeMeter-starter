

# planning.md

## Community Choice

I am using **r/LetsTalkMusic**, and **r/AskReddit** as my primary sources, filtering post by the title **"Hot Take"**.

These communites are strong fits because both contain a high volume of **opinions, critiques, and debates** that naturally range from emotional reactions to structured arguments. This variation is necessary for training a model that figures the differences between emotional claims and evidence based reasoning. It also includes both casual listeners and more analytical users, which can increase label diversity and edge cases.

---

## Labels

The task is to classify music or sport related takes based on the **level of reasoning and evidence supporting the claim**, not whether the opinion is positive or negative.

### 1. emotional

A post is labeled **emotional** if it primarily expresses feelings, insults, hype, or strong subjective reactions **without structured reasoning or evidence**.

* Example 1: “This player is a complete bum”
* Example 2: “Rock is literally the greatest genre ever made.”

**Note:** Even if the claim is extreme, it remains emotional unless it includes verifiable reasoning or structured support.

---

### 2. unsupported

A post is labeled **unsupported** if it makes a factual or semi-factual claim **without evidence, reasoning, or explanation**, even if the tone is calm.

* Example 1: “Rock music is for depressed people.”
* Example 2: “Superhero movies killed cinema”

**Note:** Unlike emotional posts, unsupported posts may sound neutral but still lack justification.

---

### 3. weakly_supported

A post is labeled **weakly_supported** if it includes some reasoning or evidence, but it is **limited, surface-level, or not sufficient to fully justify the claim**.

* Example 1: “Travis Scott is popular because of his production style and festival performances.”
* Example 2: “This band is bad because they haven’t had a hit in years.”

**Note:** Evidence exists, but it is either incomplete, oversimplified, or not well developed.

---

### 4. well_supported

A post is labeled **well_supported** if it includes **clear reasoning, multiple supporting points, or factual/contextual evidence that directly supports the claim**.

* Example 1: “Hip hop evolved through decades of cultural expression, lyrical innovation, and regional influence, shaping global music trends across multiple eras.”
* Example 2: “This artist charted on the Billboard Top 40 for multiple consecutive releases, showing sustained commercial success and audience reach.”

**Note:** The reasoning should be sufficient that most readers would agree the conclusion follows logically from the evidence. The classifier won't be able to verify the claim itself, so even if the claim is objectivly untrue it'll still fall in the 'well_sopported' label.

---

### Label Map

```python
LABEL_MAP = {
    "unsupported": {
        "id": 0,
        "reason": "Makes a claim without evidence or reasoning"
    },
    "weakly_supported": {
        "id": 1,
        "reason": "Some evidence exists but is incomplete or insufficient"
    },
    "well_supported": {
        "id": 2,
        "reason": "Clear reasoning and evidence sufficiently support the claim"
    },
    "emotional": {
        "id": 3,
        "reason": "Primarily expresses emotion, opinion, or sentiment without structured argument"
    }
}
```

---

## Hard Edge Cases

### Ambiguous Boundary Case

A key ambiguous case occurs when a post is both **emotionally charged and includes factual claims**.

* Example:
  “This artist is trash lol. They got booed off stage last year and only had one hit song that died fast. Nobody should pay to see them.”

**Issue:**
The post includes emotional language (“trash lol”) but also includes factual sounding claims that provide partial evidence.

**Handling Strategy:**

* If emotional framing dominates → label **emotional**
* If structured evidence is the main driver → label **weakly_supported** or **well_supported**


This hybrid handling is used only when separation is not reliable.

---

## Data Collection Plan

### Data Sources

* r/LetsTalkMusic, r/rap and artist specific subreddits
* r/AskReddit filtering by post under "Hot Takes"
* YouTube and TikTok comment sections on music-related content
* Personal domain knowledge of music discourse

### Dataset Size

* Target: **200+ labeled examples total**
* Goal distribution: 50 examples per label (balanced dataset)

### Balancing Strategy

If any label becomes underrepresented after 200 examples, I will correct imbalance using a round-robin collection method:

emotional → unsupported → weakly_supported → well_supported (repeat cycle)

---

## Evaluation Metrics

Accuracy alone is insufficient because labels have different levels of partial correctness.

I will use:

* **Precision (per class):** Measures how often predicted labels are correct
* **Recall (per class):** Measures how many true examples of a label are captured
* **F1-score (per class):** Balances precision and recall, especially important for minority or ambiguous classes
* **Confusion Matrix:** Shows systematic confusion between labels (e.g., emotional vs unsupported)

These metrics are appropriate because:

* Emotional vs unsupported often overlap in tone vs structure
* Weakly_supported vs well_supported depends on depth, not just correctness
* Class imbalance may bias accuracy, so per-class metrics are necessary

---

## Baseline Plan

Baseline will be a **zero-shot LLM classifier** using the prompt:

> “Classify the following music-related post into one of: emotional, unsupported, weakly_supported, well_supported. Return only the label.”

Baseline outputs will be collected on the same test set used for evaluation of the fine-tuned model.

---

## Definition of Success

The classifier will be considered **good enough for deployment** if:

* The final models accuracy ≥ **80%**
* No single class has F1-score below **0.75**
* Confusion between adjacent labels (unsupported ↔ weakly_supported) is significantly reduced compared to baseline
* Model performs consistently on unseen subreddit data (not just training distribution)

At this level, the model would be useful for:

* Moderation support in music communities
* Aggregating structured vs emotional opinions
* Filtering discussion quality for analysis tools

---

## AI Tool Plan

### 1. Label Stress-Testing

I will use an LLM to generate borderline examples between:

* emotional vs unsupported
* unsupported vs weakly_supported
* weakly_supported vs well_supported

These will be used to refine label definitions before dataset finalization.

---

### 2. Annotation Assistance

I will optionally pre-label batches using an LLM, but:

* Every label will be manually reviewed
* Corrections will be tracked in a separate “human override” log for transparency

---

### 3. Failure Analysis

After evaluation, I will feed misclassified examples into an LLM to identify:

* repeated confusion patterns
* label boundary failures
* missing feature signals in definitions

All identified patterns will be manually verified against the dataset before being included in the final report.

