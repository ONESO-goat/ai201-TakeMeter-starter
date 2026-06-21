
# README.md

# Opinion Reasoning Classifier

## Project Overview

This project explores whether machine learning models can distinguish between different levels of reasoning quality in opinion-based text. The task is not to determine whether an opinion is correct or incorrect, but rather to classify how well the opinion is supported.

The dataset was designed around online discussion posts and comments, particularly music-related discourse collected from communities such as r/LetsTalkMusic and related music discussion spaces. I also used post under r/AskReddit filtering by the words "Hot Take" as there will be a very diverse set of takes. Posts were annotated according to the amount of reasoning and evidence used to support a claim.

The project compares:

1. A zero-shot LLM baseline (Groq)
2. A fine-tuned DistilBERT classifier

The goal was to determine whether supervised fine-tuning could outperform a strong prompting-based baseline on this reasoning-quality classification task.

---

# Problem Definition

The classifier predicts one of four labels:

| Label            | Description                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------- |
| emotional        | Primarily expresses feelings, hype, insults, or subjective reactions without structured reasoning |
| unsupported      | Makes a claim without evidence or justification                                                   |
| weakly_supported | Includes some evidence or reasoning, but support is limited or incomplete                         |
| well_supported   | Includes clear reasoning, multiple supporting points, or substantial evidence                     |

The distinction focuses on argument quality rather than sentiment.

For example:

---
### Emotional

* Example 1: “This artist is complete trash.”
* Example 2: “Rock is literally the greatest genre ever made.”
  
---

### Unsupported

* Example 1: “Rock music is for depressed people.”
* Example 2: “Billie Eilish fell off.”

---

**Weakly Supported**

* Example 1: “Travis Scott is popular because of his production style and festival performances.”
* Example 2: “This band is bad because they haven’t had a hit in years.”

---

**Well Supported**

* Example 1: “Hip hop evolved through decades of cultural expression, lyrical innovation, and regional influence, shaping global music trends across multiple eras.”
* Example 2: “This artist charted on the Billboard Top 40 for multiple consecutive releases, showing sustained commercial success and audience reach.”

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

# Dataset

## Data size

Total dataset size: **200** examples

| Label            | Count |
| ---------------- | ----- |
| emotional        | 50    |
| unsupported      | 50    |
| weakly_supported | 50    |
| well_supported   | 50    |
| Total            | 200   |

**50** examples each label inside the dataset.

## Data Sources

Examples were collected from:

* r/LetsTalkMusic
* Music related Reddit discussions
* r/askReddit, filtered by "Hot Takes"
* Synthetic examples created to balance classes and stress-test label boundaries

## Dataset Design

The dataset was intentionally balanced across labels to reduce majority-class bias and to ensure each reasoning level was adequately represented during training.

Special attention was given to difficult boundaries:

* emotional vs unsupported
* unsupported vs weakly_supported
* weakly_supported vs well_supported

These boundaries represent increasing levels of evidence rather than completely separate categories.

---

# Baseline Model

The baseline was a zero-shot classifier using a large language model accessed through Groq.

Prompt:

> Classify the following post into one of: emotional, unsupported, weakly_supported, well_supported. Return only the label.

The baseline was evaluated on the same held-out test set used for the fine-tuned model.

---

# Fine-Tuned Model

Model:

* DistilBERT
* Sequence classification head
* Four output classes

**Test 1** Training summary:

| Epoch | Train Loss | Validation Loss | Validation Accuracy |
| ----- | ---------- | --------------- | ------------------- |
| 1     | No log     | 1.388           | 0.233               |
| 2     | 1.394      | 1.365           | 0.300               |
| 3     | 1.365      | 1.319           | 0.533               |

**First** test accuracy:

**53.3%**

# Evaluation Results

## Overall Accuracy

| Model                   | Accuracy |
| ----------------------- | -------- |
| Zero-shot Groq Baseline | 0.833    |
| Fine-tuned DistilBERT   | 0.533    |

The fine-tuned model underperformed the baseline by 30 percentage points.

This represents a significant regression relative to the original goal of outperforming or matching the zero-shot classifier.

---

**Final** test accuracy:

Training summary:

