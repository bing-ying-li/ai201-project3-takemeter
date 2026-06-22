# TakeMeter Project Planning

## Community

I chose the public Reddit community r/4x4.

This community discusses four-wheel-drive vehicles, off-road driving,
tyre pressure, suspension, recovery equipment, vehicle modifications,
and ownership experiences.

This community is suitable for a classification project because some
comments provide detailed technical explanations, some give opinions
without evidence, and others are only short emotional reactions.

## Labels

### reasoned_contribution

A comment that supports its opinion or recommendation with technical
reasoning, specific details, a meaningful comparison, or a clear
personal experience.

Examples:

- Lower tyre pressure increases the tyre contact area and can improve
  traction on sand.
- A rear differential locker improves traction because it allows both
  rear wheels to receive power.

### unsupported_take

A comment that makes a clear opinion, judgment, or recommendation
without enough explanation or evidence.

Examples:

- The Hilux is the best off-road vehicle.
- Just use 18 psi everywhere.

### reaction

A comment that mainly expresses emotion, humour, praise, or criticism
without providing a meaningful argument.

Examples:

- That build looks amazing!
- Bro really sent it 😂

## Hard Edge Cases

The most difficult boundary is between reasoned_contribution and
unsupported_take.

A comment will only be labeled reasoned_contribution when it explains
why the claim is reasonable or includes specific context and results.

For example:

"Use 18 psi. I have never had a problem."

This will be labeled unsupported_take because it does not explain the
vehicle, tyre type, terrain, speed, or reason for using 18 psi.

## Data Collection Plan

I will collect at least 210 public posts or comments from r/4x4.

I will aim to collect approximately:

- 70 reasoned_contribution examples
- 70 unsupported_take examples
- 70 reaction examples

The dataset will contain the columns text, label, notes, and source_url.

I will personally review every example before including it in the final
dataset.

## Evaluation Metrics

I will evaluate the model using:

- Overall accuracy
- Precision for each label
- Recall for each label
- F1 score for each label
- Macro-average F1 score
- Confusion matrix
- Analysis of incorrect predictions

Accuracy alone is not enough because the model may perform well on one
label but poorly on another.

## Definition of Success

I will consider the classifier successful if:

- Overall accuracy is at least 70%
- Macro-average F1 is at least 0.65
- No individual label has an F1 score below 0.60
- The fine-tuned model performs better than the zero-shot baseline

## AI Tool Plan

I will use ChatGPT to test difficult examples that sit between two
labels.

I may use ChatGPT to suggest preliminary labels, but I will personally
review and correct every label.

After model training, I will use ChatGPT to identify possible patterns
in the model's incorrect predictions. I will verify these patterns
myself before including them in the evaluation report.
