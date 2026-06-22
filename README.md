# TakeMeter — Motor Vehicle Maintenance & Repair Classifier

## Project Overview

TakeMeter is a text classification project that evaluates the main purpose of posts and comments from the Motor Vehicle Maintenance & Repair community.

The classifier assigns each post to one of three categories:

- `diagnostic_explanation`
- `actionable_suggestion`
- `clarification_request`

The project compares a fine-tuned DistilBERT model with a zero-shot Groq baseline.

---

## Community

I chose the Motor Vehicle Maintenance & Repair community because its discussions contain different types of technical communication. Some users explain why a vehicle problem happens, some recommend a repair or test, and others request more information before giving a diagnosis.

These differences make the community suitable for a text classification task.

---

## Label Taxonomy

### `diagnostic_explanation`

A post that mainly explains a likely cause, diagnosis, technical mechanism, or reason behind a vehicle problem.

Examples:

- “The battery voltage drops because one of the cells is damaged.”
- “Blocked A/C drains can cause water to collect behind the dashboard.”

### `actionable_suggestion`

A post that mainly recommends a specific test, repair, maintenance step, safety action, or next procedure.

Examples:

- “Check the battery voltage while the headlights are turned on.”
- “Do not drive the vehicle until the damaged tyre is replaced.”

### `clarification_request`

A post that mainly asks for missing symptoms, measurements, vehicle details, repair history, or other information needed before diagnosis.

Examples:

- “What is the year, make, and model of the vehicle?”
- “Does the noise happen when the brakes are applied?”

### Decision Rules

1. If the post mainly asks for missing information, label it `clarification_request`.
2. Otherwise, if it mainly tells the user what action to take, label it `actionable_suggestion`.
3. Otherwise, if it mainly explains what is happening or why, label it `diagnostic_explanation`.

---

## Data Collection

The dataset contains public posts and comments from the Motor Vehicle Maintenance & Repair community on Mechanics Stack Exchange.

The final dataset contains 260 examples. Each row includes the post text, label, source URL, notes, and review information.

The dataset was stored in a single CSV file and automatically divided into:

| Split      | Examples | Percentage |
| ---------- | -------: | ---------: |
| Training   |      182 |        70% |
| Validation |       39 |        15% |
| Test       |       39 |        15% |

### Label Distribution

| Label                    |   Count | Percentage |
| ------------------------ | ------: | ---------: |
| `diagnostic_explanation` |     134 |      51.5% |
| `clarification_request`  |      71 |      27.3% |
| `actionable_suggestion`  |      55 |      21.2% |
| **Total**                | **260** |   **100%** |

No class represented more than 70% of the dataset, but `diagnostic_explanation` was still noticeably larger than the other classes.

### Labeling Process

The dataset used AI-assisted pre-labeling. The labels and examples were checked during dataset preparation and error analysis.

Some inconsistent labels remained in the dataset. These became visible when the model made predictions that appeared more consistent with the written definitions than the original annotations.

---

## Difficult-to-Label Examples

### Example 1

> “The first thing I'd establish is to work out if the hub can be removed from the car with the wheel still attached. I encountered a similar scenario some years ago...”

Possible labels:

- `actionable_suggestion`
- `diagnostic_explanation`

Decision:

This was labeled `actionable_suggestion` because its main purpose is to recommend a specific first action. The explanation and personal experience support the recommendation.

### Example 2

> “I've seen somewhat the opposite to this, I think—a piston-top that has been built up and given a sloped surface, though that may have been more to do with dead-space reduction?”

Possible labels:

- `diagnostic_explanation`
- `clarification_request`

Decision:

This was labeled `diagnostic_explanation` because it proposes a possible technical explanation. The question mark and uncertainty do not make its main purpose a request for missing information.

### Example 3

> “I've done it. I don't recommend it. Sell it and buy another one instead.”

Possible labels:

- `diagnostic_explanation`
- `actionable_suggestion`

Decision:

According to the final decision rules, this is closer to `actionable_suggestion` because it gives a direct recommendation. This example revealed possible annotation inconsistency in the dataset.

---

## Fine-Tuning Approach

I used `distilbert-base-uncased` as the starting model.

The original model was trained for three epochs, but it predicted `diagnostic_explanation` for almost every test example. Its first test accuracy was 0.538, and the `actionable_suggestion` class had an F1-score of 0.

To address this problem, I used class-weighted cross-entropy loss so that mistakes on the smaller classes had greater importance.

### Final Training Setup

| Hyperparameter        | Value                     |
| --------------------- | ------------------------- |
| Base model            | `distilbert-base-uncased` |
| Epochs                | 5                         |
| Learning rate         | `2e-5`                    |
| Training batch size   | 8                         |
| Evaluation batch size | 8                         |
| Weight decay          | 0.01                      |
| Loss function         | Weighted cross-entropy    |
| Best-model metric     | Macro F1                  |