| Epoch | Train Loss | Validation Loss | Validation Accuracy |
| ----- | ---------- | --------------- | ------------------- |
| 1     | No log     | 0.715           | 0.933               |
| 2     | 0.614      | 0.636           | 0.933               |
| 3     | 0.556      | 0.522           | 0.933               |



**93.3%**

# Evaluation Results

## Overall Accuracy

| Model                   | Accuracy |
| ----------------------- | -------- |
| Zero-shot Groq Baseline | 0.950    |
| Fine-tuned DistilBERT   | 0.933    |

The fine-tuned model performed well after some re-training.

This represents a significant boost in success for the model.

---

# Further down the README we will focus on the **first** test, this is for more insight.

---

# Per-Class Metrics (Baseline)

| Label            | Precision | Recall | F1   |
| ---------------- | --------- | ------ | ---- |
| unsupported      | 0.80      | 0.50   | 0.62 |
| weakly_supported | 1.00      | 1.00   | 1.00 |
| well_supported   | 1.00      | 1.00   | 1.00 |
| emotional        | 0.60      | 0.86   | 0.71 |

Overall Accuracy: **0.833**

Macro F1: **0.83**

---

# Per-Class Metrics (Fine-Tuned Model)

The fine-tuned model achieved 53.3% accuracy for its first run then 93.3% its final turn. Error analysis throughout the process revealed that most failures involved predicting **unsupported** regardless of the true class.

This produced reasonable recall for unsupported claims but significantly harmed recall for emotional and weakly_supported examples.

The model showed evidence of collapsing toward the unsupported class when confidence was low.

---

# Fine-Tuned Model Confusion Matrix

During the first run, the confusion matrix below summarizes the major error patterns observed during evaluation.

| Actual \ Predicted | Unsupported | Weakly Supported | Well Supported | Emotional |
| ------------------ | ----------- | ---------------- | -------------- | --------- |
| Unsupported        | 5           | 1                | 1              | 1         |
| Weakly Supported   | 5           | 2                | 0              | 0         |
| Well Supported     | 3           | 1                | 4              | 0         |
| Emotional          | 5           | 0                | 1              | 1         |

The most important pattern is:

**Weakly Supported → Unsupported**

and

**Emotional → Unsupported**

These account for a large percentage of total errors.

---

# AI-Assisted Failure Analysis

To identify error patterns, I used an LLM to analyze the model's misclassified examples.

The AI tool suggested several possible explanations:

1. The model defaults to unsupported when evidence is brief.
2. Short posts are difficult because emotional language and unsupported claims often contain similar wording.
3. Weakly supported examples containing only one supporting fact are regularly mistaken for unsupported claims.

I manually reviewed the examples to verify these my assumptions.

### Verified Findings

The first and third patterns were strongly supported by the data.

Most weakly_supported examples contained exactly one supporting reason, making them structurally similar to unsupported claims.

The model appeared unable to distinguish:

> claim + one piece of evidence

from

> claim only

### Discarded Finding

The AI initially suggested sarcasm might be a major issue.

After reviewing the errors, very few examples actually contained sarcasm, well not obvious sarasm. This pattern was discarded because it was not supported by the dataset.

---

# Error Analysis

## Example 1

**Text**

> Youth participation in traditional team sports has declined steadily since 2008, with multiple surveys showing children increasingly preferring individual activities, gaming, and unstructured recreation.

**True Label**

well_supported

**Predicted**

unsupported

### Analysis

This example contains multiple supporting facts and references to survey evidence.

The model likely focused on the claim itself rather than the supporting structure.

This suggests the model did not reliably learn indicators of evidence-based reasoning.

To improve performance, more well_supported examples containing statistics, studies, and multi-sentence reasoning would likely be needed.

---

## Example 2

**Text**

> This label is failing because three of its artists left this year.

**True Label**

weakly_supported

**Predicted**

unsupported

### Analysis

The statement includes a supporting reason:

> three artists left this year

However, the support is limited, therefore it belongs in weakly_supported.

This error highlights the model's primary failure mode:

confusing unsupported claims with minimal supporting claims.

The boundary itself is difficult because both classes often contain only one sentence.

Improvement would require more examples demonstrating the difference between:

* no evidence
* one supporting fact

---

## Example 3

**Text**

> Pop music is garbage.

**True Label**

emotional

**Predicted**

unsupported

### Analysis