The best validation result occurred at epoch 4:

- Validation accuracy: 0.872
- Validation macro F1: 0.852

The fifth epoch performed worse, suggesting that the model was beginning to overfit. The best checkpoint was used for test evaluation.

---

## Zero-Shot Baseline

The zero-shot baseline used Groq's `llama-3.3-70b-versatile`.

The prompt included:

- The community name
- Definitions of all three labels
- One example for each label
- Decision rules
- An instruction to return only the exact label name

### Baseline Prompt

```text
You are classifying public posts and comments from the Motor Vehicle
Maintenance & Repair community.

Assign each post to exactly one of the following categories based on
its main purpose.

diagnostic_explanation:
The post mainly explains a likely cause, diagnosis, technical mechanism,
or reason why a vehicle problem occurs.

actionable_suggestion:
The post mainly recommends a concrete test, repair, maintenance step,
safety action, or next procedure.

clarification_request:
The post mainly asks for missing symptoms, measurements, vehicle details,
history, or other information needed before diagnosing the problem.

Decision rules:
1. If the main purpose is asking for missing information,
   choose clarification_request.
2. Otherwise, if the main purpose is telling the user what action to take,
   choose actionable_suggestion.
3. Otherwise, choose diagnostic_explanation.

Respond with only one exact label name.
Do not explain your reasoning.
```

The Groq free-tier daily token limit was reached during evaluation. Only 7 of the 39 test examples were completed under the original consistent baseline run.

The Groq results are therefore reported as a partial baseline and should not be treated as a complete comparison.

---

## Evaluation Report

### Overall Results

| Model                   | Accuracy | Macro F1 | Test Coverage |
| ----------------------- | -------: | -------: | ------------: |
| Fine-tuned DistilBERT   |    0.769 |    0.730 |         39/39 |
| Groq zero-shot baseline |    0.714 |    0.463 |          7/39 |

The fine-tuned model was evaluated on the complete test set. The Groq result only covers 17.9% of the test set, so it is provisional and not directly comparable with the complete DistilBERT result.

### Fine-Tuned Model Per-Class Results

| Label                    | Precision |   Recall | F1-score | Support |
| ------------------------ | --------: | -------: | -------: | ------: |
| `diagnostic_explanation` |      0.89 |     0.80 |     0.84 |      20 |
| `actionable_suggestion`  |      0.80 |     0.44 |     0.57 |       9 |
| `clarification_request`  |      0.62 |     1.00 |     0.77 |      10 |
| **Macro average**        |  **0.77** | **0.75** | **0.73** |  **39** |

The model performed best on `diagnostic_explanation`.

It correctly found every `clarification_request`, producing a recall of 1.00. However, its precision of 0.62 shows that it sometimes predicted this class for posts that belonged to another category.

The weakest category was `actionable_suggestion`. Only four of the nine actionable suggestions were correctly identified.

### Fine-Tuned Model Confusion Matrix

| True label \ Predicted label | `diagnostic_explanation` | `actionable_suggestion` | `clarification_request` |
| ---------------------------- | -----------------------: | ----------------------: | ----------------------: |
| `diagnostic_explanation`     |                       16 |                       1 |                       3 |
| `actionable_suggestion`      |                        2 |                       4 |                       3 |
| `clarification_request`      |                        0 |                       0 |                      10 |

### Partial Groq Baseline Confusion Matrix

| True label \ Predicted label | `diagnostic_explanation` | `actionable_suggestion` | `clarification_request` |
| ---------------------------- | -----------------------: | ----------------------: | ----------------------: |
| `diagnostic_explanation`     |                        4 |                       0 |                       1 |
| `actionable_suggestion`      |                        0 |                       0 |                       1 |
| `clarification_request`      |                        0 |                       0 |                       1 |

The partial Groq baseline handled most diagnostic explanations correctly but did not correctly identify the single actionable suggestion in the available sample.

---

## Failure Analysis

### Failure 1: Question-like wording

**Post:**

> “I've seen somewhat the opposite to this, I think—a piston-top that has been built up and given a sloped surface, though that may have been more to do with dead-space reduction?”

**True label:** `diagnostic_explanation`
**Predicted label:** `clarification_request`
**Confidence:** 0.58

The model likely focused on uncertainty phrases such as “I think,” “may,” and the question mark. However, the post's main purpose is to suggest a possible technical explanation.

### Failure 2: Advice mixed with explanation

**Post:**

> “Most of the time you won't need heavy braking in traffic. You can avoid stopping and starting by driving more calmly, leaving plenty of distance, and taking your foot off the accelerator early...”

**True label:** `actionable_suggestion`
**Predicted label:** `diagnostic_explanation`
**Confidence:** 0.52

The post includes several clear recommendations. However, it also explains braking, gears, and transmission wear. The model focused more on the technical explanation than on the main actionable purpose.

### Failure 3: Possible annotation inconsistency

**Post:**

> “I've done it. I don't recommend it. Sell it and buy another one instead.”

**True label:** `diagnostic_explanation`
**Predicted label:** `actionable_suggestion`

The model's prediction appears reasonable because the post directly recommends selling the vehicle and buying another one. This example may represent a labeling problem rather than a model failure.

### Systematic Error Pattern

The most common error involved `actionable_suggestion`.

The model struggled when advice was combined with:

- Technical explanations
- Personal experience
- Uncertain language
- Questions

It also sometimes treated question marks and uncertain words as strong signals for `clarification_request`.

More training examples containing both advice and technical reasoning could improve this boundary.

---

## Sample Classifications

| Post                                                                                                      | True label               | Predicted label          | Confidence | Correct |
| --------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------------ | ---------: | ------- |
| “By oil rag do you mean even a microfiber cloth or any rag? Are you saying to pour the oil into the rag?” | `clarification_request`  | `clarification_request`  |       0.63 | Yes     |
| “Thank you! One last question, do I have to worry about any damage to my transmission?”                   | `clarification_request`  | `clarification_request`  |       0.63 | Yes     |
| “Which engine is in the car? Is it the 455ci V8? Also, did you install a new battery?”                    | `clarification_request`  | `clarification_request`  |       0.62 | Yes     |
| “I've seen somewhat the opposite to this, I think—a piston-top that has been built up...”                 | `diagnostic_explanation` | `clarification_request`  |       0.58 | No      |
| “You can avoid stopping and starting by driving more calmly and leaving plenty of distance...”            | `actionable_suggestion`  | `diagnostic_explanation` |       0.52 | No      |

### Correct Prediction Explanation

The first example was correctly classified as `clarification_request` because its main purpose is to ask what kind of cloth should be used and how the oil should be applied.

---

## Reflection: What the Model Learned

The model learned to identify clarification requests and technical explanations fairly well. It performed best on `diagnostic_explanation` and correctly found all clarification requests in the test set.

Its main weakness was `actionable_suggestion`. The model sometimes treated advice as an explanation when the post included technical reasoning.

It also sometimes used question marks and uncertain words as shortcuts for predicting `clarification_request`.

Some errors were caused by inconsistent labels in the dataset. Overall, the model learned the main differences between the categories but struggled with posts that mixed advice, explanations, and questions.

---

## Spec Reflection

The planning document helped me define the labels and decision rules before training the model. This made the annotation and evaluation process more consistent.

My implementation changed from the original plan because the first model predicted the majority class too often. I added class weights and increased training to five epochs. This improved test accuracy from 0.538 to 0.769 and increased macro F1 from 0.29 to 0.73.

---

## AI Usage

I used ChatGPT to help improve the label definitions and identify unclear boundaries between categories. I reviewed the suggestions and decided which rules to use.

I also used ChatGPT to help debug the training code and analyze the model's errors. Based on the evaluation results, I added class weights and changed the model-selection metric to macro F1.

The dataset used AI-assisted pre-labeling. Error analysis revealed that some inconsistent labels remained, and these limitations are documented in this README.

---

## Files

The repository includes:

```text
planning.md
README.md
takemeter_ai_reviewed_all_260.csv
evaluation_results.json
confusion_matrix.png
wrong_predictions.csv
sample_classifications.csv
groq_partial_baseline_predictions.csv
ai201_project3_takemeter_starter_clean.ipynb
```

---

## How to Run

1. Open the notebook in Google Colab.
2. Select a T4 GPU runtime.
3. Install the required dependencies.
4. Upload `takemeter_ai_reviewed_all_260.csv`.
5. Run the data preparation cells.
6. Train the DistilBERT model.
7. Run the test-set evaluation.
8. Add `GROQ_API_KEY` to Colab Secrets before running the baseline section.

---

## Demo Video

Demo video:

<img src='Animation.gif' title='Video Walkthrough' width='' alt='Video Walkthrough' />

## Challenges

The main difficulty was class imbalance. The first model predicted the largest class too often, so I added class weights.

Another difficulty was separating actionable suggestions from diagnostic explanations because many posts contained both advice and technical reasoning.

I also found some inconsistent labels in the dataset, which made evaluation harder.

Finally, the Groq baseline could only process 7 of 39 test examples because of the API token limit.

---

## Author

**Name:** Bingying Li
**Course:** AI201
**Project:** TakeMeter