This example contains a strong emotional judgment without any evidence.

Humans can easily identify the emotional tone.

The model appears to focus primarily on whether evidence exists rather than recognizing emotional language.

This suggests emotional examples may have been underrepresented or insufficiently diverse during training.

Additional training examples containing insults, hype, exaggeration, and emotional reactions could strengthen this distinction.

---

# Sample Classifications

Examples below were run through the fine-tuned model.

| Example                                                                                | Predicted Label | Confidence |
| -------------------------------------------------------------------------------------- | --------------- | ---------- |
| "This band is horrible they didn't have a hit song in a decade"                        | unsupported     | 0.28       |
| "This rapper is declining because his last album debuted lower than previous projects" | unsupported     | 0.27       |
| "Hip hop is great because it dominates charts and has massive cultural influence"      | unsupported     | 0.28       |
| "That product is the worst thing I've ever bought"                                     | unsupported     | 0.27       |
| "Youth participation in traditional team sports has declined steadily since 2008..."   | unsupported     | 0.30       |

A notable observation is that confidence scores remained clustered around 0.27–0.30, indicating the model was often uncertain even when making predictions.

This supports the idea that the classifier had not learned strong decision boundaries between classes.

---

# Reflection: What the Model Learned vs. What I Intended

The intended task was to classify reasoning quality.

However, the trained model appears to have learned something closer to:

> "Does this text contain obvious, explicit evidence?"

rather than:

> "How strong is the reasoning being used?"

As a result:

* unsupported became the default prediction
* weakly_supported examples were frequently collapsed into unsupported
* emotional examples were treated as unsupported whenever no evidence was present

The model partially learned the evidence hierarchy but failed to capture emotional tone and nuanced reasoning quality.

This indicates the model learned a simplified version of the labeling scheme rather than the full conceptual distinction.

---

# Spec Reflection

## How the Spec Helped

The project specification required explicit label definitions, edge-case documentation, and confusion-matrix analysis.

This significantly improved annotation consistency because difficult examples were identified before training began rather than during evaluation.

The requirement to define edge cases also made it easier to interpret model failures later.

## How the Implementation Diverged

The original plan expected the fine-tuned model to outperform the baseline.

Instead, the zero-shot baseline substantially outperformed DistilBERT.

Because of this result, the final project focused more heavily on analyzing why fine-tuning failed rather than demonstrating a successful supervised classifier.

This divergence became an important finding itself: for small datasets and nuanced reasoning tasks, a strong LLM baseline may outperform a traditionally fine-tuned transformer.

---

# AI Usage Disclosure


# LLMs

* ChatGPT (primary helper)
* Claude

## Instance 1: Label Stress Testing

I used Claude to generate borderline examples between:

* emotional vs unsupported
* unsupported vs weakly_supported
* weakly_supported vs well_supported

The generated examples helped reveal ambiguous wording and motivated refinements to the annotation guidelines.

Several generated examples were rejected because they mixed multiple labels simultaneously.

---

## Instance 2: Failure Pattern Analysis

After evaluation, I provided the misclassified examples to ChatGPT and asked it to identify recurring error patterns.

The model suggested:

* unsupported becoming a default prediction
* weakly_supported collapsing into unsupported
* possible sarcasm-related errors

After manual review, I confirmed the first two findings but rejected the sarcasm explanation because it was not supported by the actual examples.

The final analysis in this report includes only patterns that were manually verified.

---

# Conclusion

The project successfully produced a reasoning-quality classification dataset and evaluated both zero-shot and supervised approaches.


The initial DistilBERT model achieved an accuracy of only

**53.3%**

revealing substantial confusion between unsupported, emotional, and weakly_supported examples. 

After revising the dataset and retraining, the final accuracy the DistilBERT model achieved was

**93.3%** 

compared to 95.0% for the Groq baseline. The analysis of the initial model's failures helped guide these improvements.


Error analysis revealed that the fine-tuned model struggled to find the difference between unsupported, weakly_supported, and emotional posts, frequently collapsing predictions into unsupported.

Although the supervised model underperformed expectations, the project provided valuable insight into the challenges of learning nuanced reasoning distinctions from a relatively small dataset and demonstrated the importance of comparing fine-tuning approaches against strong LLM baselines. This was a good learning process for me, understanding the training process in AI.
